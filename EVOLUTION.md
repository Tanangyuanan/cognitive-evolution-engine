# EVOLUTION.md — 进化：系统如何自我更新

> **更新频率**：每次对话后自动更新（高频变动）
> **回答的问题**：系统如何根据使用数据调整自己的读取重点？每条进化规则精确到什么？

---

## 进化原则

进化只改「法律」，不改「宪法」。
- **宪法（CONSTITUTION.md）**：9条规则 + 层级语义 + 晋升路径 → **永不修改**
- **法律（此文件描述的机制）**：读取权重 + 触发词优先级 + 分析侧重点 → **每次对话后可调整**

---

## 读取权重进化规则

### 权重定义

每层有一个基础权重（`weight`）和一个读取优先级（`reading_priority`）：

| weight 值 | 含义 |
|-----------|------|
| 2.0 | 最高优先级，完整读取，每条卡片都精细比对 |
| 1.5 | 高优先级，完整读取 |
| 1.0 | 标准读取（默认值） |
| 0.7 | 低优先级，只读最近3个月的条目 |
| 0.5 | 最低优先级，只读标题和状态字段 |

`reading_priority = weight + exploration_bonus`（见步骤3），用于本次实际读取排序。

**范围约束**：`weight` 限定在 [0.5, 2.0] 之间。`reading_priority` 可超出上限（探索项会暂时提升超低权重层）。

---

### 权重更新算法（EMA + UCB 混合，v2）

**理论基础**
- **EMA（指数移动平均）**：防止单次对话剧烈扭曲权重，引入历史惯性（Ebbinghaus, 1885）
- **UCB（上置信界）**：防止注意力盲区——长期未命中的层获得探索奖励，强制定期审视（Auer et al., 2002）
- **质量加权**：区分「新认知」「扩展」「确认」三种命中类型，信息增益越高贡献越大

**触发时机**：Phase 6，每次对话结束后执行一次，且仅执行一次。

---

**步骤1：计算质量加权信号**

回顾本次 Phase 3 的判定结果，按卡片类型统计：

| 命中类型 | 来源（Phase 3 STEP 2 判定） | 信号值 |
|---------|--------------------------|--------|
| `new_card` | <40% 语义重叠，新建卡片 | × 0.20 |
| `extended_card` | 40-80% 语义重叠，追加例证 | × 0.10 |
| `confirmed_card` | >80% 语义重叠，仅更新确认次数 | × 0.03 |

```
quality_signal[层] = new_cards × 0.20 + extended_cards × 0.10 + confirmed_cards × 0.03
```

---

**步骤2：EMA 更新基础权重（α = 0.3）**

```
如果 quality_signal[层] > 0（本次有命中）：
  target = min(2.0, 当前 weight + quality_signal[层])
  total_hits[层] += 1

如果 quality_signal[层] == 0（本次无命中）：
  target = max(0.5, 当前 weight - 0.05)

新 weight = round(0.7 × 当前 weight + 0.3 × target, 2)
新 weight = max(0.5, min(2.0, 新 weight))  # 强制范围
```

α = 0.3 的含义：本次活动占 30%，历史惯性占 70%。单次爆发不会立即主导权重，持续活跃才能稳步提升。

---

**步骤3：计算 UCB 探索奖励（不写入 weight，仅影响本次读取排序）**

```
exploration_bonus[层] = 0.15 × √(ln(session_count) / max(1, total_hits[层]))

reading_priority[层] = 新 weight + exploration_bonus[层]
```

UCB 的含义：`total_hits` 越低（长期被忽视的层），探索奖励越高，强制系统在下次读取时优先扫描。防止注意力因"马太效应"固化到少数活跃层。

**迁移说明**：`total_hits` 字段首次运行时不存在，初始化为 1，避免除零。

---

**步骤4：写入 reading-weights.md**

```yaml
# system/reading-weights.md
# 最后更新：YYYY-MM-DD（第N次会话）

layer_focus:
  L1_context:
    weight: 1.0                    # 基础权重（EMA更新）
    total_hits: 3                  # 累计命中会话数（UCB分母）
    reading_priority: 1.12         # weight + exploration_bonus（本次读取排序用）
    last_change: "EMA: 0.7×1.0 + 0.3×1.1，信号强度=0.10（1张extended）"
  L2_behavior:
    weight: 1.2
    total_hits: 5
    reading_priority: 1.29
    last_change: "EMA: 0.7×1.1 + 0.3×1.3，信号强度=0.20（1张new_card）"
  L3_cognitive:
    weight: 0.8
    total_hits: 1
    reading_priority: 1.07         # 低weight但高UCB，会被优先读取
    last_change: "EMA: 0.7×0.85 + 0.3×0.75，无命中→衰减"
  L4_core:
    weight: 1.0
    total_hits: 2
    reading_priority: 1.11
    last_change: "无命中，EMA衰减"
  ai_L1_memory:
    weight: 1.3
    total_hits: 8
    reading_priority: 1.34
    last_change: "EMA: 日报条目命中，信号强度=0.10"
  ai_L3_experience:
    weight: 0.9
    total_hits: 2
    reading_priority: 1.01
    last_change: "无命中，EMA衰减"

meta:
  session_count: 12
  dominant_layer: "ai_L1_memory"   # reading_priority 最高的层
  last_updated: YYYY-MM-DD
```

