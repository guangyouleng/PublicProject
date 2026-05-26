# RAG问答系统项目架构文档

## 项目结构实现顺序

### 0. 基础配置层 (最先实现)
```
base/
├── config.py           # 0.1 系统配置管理
├── logger.py           # 0.2 日志系统配置
└── __init__.py         # 0.3 包初始化文件
```

### 1. 数据库连接层 (第二实现)
```
mysql_qa/
├── db/
│   └── MySQLClient.py  # 1.1 MySQL数据库连接客户端
├── cache/
│   └── RedisClient.py  # 1.2 Redis缓存连接客户端
└── retrieval/
    └── bm25_search.py  # 1.3 BM25检索算法实现
```

### 2. 文档处理层 (第三实现)
```
rag_qa/
├── edu_document_loaders/
│   ├── edu_docloader.py    # 2.1 通用文档加载器
│   ├── edu_imgloader.py    # 2.2 图片文档加载器
│   ├── edu_ocr.py          # 2.3 OCR文字识别处理
│   ├── edu_pdfloader.py    # 2.4 PDF文档加载器
│   ├── edu_pptloader.py    # 2.5 PPT文档加载器
│   └── __init__.py         # 2.6 加载器包初始化
├── edu_text_spliter/
│   ├── edu_chinese_recursive_text_splitter.py  # 2.7 中文递归文本分割器
│   ├── edu_model_text_spliter.py              # 2.8 模型驱动文本分割器
│   └── __init__.py                            # 2.9 分割器包初始化
└── __init__.py                                # 2.10 RAG包初始化
```

### 3. 向量存储层 (第四实现)
```
rag_qa/
└── core/
    ├── vector_store.py     # 3.1 向量数据库存储管理
    └── __init__.py         # 3.2 核心包初始化
```

### 4. 检索增强层 (第五实现)
```
rag_qa/
└── core/
    ├── query_classifier.py     # 4.1 查询分类器(BERT模型)
    ├── strategy_selector.py     # 4.2 检索策略选择器
    ├── prompts.py              # 4.3 提示词模板定义
    ├── prompts2.py             # 4.4 扩展提示词模板
    ├── rag_system.py           # 4.5 RAG系统核心实现
    ├── rag_system2.py          # 4.6 RAG系统扩展实现
    └── __init__.py             # 4.7 核心包初始化
```

### 5. 集成问答层 (第六实现)
```
├── new_main.py         # 5.1 新版集成问答系统主文件
├── old_main.py         # 5.2 旧版集成问答系统主文件
└── api.py              # 5.3 API接口封装
```

### 6. 应用服务层 (最后实现)
```
├── app.py              # 6.1 FastAPI Web服务应用
├── use_api.py          # 6.2 API使用示例
└── rag_qa/
    └── rag_main.py     # 6.3 RAG系统独立运行入口
```

## 各文件详细作用说明

### 0. 基础配置层

#### 0.1 base/config.py
**文件作用**: 系统全局配置管理，统一管理所有外部依赖的配置信息

**内部函数作用**:
- `__init__()`: 初始化配置解析器，读取并解析config.ini配置文件
- 配置属性:
  - MySQL连接配置(MYSQL_HOST, MYSQL_USER, MYSQL_PASSWORD, MYSQL_DATABASE)
  - Redis连接配置(REDIS_HOST, REDIS_PORT, REDIS_PASSWORD, REDIS_DB)
  - Milvus向量库配置(MILVUS_HOST, MILVUS_PORT, MILVUS_DATABASE_NAME, MILVUS_COLLECTION_NAME)
  - LLM模型配置(LLM_MODEL, DASHSCOPE_API_KEY, DASHSCOPE_BASE_URL)
  - 检索参数配置(PARENT_CHUNK_SIZE, CHILD_CHUNK_SIZE, CHUNK_OVERLAP, RETRIEVAL_K, CANDIDATE_M)
  - 应用配置(VALID_SOURCES, CUSTOMER_SERVICE_PHONE)

#### 0.2 base/logger.py
**文件作用**: 系统日志管理，提供统一的日志记录功能

**内部函数作用**:
- `setup_logging()`: 配置日志记录器，设置日志格式、输出位置和编码
- `logger`: 全局日志记录器实例，供其他模块使用

### 1. 数据库连接层

#### 1.1 mysql_qa/db/MySQLClient.py
**文件作用**: MySQL数据库连接客户端，封装数据库操作

