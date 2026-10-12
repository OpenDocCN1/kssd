
【破解总结】

【算法总结】

1. 注册码必须是 20 位
2. 取用户名长度减去循环次数后与用户名 `ASCII` 码相乘，累加乘积，格式化为 `%04X` 形式。
取试炼码第 8 位、第 20 位、试炼码第 15 位、试炼码第 19 位与格式化的结果逐位比较。

【算法注册机】

〖VB 代码〗

```vb
Private Sub Command1_Click()
    If Len(Text1.Text) = 0 Then
        Text2.Text = "输入有误，请重新输入！"
    Else
        For i = 1 To Len(Text1.Text)
            j = Len(Text1.Text) - i + 1
            x = Asc(Mid(Text1.Text, i, 1)) * j
            y = y + x
        Next
        z = Right("0000" & Hex(y), 4)
        Text2.Text = Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Mid(z, 1, 1) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Mid(z, 3, 1) & Int(Rnd() * 10) & Int(Rnd() * 10) & Int(Rnd() * 10) & Mid(z, 4, 1) & Mid(z, 2, 1) 'Int(Rnd()  '"1234567" & Mid(z, 1, 1) & "901234" & Mid(z, 3, 1) & "678" & Mid(z, 4, 1) & Mid(z, 2, 1)
    End If
End Sub
```

感谢飘云老大、猫老大、`Nisy` 老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

【版权声明】破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
