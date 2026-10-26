
```asm
00536C7A    |.  8D45 FC            lea     eax, [local.1]
00536C7D    |.  E8 42D7ECFF        call    004043C4
00536C82    |.  33FF               xor     edi, edi
00536C84    |>  8D45 D4            /lea     eax, [local.11]                           ;  偶数位字符串取反操作  CF8A2A3E变成 E3A2A8FC
00536C87    |.  BA 08000000        |mov     edx, 8
00536C8C    |.  2BD7               |sub     edx, edi
00536C8E    |.  8B4D F4            |mov     ecx, [local.3]
00536C91    |.  8A5411 FF          |mov     dl, byte ptr [ecx+edx-1]
00536C95    |.  E8 12D9ECFF        |call    004045AC
00536C9A    |.  8B55 D4            |mov     edx, [local.11]
00536C9D    |.  8D45 FC            |lea     eax, [local.1]
00536CA0    |.  E8 F7D9ECFF        |call    0040469C
00536CA5    |.  47                 |inc     edi
00536CA6    |.  83FF 08            |cmp     edi, 8
00536CA9    |.^ 75 D9              \jnz     short 00536C84
00536CAB    |.  8D55 D0            lea     edx, [local.12]
00536CAE    |.  8B45 F0            mov     eax, [local.4]                             ;  eax=AEAEE1DF 取奇数位8位
00536CB1    |.  E8 BE96FFFF        call    00530374                                   ;  奇数位进入MD5运算
00536CB6    |.  8B55 D0            mov     edx, [local.12]                            ;  edx=D7419D0CCAFCC752DBCBAEABE3A2A8FC MD5运算结果
00536CB9    |.  8D45 F0            lea     eax, [local.4]
00536CBC    |.  E8 9BD7ECFF        call    0040445C
00536CC1    |.  8D4D CC            lea     ecx, [local.13]
00536CC4    |.  BA 08000000        mov     edx, 8                                     ;  取MD5运算结果后8位  E3A2A8FC
00536CC9    |.  8B45 F0            mov     eax, [local.4]
00536CCC    |.  E8 3F16F0FF        call    00438310
00536CD1    |.  8B55 CC            mov     edx, [local.13]
00536CD4    |.  8D45 F0            lea     eax, [local.4]
00536CD7    |.  E8 80D7ECFF        call    0040445C
00536CDC    |.  8B45 FC            mov     eax, [local.1]                             ;  后八位与偶数位取反进行比较，相等就注册成功
00536CDF    |.  8B55 F0            mov     edx, [local.4]
00536CE2    |.  E8 F9DAECFF        call    004047E0
00536CE7    |.  75 05              jnz     short 00536CEE
00536CE9    |.  83CE FF            or      esi, FFFFFFFF
00536CEC    |.  EB 02              jmp     short 00536CF0
00536CEE    |>  33F6               xor     esi, esi
00536CF0    |>  33C0               xor     eax, eax
00536CF2    |.  5A                 pop     edx
00536CF3    |.  59                 pop     ecx
00536CF4    |.  59                 pop     ecx
00536CF5    |.  64:8910            mov     dword ptr fs:[eax], edx
00536CF8    |.  68 126D5300        push    00536D12
00536CFD    |>  8D45 CC            lea     eax, [local.13]
00536D00    |.  BA 0D000000        mov     edx, 0D
00536D05    |.  E8 DE D6ECFF       call    004043E8
00536D0A    \.  C3                 retn
```

## 经验总结

注册码长度位22位，前三位为固定的`ERA`，后面19位里面有“`-`”三个

然后取去掉“`-`”的16位长度的字符串，按偶数位和奇数位取成两个八位字符串

