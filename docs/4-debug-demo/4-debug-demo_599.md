

# BuddyPhone v2.07

> 原文：[`www.pediy.com/kssd/pediy03/forum386.htm`](https://www.pediy.com/kssd/pediy03/forum386.htm)

 BuddyPhone v2.07 广告条去除   
  
      Write By liangs   
  
使用工具：Trw2000; W32Dasm; UltraEdit   
  
BuddyPhone 到了 2.07 版就没地方输入注册号码了，让人讨厌的广告条   
就真没办法去除了吗？其实也不是，昨天研究了一下，发现虽然不能   
输入注册号码，但要去除其广告条其实也不难。   
  
首先运行 Trw2000,用它将 BuddyPhone 载入   
下 bpx readfile，再 G   
程序会在调用 readfile 功能时中断   
BD \* 屏蔽掉所有中断   
下 PMODULE 回到程序中来   
按 F12 6 次，这时我们看到程序主框架已经出来了，当然，广告条也出来了，   
F10 返回程序上一层调用，这时，我们停在 00404709 处，下 bpx 00404709,   
重新运行程序，中断后 F8 跟入。   
  
  
\* Possible StringData Ref from Data Obj ->"key"   
                                  |   
:004077F7 68B8874700              push 004787B8   
:004077FC 8D4DC8                  lea ecx, dword ptr \[ebp\-38\]   
  
\* Possible StringData Ref from Data Obj ->"license"   
                                  |   
:004077FF 68C8874700              push 004787C8   
:00407804 51                      push ecx   
:00407805 8BC8                    mov ecx, eax   
:00407807 E8C4EC0400              call 004564D0   
:0040780C 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]   
:0040780F C645FC02                mov \[ebp\-04\], 02   
:00407813 E81E120400              call 00448A36   
:00407818 8D4DB4                  lea ecx, dword ptr \[ebp\-4C\]   
:0040781B E816120400              call 00448A36   
:00407820 6A2D                    push 0000002D   
:00407822 8D4DC8                  lea ecx, dword ptr \[ebp\-38\]   
:00407825 895DD8                  mov dword ptr \[ebp\-28\], ebx   
:00407828 E8CE110400              call 004489FB   
:0040782D 8BF8                    mov edi, eax   
:0040782F 83FFFF                  cmp edi, FFFFFFFF   
:00407832 0F84F4000000            je 0040792C   
一路按 F10，到达这里，这是我们碰到的第一个跳转，   
如果跳到 0040792C 处，广告条就不可避免的会出现。   
  
:00407838 8B55B0                  mov edx, dword ptr \[ebp\-50\]   
:0040783B 395AF8                  cmp dword ptr \[edx\-08\], ebx   
:0040783E 0F84E8000000            je 0040792C   
同上面 00407832 处一样，不能让它跳。   
  
:00407844 8B45B4                  mov eax, dword ptr \[ebp\-4C\]   
:00407847 3958F8                  cmp dword ptr \[eax\-08\], ebx   
:0040784A 0F84DC000000            je 0040792C   
又是一个同样地址的跳转，不能让它跳。   
  
......部分代码略   
  
:00407901 E800B20200              call 00432B06   
:00407906 83C408                  add esp, 00000008   
:00407909 3BF8                    cmp edi, eax   
:0040790B 7507                    jne 00407914   
此处同样不能跳，不然广告条就会出现，你试试看就知道了。   
  
:0040790D C745D801000000          mov \[ebp\-28\], 00000001   
走到这里知道了吧，上面忙活半天就是为了将\[ebp\-28\]置 1，作为已注册标志!!!   
而敌人就是想方设法要让你跳过这条语句！   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040790B(C)   
|   
:00407914 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]   
:00407917 C645FC03                mov \[ebp\-04\], 03   
:0040791B E8140C0400              call 00448534   
:00407920 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]   
:00407923 C645FC02                mov \[ebp\-04\], 02   
:00407927 E8080C0400              call 00448534   
  
现在该知道怎么改了吧   
(1) 地址 00407832  将 0F84F4000000  改为 909090909090    
(2) 地址 0040783E  将 0F84E8000000  改为 909090909090    
(3) 地址 0040784A  将 0F84DC000000  改为 909090909090    
(4) 地址 0040790B  将 7507          改为 9090   
  
重新运行程序，哈哈，广告条没有了。  

 下面的代码显然是判断注册码的：   
\* Possible StringData Ref from Data Obj ->"key"   
                                  |   
:004077F7 68B8874700              push 004787B8   
:004077FC 8D4DC8                  lea ecx, dword ptr \[ebp\-38\]   
  
\* Possible StringData Ref from Data Obj ->"license"   
                                  |   
:004077FF 68C8874700              push 004787C8   
:00407804 51                      push ecx   
:00407805 8BC8                    mov ecx, eax   
:00407807 E8C4EC0400              call 004564D0   
:0040780C 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]   
:0040780F C645FC02                mov \[ebp\-04\], 02   
:00407813 E81E120400              call 00448A36   
:00407818 8D4DB4                  lea ecx, dword ptr \[ebp\-4C\]   
:0040781B E816120400              call 00448A36   
:00407820 6A2D                    push 0000002D    //横杠字符"-"，注册码很可能分成几部分   
:00407822 8D4DC8                  lea ecx, dword ptr \[ebp\-38\]   
:00407825 895DD8                  mov dword ptr \[ebp\-28\], ebx   
:00407828 E8CE110400              call 004489FB    //判断，跟进去看看   
:0040782D 8BF8                    mov edi, eax   
:0040782F 83FFFF                  cmp edi, FFFFFFFF   
:00407832 0F84F4000000            je 0040792C   4dd466d0e1e0cc123afbe504da2462ca