

# 关于 1024 位的 RSA 因式分解速度的问题

> 原文：[`www.pediy.com/kssd/pediy10/77435.html`](https://www.pediy.com/kssd/pediy10/77435.html)

对 1024 位的 RSA 进行因式分解有没有什么好的办法可以加快速度，是否只能替换密钥了? 

分解 RSA 数，目前最快的还是 NFS，是亚指数级复杂度，1024 bits 与 512 bits 有天壤之别。  
  
有一个纸上谈兵的分解机器，不知美国佬有没有制造出来。  
The TWIRL integer factorization device  
http://people.csail.mit.edu/tromer/twirl/  
  
这种 TWIRL 机，可以花 5000 美元的制造成本,10 mins 内分解 RSA 512 bits 大数。  
可能用 1000 万美元的制造成本，1 年分解 RSA 1024 bits 大数。研发成本可能 2000 万美元。  
  
只有政府或财力大佬才能搞定。 

对称加密有 128 bits AES 就足够安全了。  
RSA 主要用于签名或交换密钥。  
  

引用: *最初由 **NetRoc**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=541447#post541447) 听说绝密的东西都是用 8192 位密钥的，加个 word 文档都要跑几分钟![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")