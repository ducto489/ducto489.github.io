---
layout: distill
title: Reinforcement Learning for Wordle
description: Training an A2C agent to solve small Wordle environments
img: assets/img/wordle.png
importance: 2
category: Machine Learning
disqus_comments: false
date: 2024-09-15
featured: true

toc:
  - name: Overview
  - name: Environment and Setup
  - name: Training Strategy
  - name: Challenges
  - name: Takeaways
---

## Overview

The experiment is available in this [Kaggle notebook](https://www.kaggle.com/dustnn/wordle-training).

I used reinforcement learning to explore Wordle as a sequential decision-making problem. The agent receives the standard Wordle-style feedback after each guess and learns a policy over a restricted vocabulary. The project is intentionally experimental: the smaller environments train, but the current approach does not yet scale well to the complete Wordle vocabulary.

## Environment and Setup

- **Base environment:** adapted from [andrewkho/wordle-solver](https://github.com/andrewkho/wordle-solver).
- **Algorithm:** Advantage Actor-Critic (A2C).
- **Vectorized environment:** DummyVecEnv was used to run multiple environment instances through the same training interface.
- **Fixed opening guess:** the first guess is set to `crate` so the learning problem starts from a consistent information-rich state.

Fixing the first move reduces the number of decisions the agent must learn and made the early reward signal more stable in my experiments.

## Training Strategy

I first trained on a toy environment containing 10 candidate words and 10 possible actions. After the agent learned that setting, I expanded the vocabulary to 100 words and transferred the model parameters.

The longer-term target is the full 2,315-word answer vocabulary. The action space grows with the vocabulary, so the simple discrete-action formulation becomes increasingly expensive and sample-inefficient.

## Challenges

The main issue is scaling. Training is fast on the 10-word environment, but learning slows substantially as the vocabulary grows. In one 100-word run, approximately seven hours of training produced a success rate around **25%**. That is better than the simple random baseline used in the experiment, but still far from a practical Wordle solver.

The experiment also highlights an important modeling issue: treating every word as an unrelated discrete action discards spelling and letter-level structure. A more scalable agent should exploit that structure rather than learning an independent action value for every word.

## Takeaways

- RL can learn useful policies in small Wordle environments, but sample efficiency becomes a major constraint as the action space grows.
- A fixed opening move simplifies the task, although it also removes one decision from the policy.
- For the full game, structured search, entropy-based heuristics, supervised pretraining, or an action representation based on word features may be more efficient than plain discrete-action RL.
- This project is most useful as an RL scaling experiment rather than as a competitive Wordle solver.
