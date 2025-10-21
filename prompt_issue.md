
## Prompt 修改记录

### 最初写法

最开始的 prompt 是：

```python
prompt = PromptTemplate(
    input_variables=["query", "document"],
    template="On a scale from 0 to 1, how relevant is the following document to the query? "
             "Query: {query}\nDocument: {document}\nRelevance score:"
)
```

这种写法没有明确要求 LLM 返回结构化 JSON，只是让模型输出相关性分数，结果模型可能输出一段说明性文本或分数，导致结构化解析失败。

---

### 第一次修改

为了让 LLM 返回结构化 JSON，prompt 改为：

```python
prompt = PromptTemplate(
    input_variables=["query", "document"],
    template=(
        "On a scale from 0 to 1, how relevant is the following document to the query? "
        "Only return a JSON object in the following format: {\"relevance_score\": <float between 0 and 1>}.\n"
        "Query: {query}\nDocument: {document}\nRelevance score:"
    )
)
```

但这样写会导致 LangChain 把 `"relevance_score"` 当成变量，报错 `KeyError: 'Input to PromptTemplate is missing variables {"relevance_score"}'`。

---

### 正确写法

最终将 `{` 和 `}` 换成 `{{` 和 `}}`，明确告诉 LangChain这是文本不是变量：

```python
prompt = PromptTemplate(
    input_variables=["query", "document"],
    template=(
        "On a scale from 0 to 1, how relevant is the following document to the query? "
        "Only return a JSON object in the following format: {{\"relevance_score\": <float between 0 and 1>}}.\n"
        "Query: {query}\nDocument: {document}\nRelevance score:"
    )
)
```

这样就不会再把 `"relevance_score"` 识别为变量，结构化解析可以正常工作，问题解决。
