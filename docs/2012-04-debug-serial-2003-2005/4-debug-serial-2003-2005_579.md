
# Magic Converter V3.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6418.htm`](https://www.pediy.com/kssd/pediy06/pediy6418.htm)

马甲透视之 `Magic Converter V3.1`

【目    标】: `Magic Converter V3.1`

【工    具】: `DEDE3.5`、`OllyDbg 1.1b`(DIY 版)

【任    务】:不脱马甲，’静态’找算法+动态得到注册码+注册机

【操作平台】: `WINDOWS 2000 ADV sp2`

【作    者】: `loveboom[DFCG]`

【相关链接】: `www.skycn.com`

【简要说明】：这是一个很好的图片转换工具，可以把图片转换成 `rtf`、`txt`、`html` 还可以在 `bmp2jpg`、`bmp2ico` 等等，还有不少像 `photoshop` 那样的滤镜功能呢，功能还真不少，难怪叫 `Magic Converter` 呢，个人感觉是一款不错的东东，如果可能的话，最好买正版，支持一下作者(再说正版的有相关的技术支持和以后的免费或付少量钱升级呢，某鸟言：“你是不是和原作者有一腿呀，怎么帮他打广告”，俺晕了!)。有动力的话作者会做更好的东西给大家用。

【详细过程】:

信息:

`Name:loveboom`

`SN:1314520`

安装完毕后，打开看看它的相关注册信息，又是一个不用在线注册东东，好心人哟!输入试试，没反应，这样知道它多数是正确后才显示信息。退出来用“照妖镜”看看，发它是有马 甲的，是 `ASPACK2.12` 加的壳(我们要不脱马甲直接透视它所以没先脱壳，再分析。)，再次运行程序进入主界面里看看同时看盾主程序的”个头”，这大概可猜出是 `borland` 公司的编工具写的程序，知道这些信息后,用 `OD` 找到入口再看一下就证实我猜的没错。现在当然要请出我们今天的“主角”`DEDE`，运行 `MC`(`MagicConverter`)后，用 `DEDE` 挂上 `MC` 的进程。现在清楚多了，等 `DEDE` 分析完毕之后，先打开窗体面板，看看对我们有用的东西，看到 `TregDlg` 没有，打开对象看看：

```
object Label6: TLabel
    Left = 16
    Top = 248
    Width = 87
    Height = 13
    Caption = 'Registration Code:'
  end

object Panel1: TPanel
    Tag = 255
    Left = 0
    Top = 311
    Width = 397
    Height = 41
    Align = alBottom
    BevelOuter = bvNone
    TabOrder = 0
    object Button1: TButton
      Left = 40
      Top = 8
      Width = 75
      Height = 25
      Caption = 'OK'
      Default = True
      TabOrder = 0
      OnClick = Button1Click    //这里我们可以知道按 OK 注册时是谁处理
    end
```

知道这些信息后，我们现在打开过程面板找到 `Dlg_Reg` 单元，在它右边双击 `Button1Click`,我看这过后分析到这些东西(错误之处还请高手指点一二)：

