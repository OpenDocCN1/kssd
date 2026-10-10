

# Core 的 crackme2

> 原文：[`www.pediy.com/kssd/pediy03/forum953.htm`](https://www.pediy.com/kssd/pediy03/forum953.htm)

原算法：   
  
1\. userinfo + strrev( userinfo ) => string   
2\. 求 md\[ 4 \] = altered\_MD5( string )   
3\. ( code0, code1 ) = decrypt( code1, code2 )   
4\. ( code1, code2 ) = decrypt( code1, code2 )   
5\. if( md\[0\] == code0 && md\[1\] == code1 && md\[2\] == code2 )   
  then regok();   
  
注解：   
1\. 头一部分是修改过的 MD5   
2\. 两重 decrypt 避免了穷举攻击，这两重 decrypt 是交错的，这使得算法可以更加有效地避免穷举攻击 。  
3\. decrypt() 中使用两个由 32 个 64 位数构成的 s-box 对输入的 64 位数进行 32 轮置换；该 s-box 是随机生成的   
4\. 这种加密是基于算法的强度，不是基于 password 或者 s-box 的强度的，存在另外一个算法，该算法既可以实现加密，也可以实现解密。crackme2 中使用的解密流程是经过有意修改的，使得解密者必须在理解它的基础上反推出该双向算法   
 4dd466d0e1e0cc123afbe504da2462ca