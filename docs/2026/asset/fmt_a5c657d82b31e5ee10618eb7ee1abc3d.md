
```python
# fuzzing_llm_engine/rag/code_base.py
def process_source_files(self, code_data, exclude_folders):
    fn_def_list_node_list = [ ]
    global_var_node_list = []
    struct_node_list = []
    enum_node_list = []
    # Patch
    api_path = list(code_data.keys())[0]
    api_path_prefix_len = api_path.find(f"{self.project_name}") + len(f"{self.project_name}")
    api_path_prefix = api_path[:api_path_prefix_len]
    logger.debug(f"{api_path_prefix}")
    for fname in code_data:
        values = code_data[fname]
        fn_def_list = values["fn_def_list"] # List of function definitions, Method Nodoes
        struct_list = values["struct_node_list"] # List of struct definitions, Struct Nodes
        include_list = values["include_list"] # include relationship between file and method
        global_varibales = values["global_variables"]
        # fn_declaraion = values["fn_declaraion"]
        enum_declaration = values["enumerate_node_list"]
        # remmove the shared folder name
        # Patch
        # fid = fname.replace(f"../docker_shared/source_code/{self.project_name}", "")
        fid = fname.replace(api_path_prefix, "")
        if os.path.dirname(fid) in exclude_folders:
            continue
        for i, fdef in enumerate(fn_def_list):
            fn_code = fdef['fn_code'].strip()
            fn_meta = fdef['fn_meta']
            fn_name = fn_meta['identifier']
            parameters = fn_meta['parameters']
            meta_node_info = {"id":f"{fid}-{fn_name}", "fid":fid, "project": self.project_name, "code":fn_code, "name":fn_name, "parameters": parameters}
            fn_def_list_node_list.append(meta_node_info)
        for gi, gv in enumerate(global_varibales):
            meta_gnode_info = {id:f"{fid}-g-{gi}", "code":gv, "name":gi,  "project": self.project_name, "fid":fid}
            global_var_node_list.append(meta_gnode_info)
        for s in struct_list:
            struct_body = s[0]
            struct_parameters = s[1]
            struc_name = s[2]
            struct_id = f"{fid}-{struc_name}"
            meta_struct_node_info = {"id":struct_id, "name":struc_name, "fid":fid,"parameters":struct_parameters, "code":struct_body}
            struct_node_list.append(meta_struct_node_info)
        # 添加  enum
        for e in enum_declaration:
            enum_body = e[0]
            enum_parameters = e[1]
            enum_name = e[2]
            enum_id = f"{fid}-{enum_name}"
            meta_enum_node_info = {"id":enum_id, "name":enum_name, "fid":fid, "parameters":enum_parameters, "code":enum_body}
            enum_node_list.append(meta_enum_node_info)
        return fn_def_list_node_list, global_var_node_list, struct_node_list, enum_node_list
```

### Patch

在 `construct_nodes_fn_doc` 函数中，使用 `replace` 来替换，这样其实可以抵消项目中有特殊符号导致的问题，但是当使用项目为 `c-ares` 时，我们的路径又是绝对地址这样就会替换失败，导致后面全部提取失败

