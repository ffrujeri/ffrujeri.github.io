---
layout: archive
title: "Environment, Task, Agent: Decomposing the POMDP for LLM Agents (and Who Owns the Tools)"
date: 2026-11-21
permalink: /posts/2026/11/environment-task-agent
excerpt: "A score on an agentic benchmark is a function of four things: the environment, the task set, the harness and the model. This post splits the POMDP along those lines, shows how to classify any change to a run, and works through the hardest boundary case: tools that a black-box harness brings with it."
tags:
- reinforcement-learning
- ai-agents
- pomdp
- evaluation
- systems-engineering
- large-language-models
---

"Model X scores 62% on benchmark Y" sounds like one fact about one thing. For an LLM agent it is a fact about four things at once: the world the agent acted in, the problems it was given, the scaffold wrapped around the model, and the model. Change any one of them and the number moves. Most confusion I see in agent evaluation and agent RL comes from not knowing which one moved.

Classical deep RL had the same four things, but the lines between them were easy to see. One `env` class held the world, its reward and its start state, and the agent was a small program around a network. LLM agents split the first across several owners and turned the second into a product. This post is the decomposition I now use to keep them apart: three parts, one rule for each, and one hard boundary case, which is tools.

---

### 1. What LLM Agents Did to the Textbook Loop

The textbook object is a POMDP:

$$\mathcal{M} = \langle S, A, T, R, \Omega, O, \rho_0 \rangle$$

states, actions, transition dynamics, reward, observations, the observation function and an initial-state distribution. In a Gymnasium environment the world's side of this lives behind `reset()` and `step()`: CartPole's physics, its reward and its start state ship in one class.

The agent's side was never just a network. In keras-rl, a `DQNAgent` is assembled from four objects: the network, a replay memory that also stacks the last frames into a state, an exploration policy, and a `Processor`, which the library describes as "a coupling mechanism between an `Agent` and its `Env`". For Atari, the processor resizes the frames and clips the rewards, so that neither the agent nor the environment has to change to fit the other. The agent was always an interface around the policy model. Russell and Norvig put it in one line: agent = architecture + program.

LLM agents changed both sides. The symbols that one `env` class used to hold now come from different owners, and the program around the model became a product of its own:

```text
SYMBOL            WHERE IT ACTUALLY LIVES
S, T, Ω, O        a container image, a database, a browser, a simulated user
A                 tool schemas from the benchmark... and tools built into the harness
ρ₀                a dataset row: which repo commit, which DB snapshot, which persona
R                 hidden tests, a rubric, an LLM judge
the policy        a model endpoint, wrapped in a CLI harness with its own loop and memory
```

I argued in an [earlier post](/posts/2026/09/verifiers-vs-gymnasium-unifying-rlvr) that $T$ and $R$ decouple naturally in tool-use systems: the sandbox executes, the verifier judges. This post takes the next step. Once the pieces have different owners, we need names for the groups, and the names need to mean something.

---

### 2. Three Parts

**The Environment is the world.** It is the POMDP with the reward and the start state removed:

$$\mathcal{E} = \langle S, A, T, \Omega, O \rangle$$

its state, the actions it accepts, how actions change the state, and what the agent gets to see. A Linux sandbox with a repository in it is an environment. So is a customer-service backend with a database and a simulated customer. The environment has no opinion about what you should do in it.

**A Task is one problem in that world:**

$$\tau = \langle \rho_0, g, R, H \rangle$$

an initial state, a goal $g$ (the instruction the agent reads), a reward rule, and the limits $H$ that bound the attempt: turns, wall-clock time, tool-call caps. SWE-bench makes the split concrete: the environment is a container that can run a Python project; a task is a commit to start from, an issue to read, and the tests that decide the reward.

A **taskset** is a collection of tasks over the same environment. In RL terms this is close to a contextual MDP: a family of problems indexed by a context, here with shared dynamics, where the context selects $\rho_0$ and $R$. A **benchmark** is a taskset with fixed rules (split, prompt wording, scoring, limits) so that scores can be compared. A benchmark is not a kind of environment. It is a set of questions asked inside one.

**The Agent is the harness plus the policy:**

$$\text{Agent} = \langle h, \pi_\theta \rangle$$

The model $\pi_\theta$ generates tokens. The harness $h$ is everything that turns a history of observations into a prompt and turns tokens back into actions: the loop, the system prompt, context compaction, tool routing, retries, stop conditions.