---

## 触发词自进化规则

### 触发词来源

触发词分两类：
- **固定触发词**（宪法，不可改）：进化记忆、提炼认知、更新记忆、复盘一下、同步记忆
- **学习触发词**（可进化）：从对话中自动识别

### 学习触发词的识别规则

当以下模式出现时，将该短语加入学习触发词列表：
1. 用户使用某个短语后，立即手动调用了认知进化引擎
2. 同一短语在3次不同对话中都出现在记忆提炼的语境里

**格式**：学习触发词存入 `system/reading-weights.md` 的 `learned_triggers` 字段：
```yaml
learned_triggers:
  - phrase: "按上次那样"
    added: "YYYY-MM-DD"
    confirmed_count: 3
  - phrase: "把今天的沉淀下来"
    added: "YYYY-MM-DD"
    confirmed_count: 1  # <3 次时为候选，不生效
```

**生效条件**：`confirmed_count >= 3` 才作为触发词生效。

---

## 分析侧重点进化规则

系统会根据累积的对话数据，调整在 Phase 2（信号提取）中优先关注什么。

### 侧重点权重

存储在 `system/reading-weights.md` 的 `analysis_focus` 字段：
```yaml
analysis_focus:
  behavioral_patterns: 1.0    # 行为模式识别的敏感度
  cognitive_shifts: 1.0       # 认知转变识别的敏感度
  skill_gap_detection: 1.0    # Skill缺口识别的敏感度
  conflict_detection: 1.0     # 认知冲突识别的敏感度
```

### 调整规则

- 某类信号在最近5次对话中每次都有命中 → 该类敏感度 +0.1
- 某类信号在最近5次对话中从未命中 → 该类敏感度 -0.05
- 范围约束：[0.5, 1.5]

---

## 进化边界（不可突破）

以下内容即使经过多次对话数据的「暗示」，也绝对不会被进化机制修改：

| 受保护内容 | 存放位置 | 原因 |
|------------|----------|------|
| 层级语义定义 | CONSTITUTION.md | 修改后所有卡片的分类依据崩塌 |
| 9条宪法规则 | CONSTITUTION.md | 系统一致性的最后防线 |
| 三步判定协议的结构 | WORKFLOW.md Phase 3 | 协议本身被修改会导致判定结果不可信 |
| 来源标注格式 | MEMORY-MAP.md | 格式改变会导致旧卡片无法被正确解析 |
| Zettelkasten卡片格式的必填字段 | CARD-FORMAT.md | 字段缺失会破坏卡片的独立可读性 |

**违反后果**：如果检测到进化机制试图修改以上内容（即Phase 6中产生了修改CONSTITUTION.md的写入操作），立即中止，输出警告：`[宪法保护] 进化机制尝试修改受保护内容，已阻止。请人工审查。`

---

## 进化健康度检查

每第10次会话，自动执行一次健康度检查：

**检查项1：权重极化检测**
```
如果任意层 weight 达到2.0 且 另一层 total_hits 仍然 <= 2：
  输出警告："[健康度警告] 读取权重出现极化，且某层几乎未被命中（total_hits≤2）。
  UCB探索项可能已不足以弥补，建议人工检查是否存在信号识别盲区。"
  不自动修正，交由人工判断。

注：UCB机制会自动为低total_hits层提高reading_priority，轻度极化会自我修正。
只有total_hits极低时才需告警。
```

**检查项2：长期未触及层检测**
```
如果某层在最近10次会话中hit_count始终为0：
  在日报末尾追加提示："[盲区提示] L?-? 层已连续10次会话无新内容，当前权重：0.?。这可能是信息自然稀疏，也可能是信号识别遗漏。"
```

**检查项3：触发词候选清理**
```
如果某个学习触发词候选在加入后的30天内 confirmed_count 仍然 <3：
  自动从列表中移除，不产生通知。
```
