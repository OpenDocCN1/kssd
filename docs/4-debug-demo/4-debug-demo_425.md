

# 发一个重定向 hosts 文件的方法

> 原文：[`www.pediy.com/kssd/pediy12/141291.html`](https://www.pediy.com/kssd/pediy12/141291.html)

用 procexp.exe 找到启动参数是 C:\\WINDOWS\\system32\\svchost.exe -k NetworkService 的 svchost 进程，  
  
然后用 winhex 打开此进程，搜索\\drivers\\etc\\hosts，修改一下这个文件名，比如\\drivers\\etc\\hao2，  
  
然后随便改改 etc 目录下的文件(FindFirstChangeNotificationW)，svchost 就会读取\\drivers\\etc\\hao2 更新 hosts cache，  
  
真正起作用的就变成 hao2，而不是 hosts 了。  
  
跟 hosts 文件有关的 DLL 是 dnsrslvr.dll 和 dnsapi.dll