# L2 · PyTorch 与资源核算 · 速记

> 一句话：L2 不教你怎么搭模型，它教你给训练"算账"——多少 FLOPs、多少显存、GPU 到底喂不喂得饱。

视频：https://www.bilibili.com/video/BV11LEA6eEuj/?p=2

## 核心矛盾

- 课程目标是"固定资源下训最好的模型"，第一步必须能**测量**效率——不知道一个张量怎么存、一次前反向做多少次运算，所有优化只能靠猜。
- 硬件规格表还常注水（按稀疏性标称），直接拿来算会错一倍，算完要除以 2。[跳 25:21](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1521)

## 关键结论

1. **内存 = 元素数 × 字节数**：fp32 4 字节 → bf16 2 → fp8 1 → fp4 半字节。bf16 因动态范围等同 fp32 成为训练默认；配 AMP 自动混合精度，matmul 用 bf16、exp 这类危险操作保留 fp32。[跳 11:33](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=693) [跳 07:40](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=460) [跳 08:04](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=484)
2. **量化 ≠ 直接训低位**：先用高精度（bf16+）把模型训好再压到 int4 容易；直接训练 1-bit 语言模型目前没有可信成果。fp8 两个版本：E4M3 偏精度、E5M2 偏动态范围。[跳 11:09](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=669) [跳 08:34](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=514)
3. **6ND**：线性层前向 `2 × token × 参数`，反向恰好是前向 2 倍，合起来 **前向 2ND + 反向 4ND = 6ND**——估训练成本的万能尺。注意 FLOPs（计算量）≠ FLOP/s（硬件速度）。[跳 21:44](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1304) [跳 42:29](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=2549) [跳 46:14](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=2774) [跳 18:52](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1132)
4. **算术强度决定瓶颈**：强度 = FLOPs ÷ 搬运字节。H100 临界约 **295**（每搬 1 字节约做 300 次运算）；ReLU 0.25、GeLU 5、点积 0.5、matvec 都内存受限，只有大 matmul 计算受限。屋顶线图 = 斜线（内存受限）+ 平台（计算受限）。[跳 29:25](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1765) [跳 34:25](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=2065) [跳 28:10](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1690)
5. **MFU = 实际 FLOP/s ÷ 标称 FLOP/s**：现代模型约 0.5 即可接受；MFU 偏低通常是内存带宽在拖后腿，不是算力不够。[跳 25:21](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=1521)
6. **显存四本账**：参数 + 梯度 + 优化器状态 + 激活。优化器状态不是速度瓶颈（不频繁参与 matmul），但体量最大、决定能不能把模型塞进显存；Adam 每参数 8 字节（一阶 + 二阶矩，fp32 保稳定）。[跳 49:33](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=2973) [跳 50:48](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=3048) [跳 50:23](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=3023)
7. **省显存两招**：梯度累积（拆 micro-batch 累加梯度，用小显存模拟大 batch）；激活检查点（只存 √L 层的激活，其余反向时重算，用时间换空间）。[跳 52:03](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=3123) [跳 52:28](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=3148) [跳 53:43](https://www.bilibili.com/video/BV11LEA6eEuj/?p=2&t=3223)

## 我的增量

- 承上启下：L2 是 L10 的"因"——decode 阶段算术强度 ≈ 1，深深落在屋顶线左侧的内存受限区，所以优化 KV Cache / 量化比优化 FLOPs 有用得多。
- 项目关联：6ND 估算力 + 屋顶线判瓶颈，正是 `warp-candidate-filter` 里判断 kernel 是访存受限还是计算受限的同一套语言。
- 面试讲法：被问"怎么估算训练成本 / 判断瓶颈" → 先 6ND 算总 FLOPs，再用算术强度对临界值判瓶颈，最后用 MFU 量化利用率。

## 术语

tensor · fp32/bf16/fp8/fp4 · AMP · 量化 · einsum · rearrange · FLOPs · FLOP/s · 6ND · 算术强度 · roofline · MFU · 梯度累积 · 激活检查点

---
笔记为个人学习整理，观点来自 CS336 课程视频字幕，仅作引用。
