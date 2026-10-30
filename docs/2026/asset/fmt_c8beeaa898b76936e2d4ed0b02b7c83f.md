
```python
# fuzzing_llm_engine/rag/kg.py

def getCodeKG_CodeBase(
    chromadb_dir: str,
    call_graph_csv: str,
    src_api_file,
    api_summary_file,
    project_name,
    saved_folder,
    llm,
    embed_model,
    exclude_folder_list,
    initGraphKG=True,
    **kwargs
):
    """
    Construct the code knowledge graph and code base.
    Parameters:
    - chromadb_dir (str): The directory to store the ChromaDB.
    - call_graph_csv: The path to the call graph CSV file.
    - src_api_file: The path to the source API file.
    - api_summary_file: The path to the API summary file.
    - graph_folder: The folder to store the graph.
    - project_name: The name of the project.
    - saved_folder: The folder to save the knowledge graph.
    - initGraphKG (bool): Whether to initialize the graph knowledge graph. Default is True. This will embed the index ndoes with vectors.
    - exclude_folder_list (list): List of folders to exclude from the code base. Default is ["c-ares/src/tools"].
    - **kwargs: Additional keyword arguments.
    Returns:
    - index_pg_all_code: The index of the code property graph.
    - index_pg_api_summary: The index of the summary property graph.
    - index_pg_api_code: Then index of the code property graph for API functions
    - summary_text_vector_index:
    - all_src_code_vector_index:
    - code_base: The code base.
    """
    logger.info(f"Constructing code knowledge graph and code base for {project_name}")
    # CodeRepository 类，获取代码库信息，并排除指定的文件夹
    code_base = get_codebase(src_api_file, project_name=project_name, exclude_folder_list=exclude_folder_list)
    # 加载 API 摘要文件
    api_summary = json.load(open(api_summary_file, "r"))
    # 是否初始化知识图谱
    if initGraphKG:
        # 构建代码调用图和相关节点
        # 图存储对象,实体节点列表,所有源代码的文本节点列表,API 摘要的文本节点列表,文件摘要的文本节点列表,API 源代码的文本节点列表
        graph_store, entities_nodes, all_src_text_nodes, summary_api_nodes, file_summary_nodes, api_src_text_nodes = getCodeCallKGGraph(
            call_graph_csv,
            code_base,
            api_summary,
            project_name
        )
        # 创建四个 Chroma 向量数据库并获取索引
        api_src_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "api_src_text_nodes"),
            "api_src_text_nodes",
            api_src_text_nodes,
            initGraphKG,
            llm,
            embed_model
        )
        api_src_vector = api_src_vector_index._vector_store

        all_src_code_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "all_src_text_nodes"),
            "all_src_text_nodes",
            all_src_text_nodes,
            initGraphKG,
            llm,
            embed_model
        )
        all_src_code_vector = all_src_code_vector_index._vector_store

        summary_api_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "summary_api_nodes"),
            "summary_api_nodes",
            summary_api_nodes,
            initGraphKG,
            llm,
            embed_model
        )
        summary_api_vector = summary_api_vector_index._vector_store

        # file_summary_nodes
        summary_file_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "file_summary_nodes"),
            "file_summary_nodes",
            file_summary_nodes,
            initGraphKG,
            llm,
            embed_model
        )
        summary_file_vector = summary_file_vector_index._vector_store

        # 构建四个相关的属性图索引
        # index_pg_all_code = PropertyGraphIndex.from_existing(\
        #             property_graph_store=graph_store, \
        #             vector_store=all_src_code_vector, \
        #             llm=llm, embed_model=embed_model,\
        #             embed_kg_nodes=True, show_progress=True,use_async=False)
        # 构建相关的属性图索引 (PropertyGraphIndex)

        index_pg_all_code = PropertyGraphIndex.from_existing(
            property_graph_store=graph_store,       # 图数据库对象
            vector_store=all_src_code_vector,       # 向量存储
            llm=llm,                                # 大语言模型
            embed_model=embed_model,                # 向量嵌入模型
            embed_kg_nodes=True,                    # 对知识图谱中的节点进行嵌入处理
            show_progress=True,                     # 显示构建过程的进度条
            use_async=False                         # 同步执行
        )
        index_pg_api_code = PropertyGraphIndex.from_existing(
            property_graph_store=graph_store,
            vector_store=api_src_vector,
            llm=llm,
            embed_model=embed_model,
            embed_kg_nodes=True,
            show_progress=True,
            use_async=False
        )

        index_pg_api_summary = PropertyGraphIndex.from_existing(
            property_graph_store=graph_store,
            vector_store=summary_api_vector,
            llm=llm,
            embed_model=embed_model,
            embed_kg_nodes=True,
            show_progress=True,
            use_async=False
        )

        index_pg_file_summary = PropertyGraphIndex.from_existing(
            property_graph_store=graph_store,
            vector_store=summary_file_vector,
            llm=llm,
            embed_model=embed_model,
            embed_kg_nodes=True,
            show_progress=True,
            use_async=False
        )
        # 存储图谱索引
        # 这一步会保存嵌入后的向量数据、图谱信息等到磁盘，供后续加载使用
        saved_pg_api_code_dir = os.path.join(saved_folder, "kg", "index_pg_api_code")
        saved_pg_all_code_dir = os.path.join(saved_folder, "kg", "index_pg_all_code")
        saved_pg_api_summary_dir = os.path.join(saved_folder, "kg", "index_pg_api_summary")
        saved_pg_file_summary_dir = os.path.join(saved_folder, "kg", "index_pg_file_summary")
        os.makedirs(saved_pg_all_code_dir, exist_ok=True)
        os.makedirs(saved_pg_api_summary_dir, exist_ok=True)
        os.makedirs(saved_pg_api_code_dir, exist_ok=True)
        os.makedirs(saved_pg_file_summary_dir, exist_ok=True)
        index_pg_all_code.storage_context.persist(persist_dir=saved_pg_all_code_dir)
        index_pg_api_summary.storage_context.persist(persist_dir=saved_pg_api_summary_dir)
        index_pg_api_code.storage_context.persist(persist_dir=saved_pg_api_code_dir)
        index_pg_file_summary.storage_context.persist(persist_dir=saved_pg_file_summary_dir)
    else:
        # 加载4个图谱索引
        saved_pg_all_code_dir = os.path.join(saved_folder, "kg", "index_pg_all_code")
        saved_pg_api_summary_dir = os.path.join(saved_folder, "kg", "index_pg_api_summary")
        saved_pg_api_code_dir = os.path.join(saved_folder, "kg", "index_pg_api_code")
        saved_pg_file_summary_dir = os.path.join(saved_folder, "kg", "index_pg_file_summary")

        all_src_code_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "all_src_text_nodes"),
            "all_src_text_nodes",
            all_src_text_nodes,
            initGraphKG,
            llm,
            embed_model
        )

        summary_api_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "summary_api_nodes"),
            "summary_api_nodes",
            summary_api_nodes,
            initGraphKG,
            llm,
            embed_model
        )

        api_src_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "api_src_text_nodes"),
            "api_src_text_nodes",
            api_src_text_nodes,
            initGraphKG,
            llm,
            embed_model
        )

        summary_file_vector_index = get_or_construct_chromadb(
            os.path.join(chromadb_dir, "file_summary_nodes"),
            "file_summary_nodes",
            file_summary_nodes,
            initGraphKG,
            llm,
            embed_model
        )

        index_pg_all_code = PropertyGraphIndex.from_existing(
            property_graph_store=SimplePropertyGraphStore.from_persist_dir(saved_pg_all_code_dir),
            vector_store=all_src_code_vector_index._vector_store,
            embed_kg_nodes=True,
            show_progress=True,
            llm=llm,
            embed_model=embed_model,
            **kwargs
        )
        index_pg_api_summary = PropertyGraphIndex.from_existing(
            property_graph_store=SimplePropertyGraphStore.from_persist_dir(saved_pg_api_summary_dir),
            vector_store=summary_api_vector_index._vector_store,
            embed_kg_nodes=True,
            show_progress=True,
            llm=llm,
            embed_model=embed_model,
            **kwargs
        )
        index_pg_api_code = PropertyGraphIndex.from_existing(
            property_graph_store=SimplePropertyGraphStore.from_persist_dir(saved_pg_api_summary_dir),
            vector_store=api_src_vector_index._vector_store,
            embed_kg_nodes=True,
            show_progress=True,
            llm=llm,
            embed_model=embed_model,
            **kwargs
        )
        index_pg_file_summary = PropertyGraphIndex.from_existing(
            property_graph_store=SimplePropertyGraphStore.from_persist_dir(saved_pg_file_summary_dir),
            vector_store=summary_file_vector_index._vector_store,
            embed_kg_nodes=True,
            show_progress=True,
            llm=llm,
            embed_model=embed_model,
            **kwargs
        )

    return index_pg_all_code, index_pg_api_summary, index_pg_api_code, index_pg_file_summary, summary_api_vector_index, all_src_code_vector_index, api_src_vector_index, code_base
```
