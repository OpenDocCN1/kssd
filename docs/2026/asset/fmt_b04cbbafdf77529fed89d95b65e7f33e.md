
# [原创]超级电话通算法分析及ESP定律操作详解，附内存注册机及如何一步步编写VC完美注册机功能篇(上)

【文章标题】: 超级电话通算法分析及ESP定律操作详解，附内存注册机及如何一步步编写VC完美注册机

【文章作者】: CCDeath

【作者邮箱】: CCDeath@163.com

【软件名称】: 超级电话通3.05

【下载地址】: 自己搜索下载

【加壳方式】: `ASPack` `1.06b` / `1.061b` -> Alexey Solodovnikov

【保护方式】: 用户名

【使用工具】: `OD+PEid`

【操作平台】: 盗版中的最低版本XP

【软件介绍】: 查询...

【作者声明】: 如果有人觉得好用，请支持共享..希望多多批评和建议...CCDeath在此感激不尽

--------------------------------------------------------------------------------

## 【详细过程】

第四篇破文，高手飘过.....我尽量把每个步骤写的清晰点...天呀，我都快养成这种习惯...

好了,我们来热热身。

### 一.ESP定律：根据堆栈平衡原理,略说就是物归原主，大跳转的意思,详情请看某位牛人写的<<广义ESP定律>>

用`VC`一步步编写注册机：先看我前一篇：

第三篇破文【原创】用`VC`一步一步编写完美注册机（上）----界面篇（附带`VC`源代码，有美妙的音乐、超酷的动态鼠标 、华丽的商业皮肤）-也可做`VC`入门

地址在这： `http://bbs.pediy.com/showthread.php?t=52350`

