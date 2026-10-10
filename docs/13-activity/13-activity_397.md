

# \[TX2010 第二阶段 第二题\]keygen 源代码(效率优化版)

> 原文：[`www.pediy.com/kssd/pediy11/124312.html`](https://www.pediy.com/kssd/pediy11/124312.html)

简单描述一下要点，已知条件:  
a = C564EEF070E69190  
b = 22996B9C33AEEFDB  
p = C564EEF070E69193  
n = C564EEF19A080B07  
px = 2223A1D595845FA2  
py = 1C4EEE9222DDDA62  
qx = 297A4A1E5B1FC99B  
qy = 880198E3724F9FEE   
  
对应椭圆曲线:  
y² = x³ + C564EEF070E69190\*x + 22996B9C33AEEFDB  
  
用 ECDLP Solver 求解出 d:  
d = 54656E63656E7420 = "Tencent "  
  
签名过程:  
e = sha1(name)  
r = px  
s = (rd+e) mod n  
r 和 s 拼起来就是 16 字节的签名结果  
直接用 px 虽然不安全，但是合法，额外的好处是省去了对大数库的依赖:)  
  
解决了这个 ecdsa 部分以后，其他算法比如 blowfish 和 sms4 都应该没什么难度了，找到标准算法，只要细心对比修改一下就可以了。

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || kg\_22.rar  
 |