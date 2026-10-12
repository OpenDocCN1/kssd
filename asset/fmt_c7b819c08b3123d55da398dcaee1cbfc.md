
`unhook_from_notification_point(HT_DBG, dbg_callback, NULL);`

### 4.5.2 UI 事件通知

`ui_notification_t` 是定义在 `kernwin.hpp` 中的一个枚举，并包括了所有的，可以由 IDA 或者插件产生的用户界面事件通知。要注册获取这些事件通知，您必须将 `hook_to_notification_point()` 的第一个函数设置为 `HT_UI`。

下面的两个列表给出了一些可以由插件接收或产生的事件通知。这些只是全部事件通知的一个子集；列出来的这些比较通用一点。

尽管这些通知可以由插件调用 `callui()` 来产生，但还有一些助手函数 (helper function) 也有同样的功能，这意味着您无需使用 `callui()`，而调用助手函数也可以达到相同目的。

| 事件通知 | 描述 | 助手函数 |
| :--- | :--- | :--- |
| `ui_jumpto` | 移动光标到某地址 | `jumpto` |
| `ui_screenea` | 返回光标位置的当前地址 | `get_screen_ea` |
| `ui_refresh` | 刷新所有反汇编界面 | `refresh_idaview_anyway` |
| `ui_mbox` | 给用户显示一个消息框 | `vwarning`, `vinfo` 等等 |
| `ui_msg` | 在 IDA 的日志窗口显示一些文本 | `deb`, `vmsg` |
| `ui_askyn` | 显示有 Yes 和 No 选项的消息框 | `askbuttons_cv` |
| `ui_askfile` | 提示用户输入文件名 | `askfile_cv` |
| `ui_askstr` | 提示用户输入一个单行的字符串 | `vaskstr` |
| `ui_asktext` | 提示用户输入一些文本 | `vasktext` |
| `ui_form` | 显示一个表格（很灵活） | `AskUsingForm_cv` |
| `ui_open_url` | 在 web 浏览器中打开一个指定的 URL | `open_url` |
| `ui_load_plugin` | 加载插件 | `load_plugin` |
| `ui_run_plugin` | 运行插件 | `run_plugin` |
| `ui_get_hwnd` | 获取 IDA 窗口的 `HWND` (窗口句柄) | 暂无 |
| `ui_get_curline` | 获取着色的反汇编信息 | `get_curline` |
| `ui_get_cursor` | 获取当前光标位置的横纵坐标 | `get_cursor` |

下面的事件通知由插件接收，并交付给您的回调函数处理。

| 事件通知 | 描述 |
| :--- | :--- |
| `ui_saving` 和 `ui_saved` | 分别表示，IDA 正在保存或者已经保存完数据库 |
| `ui_term` | IDA 已经关闭数据库 |

比如，下面的代码将产生一个 `ui_screenea` 事件通知，并用 `ui_mbox` 事件通知在 IDA 对话框中显示结果。

```
void IDAP_run(int arg)
{
    ea_t addr;
    va_list va;
    char buf[MAXSTR];
    // 获取当前光标的虚拟地址，并保存到 addr 变量
    callui(ui_screenea, &addr);
    qsnprintf(buf, sizeof(buf)-1, "Currently at: %a\n", addr);
    // 在消息框中显示相关信息
    callui(ui_mbox, mbox_inf, buf, va);
    return;
}
```

上面这种情况，您一般应该使用助手函数，这里使用 `callui()` 只不过是演示的目的。

### 4.5.3 调试器事件通知

调试器事件通知分为三种类型，底层型，高层型和函数返回事件通知；它们之间的不同点，将在接下来的章节解释清楚。上面说的的这些事件通知，都属于 `dbg_notification_t` 枚举，它定义在 `dbg.hpp` 头文件中。如果您传递给 `hook_to_notification_point()` 一个 `HT_DBG` 参数，那么，在 IDA 调试进程的时候，下面这些事件通知将被传给您的插件。

#### 4.5.3.1 底层型事件

下面这些事件摘自 `dbg_notification_t`，都表示底层型事件。底层事件通知都由调试器生成。

| 事件通知 | 描述 |
| :--- | :--- |
| `dbg_process_start` | 进程启动 |
| `dbg_process_exit` | 进程终止 |
| `dbg_library_load` | 加载了库文件 |
| `dbg_library_unload` | 卸载了库文件 |
| `dbg_exception` | 产生异常 |
| `dbg_breakpoint` | 非用户定义的断点被激活 |

您能够使用 `debug_event_t` 结构 (`idd.hpp`) 获取更多关于调试器事件通知的信息，通常是提供给 `va` 参数到您的回调函数（仅支持底层函数事件）。下面是 `debug_event_t` 结构的全貌。

