

# VASSIST\_X 1246 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6694.htm`](https://www.pediy.com/kssd/pediy06/pediy6694.htm)

脱 Armadillo 3.75D 加壳的 DLL.  
  
  
DLL 的名称.VA\_X.DLL(VASSIST\_X 1246 的加密的主要 DLL)  
使用工具:ollydbg,010Editor,LordPE,PEID.  
目的:仅仅是为了兴趣.  
声明:我是菜鸟.水平很次,很多时候可能用了很多笨的办法,如有发现,还望指出,让我有所提高:D  
本文作者:AERROR  
    
壳的执行流程/原理:  
    入口点处是一个,标准的由 VC 生成的 DLL 的入口代码.壳在执行时,首先检测 IsDubberPresent,没有的话,分配一段内存,  
将脱壳的保护代码填充进去;为.TEXT CODE 两段分配内存;  
    然后去执行这段动态代码.那里将会,把真正的代码解压到.TEXT CODE ...等段中,把 OEP 放进一个全局变量\[1EFB66C4\]  
中,并且处理 IAT,处理重定位表,检测 SOFTICE,文件有效性的检测,解压原程序中保护数据段到一个申请的内存中,调整跳转地址等等.  
    上面的完成后,就呼叫的 OEP.  
      
    过程参考了 FLY 版主的文章.但 BP GetModuleHandleA+5 这样下断点的情况和 FLY 版主所说的情况很不一样.因为我之前在不脱壳的  
    情况下(因为当时我不知道它是由 Armadillo 加壳的)用动态补丁的方法破解过它较早的版本,并把壳检测 SOFTICE 的代码和文件有效  
性效验的代码破解了,因此,比较熟悉壳的代码,我就一步一步的跟踪找出了 OEP.但因此我就没有 BP GetModuleHandleA+5 这样找出那个  
处理重定位表的代码.我用了另外的笨办法把它修复了(也不知是不是正确的,详细的做法请看下面的修复的过程).     
    同时,这个 DLL 不同于 FLY 版主那个测试的 DLL,它有一段是由壳动态分配的,并且脱壳了之后,还需要调用的代码.  
    (我真的是菜,虽然我把它搞定了,但我还是想问问,这个是不是 COPYMEM II?望知道的大侠一定要答我,谢谢:D)  
  
一.脱壳和修复  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
我自己写一个的加载 DLL 的工具,在加载这下断点,在 LdrLoadDll 下断点.  
断下后,查找命令 call dword ptr ss:\[ebp+8\],于是到了下面:  
//ntdll.dll  
77F830DB    FF75 14         push dword ptr ss:\[ebp+14\]  
77F830DE    FF75 10         push dword ptr ss:\[ebp+10\]  
77F830E1    FF75 0C         push dword ptr ss:\[ebp+C\]  
77F830E4    FF55 08         call dword ptr ss:\[ebp+8\]  -->//DLL 入口点,这个地址取决于操作系统.按 F7 跟进去.  
77F830E7    8BE6            mov esp,esi  
///  
  
  
//跟着我们来到这 DLL 的入口点.  
1EF9A977 >/$  55            push ebp  
1EF9A978  |.  8BEC          mov ebp,esp  
1EF9A97A  |.  53            push ebx  
...  
1EF9A9BB  |> \\33C0          xor eax,eax  
1EF9A9BD  |.  EB 4E         jmp short VA\_X.1EF9AA0D  
1EF9A9BF  |>  57            push edi  
1EF9A9C0  |.  56            push esi  
1EF9A9C1  |.  53            push ebx  
1EF9A9C2  |.  E8 59DAFEFF   call VA\_X.1EF88420   <-- //DLL main,这段是标准的 VC 生成的 DLL 的开始  
                                                     //的代码,然后转入的就是壳的代码段了.  
1EF9A9C7  |.  83FE 01       cmp esi,1  
1EF9A9CA  |.  8945 0C       mov dword ptr ss:\[ebp+C\],eax  
  
  
///////////////////////////////////////////////////////  
1EF88420   $  55            push ebp  
1EF88421   .  8BEC          mov ebp,esp  
1EF88423   .  6A FF         push -1  
1EF88425   .  68 004DFB1E   push VA\_X.1EFB4D00  
1EF8842A   .  68 C0A8F91E   push <jmp.&MSVCRT.\_except\_handler3>      ;  SE handler installation  
1EF8842F   .  64:A1 0000000>mov eax,dword ptr fs:\[0\]  
.........  
1EF88509   > \\E8 181B0000   call VA\_X.1EF8A026    <--// 解壳和检验,呼叫 OEP 请看下面.  
1EF8850E   .  E9 04010000   jmp VA\_X.1EF88617  
1EF88513   >  837D 0C 00    cmp dword ptr ss:\[ebp+C\],0  
  
