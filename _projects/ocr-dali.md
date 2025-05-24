---
layout: distill
title: DALI in OCR
description: Using DALI to speedup data preprocessing and augmentation in OCR
img: assets/img/dali-ocr.jpeg
importance: 1
category: Machine Learning
disqus_comments: true
date: 2024-05-14
featured: true

toc:
  - name: 1. Introduction
  - name: 2. Data Processing and Details
    subsections:
      - name: Character Normalization
      - name: Datasets Used
  - name: 3. Training Strategy
  - name: 4. Why NVIDIA DALI?
  - name: 5. When and How to Integrate DALI for Faster OCR Training
  - name: 6. Practical Integration and Experimental Results
    subsections:
      - name: Case 1 High-End GPU with Strong CPU (NVIDIA A6000)
      - name: Case 2 Cloud GPU (NVIDIA L4)
      - name: Case 3 Mid-Range GPU with Slow Storage (NVIDIA 3060 + HDD)
  - name: 7. Analysis and Conclusion
---

# Accelerating OCR Training with NVIDIA DALI: A Practical Guide and Case Study
## 1. Introduction

For a detailed exploration of the code, and methods, you can view this [Github Repo](https://github.com/ducto489/lib_ocr).

You can see how we perform inference with our pretrained model in [notebook](https://github.com/ducto489/lib_ocr/blob/training/inference/inference.ipynb).

Training Deep Learning models for Optical Character Recognition (OCR) often involves complex data loading and augmentation pipelines. These preprocessing steps, if not optimized, can become a significant bottleneck, leaving expensive GPU resources underutilized and prolonging training times. This document outlines our approach to leveraging the **NVIDIA Data Loading Library (DALI)** to accelerate the training process for our **ResNet + BiLSTM + Attention** OCR model built with **PyTorch Lightning**. We demonstrate substantial speedups compared to standard data loading methods and showcase the importance of hardware-aware pipeline configuration.

## 2. Data Processing and Details

### 2.1. Character Normalization

To create a uniform character set for the OCR model, the following text normalizations are applied to the labels:

| Original Character(s) | Description        | Normalized Character |
| :-------------------- | :----------------- | :------------------- |
| `“`, `”`             | Smart Quotes       | `"`                  |
| `’`                   | Typographical Apostrophe | `'`                  |
| `–`, `—`, `−`         | Various Dashes     | `-`                  |
| `…`                   | Ellipsis           | `...`                |
| `Ð`                   | Icelandic Eth (Uppercase) | `Đ`                  |
| `ð`                   | Icelandic Eth (Lowercase) | `đ`                  |
| `Ö`, `Ō`             | O with accents     | `O`                  |
| `Ü`, `Ū`             | U with accents     | `U`                  |
| `Ā`                   | A with macron      | `A`                  |
| `ö`, `ō`             | o with accents     | `o`                  |
| `ü`, `ū`             | u with accents     | `u`                  |
| `ā`                   | a with macron      | `a`                  |

This normalization simplifies the vocabulary the model needs to learn.

### 2.2. Datasets Used

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dataset.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

The training leverages a combined dataset from the following sources:

| Dataset                                                                                  | Train Samples | Validation Samples | Notes                       |
| :--------------------------------------------------------------------------------------- | --------------: | -------------------: | :-------------------------- |
| [vietocr](https://github.com/pbcquoc/vietocr)                                           | 441,025        | 110,257             | Random word images removed |
| [Paper (Deep Text Rec. Benchmark)](https://github.com/clovaai/deep-text-recognition-benchmark) | 3,287,346      | 6,992               |                             |
| [Synth90k](https://www.robots.ox.ac.uk/~vgg/data/text/)                                  | 7,224,612      | 802,731             |                             |
| [Cinnamon AI (Handwritten)](https://www.kaggle.com/datasets/hariwh0/cinnamon-ai-handwritten-addresses) | 1,470          | 368                 |                             |
| **Combined Total**                                                                       | **~11.0 M**    | **~0.9 M**          |                             |

**Vietnamese Data:** Please note that Vietnamese samples constitute only **1.76%** (209,120 images) of this combined dataset, from **VietOCR** (207,282) and **Cinnamon AI** (1,838). This reflects the limited availability of public Vietnamese OCR data.

## 3. Training Strategy

Due to the significant imbalance in the dataset, with English samples heavily outweighing Vietnamese ones, a direct training approach proved challenging. Training exclusively on the VietOCR dataset resulted in instability. Therefore, we adopted a two-stage training strategy:

1.  **Initial Pre-training:** The model was first trained on the entire combined dataset of approximately 11 million images. This allowed the model to learn general text recognition features from a large and diverse dataset, even though it was predominantly English.
2.  **Fine-tuning on VietOCR:** The checkpoint obtained from the initial pre-training phase was then used to fine-tune the model specifically on the VietOCR dataset. This step adapted the learned features to the nuances of Vietnamese text and improved performance on the target language.

The model we release is the one obtained after this fine-tuning process on the VietOCR data. You can see an example of how we perform inference with this model in our [inference notebook](https://github.com/ducto489/lib_ocr/blob/training/inference/inference.ipynb).

## 4. Why NVIDIA DALI?

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Pytorch-Dataloader.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

DALI is designed specifically to address data pipeline bottlenecks in deep learning workloads. Its key advantages include:

*   **Pipeline Parallelism:** DALI overlaps data loading, preprocessing, and GPU computation, minimizing idle time for both CPU and GPU.
*   **Optimized Kernels:** It provides highly optimized CPU and GPU implementations (using C++/CUDA) for common data manipulation tasks (decoding, resizing, color augmentation, etc.), executing much faster than typical Python libraries.
*   **Framework Integration:** Seamlessly integrates with popular frameworks like PyTorch, TensorFlow.
*   **Hardware Flexibility:** Allows fine-grained control over whether operations run on the CPU or GPU, enabling optimization for diverse hardware configurations.

## 5. When and How to Integrate DALI for Faster OCR Training

Before integrating DALI, it's crucial to determine if it's the right tool for your specific bottleneck. As the DALI documentation suggests:

> **"Q: How do I know if DALI can help me?**
> A: You need to check our docs first and see if DALI operators cover your use case. Then, try to run a couple of iterations of your training with a fixed data source - generating the batch once and reusing it over the test run to see if you can train faster without any data processing. If so, then the data processing is a bottleneck, and in that case, DALI may help."

Following this guidance:

1.  **Verify Operator Coverage:** Check the [official DALI documentation for supported operators](https://docs.nvidia.com/deeplearning/dali/user-guide/docs/supported_ops.html). Ensure DALI provides the necessary functions for your OCR preprocessing pipeline (e.g., image decoding, resizing, padding, rotation, color adjustments, noise addition). Our pipeline utilizes operators like `fn.decoders.image`, `fn.resize`, `fn.rotate`, `fn.color_twist`, `fn.warp_affine`, and `fn.noise.gaussian`, all readily available in DALI.
2.  **Identify the Bottleneck:** Perform the suggested test. Modify your existing training loop (without DALI) to load and preprocess *one single batch* of data, then repeatedly feed this *same batch* to the model for several training steps. Compare the training speed (e.g., iterations/second or time per step) in this fixed-data scenario to your normal training speed. If the fixed-data training is significantly faster, it strongly indicates that your data loading and preprocessing pipeline is limiting overall performance, and DALI is likely to provide a speedup.

**How DALI Delivers Performance Gains:**

If the above checks suggest DALI is suitable, here's how it achieves acceleration:

*   **Executing Operations Efficiently:** DALI replaces standard Python-based processing (like using PIL or OpenCV within a PyTorch `Dataset`) with highly optimized C++ and CUDA kernels. This drastically reduces the overhead associated with Python execution for common data manipulation tasks.
*   **Pipeline Parallelism:** DALI constructs a computational graph for your data pipeline. It can then execute different stages of this graph (e.g., reading data, CPU-based augmentation, GPU-based augmentation, transferring data to the GPU) in parallel and asynchronously with the main model training loop running on the GPU. This minimizes idle time where the GPU might be waiting for the next batch.
*   **Hardware-Adaptive Execution:** DALI allows you to specify whether individual operations should run on the CPU (`device='cpu'`) or the GPU (`device='gpu'`). This is critical for optimization. As our Case Studies show:
    *   On systems with very strong CPUs (like the A6000), performing augmentations on the CPU (`device='cpu'`) might be faster, preventing contention on the main training GPU.
    *   On systems where the CPU is less powerful relative to the GPU (like the L4), offloading augmentations to the GPU (`device='gpu'`) frees up the CPU and leads to better overall throughput.
    This flexibility allows tuning the pipeline for optimal performance on *your specific hardware*.
*   **Optimized Data Reading:** While our example uses `fn.external_source` for flexibility with individual files, DALI also offers highly optimized readers for various packed dataset formats (TFRecord, RecordIO, Caffe LMDB, **WebDataset**). If your bottleneck test reveals slow performance even with minimal augmentations, or if dealing with slow storage (like the HDD in Case 3), switching to one of these formats and using the corresponding DALI reader (`fn.readers.*`) can dramatically improve data ingestion speed *before* the processing steps.

## 6. Practical Integration and Experimental Results

Integrating DALI into our PyTorch Lightning workflow involves a few key components, demonstrated in our codebase. We define DALI pipelines using the `@pipeline_def` decorator, specifying data loading, augmentation, and processing steps using `nvidia.dali.fn` operators. For data loading from our custom format (images in a folder, labels in CSV), we utilize `fn.external_source` coupled with a Python callable (`ExternalInputCallable`) that reads image bytes and encodes labels. To feed data into the PyTorch Lightning training loop, we wrap the DALI pipeline using `DALIGenericIterator` (specifically, our `LightningWrapper` subclass for convenience) which handles batch collation and transfer to the GPU. This setup replaces the standard PyTorch `DataLoader`. For those interested in the specific implementation details, please refer to the `DALI_OCRDataModule` and associated classes within our project's source code.

We tested our DALI implementation across different hardware setups against a baseline PyTorch DataLoader (`No DALI`).

**(Note:** Dataset size and specifics impact absolute times, but relative speedups are indicative.)

### Case 1 High-End GPU with Strong CPU (NVIDIA A6000)
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ocr_training_time_a6000.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
*   *Observation:* High GPU utilization with the standard loader hid a data pipeline bottleneck. DALI running augmentations on the strong **CPU** provided the best speedup (~13%), demonstrating its superior efficiency (optimized kernels, parallelism) over standard Python processing even on capable hardware. DALI on CPU outperformed DALI on GPU here, suggesting CPU execution was more efficient for this specific workload, likely due to lower overhead.

### Case 2 Cloud GPU (NVIDIA L4)
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ocr_training_time_l4.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
*   *Observation:* On this balanced system, offloading DALI augmentations to the GPU provided the best performance, overcoming the CPU bottleneck observed in the CPU-only DALI configuration.

### Case 3 Mid-Range GPU with Slow Storage (NVIDIA 3060 + HDD)
*   **DALI (Augmentations on CPU):** Extremely slow, GPU utilization frequently hit 0%. Disk I/O was maxed out.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HDD-bottleneck.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
*   *Observation:* DALI optimizes *processing*, but cannot overcome fundamental I/O limitations from slow storage when reading many small files.
*   *Recommendation:* Combine DALI with optimized data formats like **WebDataset** using DALI's dedicated readers (`fn.readers.webdataset`) to address the data loading bottleneck first.

## 7. Analysis and Conclusion

Our experiments clearly demonstrate that:

1.  **DALI significantly reduces OCR training time** when the data pipeline is a bottleneck (up to ~25% speedup observed on L4).
2.  The **optimal placement of DALI operations (CPU vs. GPU) is hardware-dependent.** Tuning the `device` parameter for operators is crucial for maximizing performance.
3.  **I/O is critical.** On systems with slow storage, optimizing the *dataset format and reading method* (e.g., using WebDataset with DALI's readers) is essential *before* DALI's processing speedups can be fully realized.

By correctly identifying bottlenecks and leveraging DALI's optimized kernels, parallelism, and hardware-adaptive execution, we can significantly accelerate OCR model training, enabling faster experimentation and development.