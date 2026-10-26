
# 破解总结

VB程序，用到MD5算法

## 算法总结

以`tianxj`为例

1. 将`"tianxj"`转成大写即`"TIANXJ"`后与`"Chinese Character Stroke Order Animator v2.10"`相连，形成`"TIANXJChinese Character Stroke Order Animator v2.10"`
2. 取`"TIANXJChinese Character Stroke Order Animator v2.10"`的MD5值`"1d3466d7584130d896ec9fc6d48b598f"`
3. 循环取`"1d3466d7584130d896ec9fc6d48b598f"`里的字符，每次取2个作运算。比如：第1次取`"1d"`与`"1f"`作与运算后，按结果`"1d"`的顺序取字符串`"0123456789ABCDEFGHJKLMNPQRTUVWXY"`里的字符`"W"`，第2次取`"34"`与`"1f"`作与运算后按结果`"1d"`的顺序取字符串`"0123456789ABCDEFGHJKLMNPQRTUVWXY"`里的字符`"L"`，依此类推，相连得到注册码`"WL6PQ1GQNCY6LBRF"`

## 算法注册机

### VB代码

```
Private Sub Command1_Click()
If Len(Text1.Text) = 0 Then
   Text2.Text = "输入有误，请重新输入！"
Else
    SN = MD5(UCase(Text1.Text) & "Chinese Character Stroke Order Animator v2.10")
  For I = 1 To 16
    X = ("&H" & Mid(SN, 2 * I - 1, 2)) And 31
    Y = Y & Mid("0123456789ABCDEFGHJKLMNPQRTUVWXY", X + 1, 1)
  Next I
     Text2.Text = Y
End If
End Sub
```

感谢飘云老大、猫老大、Nisy老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

## 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
