---
layout: single
title: "Research & Projects"
permalink: /experience/
author_profile: true
excerpt: "Research and engineering experience in LLM quantization, KV cache management, systems software, and computer architecture."
---

My recent work focuses on efficient LLM inference. The projects below connect quantization and memory-management research with system implementation and performance analysis.

## MosaicQuant: Unified 4-bit LLM Quantization
{: #mosaicquant}

**HKUST · Advisor: Prof. Song Guo**<br>
Initial research: Jul – Sep 2025; revised in 2026 · **Co-first author**

Low-bit quantization reduces storage and computation, but outliers can introduce substantial error. Compensating for that error also introduces work that the execution system must handle efficiently. MosaicQuant addresses both sides with a **dense 4-bit base plus block-sparse 4-bit residuals** and fused execution. The current framework's accuracy evaluation covers **INT4 and FP4**.

**My contributions:** I proposed the dense–sparse low-bit compensation idea, initially explored as INT4+INT4, and conducted the main experiments. I explored CPU/GPU execution choices, identified communication and sparse-computation bottlenecks, tested cuSPARSE, and compared sparse representations before selecting block-sparse compensation.

The research developed through several changes in direction:

1. **Start with INT4+INT4 and CPU compensation.** I initially considered running the sparse branch on the CPU alongside the dense GPU computation.
2. **Test on DiT.** CPU–GPU bandwidth became the bottleneck, and I did not resolve it in that setting.
3. **Move to LLMs.** Cross-device bandwidth was no longer the central bottleneck in the explored LLM setting. This led to the initial work named ZipperQuant.
4. **Revisit GPU sparse execution.** Following a suggestion about GPU sparse-compute support, I tested cuSPARSE. Sparse computation itself became the bottleneck.
5. **Choose block sparsity.** I compared structured and block-sparse alternatives while considering quantization accuracy, and selected block-sparse residual compensation.
6. **Extend the numerical formats.** The project retained INT4 and added FP4 accuracy evaluation on Qwen3-4B/8B/14B/32B under W4A8 and W4A4. Depending on further FP4 results, NVFP4 and MXFP4 are possible future extensions.

The current framework combines activation-aware **Δ-Hessian block selection** with **ZipperEngine**. Its INT4 implementation achieves **1.21–1.24× the end-to-end throughput of TensorRT-LLM** on Qwen3-8B with an RTX 4090, using 2,048 input and 128 output tokens at batch sizes 4, 8, and 16. FP4 W4A4 reaches **74.61%** average accuracy across six zero-shot tasks on Qwen3-32B, compared with **75.2%** for W16A16. Quantization and all accuracy evaluations use an **H200**; reported FP4 results cover accuracy, while system performance measurements use INT4.

These results describe the revised manuscript. The public arXiv version is earlier and has **not yet been updated**.

[Read the Detailed Project & Research Process]({{ '/research/mosaicquant/' | relative_url }}){: .btn .btn--primary}
[Earlier arXiv Version (not yet updated)](https://arxiv.org/abs/2606.15652){: .btn}

## Tencent: KV Cache Prefetching & Offloading
{: #tencent}

**Software Development Intern · Inference Infrastructure — Base Engine Team · Beijing**<br>
Mar – Jul 2026

Following discussions with colleagues about my undergraduate KV cache work, a related team need led me to **lead proof-of-concept testing**. The tests succeeded, and the team decided to proceed with implementation. I worked on the HY4 prototype and vLLM integration effort, including step-locality prefetching and co-designing GPU/host cold-replica placement for **60K-token contexts** under prefill/decode disaggregation.

The integration introduced several systems problems:

- **Preserving the complete KV dataset:** GPU and CPU memory together held the full KV state, requiring cold data to be swapped out and needed data swapped in while preserving the full dataset.
- **Reconciling logical and physical granularity:** In the observed 64-token block case, only one token was needed, making whole-block exchange transfer about **64×** the necessary data. The resulting design introduced a **MAP layer**, retaining logical blocks for prefix-cache semantics while exchanging physical KV data at token granularity.
- **Placing cache-management work:** Profiled set-membership checks and LRU eviction analysis, identified a bandwidth bottleneck, and compared CPU and GPU prototypes. This favored executing the management work on the GPU.

My contributions covered proof-of-concept testing, integration prototypes, profiling, and the MAP design. **My work ended with the MAP design and confirming GPU execution.** Colleagues handled the subsequent SM configuration, choice of parallel execution stage, and follow-on implementation. Final production validation results have not been provided.

[Internship Notes (中文)]({{ '/research/tencent-kv-cache/' | relative_url }}){: .btn}

## KV Cache Prefetching for Long-context LLM Inference
{: #kv-cache-prefetching}

**Undergraduate thesis · USTC · Advisor: Jianliang Lu**<br>
Sep 2025 – Jun 2026

Long-context sparse attention creates a memory-management problem: the KV entries needed at each decode step must be available on the GPU, while the full cache may reside across GPU and host memory. I investigated whether access locality across consecutive steps could help prepare the next step's working set.

- **Access analysis:** Analyzed DeepSeek-V3.2 sparse-attention traces across 61 Transformer layers, measuring step-wise locality and layer-dependent cacheability.
- **Prefetching and eviction:** Designed step-locality prefetching using overlap in consecutive Top-K accesses, together with dual-watermark eviction. Prefetching runs asynchronously, while remaining misses are handled synchronously.
- **Evaluation:** Built a LongBench-v2 trace-driven KV cache simulator. Compared with on-demand loading, per-step H2D transfers decreased from **108 to 32 KV entries** (about 70%), and the miss rate from **5.28% to 1.58%**.

These are trace-driven simulation results. The related [Tencent internship]({{ '/experience/#tencent' | relative_url }}) focused on engineering this approach under production-system constraints.

## Linux Kernel bpf_trace Module in Rust
{: #rust-kernel}

**Systems course project · USTC**<br>
Apr – Jul 2024

In a team project, I worked on migrating Linux kernel `bpf_trace` code from C to Rust. My work included restructuring tracing data structures, migrating key routines, integrating the module into the Make build, and debugging behavioral consistency against the C implementation. The project explored memory safety and maintainability in kernel code.

## Earlier Architecture & FPGA Projects

### Dual-issue Out-of-order CPU

**Laboratory engineering project · Advisor: Prof. Weng, KAUST**<br>
Dec 2024 – Jun 2025

Used a new hardware-design language developed in the laboratory to design and implement a dual-issue out-of-order CPU, exploring how the language could support hardware construction.

### CPU Design Competition

**8th National College Students Computer System Ability Competition, CPU Track**<br>
May – Aug 2024 · **Third Prize, National Finals (team award)**

Our team designed an in-order, dual-issue, eight-stage CPU with AXI-4, instruction and data caches, a predecoder, and branch prediction. I worked on the CPU frontend and related pipeline modules, including FIFO and predecoder components, and participated in architecture design, optimization, and SystemVerilog functional verification.

### FPGA-based Piezoresistive Array Control & Data Acquisition

**Undergraduate research project · USTC · Advisor: Prof. Xiaohui Cai**<br>
Nov 2023 – Nov 2024

Analyzed system bottlenecks and optimized FPGA control and data acquisition with SystemVerilog. The acquisition frequency increased from **40 Hz to 80 Hz**. I also implemented a command-frame control system.
