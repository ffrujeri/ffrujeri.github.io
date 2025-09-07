---
layout: archive
title: 'Effective horizon in Reinforcement Learning'
date: 2025-09-06
permalink: /posts/2025/09/effective-horizon-rl/
excerpt: "Effective horizon in Reinforcement Learning"
tags:
  - rl
  - credit-assignment
---  

The **horizon is fundamentally a property of the *environment***, not of the agent. Here’s why:

## 1. **Environment-Defined Horizon**

* The environment specifies whether episodes end after a fixed number of steps (finite horizon) or continue indefinitely (infinite horizon).
* For example:

  * A chess game has a natural finite horizon (the game ends in a finite number of moves).
  * CartPole can run forever (until termination), so it’s modeled as an infinite-horizon MDP.

Thus, the **MDP definition itself includes the horizon** — either explicitly (finite episodes) or implicitly (continuing process).


## 2. **Agent’s Perspective**

* While the environment dictates the horizon, the **agent’s design interacts with it**:

  * In finite-horizon settings, optimal policies can be *non-stationary* (they depend on how many steps remain).
  * In infinite-horizon discounted settings, the optimal policy is *stationary*.
* The agent can *choose* to ignore long-term rewards (e.g., by setting a smaller discount factor), but that doesn’t change the environment’s horizon — it just changes how the agent values rewards.


## 3. **Example**

* **Environment property**: An episode ends after 200 steps → horizon = 200.
* **Agent choice**: Uses $\gamma = 0.9$, so it effectively only cares about the next \~10 steps (effective horizon).

So:

* **True horizon** = defined by the environment (termination structure).
* **Effective horizon** = shaped by the agent (via discounting or modeling choices).

---
In summary, the **horizon is primarily a property of the environment**, but the agent’s discount factor or planning depth determines its *effective* horizon of concern.


Continuing on examples, let's think about CartPole which is a **continuing control task** but is usually implemented with a max episode length (finite horizon). Let’s break down how **different values of $\gamma$** affect the learning dynamics:


### 1. **Low Discount Factor ($\gamma \ll 1$, e.g., 0.5–0.8)**

* **Effective horizon is short** ($\frac{1}{1-\gamma}$ ≈ 2–5 steps).
* The agent strongly prioritizes *immediate reward*.
* In CartPole, this means:

  * The agent learns quick, reflex-like corrections to prevent imminent pole falls.
  * It often ignores strategies that require short-term sacrifice for longer stability (e.g., moving the cart away to counteract pole drift).
* **Learning dynamics**: Fast convergence, but to *myopic policies* that may fail to generalize well.


### 2. **Moderate Discount Factor ($\gamma \approx 0.9–0.99$)**

* **Effective horizon is longer** ($\approx 10–100$ steps).
* The agent balances immediate and long-term stability.
* In CartPole:

  * The agent learns to not just react to the pole angle but also anticipate drift, moving the cart preemptively.
  * Policies are smoother and more robust, keeping the pole balanced for much longer.
* **Learning dynamics**: Slightly slower, but better long-term performance and higher returns.

### 3. **Near-Undiscounted ($\gamma \to 1.0$)**

* Effective horizon approaches infinity.
* The agent tries to optimize stability over the entire episode length.
* In CartPole:

  * The optimal strategy is to keep the pole upright indefinitely.
  * However, exploration becomes harder: since rewards far in the future matter almost as much as immediate ones, the *credit assignment problem* becomes severe.
  * Algorithms like Q-learning may converge more slowly or become unstable.
* **Learning dynamics**: High variance in updates, need for more careful exploration and function approximation.


### 4. **Interaction with Episode Termination**

* CartPole usually ends at 200 or 500 steps. Even with $\gamma=1$, the **true horizon is bounded** by the environment.
* So:

  * With $\gamma=1$, the agent tries to maximize survival up to that cap.
  * With lower $\gamma$, it acts as though the episode is much shorter, ignoring long-term risks.

---

### **Takeaways**

* **$\gamma$ controls the agent’s effective foresight.**
* Small $\gamma$ → greedy, reactive behavior, fast but suboptimal.
* Large $\gamma$ → farsighted, anticipatory behavior, slower but higher return.
* In CartPole, $\gamma \approx 0.95–0.99$ usually works best: it’s enough foresight to learn balancing, but not so large that learning becomes unstable.
---

# Policy Gradient Methods and $\gamma$

Policy gradient methods optimize the expected return:

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[ \sum_{t=0}^H \gamma^t r_t \right]
$$

The gradient estimate (REINFORCE-style) is:

