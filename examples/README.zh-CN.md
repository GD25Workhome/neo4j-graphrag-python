# 示例索引

本目录包含 `neo4j-graphrag` 包所支持各项功能的示例用法：

- 从 PDF 或文本进行[自动模式抽取](#schema-extraction)
- 从 PDF 或文本[构建知识图谱](#build-knowledge-graph)
- 从图中[检索](#retrieve)信息
- [问答](#answer-graphrag)（GraphRAG / Q&A）

以上每个环节都有多种可定制方式，汇总在[「自定义」一节](#customize)。

<a id="schema-extraction"></a>

## 自动模式抽取

- [从文本自动抽取模式](./automatic_schema_extraction/schema_from_text.py)

<a id="build-knowledge-graph"></a>

## 构建知识图谱

- [端到端：PDF 转图简单流水线](build_graph/simple_kg_builder_from_pdf.py)
- [端到端：文本转图简单流水线](build_graph/simple_kg_builder_from_text.py)
- [从配置文件构建 KG 流水线](build_graph/from_config_files/simple_kg_pipeline_from_config_file.py)
- [带 PDF URL 的 KG 流水线](build_graph/from_config_files/simple_kg_pipeline_from_config_file_with_url.py)

<a id="retrieve"></a>

## 检索

- [基于嵌入向量的检索器](retrieve/similarity_search_for_vector.py)
- [基于文本的检索器](retrieve/similarity_search_for_text.py)
- [使用 VectorCypherRetriever 的图检索](retrieve/vector_cypher_retriever.py)
- [混合检索器](./retrieve/hybrid_retriever.py)
- [混合 Cypher 检索器](./retrieve/hybrid_cypher_retriever.py)
- [Text2Cypher 检索器](./retrieve/text2cypher_search.py)
- [Cypher 模板检索器](./retrieve/tools/cypher_template_to_tool_example.py)


### 外部检索器

#### Weaviate

- [向量检索](customize/retrievers/external/weaviate/weaviate_vector_search.py)
- [文本检索（本地嵌入器）](customize/retrievers/external/weaviate/weaviate_text_search_local_embedder.py)
- [文本检索（远程嵌入器）](customize/retrievers/external/weaviate/weaviate_text_search_remote_embedder.py)

#### Pinecone

- [向量检索](./customize/retrievers/external/pinecone/pinecone_vector_search.py)
- [文本检索](./customize/retrievers/external/pinecone/pinecone_text_search.py)


### Qdrant

- [向量检索](./customize/retrievers/external/qdrant/qdrant_vector_search.py)
- [文本检索](./customize/retrievers/external/qdrant/qdrant_text_search.py)

<a id="answer-graphrag"></a>

## 回答：GraphRAG

- [端到端 GraphRAG](./answer/graphrag.py)
- [带消息历史的 GraphRAG](./question_answering/graphrag_with_message_history.py)
- [带 Neo4j 消息历史的 GraphRAG](./question_answering/graphrag_with_neo4j_message_history.py)

<a id="customize"></a>

## 自定义

### 检索器

- [控制 VectorRetriever 的结果格式](customize/retrievers/result_formatter_vector_retriever.py)
- [控制 VectorCypherRetriever 的结果格式](customize/retrievers/result_formatter_vector_cypher_retriever.py)
- [使用预过滤](customize/retrievers/use_pre_filters.py)
- [Text2Cypher：自定义提示词](customize/retrievers/text2cypher_custom_prompt.py)

### 大语言模型（LLM）

- [OpenAI (GPT)](./customize/llms/openai_llm.py)
- [Azure OpenAI]()
- [VertexAI (Gemini)](./customize/llms/vertexai_llm.py)
- [MistralAI](./customize/llms/mistalai_llm.py)
- [Cohere](./customize/llms/cohere_llm.py)
- [Anthropic (Claude)](./customize/llms/anthropic_llm.py)
- [Ollama](./customize/llms/ollama_llm.py)
- [自定义 LLM](./customize/llms/custom_llm.py)

- [消息历史](./customize/llms/llm_with_message_history.py)
- [基于 Neo4j 的消息历史](./customize/llms/llm_with_neo4j_message_history.py)
- [系统指令](./customize/llms/llm_with_system_instructions.py)

- [OpenAI 工具调用](./customize/llms/openai_tool_calls.py)
- [VertexAI 工具调用](./customize/llms/vertexai_tool_calls.py)
- [Ollama 工具调用](./customize/llms/ollama_tool_calls.py)


### 提示词

- [RAG 使用自定义提示词](customize/answer/custom_prompt.py)


### 嵌入器（Embedders）

- [OpenAI](./customize/embeddings/openai_embeddings.py)
- [Azure OpenAI](./customize/embeddings/azure_openai_embeddings.py)
- [VertexAI](./customize/embeddings/vertexai_embeddings.py)
- [MistralAI](./customize/embeddings/mistalai_embeddings.py)
- [Cohere](./customize/embeddings/cohere_embeddings.py)
- [Ollama](./customize/embeddings/ollama_embeddings.py)
- [自定义嵌入](./customize/embeddings/custom_embeddings.py)


### 知识图谱构建 - 流水线

- [端到端：显式组件 + 文本输入](./customize/build_graph/pipeline/kg_builder_from_text.py)
- [端到端：显式组件 + PDF 输入](./customize/build_graph/pipeline/kg_builder_from_pdf.py)
- [处理多份文档](./customize/build_graph/pipeline/kg_builder_two_documents_entity_resolution.py)
- [将词法图创建导出到另一条流水线](./customize/build_graph/pipeline/text_to_lexical_graph_to_entity_graph_two_pipelines.py)
- [从配置文件构建流水线](customize/build_graph/pipeline/from_config_files/pipeline_from_config_file.py)
- [添加事件监听以获知流水线进度](./customize/build_graph/pipeline/pipeline_with_notifications.py)
- [使用组件上下文发送组件进度通知](./customize/build_graph/pipeline/pipeline_with_component_notifications.py)


#### 组件

- 加载器（Loaders）：
  - [加载 PDF 文件](./customize/build_graph/components/loaders/pdf_loader.py)
  - [自定义](./customize/build_graph/components/loaders/custom_loader.py)
- 文本切分（Text Splitter）：
  - [固定大小切分](./customize/build_graph/components/splitters/fixed_size_splitter.py)
  - [来自 LangChain 的切分器](./customize/build_graph/components/splitters/langhchain_splitter.py)
  - [来自 LLamaIndex 的切分器](./customize/build_graph/components/splitters/llamaindex_splitter.py)
  - [自定义](./customize/build_graph/components/splitters/custom_splitter.py)
- [分块嵌入器]()
- 模式构建（Schema Builder）：
  - [用户自定义](./customize/build_graph/components/schema_builders/schema.py)
  - [自动模式抽取](./automatic_schema_extraction/schema_from_text.py)
- 实体关系抽取（Entity Relation Extractor）：
  - [基于 LLM](./customize/build_graph/components/extractors/llm_entity_relation_extractor.py)
  - [基于 LLM + 自定义提示](./customize/build_graph/components/extractors/llm_entity_relation_extractor_with_custom_prompt.py)
  - [自定义](./customize/build_graph/components/extractors/custom_extractor.py)
- [图剪枝](./customize/build_graph/components/pruners/graph_pruner.py)
- 知识图谱写入（Knowledge Graph Writer）：
  - [Neo4j 写入器](./customize/build_graph/components/writers/neo4j_writer.py)
  - [自定义](./customize/build_graph/components/writers/custom_writer.py)
- 实体解析（Entity Resolver）：
  - [FuzzyMatchResolver](./customize/build_graph/components/resolvers/fuzzy_match_entity_resolver_pre_filter.py)
  - [带预过滤的 SinglePropertyExactMatchResolver](./customize/build_graph/components/resolvers/simple_entity_resolver_pre_filter.py)
  - [带预过滤的 SpaCySemanticMatchResolver](./customize/build_graph/components/resolvers/spacy_entity_resolver_pre_filter.py)
  - [自定义解析器](./customize/build_graph/components/resolvers/custom_resolver.py)
- [自定义组件](./customize/build_graph/components/custom_component.py)


### 回答：GraphRAG

- [LangChain 兼容](./customize/answer/langchain_compatiblity.py)
- [使用自定义提示词](./customize/answer/custom_prompt.py)


## 数据库操作

- [创建向量索引](database_operations/create_vector_index.py)
- [创建全文索引](create_fulltext_index.py)
- [填充向量索引](populate_vector_index.py)
