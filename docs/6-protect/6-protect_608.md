

# VMP 之反 VM (还原代码) 之初初...

> 原文：[`www.pediy.com/kssd/pediy10/94397.html`](https://www.pediy.com/kssd/pediy10/94397.html)

VMP 之还原 VMP 初探  
作者：小娃崽\[CUG\]  
  
  这应该是一篇菜鸟一看就懂，高手不用看就懂的帖子。不知道为什么，前段时间老是在想 VMP 这个东西，可能是因为太闲的原因吧。突然想到 如果虚拟了 MOV EAX,1 这个指令最终结果还会是令 EAX 这个寄存器中的数据为 1 的。这两天休息，就下了 VMP1.08 回来虚拟了几个小程序，F8 从头到尾把程序跑了一趟。累啊，以下是一点心得，先记录下来，免得以后忘记了。  
【1】总的纲领：  
  汇编中操作数主要分为三类：寄存器操作数，内存操作数，立即操作数。CPU 指令执行的最终结果还是改变寄存器或者内存中的数据。  
  比如 ：  
  MOV EAX,1   
  mov \[10000000\],eax  
  使 EAX=1，\[10000000\]=1  
  也可以变换一种形式改变 EAX 的值，  
  比如：  
  push 1  
  pop  eax  
  不管怎样，最终结果还是一样的，只是换了个形式，VMP 也一样，只是跑了 N 条等价的伪指令，晃点我们的眼睛而已。  
  
【2】VMP 是堆栈机：  
 它有自己的"寄存器",有自己的伪指令，它对数据的操作都是通过堆栈进行的，从它的伪指令可以看出，伪指令太多了，我只看了几十条，其他的就看不下去了。  
  如【1】所述，指令的结果只是改变数据.  
   pop     dx  
   pop     ax  
   pop     cx  
   div     cx  
   push    ax  
   push    dx  
   JMP     vm\_execute //字除法，并把商和余数压入堆栈 v\_divw  
  
   pop     ecx  
   add     dword ptr \[esp\], ecx  
   jmp     vm\_execute           //可以看做是 V\_ADD  
  
   lods    byte ptr \[esi\]  
   add     al, bl  
   sub     al, 0C6  
   ror     al, 1  
   not     al  
   ror     al, 3  
   add     bl, al  
   cbw  
   cwde  
   push    eax  
   jmp     vm\_execute          //可以看做是 V\_PUSH   
  
  
   pop     eax  
   pop     dword ptr \[eax\]      
   jmp     vm\_execute         //可以看做是和 MOV \[XXXXXXXX\],reg 等价的伪指令   
  
  
。。。。。。。。。。。。  
 没猜想错的话，在虚拟机内部，VMP 对数据的操作转换到了 VM\_COMTEXT，堆栈当中，只是通过伪指令进行而已。  
 在论坛里还看到前辈们总结的 ，这个或那个再或那个等于异或什么的，还画有图，应该是精华部分吧，没看，因为我觉得我看不懂，以后用的上的话在看吧。  
【3】VMP 的构造：  
    猜想进入 VMP 之前和退出 VMP 之后，堆栈的情况应该差不多的，改变不了多少。于是对 VM 入口下了硬件访问断点。  
00404313    58              pop     eax  
00404314    61              popad                     //F9 几下来到这里 和 VMP 的入口刚好匹配得上 应该就是传说中的 VM 出口了。  
00404315    9D              popfd  
00404316    C3              retn                      //还有一条差不多的，只是这里是 retnf  
   F8 按了好久，于是干脆就写了几句脚本，专门查看 EAX 的值，发现大体结构都是差不多的。  
  
VM\_START:  
00401000 > $  68 EB4C4000   push    00404CEB             //把 PCODE 压入堆栈  
00401005   .- E9 A63A0000   jmp     00404AB0  
  
VM\_EXECUTE:  
  
00404AB0    9C              pushfd  
00404AB1    60              pushad                         
00404AB2    68 00000000     push    0  
00404AB7    8B7424 28       mov     esi, dword ptr \[esp+28\]  
00404ABB    FC              cld  
00404ABC    BF 00404000     mov     edi, 00404000  
00404AC1    89F3            mov     ebx, esi  
00404AC3    033424          add     esi, dword ptr \[esp\]  
00404AC6    AC              lods    byte ptr \[esi\]  
00404AC7    00D8            add     al, bl  
00404AC9    FEC0            inc     al  
00404ACB    F6D0            not     al  
00404ACD    C0C0 07         rol     al, 7  
00404AD0    34 D3           xor     al, 0D3  
00404AD2    00C3            add     bl, al  
00404AD4    0FB6C0          movzx   eax, al  
00404AD7    FF2485 DD404000 jmp     dword ptr \[eax\*4+4040DD\]    //开始执行伪指令   
  
0040404F    AC              lods    byte ptr \[esi\]  
00404050    00D8            add     al, bl  
00404052    34 4A           xor     al, 4A  
00404054    FEC8            dec     al  
00404056    C0C8 05         ror     al, 5  
00404059    FEC8            dec     al  
0040405B    00C3            add     bl, al  
0040405D    8F0487          pop     dword ptr \[edi+eax\*4\]       //把之前 PUSH 的寄存器保存到 VM\_COMTEXT，10 次,V\_SaveToVMContext  
00404060    E9 610A0000     jmp     00404AC6  
  
。。。。。。。。。。。。。。。。。N 条伪指令  
  
00404B32    AC              lods    byte ptr \[esi\]  
00404B33    00D8            add     al, bl  
00404B35    34 4A           xor     al, 4A  
00404B37    FEC8            dec     al  
00404B39    C0C8 05         ror     al, 5  
00404B3C    FEC8            dec     al  
00404B3E    00C3            add     bl, al  
00404B40    FF3487          push    dword ptr \[edi+eax\*4\]     //把数据压入堆栈 ，10 次,V\_VMContextToStack  
00404B43  ^ E9 7EFFFFFF     jmp     00404AC6  
  
004045B6    58              pop     eax  
004045B7    61              popad  
004045B8    9D              popfd                            //再把堆栈的数据压入到真实的寄存器   
004045B9    C3              retn  
  
 N 条伪指令下来，最终执行了几条等价的汇编指令而已。以下是我自己总结的 VMP 大概构造：  
   
 Vm\_Start  
 VM\_Execute  
 V\_SaveToVMContext  
 N 条伪指令  
 V\_VMContextToStack  
 V\_RETN    
    
【4】验证与猜想：  
  回到 MOV EAX,1 这条指令上来，我就虚拟了这个指令而已，然后在 VM 的出口下断，发现最终的结果就是 EAX=1,其他寄存器与没 VM 之前的变化不大。以下是代码片段，我可是完整的 F8 几回才看到了点真相。  
    
   V\_SaveToVMContext  10 次  
   。。。。。。。。。。。。。  
   00404421    AC              lods    byte ptr \[esi\]  
   00404422    00D8            add     al, bl  
   00404424    2C C6           sub     al, 0C6  
   00404426    D0C8            ror     al, 1  
   00404428    F6D0            not     al  
   0040442A    C0C8 03         ror     al, 3  
   0040442D    00C3            add     bl, al  
   0040442F    66:50           push    ax  
   00404431    E9 57060000     jmp     00404A8D             //V\_PUSH 5  
   。。。。。。。。。。。。   
   00404B17    AC              lods    byte ptr \[esi\]  
   00404B18    00D8            add     al, bl  
   00404B1A    C0C0 07         rol     al, 7  
   00404B1D    FEC0            inc     al  
   00404B1F    34 3E           xor     al, 3E  
   00404B21    04 2D           add     al, 2D  
   00404B23    00C3            add     bl, al  
   00404B25    8F0487          pop     dword ptr \[edi+eax\*4\]  //V\_SaveToVMContext 对应的 V\_EAX 被改变  
   00404B28  ^ E9 60FFFFFF     jmp     00404A8D  
   。。。。。。。。。。。。  
   V\_VMContextToStack 10 次  
   V\_RETN    //退出 VM  
【5】还原 VMP 的一点感悟  
  知道了事情的真相，感觉人肉还原 VMP 还是可行的，需要的只是时间和毅力，N 条伪指令下来，才还原成几条汇编指令，我才不干！我想 VMP 强大的地方应该数 PUSH,POP 操作了太多次，有的负责解码，但大多是的时候我都觉得它们跟花指令差不多。而且 HANDLE 与 PCODE 可是随机组合的，同一段程序会 VM 出不同的版本。  
如果  MOV EAX,1  VM 后的结果是 ：  
     V\_SaveToVMContext    
     V\_PUSH 5  
     V\_SaveToVMContext  
     V\_VMContextToStack  
     V\_RETN      
我想还原起来可就不费力了。  
可以从 VM 的外部着手，观察和猜测这段 VM 的大体功能，像高手们说的，把它看成是一个函数，有时候不用跟进 VM 就能看出个大概了，比如一个虚拟了的 MessageBoxA,接着进入 VM 内部，注意观察 VM\_COMTEXT，堆栈的数据变化，把无效的 PUSH，POP 给剔除，精简出有效的伪指令，再还原。而且对数据操作的指令(ADD,MUL,DIV 之类的)在 VMP 内部都有等效的伪指令，而且不难看出。  
  
【总结】：  
   终于理解了什么是 VM，高兴