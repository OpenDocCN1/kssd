

# 加解密（第三版）外壳代码 Bug

> 原文：[`www.pediy.com/kssd/pediy11/109615.html`](https://www.pediy.com/kssd/pediy11/109615.html)

加密与解密（第三版）第十六章，外壳代码存在 Bug  
  
外壳第一段中在解压第二段数据前在分配内存函数 Virtualalloc 最后一个参数当前为 PAGE\_READWRITE 在 windows2003 和 windows2008 上都会出问题，程序一跳到第二段后久会出现内存地址非法访问，程序无法执行，在 windows 其他版本系统上不会出现，可能是 2003 和 2008 服务器系统版本安全性高些吧。  
上面那参数应改为 PAGE\_EXECUTE\_READWRITE，让分配的内存具有读，写，执行权限  
改后测试就没有问题了。 

谢谢您的发现，这确实可以算是个错误。  
当初我刚开始写的时候还是用的 98，没有 DEP 保护，所以没有认真考虑这个。  
再次感谢您的认真发现。