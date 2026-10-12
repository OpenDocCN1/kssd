
```asm
        call    _Readmemory

        add     esp, 10h

        cmp     eax, 4

        jnz     short loc_42E75C

        mov     eax, [ebp+src]

        mov     [ebx+0EEh], eax

        jmp     short loc_42E766

; ---------------------------------------------------------------------------

loc_42E75C:                    ; CODE XREF: GetThreadInfo+2DE j
                                ; GetThreadInfo+2E7 j ...
        mov     dword ptr [ebx+0EEh], 0FFFFFFFFh
```

将刚得到的寄存器信息设置为有效，并检查线程信息结构体中 `oldreg` 的属性是否有效，若无效则将当前的寄存器值赋值进去：

```asm
loc_42E766:                    ; CODE XREF: GetThreadInfo+30E j
        xor     edx, edx

        xor     ecx, ecx

        mov     [ebx+0F2h], edx

        mov     [ebx+0F6h], ecx

        mov     eax, [ebp+pThreadInfo]

        mov     [eax+t_Thread.regvalid], 1 ; int  Whether reg is valid

        mov     edx, [ebp+pThreadInfo]

        cmp     dword ptr [edx+t_Thread.oldregvalid], 0 ; check ThreadInfo is read

        jnz     short j_next_Thread

        mov     ecx, [ebp+pThreadInfo]

        mov     eax, [ebp+pThreadInfo]

        lea     esi, [ecx+t_Thread.reg] ; struct  Actual contents of registers

        lea     edi, [eax+t_Thread.oldreg] ; struct  Previous contents of registers

        mov     ecx, 65h

        rep movsd               ; Thread.reg copy to Thread.oldreg

        movsw

        mov     eax, [ebp+pThreadInfo]

        mov     dword ptr [eax+61Ch], 1
```

将线程计数自加 1，将线程信息结构体指针指向下一个结构体的首地址（`ThreadInfo` 的大小为 `66Ch`）：

```asm
j_next_Thread:                ; CODE XREF: GetThreadInfo+76 j
                                ; GetThreadInfo+341 j
        inc     [ebp+i]

        add     [ebp+pThreadInfo], 66Ch ; pThreadInfo -> next ThreadInfo
```

这里为循环体的判断处，检查是否遍历完所有线程，若没有遍历完，则继续遍历：

```asm
loc_42E7C1:                    ; CODE XREF: GetThreadInfo+28 j
        mov     edx, [ebp+i]

        cmp     edx, Num_Thread

        jl      getThreadInfo

        mov     eax, [ebp+ret]
```

最后恢复现场：

```asm
_exit:                        ; CODE XREF: GetThreadInfo+19 j
        pop     edi

        pop     esi

        pop     ebx

        mov     esp, ebp

        pop     ebp

        retn

GetThreadInfo   endp

                        武汉科锐学员: angelqkm
                          2008-5-25
```
