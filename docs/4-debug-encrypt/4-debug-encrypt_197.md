

# DSA 算法

> 原文：[`www.pediy.com/kssd/tutorial/chap6-3-4.htm`](https://www.pediy.com/kssd/tutorial/chap6-3-4.htm)

Digital Signature Algorithm (DSA)是 Schnorr 和 ElGamal 签名算法的变种，被美国 NIST 作为 DSS(DigitalSignature Standard)。算法中应用了下述参数：  
  
p：L bits 长的素数。L 是 64 的倍数，范围是 512 到 1024；  
q：p - 1 的 160bits 的素因子；  
g：g = h^((p-1)/q) mod p，h 满足 h < p - 1, h^((p-1)/q) mod p > 1；  
x：x < q，x 为私钥 ；  
y：y = g^x mod p ，( p, q, g, y )为公钥；  
H( x )：One-Way Hash 函数。DSS 中选用 SHA( Secure Hash Algorithm )。  
p, q, g 可由一组用户共享，但在实际应用中，使用公共模数可能会带来一定的威胁。签名及验证协议如下：  
  
1\. P 产生随机数 k，k < q；  
2\. P 计算 r = ( g^k mod p ) mod q  
s = ( k^(-1) (H(m) + xr)) mod q  
签名结果是( m, r, s )。  
3\. 验证时计算 w = s^(-1)mod q  
u1 = ( H( m ) \* w ) mod q  
u2 = ( r \* w ) mod q  
v = (( g^u1 \* y^u2 ) mod p ) mod q  
若 v = r，则认为签名有效。  
  
　　DSA 是基于整数有限域离散对数难题的，其安全性与 RSA 相比差不多。DSA 的一个重要特点是两个素数公开，这  
样，当使用别人的 p 和 q 时，即使不知道私钥，你也能确认它们是否是随机产生的，还是作了手脚。RSA 算法却作不到。

 

 4dd466d0e1e0cc123afbe504da2462ca