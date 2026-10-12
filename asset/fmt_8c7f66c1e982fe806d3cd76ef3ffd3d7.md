
# 汇编代码分析

```
0040803E    50                 push eax                      ; 返回到这里
0040803F    8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
00408043    C68424 BC020000>   mov byte ptr ss:[esp+2BC],7
0040804B    E8 9C4B0300        call NetManag.0043CBEC       ; 检测 CALL，无需跟进！
00408050    8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
00408054    C68424 B8020000>   mov byte ptr ss:[esp+2B8],6
0040805C    E8 9E4A0300        call NetManag.0043CAFF       ; 比对 CALL
00408061    8BBC24 FC000000    mov edi,dword ptr ss:[esp+FC] ; 假序列号出现，ASCII "7878787878787878"
00408068    8B47 F8            mov eax,dword ptr ds:[edi-8] ; 计算序列号长度是否为 16 位，ds:[07150AA0]=00000010
0040806B    85C0               test eax,eax
0040806D    0F8E A4000000      jle NetManag.00408117         ; 小于 16 位则跳死！
00408073    8B4424 18          mov eax,dword ptr ss:[esp+18] ; 取真码长度，ASCII "a68849197afe5b20"，eax=10
00408077    8BF7               mov esi,edi                  ; 从这里检测真假序列号的合法性
00408079    8A10               mov dl,byte ptr ds:[eax]      ; 取每个字符，做逐字比较
0040807B    8A1E               mov bl,byte ptr ds:[esi]
0040807D    8ACA               mov cl,dl
0040807F    3AD3               cmp dl,bl
00408081    75 1E              jnz short NetManag.004080A1   ; 不等则跳向下一种检测方法（作者怕出错！）
00408083    84C9               test cl,cl
00408085    74 16              je short NetManag.0040809D
00408087    8A50 01            mov dl,byte ptr ds:[eax+1]
0040808A    8A5E 01            mov bl,byte ptr ds:[esi+1]
0040808D    8ACA               mov cl,dl
0040808F    3AD3               cmp dl,bl
00408091    75 0E              jnz short NetManag.004080A1
00408093    83C0 02            add eax,2
00408096    83C6 02            add esi,2
00408099    84C9               test cl,cl
0040809B    ^ 75 DC            jnz short NetManag.00408079
0040809D    33C0               xor eax,eax
0040809F    EB 05              jmp short NetManag.004080A6
004080A1    1BC0               sbb eax,eax                  ; 真假序列号相减
004080A3    83D8 FF            sbb eax,-1
004080A6    85C0               test eax,eax
004080A8    75 58              jnz short NetManag.00408102   ; 不等则跳向深渊！
004080AA    8B8424 04010000    mov eax,dword ptr ss:[esp+104]
004080B1    8D4C24 10          lea ecx,dword ptr ss:[esp+10]
004080B5    50                 push eax
004080B6    57                 push edi
004080B7    68 9C474700        push NetManag.0047479C
004080BC    51                 push ecx
004080BD    E8 2CDB0200        call NetManag.00435BEE
004080C2    83C4 10            add esp,10
004080C5    8D9424 FC000000    lea edx,dword ptr ss:[esp+FC]
004080CC    B9 DCC84700        mov ecx,NetManag.0047C8DC
004080D1    52                 push edx
004080D2    E8 154B0300        call NetManag.0043CBEC
004080D7    8B8424 04010000    mov eax,dword ptr ss:[esp+104]
004080DE    6A 00              push 0
004080E0    68 D0444700        push NetManag.004744D0
004080E5    68 88474700        push NetManag.00474788
004080EA    8BCD               mov ecx,ebp
004080EC    A3 E0C84700        mov dword ptr ds:[47C8E0],eax
004080F1    C705 D4C84700 0>   mov dword ptr ds:[47C8D4],1
004080FB    E8 58280300        call NetManag.0043A958
00408100    EB 37              jmp short NetManag.00408139
00408102    6A 00              push 0                       ; 全部清零
00408104    68 D0444700        push NetManag.004744D0
00408109    68 74474700        push NetManag.00474774
0040810E    8BCD               mov ecx,ebp
00408110    E8 43280300        call NetManag.0043A958       ; 到这里就宣布 Game Over 咯~~~
00408115    EB 22              jmp short NetManag.00408139
.........
```

---

## 算法总结

注册验证非常简单：

