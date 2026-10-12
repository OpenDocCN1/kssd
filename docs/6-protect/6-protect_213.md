
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



# 逆向分析文档：Ansys License Manager

## 1. 启动机制分析

最后调用的函数是 `kernel32!CreateProcessA()`，其 `pszCmdline` 命令参数为：

```
ansyslmd.exe –T changsha 8.3 –1 –c “c:\flexlm\license.dat” –lmgrd_start 40cd5776
```

若直接试运行，`ansyslmd` 能够启动，虽然会有出错信息，但很快停止。这意味着尽管 `ansyslmd` 必须由 `lmgrd` 调用，但实际上我们可以使用正确的命令选项直接运行它。进一步分析表明，找到简单的语法即可：

```
ansyslmd.exe –1 –c c:\flexlm\license.dat
```

我们对 `lmgrd.exe` 的探索可能是时间的浪费？也许吧，但不要太过执着，这种情况在逆向工程中经常遇到。让我们继续分析 `ansyslmd.exe`。

## 2. 函数调用链追踪

以下是从 `ansyslmd.exe` 内部主要调用链的汇编片段，展示了从入口点到许可证检查的关键函数：

```
sub_0045AED6 ansyslmd.exe!start()     ; maybe it is in MSVCRT module

0045AFB4: call 0045B3E0     ; call ls_app_main.c!main()

sub_0045B3E0 ls_app_main.c!main()

0045B42E: call 0045B450     ; call ls_daemon.c!ls_daemon()

sub_0045B450 ls_daemon.c!ls_daemon()

0045B46E: call 0045D100     ; call ls_app_init.c!ls_app_init()

sub_0045D100 ls_app_init.c!ls_app_init()

0045D20D: call 004869E0     ; call lm_new.c!l_n36_buf()
0045D293: call 0040ED51     ; call lm_init.c!lc_init()
0045D3A8: call 00424F50     ; call lm_set_attr.c!lc_set_attr()
0045D50B: call 00476670     ; call ls_s_funcs.c!ls_s_init()
0045E354: call 004804B0     ; call lm_daemon.c!lc_daemon()
0045E3A6: call 0045E6D4     ; call ls_app_init.c!ls_checkhost()
0045E3B7: call 00465A20     ; call ls_init_feat.c!ls_init_feat()

sub_00465A20 ls_init_feat.c!ls_init_feat()

00465A9E: call 00465AF3     ; call ls_init.c!ls_create_feats(), mode = 1, not important
00465AB4: call 00465AF3     ; call ls_init.c!ls_create_feats(), mode = 2, this one counts
00465AEA: call 00472240     ; call lsprfeat.c!ls_print_feats()

sub_00465AF3 ls_init_feat.c!ls_create_feats()

00465B33: call 00465FC3     ; call ls_init_feat.c!ls_feat_validate(), returns 0 if failure

sub_00465FC3 ls_init_feat.c!ls_feat_validate()

00466002: call 00434671     ; call lm_config.c!l_next_conf_or_marker()
00466187: call 004674EE     ; call ls_init_feat.c!good_config()

sub_004674EE ls_init_feat.c!good_config()

004675E2: call 0041889D     ; call lm_ckout.c!l_good_lic_key(), returns 0 if failure

sub_0041889D lm_ckout.c!l_good_lic_key()

00418CE6: call 0041B127     ; call lm_ckout.obj!l_crypt_private()

sub_0041B127 lm_ckout.obj!l_crypt_private()

0041B13D: call 0041B37A     ; call lm_ckout.obj!real_crypt()

sub_0041B37A lm_ckout.obj!real_crypt()

0041BE72: call 0041CE71     ; call lm_ckout.obj!l_string_key()

sub_0041CE71 lm_ckout.obj!l_string_key()

0041D8AC – 0041DB6B     ; if{} block containing L_MOVELONG() macro, not executed
0041DB70 – 0041DC66     ; else{} block, executed
0041DC9A: call 0041E0E3     ; call lm_ckout.obj!our_encrypt()
0041DD75 – 0041DEA4     ; for{} loop comparing signatures
```

看到那可爱的 `l_good_lic_key()` 了吗？我们正在其中。接下来的流程与前面类似，跟踪 `l_string_key()` 并揭示 `VENDORCODE` 和 `job` 结构。但由于滤波函数的存在，`l_string_key()` 的运行现在有所不同，因此我们必须再次检查 `l_strkey.c`。

## 3. 核心算法分析：l_string_key

以下是 `l_string_key()` 函数的核心代码片段：

