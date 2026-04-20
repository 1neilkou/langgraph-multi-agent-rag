1.读了原框架，知道state，node，edge

第一步：节点：generate_query：用户原始问题进来，LLM用query_system_promt改写成更适合检索的query
第二步：retrieve节点拿改写后的query去向量数据库里面搜，返回top-k个相关文档，存进state的docs字段
第三步：respond节点把检索到的docs+对话历史messages 一起喂给LLM，用response_system_prompt生成最终答案，返回给用户。

一、state
1.项目两张图有各自独立的state
indexstate：
负责文档写入流程的数据传递
包含：
猜测字段：docs、user_id
retrievalstate：
负责问答流程的数据传递
包含：
合并检索到的文档片段reduce_docs
管理检索对象\reduce_retriever
累积对话历史\reduce_messages
合并检索结果\reduce_retriever_doc
2.conversationstate
管理多轮对话上下文

二、node
retrieval_graph里的节点：
generate_query
retrieve
respond
indexer里面的节点：
index_docs

三、edge边
__start__ → generate_query → retrieve → respond → __end__
__start__ → index_docs → __end__

目前是直连，没有分支，可以优化的地方：加上retrieval_judge
优化地方2：在respond后面加验证答案，和引用来源，和记录轨迹logging。和fastAPI封装

四、prompt有两个
query_system_prompt给generate query节点用，把用户问题改写成检索友好的query
response_system_prompt给respond query节点用，基于检索到的文档生成回答

4.20
# LangGraph Multi-Agent RAG System

基于 LangGraph 的多智能体复杂问答与检索优化系统（开发中）

## 当前进度
- [x] Template 复制
- [ ] 本地向量库（FAISS）替换
- [ ] Query Rewrite
- [ ] 多Agent架构

第一步：配置环境env
OPENAI_API_KEY=你的key
LANGSMITH_PROJECT=langgraph-multi-agent-rag

第二步：
让claude code协助学习修改
先初始化git仓库，提交第一次仓库：init: copy langgraph template as base
feat: prepare project for faiss migration
feat: add faiss retriever support
