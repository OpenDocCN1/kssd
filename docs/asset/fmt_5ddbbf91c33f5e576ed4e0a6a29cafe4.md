
# FLEXLM 系统高级研究(翻译)（一）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-785.htm`](https://www.pediy.com/kssd/pediy08/pediy8-785.htm)

## Advanced Study On FLEXlm System

FLEXLM 系统高级研究

2004 年 6 月 23 日，truth

这是我前篇《关于软件逆向工程》文章的续文，如果不清楚该文，请见参考文献 6。

### 问题描述

成功逆反 `IMSL CNL 5.5` 的保护后，我们关注其它受 FLEXLM 保护的软件。此类软件很多—`C++/Fortran Compiler 8.0`, `Intel MKL 5.2`, `Fluent 6.0`, `Maple 7.0`, `Gauss 4.01`—这些软件都有“`permanent uncounted HOSTID=ANY`”类型的许可。许多软件也使用他们自己的诸如序列号和密码的保护配置，破解方法从 `Keygen` 到 `Patching` 都有，例如：`Mathematica`, `SAS`, `SPSS`, `SolidEdge`, `Matlab`, `Femlab`, `MathCAD`, `Lindo`, `Origin`, `Labview`, `Tecplot`。在这里我们感兴趣的是那些使用 FLEXLM 保护而没有“`permanent uncounted HOSTID=ANY`”许可的软件，我手头有这样三个：`PGI C++/Fortran Compiler 5.1`, `Altair HyperWorks 5.0`, and `ANSYS 8.0`，其中前 2 个是 permanent，可没有 `HOSTID=ANY`，并为 counted，最后一个甚至不是 permanent 的。

现在我们使用 `PGI` 和 `HyperWorks` 并不多，而 `ANSYS` 经常使用，其 counted 许可，FLEXLM 服务端 `lmgrd.exe` 和 vendor daemon（卖方后台程序） `ansyslmd.exe` 必须先启动以供应用程序的许可校验，基于 IP/ports/sockets 的 C/S 通讯方式常使我的防火墙 `ZoneAlarm Pro 3.7` 弹出窗口，询问是否阻止/允许，这些让我非常讨厌，所以决定破解它（尽管我可以设置 `ZoneAlarm` 让 `lmgrd/ansyslmd` 每次通过防火墙，但我不想这样做）。因此这篇文章的目标就是它了—`ANSYS 8.0`。

ANSYS 有着巨多的 features，比我知道其它任何软件都多，原许可总共有 223 个，这里只能列出前头的几行。

```
SERVER changsha 000795f52adc 1055

DAEMON ansyslmd ansyslmd.exe

INCREMENT ane3fl ansyslmd 9999.9999 12-nov-2006 10 1CEF0B37317C \

VENDOR_STRING=customer:00265231 ISSUED=10-Oct-2003 \

START=10-Oct-2003

INCREMENT ansys ansyslmd 9999.9999 12-nov-2006 10 B1F027206FD0 \

VENDOR_STRING=customer:00265231 ISSUED=10-Oct-2003 \

START=10-Oct-2003

INCREMENT ane3 ansyslmd 9999.9999 12-nov-2006 10 CA5505E0FE07 \

VENDOR_STRING=customer:00265231 ISSUED=10-Oct-2003 \

START=10-Oct-2003

INCREMENT anfl ansyslmd 9999.9999 12-nov-2006 10 1815F7D1236F \

VENDOR_STRING=customer:00265231 ISSUED=10-Oct-2003 \

START=10-Oct-2003
```

按惯例，我们列出使用的工具等

目标:   `ANSYS 8.0`

保护机制:   `Macrovision FLEXlm v8.3b`

工具:   `Microsoft Visual Studio 7.1` (`CL`, `NMAKE`, `DUMPBIN`, `LIB`, …)

  `Microsoft WinDbg 6.1`

  `RedHat Cygwin 1.3.5`

  `IDM UltraEdit 9.0`

  `Datarescue IDA Pro 4.3` (`FLAIR`, …)

  `URSoft W32Dasm 8.9`

资源:   `Macrovision FLEXlm 9.2 SDK source code`

### 初次尝试

