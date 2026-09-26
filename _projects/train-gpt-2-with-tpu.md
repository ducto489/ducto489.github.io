---
layout: distill
title: Train GPT-2 with TPU
description: Training a GPT-2-style 124M language model efficiently on Kaggle TPU
img: assets/img/trainTPU.jpg
importance: 1
category: Machine Learning
disqus_comments: false
date: 2024-08-29
featured: true

toc:
  - name: Overview
  - name: Key Challenges and Solutions
    subsections:
      - name: 1. Disk Space
      - name: 2. Slow Training on T4 GPUs
  - name: Techniques Used
  - name: Results
  - name: Limitations
---

## Overview

The code and experiment are available in this [Kaggle notebook](https://www.kaggle.com/code/dustnn/train-gpt-2-with-tpu).

This project implements a GPT-2-style 124M parameter language model while following architecture and training ideas from the GPT-2 and GPT-3 literature. I used FineWeb-Edu as a modern training corpus. The original GPT-2 model was trained on WebText, so this experiment should be understood as a reproduction of the model scale and training approach rather than an exact reproduction of OpenAI's original data pipeline.

The main goal was practical: determine whether a free Kaggle TPU could make a small GPT-style pretraining run feasible when the available T4 GPUs were too slow.

## Key Challenges and Solutions

### 1. Disk Space

Kaggle provides limited local disk space. Pre-tokenizing and storing a large corpus locally can exhaust that space before training starts.

**Solution:** stream examples from the dataset pipeline instead of materializing the complete tokenized corpus on disk.

### 2. Slow Training on T4 GPUs

The available T4 GPUs do not provide the same BF16 path used by newer accelerators. In my setup, FP32 training reached roughly **7,400 tokens/s**, making the full run impractical.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/BUGtrainGPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

After 12 hours, the GPU run had reached only 295 of 19,073 planned steps.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/trainGPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

**Solution:** move the run to TPU, where BF16 and multiple TPU cores provided much higher throughput.

## Techniques Used

- **Gradient accumulation** to reach a larger effective batch size.
- **BF16 mixed precision** on TPU.
- **Distributed training** across TPU cores.
- **Efficient attention implementations** where supported by the runtime.
- **Streaming data loading** to reduce local storage pressure.
- **Hardware-friendly tensor and batch dimensions**, favoring values that map efficiently to accelerator kernels.

## Results

The TPU run reached roughly **243,000 tokens/s**, about **33×** the throughput of the FP32 T4 setup measured in this experiment.

The final checkpoint reached a validation loss of **3.2754** on my validation setup and a HellaSwag accuracy of **0.2962**. I compared the latter with a GPT-2 124M reference value of **0.294463** used in the experiment.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/trainTPU.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

These numbers show that the TPU setup made the experiment computationally practical. They should not be interpreted as a strict claim that this checkpoint is universally better than the original GPT-2 124M model, because the training corpus, validation data, and evaluation setup are not identical to the original GPT-2 experiment.

## Limitations

- The training corpus differs from GPT-2's original WebText corpus.
- Validation loss is only directly comparable when tokenization and evaluation data are matched.
- The throughput comparison is specific to the Kaggle hardware and software configuration used in this project.
- A stronger reproduction would report multiple downstream benchmarks and fully document the tokenizer, data mixture, optimizer schedule, and random seeds.
