
# hying 的单步陷阱

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-834.htm`](https://www.pediy.com/kssd/pediy08/pediy8-834.htm)

`Opcode` 很乱，具体我也不清楚，但可以用

```
.586P
.Model Flat,StdCall
Option CaseMap:None
Assume Fs:Nothing
.Code
Start:
    call  _push_handler
    mov   eax, [esp+4]
    mov   eax, [eax]
    cmp   eax, 80000004h
    jnz   short _live_or_dead+1 ; to see my god...
    mov   esp, [esp+8]
    jmp   short _restore_stack
_junk_1:
    pop   eax
_live_or_dead:
    jmp   short _trap_end
    db 0E9h
_restore_stack:
    pop   fs:0
    jz    short _junk_1
    jnz   short _junk_1
_trap_end:
    jmp   short _out
_push_handler:
    push  fs:0
    jmp   short _set_seh_1
    db 0FFh
_make_exception:
    pushf
    jz    short _junk_2
    jnz   short _junk_2
_bad_address:
    db 0E9h
_junk_2:
    or    dword ptr [esp], 100h
    popf
    nop
    jmp   short _bad_address
_set_seh_1:
    mov   fs:0, esp
    jmp   short _make_exception
_out:
    ret
End  Start
```

宏，随便放

```
_Trap  MACRO  Junkcode
LOCAL  _push_handler, _live_or_dead, _restore_stack, _junk_1,_junk_2,_trap_end, _make_exception, _bad_address,_set_seh_1,_out
    call  _push_handler
    mov   eax, [esp+4]
    mov   eax, [eax]
    cmp   eax, 80000004h
    jnz   short _live_or_dead+1 ; skip
    mov   esp, [esp+8]
    jmp   short _restore_stack
_junk_1:
    pop   eax
_live_or_dead:
    jmp   short _trap_end
    db 0E9h  ; jmp ->
_restore_stack:
    pop   fs:0
    jz    short _junk_1
    jnz   short _junk_1
_trap_end:
    jmp   short _out
_push_handler:
    push  fs:0
    jmp   short _set_seh_1
    db Junkcode
_make_exception:
    pushfd
    jz    short _junk_2
    jnz   short _junk_2
_bad_address:
    db 0E9h
_junk_2:
    or    dword ptr [esp], 100h
    popfd
    nop
    jmp   short _bad_address
_set_seh_1:
    mov   fs:0, esp
    jmp   short _make_exception
_out:
    ret
ENDM
```

`4dd466d0e1e0cc123afbe504da2462ca`