$$
\nabla_\theta J(\theta) = \mathbb{E}\left[ \sum_{t=0}^H \nabla_\theta \log \pi_\theta(a_t|s_t) \, G_t \right]
$$

where

$$
G_t = \sum_{k=t}^H \gamma^{k-t} r_k
$$

is the **discounted return from time $t$**.

Thus, $\gamma$ directly influences **what signal is used to weight policy updates**.

---

## 1. **Low $\gamma$ (myopic, e.g., 0.7–0.9)**

* **Effect on returns**: $G_t$ mostly reflects rewards in the next few steps.
* **Impact on gradient updates**: Policy updates are driven by short-term outcomes.
* **Learning dynamics**:

  * Faster convergence because variance in $G_t$ is smaller (fewer far-off rewards).
  * But the learned policy is often *short-sighted* (e.g., in CartPole: it may learn to twitch reactively rather than balancing smoothly).
* **Bias/Variance trade-off**: Lower variance, higher bias (since long-term credit assignment is ignored).

---

## 2. **Moderate $\gamma$ (0.95–0.99, common default)**

* **Effect on returns**: $G_t$ incorporates a meaningful portion of the episode.
* **Impact on gradient updates**: Policy updates balance short-term corrective actions and long-term planning.
* **Learning dynamics**:

  * More stable policies (e.g., in CartPole: anticipatory movements, not just reactive corrections).
  * Slightly noisier gradients than with low $\gamma$, but usually the sweet spot in practice.

---

## 3. **High $\gamma$ (→ 1.0, farsighted)**

* **Effect on returns**: Almost the entire episode influences $G_t$.
* **Impact on gradient updates**: Credit assignment becomes very difficult — an action at time $t$ gets credit (or blame) for rewards that might happen hundreds of steps later.
* **Learning dynamics**:

  * **High variance in gradient estimates** → slower or unstable learning.
  * Exploration plays a bigger role: bad initial actions may spoil the whole return.
  * In CartPole, if you set $\gamma=1.0$, REINFORCE gradients will be very noisy because most episodes end with zero reward until the agent stumbles into balance.

---

## 4. **Connection to Variance Reduction (Baselines, Advantage Functions)**

* Policy gradient methods often use an **advantage estimate**:

  $$
  A_t = G_t - V(s_t)
  $$

  where $V(s_t)$ is a learned baseline.
* With larger $\gamma$:

  * $G_t$ has higher variance → baselines (or GAE in PPO/A2C) become *critical*.
  * **Generalized Advantage Estimation (GAE)** introduces an extra $\lambda$ parameter to trade off bias/variance, often mitigating issues caused by high $\gamma$.

---

## 5. **CartPole Example**

* **$\gamma=0.9$**: The agent learns quick reflexes but might fail to keep the pole up for 200 steps consistently.
* **$\gamma=0.99$**: The agent learns smooth cart movements to stabilize the pole for long durations.
* **$\gamma=1.0$**: In vanilla REINFORCE, training often diverges due to huge gradient variance. With PPO+GAE, it can still work, but training is slower.

---

### Key Insights

* $\gamma$ in policy gradient methods **controls the time horizon of credit assignment**.
* **Low $\gamma$:** lower variance, but myopic.
* **High $\gamma$:** farsighted, but very noisy gradients.
* In practice: $\gamma \approx 0.95–0.99$ is a good trade-off.
* Techniques like **baselines, GAE, and actor-critic architectures** are especially important at high $\gamma$.

---


# What about GRPO?

GRPO is a modern variant of policy gradient methods tailored for large language models. It mirrors PPO’s structure but **eliminates the value function** by using group-based advantage estimates:

* You sample a group of completions (outputs) from the current policy for the same prompt.
* The **advantage** for each completion is normalized relative to the group's mean and standard deviation of rewards, i.e.,

  $$
  A_i = \frac{r_i - \text{mean}(r)}{\text{std}(r)}
  $$

  ([DigitalOcean][1], [RLHF Book][2]).
* The policy update uses a clipped surrogate objective (like PPO) plus a KL divergence penalty to avoid policy drift ([DeepWiki][3], [DigitalOcean][1], [RLHF Book][2]).

Notably, GRPO replaces a learned baseline (value function) with a **statistical, group-based normalization**, bringing computational efficiency and stability ([Medium][4], [RLHF Book][2]).

---

### Does $\gamma$ Play a Role in GRPO?

No—**GRPO does not involve a discount factor**. Unlike conventional RL methods (REINFORCE, actor-critic, PPO), GRPO treats each completion's reward as a flat score—**no discounting is applied** over time or tokens.

