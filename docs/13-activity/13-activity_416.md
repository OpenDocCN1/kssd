

# 2011 Microsoft ExploitMe 活动

> 原文：[`www.pediy.com/kssd/pediy12/133332.html`](https://www.pediy.com/kssd/pediy12/133332.html)

2011 Microsoft ExploitMe 活动

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || exploit\_me.rar  
 |

shellcode 最后一字节为 cc 执行到这里引发异常，异常发生后又会执行 shellcode，于是 死循环了  
从 cc 这里要设计一段安全退出的代码