```
struct debug_event_t
{
    event_id_t eid;       // 事件代码（常用于解释 ‘info’联合）
    process_id_t pid;     // 事件发生的进程
    thread_id_t tid;      // 事件发生的线程
    ea_t ea;              // 事件发生的地址
    bool handled;         // 事件是否由调试器处理?
                          // (从系统的角度看)
    // 右边的注释说明了 eid 值
    // 与联合成员的设置有对应关系。
    union
    {
        module_info_t modinfo;  // PROCESS_START, PROCESS_ATTACH,
                                // LIBRARY_LOAD
        int exit_code;          // PROCESS_EXIT, THREAD_EXIT
        char info[MAXSTR];     // LIBRARY_UNLOAD (卸载的库文件名称)
                                // INFORMATION (若有信息，将被显示在消息窗口)
        e_breakpoint_t bpt;     // BREAKPOINT (非用户定义)
        e_exception_t exc;      // EXCEPTION
    };
};
```

比如，如果您的回调函数接收到 `dbg_library_load` 事件通知，您就能检查 `debug_event_t` 的 `modinfo` 成员，以确定加载了什么文件：

```
...
// 我们的回调函数要处理 HT_DBG 事件通知
static int idaapi dbg_callback(void *udata, int event_id, va_list va)
{
    // va 包含了一个 debug_event_t 指针
    debug_event_t *evt = va_arg(va, debug_event_t *);
    // 若事件是 dbg_library_load，我们就知道 modinfo 将被填充，
    // 而且，包含了加载的库文件名称
    if (event_id == dbg_library_load)
        msg("Loaded library, %s\n", evt->modinfo.name);
    return 0;
}

// 我们的 init 函数
int IDAP_init(void)
{
    // 注册通知点为我们的回调函数
    hook_to_notification_point(HT_DBG, dbg_callback, NULL);
    ...
}
```

#### 4.5.3.2 高层型事件通知

下面的这些事件摘自 `dbg_notification_t`，它们都是高层型事件通知，由 IDA 内核产生。

| 事件通知 | 描述 |
| :--- | :--- |
| `dbg_bpt` | 用户定义的断点被激活 |
| `dbg_trace` | 一条指令被执行（需要允许单步跟踪） |
| `dbg_suspend_process` | 进程已经被暂停 |
| `dbg_request_error` | 请求的时候，产生一个错误（参看 5.14 章节） |

每个这样的事件通知都有不同的参数，并复制给 `va` 参数，供您的回调函数使用。但没有提供 `debug_event_t`，如底层型事件通知那样。

`dbg_bpt` 事件通知与受影响的线程的线程 ID (`thread_id_t`)，和被激活断点的地址，一起复制到 `va` 参数。下面的例子在用户断点被激活的时候，显示一些信息到 IDA 的日志窗口。

```
...
int idaapi dbg_callback(void *udata, int event_id, va_list va)
{
    // Only for the dbg_bpt event notification
    if (event_id == dbg_bpt)
    {
        // 获取线程 ID
        thread_id_t tid = va_arg(va, thread_id_t);
        // 获取被激活断点的地址
        ea_t addr = va_arg(va, ea_t);
        msg("Breakpoint hit at: %a, in Thread: %d\n", addr, tid);
    }
    return 0;
}

int IDAP_init(void)
{
    hook_to_notification_point(HT_DBG, dbg_callback, NULL);
    ...
}
```

#### 4.5.3.3 函数返回型通知

最后一节里，将详细讨论同步和异步调试器函数；到目前为止，所有您需要知道的同步调试器函数只是一些普通函数 – 您调用它们，它们则完成一些事情，然后返回。而异步函数呢，在调用后就立即返回，高效地把请求放进队列，然后再后台运行。当任务完成，一个表示原来请求完成的事件通知就产生了。

下面都是函数返回型通知。

| 事件通知 | 描述 |
| :--- | :--- |
| `dbg_attach_process` | 调试器附加到进程 (IDA 4.8) |
| `dbg_detach_process` | 调试器离开于进程 (IDA 4.9) |
| `dbg_process_attach` | 调试器附加了一个进程 (IDA 4.9) |
| `dbg_process_detach` | 调试器离开了一个进程 (IDA 4.9) |
| `dbg_step_info` | 调试器步入一个函数 |
| `dbg_step_over` | 调试器步过一个函数 |
| `dbg_run_to` | 调试器已经运行到用户的光标位置 |
| `dbg_step_until_ret` | 调试器已经返回到调用者的位置 |

下面的例子在 `IDAP_run()` 中让 IDA 附加到一个进程。一旦附加成功，IDA 产生一个事件通知，`dbg_attach_process`，并由 `dbg_callback` 回调函数所处理。

