

# aspack 2.11

> 原文：[`www.pediy.com/kssd/pediy03/forum374.htm`](https://www.pediy.com/kssd/pediy03/forum374.htm)

 程序被 aspack2.11 压缩过后，它的 oep 放在压缩后文件的倒数第 2 个   
section 的第 7D-80 字节处（section name 通常为.aspack），这 4 个   
字节被加过密（其实整个 section 都被加密）。而且每次压缩的加   
密方法都不同。我只找出 2 种规律，其他的我找不出来，我以   
oep=00002DB8 为例（它在文件中是反着放的 B82D0000）：   
  
1。xor  这种情况是用某个数分别与 oep 的 4 个字节 xor，这时可以看   
oep 的第 1 个字节，因为 oep 通常为 00 ?? ?? ??，就可知道它是与哪   
个数 xor。如   
C0557878，那么 oep 的 4 个字节就分别与 78 xor。   
  
2。相加 这种情况是用某个数分别与 oep 的 4 个字节相加，同样要先   
看 oep 的第一个字节，如果相加的和大于 FF，就要减去 100。如   
68DDB0B0，那么 oep 的 4 个字节就分别与 B0 相加。（B8+B0=168-100=68）   
  
3。还有一些情况这样的，不知谁能找出它的算法（可能还有好几种）。   
  
7A55836D  4AC58DC2  62505D06  D27821CB  B82D0100  828BA10F   4dd466d0e1e0cc123afbe504da2462ca