```text
              TASK  (one of many in a taskset)
   ┌──────────────────────────────────────────────────┐
   │ initial state ρ₀ · goal g · reward R · limits H  │
   └──────┬─────────────────────────────┬───────────┬─┘
          │ goal                        │ seeds     │ scores
          ▼                             ▼           ▼
   AGENT                          ENVIRONMENT
 ┌───────────────────────┐ action ┌─────────────────────────┐
 │ harness h             │ ─────► │ state S                 │
 │   loop, context,      │        │ actions A (tools)       │
 │   memory, tool routing│ ◄───── │ transitions T           │
 │ policy π_θ (the model)│  obs   │ observations Ω, O       │
 └───────────────────────┘        └─────────────────────────┘
```

Two consequences follow immediately.

First, **evaluation and training use the same environment**. Whether you average the reward into a leaderboard number or backpropagate it through a policy gradient is a decision about what to do with $R$. It changes nothing about $\mathcal{E}$ or $\tau$. A design that has "eval environments" and "training environments" as different kinds of object has put the use of a thing into its identity.

Second, **a config is wiring, not identity**. A YAML file that says "run this taskset in this environment with this agent" names a composition. The folder it sits in is not an environment, and the file is not a benchmark.

---

### 3. The Agent Is Not the Policy

It is tempting to write Agent = policy and treat the harness as plumbing. That was not true for DQN, and the math says why. The policy in the POMDP sense maps histories to actions, $\pi(a_t \mid o_{1:t})$. The model alone does not implement that map. The composition does:

$$\pi(a_t \mid o_{1:t}) \;=\; \text{decode}\Big(\pi_\theta\big(\cdot \mid \text{context}(o_{1:t})\big)\Big)$$

The two functions around the model belong to the harness. `context` is the harness deciding what the model sees. In POMDP language it is the [belief-state](/posts/2026/07/pomdps-and-belief-states) approximator: a compaction step is a lossy belief update, and a memory file is a hand-written attempt at a sufficient statistic. `decode` is the harness turning tokens into tool calls. Training moves $\theta$. It does not move $h$, but $h$ shapes every trajectory the gradient sees.

The DQN agent from section 1 has the same shape in miniature:

| keras-rl `DQNAgent` part | Its job | In an LLM agent |
| --- | --- | --- |
| Network | The trainable function | The model $\pi_\theta$ |
| Memory, with a frame window | Builds a state from the recent history | `context`: prompt assembly, compaction, memory files |
| Exploration policy (ε-greedy) | Turns the network's output into an action | `decode`: tool-call parsing, retries, stop rules |
| Processor | Translates observations, actions and rewards between agent and environment | Chat template, tool-result formatting |

That gives two boundaries, and both are legitimate. Sutton and Barto note that the boundary "can be located at different places for different purposes." For the gradient, it sits at the model API: tokens are the only thing the model sets freely, so relative to the policy the harness behaves like part of the dynamics. For the experiment, it sits at the tool interface: the agent is harness plus model, and the environment starts at the world. This post uses the second, because that is the boundary that decides what a score compares. AlphaZero is the precedent: gradients flow only through the network, yet nobody calls the tree search part of the environment.

This is why the same model can score very differently under two harnesses on the same coding or computer-use benchmark, and why I think of a score as:

$$\text{score} = f(\text{taskset},\ \mathcal{E},\ h,\ \pi_\theta)$$

To compare models, hold the first three fixed. To compare harnesses, hold the taskset, the environment and the model fixed. A leaderboard row that changes two of the four at once measures neither. It also means "the agent is open source" is a claim about $h$ only; as I wrote [before](/posts/2026/08/the-open-source-agent-engine-paradox), the open harness and the closed model are different halves of the agent.

---

### 4. What Changed? A Classification

The practical payoff is a rule for any variant of a run: find the part that changed. The part tells you what the variant is and whether its score still compares.

| What changed | Part | What it means |
| --- | --- | --- |
| Harness: its loop, context strategy, own system prompt | Agent | An agent variant. Same task, same world. |
| Model weights | Policy | The thing under test. Compare with the harness fixed. |
| Goal wording, initial state, reward rule | Task | A different task. Scores do not compare. |
| Turn budget, time limit, tool-call cap | Task | Also a different task: the horizon is part of the problem. |
| Tools, sandbox contents, what the agent can reach | Environment | A different world. |
| Number of repeats | Sampling | Changes the variance, not the thing measured. |

Two cases come up constantly.

**Prompt adaptation.** A CLI harness owns its system prompt, so a benchmark's instructions that used to sit in the system message move into the user message. The goal text $g$ is the same; only its delivery changed. That is an agent variant.

