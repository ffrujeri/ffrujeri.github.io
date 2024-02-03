---
title: 'Value-Based Methods in RL'
date: 2023-12-17
permalink: /posts/2023/12/rl-value-based
tags:
  - ai
  - rl
  - value-based
---


As we discussed in previous posts, the ability to properly formulate a Markov Decision Process (MDP) is imperative for successful application of Reinforcement Learning (RL) methods.

MDPs serve as a framework to formalize sequential decision-making problems, in which decisions over time are decomposed into a series of subproblems. 

In canonical form, an MDP is described by a tuple $(S,A,R,P,\gamma)$.

## The markov property

By definition, any MDP fulfills the **Markov property**, aka the memoryless property. This property states that decisions do not depend on states of the past, but only on the present state. If the problem can be formulated in such a way, we can break down extremely complicated decision problems into a sequence of more manageable subproblems, to be solved independently.

It is natural to interpret the Markov property as ‘not utilizing any information from the past’. However, there is a distinction between not using states from the past and information from the past.

In mathematical terms, states including historical information are utilized in higher-order MDPs. Such models provide a richer representation of the system than first-order MDPs allow (which indeed only include present information). Taken to the extreme, a state could even include all historical information.

## State space (S)
A state $s \in S$ contains all information needed to compute decisions, rewards, and transitions; the state space S is the set containing all states. 

A core notion of MDPs is that decisions are made solely based on the current state of the system (the **memoryless** property); past states should not factor into decision-making. A common misunderstanding is that a state cannot incorporate past information; this is in fact allowed (higher-order Markov models, see Salnikov et al.).

In some cases a state is a simple scalar, such as an agent’s position on a one-dimensional line. 
Typically, the state would be a vector. Consider a robot arm, with the state described by the angles of its three joints $s=[x,y,z]$

A stock portfolio state may be described with the prevailing price level and amount held per stock $s=[p_1,p_2,…,p_n,s_1,s_2,…,s_n]$

As a guideline, the state should include exactly all relevant data used to make informed decisions — no less, no more. If certain data does not impact decision-making, rewards or transitions, it should not be incorporated in the state.

- Wikipedia (n.d.) — “A state variable is one of the set of variables that are used to describe the mathematical ‘state’ of a dynamical system”.

- Bellman (1957) — “… we have a physical system characterized at any stage by a small set of parameters, the state variables. ”

- Puterman (2014) — “At each decision epoch, the system occupies a state.”

- Sutton & Barto (2018) — “…signal to represent the basis on which the choices are made (the states)”

- Bertsekas (2018) — “…is the state of the system, an element of some space. […] Many classical problems in control theory involve a state that belongs to a Euclidean space, i.e., the space of n-dimensional vectors of real variables, where n is some positive integer.”

Powell distinguishes three state components: 

(i) *physical*: directly observable properties of the system, e.g., resources. Think the location of a truck, money on your bank account, or battery level of a drone.

(ii) *information*: non-tangible deterministic information. Can be directly observed, but is not necessarily a physical component of the system. 

Aside from properties directly derived from the system, we often use additional information in decision-making. If we have a stock portfolio, the stock prices (market data) would be information.

Information can also pertain to the past or future. Past stock prices might reveal relevant trends that cannot be gleaned solely from the present price

(iii) *belief* / *knowledge*: non-tangible probabilistic knowledge. Concretely, the belief may be represented by the parameters of a distribution.

We might not make decisions based solely on known information, but also on uncertain information or beliefs.



## Action space (A)
Integral to MDPs is the ability to exercise some degree of control over the system. The action $a \in A$ — also decision or control in some domains — describes this influence by the agent; the action space $A$ contains all (feasible) actions. 

As for the state, the action can be a simple scalar ("exercise option" $a \in \{0,1\}$), but also a high-dimensional vector ("order $a_i$ products of type $i$").

With **state-independent** actions, the same action space $A$ applies for every state. For many problems, actions are state-dependent (action space $A(s)$): e.g. the moves on a chess board depend on the current pieces and their positions. Sometimes it is convenient to presume the action space is state-independent (e.g., in Deep Q-learning) and simply mask infeasible actions.


## Reward function (R)
MDP models entail optimizing some objective function, typically a (discounted) cumulative reward. 

The reward function $R(s,a)$ captures the direct rewards, costs or contributions associated with individual decision moments, being dependent on state s and/or action a. 

