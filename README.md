# 认知进化引擎 · Cognitive Evolution Engine

一个会自我进化的 AI Skill。每次对话结束后，它自动提炼你的认知模式、行为习惯和工作流程，写入结构化的双轨记忆系统。用得越多，越懂你。

> 目前以 Claude Skill 形式实现，设计上与具体 AI 工具无关，核心逻辑可移植到任何支持自定义指令或 Skill 系统的 AI 助手。

---

## 为什么要做这个

每次和 AI 对话，你都在产出有价值的认知——一个新的决策框架、一个被验证的工作流、一个改变了你思路的类比。但这些内容随着对话窗口关闭而消失，下次对话时 AI 又回到了零点。

这个工具解决的不是「AI记忆」的技术问题，而是**认知资产的系统性沉淀问题**：如何把每一次高质量对话的产出，转化为下次可以调用的结构化知识？

---

## 设计理论

这个系统的设计建立在三个理论框架之上。

### 记忆的七宗罪（Schacter, 2001）

哈佛心理学家 Daniel Schacter 在《记忆的七宗罪》中归纳了人类记忆的七种失效模式，这份「反模式清单」是本系统防护机制的来源：

| 宗罪 | 失效描述 | 本系统的对应防护 |
|------|----------|-----------------|
| **遗忘性** Transience | 记忆随时间衰减 | L1层30天无引用自动标记`[stale]`，会话记录7天归档 |
| **心不在焉** Absent-mindedness | 编码时注意力不集中，细节丢失 | Phase 2信号提取阶段强制扫描高价值信号表 |
| **阻断** Blocking | 记忆存在但无法检索 | 卡片格式要求「何时激活」字段，显式定义检索触发条件 |
| **错误归因** Misattribution | 记忆来源被张冠李戴 | 每张卡片必须标注来源：`[用户原话]`/`[Claude提炼]`/`[共���确认]` |
| **暗示性** Suggestibility | 记忆被外部信息污染 | 三步判定协议第一步：矛盾检测，新信息不能自动覆盖旧认知 |
| **偏差** Bias | 当下知识扭曲对过去的记忆 | 认知冲突只追加演化日志，原条目永不覆盖 |
| **持续性** Persistence | 不想要的记忆反复出现 | L1层衰减机制，过期内容降权而非主动删除 |

### 卡片笔记写作法（Luhmann Zettelkasten）

德国社会学家尼克拉斯·卢曼用一个卡片盒系统写出了70多本书。其核心原则是：**每张卡片必须独立且完整——单独读这张卡片，不需要任何上下文，就能完整理解它的内容。**

传统笔记的问题在于「依赖上下文」——你看到一条笔记「用户偏好零中断」，三个月后你不知道这是什么意思、为什么重要、什么时候用。卡片笔记法要求每张卡片同时回答：是什么、为什么重要、何时激活、具体例证。

本系统所有写入记忆的内容都严格遵循 Zettelkasten 格式，并明确禁止���水账式的表格记录和依赖上下文的碎片条目。

### 惊奇大脑：Friston 预测误差理论

神经科学家 Karl Friston 的预测加工框架（Predictive Processing / Free Energy Principle）认为，大脑持续对感知输入生成预测，当实际输入与预测不符时产生**预测误差**（Prediction Error）。误差越大，信息的认知价值越高。

实用转化：对话中出现「哦，原来如此」「我没想到」「这让我重新理解了X」等表述，就是高预测误差的信号——大脑遭遇了「惊奇」，正在发生认知重构。这是最值得写入 L3 认知层的时刻。

反之，大多数对话内容的预测误差接近零（重复已知信息），不值得消耗记忆系统的写入资源。Friston 框架给了系统一个「信噪比过滤器」。

---

## 系统架构

### 双轨记忆设计

人的知识和 AI 的操作记忆**必须分开存储**。混存会导致 Schacter 的「错误归因」之罪——AI 的建议被误当成你的原生想法保存下来，污染你的认知资产。

```
记忆目录/
├── asset-space/              # 人的资产空间（关于你自身的知识）
│   ├── L1-context/           # 当下情境、临时想法（30天衰减）
│   ├── L2-behavior/          # 可重复的行为模式（出现3次以上）
│   │   ├── preferences.md    # 工具/格式/风格的稳定选择
│   │   └── workflows.md      # 做某类事的固定顺序
│   ├── L3-cognitive/         # 有理论支撑的认知框架
│   │   ├── domain-knowledge.md
│   │   ├── mental-models.md
│   │   └── decision-frameworks.md
│   └── L4-core/              # 跨所有情境成立的核心原则（需3个项目验证）
│       └── values.md
│
├── ai-space/                 # AI 操作空间（关于 AI 如何为你工作）
│   ├─�� L1-memory/daily/      # 每次对话的会话日报（7天后归档）
│   ├── L2-resource/          # 稳定的外部资源和工具引用
│   ├── L3-experience/        # 成功案例与失败坑点记录
│   └── L4-capability/        # Skills + MCP 能力索引
│
└── system/
    └── reading-weights.md    # 自进化参数配置（每次对话后自动更新）
```