（监控机器上限数量最大值：`999999999`）

1. 连接“注册码”和“监控机器上限”，组合成一个“新字符串”。
2. 把“新字符串”转换成“`MD5` 值”。
3. 把计算得出的“`MD5` 值”从第 7 位开始取，连续取 16 位，然后再转换为小写输出，作为序列号。

即：`序列号 = LCase（Mid（MD5（注册码+监控机器上限），7，16））`

## VB6 算法注册机源码 (中文版)

```vb
'//////////////////////////////////////////////////////////////////////////////
'/                     /
'/    Program Disassembler & Debugger & Cracked        /
'/    Author: KuNgBiM[DFCG]                            /
'/    E-mail: gb_1227@163.com                          /
'/    OS : WinXP、PEiD、W32Dasm、Ollydbg、Visual Basic 6 /
'/    Date :  2005-07-04                               /
'/                                                    /
'//////////////////////////////////////////////////////////////////////////////
'/  Note : If you have one or more question, email me please,thank you!    /
'//////////////////////////////////////////////////////////////////////////////

'窗体部分
Private Sub Command1_Click()
    Dim RegCode, RegNumber, SerialNumber
    RegCode = Text1.Text
    RegNumber = Text2.Text

    If RegCode = "" And RegNumber = "" Then
        Text3.Text = "【请输入相关的注册信息】"
    Else
        If RegCode = "" Then
            Text3.Text = "【请输入注册码】"
        Else
            If RegNumber = "" Then
                Text3.Text = "【请输入监控机器上限数量】"
            Else
                If RegNumber > 999999999 Then
                    Text3.Text = "【你输入的监控机器上限数量已超过最大上限】"
                Else
                    Set c1 = New clsMD5
                    SerialNumber = LCase(Mid(c1.Md5_String_Calc(RegCode + RegNumber), 7, 16))
                    Text3.Text = SerialNumber
                End If
            End If
        End If
    End If
End Sub

'/////////////////////////////////////////////////////////////////////////
'类模块部分（名称:clsMD5）
Private Const OFFSET_4 = 4294967296#
Private Const MAXINT_4 = 2147483647

Private State(4) As Long
Private ByteCounter As Long
Private ByteBuffer(63) As Byte

Private Const S11 = 7
Private Const S12 = 12
Private Const S13 = 17
Private Const S14 = 22

Private Const S21 = 5
Private Const S22 = 9
Private Const S23 = 14
Private Const S24 = 20

Private Const S31 = 4
Private Const S32 = 11
Private Const S33 = 16
Private Const S34 = 23

Private Const S41 = 6
Private Const S42 = 10
Private Const S43 = 15
Private Const S44 = 21

Property Get RegisterA() As String
    RegisterA = State(1)
End Property

Property Get RegisterB() As String
    RegisterB = State(2)
End Property

Property Get RegisterC() As String
    RegisterC = State(3)
End Property

Property Get RegisterD() As String
    RegisterD = State(4)
End Property

Public Function Md5_String_Calc(SourceString As String) As String
    MD5Init
    MD5Update LenB(StrConv(SourceString, vbFromUnicode)), StringToArray(SourceString)
    MD5Final
    Md5_String_Calc = GetValues
End Function

Public Function Md5_File_Calc(InFile As String) As String
    GoSub begin
begin:
    Dim FileO As Integer
    FileO = FreeFile
    Call FileLen(InFile)
    Open InFile For Binary Access Read As #FileO
    MD5Init
    Do While Not EOF(FileO)
        Get #FileO, , ByteBuffer
        If Loc(FileO) < LOF(FileO) Then
            ByteCounter = ByteCounter + 64
            MD5Transform ByteBuffer
        End If
    Loop
    ByteCounter = ByteCounter + (LOF(FileO) Mod 64)
    Close #FileO
    MD5Final
    Md5_File_Calc = GetValues
End Function

Private Function StringToArray(InString As String) As Byte()
    Dim I As Integer, bytBuffer() As Byte
    ReDim bytBuffer(LenB(StrConv(InString, vbFromUnicode)))
    bytBuffer = StrConv(InString, vbFromUnicode)
    StringToArray = bytBuffer
End Function

Public Function GetValues() As String
    GetValues = LongToString(State(1)) & LongToString(State(2)) & LongToString(State(3)) & LongToString(State(4))
End Function
```
