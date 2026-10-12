
# SourceFormatX

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-446.htm`](https://www.pediy.com/kssd/pediy08/pediy8-446.htm)

前 2 天偶然碰到一个软件：`SourceFormatX`，觉得还不错，就是要注册，比较烦，为了研究它的一个功能，我决定分析一下看看，没想到。。。

机器也因此重装了一次（我的笔记本没光驱！！系统没备份！！），极其气愤！看雪上也有人讨论过

主要的危险代码：

1. 用 `WinExec` 无限开窗口，直到你死机
2. 删除你注册表，删了你肯定要重装了！(强烈 BS！`![](img/45777b5dbd6b901752ab7df42fce6892.jpg)` `![](img/45777b5dbd6b901752ab7df42fce6892.jpg)`)

主页上居然写这种话，不觉得脸红！！

`http://cn.textrush.com/download.htm`

> 百分之百绿色安全无插件软件
>
> `SourceFormatX` 通过了国内外许多专业软件评测组织的安全测试，并且获得了国外专业软件评测网站授予的 `"Safe To Install"` (安全安装) 和 `"100% Clean & Safe Software"` (100% 干净安全软件) 认证。这些奖项意味着 `SourceFormatX` 的安装包内没有包含任何恶意的病毒、木马、间谍软件、流氓插件、后门软件、广告软件，而且在安装和使用过程中没有任何潜在的威胁，可以放心使用。

现将破解方法公布如下：

## 一，脱壳

修改版的压缩壳，没难度，就是要修复一下 IAT，应该没问题的

## 二，去除 ANTI

搜索命令：

```
代码:

* * *

005771BE   .  E8 B1F8E8FF   call    <jmp.&kernel32.WinExec>          ; \\WinExec

* * *

```

我这里是搜索：

```
代码:

* * *

call    00406A74

* * *

```

然后向上找：

```
代码:

* * *

0057719E   . /EB 25         jmp     short killit.005771C5            ;   这行改成 JMP 跳过 WINEXEC 调用
005771A0   > |6A 03         push    3
005771A2   . |B8 34765700   mov     eax, killit.00577634             ;

048229125055114025094102049210040021027068051099091168132234034161018208011
005771A7   . |8D95 68FBFFFF lea     edx, dword ptr ss:[ebp-498]
005771AD   . |E8 0AA5EDFF  call    killit.004516BC
005771B2   . |8B85 68FBFFFF mov     eax, dword ptr ss:[ebp-498]
005771B8   . |E8 83CEE8FF  call    killit.00404040
005771BD   . |50           push    eax                              ; |CmdLine
005771BE   . |E8 B1F8E8FF  call    <jmp.&kernel32.WinExec>          ; \\WinExec
005771C3   .^|EB DB        jmp     short killit.005771A0
005771C5   > \\33C0         xor     eax, eax

* * *

```

对除了 1 个 `WinExec` 不这样处理，其他都要跳过去 (如何判断？就看 `WinExec` 后面有没有 `JMP`！有 `JMP`，这段代码就要跳过去)

然后对所有的 `SetFilePointer`，`GetFileSize` 下断点，如果后面是 `cmp eax,xxxxx` 的就是判断是否脱壳，将后面的跳转改成 `JMP`

比如：

```
代码:

* * *

00566062   .  6A 02         push    2                                ; /Origin = FILE_END
00566064   .  6A 00         push    0                                ; |pOffsetHi = NULL
00566066   .  6A 00         push    0                                ; |OffsetLo = 0
00566068   .  8B45 D4      mov     eax, dword ptr ss:[ebp-2C]       ; |
0056606B   .  50           push    eax                              ; |hFile
0056606C   .  E8 C309EAFF  call    <jmp.&kernel32.SetFilePointer>   ; \\SetFilePointer
00566071   .  3D 27E51000  cmp     eax, 10E527
00566076   .  EB 13         jmp     short killit.0056608B


0055F105   .  6A 00         push    0                                ; /pFileSizeHigh = NULL
0055F107   .  8B45 FC      mov     eax, dword ptr ss:[ebp-4]        ; |
0055F10A   .  50           push    eax                              ; |hFile
0055F10B   .  E8 D477EAFF  call    <jmp.&kernel32.GetFileSize>      ; \\GetFileSize
0055F110   .  3D A8421200  cmp     eax, 1242A8
0055F115   . /EB 1F        jmp     short killit.0055F136

* * *

```

★下面这处千万别漏了，这个是点击格式化目录这个按钮时调用的，直接删你注册表的！！