### 层级稳定性模型

| 层级 | 内容 | 稳定性 | 衰减规则 |
|------|------|--------|----------|
| L1 | 当前情境、临时想法 | 低 | 30天无引用标记`[stale]` |
| L2 | 可重复行为、工作流程 | 中 | 手动清理 |
| L3 | 认知框架、思维模型 | 高 | 极少变动 |
| L4 | 核心价值观、身份认同 | 永久 | 不衰减 |

---

## 三步新认知判定协议

每条提炼出的信号，在写入前必须通过三步测试。

### STEP 1 · 矛盾检测（防暗示性之罪）

> 这条新信息与已有记忆是否存在直接矛盾？

- **无矛盾** → 进入 STEP 2
- **有矛盾** → 不覆盖原条目，追加演化日志：
  ```
  [认知演化 YYYY-MM-DD] 新证据与原认知冲突
  演化前：[原有认知摘要]
  演化后：[新认知摘要]
  待后续验证。
  ```

### STEP 2 · 语义距离测试（Piaget 图式理论）

Piaget 认为新信息进入认知系统只有两种命运：同化（Assimilation）或调适（Accommodation）。

| 语义重叠度 | 操作 |
|-----------|------|
| >80% 同化 | 不新建，已有卡片追加 `[confirmed: 日期]`，确认次数 +1 |
| 40–80% 扩展 | 在已有卡片「具体例证」追加新例子 |
| <40% 新建 | 创建新卡片，状态 `[新认知·待验证]` |
| 直接矛盾 | 已在 STEP 1 处理 |

### STEP 3 · 抽象层级测试（Argyris 双环学习）

Argyris 区分了两种深度的学习：
- **单环学习**：在现有框架内修正行为（「我做错了，下次这样做」）→ 写入 **L2-behavior**
- **双环学习**：质疑并修正底层假设（「我之前想错了，原来逻辑是……」）→ 写入 **L3-cognitive**

Friston 的高预测误差信号（「哦原来如此」）是双环学习发生的识别标志。

---

## 宪法 vs 法律

系统明确区分两类规则：

**宪法（CONSTITUTION.md）——永不修改：**
- 层级语义定义（L1=情境，L2=行为，L3=认知，L4=核心，对话无法改变）
- 来源标注格式（每张卡片必须有来源，三选一）
- L4层首次写入须人工确认
- 认知冲突只追加，永不覆盖

**法律（EVOLUTION.md）——每次对话后可进化：**
- 各层读取权重（命中 +0.1，未触及 -0.05，范围 [0.5, 2.0]）
- 触发词列表（新词被使用3次后自动加入）
- 分析侧重点（行为模式/认知转变/Skill缺口/冲突检测的敏感度）

进化只能在宪法划定的边界内发生。没有宪法，进化等于系统性腐化。

---

## 自进化机制

每次对话结束后，系统自动更新 `system/reading-weights.md`：

```yaml
layer_focus:
  L1_context:       1.0   # 命中 → +0.1，未触及 → -0.05
  L2_behavior:      1.0
  L3_cognitive:     1.1   # 上次最活跃层
  L4_core:          1.0
  ai_L3_experience: 1.0

meta:
  session_count: 1
  dominant_layer: "L3_cognitive"
  last_updated: 2026-03-27
```

每第10次会话自动执行健康度检查：极化检测（某层2.0而另一层0.5时警告）、盲区检测（某层连续10次无内容时提示）、过期触发词清理（候选词30天内未达3次自动移除）。

---

## 文件结构说明

每个文件回答一个问题，具有不同的更新频率——这是拆分的核心逻辑：

| 文件 | 回答的问题 | 更新频率 |
|------|-----------|----------|
| `SKILL.md` | 如何触发？执行顺序是什么？ | 极少 |
| `CONSTITUTION.md` | 什么永远不能变？ | 从不 |
| `MEMORY-MAP.md` | 每类信息写到哪个文件？ | 目录变更时 |
| `WORKFLOW.md` | 7个Phase如何执行？ | 流程优化时 |
| `CARD-FORMAT.md` | 卡片和日报格式是什么？ | 格式变更时 |
| `EVOLUTION.md` | 系统如何根据数据调整自己？ | 每次对话后 |

---

## 安装方式

```bash
# 克隆到本地
git clone https://github.com/Tanangyuanan/cognitive-evolution-engine.git

# 软链接到 Claude Skills 目录
ln -s /path/to/cognitive-evolution-engine ~/.claude/skills/cognitive-evolution-engine
```