由于有了[6]的经验，开始的时候，我以为应该是一件很轻松的事情，可事实并非如此，前面有巨大的障碍等着我们。第一点就是该如何调试 `ansys.exe`，不象 `IMSL CNL` 或者其它软件有不大的可执行文件，`ansys.exe` 可是巨大的-50.1 兆大小。使用 `W32Dasm` 或 `OllyDbg`（`W32Dasm` 在加载的时候就有麻烦了）是不大可能的，`Visual Studio`'s integrated debugger 似乎只对 VS 建立的代码有效（我试了“Debug Processes…”但失败了），`SoftICE` 可以，但鼠标使用有问题，除非没有选择了我才会去使用它。幸运地，我们找到了，那就是 `WinDbg`，它能使用内核和用户模式 2 种调试模式，拥有友好的用户界面和标准的快捷键，更重要的是能处理巨大的二进制文件。

使用 `WinDbg`，重要的是如何操纵你的目标文件，它不是一次把整个镜像装载到内存。当 `ansys.exe` 被装载后，我们在 `ntdll!DbgBreakPoint()` 模块开头：`77F813B1 CC int 3`，为了到用户代码处，我们按 `Ctrl+G` 和输入 `00401000`，没有任何动静！再次重复，却犹如变魔术般地来到了 `00401000` 地址，就可以设置断点等等了，我不清楚这是怎么回事，但确实可以了。

接下来我们跟踪程序，得到一些结果，但是都不是很有用，让人担心的是我们在巨大的代码迷宫里耗费太多的时间而找不到核心（也就是 vendor keys 和 encryption seeds）。静态分析也是一个问题，`IDA` 需要太长的时间对 `ansys.exe` 进行全部分析，数据量可达到 274 兆之大，使 `IDA` 处于不稳定状态。看来没有更好的工具对目标进行调试了。

### 后台程序 Daemon 的攻击

其思想使不直接攻击目标文件，而攻击与其相关的 vendor daemon-`ansyslmd.exe`，我们知道它包含了同目标文件一样的 keys 和 seeds，而且小多了。我们已经确认目标的的 FLEXlm 的版本使 8.3b，同 `9.2SDK` 相差不远，所以我们能够从 daemon 中得到 keys/seeds，然后生成我们预期的“`permanent uncounted HOSTID=ANY`”许可。

简单运行 `ansyslmd` 后提示“(`ansyslmd`) Vendor daemons must be run by lmgrd（Vendor daemons 必须由 lmgrd 调用运行）”，文献[4][5]也谈到了 `lmgrd` 是不同 Vendor daemons 的调用者，因此我们先运行 `lmgrd.exe`，它由很好的选项和用法的命令行帮助。

```
E:\Ansys\v80\ANSYS\licensing\intel>lmgrd

... ...

16:06:54 (lmgrd) Running lmgrd in dedicated windows ...

16:06:54 (lmgrd) Use -z to run in foreground in this window

E:\Ansys\v80\ANSYS\licensing\intel>lmgrd -z -c c:\flexlm\license.dat

... ...

16:13:03 (lmgrd) License file(s): c:\flexlm\license.dat

16:13:03 (lmgrd) lmgrd tcp-port 1055

16:13:03 (lmgrd) Starting vendor daemons ...

16:13:03 (lmgrd) Started ansyslmd (pid 1024)

16:13:03 (ansyslmd) FLEXlm version 8.3b

16:13:03 (ansyslmd) Invalid license key (inconsistent authentication code)

16:13:03 (ansyslmd) ==>INCREMENT ane3fl ansyslmd 9999.9999 permanent uncount

ed 1CEF [...]

16:13:03 (ansyslmd) Invalid license key (inconsistent authentication code)

16:13:03 (ansyslmd) ==>INCREMENT ansys ansyslmd 9999.9999 permanent uncounte

d B1F02 [...]

16:13:03 (ansyslmd) Invalid license key (inconsistent authentication code)

16:13:03 (ansyslmd) ==>INCREMENT ane3 ansyslmd 9999.9999 permanent uncounted

CA5505 [...]

16:13:03 (ansyslmd) Invalid license key (inconsistent authentication code)

16:13:03 (ansyslmd) ==>INCREMENT anfl ansyslmd 9999.9999 permanent uncounted

1815F7 [...]

16:13:03 (ansyslmd) Server started on changsha

16:13:04 (lmgrd) ansyslmd using TCP-port 3056
```

