
```
    xz = Int(k - 1 / 7)

    regname = Val(byteAry(k - 1))

    If regname > 127 Then

        regname = (255 - regname + 1) * (-1)

    End If

    xz = mb(ys) + xz + regname + lngLen

    xz = xz Mod 9

code(lngLen - k) = xz + 48

ys = ys + 1

If ys > 6 Then

    ys = 0

End If

Next k

code(lngLen) = (lngLen + 77) Mod 9 + 48

For n = 0 To lngLen

    strSN = strSN & Chr(code(n))

Next n

Text3.Text = strSN

End Sub

Private Sub Command2_Click(Index As Integer)

    Text2.Text = ""

    Text3.Text = ""

End Sub
```

`keygen` for `ez`:

用`keymake14`

1. 中断地址：`0040ce56`

   中断次数：1

   首字节：`e8`

   命令长度：5

2. 中断地址：`0040d6ad`

   中断次数：1

   首字节：`8a`

   命令长度：2

3. 选择内存方式，在选择`esi`

4. 输入个人信息，确定生成`keygen`! `4dd466d0e1e0cc123afbe504da2462ca`