* It computes advantages using raw, normalized rewards (e.g., completion-level rewards), with no reference to $\gamma$ ([DigitalOcean][1], [RLHF Book][2]).
* The algorithm is designed for tasks where the outcome (e.g., correctness of generated text) is evaluated globally, not stepwise—with no per-token discounted return.

---

### Why Doesn’t GRPO Use $\gamma$?

In environments like language model fine-tuning, each action (token generation) doesn't have an incremental reward. Instead:

* The model generates a full completion and is assessed as a whole (e.g., correctness, helpfulness).
* There's no sequential reward structure that needs discounting—hence no need for $\gamma$.

GRPO’s normalization over groups addresses variance and baseline without needing value estimation per token.

---

### Summary Table

| Algorithm Type  | Role of $\gamma$                   | GRPO’s Mechanism                  |
| --------------- | ---------------------------------- | --------------------------------- |
| REINFORCE / PPO | Discounts future rewards over time | N/A (no discounting used)         |
| GRPO            | —                                  | Uses group-normalized raw rewards |

---

### TL;DR

* **GRPO does not involve a discount factor $\gamma$**.
* It relies on **group-based normalization** of final rewards and a clipped PPO-like objective with KL regularization.
* As a result, **$\gamma$ has no influence** on GRPO’s behavior or updates.

---

The broader role of $\gamma$ remains vital in sequential RL algorithms—but for GRPO’s domain (LLM fine-tuning with final-output rewards), it simply doesn't come into play.

## Where $\gamma$ belongs when applying GRPO to sequential RL

An interesting research direction is to adapt **GRPO** (group-relative policy optimization) from LLM-style, trajectory-level rewards to **classic sequential RL** (e.g., CartPole, MuJoCo, Atari).

1. design choices (where $\gamma$ enters),
2. concrete algorithmic variants (with pseudocode),
3. how $\gamma$ changes learning dynamics and what to do about it (variance, credit assignment), and
4. practical hyperparameter recommendations.



In sequential RL you have per-step rewards $r_t$. The discount factor $\gamma$ defines the *discounted return* used for credit assignment:

$$
G_t = \sum_{k=t}^{T} \gamma^{k-t} r_k.
$$

GRPO originally uses **trajectory-level scalar rewards** (no discounting). You must decide how to map the sequential rewards into the scalar signals that GRPO normalizes over a group. Typical choices:

A. **Trajectory-level discounted return** (simple): compute a single $G=\sum_{t=0}^{T}\gamma^{t}r_t$ per trajectory and treat each trajectory as one sample in a group. Group-normalize these trajectory returns.

B. **Per-timestep discounted returns** (fine-grained): compute $G_t$ for each timestep and perform group-normalization of advantages at the timestep-level (either across the whole minibatch or within same time index / episode length bucket).

C. **Advantage + group-normalize**: compute advantage $A_t = \hat{G}_t - V_\phi(s_t)$ (or use GAE), then group-normalize the $A_t$ values across the sampled batch.

D. **Hybrid**: use trajectory-level normalization for episodic return signal and per-step baselines/advantage for policy updates.

Which to choose depends on task sparsity and horizon — below I explain tradeoffs.
---


# Algorithmic variants (pseudocode + explanations)

## Variant A — Trajectory-level GRPO (straightforward)

Use when you want the closest analogue to GRPO for episodic tasks.

1. Sample a group $G$ of $N$ trajectories $\{\tau_i\}$ under current policy.
2. For each trajectory compute discounted return

   $$
   R_i = \sum_{t=0}^{T_i} \gamma^t r_{i,t}.
   $$
3. Compute group mean $\mu$ and std $\sigma$ of $\{R_i\}$. Form normalized reward:

   $$
   \tilde{R}_i = \frac{R_i - \mu}{\sigma + \epsilon}.
   $$
4. For each trajectory, compute policy surrogate loss (PPO-like):

   $$
   L(\theta) = \mathbb{E}_{i,t}\left[\mathrm{clip}\Big(\frac{\pi_\theta(a_{i,t}|s_{i,t})}{\pi_{\text{old}}(a_{i,t}|s_{i,t})}, 1-\epsilon,1+\epsilon\Big)\cdot \tilde{R}_i \right] - \beta\,\mathrm{KL}(\pi_{\text{old}}\|\pi_\theta).
   $$

   Note: use the same normalized trajectory return $\tilde{R}_i$ as the scalar weight for all timesteps in that trajectory.
5. Update policy with standard PPO/GD.

