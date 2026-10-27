
```vb
        MD5F3 b, c, d, a, Xin(14), "fde5380c", 23
        MD5F3 a, b, c, d, Xin(1), "a4beea44", 4
        MD5F3 d, a, b, c, Xin(4), "4bdecfa9", 11
        MD5F3 c, d, a, b, Xin(7), "f6bb4b60", 16
        MD5F3 b, c, d, a, Xin(10), "bebfbc70", 23
        MD5F3 a, b, c, d, Xin(13), "289b7ec6", 4
        MD5F3 d, a, b, c, Xin(0), "e27fa", 11
        MD5F3 c, d, a, b, Xin(3), "d4ef3085", 16
        MD5F3 b, c, d, a, Xin(6), "04881d05", 23
        MD5F3 a, b, c, d, Xin(9), "d9d4d039", 4
        MD5F3 d, a, b, c, Xin(12), "e6db99e5", 11
        MD5F3 c, d, a, b, Xin(15), "1fa27cf8", 16
        MD5F3 b, c, d, a, Xin(2), "c4ac5665", 23

        ' 第四轮循环
        MD5F4 a, b, c, d, Xin(0), "f4292244", 6
        MD5F4 d, a, b, c, Xin(7), "432aff97", 10
        MD5F4 c, d, a, b, Xin(14), "ab9423a7", 15
        MD5F4 b, c, d, a, Xin(5), "fc93a039", 21
        MD5F4 a, b, c, d, Xin(12), "655b59c3", 6
        MD5F4 d, a, b, c, Xin(3), "8f0ccc92", 10
        MD5F4 c, d, a, b, Xin(10), "ffeff47d", 15
        MD5F4 b, c, d, a, Xin(1), "85845dd1", 21
        MD5F4 a, b, c, d, Xin(8), "6fa87e4f", 6
        MD5F4 d, a, b, c, Xin(15), "fe2ce6e0", 10
        MD5F4 c, d, a, b, Xin(6), "a3014314", 15
        MD5F4 b, c, d, a, Xin(13), "4e0811a1", 21
        MD5F4 a, b, c, d, Xin(4), "f7537e82", 6
        MD5F4 d, a, b, c, Xin(11), "bd3af235", 10
        MD5F4 c, d, a, b, Xin(2), "2ad7d2bb", 15
        MD5F4 b, c, d, a, Xin(9), "eb86d391", 21

        buf(0) = BigAdd(buf(0), a)
        buf(1) = BigAdd(buf(1), b)
        buf(2) = BigAdd(buf(2), c)
        buf(3) = BigAdd(buf(3), d)
    Next loopouter

    hashthis = ""
    For loopit = 0 To 3
        For loopinner = 3 To 0 Step -1
            hashthis = hashthis + Chr(Val("&H" + Mid$(buf(loopit), 1 + 2 * loopinner, 2)))
        Next loopinner
    Next loopit

MD5_Calc = hashthis
End Function

Function BigMod32Add(ByVal value1 As String, ByVal value2 As String) As String

    BigMod32Add = Right$(BigAdd(value1, value2), 8)
End Function

Public Function BigAdd(ByVal value1 As String, ByVal value2 As String) As String

Dim valueans As String
Dim loopit As Integer, tempnum As Integer

    tempnum = Len(value1) - Len(value2)
    If tempnum < 0 Then
        value1 = Space$(Abs(tempnum)) + value1
    ElseIf tempnum > 0 Then
        value2 = Space$(Abs(tempnum)) + value2
    End If

    tempnum = 0
    For loopit = Len(value1) To 1 Step -1
        tempnum = tempnum + Val("&H" + Mid$(value1, loopit, 1)) + Val("&H" + Mid$(value2, loopit, 1))
        valueans = Hex$(tempnum Mod 16) + valueans
        tempnum = Int(tempnum / 16)
    Next loopit

    If tempnum <> 0 Then
        valueans = Hex$(tempnum) + valueans
    End If

    BigAdd = Right(valueans, 8)
End Function

Public Function RotLeft(ByVal value1 As String, ByVal rots As Integer) As String

Dim tempstr As String
Dim loopit As Integer, loopinner As Integer
Dim tempnum As Integer

    rots = rots Mod 32

    If rots = 0 Then
        RotLeft = value1
        Exit Function
    End If

    value1 = Right$(value1, 8)
    tempstr = String$(8 - Len(value1), "0") + value1
    value1 = ""

    ' 转换成二进制形式
    For loopit = 1 To 8
        tempnum = Val("&H" + Mid$(tempstr, loopit, 1))

        For loopinner = 3 To 0 Step -1
            If tempnum And 2 ^ loopinner Then

                value1 = value1 + "1"

            Else
                value1 = value1 + "0"

            End If
        Next loopinner
    Next loopit
    tempstr = Mid$(value1, rots + 1) + Left$(value1, rots)

    ' 转换为十六进制
    value1 = ""
    For loopit = 0 To 7
        tempnum = 0
        For loopinner = 0 To 3
            If Val(Mid$(tempstr, 4 * loopit + loopinner + 1, 1)) Then

                tempnum = tempnum + 2 ^ (3 - loopinner)
            End If
        Next loopinner
        value1 = value1 + Hex$(tempnum)
    Next loopit

    RotLeft = Right(value1, 8)
End Function

Function BigAND(ByVal value1 As String, ByVal value2 As String) As String
Dim valueans As String
Dim loopit As Integer, tempnum As Integer

    tempnum = Len(value1) - Len(value2)
    If tempnum < 0 Then
        value2 = Mid$(value2, Abs(tempnum) + 1)
    ElseIf tempnum > 0 Then
        value1 = Mid$(value1, tempnum + 1)
    End If

    For loopit = 1 To Len(value1)
        valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) And Val("&H" + Mid$(value2, loopit, 1)))
    Next loopit

    BigAND = valueans
End Function

Function BigNOT(ByVal value1 As String) As String
Dim valueans As String
Dim loopit As Integer

    value1 = Right$(value1, 8)
    value1 = String$(8 - Len(value1), "0") + value1
    For loopit = 1 To 8
        valueans = valueans + Hex$(15 Xor Val("&H" + Mid$(value1, loopit, 1)))
    Next loopit

    BigNOT = valueans
End Function

Function BigOR(ByVal value1 As String, ByVal value2 As String) As String
Dim valueans As String
Dim loopit As Integer, tempnum As Integer

    tempnum = Len(value1) - Len(value2)
    If tempnum < 0 Then
        valueans = Left$(value2, Abs(tempnum))
        value2 = Mid$(value2, Abs(tempnum) + 1)
    ElseIf tempnum > 0 Then
        valueans = Left$(value1, Abs(tempnum))
        value1 = Mid$(value1, tempnum + 1)
    End If

    For loopit = 1 To Len(value1)
        valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) Or Val("&H" + Mid$(value2, loopit, 1)))
    Next loopit

    BigOR = valueans
End Function

Function BigXOR(ByVal value1 As String, ByVal value2 As String) As String
Dim valueans As String
Dim loopit As Integer, tempnum As Integer

    tempnum = Len(value1) - Len(value2)
    If tempnum < 0 Then
        valueans = Left$(value2, Abs(tempnum))
        value2 = Mid$(value2, Abs(tempnum) + 1)
    ElseIf tempnum > 0 Then
        valueans = Left$(value1, Abs(tempnum))
        value1 = Mid$(value1, tempnum + 1)
    End If

    For loopit = 1 To Len(value1)
        valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) Xor Val("&H" + Mid$(value2, loopit, 1)))
    Next loopit

    BigXOR = Right(valueans, 8)
End Function
```

――――――――――――――――――――――――――――――――――――――――――――

呵呵 洋洋洒洒的一大篇可是我查找了不少资料才写出来 ^\_^ 大概可以当成我们学校的Assignment了 呵呵 感谢大家看完

最后给大家出到题吧 大家可以试试自己的运气啊 就是：

51E5D4BD3323A02CCCDD0472AE2DC20B  这组数是我通过MD5算法加密一组字符串后产生的一组密码 大家猜猜我加密的字符串是什么

^\_^ 提示一下――――加空格一共20位  ^\_^

^\_^  88



仅以次文献给我们可爱的组织CCG 希望它能蒸蒸日上！

娃娃(NYDoll)  
属于中国破解组织CHiNA CrACKiNG GrOUp