完美注册机功能篇(上)最终效果图:![](https://bbs.kanxue.com/upload/attach/200709/122382_a9bc6rfhatpml17.jpg)

来，我们来干掉她：

### 二.破解过程:

查壳`ASPack` `1.06b` / `1.061b` -> Alexey Solodovnikov，一个字“脱”，我们先在用`ESP`定律2种操作方式...大侠别笑我，我很笨滴！

先来第一种了，用`OD`载入后，出现“不知如何继续，因为内存地址\*\*\*”，天呀，我怎么知道呀,点确定，忽略异常接着点否。看右边寄存器`ESP`:是`0012FFC4`.按`F8`，看到`ESP`又一次变红色，记录`ESP`地址为：`00112FFA4`，好了就是她了。看图：

图在这呢，我每张图都有详细的注释：

![](https://bbs.kanxue.com/upload/attach/200709/122382_pr043z6ukaqy4lv.jpg)

在命令行`Command`输入: `dd 0012FFC4`. 看图，找到这个地方右键"断点"->"硬件访问"->"word".为什么要这么下，置顶贴有详细说明了。

图在这呢：![](https://bbs.kanxue.com/upload/attach/200709/122382_s2gtexj1i5z7byx.jpg)

按`F9`运行，哈哈，看到一个`jmp`要条黄河了，继续按`F8`.ohoh，这就是传说中的`OEP`了。好了,右键`Dump`.有没有注意到现在的`ESP`:是`0012FFC4`.接着`dump`出来的名称就要注意了，要跟原来一样，把原来那个重名其她名，不然运行时会提示错误信息了。

`OK`，我们来第二种

跟上面一样走到`ESP`变红了，只是下断点方式不同而已了，输入`hr 0012FFC4`.接下来和上面一样了。

图在这呢： ![](https://bbs.kanxue.com/upload/attach/200709/122382_61snv5ug9knjg61.jpg)

--------------------------

查壳：`Borland Delphi` `4.0` - `5.0`

伪装码：

用户名 ========`CCDeath`=======

注册码 ========`123456789`=====

用`OD`载入脱完壳的程序，查找字符串，来到这里：

```
0047EEB4  /.  55            push    ebp                              ;  为什么找到这里，我不说了，基础请看我
前几篇，谢谢！
0047EEB5  |.  8BEC          mov     ebp, esp
0047EEB7  |.  81C4 C8FEFFFF add     esp, -138                        ;  分配堆栈空间
0047EEBD  |.  53            push    ebx
0047EEBE  |.  56            push    esi
0047EEBF  |.  57            push    edi
0047EEC0  |.  33C9          xor     ecx, ecx
0047EEC2  |.  898D C8FEFFFF mov     dword ptr [ebp-138], ecx
0047EEC8  |.  898D CCFEFFFF mov     dword ptr [ebp-134], ecx
0047EECE  |.  8945 FC       mov     dword ptr [ebp-4], eax
0047EED1  |.  33C0          xor     eax, eax
0047EED3  |.  55            push    ebp
0047EED4  |.  68 C9F04700   push    0047F0C9
0047EED9  |.  64:FF30       push    dword ptr fs:[eax]
0047EEDC  |.  64:8920       mov     dword ptr fs:[eax], esp
0047EEDF  |.  33F6          xor     esi, esi
0047EEE1  |.  8D95 CCFEFFFF lea     edx, dword ptr [ebp-134]
0047EEE7  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0047EEEA  |.  8B80 D8020000 mov     eax, dword ptr [eax+2D8]
0047EEF0  |.  E8 0331FBFF   call    00431FF8
0047EEF5  |.  8B95 CCFEFFFF mov     edx, dword ptr [ebp-134]         ;  出现我们的用户名
0047EEFB  |.  8D85 D0FEFFFF lea     eax, dword ptr [ebp-130]         ;  用户名长度也出来了
0047EF01  |.  B9 FF000000   mov     ecx, 0FF
0047EF06  |.  E8 714DF8FF   call    00403C7C
0047EF0B  |.  8D95 D0FEFFFF lea     edx, dword ptr [ebp-130]
0047EF11  |.  8D45 D2       lea     eax, dword ptr [ebp-2E]
0047EF14  |.  B1 14         mov     cl, 14
0047EF16  |.  E8 F13AF8FF   call    00402A0C
0047EF1B  |.  8A55 D2       mov     dl, byte ptr [ebp-2E]
0047EF1E  |.  84D2          test    dl, dl                           ;  用户名是否为空
0047EF20  |.  76 1C         jbe     short 0047EF3E
0047EF22  |.  B1 01         mov     cl, 1
0047EF24  |.  8D45 D3       lea     eax, dword ptr [ebp-2D]
0047EF27  |>  33DB          /xor     ebx, ebx
0047EF29  |.  8A18          |mov     bl, byte ptr [eax]              ;  从头往尾一个一个的取出字符
0047EF2B  |.  8BF9          |mov     edi, ecx                        ;  计数器i
0047EF2D  |.  81E7 FF000000 |and     edi, 0FF                        ;  i=i&&0FF
0047EF33  |.  0FAFDF        |imul    ebx, edi                        ;  R(i)*i即第i个字符的ASCII乘以i
0047EF36  |.  03F3          |add     esi, ebx                        ;  把最终的结果存放
0047EF38  |.  41            |inc     ecx                             ;  ++
0047EF39  |.  40            |inc     eax                             ;  ++
0047EF3A  |.  FECA          |dec     dl                              ;  --
0047EF3C  |.^ 75 E9         \jnz     short 0047EF27                  ;  是不是每个字符都处理完毕？
0047EF3E  |>  B8 E0EA0B00   mov     eax, 0BEAE0                      ;  把常数0BEAEO
0047EF43  |.  2BC6          sub     eax, esi                         ;  eax=把这个常数减去上面循环所得到的结
果.肯定有阴谋，接着往下走
0047EF45  |.  8BF0          mov     esi, eax
0047EF47  |.  8D45 E7       lea     eax, dword ptr [ebp-19]
0047EF4A  |.  8D55 D2       lea     edx, dword ptr [ebp-2E]
0047EF4D  |.  B1 14         mov     cl, 14
0047EF4F  |.  E8 B83AF8FF   call    00402A0C
0047EF54  |.  8A55 E7       mov     dl, byte ptr [ebp-19]
0047EF57  |.  84D2          test    dl, dl
0047EF59  |.  76 20         jbe     short 0047EF7B
0047EF5B  |.  B1 01         mov     cl, 1
0047EF5D  |.  8D45 E8       lea     eax, dword ptr [ebp-18]
0047EF60  |>  F6C1 01       /test    cl, 1                           ;  好了，我们来看下面这个循环
0047EF63  |.  74 09         |je      short 0047EF6E
0047EF65  |.  8A18          |mov     bl, byte ptr [eax]              ;  从头往尾一个一个用户名字符串
0047EF67  |.  80F3 52       |xor     bl, 52                          ;  与52异或
0047EF6A  |.  8818          |mov     byte ptr [eax], bl              ;  存储这个经过运算的字符字符
0047EF6C  |.  EB 07         |jmp     short 0047EF75
0047EF6E  |>  8A18          |mov     bl, byte ptr [eax]              ;  跟上面一样同时存储
0047EF70  |.  80F3 4C       |xor     bl, 4C
0047EF73  |.  8818          |mov     byte ptr [eax], bl
0047EF75  |>  41            |inc     ecx
0047EF76  |.  40            |inc     eax
0047EF77  |.  FECA          |dec     dl
0047EF79  |.^ 75 E5         \jnz     short 0047EF60                  ;  处理完每个字符
0047EF7B  |>  8D95 C8FEFFFF lea     edx, dword ptr [ebp-138]         ;  上面经过运算后是：[eax]=11cdc38:接着
往下走发现这个根本就没用处，折磨人而已
0047EF81  |.  8BC6          mov     eax, esi
0047EF83  |.  E8 C095F8FF   call    00408548                         ;  由下面判断是个关键CALL1
0047EF88  |.  8B85 C8FEFFFF mov     eax, dword ptr [ebp-138]         ;  晕，突然出来一串字符，肯定上面那个
CALL做的手脚，跟进上面CALL
0047EF8E  |.  50            push    eax                              ;把我们得到字符串"778306"
0047EF8F  |.  8D95 CCFEFFFF lea     edx, dword ptr [ebp-134]
0047EF95  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0047EF98  |.  8B80 E4020000 mov     eax, dword ptr [eax+2E4]
0047EF9E  |.  E8 5530FBFF   call    00431FF8
0047EFA3  |.  8B95 CCFEFFFF mov     edx, dword ptr [ebp-134]         ;假的注册码"123456789"
0047EFA9  |.  58            pop     eax
0047EFAA  |.  E8 014EF8FF   call    00403DB0                         ;  经典模式关键CALL2
0047EFAF  |.  0F85 D8000000 jnz     0047F08D
0047EFB5  |.  8B15 48E14800 mov     edx, dword ptr [48E148]          ;  Scbook.00490184
```

分析一下关键`CALL1`:进来

```
00408548  /$  83C4 F8       add     esp, -8
0040854B  |.  6A 00         push    0                                ; /Arg1 = 00000000
0040854D  |.  894424 04     mov     dword ptr [esp+4], eax           ; |eax，就是上面常数减去运算后的结果
00408551  |.  C64424 08 00  mov     byte ptr [esp+8], 0              ; |
00408556  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]           ; |
0040855A  |.  8BC2          mov     eax, edx                         ; |
0040855C  |.  BA 74854000   mov     edx, 00408574                    ; |ASCII "%d"//看到这个没有，转化为十进制
拿出计算器，呵呵，还真的是这样，就是真的注册码
00408561  |.  E8 9E0B0000   call    00409104                         ; \Scbook.00409104
```
