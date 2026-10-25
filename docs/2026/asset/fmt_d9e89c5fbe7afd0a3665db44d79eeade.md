
```
//除以 0x3。
TEST     EDX,EDX

JNZ      0040DEBE

//余数不为零就 Game Over
！

MOV      EAX,EBX
//将 EBX 的值 送入 EAX。
MOV      ESI,05
CDQ
IDIV     ESI
//除以 0x5。
TEST     EDX,EDX
JNZ      0040DEBE
//余数不为零就 Game Over ！
SHL      EBP,10
//EBP<<=0x10。
MOV      ESI,03
LEA      EAX,[ECX+EBP]
//EAX=ECX+EBP。
DIV      ESI
//除以 0x3。
TEST     EDX,EDX
JNZ      0040DEBE
//余数不为零就 Game Over ！
TEST     BL,01
//BL 和 0x1 做与运算。
JNZ      0040DEBE
//结果不为零就 Game Over ！
TEST     CL,03
//CL 和 0x3 做与运算。
JNZ      0040DEBE
//结果不为零就 Game Over ！
LEA      EAX,[EBX+EBP]
//EAX=EBX+EBP。
MOV      ECX,06
DIV      ECX
//除以 0x6。
POP      ESI
POP      EBP
POP      EBX
MOV      EAX,EDX
NEG      EAX
//余数不为零就 Game Over ！
SBB      EAX,EAX
INC      EAX
ADD      ESP,A4
RET
POP      ESI
POP      EBP
XOR      EAX,EAX
POP      EBX
ADD      ESP,A4
RET
```

### 总结一下：

合法的序列号应满足如下条件：

（1）：序列号和用户名无关；

（2）：序列号至少为 24 位，且前两位必须为“dx”，记为：`dxPQ*SN1*SN2*SN3*SN4`；

（3）：（`SN1 - PQ`） MOD 6 = 0；

（4）：（`SN2 - PQ`） AND `&H80000003&` = 0；

（5）：（`SN3 - PQ`）MOD 3 = 0；

（6）：（（`SN1 * 2^16`）+（`SN3 - PQ`）） MOD 3 = 0；

（7）：（`SN3` MOD 256）AND 3 = 0；

（8）：（`SN4 - PQ`） MOD 5= 0；

（9）：（（`SN1 * 2^16`）+（`SN4 - PQ`））MOD 6= 0；

（A）：（`SN4` MOD 256） AND 1 = 0；

写完破解纪录之后用 `Fengma's Toy beta1` 删掉前面的地址，这样初学者朋友看破文就不会偷懒了~~：）

### 注册机代码

