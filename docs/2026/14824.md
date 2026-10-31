
# 通过bilibili分析libmsaoaidsec.so的环境检测

之前在论坛看到这个帖子`https://bbs.kanxue.com/thread-277034.htm`

通过`nop`的方式过掉了`frida`检测，一时兴起，想要分析一下具体的检测逻辑

样本版本`7.26.1`，包名`tv.danmaku.bili`

---

通过上文楼主给出的方法定位了检测函数的地址，再通过`IDA`进行调试，且在调试的时候需要先在静态文件中把断点下好，这里就不再赘述调试过程，直接上分析结论

# 一、offset=0x10975处 --检测线程1

存在三个检测方法

![图片描述](upload/attach/202308/945828_DCCHGGTANJ5HWX9.png)

## 1.1 通过/proc/%d/status 读取当前进程ptracerPid 穿入第二个方法作为参数

![图片描述](upload/attach/202308/945828_2NNG2GJ5UME3JP8.png)

## 1.2 查看ptraceId对应的进程信息，读取了/proc/ptracerPid/status 获取PPid，与当前应用进程号做对比，判断是否为应用主进程的子进程进行了ptrace操作

![图片描述](upload/attach/202308/945828_53C4HGMGWZX62FN.png)

## 1.3 遍历/proc/self/task/xx/stat文件，读取当前线程状态是否为tracing

![图片描述](upload/attach/202308/945828_DHC5ADE4DP8KCB6.png)

实际过程中`stat`的输出格式是 `pid (name) status xxxxxxx`

该方法就是判断读取到的`status`是否为`“T”`（会通过 `|0x20` 将其转为`“t”`,ASCII码为`116`）

## 1.4 （猜测）检测到异常后的处理方式-等待其他大佬指正~

处理代码未作详细分析，这里只说说我根据一些字符串的猜测：

在上述三个方法检测到异常后，会调用`JNI_GetCreatedJavaVMs`方法

![图片描述](upload/attach/202308/945828_VW59PXGRJMVNRRR.png)

而这个方法是获取已经创建的 Java 虚拟机实例，根据“使用`frida-spwan`模式`hook`会导致应用短暂卡顿后，`frida`进程退出，应用恢复运行”的现象，我猜测是把被`hook`进程干掉了，以一个新进程运行了上面的Java 虚拟机实例。

# 二、offset=0x11129处 --检测线程2

该方法前半部分是各种字符串的解密，最后的四个方法才是检测

![图片描述](upload/attach/202308/945828_XJTPR6MX8EEGXFK.png)

## 2.1 检测 proc/self/task/%s/status 特征字符串 gum-js-loop gmain

![图片描述](upload/attach/202308/945828_QWYZ6SKGT2S7U6P.png)

## 2.2 检测/proc/self/fd/%s 特征字符串 injector

![图片描述](upload/attach/202308/945828_9TB8WM3AXDRRMN4.png)

## 2.3 检测/proc/self/maps 特征字符串

检测`/data/local/tmp` 及对应文件的权限 `‘x’` `‘r’`

![图片描述](upload/attach/202308/945828_S8TCQC2PVQTJR5F.png)

对命中的文件进行解析，检查 `_AGENT_1.0` `frida-agent`特征

![图片描述](upload/attach/202308/945828_97NSNXQT2RDPJJC.png)

PS：通过`010 editor` 打开`fridaserver`，可发现对应特征

## 2.4 检测 libart.so的inline hook特征

参考了这篇文章`https://bbs.kanxue.com/thread-268586.htm`

解析`libart.so` 中的`_ZN3art9ArtMethod12PrettyMethodEb`，是否存在`inlinehook`特征

读取`libart.so-_ZN3art9ArtMethod12PrettyMethodEb`

![图片描述](upload/attach/202308/945828_86PAA3VS9VG28FN.png)

确定`hook`特征后 通过执行`svc`执行进行退出（具体执行会动态解密）

![图片描述](upload/attach/202308/945828_CS3G686YJVGW4S6.png)

## 2.5 检测到异常后的退出逻辑

在上述检测逻辑后，部分使用的是直接的函数调用`exit(0)`，另外一部分是使用`svc`调用 `exit_group`，所以仅`hook` `libc`的`exit`函数是无法做到绕过检测的。

# 三、其他

## 3.1 字符串的解密

上述所有方法中的检测字符串都是通过动态解密释放的

以下是我的解密方法

![图片描述](upload/attach/202308/945828_5B7VVXRAWT3XBRP.png)

# 四、总结

1. 这个样本的检测方法`trace`特征检测、`frida`特征字符串检测、`inlinehook`特征检测
2. 字符串比较方式是直接调用的`strstr`或`strcmp`方法
3. 检测到异常的退出方法是直接调用`exit`、执行`scv`指令调用`exit_group`

小白第一次发帖，可能分析和描述中存在错漏，望大佬指点~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2023-8-7 15:47

被菜鸟泽编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
