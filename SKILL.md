---
name: tcms-adapter
version: "1.0.1"
description: |
  For tech-product marketing teams — adapts a reviewed core draft into channel-specific versions (WeChat, dev community, CN social, EN X, LinkedIn, sales one-pager) and repurposes published content.
  Never writes from scratch; that is tcms-writer's job. Keeps brand naming, data points, and customer redaction consistent across every channel version.
  Not for neutral industry research distribution — use tcms-writer/tcms-compliance-reviewer for brand-side content.
read_when:
  - 适配
  - 改写
  - 多渠道版本
  - 渠道版本
  - 公众号版
  - 社交帖
  - 英文帖
  - 二次推广
  - adapt
  - repurpose
allowed-tools:
  - read_file
  - write_to_file
  - replace_in_file
  - web_fetch
disable: false
---

# TCMS Adapter

将一篇已审核的核心稿（或已发布文章）适配为不同渠道所需的发布版本。不从零创作——那是content-writer的工作。

## 两种模式

**模式A：新内容多渠道适配**
输入：content-writer产出的核心稿
输出：指定渠道的适配版本

**模式B：存量内容二次推广**
输入：已发布文章的URL或本地文件
输出：社交帖、观点摘要、数据点提取等轻量物料

模式B步骤：
1. [确定性] 读取已发布文章
2. [LLM] 提取3个可传播的信息点（最硬的数据/最尖锐的观点/最有画面感的场景）
3. [LLM] 按目标渠道生成物料
4. [LLM] 自检（数据与原文一致/产品名规范/无敏感信息）
5. [确定性] 输出到 `content/adapted/repurpose/`

---

## Step 1: 确认输入和目标渠道 [确定性]

1. 确认核心稿路径（或已发布文章URL）
2. 确认需要适配的渠道列表
3. 读取核心稿内容

未指定渠道时默认生成全套：公众号版 / 社区版 / 中文社交帖×2 / 英文X帖×1 / 销售一页纸（仅含案例或硬数据时）

## Step 2: 读取渠道规范 [确定性]

按需读取对应渠道的规范文件（`references/channel-specs/`目录下），同时读取品牌规范（复用content-writer的brand-rules.md）。

## Step 3: 适配生成 [LLM]

各渠道版本独立生成，互不依赖：

### 公众号版
- 基于核心稿改写，降低技术细节，增加场景描述
- 开头有场景化hook，结尾保持品牌结语
- 1500-2500字

### 开发者社区版（确定性微调）
- 核心稿本身是技术博客风格时 → **直接使用核心稿**
- 只做格式微调，末尾可补延伸阅读链接
- 不作为独立LLM任务

### 中文社交帖
- 提取1个最有冲击力的数据点或场景
- hook + 核心信息 + 话题标签，不超过300字
- 每篇核心稿产出2条不同角度

### 英文X帖
- Thread格式（3-5条）或单条
- 英文思维写作，不是中文翻译
- 技术术语保留原文，每条≤280字符

### LinkedIn摘要
- 面向管理层，偏商业价值
- 200-300字，专业正式语气

### 销售一页纸
- 仅当含客户案例或硬数据时生成
- 客户挑战→方案→效果数据→适用场景，≤400字

## Step 4: 自检 [LLM]

- [ ] 字数限制
- [ ] 关键数据点保留（不遗漏不编造）
- [ ] 产品名称规范
- [ ] 客户名称脱敏延续
- [ ] 英文版避免中式英语

## Step 5: 输出 [确定性]

```
content/adapted/
├── YYYY-MM-{主题}-wechat.md
├── YYYY-MM-{主题}-community.md
├── YYYY-MM-{主题}-social-cn-1.md
├── YYYY-MM-{主题}-social-cn-2.md
├── YYYY-MM-{主题}-x-en.md
├── YYYY-MM-{主题}-linkedin.md     （如需）
└── YYYY-MM-{主题}-one-pager.md    （如需）
```

执行摘要：
```
## 执行摘要
- 核心稿：[文件名]（[字数]字）
- 生成版本：公众号([字数]) / 社交帖中文×2 / 英文X×1 / ...
- 未生成：[渠道名]（原因）
- 数据点保留检查：[N]个全部保留 / [M]个因字数限制未包含
```

---

## 硬性规则

1. **不编造数据**。适配版本中的数据必须来自核心稿。
2. **不改变核心判断**。可以简化，不能改变结论。
3. **产品名称一致**。所有版本使用brand-rules.md中的正式名称。
4. **客户脱敏延续**。核心稿中脱敏的在所有版本中保持脱敏。
5. **不跨稿引用**。只使用当前核心稿中的信息。
6. **英文质量**。宁可简短准确，不要冗长生硬。不确定的标注[需确认]。

## 失败处理

| 场景 | 处理 |
|------|------|
| 核心稿字数<500 | 提示"内容过少，建议先补充" |
| 核心稿无数据点 | 标注"无硬数据，建议以观点角度发布" |
| 核心稿含未脱敏客户名 | 自动脱敏并告警 |
| 渠道规范文件不存在 | 使用通用格式生成 |

## ⚠️ 人工介入

- 模式A：公众号版和LinkedIn版进入L1-L2审批后发布
- 模式B：社交帖进入L1轻审后发布
- 英文版如标注[需确认]，需人工校对后发布
