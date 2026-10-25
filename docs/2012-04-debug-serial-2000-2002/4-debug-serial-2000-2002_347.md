
# HEdit 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum649.htm`](https://www.pediy.com/kssd/pediy03/forum649.htm)

HEdit 2.0 的注册破解过程

HEdit 2.0 by Yuri Software

cracck by PcRocker（本文不得用于任何商业用途，转载时请保持完整性）

功能： 文件编辑器

最新版本： 不详

下载地址： 不详

最近的破解没什么进展，主要是碰到的 APP 都是用上网注册的方式，很难找到注册码，头都大了 :-（

无奈之下，只有挑软柿子捏了，大家不要笑我。

给程序打补丁当然少不了文件编辑器，如果你只为修改文件 HEdit 是相当好用的：优点是程序本身很小，不管文件有多大，打开的都非常快；缺点是执行一次以后，它会将自己作为所有的无程序关联文件的默认打开方式。

破解工具是 `SOFT-ICE`，写手记的工具嘛当然是 `W32DASM`。

该 APP 的试用期到了之后，启动时出现过期的对话框，要求输入注册码，当注册信息错误时，就告诉你错啦（还好，一声不响的让人心里没底）。话不多说，开始。

在 `S—ICE` 中下 `BPX MESSAGEBOXA`，注册失败后按 3 次 `F12` 拦截如下：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00405047(C)
|

:00405064 B801000000                mov eax, 00000001
:00405069 5E                        pop esi
:0040506A C3                        ret
:0040506B 56                        push esi      <---- 这种“天外飞仙”的调用好象经常出现
:0040506C 57                        push edi
:0040506D 8BF1                      mov esi, ecx
:0040506F E808AE0200                call 0042FE7C
:00405074 8B7804                    mov edi, dword ptr [eax+04]
:00405077 6A01                      push 00000001
:00405079 8BCE                      mov ecx, esi
:0040507B E870890100                call 0041D9F0
:00405080 85C0                      test eax, eax
:00405082 7435                      je 004050B9
:00405084 FF7660                    push [esi+60]
:00405087 FF765C                    push [esi+5C]
:0040508A 8BCF                      mov ecx, edi
:0040508C E82DEFFFFF                call 00403FBE      <---- 这个 CALL 是关键，看下面的分析
:00405091 85C0                      test eax, eax      <---- EAX=0 表示失败，比较少见
:00405093 7510                      jne 004050A5      <---- 如果不跳，让你死的好看
:00405095 6AFF                      push FFFFFFFF
:00405097 6A00                      push 00000000

\* Possible Reference to Dialog: DialogID_0088
                          |
\* Possible Reference to String Resource ID=00136: "The user name and/or registration number you entered are inc"
                          |
