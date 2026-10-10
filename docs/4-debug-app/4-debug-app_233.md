

# 金山游侠弹出窗口背后的秘密

> 原文：[`www.pediy.com/kssd/pediy11/125162.html`](https://www.pediy.com/kssd/pediy11/125162.html)

![](img/962324edb48d57effae0c0571850a66b.jpg "酷")  
   一直以来在想金山游侠在游戏中弹出的方式，在网上找了很久，网上也有很多人在问，找来找去只有个在 DX 游戏中弹出的 VC 代码，在 MSDN 下载还花了我几个积分...，该代码在不能在 DirectDraw 游戏中弹出。  
  在写大富翁 4 游戏的修改器时，该游戏不能切换，键盘钩子也无效，迫使需要一种想金山游侠似的在游戏中弹出才行。即然写外挂不公布弹出技术（我猜测外挂用的是挂钩 DirectDraw 或 D3D 函数方式，但这种方式我觉得通用性不强），那我就自已动手分析金山游侠是怎么做的...  
   经过一段漫长时间，约两个星期吧，其中一个星期是每天 8 小时分析（还好在成都电力职大学习网络与安全有的是时间，都没听老师讲的什么，真对不住许老师呀），边分析，边做测试代码，顺便说句，国产内核调试器 Syser Debug 真不稳定，动不动蓝屏。  
  那么，金山游侠按了快捷键后，到底做了些什么呢？  
  一切秘密都在 KSKnight.dll 中....  
   1、先安装三个全局钩子：CBT、Mouse、Keyboat，只有 CBT 钩子是有用的，其它的都是空的。  
  下面全是在 CBT 钩子中  
  
  2、当按下快捷键后，取得 DirectDrawCreate 地址，创建个 DX 对象接口。  
  3、用 EnumSurface 或 GetGDISurface 枚举指定进程 PID 的主表面（原来 DX 接口是个链表）接口，找到后用 QueryInterface 取得新的 DX 对象新接口，并释放原 DX 接口和查找到主表面接口。  
  4、查询新 DX 对象接口的显示模式，即 GetDisplayMode，比较是不是 8 位色彩。  
  5、再次枚举主表面，将枚举到的主表面保存。  
  6、锁定主表面  
  7、检测操作系统版本，尝试取 DINPUT.DLL，枚举所有存在的线程，对游戏进程句柄复制，创建新线程，在新线程中再次锁定游戏主表面，暂停线程（这儿我的测试代码有问题，好像金山对所有线程暂停后又能马上恢复了）。  
  8、替换原游戏的 WNDPROC 消息过程，在新消息过程中涉及绘制的全返回 0。  
     终于可以创建对话框了....，金山在这的代码还很长，没分析完。  
  
以上是大概金山的流程，我写了个测试代码，可以在大富翁 4 和星际 1.08 是弹出对话框，但有点问题--只要移动对话框，背影全是对话框的影子，在魔兽 3 中弹不出来，游戏锁死，这段还分析中...。  
目前只是完成了一部分工作，至少能在 2D 游戏中弹出了，成功后放出测试代码。  
  
   感谢，南充电业局的贾雨果（烟都让我抽了不少）和成都电业局的西风 X。  
   帖出 IDA 的片段:  
text:1002D43A                 call    sub\_1001C920    ; 取得 DirectDrawCreate 地址,放在 1006da2c  
.text:1002D43F                 test    eax, eax  
.text:1002D441                 jz      loc\_1002D50E  
.text:1002D447                 mov     edx, \[esp+78h+arg\_0\]  
.text:1002D44B                 push    edx  
.text:1002D44C                 call    sub\_1001C9D0    ; 创建 DX 对象并查找主表面接口，返回新 DX 对象接口  
.text:1002D451                 add     esp, 4  
.text:1002D454                 cmp     eax, ebp  
.text:1002D456                 mov     dword\_1006D960, eax  
.text:1002D45B                 jz      loc\_1002D50E  
.text:1002D461                 push    eax  
.text:1002D462                 call    sub\_1001CA60    ; 取得新 DX 接口的显示模式，即 GetDisplayMode  
.text:1002D467                 mov     dword\_1006D970, eax  
.text:1002D46C                 mov     eax, dword\_1006D960  
.text:1002D471                 push    eax  
.text:1002D472                 call    sub\_1001C980    ; 查找新 DX 对象接口的主表面  
.text:1002D477                 mov     esi, eax  
.text:1002D479                 add     esp, 8  
.text:1002D47C                 cmp     esi, ebp  
.text:1002D47E                 jz      short loc\_1002D4F9  
.text:1002D480                 push    edi  
.text:1002D481                 mov     ecx, 1Bh  
.text:1002D486                 xor     eax, eax  
.text:1002D488                 lea     edi, \[esp+7Ch+var\_6C\]  
.text:1002D48C                 push    ebp  
.text:1002D48D                 lea     edx, \[esp+80h+var\_6C\]  
.text:1002D491                 rep stosd               ; memset(&gameDdsd,0,sizeof(gameDdsd));  
.text:1002D493                 mov     ecx, \[esi\]  
.text:1002D495                 push    1  
.text:1002D497                 push    edx  
.text:1002D498                 push    ebp  
.text:1002D499                 push    esi  
.text:1002D49A                 mov     \[esp+90h+var\_6C\], 6Ch ; gameDdsd.dwSize = sizeof(gameDdsd);  
.text:1002D4A2                 call    dword ptr \[ecx+64h\] ; Lock 主表面，返回锁定的显存指针  
.text:1002D4A5                 test    eax, eax  
.text:1002D4A7                 pop     edi  
.text:1002D4A8                 jnz     short loc\_1002D4B4 ; 此处跳转  
.text:1002D4AA                 mov     eax, \[esi\]  
.text:1002D4AC                 push    ebp  
.text:1002D4AD                 push    esi  
.text:1002D4AE                 call    dword ptr \[eax+80h\] ; Unlock 主表面  
.text:1002D4B4  
.text:1002D4B4 loc\_1002D4B4:                           ; CODE XREF: sub\_1002D3A0+108j  
.text:1002D4B4                 push    esi             ; lpParameter  
.text:1002D4B5                 call    sub\_100233E0    ; 复制进程句柄，创建新线程，暂停目标进程线程  
.text:1002D4BA                 mov     ecx, \[esi\]  
.text:1002D4BC                 add     esp, 4  
.text:1002D4BF                 push    esi  
.text:1002D4C0                 call    dword ptr \[ecx+8\] ; 释放主表面接口  
.text:1002D4C3                 mov     edx, \[esp+78h+arg\_8\]  
.text:1002D4CA                 push    edx  
.text:1002D4CB                 push    ebx  
.text:1002D4CC                 call    sub\_1002D290    ; 创建新的窗口并显示它（最复杂、困难，没对其分析）  
.text:1002D4D1                 call    sub\_10023480    ; 恢复目标进程线程  
.text:1002D4D6                 mov     eax, dword\_1006D960  
.text:1002D4DB                 cmp     eax, ebp  
.text:1002D4DD                 jz      short loc\_1002D4EB  
.text:1002D4DF                 mov     ecx, \[eax\]  
.text:1002D4E1                 push    eax  
.text:1002D4E2                 call    dword ptr \[ecx+8\] ; 释放新 DX 对象接口