上面截图的许可文件 `c:\flexlm\license.dat` 内容填写了“`permanent uncounted HOSTID=ANY`”。由于有相同的 sign 代码，当然会有出错的报告，实际上，事情更糟糕的是，当我们使用这个新的试用许可启动 ANSYS 时（也就是 `ansys.exe` 比 `lmgrd.exe` 更好），出错信息提示“`Local checkout filter reject request (-73, 125)`”，这暗示着 ANSYS 利用了 FLEXlm 提供的滤波函数，这是我们在文献[6]中没有讨论的主题，其将在后面引起我们大量的麻烦。根据 ANSYS 的提示，我们设置环境变量 `ANS_FLEXLM_DEBUG=2`，同时得到了更多如下的信息。

```
changsha: The 'ane3fl' license is a non-demo, uncounted

license. This particular client machine is not permitted to

run with non-demo, uncounted licenses.

(This license line's encryption key: 1CEF0B37317C)

(This could cause a local checkout filter rejection message

for the 'ane3fl' feature.)
```

不管怎样，我们装载 `lmgrd.exe` 到 `W32Dasm` 和 `IDA Pro` 中进行跟踪，因为我们的 FLEXlm 开发包中有 `lmgrd.exe` 的源代码，不难确定函数调用。

```
sub_00465F80 lmgrd.exe!start()   ; maybe it is in MSVCRT module
00466061: call 00408900     ; call service.c!main()

sub_00408900 service.c!main()
00408985: call 0041B050     ; call l_timers.c!l_real_getenv()
004089B3: call 004022EF     ; call ls_lmgrd.c!main_service_thread()

sub_004022EF ls_lmgrd.c!main_service_thread()
00402315: call 00415C9F     ; call wsock32.c!l__WSAStartup()
00402336: call 00402730     ; call ls_m_init.c!ls_m_init()
00402363: call 0041A470     ; call lm_hosttype.c!lc_hosttype()
00402421: call 0040A26B     ; call LOG()
0040264A - 0040267A     ; while (1) dead loop
00402660: call 0040BDD0     ; call ls_quorum.c!ls_quorum()
00402672: call 00403EC0     ; call ls_m_main.c!ls_m_main(), **daemons = ansyslmd, select_mask = 0
        ; after the call EIP is in ntdll!NtWaitForSingleObject()

sub_00403EC0 ls_m_main.c!ls_m_main()
00403FF5 - 0040473F     ; while (1) infinite loop
00404015: call 00404998     ; call ls_m_main.c!check_chld_died()
0040414F: call 00421AAD     ; call l_date.c!l_get_date()
0040422F: call 00408870     ; call ls_timestamp.c!ls_timestamp()
00404234: call 004081A5     ; call ls_statfile.c!ls_statfile()
004045A4: call 00404745     ; call ls_m_main.c!vendor_start(), *daemons = ansyslmd
00404654: call 00414FD0     ; call l_select.c!l_select()
00404692: call 00404A10     ; call ls_m_process.c!ls_m_process()
00404732: call 0040488B     ; call ls_m_main.c!send_daemons()

sub_00404745 ls_m_main.c!vendor_start()
004047C8: call 00407330     ; call ls_startup.c!ls_startup()

sub_00407330 ls_startup.c!ls_startup()
004077BB: call 00407B8F     ; call ls_startup.c!NT_Startup()

sub_00407B8F ls_startup.c!NT_Startup()
00407D1E: call [00491540]     ; call kernel32!GetStartupInfoA()
00407D92: call [0049154C]     ; call kernel32!
```

这是相关源代码。

```
pcsock.h:

#define WSAStartup(a,b) l__WSAStartup(a,b)

lmclient.h:

#define lm_hosttype(r) lc_hosttype(lm_job, r)
#define lm_daemon(d, o, p) lc_daemon(lm_job, d, o, p)

lsserver.h:

#define LOG(x) {ls_log_prefix(LL_LOGTO_ASCII, 0); (void) ls_log_asc_printf x;}

ls_lmgrd.c:

void main_service_thread(int argc, char *argv[])
{
... ... /* lmgrd.exe is run as app, not service, run_lmgrd_as_service = 0 */
while (1)
{
ls_quorum(main_master_list, "", 0); /* Make sure quorum is up */
ls_m_main(&master_daemons, &select_mask); /* real stuff */
}
}

ls_m_main.c:
```
