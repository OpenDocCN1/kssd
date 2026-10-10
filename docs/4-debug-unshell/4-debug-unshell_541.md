

# Asprotect SKE 2.2 的 Advanced Import protection 保护技术

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-605.htm`](https://www.pediy.com/kssd/pediy08/pediy8-605.htm)

【文章标题】: Asprotect SKE 2.2 的 Advanced Import protection 保护技术浅析  
【文章作者】: kanxue  
【作者主页】: http://www.pediy.com  
【软件名称】: Asprotect SKE 2.2 build 03.19 Release 加壳的目标软件，下载：md5calculator.rar  
\--------------------------------------------------------------------------------  
  首先感谢 shoooo 的帮助！春节前那段时间开始学习 Asprotect SKE 2.x 脱壳技术，由于长时间没接触 Asprotect，感觉很是吃力，多亏 shoooo 无私地将他的许多经验技巧相授，这样才得己掌握这款猛壳的一些脱壳技术。  
  本篇文章以 Asprotect SKE 2.2 build 03.19 Release 加壳的目标为讲解实例（一个 Microsoft Visual C++ 6.0 编译的程序），压缩选项只选了一项“Advanced Import protection”，Asprotect 保护的输入表情况多种，本文仅与大家探讨 Advanced Import protection 部分，其它更完整的输入表处理类型参考 shoooo 撰写的 Asprotect SKE 经典文章：[nspack3.5 主程序脱壳分析(Aspr SKE 2.X)  ](http://bbs.pediy.com/showthread.php?s=&threadid=19313)，本文只是将 shoooo 这篇的文章提到的东西转换成实际操作过程而己 。  
  
脱 Asprotect SKE 2.x 壳会遇到如下情况：  
1. Emulate standard system functions（易）  
2. Advanced Import protection（中）  
3. stolen OEP （脱 Asprotect 难点就在这，体力＋毅力＋技巧＋时间）  
4. SDK       （脱 Asprotect 难点就在这，体力＋毅力＋技巧＋时间）  
5. 初始化表与库函数（Delphi 程序）（易）  
  
**1.什么是 Advanced Import protection**   
  
先来看一段加壳的代码，请注意 API 函数的调用部分：  
00401EC5   FF7424 08             push    dword ptr \[esp+8\]                  
00401EC9   FF15 3C504000         call    \[<&KERNEL32.GetCurrentProcess>\]  //这里，即 call    \[40503C\]  
00401ECF   50                    push    eax                               
00401ED0   FF15 38504000         call    \[<&KERNEL32.TerminateProcess>\]   //这里，即 call    \[405038\]  
00401ED6   837C24 0C 00          cmp     dword ptr \[esp+C\], 0  
  
用 Advanced Import protection 保护后的代码，其 API 调用转换成了 call 01200000 形式：  
00401EC5   FF7424 08             push    dword ptr \[esp+8\]  
00401EC9   E8 32E1DF00           call    01200000              //这里，调用 KERNEL32.GetCurrentProcess  
00401ECE   AA                    stos    byte ptr es:\[edi\]  
00401ECF   50                    push    eax  
00401ED0   E8 2BE1DF00           call    01200000              //这里，调用 KERNEL32.TerminateProcess  
00401ED5   CA                    db      CA  
  
  
以下这段文字来自 shoooo 的文章（稍有改动）：  

引用:  *最初由** shoooo **发布*  
出处：http://bbs.pediy.com/showthread.php?s=&threadid=19313  
  
01200000 什么呢？  
它是把导入函数调用的变形，原来的 call \[IAT\] 和 jmp \[IAT\]的变形  
01200000 壳用 VirtualAlloc 的分配的空间，不在区段中  
在我的机子上现在是 call 01200000，在你的机子上就可能是 call 1230000  
也就是说，call 01200000 壳经过计算后写入的  
于是我想看看，在它写入 call  01200000 前是什么样子  
  
OD 载入实例程序，清除所有断点，  
对 0401ECA 下内存写入断点 (因为 00401EC9 是'E8'，我们要的是后 4 个字节)  
  
若干次后会断在这里  
  
00AA6EBF    8945 00             mov     \[ebp\], eax     // 断在这儿：ebp 指向 00401ECA，eax 写入后，使那个地方变成 call    01200000   
00AA6EC2    6A 0A               push    0A  
00AA6EC4    E8 D3E7FDFF         call    00A8569C  
00AA6EC9    8BC8                mov     ecx, eax  
00AA6ECB    038B E4000000       add     ecx, \[ebx+E4\]  
00AA6ED1    8BD6                mov     edx, esi  
00AA6ED3    8BC3                mov     eax, ebx  
00AA6ED5    E8 AAE6FFFF         call    00AA5584  
00AA6EDA    FF0C24              dec     dword ptr \[esp\]  
00AA6EDD    03B3 E4000000       add     esi, \[ebx+E4\]  
00AA6EE3    833C24 00           cmp     dword ptr \[esp\], 0  
00AA6EE7  ^ 0F87 55FEFFFF       ja      00AA6D42         //如果还有需要处理就跳上去  
00AA6EED    53                  push    ebx  
00AA6EEE    E8 5D000000         call    00AA6F50  
00AA6EF3    0183 EC000000       add     \[ebx+EC\], eax  
00AA6EF9    B0 01               mov     al, 1  
00AA6EFB    83C4 24             add     esp, 24  
00AA6EFE    5D                  pop     ebp  
00AA6EFF    5F                  pop     edi  
00AA6F00    5E                  pop     esi  
00AA6F01    5B                  pop     ebx  
00AA6F02    C3                  retn                     //这里结束  
  
  
正如我所说，call 01200000 完全是在代码段解码后，申请空间，现在我申请到的是 01200000  
那么它就将需要变形的地方计算后写成 call 01200000，如果你申请到的是 1230000，那么你  
是 call 1230000  
  
断在这里，我当然想看一看在改写成 call 01200000 之前，那些地址是不是正常的  
很可惜，那里在改写成 call 01200000，本身就是乱掉的。  
或者在某个时候能知道那些变形地址原先的真实情况，可惜我找不到。  
也许只有作者知道在哪里  
也许根本就找不到  
因为根本就不需要  
对于 call 01200000，它加密前只要知道 2 件事，1.本身所在的地址 2.IAT 中的位置  
对于 call 01200000，现在也只要知道 2 件事，1.本身所在的地址 2.最后要去的导入函数的地址  
它没有理由记录 IAT 中的位置  
我们要做的是找出最后到达的导入函数的地址，然后找出它在 IAT 中的位置  
改成原先的 call \[IAT\] 或 jmp \[IAT\]  
  
回到正题，当我们断下时，前面可能已经处理若干个了  
要想得到全部的表  
你有好几种选择  
1. 到 oep 后，写一段代码搜索出所有的 call 01200000 的地址  
2. 想办法第一时间断在上面这个地方，即 00AA6EBF，ebp-1 就是变形的地址，保存所有的 ebp-1  
3. 也许内存中本身存在这张表，我没有去找，你可以找找  
 

  
  
00AA6EBF    8945 00             mov     \[ebp\], eax  
00AA6EC2    6A 0A               push    0A  
00AA6EC4    E8 D3E7FDFF         call    00A8569C  
00AA6EC9    8BC8                mov     ecx, eax  
00AA6ECB    038B E4000000       add     ecx, \[ebx+E4\]  
00AA6ED1    8BD6                mov     edx, esi  
00AA6ED3    8BC3                mov     eax, ebx  
00AA6ED5    E8 AAE6FFFF         call    00AA5584  
00AA6EDA    FF0C24              dec     dword ptr \[esp\]  
00AA6EDD    03B3 E4000000       add     esi, \[ebx+E4\]  
00AA6EE3    833C24 00           cmp     dword ptr \[esp\], 0  
00AA6EE7  ^ 0F87 55FEFFFF       ja      00AA6D42 //循环上去  
  
在写入每一处的 call 01200000 时，上面的代码会经过这里：  
  
00AA6D42    33C0                xor     eax, eax  
00AA6D44    8A07                mov     al, \[edi\]  
00AA6D46    8D0440              lea     eax, \[eax+eax\*2\]  
00AA6D49    8B6C83 68           mov     ebp, \[ebx+eax\*4+68\]  
00AA6D4D    8BC6                mov     eax, esi  
00AA6D4F    FFD5                call    ebp  
00AA6D51    8BE8                mov     ebp, eax  
00AA6D53    036B 24             add     ebp, \[ebx+24\]  
00AA6D56    03AB E0000000       add     ebp, \[ebx+E0\]  
00AA6D5C    EB 01               jmp     short 00AA6D5F  
00AA6D5E    E8 33C08A47         call    48352D96  
00AA6D63    098D 04408B54       or      \[ebp+548B4004\], ecx  
00AA6D69    8368 8B C6          sub     dword ptr \[eax-75\], -3A  
**00AA6D6D    FFD2                call    edx  **                       //call edx 结果在 eax  
00AA6D6F    807B 20 00          cmp     byte ptr \[ebx+20\], 0       //eax 可能是 1 或 0  
00AA6D73    0F85 3D010000       jnz     00AA6EB6  

引用:  *最初由** shoooo **发布*  
如果是 1，当前这个 call 01200000 处运行时，会重新回到调用地址，再进入导入函数  
如果是 0，当前这个 call 01200000 进入导入函数后出来，不过这种方式比较邪恶，它可能做更多的事情  

  
  
  
  
**2.获得地址表**  
  
  这里想办法获得程序里“ call    01200000 ”语句所有的地址，上面代码中“00AA6D6D   call    edx ”这个点比较好，除了能得到地址表，还能得知晓 call 01200000 是按类型 1 来处理的，带是按类型 2 来处理的。  
  对 00AA6D6D   call    edx 下硬件执行断点，重新用 OD 加载实例程序，由于己下硬件断点，因此这次直接断在这里：  
  
00AA6D6D    FFD2                call    edx                  //断在这里，此次 EBP 指向的就要处理的地址  
00AA6D6F    807B 20 00          cmp     byte ptr \[ebx+20\], 0  
00AA6D73    0F85 3D010000       jnz     00AA6EB6  
00AA6D79    3C 01               cmp     al, 1  
此时堆栈窗口的数据：  
0012FEF8   0000001A   //注意这里的数据 0x1A，表示 26 个地址要处理  
0012FEFC   D6589EF1  
0012FF00   9BCE5199  
0012FF04   00000000  
  
为了得到所有的地址表，必须写一小段补丁代码。首先用 HideOD 插件分配一个临时空间（Alloc Memory 菜单命令），我这里分配得到的地址是 01240000，在你的机子可能不是这值。  
01240000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
01240010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
01240020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
01240030  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
01240040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
然后修改 00AA6D6F 这段代码如下：  
  
  
00AA6D6D  - E9 8E927900         jmp     01240000    //跳到 patch 代码执行  
00AA6D72    90                  nop  
00AA6D73     0F85 3D010000       jnz     00AA6EB6  
00AA6D79    3C 01               cmp     al, 1  
  
在分配的空间里键入如下补丁代码：  
  
01240000    FFD2                call    edx                //壳原来的代码  
01240002    60                  pushad                     //开始补丁代码  
01240003    8B1D 30002401       mov     ebx, \[1240030\]     //取缓存地址，我这里是用 01240040 这个地址  
01240009    C1E0 1F             shl     eax, 1F            //将 EAX 的返回值 0 或 1 置高位  
0124000C    03C5                add     eax, ebp           //ebp 是所要的地址  
0124000E    8903                mov     \[ebx\], eax  
01240010    83C3 04             add     ebx, 4             //指向下一个缓存地址  
01240013    891D 30002401       mov     \[1240030\], ebx  
01240019    61                  popad  
0124001A    807B 20 00          cmp     byte ptr \[ebx+20\], 0//壳原来的代码  
0124001E  - E9 506D86FF         jmp     00AA6D73             //跳到壳里继续执行  
  
对应的二进制代码如下：  
FF D2 60 8B 1D 30 00 24 01 C1 E0 1F 03 C5 89 03 83 C3 04 89 1D 30 00 24 01 61 80 7B 20 00 E9 50  
6D 86 FF  
  
键入补丁后的数据窗口如下：  
01240000  FF D2 60 8B 1D 30 00 24 01 C1 E0 1F 03 C5 89 03  �襚?0.$拎艍  
01240010  83 C3 04 89 1D 30 00 24 01 61 80 7B 20 00 E9 50  兠?0.$a€{ .镻  
01240020  6D 86 FF 00 00 00 00 00 00 00 00 00 00 00 00 00  m?.............  
01240030  40 00 24 01  00 00 00 00 00 00 00 00 00 00 00 00  @.$............  
01240040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
01240050  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
注意：上面的红字 40 00 24 01  也是需要手工键入的，这个是一地址：01240040 ，取得的地址就会放到这个地址开始的内存中。  
  
补丁代码键好后，取消硬件断点，然后一步步走过这段补丁代码，来到：  
  
00AA6EC9    8BC8                mov     ecx, eax  
00AA6ECB    038B E4000000       add     ecx, \[ebx+E4\]  
00AA6ED1    8BD6                mov     edx, esi  
00AA6ED3    8BC3                mov     eax, ebx  
00AA6ED5    E8 AAE6FFFF         call    00AA5584  
00AA6EDA    FF0C24              dec     dword ptr \[esp\]  
00AA6EDD    03B3 E4000000       add     esi, \[ebx+E4\]  
00AA6EE3    833C24 00           cmp     dword ptr \[esp\], 0  
00AA6EE7  ^ 0F87 55FEFFFF       ja      00AA6D42  
00AA6EED    53                  push    ebx                     //在这按 F4  
00AA6EEE    E8 5D000000         call    00AA6F50  
00AA6EF3    0183 EC000000       add     \[ebx+EC\], eax  
00AA6EF9    B0 01               mov     al, 1  
00AA6EFB    83C4 24             add     esp, 24  
00AA6EFE    5D                  pop     ebp  
00AA6EFF    5F                  pop     edi  
00AA6F00    5E                  pop     esi  
00AA6F01    5B                  pop     ebx  
00AA6F02    C3                  retn  
  
  
查看数据窗口，此时得到所有的地址：  
  
01240040  15 10 40 00 AF 10 40 00 D4 10 40 00 9F 1D 40 00  @.?@.?@.?@.  
01240050  CA 1D 40 00 ED 1D 40 00 C9 1E 40 00 D0 1E 40 00  ?@.?@.?@.?@.  
01240060  24 22 40 00 6D 25 40 00 D9 25 40 00 7F 26 40 00  $"@.m%@.?@.&@.  
01240070  D8 26 40 00 E6 26 40 00 1D 27 40 00 DE 29 40 00  ?@.?@.'@.?@.  
01240080  B4 2A 40 00 BB 2A 40 00 4E 2B 40 00 B9 2C 40 00  ?@.?@.N+@.?@.  
01240090  CE 2C 40 00 59 2D 40 00 FC 3D 40 00 45 3E 40 00  ?@.Y-@.?@.E>@.  
012400A0  46 40 40 00 7A 40 40 00 00 00 00 00 00 00 00 00  F@@.z@@.........  
  
后面的补丁代码需要这些数据，因此在这用 OD 的二进制复制功能，将这些数据复制出来：  
15 10 40 00 AF 10 40 00 D4 10 40 00 9F 1D 40 00 CA 1D 40 00 ED 1D 40 00 C9 1E 40 00 D0 1E 40 00  
24 22 40 00 6D 25 40 00 D9 25 40 00 7F 26 40 00 D8 26 40 00 E6 26 40 00 1D 27 40 00 DE 29 40 00  
B4 2A 40 00 BB 2A 40 00 4E 2B 40 00 B9 2C 40 00 CE 2C 40 00 59 2D 40 00 FC 3D 40 00 45 3E 40 00  
46 40 40 00 7A 40 40 00 00 00 00 00 00 00 00 00  
  
在这随便挑选一组数据来验证一下，如 15 10 40 00 ，反过来就是：401015，OD 跳到 401015 地址处：  
00401015   E8 E6EFDF00       call    01200000  
  
  
**3.分析 call 01200000**  
  

引用:  *最初由** shoooo **发布*  
有没有想过一个有意思的问题，所有这样的调用都是进入 1200000 一个地方，但是壳却知道最后  
目的地址是哪一个导入函数，它是怎么判断的呢？  
当到了 1200000，壳能看到什么？  
1. 参数  
2. 返回地址  
第 1 种情况：鬼知道我会传什么参数，多少个参数，它不能作为评判标准的  
第 2 种情况：只有你了，Aspr 存着一张表，它记录了所有 call 01200000 的返回地址和最后导入函数的 1 对 1 关系  
  
它是加密的  
我们要做的是找出这张表，或者找到 1 个点能确定它们 1 对 1 的关系  
  
简单说一下进入 1200000 后发生了什么，一共三层  
  
第一层：保存所有当前寄存器 (出来后还要继续运行的，不能影响后面，不过它不是明目张胆的 pushad)  
第二层：1. 决定是哪一种方式的导入函数调用  
           a. 第一种方式：将 call 01200000 变成 call F00004 之类，出来后再次从原地进入 F00004 进入导入函数  
           b. 第二种方式：直接带着参数进入导入函数   
        2. 决定这个调用是 call (ff15)还是 jmp (ff25)  
           不要以为 C 的都是 call，delphi 的都是 jmp  
           c. 如果是 call (ff15)，返回地址要+1 ，比如 inc \[esp\]，因为 call 00EA0000 占 5 个字节，call (ff15)占 6 个字节  
           d. 如果是 jmp (ff25)，要 esp+4，想一下就知道原因了:)  
        3. 如果是 1.b 的情况，可能有更邪恶的对下一行的偷代码（后面会讲述）  
第三层：恢复所有的寄存器返回  
 

  
  
下面来实际跟踪一下，看看情况如何：  
00401015  |.  E8 E6EFDF00       call    01200000   //此处新建 EIP  
  
对于第一层的操作，只要一路 F7 即可，这段代码相当于 pushad。  
  
01200157    03CA            add     ecx, edx  
01200159    59              pop     ecx  
0120015A    FFD1            call    ecx          //F7 进入第二层  
0120015C    6A 2C           push    2C  
0120015E    B9 BA764000     mov     ecx, 4076BA  
01200163    B9 32524200     mov     ecx, 425232  
01200168    59              pop     ecx  
01200169    EB 02           jmp     short 0120016D  
0120016B    CD20 9C75AE00   vxdcall AE759C  
01200171    C3              retn  
  
上面这段代码可能每次跟踪都会不同，也可能是 call eax 等进入第二层。  
到了第二层，可以一路 F8:  
00AA67B8    55              push    ebp  
00AA67B9    8BEC            mov     ebp, esp  
00AA67BB    83C4 D8         add     esp, -28  
00AA67BE    53              push    ebx  
00AA67BF    56              push    esi  
00AA67C0    57              push    edi  
00AA67C1    33C0            xor     eax, eax  
00AA67C3    8945 DC         mov     dword ptr \[ebp-24\], eax  
00AA67C6    8945 D8         mov     dword ptr \[ebp-28\], eax  
00AA67C9    8945 E0         mov     dword ptr \[ebp-20\], eax  
00AA67CC    8B5D 08         mov     ebx, dword ptr \[ebp+8\]  
00AA67CF    33C0            xor     eax, eax  
……  
00AA68B7    FFD6            call    esi  
00AA68B9    8BF0            mov     esi, eax  
00AA68BB    3B75 F8         cmp     esi, dword ptr \[ebp-8\]  
00AA68BE    75 63           jnz     short 00AA6923  
00AA68C0    807B 20 00      cmp     byte ptr \[ebx+20\], 0            //如果\[ebx+20\]是 0，b 情况  
00AA68C4    74 3C           je      short 00AA6902                  //如果\[ebx+20\]是 1，a 情况  
00AA68C6    8B45 E8         mov     eax, dword ptr \[ebp-18\]  
00AA68C9    0FB640 09       movzx   eax, byte ptr \[eax+9\]  
00AA68CD    8D0440          lea     eax, dword ptr \[eax+eax\*2\]  
00AA68D0    8B5483 68       mov     edx, dword ptr \[ebx+eax\*4+68\]  
00AA68D4    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
00AA68D7    FFD2            call    edx                            //再次比较是哪一情况  
00AA68D9    3C 01           cmp     al, 1                          //eax 为 1 则是 a 情况，为 0 则是 b 情况  
00AA68DB    75 25           jnz     short 00AA6902                 //本文提供的实例只有 b 情况，因此此处会跳转  
00AA68DD    56              push    esi  
00AA68DE    8D45 FC         lea     eax, dword ptr \[ebp-4\]  
00AA68E1    50              push    eax  
00AA68E2    8B45 14         mov     eax, dword ptr \[ebp+14\]  
00AA68E5    50              push    eax  
00AA68E6    8B45 18         mov     eax, dword ptr \[ebp+18\]  
00AA68E9    50              push    eax  
00AA68EA    8B45 0C         mov     eax, dword ptr \[ebp+C\]  
00AA68ED    50              push    eax  
00AA68EE    8B45 F0         mov     eax, dword ptr \[ebp-10\]  
00AA68F1    50              push    eax  
00AA68F2    8B4D 1C         mov     ecx, dword ptr \[ebp+1C\]  
00AA68F5    8B55 10         mov     edx, dword ptr \[ebp+10\]  
00AA68F8    8BC3            mov     eax, ebx  
00AA68FA    E8 BD010000     call    00AA6ABC                        //a 情况这里 F7 进去  
00AA68FF    EB 01           jmp     short 00AA6902  
00AA6901    E8 8D45FC50     call    51A6AE93  
00AA6906    8B45 14         mov     eax, dword ptr \[ebp+14\]  
00AA6909    50              push    eax  
00AA690A    8B45 18         mov     eax, dword ptr \[ebp+18\]  
00AA690D    50              push    eax  
00AA690E    8B45 0C         mov     eax, dword ptr \[ebp+C\]  
00AA6911    50              push    eax  
00AA6912    8B45 F0         mov     eax, dword ptr \[ebp-10\]  
00AA6915    50              push    eax  
00AA6916    8B4D 1C         mov     ecx, dword ptr \[ebp+1C\]  
00AA6919    8B55 10         mov     edx, dword ptr \[ebp+10\]  
00AA691C    8BC3            mov     eax, ebx  
00AA691E    E8 95F2FFFF     call    00AA5BB8                         //b 情况这里 F7 进去  
  
  
**a 情况：**  
  
  
先看 a 情况吧，进去后一路 F8:  
  
017F258C    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
017F258F    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]  
017F2595    0345 E4         add     eax, dword ptr \[ebp-1C\]  
017F2598    8945 FC         mov     dword ptr \[ebp-4\], eax  //到了这里 eax 的值就是导函数的地址了，再往下 F8  
017F259B    57              push    edi  
017F259C    6A 00           push    0  
017F259E    8D4D E0         lea     ecx, dword ptr \[ebp-20\]  
017F25A1    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
017F25A4    8B40 3C         mov     eax, dword ptr \[eax+3C\]  
017F25A7    8B55 FC         mov     edx, dword ptr \[ebp-4\]  
017F25AA    E8 81130000     call    017F3930  
017F25AF    8945 FC         mov     dword ptr \[ebp-4\], eax  
017F25B2    8B45 E0         mov     eax, dword ptr \[ebp-20\]  
017F25B5    8B00            mov     eax, dword ptr \[eax\]  
017F25B7    E8 E8E7FFFF     call    017F0DA4  
017F25BC    8BD0            mov     edx, eax  
017F25BE    02D3            add     dl, bl  
017F25C0    8B4D FC         mov     ecx, dword ptr \[ebp-4\]   //这个点比较好  
  

引用:  *最初由** shoooo **发布*  
到了这里 \[ebp-4C\]是我们需要的导入函数的地址，dl 中的值决定了是 call(ff15)还是 jmp(ff25)  
dl 中的值不同的程序是随机，找几个 call 01200000 进去出来看一下就知道当前的程序中哪个对应 ff15，哪个对应 ff25 了  
 

  
  
**b 情况：**  
  
  
00AA5BB8    55              push    ebp  
00AA5BB9    8BEC            mov     ebp, esp  
00AA5BBB    83C4 CC         add     esp, -34  
00AA5BBE    53              push    ebx  
00AA5BBF    56              push    esi  
00AA5BC0    57              push    edi  
00AA5BC1    894D F8         mov     dword ptr \[ebp-8\], ecx  
……  
00AA5CFF    FFD7            call    edi  
00AA5D01    84C0            test    al, al  
00AA5D03    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
00AA5D06    8A40 4A         mov     al, byte ptr \[eax+4A\]  
00AA5D09    3A45 EF         cmp     al, byte ptr \[ebp-11\]            ; al 的值决定是 FF15 还是 FF25  
00AA5D0C    0F85 9C000000   jnz     00AA5DAE                         ; 不跳是 FF15 情况  
  
al 中的值，不同程序是不同的。本节实例：当 al=4c 时，对应的是 FF15  
  
FF15 分支来到：  
00AA5D15    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
00AA5D18    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]  
00AA5D1E    0145 FC         add     dword ptr \[ebp-4\], eax           ;\[ebp-4\]是解密出来的 API  
00AA5D21   /EB 01           jmp     short 00AA5D24  
  
  
FF25 分支来到：  
  
00AA5DC7    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
00AA5DCA    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]  
00AA5DD0    0145 FC         add     dword ptr \[ebp-4\], eax           ; \[ebp-4\] 是我们需要的导入函数的地址  
00AA5DD3    8D45 0C         lea     eax, dword ptr \[ebp+C\]  
  

引用:  *最初由** shoooo **发布*  
00AA5D1E 或 00AA5DD0 执行完后\[ebp-4\] 是需要的输入函数的地址  
再看看\[ebp-2c\]，如果它是 FFFFFFFF，说明这个导入函数调用是干净的如果它有值，表示它的下一行也偷了。  

  
  
  
**4.修复 call 01200000**  
  
经过上面分析，现在可以写修复代码了，每个被加密的函数都从 call 01200000 进入，在计算过程中某个点，其调用函数名会显示出来，因此我们在这个出口设断取得函数地址。  
  
FF15 分支来到：  
00AA5D15    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
00AA5D18    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]  
00AA5D1E    0145 FC         add     dword ptr \[ebp-4\], eax           ;\[ebp-4\]是解密出来的 API  
**00AA5D21   /EB 01           jmp     short 00AA5D24 **                    //在这下硬件断点  
  
  
FF25 分支来到：  
  
00AA5DC7    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
00AA5DCA    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]  
00AA5DD0    0145 FC         add     dword ptr \[ebp-4\], eax           ; \[ebp-4\] 是我们需要的导入函数的地址  
**00AA5DD3    8D45 0C         lea     eax, dword ptr \[ebp+C\] **            //在这下硬件断点  
  
  
  
再来看看实例的 IAT：  
00405000  CF C6 80 7C 78 2C 81 7C 80 A4 80 7C B9 8C 83 7C  掀€|x,亅€|箤億  
00405010  C4 CE 80 7C 2B 2E 83 7C AD 9C 80 7C 77 1D 80 7C  奈€|+.億瓬€|w€|  
00405020  28 AC 80 7C 29 B5 80 7C EE 1E 80 7C 8D 2C 81 7C  (瑎|)祤|?€|?亅  
00405030  AB 14 81 7C A2 CA 81 7C 16 1E 80 7C 0D E0 80 7C  ?亅⑹亅€|.鄝|  
00405040  8A 2B 86 7C 57 B3 80 7C 3F DC 81 7C 5F 48 81 7C  ?唡 W 硛|?軄|\_H 亅  
00405050  C7 A0 80 7C 23 CC 81 7C FD 79 93 7C A9 2C 81 7C  莀€|#虂|齳搢?亅  
00405060  69 10 81 7C 10 11 81 7C 29 29 81 7C 14 9B 80 7C  i 亅亅))亅泙|  
00405070  3D 04 93 7C 40 7A 95 7C 9F 0F 81 7C E6 2B 81 7C  =搢@z 晐?亅?亅  
00405080  43 99 80 7C 2A E8 81 7C D4 05 93 7C 81 9A 80 7C  C 檧|\*鑱|?搢仛€|  
00405090  00 00 00 00 1E AC D6 77 4A C9 D3 77 AD A8 D1 77  ....wJ 捎 w褀  
004050A0  11 12 D2 77 24 13 D2 77 9A F3 D2 77 85 CB D1 77  襴$襴汅襴吽褀  
004050B0  50 62 D2 77 1C B1 D3 77 00 00 00 00 00 00 00 00  Pb 襴庇 w........  
  
从上面的 IAT 可以看出，IAT 地址在 405000～4050B8 之间,并且这些地址没加密。   
还记得第二部分获得的地址表吧，现在可以用上了，写个补丁程序，扫描这个地址表，依次从表里的每个地址进入，然后在 00AA5D21 或 00AA5DD3 两个出口处获得函数地址，扫描 IAT 表，将找到的 IAT 地址写入程序代码中。  
  
  
用 HideOD 插件分配一个临时空间（Alloc Memory 菜单命令）:  
00C50000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00C50010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00C50020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
将如下补丁代码放上(由于水平有限，代码写的比较乱，但将 a、b 两种情况都考虑了，因此比较通用）：  
  
00C50000    A1 C000C500        mov     eax, dword ptr \[C500C0\]                 ;  指向待处理的地址列表  
00C50005    8B18               mov     ebx, dword ptr \[eax\]  
00C50007    81E3 FFFFFF7F      and     ebx, 7FFFFFFF  
00C5000D    FFE3               jmp     ebx  
00C5000F    0000               add     byte ptr \[eax\], al  
00C50011    0000               add     byte ptr \[eax\], al  
00C50013    0000               add     byte ptr \[eax\], al  
00C50015    0000               add     byte ptr \[eax\], al  
00C50017    0000               add     byte ptr \[eax\], al  
00C50019    0000               add     byte ptr \[eax\], al  
00C5001B    0000               add     byte ptr \[eax\], al  
00C5001D    0000               add     byte ptr \[eax\], al  
00C5001F    0000               add     byte ptr \[eax\], al                      ; 0c500c0 得手工填上，0c500c0＋4 处也得填上（存放那些下一句被抽的 API 的 CALL）  
00C50021    BF C000C500        mov     edi, 0C500C0                            ; EDI 存放的是指向需要处理的 API 地址表  
00C50026    8B07               mov     eax, dword ptr \[edi\]  
00C50028    8B18               mov     ebx, dword ptr \[eax\]  
00C5002A    81FB FFFFFF7F      cmp     ebx, 7FFFFFFF  
00C50030    79 49              jns     short 00C5007B                          ; 高位是 1 的走这条路线（跳）,即处理 a 方式  
00C50032    837D D4 FF         cmp     dword ptr \[ebp-2C\], -1                  ; 如是 FFFFFFFF，说明这个导入函数调用是干净的  
00C50036    74 0F              je      short 00C50047  
00C50038    8B47 04            mov     eax, dword ptr \[edi+4\]                  ; EDI＋4 处的值得手工填上，我填的是 c50160（存放那些下一句被抽的 API 的 CALL）  
00C5003B    8B1F               mov     ebx, dword ptr \[edi\]                    ; 指向待处理的 API 列表  
00C5003D    8B1B               mov     ebx, dword ptr \[ebx\]  
00C5003F    8918               mov     dword ptr \[eax\], ebx  
00C50041    83C0 04            add     eax, 4  
00C50044    8947 04            mov     dword ptr \[edi+4\], eax  
00C50047    8B5D FC            mov     ebx, dword ptr \[ebp-4\]                  ; ebp-4 是获得的 API 函数地址  
00C5004A    E8 46000000        call    00C50095                                ; 在 IAT 里搜索,正确的 IAT 项从 ESI 中返回  
00C5004F    B0 4C              mov     al, 4C                                  ; 决定是 ff15 还是 ff25,不同软件 AL 的值不同，本实例是 4c  
00C50051    66:B9 FF15         mov     cx, 15FF  
00C50055    3A45 EF            cmp     al, byte ptr \[ebp-11\]  
00C50058    74 05              je      short 00C5005F  
00C5005A    66:81C1 0010       add     cx, 1000  
00C5005F    8B07               mov     eax, dword ptr \[edi\]                    ; 指向待处理的 API 列表  
00C50061    8B18               mov     ebx, dword ptr \[eax\]  
00C50063    81E3 FFFFFF7F      and     ebx, 7FFFFFFF                           ; 将高 1 位清除  
00C50069    83C0 04            add     eax, 4                                  ; 指向下一个地址  
00C5006C    8907               mov     dword ptr \[edi\], eax  
00C5006E    66:890B            mov     word ptr \[ebx\], cx  
00C50071    83C3 02            add     ebx, 2  
00C50074    8933               mov     dword ptr \[ebx\], esi  
00C50076  ^ EB 88              jmp     short 00C50000  
00C50078    90                 nop  
00C50079    90                 nop  
00C5007A    90                 nop                                             ; 下面是处理 a 情况（本实例无这情况，但为了大家以后方便，故放出）  
00C5007B    8B5D B4            mov     ebx, dword ptr \[ebp-4C\]                 ; ebp-4c 是获得的 API 函数地址  
00C5007E    E8 12000000        call    00C50095                                ; 在 IAT 里搜索,正确的 IAT 项从 ESI 中返回  
00C50083    B0 4C              mov     al, 4C                                  ; 这个 4c 的值在不同的程序是随机  
00C50085    66:B9 FF15         mov     cx, 15FF  
00C50089    3AC2               cmp     al, dl                                  ; dl 中的值决定了是 call(ff15)还是 jmp(ff25)  
00C5008B  ^ 74 D2              je      short 00C5005F  
00C5008D  ^ EB CB              jmp     short 00C5005A  
00C5008F    0000               add     byte ptr \[eax\], al  
00C50091    0000               add     byte ptr \[eax\], al  
00C50093    0000               add     byte ptr \[eax\], al                      ; 这个 CALL 搜索 IAT 中的指定项  
00C50095    BE 00504000        mov     esi, 405000                             ; IAT 起始地址  
00C5009A    391E               cmp     dword ptr \[esi\], ebx  
00C5009C    74 0D              je      short 00C500AB  
00C5009E    83C6 04            add     esi, 4  
00C500A1    81FE B8504000      cmp     esi, 4050B8                             ; IAT 结束地址，判断是否越界  
00C500A7    77 03              ja      short 00C500AC  
00C500A9  ^ EB EF              jmp     short 00C5009A                          ; 如果在 IAT 里没找到就死循环,所以，如果死循环，你按 F12 暂停程序  
00C500AB    C3                 retn  
00C500AC  - EB FE              jmp     short 00C500AC  
  
  
  
对应的二进制代码：  
A1 C0 00 C5 00 8B 18 81 E3 FF FF FF 7F FF E3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
00 BF C0 00 C5 00 8B 07 8B 18 81 FB FF FF FF 7F 79 49 83 7D D4 FF 74 0F 8B 47 04 8B 1F 8B 1B 89  
18 83 C0 04 89 47 04 8B 5D FC E8 46 00 00 00 B0 4C 66 B9 FF 15 3A 45 EF 74 05 66 81 C1 00 10 8B  
07 8B 18 81 E3 FF FF FF 7F 83 C0 04 89 07 66 89 0B 83 C3 02 89 33 EB 88 90 90 90 8B 5D B4 E8 12  
00 00 00 B0 4C 66 B9 FF 15 3A C2 74 D2 EB CB 00 00 00 00 00 00 BE 00 50 40 00 39 1E 74 0D 83 C6  
04 81 FE B8 50 40 00 77 03 EB EF C3 EB FE  
  
  
  
  
数据窗口中的数据：  
00C50000  A1 C0 00 C5 00 8B 18 81 E3 FF FF FF 7F FF E3 00  ±.??併����?  
00C50010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00C50020  00 BF C0 00 C5 00 8B 07 8B 18 81 FB FF FF FF 7F  .坷.???侞���  
00C50030  79 49 83 7D D4 FF 74 0F 8B 47 04 8B 1F 8B 1B 89  yI 儅?t 婫??  
00C50040  18 83 C0 04 89 47 04 8B 5D FC E8 46 00 00 00 B0  兝塆媇F...  
00C50050  4C 66 B9 FF 15 3A 45 EF 74 05 66 81 C1 00 10 8B  Lf?:E 飔 f 伭.  
00C50060  07 8B 18 81 E3 FF FF FF 7F 83 C0 04 89 07 66 89  ?併���兝?f  
00C50070  0B 83 C3 02 89 33 EB 88 90 90 90 8B 5D B4 E8 12   兠?雸悙悑\]磋  
00C50080  00 00 00 B0 4C 66 B9 FF 15 3A C2 74 D2 EB CB 00  ...癓 f?:聇译?  
00C50090  00 00 00 00 00 BE 00 50 40 00 39 1E 74 0D 83 C6  .....?P@.9t.兤  
00C500A0  04 81 FE B8 50 40 00 77 03 EB EF C3 EB FE 00 00  侢窹@.w 腼秒?.  
00C500A0  04 81 FE B8 50 40 00 77 03 EB EF C3 EB FE 00 00  侢窹@.w 腼秒?.    //上面是补丁代码  
  
00C500B0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00C500C0  D0 00 C5 00 60 01 C5 00 00 00 00 00 00 00 00 00  ??\`?........     //这里有 2 个指针：C500D0，指向地址表；c51060 用来存放下行被偷的 API  
  
00C500D0  15 10 40 00 AF 10 40 00 D4 10 40 00 9F 1D 40 00  @.?@.?@.?@.     //第二部分获得的地址表  
00C500E0  CA 1D 40 00 ED 1D 40 00 C9 1E 40 00 D0 1E 40 00  ?@.?@.?@.?@.  
00C500F0  24 22 40 00 6D 25 40 00 D9 25 40 00 7F 26 40 00  $"@.m%@.?@.&@.  
00C50100  D8 26 40 00 E6 26 40 00 1D 27 40 00 DE 29 40 00  ?@.?@.'@.?@.  
00C50110  B4 2A 40 00 BB 2A 40 00 4E 2B 40 00 B9 2C 40 00  ?@.?@.N+@.?@.  
00C50120  CE 2C 40 00 59 2D 40 00 FC 3D 40 00 45 3E 40 00  ?@.Y-@.?@.E>@.  
00C50130  46 40 40 00 7A 40 40 00 00 00 00 00 00 00 00 00  F@@.z@@.........  
  
  
代码全部键入后，可以先单步跟踪一下，确定能正常工作后，接下来写一个 OllyScript.dll 插件的脚本（这招是和 shoooo 学来的，简单有效），内容如下：  
mov eip,00C50021   
run  
mov eip,00C50021   
run  
mov eip,00C50021   
run  
mov eip,00C50021   
run  
mov eip,00C50021   
run  
（地址表里有多少项就多少行，当然少些也没关系，多运行 2 次）  
  
在执行脚本时，确定除了 00AA5D21 或 00AA5DD3 两个出口处的硬件断点，无其他断点，当程序运行时，遇到硬件断点，将执行脚本程序，回到 C50021 这行继续补丁代码。用脚本的好处，是可以避免 Asprotect 校验。  
  
当补丁程序执行完毕后，再查看地址所对应的代码，例如：  
  
00401015      call    01200000 这行变成了：00401015      call    dword ptr \[4050B4\]  
  
表明程序己修复。  
  
**5.变形代码**  
  
上面的补丁代码运行后，同时会发现数据窗口生成的几个数据：  
00C50160  CA 1D 40 00 6D 25 40 00 D8 26 40 00 45 3E 40 00  ?@.m%@.?@.E>@.  
00C50170  7A 40 40 00 00 00 00 00 00 00 00 00 00 00 00 00  z@@.............  
00C50180  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00C50190  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
这几个地址就是函数下一行被偷的函数，这些地址是：401DCA,40256D,4026D8,403e45，经过查过，只有如下两处地址下一行代码被 Asprotect 变形：  
4026D8  
403e45  
  
先来看看 4026D8，现在壳里显示的是：  
004026D8  |.  FF15 5C504000    call    dword ptr \[40505C\]                     ; \\GetStdHandle  
004026DE  |.  10EA             adc     dl, ch  
004026E0  |.  83FF FF          cmp     edi, -1  
004026E3  |.  74 17            je      short 004026FC  
  
而加壳前的代码是：  
004026D8  |.  FF15 5C504000    call    dword ptr \[40505C\]     ; \\GetStdHandle  
004026DE  |.  8BF8             **mov     edi, eax**   //注意这句  
004026E0  |.  83FF FF          cmp     edi, -1  
  
  
经过比较你会发现，加壳后，Asprotect 将 GetStdHandle 函数下一句 mov edi, eax 放进壳里执行了，一般 kernel32.dll 函数会被偷下一行。在 Visual C 程序会有这情况存在，Delphi 程序不会有这问题。  
  
为了知道 Asprotect 是如何模拟处理下一行代码的，我们重新加载实例程序来跟踪。在跟踪前，你可以先 Dump 出刚修复的程序，或另开一个 OD 加载实例。  
  
加载实例后，来到 OEP，再新建 EIP 源到 004026D8：  
004026D8      E8 23D9DF00   call    01200000 //按 F7 跟进  
……  
00AA6905    50              push    eax  
00AA6906    8B45 14         mov     eax, \[ebp+14\]  
00AA6909    50              push    eax  
00AA690A    8B45 18         mov     eax, \[ebp+18\]  
00AA690D    50              push    eax  
00AA690E    8B45 0C         mov     eax, \[ebp+C\]  
00AA6911    50              push    eax  
00AA6912    8B45 F0         mov     eax, \[ebp-10\]  
00AA6915    50              push    eax  
00AA6916    8B4D 1C         mov     ecx, \[ebp+1C\]  
00AA6919    8B55 10         mov     edx, \[ebp+10\]  
00AA691C    8BC3            mov     eax, ebx  
00AA691E    E8 95F2FFFF     call    00AA5BB8                         ;  // b 情况这里 F7 进去  
进去后来到：  
00AA5D09    3A45 EF         cmp     al, \[ebp-11\]                     ; al 和 a 情况中的 dl 一样，决定是 ff15 还是 ff25  
00AA5D0C    0F85 9C000000   jnz     00AA5DAE                         ; 不跳是 FF15 情况 ,al=4c  
  
在这之前，和第三部分跟踪的过程一样，继续向下走：  
00AA5E54    837D D4 FF      cmp     dword ptr \[ebp-2C\], -1           ; 如果它是 FFFFFFFF，说明这个导入函数调用是干净的  
00AA5E58    74 33           je      short 00AA5E8D  
00AA5E5A    8B55 D4         mov     edx, \[ebp-2C\]  
00AA5E5D    8B45 F4         mov     eax, \[ebp-C\]  
00AA5E60    E8 1FFDFFFF     call    00AA5B84  
00AA5E65    8945 D8         mov     \[ebp-28\], eax  
00AA5E68    8D45 D8         lea     eax, \[ebp-28\]  
00AA5E6B    50              push    eax  
00AA5E6C    8B45 14         mov     eax, \[ebp+14\]  
00AA5E6F    50              push    eax  
00AA5E70    8B45 10         mov     eax, \[ebp+10\]  
00AA5E73    50              push    eax  
00AA5E74    8B45 0C         mov     eax, \[ebp+C\]  
00AA5E77    50              push    eax  
00AA5E78    8B45 08         mov     eax, \[ebp+8\]  
00AA5E7B    50              push    eax  
00AA5E7C    8B4D F8         mov     ecx, \[ebp-8\]  
00AA5E7F    8B55 F0         mov     edx, \[ebp-10\]  
00AA5E82    8B45 F4         mov     eax, \[ebp-C\]  
00AA5E85    E8 46000000     call    00AA5ED0                   //按 F7 进去  
  
下面就是 Asprotect 变形处理代码，用了 8 段代码，模拟 8 种情况：  
  
00AA5F7E    3A42 4B         cmp     al, \[edx+4B\]      //注意这里的 edx+4B  
00AA5F81    75 3E           jnz     short 00AA5FC1  
00AA5F83    EB 01           jmp     short 00AA5F86  
  
1====================================================================  
//edx+4B 的情况，模拟是 jmp   
00AA5F86    8B55 F8         mov     edx, \[ebp-8\]     
00AA5F89    8B9A E0000000   mov     ebx, \[edx+E0\]  
00AA5F8F    035D EC         add     ebx, \[ebp-14\]  
00AA5F92    8B55 F8         mov     edx, \[ebp-8\]  
00AA5F95    035A 14         add     ebx, \[edx+14\]  //ebx 中的值是一个地址，如 jmp xxxxx,call xxxxx  
00AA5F98    8B55 F8         mov     edx, \[ebp-8\]  
  
2====================================================================  
//edx+4A 的情况，模拟 CALL 指令  
00AA5F9B    3A42 4A         cmp     al, \[edx+4A\]  
00AA5F9E    0F85 A6020000   jnz     00AA624A      //如是 JMP 指令则从这跳走准备结束处理  
00AA5FA4    836D FC 04      sub     dword ptr \[ebp-4\], 4  
00AA5FA8    8B45 F8         mov     eax, \[ebp-8\]  
00AA5FAB    8B80 E0000000   mov     eax, \[eax+E0\]  
00AA5FB1    0345 F0         add     eax, \[ebp-10\]  
00AA5FB4    0345 F4         add     eax, \[ebp-C\]  
00AA5FB7    8B55 FC         mov     edx, \[ebp-4\]  
00AA5FBA    8902            mov     \[edx\], eax  
00AA5FBC    E9 89020000     jmp     00AA624A  
  
3====================================================================  
//edx+4F 的情况，模拟 add x,n  
00AA5FC1    8B55 F8         mov     edx, \[ebp-8\]  
00AA5FC4    3A42 4F         cmp     al, \[edx+4F\]  
00AA5FC7    75 6F           jnz     short 00AA6038  
00AA5FC9    33C0            xor     eax, eax  
00AA5FCB    8A46 05         mov     al, \[esi+5\]  
00AA5FCE    8D0440          lea     eax, \[eax+eax\*2\]  
00AA5FD1    8B55 F8         mov     edx, \[ebp-8\]  
00AA5FD4    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA5FD8    8BC3            mov     eax, ebx  
00AA5FDA    FFD2            call    edx            //(x) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
00AA5FDC    8845 EB         mov     \[ebp-15\], al  
00AA5FDF    EB 01           jmp     short 00AA5FE2  
00AA5FE1  - E9 33C08A46     jmp     47352019  
00AA5FE6    07              pop     es  
00AA5FE7    8D0440          lea     eax, \[eax+eax\*2\]  
00AA5FEA    8B55 F8         mov     edx, \[ebp-8\]  
00AA5FED    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA5FF1    8BC3            mov     eax, ebx  
00AA5FF3    FFD2            call    edx          //返回值是 n  
00AA5FF5    8BD8            mov     ebx, eax  
00AA5FF7    EB 01           jmp     short 00AA5FFA  
00AA5FF9    9A 807DEB04 742>call    far 2174:04EB7D80  
00AA6000    8A4D EB         mov     cl, \[ebp-15\]  
00AA6003    8B55 10         mov     edx, \[ebp+10\]  
00AA6006    8B45 F8         mov     eax, \[ebp-8\]  
00AA6009    E8 AA020000     call    00AA62B8  
00AA600E    03C3            add     eax, ebx  
00AA6010    50              push    eax  
00AA6011    8A4D EB         mov     cl, \[ebp-15\]  
00AA6014    8B55 10         mov     edx, \[ebp+10\]  
00AA6017    8B45 F8         mov     eax, \[ebp-8\]  
00AA601A    E8 75020000     call    00AA6294  
00AA601F    EB 03           jmp     short 00AA6024  
00AA6021    015D FC         add     \[ebp-4\], ebx  
00AA6024    8B5D F0         mov     ebx, \[ebp-10\]  
00AA6027    035D F4         add     ebx, \[ebp-C\]  
00AA602A    8B45 F8         mov     eax, \[ebp-8\]  
00AA602D    0398 E0000000   add     ebx, \[eax+E0\]  
00AA6033    E9 12020000     jmp     00AA624A  
  
4====================================================================  
//edx+50 的情况，模拟 MOV xxx1,xxx2  
00AA6038    8B55 F8         mov     edx, \[ebp-8\]  
00AA603B    3A42 50         cmp     al, \[edx+50\]  
00AA603E    75 63           jnz     short 00AA60A3  
00AA6040    33C0            xor     eax, eax  
00AA6042    8A46 05         mov     al, \[esi+5\]  
00AA6045    8D0440          lea     eax, \[eax+eax\*2\]  
00AA6048    8B55 F8         mov     edx, \[ebp-8\]  
00AA604B    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA604F    8BC3            mov     eax, ebx  
00AA6051    FFD2            call    edx            //0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  (xxx1)  
00AA6053    8845 EB         mov     \[ebp-15\], al  
00AA6056    EB 01           jmp     short 00AA6059  
00AA6058    E8 33C08A46     call    47352090  
00AA605D    06              push    es  
00AA605E    8D0440          lea     eax, \[eax+eax\*2\]  
00AA6061    8B55 F8         mov     edx, \[ebp-8\]  
00AA6064    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA6068    8BC3            mov     eax, ebx  
00AA606A    FFD2            call    edx           //0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi(xxx2)  
00AA606C    8845 EA         mov     \[ebp-16\], al  
00AA606F    8A4D EA         mov     cl, \[ebp-16\]  
00AA6072    8B55 10         mov     edx, \[ebp+10\]  
00AA6075    8B45 F8         mov     eax, \[ebp-8\]  
00AA6078    E8 3B020000     call    00AA62B8  
00AA607D    EB 01           jmp     short 00AA6080  
00AA607F    9A 508A4DEB 8B5>call    far 558B:EB4D8A50  
00AA6086    108B 45F8E805   adc     \[ebx+5E8F845\], cl  
00AA608C    0200            add     al, \[eax\]  
00AA608E    008B 5DF0035D   add     \[ebx+5D03F05D\], cl  
00AA6094    F4              hlt  
00AA6095    8B45 F8         mov     eax, \[ebp-8\]  
00AA6098    0398 E0000000   add     ebx, \[eax+E0\]  
00AA609E    E9 A7010000     jmp     00AA624A  
  
  
5====================================================================  
//edx+51 的情况，具体参考第 51 楼，VolX 的解答。  
00AA60A3    8B55 F8         mov     edx, \[ebp-8\]  
00AA60A6    3A42 51         cmp     al, \[edx+51\]  
00AA60A9    75 5E           jnz     short 00AA6109  
00AA60AB    EB 01           jmp     short 00AA60AE  
00AA60AD    E8 33C08A46     call    473520E5  
00AA60B2    05 8D04408B     add     eax, 8B40048D  
00AA60B7    55              push    ebp  
00AA60B8    F8              clc  
00AA60B9    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA60BD    8BC3            mov     eax, ebx  
00AA60BF    FFD2            call    edx  
00AA60C1    8845 EB         mov     \[ebp-15\], al  
00AA60C4    EB 01           jmp     short 00AA60C7  
00AA60C6  - E9 33C08A46     jmp     473520FE  
00AA60CB    07              pop     es  
00AA60CC    8D0440          lea     eax, \[eax+eax\*2\]  
00AA60CF    8B55 F8         mov     edx, \[ebp-8\]  
00AA60D2    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA60D6    8BC3            mov     eax, ebx  
00AA60D8    FFD2            call    edx  
00AA60DA    8BD8            mov     ebx, eax  
00AA60DC    8A4D EB         mov     cl, \[ebp-15\]  
00AA60DF    8B55 10         mov     edx, \[ebp+10\]  
00AA60E2    8B45 F8         mov     eax, \[ebp-8\]  
00AA60E5    E8 CE010000     call    00AA62B8  
00AA60EA    8B55 F8         mov     edx, \[ebp-8\]  
00AA60ED    035A 14         add     ebx, \[edx+14\]  
00AA60F0    8903            mov     \[ebx\], eax  
00AA60F2    EB 01           jmp     short 00AA60F5  
00AA60F4    9A 8B5DF003 5DF>call    far F45D:03F05D8B  
00AA60FB    8B45 F8         mov     eax, \[ebp-8\]  
00AA60FE    0398 E0000000   add     ebx, \[eax+E0\]  
00AA6104    E9 41010000     jmp     00AA624A  
  
6====================================================================  
//edx+52 的情况，模拟 mov     \[x+n\],y  
  
00AA610C    3A42 52         cmp     al, \[edx+52\]  
00AA610F    0F85 80000000   jnz     00AA6195  
00AA6115    EB 01           jmp     short 00AA6118  
00AA6117    E8 33C08A46     call    4735214F  
00AA611C    05 8D04408B     add     eax, 8B40048D  
00AA6121    55              push    ebp  
00AA6122    F8              clc  
00AA6123    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA6127    8BC3            mov     eax, ebx  
00AA6129    FFD2            call    edx              // (X) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi   
00AA612B    8845 EB         mov     \[ebp-15\], al  
00AA612E    33C0            xor     eax, eax  
00AA6130    8A46 06         mov     al, \[esi+6\]  
00AA6133    8D0440          lea     eax, \[eax+eax\*2\]  
00AA6136    8B55 F8         mov     edx, \[ebp-8\]  
00AA6139    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA613D    8BC3            mov     eax, ebx  
00AA613F    FFD2            call    edx            //(Y) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
00AA6141    8845 EA         mov     \[ebp-16\], al  
00AA6144    EB 01           jmp     short 00AA6147  
00AA6146  - E9 33C08A46     jmp     4735217E  
00AA614B    07              pop     es  
00AA614C    8D0440          lea     eax, \[eax+eax\*2\]  
00AA614F    8B55 F8         mov     edx, \[ebp-8\]  
00AA6152    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA6156    8BC3            mov     eax, ebx  
00AA6158    FFD2            call    edx             //常数 n  
00AA615A    8BD8            mov     ebx, eax  
00AA615C    8A4D EB         mov     cl, \[ebp-15\]  
00AA615F    8B55 10         mov     edx, \[ebp+10\]  
00AA6162    8B45 F8         mov     eax, \[ebp-8\]  
00AA6165    E8 4E010000     call    00AA62B8  
00AA616A    8BF0            mov     esi, eax  
00AA616C    8A4D EA         mov     cl, \[ebp-16\]  
00AA616F    8B55 10         mov     edx, \[ebp+10\]  
00AA6172    8B45 F8         mov     eax, \[ebp-8\]  
00AA6175    E8 3E010000     call    00AA62B8  
00AA617A    03F3            add     esi, ebx  
00AA617C    8906            mov     \[esi\], eax  
00AA617E    EB 01           jmp     short 00AA6181  
00AA6180    9A 8B5DF003 5DF>call    far F45D:03F05D8B  
00AA6187    8B45 F8         mov     eax, \[ebp-8\]  
00AA618A    0398 E0000000   add     ebx, \[eax+E0\]  
00AA6190    E9 B5000000     jmp     00AA624A  
  
  
7====================================================================  
//edx+4C 的情况，模拟：   
CMP x,y  
jxx n  
（特别感谢 jwh51 在这情况给与的技术指点）  
  
00AA6195    8B55 F8         mov     edx, \[ebp-8\]  
00AA6198    3A42 4C         cmp     al, \[edx+4C\]  
00AA619B    75 67           jnz     short 00AA6204  
00AA619D    EB 01           jmp     short 00AA61A0  
  
00AA61A0    8BCB            mov     ecx, ebx  
00AA61A2    8B55 10         mov     edx, \[ebp+10\]  
00AA61A5    8B45 F8         mov     eax, \[ebp-8\]  
00AA61A8    E8 6FF7FFFF     call    00AA591C   //进去  
{  
     00AA5967    8D0440          lea     eax, \[eax+eax\*2\]  
     00AA596A    8B5483 68       mov     edx, \[ebx+eax\*4+68\]  
     00AA596E    8BC7            mov     eax, edi  
     00AA5970    FFD2            call    edx                              ; (x)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
     00AA5972    8BD0            mov     edx, eax  
     00AA5974    80EA 08         sub     dl, 8  
     00AA5977    0F92C2          setb    dl  
     00AA597A    80FA 01         cmp     dl, 1  
     00AA597D    75 0F           jnz     short 00AA598E  
     00AA597F    8BC8            mov     ecx, eax  
     00AA5981    8B55 FC         mov     edx, \[ebp-4\]  
     00AA5984    8BC3            mov     eax, ebx  
     00AA5986    E8 2D090000     call    00AA62B8  
     00AA598B    8945 EC         mov     \[ebp-14\], eax  
     00AA598E    33C0            xor     eax, eax  
     00AA5990    8A46 09         mov     al, \[esi+9\]  
     00AA5993    8D0440          lea     eax, \[eax+eax\*2\]  
     00AA5996    8B5483 68       mov     edx, \[ebx+eax\*4+68\]  
     00AA599A    8BC7            mov     eax, edi  
     00AA599C    FFD2            call    edx  
     00AA599E    8845 F3         mov     \[ebp-D\], al  
     00AA59A1    EB 01           jmp     short 00AA59A4  
     00AA59A3    6933 C08A4608   imul    esi, \[ebx\], 8468AC0  
     00AA59A9    8D0440          lea     eax, \[eax+eax\*2\]  
     00AA59AC    8B5483 68       mov     edx, \[ebx+eax\*4+68\]  
     00AA59B0    8BC7            mov     eax, edi  
     00AA59B2    FFD2            call    edx                              ;  如果是 cmp x,n  情况，此处返回常数 n。否则返回 0  
     00AA59B4    8945 F4         mov     \[ebp-C\], eax  
     00AA59B7    33C0            xor     eax, eax  
     00AA59B9    8A46 06         mov     al, \[esi+6\]  
     00AA59BC    8D0440          lea     eax, \[eax+eax\*2\]  
     00AA59BF    8B5483 68       mov     edx, \[ebx+eax\*4+68\]  
     00AA59C3    8BC7            mov     eax, edi  
     00AA59C5    FFD2            call    edx                              ; (y)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
     00AA59C7    8BD0            mov     edx, eax  
     00AA59C9    80EA 08         sub     dl, 8  
     00AA59CC    0F92C2          setb    dl  
     00AA59CF    80FA 01         cmp     dl, 1  
     00AA59D2    75 12           jnz     short 00AA59E6  
}  
00AA61B3    33C0            xor     eax, eax  
00AA61B5    8A46 04         mov     al, \[esi+4\]  
00AA61B8    8D0440          lea     eax, \[eax+eax\*2\]  
00AA61BB    8B55 F8         mov     edx, \[ebp-8\]  
00AA61BE    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
00AA61C2    8BC3            mov     eax, ebx  
00AA61C4    FFD2            call    edx                              ; 跳转类型，如是机器码是 74 则返回 4,如是 75 则返回 5，依次类推  
00AA61C6    8BD8            mov     ebx, eax  
00AA61C8    8B4D 14         mov     ecx, \[ebp+14\]  
00AA61CB    8BD3            mov     edx, ebx  
00AA61CD    8B45 F8         mov     eax, \[ebp-8\]  
00AA61D0    E8 87F4FFFF     call    00AA565C  
00AA61D5    84C0            test    al, al  
00AA61D7    74 1A           je      short 00AA61F3  
00AA61D9    EB 01           jmp     short 00AA61DC  
  
00AA61DC    8B45 F8         mov     eax, \[ebp-8\]  
00AA61DF    8B98 E0000000   mov     ebx, \[eax+E0\]  
00AA61E5    035D EC         add     ebx, \[ebp-14\]  
00AA61E8    8B45 F8         mov     eax, \[ebp-8\]  
00AA61EB    0358 14         add     ebx, \[eax+14\]                    ; 跳转的地址  
00AA61EE    EB 5A           jmp     short 00AA624A  
  
00AA61F3    8B45 F8         mov     eax, \[ebp-8\]  
00AA61F6    8B98 E0000000   mov     ebx, \[eax+E0\]  
00AA61FC    035D F0         add     ebx, \[ebp-10\]  
00AA61FF    035D F4         add     ebx, \[ebp-C\]                     ; 不跳的地址  
00AA6202    EB 46           jmp     short 00AA624A  
  
  
  
8====================================================================  
//edx+4D 的情况，模拟:CMP x,y  
  
00AA6204    8B55 F8         mov     edx, \[ebp-8\]  
00AA6207    3A42 4D         cmp     al, \[edx+4D\]  
00AA620A    75 27           jnz     short 00AA6233  
00AA620C    EB 01           jmp     short 00AA620F  
  
00AA620F    8BCB            mov     ecx, ebx  
00AA6211    8B55 10         mov     edx, \[ebp+10\]  
00AA6214    8B45 F8         mov     eax, \[ebp-8\]  
00AA6217    E8 00F7FFFF     call    00AA591C   // 里面和 edx+4C 情况处理一样  
  
  
【版权声明】本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
【参考资料】[nspack3.5 主程序脱壳分析(Aspr SKE 2.X)  ](http://bbs.pediy.com/showthread.php?s=&threadid=19313)作者：shoooo  
  
  
                                                       看雪技术论坛(http://bbs.pediy.com)  
                                                       2006 年 04 月 07 日  

009ABE0053push ebx  
009ABE0156push esi  
009ABE0257push edi  
009ABE0355push ebp  
009ABE048BFAmov edi,edx  
009ABE06B3 01 mov bl,1  
009ABE088BF0mov esi,eax  
009ABE0A83C6 08 add esi,8  
009ABE0D8BEEmov ebp,esi  
009ABE0FEB 15 jmp short 009ABE26  
009ABE1155push ebp  
009ABE12FFD7call edi  //处理部分 CALL，变成 PUSH XXXXXXXX，RETN 或 JUMP XXXXXXXX  
009ABE1484C0test al,al  
009ABE1675 04 jnz short 009ABE1C  
009ABE1833DBxor ebx,ebx  
009ABE1AEB 10 jmp short 009ABE2C  
009ABE1C8B45 04 mov eax,dword ptr ss:\[ebp+4\]  
009ABE1F83C0 08 add eax,8  
009ABE2203F0add esi,eax  
009ABE248BEEmov ebp,esi  
009ABE26837D 04 00cmp dword ptr ss:\[ebp+4\],0  
009ABE2A^ 75 E5 jnz short 009ABE11  
009ABE2C8BC3mov eax,ebx  
009ABE2E5Dpop ebp  
009ABE2F5Fpop edi  
009ABE305Epop esi  
009ABE315Bpop ebx  
009ABE32C3retn  
  
009AC47C55push ebp  
009AC47D8BECmov ebp,esp  
009AC47F83C4 F8 add esp,-8  
009AC48253push ebx  
009AC48356push esi  
009AC484C645 FF 01mov byte ptr ss:\[ebp-1\],1  
009AC488A1 0C389B00 mov eax,dword ptr ds:\[9B380C\]  
009AC48D8B00mov eax,dword ptr ds:\[eax\]  
009AC48F8B30mov esi,dword ptr ds:\[eax\]  
009AC4918B5D 08 mov ebx,dword ptr ss:\[ebp+8\]  
009AC4948D45 F8 lea eax,dword ptr ss:\[ebp-8\]  
009AC4978B53 04 mov edx,dword ptr ds:\[ebx+4\]  
009AC49AE8 0923FFFF call 0099E7A8  //分配内存  
009AC49F8D53 08 lea edx,dword ptr ds:\[ebx+8\]  
009AC4A28B4B 04 mov ecx,dword ptr ds:\[ebx+4\]  
009AC4A58B45 F8 mov eax,dword ptr ss:\[ebp-8\] //要处理的字节数  
009AC4A8E8 039BFDFF call 00985FB0  
009AC4AD8B45 F8 mov eax,dword ptr ss:\[ebp-8\]  
009AC4B0E8 73FFFFFF call 009AC428  
009AC4B50333add esi,dword ptr ds:\[ebx\]  
009AC4B78BDEmov ebx,esi  
009AC4B9B8 02000000 mov eax,2  
009AC4BEE8 0164FDFF call 009828C4  
009AC4C385C0test eax,eax  
009AC4C575 13 jnz short 009AC4DA  
009AC4C7C603 68 mov byte ptr ds:\[ebx\],68  //变成 PUSH XXXXXXXX  
009AC4CA8D43 01 lea eax,dword ptr ds:\[ebx+1\]  
009AC4CD8B55 F8 mov edx,dword ptr ss:\[ebp-8\]  
009AC4D08910mov dword ptr ds:\[eax\],edx  
009AC4D283C3 05 add ebx,5  
009AC4D5C603 C3 mov byte ptr ds:\[ebx\],0C3 //RETN  
009AC4D8EB 10 jmp short 009AC4EA  
009AC4DA8B45 F8 mov eax,dword ptr ss:\[ebp-8\]  
009AC4DD2BC3sub eax,ebx  
009AC4DF83E8 05 sub eax,5  
009AC4E28D53 01 lea edx,dword ptr ds:\[ebx+1\]  
009AC4E58902mov dword ptr ds:\[edx\],eax  
009AC4E7C603 E9 mov byte ptr ds:\[ebx\],0E9 //变成 JUMP XXXXXXXX  
009AC4EA8A45 FF mov al,byte ptr ss:\[ebp-1\]  
009AC4ED5Epop esi  
009AC4EE5Bpop ebx  
009AC4EF59pop ecx  
009AC4F059pop ecx  
009AC4F15Dpop ebp  
009AC4F2C2 0400 retn 4  
  
我见过 2 个程序，吃的代码都一样，0x36 个字节，不过还原蛮简单的，其它的不是很清楚  
10003030    68 0000BC00             push 0BC0000  
10003035    C3                      retn  
  
00BC0000 > \\68 FFFFFFFF push -1  
00BC0005 .50push eax  
00BC0006 .83C8 FF or eax,FFFFFFFF  
00BC0009 .40inc eax  
00BC000A .64:8B00 mov eax,dword ptr fs:\[eax\]  
00BC000D .50push eax  
00BC000E .64:EB 02jmp short dumped\_.00BC0013  
00BC0011CDdb CD  
00BC001220db 20;CHAR ' '  
00BC0013 >81E8 F6C57142 sub eax,4271C5F6  
00BC0019 .8B4424 0C mov eax,dword ptr ss:\[esp+C\]  
00BC001D .64:8925 00000000mov dword ptr fs:\[0\],esp  
00BC0024 .896C24 0C mov dword ptr ss:\[esp+C\],ebp  
00BC0028 .BD 96594800 mov ebp,485996  
00BC002D .C1CD A7 ror ebp,0A7  
00BC0030 .8D6C24 0C lea ebp,dword ptr ss:\[esp+C\]  
00BC0034 .50push eax  
00BC0035 .C3retn 

引用:  *最初由** kimmal **发布*  
009ABE0053push ebx  
009ABE0156push esi  
009ABE0257push edi  
009ABE0355push ebp  
009ABE048BFAmov edi,edx  
........ 

  
  
嗯，会抽一些标准函数，或者库函数等  
一般 VC 的会抽这个  
 push -1  
 push eax  
 mov eax,dword ptr fs:\[0\]  
 push eax  
 mov eax,dword ptr ss:\[esp+C\]  
 mov dword ptr fs:\[0\],esp  
 mov dword ptr ss:\[esp+C\],ebp  
 lea ebp,dword ptr ss:\[esp+C\]  
 push eax  
 retn  
就是你贴的那个原型  
delphi 的有时会抽得更多， 甚至 10 几个，但都是标准样子的库函数，像拷贝内存， 取字符串长度等，集中在代码段的前面 

//edx+51 ---  MOV \[n\],xxx1  
                  00AA60A3    8B55 F8         mov     edx, \[ebp-8\]  
                  00AA60A6    3A42 51         cmp     al, \[edx+51\]  
                  00AA60A9    75 5E           jnz     short 00AA6109  
                  00AA60AB    EB 01           jmp     short 00AA60AE  
                  00AA60AD    E8 33C08A46     call    473520E5  
                  00AA60B2    05 8D04408B     add     eax, 8B40048D  
                  00AA60B7    55              push    ebp  
                  00AA60B8    F8              clc  
                  00AA60B9    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
                  00AA60BD    8BC3            mov     eax, ebx  
                  00AA60BF    FFD2            call    edx  
                  00AA60C1    8845 EB         mov     \[ebp-15\], al   <-al==xxx1  
                  //0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
                  00AA60C4    EB 01           jmp     short 00AA60C7  
                  00AA60C6  - E9 33C08A46     jmp     473520FE  
                  00AA60CB    07              pop     es  
                  00AA60CC    8D0440          lea     eax, \[eax+eax\*2\]  
                  00AA60CF    8B55 F8         mov     edx, \[ebp-8\]  
                  00AA60D2    8B5482 68       mov     edx, \[edx+eax\*4+68\]  
                  00AA60D6    8BC3            mov     eax, ebx  
                  00AA60D8    FFD2            call    edx  
                  00AA60DA    8BD8            mov     ebx, eax  
                  00AA60DC    8A4D EB         mov     cl, \[ebp-15\]  
                  00AA60DF    8B55 10         mov     edx, \[ebp+10\]  
                  00AA60E2    8B45 F8         mov     eax, \[ebp-8\]  
                  00AA60E5    E8 CE010000     call    00AA62B8  
                  00AA60EA    8B55 F8         mov     edx, \[ebp-8\]  
                  00AA60ED    035A 14         add     ebx, \[edx+14\]  <-ebx==n  
                  00AA60F0    8903            mov     \[ebx\], eax  
                  00AA60F2    EB 01           jmp     short 00AA60F5  
                  00AA60F4    9A 8B5DF003 5DF>call    far F45D:03F05D8B  
                  00AA60FB    8B45 F8         mov     eax, \[ebp-8\]  
                  00AA60FE    0398 E0000000   add     ebx, \[eax+E0\]  
                  00AA6104    E9 41010000     jmp     00AA624A