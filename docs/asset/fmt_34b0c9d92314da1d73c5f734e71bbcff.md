
# Writing JIT-Spray Shellcode for fun and profit - CHS[更新完整版]

> 原文：[`www.pediy.com/kssd/pediy12/108861.html`](https://www.pediy.com/kssd/pediy12/108861.html)

正式版奉上，各位敬请斧正

引用: 客户端浏览器的攻击一直都是所有人真正的威胁。在这里，漏洞已经不仅在浏览器中，而且也存在于插件中。银行的客户端，商业软件，防病毒软件，所有使用 ActiveX（for IE）的客户端已经被发现或者还存在很多漏洞。供应商已经做出一些措施来保护我们。软件供应商修复漏洞，操作系统厂商使用的新机制以防止攻击。但是，安全研究人员正试图找到办法绕过这些机制。在新版本浏览器（Internet Explorer 8 和 Firefox 3.5）中了使用永久 DEP 技术进行保护。新版本的操作系统中则使用了 ASLR 技术机制。这一切使老的攻击方法成为了不可能。但在 BlackHat DC 2010 上，一个有趣的方法绕过浏览器 DEP 和 ASLR 技术的 Just-In-Time 编译技术被公布了。这种方法被称为 JIT-Spray 技术。但是目前还没有一个人公布有效地 Poc。在本文中，我们将介绍如何编写一个新的 JIT Spray 攻击 shellcode，并且制作一个通用 Stage-0 shellcode，例如可以将控制权交给 metasploit 中的任何 shellcode。

如果你有趣，可以到我的博客：[`ipfans.blog127.fc2.com`](http://ipfans.blog127.fc2.com)

`ipfanscn` <at> gmail[dot]com

上传的附件

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || Writing-JIT-Spray-Shellcode-test.pdf |
| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || Writing-JIT-Spray-Shellcode-CHS.pdf |
