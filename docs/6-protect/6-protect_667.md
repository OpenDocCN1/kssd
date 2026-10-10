

# Code Virtualizer 虚拟机代码还原

> 原文：[`www.pediy.com/kssd/pediy12/129772.html`](https://www.pediy.com/kssd/pediy12/129772.html)

【文章标题】: Code Virtualizer 虚拟机代码还原  
【作者邮箱】: machinesy@gmail.com  
【软件名称】: Code Virtualizer  
【下载地址】: 自己搜索下载  
【加壳方式】: CV  
【保护方式】: VM  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  http://bbs.pediy.com/showthread.php?t=124732 blueapplez 的这篇一定读读  
  http://bbs.pediy.com/showthread.php?t=121412 理想哥的篇也一定要读读  
  如果你读懂了，那么下面的东西就是浮云了。  
    
    
  我先写了断汇编代码：  
    VIRTUALIZER\_START  
    invoke GetModuleHandle,NULL  
    mov hInstance,eax  
    xor eax,eax  
    invoke ExitProcess,0  
    VIRTUALIZER\_END  
  然后我下载了 CV 虚拟机在这里  
  Code Virtualizer 1.3.1.0new  
  http://www.pediy.com/tools/packers.htm  
    
  然后我用这个虚拟机给它加了个壳。  
    
    
  CV 虚拟机做的相当的单纯，单纯到虚拟机就是虚拟机这样的境界。  
    
  EDI-----虚拟机寄存器寄存器  
  ESI-----虚拟机 EIP 寄存器  
    
  004052E8    5F              pop     edi  
  004052E9    81EF E8524000   sub     edi, 004052E8  
  004052EF    8BC7            mov     eax, edi  
  004052F1    81C7 00504000   add     edi, 00405000  
  004052F7    3B47 2C         cmp     eax, dword ptr \[edi+2C\]  
    
  初使 EDI  
    
    
  0040532C    8B7424 24       mov     esi, dword ptr \[esp+24\]  
  00405330    8BDE            mov     ebx, esi  
  00405332    03F0            add     esi, eax  
  00405334    B9 01000000     mov     ecx, 1  
    
    
  初始 ESI  
    
    
  00405340  ^\\75 F7           jnz     short 00405339  
  00405342    AC              lods    byte ptr \[esi\]  
  00405343    E9 A8090000     jmp     00405CF0  
    
  读取第一个字节码，然后解析，然后到 VM\_Dispatch  
    
  004057E0   /E9 64210000     jmp     00407949  
  004057E5   |FF2487          jmp     dword ptr \[edi+eax\*4\]            ; GUI.004098A8  
  004057E8   |61              popad  
  004057E9   |C3              retn  
    
    
  VM\_Dispatch  
    
    
    
    
  VM 虚拟机代码如果想看的不清不楚的话，就可以只关系以下一些东西。EDI,ESI,VM\_Dispathc,VM\_PUBLIC.  
  VM\_PUBLIC 是我自己取的名字，实际上它是一个分界线。每一个 VM\_HANDLE 执行完后，都会先走到 VM\_PUBLIC,然后再回到 VM\_Dispatch，  
  我们可以使用 IDA,直接不停的命名过程函数，找到 VM\_PUBLIC..其实人肉看，也很容易看出来。我建议使用 IDA。  
    
    
  我们在 VM\_Dispatch 下段，来看每一个 VM\_HANDLE 的执行流程。如下：  
    
  CV 虚拟机大多数运算还是依然使用原始 ESP，下面我们进入第一个 VM\_HANDLE  
  004098A8    AC              lods    byte ptr \[esi\]     VM\_PLUS\_MEMREGADDR  
  004098A9    52              push    edx  
  004098AA    B6 DB           mov     dh, 0DB  
  004098AC    28F0            sub     al, dh  
  004098AE    5A              pop     edx  
    
  这个 HANDLE 其实就就是 PLUS 一个内存里边模拟的寄存器地址到 ESP。打个比方说，VM\_EAX 的内存地址是 0x40501c,那么虚拟机的执行过程就是  
  先从 ESI 取出着个地址，然后 plus 到 esp.随后进行下边这个操作  
    
    
    
  00405A6D    FF3424          push    dword ptr \[esp\]                  ; GUI.0040501C  VM\_POP\_ESP\_TOEDX  
  00405A70    5A              pop     edx  
  00405A71    55              push    ebp  
  00405A72    89E5            mov     ebp, esp  
  00405A74    81C5 04000000   add     ebp, 4  
  00405A7A    81C5 04000000   add     ebp, 4  
  00405A80    E9 B30F0000     jmp     00406A38  
  00405A85    05 2609D901     add     eax, 1D90926  
    
  刚刚 plus 的那个地址，到 EDX  
    
  随后进行下边这个操作  
    
  00405E70    8F02            pop     dword ptr \[edx\]  VM\_POPDATE to EDX  
  00405E72  ^ E9 CBF4FFFF     jmp     00405342  
  00405E77    50              push    eax  
  00405E78    B8 A64F0972     mov     eax, 72094FA6  
    
  弹出一个 ESP 到 EDX。这三句其实就构成了一个真汇编的 MOV XXX.  
  这三句的是这样的  
    
    
  VM\_PLUS\_MEMREGADDR  
  VM\_POP\_ESP\_TOEDX  
  VM\_POPDATE   
    
  程序一开始就先执行了这样 8 个这样的流程，就像是在巴 pushad 的数据全部复制到内存里边的模拟寄存器里。  
    
    
  大家脑袋里一定要明确下边这几个概念：  
  1.  
  EDI-----虚拟机寄存器寄存器  
  ESI-----虚拟机 EIP 寄存器  
  2.  
  CV 的保护没有 VM 那么强大，十分单纯，很多运算还是直接观察 ESP 就能得到结论的。所以要使用这 Lock ESP,观察 ESP。  
  3.  
  如果你没有必要全部分析清楚的话，你不用去还原每一个 VM\_HANDLE,直接看 ESP，看 EDI，ESI 你就能明白程序的大概了。  
    
  未完待续..........  
    
    
    
    
  我刚刚才看到 softworm 已经说的很清楚了...所以我就不再继续写这个帖子了。慢点我去学习他的。  
    
    
    
    
    
    
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2011 年 02 月 23 日 19:27:51

上传的附件
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || GUI.zip  
 |