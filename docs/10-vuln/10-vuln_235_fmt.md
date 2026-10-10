
# 对 CVE-2011-0073 的分析

> 原文：[`www.pediy.com/kssd/pediy12/135901.html`](https://www.pediy.com/kssd/pediy12/135901.html)

附件内容是本人对 `CVE-2011-0073` 这个 `Use-after-free` 漏洞的一些调试心得，放出来和大家分享个人的一些调试经验。不足之处恳请大家指正！![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

之前提供的 POC 中 `js_file3.js` 文件存在一些问题，而且文件名也不对![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")现在的附件中已经更新了![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

感谢 `wingdbg` 同学发现这个问题并向我提出！![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 对 CVE-2011-0073 的调试分析.rar |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || CVE-2011-0073 POC.rar |