**内部函数作用**:
- `__init__()`: 初始化MySQL连接，建立数据库连接池
- `execute_query()`: 执行查询语句，返回查询结果
- `execute_update()`: 执行更新语句，支持插入、更新、删除操作
- `close()`: 关闭数据库连接，释放资源

#### 1.2 mysql_qa/cache/RedisClient.py
**文件作用**: Redis缓存连接客户端，提供高速缓存功能

**内部函数作用**:
- `__init__()`: 初始化Redis连接，建立连接池
- `get()`: 从缓存中获取数据
- `set()`: 向缓存中设置数据
- `delete()`: 从缓存中删除数据
- `exists()`: 检查键是否存在

#### 1.3 mysql_qa/retrieval/bm25_search.py
**文件作用**: BM25检索算法实现，用于精确匹配检索

**内部函数作用**:
- `__init__()`: 初始化BM25检索器，连接Redis和MySQL客户端
- `preprocess_query()`: 预处理查询语句，去除停用词等
- `get_cached_candidates()`: 从缓存获取候选答案
- `calculate_bm25_scores()`: 计算BM25相似度分数
- `search()`: 执行BM25搜索，返回最佳匹配结果

### 2. 文档处理层

#### 2.1 rag_qa/edu_document_loaders/edu_docloader.py
**文件作用**: 通用文档加载器，支持基本文档格式

**内部函数作用**:
- `load_document()`: 加载指定路径的文档文件
- `extract_text()`: 从文档中提取纯文本内容
- `get_metadata()`: 获取文档元数据信息

#### 2.2 rag_qa/edu_document_loaders/edu_imgloader.py
**文件作用**: 图片文档加载器，处理图片格式文档

**内部函数作用**:
- `load_image()`: 加载图片文件
- `validate_image_format()`: 验证图片格式有效性

#### 2.3 rag_qa/edu_document_loaders/edu_ocr.py
**文件作用**: OCR文字识别处理，从图片中提取文字

**内部函数作用**:
- `perform_ocr()`: 执行OCR识别，提取图片中的文字内容
- `preprocess_image()`: 预处理图片以提高OCR识别准确率

#### 2.4 rag_qa/edu_document_loaders/edu_pdfloader.py
**文件作用**: PDF文档加载器，专门处理PDF格式

**内部函数作用**:
- `load_pdf()`: 加载PDF文件
- `extract_pages()`: 提取PDF页面内容
- `handle_encrypted_pdf()`: 处理加密PDF文件

#### 2.5 rag_qa/edu_document_loaders/edu_pptloader.py
**文件作用**: PPT文档加载器，处理PowerPoint格式

**内部函数作用**:
- `load_presentation()`: 加载PPT文件
- `extract_slides()`: 提取幻灯片内容
- `convert_to_text()`: 将幻灯片转换为文本

#### 2.7 rag_qa/edu_text_spliter/edu_chinese_recursive_text_splitter.py
**文件作用**: 中文递归文本分割器，专为中文文本设计

**内部函数作用**:
- `__init__()`: 初始化分割器参数(父块大小、子块大小、重叠大小)
- `split_text()`: 分割文本为父子块结构
- `_split_to_parent_chunks()`: 分割为父块
- `_split_to_child_chunks()`: 将父块进一步分割为子块

#### 2.8 rag_qa/edu_text_spliter/edu_model_text_spliter.py
**文件作用**: 模型驱动文本分割器，使用AI模型进行智能分割

**内部函数作用**:
- `split_with_model()`: 使用预训练模型进行语义感知分割
- `detect_semantic_boundaries()`: 检测语义边界点

### 3. 向量存储层

#### 3.1 rag_qa/core/vector_store.py
**文件作用**: 向量数据库存储管理，负责文档向量化和检索

**内部函数作用**:
- `__init__()`: 初始化Milvus连接，创建集合结构
- `generate_embeddings()`: 使用BGE-M3模型生成文本嵌入向量
- `add_documents()`: 将文档添加到向量数据库
- `hybrid_search_with_rerank()`: 混合搜索与重排序
- `_build_filter_expr()`: 构建过滤表达式
- `rerank()`: 使用BGE-Reranker模型对检索结果重排序

### 4. 检索增强层

#### 4.1 rag_qa/core/query_classifier.py
**文件作用**: 查询分类器，使用BERT模型判断查询类型

**内部函数作用**:
- `__init__()`: 初始化BERT分类模型
- `predict_category()`: 预测查询类别(通用知识/专业咨询)
- `preprocess_text()`: 预处理文本输入

#### 4.2 rag_qa/core/strategy_selector.py
**文件作用**: 检索策略选择器，自动选择最优检索策略

