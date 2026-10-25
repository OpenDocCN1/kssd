
```
0045F6BE  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]

0045F6C1  |.  BA 02000000   MOV EDX,2

0045F6C6  |.  E8 5141FAFF   CALL Othello2.0040381C

0045F6CB  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]

0045F6CE  |.  BA 02000000   MOV EDX,2

0045F6D3  |.  E8 4441FAFF   CALL Othello2.0040381C

0045F6D8  \.  C3            RETN
```

## 算法如下

1、把用户名每个字符Ascii码相加，得到`sum_username`

2、把`HelloCracker`的每个字符Ascii码相加，得到`sum_hellocracker`

3、计算 `IntToStr( (sum_username * 198611) xor (sum_hellocracker * 861126) )`，得到字符串`strTemp`

4、依次从`strTemp`中和`HelloCracker`中取出字符，其Ascii码相加，再减去`30H`，组成注册码，`strTemp`取完为止。

5、注册机已写成，把界面修理的Cool一点的时候即可发布。

注册码是:`OnuooLvae`

这里面有不少参数和算法好象在绕圈，但好象实际上没有参加注册码的运算？

`call 004056F4`里的代码，push来push去，mul来mul去的，却只是一个简单的两数相乘的运算？

ps:作者的生日好象是1986年11月26日 ;-)
