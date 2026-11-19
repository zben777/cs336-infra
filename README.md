# cs336-infra

Stanford CS336《Language Modeling from Scratch》精简学习笔记，按 AI Infra 方向重组。

## 这是什么

> 把 CS336 18 讲压成一讲一页的中文速记，附原视频时间戳与个人实现增量。

这门课覆盖从 tokenizer、Transformer、训练系统到推理与对齐的完整链路，是 AI Infra 方向最硬的实践课之一。但 18 讲、每讲近一小时，逐句整理成博客后信息密度很低。这里换个做法：**每讲只留一页**——核心矛盾、3-5 个必须记住的结论、可直接跳回原视频的时间戳，以及我自己的理解和项目关联。

重点放在 systems 与 inference 主线：L2 资源核算、L5-6 GPU/Triton、L10 Inference，并与 CUDA kernel 优化、KV Cache 量化等个人项目对照。目标是复习时一屏读完，而不是收藏后不再打开。

## 目录

| 目录 | 用途 |
| :--- | :--- |
| `lectures/` | 逐讲速记，一讲一页（主体） |
| `topics/` | 跨讲专题笔记（roofline、并行策略等） |
| `lab/` | 个人代码实验（kernel 复现、KV cache 量化等） |

## 当前进度

骨架搭建中。讲次索引与模板见 [`lectures/README.md`](lectures/README.md)，新讲次从 [`lectures/_TEMPLATE.md`](lectures/_TEMPLATE.md) 复制。

## 来源与版权

笔记为个人学习整理，观点来自 CS336 课程视频字幕与公开课材料，仅作引用；视频截图版权归原作者所有。
