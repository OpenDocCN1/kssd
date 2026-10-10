

# ElGamal 算法

> 原文：[`www.pediy.com/kssd/tutorial/chap6-3-3.htm`](https://www.pediy.com/kssd/tutorial/chap6-3-3.htm)

**ElGamal 算法**

　　ElGamal 算法既能用于数据加密也能用于数字签名，其安全性依赖于计算有限域上离散对数这一难题。  
密钥对产生办法。首先选择一个素数 p，两个随机数, g 和 x，g, x < p, 计算 y = g^x ( mod p )，则其公钥为 y, g 和 p。私钥是 x。g 和 p 可由一组用户共享。  
ElGamal 用于数字签名。被签信息为 M，首先选择一个随机数 k, k 与 p - 1 互质，计算  
  
a = g^k ( mod p )  
再用扩展 Euclidean 算法对下面方程求解 b：  
  
M = xa + kb ( mod p - 1 )  
  
签名就是( a, b )。随机数 k 须丢弃。  
验证时要验证下式：  
  
y^a \* a^b ( mod p ) = g^M ( mod p )  
  
同时一定要检验是否满足 1<= a < p。否则签名容易伪造。  
ElGamal 用于加密。被加密信息为 M，首先选择一个随机数 k，k 与 p - 1 互质，计算  
  
a = g^k ( mod p )  
b = y^k M ( mod p )  
  
  
( a, b )为密文，是明文的两倍长。解密时计算  
  
M = b / a^x ( mod p )  
  
　　ElGamal 签名的安全性依赖于乘法群(IFp)\* 上的离散对数计算。素数 p 必须足够大，且 p-1 至少包含一个大素数  
因子以抵抗 Pohlig & Hellman 算法的攻击。M 一般都应采用信息的 HASH 值(如 SHA 算法)。ElGamal 的安全性主要依赖于 p 和 g，若选取不当则签名容易伪造，应保证 g 对于 p-1 的大素数因子不可约。D.Bleichenbache“GeneratingElGamal Signatures Without Knowing the Secret Key”中提到了一些攻击方法和对策。ElGamal 的一个不足之处是它的密文成倍扩张。  
  
　　美国的 DSS(Digital Signature Standard)的 DSA(Digital Signature Algorithm)算法是经 ElGamal 算法演  
变而来。

 

 4dd466d0e1e0cc123afbe504da2462ca