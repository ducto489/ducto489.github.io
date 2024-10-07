---
layout: distill
title: RL play Worlde
description: Using RL to train a bot play wordle
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
  - name: Challenges and My Propose Solutions
    subsections:
      - name: 1. Challenges
      - name: 2. My Propose Solutions
  - name: Learn More
---

## Overview

I recently done a project where I used reinforcement learning (RL) to solve Wordle, a popular word puzzle game. By leveraging the power of machine learning, I trained an RL agent to learn how to guess words based on feedback. But this project is not complete yet, there are spaces for improvement and my model can't run on the full dataset yet.

## Project Structure

- **Source Environment**: [wordle-solver](https://github.com/andrewkho/wordle-solver). I customized their environment and used their hyperparameters for training.
- **Vectorized Environment**: To make training more efficient, I used a vectorized environment with DummyVecEnv, allowing the agent to train across multiple Wordle instances simultaneously.
- **First step**: In my setting, I choose the word 'crate' to optimize the information which the model can guess after fist step. It is indeed make the training sped up because the several first mean reward before I use this setting is negative but after using, it is positive mean reward

## Training
Training was conducted using A2C. I trained the agent using the first 10 words from the official Wordle vocabulary first and using 10 action to represent the word the model can choose, gradually teaching the model to make better word guesses. After the mean reward is great, I transfer learning to train on first 100 words and 100 action. And after that I train on the full dataset which include 2315 words and 2315 words

## Challenges and My Propose Solutions
### 1. Challenges
I face with a problem is that the training is too long. When I train on the first 10 words. The learning is fast. But when I transfer learning, the model is learning too slow. I spend 7 hours on training 100 words model but it get 25% right mean answer. But it is better than random guess which is 6%. 

### 2. My Propose Solutions
- In my current setting, the model see the action is a black box. So it guess randomly and when the number of action increase (i.e number of words it can choose), the learning is getting longer and longer. So if somehow I can tell the model the structure of the action it can choose, the model don't need to guess randomly to learn about that action and the model will learn to read that information to guess smartly.
- I have another idea. I will train for first 10 words. Then I increase the action space to 20 and only train with sample the target word from index 11 to 20. Then I will train the first 20 words so that the model will learn to combine the old and the new learning. I will repeat this process untill the end of the dataset.

---

## Learn More
For a detailed exploration of the code, and methods, you can view this [Kaggle Notebook](https://www.kaggle.com/dustnn/wordle-training).

