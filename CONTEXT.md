# Minecraft 攻略 Agent RAG

## 项目目标

基于 RAG 流水线架构（分块 → 向量化 → ChromaDB 存储 → 检索 → 重排序 → LLM 生成），构建一个面向 Minecraft 中文攻略的问答 Agent。

## 术语表

### Minecraft 攻略知识
Minecraft 游戏的中文攻略信息，包括生存合成、机制说明、生物/生态群系等。

### 语料库
手动编写的 markdown 攻略文件集合，位于 `docs/` 目录。一页一个 `.md` 文件，按 `##` / `###` 章节标题切分 chunk。过长章节再按 `\n\n` 二次切分，短章节（<100 字）与下一章节合并。chunk 中保留标题。

### RAG 流水线
- **Chunker**: 按 Markdown heading 切分，保留标题，合并短章节，二次切分过长章节
- **Embedder**: `shibing624/text2vec-base-chinese` → 768 维向量
- **Vector Store**: ChromaDB（EphemeralClient，内存模式）
- **Retriever**: 余弦相似度 top-k=10 检索
- **Re-ranker**: `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`，取 top-k=5
- **LLM**: DeepSeek API（OpenAI 兼容），`deepseek-v4-flash`，temperature=0.1

### 覆盖范围
基础生存合成 + 进阶机制（红石、附魔、下界/末地、下界合金等），共 29 个 `.md` 文件（约 45,000 字）。

### Agent（多步推理层）
在现有单轮 RAG 之上，新增 Plan-then-Execute Agent 层处理复杂问题。

**Router**: LLM 判断问题复杂度。简单问题→直接走单轮 RAG；复杂问题→触发 Planner。
**Planner**: LLM 将复杂问题拆解为多个独立子问题。
**Executor**: 对每个子问题独立执行 `retrieve()` + `rerank()`，合并所有结果。
**Synthesizer**: DeepSeek 基于合并后的多路 chunks，一次性生成综合答案。

简单问题：`用户 → Router → RAG → 答案`
复杂问题：`用户 → Router → Planner → Executor(多路检索) → Synthesizer → 答案`

### 形态
- `main.ipynb` — 主 RAG + Agent 流水线（Cell 1-6 为现有 RAG，新增 Cell 7-10 为 Agent）
- `docs/*.md` — 29 个手动编写的攻略文档

### 验证测试集
- **简单问题**（单轮 RAG 覆盖）7 个问题（合成×2、流程×2、机制×2、排查×1）
- **复杂问题**（Agent 多步覆盖）新增 3+ 个需要多步推理的跨主题问题
