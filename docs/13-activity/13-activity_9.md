

# 《加密与解密(第三版)》学习指引(第六章)加密算法

> 原文：[`www.pediy.com/kssd/pediy10/72206.html`](https://www.pediy.com/kssd/pediy10/72206.html)

由于本人也完全看不懂这章，请将本贴视为读后感。主题依然是学习指引，纯粹是为了保持系列的一致性  
  
作为本章的扩展阅读，你可能会对下列内容感兴趣  
  

引用:   
ECC 加密算法入门介绍 作者:zmworm   
[`www.pediy.com/bbshtml/BBS6/pediy6014.htm`](http://www.pediy.com/bbshtml/BBS6/pediy6014.htm)  
  
探讨 ECC 加密被破译的可能性 作 者: newsearch  
[`bbs.pediy.com/showthread.php?t=34446`](http://bbs.pediy.com/showthread.php?t=34446)  
  
DSA 算法的理论，实现，以及在破解中的应用 作者:娃娃\[CCG\]   
[`www.pediy.com/bbshtml/bbs6/pediy6767.htm`](http://www.pediy.com/bbshtml/bbs6/pediy6767.htm)  
  
《RSA 与大数运算》 作者:afanty   
[`www.pediy.com/bbshtml/bbs5/pediy50664.htm`](http://www.pediy.com/bbshtml/bbs5/pediy50664.htm)  
  
CRC 技术的实现与破解 作 者: vipchenji  
[`bbs.pediy.com/showthread.php?t=17195`](http://bbs.pediy.com/showthread.php?t=17195)  
  
Twofish 加密算法详解 作 者: lionel  
[`bbs.pediy.com/showthread.php?t=10108`](http://bbs.pediy.com/showthread.php?t=10108)  
  
Crypto API 学习笔记 作 者: jdxyw  
[`bbs.pediy.com/showthread.php?t=31357`](http://bbs.pediy.com/showthread.php?t=31357)   
[`bbs.pediy.com/showthread.php?t=31374`](http://bbs.pediy.com/showthread.php?t=31374)  
[`bbs.pediy.com/showthread.php?t=31490`](http://bbs.pediy.com/showthread.php?t=31490)  

在软件保护方面，我学到了什么?  
  

引用: 1. **6.1.1**介绍了 PEID 的 KANAL 插件识别算法的原理，从原理上对其攻击，可使它失效  
2. **6.5**介绍的加密库，实现注册验证功能很方便，同时也方便了破解者。按照原理自己实现会给破解者造成一些困难  
3. **6.3**介绍的算法，破解者可以通过 PATCH 算法依赖的 NP 问题中的某些参数，写出注册机。如何保护这些常量也是防爆破的关键之一 

在软件破解方面，我学到了什么？  
  

引用: 1. **6.1.1**和**6.5**分别介绍了识别密码学算法的两种快捷的方法。PEID 的 KANAL 插件和 IDA 的 SIG  
2. **6.3**记住那些 NP 难问题，以及对应的工具。patch 那些关键的参数，使得工具能在短时间内给出解。接下来就可以写注册机了  
3. 为了保证工具识别不了的情况下，依然能通过阅读汇编代码来识别。必须熟读本章介绍的算法。即使完全不理解算法