---
layout: distill
title: Reinforcement Learning play Worlde
description: Using reinforcement learning to train a bot play wordle
img: assets/img/wordle.png
importance: 2
category: Machine Learning
disqus_comments: true
date: 2024-09-15
featured: true

toc:
  - name: Overview
  - name: Project Structure
  - name: Training
  - name: Challenges
  - name: Reflections on Reinforcement Learning from this Project
---

## Overview

For a detailed exploration of the code, and methods, you can view this [Kaggle Notebook](https://www.kaggle.com/dustnn/wordle-training).

I recently done a project where I used reinforcement learning (RL) to solve Wordle, a popular word puzzle game. By leveraging the power of machine learning, I trained an RL agent to learn how to guess words based on feedback. But this project is not complete yet, there are spaces for improvement and my model can't run on the full dataset yet.

## Project Structure

- **Source Environment**: [wordle-solver](https://github.com/andrewkho/wordle-solver). I customized their environment and used their hyperparameters for training.
- **Vectorized Environment**: To make training more efficient, I used a vectorized environment with DummyVecEnv, allowing the agent to train across multiple Wordle instances simultaneously.
- **First step**: In my setting, I choose the word 'crate' to optimize the information which the model can guess after fist step. It is indeed make the training sped up because the several first mean reward before I use this setting is negative but after using, it is positive mean reward

## Training
Training was conducted using A2C. I trained the agent using the first 10 words from the official Wordle vocabulary first and using 10 action to represent the word the model can choose, gradually teaching the model to make better word guesses. After the mean reward is great, I transfer learning to train on first 100 words and 100 action. And after that I train on the full dataset which include 2315 words and 2315 words

## Challenges
I face with a problem is that the training is too long. When I train on the first 10 words. The learning is fast. But when I transfer learning, the model is learning too slow. I spend 7 hours on training 100 words model but it get 25% right mean answer. But it is better than random guess which is 6%. 

## Reflections on Reinforcement Learning from this Project

My experience with this Wordle solver project has reinforced several key insights about Reinforcement Learning:

-   **Power and Promise**: RL is undeniably a very strong and promising approach, especially for problems where the optimal strategy is not obvious or easy to codify, and learning from interaction is key.
-   **Training Cost**: The major hurdle with RL is often the significant computational cost and time required for training, particularly as the state and action spaces grow. This project clearly demonstrates that challenge.
-   **When to Consider RL**: RL truly shines when other, more direct algorithms are not readily applicable or effective. If a problem lacks a clear, derivable heuristic or if an exhaustive search is infeasible, RL offers a path to discover solutions. However, its demanding nature means it should be carefully considered against potentially simpler alternatives if they exist.

---