```c
l_string_key(job, input, inputlen, code, len, license_key)
{
    int idx = (*job->vendor) % XOR_SEEDS_ARRAY_SIZ; /* idx = a%20 = 97%20 = 17 = 0x11 */

    ... ...

    for (i = 0; i < length; i++)
    {
        ... ...
        if (!user_crypt_filter && !user_crypt_filter_gen
            && (job->flags & LM_FLAG_MAKE_OLD_KEY))
        {
            ... ...
        }
        else
        {
            for (k = 0; k < L_STRKEY_BLOCKSIZE; k++)
            {
                int shift = ((k%4) * 8);
                unsigned long mask = 0xffL << shift;
                /* SEEDS_XOR = mem_ptr2_bytes defined in l_privat.h */
                y[k] ^= (((code->data[k/4] & mask) >> shift)
                         ^ job->SEEDS_XOR[xor_arr[idx][k%4]]);

                /* job->flags not NULL, LM_FLAG_MAKE_OLD_KEY = 1 */
                if (!(job->flags & LM_FLAG_MAKE_OLD_KEY))
                    y[k] = reverse_bits(y[k]);
            }
        }
        ... ...
    }

    ... ...

    for (i = 0; i < j; i++) /* compare user-input and real checksums */
    {
        ... ...
        if (user_crypt_filter)
            (*user_crypt_filter)(job, &x, i, y[i]);

        if (x != y[i])
            return 0;
    }

    if (user_crypt_filter_gen) /* executed only in keygen, not in checkout */
    {
        j = L_STRKEY_BLOCKSIZE; /* L_STRKEY_BLOCKSIZE = 8, in lmachdep.h */
        if (len == L_SECLEN_SHORT)
            j -= 2;

        for (i = 0; i < j; i++)
            (*user_crypt_filter_gen)(job, &y[i], i);
    }

    ... ...
}
```

滤波器引起了几个至关重要的分支。`l_string_key()` 在 checkout（`ansyslmd.exe`）和 keygen（`lmcrypt.exe`）进程中都被调用。
- 在 keygen 进程中：`user_crypt_filter = 0`, `user_crypt_filter_gen != 0`
- 在 checkout 进程中：`user_crypt_filter != 0`, `user_crypt_filter_gen = 0`

特别地，`user_crypt_filter = 004098EA` 对应于 `ansyslmd`，这两个函数提供了一个对称转化的附加层。

然而，藏在 `else{}` 代码块中，`code->data[]` 和 `job->mem_ptr2_bytes[]` 仍旧通过 `xor` 揭示了 encryption seeds（加密种子）。所以我们用老方法可以获得 vendor keys 和 seeds，以下是从内存 dump 得到的数据。

### 3.1 内存数据 Dump

**VENDORCODE in `ansyslmd.exe!l_string_key()`**

```
[0012E148] - 00000004 ....   int type;
[0012E14C] - b31813e3 ....   code[0], random and different at each run
[0012E150] - 21bf965f _..!   code[1], random and different at each run
[0012E154] - 928689b2 ....   true VENDOR_KEY1
[0012E158] - cd5480b3 ..T.   true VENDOR_KEY2
[0012E15C] - fc17f999 ....   true VENDR_KEY3
[0012E160] - 05dd4739 9G..   true VENDOR_KEY4
[0012E164] - 00030008 ....   FLEXlm version (here is 8.3)
[0012E168] - 38300062 b.08
[0012E16C] - 0000332e .3..
[0012E170] - 00000000 ....
```

**job in `ansyslmd.exe!l_string_key()`**

```
[002F4068] - 00000066 f...   int type;
[002F406C] - 009800de ....   char *mem_ptr2;
[002F4070] - 5513f8bc ...U   unsigned char mem_ptr2_bytes[12]; random and different at each run
[002F4074] - 006ee662 b.n.
[002F4078] - 00bf0000 ....
[002F407C] - 00000000 ....
[002F4080] - 00000000 ....
[002F4084] - 00000000 ....
[002F4088] - 00000000 ....
[002F408C] - 33656e61 ane3   feature to checkout
[002F4090] - 00006c66 fl..
```

注意 `idx = 17` 和置换表给出 `xor_arr[17][0] = 0`, `xor_arr[17][1] = 2`, `xor_arr[17][2] = 4`, `xor_arr[17][3] = 8`。这里 `xor` 操作数与 `job->mem_ptr2_bytes[]` 匹配的是 `006213BC`，所以：

- `ENCRYPTION_SEED1 = code->data[0] ^ 006213BC = B31813E3 ^ 006213BC = B37A005F`
- `ENCRYPTION_SEED2 = code->data[1] ^ 006213BC = 21BF965F ^ 006213BC = 21DD85E3`

