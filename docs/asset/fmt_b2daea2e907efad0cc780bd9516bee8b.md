
```
jz      short loc_EB98CD22  

        cmp     [edi], ebx  

        jz      short loc_EB98CD22  

        jmp     short loc_EB98CD50              ; while compare protect Process Object  ;;  attention  
```

/////////////////////////////////////////////////////////////////////////////////////

`//EBABB000+3E8h data, length 20h`

`//47616420h address end marking`

```
00000000h: 20 64 61 47 00 00 00 00 20 64 61 47 00 00 00 00 ;  daG.... daG....  

00000010h: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ;  ................  
```

### `Themida_ZwQueryVirtualMemory:`

```
        push    ebp  

        mov     ebp, esp  

        pusha  

        call    $+5  

        pop     edx  

        sub     edx, 56C4980h  

        cmp     dword ptr [esp+28h], 0FFFFFFFFh  

        jz      short loc_EB938B53              ; if handle==NULL goto true function address  

        push    edx             ; save Absolute Address  

        push    0               ; NULL  

        lea     eax, [edx+56C4A08h]             ; edx+56C4A08h save Object  

        push    eax  

        push    0               ; KernelMode  

        xor     eax, eax  

        push    eax             ; NULL  

        push    10h             ; ACCESS  

        push    dword ptr [ebp+8]               ; process handle  

        mov     eax, 8044D57Ah  

        call    eax  

        ; call function ObReferenceObjectByHandle  get process's handle to object  

        pop     edx  

        cmp     dword ptr [edx+56C4A08h], 0  

        jz      short loc_EB938B53    ; if process object==0 to goto true function address  

        mov     eax, [edx+56C4A08h]  

        mov     ebx, eax  

        and     ebx, 7FFFFFFFh  

        mov     esi, 0EBAC8000h  

loc_EB938B19:
```

///////////////////////////////////////////////////////////////////////////////////// attention

```
        add     esi, 4  

        cmp     dword ptr [esi], 47616420h  ; constant 47616420h,address end marking   

        jz      short loc_EB938B53              ; jump of call system true function  

        cmp     [esi], eax  

        jz      short loc_EB938B2E  

        cmp     [esi], ebx  

        jz      short loc_EB938B2E  

        jmp     short loc_EB938B19    ; while compare protect Process Object  ;;  attention  
```

///////////////////////////////////////////////////////////////////////////////////// attention

`loc_EB938B2E:             ;is protect process`

```
        push    fs  

        mov     eax, 30h  

        mov     fs, ax  

        mov     eax, large fs:124h    ; ETHREAD  

        mov     eax, [eax+44h]        ; KPROCESS       

        pop     fs  

        cmp     eax, [edx+56C4A08h]  

        jz      short loc_EB938B53  
```

///////////////////////////////////////////////////////////////////////////////////// attention

```
        mov     dword ptr [esp+28h], 0           ;fuck Themida, handle=0;  

loc_EB938B53:
```

`///////////system function`

```
        popa  

        pop     ebp  

        push    804D1DFAh  ;ZwQueryVirtualMemory  

        retn  
```

`//EBAC8000h data, length 20h`

`//47616420h address end marking`

`//813E5020h Themida protect process object`

```
00000000h: 20 64 61 47 20 50 3E 81 20 64 61 47 00 00 00 00 ;  daG P>?daG....  

00000010h: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ;  ................  
```

### `Themida_ZwWriteVirtualMemory:`

```
        push    ebp  

        mov     ebp, esp  

        pusha  

        call    $+5  

        pop     edx  

        sub     edx, 567DB24h  

        xor     edi, edi                  ; attention ,edi=0 call write,edi=1 call read  

        jmp     short loc_EB938029  
```

### `Themida_NtAllocateVirtualMemory:`

```
        push    ebp  

        mov     ebp, esp  

        pusha  

        call    $+5  

        pop     edx  

        sub     edx, 567DB38h  

        mov     edi, 1  

loc_EB938029:
```

```
        push    edx  

        push    0  

        lea     eax, [edx+567DBB3h]  

        push    eax  

        push    1  

        mov     eax, 80481EA4h  

        xor     eax, eax  

        push    eax  

        push    10h  

        push    dword ptr [ebp+8]  

        mov     eax, 8044D57Ah  

        call    eax  

        pop     edx  

        mov     eax, [edx+567DBB3h]  

        lea     esi, [edx+567DBB7h]  

        mov     ecx, 0EB938956h              ;see my comment  address  EB99C956h  

        jmp     ecx  
```

### `Themida_ZwReadVirtualMemory:`

```
        push    ebp  

        mov     ebp, esp  

        pusha  

        call    $+5  

        pop     edx  

        sub     edx, 58BC582h  

        mov     edi, 1                      ; attention ,edi=1 call read,edi=0 call write  

        push    edx             ; save Absolute Address  

        push    0               ; NULL  

        lea     eax, [edx+58BC5FDh]             ; edx+58BC5FDh save Object  

        push    eax  

        push    1         ; UserMode  

        mov     eax, 80481EA4h        ; PsProcessType,no use seem  

        xor     eax, eax                  ; eax=0;  

        push    eax                     ; NULL  

        push    10h                     ; ACCESS  

        push    dword ptr [ebp+8]               ; process handle  

        mov     eax, 8044D57Ah  

        call    eax          

   ; call function ObReferenceObjectByHandle  get read process of memory's handle to object  
```

```
////////////////////////////////////////////////////////////////////////////////////////////                     pop     edx  

        mov     eax, [edx+58BC5FDh]             ; process object  

        lea     esi, [edx+58BC601h]             ; edx+58BC5FDh+4,Themida data address  

        mov     ecx, 0EB99C956h                

        jmp     ecx                 ; to Themida code  

////////////////////////////////////////////////////////////////////////////////////////////
```

```
    ............  

    ............  

    ............  
```

`//constant 4E67EEF4h==address start marking`

`//constant 4E67EEF5h==address end marking`

`//edx+58BC5FDh data, length 20h`

```
00000000h: 20 00 2E 81 F4 EE 67 4E BC 3A 10 20 00 00 40 00 ;  ..侓頶 N?. ..@.  

00000010h: 00 00 E5 02 F5 EE 67 4E 00 00 00 00 00 00 00 00 ; ..?躅 gN........  
```

`//81394820h ;process object`

`//edx+58BC601h-4 data, length 4`

```
00000000h: 20 48 39 81                                 ;  H9  
```

`//attention`

`//80414520h Themida protect process object`

`//00400000h Themida protect process base address`

`//02E40000h Themida protect process memory size`

`//EB99C410 data, length 20h`

```
00000000h: F4 EE 67 4E 20 45 41 81 00 00 40 00 00 00 E4 02 ; 纛 gN EA?.@...?  

00000010h: F5 EE 67 4E 00 00 00 00 00 00 00 00 00 00 00 00 ; 躅 gN............  
```

`EB99C956:`

`loc_EB99C956:`

```
        cmp     dword ptr [esi], 4E67EEF5h      ; constant 4E67EEF5h,address end marking  

        jz      short loc_EB99C967  

        cmp     [esi], eax  
```
