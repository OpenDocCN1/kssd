

# 学习 VMP 1.22 的一点心得体会

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-566.htm`](https://www.pediy.com/kssd/pediy09/pediy09-566.htm)

> 致谢:  
> bughoho,forgot,shoooo  
>   
> 试验程序用的 max speed+的 VMP.  
>   
> VMP 是堆栈机。VMP 使用真堆栈,没有 V\_Stack  
> 什么是堆栈机,简单的理解就是一切的指令,中间结果,数据"放"在堆栈.准确的说应该是通过堆栈关联.  
> 要准确的理解可以参考这本书.[<<Stack Computer>>](http://www.ece.cmu.edu/~koopman/stack_computers/)  
>   
> 由于 VMP 是堆栈机所以指令系统和运作方式不太像我们熟悉的 X86 系统.对分析造成一些障碍.  
>   
>   
> VMP 的入口代码:  
>   
> 最开始是两个 push imm32 jmp 的组合,依次压入解码的 Key 和 PCODE 的首地址(V\_PCODESEG)  
> 然后是下面的代码.    
> (nextpcode 不是很确切.因为只有个别 handler 才 jmp 回 nextpcode,大部分情况时 handler 里面自己+V\_EIP)  
>   
> vmp0:0040304C                                   public VM\_INIT  
> .vmp0:0040304C                   VM\_INIT:                                ; CODE XREF: start+2824j  
> .vmp0:0040304C 51                                push    ecx  
> .vmp0:0040304D 55                                push    ebp  
> .vmp0:0040304E 53                                push    ebx  
> .vmp0:0040304F 56                                push    esi  
> .vmp0:00403050 51                                push    ecx  
> .vmp0:00403051 57                                push    edi  
> .vmp0:00403052 52                                push    edx  
> .vmp0:00403053 50                                push    eax  
> .vmp0:00403054 9C                                pushf  
> .vmp0:00403055 68 00 00 00 00                    push    0  
> .vmp0:0040305A 8B 74 24 28                       mov     esi, \[esp+28h\]  ; VM\_CODESEG=004037B2  
> .vmp0:0040305E BF 00 30 40 00                    mov     edi, offset VM\_Context  
> .vmp0:0040305E  
> .vmp0:00403063  
> .vmp0:00403063                   VM\_NextPCode:                           ; CODE XREF: V\_JMP+1j  
> .vmp0:00403063 89 F3                             mov     ebx, esi        ; ebx:previous\_pcode 的地址  
> .vmp0:00403065 03 34 24                          add     esi, \[esp\]      ; esi:指向 current\_pcode  
> .vmp0:00403065                                                           ;  
> .vmp0:00403065                                                           ; 首次进入 esi=ebx  
> .vmp0:00403065  
> .vmp0:00403068  
> .vmp0:00403068                   Vm\_Execute:                             ; CODE XREF: V\_ADD\_BYTE+6j  
> .vmp0:00403068                                                           ; V\_PUSH\_ESP+1j  
> .vmp0:00403068                                                           ; V\_LOAD\_BYTE\_CONTEXT+16j  
> .vmp0:00403068                                                           ; V\_LOAD\_BYTE\_BY\_ADDR\_DS+7j  
> .vmp0:00403068                                                           ; V\_ADD\_DWORD+5j  
> .vmp0:00403068                                                           ; V\_POP\_SP+2j ...  
> .vmp0:00403068 8A 0E                             mov     cl, \[esi\]       ; cl=cipher\_pcode  
> .vmp0:0040306A 00 D9                             add     cl, bl  
> .vmp0:0040306C C0 C9 05                          ror     cl, 5  
> .vmp0:0040306F F6 D9                             neg     cl  
> .vmp0:00403071 46                                inc     esi             ; esi:指向数据，或者下一个 pcode  
> .vmp0:00403072 80 E9 52                          sub     cl, 52h  
> .vmp0:00403075 D0 C1                             rol     cl, 1  
> .vmp0:00403077 FE C1                             inc     cl  
> .vmp0:00403079 80 F1 78                          xor     cl, 78h  
> .vmp0:0040307C 80 C1 19                          add     cl, 19h  
> .vmp0:0040307F 00 CB                             add     bl, cl  
> .vmp0:00403081 0F B6 C1                          movzx   eax, cl  
> .vmp0:00403084 8D 14 85 89 32 40+                lea     edx, VM\_HANDLER\_TABLE\[eax\*4\]  
> .vmp0:0040308B FF 22                             jmp     dword ptr \[edx\]  
>   
>   
> 上面就是 VMP 的主循环.  
> 在主循环和 handler 中 esi 就是 V\_EIP edi 就是指向 V\_CONTEXT.ebx 保持一个 key.  
> 看见看出 VMP 的解码和执行流程有关.解码有一定难度.直接修改 PCODE 好像也很困难.  
>   
> VM\_EXECUTE 主要功能就是解码 PCODE 然后 jmp 到相应的 Handler.  
>   
> VMP 的指令:  
> 我觉得 VMP 的指令如果用类似浮点指令的方式命名可能会容易理解一些.  
> FPU 貌似也是堆栈机(我看来是,也不清楚具体的内部结构).  
> 比如 fld fst 之类的指令.  
> 没有 fmov 之类的指令.因为 FPU 处理任何指令.都是通过浮点堆栈取数据.  
>   
> VMP 也类似.  
> 在我自己的+的小试验程序里面,确实也没有发现 V\_MOV 类似的 handler.不知道 http://bbs.pediy.com/showthread.php?t=54535 中的 V\_MOV 是什么样子的 handler.  
> 希望能和 wangdell 请教一下.  
>   
> 可能是我的程序太小,只有 4 行指令  
> push 0  
> push imm32  
> push imm32  
> call    MessageBox  
> 是不是 VMP 没有生成相关的 handler?  
>   
>   
> 识别 handler 的方法我是连蒙带猜.  
>   
> 不过可以通过最开始的 PCODE 快速的确定 V\_STORE\_CONTEXT\_DWORD 的 Handler  
> 我通过 OD 记录执行的 Handler 的地址.然后根据手动还原的表.打出了一份 VMP 保护  
> push 0  
> push imm32  
> push imm32  
> Call   MesssageBox 的 PCODE  
> 如下: (还没有记录 PCODE 的参数).能大概熟悉一下指令系统.STORE 类指令就是把数据存入某地 (CONTEXT,内存地址).LOAD 类是把数据从某地 push 到堆栈上.  
>   
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE\_CODESEG  
> V\_STORE\_CONTEXT\_WORD  
> V\_STORE\_CONTEXT\_WORD  
> V\_SHL\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE\_CODESEG  
> V\_STORE\_CONTEXT\_WORD  
> V\_STORE\_CONTEXT\_WORD  
> V\_SHL\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STROE\_WORD\_CONTEXT  
> V\_PUSH\_ESP  
> V\_LOAD\_BYTE2DWORD\_CODESEG  
> V\_ADD\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_WORD\_BY\_ADDR\_SS  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE2DWORD\_CODESEG  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_ADD\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_PUSH\_ESP  
> V\_LOAD\_DWORD\_BY\_ADDR\_SS  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_NOTA\_AND\_NOTB\_0  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_DWORD\_CODESEG  
> V\_NOTA\_AND\_NOTB\_0  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_DWORD\_CODESEG  
> V\_NOTA\_AND\_NOTB\_0  
> V\_STORE\_CONTEXT\_DWORD  
> V\_NOTA\_AND\_NOTB\_0  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE2DWORD\_CODESEG  
> V\_ADD\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE\_CODESEG  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_SHLD\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_BYTE2DWORD\_CODESEG  
> V\_ADD\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_JMP  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
> V\_STORE\_CONTEXT\_DWORD  
>   
> V\_LOAD\_BYTE2DWORD\_CODESEG            //push 0  
>   
> V\_LOAD\_DWORD\_CODESEG                     //push "HelloVm"  
> V\_LOAD\_DWORD\_CODESEG                     //push "HelloVm"  
>   
> V\_LOAD\_BYTE2DWORD\_CODESEG           //push 0  
>   
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_DWORD\_CODESEG  
> V\_ADD\_DWORD                                   
> V\_STORE\_CONTEXT\_DWORD  
>   
> V\_LOAD\_DWORD\_CODESEG             //这两条指令就是从 IAT 取 MessageBox 的地址。  
> V\_LOAD\_DWORD\_BY\_ADDR\_DS        //可见 IAT 地址是直接保存在 PCODE 数据里面。不知道能不能类推\[imm32\]的寻址方式  
>   
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_LOAD\_CONTEXT\_DWORD  
> V\_VM\_EXIT  
>   
> 通过 PCODE 可以大概了解 VMP 的指令体系和大致的运作方式.  
>   
> 话说静态还原：  
> A.能自动识别 Handler.这一点我觉得有难度。因为已经有混淆版 VMP 出现。可能是正版也可能私版 VM.  
> B.区分 VM 自身结构的代码和被保护的代码.这个我认为相当困难。也许是我经验不够。如果有大量的分析成果的积累。或许能找到某种模式。  
> C.逻辑表达式的自动化简。自动化简可以办到。但是要求 VMP 不打乱逻辑顺序。(目前我也不知道 VMP 有没有打乱逻辑顺序)。  
> 如只是一个萝卜一个坑的拆指令，是有办法自动化简的。  
>   
> 附件中有试验程序和 IDB 文件  
>   
> 水平有限,难免错漏,请各位斧正 

 4dd466d0e1e0cc123afbe504da2462ca