
![图片74 fnstenv](upload/attach/202003/817966_7357XSBQ8CPG6XS.jpg)

如此一来，`pop ebp`后`EBP`寄存器的值是`fpu_instruction_pointer`——`fldpi`指令位置：

![图片75 EBP](upload/attach/202003/817966_JWYGZMKX8PQ9T9S.jpg)

由`EBP`计算出需要解密的数据起始位置，`EDX`中存储的是数据长度(`0x315`)：

![图片76 Decrypt](upload/attach/202003/817966_7MFAFN6FVXZPT7M.jpg)

接着执行解密后的指令：

![图片77 Execute](upload/attach/202003/817966_VX2F5HMVY74EXFT.jpg)

![图片78 Execute](upload/attach/202003/817966_AAHUXR3ZJHZSUT9.jpg)

跳转后，执行相应指令，接下来`call`调用：

![图片79 call](upload/attach/202003/817966_HH27JACHHU3FEHQ.jpg)

`sub_562B2F`功能是获取指定的系统函数调用地址，此次是`kernel32.VirtualAlloc()`：

![图片80 ReturnVirtualAddr](upload/attach/202003/817966_TBPXQTHAKU6KXWC.jpg)

![图片81 ReturnVirtualAddr](upload/attach/202003/817966_TA9MMAE6HE5D5PZ.jpg)

之后调用`VirtualAlloc()`申请内存空间：

![图片82 VirtualAlloc](upload/attach/202003/817966_3H4946MPKA5A3HM.jpg)

向申请的内存空间中写入数据：

![图片83 WriteMem](upload/attach/202003/817966_AUBN9FJM74T8MQ7.jpg)

调用`sub_562B2F`获取`kernel32.Wow64DisableWow64FsRedirection()`调用地址：

![图片84 ReturnWow64DisableWow64FsRedirectionAddr](upload/attach/202003/817966_M5CP33QR2MV6N2N.jpg)

`LoadLibrary(shell32)`：

![图片85 LoadLibrary(shell32)](upload/attach/202003/817966_EKNNSPMDDNJC3NH.jpg)

传递参数给`sub_562B2F`，获取`shell32.ShellExcute()`调用地址：

![图片86 ReturnShellExcuteAddr](upload/attach/202003/817966_FUK6VDMEQR2RHUP.jpg)

![图片87 ReturnShellExcuteAddr](upload/attach/202003/817966_RDFH6QQEN8Y6TZ4.jpg)

`LoadLibrary(urlmon)`：

![图片88 LoadLibrary(urlmon)](upload/attach/202003/817966_JMHKNBBNWDR8W4M.jpg)

获取`urlmon.URLDownloadToFile()`调用地址：

![图片89 ReturnURLDownloadToFileAddr](upload/attach/202003/817966_XMD4KVRK6GECXHY.jpg)

![图片90 ReturnURLDownloadToFileAddr](upload/attach/202003/817966_G9JNF4K6CS2QEM7.jpg)

调用`URLDownloadToFile()`，其传递参数如图：

![图片91 URLDownloadToFile](upload/attach/202003/817966_A3AXDPAPCTC4P62.jpg)

![图片92 URLDownloadToFile](upload/attach/202003/817966_ZFJ3NXEJAR99J32.jpg)

读取文件：

![图片93 CreateFile](upload/attach/202003/817966_VM2E82E3SUERE3N.jpg)

![图片94 ReadFile](upload/attach/202003/817966_KU5W56FS3XKUBP7.jpg)

由于没有获取到文件，计算出的`EBX`值错误：

![图片95 End](upload/attach/202003/817966_ASUSAGFM8K5UVZ9.jpg)

故至此结束。

## 0x06 参考链接

- <a href="elink@2baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3I4x3K6b7#2x3U0V1&6">Office恶意文件解析与混淆研究</a>

