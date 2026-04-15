# Retriever 子类实现逻辑解析

本文基于 `src/neo4j_graphrag/retrievers/base.py` 的 `Retriever` 抽象类，扫描工程中的继承实现，并说明各子类的核心职责、执行流程与差异。

## 1. 继承结构总览

### 1.1 抽象基类

- `Retriever`：所有检索器的统一入口，定义 `search()` 模板方法和 `get_search_results()` 抽象方法。
- `ExternalRetriever`：面向外部向量库（非 Neo4j 原生向量索引）的抽象扩展层，继承自 `Retriever`。

### 1.2 生产代码中的直接/间接子类

- `VectorRetriever`
- `VectorCypherRetriever`
- `HybridRetriever`
- `HybridCypherRetriever`
- `Text2CypherRetriever`
- `ToolsRetriever`
- `WeaviateNeo4jRetriever`（继承 `ExternalRetriever`）
- `QdrantNeo4jRetriever`（继承 `ExternalRetriever`）
- `PineconeNeo4jRetriever`（继承 `ExternalRetriever`）

补充：工程中在 `tests/` 与 `examples/` 还有一些示例/测试专用子类（如 `MockRetriever`、`StaticRetriever` 等），不属于核心生产实现。

## 2. 基类 `Retriever` 的模板方法逻辑

`Retriever.search()` 是统一出口，流程如下：

1. 调用子类实现的 `get_search_results()`，得到 `RawSearchResult`（原始 `neo4j.Record` 列表 + 可选 metadata）。
2. 获取格式化函数（`get_result_formatter()`），逐条把 record 转换为 `RetrieverResultItem`。
3. 拼装 `RetrieverResult(items, metadata)` 返回，并自动写入 `metadata["__retriever"]` 标记来源类名。

这意味着各子类只需要关注：

- 如何拿到原始记录（检索策略）
- 是否提供自定义 `result_formatter`（展示格式）

另外，`Retriever.__init__()` 默认会做 Neo4j 版本能力检查（向量索引、metadata filtering 支持）；可由子类通过 `VERIFY_NEO4J_VERSION = False` 关闭。

## 3. 各子类实现逻辑

## 3.1 `VectorRetriever`

定位：Neo4j 原生向量索引检索。

核心流程：

1. 初始化阶段使用 Pydantic 模型校验参数。
2. 通过 `_fetch_index_infos(index_name)` 从 `SHOW VECTOR INDEXES` 读取索引绑定的标签、向量属性、维度。
3. `get_search_results()` 中验证查询参数（`query_text`/`query_vector`/`top_k`/`filters`）。
4. 若传 `query_text`，则调用 embedder 生成 `query_vector`。
5. 调 `get_search_query(SearchType.VECTOR, ...)` 生成 Cypher。
6. `driver.execute_query()` 执行后返回 `RawSearchResult`，metadata 附带查询向量。

特点：标准向量检索实现，支持 metadata 过滤与候选池倍率（`effective_search_ratio`）。

## 3.2 `VectorCypherRetriever`

定位：向量召回 + 追加 Cypher 图遍历/重构结果。

相比 `VectorRetriever` 多出的能力：

- 接收 `retrieval_query`，在基础向量召回后继续做图查询。
- 支持 `query_params` 透传到追加查询。

流程与 `VectorRetriever` 类似，但在生成查询时传入 `retrieval_query`，实现“先 ANN 召回，再图模式扩展”的组合检索。

## 3.3 `HybridRetriever`

定位：向量检索 + 全文检索融合（Hybrid）。

核心流程：

1. 同时持有 `vector_index_name` 与 `fulltext_index_name`。
2. 输入 `query_text` 必填，可选 `query_vector`（若未给且有 embedder，则由文本生成向量）。
3. 通过 `get_search_query(SearchType.HYBRID, ...)` 生成融合查询。
4. 支持 `ranker`（如 naive/linear）与 `alpha`（线性融合权重）。
5. 捕获 Lucene 解析异常并转成 `SearchQueryParseError`。

