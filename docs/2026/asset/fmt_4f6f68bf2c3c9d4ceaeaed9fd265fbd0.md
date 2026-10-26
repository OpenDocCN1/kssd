
# 【破解总结】

## 【算法总结】

### 算法总结

1. 将用户名的各个字符进行替换

替换规则如下

(1) 英文字母大小写进行查表

密码表 "`aqbicHdSeXfMgkhEiVjZkelRmynBoKpdqTrAsFtWulvjwDxIyPzzAxBpCODkEgFYGmHtIaJrKQLnMsNuOUPGQJRLSNTbUcVfWhXoYwZC`"

找相应字符后取紧接着的下一个字符

(2) 非英文字母则统一替换为 `I` 或 `p` 或 `q`

2. 若替换后的长度小于 16 位，则用 "`IsaeBREbmpaqdslg`" 补充到 16 位

## 【算法注册机】

易语言代码

``` 
.版本 2

.子程序 _按钮1_被单击

.局部变量 M, 文本型

.局部变量 i, 整数型

.局部变量 j, 整数型

.局部变量 a, 文本型

.局部变量 b, 整数型

.局部变量 c, 文本型

.局部变量 x, 文本型

.局部变量 y, 文本型

.局部变量 z, 文本型

M ＝ “aqbicHdSeXfMgkhEiVjZkelRmynBoKpdqTrAsFtWulvjwDxIyPzzAxBpCODkEgFYGmHtIaJrKQLnMsNuOUPGQJRLSNTbUcVfWhXoYwZC”

.如果 (取文本长度 (编辑框1.内容) ＝ 0)

    编辑框2.内容 ＝ “请输入用户名”

.否则

    .计次循环首 (取文本长度 (M) ÷ 2, i)

        x ＝ x ＋ 取文本中间 (M, 2 × i － 1, 1)

        y ＝ y ＋ 取文本中间 (M, 2 × i, 1)

    .计次循环尾 ()

    .计次循环首 (取文本长度 (编辑框1.内容), j)

        a ＝ 取文本中间 (编辑框1.内容, j, 1)

        b ＝ 寻找文本 (x, a, 1, 假)

        c ＝ c ＋ 取文本中间 (y, b, 1)

    .计次循环尾 ()

    编辑框2.内容 ＝ 取文本左边 (c ＋ “IsaeBREbmpaqdslg”, 16)

.如果结束
```

☆此文应学友“絕戀de煩神”之邀而作☆

感谢飘云老大、猫老大、Nisy老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

## 【版权声明】

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
