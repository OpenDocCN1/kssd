
# File Shredder 2000

> 原文：[`www.pediy.com/kssd/pediy05/pediy50667.htm`](https://www.pediy.com/kssd/pediy05/pediy50667.htm)

破解对象：`File Shredder 2000` 一款不错的清理工具

工具：`trw2000`, `wd32asm` 黄金版，`fi`

## 过程如下

用 `wd32asm` 反汇编查找

String Resource ID=`05001`: "Software registration was successfully completed. Thank you"

得到：

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  `:00402718(U), :00402724(C)`

```
:0040273F 8B550C                   mov edx, dword ptr [ebp+0C]
:00402742 83C232                   add edx, 00000032
:00402745 52                       push edx
:00402746 8B450C                   mov eax, dword ptr [ebp+0C]
:00402749 50                       push eax
:0040274A E8613B0000               call 004062B0；计算注册码
:0040274F 83C408                   add esp, 00000008
:00402752 3985F8FEFFFF             cmp dword ptr [ebp+FFFFFEF8], eax；关键比较，
                                                     在此下断点可看到注册码
:00402758 741B                     je 00402775； ；正确跳到注册成功
:0040275A 68CFEA0000               push 0000EACF
```

## 下面是注册码运算过程

* Referenced by a CALL at Addresses:
  `:00402479 , :0040270A , :00402731 , :0040274A , :0040518D`

```
:004062B0 8B442404                 mov eax, dword ptr [esp+04]；输入的用户名
:004062B4 56                       push esi
:004062B5 8B35D4FE4100             mov esi, dword  ptr [0041FED4] ；一个常数：95989598
:004062BB 50                       push eax      
:004062BC 81CE78030000             or esi, 00000378
:004062C2 E869020000               call 00406530 ；第一步计算
:004062C7 8B4C2410                 mov ecx, dword ptr [esp+10]；输入的公司名
:004062CB 03F0                     add esi, eax
:004062CD 51                       push ecx
:004062CE E85D020000               call 00406530 ；第二步计算
:004062D3 83C408                   add esp, 00000008
:004062D6 03C6                     add eax, esi；eax=注册码
:004062D8 5E                       pop esi
:004062D9 C3                       ret
```

## 下面是整个上面两个运算注册码的过程

* Referenced by a CALL at Addresses:
  `:004026C4 , :004062C2 , :004062CE`

```
:00406530 51                       push ecx
:00406531 53                       push ebx
:00406532 8B5C240C                 mov ebx, dword ptr [esp+0C]
:00406536 56                       push esi
:00406537 33F6                     xor esi, esi
:00406539 53                       push ebx
:0040653A 8974240C                 mov dword ptr [esp+0C], esi

* Reference To: KERNEL32.lstrlenA, Ord:0308h

                                                     |
:0040653E FF15D4414100               Call dword ptr [004141D4]
:00406544 85DB                     test ebx, ebx
:00406546 744F                     je 00406597
:00406548 85C0                     test eax, eax
:0040654A 744B                     je 00406597
:0040654C 33D2                     xor edx, edx
:0040654E 85C0                     test eax, eax
:00406550 7E45                     jle 00406597
:00406552 55                       push ebp
:00406553 57                       push edi

* Possible StringData Ref from Data Obj ->"|b!pz*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw9"
                                                     ->"4cmqpfhw"

                                                     |
:00406554 BE24774100               mov esi, 00417724
:00406559 BF01000000               mov edi, 00000001
:0040655E 2BF3                     sub esi, ebx
:00406560 8BCB                     mov ecx, ebx
:00406562 2BFB                     sub edi, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040658D(C)
  |
:00406564 0FBE1C0E                 movsx ebx, byte ptr [esi+ecx]；系统给定的一串字母就在上面
:00406568 0FBEAC10EC764100         movsx ebp, byte ptr [eax+edx+004176EC]；系统给定的一串字母
:00406570 0FAFDD                   imul ebx, ebp
:00406573 8D2C0F                   lea ebp, dword ptr [edi+ecx];ebp=第几次循环
:00406576 0FAFDD                   imul ebx, ebp
:00406579 0FBE29                   movsx ebp, byte ptr [ecx]；你输入的姓名或公司名
                                                     的 ASCII 码值按字节取
:0040657C 0FAFDD                   imul ebx, ebp
:0040657F 8B6C2410                 mov ebp, dword ptr [esp+10]
:00406583 03EB                     add ebp, ebx；累加直到结束
:00406585 42                       inc edx
:00406586 41                       inc ecx
:00406587 3BD0                     cmp edx, eax
:00406589 896C2410                 mov dword ptr [esp+10], ebp
:0040658D 7CD5                     jl 00406564
:0040658F 8BC5                     mov eax, ebp
:00406591 5F                       pop edi
:00406592 5D                       pop ebp
:00406593 5E                       pop esi
:00406594 5B                       pop ebx
:00406595 59                       pop ecx
:00406596 C3                       ret
```

## 算法总结与 VB6 程序

以上算法总结如下，并附 `VB6` 程序一个供参考。

这是乘数集合：

`#serB&nz|mfM1/5(!sd$Mq.{s]+sFjtKpzSdtzoXqmb^Al@dv:s?x/`

这是被乘数集合：

`|b!pz*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw`

算法如下：

先计算姓名的位数 `i`，然后用被乘数*乘数（开始的是乘数的第 `i` 位）再乘以

第几次循环再乘以姓名的 `ASCII` 码，进行累加。

计算结束加上一个常量 `a=959897f8`

然后计算公司名的位数 `j`，用同样的运算方法

最后都加到 `a` 上即可！

### `vb6` 程序

```vb
Dim beichengshu As String
Dim chengshu As String
Dim mima
Dim xingming As String
Dim gongsiming As String
Dim c

Private Sub Command1_Click()
a = Len(Text1.Text)
b = Len(Text2.Text)

If a Or b = "" Then
  MsgBox "请输入你的姓名和公司名", vbOKOnly
End If

xingming = Text1.Text
gongsiming = Text2.Text

For i = 1 To a
  c = Asc(Mid$(beichengshu, i, 1)) * Asc(Mid$(chengshu, i + a, 1)) * i * Asc(Mid$(xingming, i, 1))
  mima = mima + c
Next i

mima = mima + 2509805560#

For i = 1 To b
  c = Asc(Mid$(beichengshu, i, 1)) * Asc(Mid$(chengshu, i + b, 1)) * i * Asc(Mid$(gongsiming, i, 1))
  mima = mima + c
Next i

Text3.Text = mima
End Sub

Private Sub Command2_Click()
Unload Me
End
End Sub

Private Sub Form_Load()
mima = 0
beichengshu = "|b!pz*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw"
chengshu = "#serB&nz|mfM1/5(!sd$Mq.{s]+sFjtKpzSdtzoXqmb^Al@dv:s?x/"
End Sub
```

以上程序经过 `VB6` 调试  `4dd466d0e1e0cc123afbe504da2462ca`
