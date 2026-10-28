
# 大数计算器的VB代码，其实肯去发掘，VB能干很多东西。 (15千字)

```vb
Attribute VB_Name = "BigNumbers"

Option Explicit

Public Const MaxLenth = 20000

Dim Pom0(MaxLenth) As Long
Dim Pom1(MaxLenth) As Long
Dim Pom2(MaxLenth) As Long
Dim Pom3(MaxLenth) As Long
Dim Pom4(MaxLenth) As Long
Dim Pom5(MaxLenth) As Long
Dim Pom6(MaxLenth) As Long
Dim Pom7(MaxLenth) As Long
Dim LengthPom0 As Long
Dim LengthPom1 As Long
Dim LengthPom2 As Long
Dim LengthPom3 As Long
Dim LengthPom4 As Long
Dim LengthPom5 As Long
Dim LengthPom6 As Long
Dim LengthPom7 As Long
Dim K1(MaxLenth) As Long
Dim K10(MaxLenth) As Long
Dim K100(MaxLenth) As Long
Dim K200(MaxLenth) As Long
Dim K10000(MaxLenth) As Long
Dim A(MaxLenth) As Long
Dim B(MaxLenth) As Long
Dim C(MaxLenth) As Long
Dim D(MaxLenth) As Long
Dim LengthA As Long
Dim LengthB As Long
Dim LengthC As Long
Dim LengthD As Long

Function CompareB(A() As Long, LengthA As Long, B() As Long, LengthB As Long) As Long
    Dim I As Long
    Select Case LengthA - LengthB
        Case Is < 0
            CompareB = -1
        Case Is = 0
            CompareB = 0
            For I = 1 To LengthA
                Select Case A(LengthA - I + 1) - B(LengthA - I + 1)
                    Case Is < 0
                        CompareB = -1
                        Exit For
                    Case Is = 0
                    Case Is > 0
                        CompareB = 1
                        Exit For
                End Select
            Next I
        Case Is > 0
            CompareB = 1
    End Select
End Function

Function BigNumberToText(C() As Long, Length As Long) As String
    Dim I As Long
    Dim Pom As String
    If C(0) = -1 Then
        Pom = "-"
    Else
        Pom = ""
    End If
    Pom = Pom & Format$(C(Length), "0")
    For I = Length - 1 To 1 Step -1
        Pom = Pom & Format$(C(I), "0000")
    Next I
    BigNumberToText = Pom
End Function

Sub TextToBigNumber(Tekst As String, A() As Long, LengthA As Long)
    Dim I As Long
    Dim Prvi As String
    Dim MaxLenthPrvi As Long
    Dim Ostatak As Long
    Prvi = Trim$(Tekst)
    If IsItBigNumber(Prvi) Then
        A(0) = 0
        If Left$(Prvi, 1) = "+" Then
            Prvi = Right$(Prvi, Len(Prvi) - 1)
        End If
        If Left$(Prvi, 1) = "-" Then
            Prvi = Right$(Prvi, Len(Prvi) - 1)
            A(0) = -1
        End If
        MaxLenthPrvi = Len(Prvi)
        If (MaxLenthPrvi \ 4) * 4 = MaxLenthPrvi Then
            LengthA = MaxLenthPrvi \ 4
            For I = 1 To LengthA
                A(I) = Mid$(Prvi, MaxLenthPrvi - I * 4 + 1, 4)
            Next I
        Else
            LengthA = MaxLenthPrvi \ 4 + 1
            Ostatak = MaxLenthPrvi Mod 4
            For I = 1 To LengthA - 1
                A(I) = Mid$(Prvi, MaxLenthPrvi - I * 4 + 1, 4)
            Next I
            A(LengthA) = Mid$(Prvi, 1, Ostatak)
        End If
    Else
        A(1) = 0
        LengthA = 1
    End If
End Sub

Sub MinusB(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long)
    Select Case CompareB(A, LengthA, B, LengthB)
        Case Is < 0
            Call MinusBV(B, LengthB, A, LengthA, C, LengthC)
            C(0) = -1
        Case Is = 0
            C(1) = 0
            LengthC = 1
            C(0) = 0
        Case Is > 0
            Call MinusBV(A, LengthA, B, LengthB, C, LengthC)
            C(0) = 0
    End Select
End Sub

Function IsItBigNumber(Ulaz As String) As Boolean
    Dim Pom As String
    Dim Pom1 As String
    Dim I As Long
    Dim IsItBigNumber1 As Boolean
    Pom1 = Ulaz
    Pom = Left$(Pom1, 300)
    If IsNumeric(Pom) Then
        If InStr(1, Pom, "e", 1) > 0 Then
            IsItBigNumber1 = False
        Else
            If InStr(1, Pom, ".", 1) > 0 Then
                IsItBigNumber1 = False
            Else
                If InStr(1, Pom, ",", 1) > 0 Then
                    IsItBigNumber1 = False
                Else
                    IsItBigNumber1 = True
                End If
            End If
        End If
    Else
        IsItBigNumber1 = False
    End If
    If IsItBigNumber1 Then
        For I = 1 To Len(Pom1) \ 300
            Pom = Mid$(Pom1, 300 * I, 300)
            If IsNumeric(Pom) Then
                If InStr(1, Pom, "e", 1) > 0 Then
                    IsItBigNumber1 = False
                Else
                    If InStr(1, Pom, ".", 1) > 0 Then
                        IsItBigNumber1 = False
                    Else
                        If InStr(1, Pom, ",", 1) > 0 Then
                            IsItBigNumber1 = False
                        Else
                            If InStr(1, Pom, "-", 1) > 0 Then
                                IsItBigNumber1 = False
                            Else
                                If InStr(1, Pom, "+", 1) > 0 Then
                                    IsItBigNumber1 = False
                                Else
                                    IsItBigNumber1 = True
                                End If
                            End If
                        End If
                    End If
                End If
            Else
                IsItBigNumber1 = False
            End If
        Next I
    End If
    IsItBigNumber = IsItBigNumber1
End Function

Sub AddBSigned(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long)
    If A(0) = 0 And B(0) = 0 Then
        Call AddB(A, LengthA, B, LengthB, C, LengthC)
    End If
    If A(0) < 0 And B(0) < 0 Then
        Call AddB(A, LengthA, B, LengthB, C, LengthC)
        C(0) = -1
    End If
    If A(0) = 0 And B(0) < 0 Then
        Call MinusB(A, LengthA, B, LengthB, C, LengthC)
    End If
    If A(0) < 0 And B(0) = 0 Then
        Call MinusB(B, LengthB, A, LengthA, C, LengthC)
    End If
End Sub

Sub MinusBSigned(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long)
    If A(0) = 0 And B(0) = 0 Then
        Call MinusB(A, LengthA, B, LengthB, C, LengthC)
    End If
    If A(0) < 0 And B(0) < 0 Then
        Call MinusB(B, LengthB, A, LengthA, C, LengthC)
    End If
    If A(0) = 0 And B(0) < 0 Then
        Call AddB(A, LengthA, B, LengthB, C, LengthC)
        C(0) = 0
    End If
    If A(0) < 0 And B(0) = 0 Then
        Call AddB(B, LengthB, A, LengthA, C, LengthC)
        C(0) = -1
    End If
End Sub

Sub MultBSigned(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long)
    Call MultB(A, LengthA, B, LengthB, C, LengthC)
    If (A(0) = 0 And B(0) = 0) Or (A(0) < 0 And B(0) < 0) Then
        C(0) = 0
    Else
        If LengthC = 1 And C(1) = 0 Then
            C(0) = 0
        Else
            C(0) = -1
        End If
    End If
End Sub

Sub CopyB(A() As Long, LengthA As Long, B() As Long, LengthB As Long)
    Dim I As Long
    LengthB = LengthA
    For I = 0 To LengthA
        B(I) = A(I)
    Next I
End Sub

Sub DivBSigned(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long, D() As Long, LengthD As Long)
    Call DivB(A, LengthA, B, LengthB, C, LengthC, D, LengthD)
    If (A(0) = 0 And B(0) = 0) Or (A(0) < 0 And B(0) < 0) Then
        C(0) = 0
    Else
        If LengthC = 1 And C(1) = 0 Then
            C(0) = 0
        Else
            C(0) = -1
        End If
    End If
End Sub

Sub DivB(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long, D() As Long, LengthD As Long)
    If LengthB = 1 And B(1) = 0 Then
        C(1) = 0
        LengthC = 1
        C(0) = 0
        Exit Sub
    End If
    If LengthB = 1 And B(1) = 1 Then
        Call CopyB(A, LengthA, C, LengthC)
        Exit Sub
    End If
    If LengthA = 1 And A(1) = 0 Then
        C(1) = 0
        LengthC = 1
        C(0) = 0
        Exit Sub
    End If
    Select Case CompareB(A, LengthA, B, LengthB)
        Case Is < 0
            C(1) = 0
            LengthC = 1
            C(0) = 0
        Case Is = 0
            C(1) = 1
            LengthC = 1
            C(0) = 0
        Case Is > 0
            Call DivBInt(A, LengthA, B, LengthB, C, LengthC, D, LengthD)
    End Select
End Sub

Sub DivBInt(A() As Long, LengthA As Long, B() As Long, LengthB As Long, C() As Long, LengthC As Long, D() As Long, LengthD As Long)
    Dim I As Long
    Dim J As Long
    Dim StrA As String
    Dim StrB As String
    Dim StrC As String
    Dim MaxLenthStrA As Long
    Dim MaxLenthStrB As Long
    Dim TR As String
    K10(1) = 10
    StrA = BigNumberToText(A, LengthA)
    If Left$(StrA, 1) = "-" Then StrA = Right$(StrA, Len(StrA) - 1)
    StrB = BigNumberToText(B, LengthB)
    If Left$(StrA, 1) = "-" Then StrA = Right$(StrA, Len(StrA) - 1)
    MaxLenthStrA = Len(StrA)
    MaxLenthStrB = Len(StrB)
    J = 0
    Call TextToBigNumber(Left$(StrA, MaxLenthStrB), Pom2, LengthPom2)
    Do While CompareB(Pom2, LengthPom2, B, LengthB) >= 0
        J = J + 1
        Call MinusBV(Pom2, LengthPom2, B, LengthB, Pom3, LengthPom3)
        Call CopyB(Pom3, LengthPom3, Pom2, LengthPom2)
    Loop
End Sub
```
