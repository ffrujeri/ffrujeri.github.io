---
layout: archive
title: 'Evaluation of Foundation Models'
date: 2024-04-07
permalink: /posts/2024/04/fm-evaluation
excerpt: "What do we need to properly evaluate a foundation model?"
tags:
  - ai
  - evaluation
---

## Evaluation is about answering the question: How good is our model?
- Assign higher probability to *real* or *frequently observed* sequences than *rarely observed* (or in LM *ungrammatical* sentences).
  - A **test set** is used (different from the **training set**) totally unseen / unused.
  - An **evaluation metric** tells us how well our model does on the test set

## N-gram Probabilities and Markov assumption
- Recall of the definition of conditional probabilities and chain rule
$$P(A,B) = P(A|B) P(B)$$
$$P(x_1,x_2,x_3,...,x_n) = P(x_1)P(x_2|x_1)P(x_3|x_1,x_2)...P(x_n|x_1,...,x_{n-1})$$

- We do everything in log probs:
  - Avoid underflow
  - Also adding is faster than multiplying 

$$(p_1 * p_2 * p_3 * p_4) <> \log p_1 + \log p_2 + \log p_3 + \log p_4$$

## Extrinsic evaluation of N-gram models
- Best evaluation for comparing models A and B is put test in a downstream task
  - E.g. for LM spelling corrector, speech recognizer, Machine Translation systetm
  - Generate completions for the downstream task and get performance metrics
    - E.g. How many misspelled words were corrected properly?
    - How many words were translated correctly?
- Downsides
  - time-consuming, can take days or weeks
  - Bias introduced by the design of the downstream task

## Intrinsic evaluation
- **Perplexity**
  - Caviat: it is a bad approximation unless the data looks just like the training data, but it quite useful for pilot experiments

- The Shannon Game: How well can we predict the next word?
  - From this perspective, a better model is the one that assigns a higher probability to the element that actually occurred in the sequence
  - The best LM for example is the one that best predicts an unseen test set, meaning gives the highest P(sentence)
  - Perplexity is the probability of the test set, normalized by the number of words

$$PP(W) = P(w_1w_2...w_n)^{1/N}$$

- From Josh Goodman
  - Perplexity of a task is weighted equivalent branching factor
    - How hard is the task of recognizing digits?: 0,1,2,3,4,5,6,7,8,9.
      - Perplexity 10. $PP(W) = P(w_1w_2...w_N)^{1/N} = ((1/10)^N)^{-1/N} = (1/10)^{-1} = 10$

    - How hard is it to do speech recognition for 30,000 different speakers
      - Perplexity = 30000

    - What if the system needs to recognize Operator (1 in 4), Sales (1 in 4), Tech support (1 in 4), 30,000 names (1 in 120,000 each)
      - Perplexity is 54

- The lower the perplexity the better the model
E.g.: Training 38 million words, test 1.5 million words, WSJ

| N-gram order | Unigram |Bigram | Trigram |
|--------------|---------|-------|---------|
| Perplexity   |    963  |   170 |   109   |
  

## References
- [Evaluation and Perplexity](https://www.youtube.com/watch?v=BAN3NB_SNHY&t=55s)
- [SRILM](https://www.speech.sri.com/projects/srilm/)
- [Perplexity](https://huggingface.co/docs/transformers/en/perplexity)
- [Evaluation Metrics for Language Modeling](https://thegradient.pub/understanding-evaluation-metrics-for-language-models/)