```
004BFE0E   64FF30            push    dword ptr fs:[eax]
004BFE11   648920            mov     fs:[eax], esp
004BFE14   8D55E8            lea     edx, [ebp-$18]
004BFE17   8B45FC            mov     eax, [ebp-$04]

* Reference to control NameBox : TEdit

004BFE1A   8B8014030000      mov     eax, [eax+$0314] { 传入 TEdit 的 ID }

* Reference to: Controls.TControl.GetText(TControl):TCaption;

004BFE20   E80FC1F8FF        call    0044BF34
004BFE25   837DE800          cmp     dword ptr [ebp-$18], +$00 { 判断用户名是否为空 }
004BFE29   7511              jnz     004BFE3C { 不为空就跳下一步 }
004BFE2B   A158625800        mov     eax, dword ptr [$00586258] { 你没有输入用户名的话，就到这里了。 }
004BFE30   8B00              mov     eax, [eax] { 传入没有输入用户名的信息，为什么这么肯定呢? }

004BFE32   E8098E0800        call    00548C40 { 因为我们没输入信息，它有提示信息嘛。这里相当于 MessageBoxA 显示 }
004BFE37   E933020000        jmp     004C006F { 这里跳去 Over 处 }

004BFE3C   8D55E4            lea     edx, [ebp-$1C]
004BFE3F   8B45FC            mov     eax, [ebp-$04]

* Reference to control NameBox : TEdit

004BFE42   8B8014030000      mov     eax, [eax+$0314] { 这里再次取用户名 }

* Reference to: Controls.TControl.GetText(TControl):TCaption;

004BFE48   E8E7C0F8FF        call    0044BF34
004BFE4D   8B45E4            mov     eax, [ebp-$1C]
004BFE50   8D55F4            lea     edx, [ebp-$0C]

004BFE53   E86C840800        call    005482C4 { 这里就是关键算法 Call }

004BFE58   8D55E0            lea     edx, [ebp-$20]
004BFE5B   8B45FC            mov     eax, [ebp-$04]

* Reference to control CodeBox : TEdit

004BFE5E   8B801C030000      mov     eax, [eax+$031C] { 这里取我们输入的假码 }

* Reference to: Controls.TControl.GetText(TControl):TCaption;

004BFE64   E8CBC0F8FF        call    0044BF34
004BFE69   8B45E0            mov     eax, [ebp-$20] { 假码入 eax }
004BFE6C   8B55F4            mov     edx, [ebp-$0C] { 真码入 edx，看到这里就有好戏看了 }

* Reference to: System.@LStrCmp;

004BFE6F   E8E84CF4FF        call    00404B5C { 这里很明显就是真假码比较,关键比较 }
004BFE74   0F85DE000000      jnz     004BFF58 { 不相等就 over 了 }

004BFE7A   B201              mov     dl, $01 { 如果正确的话就从这里开始把注册信息写入注册表 }
004BFE7C   A1400E4700        mov     eax, dword ptr [$00470E40]

* Reference to: Registry.TRegistry.Create(TRegistry;boolean);overload;

004BFE81   E8BA10FBFF        call    00470F40 { 看到了吧，都已经开始 TRegistry.Create 了，这更说明上面的判断是正确的 }
004BFE86   8945F0            mov     [ebp-$10], eax
004BFE89   33C0              xor     eax, eax
004BFE8B   55                push    ebp
004BFE8C   680CFF4B00        push    $004BFF0C

***** TRY
```

得到这些信息后，我们再双击打开关键算法 `Call` 看看它的算法是怎么回事。

进入后：