**Comments:** simple, low memory (only one scalar per trajectory). But using the same scalar for all timesteps reduces temporal credit resolution — appropriate when trajectory-level reward is the natural signal (sparse episodic tasks).

---

## Variant B — Per-timestep GRPO with discounted returns

Better when fine-grained credit assignment matters.

1. Sample transitions (or full trajectories). For each transition compute $G_{i,t}$ (discounted return).
2. Compute group mean/std over the set $\{G_{i,t}\}$ (optionally normalize within time buckets to avoid mixing different time depths).
3. Use normalized $\tilde{G}_{i,t}$ as the weight in the surrogate loss:

   $$
   L(\theta)=\mathbb{E}_{i,t}\left[\mathrm{clip}\big(\rho_{i,t},1-\epsilon,1+\epsilon\big)\cdot \tilde{G}_{i,t}\right] - \beta\,\mathrm{KL}.
   $$

   where $\rho_{i,t}$ is the importance ratio.

**Comments:** higher variance (many $G_{i,t}$ samples), but better credit assignment. If you use this, pairing with a learned baseline (value) or GAE is recommended.

---

## Variant C — Advantage GRPO (recommended for policy-gradient stability)

Combine GAE/value baseline with group normalization.

1. Collect batch of trajectories. Fit/compute value $V_\phi$.
2. Compute advantages (GAE):

   $$
   \hat{A}_{i,t} = \text{GAE}_\lambda(s_{i,t}).
   $$
3. Compute group statistics over $\{\hat{A}_{i,t}\}$ and normalize:

   $$
   \tilde{A}_{i,t} = \frac{\hat{A}_{i,t}-\mu}{\sigma+\epsilon}.
   $$
4. Use $\tilde{A}_{i,t}$ in PPO-style surrogate loss.

**Comments:** retains per-step credit assignment while inheriting GRPO’s variance-stabilization via group normalization. This is the closest to modern actor-critic practice and typically the most robust.

---

## How does $\gamma$ affect dynamics in these variants?

General principles:

* **$\gamma$ controls effective horizon and credit assignment.**

  * Small $\gamma$ ⇒ short effective horizon ⇒ $G$ (or $G_t$) dominated by immediate rewards → lower variance, more bias toward myopic policies.
  * Large $\gamma$ (close to 1) ⇒ long effective horizon ⇒ returns depend on many future steps → higher variance in $G$ or $A$.

* **Group-normalization reduces scale/variance across the group but does *not* remove long-horizon credit assignment difficulty.**

  * Normalizing returns/advantages makes gradients numerically stable and comparable across trajectories, but if $\gamma$ is large and the reward is sparse, individual $G$ or $\hat{A}$ values will still have huge variability because of delayed outcomes — you still need larger sample sizes or temporal-credit solutions.

* **Specific effects per variant**

  * Variant A (trajectory-level): with large $\gamma$ you emphasize long-term survival; group-normalization helps training stability but you lose which actions in the episode caused success/failure. Long gamma + trajectory-level scalar → you may need longer trajectories and bigger group sizes to get reliable signal.
  * Variant B (per-timestep): large $\gamma$ dramatically increases variance of $G_{t}$; normalization helps, but you’ll see noisy policy updates unless you also use baselines or GAE.
  * Variant C (advantage+GAE): you mitigate variance via baseline and the $\lambda$ parameter. For large $\gamma$, set $\lambda$ < 1 to reduce variance (GAE is essentially another bias/variance knob complementary to $\gamma$).

* **Exploration / optimization**

  * Larger $\gamma$ makes early exploratory actions more consequential — may require stronger exploration (entropy, larger policy noise) and larger groups/batches so that the group normalization reliably captures performance differences.
  * Smaller $\gamma$ reduces the dependency on long strategic behavior, making exploration less costly but risking suboptimal long-run behavior.

---

## Thought Experiments

1. **Start with conventional RL defaults**: for many control tasks $\gamma \in [0.98,0.995]$ is common; for episodic tasks with short horizons $\gamma=0.99$ is a good starting point. For CartPole, $\gamma=0.99$ is typical.

2. **Prefer Variant C (Advantage + GAE + group-normalize)** as your first implementation. It gives the best bias/variance control.

3. **If rewards are sparse or horizon is long (high $\gamma$)**:

   * Increase batch / group size (GRPO benefits from larger groups to estimate group mean/std reliably).
   * Use GAE with $\lambda \in [0.9,0.97]$ (lower $\lambda$ to reduce variance if needed).
   * Consider normalizing advantages per-trajectory length buckets (to avoid mixing early- and late-timestep advantages).
   * Use stronger entropy regularization or explicit exploration schedules.

