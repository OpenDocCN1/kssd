
引擎地址：[Releases · github/codeql-cli-binaries](https://bbs.kanxue.com/elink@3dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3V1k6U0L8$3c8W2M7h3I4Q4x3X3c8U0L8r3W2Q4x3X3c8T1K9h3&6S2M7X3W2W2M7#2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6y4r3M7r3q4Y4k6g2)9K6c8o6V1`.)

SDK地址：[Release codeql-cli/v2.7.3 · github/codeql](https://bbs.kanxue.com/elink@892K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3V1k6U0L8$3c8W2M7h3I4Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5c8Y4c8S2k6#2)9J5c8X3y4G2k6r3g2I4L8q4)9J5k6r3y4D9K9g2)9J5y4e0u0r3N6U0u0Q4x3X3f1%4i4K6u0W2x3H3`.`.)

当 SDK+引擎 都降级后才能使用源代码正常执行，否则需要重写 ql 很麻烦。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_ZHX7ZTS5R7KMB8S.jpg "null")

`extract_src_test_api_call_graph`

作用：提取源码和测试代码中的 API 调用图(Call Graph)

``` 
    # read the function name and its source code name from the returned dict of extract_api_from_head     def extract_src_test_api_call_graph(self, data: Dict, pool_num=10) -> Dict:         """             提取源码和测试代码中的 API 调用图(Call Graph)             支持多线程并行提取，每个线程用一份数据库副本。         ToDO: multple thread SUPPORT, need to keep the copy database for each thread         Extracts the source and test API information from the repository.         """         logger.info("Extracting source and test API information from the repository.")         src_api = []         # 每个文件中提取出 fn_def_list(函数定义列表)         for src_file in data['src']:             fn_def_list = data['src'][src_file]['fn_def_list']             #  遍历函数定义，取出每个函数的名字             for item in fn_def_list:                 fn_name = item['fn_meta']['identifier']                 # 原始路径替换为容器内的路径 /src/，供 docker 中的 codeql 使用                 src_api.append((fn_name, src_file.replace(f'{args.shared_llm_dir}/source_code/', '/src/')))                 # (函数名, 文件路径) example    ->    ('jni_get_class', '/src/c-ares/src/lib/ares_android.c')                  '''             构造一个任务列表 eggs,包含每个函数所需的信息             [函数名,             文件路径,             CodeQL 数据库路径,             输出结果文件夹路径,             LLM 工作目录路径]         '''         eggs = [ (api[0].strip(), api[1].strip(), self.database_db, self.output_results_folder, self.shared_llm_dir) for api in src_api ]         logger.info(f"Total number of API to be processed: {len(eggs)}")         logger.info("Copy Database for each thread.")         # 多线程处理，为每个线程拷贝一份数据库         # docker_shared/codeqldb/project ->(copy)  docker_shared/codeqldb/project_n          for i in tqdm(range(pool_num)):             shutil.copytree(self.database_db, f'{self.database_db}_{i}', dirs_exist_ok=True)             queue_id.put(i)         # 并发运行 handle_extract_api_call_graph_multiple_path 函数         with Pool(pool_num) as pool:                results = list(tqdm(pool.imap(RepositoryAgent.handle_extract_api_call_graph_multiple_path, eggs), total=len(eggs), desc='Processing transactions'))              # 删除拷贝用的临时数据库         for i in range(pool_num):             shutil.rmtree(f'{self.database_db}_{i}')
```

「1」 根据之前 `extract_api_from_head()` 提出出来的api文件，把每个文件中的`fn_def_list`(函数定义列表)提取出来。

「2」 构造一个任务列表 `eggs`,中包含每个函数所需的信息，[函数名,文件路径,CodeQL 数据库路径,输出结果文件夹路径,LLM 工作目录路径]

「3」 因为整个项目太大了，使用codeql 提取出调用关系又慢，所以必须要使用多线程，使用多线程需要为每个线程拷贝一份数据库，然后并发运行 `handle_extract_api_call_graph_multiple_path` 函数，执行完后删除拷贝用的临时数据库。

### handle_extract_api_call_graph_multiple_path

该函数作用主要就是 `_extract_call_graph()` 的封装

```
 @staticmethod     def handle_extract_api_call_graph_multiple_path(item):         global queue_id         pid = os.getpid()  # Get the current process ID         # 从 queue_id 队列中取出一个数据库副本编号（bid）。         bid = queue_id.get()         logger.info(f"============================ {pid} Consuming {bid}")         fn_name, fn_file_name, dbbase, outputfolder, shared_llm_dir = item         # 调用 _extract_call_graph 方法执行 API 调用图提取逻辑         RepositoryAgent._extract_call_graph(shared_llm_dir, fn_name, fn_file_name, f"{dbbase}_{bid}", outputfolder, bid)         queue_id.put(bid)
```

### _extract_call_graph

作用：遍历 eggs 并调用 `extract_call_graph.sh` 从中提取所有函数的调用关系。并将 bqrs 转换为 csv。

```
     @staticmethod     def _extract_call_graph(shared_llm_dir, fn_name, fn_file_name, dbbase, outputfolder, pid):         """             Extracts the call graph from the repository.             从代码仓库中提取某个函数的调用图             shared_llm_dir: 映射的主机目录根路径，存放查询脚本等。             fn_name: 函数名。             fn_file_name: 函数所在文件名。             dbbase: codeql 副本数据库的路径             outputfolder: 提取结果输出目录。             pid: 当前进程的编号         """         logger.info("Extracting call graph from the repository.")         extract_shell_script = f"{shared_llm_dir}/qlpacks/cpp_queries/extract_call_graph.sh"         pname = fn_file_name.replace('/', '_')         if os.path.isfile(extract_shell_script):             if not os.path.isfile(f"{outputfolder}/call_graph/{pname}@{fn_name}_call_graph.bqrs"):                 logger.info(f"{outputfolder}/call_graph/{pname}@{fn_name}_call_graph.bqrs")                 # convert dbbase and outputfolder to the absolute path                 dbbase = os.path.abspath(dbbase)                 outputfolder = os.path.abspath(outputfolder)                 #fn_name="ares__dns_options_free"                 logger.info(f"Extracting call graph for {fn_name} in {fn_file_name}.")                                  run_command([extract_shell_script, fn_name, fn_file_name, dbbase, outputfolder, str(pid)])                 logger.info("Call graph is converted into the csv file.")                 #fn_file="${fn_file//\//_}"             if not os.path.isfile(f"{outputfolder}/call_graph/{pname}@{fn_name}_call_graph.csv"):                     run_converted_csv(f"{outputfolder}/call_graph/{pname}@{fn_name}_call_graph.bqrs")           else:             assert False, f"Extract call graph shell script {extract_shell_script} does not exist. PWD {os.getcwd()}"
```

### extract_call_graph.sh

做了一些初始化，核心语句就三句

```
# 复制查询模板到生成的查询文件 cp "$QUERY_TEMPLATE" "$QUERY"   # 替换为实际函数名 sed -i "s/ENTRY_FNC/$fn_name/g" "$QUERY"  # 执行CodeQL查询 codeql query run "$QUERY" --database="$dbbase" --output="$outputfile"
```

执行这个脚本后就可以使用 codeql 查询所有文件中的所有函数的调用关系以供后面输出fuzz的target文件。

```
#!/bin/bash # 立即退出脚本，如果任何命令返回非零状态 set -e   # 获取当前脚本的完整路径 script_path=$(realpath "$0")   # 提取脚本所在的目录路径 script_dir=$(dirname "$script_path")   # 切换到脚本所在的工作目录 cd "$script_dir"   # 从参数中获取函数名、文件路径、数据库路径、输出文件夹和进程ID fn_name=$1  # 函数名 fn_file=$2   fn_file="${fn_file//\//_}"   # 文件路径（替换斜杠为下划线） dbbase=$3   # codeql数据库路径 outputfolder=$4  # 输出文件夹路径 pid=$5      # 进程ID # 打印相关信息 echo "Script Dir ====== $script_dir" echo "Database path: $dbbase" echo "Output folder: $outputfolder" echo "Process ID: $pid" # 检查输出文件夹是否存在，不存在则创建 [ -d "$outputfolder/call_graph" ] || mkdir -p "$outputfolder/call_graph"   # 定义输出文件路径 outputfile="$outputfolder/call_graph/${fn_file}@${fn_name}_call_graph.bqrs"   # 定义查询模板文件路径 QUERY_TEMPLATE="./extract_call_graph_template.ql"   # 定义生成的查询文件名 QUERY="call_graph_${pid}.ql"   # 打印信息：复制模板并生成查询文件 echo "Copying template and generating query file..."   # 复制查询模板到生成的查询文件 cp "$QUERY_TEMPLATE" "$QUERY"   # 使用sed命令将模板中的占位符ENTRY_FNC替换为实际函数名 sed -i "s/ENTRY_FNC/$fn_name/g" "$QUERY"   # 打印信息：运行CodeQL查询 echo "Running query: codeql query run $QUERY --database=$dbbase --output=$outputfile"   # 执行CodeQL查询 if codeql query run "$QUERY" --database="$dbbase" --output="$outputfile"; then       # 如果查询成功，打印信息：转换BQRS文件为CSV     echo "Query executed successfully. Converting BQRS to CSV."       # 定义CSV输出文件路径     csv_output="${outputfile%.bqrs}.csv"       # 将BQRS文件解码为CSV格式     if codeql bqrs decode --format=csv "$outputfile" --output="$csv_output"; then           # 如果转换成功，打印信息         echo "BQRS file successfully converted to CSV: $csv_output"     else         # 如果转换失败，打印错误信息并退出         echo "Error converting BQRS to CSV"         exit 1     fi else     # 如果查询失败，打印错误信息并退出     echo "Error executing CodeQL query"     exit 1 fi # 删除临时生成的查询文件 rm "$QUERY"
```

大致会类似执行以下语句