首次使用时告知 AI 你的记忆目录位置：
> 「我的记忆目录在 /path/to/your/memory-directory，请按这个路径写入。」

**触发词：** 进化记忆 / 提炼认知 / 更新记忆 / 复盘一下 / 同步记忆

对话自然结束时也会自动触发（「好的谢谢」「就这样」「再见」）。

---

---

# Cognitive Evolution Engine

A self-evolving AI Skill. After each conversation, it automatically distills cognitive patterns, behavioral habits, and workflows, writing them into a structured dual-track memory system. The more you use it, the better it understands you.

> Currently implemented as a Claude Skill. The design is tool-agnostic — the core logic is portable to any AI assistant that supports custom instructions or a Skill system.

---

## Why This Exists

Every high-quality AI conversation produces valuable cognition — a new decision framework, a validated workflow, an analogy that shifted your thinking. But it all disappears when the window closes. The next conversation starts at zero.

This tool doesn't solve the technical problem of "AI memory." It solves the **systematic accumulation of cognitive assets**: how do you turn the output of each quality conversation into structured knowledge you can actually call on later?

---

## Theoretical Foundations

### The Seven Sins of Memory (Schacter, 2001)

Harvard psychologist Daniel Schacter catalogued seven ways human memory fails. This failure-mode inventory is the source of the system's protective rules:

| Sin | Failure | System's Defense |
|-----|---------|-----------------|
| **Transience** | Memory decays over time | L1 entries marked `[stale]` after 30 days without reference |
| **Absent-mindedness** | Poor encoding loses detail | Phase 2 forces systematic signal extraction against a defined table |
| **Blocking** | Memory exists but can't be retrieved | Card format requires a "when to activate" field — explicit retrieval triggers |
| **Misattribution** | Source of memory is confused | Every card requires a source tag: `[user's words]` / `[Claude's synthesis]` / `[mutually confirmed]` |
| **Suggestibility** | Memory is contaminated by external input | Step 1 of the 3-step protocol: contradiction detection — new info cannot auto-overwrite existing cognition |
| **Bias** | Present knowledge distorts past memory | Cognitive conflicts append-only as evolution logs — original entries never overwritten |
| **Persistence** | Unwanted memories keep surfacing | L1 decay mechanism deprioritizes rather than deletes stale content |

### Zettelkasten (Luhmann)

German sociologist Niklas Luhmann used a slip-box system to write 70+ books. The core principle: **every card must be independently readable — you should fully understand it without any surrounding context.**

The problem with conventional notes is context-dependence. "User prefers zero interruptions" means nothing three months later. Zettelkasten requires every card to simultaneously answer: what it is, why it matters, when to activate it, and a concrete example.

All content written into this memory system follows strict Zettelkasten format. Simple one-liners and context-dependent fragments are explicitly listed as anti-patterns.

### The Surprised Brain: Friston's Predictive Error Theory

Neuroscientist Karl Friston's Predictive Processing framework proposes that the brain continuously generates predictions about incoming information. When reality doesn't match the prediction, a **prediction error** is generated. The larger the error, the higher the cognitive value of the information.

Practical translation: expressions like "oh, I see now," "I didn't expect that," "this makes me rethink X" are high prediction error signals — the brain encountered genuine surprise and is restructuring its model. This is the optimal moment to write to the L3 cognitive layer.

Most conversation content has near-zero prediction error (repeating known information) and doesn't justify a memory write. Friston's framework provides a signal-to-noise filter.

---

## System Architecture

### Dual-Track Memory Design

Human knowledge and AI operational memory **must be stored separately**. Mixing them triggers Schacter's misattribution sin — AI suggestions get stored as the user's original thoughts, contaminating the human knowledge track.

```
memory-directory/
├── asset-space/              # Human knowledge (about you)
│   ├── L1-context/           # Current context, temporary ideas (30-day decay)
│   ├── L2-behavior/          # Repeatable patterns (3+ occurrences)
│   │   ├── preferences.md    # Stable tool/format/style choices
│   │   └── workflows.md      # Fixed step sequences for recurring task types
│   ├── L3-cognitive/         # Frameworks with "why" reasoning attached
│   │   ├── domain-knowledge.md
│   │   ├── mental-models.md
│   │   └── decision-frameworks.md
│   └── L4-core/              # Cross-context principles (requires 3-project validation)
│       └── values.md
│
├── ai-space/                 # AI operational memory (how the AI works for you)
│   ├── L1-memory/daily/      # Session logs (archived after 7 days)
│   ├── L2-resource/          # Stable external references and tools
│   ├── L3-experience/        # Success patterns and failure/pitfall records
│   └── L4-capability/        # Skills + MCP tool index
│
└── system/
    └── reading-weights.md    # Self-evolution config (auto-updated each session)
```

