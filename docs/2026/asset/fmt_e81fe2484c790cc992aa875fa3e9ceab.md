

# [原创] CVE-2012-0158 漏洞分析与复现

## 一、漏洞信息

### 1. 漏洞简述

- 漏洞名称：Microsoft Office `MSCOMCTL.ocx` 栈溢出漏洞
- 漏洞编号：`CVE-2012-0158`
- 漏洞类型：栈溢出
- 漏洞影响：信息泄露
- `CVSS` 评分：9.3（High）
- 利用难度：Medium
- 基础权限：不需要

### 2. 组件概述

Microsoft Office 是由微软公司开发的一套基于 Windows 操作系统的办公软件套装。常用组件有 Word、Excel、PowerPoint 等。最新版本为 Microsoft 365 及 Office 2019（正式版本）。

### 3. 漏洞影响

Windows 系统下 Microsoft Office 2003 SP3、2007 SP2 和 SP3、2010 和 2010 SP3。

### 4. 解决方案

- 不要打开来自不受信任来源的 Microsoft Office 和 WordPad 文档。
- 防止易受攻击的 ActiveX 控件在 Internet Explorer 中运行。
- 更新 Microsoft Office 版本至高于 2010 的版本。

<a href="elink@91fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6i4m8V1j5i4c8W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3x3U0l9I4x3W2)9J5c8X3#2K6x3e0u0Q4x3X3b7H3x3U0N6Q4x3U0y4X3M7X3g2I4N6h3g2F1N6r3I4&6i4K6u0V1j5i4y4C8k6h3c8Q4x3X3c8I4N6h3g2K6N6r3W2G2L8Y4y4Q4x3X3c8X3j5i4q4Q4x3X3c8J5k6h3I4S2N6r3g2V1i4K6u0V1N6r3!0Q4x3X3c8@1K9r3W2K6i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8#2M7r3c8S2N6r3f1`.">官方文档链接</a>

## 二、漏洞复现

### 1. 环境搭建

- 靶机环境：Windows XP x86 SP3
- 靶机配置：`192.168.44.153`
- 攻击机环境：Kali Linux 2021.2
- 攻击机配置：`192.168.44.152`

### 2. 复现过程

本次复现设置的 `payload` 为在目标系统中启动 `calculator.exe`。

#### （1）Kali 中启动 Metasploit 生成木马文件

搜索 Adobe 渗透模块：

```text
msf > search cve-2012-0158
```

调用渗透模块：

```text
msf > use 0
```

![1.PNG](upload/attach/202108/917290_AMMTVQ7JR46NJTG.jpg)

设置 `payload` 为执行命令行：

```text
msf exploit(windows/fileformat/ms12_027_mscomctl_bof) > set payload 29
```

展示配置：

```text
msf exploit(windows/fileformat/ms12_027_mscomctl_bof) > show options
```

设置命令为运行 `calculator.exe`：

```text
msf exploit(windows/fileformat/ms12_027_mscomctl_bof) > set cmd calc
```

生成木马文件：

```text
msf exploit(windows/fileformat/ms12_027_mscomctl_bof) > exploit
```

![2.PNG](upload/attach/202108/917290_7PPKTADKH5GZSXT.jpg)

![3.PNG](upload/attach/202108/917290_RG5XKPBPGD9ZKZV.jpg)

#### （2）将 Word 木马文件发送至靶机

![4.PNG](upload/attach/202108/917290_2BYCAY5NR323V9E.jpg)

#### （3）运行 Word

![5.PNG](upload/attach/202108/917290_HJMTVUKY44GZV2F.jpg)

可看到跳出计算器窗口。

## 三、漏洞分析

### 1. 基本信息

- 漏洞文件：`msf.doc`
- 漏洞函数：自定义函数 `sub_275C89C7`
- 漏洞对象：Word 文档

### 2. 背景知识

#### 1. 栈溢出

程序将参数传入栈中时没有检查传入的参数是否大于预定的长度，导致栈中关键数据被参数覆盖。

#### 2. 跳板技术

从汇编角度讲，在函数执行 `retn` 返回前，`ESP` 刚好指向栈中存放返回地址的 `+0x4` 地址。若是发生栈溢出，将函数的返回地址覆盖为 `jmp esp`，剩下的部分覆盖为 `shellcode`，程序就会在返回时自动跳转到 `shellcode` 地址。这种方法能够增加 `shellcode` 的写入空间，并且绕过 `ASLR`。

### 3. 详细分析

#### 1. 基础分析

参考《漏洞战争》及其配套资料，首先尝试运行配套资料中的 `poc.doc` 文件，会发生：

