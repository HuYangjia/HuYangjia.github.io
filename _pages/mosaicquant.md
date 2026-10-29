---
layout: single
title: "MosaicQuant"
permalink: /research/mosaicquant/
author_profile: true
excerpt: "Unified 4-bit LLM quantization with INT4 and FP4 accuracy evaluation, and the research path from CPU compensation to block-sparse GPU execution."
---

**Inlier-Outlier Disaggregation for Unified 4-Bit LLM Quantization**

HKUST · Advisor: Prof. Song Guo · **Co-first author**<br>
Initial research: Jul – Sep 2025 · Revised manuscript, 2026

[Earlier arXiv Version](https://arxiv.org/abs/2606.15652){: .btn .btn--primary}
[中文研究回顾]({{ '/blog/#mosaicquant' | relative_url }}){: .btn}
[All Projects]({{ '/experience/' | relative_url }}){: .btn}

**Version note:** The linked arXiv paper is an earlier version and **has not yet been updated** to the revised manuscript described here. The method, INT4/FP4 accuracy results, and system evaluation below reflect the revised manuscript.

MosaicQuant combines a **dense 4-bit base with block-sparse 4-bit residuals** to compensate for outlier-induced quantization error. The framework couples this representation with activation-aware block selection and fused execution, with accuracy evaluated in **INT4 and FP4**. Its reported system performance uses the INT4 implementation.

I **proposed the dense–sparse low-bit compensation idea and conducted the main experiments**. My work also included exploring CPU/GPU execution, analyzing bottlenecks, testing cuSPARSE, and comparing sparse representations. This page describes how those experiments shaped the project, followed by the current framework and results.

## The Starting Question

Four-bit quantization can reduce the memory and computation required by LLM inference, but a small set of outliers can cause substantial quantization error. Keeping a separate compensation branch provides a way to recover information that the base representation loses. The systems question is how much that branch costs in data movement and computation.

My initial idea was to use **two INT4 branches**, with the sparse part computed on the CPU and the dense part on the GPU. The project evolved as experiments revealed different bottlenecks in different model families and execution choices.

## Research Process

### 1. INT4+INT4 and an Initial CPU–GPU Split

I proposed an INT4+INT4 scheme and initially considered assigning the sparse branch to the CPU. The motivation was to keep the main computation on the GPU while using the CPU for the smaller compensation workload.

This was the starting execution strategy. Whether it was practical depended on the cost of moving data between the two devices and carrying out the sparse computation.

### 2. DiT Experiments Exposed a Bandwidth Bottleneck

I first explored the idea on diffusion transformers (DiTs). In those experiments, **CPU–GPU bandwidth became the bottleneck**. I did not resolve that problem in the DiT setting, so the next step was to investigate another workload.

The immediate issue in this stage was the communication cost of the explored CPU–GPU execution path. That observation drove the change in model family.

### 3. Moving to LLMs Led to ZipperQuant

I moved the investigation to LLMs. In the LLM setting I explored, cross-device bandwidth was no longer the central problem, allowing the quantization idea to develop further. This became the initial work called **ZipperQuant** in 2025.

ZipperQuant and MosaicQuant are successive versions of the same research project. The later version changed the execution approach and was revised in 2026 under the MosaicQuant name.

### 4. GPU Sparse Execution Introduced Another Bottleneck

After a suggestion about GPU sparse-compute support, I revisited the device assignment and tried running the sparse work on the GPU. I used **cuSPARSE** to test that direction.

The performance was unsatisfactory: **the sparse computation itself became the bottleneck**. Moving the work to the GPU therefore led to a further question about the structure and execution cost of the sparse branch.

### 5. Comparing Sparse Representations

I compared structured-sparse and block-sparse alternatives while considering the quantization accuracy loss. I selected **block-sparse compensation** as the more suitable direction for this work.

This connected the quantization choice to the execution design: the sparse representation had to preserve useful residual information and provide a form of work the GPU could process efficiently. The final framework combines block selection with a specialized execution engine.

### 6. Extending from INT4 to FP4

The initial numerical format was INT4. The project subsequently retained INT4 and added **FP4 accuracy evaluation**, extending the method's scope to unified 4-bit dense–sparse residual compensation.

The revised manuscript evaluates FP4 on **Qwen3-4B, 8B, 14B, and 32B** under both W4A8 and W4A4. Quantization and all accuracy evaluations run on an H200. System throughput and kernel-latency measurements use the **INT4 implementation on an RTX 4090**; the current FP4 experiments evaluate accuracy.

### 7. Possible Further Format Extensions

Depending on further FP4 results, I would like to explore **NVFP4 and MXFP4**. These remain conditional future directions. The manuscript also identifies MXFP4-compatible packing, block scaling, and kernel adaptation for native FP4 computation as future work.

## The Final Framework

![MosaicQuant represents weights as the sum of a dense 4-bit base and block-sparse 4-bit residuals.]({{ '/images/mosaicquant-overview.svg' | relative_url }})

### Dense Base and Sparse Residuals

The weight representation is the sum of a **dense 4-bit base** and a small collection of **sparse 4-bit residual blocks**. Channel scaling first moves activation outliers into the weights, with the scaling strength selected per layer using calibration output error. The residual branch then compensates for important errors left by the base quantization.

### Activation-aware Δ-Hessian Block Selection

The framework scores residual blocks using the reduction in squared quantization error, weighted by sensitivity derived from calibration activations. It selects the Top-K blocks with the greatest estimated benefit. The optimality argument applies to the objective under a diagonal Hessian approximation.

### ZipperEngine: Fused Dense and Sparse Execution

ZipperEngine integrates sparse compensation and output merging into the dense GEMM execution. It fuses quantization with layout packing, reuses activation and output tiles, and overlaps memory access, quantization, and computation at the subtile level to hide auxiliary work. The system evaluation uses its INT4 execution path.

MosaicQuant is integrated into **vLLM**, with ZipperEngine serving as a custom execution backend for quantized linear layers. vLLM handles model execution, request scheduling, and KV cache management, while ZipperEngine jointly computes the dense base and selected sparse residual blocks.

These components form the team's framework. My specific contributions were the low-bit compensation idea, initially explored as INT4+INT4, the main experiments, CPU/GPU bottleneck analysis, cuSPARSE validation, and the choice of block-sparse compensation.

## Results and Evaluation Context

| Evaluation | Reported result | Setting |
| --- | --- | --- |
| INT4 end-to-end throughput | **1.21–1.24×** that of TensorRT-LLM | Qwen3-8B, RTX 4090, 2,048 input / 128 output tokens, batch sizes 4/8/16 |
| FP4 W4A4 zero-shot accuracy | **74.61%** vs. **75.2%** W16A16 | Qwen3-32B, six tasks, H200; 0.59 percentage-point gap |
| INT4 W4A4 zero-shot accuracy | **71.2%** vs. **73.5%** W16A16 | LLaMA3-8B, six tasks, H200 |
| Additional compensation memory | About **8.5–9.5%** | INT4, 10% residual-block budget |
| Additional compensation latency | About **5.3–6.2%** | INT4, 10% residual-block budget |

Calibration uses 128 sequences of 2,048 tokens from WikiText-2. **Quantization and all accuracy evaluations use an H200**, including the W16A16 references. **INT4 throughput and kernel-latency measurements use an RTX 4090**. Throughput and compensation overhead describe different evaluations.

### Qwen3 Accuracy across INT4 and FP4

The following results are average accuracy (%) across six zero-shot tasks under **W4A4**, from the revised manuscript's Tables 1 and 4. W4A8 results are also reported in the manuscript.

| Model | W16A16 | MosaicQuant INT4 | MosaicQuant FP4 |
| --- | --- | --- | --- |
| Qwen3-4B | 69.4 | 62.61 | 66.91 |
| Qwen3-8B | 71.6 | 67.38 | 70.54 |
| Qwen3-14B | 74.2 | 71.86 | 73.50 |
| Qwen3-32B | 75.2 | 72.27 | 74.61 |

The FP4 accuracy gap to W16A16 ranges from **0.59 to 2.49 percentage points** across these four models. Native FP4 inference-performance evaluation remains future work.

## What the Experiments Changed

Across the project, the limiting factor shifted from cross-device bandwidth in the DiT setting to sparse execution in the GPU implementation. Each observation changed the next design choice: the workload, the device assignment, and finally the sparse representation.

The later addition of FP4 broadened the numerical-format evaluation while retaining the original INT4 path. For me, the central lesson was to evaluate the quantization representation together with its execution path. A compensation branch needs to recover accuracy, and its data movement and computation have to fit the system running it.

## Manuscript and Public Version

**MosaicQuant: Inlier-Outlier Disaggregation for Unified 4-Bit LLM Quantization**<br>
Co-first author · Revised manuscript, 2026

The public arXiv version is **earlier and has not yet been updated** to the manuscript reflected in the results above.

[Earlier Abstract](https://arxiv.org/abs/2606.15652) · [Earlier Full Text](https://arxiv.org/html/2606.15652v1) · [Earlier PDF](https://arxiv.org/pdf/2606.15652)
