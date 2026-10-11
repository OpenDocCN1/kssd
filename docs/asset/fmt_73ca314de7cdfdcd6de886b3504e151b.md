
```
    lea     ebx,[ebp+035D4CFA]

    call    ebx

    ; 第 2 段

    pop     ebx

    pop     eax

l_00B2E242:

    cmp     dword ptr [ebp+034F0C4D],0

    jz      l_00B2E58A

    push    eax

    push    ebx

    mov     eax,0000047B

    mov     [ebp+034F1B41],eax

    lea     ebx,[ebp+035D49E9]

    call    ebx

    ; 第 3 段

    pop     ebx

    pop     eax

l_00B2E58A:

    mov     eax,eax

    lea     eax,[ebp+03613F38]

    mov     dword ptr [eax],00000094

    mov     byte ptr [ebp+034F1A5D],47

    push    93C8F7C1

    push    dword ptr [ebp+034F0B35]

    lea     eax,[ebp+035E5263]

    call    eax

    ; 第 4 段

    lea     edi,[ebp+03613F38]

    push    edi

    call    eax

    ; 第 5 段

    lea     eax,[ebp+03613F38]

    cmp     dword ptr [eax+10],1

    jnz     l_00B2EA5A

    cmp     dword ptr [eax+8],0

    jnz     l_00B2EA5A

    jmp     l_00B2EE3C

l_00B2EA5A:

    mov     byte ptr [ebp+034F1A5D],49

    push    AFAAAF5A

    push    dword ptr [ebp+034F0B35]

    lea     eax,[ebp+035E5263]

    call    eax

    ; 第 6 段 

    mov     [ebp+034F0EBD],eax

    call    eax

    ; 第 7 段

    or      eax,eax

    jnz     l_00B2EDB0

    cmp     dword ptr [ebp+034F1D8D],0

    jz      l_00B2EE3C

l_00B2EDB0:

    lea     edi,[ebp+034F4CF3]

    mov     eax,1

    jmp     edi

l_00B2EE3C:

    xor     [ebp+034F25F9],eax

    add     eax,[ebp+034F1695]

    add     [ebp+034F0FED],eax

    xor     eax,ebx

    add     [ebp+034F1695],eax

    mov     eax,eax

    cmp     dword ptr [ebp+034F2C5D],0

    jnz     l_00B2F122

    cmp     dword ptr [ebp+034F0C4D],0

    jz      l_00B2F416

l_00B2F122:

    push    eax

    push    ebx

    mov     eax,eax

    mov     eax,0000047B

    mov     [ebp+034F26A1],eax

    lea     ebx,[ebp+035D4D61]

    call    ebx

    ; 第 8 段

    pop     ebx

    pop     eax

l_00B2F416:

    cmp     dword ptr [ebp+034F0C4D],0

    jz      l_00B2F75E

    push    eax

    push    ebx

    mov     eax,0000047B

    mov     [ebp+034F26A1],eax

    lea     ebx,[ebp+035D4A02]

    call    ebx

    ; 第 9 段

    pop     ebx

    pop     eax

l_00B2F75E:

    mov     eax,eax

    mov     eax,[ebp+03613FCC]

    xor     eax,7475EDFF

    add     eax,ebp

    jmp     eax
```

其中第 7 段内红色的代码检查的标记，检测到 `debugger` 时为 `1`。这个标记的设置其实并不在 `vm` 内。

![](img/c520079d48764b8de7b1261d21eca8a4.jpg)

可以编程将解码数据直接汇编为机器码，我没有试过 J。

致谢

- `www.pediy.com`
- `www.unpack.cn`
- `4dd466d0e1e0cc123afbe504da2462ca`
