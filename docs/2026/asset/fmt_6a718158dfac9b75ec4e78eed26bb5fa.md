
### get_codebase

`get_codebase` 主要就是 `CodeRepository.construct_nodes_fn_doc` 的封装。

```
def get_codebase(src_api_file, project_name="c-ares", exclude_folder_list=["c-ares/src/tools"]) -> CodeRepository:
    method_code = json.load(open(src_api_file, "r"))
    pkg = CodeRepository(project_name)
    file_id_dict, (struct_def, typdef_struct_alias), (enum_def, typdef_enum_alias), fn_def = pkg.construct_nodes_fn_doc(method_code, exclude_folders=exclude_folder_list)
    pkg.file_id_mapping = file_id_dict
    pkg.struct_def = struct_def
    pkg.typdef_struct_alias = typdef_struct_alias
    pkg.enum_def = enum_def
    pkg.typdef_enum_alias = typdef_enum_alias
    pkg.fn_def = fn_def
    pkg.graphs = [] #graphs
    return pkg
```

### construct_nodes_fn_doc

**从 API 数据（源码和头文件）中提取结构化的节点信息，包括文件、函数、结构体、枚举等，为后续构建图谱提供语义实体**。

「1」 获取源文件和头文件信息，获取项目路径前缀（这里是我Patch的部分，因为当路径为绝对地址，按照源代码的相对地址就会替换失败，导致后面全部提取失败，提取前缀增加兼容性。后续前缀会被替换为空，然后再将数据集合。）

「2」 遍历所有源文件和头文件，并读取文件内容（读取的时候做了编码兼容），使用 `replace()` 替换前缀（PS：这点很重要影响后续整个流程），构建文件节点字典 `file_id_dict`。

「3」 调用 `process_source_files()` 提取源代码中的函数定义、全局变量、结构体、枚举节点（`src_fn_def_list_list, src_global_var_node_list, src_struct_node_list, src_enum_node_list`）。

「4」 构建 **结构体、枚举** 定义与别名映射，最终两个字典形如：`struct_def: struct 名字 -> struct 定义节点`，`typdef_struct_alias: struct typedef别名 -> 原始 struct 名`。

「5」 构建函数定义映射 `fn_dec`，每个函数以 “**文件路径-函数名**” 为键，保存其定义信息。

```
# 构建代码图中的各种节点信息（函数、结构体、枚举、文件等）。
def construct_nodes_fn_doc(self, api_data, exclude_folders = []):
    '''
        构建代码图中的各种节点信息（函数、结构体、枚举、文件等）。
        节点ID的格式:
            - project_name/file_path-method_name-param_name
            - project_name/file_path-struct_name-param_name
            - project_name/file_path-method_name-statement_id
        参数：
            - api_data: 包含源代码和头文件信息的字典，格式：
                {
                    "src": {文件路径: 函数定义},
                    "head": {头文件路径: 函数定义}
                }
            - exclude_folders: 排除分析的文件夹列表
        返回：
            - file_id_dict: 文件节点的元数据字典
            - (struct_def, typdef_struct_alias): 结构体定义和typedef结构体别名映射
            - (enum_def, typdef_enum_alias): 枚举定义和typedef枚举别名映射
            - fn_dec: 函数定义信息字典
    '''
    source_code = api_data["src"]
    header_code = api_data["head"]
    api_path = list(source_code.keys())[0]
    api_path_prefix_len = api_path.find(f"{self.project_name}") + len(f"{self.project_name}")
    api_path_prefix = api_path[:api_path_prefix_len]
    file_id_dict = {}
    # 遍历 project/codebase/api/src_api.json 中所有 keys -> 构造文件节点的元数据字典 file_id_dict
    for idx, all_files in enumerate([list(source_code.keys()), list(header_code.keys()) ]):
        for fname in all_files:
            logger.info(f"Processing file: {fname}")
            # 读取文件内容，做了编码处理
            try:
                with open(fix_file_path(fname), "r", encoding='utf-8') as f:
                    file_code = f.read()
            except UnicodeDecodeError:
                with open(fix_file_path(fname), "rb") as f:
                    raw_data = f.read()
                detected_encoding = chardet.detect(raw_data)['encoding']
                logger.info(f"Detected encoding for {fname}: {detected_encoding}")
                try:
                    file_code = raw_data.decode(detected_encoding)
                except:
                    logger.error(f"Failed to decode {fname} with detected encoding {detected_encoding}. Skipping this file.")
                    continue
            # Patch
            #fid = fname.replace(f"../docker_shared/source_code/{self.project_name}", "")
            fid = fname.replace(api_path_prefix, "")
            # 构建该文件的节点元数据
            metadata_file_node = { "id":fid, "file_name":os.path.basename(fname), "file_path":fid, "project":self.project_name, "code":file_code}
            file_id_dict[fid] = metadata_file_node

            # 提取源代码中的函数定义、全局变量、结构体、枚举节点
            src_fn_def_list_list, src_global_var_node_list, src_struct_node_list, src_enum_node_list = self.process_source_files(source_code, exclude_folders)

            head_fn_def_list_list, head_global_var_node_list, head_struct_node_list, head_enum_node_list = self.process_source_files(header_code, exclude_folders)

            ### 构建结构体定义与 typedef 映射 ###
            struct_def = {} # struct 名字 -> struct 定义节点
            typdef_struct_alias = {}  # struct typedef别名 -> 原始 struct 名
            # struct_declar = {}
            for struct in head_struct_node_list + src_struct_node_list:
                if len(struct['name'].strip()) != 0 and len(struct['parameters']) != 0:
                    struct_def[struct['name']] = struct # 有名且有字段的定义
                else:
                    # 处理 typedef struct 形式
                    if struct["code"].startswith("typedef"):
                        alias_name, name = struct["code"].split()[-1], struct["code"].split()[-2]
                        typdef_struct_alias[alias_name] = name
            # 与 struct 逻辑一致
            enum_def =  {}
            typdef_enum_alias = {}
            # enum_declar = {}
            for enum in head_enum_node_list + src_enum_node_list:
                if len(enum['name'].strip()) != 0 and len(enum['parameters']) != 0:
                    enum_def[enum['name']] = enum
                else:
                    if enum["code"].startswith("typedef"):
                        alias_name, name = enum["code"].split()[-1], enum["code"].split()[-2]
                        typdef_enum_alias[alias_name] = name

            fn_dec = {}
            for fdef in src_fn_def_list_list + head_fn_def_list_list:
                fn_dec[ f"{fdef['fid']}-{fdef['name']}"] = fdef # 文件id-函数名 -> 函数定义

            return file_id_dict, (struct_def, typdef_struct_alias), (enum_def, typdef_enum_alias), fn_dec
```

### process_source_files

**从预处理后的代码结构信息中提取并标准化出“节点”数据（函数、结构体、枚举、全局变量）**，为后续图谱建模准备实体数据。

「1」 提取项目路径前缀（这里是我Patch的部分，因为当路径为绝对地址，按照源代码的相对地址就会替换失败，导致后面全部提取失败，提取前缀增加兼容性。后续前缀会被替换为空，然后再将数据集合。）

「2」 遍历所有文件解析每个文件的节点并跳过指定排除的文件夹

「3」 构建 **函数、结构体、枚举、全局变量** 4 个节点信息（重点），标准化成 `{fid}-{函数名}` 的唯一 ID，附带代码和参数信息。
