
# Protection Plus

> 原文：[`www.pediy.com/kssd/pediy06/pediy6205.htm`](https://www.pediy.com/kssd/pediy06/pediy6205.htm)

谈谈商用保护的破解

这里的商用保护，指直接使用第三方的工具，利用其编程接口实现软件的保护部分，而非原作者从头开始自行开发，比如 `Flexlm`, `SoftLocx` 等。直接在可执行文件上加壳的情况不在讨论之列。商用保护往往是众矢之的，研究的人比较多，在互联网上可以找到类似案例，结果是一损俱损，搞定一个，倒下一片。`Flexlm` 就是最好的例子。使用 `dongle` 进行保护的，也有类似之处（但不尽相同）。

破解商用保护，最重要的是尽可能地获取其资料，如果有正规的使用手册、演示代码等是最好的。这在破解中起着决定性的作用。

破解第一步，就是仔细阅读待破解软件的文档，不要漏过任何蛛丝马迹，切不可一头扎进 `SoftIce` 或 `IDA` 中瞎找。很多时候从 `SoftIce` 和 `IDA Pro` 中费心劳神得出的东西，文档中讲得明明白白。使用商用保护的软件，其文档常常会有说明。如果没有，在跟踪以及阅读反汇编代码的过程中也一定会暴露出来。这时候应当立即停下破解工作，到网上搜索一切相关信息，最好能下载到保护软件的试用版本，以及查找是否有相关的破解案例。尽可能地了解该保护软件的使用方法，各种编程接口，各错误码的含义，研究其弱点。从实战经验看，只要掌握了保护软件的使用方式，一般都能发现其弱点。比如 `Softlocx`，其检查软件的返回结果是个字符串，为“`LICENSED`”、“`EVALUATION`”、“`EXPIRED`”等等，应用软件根据这样的字符串来决定执行路径。这样的保护会强吗？

破解商用保护，比较简单省力的方法是找到保护软件与被保护软件的连接点，打断其连接，强迫被保护的软件正常运行。这种方式一般需要修改其二进制代码，打个补丁。这是下策，非不得以不为之。

上策嘛，当然是完全保留原有的保护机制，或是写个注册机，或是截取一个序列号或 `license`，或是凑个 `keyfile`。总之，以尽量不打补丁为原则。当然，这样做意味着要与专业对手交锋，难度大得多。很多时候要把加密算法搞懂，写出个逆运算，几乎是不可能的。有些算法根本不可逆。

不过，可以换一个方式。`Think like a cracker`！我的办法是，把保护软件的资料好好研究，通过 `softIce` 跟踪，获取足够的数据，直接写个程序，调用保护软件，让它为我们生成一个合法的结果。这一招屡试不爽。

以下的例子是个软件工程类的国外商业软件，名字隐去。本文是根据以前的破解笔记整理的，软件可能早已升过级了:-)。安装完运行时，出现一个 `license` 配置对话框，提示超期日期。另外有以下数据：

`Computer ID: 496861184`

`Session ID: 286021360`

下面要求输入 `Product Unlock Code`。如果不输，可以 `Evaluation` 方式运行。

## 1. 初步分析

在对主执行文件稍加分析后，即可将疑点集中到一个叫 `ETLicens.dll` 的动态链接库上。这个文件的文件名及其中的资源（包括 `Dialog`，`String` 等）是如此的露骨，简直就是 `crack` 的不二人选。其输出的函数也是如此。我以为就要在此解决战斗了:-)。实际上，也确实可以在此 `patch`，以暴破搞定。但给人的感觉不完美，需要 `patch` 的位置较多，没有一个合适的着力点。

在分析过程中可得到以下一些有趣的东西：

1. 一个反复出现的字符串“`ERStudioRulez!!`”。这是什么？
2. 会多次访问一个注册表项：
   `LOCALMACHINE\\Software\\Classes\\sysdskdrv\\75f42175d63bd284`，？

另有一个对应的字符串“`:LM:Software\\Classes\\sysdskdrv\\75f42175d63bd284`”。

在这个 `Dll` 中花了不少工夫，结果完全弄偏了方向。注意到此 `Dll` 还使用了另一个 `Dll`：`KeyLbE32.dll`。反编译一下这个文件，再用 `ResHacker` 看看，其 `CompanyName` 为“`Concept Software, Inc.`”。是另一家的东西？

## 2. 深入研究与原理

到 `Google` 上查一下，可在 `http://www.softwarekey.com` 上下到此保护软件：`Protection Plus V4.109`，与 `KeyLbE32.dll` 中的版本信息完全一致。研究一下此软件的使用方式。

`Protection Plus` 只是 `SoftwareKey` 产品系列中的一个，暂且不管别的东西，先只看这一个。产品基本介绍如下：

`Control Applications:`

. Create demos, rentals, pay-per-use applications.

. Immediate software activation via phone, fax, e-mail, or the Internet.

. Extend a demo, rental, or lease for an additional period of time.

. Turn on one or more menu options or applications in a suite.

. Convert application from single-user mode to multi-user mode.

. Protect using either fixed or floating network licensing.

. Modify the number of allowed network workstations.

. Convert from "lite" version to professional version.

. Increase or decrease a counter.

. Trigger any user-defined action.

`Enforce License Compliance:`

. Lock the application to run on the current computer or network server only.

. Limit the number of allowed network users or computers.

. Terminate rental or lease applications that are not paid.

. Turn an illegal copy into a demo or disable it.

