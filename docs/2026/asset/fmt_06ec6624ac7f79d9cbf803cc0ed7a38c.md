
## 汇编代码

```
0046F360   ADD DL,CL                            与字符取高4位的结果相加
0046F362   MOV BYTE PTR DS:[EAX+EBX-1],DL       保存结果
0046F366   INC ESI
0046F367   CMP ESI,6
0046F36A   JLE SHORT HWN.0046F371
0046F36C   MOV ESI,1
0046F371   INC EBX
0046F372   DEC EDI
0046F373   JNZ SHORT HWN.0046F331               循环，这处循环体应该是将一个字串一个一个的与"anloer"
0046F375   MOV EAX,DWORD PTR SS:[EBP-8]         进行计算处理，但验证码的时候每次调用这个函数传的字串
0046F378   MOV EDX,DWORD PTR SS:[EBP-4]         都只有一个字符，因此每次都没循环，只走一次
0046F37B   CALL HWN.0040408C
0046F380   XOR EAX,EAX
0046F382   POP EDX
0046F383   POP ECX
0046F384   POP ECX
0046F385   MOV DWORD PTR FS:[EAX],EDX
0046F388   PUSH HWN.0046F39D
0046F38D   LEA EAX,DWORD PTR SS:[EBP-4]
0046F390   CALL HWN.00404038
0046F395   RETN
0046F396   JMP HWN.00403A3C
0046F39B   JMP SHORT HWN.0046F38D
0046F39D   POP EDI
0046F39E   POP ESI
0046F39F   POP EBX
0046F3A0   MOV ESP,EBP
0046F3A2   POP EBP
0046F3A3   RETN
```

好，`46F2F4`这个函数也分析清楚了，它将传入的字符的低四位与字符`"a"`的低四位作异或运算，变换传入的字符。

按追出的算法计算出一个可用的注册码：`A1ED-WEDF-EDSR-CEZD`，填入注册，成功！

## 破解总结

注册码验证算法如下：

1. 注册码必须是19位的。

2. 分别将注册码8、12、19、4位的字符的低四位与字符`"a"`的低四位进行`XOR`运算，高四位不动，结果分别与注册码3、7、11、17位比较是否相等。

3. 分别比较注册码1、2、5、6、9、10、13、14、15、16、18位是否是`"A、1、-、W、F、-、S、R、-、C、Z"`。

这个程序的用户名信息没有参与验证，注册成功后信息加密保存在注册表 `HKEY_LOCAL_MACHINE\SOFTWARE\Anloer Software\Hide Window Now\` 中名为`SN`的键值中，删掉就又变为不注册的了。函数`46FD10`是验证函数，如注册表里有相应键值，程序初始处也要调用这个函数验证，以确定是否显示`NAG`窗口。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
