---
layout: distill
title: Train GPT-2 with TPU
description: Using TPU to speedup and recreate GPT-2 result
img: assets/img/trainTPU.png
importance: 1
category: ML
disqus_comments: true

toc:
  - name: Challenges
    subsections:
      - name: Disk Memory
      - name: Training Extremely Slow
  - name: Result
---

This project was inspired by [Andrej Karparthy's video](https://youtu.be/l8pRSuU81PU?si=6_loh9yI5Fj6ut1g). 
But if you don't have powerful GPUs or have money for GPU rental. We still can achieve GPT-2 124M performance with TPU on Kaggle! But we have some problems to solve.
If you want to dive deep into my code, please have a look at this [notebook](https://www.kaggle.com/code/dustnn/train-gpt-2-with-tpu).

## Challenges
### Disk Memory
In Kaggle we only have 40GB of disk memory. If we use the saving and loading data technique in Andrej Karpathy's videos,  we end up running out of disk space before the training stuff begins

**Solution:** Use streaming techniques in the datasets library and two separate datasets for training and valuation. 

### Training Extremely Slow

GPU T4 doesn't support BF16. If we use float16 the loss will increase. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets\img\BUGtrainGPU.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Using float32 is a pain in the neck when it slows down the training process very much. We achieved 7.400 tokens/sec. For comparison, Andrej Karparthy achieved 1.242.000 tokens/sec. After 12 hours of training, we reach the 295/19073 step. Not even close! 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets\img\trainGPU.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

**Solution:** TPU supports BF16 and it is even faster!

---

## Result
After applying TPU and BF16 and running for 18 hours I finally **surpassed GPT-2 124M model**! We achieved 243.000 tokens/sec meaning that we sped up the training by 243.000 / 7.400 = **33 times**!

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets\img\trainTPU.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>