注册算法:偶数位字符串取反=`MD5`(奇数位）的后面八位

虽然这个软件用到加密算法 `MD5`，并且还不是完全明码比较，不过注册流程太清楚，相对任意跟踪出注册码

一组`key`：`ERA---ACEFA8EAE21AD3FE`

最后的注册码保存在`strings.ini`文件里面

```
[Common]
Name=zeromk2@crsky.com
Code=ERA---ACEFA8EAE21AD3FE
```

花了几分钟补了个注册机代码

## VB注册机源代码

`MD5`代码来自网络，被放在唯一的模块里面

```vb
Option Explicit

Private Sub Command1_Click()

Dim ser(15) As String
Dim i As Integer, j As Integer, key1 As String, key2 As String, key3 As String

For i = 0 To 15
      Randomize
      ser(i) = Chr(Int(Rnd * (90 - 65 + 1) + 65))
Next i

For i = 0 To 7
     key1 = key1 & ser(i * 2)
Next i

'key1 = "AEAEE1DF"
key1 = Md5_String_Calc(key1)
key2 = UCase(Mid(key1, (Len(key1) - 7), 8))
ser(1) = Mid(key2, 8, 1)
ser(3) = Mid(key2, 7, 1)
ser(5) = Mid(key2, 6, 1)
ser(7) = Mid(key2, 5, 1)
ser(9) = Mid(key2, 4, 1)
ser(11) = Mid(key2, 3, 1)
ser(13) = Mid(key2, 2, 1)
ser(15) = Mid(key2, 1, 1)

For i = 0 To 15
      key3 = key3 & ser(i)
Next i

Text1.Text = "ERA---" & key3
Clipboard.Clear
Clipboard.SetText Text1.Text

End Sub

Private Sub Command2_Click()
End
End Sub
```

## MD5 Module sourcecode

```vb
Option Explicit

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
On Error GoTo errorhandler
GoSub begin
errorhandler:
Md5_File_Calc = ""
Exit Function
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
Dim i As Integer, bytBuffer() As Byte
ReDim bytBuffer(LenB(StrConv(InString, vbFromUnicode)))
bytBuffer = StrConv(InString, vbFromUnicode)
StringToArray = bytBuffer
End Function

Public Function GetValues() As String
GetValues = LongToString(State(1)) & LongToString(State(2)) & LongToString(State(3)) & LongToString(State(4))
End Function

Private Function LongToString(Num As Long) As String
Dim A As Byte, B As Byte, C As Byte, d As Byte
A = Num And &HFF&
If A < 16 Then LongToString = "0" & Hex(A) Else LongToString = Hex(A)
B = (Num And &HFF00&) \ 256
If B < 16 Then LongToString = LongToString & "0" & Hex(B) Else LongToString = LongToString & Hex(B)
C = (Num And &HFF0000) \ 65536
If C < 16 Then LongToString = LongToString & "0" & Hex(C) Else LongToString = LongToString & Hex(C)
If Num < 0 Then d = ((Num And &H7F000000) \ 16777216) Or &H80& Else d = (Num And &HFF000000) \ 16777216
If d < 16 Then LongToString = LongToString & "0" & Hex(d) Else LongToString = LongToString & Hex(d)
End Function

Public Sub MD5Init()
ByteCounter = 0
State(1) = UnsignedToLong(1732584193#)
State(2) = UnsignedToLong(4023233417#)
State(3) = UnsignedToLong(2562383102#)
State(4) = UnsignedToLong(271733878#)
End Sub

Public Sub MD5Final()
Dim dblBits As Double, padding(72) As Byte, lngBytesBuffered As Long
padding(0) = &H80
dblBits = ByteCounter * 8
lngBytesBuffered = ByteCounter Mod 64
If lngBytesBuffered <= 56 Then MD5Update 56 - lngBytesBuffered, padding Else MD5Update 120 - ByteCounter, padding
padding(0) = UnsignedToLong(dblBits) And &HFF&
padding(1) = UnsignedToLong(dblBits) \ 256 And &HFF&
padding(2) = UnsignedToLong(dblBits) \ 65536 And &HFF&
padding(3) = UnsignedToLong(dblBits) \ 16777216 And &HFF&
padding(4) = 0
padding(5) = 0
padding(6) = 0
padding(7) = 0
MD5Update 8, padding
End Sub

Public Sub MD5Update(InputLen As Long, InputBuffer() As Byte)
```
