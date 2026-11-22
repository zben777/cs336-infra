# L10 · Inference · 速记

> 一句话：训练能并行，推理只能逐个吐 token，所有优化都在对抗内存带宽墙。

视频：https://www.bilibili.com/video/BV11LEA6eEuj/?p=10

## 核心矛盾

- 训练时序列维度可并行，attention/MLP 变成大矩阵乘，算术强度高、硬件喂得饱；推理是自回归，token 逐个生成，变成一堆又薄又碎的矩阵运算，硬件大部分时间搬数据而非算数。
- **prefill 计算受限**（注意力算术强度 S/2），**decode 内存受限**（注意力强度 ≈ 1）。[跳 23:27](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1407) [跳 26:22](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1582)
- H100 要算术强度 ~295 才算跑满，decode 差两个数量级——这就是"推理是 memory-bound"的根源。[跳 24:17](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1457)

## 关键结论

1. **分两阶段看**：prefill 填满 KV Cache（B×T 够大，不是瓶颈），decode 每步 T=1 复用 KV Cache，强度塌到 ~1。[跳 18:02](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1082) [跳 21:47](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1307)
2. **KV Cache 是显存大户**：大小 = B × S × (K/N × head_dim × 层数) × 2(bf16) × 2(K+V)，batch 一大先爆它，且延迟随 B 线性涨。[跳 29:17](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1757) [跳 30:57](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=1857)
3. **GQA / 跨层共享**：head 间共享 KV 把缓存降到 K/N；跨层共享再压一层，省内存即提速（因为是 memory-bound）。[跳 36:17](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=2177) [跳 43:40](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=2620)
4. **量化**：bf16 → int4，配 QAT 保精度；只对 memory-bound 的 decode 真正有效。[跳 50:29](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=3029)
5. **投机解码**：小模型猜、大模型验，接受率高时延迟降数倍。[跳 57:23](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=3443)
6. **TTFT 与吞吐天然冲突**：TTFT 小要小 batch，吞吐大要大 batch，调参就是在这条曲线上取点。[跳 35:02](https://www.bilibili.com/video/BV11LEA6eEuj/?p=10&t=2102)

## 我的增量

- 项目关联：vLLM-TQ 的 KV Cache 4-bit 正是结论 4 的落点；warp-candidate-filter 的访存/swizzle 优化经验可迁移到量化 kernel。
- 面试讲法：被问"推理为什么慢" → 先分 prefill/decode，再上 roofline + 算术强度，最后落到 KV Cache 和量化的取舍。
- 待验证：vLLM-TQ 加速比 [待补]；4090 上实测 KV Cache int4 的吞吐/延迟曲线，对比 GQA 的帕累托前沿。

## 术语

prefill · decode · KV Cache · GQA · MQA · 跨层共享 · QAT · 量化 · 算术强度 · roofline · TTFT · speculative decoding

---
笔记为个人学习整理，观点来自 CS336 课程视频字幕，仅作引用。
