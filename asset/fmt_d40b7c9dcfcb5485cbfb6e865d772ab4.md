
软件的功能概述：

通过 Internet 打电话的软件，据说效果很不错。我没用过。

保护方法：

时间限制。用 `Rsagnt32.dll` 来保护。

破解过程：

用 `bpx hmemcpy` 中断。被 `SoftIce` 拦截后，(`BD *`) 清除所有断点，连续按 `F12` 一直来到 `Rsagnt32.dll` 的代码区，代码窗和命令窗之间的分隔线变为：

```
--------RSAGNT32!.TEXT+nnnnn----------
```

再 `F10` 一路按下。

```
* Reference To: USER32.GetDlgItemTextA, Ord:00EDh
                                  |
:10001E8C FF1578050310            Call dword ptr [10030578]
     ^^^^
（:00691E8C<-----在 debug 时，看到的地址形式,其余类推。）
```

```
:10001e92 b9ffffffff             mov ecx, FFFFFFFF<--------|连续按 f12 后到这里。
:10001e97 2bc0                    sub eax, eax              |这是计算字符个数的典型语句段。
:10001E99 F2                      repnz                     |ECX 的值就是字符个数。
:10001E9A AE                      scasb                     |
:10001E9B F7D1                    not ecx                   |
:10001E9D 49                      dec ecx<------------------|
:10001e9e 83f90a                  cmp ecx, 0000000A<---------unlock
7440 code 的字符数应为 10(d)
:10001ea1 je 10001EE3
:10001EA3 8D442474                lea eax, dword ptr [esp+74]
```

```
* Possible StringData Ref from Data Obj ->"Sorry, that unlocking code is "
                                        ->"not valid for this program."
                                  |
(略)
* Possible StringData Ref from Data Obj ->"Invalid Unlocking Code"
                                  |
(略)
* Reference To: USER32.MessageBoxA, Ord:0188h
                                  |
(略)
:10001EE2 C3                      ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001EA1(C)
|
:10001EE3 BFD03A0210              mov edi, 10023AD0<-------------|;personal code 的地址
:10001ee8 b9ffffffff              mov ecx, FFFFFFFF              |
:10001EED 2BC0                    sub eax, eax                   |
:10001EEF F2                      repnz                          |
:10001EF0 AE                      scasb                          |
:10001EF1 F7D1                    not ecx                        |
:10001EF3 2BF9                    sub edi, ecx                   |
:10001EF5 8BC1                    mov eax, ecx                   |
:10001EF7 C1E902                  shr ecx, 02                    |
:10001EFA 8BF7                    mov esi, edi                   |
:10001EFC 8D7C240C                lea edi, dword ptr [esp+0C]    |拷贝 personal code
:10001F00 F3                      repz                           |
:10001F01 A5                      movsd                          |
:10001F02 8BC8                    mov ecx, eax                   |
:10001F04 83E103                  and ecx, 00000003              |
:10001F07 F3                      repz                           |
:10001F08 A4                      movsb<-------------------------|
:10001f09 8d4c2440                lea ecx, dword ptr [esp+40]
:10001F0D A1B4780210              mov eax, dword ptr [100278B4]
:10001F12 51                      push ecx
:10001F13 05B6000000              add eax, 000000B6
:10001F18 8D4C2410                lea ecx, dword ptr [esp+10]
:10001F1C 50                      push eax
:10001F1D 51                      push ecx
:10001F1E E8BD170000              call 100036E0<----------------计算 unlock code 的过程
:10001f23 8d4c244c                lea ecx, dword ptr [esp+4C]<--real ucode 的地址(1)
:10001f27 83c40c                  add esp, 0000000C
:10001F2A 68B0840210              push 100284B0<----------------fake
51 ucode 的地址(2)
:10001f2f push ecx
:10001F30 E84B8A0100              call 1001A980<----------------真假注册码的比较过程
:10001f35 83c408                  add esp, 00000008
:10001F38 85C0                    test eax, eax
:10001F3A A1B4780210              mov eax, dword ptr [100278B4]
:10001F3F 7512                    jne 10001F53<---------------------------------(3)
:10001f41 66c740040100            mov [eax+04], 0001
:10001F47 33C0                    xor eax, eax
:10001F49 5F                      pop edi
:10001F4A 5E                      pop esi
:10001F4B 5B                      pop ebx
:10001F4C 81C430010000            add esp, 00000130
:10001F52 C3                      ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001F3F(C)
|
:10001F53 8D4C2474                lea ecx, dword ptr [esp+74]
```