```
***** TRY

005482DF   64FF30            push    dword ptr fs:[eax]
005482E2   648920            mov     fs:[eax], esp
005482E5   85F6              test    esi, esi { 判断用户名情况 }
005482E7   0F84DC000000      jz      005483C9 { 这里跳就没得玩了 }

005482ED   8BC6              mov     eax, esi { 用户名入 EAX }

* Reference to: System.@LStrLen(String):Integer;

005482EF   E824C7EBFF        call    00404A18 { 计算用户名长度 }
005482F4   85C0              test    eax, eax { 这里测试长度是不是为 0 }
005482F6   0F8E85000000      jle     00548381 { 如果这里跳的话就 over 了 }

005482FC   8945F4            mov     [ebp-$0C], eax { 用户名长度入[ebp-c] }
005482FF   BB01000000        mov     ebx, $00000001 { 给 ebx 赋值 1 }

00548304   33C0              xor     eax, eax
00548306   8A441EFF          mov     al, byte ptr [esi+ebx-$01] { 这里开始依次取用户进行相关运算 }
0054830A   99                cdq
0054830B   F7FB              idiv    ebx  { 把取出的值整除 EBX,也就是 EAX=EAX/EBX,第一次 EBX=1 }
0054830D   8BF8              mov     edi, eax { 结果再保存一份到 edi 中 }
0054830F   8BC6              mov     eax, esi { 这里再次到用户名放到 EAX 中 }

* Reference to: System.@LStrLen(String):Integer;

00548311   E802C7EBFF        call    00404A18 { 计算长度 }
00548316   50                push    eax { 用户名长度入栈 }
00548317   8BC7              mov     eax, edi { 把 EDI 的值放回 eax 中 }
00548319   5A                pop     edx { 这里用户名长度出栈到 edx 中 }
0054831A   8BCA              mov     ecx, edx { 用户名长度再入 ecx }
0054831C   99                cdq
0054831D   F7F9              idiv    ecx  { 再用 EAX 的值整除 ECX 的值 EAX=EAX/ECX }
0054831F   33D2              xor     edx, edx { 清除 EDX }
00548321   8A541EFF          mov     dl, byte ptr [esi+ebx-$01] { 这句和上面的一样了，不过是取值放到 EDX 中 }
00548325   83C203            add     edx, +$03 { 把取出的值加上 3 }
00548328   F7EA              imul    edx  { 这里就是 EAX=EAX*EDX }
0054832A   8D4DF0            lea     ecx, [ebp-$10]
0054832D   BA03000000        mov     edx, $00000003 { 给 EDX 赋值,EDX=3 }

* Reference to: SysUtils.IntToHex(Integer;Integer):AnsiString;overload;

00548332   E88D0FECFF        call    004092C4 { 这里把上面的结果变换成字符串 }
00548337   8B55F0            mov     edx, [ebp-$10] { 结果传入 EDX }
0054833A   8D45F8            lea     eax, [ebp-$08]

* Reference to: System.@LStrCat;

0054833D   E8DEC6EBFF        call    00404A20 { 连接字符串 }
00548342   8B45F8            mov     eax, [ebp-$08] { 结果放入 eax 中 }

* Reference to: System.@LStrLen(String):Integer;

00548345   E8CEC6EBFF        call    00404A18 { 计算长度上面的字符串的长度 }
0054834A   50                push    eax { 长度入栈 }
0054834B   8BC7              mov     eax, edi { 这里看到 53830D 那里就知道怎么回事了 }
0054834D   5A                pop     edx { 长度出栈到 edx 中 }
0054834E   8BCA              mov     ecx, edx { 再把长度入 ECX }
00548350   99                cdq
00548351   F7F9              idiv    ecx  { 用 EAX 的值整除 ECX=EAX/ECX }
```



# 汇编代码分析

```
00548353   8B55F8                 mov     edx, [ebp-$08] { 这里对应上面的 548342 那里 }
00548356   0FB6541AFF             movzx   edx, byte ptr [edx+ebx-$01] { 再取运算后的第 N 位,N=循环次数 }
0054835B   83C202                 add     edx, +$02 { 取出的值加 2=EDX=EDX+2 }
0054835E   F7EA                   imul    edx  { 这里再 EAX=EAX*EDX }
00548360   8D4DEC                 lea     ecx, [ebp-$14]
00548363   BA02000000             mov     edx, $00000002 { 赋值 EDX=2 }

* Reference to: `SysUtils.IntToHex(Integer;Integer):AnsiString;` overload;

00548368   E8570FECFF             call    004092C4 { 这里也是变换成字符串 }
0054836D   8B55EC                 mov     edx, [ebp-$14] { 结果保存在 EDX 中 }
00548370   8B45FC                 mov     eax, [ebp-$04]

* Reference to: `System.@LStrCat;`

00548373   E8A8C6EBFF             call    00404A20 { 连接字符串 }
00548378   8B45FC                 mov     eax, [ebp-$04] { 把地址放到 EAX 中去 }
0054837B   43                     inc     ebx { EBX 自加 1 }
0054837C   FF4DF4                 dec     dword ptr [ebp-$0C] { 用户名长度自减 1 }
0054837F   7583                   jnz     00548304 { 如果没有运算完就跳回去继续 }
00548381   8B45FC                 mov     eax, [ebp-$04] { 保存真码的地址放入 EAX }
00548384   8B00                   mov     eax, [eax] { 这里取真码到 EAX 中 }

* Reference to: `System.@LStrLen(String):Integer;`

00548386   E88DC6EBFF             call    00404A18 { 计算运算后的字符串长度 }
0054838B   83F818                 cmp     eax, +$18 { 判断它是不是小于或等于 18(DEC 24) }
0054838E   7E27                   jle     005483B7 { 不是就跳 }
00548390   8D45E8                 lea     eax, [ebp-$18] { 大于 18 位的话就走这条路 }
00548393   50                     push    eax
00548394   8B45FC                 mov     eax, [ebp-$04]
00548397   8B00                   mov     eax, [eax] { 传入运算结果的字符串 }
00548399   B918000000             mov     ecx, $00000018 { 传入 18 }
0054839E   33D2                   xor     edx, edx

* Reference to: `System.@LStrCopy;`

005483A0   E8CBC8EBFF             call    00404C70 { 这里也就是如果运算结果大于 18 位的话就到前 18 位. }
005483A5   8B4DE8                 mov     ecx, [ebp-$18] { 取后的结果入 ecx 中 }
005483A8   8B45FC                 mov     eax, [ebp-$04]

* Possible String Reference to: `'MC3'`

005483AB   BA04845400             mov     edx, $00548404 { 这里传入 MC3 这个字符串 }

* Reference to: `System.@LStrCat3;`

005483B0   E8AFC6EBFF             call    00404A64 { 这里就是把 MC3 和运算后的结果连接起来 }
005483B5   EB12                   jmp     005483C9 { 这里跳去返回的地方了 }
005483B7   8B4DFC                 mov     ecx, [ebp-$04] { 如果运算结果小于或等于 18 位就到这条路 }
005483BA   8B09                   mov     ecx, [ecx] { 结果传入 ECX 中 }
005483BC   8B45FC                 mov     eax, [ebp-$04]

* Possible String Reference to: `'MC3'`

005483BF   BA04845400             mov     edx, $00548404 { 传入固定值"MC3' }

* Reference to: `System.@LStrCat3;`

005483C4   E89BC6EBFF             call    00404A64 { 这里同样进行连接操作 }
005483C9   33C0                   xor     eax, eax { 到这里也就得到了程序的全部算法了。 }
005483CB   5A                     pop     edx
005483CC   59                     pop     ecx
005483CD   59                     pop     ecx
005483CE   648910                 mov     fs:[eax], edx
```

