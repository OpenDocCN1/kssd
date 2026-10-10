

# patch

> 原文：[`www.pediy.com/kssd/pediy05/pediy50136.htm`](https://www.pediy.com/kssd/pediy05/pediy50136.htm)

其实是老生常谈。   
  
TELock 针对 API 的 anti-BPX 很容易绕过，它不让设“BPX VirtualProtectEx”， 那就设“BPX VirtualProtectEx+1”好了。其余类推。   
  
anti-BPM 如下，设一个“BPINT3”就到了。   
流程如下：   
1、首先由 int 3 引起一个软件断点异常，在异常处理程序中设置 4 个硬件断点。   
2、每个硬件调试异常激活时，在异常处理程序中统计硬件调试异常的次数。这个次数在以后解密数据时有用。在 SoftICE 中用 F8 跟到下面的地方就不能动了就是因为它设了 4 个硬件断点。   
3、除 0 异常发生时，清除 dr0～dr7 的内容，所以你在此之前设的 BPM 断点失效了。所以只要在它清除 dr0～dr7 之后再设 BPM 断点就行了。   
  
借鉴一下它的这个思路，就可以很方便地制作被加过壳的程序的 inline patch（SMC）。只要在加过壳的程序的开头某处插入类似的 SEH 结构，并在异常处理程序中针对要 patch 的地方设置硬件断点，等硬件断点激活后进行 patch 即可。既方便又通用，不管程序被加了多少层壳都可以。这等于是在进程内嵌了一个微型的 debugger，和进程外的 debugger 型的 loader 类似。   
  
TELock 的磁盘文件 CRC 校验用“bpx CreateFileA+1”搞定，内存 CRC32（可用来 anti-BPX）也很容易跟到。   
  
001B:005BD07F  POP      EBP   
001B:005BD080  LEA      EAX,\[EBP+46\]   
001B:005BD083  PUSH      EAX   
001B:005BD084  XOR      EAX,EAX   
001B:005BD086  PUSH      DWORD PTR FS:\[EAX\]   
001B:005BD089  MOV      FS:\[EAX\],ESP          //set up own exception  handling frame   
001B:005BD08C  INT      3                    //software breakpoint  exception   
001B:005BD08D  NOP   
001B:005BD08E  MOV      EAX,EAX   
001B:005BD090  STC                            //1st hardware  breakpoint here   
001B:005BD091  NOP   
001B:005BD092  LEA      EAX,\[EBX\*2+00001234\]   
001B:005BD099  CLC                            //2nd  hardware breakpoint here   
001B:005BD09A  NOP   
001B:005BD09B  SHR      EBX,05   
001B:005BD09E  CLD                            //3rd  hardware breakpoint here   
001B:005BD09F  NOP   
001B:005BD0A0  ROL      EAX,07   
001B:005BD0A3  NOP                            //4th  hardware breakpoint here   
001B:005BD0A4  NOP   
001B:005BD0A5  XOR      EBX,EBX   
001B:005BD0A7  DIV      EBX                  //devide-by-zero  exception   
001B:005BD0A9  POP      DWORD PTR FS:\[0000\]  //will continue execution here after exception handling   
001B:005BD0AF  ADD      ESP,04   
001B:005BD0B2  MOV      SI,4647   
001B:005BD0B6  MOV      DI,4A4D   
001B:005BD0BA  MOV      AL,\[EBP+00000099\]   
001B:005BD0C0  JMP      005BD161   
                                              //This is the entry point of own exception handler   
001B:005BD0C5  MOV      EAX,\[ESP+04\]          //EXCEPTION\_POINTERS.ExceptionRecord    
001B:005BD0C9  MOV      ECX,\[ESP+0C\]          //EXCEPTION\_POINTERS.ContextRecord   
001B:005BD0CD  INC      DWORD PTR \[ECX+000000B8\]   
001B:005BD0D3  MOV      EAX,\[EAX\]            //switch(ExceptionRecord->ExceptionCode)    
001B:005BD0D5  CMP      EAX,C0000094          //case EXCEPTION\_INT\_DIVIDE\_BY\_ZERO:   
001B:005BD0DA  JNZ      005BD100   
001B:005BD0DC  INC      DWORD PTR \[ECX+000000B8\]   
001B:005BD0E2  XOR      EAX,EAX   
001B:005BD0E4  AND      \[ECX+04\],EAX                //dr0 = 0   
001B:005BD0E7  AND      \[ECX+08\],EAX                //dr1  = 0   
001B:005BD0EA  AND      \[ECX+0C\],EAX                //dr2 = 0   
001B:005BD0ED  AND      \[ECX+10\],EAX                //dr3 = 0   
001B:005BD0F0  AND      DWORD PTR \[ECX+14\],FFFF0FF0  //dr6 = FFFF0FF0   
001B:005BD0F7  AND      DWORD PTR \[ECX+18\],0000DC00  //dr7 = 0000DC00   
001B:005BD0FE  JMP      005BD160   
001B:005BD100  CMP      EAX,80000004        //case EXCEPTION\_SINGLE\_STEP:   
001B:005BD105  JZ        005BD113   
001B:005BD107  CMP      EAX,80000003        //case EXCEPTION\_BREAKPOINT:   
001B:005BD10C  JZ        005BD120   
001B:005BD10E  PUSH      01   
001B:005BD110  POP      EAX   
001B:005BD111  JMP      005BD160   
001B:005BD113  CALL      005BD119   
001B:005BD118    
001B:005BD119  POP      EAX   
001B:005BD11A  INC      BYTE PTR \[EAX\]                //increase the number of hardware breakpoints    
001B:005BD11C  SUB      EAX,EAX   
001B:005BD11E  JMP      005BD160   
  
001B:005BD120  MOV      EAX,\[ECX+000000B4\]   
001B:005BD126  LEA      EAX,\[EAX+24\]   
001B:005BD129  MOV      \[ECX+04\],EAX                  //dr0 = 005BD0A3   
001B:005BD12C  MOV      EAX,\[ECX+000000B4\]   
001B:005BD132  LEA      EAX,\[EAX+1F\]   
001B:005BD135  MOV      \[ECX+08\],EAX                  //dr1 = 005BD09E   
001B:005BD138  MOV      EAX,\[ECX+000000B4\]   
001B:005BD13E  LEA      EAX,\[EAX+1A\]   
001B:005BD141  MOV      \[ECX+0C\],EAX                  //dr2 = 005BD099   
001B:005BD144  MOV      EAX,\[ECX+000000B4\]   
001B:005BD14A  LEA      EAX,\[EAX+11\]   
001B:005BD14D  MOV      \[ECX+10\],EAX                  //dr3 = 005BD090   
001B:005BD150  XOR      EAX,EAX   
001B:005BD152  AND      DWORD PTR \[ECX+14\],FFFF0FF0  //dr6 = FFFF0FF0   
001B:005BD159  MOV      DWORD PTR \[ECX+18\],00000155  //dr7 = 00000155   
001B:005BD160  RET   
001B:005BD161  SUB      AL,04                        //AL = (number of hardware breakpoints - 4)   
001B:005BD163  MOV      \[EBP+00000099\],AL            //for data decryption later   4dd466d0e1e0cc123afbe504da2462ca