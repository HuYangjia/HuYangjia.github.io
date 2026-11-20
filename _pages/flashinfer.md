---
layout: single
title: "FlashInfer：GPU 采样边界修复"
permalink: /research/flashinfer-sampling/
author_profile: true
lang: zh-CN
excerpt: "FlashInfer PR #3774：修复 rejection sampling 的边界 tie 过度接受，统一采样与过滤语义，并迭代 SM90 的正确性、测试与性能。"
---

**开源贡献 · CUDA/C++ · PR 作者**<br>
2026 年 6 月发起，持续修订至 11 月 · **评审中，尚未合并**（2026-11-11 核对）

[Pull Request #3774](https://github.com/flashinfer-ai/flashinfer/pull/3774){: .btn .btn--primary}
[经历介绍（English）]({{ '/experience/#flashinfer' | relative_url }}){: .btn}
[Research Notes]({{ '/blog/' | relative_url }}){: .btn}

## 要解决的问题

当多个 token 的概率恰好相同，并且这组 token 横跨 top-k 或 top-p 的截断边界时，只按概率值比较，可能把本应排除的 token 也接受下来。修复需要同时保证保留集合正确，以及最终样本确实来自这个集合。

我在 FlashInfer 发起了 [PR #3774](https://github.com/flashinfer-ai/flashinfer/pull/3774)，围绕这一问题修改 CUDA rejection sampling，并在维护者的反馈下将规则扩展到相关过滤和归一化接口。

## 从局部修复到统一接口语义

PR 使用统一顺序：**概率或 logit 降序，相同值按原始 token 索引升序**。

- **Top-k：**恰好保留 `min(k, vocab_size)` 个位置。
- **Top-p：**保留累计概率达到阈值的最短前缀，包括跨过阈值的那个 token。
- **接口一致性：**将同一规则应用到直接采样、概率重新归一化、logit masking，以及 top-k-first 的不同执行路径，同时保留各过滤顺序原有的中间归一化语义。

例如，概率为 `[0.4, 0.3, 0.3]`、`k=2` 时，保留原始索引为 0 和 1 的两个位置。相同输入下，`p=0.6` 的最短前缀同样包含这两个位置，累计概率为 0.7。

修复还涉及 rejection loop 的退出条件：已经被边界规则拒绝的 proposal，不能因为搜索区间收敛而作为最终样本返回。规则和用户可见的行为变化也同步写入了 [PR 中的 API 文档与代码](https://github.com/flashinfer-ai/flashinfer/pull/3774/files)。

## CUDA 实现与架构适配

评审过程逐步暴露了单个 kernel 之外的问题：新增逻辑带来的寄存器压力可能让 1,024-thread kernel 无法启动；上游新增的 SM90 split-row 路径也需要同样的边界修复。

后续修订中，我为 rejection kernels 增加 launch bounds，并把边界处理扩展到跨 CTA rank 的 cluster 路径。各 rank 利用已经暂存的数据分担处理，包含最终保留边界的 rank 再确定原始索引 cutoff，保留合法 proposal 和已有的随机数历史。

针对频繁出现相同值的输入，还需要控制额外扫描与同步的成本。最新修订在已有 top-p 阈值计算中携带 pivot 及以上的概率质量；当整组确定处于可保留范围内时，跳过 equality 验证。跨边界或数值上不确定的情况继续走精确处理。这一调整来自与维护者的[性能评审讨论](https://github.com/flashinfer-ai/flashinfer/pull/3774#discussion_r4515202214)。

## 精度与性能的取舍

我比较了原有 double 累加与 compensated float summation。补偿累加对部分 renormalization 用例有帮助，但同时带来 sampling 回归，因此最新默认实现保留原有 double 累加；补偿实现通过显式开关启用，并使用独立的 JIT 模块与缓存标识，便于复现实验。

PR 最近一轮 H800 测试在同一块 GPU 上比较固定 main、上一版 PR，以及新修订的默认和补偿模式，覆盖 fp32、bf16-derived 和严格无重复值的输入。

| 观察 | 比较条件 |
| --- | --- |
| 默认修订的 top-p 延迟下降 **4.81%–13.79%** | 相对上一版 PR，所报告的 fp32/bf16-derived 输入，batch 1/32/512 |
| 开启补偿累加后，batch-512 top-p 延迟增加约 **10.97%–11.15%** | 相对新修订的默认模式 |
| 默认模式仍有性能开销 | 相对固定 main，所报告的采样项最高约 **+6.22%**，top-p renorm 最高约 **+34.88%** |

这些数字都是特定测试中的**延迟变化**。相对上一版 PR 的改善与相对固定 main 的剩余开销，使用的是不同基线。完整环境、输入、提交版本与结果表见我的[最新实验回复](https://github.com/flashinfer-ai/flashinfer/pull/3774#discussion_r4524159032)。

## 回归验证

新增或强化的 GPU 测试覆盖非法边界候选、8192/131072 等大词表、跨 rank 边界、保留集合上的采样分布、signed zero、累计阈值精度，以及直接采样与过滤路径的一致性。

最新 PR 记录中，默认和补偿两种模式分别运行三份测试文件，每种模式均为 **2,727 passed、792 skipped、0 failed**。性能实验也记录源代码、输入和二进制标识，以核对不同实现使用的测试条件。

## 当前进展

截至 **2026 年 11 月 11 日**，[PR 仍然开放并处于评审中](https://github.com/flashinfer-ai/flashinfer/pull/3774)。这段经历包括边界语义梳理、CUDA 实现、GPU 回归测试，以及根据维护者反馈反复验证正确性与性能取舍。
