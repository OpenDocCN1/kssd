

# ASTON Ver1.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50144.htm`](https://www.pediy.com/kssd/pediy05/pediy50144.htm)

1、Asprotect1.22 保护，G 到 OEP（41f544）处直接 Dump 取程序。   
2、按正常方法用 ImportREC 修正 IAT。   
3、运行脱壳后的程序非法操作：

暗桩一 

> 这个程序关键是 PropertySheet 函数的 PROPSHEETHEADER 结构数组，脱壳后与脱壳前该结构值变化了。   
>   
> int PropertySheet(   
>   
>     LPCPROPSHEETHEADER lppsph       
>   );       
>   
>   
>   
> 0167:0041ED31 C743040C010000    MOV    DWord Ptr \[EBX+04\],0000010C //0000010C 传给 PROPSHEETHEADER 结构    
> 0167:0041ED38 33C9              XOR    ECX,ECX   
> 0167:0041ED3A 894B08            MOV    \[EBX+08\],ECX   
> 0167:0041ED3D 8B12              MOV    EDX,\[EDX\]   
> 0167:0041ED3F 89530C            MOV    \[EBX+0C\],EDX   
> 0167:0041ED42 C7431001000000    MOV    DWord Ptr \[EBX+10\],00000001   
> 0167:0041ED49 8B15FC104200      MOV    EDX,\[004210FC\]   
> 0167:0041ED4F 8B12              MOV    EDX,\[EDX\]   
> 0167:0041ED51 895314            MOV    \[EBX+14\],EDX   
> 0167:0041ED54 C7431808000000    MOV    DWord Ptr \[EBX+18\],00000008   
> 0167:0041ED5B 8B1510164200      MOV    EDX,\[00421610\]   
> 0167:0041ED61 8B12              MOV    EDX,\[EDX\]   
> 0167:0041ED63 89531C            MOV    \[EBX+1C\],EDX   
> 0167:0041ED66 894320            MOV    \[EBX+20\],EAX   
> 0167:0041ED69 B8688F4100        MOV    EAX,00418F68   
> 0167:0041ED6E 894324            MOV    \[EBX+24\],EAX   
> 0167:0041ED71 53                PUSH    EBX   
> 0167:0041ED72 55                PUSH    EBP   
> 0167:0041ED73 31D2              XOR    EDX,EDX   
> 0167:0041ED75 89E5              MOV    EBP,ESP   
> 0167:0041ED77 55                PUSH    EBP   
> 0167:0041ED78 68ADED4100        PUSH    0041EDAD //改成 PUSH 0041EDAB，出异常就跳到此，哈哈，爽。   
> 0167:0041ED7D 6890EA4100        PUSH    0041EA90   
> 0167:0041ED82 648B02            MOV    EAX,FS:\[EDX\]  
> 0167:0041ED85 890534224300      MOV    \[00432234\],EAX   
> 0167:0041ED8B 50                PUSH    EAX   
> 0167:0041ED8C 648922            MOV    FS:\[EDX\],ESP //建立 SEH   
> 0167:0041ED8F BA54104200        MOV    EDX,00421054   
> 0167:0041ED94 B90F000000        MOV    ECX,0000000F   
>   
> 下面一段代码 SMC 还原 00421054 处指令：   
> 0167:0041ED99 FE02              INC    Byte Ptr \[EDX\]   
> 0167:0041ED9B 42                INC    EDX   
> 0167:0041ED9C 49                DEC    ECX   
> 0167:0041ED9D 75FA              JNZ    0041ED99   
>   
> 0167:0041ED9F B854104200        MOV    EAX,00421054   
> 0167:0041EDA4 6820114000        PUSH    00401120   
> 0167:0041EDA9 FFD0              CALL    Near EAX//即 CALL  00421054   
>   
> {   
> 以下一段代码，程序用上述 SMC 形成：   
> .........   
> 0167:00421056  MOV    EAX,\[EAX+02\]   
> 0167:00421059  MOV    EAX,\[\`KERNEL32!GetLocalTime\`\]   
> 0167:0042105B  PUSH    DWord Ptr \[\`KERNEL32!GetLocalTime\`\]    
> 0167:0042105D  POP    DWord Ptr \[\`KERNEL32!GetLocalTime\`\]//脱后的程序在此产生异常会跳到 0041EDAD     
> 0167:0042105F  JMP    Near EBX   
> }   
> 0167:0041EDAB EB13              JMP    0041EDC0   
> 0167:0041EDAD 8B053C224300      MOV    EAX,\[0043223C\]   
> 0167:0041EDB3 B9F3000000        MOV    ECX,000000F3   
> 0167:0041EDB8 21C8              AND    EAX,ECX   
> 0167:0041EDBA 89053C224300      MOV    \[0043223C\],EAX//清 PROPSHEETHEADER 结构的值 0000010C 为 0    
> 0167:0041EDC0 BA54104200        MOV    EDX,00421054   
> 0167:0041EDC5 B90F000000        MOV    ECX,0000000F   
> 0167:0041EDCA FE0A              DEC    Byte Ptr \[EDX\]  
>   
> 0167:0041EDCC 42                INC    EDX   
> 0167:0041EDCD 49                DEC    ECX   
> 0167:0041EDCE 75FA              JNZ    0041EDCA   
> 0167:0041EDD0 31D2              XOR    EDX,EDX   
> 0167:0041EDD2 8B0534224300      MOV    EAX,\[00432234\]   
> 0167:0041EDD8 648902            MOV    FS:\[EDX\],EAX   
> 0167:0041EDDB 89EC              MOV    ESP,EBP   
> 0167:0041EDDD 5D                POP    EBP   
> 0167:0041EDDE 5B                POP    EBX   
> 0167:0041EDDF 53                PUSH    EBX   
> 0167:0041EDE0 E89F29FEFF        CALL    00401784//　　CALL　PropertySheet    
> //脱壳后 D EBX 会发现从 PROPSHEETHEADER 结构数据有一处不对，就是 0000010C 值，修正后程序所有的界面出来了，但还是非法操作。  

暗桩二 

> 本想用 hmempcy 找出错的地方，但调用太多了，不成功，后来想到消息函数：   
> hdc=BeginPaint(Hwnd,&ps);   
> 程序用它获取设备描述表，调用的也很频繁的，所以在 0167:0041ED78 后，启用 BeginPaint 断点，中断 10 次，来到如下：   
>   
> 0167:004190A6 50                PUSH    EAX   
> 0167:004190A7 8B4508            MOV    EAX,\[EBP+08\]   
> 0167:004190AA 50                PUSH    EAX   
> 0167:004190AB E88881FEFF        CALL    00401238//调用 BeginPaint 函数   
> 0167:004190B0 8945F4            MOV    \[EBP\-0C\],EAX   
> 0167:004190B3 6A01              PUSH    00000001   
> 0167:004190B5 8B45F4            MOV    EAX,\[EBP\-0C\]   
> 0167:004190B8 50                PUSH    EAX   
> 0167:004190B9 E84A81FEFF        CALL    00401208   
> 0167:004190BE A168104200        MOV    EAX,\[00421068\]   
> 0167:004190C3 50                PUSH    EAX   
> 0167:004190C4 A164104200        MOV    EAX,\[00421064\]   
> 0167:004190C9 50                PUSH    EAX   
> 0167:004190CA E86180FEFF        CALL    00401130   
> 0167:004190CF FFD0              CALL    Near EAX//  这里非法操作！运行这 EAX=000000，你 nop 掉就可成功   
> 0167:004190D1 8945F8            MOV    \[EBP\-08\],EAX   
> 0167:004190D4 8D45F8            LEA    EAX,\[EBP\-08\]   
> 0167:004190D7 50                PUSH    EAX 
> 
>  4dd466d0e1e0cc123afbe504da2462ca