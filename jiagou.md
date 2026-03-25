``````mermaid

graph TD
    %% --- 外部输入与接入层 ---
    User["用户/业务系统"] -->|"提问 (TraceID initiated)"| API_Gateway["API 网关 / 接入层"]

    subgraph "运行态 (Runtime): LangGraph 有状态流转"
        direction TB
        
        %% --- 状态与记忆 ---
        StateStore[("Postgres/Redis Checkpoint & Memory")] <==> Graph_Engine
        
        %% --- 核心逻辑节点 ---
        Graph_Engine(["LangGraph 引擎核心"]) --> Init_State["节点 1: 初始化状态与加载记忆"]
        Init_State --> LLM_Router{节点 2: LLM 自主路由器}
        
        %% --- 查询扩展分支 ---
        LLM_Router -->|"触发 Step-Back"| StepBack_Node["节点 3: 抽象原则生成"]
        LLM_Router -->|"触发 HyDE"| HyDE_Node["节点 4: 假设性文档生成"]
        LLM_Router -->|"直接检索"| Pre_Retriever["合并查询/上下文"]
        
        StepBack_Node --> Pre_Retriever
        HyDE_Node --> Pre_Retriever
        
        %% --- 检索与循环 ---
        Pre_Retriever --> Retriever_Node["节点 5: 向量检索器"]
        Retriever_Node --> Grader_Node{节点 6: LLM 检索内容评分}
        
        %% --- 闭环控制与重写 ---
        Grader_Node -->|"不相关 & 未达循环上限"| Rewriter_Node["节点 7: 查询重写器"]
        Rewriter_Node -->|"更新 current_query"| Retriever_Node
        
        %% --- 生成与终结 ---
        Grader_Node -->|"相关 / 达到循环上限"| Generator_Node["节点 8: 答案生成"]
        Generator_Node --> End_State["节点 9: 结束状态保存"]
        
        %% --- 降级处理逻辑 ---
        Degradation_Mgr["降级策略管理器"] -.->|"监控 & 触发"| Graph_Engine
    end

    %% --- 数据与模型层 ---
    subgraph "Infrastructure: 数据与模型"
        direction LR
        VectorDB[("向量数据库 Milvus/Pinecone")]
        Primary_LLM["GPT-4/Claude3 (高性能 LLM)"]
        Backup_LLM["Llama3-70B/Local (轻量 LLM)"]
        Embed_Model["Embedding Model"]
    end
    
    Retriever_Node ==> VectorDB
    LLM_Router ==> Primary_LLM
    Generator_Node ==> Primary_LLM
    
    %% --- 观测态 ---
    subgraph "观测态: Semantic Monitor & Tracing"
        direction TB
        OTel_Collector["OpenTelemetry Collector"]
        
        Graph_Engine -.->|"Span Data"| OTel_Collector
        VectorDB -.->|"Span Data"| OTel_Collector
        Primary_LLM -.->|"Span Data"| OTel_Collector

        OTel_Collector --> Trace_DB[("Trace Storage Jaeger/Tempo")]
        OTel_Collector --> Semantic_Analyzer["语义监控分析器"]
        
        Semantic_Analyzer -->|"实时输入异常检测"| Alert_Sys["告警系统"]
        Semantic_Analyzer -->|"语义漂移/毒性检测"| Dashboard["监控大盘"]
    end

    %% --- 评估态 ---
    subgraph "评估态: Ragas 自动化评估"
        direction TB
        Production_Logs[("生产环境日志/Traces")] --> Ragas_Eval_Engine["Ragas 评估引擎"]
        Ragas_Eval_Engine --> Ragas_Metrics{Ragas 指标}
        Ragas_Metrics -->|"Faithfulness"| Eval_Report["评估报告与回归分析"]
        Ragas_Metrics -->|"Answer Relevance"| Eval_Report
        Ragas_Metrics -->|"Context Precision"| Eval_Report
    end

    %% --- 开发/测试态 ---
    subgraph "DevTest: 金融级自动化测试 CI/CD"
        TestCase_DB[("1200+ 金融合同测试用例库")] --> Test_Runner["自动化测试运行器"]
        
        subgraph "Test_Types"
            Test_Runner --> Unit_Tests["1. 单元测试"]
            Test_Runner --> Integration_Tests["2. 集成测试"]
            Test_Runner --> Adversarial_Tests["3. 对抗性测试"]
        end
        
        Integration_Tests ==>|"模拟请求"| API_Gateway
        Test_Runner -.->|"输出数据"| Ragas_Eval_Engine
    end

    End_State -->|"Response"| API_Gateway
    API_Gateway -->|"最终回答 + TraceID"| User
    
    Degradation_Mgr -.-> Backup_LLM
    Degradation_Mgr -.-> Static_Response["返回降级静态提示"]
```
