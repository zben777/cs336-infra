# L1 · 课程概览与分词 · 速记

> 一句话：唯一可靠的理解方式是从零构建；四条论证收束成一个问题——固定预算下如何构建最好的模型；第一个零件是分词，主流方案是 BPE。

视频：https://www.bilibili.com/video/BV11LEA6eEuj/?p=1

## 核心矛盾

- 抽象层会漏：只调 prompt 会锁死你能考虑的方案集合，前沿模型又贵又不公开，所以只能自己把整个技术栈推倒重来。[跳 02:56](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=176) [跳 03:21](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=201)
- 小模型 ≠ 大模型缩小版：算力占比随规模变（MLP 44%→80%）、行为随规模涌现，小规模实验不能外推。[跳 04:51](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=291) [跳 05:35](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=335)
- 分词的核心矛盾：词表要大（压缩比高）还是序列要短（注意力便宜）——后面所有方案都是在这两者间取权衡。

## 四条论证（前半场主线）

四点不是并列知识点，是一条递进的因果链：

1. **① 从零构建**：抽象层会漏，只靠 prompt 做基础研究会被锁死设计空间；动手搭一遍才能拿到可迁移的**机制 + 思维模式**（直觉只能在能做大实验的地方获得）。[跳 02:56](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=176) [跳 06:00](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=360)
2. **② 规模改变一切**（所以难外推）：训练 GPT-4 传闻 ~1 亿美元、现已到 10 亿量级且细节全不公开；MLP 算力占比 44%→80%，上下文学习到临界规模才涌现；有些建模决策（如 SwiGLU）只能实验试出来。[跳 04:01](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=241) [跳 07:18](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=438)
3. **③ 效率是主线**（所以效率定上限）："苦涩的教训"常被误读成"规模决定一切"，正确解读是**可扩展的算法才是关键**。模型表现 ≈ 效率 × 资源；ImageNet 算法效率 2012–2019 提升 44 倍，规模越大效率越值钱。[跳 07:50](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=470) [跳 08:40](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=520)
4. **④ 固定预算下的最优模型**（收束为一个优化问题）：**在给定的数据和计算预算下，能构建出的最优模型是什么？** 把预算钉死（C ≈ 6ND），问题就变成预算怎么分——模型做大 (N↑) 还是数据喂多 (D↑)：Kaplan vs Chinchilla 计算最优，经验法则 D ≈ 20×N（8B ≈ 1.4T token），但如今为压推理成本故意喂到超出计算最优。[跳 08:47](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=527) [跳 38:25](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=2305)
   - **最妙一句**：可预测性至少不亚于最优性——跑一堆小实验拟合缩放律去预测大模型，MARIN 还先预测再真训验证。[跳 39:40](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=2380)

## 分词与 BPE（后半场硬知识）

1. **分词是双向翻译官**：Unicode 字符串 ↔ token 索引序列，必须支持无损失往返转换；"hello" 和 " hello" 是不同 token，数字切法也因分词器而异。[跳 50:51](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3051) [跳 51:41](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3101)
2. **压缩比 = 字节数 ÷ token 数**，越大越好（注意力是序列长度平方，序列短一半省四倍）；但靠增大词表提压缩比会引发稀疏性，多语言词表现已 10–20 万。[跳 52:31](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3151)
3. **三种朴素方案都有硬伤**：字符级词表 ~15 万但利用率低；字节级词表仅 256 但序列长、压缩比 = 1；词级有语义但词表无界、测试遇 UNK。[跳 53:21](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3201) [跳 55:01](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3301)
4. **BPE = 折中方案**：从 256 个字节出发，① 统计相邻对频次 → ② 合并最高频对成新 token（如 256 = "th"）→ ③ 重复，序列越来越短、词表越来越大；任何文本都能分词、无 UNK。玩具示例 "The Cat in the Hat" 压缩率 1.5，首次用于 LM 是 GPT-2。[跳 56:16](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3376) [跳 57:05](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3425)
5. **作业一的坑**：朴素 `encode` 要遍历所有合并规则（≈词表大小，10 万级）非常慢；改成只遍历"关键合并"并建索引。工程上还要先分块再分词，Python 不够快可换 Rust。[跳 58:45](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=3525)

## 课程地图（一句话带过）

五大部分对应五项作业：**基础**（分词/架构/优化器/训练）、**系统**（内核/并行/推理）、**缩放定律**（拟合+外推）、**数据**（抓取/过滤/清洗）、**对齐**（DPO/GRPO）。[跳 21:26](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=1286)
资源核算贯穿全课：训练约需 **6ND** FLOPs，瓶颈常在内存搬运而非算力，用 roofline 判断——这条线在 L2 展开。[跳 28:27](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=1707) [跳 29:01](https://www.bilibili.com/video/BV11LEA6eEuj/?p=1&t=1741)

## 我的增量

- 项目关联：第 ② 点"MLP 占比随规模变"直接影响 kernel 优化的判断——小规模上调出的注意力 kernel 策略，到大规模可能打偏，要看 FFN。第 ④ 点"固定预算下最大化效率"是 vLLM-TQ / warp-candidate-filter 整条技术路线的理论根据：写 kernel、做量化、调并行本质都是在固定预算下榨出更多有效算力。
- 面试讲法：被问"为什么要从零学 / 你怎么理解 scaling" → 用小模型不可外推（算力占比变 + 涌现）解释，再落到"效率 × 资源"和"可预测性 ≥ 最优性"。
- 待验证：在本地 tokenizer 上实测压缩比随词表大小的变化曲线；对照 L2 的 6ND 手算一遍 8B 模型 ≈ 1.4T token 的计算最优。

## 术语

tokenization · tokenizer · roundtrip · compression ratio · BPE · byte pair encoding · UNK · vocabulary · GPT-2 · 6ND · roofline · scaling recipe · compute-optimal · Chinchilla · 涌现 · 效率 × 资源

---
笔记为个人学习整理，观点来自 CS336 课程视频字幕，仅作引用。