Sometimes the reward is evident, e.g., the monetary gains and losses resulting from stock trading. In other cases, more artificial constructs are needed, for instance the reward for winning a game or reaching a target. It is then necessary to align rewards with the objectives of solving the problem.
Non-zero rewards are not necessarily incurred at each epoch, e.g., they might only be received at the end of an episode (winning/losing a game). 

In RL, we often resort to reward shaping, introducing intermediate rewards to recognize ‘good’ actions and guide the algorithm. Reward shaping should be used with care, as it alters the original problem setting and may introduce unexpected behavior.


## Transition function (P)
In an MDP, we take discrete time steps in which we move from one state $(s)$ to the next $(s')$. This transition function depends on the present state $s$, the action $a$ and random information $ω$. 

The transition can be broken down into a deterministic and a stochastic part, in which a describes the deterministic component and $ω$ the stochastic component (although often a mixture, the transition might be fully deterministic or fully stochastic).

In MDP formulations, the transition function is often hidden in the deceptively simple probability $P(s'|s,a)$

Deceptive indeed, as it is often the most difficult component to model. Truthfully, the transition function may range from a trivial deterministic function (e.g., a step in GridWorld) to a highly complex Digital Twin representation of reality, with many uncertainties to model.

Although the transition function is a necessary component of the MDP, in RL we do not necessarily define it explicitly. Instead, we may observe transitions from the environment. If we learn to play a game of Super Mario, we likely don’t know the actual transition functions; we simply try moves and observe what happens. 

We may even utilize real-world observations (e.g., when controlling wind farm that depends on weather circumstances). In this case, the world itself is our transition function!

## Discount factor ($\gamma$)
The discount factor $γ \in [0,1]$ is optional for finite horizon MDPs and therefore not always included in the MDP tuple. It indicates to what extent future rewards are factored into current decision-making, with $\gamma=0$ completely dismissing future rewards and $\gamma=1$ weighing all future rewards equally. 

There are various rationales for working with discount rates, the chief one is to ensure converging value functions for infinite horizon problems.

Any setting $\gamma<1$ suffices to guarantee (theoretical) convergence, but the exact setting is important for optimization purposes. A general rule-of-thumb is that the more uncertain the system is, the higher the discount rate is set, reflecting the limited impact that actions today have on performance in the future.

## References:
- [Five Building Blocks of MDPs](https://towardsdatascience.com/the-five-building-blocks-of-markov-decision-processes-997dc1ab48a7), Wouter van Heeswijk

- [A Deep dive into problem states](https://towardsdatascience.com/a-deep-dive-into-problem-states-498ad0746c98), Wouter van Heeswijk

- Powell, W. B. (2019). A unified framework for stochastic optimization. European Journal of Operational Research, 275(3), 795–821.

- Salnikov, V., Schaub, M. & Lambiotte, R. Using higher-order Markov models to reveal flow-based communities in networks. Sci Rep 6, 23194 (2016). https://doi.org/10.1038/srep23194

- Bellman, R. (1957). A Markovian decision process. Journal of mathematics and mechanics, 679–684.

- Bertsekas, D. (2019). Reinforcement learning and optimal control. Athena Scientific.

- Powell, W. B. (2014). Clearing the jungle of stochastic optimization. In Bridging data and decisions (pp. 109–137). Informs.

- Powell, W. B., & Meisel, S. (2015). Tutorial on stochastic optimization in energy — Part II: An energy storage illustration. IEEE Transactions on Power Systems, 31(2), 1468–1475.

- Powell, W. B. (2022). Reinforcement Learning and Stochastic Optimization: A unified framework for sequential decisions. 

- John Wiley & Sons.Powell, W. B. (n.d.). Modeling. https://castlelab.princeton.edu/modeling/

- Puterman, M. L. (2014). Markov decision processes: discrete stochastic dynamic programming. John Wiley & Sons.

- Sutton, R. S., & Barto, A. G. (2018). Reinforcement learning: An introduction. MIT Press.

- Wikipedia (n.d.) State variable. https://en.wikipedia.org/wiki/State_variable

- An Interview with George B. Dantzig: The Father of Linear Programming — The College Mathematical Journal (1986). http://www.phpsimplex.com/en/Dantzig_interview.htm

- W.B. Powell (2011). Approximate Dynamic Programming, John Wiley and Sons, 2nd edition.
