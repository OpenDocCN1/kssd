
# 军队个人总结助手

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-782.htm`](https://www.pediy.com/kssd/pediy08/pediy8-782.htm)

【破文标题】菜鸟破解之----军队个人总结助手注册算法破解分析（简单）

【破文作者】`紫色缘[TFW][PCG]`

【作者邮箱】`Cn_Fish@126.com`

【作者主页】`www.cniso.org`

【破解工具】`OD`、`PEID`

【破解平台】`Win9x/NT/2000/XP`

【软件名称】某个人总结助手

【软件大小】`1MB`

【原版下载】不提供~

【保护方式】无

【软件简介】

这软件是我在 `kanxue` 论坛网友的网络 U 盘那边无意间下载下来的~

此软件算法是非常的简单~对于初学算法者相当的有好处~ （高手请略过）

------------------------------------------------------------------------

【破解过程】

1、查壳后显示 `Microsoft Visual Basic 5.0 / 6.0` 无壳。又有软柿子吃了。呵呵~

2、启动主程序后，在输入假码后出现错误提示框。嘿嘿~~有错误的提示框就比较好办了！

3、载入 `OD`，`F3` 一下，出现“打开 32 位可执行文件”选择要分析的主程序后。直接来到以下的代码处：

```
004022EC  >  $  68 087C4000   push    00407C08                  ;  来到此处~~ 一般 VB 的程序都载入到此！

004022F1    .  E8 EEFFFFFF   call    <jmp.&MSVBVM60.#100>

004022F6    .  0000           add     [eax], al

004022F8    .  0000           add     [eax], al

004022FA    .  0000           add     [eax], al

004022FC    .  3000           xor     [eax], al

004022FE    .  0000           add     [eax], al

00402300    .  48             dec     eax
```

====================================================================================================

直接在命令行下断点：`bp rtcMsgBox`（消息提示断点，验证了第 2 步骤~~~）

`F9` 运行，出现注册框，输入本机注册码：“`111111111`” 然后点“现在注册”按钮，直接断在以下代码：

```
660DC5F3  >  55              push    ebp                          ; 断在此处~~~~

660DC5F4    8BEC            mov     ebp, esp

660DC5F6    83EC 4C         sub     esp, 4C

660DC5F9    8B4D 14         mov     ecx, [ebp+14]

660DC5FC    53              push    ebx

660DC5FD    56              push    esi

660DC5FE    57              push    edi

660DC5FF    66:8339 0A      cmp     word ptr [ecx], 0A