```
...
int idaapi dbg_callback(void *udata, int event_id, va_list va)
{
    // 获取被附加进程的 ID
    process_id_t pid = va_arg(va, process_id_t);
    // 若您使用 IDA 4.9, 更改 dbg_attach_process
    // 为 dbg_process_attach
    if (event_id == dbg_attach_process)
        msg("Successfully attached to PID %d\n", pid);
    return 0;
}

void IDAP_run(int arg)
{
    int res;
    // 附加到一个进程，用法参看第五章
    attach_process(NO_PROCESS, res);
    return;
}

int IDAP_init(void) {
    hook_to_notification_point(HT_DBG, dbg_callback, NULL);
    ...
}
```

### 4.6 字符串

我们可以用 SDK 来访问 IDA 中的字符串窗口，而每个二进制文件（被打开的文件）的字符串由 `string_info_t` 结构表示，该结构定义在 `strlist.hpp` 头文件中。下面是该结构的部分定义。

```
struct string_info_t
{
    ea_t ea;       // 字符串的地址
    int length;    // 字符串长度
    int type;      // 字符串类型 (0=C 语言, 1=Pascal, 2=Pascal 2 字节
                   // 3=Unicode, etc.)
    ...
};
```

请注意，上面的结构并不含字符串。要获取字符串，您需要使用 `get_bytes()` 或者 `get_many_bytes()` 来从二进制文件中提取。要得到所有有效的字符串，可以使用如下代码：

```
// 遍历所有字符串
for (int i = 0; i < get_strlist_qty(); i++) {
    char string[MAXSTR];
    string_info_t si;
    // 获取字符串项目
    get_strlist_item(i, &si);
    if (si.length < sizeof(string)) {
        // 从二进制文件中得到字符串
        get_many_bytes(si.ea, string, si.length);
        if (si.type == 0) // C 字符串
            msg("String %d: %s\n", i, string);
        if (si.type == 3) // Unicode
            msg("String %d: %S\n", i, string);
    }
}
```

上面的一些函数将在第五章《函数》中，详细讨论。

# 第五章 函数

这一章，将以导出的 IDA SDK 函数的不同作用，来分类进行描述。从最简单的开始，然后使用更复杂一些的函数。还将提供使用这些函数的简单例子，而且在第六章《实例代码》中，将提供更多的内容。显然，这里并不是一个完整的介绍（更完整的请参阅 SDK 中的头文件），但总的来看，仍然是一些很有用的函数。

例子的重要事项：下面所有的函数，都可以在 `IDAP_run()`，`IDAP_init()` 或 `IDAP_term()` 函数中使用，除非有特殊情况。下面的任何一个例子，都可以粘贴到 3.4 章节中插件模板的 `IDAP_run()` 函数中，而且应该能正常运行。每个函数或例子需要的头文件都会写明。

## 5.1 常用函数的替代

IDA 提供了很多常用 C 标准库函数的替代调用。推荐您用下面这些替代函数，而不是 C 标准库函数。在 IDA 4.9 中，许多 C 标准库函数都不再有效，您应该使用 IDA 的替代函数。

| C 库函数 | IDA 替代函数 | 定义在 |
| :--- | :--- | :--- |
| `fopen`, `fread`, `fwrite`, `fseek`, `fclose` | `qfopen`, `qfread`, `qfwrite`, `qfseek`, `qfclose` | `fpro.h` |
| `fputc`, `fgetc`, `fputs`, `fgets` | `qfputc`, `qfgetc`, `qfputs`, `qfgets` | `fpro.h` |
| `vfprintf`, `vfscanf`, `vprintf` | `qfprintf`, `qfscanf`, `qvprintf` | `fpro.h` |
| `strcpy`, `strncpy`, `strcat`, `strncat` | `qstrncpy`, `qstrncat` | `pro.h` |
| `sprintf`, `snprintf`, `wsprintf` | `qsnprintf` | `pro.h` |
| `open`, `close`, `read`, `write`, `seek` | `qopen`, `qclose`, `qread`, `qwrite`, `qseek` | `pro.h` |
| `mkdir`, `isdir`, `filesize` | `qmkdir`, `qisdir`, `qfilesize` | `pro.h` |
| `exit`, `atexit` | `qexit`, `qatexit` | `pro.h` |
| `malloc`, `calloc`, `realloc`, `strdup`, `free` | `qalloc`, `qcalloc`, `qrealloc`, `qstrdup`, `qfree` | `pro.h` |

强烈推荐您使用上面这些替代函数，但是如果您在写一个老版本上面的插件，而且因为一些原因要用到 C 标准库函数，您可以用 `-DUSE_DANGEROUS_FUNCTIONS` 或 `-DUSE_STANDARD_FILE_FUNCTIONS` 预定义来进行编译。

## 5.2 消息框
