
# [原创]docPrint Pro 8.0 - 'Add URL' Buffer Overflow 逆向分析

## 调试环境
调试器：`ollydbg`
平台：`windows 10 x64`（作者的`poc`利用平台为`Windows 7 32-bit`）

7-26的一个缓冲区溢出漏洞，`PE`结构的`DllCharacteristics`字段为`0`，意味着此程序未开启`ASLR`和`DEP`。

![](upload/attach/202007/870536\_9PEWMSNGEF2GGFE.png)

![](upload/attach/202007/870536\_8HG3PUGTRSBVVX6.png)

根据`poc`中的提示，打开文件，并`attach`程序，第一次会在`0x405E41`处断下并发生违规写入。那么首先需要找到的便是溢出地址的产生即`edi`的地址从哪来。

![](upload/attach/202007/870536\_9A5D93R6X8GMAWW.png)

观察寄存器发现`ebp`为`0`,说明并不能使用栈回溯，观察`0x405e19`处的代码，`je 0x405e26`中间跳过`retn`，说明函数的入口在`0x405e00`,既然是函数调用那么必然要使用`call`，那么必定会压入函数返回地址，在`0x405e00`出下断，那么`esp`处就是函数的返回地址。在`0x405e00`断下后发现`edi`为`0x1`。

说明`edi`并没有赋值，那么赋值动作就发生在`0x405e1-0x40500`中间，阅读代码`0x405e31`处给`edx`赋值，`0x405e3c`处将`edx`的值赋给`edi`。那么就得到了漏洞发生地址，缓冲区大小=`函数返回地址-esp+0x130`。

漏洞是由于获取文件属性失败时，拷贝字符串发生的缓冲区溢出漏洞

## 附件
上传的附件：
- [48719.py](javascript:void(0);)
  （5.20kb，3次下载）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
