

# 病毒也来 PEDIY

> 原文：[`www.pediy.com/kssd/pediy10/74527.html`](https://www.pediy.com/kssd/pediy10/74527.html)

【文章作者】:  newjueqi    
【作者邮箱】: zengjiansheng1@126.com  
【作者 QQ 号】: 190678908(sdf)  
【编写语言】: VC++6.0  
【使用工具】: OD  
【操作平台】: XP-SP2  
【作者声明】: 昨天做了一个病毒的汇编源码分析（链接：http://bbs.pediy.com/showthread.php?t=74520 或者看附件），分析完这个病毒后的感慨就是这个病毒一点都不像病毒，反而像一个 U 盘编程的示范代码（虽然金山毒霸每次遇到它都兴奋得叫起来 ^-^）， 后来就干脆给病毒来一个 PEDIY（直接修改汇编代码使病毒在运行时弹出一个对话框告诉用户中毒了），只是出于研究的目的，失误之处敬请大家赐教!  
  
网页上有的图片不太清晰，清晰版请看附件  
  
具体思路就是修改病毒注册的消息处理函数地址，通过提前处理 WM\_CREATE 消息显示一个对话框，然后才返回到原来病毒的函数处理地址。  
  
查看分析过程，可知病毒以载入 MessageBoxA, 省下修改输入表的内容。  
首先用 LordPE 查看区块信息，如下图所示  
![](img/329d4e103b489cec6c47a3f0ff8f7f43.jpg)  
  
从上图可看到.text 的节区空隙为 7000-6B1A=4E6,即可放 1000 多字节，足够放入所需要的代码，另外从文件偏移 8000 的地方是数据区，可找空白的存放 MessageBoxA 函数所有的对话框标题和内容。  
  
首先，用 Winhex 打开病毒文件，我选择在 00008270 的地方存放内容文本，00008280 存放 MessageBoxA 需要的文本，完成后如下图所示  
![](img/b9d87de1b7dca061c9588439d7cd1708.jpg)  
  
接着就开始修改代码，从 http://bbs.pediy.com/showthread.php?t=74520 或附件的病毒分析可知：  
  
00401010         |.  897C24 28       mov     dword ptr \[esp+28\], edi              ; |  
00401014         |.  C74424 2C 00114>mov     dword ptr \[esp+2C\], 00401100    ;窗口过程的地址|  
0040101C         |.  897C24 30       mov     dword ptr \[esp+30\], edi              ; |  
00401020         |.  897C24 34       mov     dword ptr \[esp+34\], edi              ; |  
00401024         |.  897424 38       mov     dword ptr \[esp+38\], esi              ; |  
00401028         |.  897C24 3C       mov     dword ptr \[esp+3C\], edi              ;   
0040102C         |.  897C24 40       mov     dword ptr \[esp+40\], edi              ; |  
00401030         |.  897C24 44       mov     dword ptr \[esp+44\], edi              ; |  
00401034         |.  897C24 48       mov     dword ptr \[esp+48\], edi              ; |  
00401038         |.  C74424 4C 4C804>mov     dword ptr \[esp+4C\], 0040804C         ; 00401040         |.  FF15 18714000   call    dword ptr \[<&USER32.RegisterClassA>\] ; \\RegisterClassA  
  
00401100 就是病毒原来的消息处理地址，根据 LordPE 查看区块信息，在 00406B1B 处插入新增的代码，所以把跳转改为下图所示  
![](img/0786809c5595faddcd3903c75af2eaa1.jpg)  
  
接下来我们来分析一下函数的消息处理过程  
00401100   .  8B4424 08     mov     eax, dword ptr \[esp+8\]  
00401104   .  3D 13010000   cmp     eax, 113                         ;  Switch (cases 1..219)  
00401109   .  0F87 9C000000 ja      004011AB  
0040110F   .  74 60         je      short 00401171  
00401111   .  8BC8          mov     ecx, eax  
00401113   .  49            dec     ecx  
00401114   .  74 21         je      short 00401137 ；这里就跳到 WM\_CREATE 消息  
  
从上面可看到 Switch 语句里跳转到 WM\_CREATE 的消息处理的过程，我们可以进行一定程度上的简化。  
  
在地址 00406B1B 处开始输入我们 DIY 的代码  
  
00406B1B      8B4C24 08     mov     ecx, dword ptr \[esp+8\]  
；注意里，这里必须执行完 mov  ecx, dword ptr \[esp+8\]后才能 pushad， 因为如果执行了 pushad 操作，那么 esp 的值会发生变化，dword ptr \[esp+8\]的值就会受到影响  
00406B1F      60            pushad ；保护寄存器现场  
00406B20      49            dec     ecx  
00406B21      74 06         je      short 00406B29 ；比较是否是 WM\_CREATE 消息，如果不是的话就跳回病毒原来的消息处理函数  
00406B23      61            popad  ；恢复寄存器现场  
00406B24    ^ E9 D7A5FFFF   jmp     00401100 ；原来的消息处理函数  
  
下面就是我们弹出对话框的部分  
00406B29      6A 00         push    0 ；Style  
00406B2B      68 80824000   push    00408280              ;  Title, ASCII "ha ha"  
00406B30      68 70824000   push    00408270              ;  Text, ASCII "this is a virus"  
00406B35      6A 00         push    0 ; Owner  
  
      输入 MessageBoxA 的地址时就要注意了，.MessageBoxA 的地址千万不能用硬编码，因为随着操作系统的不同，.MessageBoxA 的函数地址也不相同，所以必须要获得当前操作系统中.MessageBoxA 的地址，据我所知方法有两个：  
1.  在输入表添加 MessageBoxA 函数，使输入表里拥有 MessageBoxA 的函数地址  
2.  用操作系统的特性先获取 GetProcAddress 函数的地址，再用 GetProcAddress 函数获取 MessageBoxA 的地址，方法在《Q 版黑客缓冲区溢出教程》里有详细的介绍  
  
  
   由于病毒已有使用 MessageBoxA 的地方，所以只要获取 MessageBoxA 在输入表的地址就 ok  
病毒里使用的 MessageBoxA 如下图  
![](img/b5cb9e2e2ac6a73b6ff2d6d3b368845c.jpg)  
  
40711c 就是输入表里 MessageBoxA 的地址  
  
  
00406B37      FF15 1C714000 call    dword ptr \[<&USER32.MessageBoxA>\]      ;  USER32.MessageBoxA  
00406B3D      61            popad  ；恢复寄存器现场  
00406B3E    ^ E9 BDA5FFFF   jmp     00401100 ；跳到原来的消息处理处  
  
完成的输入内容如下图红色所示  
![](img/09f46aa8ac9d61d73ba9c5bb819a09a4.jpg)  
  
在 OD 的反汇编窗口点右键-->复制到可执行文件-->右键保存文件(全部)-->选择你要保存的文件名-->保存，ok，这样汇编代码部分就修改完成。  
  
最后的工作就是修改区块表的信息。  
原来.text 区块头信息如下：  
![](img/47c2cfc5b39edfaee012f0eb175b52d8.jpg)  
  
5B1A 就是原来的区块大小，由于现在我们在原来的.text 区块尾增加了代码，必须改掉原来的.text 区块头信息，从增加的代码地址可看出，我们可以把.text 区块尾增加到 00406B44,转化成文件文件偏移就是 5B44,修改的地方如下图所示：  
![](img/530bb2ca01db4ebb9fcf61599cc629e6.jpg)  
  
哈哈，现在可以体现一下 PEDIY 的快乐!!! ^-^  
运行后效果如下图：  
![](img/90005936bdacf6fb4860598c70e14b10.jpg)