**Limits as harness arguments.** A research-QA task allows 15 turns and three searches. Someone adds a config for another harness with 150 turns and no search cap, because those happen to be harness flags. Nothing in the taskset changed on disk, yet it is a different task: $H$ moved. Limits belong with the task so that they survive a harness swap.

A side effect of this table is that it needs no new vocabulary. It is tempting to coin words for how one config relates to another (a "variant", a "flavor", a "mode"). Every such relation can be read off the three parts: same environment and taskset, different agent; same environment, different taskset. If each run simply names its environment, its taskset and its agent, comparability is a lookup, and nobody has to learn a fourth noun.

---

### 5. Who Runs the Episode?

An episode has an order: reset the world to $\rho_0$, hand the goal to the agent, alternate actions and observations, stop, compute the reward, clean up. Something has to own that order. In the model, the answer is clear: **it is not the agent.**

Sutton and Barto put the reward outside the agent on purpose. The agent can know everything about how the reward is computed, but it cannot be the one computing it. The same goes for the initial state. Yet a common pattern in LLM benchmarks is an "agent" that resets its own world, runs its own loop, calls the grader itself and returns a number. It works, and it couples everything: you cannot swap the harness without rewriting the benchmark, and a failure in the grader is reported as a failure of the agent.

Simulated users are the sharpest version of this. In a τ-bench-style task, the policy talks to an LLM playing a customer with a hidden goal. From the evaluated agent's side, that customer is part of the environment: it is the transition function that produces the next observation, and its hidden persona is part of $\rho_0$. You can also model it as a second agent in a two-player game, with its own harness and model. Both are defensible. What is not defensible is running the simulator inside the evaluated agent's harness. When that happens, the simulator's tokens are billed to the policy, its outages are logged as the policy's failures, and no other harness can take the test.

The rule I use: the side that defines the task runs the episode. The agent acts only when it is asked to.

---

### 6. The Tool Problem: Whose Action Space?

Now the hard case. In the math, $A$ belongs to the environment. In practice, tools come from two directions:

- **Environment tools**: function schemas or an MCP server that the benchmark provides. Their implementation, and the state they touch, is under the experimenter's control.
- **Harness tools**: a black-box CLI agent arrives with a shell, file editing, web search and fetch, a todo list, memory, sub-agents. The experimenter did not write them and often cannot change them.

So which action space did the policy actually act in? Sutton and Barto gave the answer decades before anyone shipped a coding agent: *"anything that cannot be changed arbitrarily by the agent is considered to be outside of it and thus part of its environment,"* and the boundary *"represents the limit of the agent's absolute control, not of its knowledge."*

Apply that literally and the question stops being "who implemented the tool" and becomes "what state does the tool read or change":

| Built-in tool | State it touches | Part |
| --- | --- | --- |
| Todo list, memory notes, sub-agents, context compaction | The agent's own state, which it can rewrite at will | Agent. This is internal computation, like chain-of-thought. |
| Shell, file read and edit | The workspace, which answers back and gets graded | Environment, reached through an interface the harness implements. |
| Web search and fetch, package installs, external APIs | State outside the world the experimenter built | Environment, and it just got larger. |

A tool that reads or changes world state is **an interface onto the environment's action space, whoever implements it**. The harness can own the implementation. The effect always belongs to the environment.

The third row is where benchmarks quietly break. A closed-book QA benchmark is not closed-book if the harness ships a search tool. A repo-fixing task leaks if the fix is one `pip download` or one web fetch away. The task did not change and the model did not change; the environment grew a door.

---

### 7. Control the Reach, Not the Tool

You cannot control a black box's tools. You can control what they reach, and one mechanism covers most of it:

```text
 ┌────────────────── episode sandbox: the environment's state ──────────────────┐
 │                                                                              │
 │   ┌── harness (black box) ──────┐   shell, edit   ┌──────────────────────┐   │
 │   │ todo · memory · sub-agents  │ ──────────────► │ workspace            │   │
 │   │      (agent-internal)       │                 │ (the verifier reads) │   │
 │   └──────┬───────────────┬──────┘                 └──────────────────────┘   │
 │          │               │                                                   │
 └──────────┼───────────────┼────────────────────────────────────────── ✕ ──────┘
            │ model calls   │ environment tools                 everything else:
            ▼               ▼                                   closed unless the
      model endpoint    tool server                             environment opens it
```

Run the harness inside the sandbox the environment seeds, and let **the environment define that sandbox's network rule**. The built-in shell now acts on state the experimenter set up and the verifier can read. The built-in web tools hit a closed door. If a task genuinely needs the web, the environment offers search as one of its own tools, and every harness gets the same one.

