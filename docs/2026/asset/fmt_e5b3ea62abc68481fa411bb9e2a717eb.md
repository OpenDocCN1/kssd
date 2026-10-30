
```
# fuzzing_llm_engine/rag/kg.py def getCodeCallKGGraph(method_call_csv_file:str, code_base:CodeRepository, api_summary:dict,project_name:str):
    """
    构建用于图查询的节点。
    参数:
        method_call_csv_file (str): 方法调用关系的 CSV 文件路径。
        code_base (CodeRepository): 提供代码信息的对象。
        api_summary (dict): API 摘要信息。
        project_name (str): 项目名称。
    返回:
        graph_store (SimplePropertyGraphStore): 图存储对象。
        entities (list): 实体节点列表。
        all_src_text_nodes (list): 所有源代码的文本节点列表。
        summary_api_nodes (list): API 摘要的文本节点列表。
        file_summary_nodes (list): 文件摘要的文本节点列表。
        API_src_text_nodes (list): API 源代码的文本节点列表。
    """
    # 初始化图存储对象
    graph_store = SimplePropertyGraphStore()
    # Step 1: Read the CSV file with pandas # Step 1: 使用 pandas 读取 call_graph_csv 文件
    df = pd.read_csv(method_call_csv_file) # Step 2: 初始化实体节点、关系和源代码块的列表
    # Step 2: Init Text Node with the specific id. The id has to be the same with EntityNode (name is id)
    entities = []               # 所有实体节点列表
    relations = []              # 所有关系边列表
    methods_in_codebase = [ ]   # 所有项目中定义的方法名
    all_src_text_nodes = [ ]    # 所有函数定义对应的文本节点
    API_src_text_nodes = [ ]    # 所有 API 摘要关联函数的源代码节点
    summary_api_nodes = [ ]     # API 摘要的文本节点
    file_summary_nodes = [ ]    # 文件级别的摘要文本节点
    ids_in_graph = []           # 所有已加入图谱中的 id
    # 维护 methods_in_codebase 遍历所有函数定义提取函数名
    for fn_id in code_base.fn_def:
        parts = fn_id.split("-") # 路径或项目自带 - ，这不就提取的有问题吗
        fid = parts[0]
        method_name = parts[1] # 可以尝试用 -1 索引,但是 fid 仍然有问题
        if method_name == "":
            continue
        methods_in_codebase.append(method_name)
    # Step 3: 初始化实体、关系和源代码块的列表                   # Step 3: Initialize lists for entities, relations, and source chunks
    # 遍历 call_graph_csv 中每一行记录，构建调用关系图
    for _, row in df.iterrows():
        caller = row['caller']  # 调用方函数名
        callee = row['callee']  # 被调用方函数名
        # if caller == "ares__buf_append" or callee=="ares__buf_append":
        #     print("debug")
        caller_src_code = ""        # 调用方法的源代码（默认空）
        callee_src_code = ""        # 被调用方法的源代码（默认空）
        relationship_call = "CALLS" # 默认关系为调用 CALLS
        caller_src = row['caller_src']  # 调用方法的文件路径
        callee_src = row['callee_src']  # 被调用方法的文件路径
        # 跳过缺失源码路径的记录  pd.isna -> 检测缺失值的方法
        if pd.isna(caller_src) or pd.isna(callee_src):
            continue
        callee_src = callee_src.replace(f"/src/{project_name}", "")
        caller_src = caller_src.replace(f"/src/{project_name}", "")
        # 获取函数签名
        caller_signature = row['caller_signature']
        callee_signature = row['callee_signature']
        # Step check if the method is in code or from standard or third library
        # 判断调用方法是否是项目内部方法,如果是则提取 函数代码
        if caller in methods_in_codebase:
            caller_label = "METHOD"
            caller_src_id = f"{caller_src}-{caller}"
            if caller_src_id in code_base.fn_def: 
                caller_src_code = code_base.fn_def[caller_src_id]['code'] 
        else:
            caller_label = "LIBRARY_METHOD"
            relationship_call = "LIBRARY_CALLS"
        if callee in methods_in_codebase:
            callee_label = "METHOD"
            callee_src_id = f"{callee_src}-{callee}"
            if callee_src_id in code_base.fn_def:
                callee_src_code = code_base.fn_def[callee_src_id]['code'] 
        else:
            callee_label = "LIBRARY_METHOD"
            relationship_call = "LIBRARY_CALLS"
        # Step 4: 创建调用者和被调用者的实体节点，添加调用者函数节点                   # Step 4: Create entities for the caller and callee
        # add entity caller funciton node 
        if caller_src_code.strip():
            caller_properties = {"signature": caller_signature, "file": caller_src, "source code": f"```code\n{caller_src_code}\n```"}
        else:
            caller_properties = {"signature": caller_signature, "file": caller_src}
        # 如有摘要则加入摘要属性
        caller_summary = check_funciton_has_summary(caller_src, caller, api_summary)
        if len(caller_summary):
            caller_properties['summary'] = caller_summary
        # 构建 caller 的 EntityNode
        caller_entity = EntityNode(
            name=f"{caller_src}-{caller}",
            label=caller_label,
            properties=caller_properties
        )
        # 函数的实体节点 加入图谱的 id 形式为 <caller_src>-<caller> (<相对文件路径>-<函数名>)
        ids_in_graph.append(f"{caller_src}-{caller}")
        # check if file summary in api_summary
        # 若文件有摘要，则构造文件级实体节点
        file_name = os.path.basename(caller_src)
        if file_name in api_summary:
            file_summary = api_summary[file_name]["file_summary"]
            file_proerties = {"file summary":file_summary}
        else:
            file_proerties = {}
        # 构建文件的 EntityNode
        caller_file_entity = EntityNode(
            name=f"{caller_src}",
            label="File",
            properties=file_proerties
        )
        # 源文件节点 加入图谱的 id 形式为 <caller_src>(文件路径)
        ids_in_graph.append(f"{caller_src}")
        # add entity calee funciton node 
        # 构造 callee 的属性 ，与 caller 逻辑一致
        if callee_src_code.strip():
            callee_properties = {"signature": callee_signature, "file": callee_src, "source code": f"```code\n{callee_src_code}\n```"}
        else:
            callee_properties =  {"signature": callee_signature, "file": callee_src}
        callee_summary = check_funciton_has_summary(callee_src, callee, api_summary)
        if len(callee_summary):
            callee_properties['summary'] = callee_summary
        callee_entity = EntityNode(
            name=f"{callee_src}-{callee}",
            label=callee_label,
            properties=callee_properties
        )
        # check if file summary in api_summary
        file_name = os.path.basename(callee_src)
        if file_name in api_summary:
           file_summary = api_summary[file_name]["file_summary"]
           file_proerties = {"summary":file_summary}
        ids_in_graph.append(f"{callee_src}-{callee}")
        callee_file_entity = EntityNode(
            name=f"{callee_src}",
            label="File",
            properties=file_proerties
        )
        ids_in_graph.append(f"{callee_src}")
        # 汇总到 entities ，所有实体节点列表
        entities.extend([caller_entity, callee_entity,caller_file_entity, callee_file_entity]) # Step 5: 创建调用者和被调用者之间的关系(使用边来表示)
        # Step 5: Create a relation between caller and callee
        # 边类 Relation ,一个函数边(调用)，两个文件边(包含)
        relation = Relation(
            label=relationship_call,    # label(关系类型):  "CALLS" 或 "CONTAIN"
            source_id=caller_entity.id, # 起点节点的 ID
            target_id=callee_entity.id, # 终点节点的 ID
            properties={}               # 关系的附加属性
        )
        file_relation1 = Relation(
            label="CONTAIN",
            source_id=caller_file_entity.id,
            target_id=caller_entity.id,
            properties={}
        )
        file_relation2 = Relation(
            label="CONTAIN",
            source_id=callee_file_entity.id,
            target_id=callee_entity.id,
            properties={}
        )
        relations.append(relation)
        relations.append(file_relation1)
        relations.append(file_relation2) # Step 6: 将实体、关系和文本节点插入到图存储中
    # Step 5: Upsert entities, relations, and text nodes into the graph store
    graph_store.upsert_nodes(entities) # node 插入图
    graph_store.upsert_relations(relations) # relation 插入图
    # graph_store.upsert_llama_nodes(source_chunks)
    index_ids = []          # 用来存储节点的 ID，用于索引和去重。
    unique_index_ids = []   # 用来存储唯一的节点 ID，避免重复。
    file_id = []            # 用来存储文件的 ID，用来确保每个文件的总结信息仅插入一次。
    for fn_id in code_base.fn_def:
        parts = fn_id.split("-")
        fid = parts[0]
        method_name = parts[1]
        if method_name == "":
            continue
        # if method_name == "ares__buf_append":
        #     print(k)
        #     print(code_base.fn_def[k])
        #     print("dbug")
        # 获取 code 
        if code_base.fn_def[fn_id]["code"].strip():
            src_node = TextNode(
                id_= fn_id,
                text=code_base.fn_def[fn_id]["code"].strip()
            )
            # 如果该函数的 ID 已经存在于图中
            # 且该文本节点还没有被添加到 all_src_text_nodes(所有函数定义对应的文本节点) 中，则添加。
            if fn_id in ids_in_graph and src_node not in all_src_text_nodes:
                all_src_text_nodes.append(src_node)
                index_ids.append(fn_id)
            # 获取文件名
        file_name = os.path.basename(fid)
        # 如果文件名存在于 project/api_summary/api_with_summary.json 中
        if file_name in api_summary:
            for fn_name in api_summary[file_name]:
                if fn_name == "file_summary":
                   sum_node = TextNode(
                        id_= fid,
                        text=api_summary[file_name][fn_name]
                    )
                   if sum_node.id_ in ids_in_graph and sum_node.id_ not in file_id:
                        file_summary_nodes.append(sum_node)
                        file_id.append(sum_node.id_)
                else:
                    sum_node = TextNode(
                        id_= f"{fid}-{fn_name}",
                        text=api_summary[file_name][fn_name]
                    )
                if sum_node.id_ in ids_in_graph and sum_node not in summary_api_nodes:
                        summary_api_nodes.append(sum_node)
                    index_ids.append(sum_node.id_)
                for fn_name in api_summary[file_name]:
                if fn_name == "file_summary":
                    continue
                if f"{fid}-{fn_name}" in code_base.fn_def:
                    api_src_code_node = TextNode(
                        id_= f"{fid}-{fn_name}",
                        text=code_base.fn_def[f"{fid}-{fn_name}"]["code"].strip()
                    )
                    # assert api_src_code_node.id_ not in unique_index_ids
                    unique_index_ids.append(api_src_code_node.id_)
                    # else:
                    #     print(f"Debug {fid}-{fn_name}")
                    if api_src_code_node.id_ in ids_in_graph and api_src_code_node not in API_src_text_nodes:
                        API_src_text_nodes.append(api_src_code_node)
    # json.dump(ids_in_graph, open("ids_in_graph.json", "w"), indent=2)
    # json.dump(index_ids, open("index_ids.json", "w"), indent=2)
    # json.dump(unique_index_ids, open("unique_index_ids.json", "w"), indent=2)
    # logger.info(f"Number of API_src_text_nodes: {len(API_src_text_nodes)}")
    # logger.info(f"Number of len API_src_text_nodes: {len(list(set([n.id_ for n in API_src_text_nodes])))}")
    return graph_store, entities, all_src_text_nodes, summary_api_nodes,  file_summary_nodes, API_src_text_nodes
```
