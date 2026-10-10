

# Winzip8.0

> 原文：[`www.pediy.com/kssd/pediy03/forum804.htm`](https://www.pediy.com/kssd/pediy03/forum804.htm)

\[Options\]   
CommandLine=winzip32.exe   
Mode=2   
First\_Break\_Address=407AA9   
First\_Break\_Address\_Code=56   
First\_Break\_Address\_Code\_Lenth=1   
Second\_Break\_Address=4692F3   
Second\_Break\_Address\_Code\_Lenth=2   
Save\_Code\_Address=EDI 

\------------------------------------------------------------------------------------   
PeterChen:   
以上是个人总结,学习就是要好好的总结才能得出新的好经验!   
\[Options\]   
CommandLine=   
Mode=   
First\_Break\_Address=   
First\_Break\_Address\_Code=   
First\_Break\_Address\_Code\_Lenth=   
Save\_Code\_Address=   
\-------------------------------------------------------------------------------------   
整个注册机程序内容就是这些内容!   
CommandLine              这是被取注册码的 EXE 文件名      
Mode                    采用模式读取注册码   
First\_Break\_Address      放输入密码或者是 ID (假的密码),的地址!   
First\_Break\_Address\_Code 通过是取输入汇编内容,的十六值(机器码)   
                        前两个代码!也就是 1byet 长度!   
First\_Break\_Address\_Code\_Lenth 这是填长度,由于是取代码   
                              总长度是 2,才对一时之把字符串   
                              与汉字混淆了!数字与字母都是   
                              一个才半个字节才对.   
Save\_Code\_Address        就是放真正密码的地址或者是寄存器   
大一致上就是这样了,其它模式大小同理!   
不过要注意只能作用 trw2000 or sice,直接找到密码软件的注册机   
也许只能作出用"d"找到密码的注册机!   
\--------------------------------------------------------------------------------------  4dd466d0e1e0cc123afbe504da2462ca