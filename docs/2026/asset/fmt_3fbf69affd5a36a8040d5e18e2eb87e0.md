
# [原创]ActArp算法分析

- 【文章标题】: ActArp算法分析
- 【文章作者】: Monster
- 【作者邮箱】: suphack@vip.qq.com
- 【作者主页】: a6aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3E0W2M7X3#2Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8R3`.`.
- 【作者QQ号】: 389264167
- 【软件名称】: ActArp
- 【软件大小】: 319 KB
- 【下载地址】: 自己搜索下载
- 【加壳方式】: UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo
- 【保护方式】: 注册码
- 【编写语言】: Borland Delphi 6.0 - 7.0
- 【使用工具】: OD+IDA+Dede
- 【操作平台】: WinXp
- 【软件介绍】: 就是一个Arp工具
- 【作者声明】: 原文地址：7d3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3E0W2M7X3#2Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4u0W2j5h3c8Q4x3X3g2H3K9s2m8Q4x3V1j5J5z5q4)9J5k6h3S2@1L8b7`.`.

---

## 详细过程

虽然在住院，可是今天还是收到黑X第三期的样刊，心里那个高兴啊，嘿嘿，打开光盘看了一下，有个叫ActArp（ARP扫描检测程序）的工具，打开看了一下，还不错，可是需要注册（要钱），郁闷，这可不是我的作风，偶给你破了，看你还要钱不。

先用PEID检查一下，结果是UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo，嘿嘿，UPX的壳还是很好脱地，用OD载入，遇到`pushad`指令后直接`hr esp`就可以搞定，这不是本文的重点，就不具体讲了。脱完后再使用PEID检查一下，结果是Borland Delphi 6.0 - 7.0，嘿嘿，不是VB就好，最讨厌搞VB的了。

先随便注册一下看看，输入用户名，序列号，注册码后按“确定”，就会弹出一个提示错误的对话框。

既然是Delphi的，那我们可以对他进行反编译一下，我们打开Dede，把目标程序加载进去，现在我们找到了“确定”按钮的“Click”事件的RVA，是`00466B30`。

现在我们再打开OD来分析，把脱壳后的程序载入OD，按`Ctrl+G`后会弹出一个小框，我们输入刚才得到的“确定”按钮的“Click”事件的RVA，按确定后就会来到`00466B30`所指的代码处，我们在这里按`F2`下一个断点。

我们在OD中按`F9`让程序运行起来，还是输入刚才的用户名，序列号，注册码后按“确定”，程序就会被断在了我们刚才下断的地方，我们按`F8`往下跟踪几步就会看到一个`jnz`指令，代码如下：

```
00466B88    84C0            test    al, al                           ; 比较语句
  00466B8A    75 25           jnz     short 00466BB1                   ; 注册成功则跳
  00466B8C    6A 10           push    10
  00466B8E    B9 606C4600     mov     ecx, 00466C60                    ; 错误
  00466B93    BA 686C4600     mov     edx, 00466C68                    ; 无效的注册码，请与销售人员联系。
  00466B98    A1 E4B24600     mov     eax, dword ptr [46B2E4]
  00466B9D    8B00            mov     eax, dword ptr [eax]
  00466B9F    E8 0070FFFF     call    0045DBA4                         ; 弹出错误对话框
```

我们可以看到，`00466B8A`外的`jnz`就是关键跳转，也就是我们常说的暴点，我们现在只要把这里修改成`jmp`就可以实现暴破了，可我们的目标不是暴破，我们来看看他的注册算法。

全段代码如下：

