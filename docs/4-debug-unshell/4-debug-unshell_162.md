

# ACProtect 之补区段法(OEP Obfuscation, API redirection)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-486.htm`](https://www.pediy.com/kssd/pediy09/pediy09-486.htm)

> 【文章标题】: ACProtect 之补区段法(OEP Obfuscation, API redirection)  
> 【文章作者】: wynney  
> 【软件名称】: 1 个小东西  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: AntiCrack Protector 1.0x -> RISCO Software Inc.  
> 【使用工具】: Topo  
> 【作者声明】: 论坛冷冷清清，浇浇水：）  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   一、写在前面：  
>   东西很简单，本文所说的只是用一个方法来解决 Stolen Code，没什么技术含量。  
>   关于 ACProtect 的 Stolen Code，以前有不少高人有具体分析，发文章之前，简单搜索了下论坛，看到这篇介绍的很详细。  
>   《simonzh2000 分析的 Acprotect1.21 加密的电脑中国象棋》。本文所介绍的是，如何用“傻瓜式”的方法来解决那些字节  
>   抽取得太多的，而你又没耐性抑或没能力去分析的情况。抛砖引玉，勿见笑。  
>     
>   二、Fix IAT&Go to OEP  
>     
>   忽略所有异常。  
>     
>   00400000   00001000   NgaMy                 PE header     Imag   R         RWE  
>   00401000   00009000   NgaMy      .text      code          Imag   R         RWE  
>   0040A000   00002000   NgaMy      .rdata     code,data     Imag   R         RWE  
>   0040C000   00004000   NgaMy      .data      code          Imag   R         RWE  
>   00410000   0000F000   NgaMy      .rsrc      code,resourc  Imag   R         RWE  
>   0041F000   0001E000   NgaMy      .perplex   code,imports  Imag   R         RWE  
>     
>   .rdata 段，F2，Shift+F9  
>   注：Delphi 和 BCB 的程序是.idata 段  
>     
>   
> 
> 引用: 0043383D     8B46 0C            mov eax,dword ptr ds:\[esi+C\]            ; 停在这里  
>   00433840     0BC0               or eax,eax                              ; 这一段代码相信大家很熟悉了，fly 分析得很详细  
>   00433842     0F84 25020000      je NgaMy.00433A6D  
>   00433848     8366 0C 00         and dword ptr ds:\[esi+C\],0  
>   0043384C     03C2               add eax,edx  
>   0043384E     8BD8               mov ebx,eax  
>   00433850     56                 push esi  
>   00433851     57                 push edi  
>   00433852     50                 push eax  
>   00433853     8BF3               mov esi,ebx  
>   00433855     8BFB               mov edi,ebx  
>   00433857     AC                 lods byte ptr ds:\[esi\]  
>   00433858     C0C0 03            rol al,3  
>   0043385B     AA                 stos byte ptr es:\[edi\]  
>   0043385C     803F 00            cmp byte ptr ds:\[edi\],0  
>   0043385F   ^ 75 F6              jnz short NgaMy.00433857  
>   00433861     58                 pop eax  
>   00433862     5F                 pop edi  
>   00433863     5E                 pop esi  
>   00433864     50                 push eax  
>   00433865     FF95 90E24100      call dword ptr ss:\[ebp+41E290\]  
>   0043386B     0BC0               or eax,eax  
>   0043386D     75 43              jnz short NgaMy.004338B2  
>   0043386F     90                 nop  
>   00433870     90                 nop  
>   00433871     90                 nop  
>   00433872     90                 nop  
>   00433873     53                 push ebx  
>   00433874     FF95 94E24100      call dword ptr ss:\[ebp+41E294\]  
>   0043387A     0BC0               or eax,eax  
>   0043387C     75 34              jnz short NgaMy.004338B2  
>   0043387E     90                 nop  
>   0043387F     90                 nop  
>   00433880     90                 nop  
>   00433881     90                 nop  
>   00433882     8B95 1FFC4000      mov edx,dword ptr ss:\[ebp+40FC1F\]  
>   00433888     0195 1D1F4000      add dword ptr ss:\[ebp+401F1D\],edx  
>   0043388E     0195 211F4000      add dword ptr ss:\[ebp+401F21\],edx  
>   00433894     6A 00              push 0  
>   00433896     FFB5 1D1F4000      push dword ptr ss:\[ebp+401F1D\]  
>   0043389C     FFB5 211F4000      push dword ptr ss:\[ebp+401F21\]  
>   004338A2     6A 00              push 0  
>   004338A4     FF95 9CE24100      call dword ptr ss:\[ebp+41E29C\]  
>   004338AA     6A 00              push 0  
>   004338AC     FF95 98E24100      call dword ptr ss:\[ebp+41E298\]  
>   004338B2     60                 pushad  
>   004338B3     2BC0               sub eax,eax  
>   004338B5     8803               mov byte ptr ds:\[ebx\],al  
>   004338B7     43                 inc ebx  
>   004338B8     3803               cmp byte ptr ds:\[ebx\],al  
>   004338BA   ^ 75 F9              jnz short NgaMy.004338B5  
>   004338BC     61                 popad  
>   004338BD     8985 17FC4000      mov dword ptr ss:\[ebp+40FC17\],eax  
>   004338C3     C785 1BFC4000 0000>mov dword ptr ss:\[ebp+40FC1B\],0  
>   004338CD     8B95 1FFC4000      mov edx,dword ptr ss:\[ebp+40FC1F\]  
>   004338D3     8B06               mov eax,dword ptr ds:\[esi\]  
>   004338D5     0BC0               or eax,eax  
>   004338D7     75 07              jnz short NgaMy.004338E0  
>   004338D9     90                 nop  
>   004338DA     90                 nop  
>   004338DB     90                 nop  
>   004338DC     90                 nop  
>   004338DD     8B46 10            mov eax,dword ptr ds:\[esi+10\]  
>   004338E0     03C2               add eax,edx  
>   004338E2     0385 1BFC4000      add eax,dword ptr ss:\[ebp+40FC1B\]  
>   004338E8     8B18               mov ebx,dword ptr ds:\[eax\]  
>   004338EA     8B7E 10            mov edi,dword ptr ds:\[esi+10\]  
>   004338ED     03FA               add edi,edx  
>   004338EF     03BD 1BFC4000      add edi,dword ptr ss:\[ebp+40FC1B\]  
>   004338F5     85DB               test ebx,ebx  
>   004338F7     0F84 62010000      je NgaMy.00433A5F  
>   004338FD     F7C3 00000080      test ebx,80000000  
>   00433903     75 1D              jnz short NgaMy.00433922  
>   00433905     90                 nop  
>   00433906     90                 nop  
>   00433907     90                 nop  
>   00433908     90                 nop  
>   00433909     03DA               add ebx,edx  
>   0043390B     83C3 02            add ebx,2  
>   0043390E     56                 push esi  
>   0043390F     57                 push edi  
>   00433910     50                 push eax  
>   00433911     8BF3               mov esi,ebx  
>   00433913     8BFB               mov edi,ebx  
>   00433915     AC                 lods byte ptr ds:\[esi\]  
>   00433916     C0C0 03            rol al,3  
>   00433919     AA                 stos byte ptr es:\[edi\]  
>   0043391A     803F 00            cmp byte ptr ds:\[edi\],0  
>   0043391D   ^ 75 F6              jnz short NgaMy.00433915  
>   0043391F     58                 pop eax  
>   00433920     5F                 pop edi  
>   00433921     5E                 pop esi  
>   00433922     3B9D 1FFC4000      cmp ebx,dword ptr ss:\[ebp+40FC1F\]  
>   00433928     7C 11              jl short NgaMy.0043393B  
>   0043392A     90                 nop  
>   0043392B     90                 nop  
>   0043392C     90                 nop  
>   0043392D     90                 nop  
>   0043392E     83BD 02244000 00   cmp dword ptr ss:\[ebp+402402\],0  
>   00433935     75 0A              jnz short NgaMy.00433941  
>   00433937     90                 nop  
>   00433938     90                 nop  
>   00433939     90                 nop  
>   0043393A     90                 nop  
>   0043393B     81E3 FFFFFF0F      and ebx,0FFFFFFF  
>   00433941     53                 push ebx  
>   00433942     FFB5 17FC4000      push dword ptr ss:\[ebp+40FC17\]  
>   00433948     FF95 8CE24100      call dword ptr ss:\[ebp+41E28C\]  
>   0043394E     3B9D 1FFC4000      cmp ebx,dword ptr ss:\[ebp+40FC1F\]  
>   00433954     7C 0F              jl short NgaMy.00433965  
>   00433956     90                 nop  
>   00433957     90                 nop  
>   00433958     90                 nop  
>   00433959     90                 nop  
>   0043395A     60                 pushad  
>   0043395B     2BC0               sub eax,eax  
>   0043395D     8803               mov byte ptr ds:\[ebx\],al  
>   0043395F     43                 inc ebx  
>   00433960     3803               cmp byte ptr ds:\[ebx\],al  
>   00433962   ^ 75 F9              jnz short NgaMy.0043395D  
>   00433964     61                 popad  
>   00433965     0BC0               or eax,eax  
>   00433967   ^ 0F84 15FFFFFF      je NgaMy.00433882  
>   0043396D     3B85 9CE24100      cmp eax,dword ptr ss:\[ebp+41E29C\]       ; 比较是否是 MessageBoxA  
>   00433973     74 20              je short NgaMy.00433995                 ; nop 掉  
>   00433975     90                 nop  
>   00433976     90                 nop  
>   00433977     90                 nop  
>   00433978     90                 nop  
>   00433979     3B85 9D014100      cmp eax,dword ptr ss:\[ebp+41019D\]       ; 比较是否是 RegisterHotKey  
>   0043397F     74 09              je short NgaMy.0043398A                 ; nop 掉  
>   00433981     90                 nop  
>   00433982     90                 nop  
>   00433983     90                 nop  
>   00433984     90                 nop  
>   00433985     EB 14              jmp short NgaMy.0043399B  
>   00433987     90                 nop  
>   00433988     90                 nop  
>   00433989     90                 nop  
>   0043398A     8D85 0A024100      lea eax,dword ptr ss:\[ebp+41020A\]  
>   00433990     EB 09              jmp short NgaMy.0043399B  
>   00433992     90                 nop  
>   00433993     90                 nop  
>   00433994     90                 nop  
>   00433995     8D85 24024100      lea eax,dword ptr ss:\[ebp+410224\]  
>   0043399B     56                 push esi  
>   0043399C     FFB5 17FC4000      push dword ptr ss:\[ebp+40FC17\]  
>   004339A2     5E                 pop esi  
>   004339A3     39B5 FA234000      cmp dword ptr ss:\[ebp+4023FA\],esi  
>   004339A9     74 15              je short NgaMy.004339C0  
>   004339AB     90                 nop  
>   004339AC     90                 nop  
>   004339AD     90                 nop  
>   004339AE     90                 nop  
>   004339AF     39B5 FE234000      cmp dword ptr ss:\[ebp+4023FE\],esi  
>   004339B5     74 09              je short NgaMy.004339C0  
>   004339B7     90                 nop  
>   004339B8     90                 nop  
>   004339B9     90                 nop  
>   004339BA     90                 nop  
>   004339BB     EB 63              jmp short NgaMy.00433A20  
>   004339BD     90                 nop  
>   004339BE     90                 nop  
>   004339BF     90                 nop  
>   004339C0     80BD D2594100 00   cmp byte ptr ss:\[ebp+4159D2\],0  
>   004339C7     74 57              je short NgaMy.00433A20                 ; Magic Jump，改 Jmp  
>   004339C9     90                 nop  
>   004339CA     90                 nop  
>   004339CB     90                 nop  
>   004339CC     90                 nop  
>   004339CD     EB 07              jmp short NgaMy.004339D6  
>   004339CF     90                 nop  
>   004339D0     90                 nop  
>   004339D1     90                 nop  
>   004339D2     0100               add dword ptr ds:\[eax\],eax  
>   004339D4     0000               add byte ptr ds:\[eax\],al  
>   004339D6     8BB5 E4FC4000      mov esi,dword ptr ss:\[ebp+40FCE4\]  
>   004339DC     83C6 0D            add esi,0D  
>   004339DF     81EE EA1B4000      sub esi,NgaMy.00401BEA  
>   004339E5     2BF5               sub esi,ebp  
>   004339E7     83FE 00            cmp esi,0  
>   004339EA     7F 34              jg short NgaMy.00433A20  
>   004339EC     90                 nop  
>   004339ED     90                 nop  
>    
> 
>     
>   作如上修改后，就可以用 ImportREC 来修复 IAT 了，到 00401000 段，下内存访问断点  
>     
>   第 1 次 Shift+F9  
>   
> 
> 引用: 00403D38     68 8C3D4000        push NgaMy.00403D8C                     ; 停在这  
>   00403D3D     64:A1 00000000     mov eax,dword ptr fs:\[0\]  
>   00403D43     50                 push eax  
>   00403D44     8B4424 10          mov eax,dword ptr ss:\[esp+10\]  
>   00403D48     896C24 10          mov dword ptr ss:\[esp+10\],ebp  
>   00403D4C     8D6C24 10          lea ebp,dword ptr ss:\[esp+10\]  
>   00403D50     2BE0               sub esp,eax  
>   00403D52     53                 push ebx  
>   00403D53     56                 push esi  
>   00403D54     57                 push edi  
>   00403D55     8B45 F8            mov eax,dword ptr ss:\[ebp-8\]  
>   00403D58     8965 E8            mov dword ptr ss:\[ebp-18\],esp  
>   00403D5B     50                 push eax  
>   00403D5C     8B45 FC            mov eax,dword ptr ss:\[ebp-4\]  
>   00403D5F     C745 FC FFFFFFFF   mov dword ptr ss:\[ebp-4\],-1  
>   00403D66     8945 F8            mov dword ptr ss:\[ebp-8\],eax  
>   00403D69     8D45 F0            lea eax,dword ptr ss:\[ebp-10\]  
>   00403D6C     64:A3 00000000     mov dword ptr fs:\[0\],eax  
>   00403D72     C3                 retn                                    ; F4，F8  
>    
> 
>     
>   第 2 次 Shift+F9  
>   
> 
> 引用: 00405560     3D 00100000        cmp eax,1000                            ; 停在这  
>   00405565     73 0E              jnb short NgaMy.00405575  
>   00405567     F7D8               neg eax  
>   00405569     03C4               add eax,esp  
>   0040556B     83C0 04            add eax,4  
>   0040556E     8500               test dword ptr ds:\[eax\],eax  
>   00405570     94                 xchg eax,esp  
>   00405571     8B00               mov eax,dword ptr ds:\[eax\]  
>   00405573     50                 push eax  
>   00405574     C3                 retn                                    ; F4，F8  
>    
> 
>     
>   第 3 次 Shift+F9，到达 FOEP  
>   
> 
> 引用:   0040305C     83F9 02            cmp ecx,2                               ; FOEP，上面被抽了不少字节  
>   0040305F     74 0C              je short NgaMy.0040306D  
>   00403061     81CE 00800000      or esi,8000  
>   00403067     8935 B0DE4000      mov dword ptr ds:\[40DEB0\],esi  
>   0040306D     C1E0 08            shl eax,8  
>    
> 
>     
>   到此，脱壳吧，ImportREC 修复，OEP 填 305C，全部有效，Fix Dump。  
>   现在脱壳修复后的文件当然是运行不了的，因为 OEP 没修复嘛：）  
>     
>   三、Find Stolen Code Binary  
>   Ctrl+F12，重新再入  
>   
> 
> 引用:   0041F000 N>  60                 pushad                                  ; EP，F8  
>   0041F001     E8 01000000        call NgaMy.0041F007                     ; ESP=0012FFA4,hr 0012FFA4,Shift+F9  
>    
> 
>     
>   几次 Shift+F9 后来到这里  
>   
> 
> 引用:   004365F4     8915 F5FD4100      mov dword ptr ds:\[41FDF5\],edx           ; 几次 Shift+F9 后来到这里，删除硬件断点  
>   004365FA     FF35 F5FD4100      push dword ptr ds:\[41FDF5\]              ; 这里就是 Stolen Code 开始的地方了  
>   00436600     8F05 2DFE4100      pop dword ptr ds:\[41FE2D\]               ; ntdll.KiFastSystemCallRet  
>   00436606     FF35 2DFE4100      push dword ptr ds:\[41FE2D\]              ; ntdll.KiFastSystemCallRet  
>   0043660C     C70424 60000000    mov dword ptr ss:\[esp\],60  
>   00436613     56                 push esi  
>   00436614     890C24             mov dword ptr ss:\[esp\],ecx  
>   00436617     68 8DFD4100        push NgaMy.0041FD8D  
>   0043661C     59                 pop ecx  
>   0043661D     8919               mov dword ptr ds:\[ecx\],ebx  
>   0043661F     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   00436622     8F05 ADFE4100      pop dword ptr ds:\[41FEAD\]  
>   00436628     FF35 8DFD4100      push dword ptr ds:\[41FD8D\]  
>   0043662E     C70424 48A24000    mov dword ptr ss:\[esp\],NgaMy.0040A248  
>   00436635     8905 B9FD4100      mov dword ptr ds:\[41FDB9\],eax  
>   0043663B     FF35 B9FD4100      push dword ptr ds:\[41FDB9\]  
>   00436641     90                 nop  
>   00436642     90                 nop  
>   00436643     60                 pushad  
>   00436644     E8 01000000        call NgaMy.0043664A                     ; F4，ESP=0012FF98，hr esp，F9  
>     
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   89 15 F5 FD 41 00 FF 35 F5 FD 41 00 8F 05 2D FE 41 00 FF 35 2D FE 41 00 C7 04 24 60 00 00 00 56  
>   89 0C 24 68 8D FD 41 00 59 89 19 8B 0C 24 8F 05 AD FE 41 00 FF 35 8D FD 41 00 C7 04 24 48 A2 40  
>   00 89 05 B9 FD 41 00 FF 35 B9 FD 41 00 90 90  
>    
> 
>     
>   
> 
> 引用:   00436F16     68 1DFD4100        push NgaMy.0041FD1D                     ; 删除硬件断点  
>   00436F1B     58                 pop eax  
>   00436F1C     8930               mov dword ptr ds:\[eax\],esi  
>   00436F1E     8F05 79FC4100      pop dword ptr ds:\[41FC79\]  
>   00436F24     8B05 79FC4100      mov eax,dword ptr ds:\[41FC79\]  
>   00436F2A     FF35 1DFD4100      push dword ptr ds:\[41FD1D\]  
>   00436F30     56                 push esi  
>   00436F31     891C24             mov dword ptr ss:\[esp\],ebx  
>   00436F34     C70424 383D4000    mov dword ptr ss:\[esp\],NgaMy.00403D38  
>   00436F3B     8B3424             mov esi,dword ptr ss:\[esp\]  
>   00436F3E     8F05 A5FE4100      pop dword ptr ds:\[41FEA5\]               ; NgaMy.00403D38  
>   00436F44     8905 01FF4100      mov dword ptr ds:\[41FF01\],eax  
>   00436F4A     FF35 01FF4100      push dword ptr ds:\[41FF01\]  
>   00436F50     891C24             mov dword ptr ss:\[esp\],ebx  
>   00436F53     56                 push esi  
>   00436F54     C70424 45FE4100    mov dword ptr ss:\[esp\],NgaMy.0041FE45  
>   00436F5B     8F05 31FE4100      pop dword ptr ds:\[41FE31\]               ; NgaMy.0041FE45  
>   00436F61     90                 nop  
>   00436F62     90                 nop  
>   00436F63     60                 pushad  
>   00436F64     E8 01000000        call NgaMy.00436F6A                     ; F4，ESP=0012FF94，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   68 1D FD 41 00 58 89 30 8F 05 79 FC 41 00 8B 05 79 FC 41 00 FF 35 1D FD 41 00 56 89 1C 24 C7 04  
>   24 38 3D 40 00 8B 34 24 8F 05 A5 FE 41 00 89 05 01 FF 41 00 FF 35 01 FF 41 00 89 1C 24 56 C7 04  
>   24 45 FE 41 00 8F 05 31 FE 41 00  
>    
> 
>     
>   
> 
> 引用:   0043783F     8B1D 31FE4100      mov ebx,dword ptr ds:\[41FE31\]           ; 删除硬件断点  
>   00437845     8933               mov dword ptr ds:\[ebx\],esi  
>   00437847     8F05 39FC4100      pop dword ptr ds:\[41FC39\]  
>   0043784D     FF35 39FC4100      push dword ptr ds:\[41FC39\]  
>   00437853     5B                 pop ebx  
>   00437854     8F05 09FE4100      pop dword ptr ds:\[41FE09\]  
>   0043785A     891D 21FC4100      mov dword ptr ds:\[41FC21\],ebx  
>   00437860     FF35 21FC4100      push dword ptr ds:\[41FC21\]  
>   00437866     C705 19FC4100 09FE>mov dword ptr ds:\[41FC19\],NgaMy.0041FE0>  
>   00437870     8B1D 19FC4100      mov ebx,dword ptr ds:\[41FC19\]           ; NgaMy.0041FE09  
>   00437876     8B33               mov esi,dword ptr ds:\[ebx\]  
>   00437878     8F05 FDFB4100      pop dword ptr ds:\[41FBFD\]  
>   0043787E     8B1D FDFB4100      mov ebx,dword ptr ds:\[41FBFD\]  
>   00437884     FF15 45FE4100      call dword ptr ds:\[41FE45\]              ; NgaMy.00403D38  
>   0043788A     90                 nop  
>   0043788B     90                 nop  
>   0043788C     60                 pushad  
>   0043788D     E8 01000000        call NgaMy.00437893                     ; F4，ESP=0012FF24，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   8B 1D 31 FE 41 00 89 33 8F 05 39 FC 41 00 FF 35 39 FC 41 00 5B 8F 05 09 FE 41 00 89 1D 21 FC 41  
>   00 FF 35 21 FC 41 00 C7 05 19 FC 41 00 09 FE 41 00 8B 1D 19 FC 41 00 8B 33 8F 05 FD FB 41 00 8B  
>   1D FD FB 41 00 FF 15 45 FE 41 00  
>    
> 
>     
>   
> 
> 引用:   0043813D     890D B1FD4100      mov dword ptr ds:\[41FDB1\],ecx           ; 删除硬件断点  
>   00438143     FF35 B1FD4100      push dword ptr ds:\[41FDB1\]  
>   00438149     8F05 B5FC4100      pop dword ptr ds:\[41FCB5\]  
>   0043814F     FF35 B5FC4100      push dword ptr ds:\[41FCB5\]  
>   00438155     56                 push esi  
>   00438156     BE FDFC4100        mov esi,NgaMy.0041FCFD  
>   0043815B     893E               mov dword ptr ds:\[esi\],edi  
>   0043815D     5E                 pop esi  
>   0043815E     FF35 FDFC4100      push dword ptr ds:\[41FCFD\]              ; ntdll.7C930738  
>   00438164     68 94000000        push 94  
>   00438169     8F05 E5FC4100      pop dword ptr ds:\[41FCE5\]  
>   0043816F     FF35 E5FC4100      push dword ptr ds:\[41FCE5\]  
>   00438175     5F                 pop edi  
>   00438176     893D 3DFE4100      mov dword ptr ds:\[41FE3D\],edi  
>   0043817C     FF35 3DFE4100      push dword ptr ds:\[41FE3D\]  
>   00438182     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   00438185     8F05 7DFE4100      pop dword ptr ds:\[41FE7D\]  
>   0043818B     90                 nop  
>   0043818C     90                 nop  
>   0043818D     60                 pushad  
>   0043818E     50                 push eax                                ; F4，ESP=0012FF1C，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   89 0D B1 FD 41 00 FF 35 B1 FD 41 00 8F 05 B5 FC 41 00 FF 35 B5 FC 41 00 56 BE FD FC 41 00 89 3E  
>   5E FF 35 FD FC 41 00 68 94 00 00 00 8F 05 E5 FC 41 00 FF 35 E5 FC 41 00 5F 89 3D 3D FE 41 00 FF  
>   35 3D FE 41 00 8B 0C 24 8F 05 7D FE 41 00  
>    
> 
>     
>   
> 
> 引用:   00438ACD     8B3C24             mov edi,dword ptr ss:\[esp\]              ; 删除硬件断点  
>   00438AD0     8F05 79FD4100      pop dword ptr ds:\[41FD79\]               ; ntdll.7C930738  
>   00438AD6     8935 25FC4100      mov dword ptr ds:\[41FC25\],esi  
>   00438ADC     FF35 25FC4100      push dword ptr ds:\[41FC25\]  
>   00438AE2     890C24             mov dword ptr ss:\[esp\],ecx  
>   00438AE5     8B3C24             mov edi,dword ptr ss:\[esp\]  
>   00438AE8     8F05 B9FC4100      pop dword ptr ds:\[41FCB9\]  
>   00438AEE     8F05 19FE4100      pop dword ptr ds:\[41FE19\]  
>   00438AF4     8905 89FD4100      mov dword ptr ds:\[41FD89\],eax  
>   00438AFA     FF35 89FD4100      push dword ptr ds:\[41FD89\]  
>   00438B00     57                 push edi  
>   00438B01     BF 19FE4100        mov edi,NgaMy.0041FE19  
>   00438B06     8BC7               mov eax,edi  
>   00438B08     5F                 pop edi  
>   00438B09     8B08               mov ecx,dword ptr ds:\[eax\]  
>   00438B0B     8F05 95FC4100      pop dword ptr ds:\[41FC95\]  
>   00438B11     8B05 95FC4100      mov eax,dword ptr ds:\[41FC95\]  
>   00438B17     53                 push ebx  
>   00438B18     90                 nop  
>   00438B19     90                 nop  
>   00438B1A     60                 pushad  
>   00438B1B     50                 push eax                                ; F4，ESP=0012FF20，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   8B 3C 24 8F 05 79 FD 41 00 89 35 25 FC 41 00 FF 35 25 FC 41 00 89 0C 24 8B 3C 24 8F 05 B9 FC 41  
>   00 8F 05 19 FE 41 00 89 05 89 FD 41 00 FF 35 89 FD 41 00 57 BF 19 FE 41 00 8B C7 5F 8B 08 8F 05  
>   95 FC 41 00 8B 05 95 FC 41 00 53  
>    
> 
>     
>   
> 
> 引用:   004393FF     8F05 5DFE4100      pop dword ptr ds:\[41FE5D\]               ; 删除硬件断点  
>   00439405     FF35 5DFE4100      push dword ptr ds:\[41FE5D\]  
>   0043940B     890C24             mov dword ptr ss:\[esp\],ecx  
>   0043940E     893D 91FE4100      mov dword ptr ds:\[41FE91\],edi  
>   00439414     FF35 91FE4100      push dword ptr ds:\[41FE91\]  
>   0043941A     8F05 81FC4100      pop dword ptr ds:\[41FC81\]  
>   00439420     891D 89FE4100      mov dword ptr ds:\[41FE89\],ebx  
>   00439426     FF35 89FE4100      push dword ptr ds:\[41FE89\]  
>   0043942C     68 81FC4100        push NgaMy.0041FC81  
>   00439431     5B                 pop ebx  
>   00439432     8B0B               mov ecx,dword ptr ds:\[ebx\]  
>   00439434     8F05 C9FC4100      pop dword ptr ds:\[41FCC9\]  
>   0043943A     8B1D C9FC4100      mov ebx,dword ptr ds:\[41FCC9\]  
>   00439440     57                 push edi  
>   00439441     890424             mov dword ptr ss:\[esp\],eax  
>   00439444     890C24             mov dword ptr ss:\[esp\],ecx  
>   00439447     8B0424             mov eax,dword ptr ss:\[esp\]  
>   0043944A     90                 nop  
>   0043944B     90                 nop  
>   0043944C     60                 pushad  
>   0043944D     76 03              jbe short NgaMy.00439452                ; F4，ESP=0012FF1C，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   8F 05 5D FE 41 00 FF 35 5D FE 41 00 89 0C 24 89 3D 91 FE 41 00 FF 35 91 FE 41 00 8F 05 81 FC 41  
>   00 89 1D 89 FE 41 00 FF 35 89 FE 41 00 68 81 FC 41 00 5B 8B 0B 8F 05 C9 FC 41 00 8B 1D C9 FC 41  
>   00 57 89 04 24 89 0C 24 8B 04 24  
>    
> 
>     
>   
> 
> 引用:   00439D39     8F05 D5FD4100      pop dword ptr ds:\[41FDD5\]  
>   00439D3F     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   00439D42     8F05 4DFC4100      pop dword ptr ds:\[41FC4D\]  
>   00439D48     50                 push eax  
>   00439D49     891424             mov dword ptr ss:\[esp\],edx  
>   00439D4C     8F05 BDFE4100      pop dword ptr ds:\[41FEBD\]               ; ntdll.KiFastSystemCallRet  
>   00439D52     FF35 BDFE4100      push dword ptr ds:\[41FEBD\]              ; ntdll.KiFastSystemCallRet  
>   00439D58     51                 push ecx  
>   00439D59     B9 DDFD4100        mov ecx,NgaMy.0041FDDD  
>   00439D5E     8939               mov dword ptr ds:\[ecx\],edi  
>   00439D60     59                 pop ecx  
>   00439D61     FF35 DDFD4100      push dword ptr ds:\[41FDDD\]  
>   00439D67     C705 A9FE4100 6055>mov dword ptr ds:\[41FEA9\],NgaMy.0040556>  
>   00439D71     FF35 A9FE4100      push dword ptr ds:\[41FEA9\]              ; NgaMy.00405560  
>   00439D77     8B3C24             mov edi,dword ptr ss:\[esp\]  
>   00439D7A     8F05 95FD4100      pop dword ptr ds:\[41FD95\]               ; NgaMy.00405560  
>   00439D80     891D 29FD4100      mov dword ptr ds:\[41FD29\],ebx  
>   00439D86     90                 nop  
>   00439D87     90                 nop  
>   00439D88     60                 pushad  
>   00439D89     E8 01000000        call NgaMy.00439D8F                     ; F4，ESP=0012FF1C，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   8F 05 D5 FD 41 00 8B 0C 24 8F 05 4D FC 41 00 50 89 14 24 8F 05 BD FE 41 00 FF 35 BD FE 41 00 51  
>   B9 DD FD 41 00 89 39 59 FF 35 DD FD 41 00 C7 05 A9 FE 41 00 60 55 40 00 FF 35 A9 FE 41 00 8B 3C  
>   24 8F 05 95 FD 41 00  
>    
> 
>     
>   
> 
> 引用:   0043A6FB     FF35 29FD4100      push dword ptr ds:\[41FD29\]              ; 删除硬件断点  
>   0043A701     8BDF               mov ebx,edi  
>   0043A703     8BD3               mov edx,ebx  
>   0043A705     5B                 pop ebx  
>   0043A706     8F05 E9FE4100      pop dword ptr ds:\[41FEE9\]  
>   0043A70C     8B3D E9FE4100      mov edi,dword ptr ds:\[41FEE9\]  
>   0043A712     52                 push edx  
>   0043A713     891C24             mov dword ptr ss:\[esp\],ebx  
>   0043A716     68 9DFE4100        push NgaMy.0041FE9D  
>   0043A71B     5B                 pop ebx  
>   0043A71C     8913               mov dword ptr ds:\[ebx\],edx  
>   0043A71E     8B1C24             mov ebx,dword ptr ss:\[esp\]  
>   0043A721     8F05 49FE4100      pop dword ptr ds:\[41FE49\]  
>   0043A727     8B1424             mov edx,dword ptr ss:\[esp\]  
>   0043A72A     8F05 69FD4100      pop dword ptr ds:\[41FD69\]               ; ntdll.KiFastSystemCallRet  
>   0043A730     FF15 9DFE4100      call dword ptr ds:\[41FE9D\]              ; NgaMy.00405560  
>   0043A736     8965 E8            mov dword ptr ss:\[ebp-18\],esp  
>   0043A739     8925 C5FD4100      mov dword ptr ds:\[41FDC5\],esp  
>   0043A73F     891D 21FD4100      mov dword ptr ds:\[41FD21\],ebx  
>   0043A745     FF35 21FD4100      push dword ptr ds:\[41FD21\]  
>   0043A74B     60                 pushad  
>   0043A74C     74 03              je short NgaMy.0043A751                 ; F4，ESP=0012FF8C，hr esp，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   FF 35 29 FD 41 00 8B DF 8B D3 5B 8F 05 E9 FE 41 00 8B 3D E9 FE 41 00 52 89 1C 24 68 9D FE 41 00  
>   5B 89 13 8B 1C 24 8F 05 49 FE 41 00 8B 14 24 8F 05 69 FD 41 00 FF 15 9D FE 41 00 89 65 E8 89 25  
>   C5 FD 41 00 89 1D 21 FD 41 00 FF 35 21 FD 41 00  
>    
> 
>     
>   
> 
> 引用:   0043B097     68 C5FD4100        push NgaMy.0041FDC5  
>   0043B09C     5B                 pop ebx  
>   0043B09D     8B33               mov esi,dword ptr ds:\[ebx\]  
>   0043B09F     8B1C24             mov ebx,dword ptr ss:\[esp\]  
>   0043B0A2     8F05 A9FC4100      pop dword ptr ds:\[41FCA9\]  
>   0043B0A8     893E               mov dword ptr ds:\[esi\],edi  
>   0043B0AA     57                 push edi  
>   0043B0AB     8F05 F5FE4100      pop dword ptr ds:\[41FEF5\]  
>   0043B0B1     FF35 F5FE4100      push dword ptr ds:\[41FEF5\]  
>   0043B0B7     893424             mov dword ptr ss:\[esp\],esi  
>   0043B0BA     FF15 BCA04000      call dword ptr ds:\[40A0BC\]              ; NgaMy.0041F23F  
>   0043B0C0     8B4E 10            mov ecx,dword ptr ds:\[esi+10\]  
>   0043B0C3     50                 push eax  
>   0043B0C4     B8 F9FB4100        mov eax,NgaMy.0041FBF9  
>   0043B0C9     8910               mov dword ptr ds:\[eax\],edx  
>   0043B0CB     58                 pop eax  
>   0043B0CC     FF35 F9FB4100      push dword ptr ds:\[41FBF9\]  
>   0043B0D2     56                 push esi  
>   0043B0D3     C70424 ACDE4000    mov dword ptr ss:\[esp\],NgaMy.0040DEAC  
>   0043B0DA     8B1424             mov edx,dword ptr ss:\[esp\]  
>   0043B0DD     8F05 ADFD4100      pop dword ptr ds:\[41FDAD\]               ; NgaMy.0040DEAC  
>   0043B0E3     890A               mov dword ptr ds:\[edx\],ecx  
>   0043B0E5     90                 nop  
>   0043B0E6     90                 nop  
>   0043B0E7     60                 pushad  
>   0043B0E8     E8 01000000        call NgaMy.0043B0EE                     ; F4，ESP=0012FF8C，F9  
>     
>   二进制复制 pushad 上面的代码\[nop 可以不复制\]  
>   68 C5 FD 41 00 5B 8B 33 8B 1C 24 8F 05 A9 FC 41 00 89 3E 57 8F 05 F5 FE 41 00 FF 35 F5 FE 41 00  
>   89 34 24 FF 15 BC A0 40 00 8B 4E 10 50 B8 F9 FB 41 00 89 10 58 FF 35 F9 FB 41 00 56 C7 04 24 AC  
>   DE 40 00 8B 14 24 8F 05 AD FD 41 00 89 0A  
>    
> 
>     
>   
> 
> 引用:   0043B9DA     8F05 29FE4100      pop dword ptr ds:\[41FE29\]               ; 删除硬件断点  
>   0043B9E0     FF35 29FE4100      push dword ptr ds:\[41FE29\]  
>   0043B9E6     5A                 pop edx  
>   0043B9E7     8B46 04            mov eax,dword ptr ds:\[esi+4\]  
>   0043B9EA     A3 B8DE4000        mov dword ptr ds:\[40DEB8\],eax  
>   0043B9EF     8B56 08            mov edx,dword ptr ds:\[esi+8\]  
>   0043B9F2     52                 push edx  
>   0043B9F3     8F05 3DFD4100      pop dword ptr ds:\[41FD3D\]  
>   0043B9F9     FF35 3DFD4100      push dword ptr ds:\[41FD3D\]  
>   0043B9FF     8F05 BCDE4000      pop dword ptr ds:\[40DEBC\]  
>   0043BA05     8B76 0C            mov esi,dword ptr ds:\[esi+C\]  
>   0043BA08     81E6 FF7F0000      and esi,7FFF  
>   0043BA0E     53                 push ebx  
>   0043BA0F     BB 35FE4100        mov ebx,NgaMy.0041FE35                  ; ASCII "(  
> "  
>   0043BA14     8933               mov dword ptr ds:\[ebx\],esi  
>   0043BA16     5B                 pop ebx  
>   0043BA17     FF35 35FE4100      push dword ptr ds:\[41FE35\]  
>   0043BA1D     8F05 B0DE4000      pop dword ptr ds:\[40DEB0\]  
>   0043BA23     90                 nop  
>   0043BA24     90                 nop  
>   0043BA25     60                 pushad  
>   0043BA26     E8 01000000        call NgaMy.0043BA2C                     ; F4，ESP=0012FF90，hr esp，F9  
>     
>   8F 05 29 FE 41 00 FF 35 29 FE 41 00 5A 8B 46 04 A3 B8 DE 40 00 8B 56 08 52 8F 05 3D FD 41 00 FF  
>   35 3D FD 41 00 8F 05 BC DE 40 00 8B 76 0C 81 E6 FF 7F 00 00 53 BB 35 FE 41 00 89 33 5B FF 35 35  
>   FE 41 00 8F 05 B0 DE 40 00  
>    
> 
>     
>   0043BE77    /EB 01              jmp short NgaMy.0043BE7A  
>     
>   0043BE7A   - FF25 BCBE4300      jmp dword ptr ds:\[43BEBC\]               ; 跳向 FOEP  
>     
>   以上的二进制代码就是所有的 Stolen Code 了，汇总下吧  
>     
>   
> 
> 引用:   89 15 F5 FD 41 00 FF 35 F5 FD 41 00 8F 05 2D FE 41 00 FF 35 2D FE 41 00 C7 04 24 60 00 00 00 56  
>   89 0C 24 68 8D FD 41 00 59 89 19 8B 0C 24 8F 05 AD FE 41 00 FF 35 8D FD 41 00 C7 04 24 48 A2 40  
>   00 89 05 B9 FD 41 00 FF 35 B9 FD 41 00 90 90 68 1D FD 41 00 58 89 30 8F 05 79 FC 41 00 8B 05 79  
>   FC 41 00 FF 35 1D FD 41 00 56 89 1C 24 C7 04 24 38 3D 40 00 8B 34 24 8F 05 A5 FE 41 00 89 05 01  
>   FF 41 00 FF 35 01 FF 41 00 89 1C 24 56 C7 04 24 45 FE 41 00 8F 05 31 FE 41 00 90 90 8B 1D 31 FE  
>   41 00 89 33 8F 05 39 FC 41 00 FF 35 39 FC 41 00 5B 8F 05 09 FE 41 00 89 1D 21 FC 41 00 FF 35 21  
>   FC 41 00 C7 05 19 FC 41 00 09 FE 41 00 8B 1D 19 FC 41 00 8B 33 8F 05 FD FB 41 00 8B 1D FD FB 41  
>   00 FF 15 45 FE 41 00 90 90 89 0D B1 FD 41 00 FF 35 B1 FD 41 00 8F 05 B5 FC 41 00 FF 35 B5 FC 41  
>   00 56 BE FD FC 41 00 89 3E 5E FF 35 FD FC 41 00 68 94 00 00 00 8F 05 E5 FC 41 00 FF 35 E5 FC 41  
>   00 5F 89 3D 3D FE 41 00 FF 35 3D FE 41 00 8B 0C 24 8F 05 7D FE 41 00 90 90 55 8F 05 E9 FD 41 00  
>   8B 3C 24 8F 05 79 FD 41 00 89 35 25 FC 41 00 FF 35 25 FC 41 00 89 0C 24 8B 3C 24 8F 05 B9 FC 41  
>   00 8F 05 19 FE 41 00 89 05 89 FD 41 00 FF 35 89 FD 41 00 57 BF 19 FE 41 00 8B C7 5F 8B 08 8F 05  
>   95 FC 41 00 8B 05 95 FC 41 00 53 90 90 8F 05 5D FE 41 00 FF 35 5D FE 41 00 89 0C 24 89 3D 91 FE  
>   41 00 FF 35 91 FE 41 00 8F 05 81 FC 41 00 89 1D 89 FE 41 00 FF 35 89 FE 41 00 68 81 FC 41 00 5B  
>   8B 0B 8F 05 C9 FC 41 00 8B 1D C9 FC 41 00 57 89 04 24 89 0C 24 8B 04 24 90 90 8F 05 D5 FD 41 00  
>   8B 0C 24 8F 05 4D FC 41 00 50 89 14 24 8F 05 BD FE 41 00 FF 35 BD FE 41 00 51 B9 DD FD 41 00 89  
>   39 59 FF 35 DD FD 41 00 C7 05 A9 FE 41 00 60 55 40 00 FF 35 A9 FE 41 00 8B 3C 24 8F 05 95 FD 41  
>   00 89 1D 29 FD 41 00 90 90 FF 35 29 FD 41 00 8B DF 8B D3 5B 8F 05 E9 FE 41 00 8B 3D E9 FE 41 00  
>   52 89 1C 24 68 9D FE 41 00 5B 89 13 8B 1C 24 8F 05 49 FE 41 00 8B 14 24 8F 05 69 FD 41 00 FF 15  
>   9D FE 41 00 89 65 E8 89 25 C5 FD 41 00 89 1D 21 FD 41 00 FF 35 21 FD 41 00  
>    
> 
>     
>   四、Fix Stolen Code  
>     
>   打开 Topo-->选择上面脱壳后修复的文件-->选择“创建一个新的区段”-->在新增加的字节中填写 1000 吧\[这个大小够我们写代码了\]-->  
>   -->点执行-->提示新增加的区段起始地址为 0043E000-->此时 topo 将备份原程序  
>     
>   OD 打开 topo 修改的程序，Ctrl+G：0043E000，粘贴以上二进制  
>     
>   
> 
> 引用:   0043E000 U>  8915 F5FD4100      mov dword ptr ds:\[41FDF5\],edx           ; ntdll.KiFastSystemCallRet  
>   0043E006     FF35 F5FD4100      push dword ptr ds:\[41FDF5\]              ; ntdll.KiFastSystemCallRet  
>   0043E00C     8F05 2DFE4100      pop dword ptr ds:\[41FE2D\]               ; ntdll.KiFastSystemCallRet  
>   0043E012     FF35 2DFE4100      push dword ptr ds:\[41FE2D\]              ; ntdll.KiFastSystemCallRet  
>   0043E018     C70424 60000000    mov dword ptr ss:\[esp\],60  
>   0043E01F     56                 push esi  
>   0043E020     890C24             mov dword ptr ss:\[esp\],ecx  
>   0043E023     68 8DFD4100        push UnPacked.0041FD8D  
>   0043E028     59                 pop ecx  
>   0043E029     8919               mov dword ptr ds:\[ecx\],ebx  
>   0043E02B     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   0043E02E     8F05 ADFE4100      pop dword ptr ds:\[41FEAD\]  
>   0043E034     FF35 8DFD4100      push dword ptr ds:\[41FD8D\]  
>   0043E03A     C70424 48A24000    mov dword ptr ss:\[esp\],UnPacked.0040A24>  
>   0043E041     8905 B9FD4100      mov dword ptr ds:\[41FDB9\],eax  
>   0043E047     FF35 B9FD4100      push dword ptr ds:\[41FDB9\]  
>   0043E04D     90                 nop  
>   0043E04E     90                 nop  
>   0043E04F     68 1DFD4100        push UnPacked.0041FD1D  
>   0043E054     58                 pop eax  
>   0043E055     8930               mov dword ptr ds:\[eax\],esi  
>   0043E057     8F05 79FC4100      pop dword ptr ds:\[41FC79\]  
>   0043E05D     8B05 79FC4100      mov eax,dword ptr ds:\[41FC79\]  
>   0043E063     FF35 1DFD4100      push dword ptr ds:\[41FD1D\]  
>   0043E069     56                 push esi  
>   0043E06A     891C24             mov dword ptr ss:\[esp\],ebx  
>   0043E06D     C70424 383D4000    mov dword ptr ss:\[esp\],UnPacked.00403D3>  
>   0043E074     8B3424             mov esi,dword ptr ss:\[esp\]  
>   0043E077     8F05 A5FE4100      pop dword ptr ds:\[41FEA5\]               ; UnPacked.00403D38  
>   0043E07D     8905 01FF4100      mov dword ptr ds:\[41FF01\],eax  
>   0043E083     FF35 01FF4100      push dword ptr ds:\[41FF01\]  
>   0043E089     891C24             mov dword ptr ss:\[esp\],ebx  
>   0043E08C     56                 push esi  
>   0043E08D     C70424 45FE4100    mov dword ptr ss:\[esp\],UnPacked.0041FE4>  
>   0043E094     8F05 31FE4100      pop dword ptr ds:\[41FE31\]               ; UnPacked.0041FE45  
>   0043E09A     90                 nop  
>   0043E09B     90                 nop  
>   0043E09C     8B1D 31FE4100      mov ebx,dword ptr ds:\[41FE31\]           ; UnPacked.0041FE45  
>   0043E0A2     8933               mov dword ptr ds:\[ebx\],esi  
>   0043E0A4     8F05 39FC4100      pop dword ptr ds:\[41FC39\]  
>   0043E0AA     FF35 39FC4100      push dword ptr ds:\[41FC39\]  
>   0043E0B0     5B                 pop ebx  
>   0043E0B1     8F05 09FE4100      pop dword ptr ds:\[41FE09\]  
>   0043E0B7     891D 21FC4100      mov dword ptr ds:\[41FC21\],ebx  
>   0043E0BD     FF35 21FC4100      push dword ptr ds:\[41FC21\]  
>   0043E0C3     C705 19FC4100 09FE>mov dword ptr ds:\[41FC19\],UnPacked.0041>  
>   0043E0CD     8B1D 19FC4100      mov ebx,dword ptr ds:\[41FC19\]           ; UnPacked.0041FE09  
>   0043E0D3     8B33               mov esi,dword ptr ds:\[ebx\]  
>   0043E0D5     8F05 FDFB4100      pop dword ptr ds:\[41FBFD\]  
>   0043E0DB     8B1D FDFB4100      mov ebx,dword ptr ds:\[41FBFD\]  
>   0043E0E1     FF15 45FE4100      call dword ptr ds:\[41FE45\]              ; UnPacked.00403D38  
>   0043E0E7     90                 nop  
>   0043E0E8     90                 nop  
>   0043E0E9     890D B1FD4100      mov dword ptr ds:\[41FDB1\],ecx  
>   0043E0EF     FF35 B1FD4100      push dword ptr ds:\[41FDB1\]  
>   0043E0F5     8F05 B5FC4100      pop dword ptr ds:\[41FCB5\]  
>   0043E0FB     FF35 B5FC4100      push dword ptr ds:\[41FCB5\]  
>   0043E101     56                 push esi  
>   0043E102     BE FDFC4100        mov esi,UnPacked.0041FCFD  
>   0043E107     893E               mov dword ptr ds:\[esi\],edi  
>   0043E109     5E                 pop esi  
>   0043E10A     FF35 FDFC4100      push dword ptr ds:\[41FCFD\]  
>   0043E110     68 94000000        push 94  
>   0043E115     8F05 E5FC4100      pop dword ptr ds:\[41FCE5\]  
>   0043E11B     FF35 E5FC4100      push dword ptr ds:\[41FCE5\]  
>   0043E121     5F                 pop edi  
>   0043E122     893D 3DFE4100      mov dword ptr ds:\[41FE3D\],edi  
>   0043E128     FF35 3DFE4100      push dword ptr ds:\[41FE3D\]  
>   0043E12E     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   0043E131     8F05 7DFE4100      pop dword ptr ds:\[41FE7D\]  
>   0043E137     90                 nop  
>   0043E138     90                 nop  
>   0043E139     55                 push ebp  
>   0043E13A     8F05 E9FD4100      pop dword ptr ds:\[41FDE9\]  
>   0043E140     8B3C24             mov edi,dword ptr ss:\[esp\]  
>   0043E143     8F05 79FD4100      pop dword ptr ds:\[41FD79\]  
>   0043E149     8935 25FC4100      mov dword ptr ds:\[41FC25\],esi  
>   0043E14F     FF35 25FC4100      push dword ptr ds:\[41FC25\]  
>   0043E155     890C24             mov dword ptr ss:\[esp\],ecx  
>   0043E158     8B3C24             mov edi,dword ptr ss:\[esp\]  
>   0043E15B     8F05 B9FC4100      pop dword ptr ds:\[41FCB9\]  
>   0043E161     8F05 19FE4100      pop dword ptr ds:\[41FE19\]  
>   0043E167     8905 89FD4100      mov dword ptr ds:\[41FD89\],eax  
>   0043E16D     FF35 89FD4100      push dword ptr ds:\[41FD89\]  
>   0043E173     57                 push edi  
>   0043E174     BF 19FE4100        mov edi,UnPacked.0041FE19  
>   0043E179     8BC7               mov eax,edi  
>   0043E17B     5F                 pop edi  
>   0043E17C     8B08               mov ecx,dword ptr ds:\[eax\]  
>   0043E17E     8F05 95FC4100      pop dword ptr ds:\[41FC95\]  
>   0043E184     8B05 95FC4100      mov eax,dword ptr ds:\[41FC95\]  
>   0043E18A     53                 push ebx  
>   0043E18B     90                 nop  
>   0043E18C     90                 nop  
>   0043E18D     8F05 5DFE4100      pop dword ptr ds:\[41FE5D\]  
>   0043E193     FF35 5DFE4100      push dword ptr ds:\[41FE5D\]  
>   0043E199     890C24             mov dword ptr ss:\[esp\],ecx  
>   0043E19C     893D 91FE4100      mov dword ptr ds:\[41FE91\],edi  
>   0043E1A2     FF35 91FE4100      push dword ptr ds:\[41FE91\]  
>   0043E1A8     8F05 81FC4100      pop dword ptr ds:\[41FC81\]  
>   0043E1AE     891D 89FE4100      mov dword ptr ds:\[41FE89\],ebx  
>   0043E1B4     FF35 89FE4100      push dword ptr ds:\[41FE89\]  
>   0043E1BA     68 81FC4100        push UnPacked.0041FC81  
>   0043E1BF     5B                 pop ebx  
>   0043E1C0     8B0B               mov ecx,dword ptr ds:\[ebx\]  
>   0043E1C2     8F05 C9FC4100      pop dword ptr ds:\[41FCC9\]  
>   0043E1C8     8B1D C9FC4100      mov ebx,dword ptr ds:\[41FCC9\]  
>   0043E1CE     57                 push edi  
>   0043E1CF     890424             mov dword ptr ss:\[esp\],eax  
>   0043E1D2     890C24             mov dword ptr ss:\[esp\],ecx  
>   0043E1D5     8B0424             mov eax,dword ptr ss:\[esp\]  
>   0043E1D8     90                 nop  
>   0043E1D9     90                 nop  
>   0043E1DA     8F05 D5FD4100      pop dword ptr ds:\[41FDD5\]  
>   0043E1E0     8B0C24             mov ecx,dword ptr ss:\[esp\]  
>   0043E1E3     8F05 4DFC4100      pop dword ptr ds:\[41FC4D\]  
>   0043E1E9     50                 push eax  
>   0043E1EA     891424             mov dword ptr ss:\[esp\],edx  
>   0043E1ED     8F05 BDFE4100      pop dword ptr ds:\[41FEBD\]  
>   0043E1F3     FF35 BDFE4100      push dword ptr ds:\[41FEBD\]  
>   0043E1F9     51                 push ecx  
>   0043E1FA     B9 DDFD4100        mov ecx,UnPacked.0041FDDD  
>   0043E1FF     8939               mov dword ptr ds:\[ecx\],edi  
>   0043E201     59                 pop ecx  
>   0043E202     FF35 DDFD4100      push dword ptr ds:\[41FDDD\]  
>   0043E208     C705 A9FE4100 6055>mov dword ptr ds:\[41FEA9\],UnPacked.0040>  
>   0043E212     FF35 A9FE4100      push dword ptr ds:\[41FEA9\]  
>   0043E218     8B3C24             mov edi,dword ptr ss:\[esp\]  
>   0043E21B     8F05 95FD4100      pop dword ptr ds:\[41FD95\]  
>   0043E221     891D 29FD4100      mov dword ptr ds:\[41FD29\],ebx  
>   0043E227     90                 nop  
>   0043E228     90                 nop  
>   0043E229     FF35 29FD4100      push dword ptr ds:\[41FD29\]  
>   0043E22F     8BDF               mov ebx,edi  
>   0043E231     8BD3               mov edx,ebx  
>   0043E233     5B                 pop ebx  
>   0043E234     8F05 E9FE4100      pop dword ptr ds:\[41FEE9\]  
>   0043E23A     8B3D E9FE4100      mov edi,dword ptr ds:\[41FEE9\]  
>   0043E240     52                 push edx  
>   0043E241     891C24             mov dword ptr ss:\[esp\],ebx  
>   0043E244     68 9DFE4100        push UnPacked.0041FE9D  
>   0043E249     5B                 pop ebx  
>   0043E24A     8913               mov dword ptr ds:\[ebx\],edx  
>   0043E24C     8B1C24             mov ebx,dword ptr ss:\[esp\]  
>   0043E24F     8F05 49FE4100      pop dword ptr ds:\[41FE49\]  
>   0043E255     8B1424             mov edx,dword ptr ss:\[esp\]  
>   0043E258     8F05 69FD4100      pop dword ptr ds:\[41FD69\]  
>   0043E25E     FF15 9DFE4100      call dword ptr ds:\[41FE9D\]  
>   0043E264     8965 E8            mov dword ptr ss:\[ebp-18\],esp  
>   0043E267     8925 C5FD4100      mov dword ptr ds:\[41FDC5\],esp  
>   0043E26D     891D 21FD4100      mov dword ptr ds:\[41FD21\],ebx  
>   0043E273     FF35 21FD4100      push dword ptr ds:\[41FD21\]  
>   0043E279     90                 nop  
>   0043E27A     90                 nop  
>   0043E27B   - E9 DC4DFCFF        jmp UnPacked.0040305C                   ; 直接跳向 FOEP  
>    
> 
>   复制以上修改，保存为另外一个文件吧  
>   打开 LordPE，修正 OEP 为 3E000，只选上“验证 PE”重建 PE。到此脱壳完成。  
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca