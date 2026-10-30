
即使只提取相关 `API` 的 `graph` 行数也是巨大的，导致发送的 `tokens` 巨大，在源代码中是没有做切片处理的，但是 `deepseek` 的 `tokens` 为 `6w` 多，测试的时候需要发送某个 `API` 的完整 `graph` 的 `tokens` 都达到了 `30w` 多，导致报错。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_WMDQHRMNN57GJE2.jpg)

解决方案：

1. 做切片处理
2. 对内容进行缩减
3. 更换模型

因为只是 `API` 的描述信息，所以 `graph` 少的情况下，影响不算特别大，所以这里使用对 `graph` 行数进行缩减，只发送前 `50` 行的调用关系。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_DHRNF26KXU4CCVR.jpg)

## 5、代码知识图谱（KG）

「1」 调用 `build_kg_query()` 构建知识图谱，返回值为 4 个语义检索索引、3 个图谱索引（这里的 `file_summary` 图谱不需要）、1 个元数据 `code_base`。

| 对象名 | 描述 |
| --- | --- |
| `pg_all_code_index` | 所有代码的图谱索引 |
| `pg_api_summary_index` | API 摘要的图谱索引 |
| `pg_api_code_index` | 代码的图谱索引 |
| `pg_file_summary_index` | 摘要文本的图谱索引 |
| `summary_text_vector_index` | 摘要文本的语义向量索引 |
| `all_src_code_vector_index` | 所有源码的语义向量索引 |
| `api_src_vector_index` | 所有 API 源码的向量索引 |
| `code_base` | 元信息 |

「2」 将这 **4 个** 图谱索引对象转换为 `BaseRetriever` 对象，设置 `similarity_top_k=3` 表示每次最多返回 `3` 个相似结果，为后续对话、查询、代码摘要生成等提供 **“上下文信息检索器”**。

「3」 创建一个支持混合多种信息源检索的 `CodeGraphRetriever` 对象，汇聚代码、摘要、结构等对象，并将 `CodeGraphRetriever` 设置到 `FuzzingPlanner`。

- `mode="HYBRID"`：表示它将同时从多个信息源中选取最合适的检索结果（例如代码 + 摘要的混合搜索）。

```
# fuzzing_llm_engine/fuzzing.py
# main()       logger.info(f"Init KG Model")
# index_pg_all_code, index_pg_api_summary, index_pg_api_code, index_pg_file_summary, summary_api_vector_index, all_src_code_vector_index, api_src_vector_index, code_base
# 初始化知识图谱,
# 并存储 summary_file_vector_index,api_src_vector_index,summary_api_vector_index,all_src_code_vector_index 图谱索引
pg_all_code_index, pg_api_summary_index, pg_api_code_index, pg_file_summary_index, summary_text_vector_index, all_src_code_vector_index, api_src_vector_index, code_base = \
    build_kg_query(
        chromadb_dir, # 用于存放 chromaDB 向量库的文件夹
        call_graph_csv, # API graph 的文件路径
        all_src_api_file, # 所有源代码 API 的文件路径
        api_summary_file, # API 摘要文件路径
        project_name, # 项目名
        kg_saved_folder, # 矩量图系列 index 存储路径
        initGraphKG = True, # 是否初始化知识图谱
        exclude_folder_list=exclude_folder_list, # 要排除的文件夹列表
        llm=llm_analyzer, # 用于分析代码的 LLM
        embed_model=llm_embedding # 文本向量化的 LLM
    )
# 返回值为 4个语义检索索引,3个图谱索引(这里的file_summary图谱不需要),1一个元数据code_base
'''
    pg_all_code_index        ->  所有代码的索引
    pg_api_summary_index     ->  API 摘要的索引
    pg_api_code_index        ->  代码的索引
    pg_file_summary_index    ->  文件摘要的索引
    summary_text_vector_index ->  摘要文本的向量索引
    all_src_code_vector_index ->  所有源代码的向量索引
    api_src_vector_index     ->  源代码的向量索引
    code_base                ->  原始代码
'''
# 将索引对象转换为 BaseRetriever 对象，设置 similarity_top_k=3 表示每次最多返回 3 个相似结果
# 这些 BaseRetriever 是从不同维度（代码、摘要、文件）对输入进行检索，作为提示或上下文的一部分。
pg_index_all_code_retriever = pg_all_code_index.as_retriever(similarity_top_k=3)
pg_index_api_summary_retriever = pg_api_summary_index.as_retriever(similarity_top_k=3)
pg_index_api_code_retriever = pg_api_code_index.as_retriever(similarity_top_k=3)
pg_index_file_summary_retriever = pg_file_summary_index.as_retriever(similarity_top_k=3)
# 初始化一个 CodeGraphRetriever,代码检索器
# mode="HYBRID" 使用混合检索方式
code_graph_retriever = CodeGraphRetriever(pg_index_all_code_retriever, pg_index_api_summary_retriever, pg_index_api_code_retriever, pg_index_file_summary_retriever, mode="HYBRID")
# 将 CodeGraphRetriever 设置到 FuzzingPlanner
plan_agent.set_code_graph_retriever(code_graph_retriever)
```

### `getCodeKG_CodeBase()`

`build_kg_query()` 是 `getCodeKG_CodeBase()` 的封装，这里直接看 `getCodeKG_CodeBase()`。

「1」 提取元数据，类型为 `CodeRepository`，通过 `get_codebase()` 获取代码库信息，包含项目的结构、文件路径、函数定义等内容，并可根据 `exclude_folder_list` 排除部分文件夹。

「2」 加载 `API` 摘要，判断是否初始化图谱。

「2.1」 初始化图谱：

[2.11] 基于 `graph` 调用图 `CSV` 和源码，构建代码调用图，并生成 5 类“文本节点”用于后续向量化：

- `all_src_text_nodes`：所有源代码的文本节点列表
- `summary_api_nodes`：每个 API 的摘要节点列表
- `file_summary_nodes`：文件摘要的文本节点列表
- `api_src_text_nodes`：每个 API 的源代码文本节点列表

[2.12] 对以上节点进行语义向量化（构建 `Chroma` 向量库）

[2.13] 构建 `4` 个属性图索引（`Property Graph Index`），每个索引代表一个不同视角的图谱，并结合对应的向量数据库，供后续查询使用。

[2.14] 持久化图谱索引：调用 `.persist()` 将图谱结构和向量嵌入结果写入磁盘，便于下次直接加载。

「2.2」 加载图谱（不初始化图谱），区别有：是否调用 `getCodeCallKGGraph()`，配合已存在的 `vector_store` 和 `property_graph_store` 恢复 `4` 个图谱索引对象。
