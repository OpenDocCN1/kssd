
对`procmon.exe`的`winmain`函数进行粉碎生成`procmon2.exe`

![](https://bbs.kanxue.com/upload/attach/202001/784599_PKAF62ZZ987C55W.jpg)

`procmon2.exe`正常运行

![](https://bbs.kanxue.com/upload/attach/202001/784599_ZKHZKZQD7VCKDF7.jpg)

**混淆前:**

![](https://bbs.kanxue.com/upload/attach/202001/784599_9B4NBNQ7EQMQD3D.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/784599_73SMNA5C9ZA8XER.jpg)

**混淆后:**

![](https://bbs.kanxue.com/upload/attach/202001/784599_T53TBE7Y7TBJTUD.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/784599_B3S2QV85EQK3RR9.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/784599_NEPVVG57P6KVF72.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/784599_ST3EXMKUEKKHKCR.jpg)

原理很简单，只实现了乱序粉碎的功能，而且是很简单的**函数粉碎**，只作为**学习的一个玩具demo**参考

**代码变形，常量隐藏，导入表加密等等功能都没有加入**。**没什么技术含量**，代码很垃圾，**大佬轻喷**

发个demo bin玩玩，把`Obfuscater.exe`和`procmon.exe`放在同一目录，运行即可生成`procmon.obf.exe`

`Obfuscater.exe`处理的exe和函数我都写死了，想用的大佬可以**自行逆向patch**一下

![](https://bbs.kanxue.com/upload/attach/202001/784599_N2BTK3NTFWDJCKA.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
