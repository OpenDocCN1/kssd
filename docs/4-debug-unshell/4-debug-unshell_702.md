

# FSG 1.33 -> dulek/xt 简单分析

> 原文：[`www.pediy.com/kssd/pediy10/100085.html`](https://www.pediy.com/kssd/pediy10/100085.html)

FSG 1.33 -> dulek/xt 大体思路：  
  
1. FSG 首先会初始化 ESI（需要解压数据的地址）和 EDI（解压后数据存放的地址）；  
2. 00404B9B    74 2F           je      short 00404BCC  
    该处代码每实现一次都会都会完成一段数据的解密工作，每次解压完成后可以看一下内存映像，看看解压后的效果；  
3. 然后再次初始化 ESI 和 EDI，进行下一段数据的解密；  
  
  
1.  首先，我们利用 PeiD 查看一下区段信息，内容如下：  
  
 这里需要注意：  
第一个区段的文件偏移和文件大小都为零，相对虚拟地址为 1000，大小为 21000，所以当用 OD 载入内存的时候，内存地址 401000---421FFF（第一区段的内存空间）区域内，所有的数据都是 0，而这毫无以为是没有意义的。在这里我们就会猜想到，外壳程序肯定会向这个内存区域填充数据，而数据的来源就应该是第二个区段的数据，这只是我们的假设，后面我们会利用代码来验证我们的假设。  
2.  利用 OD 载入我们的程序，在数据窗口中我们查看一下各个区段的数据：  
第一个区段（401000421FFF）的内容都为 0：  
  
第二个区段（422000---430FFF）为一些无规则的数据，这就是需要解压的数据：  
  
3.  看完了内存数据的分配，下面让我们主要看一下我们的代码：  
00430764 >  BE A4014000     mov     esi, 004001A4      ; 这是个定值，用于提取数据  
00430769    AD              lods    dword ptr \[esi\]     ; 提取 400198，用于定位后面的子程序调用  
0043076A    93              xchg    eax, ebx           ; 并保存到 EBX  
0043076B    AD              lods    dword ptr \[esi\]     ; 提取 401000（第一个区段的虚拟地址）  
0043076C    97              xchg    eax, edi           ; 并保存到 EDI  
0043076D    AD              lods    dword ptr \[esi\]   ; 提取 422CE4（需要解压的数据的地址，位于第二区段中）  
0043076E    56              push    esi             ; 保存 ESI=4001B0  
0043076F    96              xchg    eax, esi  
00430770    B2 80           mov     dl, 80  
00430772    A4              movs    byte ptr es:\[edi\], byte ptr \[esi\]         ; 向第一区段解压数据  
下图：利用 4001A4 来定位要提取的数据：  
  
4.  初始化好 ESI（需要解压的数据的地址）和 ESI（存放数据的地址），就开始复制数据了，代码如下：  
00430772    A4              movs    byte ptr es:\[edi\], byte ptr \[esi\]  ; 向第一区段解压数据  
00430773    B6 80           mov     dh, 80  
00430775    FF13            call    dword ptr \[ebx\]  ; EBX=400198,该调用主要是进行跳转的判断，小菜不懂算法，哈哈  
00430777  ^ 73 F9           jnb     short 00430772  
00430779    33C9            xor     ecx, ecx  
0043077B    FF13            call    dword ptr \[ebx\]  
0043077D    73 16           jnb     short 00430795  
0043077F    33C0            xor     eax, eax  
00430781    FF13            call    dword ptr \[ebx\]  
00430783    73 1F           jnb     short 004307A4  
00430785    B6 80           mov     dh, 80  
00430787    41              inc     ecx  
00430788    B0 10           mov     al, 10  
0043078A    FF13            call    dword ptr \[ebx\]  
0043078C    12C0            adc     al, al  
0043078E  ^ 73 FA           jnb     short 0043078A  
00430790    75 3C           jnz     short 004307CE  
00430792    AA              stos    byte ptr es:\[edi\]  
00430793  ^ EB E0           jmp     short 00430775  
00430795    FF53 08         call    dword ptr \[ebx+8\]  
00430798    02F6            add     dh, dh  
0043079A    83D9 01         sbb     ecx, 1  
0043079D    75 0E           jnz     short 004307AD  
0043079F    FF53 04         call    dword ptr \[ebx+4\]  
004307A2    EB 26           jmp     short 004307CA  
004307A4    AC              lods    byte ptr \[esi\]  
004307A5    D1E8            shr     eax, 1  
004307A7    74 2F           je      short 004307D8  ；当该处跳转实现的时候，就完成了一段数据的解压工作，然后需要重新赋值 ESI 和 EDI，进行下一断数据的解压；  
004307A9    13C9            adc     ecx, ecx  
004307AB    EB 1A           jmp     short 004307C7  
004307AD    91              xchg    eax, ecx  
004307AE    48              dec     eax  
004307AF    C1E0 08         shl     eax, 8  
004307B2    AC              lods    byte ptr \[esi\]  
004307B3    FF53 04         call    dword ptr \[ebx+4\]  
004307B6    3D 007D0000     cmp     eax, 7D00  
004307BB    73 0A           jnb     short 004307C7  
004307BD    80FC 05         cmp     ah, 5  
004307C0    73 06           jnb     short 004307C8  
004307C2    83F8 7F         cmp     eax, 7F  
004307C5    77 02           ja      short 004307C9  
004307C7    41              inc     ecx  
004307C8    41              inc     ecx  
004307C9    95              xchg    eax, ebp  
004307CA    8BC5            mov     eax, ebp  
004307CC    B6 00           mov     dh, 0  
004307CE    56              push    esi    ; （422CEE）（422CFD）（422CFE）  
004307CF    8BF7            mov     esi, edi  
004307D1    2BF0            sub     esi, eax  
004307D3    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
004307D5    5E              pop     esi  
004307D6  ^ EB 9D           jmp     short 00430775  
004307D8    8BD6            mov     edx, esi   ；运行到该处时，需要重新计算 ESI 和 EDI，此时可以打开内存镜像，查看一下解压后的数据；  
004307DA    5E              pop     esi  
004307DB    AD              lods    dword ptr \[esi\]  
004307DC    48              dec     eax  
5.  重新计算 ESI 和 EDI 的方法：  
  
  
其实利用该图来获得 EDI（解压数据的存放地址，按顺序依次给 EDI 赋值），ESI 从 422CE4 依次增长。  
6.  解压的大体流程：   
第一段需要解压的数据：把区域 4220CE4---42E3A0 的数据解压到 401000；  
第二段需要解压的数据：把区域 42E3A0----42F24B 的数据解压到 415000；（用来存放 IAT）  
第三段需要解压的数据：把区域 42F24B----42F787 的数据解压到 41A000；  
第四段需要解压的数据：把区域 42F787-----43000C 的数据解压到 41F000；  
第五次需要解压的数据：把区域 43000C-----430764 的数据解压到 4019DB ；（解压出来的数据是函数名）  
7. 当函数名解压出来之后，就要来获得各个函数的地址，然后填充 IAT 啦：  
  下图就是解压出来的数据：  
  
  
然后我们看一下 IAT 的填充代码：  
004307E7  ^\\EB 87           jmp     short 00430770  
004307E9    AD              lods    dword ptr \[esi\]  ; 获得 LoadLibraryA 的地址  
004307EA    93              xchg    eax, ebx         ; 保存到 EBX  
004307EB    5E              pop     esi  
004307EC    46              inc     esi  
004307ED    AD              lods    dword ptr \[esi\]  ; 得到 IAT 的虚拟地址  
004307EE    97              xchg    eax, edi       ; 保存到 EDI  
004307EF    56              push    esi  
004307F0    FF13            call    dword ptr \[ebx\]  ; 利用 LoadLibraryA 加载 KERNEL32.DLL  
004307F2    95              xchg    eax, ebp   ;模块基址保存到 EBP  
004307F3    AC              lods    byte ptr \[esi\]   ; 获得函数名  
004307F4    84C0            test    al, al  
004307F6  ^ 75 FB           jnz     short 004307F3  
004307F8    FE0E            dec     byte ptr \[esi\]     ; 减一  
004307FA  ^ 74 F0           je      short 004307EC    ; 如果等于零，则处理另一个模块的函数，不等于零则还是本模块  
004307FC    79 05           jns     short 00430803  
004307FE    46              inc     esi  
004307FF    AD              lods    dword ptr \[esi\]  
00430800    50              push    eax  
00430801    EB 09           jmp     short 0043080C  
00430803    FE0E            dec     byte ptr \[esi\]    ; 再减一，获得函数名  
00430805  - 0F84 5B1EFDFF   je      00402666  ;等于 0 时候处理完成了所有的函数，跳到 OEP  
0043080B    56              push    esi       ; 函数名地址入栈  
0043080C    55              push    ebp     ; 模块基址入栈  
0043080D    FF53 04         call    dword ptr \[ebx+4\]  ; 调用 GetProcAddress  
00430810    AB              stos    dword ptr es:\[edi\]  ; 将地址填充到 IAT  
00430811  ^ EB E0           jmp     short 004307F3   ; 处理下一个函数  
IAT 填充的思路：  
要想填充 IAT，我们就要活得 IAT 的地址，该地址保存在每个模块名的前面的四个字节中，将该值给 EDI，就可以利用 stos，保存函数地址了。  
函数名的查找过程：扫描每个字节，然后进行减一操作，若为零，则当前模块的函数处理完毕，需要处理下一个模块，如果不等于零，则再次减一，此次若等于零，则表示所有的函数处理完成了，直接跳到 OEP 就可以了，若不等于零，则处理本模块的下一个函数。