基于 LangGraph 的多智能体复杂问答与检索优化系统，
当前从 retrieval-agent-template 演进，计划完成本地向量库、问题路由、Query Rewrite、证据校验与重试机制。

## Status
- [x] 从 template 初始化项目
- [ ] 替换为 FAISS 本地向量库
- [ ] 增加 question router
- [ ] 增加 query rewrite
- [ ] 增加 evidence checker
- [ ] 增加 retry/fallback

## Tech Stack
- LangGraph
- LangChain
- Python
- OpenAI API
- FAISS（计划替换）

## Notes
This project is initialized from a LangGraph retrieval template and is being extended into a multi-agent complex QA and retrieval optimization system.