- <a href="elink@6c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9K9h3c8@1k6i4u0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`."><mark class="encrypted">009K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9K9h3c8@1k6i4u0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`.</mark></a>

- <a href="elink@f61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5%4y4e0f1@1z5l9`.`.">Office文件格式基础知识</a>

## 0x01 漏洞描述

- **成因**：Windows的公式编辑器`EQNEDT32.EXE`读入包含MathType的OLE数据，在拷贝公式字体名称时没有对名称长度进行校验，使得攻击者可以通过刻意构造的数据内容覆盖栈上的函数返回地址，从而劫持程序流程。

- **影响版本**：Microsoft Office 2007 Service Pack 3, Microsoft Office 2010 Service Pack 2, Microsoft Office 2013 Service Pack 1, Microsoft Office 2016

- **POC**：[5f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9K9h3c8@1k6i4u0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`.](https://bbs.kanxue.com/elink@b3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9K9h3c8@1k6i4u0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6t1`.)

## 0x02 漏洞分析

> 笔者复现及分析环境：Windows 7 Service Pack 1、Microsoft Office 2010、x32dbg、IDA 7.0

`EQUATION.exe`存在：

![图片1 Equation.exe](https://bbs.kanxue.com/upload/attach/202003/817966_3YG2VC78TAAKHE7.jpg)

设置注册表项`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\EQNEDT32.EXE`：

![图片2 regedit.exe](https://bbs.kanxue.com/upload/attach/202003/817966_C5JDMA86EK75S57.jpg)

`Debugger`键值为x32dbg路径。

生成POC：

![图片3 POC](https://bbs.kanxue.com/upload/attach/202003/817966_ZQA34PFXQ59WP6F.jpg)

打开该文档，于`WinExec()`函数处设断：

![图片4 WinExec](https://bbs.kanxue.com/upload/attach/202003/817966_MQ84ABWUBDVC6Y2.jpg)

成功断下后，查看栈中返回地址：

![图片5 ReturnAdd](https://bbs.kanxue.com/upload/attach/202003/817966_WSVNX63APACHY6J.jpg)

继续向上查看栈，发现调用`WinExec()`的函数：

![图片6 Stack](https://bbs.kanxue.com/upload/attach/202003/817966_NVSD9TJQZSGZH36.jpg)

通过IDA分析`sub_4115A7`功能：

![图片7 IDA](https://bbs.kanxue.com/upload/attach/202003/817966_MJ2MSVEAZAVDNGN.jpg)

跟进`sub_41160F`查看：

![图片8 sub41160F](https://bbs.kanxue.com/upload/attach/202003/817966_AR6VPGGJQZSXDNT.jpg)

未校验长度，直接使用`strcpy()`函数，此处应该就是漏洞触发位置。进一步确定具体位置：

![图片9 strcpy](https://bbs.kanxue.com/upload/attach/202003/817966_75GVVUSBBHN8Y4Q.jpg)

于`0x41165B`处设断，重新运行。第二次成功断下后，查看`ESI`寄存器指向内存内容：

![图片10 ESI](https://bbs.kanxue.com/upload/attach/202003/817966_4DF6FN6QK6FGJW6.jpg)

此时`ECX`寄存器值为`0xC`，即复制48个字节到`EDI`寄存器指向内存，而`var_28`实际大小只有36个字节：

![图片11 EDI](https://bbs.kanxue.com/upload/attach/202003/817966_C4V6EFSPEY59FYS.jpg)

到达函数结束处：

![图片12 FunEnd](https://bbs.kanxue.com/upload/attach/202003/817966_YP9PNVFGE66XB6C.jpg)

`leave`指令执行完毕后，栈顶`0x18F1D0`处值为`0x430C12`，即调用`WinExec()`。而传递参数正是`0x18F350`指向内存中的cmd指令：

![图片13 shellcode](https://bbs.kanxue.com/upload/attach/202003/817966_92QWJJ6ZNSTPNNM.jpg)

成功弹出计算器：

![图片14 calc.exe](https://bbs.kanxue.com/upload/attach/202003/817966_7UKXTBHM6FVVS6P.jpg)

---

下面对使用到的POC进行简要分析。各变量含义由命名可知，RTF文档格式并非本文重点，如读者此前对RTF文档格式没有了解，建议先阅读文末**参考链接**中有关RTF文档格式的文章后再看POC源码。

![图片15 RTF](https://bbs.kanxue.com/upload/attach/202003/817966_82FTXB4FPUMHBS9.jpg)

首先判断命令长度是否小于43，而43这个数字是因为：

![图片16 CmdLen](https://bbs.kanxue.com/upload/attach/202003/817966_26XW8PB8USBVCTA.jpg)

上图选中部分是插入命令处，具体偏移由POC中`COMMAND_OFFSET(0x949*2)`变量给出。

将命令插入到构造数据中之后，函数返回拼接好的OLE。下面将OLE嵌入到RTF文档中：

![图片17 OLE](https://bbs.kanxue.com/upload/attach/202003/817966_462SXNTV2MTS96Y.jpg)

## 0x03 摩诃草(APT-C-09)组织某样本分析

> MD5：0D38ADC0B048BAB3BD91861D42CD39DF

于`0x41165B`处设断，在第二次断下时，各寄存器值如下：

![图片18 register](https://bbs.kanxue.com/upload/attach/202003/817966_H34PP2D4PC553PU.jpg)

继续执行到函数结束处`leave`指令：

![图片19 leave](https://bbs.kanxue.com/upload/attach/202003/817966_ZDFXUXYD2VZR54G.jpg)

`0x18F230`地址处值`0x430C47`即覆盖后的函数返回地址：

![图片20 FunReturn](https://bbs.kanxue.com/upload/attach/202003/817966_Q5W5NDAJZCSG299.jpg)

而该地址处指令是`ret`，有些出乎意料。继续向下执行，来到`0x18F3B0`处，正是`0x18F234`地址处值：

![图片21 ret](https://bbs.kanxue.com/upload/attach/202003/817966_JZRWM9VRZKM9UD8.jpg)

这方才是构造者意欲执行的指令。经过蓝色方框中的一系列运算后，`EBX`指向是真正的Shellcode：

![图片22 shellcode](https://bbs.kanxue.com/upload/attach/202003/817966_JDEHU7BPFX64NKH.jpg)

上述内容均可在OLE中查看(路径`\xl\embeddings`)：

![图片23 OLE](https://bbs.kanxue.com/upload/attach/202003/817966_DPBDWMA4W7EDM4P.jpg)

![图片24 OLE](https://bbs.kanxue.com/upload/attach/202003/817966_R87PEMKW2DP9EH9.jpg)

将OLE`0x1000`—`0x1520`中数据复制到一bin文件后，通过IDA查看。`sub_247`功能如下：

![图片25 IDA](https://bbs.kanxue.com/upload/attach/202003/817966_VQWZUANS2ABUUN5.jpg)

该函数接受的第二个参数即上文提到的`EBX`指向地址，于OLE中位置是`0x1040`，而`0x1040`+`0x558`处内容如下：

![图片26 PE](https://bbs.kanxue.com/upload/attach/202003/817966_N2N75QVUUZ8MTVN.jpg)

故该函数第一个功能是修正PE文件头。第二个功能流程如下：

![图片27 PEWrite](https://bbs.kanxue.com/upload/attach/202003/817966_H855FDNDW8CT5KU.jpg)

![图片28 PEWrite](https://bbs.kanxue.com/upload/attach/202003/817966_CP3PH6HXY783URU.jpg)

![图片29 PEWrite](https://bbs.kanxue.com/upload/attach/202003/817966_GKMF3M5C2MS6Y9U.jpg)

![图片30 PEWrite](https://bbs.kanxue.com/upload/attach/202003/817966_PRBKJTEYDBCBRBK.jpg)

将`0x1040`+`0x558`后的PE文件数据写入到`%APPDATA%\MSBuild.exe`中。第三个功能流程如下：

![图片31 RegeditWrite](https://bbs.kanxue.com/upload/attach/202003/817966_RKR5DYQE2WUXZEW.jpg)

![图片32 RegeditWrite](https://bbs.kanxue.com/upload/attach/202003/817966_CHM3A83VTUHXYCY.jpg)

![图片33 RegeditWrite](https://bbs.kanxue.com/upload/attach/202003/817966_XCVYYKDSET655Y5.jpg)

将`%APPDATA%\MSBuild.exe`写入注册表run项键值`lollipop`中。

## 0x04 响尾蛇(SideWinder)组织某样本分析

将文档拖进WinHex查看：
