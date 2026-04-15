# schema_from_text：提示词中文翻译与运行时实际 Prompt

## 1. 范围与结论

- 源提示词定义位置：`src/neo4j_graphrag/generation/prompts.py` 中 `SchemaExtractionTemplate.DEFAULT_TEMPLATE`。
- 运行链路位置：`examples/customize/build_graph/components/schema_builders/schema_from_text.py` 的 `inferred_schema = await schema_extractor.run(text=TEXT)`。
- 本例 `run` 未传 `examples`，因此 `examples=""`；最终 prompt 中 `Examples:` 下方为空行，然后直接进入 `Input text:`。

---

## 2. 提示词中文翻译（对照原模板）

以下为 `SchemaExtractionTemplate.DEFAULT_TEMPLATE` 的中文翻译（语义等价，便于理解）：

```text
你是一个顶级算法，专门用于以结构化格式提取带标签的属性图 schema。

请基于输入文本生成一个泛化的图 schema。识别关键的节点类型、关系类型以及属性类型。

重要规则：
1. 只返回抽象的 schema 信息，不要返回具体实例数据。
2. 节点类型标签使用单数 PascalCase（例如：Person、Company、Product）。
3. 关系类型标签使用 UPPER_SNAKE_CASE（例如：WORKS_FOR、MANAGES）。
4. 仅当能够较高置信度推断类型时才包含属性定义，否则省略该属性。
5. 定义 patterns 时，确保提到的每个节点标签和关系标签都存在于 node_types / relationship_types 列表中。
6. 不要创建文本中没有明确提及的节点类型。
7. schema 要尽量精简，只保留清晰可识别的模式。
8. 唯一性约束（UNIQUENESS）：
8.1 UNIQUENESS 是可选项；每个 node_type 可以有，也可以没有唯一性约束（通常最多一个）。
8.2 仅对样本中看起来缺失值较少的属性设置唯一性约束。
8.3 约束通过 node_type 标签引用节点类型，并指定唯一属性名。
8.4 如果某属性出现在唯一性约束中，它必须也出现在对应 node_type 的 properties 中。
9. 必填属性（required）：
9.1 若某属性对该节点/关系类型的每个实例都必须存在（不可空），标为 required=true。
9.2 若某属性是可选、在部分实例中可能缺失，标为 required=false。
9.3 标识符、名称或核心特征属性通常应为 required=true。
9.4 电话、描述、元数据等补充信息通常应为 required=false。
9.5 不确定时，默认 required=false。
9.6 若属性被施加 UNIQUENESS 约束，则必须标为 required=true。
10. 节点/关系标签禁止使用双下划线（__）作为前缀或后缀（如 __Person__、__KNOWS__）。

可接受的属性类型：
BOOLEAN, DATE, DURATION, FLOAT, INTEGER, LIST, LOCAL_DATETIME, LOCAL_TIME, POINT, STRING, ZONED_DATETIME, ZONED_TIME。

请返回一个严格符合以下结构的合法 JSON 对象：
{
  "node_types": [
    {
      "label": "Person",
      "properties": [
        {
          "name": "name",
          "type": "STRING",
          "required": true
        },
        {
          "name": "email",
          "type": "STRING",
          "required": false
        }
      ]
    }
    ...
  ],
  "relationship_types": [
    {
      "label": "WORKS_FOR"
    }
    ...
  ],
  "patterns": [
    {"source": "Person", "relationship": "WORKS_FOR", "target": "Company"},
    ...
  ],
  "constraints": [
    {
      "type": "UNIQUENESS",
      "node_type": "Person",
      "property_name": "name"
    }
    ...
  ]
}

Examples:
{examples}

Input text:
{text}
```

---

## 3. 运行时实际给模型的 Prompt（本示例）

说明：由 `SchemaFromTextExtractor.run(text=TEXT)` 调用 `SchemaExtractionTemplate.format(text=TEXT, examples="")` 生成。  
因此下面内容就是本次流程发给模型的“实际 Prompt 文本形态”。

