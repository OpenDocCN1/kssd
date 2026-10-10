

# 初级 peDIY，给窗口增加最大化按钮

> 原文：[`www.pediy.com/kssd/pediy11/120169.html`](https://www.pediy.com/kssd/pediy11/120169.html)

【破文标题】初级 peDIY，给窗口增加最大化按钮  
【破文作者】zaas\[PYG\]  
【破解工具】OllyICE，PEiD v0.94  
【破解平台】WinXP  
  
这两天在用【职称工作信息系统个人版】申报职称，填报的这软件忒弱智，偌大的屏幕只能在这样一个小窗口里拖拽，不胜其烦。  
![](img/7487ef60c4211dcf1603323170ceeceb.jpg)  
遂打算自己动手，丰衣足食。  
最初以为用 exescope 改一下资源就 OK 的，查了一下壳，没想到是 Watcom C 的程序，exescope 看不到窗口信息。  
无奈，只好 OD 了。无非是创建窗口而已，直接 bp CreateWindowExA。断下。  

代码:

```
10BDB6D7    |.  51                                  push ecx                                                     ; /lParam
10BDB6D8    |.  8B4E 44                             mov ecx,dword ptr ds:\[esi+44\]                                ; |
10BDB6DB    |.  52                                  push edx                                                     ; |hInst
10BDB6DC    |.  8B56 40                             mov edx,dword ptr ds:\[esi+40\]                                ; |
10BDB6DF    |.  6A 00                               push 0                                                       ; |hMenu = NULL
10BDB6E1    |.  50                                  push eax                                                     ; |hParent
10BDB6E2    |.  8B4424 4C                           mov eax,dword ptr ss:\[esp+4C\]                                ; |
10BDB6E6    |.  03C8                                add ecx,eax                                                  ; |
10BDB6E8    |.  8B46 3C                             mov eax,dword ptr ds:\[esi+3C\]                                ; |
10BDB6EB    |.  51                                  push ecx                                                     ; |Height
10BDB6EC    |.  8B4E 38                             mov ecx,dword ptr ds:\[esi+38\]                                ; |
10BDB6EF    |.  52                                  push edx                                                     ; |Width
10BDB6F0    |.  8B5424 28                           mov edx,dword ptr ss:\[esp+28\]                                ; |
10BDB6F4    |.  50                                  push eax                                                     ; |Y
10BDB6F5    |.  8B4424 40                           mov eax,dword ptr ss:\[esp+40\]                                ; |
10BDB6F9    |.  51                                  push ecx                                                     ; |X
10BDB6FA    |.  8B4E 08                             mov ecx,dword ptr ds:\[esi+8\]                                 ; |
10BDB6FD    |.  52                                  push edx                                                     ; |Style
10BDB6FE    |.  8B5424 38                           mov edx,dword ptr ss:\[esp+38\]                                ; |
10BDB702    |.  50                                  push eax                                                     ; |WindowName
10BDB703    |.  51                                  push ecx                                                     ; |Class
10BDB704    |.  52                                  push edx                                                     ; |ExtStyle
10BDB705    |.  FF15 3CD8DB10                       call dword ptr ds:\[<&USER32.CreateWindowExA>\]                ; \\CreateWindowExA

```

Style 来自 edx，值为 0x86c80000，翻了翻 msdn：  
代码:

```
WS\_SYSMENU  00080000h  带系统菜单（即带标题栏左上角的图标）

WS\_THICKFRAME  00040000h  可以拖动调整大小的边框

WS\_MINIMIZEBOX  00020000h  有最小化按钮

WS\_MAXIMIZEBOX  00010000h  有最大化按钮

```

第四位为窗口按钮，软件默认值为 8，即只有标题栏左上角的图标。如果要加最大化图标，8+1=9，全打开则 8+4+2+1=0xF。  
向上翻程序，发现 edx 值来自：  
代码:

```
10BDAF90 PB>/$  81EC 84020000                       sub esp,284
10BDAF96    |.  8B8424 88020000                     mov eax,dword ptr ss:\[esp+288\]
10BDAF9D    |.  53                                  push ebx
10BDAF9E    |.  55                                  push ebp
10BDAF9F    |.  56                                  push esi
10BDAFA0    |.  8B48 52                             mov ecx,dword ptr ds:\[eax+52\]
10BDAFA3    |.  8B68 04                             mov ebp,dword ptr ds:\[eax+4\]
10BDAFA6    |.  8B50 10                             mov edx,dword ptr ds:\[eax+10\]
10BDAFA9    |.  8BB424 98020000                     mov esi,dword ptr ss:\[esp+298\]
10BDAFB0    |.  8B81 A0000000                       mov eax,dword ptr ds:\[ecx+A0\]
10BDAFB6    |.  33DB                                xor ebx,ebx
10BDAFB8    |.  894424 28                           mov dword ptr ss:\[esp+28\],eax
10BDAFBC    |.  8B46 10                             mov eax,dword ptr ds:\[esi+10\]

```

此时  
ds:\[02492748\]=96C80000  
eax=00400000 (hnzcdm.00400000)  
PBVM90.FN\_CtlCreate+2C  
初始值并非 0x86c80000，看来直接修改不可行。继续向下走，发现有两处运算:  
代码:

```
10BDB287    |.  8B4C24 10                           mov ecx,dword ptr ss:\[esp+10\]                                ;  Case 4074 of switch 10BDAFE1
10BDB28B    |.  8B5424 20                           mov edx,dword ptr ss:\[esp+20\]
10BDB28F    |.  81E1 FFFFFFEF                       and ecx,EFFFFFFF
/////////////////////////////////////////////////////////////////////////////////
10BDB2F1    |.  8B4C24 10                           mov ecx,dword ptr ss:\[esp+10\]
10BDB2F5    |.  8B4424 14                           mov eax,dword ptr ss:\[esp+14\]
10BDB2F9        81E1 FFFFC8FF                       and ecx,FFC8FFFF

```

第二次运算后得出最终的 Window Style。直接修改为：  
10BDB2F9        81C9 0000CD87                       or ecx,86CD0000  
保存。  
![](img/616e32f30c451df1ca919326f0a2a875.jpg)  
收工。  
【版权声明】破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!