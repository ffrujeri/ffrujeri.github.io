---
title: 'Alpha Geometry: Solving olympiad geometry without human demonstrations.'
date: 2024-02-04
permalink: /posts/2024/02/alpha-geometry
tags:
  - ai
  - rl
  - llm
  - neuro-symbolic-ai
  - geometry
  - olympiad-level-problems
  - theorem-proving
---

[Blog](https://deepmind.google/discover/blog/alphageometry-an-olympiad-level-ai-system-for-geometry/)
[Paper](https://www.nature.com/articles/s41586-023-06747-5)

## Outline:
- Problem Statement
- Core Contribution: Synthetic Data Generation
- Sampling Premises
- Symbolic Deduction
- Traceback
- Auxiliary Construction
- Experimental Results
- Problem Representation
- Final Comments

## Key takeaways:

- AlphaGeometry is a system that can solve geometry problems without any human demonstrations. It does this by using a combination of a symbolic solver and a large language model.

- The symbolic solver can reason about the things that already exist in the problem, but it cannot introduce new things. The language model is used to suggest new things to construct, such as points, lines, or circles.

- The system works by first trying to solve the problem with the things that are already there. If it can't solve the problem, it asks the language model for a suggestion. It then adds the suggestion to the problem and tries to solve it again. This process repeats until the problem is solved.

- AlphaGeometry is a very specialized system that only works for a certain type of geometry problem. However, it is very good at solving these problems, and it is often better than humans.

## Examples:

- AlphaGeometry can only solve problems that can be expressed in its limited - language. Additionally, AlphaGeometry is not always able to find the shortest or most elegant proof.

## Neural Symbolic Solvers (NSSs)

- Neural Symbolic Solvers (NSSs) combine the strengths of *neural networks (flexible learning, pattern recognition)* with *symbolic reasoning (logical inference, interpretability)*. They aim to address complex tasks that require both robust feature learning and structured reasoning.

### Key Components:

- Neural Network Component: Typically a deep neural network (DNN) trained on large datasets. Extracts relevant features and patterns from input data.
Represents knowledge in a continuous, distributed manner.

- Symbolic Reasoning Component:
  - Employs symbolic representations (logic, rules, graphs).
  - Performs logical inference, reasoning, and knowledge manipulation.
  - Enables interpretability and explainability.

- Integration Approaches:
  - Loose Coupling: Neural network and symbolic reasoner interact as separate modules, exchanging information through shared representations or intermediate results.
  - Tight Integration: Neural network components are embedded within the symbolic reasoning process, directly influencing reasoning steps.