**内部函数作用**:
- `__init__()`: 初始化策略选择器和提示词模板
- `select_strategy()`: 根据查询内容选择检索策略
- `call_dashscope()`: 调用大语言模型辅助策略选择

#### 4.3 rag_qa/core/prompts.py
**文件作用**: 提示词模板定义，存储各类LLM提示词

**内部函数作用**:
- `rag_prompt()`: RAG问答提示词模板
- `hyde_prompt()`: HyDE策略提示词模板
- `subquery_prompt()`: 子查询策略提示词模板
- `backtracking_prompt()`: 回溯问题策略提示词模板

#### 4.5 rag_qa/core/rag_system.py
**文件作用**: RAG系统核心实现，整合所有检索增强功能

**内部函数作用**:
- `__init__()`: 初始化RAG系统组件
- `generate_answer()`: 生成最终答案的主要方法
- `retrieve_and_merge()`: 根据策略检索并合并文档
- `_retrieve_with_hyde()`: 假设文档检索(HyDE)
- `_retrieve_with_subqueries()`: 子查询检索
- `_retrieve_with_backtracking()`: 回溯问题检索
- `_consume_llm()`: 消费流式LLM生成器

### 5. 集成问答层

#### 5.1 new_main.py
**文件作用**: 新版集成问答系统主文件，整合MySQL和RAG两种检索方式

**内部函数作用**:
- `IntegratedQASystem.__init__()`: 初始化集成问答系统
- `IntegratedQASystem.call_dashscope()`: 调用DashScope API
- `IntegratedQASystem._fetch_recent_history()`: 获取最近对话历史
- `IntegratedQASystem.get_session_history()`: 获取会话历史
- `IntegratedQASystem.update_session_history()`: 更新会话历史
- `IntegratedQASystem.clear_session_history()`: 清除会话历史
- `IntegratedQASystem.query()`: 核心查询方法，支持流式输出
- `main()`: 命令行交互主函数

#### 5.2 old_main.py
**文件作用**: 旧版集成问答系统主文件，较简单的实现版本

**内部函数作用**:
- `IntegratedQASystem.__init__()`: 初始化问答系统
- `IntegratedQASystem.call_dashscope()`: 调用DashScope API
- `IntegratedQASystem.query()`: 查询方法
- `main()`: 主函数

#### 5.3 api.py
**文件作用**: API接口封装，提供对外服务接口

**内部函数作用**:
- 封装各种查询接口
- 提供RESTful API服务

### 6. 应用服务层

#### 6.1 app.py
**文件作用**: FastAPI Web服务应用，提供完整的Web API服务

**内部函数作用**:
- `create_app()`: 创建FastAPI应用实例
- 路由处理函数:
  - `/api/create_session`: 创建新会话
  - `/api/history/{session_id}`: 获取会话历史
  - `/api/query`: 非流式查询接口
  - `/api/stream`: 流式查询WebSocket接口
  - `/health`: 健康检查
  - `/api/sources`: 获取支持的学科类别
- `check_greeting()`: 检查问候语并返回预设回复
- `QueryRequest`: 查询请求数据模型
- `QueryResponse`: 查询响应数据模型

#### 6.2 use_api.py
**文件作用**: API使用示例，演示如何调用系统API

**内部函数作用**:
- `demo_api_call()`: API调用示例
- `test_connection()`: 测试API连接
- `batch_query()`: 批量查询示例

#### 6.3 rag_qa/rag_main.py
**文件作用**: RAG系统独立运行入口，可单独运行RAG功能

**内部函数作用**:
- `main()`: 主函数，支持数据处理模式和查询模式
- `process_documents()`: 处理文档并存储到向量库
- `interactive_query()`: 交互式查询模式
- `call_dashscope()`: 调用DashScope API函数

## 实现顺序说明

按照从底层基础设施到上层应用的顺序实现：

1. **基础配置层(0)**: 首先实现，为整个系统提供配置和日志支持
2. **数据库连接层(1)**: 实现数据存储和缓存连接，为检索功能提供基础
3. **文档处理层(2)**: 实现文档加载和分割功能，为向量化准备数据
4. **向量存储层(3)**: 实现向量数据库操作，为核心RAG功能奠定基础
5. **检索增强层(4)**: 实现各种检索策略和增强技术
6. **集成问答层(5)**: 整合所有功能，形成完整的问答系统
7. **应用服务层(6)**: 提供Web服务和API接口，使系统可对外提供服务

这种实现顺序确保了每层都建立在稳定的下层基础之上，有利于系统的稳定性和可维护性。