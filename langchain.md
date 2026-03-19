langChain 1.x  1.抽象统一化，收敛核心概念，简化开发;
2.统一使用 create_agent，底层由 LangGraph 驱动，一行代码即可完成复杂功能 ;
3.原生支持状态管理、循环、分支和持久化，让代理具备企业级韧性;
4.引入 content_blocks 标准属性，将文本、工具调用、图片等输出统一为类型化块，切换模型成本趋近于零;
5.由 LangGraph 的检查点（Checkpointer）机制统一处理，能保存和恢复完整的对话状态，更强大和通用
6.核心抽象进一步精简，旧版 Agent 执行器和部分模块被移至 langchain-legacy 以保持兼容

langChain 0.3
1.核心哲学	功能模块化，提供多种选择
2.Agent构建方式	存在多种代理类型（如initialize_agent），API 多样
3.底层引擎	基于 AgentExecutor
4.模型输出	不同模型返回格式各异，需要自行解析
5.记忆机制	各类专门的 Memory 类	
6.包结构	部分工具和集成在 langchain-community 中	


## rag类型
1.基础RAG 向量检索+生成
2.多模态RAG
3.HyDERAG 假设文档
4.纠错RAG 会自动纠错 不能出错的场景
5.图谱RAG 知识图谱RAG
6.混合RAG 向量+关键词
7.自适应RAG 简单问题直接答 复杂问题走agemt
8.智能体RAG
9.自反思RAG 自己判断是否需要检索还是输出答案