```text
You are a top-tier algorithm designed for extracting a labeled property graph schema in
structured formats.

Generate a generalized graph schema based on the input text. Identify key node types,
their relationship types, and property types.

IMPORTANT RULES:
1. Return only abstract schema information, not concrete instances.
2. Use singular PascalCase labels for node types (e.g., Person, Company, Product).
3. Use UPPER_SNAKE_CASE labels for relationship types (e.g., WORKS_FOR, MANAGES).
4. Include property definitions only when the type can be confidently inferred, otherwise omit them.
5. When defining patterns, ensure that every node label and relationship label mentioned exists in your lists of node types and relationship types.
6. Do not create node types that aren't clearly mentioned in the text.
7. Keep your schema minimal and focused on clearly identifiable patterns in the text.
8. UNIQUENESS CONSTRAINTS:
8.1 UNIQUENESS is optional; each node_type may or may not have exactly one uniqueness constraint.
8.2 Only use properties that seem to not have too many missing values in the sample.
8.3 Constraints reference node_types by label and specify which property is unique.
8.4 If a property appears in a uniqueness constraint it MUST also appear in the corresponding node_type as a property.
9. REQUIRED PROPERTIES:
9.1 Mark a property as "required": true if every instance of that node/relationship type MUST have this property (non-nullable).
9.2 Mark a property as "required": false if the property is optional and may be absent on some instances.
9.3 Properties that are identifiers, names, or essential characteristics are typically required.
9.4 Properties that are supplementary information (phone numbers, descriptions, metadata) are typically optional.
9.5 When uncertain, default to "required": false.
9.6 If a property has a UNIQUENESS constraint, it MUST be marked as "required": true.
10. Never use double underscores (__) as a prefix or suffix in node labels or relationship types (e.g. __Person__ or __KNOWS__ are forbidden).

Accepted property types are: BOOLEAN, DATE, DURATION, FLOAT, INTEGER, LIST,
LOCAL_DATETIME, LOCAL_TIME, POINT, STRING, ZONED_DATETIME, ZONED_TIME.

Return a valid JSON object that follows this precise structure:
{
  "node_types": [
    {
      "label": "Person",
      "properties": [
        {
          "name": "name",
          "type": "STRING",
          "required": true
        },
        {
          "name": "email",
          "type": "STRING",
          "required": false
        }
      ]
    }
    ...
  ],
  "relationship_types": [
    {
      "label": "WORKS_FOR"
    }
    ...
  ],
  "patterns": [
    {"source": "Person", "relationship": "WORKS_FOR", "target": "Company"},
    ...
  ],
  "constraints": [
    {
      "type": "UNIQUENESS",
      "node_type": "Person",
      "property_name": "name"
    }
    ...
  ]
}

Examples:

Input text:
Acme Corporation was founded in 1985 by John Smith in New York City.
The company specializes in manufacturing high-quality widgets and gadgets
for the consumer electronics industry.

Sarah Johnson joined Acme in 2010 as a Senior Engineer and was promoted to
Engineering Director in 2015. She oversees a team of 12 engineers working on
next-generation products. Sarah holds a PhD in Electrical Engineering from MIT
and has filed 5 patents during her time at Acme.

The company expanded to international markets in 2012, opening offices in London,
Tokyo, and Berlin. Each office is managed by a regional director who reports
directly to the CEO, Michael Brown, who took over leadership in 2008.

Acme's most successful product, the SuperWidget X1, was launched in 2018 and
has sold over 2 million units worldwide. The product was developed by a team led
by Robert Chen, who joined the company in 2016 after working at TechGiant for 8 years.

The company currently employs 250 people across its 4 locations and had a revenue
of $75 million in the last fiscal year. Acme is planning to go public in 2024
with an estimated valuation of $500 million.
```

---

## 4. 补充说明

- 模板中写的是 `{{ ... }}`（双花括号）以避免被 `str.format` 当占位符；运行时会还原成单花括号 `{ ... }`，所以上方“实际 Prompt”中的 JSON 结构是单花括号。
- 本文只还原“业务层传入模型的 prompt 内容”。具体最终 HTTP 请求体还会再叠加 `model_name` 与 `model_params`（如 `response_format`、`temperature`、`max_tokens`）等参数。