```
* Possible StringData Ref from Data Obj ->"Sorry, that unlocking code is "
                                        ->"not valid for this program."

(略)
* Possible StringData Ref from Data Obj ->"Invalid Unlocking Code"
(略)
* Reference To: USER32.MessageBoxA, Ord:0188h
(略)
:10001F8C C3                      ret
```

按 `F10`，经过上面(1)处时，(`d ecx`) 在数据窗会看到真正的 `Unlock code`；到(2)处时，(`d 6984B0`) 会看到我输入的假注册码。
再看(3)处的跳转语句，可以看到 `call 1001A980` 的返回值 `eax` 应为 `0`，否则注册失败。而 `call 1001A980` 实际上就是真假注册码的比较过程。值得一提的是 Patch 程序时，应在 `call 1001A980` 内进行(只 PATCH 一处即可)；而不宜在 `call 1001A980` 外，因为程序在别的地方也调用了 `call 1001A980`。看下面：

```
********************************************************************
在该 DLL 中还发现另一处取得 fake code，计算 real code 和两者比较的代码：
********************************************************************
* Reference To: USER32.GetDlgItemTextA, Ord:00EDh
                                  |
:100030DA FF1578050310            Call dword ptr [10030578]
:100030E0 B9FFFFFFFF              mov ecx, FFFFFFFF
:100030E5 2BC0                    sub eax, eax
:100030E7 F2                      repnz
:100030E8 AE                      scasb
:100030E9 F7D1                    not ecx
:100030EB 49                      dec ecx
:100030EC 83F90A                  cmp ecx, 0000000A
:100030EF 743E                    je 1000312F
:100030F1 A1B4780210              mov eax, dword ptr [100278B4]
```

```
* Possible StringData Ref from Data Obj ->"Sorry, that unlocking code is "
                                        ->"not valid for this program."
                                  |
(略)
* Possible StringData Ref from Data Obj ->"Invalid Unlocking Code"
                                  |
(略)
* Reference To: USER32.MessageBoxA, Ord:0188h
                                  |
(略)
:1000312E C3                      ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100030EF(C)
|
:1000312F 6860C60110              push 1001C660
:10003134 A1B4780210              mov eax, dword ptr [100278B4]
:10003139 05B6000000              add eax, 000000B6
:1000313E 50                      push eax
:1000313F 68E83A0210              push 10023AE8
:10003144 E897050000              call 100036E0<----------------计算 unlock code 的过程
:10003149 83c40c                  add esp, 0000000C
:1000314C 6850C60110              push 1001C650<----------------fake ucode 的地址
:10003151 6860c60110              push 1001C660<----------------real ucode 的地址
:10003156 e825780100              call 1001A980<----------------真假注册码的比较过程
:1000315b 83c408                  add esp, 00000008
:1000315E 85C0                    test eax, eax
:10003160 A1B4780210              mov eax, dword ptr [100278B4]
:10003165 7511                    jne 10003178
:10003167 66C740040100            mov [eax+04], 0001
:1000316D 33C0                    xor eax, eax
:1000316F 5F                      pop edi
:10003170 5E                      pop esi
:10003171 81C400010000            add esp, 00000100
:10003177 C3                      ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10003165(C)
|
:10003178 8D4C2408                lea ecx, dword ptr [esp+08]
```

```
* Possible StringData Ref from Data Obj ->"Sorry, that unlocking code is "
                                        ->"not valid for this program."
(略)
* Possible StringData Ref from Data Obj ->"Invalid Unlocking Code"
(略)
* Reference To: USER32.MessageBoxA, Ord:0188h
(略)
:100031B0 C3                      ret
```

最后要和网友共享成果：(1) PATCH 这个 DLL 并把它发布到网上，使网友们可以以任意 `Unlock Code` 注册，而不用管 `Personal Code` 是什么。(2) 作出注册机，使不会破解的网友可以由自己的 `Personal Code` 计算出 `Unlock Code`。下面就是 `Unlock Code` 的计算过程。

```
* Referenced by a CALL at Addresses:********************************************************************注册码计算过程
|:10001F1E   , :10003144   
|
:100036E0 83EC18                  sub esp, 00000018
```

```
* Possible StringData Ref from Data Obj ->"PQYZMERUOA"
                                  |
:100036E3 B948C90110              mov ecx, 1001C948
:100036E8 8D542400                lea edx, dword ptr [esp]
:100036EC 53                      push ebx
:100036ED 56                      push esi
:100036EE 8B01                    mov eax, dword ptr [ecx]
:100036F0 57                      push edi
:100036F1 8B5904                  mov ebx, dword ptr [ecx+04]
:100036F4 55                      push ebp
:100036F5 8902                    mov dword ptr [edx], eax
:100036F7 668B6908                mov bp, word ptr [ecx+08]
:100036FB 8A410A                  mov al, byte ptr [ecx+0A]
```

