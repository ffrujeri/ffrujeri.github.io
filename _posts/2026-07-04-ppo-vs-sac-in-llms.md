---
layout: archive
title: 'Why PPO Dominates RLHF (and SAC Falls Short)'
date: 2026-07-04
permalink: /posts/2026/07/ppo-vs-sac-in-llms
excerpt: "Exploring why Soft Actor-Critic struggles with LLM alignment and why Latent Text Diffusion might change the paradigm."
tags:
  - reinforcement-learning
  - large-language-models
  - generative-ai
---

Soft Actor-Critic (SAC) is arguably the king of continuous robotic control, known for its sample efficiency and robustness. However, when researchers align Large Language Models (LLMs) using Reinforcement Learning from Human Feedback (RLHF), they almost exclusively rely on Proximal Policy Optimization (PPO). 

SAC fundamentally clashes with the mechanics of LLMs across three major fronts: the action space, the exploration strategy, and memory scaling.

### 1. The Action Space: Continuous vs. Massive Discrete

SAC was built for continuous action spaces (e.g., assigning a torque value between -1.0 and 1.0). Its neural network outputs a continuous probability distribution to pick exact values.

Language generation, however, is fundamentally discrete. An LLM must choose exactly one token out of a vocabulary of 50,000 to 100,000 tokens. To apply a discrete version of SAC, you would have to calculate a Q-value ($Q(s, a)$) for every single possible action to find the maximum. Calculating 50,000 Q-values for every single word generated in a 1,000-word essay is computationally paralyzing. 

PPO, as a Policy Gradient method, sidesteps this by directly outputting probabilities and updating the network based only on the single token it actually sampled.

### 2. Exploration vs. Gibberish (Entropy)

SAC mathematically maximizes Entropy. It adds an entropy bonus to its objective function, forcing the agent to explore as many different, chaotic actions as possible while achieving the goal.

If you force an LLM to "explore wildly," it doesn't discover a creative new way to write an essay—it just destroys its own grammar and starts outputting gibberish. In RLHF, we want the model to stay as close to normal human language as possible. PPO enforces this using a KL-Divergence penalty, which punishes the LLM if it deviates too far from the original Base Model.

### 3. The Replay Buffer Memory Explosion

SAC is an off-policy algorithm, storing past experiences in a massive Replay Buffer. For a robot, a state is a small array of numbers. For an LLM, the "state" is the entire context window of text, and the neural network itself can be billions of parameters. Storing millions of generated text trajectories for off-policy Q-learning would require an astronomical amount of VRAM.

PPO is on-policy. It generates a batch of text, trains on it immediately, and throws it away, keeping memory overhead manageable for massive clusters of GPUs.

## A Continuous Future: Latent Text Diffusion

A cutting-edge frontier in AI research aims to bypass the massive discrete action space problem by shifting the RL target from the final discrete token to the internal continuous latent representation (the embeddings).

If the action space becomes continuous (a continuous vector representing the optimal next concept), algorithms like SAC theoretically become viable. However, doing this autoregressively crashes into the curse of dimensionality and the "Nearest Neighbor" problem of mapping continuous outputs back to discrete words.

Instead, researchers are applying this successfully in **Latent Text Diffusion**. Unlike sequential generation (like GPT), Latent Diffusion generates text like an image. It creates a block of continuous random noise and applies a neural network to denoise the entire sequence simultaneously. 

To solve the discrete decoding problem (where final continuous vectors don't perfectly align with vocabulary coordinates), they use an **Anchoring Loss**. This embedding regularization mathematically forces the final denoised vector to land *exactly* on the coordinates of a real word in the vocabulary matrix, making the snap back to discrete text seamless and grammatically stable.

## The Simplicity of GRPO

Most recently, advancements like GRPO (Group Relative Policy Optimization) have even dropped the traditional Value function entirely to save massive amounts of GPU memory. Instead of a complex Critic neural network, GRPO averages the scores of a few parallel generations to establish a baseline. This simple statistical trick effectively replaces the Critic network for LLMs, demonstrating how algorithmic simplicity and memory efficiency often win in modern LLM scale.