## 验证与总结

看完这些后，我怕我的判断有误，所以用 `OD` 再跟了一次，所以就找到了自己的注册码。

现在已经知道注册算法了吧。我就懒一下不写总结了。如果你看不明白的话，那 `VB` 源码应该没问题吧：

## 注册机源码 (VB.Net)

```vb.net
    '//本注册机在 VB.net 下通过测试,好久没用 VB，所以乱了好多.
    
        Dim szValue As String, i As Long, lenszValue As Long, EAsc As Integer, Easc1 As Integer, szEasc As String, Rlen As Integer
        Dim Easc2 As Integer, szRet As String, szEasc1 As String, Easc3 As Integer
        If Len(Trim(TextBox1.Text)) = 0 Then
            MsgBox("Please Enter your name!")
            Exit Sub
        End If
        If Len(TextBox1.Text) > 10 Then
            MsgBox("Your name is too long.")
            Exit Sub
        End If
        szValue = TextBox1.Text
        lenszValue = Len(szValue)
        For i = 1 To lenszValue
            EAsc = Asc(Mid(szValue, i, 1))
            Easc1 = EAsc
            EAsc = Int(EAsc / i)
            Easc3 = EAsc
            EAsc = Int(EAsc / lenszValue)
            Easc1 = Easc1 + 3
            EAsc = EAsc * Easc1
            szEasc = Hex$(EAsc)
            szEasc1 = szEasc1 & szEasc
            Rlen = Len(szEasc1)
            Easc2 = Int(Easc3 / Rlen)
            EAsc = Asc(Mid(szEasc1, i, 1))
            EAsc = EAsc + 2
            Easc2 = Easc2 * EAsc
            If Easc2 = 0 Then
                szRet = szRet & "00"
            Else
                szRet = szRet & Hex$(Easc2)
            End If
        Next
        If Len(szRet) > 24 Then
            szRet = Mid(szRet, 1, 24) '//如果是 VB6 的话直接 Left$(szRet, 1) 就可以了，但 VB.net 不再有 left$函数
        End If
        TextBox2.Text = "MC3" & szRet
```

全文完，谢谢你能够坚持到底,看我在这里“灌水”!

## 后记

**正确信息：**

- Name: `loveboom`
- SN: `MC37BC25BD46843450000`

