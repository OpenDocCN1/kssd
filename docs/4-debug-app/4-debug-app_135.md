

# LordPE 只显示 60 个进程 fix

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-444.htm`](https://www.pediy.com/kssd/pediy08/pediy8-444.htm)

LordPE 只能显示 60 个进程  
  
LoarPE 是用 psapi 的 EnumProcesses 来枚举进程的  
跟了一下发现是封装在 procs.dll 里的 GetNumBerOfProcesses 函数出了问题  
代码如下:  
20001200 >  A1 FC240020     mov     eax, dword ptr \[200024FC\]  
20001205    81EC 1C020000   sub     esp, 21C  //在这里 $21c-$F0=$12c   
  //增加一下内存 改为支持 256 个的就是 256\*4=1024+300=1324 即$52C 就是 sub esp, 52C   
2000120B    85C0            test    eax, eax  
2000120D    56              push    esi  
2000120E    75 7D           jnz     short 2000128D  
20001210    6A 00           push    0  
20001212    6A 02           push    2  
20001214    FF15 E8240020   call    dword ptr \[200024E8\]  
2000121A    8BF0            mov     esi, eax  
2000121C    83FE FF         cmp     esi, -1  
2000121F    75 0A           jnz     short 2000122B  
20001221    33C0            xor     eax, eax  
20001223    5E              pop     esi  
20001224    81C4 1C020000   add     esp, 21C  //这里也要改为 add esp, 52C    
2000122A    C3              retn  
2000122B    8D4424 08       lea     eax, dword ptr \[esp+8\]  
2000122F    C74424 08 28010>mov     dword ptr \[esp+8\], 128  
20001237    50              push    eax  
20001238    56              push    esi  
20001239    FF15 DC240020   call    dword ptr \[200024DC\]  
2000123F    85C0            test    eax, eax  
20001241    75 11           jnz     short 20001254  
20001243    56              push    esi  
20001244    FF15 08100020   call    dword ptr \[<&KERNEL32.CloseHandle>\]        ; kernel32.CloseHandle  
2000124A    33C0            xor     eax, eax  
2000124C    5E              pop     esi  
2000124D    81C4 1C020000   add     esp, 21C  //这里也要改为 add esp, 52C  
20001253    C3              retn  
20001254    8D4C24 08       lea     ecx, dword ptr \[esp+8\]  
20001258    57              push    edi  
20001259    51              push    ecx  
2000125A    56              push    esi  
2000125B    BF 01000000     mov     edi, 1  
20001260    FF15 E4240020   call    dword ptr \[200024E4\]  
20001266    85C0            test    eax, eax  
20001268    74 11           je      short 2000127B  
2000126A    8D5424 0C       lea     edx, dword ptr \[esp+C\]  
2000126E    47              inc     edi  
2000126F    52              push    edx  
20001270    56              push    esi  
20001271    FF15 E4240020   call    dword ptr \[200024E4\]  
20001277    85C0            test    eax, eax  
20001279  ^ 75 EF           jnz     short 2000126A  
2000127B    56              push    esi  
2000127C    FF15 08100020   call    dword ptr \[<&KERNEL32.CloseHandle>\]        ; kernel32.CloseHandle  
20001282    8BC7            mov     eax, edi  
20001284    5F              pop     edi  
20001285    5E              pop     esi  
20001286    81C4 1C020000   add     esp, 21C  //这里也要改为 add esp, 52C  
2000128C    C3              retn  
主要问题出在这里,明显是申请的数组太小了见下面的 Delphi 代码  
2000128D    8D4424 04       lea     eax, dword ptr \[esp+4\]  
20001291    8D8C24 30010000 lea     ecx, dword ptr \[esp+130\]  
20001298    50              push    eax    //返回的实际大小  
20001299    68 F0000000     push    0F0    //数组大小 这里硬性指定了为 240/4=60 就是就最大是 60 了 改为 push 0400 就是 256\*4=1024 了  
2000129E    51              push    ecx    //指针指向存放进程 ID 的数组  
2000129F    FF15 EC240020   call    dword ptr \[200024EC\]                       ; psapi.EnumProcesses  
  
200012A5    85C0            test    eax, eax  
200012A7    75 08           jnz     short 200012B1  
200012A9    5E              pop     esi  
200012AA    81C4 1C020000   add     esp, 21C  //这里也要改为 add esp, 52C  
200012B0    C3              retn  
200012B1    8B4424 04       mov     eax, dword ptr \[esp+4\]  
200012B5    5E              pop     esi  
200012B6    C1E8 02         shr     eax, 2  
200012B9    81C4 1C020000   add     esp, 21C  //这里也要改为 add esp, 52C  
200012BF    C3              retn  
  
对应的 Delphi 代码应是  
var  
    lProcess : array \[0..239\] of DWord;  
    dwSize : DWord;  
begin    
    if not EnumProcesses(@lProcess, SizeOf(lProcess), dwSize) then Exit;  
end;  
  
  
经过上面的修改后有足够存放 256 个进程的内存空间了  
测试一下 :( 程序出错了应还有地方存在这样的问题  
  
在上面的函数返回  
来到了 exe 中的这里  
004060B6  |.  8BF8          mov     edi, eax  
004060B8  |.  85FF          test    edi, edi  
004060BA  |.  897C24 10     mov     dword ptr \[esp+10\], edi  
004060BE  |.  0F84 98020000 je      0040635C  
004060C4  |.  68 F0000000   push    0F0                                        ;  这里也硬性指定为 60 了 改为 push 400   \\ z  jnkl,9,7km km 6 v  
004060C9  |.  68 40EB4100   push    0041EB40                                   ;  这个地址是存入进程数据的  
004060CE  |.  E8 27270100   call    <jmp.&procs.GetProcessIDList>  
004060D3  |.  85C0          test    eax, eax  
004060D5  |.  0F84 81020000 je      0040635C  
004060DB  |.  55            push    ebp  
004060DC  |.  8B2D C4924100 mov     ebp, dword ptr \[<&USER32.wsprintfA>\]       ;  USER32.wsprintfA  
004060E2  |.  85FF          test    edi, edi  
004060E4  |.  0F86 D2010000 jbe     004062BC  
004060EA      BF 40EB4100   mov     edi, 0041EB40  
  
F8 跟了一会原来 41EB40 这地址空间不足,超过 60 个进程后覆盖了后面的部分代码  
只好另外找地方了 :(  
在程序尾部 4189A2 的地方有 n 多的 00 计算一下足够存放 256 个进程的数据,就用那里吧...  
  
再来搜索一下全部的常量 41EB40 发现还有 n 处用到了  
参考位于 LordPE:.text 到常量 41EB40  
地址       反汇编                                    注释  
00404A01   mov     edx, dword ptr \[esi\*4+41EB40\]  
00404D06   mov     ecx, dword ptr \[eax\*4+41EB40\]  
00404DC4   mov     eax, dword ptr \[eax\*4+41EB40\]  
00404EC2   mov     edx, dword ptr \[eax\*4+41EB40\]  
00404F72   mov     eax, dword ptr \[eax\*4+41EB40\]  
00404F9E   mov     eax, dword ptr \[eax\*4+41EB40\]  
00405014   mov     edx, dword ptr \[eax\*4+41EB40\]  
0040592B   mov     edx, dword ptr \[41EB40\]           ds:\[0041EB40\]=00000000  
00405ADE   mov     edi, dword ptr \[eax\*4+41EB40\]  
004060C9   push    0041EB40                          原来是 41eb40 改 4189a2  
004060EA   mov     edi, 0041EB40                     0041EB40=0041EB40  
004062C5   mov     eax, dword ptr \[ebx\*4+41EB40\]  
  
这些地方的 41EB40 都要改为 4189A2  
  
这个函数是 dump 进程时用到的  
00406690      81EC 2C050000 sub     esp, 52C                         ;  这里原来也是 f0  
00406696  |.  8D4424 00     lea     eax, dword ptr \[esp\]  
0040669A  |.  56            push    esi  
0040669B      68 00040000   push    400                              ;  这里原来也是 f0  
004066A0  |.  50            push    eax  
004066A1  |.  E8 54210100   call    <jmp.&procs.GetProcessIDList>  
004066A6  |.  85C0          test    eax, eax  
004066A8  |.  74 26         je      short 004066D0  
004066AA  |.  E8 45210100   call    <jmp.&procs.GetNumberOfProcesses>  
004066AF  |.  85C0          test    eax, eax  
004066B1  |.  74 1D         je      short 004066D0  
004066B3  |.  33C9          xor     ecx, ecx  
004066B5  |.  85C0          test    eax, eax  
004066B7  |.  76 17         jbe     short 004066D0  
004066B9      8BB424 F80000>mov     esi, dword ptr \[esp+F8\]          ;  取传进来的 PID 改为 \[esp + 534\]  
004066C0  |.  8D5424 04     lea     edx, dword ptr \[esp+4\]  
004066C4  |>  3932          /cmp     dword ptr \[edx\], esi  
004066C6  |.  74 12         |je      short 004066DA  
004066C8  |.  41            |inc     ecx  
004066C9  |.  83C2 04       |add     edx, 4  
004066CC  |.  3BC8          |cmp     ecx, eax  
004066CE  |.^ 72 F4         \\jb      short 004066C4  
004066D0  |>  33C0          xor     eax, eax  
004066D2  |.  5E            pop     esi  
004066D3      81C4 2C050000 add     esp, 52C  
004066D9  |.  C3            retn  
004066DA  |>  B8 01000000   mov     eax, 1  
004066DF  |.  5E            pop     esi  
004066E0      81C4 2C050000 add     esp, 52C  
004066E6  \\.  C3            retn  
  
最后还要把 LordPE.exe 的区段.text 改为可写  
  
  
procs.dll 中这些地方还要修改  
GetProcessBaseSize 中的  
200017DF    8D8D 88FAFFFF   lea     ecx, dword ptr \[ebp-578\]  
200017E5    51              push    ecx  
200017E6    68 00040000     push    400                              ; 这里也改 push f0 为 push 400  
200017EB    8D95 94FAFFFF   lea     edx, dword ptr \[ebp-56C\]  
200017F1    52              push    edx  
200017F2    FF15 EC240020   call    dword ptr \[200024EC\]             ; psapi.EnumProcesses  
200017F8    85C0            test    eax, eax  
  
GetProcessPathID 中的  
20001F54    8D5424 14       lea     edx, dword ptr \[esp+14\]  
20001F58    8D8424 4C020000 lea     eax, dword ptr \[esp+24C\]  
20001F5F    52              push    edx  
20001F60    68 F0000000     push    0F0                              ; push f0 改 push 400  
20001F65    50              push    eax  
20001F66    FF15 EC240020   call    dword ptr \[200024EC\]             ; psapi.EnumProcesses  
20001F6C    85C0            test    eax, eax  
  
  
基本上就行了的  
  
新上传一个版本 写了一个 LordPe\_Fix.dll 的来进行 fix  
  

 4dd466d0e1e0cc123afbe504da2462ca