![6.PNG](upload/attach/202108/917290_FUEG4YM66P3R8D3.jpg)

错误报告中显示出现了错误的访问地址 `0x41414141`。

使用二进制编辑器查看 `poc` 文件原格式：

![7.PNG](upload/attach/202108/917290_DFBTH4GZRGP2249.jpg)

看到 `41414141` 字符串。

从 `DOCF11E0` 开始是 OLE 签名，从这里开始一直至结束部分能够被 OffVis 识别。通过将文件中这部分数据单独提取出来，并制成二进制文件，以放在 OffVis 中查看。

制成二进制文件过程如下：

先将这部分数据复制到文档中。

8

将文档中的十六进制内容编码成二进制文件，这里使用 Python 语言实现：

```python
# 读取保存的 txt 文档
a = open('a.text')

# 将文档中的内容转为字符串形式
content = a.read()

# 将字符串以十六进制形式编码为二进制数据
content = content.decode('hex')

# 新建文件 b
b = open('b', 'wr')

# 二进制数据写入文件中
b.write(content)

# 关闭文件
a.close()
b.close()
```

将输出的文件 `b` 放在 OffVis 中查看格式，搜索 `41414141` 十六进制内容，找到其位置。

![9.PNG](upload/attach/202108/917290_UCD3TB67AN8HZN5.jpg)

`0x41414141` 位于 Data 字段。

#### 2. 动态分析

调试前需要先设置 OD。首先打开“选项”-“调试选项”，取消图中忽略的中断和异常：

![10.PNG](upload/attach/202108/917290_7NK2FTJGJ44FPJ6.jpg)

“插件”-“StrongOD”-“Options”，取消勾选“Skip Some Exceptions”。

![11.PNG](upload/attach/202108/917290_WCAS5V7B3MYBA3W.jpg)

打开 Microsoft Word，使用 OD 附加进程 `WINWORD`，按 `F9` 运行后打开 `poc` 文件。

![12.PNG](upload/attach/202108/917290_UGSPRM8V9X7S9X7.jpg)

继续按 `F9`，让程序运行，此时系统发生报错，提示为：

![13.PNG](upload/attach/202108/917290_P99BBYB3VBXHVGW.jpg)

查看栈中数据：

![14.PNG](upload/attach/202108/917290_8W99B33GN9DRXK5.jpg)

可发现最近的返回地址为 `0x275C8A0A`，在反汇编窗口中跟随：

![15.PNG](upload/attach/202108/917290_Q9TK2NPXW9SMBGY.jpg)

可看到函数 `MSCOMCTL.275C876D`，初步断定是这个函数引起的栈溢出。在这里设置断点，重新运行，到达断点处，此时栈中还没有发生溢出。

![16.PNG](upload/attach/202108/917290_T5FTFU3YDUC4EMT.jpg)

单步步入进一步确定发生溢出的位置。当运行到指令：

```asm
275C87CB F3:A5 rep movs dword ptr es:[edi],dword ptr ds:[esi]
```

发现栈中数据逐渐被覆盖，一直执行这个 `rep movs` 指令，栈的前后变化如图：

![17.PNG](upload/attach/202108/917290_C6J6XARVAVMDS6S.jpg)

![18.PNG](upload/attach/202108/917290_EC7DDZRVRVZC6PW.jpg)

可以看到从 `0x001215DC` 开始，往下的数据被逐个覆盖掉。至此，成功找到发生栈溢出的函数 `MSCOMCTL.275C876D`。

测试过 `poc.doc` 后，尝试分析用 `msf` 生成的运行计算器的 `msf.doc`。

重新运行 Word 并附加，按 `F9`，打开 `msf.doc` 文件，按 `F9` 运行。由于之前测试 `poc` 时在函数 `MSCOMCTL.275C876D` 处设置了断点，此次运行时程序自动断在了此处，跟进后运行至：

```asm
275C87CB F3:A5 rep movs dword ptr es:[edi],dword ptr ds:[esi]
```

指令执行前后栈中变化如下：

![19.PNG](upload/attach/202108/917290_ZRKQ24ZKNMCC6NF.jpg)

![20.PNG](upload/attach/202108/917290_MMCPG99RBJHMZKK.jpg)

图中可以看到栈中的返回地址被覆盖为 `27583C30`，在反汇编窗口中查看该地址指令：

![21.PNG](upload/attach/202108/917290_WU76YRCPU3PCE5Y.jpg)

指令为 `jmp esp`，使用跳板修改程序执行流。这说明栈中返回地址以下（`0x001215EC` 以下）的内容为 `shellcode`。在数据窗口和反汇编窗口中跟随这部分内容：

