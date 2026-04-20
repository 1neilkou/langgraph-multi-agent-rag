# LangGraph Multi-Agent RAG

基于 LangGraph 的多智能体复杂问答与检索优化系统（开发中）。

该项目从官方 retrieval-agent-template 演进，目标是构建一个具备问题理解、查询重写、检索优化、证据校验与多轮推理能力的 Agent 系统。

---

## 🚀 Current Progress

- [x] 项目初始化（基于 LangGraph retrieval template）
- [x] Git 仓库搭建 & DEV_LOG 建立
- [ ] 替换检索后端为 FAISS（本地向量库）
- [ ] 增加 Question Router（问题分类）
- [ ] 增加 Query Rewrite（复杂问题拆分）
- [ ] 检索优化（基于 rewrite query）
- [ ] 增加 Evidence Checker（证据校验）
- [ ] 增加 Retry / Fallback 机制
- [ ] 多智能体架构重构（Planner / Retriever / Critic）

---

## 🧠 Project Overview

当前系统基于 LangGraph 构建流程化 Agent：

User Query  
→ Retrieval  
→ Answer Generation  

后续将逐步演进为：

User Query  
→ Planner Agent（问题理解 / 路由）  
→ Query Rewrite  
→ Retriever Agent  
→ Synthesizer Agent  
→ Critic Agent（校验 & 重试）

---

## 🧩 Tech Stack

- LangGraph
- LangChain
- Python
- OpenAI API（当前）
- FAISS（计划替换本地向量库）

---

## 📂 Project Structure

```bash
src/
  retrieval_graph/
    graph.py          # Agent 流程定义
    state.py          # 状态管理
    retrieval.py      # 检索逻辑（待改造）
    configuration.py  # 配置
    prompts.py        # Prompt 定义