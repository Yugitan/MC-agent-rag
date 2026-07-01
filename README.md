# MC-Agent-RAG: Minecraft 攻略 Agent

基于检索增强生成（RAG）的 Minecraft 中文攻略问答系统。

## 功能

输入 Minecraft 相关问题，系统自动检索攻略语料库并生成答案：

```
Q: 铁镐怎么做？需要哪些材料？
A: 3 个铁锭（横排）+ 2 根木棍（竖排），在工作台合成。

Q: 怎么去末地？
A: ① 找要塞（末影之眼追踪）→ ② 放 12 个末影之眼激活传送门 → ③ 跳入
```

## 架构

```
文档加载 → 标题切分 → 向量化 → ChromaDB 检索 → 重排序 → LLM 生成
```

| 组件 | 选型 |
|------|------|
| 向量模型 | `shibing624/text2vec-base-chinese` (768 维) |
| 向量数据库 | ChromaDB（内存模式） |
| 重排序 | `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` |
| LLM | DeepSeek API (`deepseek-v4-flash`, temperature=0.1) |
| 切分策略 | 按 Markdown 标题切分，合并短章节，二次切分长段落 |

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

打开 `main.ipynb`，依次执行所有单元格即可。Cell 7 会自动运行 7 个测试问题。

## 项目结构

```
├── main.ipynb          # RAG 流水线（加载→向量化→检索→重排序→生成）
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

| 类型 | 问题 | 结果 |
|------|------|------|
| 合成 | 铁镐怎么做？ | ✅ |
| 合成 | 怎么合成附魔台？ | ✅ |
| 流程 | 怎么去末地？ | ✅ |
| 流程 | 下界合金装备怎么做？ | ✅ |
| 机制 | 附魔台最高多少级？ | ✅ |
| 机制 | 红石中继器有什么作用？ | ✅ |
| 排查 | 地狱门没生成怎么办？ | ✅ |