这是我的第一次，第一次没脱马甲，没有动态调试，纯用 `DEDE`“静态”分析其算法，所以不可避免存在不足之处，还请各位老大多多指点。这应该是我第二次发不脱马甲来破解的文章吧，之所以现在不想脱壳来破解是因为，以前我中了壳的“毒”，遇到很多猛壳(我想应该不是壳猛，是我太菜的原因)，不知道如何脱，有的就脱壳后的自检验搞得我头晕(因为我太菜了，所以经常看得头晕!)。所以只能搞小动作了。

BTW：生活就像强奸，不得不从.破解像泡妞，喜欢怎么样泡就怎么样泡。

献给我亲爱的 `DFCG`，愿它能够蒸蒸日上!

**Thanks:**

`Fly` `辉仔` `yock`、`jingulong`、二哥、所有曾经关心支持或帮助过我的朋友！谢谢您们!

by `loveboom[DFCG]`

Email: `bmd2chen@tom.com`

## 勘误与陷阱说明

感谢 `laoxuetong` 等朋友指点要不我中了作者的陷阱还不知道呢，因为当时发现提示成功后就没去管它了，我自己没有用那东西.

**修复信息如下：**

```
* Reference to: `Controls.TControl.GetText(TControl):TCaption;`

004BFE64   E8CBC0F8FF             call    0044BF34
004BFE69   8B45E0                 mov     eax, [ebp-$20]
004BFE6C   8B55F4                 mov     edx, [ebp-$0C]

* Reference to: `System.@LStrCmp;`

004BFE6F   E8E84CF4FF             call    00404B5C
004BFE74   0F85DE000000           jnz     004BFF58 { 这里要跳才行，不跳就中作者的陷阱了 }

……

004BFF4B   8B45FC                 mov     eax, [ebp-$04]

* Reference to: `Forms.TCustomForm.Close(TCustomForm);`

004BFF4E   E88D8BFAFF             call    00468AE0
004BFF53   E917010000             jmp     004C006F
004BFF58   8D55C8                 lea     edx, [ebp-$38] { 跳到这条路才是真正的注册 }
004BFF5B   8B45FC                 mov     eax, [ebp-$04] { 准备获取用户名 }

* Reference to control `NameBox` : `TEdit`

004BFF5E   8B8014030000           mov     eax, [eax+$0314]

* Reference to: `Controls.TControl.GetText(TControl):TCaption;`

004BFF64   E8CBBFF8FF             call    0044BF34 { 获取用户名 }
004BFF69   8B45C8                 mov     eax, [ebp-$38] { 用户名入 eax }
004BFF6C   8D55F8                 lea     edx, [ebp-$08]

004BFF6F   E838920800             call    005491AC { 这就是真正的算法 call 了 }
004BFF74   8D55C4                 lea     edx, [ebp-$3C]
004BFF77   8B45FC                 mov     eax, [ebp-$04]

* Reference to control `CodeBox` : `TEdit`

004BFF7A   8B801C030000           mov     eax, [eax+$031C]

* Reference to: `Controls.TControl.GetText(TControl):TCaption;`

004BFF80   E8AFBFF8FF             call    0044BF34
004BFF85   8B45C4                 mov     eax, [ebp-$3C] { 假码入 eax }
004BFF88   8B55F8                 mov     edx, [ebp-$08] { 真码入 edx }

* Reference to: `System.@LStrCmp;`

004BFF8B   E8CC4BF4FF             call    00404B5C
004BFF90   0F85D9000000           jnz     004C006F { 这里就不能再跳了，一跳真的 over 了 }
004BFF96   B201                   mov     dl, $01
```

**正确信息：**

- name: `loveboom`
- SN: `MC31A7D99BCAACB1754B754F903EE5CDA2D7`

**保存位置：**

```
[HKEY_LOCAL_MACHINE\SOFTWARE\KekSoft\Magic Converter\]

"FilePath"="C:\Program Files\KekSoft\Magic Converter\"
"Name"="loveboom"
```



`"Code"="MC31A7D99BCAACB1754B754F903EE5CDA2D7"`

`"ID"=dword:00000001`

`4dd466d0e1e0cc123afbe504da2462ca`