. Force returned software or fraudulent purchases to stop working.

`Other Features:`

. Detects click backdating or demo reinstallation to gain additional usage.

. Encrypt/decrypt user data.

. Personalize and/or serialize without recompiling.

. Extremely flexible programming interface means applications limited only by your imagination!

看起来很强大，可供调用的函数很多:-)。这里就不一一列举。只提一下与 `crack` 直接相关的几个概念。

. `License Files`

License Files contain many data fields that allow you to store information to control the flow of execution in your application. The fields can be initialized before sending your application using our Windows-based License File Editor application and/or can be manipulated by the application remotely using Trigger Codes. License Files may be stored in a regular file in any directory, the Windows registry, a Marx Software Security Crypto-Box Hardware Key, or on an Internet Web Server. There are many character, string, numeric/bit, and date fields in the License File. Data ranges from pre-defined fields such as serial number, expiration date, and number of allowed network users, to user-defined fields.

注意 `license` 的保存形式：可以是常规的文件，注册表项（回想一下 1 中的那个:-），在硬件狗中，或在 Web 服务器上。

. `Trigger Codes` - Adds Remote Licensing Control

The PLUS system has a mechanism to send one-time secure remote signals to your application while running on the Client's computer. The user contacts you and gives you 2 six to eight-digit numbers that are displayed on a dialog box on their computer. These numbers can be sent by phone, FAX, e-mail, etc. In our Windows-based License File Editor application, these two numbers are entered; and, depending on what function you want to perform on their application, you return them one of 50 six to eight-digit numbers. In addition to the remote signal sent, you can also send an encrypted number. The encrypted number can contain data such as the number of allowed workstations, number of pay-per-use events, etc.

想想那个 `Unlock Code`。

主要涉及的概念就这两个。函数很多，只要动手写个小程序，尝试几次，即可找到破解之道（破绽百出:-）。`license` 配置对话框中的 `Computer ID` 是机器标识。`Session ID` 没有直接的对应物，象是 `Trigger Event ID` 与 `Trigger Event Data` 加密组合的结果。在第一步中提到的两个数据含义如下：

1. 字符串“`ERStudioRulez!!`”为保护 `license` 文件的 `password`:-)
2. `LOCALMACHINE\\Software\\Classes\\sysdskdrv\\75f42175d63bd284` 下就是保存在注册表中的 `license`

破解时的有些细节记不清了,笔记随手乱写不完整:-(，试了试生成自己的 `Unlock Code`，当时没成功，好象是创建 `license` 时有的 `field` 没填对。干脆调用 `pp_copyadd` 直接设为合法的机器。至于为什么用了内嵌汇编语言的方式来调用函数，大概是有些参数拿不稳，要对照 `IDA` 中的使用方式吧（也记不清了:-）。这份代码是随手写的，能工作就没管了（未检测各函数的返回值），有点虎头蛇尾呵呵。

以下为设置合法 `license` 的代码：

```cpp
///////////////////////////////////////
#include "stdafx.h"
#include "ETLicense.h"
CWinApp theApp;
using namespace std;
int _tmain(int argc, TCHAR* argv[], TCHAR* envp[])
{
    int nRetCode = 0;
    HINSTANCE hCommProt=::LoadLibrary("KeyLbE32.dll");
    FARPROC pp_lfopen,pp_lfclose,pp_copyadd;
    pp_lfopen=::GetProcAddress(hCommProt,MAKEINTRESOURCE(0x31));
    pp_lfclose=::GetProcAddress(hCommProt,MAKEINTRESOURCE(0x2C));
    pp_copyadd=::GetProcAddress(hCommProt,MAKEINTRESOURCE(0xC));
    CString strSysDiskInfo(":LM:Software\\Classes\\sysdskdrv\\75f42175d63bd284");
    char * lpszKeyName=strSysDiskInfo.GetBuffer(128);
    long hLicFile=0;
    char szPassword[]="ERStudioRulez!!";
    char *lpszPass=szPassword;
    _asm
    {
        lea eax,hLicFile
        push eax//存放句柄
        push lpszPass//license 文件密码
        push 2//类型 LF_REGISTRY
        push 0//license 文件属性 LF_CREATE_NORMAL
        push lpszKeyName //license 文件名（注册表项）
        call pp_lfopen
        push 0x1D9D8000 //Computer ID，也可调 pp_compno()获取
        push 2 //COPYADD_ERASEALL,删除现有值
        push hLicFile
        call pp_copyadd //把我这台机器加为合法用户吧:-)
        push hLicFile
        call pp_lfclose //关闭 license 文件
    }
    ::FreeLibrary(hCommProt);
    return nRetCode;
}
////////////////////////////////////////////////
```

## 3. 总结

总结一下破解用 `Protection Plus` 保护的软件。

先下载开发包，可以用 `IDA Pro` 的 `Flair` 做出 `Sig` 文件（我破解成功后就没干这个了，手懒:-）。拦截 `pp_lfopen` 函数，要访问 `license` 数据，必调此函数。在此可获取最重要的数据：

. `license` 文件名（与 `license` 位置相关）
. `license` 位置（可能是真正的文件名、注册表项、`dongle` 或是一个网址，如果是个网址可能会麻烦些:-）
. 用于保护 `license` 文件的 `password`

有了这些数据，能干什么？借用一下 `SoftwareKey` 的广告语：`Limited only by your imagination!` 好好研究联机帮助吧:-)。

4dd466d0e1e0cc123afbe504da2462ca
