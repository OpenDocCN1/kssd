
# 万能五笔 6.46 版 `CopySo` 去除 程序修改分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-417.htm`](https://www.pediy.com/kssd/pediy07/pediy7-417.htm)

我就是从 PEDIY 开始学习软件破解的。在此首先表示感谢！

本人刚才开始学习软件破解，在此之前，做过新概念英语学习机的破解补钉和内存注册机（只是自己用啊）。这次修改万能五笔，主要目的是：

* 学习修改软件
* `CopySo` 实在有点烦：有时拖动输入法框，却点开了 `CopySo`。而且 `CopySo` 本身也是去调用其它搜索软件的，有点多此一举了。本人喜欢简洁至上。

修改过程如下：

工具：`OllyDBG`，`Reshacker`（`EXESCOPE`），`UltraEdit`

### 1. `CopySo` 去除

打开 `CopySo` 时是进行外部调用，一般用 `shellexecutea` 或 `createprocessa`。

打开 `OllyDBG`，调入 `wnwb.exe`。

下断：`bpx shellexecutea`。

跟踪点击 `CopySo`、五笔网站、五笔邮件：三个按钮，将断下来的 `call dword ptr ds:[<&SHELL32.ShellExecuteA>]` 修改为 `nop`。

注：本机跟踪 `shellexecutea` 时，`OD` 老是死了，所以干脆将所有调用 `shellexecutea` 的全部改为 `nop` 了。反正输入法就是输入法，也不需要它去调用其它东东或上网。

一测试，还会调用 `CopySo`。

再下断：`bpx createprocessa`，跟踪点击 `CopySo`，在中断处的 `call createprocessa`，改为 `nop`。

再测试，发现在无 `wnwbio.exe` 文件时，不会启动 `CopySo` 了，但有 `wnwbio.exe` 文件时，还是会启动 `CopySo`。

调入 `wnwbio.exe`，下断：`bpx shellexecutea`，只找到一处，改为 `nop`。

这下，`CopySo` 全去除了。

### 2. 去除加在“快速启动”和桌面上的 `copyso` 连接

当无 `wnwb.ini` 文件时，启动万能五笔后，会自动在“快速启动”和桌面上的加上 `copyso` 快捷方式。

对有关文件操作的 API 下断点后，未能跟踪到。

不行，就换个办法了：

查找 ASCII 码：`copyso.lnk` 和 `wnwb.lnk`，找到后，查看后面附近的代码，将其后面有关 `call` 的代码全部下断，发现在 `442E79` 和 `442E96`、`442F8B` 均有 `call 45c3d0`，此 `call` 运行后，就创建快捷方式了。

这一段程序是子程序，是在主程序中调用。主程序调用两次，一次是建立 `WNWB` 的快捷方式，第二次是建立 `COPYSO` 的快捷方式。由于两个都不需要，所以直接将 `442E79` 和 `442E96`、`442F8B` 改为 `nop` 就行了。

### 3. 输入法界面上“`COPYSO`”“网”“信”按钮的去除

这些显示实际是位图，也可以用 `Reshacker` 删除相应位图。但删除后，有时会显示一个黑块。这里仍然在程序中去除相关显示的代码。

先取消皮肤显示。

然后下断：`bpx bitblt`。

运行程序，可以实现单步显示位图。但 `call bitblt` 是在一个子程序中，无法定位每一个位图的调用位置。因此顺着 `call bitblt` 回查代码，找到子程序入口：`4412AE`。

对 `call 4412AE` 进行下断：鼠标右键/搜索/所有命令，输入 `call 4412AE`，对找到的所有命令设置断点。

按 `F9`，一步一步运行，确定那一个调用是显示相应位图的。确认完后，将该调用改为 `nop`。

注意：不同的输入法窗口类型显示位图的调用位置不同，要对每一个对应输入法窗口类型进行跟踪。

### 4. `COPYSO` 提示信息去除

在上图所示的 2 中，会显示有关 `COPYSO` 的提示信息，当然，其中也会显示一些五笔使用技巧。

用 `Reshacker`（`EXESCOPE`）打开 `wnwb.exe`，查看位图，将关于五笔使用技巧的位图导出。再查到有关 `COPYSO` 的提示信息，将其更换为五笔使用技巧位图即可。

![](img/a2004b857b689b5af00a39c1e7db9a70.jpg)

### 5. 鼠标悬停提示去除

虽然去掉了 `COPYSO`，五笔“网”“信”等显示，但鼠标悬停时仍会显示如上图所示 2 的提示。

对 `GetClientRect` 下断，虽然在鼠标悬停时中断了，但 `OD` 也死了。所以未能跟踪到代码。

解决办法：用 `UltraEdit`，搜索相关提示内容，找到后用 `00` 替换。

### 6. 菜单修改

用 `Reshacker` 将无用的菜单项删除即可。

### 其它

后来发现 `wnwb` 还对 `IE` 地址栏进行劫持，即对 `IE` 地址栏中的内容进行监控，若不是有效 URL，就将地址栏中的内容改为 `go.copyso.com/.....`。以 `IEFRAME` 为关键词，找到相关代码，将调用代码 `NOP` 就行了。

然后，它还在注册表中写入劫持 `IE` 地址栏插件的相关注册表项：

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\TypeLib\{C0CDC83C-FEA7-499F-9BE7-A9AB4EAED292}]`

`[HKEY_CLASSES_ROOT\CLSID\{ED8DFC5C-10EF-45AB-9DC2-0639AFF5A270}]`

以 `ED8DFC5C-10EF-45AB-9DC2-0639AFF5A270` 为关键词，找到相关代码，将调用代码 `NOP` 就行了。

### 问题

在 `OD` 中，对 `shellexecutea` 等下断后，运行程序，点击后中断，此时显示 `ID XXXXXX` 新的进程已创建，然后 `OD` 就死了。

这个不知是什么问题，请大家指教啊。

下载：[`mywb.ys168.com`](http://mywb.ys168.com)

`4dd466d0e1e0cc123afbe504da2462ca`