```
* Possible StringData Ref from Data Obj ->"JIKLICBXZC"
                                  |
:100036FE B93CC90110              mov ecx, 1001C93C
:10003703 895A04                  mov dword ptr [edx+04], ebx
:10003706 66896A08                mov word ptr [edx+08], bp
:1000370A 8B5904                  mov ebx, dword ptr [ecx+04]
:1000370D 668B6908                mov bp, word ptr [ecx+08]
:10003711 88420A                  mov byte ptr [edx+0A], al
:10003714 8D54241C                lea edx, dword ptr [esp+1C]
:10003718 8B01                    mov eax, dword ptr [ecx]
:1000371A 6633FF                  xor di, di
:1000371D 8902                    mov dword ptr [edx], eax
:1000371F 8A410A                  mov al, byte ptr [ecx+0A]
:10003722 8B742434                mov esi, dword ptr [esp+34]
:10003726 8B4C2430                mov ecx, dword ptr [esp+30]
:1000372A 895A04                  mov dword ptr [edx+04], ebx
:1000372D 66896A08                mov word ptr [edx+08], bp
:10003731 88420A                  mov byte ptr [edx+0A], al
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000377B(C)
|
:10003734 8B44242C                mov eax, dword ptr [esp+2C]
:10003738 B219                    mov dl, 19
:1000373A 0FBFEF                  movsx ebp, di
:1000373D B319                    mov bl, 19
:1000373F 8A440500                mov al, byte ptr [ebp+eax]
:10003743 32442C10                xor al, byte ptr [esp+ebp+10]
:10003747 32440D00                xor al, byte ptr [ebp+ecx]
:1000374B 2AE4                    sub ah, ah
:1000374D F6F2                    div dl
:1000374F 8AC4                    mov al, ah
:10003751 8A542C1C                mov dl, byte ptr [esp+ebp+1C]
:10003755 0441                    add al, 41
:10003757 88443500                mov byte ptr [ebp+esi], al
:1000375B 32540D00                xor dl, byte ptr [ebp+ecx]
:1000375F 32D0                    xor dl, al
:10003761 8AC2                    mov al, dl
:10003763 2AE4                    sub ah, ah
:10003765 F6F3                    div bl
:10003767 8AC4                    mov al, ah
:10003769 0441                    add al, 41
:1000376B 3C4F                    cmp al, 4F
:1000376D 7502                    jne 10003771
:1000376F B058                    mov al, 58
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000376D(C)
|
:10003771 6647                    inc di
:10003773 88443500                mov byte ptr [ebp+esi], al
:10003777 6683FF0A                cmp di, 000A
:1000377B 7CB7                    jl 10003734
:1000377D C6460A00                mov [esi+0A], 00
:10003781 5D                      pop ebp
:10003782 5F                      pop edi
:10003783 5E                      pop esi
:10003784 5B                      pop ebx
:10003785 83C418                  add esp, 00000018
:10003788 C3                      ret
```

把上面的汇编语句翻译成 VB6，是下面的样子：

注册机源程序:

```vb
Private Sub Command1_Click()
  
  Dim n(1 To 10) As Integer
  Dim c(1 To 10) As Integer
  Dim d(1 To 10) As Integer
  Dim e(1 To 10) As Integer
  Dim i As Integer
  Dim t As Integer
  Dim ucode As String
  
  For i = 1 To 10
    c(i) = Asc(Mid$("PQYZMERUOA", i, 1))
  Next i
  For i = 1 To 10
    d(i) = Asc(Mid$("NVEW2973hb", i, 1))
  Next i
  For i = 1 To 10
    e(i) = Asc(Mid$("JIKLICBXZC", i, 1))
  Next i
  For i = 1 To Len(Trim$(Text1.Text))
   Text1.SelStart = i - 1
   Text1.SelLength = 1
   n(i) = Asc(Text1.SelText)
  Next i
  
  ucode = ""
  For i = 1 To 10
    t = (((n(i) Xor c(i) Xor d(i)) Mod &H19 + &H41) Xor e(i) Xor d(i)) Mod &H19 + &H41
    If (t = &H4F) Then
      t = &H58
    End If
    ucode = ucode & Chr(t)
  Next i
  
  Text2.Text = ucode
  
End Sub

Private Sub Command2_Click()
 Unload Me
End Sub

Private Sub Form_Load()
  Me.Move 3500, 2500
  
End Sub

Private Sub Text1_Change()
 If Len(Trim$(Text1.Text)) >= 10 Then
  Command1.Enabled = True
 Else
  Command1.Enabled = False
  Text2.Text = ""
 End If
End Sub
```