![22.PNG](upload/attach/202108/917290_ZTNKX9TGCWBK7VN.jpg)

能够看到 `shellcode` 前面是一堆“滑板指令”，运行后内存数据无任何变化。一直运行到这里时：

![23.PNG](upload/attach/202108/917290_W7W9US3W5UHYDHP.jpg)

能够看到一段循环指令，指令后方出现 `call 001217B1`。单步执行这个循环，发现数据窗口中地址 `0x001217B1` 的内容在逐渐发生变化，初步断定为 `shellcode` 在解码操作。将其全部解码后，数据窗口如图所示：

![24.PNG](upload/attach/202108/917290_TQ4YY4UHGJMG7QR.jpg)

继续跟进 `call 001217B1`，在执行指令 `001217B4 8D85 B2000000 lea eax,dword ptr ss:[ebp+0xB2]` 后，发现寄存器 `EAX` 指向了字符串 `"calc"`。继续向下执行 `call ebp`，此时系统弹出计算器，说明此步执行了运行计算器函数。继续运行后程序结束。

![25.PNG](upload/attach/202108/917290_7AH8VWD3TR9MYES.jpg)

![26.PNG](upload/attach/202108/917290_JRFVX5PX8VHMCH7.jpg)

#### 3. 静态分析

##### 1. IDA 分析

使用 IDA 打开 `C:\WINDOWS\system32\MSCOMCTL.OCX`，查看地址 `0x275C8A0A` 所在的函数（从地址向上找 `push ebp`），找到函数 `sub_275c89c7`，分析该函数的代码：

```asm
.text:275C89C7 push ebp
.text:275C89C8 mov ebp, esp
.text:275C89CA sub esp, 14h ; 开辟了0x14站空间
.text:275C89CD push ebx
.text:275C89CE mov ebx, [ebp+bstrString]
.text:275C89D1 push esi
.text:275C89D2 push edi
.text:275C89D3 push 0Ch ; dwBytes
.text:275C89D5 lea eax, [ebp+var_14]
.text:275C89D8 push ebx ; lpMem
.text:275C89D9 push eax ; int
.text:275C89DA call sub_275C876D
.text:275C89DF add esp, 0Ch ; 用掉0xC，还剩0x8
.text:275C89E2 test eax, eax
.text:275C89E4 jl short loc_275C8A52
.text:275C89E6 cmp [ebp+var_14], 6A626F43h
.text:275C89ED jnz loc_275D3085
.text:275C89F3 cmp [ebp+dwBytes], 8
.text:275C89F7 jb loc_275D3085
.text:275C89FD push [ebp+dwBytes] ; dwBytes
.text:275C8A00 lea eax, [ebp+var_8]
.text:275C8A03 push ebx ; lpMem
.text:275C8A04 push eax ; int
.text:275C8A05 call sub_275C876D ; 出现溢出的函数
.text:275C8A0A mov esi, eax
```

在进入函数 `sub_275C876D` 之前，堆栈还剩 `0x8` 大小的栈空间，按 `F5` 转化为 C 伪代码如下：

```cpp
int __stdcall sub_275C89C7(int a1, BSTR bstrString)
{
    BSTR v2; // ebx@1
    int result; // eax@1
    int v4; // esi@4
    int v5; // [sp+Ch] [bp-14h]@1
    SIZE_T dwBytes; // [sp+14h] [bp-Ch]@3
    int v7; // [sp+18h] [bp-8h]@4
    int v8; // [sp+1Ch] [bp-4h]@8

    v2 = bstrString;
    result = sub_275C876D((int)&v5, bstrString, 0xCu); //此处用掉了0xC站空间
    if (result >= 0)
    {
        if (v5 == 1784835907 && dwBytes >= 8) //此处没有对字符串限制大小，反而大于约定长度也可以
        {
            v4 = sub_275C876D((int)&v7, v2, dwBytes); //此处发生溢出
            if (v4 >= 0)
            {
                if (!v7)
                    goto LABEL_8;

                bstrString = 0;
                v4 = sub_275C8A59((UINT)&bstrString, (int)v2);
                if (v4 >= 0)
                {
                    sub_27585BE7(bstrString);
                    SysFreeString(bstrString);

                LABEL_8:
                    if (v8)
                        v4 = sub_275C8B2B(a1 + 20, v2);

                    return v4;
                }
            }

            return v4;
        }

        result = -2147418113;
    }

    return result;
}
```

从代码中看出，没有对输入的字符串进行限制大小，跟进 `sub_275C876D` 函数查看。