![](https://bbs.kanxue.com/upload/attach/202507/1026022_N48MH9SEFJGR9P7.jpg "null")

上面代码在 `fuzzing_llm_engine/rag/kg.py` 中的 `get_codebase()` 被调用，原目的是替换路径用来构建源数据为图存储对象

![](https://bbs.kanxue.com/upload/attach/202507/1026022_NPMNRXT6CE63PF7.jpg "null")

但在后面的 `getCodeCallKGGraph()` 中调用，使用 `"-"` 来分割文件名和文件函数，这里就有问题了，当路径为绝对地址不能被 `replace()` 时，就会导致全部提取失败。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_UQKQ8NHSWPRUD2H.jpg "null")

解决方案：

1、替换 `"-"`，用其他更特殊的符号如 `"$"` 来区别

2、使用相对路径进行 `api` 的提取

3、修改 `replace` 函数代码

这里使用第三种方案，添加前缀获取并替换模式

在 `construct_nodes_fn_doc()` 函数中修改以下代码

![](https://bbs.kanxue.com/upload/attach/202507/1026022_VHCS3NPKN6K8KWY.jpg "null")

在该函数中还会调用 `process_source_files()` 里面也使用了类似逻辑

![](https://bbs.kanxue.com/upload/attach/202507/1026022_72JBEF8XYR9W7NK.jpg "null")

修改如下

![](https://bbs.kanxue.com/upload/attach/202507/1026022_Q9793Z35WQH49VD.jpg "null")

修改效果，这样修改之后就能规避项目或者路径中自带的 `-` 干扰。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_4KDRPMBM6J9D498.jpg "null")

### `get_or_construct_chromadb`

「1」 创建一个持久化的 Chroma 客户端(连接到磁盘上的数据)

「2」 获取或创建指定名称的 `collection`（类似于数据库中的表）。

「3」 用 `collection` 构建 `ChromaVectorStore`，封装 `collection`，变成 `llama-index` 中支持的向量存储接口。

「4.1」 初始化新的向量索引：

设置 LLM 和嵌入模型、创建向量存储上下文、构建向量索引并写入 `TextNodes`。

「4.2」 不初始化新的向量索引,从现有向量数据库加载索引。

```python
# 获取或构建一个 Chroma 向量数据库索引
def get_or_construct_chromadb(chromadb_dir: str, chromadb_name: str, nodes: list[TextNode], initVector: bool, llm, embed_model) -> VectorStoreIndex:
    """
    获取或构建一个 Chroma 向量数据库索引(ChromaVectorStore)。
    如果 initVector 为 True,则使用提供的 TextNode 列表初始化新的向量存储。
    否则，将尝试从指定路径加载已有的向量存储。
    参数:
        chromadb_dir (str): ChromaDB 所在的目录路径。
        chromadb_name (str): 数据库集合名称。
        nodes (list[TextNode]): 待添加到向量数据库的文本节点列表。
        initVector (bool): 是否初始化新的向量存储。
        llm: 大语言模型(用于配置 Settings)。
        embed_model: 向量嵌入模型(用于配置 Settings)。
    返回:
        VectorStoreIndex: 返回一个可用于查询的向量索引对象。
    """
    print(f"Constructing ChromaDB for {chromadb_name}")
    print(f"ChromaDB directory: {chromadb_dir}")
    # 创建一个持久化的 Chroma 客户端(连接到磁盘上的数据)
    client = chromadb.PersistentClient(chromadb_dir)
    # 获取或创建指定名称的 collection（类似于数据库中的表）
    collection = client.get_or_create_collection(chromadb_name)
    # 构建一个 Chroma 向量存储对象
    cvs = ChromaVectorStore(chroma_collection=collection)
    if initVector:
        Settings.llm=llm
        Settings.embed_model=embed_model
        # 创建一个默认的存储上下文，并注入当前的向量存储
        storage_context = StorageContext.from_defaults(vector_store=cvs)
        # nodes_len = len(nodes)
        # nodes_idls = [ n.id_ for n in nodes ]
        # assert len(set(nodes_idls)) == nodes_len
        # 构建向量索引，将 TextNode 列表写入向量数据库中
        vector_index = VectorStoreIndex(nodes, storage_context=storage_context)
        # 从向量索引中获取最新的向量存储
        cvs = vector_index._vector_store
    else:
        # 如果不初始化，直接从已有的向量存储加载向量索引
        storage_context = StorageContext.from_defaults(vector_store=cvs)
        vector_index = VectorStoreIndex.from_vector_store(
                        cvs, storage_context=storage_context )
    return vector_index
```

#### `getCodeCallKGGraph`

根据函数调用关系和代码信息，构建一个用于**图查询**（`Knowledge Graph Query`）的**知识图谱**，其中包括函数、文件、调用关系、函数摘要、源代码等多种信息，最终返回构建好的**图谱及其相关节点集合**。

「1」 使用 `pandas` 读取 `call_graph_csv` 文件、初始化图存储对象。

「2」 初始化实体节点、关系和源代码块的列表，维护 `methods_in_codebase(所有项目中定义的方法名)` -> 遍历所有函数定义提取函数名。

「4」 逐行构建调用图中的实体节点和边,对每一条调用记录：

- 判断 `caller` 和 `callee` 是否为项目中的函数（否则视为库函数）
- 提取函数签名、源代码（如果有）
- 判断是否有 API 摘要信息并加入属性
- 为函数和文件创建实体节点 (`EntityNode`)
- 为调用关系创建边 (`Relation`)：`CALLS`：函数调用、`CONTAIN`：文件包含函数。

「4」 插入节点和边到图谱

![](https://bbs.kanxue.com/upload/attach/202507/1026022_Q6HMVW2QKBA5GHY.jpg "null")

「5」构建函数定义和摘要的文本节点 (`TextNode`)，遍历所有函数定义 `fn_def` ：

如果该函数已添加到图谱中，则构建对应的 `TextNode`（纯文本节点）

- `all_src_text_nodes`：所有函数代码节点
- `summary_api_nodes`： 函数级摘要节点
- `file_summary_nodes`：文件级摘要节点
- `API_src_text_nodes`：摘要中提到的 API 函数的源码节点