```
CODE:00466B35                 push    ecx
  CODE:00466B36                 push    ecx
  CODE:00466B37                 push    ecx
  CODE:00466B38                 push    ecx
  CODE:00466B39                 push    ecx
  CODE:00466B3A                 push    ebx
  CODE:00466B3B                 push    esi
  CODE:00466B3C                 mov     ebx, eax
  CODE:00466B3E                 xor     eax, eax
  CODE:00466B40                 push    ebp
  CODE:00466B41                 push    offset loc_466C50
  CODE:00466B46                 push    dword ptr fs:[eax]
  CODE:00466B49                 mov     fs:[eax], esp
  CODE:00466B4C                 lea     edx, [ebp+var_4]
  CODE:00466B4F                 mov     eax, [ebx+34Ch]
  CODE:00466B55                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取用户名
  CODE:00466B5A                 mov     eax, [ebp+var_4]
  CODE:00466B5D                 push    eax
  CODE:00466B5E                 lea     edx, [ebp+var_8]
  CODE:00466B61                 mov     eax, [ebx+344h]
  CODE:00466B67                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取序列号
  CODE:00466B6C                 mov     eax, [ebp+var_8]
  CODE:00466B6F                 push    eax
  CODE:00466B70                 lea     edx, [ebp+var_C]
  CODE:00466B73                 mov     eax, [ebx+33Ch]
  CODE:00466B79                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取注册码
  CODE:00466B7E                 mov     eax, [ebp+var_C]
  CODE:00466B81                 pop     edx
  CODE:00466B82                 pop     ecx
  CODE:00466B83                 call    sub_466024
  CODE:00466B88                 test    al, al
  CODE:00466B8A                 jnz     short loc_466BB1
  CODE:00466B8C                 push    10h             ; uType
  CODE:00466B8E                 mov     ecx, offset dword_466C60
  CODE:00466B93                 mov     edx, offset dword_466C68 ; lpText
  CODE:00466B98                 mov     eax, off_46B2E4
  CODE:00466B9D                 mov     eax, [eax]      ; int
  CODE:00466B9F                 call    @Forms@TApplication@MessageBox$qqrpxct1i ; Forms::TApplication::MessageBox(char *,char *,int)
  CODE:00466BA4                 xor     eax, eax
  CODE:00466BA6                 mov     [ebx+24Ch], eax
  CODE:00466BAC                 jmp     loc_466C35
  CODE:00466BB1 ; ---------------------------------------------------------------------------
  CODE:00466BB1
  CODE:00466BB1 loc_466BB1:                             ; CODE XREF: _TAboutBox_OKButtonClick+5A j
  CODE:00466BB1                 lea     edx, [ebp+var_10]
  CODE:00466BB4                 mov     eax, [ebx+33Ch]
  CODE:00466BBA                 call    @Controls@TControl@GetText$qqrv ; Controls::TControl::GetText(void)
  CODE:00466BBF                 mov     edx, [ebp+var_10]
  CODE:00466BC2                 mov     eax, off_46B360
  CODE:00466BC7                 call    @System@@LStrAsg$qqrpvpxv ; System::__linkproc__ LStrAsg(void *,void *)
  CODE:00466BCC                 lea     edx, [ebp+var_14]
  CODE:00466BCF                 mov     eax, [ebx+34Ch]
  CODE:00466BD5                 call    @Controls@TControl@GetText$qqrv ; Controls::TControl::GetText(void)
  CODE:00466BDA                 mov     edx, [ebp+var_14]
  CODE:00466BDD                 mov     eax, off_46B150
  CODE:00466BE2                 call    @System@@LStrAsg$qqrpvpxv ; System::__linkproc__ LStrAsg(void *,void *)
  CODE:00466BE7                 mov     dl, 1
  CODE:00466BE9                 mov     eax, off_465E40
  CODE:00466BEE                 call    sub_4664DC
  CODE:00466BF3                 mov     ebx, eax
  CODE:00466BF5                 mov     eax, off_46B360
  CODE:00466BFA                 mov     eax, [eax]
  CODE:00466BFC                 push    eax
  CODE:00466BFD                 mov     ecx, offset _str_User.Text
  CODE:00466C02                 mov     edx, offset _str_Register.Text
  CODE:00466C07                 mov     eax, ebx
  CODE:00466C09                 mov     esi, [eax]
  CODE:00466C0B                 call    dword ptr [esi+4]
  CODE:00466C0E                 mov     eax, off_46B150
  CODE:00466C13                 mov     eax, [eax]
  CODE:00466C15                 push    eax
  CODE:00466C16                 mov     ecx, offset _str_RegCode.Text
  CODE:00466C1B                 mov     edx, offset _str_Register.Text
  CODE:00466C20                 mov     eax, ebx
  CODE:00466C22                 mov     esi, [eax]
  CODE:00466C24                 call    dword ptr [esi+4]
  CODE:00466C27                 mov     eax, ebx
  CODE:00466C29                 mov     edx, [eax]
  CODE:00466C2B                 call    dword ptr [edx+54h]
  CODE:00466C2E                 mov     eax, ebx
  CODE:00466C30                 call    @System@TObject@Free$qqrv ; System::TObject::Free(void)
  CODE:00466C35
  CODE:00466C35 loc_466C35:                             ; CODE XREF: _TAboutBox_OKButtonClick+7C j
  CODE:00466C35                 xor     eax, eax
  CODE:00466C37                 pop     edx
  CODE:00466C38                 pop     ecx
  CODE:00466C39                 pop     ecx
  CODE:00466C3A                 mov     fs:[eax], edx
  CODE:00466C3D                 push    offset loc_466C57
  CODE:00466C42
  CODE:00466C42 loc_466C42:                             ; CODE XREF: _TAboutBox_OKButtonClick+125 j
  CODE:00466C42                 lea     eax, [ebp+var_14]
  CODE:00466C45
  CODE:00466C45 ; __fastcall Ibdatabaseinfo::TIBDatabaseInfo::GetReads(void)
  CODE:00466C45 @Ibdatabaseinfo@TIBDatabaseInfo@GetReads$qqrv:
  CODE:00466C45                 mov     edx, 5
  CODE:00466C4A                 call    @System@@LStrArrayClr$qqrpvi ; System::__linkproc__ LStrArrayClr(void *,int)
  CODE:00466C4F                 retn
```

可以看出`00466B83`处的`call sub_466024`就是算法，我们看一下上面的几句：
