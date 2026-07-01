# MC-Agent-RAG: Minecraft 攻略 Agent

基于 RAG + Agent（多步推理）的 Minecraft 中文攻略问答系统。

## 功能

输入 Minecraft 相关问题，系统自动路由问题复杂度并检索攻略语料库生成答案：

```
Q: 铁镐怎么做？需要哪些材料？
A: 3 个铁锭（横排）+ 2 根木棍（竖排），在工作台合成。

Q: 怎么去末地？
A: ① 找要塞（末影之眼追踪）→ ② 放 12 个末影之眼激活传送门 → ③ 跳入
```

## 架构

```
                  ┌─────────────────────┐
                  │    用户问题           │
                  └──────────┬──────────┘
                             ▼
                  ┌─────────────────────┐
                  │  Router (LLM 判断)   │
                  └──────────┬──────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                              ▼
     ┌────────────────┐          ┌──────────────────────┐
     │  简单问题        │          │  复杂问题              │
     │  单轮 RAG       │          │  Planner 拆解子问题    │
     │  检索→重排序→生成 │          │  ↓ 多路检索→合并→生成 │
     └────────┬───────┘          └──────────┬───────────┘
              └──────────┬──────────────────┘
                         ▼
              ┌─────────────────────┐
              │  DeepSeek 生成答案    │
              └─────────────────────┘
```

| 组件 | 选型 |
|------|------|
| Router | DeepSeek (`deepseek-v4-flash`, temperature=0) 判断 simple / complex |
| 向量模型 | `shibing624/text2vec-base-chinese` (768 维) |
| 向量数据库 | ChromaDB（内存模式） |
| 重排序 | `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` |
| LLM | DeepSeek API (`deepseek-v4-flash`, temperature=0.1) |
| 切分策略 | 按 Markdown 标题切分，合并短章节，二次切分长段落 |
| Agent 模式 | Plan-then-Execute：Router → Planner → Executor → Synthesizer |

## 快速开始

### 1. 环境准备

确保已安装 [uv](https://docs.astral.sh/uv/getting-started/installation/) 和 [Jupyter](https://jupyter.org/install)。

在项目根目录安装依赖：

```bash
uv sync
```

### 2. 配置 API Key

在项目根目录创建 `.env` 文件：

```
DEEPSEEK_API_KEY=你的DeepSeek API Key
```

### 3. 运行

```bash
uv run --with jupyter jupyter lab
```

打开 `main.ipynb`，依次执行所有单元格即可。Cell 8 会自动运行全部 10 个测试问题（7 简单 + 3 复杂）。

## 项目结构

```
├── main.ipynb          # RAG 流水线 + Agent 多步推理
│   ├── Cell 1-6        # 文档加载 → 向量化 → ChromaDB → 检索 → 重排序 → 生成
│   ├── Cell 7          # Agent 核心（Router / Planner / Executor / Synthesizer）
│   └── Cell 8          # 完整测试套件（7 简单 + 3 复杂）
├── docs/               # 攻略语料库（29 个 .md 文件）
│   ├── crafting.md     # 合成系统
│   ├── smelting.md     # 熔炼系统
│   ├── mining.md       # 采矿
│   ├── nether.md       # 下界
│   ├── the_end.md      # 末地
│   ├── enchanting.md   # 附魔
│   ├── redstone.md     # 红石
│   └── ...             # 共 29 篇攻略
├── .env                # API Key（不提交到 Git）
├── pyproject.toml      # 依赖配置
└── CONTEXT.md          # 项目术语表
```

## 快速添加新攻略

1. 在 `docs/` 下新建 `.md` 文件，用 `##` / `###` 标题组织内容
2. 重新执行 `main.ipynb`，系统自动完成切分和索引

## 测试覆盖

### 简单问题（单轮 RAG）

| 类型 | 问题 | 路由 |
|------|------|------|
| 合成 | 铁镐怎么做？需要哪些材料？ | simple ✅ |
| 合成 | 怎么合成附魔台？ | simple ✅ |
| 合成 | 下界合金锭的合成配方是什么？ | simple ✅ |
| 流程 | 末地传送门怎么激活？ | simple ✅ |
| 机制 | 附魔台最高能附到多少级？怎么提高等级上限？ | simple ✅ |
| 机制 | 红石中继器有什么作用？ | simple ✅ |
| 排查 | 为什么我的地狱门没生成？怎么正确搭建？ | simple ✅ |

### 复杂问题（Agent 多步推理）

| 类型 | 问题 | 路由 |
|------|------|------|
| 跨主题流程 | 从零开始做一套下界合金装备，需要经过哪些步骤？ | complex ✅ |
| 红石机械 | 怎么建立一个自动熔炉系统？ | complex ✅ |
| 跨主题备战 | 如何准备去打末影龙？从装备到附魔到战斗策略 | complex ✅ |