```vb
Dim EAX1 As Integer
Dim EAX2 As Integer
Dim EAX3 As Integer
Dim EAX4 As Integer
Dim EBX2 As Integer
Dim EBX3 As Integer
Dim EBX4 As Integer
Dim ECX3 As Integer
Dim EDX1 As Integer
Dim EDX2 As Integer
Dim EDX4 As Integer
Dim EBP1 As Integer
Dim Temp_A As Integer
Dim Temp_B As Integer
Dim Condition_A As Integer
Dim Condition_B As Integer
Dim Condition_C As Integer
Dim Condition_D As Integer
Dim Condition_E As Integer
Dim Condition_F As Integer
Dim Condition_G As Integer
Dim SN_A As String
Dim SN_B As String
Dim SN_C As String
Dim SN_D As String
Dim SHL As Double
Dim EAX31 As Double
Dim EAX41 As Double
Private Sub Command1_Click()
  About_message = MsgBox("This keygen made by fengma(fengma@2911.net) !", 4096 + vbExclamation, "About...")
End Sub
Private Sub Command2_Click()
If Len(Text1.Text) < 2 Then
  message_enter_name = MsgBox("Use a name at least 2 Chars long , please !", 4096 + vbExclamation, "Information")
  Exit Sub
End If
If Command2.Caption = "Get Now !" Then
Randomize
Dim Data()
Dim SN1_(4), SN2_(4), SN3_(4), SN4_(4) As String
Data = Array("1","2","3","4","5","6","7","8","9","0","q","w","e","r","t","y","u","i","o","p","a","s","d","f","g","h","j","k","l","z","x","c","v","b","n","m")
Do
SN_B = ""
EBX2 = 0
For i = 1 To 4
  SN2_(i) = Data(Int(Rnd * 36))
  EAX2 = Asc(SN2_(i)) - 48
    If EAX2 > 9 Then
     EAX2 = EAX2 - 39
    End If
  EDX2 = EBX2 + EBX2 * 8
  EBX2 = EAX2 + EDX2 * 2
  SN_B = SN_B & SN2_(i)
Next i
Condition_B = (EBX2 - 10) And &H80000003
Loop While Condition_B <> 0
Do
SN_A = ""
EBP1 = 0
For i = 1 To 4
  SN1_(i) = Data(Int(Rnd * 36))
  EAX1 = Asc(SN1_(i)) - 48
    If EAX1 > 9 Then
      EAX1 = EAX1 - 39
    End If
  EDX1 = EBP1 + EBP1 * 8
  EBP1 = EAX1 + EDX1 * 2
  SN_A = SN_A & SN1_(i)
Next i
Condition_A = (EBP1 - 10) Mod 6
SN_C = ""
EBX3 = 0
For i = 1 To 4
  SN3_(i) = Data(Int(Rnd * 36))
  EAX3 = Asc(SN3_(i)) - 48
    If EAX3 > 9 Then
     EAX3 = EAX3 - 39
    End If
  ECX3 = EBX3 + EBX3 * 8
  EBX3 = EAX3 + ECX3 * 2
  SN_C = SN_C & SN3_(i)
Next i
Condition_C = (EBX3 - 10) Mod 3
SHL = (EBP1 - 10) * 2 ^ 16
EAX_31 = SHL + (EBX3 - 10)
Do
If EAX_31 > 4294967296# Then
  EAX_31 = EAX_31 - 4294967296#
End If
Loop While EAX_31 > 4294967296#
Temp_A = 0
For i = 1 To Len(EAX_31)
  Temp_A = Temp_A + Asc(Mid$(EAX_31, i, 1)) - 48
Next i
Condition_D = Temp_A Mod 3
Condition_E = ((EBX3 - 10) Mod 256) And 3
SN_D = ""
EBX4 = 0
For i = 1 To 4
  SN4_(i) = Data(Int(Rnd * 36))
  EAX4 = Asc(SN4_(i)) - 48
    If EAX4 > 9 Then
     EAX4 = EAX4 - 39
    End If
  EDX4 = EBX4 + EBX4 * 8
  EBX4 = EAX4 + EDX4 * 2
  SN_D = SN_D & SN4_(i)
Next i
Condition_F = (EBX4 - 10) Mod 5
EAX_41 = SHL + (EBX4 - 10)
Do
If EAX_41 >= 4294967296# Then
  EAX_41 = EAX_41 - 4294967296#
End If
Loop While EAX_41 >= 4294967296#
Temp_B = 0
For i = 1 To Len(EAX_41)
  Temp_B = Temp_B + Asc(Mid$(EAX_41, i, 1)) - 48
Next i
Condition_G = Temp_B Mod 2
Condition_H = Temp_B Mod 3
Condition_I = ((EBX4 - 10) Mod 256) And 1
Loop While Condition_A + Condition_C + Condition_D + Condition_E + Condition_F + Condition_G + Condition_H + Condition_I <> 0
Text2.Text = "DX10-" & UCase(SN_A) & "-" & UCase(SN_B) & "-" & UCase(SN_C) & "-" & UCase(SN_D)
With Command2
.Font = Tahoma
.Caption = "Bye !"
End With
Else
End
End If
End Sub
```

fengma[FCG]

Cracked
