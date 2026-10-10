

# QQScLauncher 简单分析

> 原文：[`www.pediy.com/kssd/pediy12/141688.html`](https://www.pediy.com/kssd/pediy12/141688.html)

QQScLauncher 是什么？  
QQ2011 正式版有一个把好友拖到桌面的功能，发现快捷方式里的目标程序时 QQScLauncher.exe，如:  

代码:

```
"C:\\Program Files\\Tencent\\QQ\\Bin\\QQScLauncher.exe" /uin:842342202 /quicklunch:1D0BEC54CBEE33C09ED6F8089A7528E12EA3F4892C20DE8B0E8EF1144D765CD46176A599C09696DF



```

简单的逆了一下这个程序的逻辑。后面那串很长的十六进制的解码是在 QQ 主程序里面完成的，暂时没有去找，有兴趣的朋友可以试试看。  
说明：

1.  程序没有经过处理的，很适合新手逆一逆![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")
2.  只是感兴趣，没有任何其他意图，请勿将本帖所述内容用于非法用途，否则后果自负。

上传的附件
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || QQScLauncher.zip  
 |
| ![](img/b7e61e3627e1e92d7b948dd8fdd24fa6.jpg) || QQScLauncher 逆向分析.doc  
 |
| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || QQScLauncher 逆向分析.pdf  
 |