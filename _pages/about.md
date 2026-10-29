---
permalink: /
title: "About Me"
author_profile: true
excerpt: "PhD student at HKUST working on efficient LLM inference, low-bit quantization, and GPU–CPU memory management."
redirect_from:
  - /about/
  - /about.html
---

I am **Yangjia Hu (胡揚嘉)**, a PhD student in Computer Science and Engineering at the **Hong Kong University of Science and Technology (HKUST)**, advised by **Prof. Song Guo**. I joined HKUST in September 2026.

My research focuses on **efficient LLM inference**, especially low-bit quantization and GPU–CPU memory management. I am interested in connecting algorithm design with the memory, communication, and execution constraints of real systems.

I received my bachelor's degree in Computer Science and Technology from the **University of Science and Technology of China (USTC)** in June 2026. I also interned with Tencent's Inference Infrastructure — Base Engine Team, working on KV cache prefetching and offloading for long-context inference.

[English Research CV]({{ '/files/HuYangjia_CV.pdf' | relative_url }}){: .btn .btn--primary}
[Research & Projects]({{ '/experience/' | relative_url }}){: .btn}
[Email](mailto:yhudv@connect.ust.hk){: .btn}

## Selected Research & Industry Experience

### MosaicQuant: Inlier-Outlier Disaggregation for Unified 4-Bit LLM Quantization

**Co-first author** · HKUST · Revised in 2026

I proposed **dense–sparse 4-bit residual compensation** and conducted the main experiments. MosaicQuant combines a dense 4-bit base with block-sparse 4-bit residuals, with accuracy evaluated in **INT4 and FP4**. The INT4 implementation achieves up to **1.24× the end-to-end throughput of TensorRT-LLM** on Qwen3-8B with an RTX 4090. FP4 W4A4 reaches **74.61%** average accuracy across six zero-shot tasks on Qwen3-32B, compared with **75.2%** for W16A16.

[Earlier arXiv Version (not yet updated)](https://arxiv.org/abs/2606.15652) · [Project & Research Process]({{ '/research/mosaicquant/' | relative_url }}) · [中文研究回顾]({{ '/blog/#mosaicquant' | relative_url }})

### Tencent: KV Cache Prefetching & Offloading

**Software Development Intern** · Inference Infrastructure — Base Engine Team · Beijing · Mar – Jul 2026

I led successful **proof-of-concept testing** for a KV cache offloading approach and worked on vLLM integration prototypes. The design adds a **mapping layer** to preserve block-level prefix caching while exchanging KV data at token granularity. My work concluded with the MAP design and confirming GPU execution for cache-management operations. Colleagues handled subsequent SM configuration, overlap scheduling, and implementation.

[Work Description]({{ '/experience/#tencent' | relative_url }}) · [实习工作记录]({{ '/research/tencent-kv-cache/' | relative_url }})

### KV Cache Prefetching for Long-context LLM Inference

**Undergraduate thesis** · USTC · Sep 2025 – Jun 2026 · Advisor: Jianliang Lu

I studied step-wise access locality in DeepSeek-V3.2 sparse attention, designed locality-based prefetching and dual-watermark eviction, and built a LongBench-v2 trace-driven simulator. In this simulation, per-step H2D transfers fell from **108 to 32 KV entries**, and the cache miss rate from **5.28% to 1.58%**, compared with on-demand loading.

[Thesis Details]({{ '/experience/#kv-cache-prefetching' | relative_url }})

## Education

- **Hong Kong University of Science and Technology**, Hong Kong<br>
  PhD student in Computer Science and Engineering · Sep 2026 – present<br>
  Advisor: Prof. Song Guo
- **University of Science and Technology of China**, Hefei, China<br>
  B.Eng. in Computer Science and Technology · Sep 2022 – Jun 2026<br>
  GPA: **3.91/4.3** · Rank: **4/167**

## Selected Honors

- **Guo Moruo Scholarship** (top 1%), 2026
- **Hong Kong Government Scholarship**, 2026
- **National Scholarship** (top 2%), 2024 and 2025
- **Third Prize, National Finals**, 8th National College Students Computer System Ability Competition, CPU Track (team award), 2024
- **National Encouragement Scholarship**, 2023

## Technical Background

Python, C/C++, Rust, CUDA, PyTorch, and vLLM; experience with quantization experiments, GPU/CPU performance analysis, trace-driven simulation, and KV cache management. My earlier projects also involved CPU microarchitecture, SystemVerilog, and FPGA systems.

## Contact

**Email:** [yhudv@connect.ust.hk](mailto:yhudv@connect.ust.hk)<br>
**Affiliation:** Computer Science and Engineering, HKUST, Hong Kong