/////////////////////////////////////////////////////  
1EF8A026  /$  55            push ebp  
1EF8A027  |.  8BEC          mov ebp,esp  
1EF8A029  |.  81EC 08010000 sub esp,108  
1EF8A02F  |.  6A 00         push 0                                   ; /Value = NULL  
1EF8A031  |.  68 D032FB1E   push VA\_X.1EFB32D0                       ; |VarName = "INITIALIZEDLLADDR"  
1EF8A036  |.  FF15 4430FB1E call dword ptr ds:\[<&KERNEL32.SetEnviron>; \\SetEnvironmentVariableA  
1EF8A03C  |.  833D D468FB1E>cmp dword ptr ds:\[1EFB68D4\],0  
1EF8A043  |.  74 20         je short VA\_X.1EF8A065  
1EF8A045  |.  A1 D468FB1E   mov eax,dword ptr ds:\[1EFB68D4\]  
1EF8A04A  |.  8945 F8       mov dword ptr ss:\[ebp\-8\],eax  
1EF8A04D  |.  C705 D468FB1E>mov dword ptr ds:\[1EFB68D4\],0  
1EF8A057  |.  FF55 F8       call dword ptr ss:\[ebp\-8\]  
1EF8A05A  |.  85C0          test eax,eax  
1EF8A05C  |.  75 07         jnz short VA\_X.1EF8A065  
1EF8A05E  |.  33C0          xor eax,eax  
1EF8A060  |.  E9 C0000000   jmp VA\_X.1EF8A125  
1EF8A065  |>  E8 EAFEFFFF   call VA\_X.1EF89F54  <---- //请看下面,解壳和检测 SOFTICE 和 Win32debug  
1EF8A06A  |.  8945 FC       mov dword ptr ss:\[ebp\-4\],eax  
1EF8A06D  |.  837D FC 01    cmp dword ptr ss:\[ebp\-4\],1  
1EF8A071  |.  75 40         jnz short VA\_X.1EF8A0B3  
1EF8A073  |.  833D C466FB1E>cmp dword ptr ds:\[1EFB66C4\],0  
1EF8A07A  |.  74 30         je short VA\_X.1EF8A0AC  
1EF8A07C  |.  68 E868FB1E   push VA\_X.1EFB68E8  
1EF8A081  |.  6A 01         push 1  
1EF8A083  |.  8B0D 9066FB1E mov ecx,dword ptr ds:\[1EFB6690\]  
1EF8A089  |.  51            push ecx  
1EF8A08A  |.  FF15 C466FB1E call dword ptr ds:\[1EFB66C4\]<--------// 呼叫 OEP,请看 1EE4868E  
1EF8A090  |.  8945 FC       mov dword ptr ss:\[ebp\-4\],eax  
1EF8A093  |.  837D FC 00    cmp dword ptr ss:\[ebp\-4\],0  
1EF8A097  |.  75 0C         jnz short VA\_X.1EF8A0A5  
1EF8A099  |.  E8 5FFFFFFF   call VA\_X.1EF89FFD  
1EF8A09E  |.  33C0          xor eax,eax  
  
  
  
  
  
//////////////////////////////////////////////////////////////////////////////////////////////////  
1EF89F54  /$  55            push ebp  
1EF89F55  |.  8BEC          mov ebp,esp  
1EF89F57  |.  81EC 10010000 sub esp,110  
1EF89F5D  |.  C645 F8 00    mov byte ptr ss:\[ebp\-8\],0  
1EF89F61  |.  68 8435FB1E   push VA\_X.1EFB3584                       ; /FileName = "Kernel32"  
1EF89F66  |.  FF15 6030FB1E call dword ptr ds:\[<&KERNEL32.LoadLibrar>; \\LoadLibraryA  
1EF89F6C  |.  8945 F4       mov dword ptr ss:\[ebp\-C\],eax  
1EF89F6F  |.  837D F4 00    cmp dword ptr ss:\[ebp\-C\],0  
1EF89F73  |.  74 4C         je short VA\_X.1EF89FC1  
1EF89F75  |.  68 7035FB1E   push VA\_X.1EFB3570                       ; /ProcNameOrOrdinal = "IsDebuggerPresent"  
1EF89F7A  |.  8B45 F4       mov eax,dword ptr ss:\[ebp\-C\]             ; |  
1EF89F7D  |.  50            push eax                                 ; |hModule  
1EF89F7E  |.  FF15 5830FB1E call dword ptr ds:\[<&KERNEL32.GetProcAdd>; \\GetProcAddress  
1EF89F84  |.  8945 F0       mov dword ptr ss:\[ebp\-10\],eax            ;  KERNEL32.IsDebuggerPresent  
1EF89F87  |.  837D F0 00    cmp dword ptr ss:\[ebp\-10\],0  
1EF89F8B  |.  74 34         je short VA\_X.1EF89FC1  
1EF89F8D  |.  FF55 F0       call dword ptr ss:\[ebp\-10\]  
1EF89F90  |.  85C0          test eax,eax  
1EF89F92  |.  74 2D         je short VA\_X.1EF89FC1  
1EF89F94  |.  FF15 2031FB1E call dword ptr ds:\[<&KERNEL32.GetCurrent>; \[GetCurrentProcessId  
1EF89F9A  |.  50            push eax                                 ; /<%X>  
1EF89F9B  |.  68 6835FB1E   push VA\_X.1EFB3568                       ; |format = "%X:DAF"  
1EF89FA0  |.  8D8D F0FEFFFF lea ecx,dword ptr ss:\[ebp\-110\]           ; |  
1EF89FA6  |.  51            push ecx                                 ; |s  
1EF89FA7  |.  FF15 5031FB1E call dword ptr ds:\[<&MSVCRT.sprintf>\]    ; \\sprintf  
1EF89FAD  |.  83C4 0C       add esp,0C  
1EF89FB0  |.  8D95 F0FEFFFF lea edx,dword ptr ss:\[ebp\-110\]  
1EF89FB6  |.  52            push edx                                 ; /MutexName  
1EF89FB7  |.  6A 00         push 0                                   ; |InitialOwner = FALSE  
1EF89FB9  |.  6A 00         push 0                                   ; |pSecurity = NULL  
1EF89FBB  |.  FF15 8830FB1E call dword ptr ds:\[<&KERNEL32.CreateMute>; \\CreateMutexA  
1EF89FC1  |>  68 6C86F81E   push VA\_X.1EF8866C                       ; /handler = VA\_X.1EF8866C  
1EF89FC6  |.  FF15 4031FB1E call dword ptr ds:\[<&MSVCRT.\_set\_new\_han>; \\\_set\_new\_handler  
1EF89FCC  |.  83C4 04       add esp,4  
1EF89FCF  |.  E8 C8E6FFFF   call VA\_X.1EF8869C        <--------------- //跟进去.看下面  
1EF89FD4  |.  8945 FC       mov dword ptr ss:\[ebp\-4\],eax  
1EF89FD7  |.  6A 00         push 0                                   ; /handler = NULL  
1EF89FD9  |.  FF15 4031FB1E call dword ptr ds:\[<&MSVCRT.\_set\_new\_han>; \\\_set\_new\_handler  
1EF89FDF  |.  83C4 04       add esp,4  
1EF89FE2  |.  837D FC 01    cmp dword ptr ss:\[ebp\-4\],1  
1EF89FE6  |.  75 0E         jnz short VA\_X.1EF89FF6  
1EF89FE8  |.  68 A866FB1E   push VA\_X.1EFB66A8  
1EF89FED  |.  FF15 CC66FB1E call dword ptr ds:\[1EFB66CC\]  
1EF89FF3  |.  83C4 04       add esp,4  
1EF89FF6  |>  8B45 FC       mov eax,dword ptr ss:\[ebp\-4\]  
1EF89FF9  |.  8BE5          mov esp,ebp  
1EF89FFB  |.  5D            pop ebp  
1EF89FFC  \\.  C3            retn  
  
  
/////////////////////////////////////////////////////////////////////////////////////  
1EF8869C  /$  55            push ebp  
1EF8869D  |.  8BEC          mov ebp,esp  
1EF8869F  |.  83EC 5C       sub esp,5C  
1EF886A2  |.  A1 3033FB1E   mov eax,dword ptr ds:\[1EFB3330\]  
1EF886A7  |.  3305 1833FB1E xor eax,dword ptr ds:\[1EFB3318\]  
1EF886AD  |.  3305 3C33FB1E xor eax,dword ptr ds:\[1EFB333C\]  
1EF886B3  |.  83E0 03       and eax,3  
1EF886B6  |.  50            push eax                                 ; /Arg1  
1EF886B7  |.  E8 74D3FEFF   call VA\_X.1EF75A30                       ; \\VA\_X.1EF75A30  
1EF886BC  |.  83C4 04       add esp,4  
................  
.  
....略......  
.  
.  
1EF888CF  |.  FF15 3430FB1E call dword ptr ds:\[<&KERNEL32.GetModuleH>; \\GetModuleHandleA  
1EF888D5  |.  8945 A4       mov dword ptr ss:\[ebp\-5C\],eax  
1EF888D8  |>  8B55 A4       mov edx,dword ptr ss:\[ebp\-5C\]  
1EF888DB  |.  8955 E4       mov dword ptr ss:\[ebp\-1C\],edx  
1EF888DE  |.  A1 E432FB1E   mov eax,dword ptr ds:\[1EFB32E4\]  
1EF888E3  |.  8945 E8       mov dword ptr ss:\[ebp\-18\],eax  
1EF888E6  |.  C745 EC FFFFF>mov dword ptr ss:\[ebp\-14\],-1  
1EF888ED  |.  8D4D C4       lea ecx,dword ptr ss:\[ebp\-3C\]  
1EF888F0  |.  51            push ecx  
1EF888F1  |.  FF55 F0       call dword ptr ss:\[ebp\-10\]  <--- //呼叫 008CEDBF,这里将进行对原来的代码的解压  
                                                             //检验文件的有效性,检测 SOFTICE 是否存在  
                                                             //对 IAT 表进行处理,重定位表的处理,请看 008D574E 处.  
                                                             //脱壳时,应该在这里对 008D574E 下内存访问断点,  
                                                             //因为到这里 008D574E 所占的内存才被分配,并填入代码.  
                                                             //如果这里跟进这个内存的段,查找"armVersion"的参考字符串,  
                                                             //找到后上翻一行, 可以看到  
                                                             //ASCII "3.75D"  
                                                             //<armVersion xsi:type="xsd:string">%s</armVersion  
                                                             //因此,可知 armVersion 为 3.75D  
  
0103481C    68 0C300601     PUSH 106300C                        
                                                               
                                                               
1EF888F4  |.  83C4 04       add esp,4  
1EF888F7  |.  8945 FC       mov dword ptr ss:\[ebp\-4\],eax  
  
/////////////////////////////////////////////////////////////////////////////////  
  
008CEDBF    55              push ebp  
008CEDC0    8BEC            mov ebp,esp  
008CEDC2    6A FF           push -1  
008CEDC4    68 E8E78D00     push 8DE7E8  
008CEDC9    68 50DA8D00     push 8DDA50                              ; jmp to MSVCRT.\_except\_handler3  
.  
....略......  
.  
008CEE5F    FF75 D4         push dword ptr ss:\[ebp\-2C\]  
008CEE62    E8 4526FEFF     call 008B14AC              <---- //解压代码函数  
008CEE67    83C4 10         add esp,10  
008CEE6A    A1 A0CF8E00     mov eax,dword ptr ds:\[8ECFA0\]  
  
.  
....略......  
.  
  
008CF4DE    897D BC         mov dword ptr ss:\[ebp\-44\],edi  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
//这里进行文件的效检  
//RawOffset == C3CDA(在文件中查找\[1F011CDA\]的数据得到的) == (802010)10  
//esi ==  1F011CDA  == (520166618)  
//edi ==  1F0144DA  
//把计算结果和 esi 至 edi 段数据进行 XOR 运算后,再把结果保存到 ESI-EDI 段中.  
008CF4E1    3BF7            cmp esi,edi                              ; VA\_X.1F0144DA  
008CF4E3    73 12           jnb short 008CF4F7  
008CF4E5    8D4D C4         lea ecx,dword ptr ss:\[ebp\-3C\]  
008CF4E8    E8 131BFEFF     call 008B1000  
008CF4ED    3106            xor dword ptr ds:\[esi\],eax  
008CF4EF    83C6 04         add esi,4  
008CF4F2    8975 C0         mov dword p-tr ss:\[ebp\-40\],esi  
008CF4F5  ^ EB EA           jmp short 008CF4E1  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
008CF4F7    C705 88428E00 A>mov dword ptr ds:\[8E4288\],8E4CA4         ; ASCII "LP4"  
008CF501    A0 186A8E00     mov al,byte ptr ds:\[8E6A18\]  
  
  
.  
....略......  
.  
008CF778    83C4 28         add esp,28  
008CF77B    50              push eax  
008CF77C    53              push ebx  
008CF77D    E8 9992FFFF     call 008C8A1B  
008CF782    59              pop ecx  
008CF783    50              push eax  
008CF784    E8 EFBB0000     call 008DB378  <-- show dialog  
008CF789    83C4 10         add esp,10  
  
  
  
  
  
  
//////////////////////////////////////////////////////////  
//壳对于 IAT 处理的原理:  
//一一读取处理过的一个函数和 DLL 和字符串列表,将它和壳的输入函数作比较,  
//如果壳中有的话,就把地址指向壳的地址(经过处理的),如果没有的话就指向原始的地址.  
  
008D574E    FF15 3CE18D00   call dword ptr ds:\[8DE13C\]                         ; KERNEL32.VirtualProtect  处理 IAT  
008D5754    83A5 A4D4FFFF 0>and dword ptr ss:\[ebp\-2B5C\],0  
008D575B    FF15 90E28D00   call dword ptr ds:\[8DE290\]                         ; KERNEL32.GetTickCount  
008D5761    8985 A0D4FFFF   mov dword ptr ss:\[ebp\-2B60\],eax  
.  
....略......  
.  
008D5898    FF15 DCE28D00   call dword ptr ds:\[8DE2DC\]                         ; MSVCRT.strchr  
008D589E    59              pop ecx  
008D589F    59              pop ecx  
008D58A0    40              inc eax  
008D58A1    8985 7CD8FFFF   mov dword ptr ss:\[ebp\-2784\],eax  
008D58A7    83BD 98D4FFFF 0>cmp dword ptr ss:\[ebp\-2B68\],0  
008D58AE    74 71           je short 008D5921                  <------------//这里改为 jmp 008D5921  直接跳过比较  
                                                                           //让它全部填入真实原始的输入函数地址,  
                                                                             
008D58B0    8B85 98D4FFFF   mov eax,dword ptr ss:\[ebp\-2B68\]  
008D58B6    8985 58D2FFFF   mov dword ptr ss:\[ebp\-2DA8\],eax  
008D58BC    EB 0F           jmp short 008D58CD  
008D58BE    8B85 58D2FFFF   mov eax,dword ptr ss:\[ebp\-2DA8\]  
008D58C4    83C0 0C         add eax,0C  
008D58C7    8985 58D2FFFF   mov dword ptr ss:\[ebp\-2DA8\],eax  
008D58CD    8B85 58D2FFFF   mov eax,dword ptr ss:\[ebp\-2DA8\]  
008D58D3    8378 08 00      cmp dword ptr ds:\[eax+8\],0  
008D58D7    74 48           je short 008D5921  
008D58D9    68 00010000     push 100  
008D58DE    8D85 58D1FFFF   lea eax,dword ptr ss:\[ebp\-2EA8\]  
008D58E4    50              push eax  
008D58E5    8B85 58D2FFFF   mov eax,dword ptr ss:\[ebp\-2DA8\]  
008D58EB    FF30            push dword ptr ds:\[eax\]  
008D58ED    E8 E026FEFF     call 008B7FD2  
008D58F2    83C4 0C         add esp,0C  
008D58F5    8D85 58D1FFFF   lea eax,dword ptr ss:\[ebp\-2EA8\]  
008D58FB    50              push eax  
008D58FC    FFB5 60D2FFFF   push dword ptr ss:\[ebp\-2DA0\]  
008D5902    FF15 48E38D00   call dword ptr ds:\[8DE348\]                         ; MSVCRT.\_stricmp  
008D5908    59              pop ecx  
008D5909    59              pop ecx  
008D590A    85C0            test eax,eax  
008D590C    75 11           jnz short 008D591F  
008D590E    8B85 58D2FFFF   mov eax,dword ptr ss:\[ebp\-2DA8\]  
008D5914    8B40 08         mov eax,dword ptr ds:\[eax+8\]  
008D5917    8985 64D2FFFF   mov dword ptr ss:\[ebp\-2D9C\],eax  
008D591D    EB 02           jmp short 008D5921  
008D591F  ^ EB 9D           jmp short 008D58BE  
008D5921    8B85 A4D4FFFF   mov eax,dword ptr ss:\[ebp\-2B5C\]  
008D5927    40              inc eax  
.  
....略......  
.  
008D5D04    891481          mov dword ptr ds:\[ecx+eax\*4\],edx  
008D5D07  ^ E9 47FFFFFF     jmp 008D5C53  
008D5D0C    FFB5 48D8FFFF   push dword ptr ss:\[ebp\-27B8\]<--//在这个之前要将 008D58AE 处的代码改回原来的样子 je short 008D5921,  
008D5D12    E8 CDB5FDFF     call 008B12E4    <--------      //因为这里要检查前面的代码,而决定是否改变 008D5D17 以后的代码.  
008D5D17    59              pop ecx  
008D5D18    EB 03           jmp short 008D5D1D  
008D5D1A    D6              salc  
008D5D1B    D6              salc  
008D5D1C    8BA1 CCCF8E00   mov esp,dword ptr ds:\[ecx+8ECFCC\]  
008D5D22    8985 40D1FFFF   mov dword ptr ss:\[ebp\-2EC0\],eax  
008D5D28    83BD 40D1FFFF 0>cmp dword ptr ss:\[ebp\-2EC0\],0  
008D5D2F    74 36           je short 008D5D67  
008D5D31    8B85 40D1FFFF   mov eax,dword ptr ss:\[ebp\-2EC0\]  
008D5D37    8338 00         cmp dword ptr ds:\[eax\],0  
008D5D3A    74 2B           je short 008D5D67  
008D5D3C    8B85 40D1FFFF   mov eax,dword ptr ss:\[ebp\-2EC0\]  
008D5D42    8B00            mov eax,dword ptr ds:\[eax\]  
008D5D44    8B00            mov eax,dword ptr ds:\[eax\]  
  
  
  
///////////////////////////////////////////////////////////  
  
1EE4868E    55              push ebp                        //oep  
1EE4868F    8BEC            mov ebp,esp  
1EE48691    53              push ebx  
1EE48692    8B5D 08         mov ebx,dword ptr ss:\[ebp+8\]  
1EE48695    56              push esi  
1EE48696    8B75 0C         mov esi,dword ptr ss:\[ebp+C\]  
1EE48699    57              push edi  
....略......  
//////////////  
  
  
  
  
//  
//在 OEP 处,把 DLL Dump 下来,保存为 DUMPED\_.DLL,并且把 2940000 开始,大小为 20000 的区段 DUMP 下来保存为 2940000.DMP,用 URC 修正 IAT.  
//用修改入口点为 14868E .  
//  
//因为 2940000.DMP 这段是由壳程序动态分配然后填入代码的,当没有壳运行的时候它不存在,但程序又必须要调用这段代码.  
//我们必须想办法将它并入 PE 文件的段中,并修改相应的跳转地址.  
//下面是具体对于 DUMPED\_.DLL 的修改过程.  
//用二进制编辑器打开 2940000.DMP,  
//查找 3C1C,替换为 AC FF,  
//查找 3D1C,替换为 AD FF,  
//保存.  
//这里相应的把跳转地址都改向了 1ED01000-1ED2C3A0 段.  
//  
//用 LDRPE 打开 DUMPED 下来的文件,从文件中载入 294000 那段,将它的 VA 改为 540000(为了配合上面做的修改,使其 VA 为 1F240000),  
//使用 LDRPE 保留 VALIDATE 一项,对 DUMPED\_.DLL 进行 REBUILD.  
//再用 010Editor 打开 DUMPED\_.DLL,选择 1000-2B3A0 范围的段,对于选择的段:  
//查找 C2E3,替换为 52 00  
//查找 C3E3,替换为 53 00  
//这里把所有 JMP 294XXXX 改为了 JMP 1f24XXXX 了.  
//  
  
二.重定表的修复.  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
    像上面的说那样,我没有办法 BP GetModuleHandleA+5 这样下断点,找到相关的代码.于是我用 Armadillo 3.76 加密了一个简单的  
DLL,然后把这个 DLL 也一样的 DUMP 下来;再观察 DUMP 下的 DLL 的重定位表的和原来的区别,发现没有任何不同,只是目录表中的 VA 和大小不同了.  
并且 DUMP 下的 DLL 也有.reloc 段,并且这个段的 VA 和原来的 DLL 一样,只是大小不同了.并且我能过跟踪一个 API 调用来证实了这个想法.  
  
    因此,我得出结论,VA 就是.reloc 的 VA,而大小,因为 Reloc 的数据的本身特征,我们可用 010Editor 打开 DUMPED\_.DLL,通过观察.reloc 数据  
的变化确定它的大小,因此得出重定位表的 VA 是:246000,大小是:00006DBC.  
    用 Lord PE 修改一个这个目录的 VA 和大小,REBUILD PE,搞定.:D  
  
  
三.验证的过程  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
//////////////////////////////////////////////////  
//下面是输出函数 COMSETUP 的代码.  
1ED99C10 >  83EC 24         sub esp,24  
1ED99C13    E8 A858FAFF     call VA\_X.1ED3F4C0    <--- //认证函数,返回 1 表示成功,0 表示失败,请看下面  
1ED99C18    85C0            test eax,eax  
1ED99C1A    75 17           jnz short VA\_X.1ED99C33  
1ED99C1C    8D4424 28       lea eax,dword ptr ss:\[esp+28\]  
....略......  
  
///////////////////////////////////////////////////////////////  
1ED3F4C0    6A FF           push -1  
1ED3F4C2    68 5095E81E     push VA\_X.1EE89550  
1ED3F4C7    64:A1 00000000  mov eax,dword ptr fs:\[0\]  
1ED3F4CD    50              push eax  
1ED3F4CE    64:8925 0000000>mov dword ptr fs:\[0\],esp  
1ED3F4D5    83EC 2C         sub esp,2C  
1ED3F4D8    53              push ebx  
1ED3F4D9    33DB            xor ebx,ebx  
1ED3F4DB    56              push esi  
1ED3F4DC    53              push ebx  
1ED3F4DD    68 43010000     push 143  
1ED3F4E2    68 00FBEF1E     push VA\_X.1EEFFB00                                 ; ASCII "EDL:"  
1ED3F4E7    E8 F4210300     call VA\_X.1ED716E0  
1ED3F4EC    68 28E9EF1E     push VA\_X.1EEFE928                                 ; ASCII "UserName"  
1ED3F4F1    68 00E9EF1E     push VA\_X.1EEFE900                                 ; ASCII "Software\\Whole Tomato\\Visual Assist X"  
.  
....略......  
.  
1ED3F83B    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
1ED3F83F    68 94FDEF1E     push VA\_X.1EEFFD94                                 ; ASCII "DAYSINSTALLED"  
1ED3F844    51              push ecx  
1ED3F845    E8 D67A0900     call VA\_X.1EDD7320  
1ED3F84A    8B10            mov edx,dword ptr ds:\[eax\]  
1ED3F84C    83C4 08         add esp,8  
1ED3F84F    8B4A F8         mov ecx,dword ptr ds:\[edx\-8\]  
1ED3F852    3BCB            cmp ecx,ebx  
1ED3F854    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
1ED3F858    0F944424 0B     sete byte ptr ss:\[esp+B\]  
1ED3F85D    E8 4E5B0800     call VA\_X.1EDC53B0  
1ED3F862    385C24 0B       cmp byte ptr ss:\[esp+B\],bl  
1ED3F866    74 2A           je short VA\_X.1ED3F892        <-----//License 是否正确,正确就跳向 1ED3F892,否则返回 0  
//  
//修改为:  
//1ED3F862    885C24 0B       mov byte ptr ss:\[esp+B\],bl  
//1ED3F866    EB 2A           jmp short VA\_X.1ED3F892  
//  
1ED3F868    A1 FCDDF11E     mov eax,dword ptr ds:\[1EF1DDFC\]  
1ED3F86D    53              push ebx  
1ED3F86E    68 78FBEF1E     push VA\_X.1EEFFB78                                 ; ASCII "License"  
1ED3F873    68 8CFDEF1E     push VA\_X.1EEFFD8C                                 ; ASCII "Error"  
1ED3F878    50              push eax  
1ED3F879    FF15 5C78EB1E   call dword ptr ds:\[1EEB785C\]  
1ED3F87F    5E              pop esi  
1ED3F880    33C0            xor eax,eax  
1ED3F882    5B              pop ebx  
1ED3F883    8B4C24 2C       mov ecx,dword ptr ss:\[esp+2C\]  
1ED3F887    64:890D 0000000>mov dword ptr fs:\[0\],ecx  
1ED3F88E    83C4 38         add esp,38  
1ED3F891    C3              retn  
  
1ED3F892    55              push ebp                    <---//正确的话到这里进行进一步的比较  
1ED3F892    55              push ebp  
1ED3F893    57              push edi  
1ED3F894    8D4C24 34       lea ecx,dword ptr ss:\[esp+34\]  
1ED3F898    68 84FDEF1E     push VA\_X.1EEFFD84                                 ; ASCII "EXPIRED"  
1ED3F89D    51              push ecx  
1ED3F89E    E8 7D7A0900     call VA\_X.1EDD7320  
1ED3F8A3    8BF0            mov esi,eax  
1ED3F8A5    8D5424 38       lea edx,dword ptr ss:\[esp+38\]  
1ED3F8A9    68 94FDEF1E     push VA\_X.1EEFFD94                                 ; ASCII "DAYSINSTALLED"  
1ED3F8AE    52              push edx  
1ED3F8AF    C74424 54 09000>mov dword ptr ss:\[esp+54\],9  
1ED3F8B7    E8 647A0900     call VA\_X.1EDD7320  
1ED3F8BC    8BF8            mov edi,eax  
1ED3F8BE    8D4424 3C       lea eax,dword ptr ss:\[esp+3C\]  
1ED3F8C2    68 78FDEF1E     push VA\_X.1EEFFD78                                 ; ASCII "DAYSLEFT"  
1ED3F8C7    50              push eax  
1ED3F8C8    C64424 5C 0A    mov byte ptr ss:\[esp+5C\],0A  
1ED3F8CD    E8 4E7A0900     call VA\_X.1EDD7320  
1ED3F8D2    83C4 18         add esp,18  
1ED3F8D5    8BE8            mov ebp,eax  
1ED3F8D7    C64424 44 0B    mov byte ptr ss:\[esp+44\],0B  
1ED3F8DC    E8 2F560800     call VA\_X.1EDC4F10  
1ED3F8E1    8B08            mov ecx,dword ptr ds:\[eax\]  
1ED3F8E3    68 70FDEF1E     push VA\_X.1EEFFD70                                 ; ASCII "EDL: "  
1ED3F8E8    894C24 18       mov dword ptr ss:\[esp+18\],ecx  
.  
....略......  
.  
1ED3F9F3    50              push eax  
1ED3F9F4    E8 675B0A00     call va\_x.1EDE5560  
1ED3F9F9    83C4 0C         add esp,0C  
1ED3F9FC    3BC3            cmp eax,ebx  
1ED3F9FE    5F              pop edi  
1ED3F9FF    5D              pop ebp  
1ED3FA00    0F84 6E010000   je va\_x.1ED3FB74      <-------- //已经注册,直接返回真  
  
1ED3FA06    A1 38DEF11E     mov eax,dword ptr ds:\[1EF1DE38\]  
1ED3FA0B    BE 05000000     mov esi,5  
1ED3FA10    3BC6            cmp eax,esi  
1ED3FA36    83C4 0C         add esp,0C  
1ED3FA39    3BC3            cmp eax,ebx  
1ED3FA3B    0F84 33010000   je va\_x.1ED3FB74    <------ //如果没有过期的?  
  
1ED3FA41    6A 01           push 1  
1ED3FA43    E8 488C0900     call va\_x.1EDD8690  <--- //弹出注册窗口  
1ED3FA48    83C4 04         add esp,4  
1ED3FA4B    85C0            test eax,eax    
1ED3FA4D    0F84 FD000000   je va\_x.1ED3FB50<--//如果是取消的话,直接返回假  
1ED3FA53    53              push ebx  
1ED3FA54    68 36010000     push 136  
1ED3FA59    68 00FBEF1E     push va\_x.1EEFFB00                                 ; ASCII "EDL:"  
  
.  
....略......  
.  
1ED3FA87   /75 12           jnz short VA\_X.1ED3FA9B  
1ED3FA89   |E8 D3EB1300     call VA\_X.1EE7E661  
1ED3FA8E   |8B40 08         mov eax,dword ptr ds:\[eax+8\]  
1ED3FA91   |53              push ebx  
1ED3FA92   |53              push ebx  
1ED3FA93   |50              push eax  
1ED3FA94   |E8 97540A00     call VA\_X.1EDE4F30  
1ED3FA99   |EB 10           jmp short VA\_X.1ED3FAAB  
1ED3FA9B   \\E8 C1EB1300     call VA\_X.1EE7E661  
1ED3FAA0    8B40 08         mov eax,dword ptr ds:\[eax+8\]  
1ED3FAA3    53              push ebx  
1ED3FAA4    53              push ebx  
1ED3FAA5    50              push eax  
1ED3FAA6    E8 F54A0A00     call VA\_X.1EDE45A0  
1ED3FAAB    83C4 0C         add esp,0C  
1ED3FAAE    3BC3            cmp eax,ebx  
1ED3FAB0    0F84 BE000000   je VA\_X.1ED3FB74          <-------------- //看看注册码正确跳到返回真,否则看是否过期了.  
/\*修改为:  
1ED3FAAE    8BC3            mov eax,ebx  
1ED3FAB0    E9 BF000000     jmp VA\_X.1ED3FB74  
1ED3FAB5    90              nop  
\*/  
  
  
1ED3FAB6    8D4424 30       lea eax,dword ptr ss:\[esp+30\]    
1ED3FABA    68 64FDEF1E     push VA\_X.1EEFFD64                                 ; ASCII "CLOCKBACK"  
1ED3FABF    50              push eax  
1ED3FAC0    E8 5B780900     call VA\_X.1EDD7320  
1ED3FAC5    8B08            mov ecx,dword ptr ds:\[eax\]  
1ED3FAC7    83C4 08         add esp,8  
1ED3FACA    8B71 F8         mov esi,dword ptr ds:\[ecx\-8\]  
1ED3FACD    8D4C24 30       lea ecx,dword ptr ss:\[esp+30\]  
1ED3FAD1    E8 DA580800     call VA\_X.1EDC53B0  
1ED3FAD6    3BF3            cmp esi,ebx  
1ED3FAD8    74 17           je short VA\_X.1ED3FAF1  
1ED3FADA    8B15 FCDDF11E   mov edx,dword ptr ds:\[1EF1DDFC\]  
1ED3FAE0    52              push edx  
1ED3FAE1    53              push ebx  
1ED3FAE2    68 C8FCEF1E     push VA\_X.1EEFFCC8                                 ; ASCII "Visual Assist was unable to validate your trial. You may need to reboot your system. If the problem persists, email Support@WholeTomato.com.  
  
Error: CBX-3"  
1ED3FAE7    E8 94180300     call VA\_X.1ED71380  
1ED3FAEC    83C4 0C         add esp,0C  
1ED3FAEF    EB 5F           jmp short VA\_X.1ED3FB50  
1ED3FAF1    8D4424 30       lea eax,dword ptr ss:\[esp+30\]  
1ED3FAF5    68 B8FCEF1E     push VA\_X.1EEFFCB8                                 ; ASCII "CLOCKFORWARD"  
1ED3FAFA    50              push eax  
1ED3FAFB    E8 20780900     call VA\_X.1EDD7320  
1ED3FB00    8B08            mov ecx,dword ptr ds:\[eax\]  
1ED3FB02    83C4 08         add esp,8  
1ED3FB05    8B71 F8         mov esi,dword ptr ds:\[ecx\-8\]  
1ED3FB08    8D4C24 30       lea ecx,dword ptr ss:\[esp+30\]  
1ED3FB0C    E8 9F580800     call VA\_X.1EDC53B0  
1ED3FB11    3BF3            cmp esi,ebx  
1ED3FB13    74 17           je short VA\_X.1ED3FB2C  
1ED3FB15    8B15 FCDDF11E   mov edx,dword ptr ds:\[1EF1DDFC\]  
1ED3FB1B    52              push edx  
1ED3FB1C    53              push ebx  
1ED3FB1D    68 1CFCEF1E     push VA\_X.1EEFFC1C                                 ; ASCII "Visual Assist was unable to validate your trial. You may need to reboot your system. If the problem persists, email Support@WholeTomato.com.  
  
Error: CFX-3"  
1ED3FB22    E8 59180300     call VA\_X.1ED71380  
1ED3FB27    83C4 0C         add esp,0C  
1ED3FB2A    EB 24           jmp short VA\_X.1ED3FB50  
1ED3FB2C    8D4424 30       lea eax,dword ptr ss:\[esp+30\]  
1ED3FB30    68 84FDEF1E     push VA\_X.1EEFFD84                                 ; ASCII "EXPIRED"  
1ED3FB35    50              push eax  
1ED3FB36    E8 E5770900     call VA\_X.1EDD7320  
1ED3FB3B    8B08            mov ecx,dword ptr ds:\[eax\]  
1ED3FB3D    83C4 08         add esp,8  
1ED3FB40    8B71 F8         mov esi,dword ptr ds:\[ecx\-8\]  
1ED3FB43    8D4C24 30       lea ecx,dword ptr ss:\[esp+30\]  
1ED3FB47    E8 64580800     call VA\_X.1EDC53B0  
1ED3FB4C    3BF3            cmp esi,ebx  
1ED3FB4E    74 37           je short VA\_X.1ED3FB87    <-- // 如果没有过期的话..返回真?  
  
1ED3FB50    8D4C24 10       lea ecx,dword ptr ss:\[esp+10\]  
1ED3FB54    C74424 3C FFFFF>mov dword ptr ss:\[esp+3C\],-1  
1ED3FB5C    E8 4F580800     call VA\_X.1EDC53B0  
1ED3FB61    5E              pop esi  
1ED3FB62    33C0            xor eax,eax  
1ED3FB64    5B              pop ebx  
1ED3FB65    8B4C24 2C       mov ecx,dword ptr ss:\[esp+2C\]  
1ED3FB69    64:890D 0000000>mov dword ptr fs:\[0\],ecx  
1ED3FB70    83C4 38         add esp,38  
1ED3FB73    C3              retn  
  
1ED3FB74    68 30F2D31E     push VA\_X.1ED3F230  
1ED3FB79    68 E8030000     push 3E8  
1ED3FB7E    6A 02           push 2  
1ED3FB80    53              push ebx  
1ED3FB81    FF15 6477EB1E   call dword ptr ds:\[1EEB7764\]                       ; USER32.SetTimer  
  
1ED3FB87    6A 01           push 1                            //  
1ED3FB89    E8 A2AEFEFF     call VA\_X.1ED2AA30  
1ED3FB8E    6A 01           push 1  
1ED3FB90    E8 8B9B0400     call VA\_X.1ED89720  
1ED3FB95    6A 01           push 1  
1ED3FB97    E8 04870500     call VA\_X.1ED982A0  
1ED3FB9C    6A 01           push 1  
1ED3FB9E    E8 BD060600     call VA\_X.1EDA0260  
1ED3FBA3    6A 01           push 1  
1ED3FBA5    E8 56250700     call VA\_X.1EDB2100  
1ED3FBAA    6A 01           push 1  
1ED3FBAC    E8 2F720700     call VA\_X.1EDB6DE0  
1ED3FBB1    6A 01           push 1  
1ED3FBB3    E8 B8EE0200     call VA\_X.1ED6EA70  
1ED3FBB8    A1 6077F11E     mov eax,dword ptr ds:\[1EF17760\]  
1ED3FBBD    83C4 1C         add esp,1C  
1ED3FBC0    3BC3            cmp eax,ebx  
1ED3FBC2    74 0D           je short VA\_X.1ED3FBD1  
1ED3FBC4    68 00FCEF1E     push VA\_X.1EEFFC00                                 ; ASCII "InitInstance EdDll loaded"  
1ED3FBC9    E8 B21D0300     call VA\_X.1ED71980  
1ED3FBCE    83C4 04         add esp,4  
1ED3FBD1    53              push ebx  
1ED3FBD2    68 99010000     push 199  
1ED3FBD7    68 00FBEF1E     push VA\_X.1EEFFB00                                 ; ASCII "EDL:"  
1ED3FBDC    E8 FF1A0300     call VA\_X.1ED716E0  
1ED3FBE1    83C4 0C         add esp,0C  
1ED3FBE4    8D4C24 10       lea ecx,dword ptr ss:\[esp+10\]  
1ED3FBE8    C74424 3C FFFFF>mov dword ptr ss:\[esp+3C\],-1  
1ED3FBF0    E8 BB570800     call VA\_X.1EDC53B0  
1ED3FBF5    8B4C24 34       mov ecx,dword ptr ss:\[esp+34\]  
1ED3FBF9    5E              pop esi  
1ED3FBFA    B8 01000000     mov eax,1  
1ED3FBFF    5B              pop ebx  
1ED3FC00    64:890D 0000000>mov dword ptr fs:\[0\],ecx  
1ED3FC07    83C4 38         add esp,38  
1ED3FC0A    C3              retn  
1ED3FC0B    90              nop

 4dd466d0e1e0cc123afbe504da2462ca