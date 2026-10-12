
# 长沙 vod 点歌系统

> 原文：[`www.pediy.com/kssd/pediy06/pediy6007.htm`](https://www.pediy.com/kssd/pediy06/pediy6007.htm)

**破解目标**：长沙 vod 点歌系统（注册算法分析）

**大小**：815K

**破解工具**：`W32Dasm 10.0`

**日期**：2003.5.3

**破解难度**：中

**下载地址**：无

**软件特征**：主文件 `vod.EXE` 未加壳，每次启动检测 `C:\WINDOWS\MACE.DLL`（注册码保存在 `MACE.DLL` 中）。注册码不正确则提示注册，机器码格式为 `SY_000000000000`，注册码格式为 `SN_000000000000`。

---

打开 `W32DasmV10.0` 参考串式数据参考 => `"C:\WINDOWS\MACE.DLL"` 来到：

```
* Reference To: MFC42.Ordinal:021C, Ord:021Ch
                          |
:0044ED38 E883410000              Call 00452EC0
:0044ED3D 6A00                    push 00000000
:0044ED3F 6A00                    push 00000000

* Possible StringData Ref from Data Obj -> "C:\WINDOWS\MACE.DLL"
                          |
:0044ED41 6828414700              push 00474128
:0044ED46 8D4C2428                lea ecx, dword ptr [esp+28]
:0044ED4A C68424AC00000001        mov byte ptr [esp+000000AC], 01

* Reference To: MFC42.Ordinal:144A, Ord:144Ah
                          |
:0044ED52 E8AD450000              Call 00453304
:0044ED57 8D442414                lea eax, dword ptr [esp+14]
:0044ED5B 8D4C241C                lea ecx, dword ptr [esp+1C]
:0044ED5F 50                      push eax
```

一路往下走来到：

```
* Reference To: MFC42.Ordinal:0217, Ord:0217h
                          |
:0044ED8E E865420000              Call 00452FF8       <-----将机器码翻转，取前 12 位。
:0044ED93 8D442410                lea eax, dword ptr [esp+10]
:0044ED97 8BCE                    mov ecx, esi
:0044ED99 50                      push eax
:0044ED9A E881210000              call 00450F20       <-----计算注册码的 call (跟进）
:0044ED9F 8B00                    mov eax, dword ptr [eax]    <<-----d eax  (正确注册码）
:0044EDA1 8B4C2414                mov ecx, dword ptr [esp+14]

* Reference To: MSVCRT._mbscmp, Ord:0159h
```

```
:00450F20 6AFF                    push FFFFFFFF
:00450F22 68270A4600             push 00460A27
:00450F27 64A100000000           mov eax, dword ptr fs:[00000000]
:00450F2D 50                     push eax
:00450F2E 64892500000000         mov dword ptr fs:[00000000], esp
:00450F35 83EC38                 sub esp, 00000038
:00450F38 56                     push esi
:00450F39 C744240800000000       mov [esp+08], 00000000
:00450F41 68B8434700             push 004743B8
:00450F46 8D4C2408               lea ecx, dword ptr [esp+08]
:00450F4A C744244801000000       mov [esp+48], 00000001

* Reference To: MFC42.Ordinal:0219, Ord:0219h       <<<--------算法
                          |
:00450F52 E8A7200000             Call 00452FFE
:00450F57 8D4C2450               lea ecx, dword ptr [esp+50]
:00450F5B C644244402             mov [esp+44], 02  <-----每次取处理过的机器码前两位
:00450F60 C744242449000000       mov [esp+24], 00000049  〈----
:00450F68 C74424281A000000       mov [esp+28], 0000001A     |
:00450F70 C744242C25000000       mov [esp+2C], 00000025     |
:00450F78 C744243034000000       mov [esp+30], 00000034     | --（简单的和运算）
:00450F80 C744243412000000       mov [esp+34], 00000012     |
:00450F88 C74424383F000000       mov [esp+38], 0000003F〈----
```

后面的几个 `call` 即将运算后的结果反转，在前面加 `SN_` 既为正确注册码。

下面是我用 VB 编写的注册机原代码！（哈哈！初学 VB，水平馊，大家别见笑）

```vb
Private Sub Command1_Click()
   A = Text1.Text
   k = Len(A)
   If k = 15 Then
   For b = 1 To k - 3
   Cryptograph = Cryptograph + Mid(A, k + 1 - b, 1)
   Next b
   For j = 1 To 6
    CryptographStr(j) = Mid(Cryptograph, j * 2 - 1, 2)
   Select Case j
   Case 6
   If (("&H" & (CryptographStr(6))) + &H3F) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(6))) + &H3F))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(6))) - &H3F)))
   End If
   Case 5
    If (("&H" & (CryptographStr(5))) + &H12) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(5))) + &H12))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(5))) - &H12)))
   End If
   Case 4
    If (("&H" & (CryptographStr(4))) + &H34) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(4))) + &H34))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(4))) - &H34)))
   End If
   Case 3
    If (("&H" & (CryptographStr(3))) + &H25) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(3))) + &H25))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(3))) - &H25)))
   End If
   Case 2
    If (("&H" & (CryptographStr(2))) + &H1A) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(2))) + &H1A))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(2))) - &H1A)))
   End If
   Case 1
    If (("&H" & (CryptographStr(1))) + &H49) < 255 Then
   PWstr = PWstr & CStr(Hex(("&H" & (CryptographStr(1))) + &H49))
   Else
   PWstr = PWstr & CStr(Hex(Not (Not ("&H" & (CryptographStr(1))) - &H49)))
   End If
   Case Else
    End
   End Select
    Next j
    For c = 1 To k - 3
    PassWord = PassWord + Mid(PWstr, 13 - c, 1)
    Next c
   Text2.Text = "SN_" & PassWord
   Else
   MsgBox ("机器码位数不对！！！")
   End If
   End Sub
```

完工！！！

johnroot  
2003.5.3 上午 于长沙

`4dd466d0e1e0cc123afbe504da2462ca`
