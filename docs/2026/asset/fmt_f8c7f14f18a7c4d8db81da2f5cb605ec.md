
# 【破解总结】

--------------------------------------------------------------

## 【算法总结】

注册码=`用户名ASCII码累加值*22C3h+4DCE424h`

--------------------------------------------------------------

## 【算法注册机】

算法比较简单，用`keymake`作算法注册机比较快

`KeyGen.rek`

```
.const

.data

szHomePage db "a68K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9h3&6S2M7s2W2Y4i4K6u0W2j5$3!0E0",0

szEmail    db "mailto:tianxj_2007@126.com",0

szErrMess  db "请输入用户名！",0

szFMT    db "%d",0

szBuffer db 50 dup (0)

.code

        MOV ESI,EAX

invoke lstrlen,esi

        MOV EDX,EAX

        XOR ECX,ECX

        XOR EAX,EAX

@MP4Video_0040F16A:

        MOVSX EBX,BYTE PTR DS:[EAX+ESI]

        ADD ECX,EBX

        INC EAX

        CMP EAX,EDX

        JL @MP4Video_0040F16A

        IMUL ECX,ECX,022C3h

        ADD ECX,04DCE424h

        MOV ESI,ECX

invoke wsprintf,addr szBuffer,addr szFMT,esi

lea eax,szBuffer
```

--------------------------------------------------------------

感谢飘云老大、猫老大、Nisy老大以及很多前辈们的学习教程以及徐超等所有帮助过我的论坛兄弟姐妹们！谢谢

--------------------------------------------------------------

【版权声明】破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
