
```
00402DC6   .  FF15 D4104000 call    dword ptr [<&MSVBVM60.__vbaFileO>;  MSVBVM60.__vbaFileOpen  

00402DCC   .  8B06          mov     eax, dword ptr [esi]            ;  //打开文件，准备写入  

00402DCE   .  56            push    esi  

00402DCF   .  FF90 00030000 call    dword ptr [eax+300]  

00402DD5   .  8D4D A0       lea     ecx, dword ptr [ebp-60]  

00402DD8   .  50            push    eax  

00402DD9   .  51            push    ecx  

00402DDA   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaObjSe>;  MSVBVM60.__vbaObjSet  

00402DE0   .  8BF8          mov     edi, eax  

00402DE2   .  8D45 A8       lea     eax, dword ptr [ebp-58]  

00402DE5   .  50            push    eax  

00402DE6   .  57            push    edi  

00402DE7   .  8B17          mov     edx, dword ptr [edi]  

00402DE9   .  FF92 A0000000 call    dword ptr [edx+A0]  

00402DEF   .  85C0          test    eax, eax  

00402DF1   .  DBE2          fclex  

00402DF3   .  7D 12         jge     short 00402E07  

00402DF5   .  68 A0000000   push    0A0  

00402DFA   .  68 10264000   push    00402610  

00402DFF   .  57            push    edi  

00402E00   .  50            push    eax  

00402E01   .  FF15 44104000 call    dword ptr [<&MSVBVM60.__vbaHresu>;  MSVBVM60.__vbaHresultCheckObj  

00402E07   >  8B4D A8       mov     ecx, dword ptr [ebp-58]  

00402E0A   .  51            push    ecx  

00402E0B   .  FF15 1C104000 call    dword ptr [<&MSVBVM60.__vbaLenBs>;  MSVBVM60.__vbaLenBstr  

00402E11   .  8BC8          mov     ecx, eax  

00402E13   .  FF15 8C104000 call    dword ptr [<&MSVBVM60.__vbaI2I4>>;  MSVBVM60.__vbaI2I4  

00402E19   .  8D4D A8       lea     ecx, dword ptr [ebp-58]  

00402E1C   .  8985 E4FEFFFF mov     dword ptr [ebp-11C], eax  

00402E22   .  66:C746 46 01>mov     word ptr [esi+46], 1  

00402E28   .  FF15 20114000 call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr  

00402E2E   .  8D4D A0       lea     ecx, dword ptr [ebp-60]  

00402E31   .  FFD3          call    ebx  

00402E33   >  66:8B95 E4FEF>mov     dx, word ptr [ebp-11C]  

00402E3A   .  8B06          mov     eax, dword ptr [esi]  

00402E3C   .  66:3956 46    cmp     word ptr [esi+46], dx             ;  //判断是否转换完成  

00402E40   .  56            push    esi  

00402E41   .  0F8F F5000000 jg      00402F3C                            ;  //跳走  

00402E47   .  FF90 00030000 call    dword ptr [eax+300]  

00402E4D   .  8D4D A0       lea     ecx, dword ptr [ebp-60]  

00402E50   .  50            push    eax  

00402E51   .  51            push    ecx  

00402E52   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaObjSe>;  MSVBVM60.__vbaObjSet  

00402E58   .  8BF8          mov     edi, eax  

00402E5A   .  8D45 A8       lea     eax, dword ptr [ebp-58]  

00402E5D   .  50            push    eax  

00402E5E   .  57            push    edi  

00402E5F   .  8B17          mov     edx, dword ptr [edi]  

00402E61   .  FF92 A0000000 call    dword ptr [edx+A0]  

00402E67   .  85C0          test    eax, eax  

00402E69   .  DBE2          fclex  

00402E6B   .  7D 12         jge     short 00402E7F  

00402E6D   .  68 A0000000   push    0A0  

00402E72   .  68 10264000   push    00402610  

00402E77   .  57            push    edi  

00402E78   .  50            push    eax  

00402E79   .  FF15 44104000 call    dword ptr [<&MSVBVM60.__vbaHresu>;  MSVBVM60.__vbaHresultCheckObj  

00402E7F   >  8B45 A8       mov     eax, dword ptr [ebp-58]           ;  //EAX 指向 UNICODE 的注册码  

00402E82   .  8D8D 7CFFFFFF lea     ecx, dword ptr [ebp-84]  

00402E88   .  0FBF56 46     movsx   edx, word ptr [esi+46]  

00402E8C   .  8945 94       mov     dword ptr [ebp-6C], eax  

00402E8F   .  51            push    ecx  

00402E90   .  8D45 8C       lea     eax, dword ptr [ebp-74]  

00402E93   .  52            push    edx                                ;  //第几位  

00402E94   .  8D8D 6CFFFFFF lea     ecx, dword ptr [ebp-94]  

00402E9A   .  50            push    eax  

00402E9B   .  51            push    ecx  

00402E9C   .  C745 84 01000>mov     dword ptr [ebp-7C], 1  

00402EA3   .  C785 7CFFFFFF>mov     dword ptr [ebp-84], 2  

00402EAD   .  C745 A8 00000>mov     dword ptr [ebp-58], 0  

00402EB4   .  C745 8C 08000>mov     dword ptr [ebp-74], 8  

00402EBB   .  FF15 6C104000 call    dword ptr [<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar  

00402EC1   .  0FBF7E 46     movsx   edi, word ptr [esi+46]           ;  //取其中一位字符  

00402EC5   .  83FF 15       cmp     edi, 15  

00402EC8   .  72 06         jb      short 00402ED0  

00402ECA   .  FF15 7C104000 call    dword ptr [<&MSVBVM60.__vbaGener>;  MSVBVM60.__vbaGenerateBoundsError  

00402ED0   >  8D95 6CFFFFFF lea     edx, dword ptr [ebp-94]  

00402ED6   .  8D45 A4       lea     eax, dword ptr [ebp-5C]  

00402ED9   .  52            push    edx  

00402EDA   .  50            push    eax  

00402EDB   .  FF15 C4104000 call    dword ptr [<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarVal  

00402EE1   .  50            push    eax                                ;  //转换成数字  

00402EE2   .  FF15 34104000 call    dword ptr [<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr  

00402EE8   .  8BC8          mov     ecx, eax                           ;  //转成 ANSI 的字符  

00402EEA   .  FF15 9C104000 call    dword ptr [<&MSVBVM60.__vbaUI1I2>;  MSVBVM60.__vbaUI1I2  

00402EF0   .  8B4D D0       mov     ecx, dword ptr [ebp-30]  

00402EF3   .  880439        mov     byte ptr [ecx+edi], al             ;  //转换后的字符存入  

00402EF6   .  8D4D A4       lea     ecx, dword ptr [ebp-5C]  

00402EF9   .  FF15 20114000 call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr  

00402EFF   .  8D4D A0       lea     ecx, dword ptr [ebp-60]  

00402F02   .  FFD3          call    ebx  

00402F04   .  8D95 6CFFFFFF lea     edx, dword ptr [ebp-94]  

00402F0A   .  8D85 7CFFFFFF lea     eax, dword ptr [ebp-84]  

00402F10   .  52            push    edx  

00402F11   .  8D4D 8C       lea     ecx, dword ptr [ebp-74]  

00402F14   .  50            push    eax  

00402F15   .  51            push    ecx  

00402F16   .  6A 03         push    3  

00402F18   .  FF15 20104000 call    dword ptr [<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  

00402F1E   .  66:8B56 46    mov     dx, word ptr [esi+46]  

00402F22   .  B8 01000000   mov     eax, 1  

00402F27   .  83C4 10       add     esp, 10  

00402F2A   .  66:03D0       add     dx, ax  

00402F2D   .  0F80 12080000 jo      00403745  

00402F33   .  66:8956 46    mov     word ptr [esi+46], dx  

00402F37   .^ E9 F7FEFFFF    jmp     00402E33  
```

原来是写进控件里了，继续往下跟

```
00403012   > /66:8B46 46    mov     ax, word ptr [esi+46]  

00403016   . |B9 14000000   mov     ecx, 14                        ;  //ANSI 格式注册码 & 空格，达到 20 位了吗  

0040301B   . |66:3BC1       cmp     ax, cx  

0040301E   . |7F 3A         jg      short 0040305A                ;  //跳走进入写入文件过程  

00403020   . |0FBFF8        movsx   edi, ax  

00403023   . |83FF 15       cmp     edi, 15  

00403026   . |72 06         jb      short 0040302E  

00403028   . |FF15 7C104000 call    dword ptr [<&MSVBVM60.__vbaGener>;  MSVBVM60.__vbaGenerateBoundsError  

0040302E   > |68 C0264000   push    004026C0  

00403033   . |FF15 34104000 call    dword ptr [<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr  

00403039   . |8BC8          mov     ecx, eax                       ;  //eax=0x20，即空格  

0040303B   . |FF15 9C104000 call    dword ptr [<&MSVBVM60.__vbaUI1I2>;  MSVBVM60.__vbaUI1I2  

00403041   . |8B55 D0       mov     edx, dword ptr [ebp-30]        ;  //合并字符  
```
