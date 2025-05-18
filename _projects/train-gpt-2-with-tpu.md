---
layout: distill
title: Train GPT-2 with TPU
description: Using TPU to speedup and recreate GPT-2 result
img: assets/img/trainTPU.jpg
importance: 1
category: Machine Learning
disqus_comments: true
date: 2024-08-29
featured: true

toc:
  - name: Overview
  - name: Key Challenges and Solutions
    subsections:
      - name: 1. Disk Memory Limitation
      - name: 2. Slow Training on GPU T4 x2
  - name: Techniques Used
  - name: Result
---

## Overview

For a detailed exploration of the code, datasets, and methods, you can view this [Kaggle Notebook](https://www.kaggle.com/code/dustnn/train-gpt-2-with-tpu).

This project was inspired by [Andrej Karparthy's video](https://youtu.be/l8pRSuU81PU?si=6_loh9yI5Fj6ut1g). Here I stick to the GPT-2 and GPT-3 paper to reproduce the model and techniques. I applied gradient accumulation, distributed data parallel (GPU and TPU), half-precision, flash attention, and nice numbers (the number that can be divided by 2 the most). I trained on FineWeb (EDU), which is the same dataset that GPT -2 has trained on. For evaluation, I used a different dataset and a HellaSwag for comparison to the GPT-2 paper.

But if you don't have powerful GPUs or have money for GPU rental. We still can achieve GPT-2 124M performance with TPU on Kaggle! But we have some problems to solve.

## Key Challenges and Solutions
### 1. Disk Memory Limitation
In Kaggle we only have 40GB of disk memory. If we use the saving and loading data technique in Andrej Karpathy's videos,  we end up running out of disk space before the training stuff begins.

**Solution:** Implemented streaming techniques from the datasets library, allowing for efficient data handling during training and evaluation.

### 2. Slow Training on GPU T4 x2

GPU T4 doesn't support BF16. If we use float16 the loss will increase. 

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BUGtrainGPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Using float32 causes pain in the neck when it slows down the training process very much. We achieved 7.400 tokens/sec. For comparison, Andrej Karparthy achieved 1.242.000 tokens/sec. After 12 hours of training, we reach the 295/19073 step. Not even close! 

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/trainGPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

**Solution:** TPU supports BF16 and it is even faster!

## Techniques Used
- **Gradient Accumulation**: To simulate larger batch sizes without requiring extensive memory.
- **Half-Precision (BF16)**: Leveraging TPU's support for BF16 to improve training speed and reduce memory usage.
- **Distributed Data Parallel**: To accelerate training by distributing the workload across multiple TPU cores.
- **Flash Attention**: Optimized attention mechanism for faster computation.
- **"Nice Numbers"**: Applied values divisible by 2 as much as possible to optimize memory usage.
- And some other TPU optimization.

## Result
After applying TPU, BF16, and some other TPU optimization and running for 18 hours I finally **surpassed GPT-2 124M model** with validation loss 3.2754 over 3.2924 and HellaSwag evaluation 0.2962 over 0.294463! We achieved 243.000 tokens/sec meaning that we sped up the training by 243.000 / 7.400 = **33 times** compared to GPU T4 x2!

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/trainTPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Here are some fun text examples that generated from my model:
- Hello, I'm a language model, and I do have a great understanding of everything about AI (including its applications). Although I don't think it's a
- Hello, I'm a language model, and here's all I want to put a bit more energy into figuring out what words in English could mean in English.
- Hello, I'm a language model, and I can help you find language translations where you can even use "words" in dictionaries or translations. I'm
- Hello, I'm a language model, I'm a linguist for Learning English. I'm trying to teach people language and English through a series of lectures in
---