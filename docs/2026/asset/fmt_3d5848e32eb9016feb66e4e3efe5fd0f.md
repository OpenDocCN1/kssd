
编辑`lm_code.h`文件，在其中填入所找到的加了密的`seeds`值和vendor keys 1到4，先设置`key 5`为0。

运行`genlic32.exe`。如果跳出一个错误框，则表示`lm_code.h`中信息有错；输入在前面找到的feature名称，点击“permanent”和“run anywhere”，这样就不需要一个server daemon在运行。点击“Make License”，填好文件名`license.dat`，然后点击“Make License”，查看License文件，应该看起来像这样：

![](https://bbs.kanxue.com/upload/attach/200803/93_krdkk2grpqdz2fa.gif)

## 5．寻找Vendor Key 5

FlexLm提供基于Vendor名称的5个Key，因此它必须以某种方式进行校验。如果在`lm_code.h`里的`Key1`到`Key4`或Vendor名称有错，那么`genlic32.exe`就不会正常工作。

再次装载目标文件，设置`lc_checkout`断点，载入daemon DLL，开始单步执行，单步跟踪到里面。程序检查`licence.dat`文件：

`l_validversion()`、`l_compare_version()` 检查license文件中的版本；

`l_date()`、`l_extract_date()` 在加密的Key中展开日期（无时间限制的日期被转化为1-jan-0）；

`l_start_ok()` 运行日期是正确的；

`l_host()` 运行的机器是否正确；

然后进入一些有很多XOR运算的函数中，这里是用XOR运算加密Seed的地方，可以看到`Key1`、`Key 2`、`Key 3`、`Key 4`以及Vendor名称，然后碰到如下代码：

```
call 10021160                    ; 将key 5放到ebx 

add  esp, 0000000C 

mov  eax, dword ptr [edi+04]     ; 从license.dat取seed1 

xor  eax, ebx                    ; seed1 XOR key5   

lea  ecx, dword ptr [ebp-58]    

push ecx 

mov  dword ptr [ebp-54], eax     ; 保存seed1 

mov  eax, dword ptr [edi+08]     ; 从license.dat取Seed2

mov  edi, dword ptr [ebp+0C] 

xor  eax, ebx                    ; seed2 XOR key5 

mov  dword ptr [ebp-50], eax     ; 保存seed2 

lea  eax, dword ptr [edi+54] 

push eax 

push esi 

call LMGR326A.l_extract_date

```

就这样，得到`key5`和经过XOR的`seeds1`和`Seed2`。

## 6．制作最终的`license.dat`

在`lm_code.h`中替换掉`seeds`和`key5`的值，运行`genlic32.exe`制作完全的licence文件。

上面这一篇教程是针对使用Flexlm的DLL加密的程序。其函数名称可以从DLL的输出表中得到，给予解密一个很大的提示。但是对于那些没有使用DLL的加密程序，该如何发现这些关键函数的位置呢?

这时候可以使用IDA和SoftICE来进行解密。首先用IDA的辅助工具Flair（该工具仅IDA注册用户有）建立Flexlm SDK的Sig文件，然后使用IDA对目标程序进行反汇编，再将建立好的Flexlm的Sig作用于已经反汇编的目标文件，这样就能直观地看到Flex关键函数所在的地址。

在SoftICE中载入该目标程序，然后在IDA中得出的关键函数地址处设置断点，运行就可以在所期望的地方停下来，使用`D`命令得到需要的信息。

因为已经存在Flexlm的辅助解密工具，可以根据部分信息`Key1`～`Key4`、Vendor名称得到另外的信息`Key5`和`Seeds`。这样，解密工作就可简化为在`_lc_init`处设置断点，获得部分加密信息，使用Flexlm辅助工具得到另外的信息`Key5`等，然后在`_lc_checkout`处设置断点获得加密程序的Feature名称，最后使用所有得到的信息，生成没有限制的license文件。

看雪软件安全

[d5aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@fdcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0)

本文结稿时间：2003.1

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)

上传的附件：

- [`Flexlm01.gif`](javascript:void(0);)

  （18.39kb，3607次下载）

- [`Flexlm02.gif`](javascript:void(0);)

  （18.35kb，3606次下载）

- [`Flexlm03.gif`](javascript:void(0);)

  （2.72kb，3583次下载）

- [`Flexlm04.gif`](javascript:void(0);)

  （2.99kb，3578次下载）

- [`Flexlm05.gif`](javascript:void(0);)

  （29.17kb，3596次下载）

- [`Flexlm06.gif`](javascript:void(0);)

  （5.02kb，3567次下载）

- [`Flexlm07.gif`](javascript:void(0);)

  （2.57kb，3573次下载）

- [`Flexlm08.gif`](javascript:void(0);)

  （2.18kb，3585次下载）