4. **If rewards are dense or you want fast convergence**:

   * You can reduce $\gamma$ slightly (0.95–0.99), which reduces variance and speeds learning — but check asymptotic performance.

5. **Group size & normalization**

   * Small groups (e.g., groups of 4–8 trajectories) work for LLMs but in sequential RL prefer groups = minibatch of many transitions or several full episodes (e.g., 32–256 trajectories/transitions) so the mean/std estimates are stable.
   * Add clipping / robust statistics (e.g., clip normalized values to a reasonable range) to avoid a few outliers dominating.

6. **Time-dependent policies / finite-horizon tasks**

   * If the environment is finite-horizon and optimal policy is time-dependent, include the time index in the policy input or maintain a time-conditioned policy.

---

## Practical pseudocode for the recommended approach (Variant C)

```text
for iteration=1..N:
    collect K trajectories (or M transitions) using π_old
    compute value estimates Vφ(s) (fit value network if needed)
    compute GAE advantages A_hat_{i,t} with gamma, lambda
    compute group mean μ and std σ over {A_hat_{i,t}} in batch
    normalized_A = (A_hat - μ) / (σ + eps)
    compute PPO clipped surrogate loss using normalized_A as advantage weights
    add KL penalty / entropy regularizer
    update policy θ (and value φ)
    optionally anneal gamma?  (usually do not)
```

Notes:

* **Do not change $\gamma$ on-policy within the same batch** — choose $\gamma$ at experiment start.
* You can run ablations where you sweep $\gamma$ ∈ {0.95, 0.98, 0.99, 0.995, 1.0} and also sweep group size and $\lambda$.

---

## Some open questions

* **Normalization bias**: group normalization transforms the objective: when you normalize advantages, you’re changing the relative scale and possibly introducing bias in gradient direction (it’s empirical but often helpful). Be mindful when comparing to baselines.
* **Non-i.i.d. returns across trajectories**: if trajectories are correlated (e.g., same start state), group statistics may underestimate variance — consider stratified grouping.
* **Gamma vs episode cap**: if environment has a hard episode cap $T$, $\gamma$ interacts with that cap; effective horizon ≈ min(T, 1/(1-γ)). When T is small relative to 1/(1-γ), raising gamma past that has diminishing returns.
* **Theoretic sample complexity**: increasing $\gamma$ generally worsens sample complexity due to larger variance; group-normalization reduces variance but does not change the fundamental dependence on horizon.

---

## Quick actionable checklist to implement & tune

1. Implement GRPO with **advantage normalization** (Variant C). Use GAE.
2. Choose initial $\gamma=0.99$ (CartPole/MuJoCo) or $\gamma=0.998$ (very long-horizon control) — pick based on environment horizon.
3. Group/minibatch size: start with 64–256 transitions; increase if normalized stats are noisy.
4. If training is unstable with high $\gamma$: reduce $\lambda$ in GAE, or reduce $\gamma$ slightly, increase group size, add value function regularization.
5. Report ablations: (γ, λ, group size, clipping ε, KL β). Track both learning speed and final performance.

---

### Final takeaway (one-paragraph)

You *must* retain $\gamma$ as the discounted-return horizon when moving GRPO into sequential RL. Use GRPO’s group-normalization as a **variance-stabilizing wrapper** around standard advantage estimation (GAE + baseline). For stability and good credit assignment, prefer per-timestep advantages (Variant C) normalized across the group — tune $\gamma$ to reflect the environment’s true horizon (typical defaults 0.98–0.995 for continuous control), and counteract the higher variance induced by large $\gamma$ with larger groups, GAE $\lambda<1$, and stronger baselines/regularizers.

---



[1]: https://www.digitalocean.com/community/conceptual-articles/group-relative-policy-optimization-reinforcement-learning?utm_source=chatgpt.com "GRPO in Reinforcement Learning Explained | DigitalOcean"
[2]: https://rlhfbook.com/c/11-policy-gradients.html?utm_source=chatgpt.com "Policy Gradient Algorithms | RLHF Book by Nathan Lambert"
[3]: https://deepwiki.com/lzhxmu/CPPO/2.1-grpo-algorithm?utm_source=chatgpt.com "GRPO Algorithm | lzhxmu/CPPO | DeepWiki"
[4]: https://medium.com/better-ml/group-relative-policy-optimization-grpo-the-deep-seek-cheat-code-5c13a2c86317?utm_source=chatgpt.com "Group Relative Policy Optimization (GRPO): deepseek’s RL cheat-code | by Jaideep Ray | Better ML | Jul, 2025 | Medium"


