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

### 形态
- `main.ipynb` — 主 RAG 流水线（加载文档 → 向量化 → ChromaDB → 检索 → 重排序 → 生成）
- `docs/*.md` — 29 个手动编写的攻略文档

### 验证测试集
7 个测试问题（合成×2、流程×2、机制×2、排查×1），在 Cell 7 中一次性输出所有结果。