### Layer Stability Model

| Layer | Contents | Stability | Decay Rule |
|-------|----------|-----------|------------|
| L1 | Current context, temporary ideas | Low | Marked `[stale]` after 30 days without reference |
| L2 | Repeatable behaviors, workflows | Medium | Manual cleanup |
| L3 | Cognitive frameworks, mental models | High | Rarely changes |
| L4 | Core values, identity-level principles | Permanent | Never decays |

---

## The 3-Step Cognition Judgment Protocol

Every extracted signal passes three sequential tests before being written.

### Step 1 · Contradiction Detection (against Suggestibility)

> Does this new information contradict anything already stored?

- **No conflict** → proceed to Step 2
- **Conflict found** → append evolution log to original entry, never overwrite

### Step 2 · Semantic Distance Test (Piaget Schema Theory)

Piaget argued that new information entering an existing knowledge structure has two possible fates: assimilation or accommodation.

| Overlap | Action |
|---------|--------|
| >80% Assimilation | Append `[confirmed: date]` to existing card, increment count |
| 40–80% Extension | Append new example to existing card's evidence section |
| <40% Accommodation | Create new card, status `[new cognition · pending validation]` |
| Direct contradiction | Already handled in Step 1 |

### Step 3 · Abstraction Level Test (Argyris Double-Loop Learning)

Argyris distinguished two depths of learning:
- **Single-loop**: correcting behavior within the existing framework → write to **L2-behavior**
- **Double-loop**: questioning and revising underlying assumptions → write to **L3-cognitive**

Friston's high prediction error signals ("oh I see now") are the recognition markers for double-loop learning.

---

## Constitution vs. Laws

**Constitution (CONSTITUTION.md) — never modified:**
- Layer semantic definitions (no conversation can change what L1–L4 mean)
- Source attribution format required on every card
- L4 first-time entries require explicit human confirmation
- Cognitive conflicts are append-only, never overwritten

**Laws (EVOLUTION.md) — evolves after each session:**
- Per-layer reading weights (hit → +0.1, no hit → -0.05, bounded [0.5, 2.0])
- Trigger word list (new words added after 3 confirmed uses)
- Analysis focus dimensions (behavioral / cognitive / skill-gap / conflict detection sensitivity)

Evolution can only happen within the boundaries the constitution defines. Without a constitution, evolution is systematic corruption.

---

## Self-Evolution Mechanism

After each session, the system automatically updates `system/reading-weights.md`:

```yaml
layer_focus:
  L1_context:       1.0   # hit → +0.1, no hit → -0.05
  L2_behavior:      1.0
  L3_cognitive:     1.1   # most active last session
  L4_core:          1.0
  ai_L3_experience: 1.0

meta:
  session_count: 1
  dominant_layer: "L3_cognitive"
  last_updated: 2026-03-27
```

Every 10 sessions, an automatic health check runs: polarization detection, blind spot detection, and expired trigger word cleanup.

---

## File Structure

Each file answers a single question and has a different update frequency:

| File | Question Answered | Update Frequency |
|------|------------------|-----------------|
| `SKILL.md` | How is it triggered? What's the execution sequence? | Rarely |
| `CONSTITUTION.md` | What can never change? | Never |
| `MEMORY-MAP.md` | Where does each type of information get written? | When directory changes |
| `WORKFLOW.md` | How do the 7 phases execute? | When process improves |
| `CARD-FORMAT.md` | What's the card and daily report format? | When format changes |
| `EVOLUTION.md` | How does the system adjust itself based on data? | After every session |

---

## Installation

```bash
git clone https://github.com/Tanangyuanan/cognitive-evolution-engine.git
ln -s /path/to/cognitive-evolution-engine ~/.claude/skills/cognitive-evolution-engine
```

On first use, tell your AI assistant where your memory directory lives:
> "My memory directory is at /path/to/your/memory-directory. Please write to that path."

**Trigger words:** 进化记忆 / 提炼认知 / 更新记忆 / 复盘一下 / 同步记忆

Also triggers automatically when a conversation naturally ends.

---

## Design Philosophy

**Failure-first design**: Every constitutional rule was derived by inverting one of Schacter's seven failure modes. Each constraint exists because its absence causes a specific, named failure.

**Dual-track separation**: Human cognitive assets and AI operational memory evolve independently. Mixing them causes misattribution — the most insidious of Schacter's seven sins because the user may never notice it's happening.

**Constitution before evolution**: The system's evolvability is only safe because its semantic foundations are immutable. Designing what cannot change is the prerequisite for safely designing what can.

---

## License

MIT © Tanangyuanan