## 4. 初始化与参数验证

在我们讨论滤波器如何工作前，指出几件值得重视的事情。首先，初始化在 `ls_app_init()` 里完成，在此我们可以很清楚地知道它是如何处理命令行选项的。这不仅说明我们先前在 `ansyslmd` 中看到的行为，而且暗示着只要 `argc >= 4`，vendor daemon 就可运行。

相关代码片段：

```c
keysize = (*L_NEW_JOB)(vendor_name, &vcode, 0, 0, 0, 0);
memcpy(vendorkeys, &vcode, sizeof(VENDORCODE)); /* vendorkeys[] comes from lmcode.c */

for (i = 1; i < keysize; i++)
    (*L_NEW_JOB)(vendor_name, &vendorkeys[i], 0, 0, 0, 0);

... ...

if (ls_a_user_crypt_filter)
    lc_set_attr(lm_job, LM_A_USER_CRYPT_FILTER, (LM_A_VAL_TYPE)ls_a_user_crypt_filter);

... ...

if (argc < 4) /* check the command line argument number */
{
    if ((argc > 1) && (!strcmp(argv[1], "-v") || !strcmp(argv[1], "-V")))
    {
        printf("%s v%d.%d%s - "COPYRIGHT_STRING(1988) "\n",
               lm_job->vendor, lm_job->code.flexlm_version,
               lm_job->code.flexlm_revision, lm_job->code.flexlm_patch);
        exit(0);
    }
    else
    {
        LOG((lmtext("Vendor daemons must be run by lmgrd\n")));
        exit(EXIT_BADCALL);
    }
}

... ...

master_list = ls_checkhost(master_list);
ls_init_feat(*list);
... ...
```

第二点，`ls_init_feat()` 内部函数被调用了 2 次。第一次调用是无关紧要的，只有第二次调用真正起作用，所以记得调试时跳过第一次。

相关定义：

```c
#define CREATE_SUITES 1
#define CREATE_FEATURES 2
```

```c
ls_init_feat(LM_SERVER * master_list)
{
    ... ...
    ls_create_feats(features, master_list, &f, CREATE_SUITES);
    ls_create_feats(features, master_list, &f, CREATE_FEATURES);
    ... ...
    ls_print_feats();
}
```



第三，获得 `vendor keys` 和 `encryption seeds` 后，我们开始在 `FLEXlm SDK` 中重建验证它们。在跟踪中我们在滤波器前获得 `feature ane3fl` 的 `hash code` 是 `y=4BE34851A0A5`，`lmcrypt.exe` 按道理应该给出同样的数字，但是我们在建立 `utils` 中碰到了一个编译错误。  

```
lmnewgen ansyslmd -o lm_new.c  

v8.1+ FLEXlm, non-CRO  

lc_init failed: Invalid FLEXlm key data supplied  

FLEXlm error: -44,49  
```

该错误耗费了我大量的时间，甚至怀疑 `vendor keys` 也被滤波器模糊了（这个错误在 `lmseeds.h` 生成前发生）。仔细检查了每件事情，包括在 `lm_code.h` 和 `pc.mak` 中的 `ansyslmd` 之 `vendor` 名称（低级错误），但是仍然不行。在毫无头绪的情况下为了解决到底为什么我不得不调试 `lmnewgen.exe`，最后确定了导致错误的原因是文献[6]中声称无用的三个常量。  

`lm_ckout.c!l_sg()`:  

```
x = 0x6f7330b8; /* v8.x */  
```

`lmnewgen.c!VKEY5()`:  

```
x = 0x6f7330b8; /* v8.x */  
```

`l_key.c!l_zinit()`:  

```
/* z = crokey_flag ? 0x67607419 : 0x3cde3ebf; v8.x */  

z = crokey_flag ? 0x62586954 : 0x72346B53; /* v9.x */  
```

它们在 `checkout` 进程没有用到的情况下是不用的，但是它们在 `vendor key` 校验时在 `keygen` 中参与了，至少在 `l_zinit()` 中对于 `z`，我们在文献[6]中不难分析这点，问题明显时版本数字： 在文献[6]中是 `9.2`，而这里是 `8.3`，这在 `l_zinit()` 是紧要的。在我们 `l_zinit()` 中的内容后，编译和连接都通过了。然后修改 `lmseeds.h`，重建和运行 `lmcrypt.exe`，输出了 `4BE34851A0A5` 的结果，到现在为止，我们可以确信 `vendor keys` 和 `encryption seeds` 是真正正确的。  

`4dd466d0e1e0cc123afbe504da2462ca`