660DC603    B8 04000280     mov     eax, 80020004
```

====================================================================================================

然后直接按 `ALT+F9` 运行到达消息代码处~~

```
004A5C68    .  FF15 58104000 call    [<&MSVBVM60.#595>]                    ;  MSVBVM60.rtcMsgBox

004A5C6E    .  8D4D 90       lea     ecx, [ebp-70]

004A5C71    .  8D55 A0      lea     edx, [ebp-60]

004A5C74    .  51           push    ecx

004A5C75    .  8D45 B0      lea     eax, [ebp-50]

004A5C78    .  52           push    edx

004A5C79    .  8D4D C0      lea     ecx, [ebp-40]

004A5C7C    .  50           push    eax

004A5C7D    .  51           push    ecx

004A5C7E    .  6A 04        push    4
```

====================================================================================================

因为要分析其算法，所以向上找其触发事件代码处：

```
004A5A60    >  \55           push    ebp                          ;  再此下断,然后 F8 单步跟踪~~~

004A5A61    .  8BEC          mov     ebp, esp

004A5A63    .  83EC 0C       sub     esp, 0C

004A5A66    .  68 B6204000   push    <jmp.&MSVBVM60.__vbaExceptHandle>;  SE 处理程序安装

004A5A6B    .  64:A1 0000000>mov     eax, fs:[0]

004A5A71    .  50           push    eax

004A5A72    .  64:8925 00000>mov     fs:[0], esp

004A5A79    .  81EC D0000000 sub     esp, 0D0

.....................................中间省略 N 行无关紧要的代码.......................................

004A5B60    .  /7D 12       jge     short 004A5B74

004A5B62    .  |68 A0000000  push    0A0

004A5B67    .  |68 DCF24000 push    0040F2DC

004A5B6C    .  |57           push    edi

004A5B6D    .  |50           push    eax

004A5B6E    .  |FF15 4C104000 call    [<&MSVBVM60.__vbaHresultCheckObj>;  MSVBVM60.__vbaHresultCheckObj

004A5B74    >  \8B55 E0      mov     edx, [ebp-20]                 ;  本机注册码送入 EDX

004A5B77    .  8B45 E4       mov     eax, [ebp-1C]                 ;  本机用户名送入 EAX

004A5B7A    .  52           push    edx                             ;  EDX 入栈

004A5B7B    .  50           push    eax                             ;  EAX 入栈

004A5B7C    .  FF15 04114000 call    [<&MSVBVM60.__vbaR8Str>]      ;  MSVBVM60.__vbaR8Str

004A5B82    .  DCC0          fadd    st, st                        ;  加实数：st+st (st=207423487.00000000000)

004A5B84    .  8B4D E8       mov     ecx, [ebp-18]                 ;  本机用户名送入 ECX

004A5B87    .  51           push    ecx                             ;  ECX 入栈

004A5B88    .  DC05 70204000 fadd    qword ptr [402070]           ;  相加实数 2000

004A5B8E    .  DFE0          fstsw   ax                              ;  保存状态字的值到 AX

004A5B90    .  A8 0D         test    al, 0D                         ;  测试 al 与 OD

004A5B92    .  0F85 84030000 jnz     004A5F1C                       ;  不相等则跳

004A5B98    .  DD9D 24FFFFFF fstp    qword ptr [ebp-DC]           ;  然后再执行一次出栈操作

004A5B9E    .  FF15 04114000 call    [<&MSVBVM60.__vbaR8Str>]      ;  MSVBVM60.__vbaR8Str

004A5BA4    .  DC0D 68204000 fmul    qword ptr [402068]           ;  乘上一个实数 6

004A5BAA    .  83EC 08       sub     esp, 8

004A5BAD    .  DC05 60204000 fadd    qword ptr [402060]           ;  乘了后的值 相加上一个实数 1966

004A5BB3    .  DC85 24FFFFFF fadd    qword ptr [ebp-DC]           ;  2 者得数后的实数相加

004A5BB9    .  DFE0          fstsw   ax                              ;  保存状态字的值到 AX

004A5BBB    .  A8 0D         test    al, 0D                         ;  测试 al 与 OD

004A5BBD    .  0F85 59030000 jnz     004A5F1C                       ;  不相等则跳

004A5BC3    .  DD1C24        fstp    qword ptr [esp]                ;  然后再执行一次出栈操作

004A5BC6    .  FF15 B8104000 call    [<&MSVBVM60.__vbaStrR8>]       ;  MSVBVM60.__vbaStrR8

004A5BCC    .  8BD0          mov     edx, eax                       ;  EAX 送入 EDX，EAX 即为真码

004A5BCE    .  8D4D DC       lea     ecx, [ebp-24]

004A5BD1    .  FF15 54114000 call    [<&MSVBVM60.__vbaStrMove>]     ;  MSVBVM60.__vbaStrMove

004A5BD7    .  50           push    eax                             ;  EAX 入栈

004A5BD8    .  FF15 88104000 call    [<&MSVBVM60.__vbaStrCmp>]     ;  MSVBVM60.__vbaStrCmp

004A5BDE    .  8BF8          mov     edi, eax

004A5BE0    .  8D55 DC       lea     edx, [ebp-24]

004A5BE3    .  F7DF          neg     edi

004A5BE5    .  8D45 E0       lea     eax, [ebp-20]

004A5BE8    .  52           push    edx

004A5BE9    .  8D4D E4       lea     ecx, [ebp-1C]

004A5BEC    .  50           push    eax

004A5BED    .  1BFF          sbb     edi, edi

004A5BEF    .  8D55 E8       lea     edx, [ebp-18]

004A5BF2    .  51           push    ecx

004A5BF3    .  52           push    edx

004A5BF4    .  F7DF          neg     edi

004A5BF6    .  6A 04        push    4

004A5BF8    .  F7DF          neg     edi

004A5BFA    .  FF15 18114000 call    [<&MSVBVM60.__vbaFreeStrList>] ;  MSVBVM60.__vbaFreeStrList

004A5C00    .  8D45 D0       lea     eax, [ebp-30]

004A5C03    .  8D4D D4       lea     ecx, [ebp-2C]

004A5C06    .  50           push    eax

004A5C07    .  8D55 D8       lea     edx, [ebp-28]

004A5C0A    .  51           push    ecx

004A5C0B    .  52           push    edx

004A5C0C    .  6A 03        push    3

004A5C0E    .  FF15 34104000 call    [<&MSVBVM60.__vbaFreeObjList>] ;  MSVBVM60.__vbaFreeObjList

004A5C14    .  83C4 24       add     esp, 24

004A5C17    .  B9 04000280   mov     ecx, 80020004

004A5C1C    .  B8 0A000000   mov     eax, 0A

004A5C21    .  894D 98       mov     [ebp-68], ecx

004A5C24    .  66:85FF       test    di, di

004A5C27    .  8945 90       mov     [ebp-70], eax

004A5C2A    .  894D A8       mov     [ebp-58], ecx

004A5C2D    .  8945 A0       mov     [ebp-60], eax

004A5C30    .  894D B8       mov     [ebp-48], ecx

004A5C33    .  8945 B0       mov     [ebp-50], eax

004A5C36    .  0F84 93000000 je      004A5CCF                      ;爆破点，JE--->JMP
```

------------------------------------------------------------------------

【算法总结】

1、本机用户名出现后相加一次，然后再相加一个实数 `2000`

2、本机用户名的值乘以 `6`，然后再相加一个实数 `1966`

3、最后 `1` 和 `2` 的值相加起来，即是注册码。



下面是粗糙的 VB 算法注册机源码：

建立 `2` 个 `TextBox`，`1` 个 `CommandButton`。

```vb
Private Sub Command1_Click()
strcmp = Text1.Text
a = Val(Text1.Text)
sn1 = a + a + 2000
sn2 = a * 6 + 1966
sn = sn1 + sn2
Text2.Text = sn
End Sub
```

---

【版权声明】本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
