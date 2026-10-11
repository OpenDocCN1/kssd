
# 简单分析某 PB 软件+VB 注册机

> 原文：[`www.pediy.com/kssd/pediy12/143648.html`](https://www.pediy.com/kssd/pediy12/143648.html)

【文章标题】: 简单分析某 PB 软件+VB 注册机

【文章作者】: `playx`

【作者邮箱】: `playx@km169.net`

【作者主页】: `http://playx.ys168.com`

【作者 QQ 号】:

【软件名称】: 某代收货款综合管理系统

【软件大小】: `1x m`

【下载地址】: `http://115.com/file/clftzpso#`

【加壳方式】: `NO`

【保护方式】: 序列号

【编写语言】: `PB9.0`

【使用工具】: `OD`，`PBDeCompile`，`shudepb`，`windows 计算器`

【操作平台】: `winxp-sp3`

【软件介绍】: 一款物流企业代收货款软件，PB 开发。

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

最近帮朋友分析一个物流类代收款软件，安装后到目录下一看，一大堆 PB 打头的 DLL 文件。不用说，肯定就是 PB 开发编译的软件。

按照朋友说的步骤，试着运行一下，发现只是序列号保护，一机一码，并且没有加壳。

`OD` 载入试着跟踪一下。

引用: 

```
10001B30  >  $  55                push    ebp                ;  (initial cpu selection)//停在这里
  10001B31    .  8BEC             mov     ebp, esp
  10001B33    .  6A FF            push    -1
  10001B35    .  68 A0500010      push    100050A0
  10001B3A    .  68 28300010      push    10003028                 ;  SE 处理程序安装
  10001B3F    .  64:A1 0000000>  mov     eax, dword ptr fs:[0]
  10001B45    .  50               push    eax
  10001B46    .  64:8925 00000>  mov     dword ptr fs:[0], esp
  10001B4D    .  83C4 A8          add     esp, -58
  10001B50    .  53               push    ebx
  10001B51    .  56               push    esi
  10001B52    .  57               push    edi
  10001B53    .  8965 E8          mov     dword ptr [ebp-18], esp
  10001B56    .  FF15 24500010   call    dword ptr [<&KERNEL32.GetVersio>;  kernel32.GetVersion
  10001B5C    .  33D2            xor     edx, edx
  10001B5E    .  8AD4            mov     dl, ah
  10001B60    .  8915 24870010   mov     dword ptr [10008724], edx
  10001B66    .  8BC8            mov     ecx, eax
  10001B68    .  81E1 FF000000  and     ecx, 0FF
  10001B6E    .  890D 20870010   mov     dword ptr [10008720], ecx
  10001B74    .  C1E1 08         shl     ecx, 8
  10001B77    .  03CA            add     ecx, edx
  10001B79    .  890D 1C870010   mov     dword ptr [1000871C], ecx
  10001B7F    .  C1E8 10         shr     eax, 10
  10001B82    .  A3 18870010     mov     dword ptr [10008718], eax
  10001B87    .  E8 64130000     call    10002EF0
  10001B8C    .  85C0            test    eax, eax
  10001B8E    .  75 0A           jnz     short 10001B9A
  10001B90    .  6A 1C           push    1C
  10001B92    .  E8 69010000     call    10001D00
  10001B97    .  83C4 04         add     esp, 4
  10001B9A    >  C745 FC 00000>  mov     dword ptr [ebp-4], 0
  10001BA1    .  E8 4A110000     call    10002CF0
  10001BA6    .  E8 65060000     call    10002210
  10001BAB    .  FF15 10500010   call    dword ptr [<&KERNEL32.GetComman>; [GetCommandLineA
  10001BB1    .  A3 B4890010     mov     dword ptr [100089B4], eax
  10001BB6    .  E8 D50F0000     call    10002B90
  10001BBB    .  A3 BC840010     mov     dword ptr [100084BC], eax
  10001BC0    .  85C0            test    eax, eax
  10001BC2    .  74 09           je      short 10001BCD
  10001BC4    .  A1 B4890010     mov     eax, dword ptr [100089B4]
  10001BC9    .  85C0            test    eax, eax
  10001BCB    .  75 0A           jnz     short 10001BD7
  10001BCD    >  6A FF           push    -1
  10001BCF    .  E8 7C090000     call    10002550
  10001BD4    .  83C4 04         add     esp, 4
  10001BD7    >  E8 040D0000     call    100028E0
  10001BDC    .  E8 0F0C0000     call    100027F0
  10001BE1    .  E8 3A090000     call    10002520
  10001BE6    .  8B35 B4890010   mov     esi, dword ptr [100089B4]
  10001BEC    .  8975 9C         mov     dword ptr [ebp-64], esi
  10001BEF    .  803E 22         cmp     byte ptr [esi], 22
  10001BF2    .  0F85 BE000000  jnz     10001CB6
  10001BF8    >  46              inc     esi
  10001BF9    .  8975 9C         mov     dword ptr [ebp-64], esi
  10001BFC    .  8A06            mov     al, byte ptr [esi]
  10001BFE    .  3C 22          cmp     al, 22
  10001C00    .  74 1C          je      short 10001C1E
  10001C02    .  84C0            test    al, al
  10001C04    .  74 18          je      short 10001C1E
  10001C06    .  25 FF000000    and     eax, 0FF
  10001C0B    .  50              push    eax
  10001C0C    .  E8 9FFEFFFF     call    10001AB0
  10001C11    .  83C4 04         add     esp, 4
  10001C14    .  85C0            test    eax, eax
  10001C16    .  ^ 74 E0        je      short 10001BF8                 //循环取得安装位置和文件名
  10001C18    .  46              inc     esi
  10001C19    .  8975 9C         mov     dword ptr [ebp-64], esi
  10001C1C    .  ^ EB DA        jmp     short 10001BF8
  10001C1E    >  803E 22         cmp     byte ptr [esi], 22
  10001C21    .  75 04          jnz     short 10001C27
  10001C23    .  46              inc     esi
  10001C24    .  8975 9C         mov     dword ptr [ebp-64], esi
  10001C27    >  8A06            mov     al, byte ptr [esi]
  10001C29    .  84C0            test    al, al
  10001C2B    .  74 0A          je      short 10001C37
  10001C2D    .  3C 20          cmp     al, 20
  10001C2F    .  77 06          ja      short 10001C37
  10001C31    .  46              inc     esi
  10001C32    .  8975 9C         mov     dword ptr [ebp-64], esi
  10001C35    .  ^ EB F0        jmp     short 10001C27
  10001C37    >  C745 D0 00000>  mov     dword ptr [ebp-30], 0
  10001C3E    .  8D45 A4         lea     eax, dword ptr [ebp-5C]
  10001C41    .  50              push    eax                            ; /pStartupinfo
  10001C42    .  FF15 20500010   call    dword ptr [<&KERNEL32.GetStartu>; \GetStartupInfoA
  10001C48    .  F645 D0 01      test    byte ptr [ebp-30], 1
  10001C4C    .  74 0A          je      short 10001C58
  10001C4E    .  8B45 D4         mov     eax, dword ptr [ebp-2C]
  10001C51    .  25 FFFF0000    and     eax, 0FFFF
  10001C56    .  EB 05          jmp     short 10001C5D
  10001C58    >  B8 0A000000     mov     eax, 0A
  10001C5D    >  50              push    eax
  10001C5E    .  56              push    esi
  10001C5F    .  6A 00           push    0
  10001C61    .  6A 00           push    0                              ; /pModule = NULL
  10001C63    .  FF15 1C500010   call    dword ptr [<&KERNEL32.GetModule>; \GetModuleHandleA
  10001C69    .  50              push    eax
  10001C6A    .  E8 91F3FFFF     call    10001000                       //这里 F7 跟进，否则跑飞
  10001C6F    .  8945 A0         mov     dword ptr [ebp-60], eax
  10001C72    .  50              push    eax
  10001C73    .  E8 D8080000     call    10002550
  10001C78    .  EB 21          jmp     short 10001C9B
  10001C7A    .  8B45 EC         mov     eax, dword ptr [ebp-14]
  10001C7D    .  8B08            mov     ecx, dword ptr [eax]
  10001C7F    .  8B09            mov     ecx, dword ptr [ecx]
  10001C81    .  894D 98         mov     dword ptr [ebp-68], ecx
  10001C84    .  50              push    eax
  10001C85    .  51              push    ecx
  10001C86    .  E8 D5090000     call    10002660
  10001C8B    .  83C4 08         add     esp, 8
  10001C8E    .  C3              retn
  10001C8F    .  8B65 E8         mov     esp, dword ptr [ebp-18]
  10001C92    .  8B55 98         mov     edx, dword ptr [ebp-68]
  10001C95    .  52              push    edx
  10001C96    .  E8 D5080000     call    10002570
  10001C9B    >  83C4 04         add     esp, 4
  10001C9E    .  C745 FC FFFFF>  mov     dword ptr [ebp-4], -1
  10001CA5    .  8B4D F0         mov     ecx, dword ptr [ebp-10]
  10001CA8    .  64:890D 00000>  mov     dword ptr fs:[0], ecx
  10001CAF    .  5F              pop     edi
  10001CB0    .  5E              pop     esi
  10001CB1    .  5B              pop     ebx
  10001CB2    .  8BE5            mov     esp, ebp
  10001CB4    .  5D              pop     ebp
  10001CB5    .  C3              retn
  10001CB6    >  803E 20         cmp     byte ptr [esi], 20
```



## 调试跟踪

F7 跟进来到这里

```text
10001CB9   .^ 0F86 68FFFFFF jbe     10001C27
10001CBF   .  46             inc     esi
10001CC0   .  8975 9C        mov     dword ptr [ebp-64], esi
10001CC3   .^ EB F1          jmp     short 10001CB6
```

引用：

```text
10001000  /$  56             push    esi
10001001  |.  57             push    edi
10001002  |.  FF15 10500010 call    dword ptr [&KERNEL32.GetComman]; [GetCommandLineA
10001008  |.  8BF0           mov     esi, eax
1000100A  |.  8A06           mov     al, byte ptr [esi]
1000100C  |.  46             inc     esi
1000100D  |.  3C 22          cmp     al, 22
1000100F  |.  75 3E          jnz     short 1000104F
10001011  |.  E8 DA000000    call    100010F0
10001016  |.  85C0           test    eax, eax
10001018  |.  6A 22          push    22
1000101A  |.  56             push    esi
1000101B  |.  74 07          je      short 10001024
1000101D  |.  E8 7E070000    call    100017A0
10001022  |.  EB 05          jmp     short 10001029
10001024  |>  E8 B7060000    call    100016E0
10001029  |>  8BD0           mov     edx, eax
1000102B  |.  83C4 08        add     esp, 8
1000102E  |.  85D2           test    edx, edx
10001030  |.  74 44          je      short 10001076
10001032  |.  8BFA           mov     edi, edx
10001034  |.  83C9 FF        or      ecx, FFFFFFFF
10001037  |.  33C0           xor     eax, eax
10001039  |.  F2:AE          repne   scas byte ptr es:[edi]
1000103B  |.  F7D1           not     ecx
1000103D  |.  49             dec     ecx
1000103E  |.  83F9 02        cmp     ecx, 2
10001041  |.  77 07          ja      short 1000104A
10001043  |.  BE B0840010    mov     esi, 100084B0
10001048  |.  EB 2C          jmp     short 10001076
1000104A  |>  8D72 02        lea     esi, dword ptr [edx+2]
1000104D  |.  EB 27          jmp     short 10001076
1000104F  |>  E8 9C000000    call    100010F0
10001054  |.  85C0           test    eax, eax
10001056  |.  6A 20          push    20
10001058  |.  56             push    esi
10001059  |.  74 07          je      short 10001062
1000105B  |.  E8 40070000    call    100017A0
10001060  |.  EB 05          jmp     short 10001067
10001062  |>  E8 79060000    call    100016E0
10001067  |>  83C4 08        add     esp, 8
1000106A  |.  BE B4840010    mov     esi, 100084B4
1000106F  |.  85C0           test    eax, eax
10001071  |.  74 03          je      short 10001076
10001073  |.  8D70 01        lea     esi, dword ptr [eax+1]
10001076  |>  8B4C24 18      mov     ecx, dword ptr [esp+18]
1000107A  |.  8B5424 10      mov     edx, dword ptr [esp+10]
1000107E  |.  8B4424 0C      mov     eax, dword ptr [esp+C]
10001082  |.  6A 00          push    0
10001084  |.  68 E8030000    push    3E8
10001089  |.  51             push    ecx
1000108A  |.  56             push    esi
1000108B  |.  52             push    edx
1000108C  |.  50             push    eax
1000108D  |.  E8 2E060000    call    <jmp.&PBVM90.#137>         //这里开始解释运行。
10001092  |.  5F             pop     edi
10001093  |.  5E             pop     esi
10001094  \.  C2 1000        retn    10
```

下了几个断点，跟了一会，虽然能断下，但没有任何结果。众所周知，`PB` 编译的软件和 `VB`、`VF` 类似，都是解释类语言，跟踪调试起来很头大，一直会在 `DLL` 中打转，即使找到了关键点，也是在 `DLL` 中。

既然是解释执行，如果代码没有混淆的话，应该可以用工具还原，于是就想到了用反编译软件读出源码来分析一下。

先找到一款 `PBDeCompile`，软件不错，但是 `DEMO` 版有长度和行数限制，读出的代码关键地方看不到。于是又找了一款 `shudepb`，这个不错，虽然也是 `DEMO` 版，但基本没有限制，所有的代码都能读出。

在反编译模块中，使用全局搜索出错提示，找到了两个地方：

引用：`w_registration.win->cb_save(commandbutton)->Events->clicked (none) returns long [pbm_bnclicked]->`

```text
0015:      messagebox("注册提示","注册失败，请重新输入注册码！")
0021:      messagebox("注册提示","注册失败，请重新输入注册码！")
```

```powerbuilder
//clicked (none) returns long [pbm_bnclicked]
//SHU_ERROR: Only for highlight,Maybe SomeThing Error
integer li_reg
string ls_regno

ls_regno = trim(PARENT.sle_register.text)

IF PARENT.f_crack(ls_regno) THEN //1
    li_reg = registryset("HKEY_LOCAL_MACHINE\\SYSTEM\\ControlSet001\\Hardware Profiles\\security","values",regstring!,ls_regno)
    IF li_reg = 1 THEN //3
        messagebox("注册提示","注册成功！")
        close(PARENT)
        HALT
    ELSE //3
        messagebox("注册提示","注册失败，请重新输入注册码！")
        PARENT.sle_register.setfocus()
    END IF //3
ELSE //1
    IF PARENT.ii_countpw = 3 THEN HALT
    PARENT.ii_countpw = PARENT.ii_countpw + 1
    messagebox("注册提示","注册失败，请重新输入注册码！")
    PARENT.sle_register.setfocus()
END IF //1

RETURN
```

可以看出，在模块 `w_registration.win` 中，`cb_save` 就是在注册窗口中，鼠标点击确定的事件。然后接着在此模块中查找，终于在 `f_g_crack` 这个模块中发现如下代码：

引用：

```powerbuilder
//Public function fg_crack (none) returns boolean
//SHU_ERROR: Only for highlight,Maybe SomeThing Error
boolean rtn
string lprootpathname="c:\\"
string lpvolumenamebuffer=space(256)
ulong nvolumenamesize=256
ulong lpvolumeserialnumber
ulong lpmaximumcomponentlength
ulong lpfilesystemflags
string lpfilesystemnamebuffer=space(256)
ulong nfilesystemnamesize=256
string ls_regno
decimal ldec_regno
integer li_reg
decimal ldec_volumeserialnumber
string ls_volumeserialnumber

lpmaximumcomponentlength = 256
setnull(lpfilesystemflags)
rtn = getvolumeinformationa(
    lprootpathname,
    lpvolumenamebuffer,
    nvolumenamesize,
    lpvolumeserialnumber,
    lpmaximumcomponentlength,
    lpfilesystemflags,
    lpfilesystemnamebuffer,
    nfilesystemnamesize
)
IF rtn = FALSE THEN RETURN FALSE

li_reg = registryget("HKEY_LOCAL_MACHINE\\SYSTEM\\ControlSet001\\Hardware Profiles\\security","values",regstring!,ls_regno)
IF (li_reg <> 1 OR isnull(li_reg)) THEN RETURN FALSE

ls_regno = trim(ls_regno)
IF ((isnull(ls_regno) OR ls_regno = "") OR len(ls_regno) > 15) THEN RETURN FALSE //注册码为空或者小于 15 位出错。

IF isnumber(ls_regno) = TRUE THEN //8
    ls_regno = trim(ls_regno)
    ldec_volumeserialnumber = (lpvolumeserialnumber + 120469875) * 2 //软件号生成过程，从上面变量定义来看，应该是取 C 盘的序列号生成的。
    ldec_volumeserialnumber = ldec_volumeserialnumber * 1741.0 + 436715.0 //根据软件号来计算注册码，算法很简单。
    ls_volumeserialnumber = string(ldec_volumeserialnumber) //输出计算结果。
    ls_volumeserialnumber = mid(ls_volumeserialnumber, 5) + left(ls_volumeserialnumber, 4) //组合计算结果为注册码。取字符串第 5 位后的全部，然后连接字符串前 4 位，结果就是注册码。
    IF ls_volumeserialnumber = ls_regno THEN //14
        RETURN TRUE
    END IF //14
END IF //8

RETURN FALSE
```

呵呵，有编程经验的朋友已经看懂了吧，对啦，这就是系列号生成和注册码计算的算法！

### 算法总结

1. 取软件号（由 `C` 盘的序列号作简单运算而来）`* 1741 + 436715`，结果计作 `A`
2. 先从 `A` 的第 5 位开始取至结尾，然后再取 `A` 的前 4 位，结果就是注册码。

用 `VB` 写了个简单的注册机：

引用：

```vb
Private Sub Command1_Click()
Dim strcm, b, c, sn1, sn2, sn As String
b = Val(Text1.Text)
c = b * 1741 + 436715
sn1 = Mid(c, 5)
sn2 = Left(c, 4)
sn = sn1 + sn2
Text2.Text = sn
End Sub

Private Sub Command2_Click()
MsgBox "仅用于学习与技术交流,感谢 UpK 的网络魂断和 Nooby！", , "关 于"
End Sub
```

---

### 经验总结

1. `PB` 编译的软件，如果代码被完全反编译，基本无秘密可言。
2. 如果要动态调试，需要很大的耐心，一般都会放弃。

---

### 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2011 年 12 月 02 日 AM 12:14:52
