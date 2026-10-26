
## 汇编代码片段

```
00403F9C    .  8BC8                  mov ecx,eax                               ; |
00403F9E    .  E8 8EFD0100           call MpegJoin.00423D31                    ; \MpegJoin.00423D31
00403FA3    .  8B7E 74               mov edi,dword ptr ds:[esi+74]
00403FA6    .  E8 B5210200           call MpegJoin.00426160
00403FAB    .  8B40 04               mov eax,dword ptr ds:[eax+4]
00403FAE    .  57                    push edi                                  ; /Arg3
00403FAF    .  68 BCAA4200           push MpegJoin.0042AABC                    ; |Arg2 = 0042AABC ASCII "registration_code"
00403FB4    .  68 D0AA4200           push MpegJoin.0042AAD0                    ; |Arg1 = 0042AAD0 ASCII "Option"
00403FB9    .  8BC8                  mov ecx,eax                               ; |
00403FBB    .  E8 71FD0100           call MpegJoin.00423D31                    ; \MpegJoin.00423D31
00403FC0    .  5B                    pop ebx
00403FC1    .  5F                    pop edi
00403FC2    .  8BCE                  mov ecx,esi
00403FC4    .  5E                    pop esi
00403FC5    .  5D                    pop ebp
00403FC6    .  83C4 0C               add esp,0C
00403FC9    .  E9 118B0100           jmp MpegJoin.0041CADF
00403FCE    >  6A 00                 push 0
00403FD0    .  6A 00                 push 0
00403FD2    .  68 A0AE4200           push MpegJoin.0042AEA0                    ;  ASCII "Registration failed!"
00403FD7    .  E8 19FC0100           call MpegJoin.00423BF5
00403FDC    .  5B                    pop ebx
00403FDD    .  5F                    pop edi
00403FDE    .  5E                    pop esi
00403FDF    .  5D                    pop ebp
00403FE0    .  83C4 0C               add esp,0C
00403FE3    .  C3                    retn
```

## 算法注册机

```vb
Dim serial1, serial2, serial3, serial4, serial5, serial6 As Integer

Dim sum As Integer

Dim i As Integer

Private Sub Command1_Click()

If Len(Text1.Text) < 2 Then

PROMPT = MsgBox("用户名至少两位以上！", vbOKOnly, "ERROR")

End If

serial1 = (Asc(Mid(Text1.Text, 1, 1)) Or &H44) Mod &HA
Rem 计算注册码第1位

serial2 = (Asc(Mid(Text1.Text, 2, 1)) Or &H43) Mod &HA
Rem 计算注册码第2位

serial3 = (Asc(Mid(Text1.Text, 1, 1)) Or &H43) Mod &HA
Rem 计算注册码第3位

serial4 = (Asc(Mid(Text1.Text, 2, 1)) Or &H44) Mod &HA
Rem 计算注册码第4位

   sum = 0

    For i = 1 To Len(Text1.Text)

       sum = sum + Asc(Mid(Text1.Text, i, 1))

    Next i

serial5 = sum Mod &HA
Rem 计算注册码第5位

serial6 = Fix(Rnd * 900) + 100
Rem 随机产生三位数

Text2.Text = Trim(Str(serial1)) + Trim(Str(serial2)) + Trim(Str(serial3)) + Trim(Str(serial4)) + Trim(Str(serial5)) _
             + Trim(Str(serial6))
Rem text2显示的就是注册码

End Sub
```

## 备注

国内的软件比较难破,这个软件可能和`open Viedo Converter`软件作者是同一个人，算法一模一样！

## 版权声明

"破"文一篇，请随意转载，但请保持文章的完整性

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
