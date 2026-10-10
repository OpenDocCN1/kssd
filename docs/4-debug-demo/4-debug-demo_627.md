

# 去除一个魔方小游戏的弹出广告链接（适合初学者）

> 原文：[`www.pediy.com/kssd/pediy11/124595.html`](https://www.pediy.com/kssd/pediy11/124595.html)

前几天我玩了一个 3d 的魔方小游戏，气人的是它竟然一打开就出来作者的博客。作者竟然用这种手段骗点击率。愤怒之下，将其破解，原来很简单。  
  
OD 载入，用超级字串参考查广告链接"http://hi.baidu.com/liusijian0710"，定位到  
004087B5  处：  
  
004087AF      6A 01         push    1  
004087B1  |.  6A 00         push    0                                ; |DefDir = NULL  
004087B3  |.  6A 00         push    0                                ; |Parameters = NULL  
004087B5  |.  68 38A54100   push    0041A538                         ; |http://hi.baidu.com/liusijian0710  
004087BA  |.  68 30A54100   push    0041A530                         ; |open  
004087BF  |.  51            push    ecx                              ; |hWnd => NULL  
004087C0  |.  FF15 2CA14100 call    dword ptr \<&SHELL32.ShellExecut>; \\ShellExecuteA  
004087C6  |.  33C0          xor     eax, eax  
004087C8  |.  BD 12000000   mov     ebp, 12  
  
  
在 004087C0 处调用了 API 函数 ShellExecuteA 来打开广告链接，前边几句是把 ShellExecuteA 的参数压入堆栈，广告链接就是其中一个参数。  
  
只要把  
  
004087AF      6A 01         push    1   
修改为：  
  
004087AF     /EB 15         jmp     short 004087C6  
  
就跳过广告链接了，哈哈！

上传的附件
| ![ || 3D 魔方游戏.rar  
 |