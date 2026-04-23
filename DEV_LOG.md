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

# 4.20学习记录 
## LangGraph Multi-Agent RAG System

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

先初始化git仓库，提交第一次仓库：init: copy langgraph template as base
feat: prepare project for faiss migration
feat: add faiss retriever support
学习git：本地版本管理，github是远程代码托管
git init把当前文件夹变成Git仓库
git add把改动放进准备提交区
git commit生成一个版本快照
git state看现在改了什么
git log 看历史提交

第三步：
让claude code协助学习修改
学习：
1.graph.py 的执行流程
这是一个 pipeline 式的 RAG 图，没有循环也没有条件分支，属于最简单的 DAG 结构。generate_query 是唯一有业务判断的节点，用轮次来决定是否需要 LLM 介入。
2.
InputState 是对外的窄接口，State 是内部完整状态。这种设计的好处是：外部调用者只需要传 messages，不需要关心内部中间变量，符合信息隐藏原则。
"这里用了两层配置类的继承。IndexConfiguration 管检索相关配置，Configuration 继承它并追加 LLM 相关配置。from_runnable_config 用 dataclasses.fields() 做反射，只提取当前类声明的字段，避免传入多余参数报错

# 4.23学习记录

## 第一步：cluade辅助理解：
1.graph.py线性管道，三个节点，无分支
messages
generate_query 生成query 唯一有业务判断的节点，用伦茨决定LLM是否介入
retrieve
respond
messages 追加AI回复

2.state.py的状态字段含义
IndexState   ← 只用于写入流程
InputState   ← 对外接口（只暴露 messages）
State        ← 继承 InputState，内部完整状态（加了 queries + retrieved_docs）

3.configuration.py
IndexConfiguration（基础：user_id、embedding_model、retriever_provider、search_kwargs）
    └── Configuration（继承，追加：response_model、query_model、两个 system prompt）

4.retrieal.py
with retrieval.make_retriever(config) as retriever:
    response = await retriever.ainvoke(...)

为什么用 contextmanager？ 向量库连接是外部资源，如果不显式关闭会泄漏连接。contextmanager 保证"用完必还"，不管中间是否出错。

总结
graph.py        ← 编排层：定义"做什么"和"按什么顺序做"
state.py        ← 数据层：定义节点之间传什么、怎么合并
configuration.py← 配置层：运行时参数，决定用哪些模型和后端
retrieval.py    ← 基础设施层：连接外部向量数据库，屏蔽差异
prompts.py      ← 内容层：LLM 的指令模板
utils.py        ← 工具层：通用辅助函数，无业务逻辑

依赖方向
graph.py → state.py
graph.py → configuration.py
graph.py → retrieval.py
graph.py → utils.py
retrieval.py → configuration.py
configuration.py → prompts.py

## 第二步 加FAISS：

改进具体做法：
1.修改configuration.py
2.修改retrieval.py ，新增 make_faiss_retriever
3.在 make_retriever 的 match 里加 case "faiss"
4.pyproject.toml 加依赖
5.创建 docs/ 目录和测试文档
### 1.
改动前
Literal["elastic", "elastic-local", "pinecone", "mongodb"]
default="elastic"

改动后
Literal["elastic", "elastic-local", "pinecone", "mongodb", "faiss"]
default="faiss"

### 2.
make_elastic_retriever()  ← 原有
make_pinecone_retriever() ← 原有
make_mongodb_retriever()  ← 原有
make_faiss_retriever()    ← 新增
make_retriever()          ← 原有，加了一行 case "faiss"

### 3.
"langchain-community>=0.2.0",   ← FAISS 和 TextLoader 的依赖包
"faiss-cpu>=1.7.4",             ← Facebook FAISS 本体

### 4.
原有后端的依赖链：
你的代码 → LangChain → Elasticsearch SDK → Elasticsearch Server → 云账号 → API Key

FAISS 的依赖链：
你的代码 → LangChain → faiss-cpu → 内存
                     → OpenAI SDK → OpenAI API


## 第三步：trace
langSmith，接入api
1.检查langSmith接入
2.启动开发服务器
uv run langgraph dev
相当于启动了一个本地 API 服务器（跑在 localhost:2024）
3.触发一次调用
curl -s -X POST http://localhost:2024/runs/stream \
  -H "Content-Type: application/json" \
  -d '{
    "assistant_id": "retrieval_graph",
    "input": {
      "messages": [{"role": "user", "content": "What is LangGraph?"}]
    },
    "config": {
      "configurable": {"user_id": "local"}
    }
  }' | head -20

  4.观察结果
