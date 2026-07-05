---
layout: archive
title: 'Navigating Uncertainty with POMDPs and Belief States'
date: 2026-07-04
permalink: /posts/2026/07/pomdps-and-belief-states
excerpt: "How agents act rationally when the rules of the world are hidden, utilizing Partially Observable Markov Decision Processes."
tags:
  - reinforcement-learning
  - probability
---

When an environment is non-stationary or missing key pieces of information that dictate the rules, standard Reinforcement Learning algorithms will often endlessly chase their own tails.

Imagine an RL agent trading stocks. The market behaves differently during a bull market versus a bear market. If the agent doesn't know which market regime it's in, the environment appears non-stationary.

This is modeled using a Partially Observable Markov Decision Process (POMDP). Instead of learning a value for a physical state $V(s)$, the agent maintains a **Belief State** $b(s)$—a probability distribution over all the possible hidden contexts. As the agent observes changing rewards, it uses Bayesian inference to update its belief about which underlying regime it is currently operating in.

## Updating the Belief State

To update its belief state, a POMDP agent relies on one of the most powerful tools in statistics: Bayes' Theorem. Because the agent cannot see the true state of the environment, it maintains a probability distribution over all possible states. When it takes an action and receives a new observation, it uses the rules of probability to mathematically fold that new clue into its existing worldview.

The agent needs three pieces of information to perform this update:
1.  **The Current Belief $b(s)$**: The probability that the agent is currently in state $s$.
2.  **The Transition Model $T(s' | s, a)$**: The physics of the world. If I am in state $s$ and take action $a$, what is the probability I end up in state $s'$?
3.  **The Observation Model $O(o | s', a)$**: The sensor reliability. If I take action $a$ and land in state $s'$, what is the probability I will see observation $o$?

When the agent takes an action $a$ and receives an observation $o$, the new belief for any state $s'$, denoted as $b'(s')$, is calculated as:

$$b'(s') = \eta \cdot O(o | s', a) \sum_{s \in S} T(s' | s, a) b(s)$$

This is a logical process of Predicting and Correcting:
*   **Predict (Summation)**: "Based on where I thought I was, and the action I just took, where am I likely to be now?"
*   **Correct (Observation Model)**: "I just observed $o$. If I were truly in state $s'$, how likely is it that my sensors would show me $o$?"
*   **Normalize ($\eta$)**: A constant to make sure your new belief state forms a valid probability distribution that sums to 1.

## The Observation Model in Action

Imagine a robotic vacuum cleaner navigating a simple grid world with a cheap infrared sensor. Sometimes dark baseboards absorb light (false negative) and sunlight reflects (false positive). 

If the robot was 95% confident the path ahead was clear, but the sensor outputs a BEEP (obstacle). 

*   Testing "Wall" hypothesis: 0.05 (prior) * 0.90 (true positive rate) = 0.045
*   Testing "Clear" hypothesis: 0.95 (prior) * 0.05 (false positive rate) = 0.0475

Even though the sensor beeped, the robot's prior belief was so strong that the math tells it the beep was slightly more likely to be a glitch! After normalizing, it's roughly 51% confident the path is clear, and 49% confident there is a wall. It will likely take another action to scan again before making a firm decision.

## Do We Use the Value Function in POMDPs?

Yes, but with a massive twist: because the agent does not know what physical state it is in, it uses a **Belief Value Function** $V(b)$ that asks, "How good is it to be this confident about which room I am in?"

The exact POMDP Bellman equation is:

$$V(b) = \max_a \left( R(b, a) + \gamma \sum_{o \in O} P(o | b, a) V(b') \right)$$

Instead of a single smooth curve or lookup table, $V(b)$ is constructed out of a set of intersecting straight lines (or hyperplanes) called **Alpha Vectors**, mapping an infinite number of probabilities to values. 

In modern Deep RL, computing exact Alpha Vectors is bypassed entirely. We use a Recurrent Neural Network (RNN) or Transformer to compress the history of actions and observations into a hidden vector $h$, which acts as our approximate belief state. The neural network smoothly estimates the true piecewise linear surface without having to explicitly calculate Bayes' Theorem.
