

# exploit\_me\_A 的 shellcode 构造与突破

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-360.htm`](https://www.pediy.com/kssd/pediy09/pediy09-360.htm)

> 这次 exploit\_me 本来很感兴趣的，可是好像日子选太好了啊![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ，都是聚会，结果 B 就没有去做了，A 也就没必要提交了，偶的书啊。  
>   
>       其实我对 shellcode 并没有什么研究，也就马马虎虎跟过一个 ani 漏洞的 shellcode，就一点皮毛，讲的不对，还望多多指正。  
>   
>       先来看看 exploit\_me\_A 溢出发生的函数  
> 
> 代码:
> 
> ```
> @exploit\_\_00401000:                          ;<= Procedure Start
> 
>         sub     esp, 0C8h
>         or      ecx, 0FFFFFFFFh
>         xor     eax, eax
>         lea     edx, dword ptr \[esp\]
>         push    esi
>         push    edi
>         mov     edi, dword ptr \[esp+0D4h\]
>         push    040904Ch         ; ASCII "\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"
>             
> ; 源字符串计数，不能含有 null 字符，否则即被截断
>         repne   scas byte ptr es:\[edi\]       ; edi=pszRecv 
>         not     ecx                          ; nRecvLen
>         sub     edi, ecx                    
>         mov     eax, ecx
>         mov     esi, edi
>         mov     edi, edx
>         shr     ecx, 2
>         rep     movs dword ptr es:\[edi\], dword ptr \[esi\]
>         mov     ecx, eax
>         and     ecx, 3
>         rep     movs byte ptr es:\[edi\], byte ptr \[esi\]
>         ……
> 
> ```
> 
> 用 hexrays 方便的可以看到 c 代码是  
> 代码:
> 
> ```
> int exploit\_\_00401000(const char\* pszRecv)
> {
>   char szRecv\[200\];
>   strcpy(szRecv,pszRecv);
> ……
> }
> 
> ```
> 
> 明显，pszRecv 没有被经过边界检查就拷给了 szRecv,而 szRecv 仅仅只有 200 bytes,当 pszRecv 中长度大于 200 时，即导致溢出发生  
> 更正  
> 可以确定溢出的条件：  
> pszRecv 必须大于 200 字节才能溢出，且不能含 null 字符  
>   
> 构造 shellcode  
>   
> 21 世纪，shellcode 什么最重要－－通用性，地球人都知道  
> 以下言论可能直接导致众人笑偶门外汉。  
> shellcode 其实通用性非常局限，无论是暴力搜索 kernel32 base 还是 peb 取 kernel32 base，最后还是要借助 jmp ebx 或者 jmp esp，那就不得不硬编码，windows 补丁如此最多。。。通用是不是有局限了呢（偶的半 sp2 中 kernel32 基地址就是 0x4FAD0000）。  
>   
> 偶要求不是太高，有没有可以在 xp 全系列里通用的 jmp esp 或 jmp ebx 地址呢（包括任意不完全补丁的 xp）？可能会有，但是目前为止还没有被统计出来吧，xp 补丁无穷技术就是最好的 shellcode 防火墙![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
>   
> 要在 xp 全系列通用？开始有人要嘘了，还好，针对这个特定的 exploit\_me\_A，确实可以做到！  
>   
> 偶们知道 exe 文件基地址在编译时就是指定的，默认是 0x400000，不象 dll，exe 基地址在运行是也不会改变(说的有点心虚，高手快来指正)，那 exe 文件中某个 retn 的地址是不是就是确定的呢  
>   
> 哈哈，好了，我们回顾下溢出发生的函数  
> 代码:
> 
> ```
> int exploit\_\_00401000(const char\* pszRecv)
> {
>   char szRecv\[200\];
>   strcpy(szRecv,pszRecv);
> ……
> }
> 
> ```
> 
> pszRecv，就是他  
> “伟大的 Recv 指针，他继承了指针的光荣的传统，p－s－z－R－e－c－v，这一刻他灵魂附体！在这一刻！他不是一个人在战斗！他不是一个人！！”  
> 我们可以放弃 esp，放弃 ebx，放弃 abdce 了！  
>   
> 当这个溢出函数被调用时，堆栈为  
> 代码:
> 
> ```
> Esp-> return Address
>           pszRevc
>           ……
> 
> ```
> 
> ![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif") ，只要将 return Address 覆盖为 exploit\_me\_A 中任意一个 retn 地址！就控制 eip 流向 shellcode 了！  
>   
> 按照前面的利用方法，shellcode 被构造成  
>   
> < 200 bytes ><4 bytes>  
> | shellcode1   |   pRetn  |  
>   
> retn 就选溢出发生函数 function\_00401000 的好了   
> 代码:
> 
> ```
> 00401000   sub     esp, 0C8
> ......
> 004010A4   retn
> 
> ```
> 
> retn 地址最高字节为 00，这样 shellcode 为  
> < 200 bytes ><4 bytes>  
> | shellcode1   |A4104000|  
>   
> 遇 0 截断，实际 shellcode 只有 203 字节  
>   
> 幸好，function\_00401000 调用处只有一个，覆盖的返回地址始终为 0x00401255，最高字节亦始终为 00  
>   
> 所以不怕返回地址最高字节没有覆盖到而导致控制 eip 失败。  
>   
> 代码:
> 
> ```
> 0040124B   lea     edx, dword ptr \[esp+34\]         ;  loc\_40124B
> 0040124F   push    edx
> 00401250   call    00401000
> 00401255   add     esp, 4
> ......
> 
> ```
> 
> 留下一个问题，实际可执行的 shellcode 被限制成了 200 字节,因为不能覆盖 pszRevc。  
>   
>   
>   
> 突破 200 字节限制  
>   
> 向上查找红色句子－－溢出的限制条件－－pszRecv 必须大于 200 字节才能溢出，且不能含 null 字符  
>   
> 把限制条件转换为利用条件吧  
>   
> 再回顾下溢出发生的函数  
> 代码:
> 
> ```
> int exploit\_\_00401000(const char\* pszRecv)
> {
>   char szRecv\[200\];
>   strcpy(szRecv,pszRecv);
> ……
> }
> 
> ```
> 
> shellcode 被执行实在 pszRecv 指向的堆栈中，而导致溢出覆盖的是在 szRecv 指向的堆栈中，  
> 所以，只要从 pszRecv 拷到 szRecv 的 shellcode 满足溢出长度即可  
>   
> 这样，可以把把 shellcode 构造成这样  
>   
> <         200 bytes      > < 4 bytes > <512-204bytes>  
> |shellcode1...jmp $+4 | A4104000 |     shellcode2    |  
>   
> 好了，突破可执行 shellcode 200 字节限制了  
>   
> 代码:
> 
> ```
> 0040120B   push    eax                             ; /Flags => 0
> 0040120C   rep     stos dword ptr es:\[edi\]         ; |
> 0040120E   lea     ecx, dword ptr \[esp+38\]         ; |
> 00401212   push    200                             ; |BufSize = 200 (512.)
> 00401217   push    ecx                             ; |Buffer
> 00401218   push    ebx                             ; |Socket
> 00401219   call    dword ptr \[<&WS2\_32.#16>\]       ; \\recv
> ...
> 
> ```
> 
> 现在，shellcode 扩展到 512 字节了，这个长度可以做很多事了  
>   
> 512 能不能突破？留给大家玩了  
>   
> 代码:
> 
> ```
> 附件里的 shellcode 我直接用了导入表
> 名称位于 exploit\_, 条目 10
>  地址=00408024
>  区段=.rdata
>  类型=输入    (已知)
>  名称=KERNEL32.LoadLibraryA
> 
> 名称位于 exploit\_, 条目 11
>  地址=00408028
>  区段=.rdata
>  类型=输入    (已知)
>  名称=KERNEL32.ExitProcess
> 
> 名称位于 exploit\_, 条目 46
>  地址=004080B4
>  区段=.rdata
>  类型=输入    (已知)
>  名称=KERNEL32.GetProcAddress
> 
> ```

 4dd466d0e1e0cc123afbe504da2462ca