:00405099 6888000000                push 00000088
:0040509E E8BCFC0100                call 00424D5F      <---- 显示出错的消息框
:004050A3 EB14                      jmp 004050B9      <---- 你会停在这里

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405093(C)
|
:004050A5 FF7660                    push [esi+60]
:004050A8 FF765C                    push [esi+5C]
:004050AB 8BCF                      mov ecx, edi
:004050AD E8CFEEFFFF                call 00403F81
:004050B2 8BCE                      mov ecx, esi
:004050B4 E86A5A0100                call 0041AB23

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405082(C), :004050A3(U)
|
:004050B9 5F                        pop edi
:004050BA 5E                        pop esi
:004050BB C3                        ret
```

这里是计算与比较注册码的 `call 403FBE`:

\* Referenced by a CALL at Addresses:
```
|:00403A3D  , :00403CDE  , :00403E07  , :0040508C
|
```

```
:00403FBE 55                       push ebp
:00403FBF 8BEC                      mov ebp, esp
:00403FC1 81EC00010000              sub esp, 00000100
:00403FC7 6800010000                push 00000100
:00403FCC 8D8D00FFFFFF              lea ecx, dword ptr [ebp+FFFFFF00]
:00403FD2 FF7508                    push [ebp+08]
:00403FD5 51                        push ecx
:00403FD6 E8054F0000                call 00408EE0      <---- 将注册姓名放在 [EBP-100]
:00403FDB 83C40C                    add esp, 0000000C      <---- 到[EBP]的剩余字节以 00 填充
:00403FDE 8D8D00FFFFFF              lea ecx, dword ptr [ebp+FFFFFF00]
:00403FE4 51                        push ecx
:00403FE5 E856200100                call 00416040      <---- 将注册姓名转为大写，见下面
:00403FEA 83C404                    add esp, 00000004
:00403FED B904000000                mov ecx, 00000004

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404015(C)
|
:00403FF2 8BC1                      mov eax, ecx
:00403FF4 41                        inc ecx
:00403FF5 99                        cdq      <---- 相当于 MOV EDX，00
:00403FF6 33C2                      xor eax, edx
:00403FF8 2BC2                      sub eax, edx
:00403FFA 83E003                    and eax, 00000003      <---- 这几个语句使 EAX 按顺序取 0、1、2、3
:00403FFD 33C2                      xor eax, edx
:00403FFF 2BC2                      sub eax, edx
:00404001 8A940DFFFEFFFF            mov dl, byte ptr [ebp+ecx-00000101]  <---- 从第五个字节开始取一个字节
:00404008 30940500FFFFFF            xor byte ptr [ebp+eax-00000100], dl  <---- 与前四个字节中的一个取异或
:0040400F 81F900010000              cmp ecx, 00000100      <---- 注册姓名最长为 255 个字节
:00404015 7CDB                      jl 00403FF2
:00404017 8B8500FFFFFF              mov eax, dword ptr [ebp+FFFFFF00]  <---- 结果送 EAX
:0040401D 85C0                      test eax, eax
:0040401F 7D02                      jge 00404023
:00404021 F7D8                      neg eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040401F(C)
|
:00404023 39450C                    cmp dword ptr [ebp+0C], eax   <---- 与输入的数字简单的比较
:00404026 7509                      jne 00404031
:00404028 85C0                      test eax, eax
:0040402A B801000000                mov eax, 00000001
:0040402F 7502                      jne 00404033

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404026(C)
|
:00404031 33C0                      xor eax, eax      <---- EAX=0 就错了

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040402F(C)
|
:00404033 8BE5                      mov esp, ebp
:00404035 5D                        pop ebp
:00404036 C20800                    ret 0008
```

这个 `CALL` 是将小写字母转为大写，汇编很熟的就略过吧。

\* Referenced by a CALL at Address:
```
|:00403FE5
|
```

```
:00416040 83EC04                    sub esp, 00000004
:00416043 53                        push ebx
:00416044 56                        push esi
:00416045 57                        push edi
:00416046 55                        push ebp
:00416047 33ED                      xor ebp, ebp
:00416049 392D984C4400              cmp dword ptr [00444C98], ebp
:0041604F 752C                      jne 0041607D
:00416051 8B5C2418                  mov ebx, dword ptr [esp+18]
:00416055 8BC3                      mov eax, ebx
:00416057 803B00                    cmp byte ptr [ebx], 00    <---- 注册姓名是否为空
:0041605A 7417                      je 00416073

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00416071(C)
|
:0041605C 8A08                      mov cl, byte ptr [eax]
:0041605E 80F961                    cmp cl, 61      <---- 61 is 'a'
:00416061 7C0A                      jl 0041606D
:00416063 80F97A                    cmp cl, 7A      <---- 7a is 'z'
:00416066 7F05                      jg 0041606D
:00416068 80E920                    sub cl, 20
:0041606B 8808                      mov byte ptr [eax], cl

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00416061(C), :00416066(C)
|
:0041606D 40                        inc eax
:0041606E 803800                    cmp byte ptr [eax], 00
:00416071 75E9                      jne 0041605C

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041605A(C)
|
:00416073 8BC3                      mov eax, ebx
:00416075 5D                        pop ebp
:00416076 5F                        pop edi
:00416077 5E                        pop esi
```



:00416078 5B  pop ebx

:00416079 83C404  add esp, 00000004

:0041607C C3  ret

用我名字的注册码----name：`pcrocker`（与大小写无关）code：`488048659`

该 APP 的注册支持一个字节：如‘a’<-----> `65`，也支持汉字：‘我爱你’<-----> `1364184822`（谁的手机号）

原理清楚后，写注册机就很容易，我这里 WINDOWS 下的编辑器只有 VB，可惜我不愿用也不会用，哪位有兴趣就写出注册机。

感觉采用这种注册形式的 APP 比较憨厚，最容易 CRACCK，不要破的太过分。

时间：2000-02-23

先在 Excel 中录制一个宏，然后编辑它，用下面的程序覆盖原有的空宏。

A1 里填上名字（用英文，我不会解决中文的问题 :( ）。

运行宏，A2 中应该是注册码了。:)

我的编程水平很差，别笑话我。

\------------------ cut here -----------------------

```vb
Option Base 1

Sub 宏1()

'

Dim HAryNum(4) As Integer, HAryStr(4) As String

Dim OutCode As Long, InName As String, CurStrNum As String

Dim i As Integer, Dgt As Integer

Dim OCstr As String

InName = Range("A1").Value

InName = UCase(InName)

For i = 1 To 4
    HAryNum(i) = Asc(Mid(InName, i, 1))
Next

For i = 5 To &H100
    Dgt = ((i - 1) And 3) + 1
    If i <= Len(InName) Then
        CurStrNum = Asc(Mid(InName, i, 1))
    Else
        CurStrNum = &H0
    End If
    HAryNum(Dgt) = HAryNum(Dgt) Xor CurStrNum
Next

For i = 4 To 1 Step -1
    HAryStr(i) = Hex(HAryNum(i))
    If Len(HAryStr(i)) = 1 Then
        HAryStr(i) = "0" & HAryStr(i)
    End If
    OCstr = OCstr + HAryStr(i)
Next

OutCode = "&H" + OCstr

Range("A2").Value = OutCode

'

End Sub
```

\------------------ end ------------------  `4dd466d0e1e0cc123afbe504da2462ca`