```
代码:

* * *

00564159   .  6A 02         push    2                                ; /Origin = FILE_END
0056415B   .  6A 00         push    0                                ; |pOffsetHi = NULL
0056415D   .  6A 00         push    0                                ; |OffsetLo = 0
0056415F   .  8B45 F0      mov     eax, dword ptr ss:[ebp-10]       ; |
00564162   .  50           push    eax                              ; |hFile
00564163   .  E8 CC28EAFF  call    <jmp.&kernel32.SetFilePointer>   ; \\SetFilePointer
00564168   .  3D 508D0F00  cmp     eax, 0F8D50
0056416D   .  90           nop
0056416E   .  E9 F7000000  jmp     killit.0056426A

* * *

```

## 三，破解

至此，anti 已经基本结束，下面开始破解！首先在当前目录下面建一个文件 `License.dat`，文件名千万别错，里面内容大于 10 个字符就可以了（怎么知道的，自己看吧，实在气愤！）

破解点怎么找到的就不说了，很简单的，破解点一共 4 个

```
代码:

* * *

突破格式化所有功能：

004FB527     /75 00         jnz     short killit.004FB529            ;  比较
004FB529   . \\C645 DF 01  mov     byte ptr ss:[ebp-21], 1          ;  标志


突破复制，保存大小限制的功能：
00561E60   .  8B45 DC      mov     eax, dword ptr ss:[ebp-24]
00561E63   .  8B55 F4      mov     edx, dword ptr ss:[ebp-C]
00561E66   .  E8 2121EAFF  call    killit.00403F8C
00561E6B   .  75 04        jnz     short killit.00561E71            ;  比较
00561E6D   .  C645 EF 01   mov     byte ptr ss:[ebp-11], 1          ;  标志


NAG 窗口去除：
00563612   > \\F645 CE 01  test    byte ptr ss:[ebp-32], 1          ;  是否显示 LOG 窗口
00563616   .  74 57         je      short SourceFo.0056366F          ;  跳走不显示

0056359A   .  F645 CE 01   test    byte ptr ss:[ebp-32], 1
0056359E   .  74 4D         je      short killit.005635ED           ;  nag 窗口，跳走

* * *

```

做内存补丁，这 4 个点就可以用了如果你是脱壳的，那么程序有校验，格式化后会变成乱码！！汗～～真受不了。。。

去除脱壳校验：

```
代码:

* * *

0045A000  /$  53           push    ebx                              ;  计算校验值
0045A001  |.  89C3         mov     ebx, eax
0045A003  |.  B8 FFFFFFFF  mov     eax, -1
0045A008  |.  56           push    esi
0045A009  |.  85D2         test    edx, edx
0045A00B  |.  74 16        je      short SourceFo.0045A023
0045A00D  |>  0FB633       /movzx   esi, byte ptr ds:[ebx]
0045A010  |.  0FB6C8      |movzx   ecx, al
0045A013  |.  31F1        |xor     ecx, esi
0045A015  |.  C1E8 08     |shr     eax, 8
0045A018  |.  33048D C09B5700 |xor eax, dword ptr ds:[ecx*4+579BC0]
0045A01F  |.  43          |inc     ebx
0045A020  |.  4A          |dec     edx
0045A021  |.^ 75 EA       \\jnz     short SourceFo.0045A00D
0045A023  |>  5E          pop     esi
0045A024  |.  F7D0        not     eax
0045A026  |.  5B          pop     ebx
0045A027  \\.  C3         retn

* * *

```

这里会程序输入表进行校验，返回一个校验值 `eax`，如果校验值不对，那么格式化后会变成乱码。脱壳后要修复 IAT，所以校验值肯定不对，运行未脱壳版，跑完这个校验值计算过程，把值记下来

一共变了 3 个寄存器：

```
代码:

* * *

EAX = 1DFF122A
ECX = C9
EDX = 0

* * *

```

重写这个函数：

```
代码:

* * *

0045A000  /$  B8 2A12FF1D mov     eax, 1DFF122A
0045A005  |.  33C9        xor     ecx, ecx
0045A007  |.  B1 C9       mov     cl, 0C9
0045A009  |.  33D2        xor     edx, edx
0045A00B  \\.  C3        retn

* * *

```

ok，破解完成。

最后是 ABOUT 窗口：

```
代码:

* * *

0052A580   .  BA 5CA75200 mov     edx, killit.0052A75C            ;  未注册用户
0052A585   .  E8 6E50F0FF call    killit.0042F5F8

* * *

```

字符串就自己改了

`4dd466d0e1e0cc123afbe504da2462ca`