特点：适合“语义 + 关键词”混合召回场景。

## 3.4 `HybridCypherRetriever`

定位：Hybrid 召回 + 追加 Cypher。

逻辑是 `HybridRetriever` 的增强版：

- 在混合召回基础上引入 `retrieval_query`。
- 支持 `query_params` 注入下游 Cypher。

适用于先做广召回，再沿图关系做定制加工输出。

## 3.5 `Text2CypherRetriever`

定位：自然语言 -> Cypher -> Neo4j 查询。

核心流程：

1. 初始化时准备 schema（优先用传入 schema；否则自动调用 `get_schema()` 拉取）。
2. `get_search_results()` 中把用户 `query_text` 与 schema、examples 组装进提示词模板。
3. 调用 LLM 生成 Cypher，`extract_cypher()` 负责从代码块中抽取语句并修正带空格标识符的反引号。
4. 执行生成的 Cypher 并返回记录；metadata 含最终 Cypher 文本。

特点：非向量检索路径，偏“结构化问答/图查询生成”。

## 3.6 `ToolsRetriever`

定位：让 LLM 在多个 Tool 间做路由编排。

核心逻辑：

1. 输入是 `tools` 列表（通常由其他 retriever 的 `convert_to_tool()` 转换而来）。
2. `llm.invoke_with_tools()` 让模型选择要调用的工具及参数。
3. 逐个执行工具并聚合结果，统一转为 `RawSearchResult`。
4. 对空工具列表、无 tool call、执行异常都给出可读 metadata。

特点：它本身不直接做图检索，而是“检索编排器”。

## 3.7 外部向量库子类（`ExternalRetriever` 体系）

共性（Weaviate/Qdrant/Pinecone）：

1. 在外部向量库完成向量近邻检索，得到 `(external_id, score)` 列表。
2. 通过 `id_property_external` 与 `id_property_neo4j` 映射到 Neo4j 节点。
3. 调 `get_match_query(...)` 在 Neo4j 中回查节点及属性，返回统一格式。

### 3.7.1 `WeaviateNeo4jRetriever`

- 支持 `near_vector` 与 `near_text` 两条路径。
- 若配置本地 embedder，可先本地嵌入再发起 `near_vector`。
- 支持 `weaviate_filters`。

### 3.7.2 `QdrantNeo4jRetriever`

- 必须有向量输入；若只给文本则要求 embedder 生成向量。
- 调 `client.query_points()`，可通过 `id_property_getter` 自定义从 `ScoredPoint` 抽取匹配 ID 的策略。

### 3.7.3 `PineconeNeo4jRetriever`

- 与 Qdrant 类似，先在 Pinecone 查询，再回 Neo4j。
- 支持 `pinecone_filter`。

## 4. 设计层面的统一抽象

从架构上，这组类体现了一致的“检索模板”：

- **参数校验层**：Pydantic model 统一做初始化与查询参数校验。
- **检索执行层**：每个子类实现 `get_search_results()`，可走 Neo4j 原生索引、外部向量库或 LLM 生成查询。
- **结果规范层**：最终都回到 `RetrieverResult`/`RetrieverResultItem` 的统一输出协议。
- **可工具化层**：所有 retriever 可经 `convert_to_tool()` 变成 Tool，被 `ToolsRetriever` 编排。

## 5. 快速对比（选型视角）

- 需要 Neo4j 原生向量检索：`VectorRetriever`
- 需要向量召回后图扩展：`VectorCypherRetriever`
- 需要语义 + 关键词混合：`HybridRetriever`
- 需要混合召回后图扩展：`HybridCypherRetriever`
- 需要自然语言直接查图：`Text2CypherRetriever`
- 需要多检索器自动路由：`ToolsRetriever`
- 向量数据在外部库：`WeaviateNeo4jRetriever` / `QdrantNeo4jRetriever` / `PineconeNeo4jRetriever`