Three details are worth knowing.

**The rule is the environment's even when the sandbox is the agent's.** On a pure reasoning benchmark the environment lends nothing, so a harness that needs a sandbox starts its own. Its network rule still decides what the policy can reach, so it is still part of the world's definition. Whoever starts the container, the environment says what the container may talk to.

**One leak a sandbox cannot see.** Some tools are executed by the model provider, not by the harness: hosted web search or code execution that runs on the provider's servers and returns results inside the model response. Those calls travel over the one connection you must leave open. If the benchmark is closed-book, the hosted tools have to be off in the request itself.

**When you cannot close it, say so.** If a harness's built-in tool reaches outside and cannot be disabled, the run happened in a larger environment. That is allowed. Reporting it under the original benchmark's name is not.

For training, all of this matters more than for evaluation. A policy optimized against an open door learns to use the door: fetching the upstream fix is a much easier gradient than learning to debug. And a world that includes the live internet is non-stationary, so two rollouts of the "same" task are not samples from the same MDP.

I would resist building more machinery than this until a real run demands it. Sandbox plus network rule, defined by the environment, is one concept. Per-tool allowlists negotiated between every harness and every benchmark is a combinatorial product.

---

### 8. Servers Are Plumbing

One last separation. Real systems run these parts as processes: a model server, an agent server, an environment server, sometimes a separate server for tools and grading, containers for the sandbox. It is easy to start reasoning in those boxes, and to say things like "the environment is the server that orders the episode."

That confuses transport with meaning. The decomposition above would be identical if everything ran in one Python process. Whether the tools and the verifier live in the same process as the episode loop or in another one is a scaling decision. Either way they implement the same thing: the environment and the task's reward rule. When a design discussion gets stuck, I find it helps to ask the question twice: once about the POMDP, and only then about the processes.

---

### 9. Questions I Now Use

When I read a benchmark result, an agent paper or a design doc, these are the checks I run:

1. **Two scores on the "same benchmark" differ by ten points. Which of the four moved?** (Environment, taskset, harness or model. If more than one, the comparison is not interpretable.)
2. **Where do the limits live?** (If the turn budget is a harness flag, swapping harnesses silently changed the task.)
3. **Who computed the reward, and who reset the world?** (If the answer is "the agent", the harness cannot be swapped and failures are misattributed.)
4. **Which tools did the policy have that the benchmark did not provide?** (List the harness's built-ins. Sort them: agent-internal, world state, outside reach.)
5. **What could the sandbox talk to?** (If nobody can answer, assume everything.)
6. **Were provider-hosted tools on?** (The sandbox cannot tell you.)
7. **A simulated user failed mid-episode. Whose failure is it?** (The environment's. It should be retried, not scored against the policy.)
8. **You moved from one process to five servers. What changed in the POMDP?** (Nothing. If something did, the architecture leaked into the semantics.)

None of this is new theory. It is the agent–environment boundary from chapter 3 of the RL textbook, applied to systems where the boundary is no longer drawn for us by a `step()` function. Keeping the three parts apart, and deciding each hard case by asking what state is being touched, is the simplest tool I know for settling design arguments about agent environments.

---

### References

- Sutton & Barto, *Reinforcement Learning: An Introduction*, 2nd ed., §3.1 "The Agent–Environment Interface"
- Russell & Norvig, *Artificial Intelligence: A Modern Approach*, ch. 2 "Intelligent Agents" (agent = architecture + program)
- [keras-rl](https://github.com/keras-rl/keras-rl): `Agent` and `Processor` in `rl/core.py`, and the Atari example in `examples/dqn_atari.py`
- Silver et al., [Mastering Chess and Shogi by Self-Play with a General Reinforcement Learning Algorithm](https://arxiv.org/abs/1712.01815) (AlphaZero)
- Kaelbling, Littman & Cassandra, "Planning and Acting in Partially Observable Stochastic Domains", *Artificial Intelligence*, 1998
- Hallak, Di Castro & Mannor, [Contextual Markov Decision Processes](https://arxiv.org/abs/1502.02259)
- [Gymnasium: A Standard Interface for Reinforcement Learning Environments](https://arxiv.org/abs/2407.17032)
- [SWE-bench](https://arxiv.org/abs/2310.06770) · [τ-bench](https://arxiv.org/abs/2406.12045) · [OSWorld](https://arxiv.org/abs/2404.07972) · [Humanity's Last Exam](https://arxiv.org/abs/2501.14249)
- [Model Context Protocol](https://modelcontextprotocol.io)
