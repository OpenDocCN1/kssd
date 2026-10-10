

# Themida 脱壳(VC++ 7.0 之 Stolen Code 还原的一种思路)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-492.htm`](https://www.pediy.com/kssd/pediy09/pediy09-492.htm)

> 【文章标题】: Themida 脱壳(VC++ 7.0 之 Stolen Code 还原的一种思路)  
> 【文章作者】: wynney  
> 【软件名称】: 奇迹世界  
> 【下载地址】: 下面 FTP 打包下载  
> 【编写语言】: Microsoft Visual C++ 7.0  
> 【作者声明】: 一个朋友让帮弄的，看了下，借用 okdodo 的脚本到达伪 OEP，被抽取了近 60 字节  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   
>   一、前言  
>     
>   很多加壳软件对 OEP 的 VM 大多数对 VC++的 VM 一般都很多，而且基本上是最多的，其他语言就稍少些了，VB 就更少了=没有，呵呵  
>     
>   二、达到伪 OEP\[用脚本\]  
>     
>   如果大家想研究如何修复 IAT，到达伪 OEP 的话，可以研究下 okdodo 的脚本，大大减少体力劳动，感谢 ing  
>     
>   
> 
> 引用:   00610E50    68 1CAE6000     push    0060AE1C                         ;  脚本停在这,记住①，Call 00610E50，看堆栈   
>   00610E55    64:A1 00000000  mov     eax, dword ptr fs:\[0\]  
>   00610E5B    50              push    eax  
>   00610E5C    8B4424 10       mov     eax, dword ptr \[esp+10\]  
>   00610E60    896C24 10       mov     dword ptr \[esp+10\], ebp  
>   00610E64    8D6C24 10       lea     ebp, dword ptr \[esp+10\]  
>   00610E68    2BE0            sub     esp, eax  
>   00610E6A    53              push    ebx  
>   00610E6B    56              push    esi  
>   00610E6C    57              push    edi  
>   00610E6D    8B45 F8         mov     eax, dword ptr \[ebp-8\]  
>   00610E70    8965 E8         mov     dword ptr \[ebp-18\], esp  
>   00610E73    50              push    eax  
>   00610E74    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
>   00610E77    C745 FC FFFFFFF>mov     dword ptr \[ebp-4\], -1  
>   00610E7E    8945 F8         mov     dword ptr \[ebp-8\], eax  
>   00610E81    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
>   00610E84    64:A3 00000000  mov     dword ptr fs:\[0\], eax  
>   00610E8A    C3              retn                                     ; 直接 F4，F8 到 VM 中  
>    
> 
>     
>   0013FF8C   009433DC    
>   0013FF90   0069A378    ②,push 0069A378   
>   0013FF94   00000060    ③,push 60   
>     
>     
>   在 Code 段 F2，Shift+F9  
>     
>   
> 
> 引用:   0060AF20    3D 00100000     cmp     eax, 1000                     ;  停在这，记住④ ，Call 0060AF20  
>   0060AF25    73 0E           jnb     short 0060AF35  
>   0060AF27    F7D8            neg     eax  
>   0060AF29    03C4            add     eax, esp  
>   0060AF2B    83C0 04         add     eax, 4  
>   0060AF2E    8500            test    dword ptr \[eax\], eax  
>   0060AF30    94              xchg    eax, esp  
>   0060AF31    8B00            mov     eax, dword ptr \[eax\]  
>   0060AF33    50              push    eax  
>   0060AF34    C3              retn                                  ; 直接 F4，F8 到 VM 中  
>    
> 
>     
>   在 Code 段 F2，Shift+F9  
>     
>   
> 
> 引用:   00E85C5F    FF32             push    dword ptr \[edx\]               ; ■停在这，kernel32.GetVersionExA   
>   00E85C61    E9 A0490400      jmp     00ECA606  
>   00E85C66    21C2             and     edx, eax  
>   00E85C68    05 F047F263      add     eax, 63F247F0  
>   00E85C6D    2D DA021626      sub     eax, 261602DA  
>    
> 
>     
>   EDX 里面的值得就是 GetVersionExA 所在的地址了   
>   从上面的 kernel32.GetVersionExA 可以看得出来，这个应该是 VC++ 7.0 编译的  
>   找来一个 VC++ 7.0 的例子，或者就用程序目录下没有加壳的 SUN.exe\[也是 VC++ 7.0 编译的\]以做下用：）  
>     
>   三、找回 Stolen Code  
>     
>   Alt+M，Ctrl+B 搜索这么一段代码，VC++ 7.0 都有的，而且一般 VM 不到这里：）  
>   
>   
> 
> 引用:   xor     eax, eax  
>   cmp     dword ptr \[ecx+E8\], esi  
>   setne   al  
>   mov     dword ptr \[ebp-1C\], eax  
>   push    1  
>     
>   33 C0 39 B1 E8 00 00 00 0F 95 C0 89 45 E4 6A 01  
>    
> 
>   找到了 0060CC7C，Ctrl+G：0060CC7C，看看被 VM 了不少字节，可以确认 0060CBC3 就是 OEP 了  
>     
>   
> 
> 引用:   0060CBFA    8B56 08          mov     edx, dword ptr \[esi+8\]  
>   0060CBFD    8915 2CD86F00    mov     dword ptr \[6FD82C\], edx  
>   0060CC03    8B76 0C          mov     esi, dword ptr \[esi+C\]  
>   0060CC06    81E6 FF7F0000    and     esi, 7FFF  
>   0060CC0C    8935 20D86F00    mov     dword ptr \[6FD820\], esi  
>    
> 
>     
>   我们需要找的上面被 Stolen Code 的代码了  
>   根据自己找来的 VC++ 7.0 程序对比下  
>     
>   
> 
> 引用:   00419EDA > $  6A 60         push    60                               ;   ◆   
>   00419EDC   .  68 78024600   push    00460278                         ;   ⑴需要找的第 1 点   
>   00419EE1   .  E8 2A020000   call    0041A110                         ;   ⑵需要找的第 2 点   
>   00419EE6   .  BF 94000000   mov     edi, 94  
>   00419EEB   .  8BC7          mov     eax, edi  
>   00419EED   .  E8 DED8FFFF   call    004177D0                         ;   ⑶需要找的第 3 点   
>   00419EF2   .  8965 E8       mov     dword ptr \[ebp-18\], esp  
>   00419EF5   .  8BF4          mov     esi, esp  
>   00419EF7   .  893E          mov     dword ptr \[esi\], edi  
>   00419EF9   .  56            push    esi                              ; /pVersionInformation  
>   00419EFA   .  FF15 9CA24500 call    dword ptr \[<&KERNEL32.GetVersion>; \\GetVersionExA         ;■  
>   00419F00   .  8B4E 10       mov     ecx, dword ptr \[esi+10\]  
>   00419F03   .  890D 0C654700 mov     dword ptr \[47650C\], ecx          ;   ⑷47650C 是需要找的第 4 点   
>   00419F09   .  8B46 04       mov     eax, dword ptr \[esi+4\]  
>   00419F0C   .  A3 18654700   mov     dword ptr \[476518\], eax          ;   ⑸476518 是需要找的第 5 点   
>   00419F11   .  8B56 08       mov     edx, dword ptr \[esi+8\]  
>   00419F14   .  8915 1C654700 mov     dword ptr \[47651C\], edx          ;   ⑹注意这个 47651C 和上面 2 个地址的关系   
>   00419F1A   .  8B76 0C       mov     esi, dword ptr \[esi+C\]  
>   00419F1D   .  81E6 FF7F0000 and     esi, 7FFF  
>   00419F23   .  8935 10654700 mov     dword ptr \[476510\], esi          ;   ⑺注意这个 476510 和上面 3 个地址的关系   
>   00419F29   .  83F9 02       cmp     ecx, 2  
>    
> 
>     
>     看得出来，我们需要找的 4 点中⑷、⑸两点是最麻烦的，但是，大家看看⑷、⑸、⑹、⑺有啥关系？  
>     
>       
>     以 ⑺为基点有以下关系  
>     ◆=③  
>     ⑴=②，⑵=①，⑶=④  
>     ⑷=⑺-04，⑸=⑺+08，⑹=⑺+0C  
>       
>     那么其他的都一样的啦  
>     
>     对 VC++ 7.0 而言，以上关系是永远存在的  
>       
>     
>   
> 
> 引用:   0060CBC3 S>  6A 60             push 60                                     ; ◆  
>   0060CBC5     68 78A36900       push SungameU.0069A378                      ; ⑴  
>   0060CBCA     E8 81420000       call SungameU.00610E50                      ; ⑵  
>   0060CBCF     BF 94000000       mov edi,94  
>   0060CBD4     8BC7              mov eax,edi  
>   0060CBD6     E8 45E3FFFF       call SungameU.0060AF20                      ; ⑶  
>   0060CBDB     8965 E8           mov dword ptr ss:\[ebp-18\],esp  
>   0060CBDE     8BF4              mov esi,esp  
>   0060CBE0     893E              mov dword ptr ds:\[esi\],edi  
>   0060CBE2     56                push esi  
>   0060CBE3     FF15 88706700     call dword ptr ds:\[<&kernel32.GetVersionExA>; kernel32.GetVersionExA  
>   0060CBE9     8B4E 10           mov ecx,dword ptr ds:\[esi+10\]  
>   0060CBEC     890D 1CD86F00     mov dword ptr ds:\[6FD81C\],ecx               ; ⑷  
>   0060CBF2     8B46 04           mov eax,dword ptr ds:\[esi+4\]  
>   0060CBF5     A3 18D86F00       mov dword ptr ds:\[6FD818\],eax               ; ⑸  
>   0060CBFA     8B56 08           mov edx,dword ptr ds:\[esi+8\]  
>   0060CBFD     8915 2CD86F00     mov dword ptr ds:\[6FD82C\],edx               ; ⑹  
>   0060CC03     8B76 0C           mov esi,dword ptr ds:\[esi+C\]  
>   0060CC06     81E6 FF7F0000     and esi,7FFF  
>   0060CC0C     8935 20D86F00     mov dword ptr ds:\[6FD820\],esi               ; ⑺  
>   0060CC12     83F9 02           cmp ecx,2  
>   0060CC15     74 0C             je short SungameU.0060CC23  
>    
> 
>     
>   OK 了，Stolen Code 还原了，可以运行了  
>   Dump，Fix IAT，拿 Resource Binder 稍微优化下巴  
>     
> 目标程序打包下载，FTP:/upload/09.04.07.Themida.UnPacking.SunGame.By.wynney/  
> \--------------------------------------------------------------------------------  
>   
> 【经验总结】  
>   1、对各种编译语言的 OEP 熟悉一些的话，对修复 Stolen Code 大有帮助  
>   2、各种语言的 OEP 都有某些固定结构的，多观察下有好处：）  
>   3、在恢复 Stolen Code 的时候要学会看寄存器和堆栈  
>   
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 04 月 09 日 

 4dd466d0e1e0cc123afbe504da2462ca