

# 说说处理加密 IAT 的一种常用方法

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-497.htm`](https://www.pediy.com/kssd/pediy09/pediy09-497.htm)

> 【文章标题】: 说说处理加密 IAT 的一种常用方法  
> 【文章作者】: wynney  
> 【下载地址】: 打包下载  
> 【作者声明】: 凑热闹~^\_^  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   
>   零、前言   
>   
>   在论坛上看到很多朋友说一些教程看不懂，需要多讲一些原理，抽点空写了这篇文章，大侠飘过：）  
>   
>   一、引述   
>     
>   在讲述这个方法之前，我们先看一个实例，壳子是 CrypToCrackPeProtector 0.93 的，最后一个区段名是.ccp3p。它加密  
>   的软件的输入表是加密了的，算是一个简单的加密壳了。  
>     
>   忽略所有异常  
>     
>    
> 
>  引用:   0046B000 >  5B              pop     ebx                              ; EP  
>   0046B001    81E3 00FFFFFF   and     ebx, FFFFFF00  
>   0046B007    66:813B 4D5A    cmp     word ptr \[ebx\], 5A4D  
>   0046B00C    75 33           jnz     short 0046B041  
>   0046B00E    8BF3            mov     esi, ebx  
>    
> 
> ??   
>    
> 
>  引用:   00400000   00001000   CrypToCr              PE header     Imag   R         RWE  
>   00401000   0004A000   CrypToCr   .text      code          Imag   R         RWE  
>   0044B000   0000C000   CrypToCr   .rdata                   Imag   R         RWE  
>   00457000   00009000   CrypToCr   .data      data          Imag   R         RWE  
>   00460000   00003000   CrypToCr   .idata                   Imag   R         RWE  
>   00463000   00008000   CrypToCr   .rsrc      resources     Imag   R         RWE  
>   0046B000   00002000   CrypToCr   .ccp3p     SFX,imports   Imag   R         RW  
>    
> 
> ??   
>   我们先在.rsrc 区段 F2，F9，再在.text 段 F2，F9，就到 OEP 了  
>     
>    
> 
>  引用:   004271B0    55              push    ebp                        ; OEP  
>   004271B1    8BEC            mov     ebp, esp  
>   004271B3    6A FF           push    -1  
>   004271B5    68 600E4500     push    00450E60  
>   004271BA    68 C8924200     push    004292C8  
>   004271BF    64:A1 00000000  mov     eax, dword ptr fs:\[0\]  
>   004271C5    50              push    eax  
>   004271C6    64:8925 0000000>mov     dword ptr fs:\[0\], esp  
>   004271CD    83C4 A8         add     esp, -58  
>   004271D0    53              push    ebx  
>   004271D1    56              push    esi  
>   004271D2    57              push    edi  
>   004271D3    8965 E8         mov     dword ptr \[ebp-18\], esp  
>   004271D6    FF15 DC0A4600   call    dword ptr \[460ADC\]         ; 应该是 GetVersion，其他的 IAT 也都加密了  
>    
> 
> ??   
>   我们需要找到，IAT 加密的地方  
>     
>   重新加载目标程序，命令行 DD 460ADC，下内存访问断点，F9 几次  
>     
>    
> 
>  引用:   0046B338    893C8A          mov     dword ptr \[edx+ecx\*4\], edi     ;  中断在这，看寄存器  
>   0046B33B    807F 05 55      cmp     byte ptr \[edi+5\], 55  
>   0046B33F    73 0C           jnb     short 0046B34D  
>   0046B341    2B47 01         sub     eax, dword ptr \[edi+1\]  
>    
> 
> ??   
>    
> 
>  引用:   EAX 7C8114AB kernel32.GetVersion  
>   ECX 0000005A  
>   EDX 00460974 CrypToCr.00460974  
>   EBX 00400000 CrypToCr.00400000  
>   ESP 0013FFB4  
>   EBP 7C800000 kernel32.7C800000  
>   ESI 00460014 CrypToCr.00460014  
>   EDI 0046B823 CrypToCr.0046B823  
>   EIP 0046B338 CrypToCr.0046B338  
>    
> 
> ??   
>   可以看到 eax 里才是真正的 IAT 了，把 edi 改成 eax，然后到 00401000 段下断，F9，到达 OEP  
>   IAT 全部都恢复了：）  
>     
>     
>    
> 
>   引用:   /\*  
>   Script written by wynney  
>     
>   Date:   2007-04-13  
>   Script:CrypToCrackPeProtector Fixer  
>   Environment : OllyDbg 1.1, ODBGScript 1.52,Winxp Sp2  
>   Debugging options: Tick all items in OllyDbg's Debugging Options-Exceptions  
>     
>   Thanks :  
>            kanxue     - author of HideOD         
>            hnhuqiong  - author of ODbgScript 1.52  
>   \*/  
>   var OEP  
>   var temp  
>   var IAT  
>     
>   Start:  
>                 find eip,#2D000000805055FF5424??5A595B8B7C2404#  
>                 cmp $RESULT,0  
>                 je Error  
>                 mov temp,$RESULT  
>                 mov IAT,temp  
>                 mov OEP,temp  
>                 sub OEP,25  
>                 add IAT,12  
>                 bp IAT  
>                 esto  
>                 bc IAT  
>                 repl eip,#893C8A#,#89048A#,3  
>                 bp OEP  
>                 esto  
>                 bc OEP  
>                 sti  
>     
>   Done:  
>                 dpe "\\\\UnPacked.exe", eip  
>                 cmt eip,"OEP"  
>                 ret  
>                 
>   Error:  
>                 msg "错误"  
>                 ret  
>     
> 
>   
>     
>    二、PELock 1.0x   
>     
>   前面看了之后，大家应该明白点了，这是最基本的 IAT 加密法子，从中我们也可以看到最基本的 IAT 处理法子  
>   但是，有很多壳有壳校验或内存校验，例如：PELock  
>     
>   我们用上面的方法，再入 PELock.exe 程序，F9，Alt+E，双击目标程序所在行，Ctrl+B：FF 25  
>     
>    
> 
>  引用:   0098089F  - FF25 E4314000   jmp     dword ptr \[4031E4\]  
>   009808A5  - E9 B014A8FF     jmp     Dll\_Load.00401D5A  
>   009808AA    B0 FF           mov     al, 0FF  
>   009808AC    15 C4314000     adc     eax, 4031C4  
>   009808B1  - E9 F214A8FF     jmp     Dll\_Load.00401DA8  
>   009808B6    F2:             prefix repne:  
>   009808B7    FF15 C0314000   call    dword ptr \[4031C0\]  
>    
> 
> ??   
>   重新加载目标程序，DD 4031E4，下内存访问断点，几次 F9  
>     
>    
> 
>  引用:   00034661    8919            mov     dword ptr \[ecx\], ebx             ; 中断在这，看寄存器  
>   00034663    EB 03           jmp     short 00034668  
>   00034665    92              xchg    eax, edx  
>   00034666    E1 7C           loopde  short 000346E4  
>    
> 
> ??   
>    
> 
>  引用:   EAX 77BF27FA msvcrt.\_\_CxxFrameHandler  
>   ECX 004031E4 ASCII """:"  
>   EDX 77BE0000 msvcrt.77BE0000  
>   EBX 003B0325  
>   ESP 0013FFAC  
>   EBP 0003062D  
>   ESI 003A0426  
>   EDI 003A038F ASCII "MSVCRT.dll"  
>   EIP 00034661  
>    
> 
> ??   
>   eax 里才是真正的 IAT，ebx 里是壳的地址。  
>   那么我们把 ebx 改成 eax，F9 运行看看，出现错误：（  
>   那么我们再来看一次，看看是那里出错，改了 ebx 之后，一路 F8，F7 交替走  
>     
>    
> 
>  引用:   00034351    2B848D 143B0000 sub     eax, dword ptr \[ebp+ecx\*4+3B14\]   ; 到这就走不下去了  
>   00034358    D3C0            rol     eax, cl  
>   0003435A    49              dec     ecx  
>    
> 
> ??   
>   看来 00034351 处就是 CrC 校验的地方了  
>   我们解决他的法子有 2 个，这里说个简单的法子吧  
>     
>   同时对 00034661 和 00034351 下内存断点，运行到 00034661 后把 eax 改成 ebx，F9 运行到 00034351 马上又把 ebx 改成 eax，  
>   如此往替，所有的 IAT 就都修复完了，这个操作如果手动的确是麻烦，呵呵，用脚本吧？  
>     
>     
>    
> 
>   引用:   Compar:  
>   cmp  eip,IATAddr         //处理输入表的位置  
>   je FixIAT  
>   cmp  eip,CrCAddr        //CRC 的位置  
>   jne GotoOEP  
>   esto  
>   jmp Compar  
>     
>   FixIAT:  
>   ASM IATAddr,"mov \[ecx\],eax"   
>   sti  
>   ASM IATAddr,"mov \[ecx\],ebx"   
>   esto  
>   jmp Compar  
>     
> 
>   
>     
>    三、ASProtect 1.31   
>     
>     说下两个分支的处理  
>   
>      
> 
>  引用:     00D1574B    8B5424 0C       mov     edx, dword ptr \[esp+C\]           ; 找到这里  
>     00D1574F    8902            mov     dword ptr \[edx\], eax             ; ①ebp 里是 IAT，把 eax 改成 ebp  
>     00D15751    8B4424 0C       mov     eax, dword ptr \[esp+C\]  
>     00D15755    8906            mov     dword ptr \[esi\], eax  
>     00D15757    0FB74424 04     movzx   eax, word ptr \[esp+4\]  
>     00D1575C    0143 08         add     dword ptr \[ebx+8\], eax  
>      
> 
> ????   
>       
>      
> 
>  引用:     00D15672    8B5424 0C       mov     edx, dword ptr \[esp+C\]             
>     00D15676    8902            mov     dword ptr \[edx\], eax             ; ②ebp 里是 IAT，把 eax 改成 ebp  
>     00D15678    8D4C24 0C       lea     ecx, dword ptr \[esp+C\]  
>     00D1567C    8A5424 07       mov     dl, byte ptr \[esp+7\]  
>     00D15680    8BC3            mov     eax, ebx  
>     00D15682    E8 F9F7FFFF     call    00D14E80                         ; ③加密 Call，nop 掉  
>     00D15687    8BC6            mov     eax, esi  
>     00D15689    83E8 02         sub     eax, 2  
>     00D1568C    66:8338 00      cmp     word ptr \[eax\], 0  
>     00D15690    75 10           jnz     short 00D156A2                   ; ④改 jmp  
>      
> 
> ??   
>   IAT 处理完毕，撤消修改，以免后面出现内存保护错误  
>     
>    四、ASProtect 2.0 Alpha   
>     
>    两个分支的处理  
>   
>     
> 
>  引用:     00A8CEAD    8B5424 0C       mov     edx, dword ptr \[esp+C\]     
>     00A8CEB1    8902            mov     dword ptr \[edx\], eax      ; ①ebp 里才是 IAT，把 eax 改成 ebp  
>     00A8CEB3    8B4424 0C       mov     eax, dword ptr \[esp+C\]  
>     00A8CEB7    8906            mov     dword ptr \[esi\], eax  
>     00A8CEB9    0FB74424 04     movzx   eax, word ptr \[esp+4\]  
>     00A8CEBE    0143 08         add     dword ptr \[ebx+8\], eax  
>      
> 
> ????   
>       
>      
> 
>  引用:     00A83780    53              push    ebx                       ; ②找到这，Patch 这里  
>     00A83781    8BD8            mov     ebx, eax  
>     00A83783    8BC3            mov     eax, ebx  
>     00A83785    E8 56FFFFFF     call    00A836E0  
>     00A8378A    C603 E8         mov     byte ptr \[ebx\], 0E8       ; 这里是 IAT 加密，给改成 EB 了  
>     00A8378D    43              inc     ebx  
>     00A8378E    8903            mov     dword ptr \[ebx\], eax  
>     00A83790    5B              pop     ebx  
>     00A83791    C3              retn 3  
>      
> 
> ????   
>     自己 Patch 代码  
>     
>      
> 
>  引用:     00A83780    66:C700 FF15    mov     word ptr \[eax\], 15FF      ; Call 类型的，如果是 jmp 类型的把 15FF 改成 25FF  
>     00A83785    83C0 02         add     eax, 2  
>     00A83788    8910            mov     dword ptr \[eax\], edx  
>     00A8378A    892A            mov     dword ptr \[edx\], ebp      ; ebp 是正确的 IAT  
>     00A8378C    90              nop  
>     00A8378D    90              nop  
>     00A8378E    90              nop  
>     00A8378F    90              nop  
>     00A83790    90              nop  
>     00A83791    C3              retn  
>      
> 
> ??   
>   IAT 处理完毕，最好是撤消修改，以免后面出现内存保护错误  
>   
>    五、MoleBox   
>     
>   上面我们说的都是比较明显的加密 IAT，现在来看个不明显的  
>     
>   MoleBox 这个就不用多说了，很容易到达 OEP，我们看看 IAT 表，部分 IAT 是加密的  
>     
>    
> 
>  引用:   00406DF8  - FF25 04756300   jmp     dword ptr \[637504\]               ; flashfxp.0073BED3  
>   00406DFE    8BC0            mov     eax, eax  
>   00406E00  - FF25 00756300   jmp     dword ptr \[637500\]               ; flashfxp.0073BE34  
>   00406E06    8BC0            mov     eax, eax  
>   00406E08  - FF25 FC746300   jmp     dword ptr \[6374FC\]               ; flashfxp.0073BFCE  
>    
> 
> ??   
>   重新加载程序，DD 637504，下内存访问断点，几次 F9  
>     
>    
> 
>  引用:   00715B6F    FF96 20CB3200   call    dword ptr \[esi+32CB20\]           ; 从这里出来的  
>   00715B75    09C0            or      eax, eax  
>   00715B77    74 07           je      short 00715B80  
>   00715B79    8903            mov     dword ptr \[ebx\], eax             ; 到达这里，注意 eax  
>   00715B7B    83C3 04         add     ebx, 4  
>   00715B7E  ^ EB E1           jmp     short 00715B61  
>    
> 
> ??   
>    
> 
>  引用:   EAX 0073BED3 flashfxp.0073BED3     ；填充无效指针  
>   ECX 0013FFE0  
>   EDX 00CA09D8  
>   EBX 00637504 flashfxp.00637504  
>   ESP 0013FFA4  
>   EBP 7C800000 kernel32.7C800000  
>   ESI 00401000 flashfxp.00401000  
>   EDI 007128E0 flashfxp.007128E0  
>    
> 
> ??   
>   he 00715B6F  
>     
>   重载，F9，中断下来后 F7，跟踪发现  
>     
>    
> 
>  引用:   0073C4E9    FF75 E4         push    dword ptr \[ebp-1C\]               ; \[ebp-1C\]里才是真正的 IAT  
>   0073C4EC    8B0D 34707400   mov     ecx, dword ptr \[747034\]  
>   0073C4F2    E8 3E070000     call    0073CC35                         ; 加密 Call  
>   0073C4F7    8945 E0         mov     dword ptr \[ebp-20\], eax          ; call 会产生一个值给 eax，0 或 1  
>   0073C4FA    837D E0 00      cmp     dword ptr \[ebp-20\], 0            ; eax 再给\[ebp-20\]，再比较  
>   0073C4FE    74 08           je      short 0073C508                   ; 为 0 则不加密，为 1 则加密，Magic Jump，改 jmp  
>   0073C500    8B45 E0         mov     eax, dword ptr \[ebp-20\]          ; 加密则把值传回 eax  
>   0073C503    8B00            mov     eax, dword ptr \[eax\]             ; \[eax\]里就是无效指针，传给 eax  
>   0073C505    8945 E4         mov     dword ptr \[ebp-1C\], eax          ; 无效指针再传给\[ebp-1C\]  
>   0073C508    834D FC FF      or      dword ptr \[ebp-4\], FFFFFFFF  
>   0073C50C    E8 02000000     call    0073C513  
>   0073C511    EB 1C           jmp     short 0073C52F  
>   0073C513    A1 34707400     mov     eax, dword ptr \[747034\]  
>   0073C518    8945 D4         mov     dword ptr \[ebp-2C\], eax  
>   0073C51B    837D D4 00      cmp     dword ptr \[ebp-2C\], 0  
>   0073C51F    74 0D           je      short 0073C52E  
>   0073C521    8B45 D4         mov     eax, dword ptr \[ebp-2C\]  
>   0073C524    83C0 10         add     eax, 10  
>   0073C527    50              push    eax  
>   0073C528    FF15 18F07300   call    dword ptr \[<&KERNEL32.LeaveCriti>; ntdll.RtlLeaveCriticalSection  
>   0073C52E    C3              retn  
>   0073C52F    8B45 E4         mov     eax, dword ptr \[ebp-1C\]          ; IAT 或无效指针再传给 eax  
>   0073C532    8B4D F0         mov     ecx, dword ptr \[ebp-10\]  
>   0073C535    64:890D 0000000>mov     dword ptr fs:\[0\], ecx  
>   0073C53C    5F              pop     edi  
>   0073C53D    5E              pop     esi  
>   0073C53E    5B              pop     ebx  
>   0073C53F    C9              leave  
>   0073C540    C2 0800         retn    8                                ; 返回后把 eax 的值传给 IAT 表地址  
>    
> 
> ??   
>   我们把 Magic Jump 改成 jmp 之后 IAT 不再加密：）  
>   这里需要说明的是，这个法子跟我原先的文章里的法子是不一样的，但是可以达到相同的效果  
>     
>   于是，我们可以写个脚本  
>     
>     
>    
> 
>   引用:   /\*  
>   Script written by wynney  
>     
>   Date:   2007-04-13  
>   Script:MoleBox  
>   Environment : OllyDbg 1.1, ODBGScript 1.52,Winxp Sp2  
>   Debugging options: Tick all items in OllyDbg's Debugging Options-Exceptions  
>     
>   Thanks :  
>            kanxue     - author of HideOD         
>            hnhuqiong  - author of ODbgScript 1.52  
>   \*/  
>   #log  
>   var temp  
>     
>   log eip  
>   STI  
>   STO  
>   mov temp,esp  
>   bphws temp,"r"                //esp 定律,用来找 OEP  
>   esto  
>   bphwcall  
>   find eip,#8945??837D??0074088B45??8B008945??834D??FF#  
>   cmp $RESULT,0  
>   je err  
>   mov temp,$RESULT  
>   repl temp,#8945??837D??0074088B45??8B008945??834D??FF#,#8945??837D??00EB088B45??8B008945??834D??FF#,21  
>   sti  
>   sti  
>   mov temp,eax  
>   bp temp  
>   esto  
>   bc temp  
>   sti  
>   mov temp,esp  
>   bphws temp,"r"   
>   esto  
>   bphwcall  
>   mov temp,eip  
>   add temp,0D  
>   bp temp  
>   esto  
>   bc temp  
>   sti  
>   dpe "\\\\UnPacked.exe", eip  
>   cmt eip, "This is the OEP!  Found By: wynney "    
>   ret  
>     
>   err:  
>   msg "error"  
>   ret   
>     
> 
>   
>     
> \--------------------------------------------------------------------------------  
> 【经验总结】  
>   1、若有的壳有内存访问断点校验，可以换用内存写入断点，或者是硬件访问/写入断点  
>   2、如果不是很明显的寄存器填充，那么一般 IAT 加密就在附近，多跟踪下：）  
>   3、很多时候，并不是单纯的修改可以解决的，需要自己 Patch 代码，多了解下汇编吧，这个是别人教不会的  
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 04 月 14 日 wynney 

 4dd466d0e1e0cc123afbe504da2462ca