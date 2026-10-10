

# IDA 插件 Hex-Rays.Decompiler.v1.0.for.IDA5.2 的小 bug

> 原文：[`www.pediy.com/kssd/pediy10/62949.html`](https://www.pediy.com/kssd/pediy10/62949.html)

如题，对 strlen 函数的编译存在小纰漏。  
mov     edi, offset aString ; "hello"  
or      ecx, 0FFFFFFFFh  
xor     eax, eax  
repne scasb  
not     ecx  
dec     ecx  
  
利用插件生成的函数为 strlen("hello")-1;  
  
看来最后的 dec 误导了 hex-Ray。希望大家注意。  
对不起又火星了一把，高手飘过。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")