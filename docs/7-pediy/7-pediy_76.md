

# SMC

> 原文：[`www.pediy.com/kssd/pediy05/pediy50315.htm`](https://www.pediy.com/kssd/pediy05/pediy50315.htm)

某个软件因为有壳，在将壳部分仿真完成后，发现还需要对某个地址 SMC，故在解压缩完成后，   
将要跳到真正入口处时，先跳到 SMC 处执行补丁，然后再跳到真正入口处。采用了增加一个节   
的方法扩充 SMC 代码(怎样增加一个节，因为很简单，不再做说明)   
  为什么不直接脱壳呢？因为这是个 DLL，脱壳很麻烦的，要处理引入表、冲定位项，干脆仿真   
通过算了，这样的问题是要修改某个内存地址的时候，需要用 SMC，不能直接修改了。。   
  
\--------------------------------------------------------------------------------   
//要修改的地址 Address=1067206h-00029302h=103DF04h 开始的 5 字节   
//要修改的地址将以 1067206h 作为基准,因为这个基准地址我们可以很容易得到且不受任何    
//因素影响,上面为什么使用“-”，因为我将节加到最后，地址肯定大于原程序中的任何地   
//址。   
\--------------------------------------------------------------------------------   
//补丁程序   
//1.因为原来程序引入表中没有"VirtualQuery"函数，所以要先取得   
//2.以下用到的其它函数，原来程序引入表中都有，所以直接使用    
\--------------------------------------------------------------------------------   
EAX\=00000000  EBX\=01067012  ECX\=00000000  EDX\=01030000  ESI\=00000004          
EDI\=FFFFFFFC  EBP\=01067206  ESP\=016FFBC0  EIP=0106720F  o d I s z a P c      
CS=0167  DS\=016F  SS\=016F  ES\=016F  FS\=3C8F  GS\=0000                        
\--------------------------------------------------------------------------------   
016F:01066FE2 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................   
016F:01066FF2 00 00 00 00 00 00 00 00-00 00 00 00 00 00 E9 FB  ................   
016F:01067002 01 00 00 56 69 72 74 75-61 6C 51 75 65 72 79 00  ...VirtualQuery.   
016F:01067012 4B 65 72 6E 65 6C 33 32-2E 44 4C 4C 00 00 00 00  Kernel32.DLL....   
016F:01067022 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................   
016F:01067032 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................   
\--------------------------------------------------------------------------------   
0167:01067200  PUSHAD   
0167:01067201  CALL      01067206   
0167:01067206  POP      EBP          ;此时,ebp = 1067206h   
0167:01067207  MOV      EBX,EBP   
0167:01067209  SUB      EBX,000001F4  ;EBX\-1F4h = "Kernel32.DLL"   
0167:0106720F  PUSH      EBX   
0167:01067210  CALL      \[KERNEL32!GetModuleHandleA\]   
0167:01067216  TEST      EAX,EAX   
0167:01067218  JZ        010672AC   
0167:0106721E  MOV      EDX,EAX   
0167:01067220  MOV      EBX,EBP   
0167:01067222  SUB      EBX,00000201  ;EBX\-201h 指向"VirtualQuery"   
0167:01067228  PUSH      EBX   
0167:01067229  PUSH      EDX   
0167:0106722A  NOP   
0167:0106722B  NOP   
0167:0106722C  NOP   
0167:0106722D  NOP   
0167:0106722E  NOP   
0167:0106722F  NOP   
//   
0167:01067230  CALL      \[KERNEL32!GetProcAddress\] ;取得 VirtualQuery 函数地址   
0167:01067236  TEST      EAX,EAX   
0167:01067238  JZ        010672AC   
0167:0106723A  MOV      EBX,EBP   
0167:0106723C  SUB      EBX,00000106 ;EBX\-106h 指向 VirtualQuery 函数参数区(参数设置参考 SDK)   
0167:01067242  PUSH      1C          ;VirtualQuery 函数参数区长度   
0167:01067244  PUSH      EBX          ;PMEMORY\_BASIC\_INFORMATION   
0167:01067245  MOV      EBX,EBP   
0167:01067247  SUB      EBX,00029302 ;要修改的地址 Address=EBX\-00029302h=103DF04h   
0167:0106724D  PUSH      EBX          ;address of region   
0167:0106724E  CALL      EAX          ;调用 VirtualQuery 取得 Address 所属页的页基址   
//   
0167:01067250  MOV      EBX,EBP   
0167:01067252  SUB      EBX,000001E6 ;EBX\-1e6h = VirtualProtect 参数，返回的原来的页属性   
0167:01067258  PUSH      EBX   
0167:01067259  PUSH      00000040    ;新的页属性(PAGE\_EXECUTE\_READWRITE)   
0167:0106725E  MOV      EBX,EBP   
0167:01067260  SUB      EBX,00000106         ;EBX\-106h = VirtualProtect 参数区,数据来自 VirtualQuery   
0167:01067266  PUSH      DWORD PTR \[EBX+0C\] ;VirtualProtect 参数,size of the region   
0167:01067269  PUSH      DWORD PTR \[EBX\]    ;VirtualProtect 参数,address of region of committed   
                                            ;pages   
0167:0106726B  NOP   
0167:0106726C  NOP   
0167:0106726D  NOP   
0167:0106726E  CALL      \[KERNEL32!VirtualProtect\] ;调用 VirtualProtect 改变页属性   
0167:01067274  MOV      EBX,EBP   
0167:01067276  SUB      EBX,00029302  ;要修改的地址 Address=EBX\-00029302h=103DF04h   
0167:0106727C  MOV      DWORD PTR \[EBX\],b3b2b1b0  ;将原来 4 字节改为 b3b2b1b0   
0167:01067282  MOV      BYTE PTR \[EBX+04\],XX      ;将原来 1 字节改为 XXh   
  
//下面恢复修改的页属性   
0167:01067286  MOV      EBX,EBP   
0167:01067288  SUB      EBX,000001E4  ;VirtualProtect 参数，原来的页属性，此时这个参数对   
                                      ;我们没用，但必须有   
0167:0106728E  PUSH      EBX   
0167:0106728F  MOV      EBX,EBP   
0167:01067291  SUB      EBX,000001E6  ;保存的原来的页属性   
0167:01067297  PUSH      DWORD PTR \[EBX\]   
0167:01067299  MOV      EBX,EBP   
0167:0106729B  SUB      EBX,00000106          ;EBX\-106h = VirtualProtect 参数区,数据来自 VirtualQuery   
0167:010672A1  PUSH      DWORD PTR \[EBX+0C\] ;VirtualProtect 参数,size of the region   
0167:010672A4  PUSH      DWORD PTR \[EBX\]    ;VirtualProtect 参数,address of region of committed   
                                            ;pages   
0167:010672A6  CALL      \[KERNEL32!VirtualProtect\]  ;调用 VirtualProtect 恢复页属性   
0167:010672AC  POPAD   
0167:010672AD  JMP      XXXXXXXX        ;跳转到原来程序处继续执行   
  
  
  因为涉及某个国产软件，恕不说明具体程序。。。   
                                Spring.W   
                                2002/10/20  4dd466d0e1e0cc123afbe504da2462ca