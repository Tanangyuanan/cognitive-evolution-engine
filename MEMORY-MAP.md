# MEMORY-MAP.md — 地图：每类信息写到哪里

> **更新频率**：目录结构变更时才改（高稳定性）
> **回答的问题**：提炼出一条信息后，它应该写入哪个文件？

---

## 双轨目录全景

```
记忆测试202603/
├── asset-space/                    # 人的资产空间（关于用户自身的知识）
│   ├── L1-context/                 # 当下的情境、项目、想法（时效性强，30天衰减）
│   │   ├── my-projects.md          # 正在进行的项目条目
│   │   └── ideas.md                # 未分类的单次想法
│   ├── L2-behavior/                # 可重复的行为和模式（出现3次以上）
│   │   ├── preferences.md          # 工具/格式/风格的稳定选择
│   │   └── workflows.md            # 做某类事的固定顺序（3次→Skill候选）
│   ├── L3-cognitive/               # 有理论支撑的认知框架（附带"为什么"）
│   │   ├── domain-knowledge.md     # 某领域的系统性认知地图
│   │   ├── mental-models.md        # 用什么类比/结构拆解问题
│   │   └── decision-frameworks.md  # 判断和选择时用的框架
│   └── L4-core/                    # 跨所有情境成立的核心（须跨3个不同项目验证）
│       └── values.md               # 价值观、身份认同、最高优先级原则
│
├── ai-space/                       # AI的操作空间（关于AI如何工作的知识）
│   ├── L1-memory/                  # 会话记录（7天后归档）
│   │   └── daily/
│   │       └── YYYY-MM-DD.md       # 每次对话的会话日报
│   ├── L2-resource/                # AI可调用的稳定资源
│   │   └── references.md           # 外部链接、工具、API、文档
│   ├── L3-experience/              # AI执行任务后的经验沉淀
│   │   ├── success-cases.md        # 成功案例（含可复用的操作模式）
│   │   └── failure-cases.md        # 失败/坑点记录（含规避规则）
│   └── L4-capability/              # AI的核心能力清单
│       └── skills-index.md         # 已创建的Skills + MCP工具索引
│
└── system/                         # 系统配置（由进化机制自动更新）
    └── reading-weights.md          # 各层读取权重 + 自进化参数
```

---

## 信号 → 写入位置 速查表

| 信号类型 | 来源特征 | 写入位置 |
|----------|----------|----------|
| 用户说"我这次在做X项目" | 当前项目/情境 | `asset-space/L1-context/my-projects.md` |
| 用户的单次想法/临时偏好 | 首次出现，时效性强 | `asset-space/L1-context/ideas.md` |
| 用户的工具/格式稳定选择 | 出现2次以上 | `asset-space/L2-behavior/preferences.md` |
| 用户做某类事的固定顺序 | 可重复的流程 | `asset-space/L2-behavior/workflows.md` |
| 某领域的系统性知识 | 附带理论框架/解释 | `asset-space/L3-cognitive/domain-knowledge.md` |
| 用来拆解问题的思维模型 | 类比/结构/视角 | `asset-space/L3-cognitive/mental-models.md` |
| 判断选择时用的决策框架 | 含判断标准/权衡 | `asset-space/L3-cognitive/decision-frameworks.md` |
| 跨多个项目依然成立的原则 | 3个不同情境验证 | `asset-space/L4-core/values.md` |
| 本次对话摘要 | 每次对话结束 | `ai-space/L1-memory/daily/YYYY-MM-DD.md` |
| AI发现可复用的工具/资源 | 外部链接/API | `ai-space/L2-resource/references.md` |
| AI成功完成某类任务的模式 | 可复用操作序列 | `ai-space/L3-experience/success-cases.md` |
| AI执行中遇到的坑点 | 失败/异常/规避规则 | `ai-space/L3-experience/failure-cases.md` |
| Skill/MCP的新增或更新 | 能力变化 | `ai-space/L4-capability/skills-index.md` |

---

## 归类歧义处理规则

**歧义1：不确定是 L2 还是 L3**
- 有"为什么这样做"的解释 → L3
- 只有"我这样做"的描述 → L2

**歧义2：不确定是 asset-space 还是 ai-space**
- 关于"用户是什么/用户的知识" → asset-space
- 关于"AI执行了什么/AI的经验" → ai-space

**歧义3：不确定是 domain-knowledge 还是 mental-models**
- 某领域的专业知识地图 → domain-knowledge.md
- 跨领域通用的思维工具 → mental-models.md

**歧义4：不确定是 mental-models 还是 decision-frameworks**
- 如何看待/理解问题的视角 → mental-models.md
- 如何做出判断/选择的规则 → decision-frameworks.md

---

## 来源标注规范（宪法条款，不可省略）

每条卡片的元数据必须包含来源标注，三选一：
- `[用户原话]` — 用户明确陈述的内容（含截图中的文字）
- `[Claude提炼]` — Claude从对话模式中归纳，用户未明确说出
- `[共同确认]` — 双方在对话中明确同意的结论

**违反后果**：无来源标注的条目在下次读取时标记 `[来源缺失·待核查]`，不参与权重计算。
