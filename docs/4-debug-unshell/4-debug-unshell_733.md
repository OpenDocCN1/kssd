

# Krypton v0.3

> 原文：[`www.pediy.com/kssd/pediy03/forum1161.htm`](https://www.pediy.com/kssd/pediy03/forum1161.htm)

Krypton v0.3 加的壳就是一大堆的   
PUSH ECX   
JMP xxxxxxxx   
POP ECX   
JMP xxxxxxxx   
令人看到就烦, 还有就是还原程序除了 call API 外极少见到 call, ret 指令,   
我们的 F12 就没用武之地, 想偷懒都不行, 要找到 OEP 还真的不容易.   
尽管已脱过该壳, 还没有想到十分有效, 快捷找 OEP 的方法. 有都是要按 F8 很久才找到.   
要是从程序入口开始看, 程序先是找 kernel32.dll 的 ImageBase(exe 中不用 Import 的方法),   
再找 GetProcAddress 的地址, 好“长”的一段程序(一大堆花指令), 然后就.....(烦!)   
  
下面说说怎么找 OEP, 加壳程序一定要还原原程序吧, 一般程序的 imagebase 是 400000,   
section alignment 是 1000, 我们可以 bpmb cs:401001 w (为什么不是 401000 ? 第一个   
一般是比较特殊的, 我们就用第二个), 然后 g, 看 g 了多少次原程序就运行, 就从最后   
一次看起, 跳出该还原程序, 我看到的是:   
dec ecx   
jnz continue\_restore  ; 还没有还原完就跳   
jmp xxxxxxxx   
  
之后是:   
dec esi  ; 还剩的 section 数   
jnz continue\_restore\_next\_section  ; 还没有还原完所有的 section 就跳   
jmp xxxxxxxx   
  
经过一阵的 F8 后来到:   
016F:006F25D0  CMP      WORD \[EBP+00406596\],FEEB   
016F:006F25D9  JNZ      NEAR 006F086E   
016F:006F25DF  MOV      \[EBP+0040764C\],EAX   
016F:006F25E5  MOV      DWORD \[EBP+00407650\],00   
016F:006F25EF  CMP      DWORD \[EBP+0041CD81\],BYTE +00   
016F:006F25F6  JNZ      006F261E   
016F:006F25F8  MOV      EDX,\[EBP+00407665\]   
016F:006F25FE  ADD      EDX,\[EBP+00407660\]  ; EDX = OEP   
016F:006F2604  MOV      \[EBP+0041CD81\],EDX   
016F:006F260A  MOV      EDX,004077ED   
016F:006F260F  ADD      EDX,EBP   
016F:006F2611  PUSH    EDX   
016F:006F2612  PUSH    DWORD \[WORD FS:00\]   
016F:006F2618  MOV      \[WORD FS:00\],ESP   
016F:006F261E  MOV      EDX,\[EBP+00407660\]   
016F:006F2624  MOV      EAX,\[ESI\]   
016F:006F2626  OR      EAX,EAX   
016F:006F2628  JNZ      006F262D   
016F:006F262A  MOV      EAX,\[ESI+10\]   
016F:006F262D  ADD      EAX,EDX   
016F:006F262F  ADD      EAX,\[EBP+00407650\]   
016F:006F2635  MOV      EBX,\[EAX\]   
016F:006F2637  MOV      EDI,\[ESI+10\]   
016F:006F263A  ADD      EDI,EDX   
016F:006F263C  ADD      EDI,\[EBP+00407650\]   
016F:006F2642  TEST    EBX,EBX   
016F:006F2644  JZ      NEAR 006F3016      ; 处理完 Import table 就跳   
016F:006F264A  TEST    EBX,11000000   
  
  
016F:006F376B  MOV      BYTE \[EBX\],00      ; 清空运行过的代码   
016F:006F376E  INC      EBX   
016F:006F376F  CMP      EBX,ECX   
016F:006F3771  JNZ      006F376B   
016F:006F3773  SUB      EAX,EAX   
016F:006F3775  MOVZX    ECX,WORD \[ESI+EDX+06\]   
016F:006F377A  SHL      EAX,05   
016F:006F377D  SHL      ECX,03   
016F:006F3780  MOV      \[EAX\],ECX      ; Raise exception!   
  
dd fs:0   
37BF:0000      0065FE08 xxxxxxxx xxxxxxxx xxxxxxxx   
  
dd ds:65FE08   
0177:0065FE08  0065FF68 006F3825 xxxxxxxx xxxxxxxx   
  
u cs:6F3825   
016F:006F3825  MOV      EAX,\`DOSMGR\_BackFill\_Allowed\`   
016F:006F382A  MOV      ESP,\[EAX\]   
016F:006F382C  POP      DWORD \[WORD FS:00\]   
016F:006F3832  CALL    006F3837   
016F:006F3837  POP      EBP   
016F:006F3838  SUB      EBP,004077FF   
016F:006F383E  CMP      BYTE \[EBP+0040948A\],FF  ; K-Execution ?   
016F:006F3845  JZ      006F3858                ; jump when yes ?   
016F:006F3847  MOV      ECX,\[EBP+0041CD7A\]   
016F:006F384D  MOV      \[ESP\],ECX   
016F:006F3850  MOV      EDX,\[EBP+0041CD81\]   
016F:006F3856  JMP      EDX                    ; jump to OEP   
016F:006F3858  CALL    006F385D   
016F:006F385D  POP      EBP   
016F:006F385E  SUB      EBP,00407825   
016F:006F3864  JMP      SHORT 006F38A4   
  
如果是 K-Execution 则将 FF1500000000 替换为 FF15xxxxxxxx :   
即 CALL NEAR \[00000000\]  -->  CALL NEAR \[xxxxxxxx\], 用来动态还原代码.   
  
跟着是将 CALL NEAR \[00000001\] 替换为 ? (不知道:) (难道是 K-Loader?)   
  
好, 先 bpx OEP, dump file, 再用 Import REConstructor Get Import, 可以得到 Import table 的 RVA,   
但不知道是哪个 API.   
如:  rva:00006000 ptr:0067001F   
  
u 67001F   
016F:0067001F  ADD      DWORD \[0067003A\],75188466  ;\[0067003A\] --> 4AE04B7A   
016F:00670029  MOV      EAX,\[0067003A\]            ;eax = BFF8CFE0 SetHandleCount   
016F:0067002E  SUB      DWORD \[0067003A\],75188466   
016F:00670038  JMP      EAX   
  
这下可就有点麻烦了, 怎样才能重建 Import table 呢? 最初是想用程序来把上面的代码替换为   
JMP BFF8CFE0 的, 但经过细看其重定向过程, 发现只需改一下该过程, 就可以直接把 BFF8CFE0   
放到 rva:00006000 里去. 先简单说一下其重定向过程如下:   
  
eax = GetProcAddress   
pusha   
生成 016F:016F:0067001F 处的代码   
popa   
mov eax, \[ebp+xxxxxxxx\]  ; \[ebp+xxxxxxxx\] ＝ 0067001F , 将这句 nop 掉, eax 就是 BFF8CFE0, 就这样搞定 Import table 的问题   
mov \[406000\], eax   
  
  
跟着就是还原 K-Execution 的 call 为 push 代码:   
K-Execution 的描述请看上面的贴, 这里不重复了. 加壳程序是靠返回地址来动态还原代码的,   
而且都是用同一个地址的还原程序(上面所说的 CALL NEAR \[00000000\]  -->  CALL NEAR \[xxxxxxxx\],   
而 CALL NEAR \[00000001\]  -->  ? 的, 因为我加壳后的程序没有, 所以暂且忽律, 我想是同理).   
我们可以写一程序, 在还原后的程序里查找 CALL NEAR \[xxxxxxxx\], 机器码是 FF15xxxxxxxx,   
先 push 我们要其返回的地址, 再将找到后的地址加 6(push ... 为 6 个字节)入栈,   
再跳到\[xxxxxxxx\]让其来还原, 呵呵, 我们就像是坐收渔人之利.   
当然其还原程序还要作些修改, 就是只让其还原我们想要的 push ..., push 之后的指令不让其改动, 就是:   
MOV WORD \[ECX+06\], 15FF   
MOV \[ECX+08\], EDX   
把这两句 nop 掉(幸好这里没有自检), 最后在 ret 指令前, 将我们 push 进去的查找地址出栈后才执行 ret 指令回到到我们写的程序继续执行.   
  
程序如下, 写得很丑, 见笑了.   
要是直接用汇编写就更好, 最后就是把这段程序搬到被加壳的程序里, 放哪里? 不用说了吧:)   
  
void GetDumpAddr(LPVOID pmapbase, DWORD dwSize, unsigned char \*restore\_proc)   
{  //假设我们的 pmapbase=0x401000, restore\_proc 为还原程序地址   
  unsigned long i, fixaddr;   
  for(i=0;i<dwSize-6;)   
  { //我这里查找 FF15A78D7000   
    if (\*(DWORD \*)((unsigned char\*)pmapbase+i) == 0x8da715ff && \*(DWORD \*)((unsigned char\*)pmapbase+i+2) == 0x00708da7)   
    {   
      fixaddr = (unsigned long)(unsigned char\*)pmapbase + i + 6;   
      \_\_asm push offset conti  //运行完还原程序的返回地址, 实际搬到被加壳程序空间后看看要不要改    
      \_\_asm push fixaddr      //找到的地址   
      \_\_asm jmp restore\_proc  //跳到还原程序   
conti:   
      i+=6;   
    }   
    else   
      i++;   
  }   
  return;   
} 

不知为何我用来试加壳的程序没有 CALL NEAR \[00000001\], 后来加 notepad.exe 就有了.   
还原改代码的方法也很简单, 因为该 call 最后的指令是 jmp near \[eax\], 跳到由加壳生成的重定向地址去,   
而此时的 eax 就是原程序要 call 的地址, 我们可以在该 jmp 前写   
mov ebx, dword ptr \[esp\]   
sub ebx, byte 4   
mov dword ptr \[ebx\], eax   
ret   
再用上面贴的方法还原.   
  
的确, 该加壳方法要是在加壳时没有检测原程序有否 FF1500000000 和 FF1501000000 时, 还原时就有可能出错.   
  
  
更正上面贴的一点错:   
  
\>>016F:006F2644  JZ      NEAR 006F3016      ; 处理完 Import table 就跳   
  
应该是处理完一个 dll 时跳, 判断处理完整个 Import table 不是这里.  4dd466d0e1e0cc123afbe504da2462ca