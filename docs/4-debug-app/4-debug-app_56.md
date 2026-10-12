
# 输出软件中的加密数据

> 原文：[`www.pediy.com/kssd/pediy11/113538.html`](https://www.pediy.com/kssd/pediy11/113538.html)

在看雪学了不少东西，也发个贴。

最近使用了某个软件感觉不错，里面的数据很有用，但是其数据是加密的，自己想无法进行大数据量的分析，于是产生了将其数据解密的想法。

想法有了，下面就是开始执行了。首先使用 `CryptoSearcher` 扫描算法，结果扫描出了包括 `MD5`、`TEA`、`DES` 等 12 个算法，![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")。没有办法了只好拿 `OD` 出来不停的 `F2`、`F9`，终于找到了对数据进行解密的地方了。

但是 `EXE` 中的内部函数，在外面调不到啊，咋办呢？triones 的[利用 EXE 中的数据与代码](http://bbs.pediy.com/showthread.php?t=113450)这篇文章介绍了一种方法，本文介绍另外一种方式。

前面提到过解密函数了，那么此时明文和密文是同时出现的，我们把明文和密文输出就可以了。为了简便我们将其输出到系统的标准输出中。

废话完毕，进入正题，下面将把 `xxx.exe` 中数据输出到一个 `run.log` 文件中。

### 原始程序中对数据的解密处理

```
0057F9BC     8B55 F8       mov     edx, dword ptr [ebp-8]           ;   此时明码和密文是同时出现的。EBP_8＝明文；EBP_4=密文

0057F9BF     8BC3          mov     eax, ebx

0057F9C1  |.  E8 CA62E8FF  call    00405C90                         ;   执行完自己的操作后需要返回到此处
```

### 原始程序改为以下内容

```
jmp     0075EFE5                                                ;   跳转到我们要输出数据的地方

call    00405C90                                                ;   打印完成后，返回到此处
```

### 在此处完成信息输出

```
0075EFE5   >  8B45 FC       mov     eax, dword ptr [ebp-4]           ;   将密文放入 eax

0075EFE8   .  E8 BFFFFFFF  call    0075EFAC                         ;   调用自定义函数输出密文

0075EFED   .  8B45 F8       mov     eax, dword ptr [ebp-8]           ;   将明文放入 eax

0075EFF0   .  E8 B7FFFFFF  call    0075EFAC                         ;   调用自定义函数输出明文

0075EFF5   .  8B55 F8       mov     edx, dword ptr [ebp-8]           ;   执行原跳转处的操作

0075EFF8   .  8BC3          mov     eax, ebx                         ;   执行原跳转处的操作

0075EFFA   .^ E9 C209E2FF  jmp     0057F9C1                         ;   返回原程序处，进行其他处理
```

### 自定义的输出函数

```
0075EFAC  /$  55            push    ebp                              ;   用来打印 Unicode 字符串的函数

0075EFAD  |.  8945 DC       mov     dword ptr [ebp-24], eax          ;   将 eax 中的待打印结果放入堆栈，以便后续使用

0075EFB0  |.  50            push    eax                              ; /String

0075EFB1  |.  E8 F3AA0A7C  call    kernel32.lstrlenW                ; \\计算待待打印字符串的长度

0075EFB6      50            push    eax                              ;   将计算出的长度*2 后的结果入堆栈：ebp-28

0075EFB7      6A 00         push    0                                ;   将计数压入堆栈（相当于声明一个新的变量）

0075EFB9      8B45 DC       mov     eax, dword ptr [ebp-24]          ;   将堆栈中的待打印数据取出放到 eax 中

0075EFBC      50            push    eax                              ;   将待打印数据放入堆栈，准备打印

0075EFBD      E8 A8284B77  call    msvcrt.printf                    ;   调用 printf 将数据输出到控制台

0075EFC2      58            pop     eax                              ;   恢复堆栈

0075EFC3      40            inc     eax                              ;   取下一个字符

0075EFC4      40            inc     eax                              ;   取下一个字符

0075EFC5      8345 D4 01    add     dword ptr [ebp-2C], 1            ;   计数器+1

0075EFC9      8B4D D4       mov     ecx, dword ptr [ebp-2C]          ;   将计数器数据放入 ecx

0075EFCC      3B4D D8       cmp     ecx, dword ptr [ebp-28]          ;   比较是否已经打印到了最后

0075EFCF     ^ 7C EB         jl      short 0075EFBC                 ;   如果没有到最后，则继续打印

0075EFD1      58            pop     eax                              ;   恢复堆栈

0075EFD2      58            pop     eax                              ;   恢复堆栈

0075EFD3      58            pop     eax                              ;   恢复堆栈

0075EFD4      68 C6F07500   push    0075F0C6                         ;   取@符号到堆栈，以便打印

0075EFD9      E8 8C284B77  call    msvcrt.printf                    ;   输出@

0075EFDE      58            pop     eax                              ;   恢复堆栈

0075EFDF      C3            retn                                     ;   函数返回
```

最后在 `xxx.exe` 的同目录下新建一个 `run.bat` 文件，文件中内容如下：`xxx.exe >run.log`。双击 `run.bat` 运行程序，然后再关闭程序，此时在 `xxx.exe` 目录下就生成了一个 `run.log` 文件，此文件中存储这以“@”符号分隔的密文和明文。
