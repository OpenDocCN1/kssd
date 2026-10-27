
## `k_nxt2` ~ `k_nxt3`

```
k_nxt2:        mov al, 0f0h
        jmp k_nxt1
k_nxt3:        mov al, 0c8h
        jmp k_nxt1
```

## `k_next1`

```
k_next1:mov al, 0b8h                                ;MOV reg, value
        jmp k_nxt1
```

## `k_next2`

```
k_next2:mov al, 68h                                        ;PUSH value
        stosb                                        ;POP reg
        xchg eax, edx
        stosd
        mov al, ch
        add al, 58h
        jmp i_end1
greg3        endp
```

## `greg4` proc

```
greg4        proc
        mov edx, 0                         ;get key increment value
key_inc = dword ptr $ - 4
i_next:        push 3
        call random
        test eax, eax
        je i_next0
        cmp al, 1
        je i_next1
        cmp al, 2
        je i_next2
        mov al, 90h                                ;XCHG EAX, reg
        add al, ch                                ;XOR reg, reg
        stosb                                                        ;OR reg, EAX
        push ebx                                        ;ADD reg, value
        mov bl, ch
        call make_xor
        pop ebx
        mov al, 0bh
        stosb
        mov al, 18h
        add al, ch
        rol al, 3
        stosb
i_next0:mov al, 81h                                ;ADD reg, value
        stosb
        mov al, 0c0h
        add al, ch
        stosb
        xchg eax, edx
        jmp n_end1
i_next1:mov al, 0b8h                                ;MOV EAX, value
        stosb                                                                                        ;ADD reg, EAX
        xchg eax, edx
        stosd
        mov al, 3
        stosb
        mov al, 18h
        or al, ch
        rol al, 3
i_end1:        stosb
i_end2:        call rjunk
        ret
i_next2:mov al, 8bh                                ;MOV EAX, reg
        stosb                                                                                ;ADD EAX, value
        mov al, 0c0h                                                ;XCHG EAX, reg
        add al, ch
        stosb
        mov al, 5
        stosb
        xchg eax, edx
        stosd
        mov al, 90h
        add al, ch
        jmp i_end1
greg4        endp
```

## `greg5` proc

```
greg5        proc
        push ecx
        mov ch, bh
        push 4
        pop edx
        push 2
        call random
        test eax, eax
        jne ng5
        call i_next                                ;same as previous, value=4
        pop ecx
        jmp k_end
ng5:        mov al, 40h                                ;4x inc reg
        add al, ch
        pop ecx
        stosb
        stosb
        stosb
        jmp i_end1
greg5        endp
```

## `greg6` proc

```
greg6        proc
        push 5
        call random
        test eax, eax
        je d_next0
        cmp al, 1
        je d_next1
        cmp al, 2
        je d_next2
        mov al, 83h                                ;SUB reg, 1
        stosb
        mov al, 0e8h
        add al, cl
        stosb
        mov al, 1
        jmp i_end1
d_next0:mov al, 48h                                ;DEC reg
        add al, cl
        jmp i_end1
d_next1:mov al, 0b8h                                ;MOV EAX, random_value
        stosb                                                                                        ;SUB reg, EAX
        push 0                                                                                ;ADD reg, random_value-1
        call random
        mov edx, eax
        stosd
        mov al, 2bh
        stosb
        mov al, 18h
        add al, cl
        rol al, 3
        stosb
        mov al, 81h
        stosb
        mov al, 0c0h
        add al, cl
        stosb
        dec edx
        mov eax, edx
        jmp n_end1
d_next2:mov al, 90h                                ;XCHG EAX, reg
        add al, cl                                                        ;DEC EAX
        stosb                                                                                ;XCHG EAX, reg
        mov al, 48h
        stosb
        mov al, 90h
        add al, cl
        jmp i_end1
greg6        endp
```

## `greg7` proc

```
greg7        proc
        mov edx, [esp+4]
        dec edx
        push 2
        call random
        test eax, eax
        je l_next0
        mov al, 51h                                ;PUSH ECX
        stosb                                                        ;MOV ECX, reg
        mov al, 8bh                                ;JECXZ label
        stosb                                                        ;POP ECX
        mov al, 0c8h                        ;JMP decrypt_loop
        add al, cl                                ;label:
        stosb                                                        ;POP ECX
        mov eax, 0eb5903e3h
        stosd
        sub edx, edi
        mov al, dl
        stosb
        mov al, 59h
        jmp l_next
l_next0:push ebx                ;XOR EAX, EAX
        xor bl, bl                                ;DEC EAX
        call make_xor                        ;ADD EAX, reg
        pop ebx                                          ;JNS decrypt_loop
        mov al, 48h
        stosb
        mov al, 3
        stosb
        mov al, 0c0h
        add al, cl
        stosb
        mov al, 79h
        stosb
        sub edx, edi
        mov al, dl
l_next:        stosb
        call rjunk
        ret Pshd
greg7        endp
```

## `rjunkjc` / `rjunk` proc

```
rjunkjc:push 7
        call random
        jmp rjn
rjunk        proc                        ;junk instruction generator
        push 8
        call random                ;0=5, 1=1+2, 2=2+1, 3=1, 4=2, 5=3, 6=none, 7=dummy jump and call
                                                                ;左侧索引(eax,随机数) = 右侧(垃圾指令字节数)
rjn:        test eax, eax
        je j5
        cmp al, 1
        je j_1x2
        cmp al, 2
        je j_2x1
        cmp al, 4
        je j2
        cmp al, 5
        je j3
        cmp al, 6
        je r_end
        cmp al, 7
        je jcj
j1:        call junx1                ;one byte junk instruction
        nop
        dec eax
        SALC
        inc eax
        clc
        cwde
        stc
        cld
junx1:        pop esi
        push 8
        call random
        add esi, eax
        movsb
        ret
j_1x2:        call j1                        ;one byte and two byte
        jmp j2
j_2x1:        call j2                        ;two byte and one byte
        jmp j1
j3:        call junx3
        db        0c1h, 0c0h        ;rol eax, ...
        db        0c1h, 0e0h        ;shl eax, ...
        db        0c1h, 0c8h        ;ror eax, ...
        db        0c1h, 0e8h        ;shr eax, ...
        db        0c1h, 0d0h        ;rcl eax, ...
        db        0c1h, 0f8h        ;sar eax, ...
        db        0c1h, 0d8h        ;rcr eax, ...
        db        083h, 0c0h
        db        083h, 0c8h
        db        083h, 0d0h
        db        083h, 0d8h
        db        083h, 0e0h
        db        083h, 0e8h
        db        083h, 0f0h
        db        083h, 0f8h        ;cmp eax, ...
        db        0f8h, 072h        ;clc; jc ...
        db        0f9h, 073h        ;stc; jnc ...
junx3:        pop esi                        ;three byte junk instruction
        push 17
        call random
        imul eax, 2
        add esi, eax
        movsb
        movsb
r_ran:        push 0
        call random
        test al, al
        je r_ran
        stosb
        ret
j2:        call junx2
        db        8bh                ;mov eax, ...
        db        03h                ;add eax, ...
        db        13h                ;adc eax, ...
        db        2bh                ;sub eax, ...
        db        1bh                ;sbb eax, ...
        db        0bh                ;or eax, ...
        db        33h                ;xor eax, ...
        db        23h                ;and eax, ...
```
