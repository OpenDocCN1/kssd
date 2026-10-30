
- `load_extras`：可以通过设置`-x`选项加载任意一个dict文件（字典）或者目录
- `find_timeout`：设置超时时间
- `detect_file_args&&setup_stdio_file`：检测末尾是否有`@@`
  - 如果有`@@`，则`afl-fuzz`会从文件读取，无论是默认的`<out_dir>/.cur_input`还是用户指定的（`-f`）。
  - 如果没有`@@`但有`-f`，则表明`afl`会从用户指定的文件读取
  - 如果没有`@@`且没有`-f`，则表明从标准输入读取。
- `check_binary`：检查二进制文件的合法性
- `get_cur_time`：获取当前的准确时间
- `get_qemu_argv`：解析`qemu`的参数

例如：`./afl-fuzz -Q -i testcases/input -o findings -- ./target_binary @@`运行这个命令

那么在该函数中就会将其解析成：`./afl-qemu-trace -- ./target_binary @@`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
