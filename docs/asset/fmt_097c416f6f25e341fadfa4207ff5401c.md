
# 淘来的几个 SMC 代码

> 原文：`www.pediy.com/kssd/pediy09/pediy09-631.htm`
>
> 第一个程序独立运行时与在 OD 中运行时 `GetModuleHandleA` 取得的句柄值不一样，OD 中将大于 `70000000`，好象只对 XP 有效。其它程序自己用 OD 试试。

### 代码

```asm
.data       hInst            dd        ?
            szStr            db        ?
            szBytes          db        ?
CTEXT        MACRO  y:VARARG
             LOCAL  sym
             CONST  segment
             ifidni <y>,<>
                 sym db 0
             else
                 sym db y,0
             endif
             CONST  ends
             exitm <offset sym>
           ENDM

.code
start:
          invoke   VirtualProtect, (start+02Eh), 04h, PAGE_EXECUTE_READWRITE, ADDR szBytes
          mov      eax, (start+02Eh)
          xor      ebx, ebx
          mov      bl, byte ptr cs:[eax]
          xor      bl, 035h
          mov      byte ptr ds:[eax], bl
          invoke   GetModuleHandleA, 0
          mov      hInst, eax
          invoke   GetModuleHandleA, 0
          invoke   wsprintf, ADDR szStr, CTEXT("模块基址为：0x%x"), eax
          invoke   MessageBox, NULL, ADDR szStr, CTEXT("SMC 检测 OD"), MB_OK or MB_APPLMODAL
          invoke   ExitProcess, 0
          int      3
          nop

end start
```

### 代码

```asm
start:
      mov  esi, (ProgramEnd-start)
      invoke VirtualProtect, 401000h, esi, PAGE_EXECUTE_READWRITE, ADDR oldProt ; enable write to code section
      test eax, eax
      jnz  _patch

      invoke MessageBox, NULL, szErr, szErr, MB_OK   ; error, show it and quit
      jmp  _end

_patch:
      lea  edi, _change
      mov  al, 0e8h
          ; call opcode
      stosb
      mov  eax, 00000008h ; Will be 08 00 00 00 when written to memory
      stosd

      invoke VirtualProtect, 401000h, esi, oldProt, ADDR oldProt ; restore the old protection settings

_msgBox:
      push NULL
      push offset szTitle
      push offset szText
      push NULL

_change:
      nop
      ; here will be inserted 'call MessageBox'
      nop
      nop
      nop
      nop

_end:
      invoke ExitProcess, NULL

ProgramEnd:

end start
```

### 代码

```asm
start:
      mov eax, offset myMod2
      invoke VirtualProtect, eax, 4, PAGE_EXECUTE_READWRITE, addr oldprotect
      mov eax, offset myMod2
      mov WORD PTR[eax], 00B8h
      mov WORD PTR[eax+1*2], 0009h
      mov WORD PTR[eax+2*2], 0000h
      mov WORD PTR[eax+3*2], 0000h
      mov WORD PTR[eax+4*2], 0000h
      invoke MessageBox, 0, addr szText, addr szTitle, MB_OK
      invoke ExitProcess, 0

myMod2 proc
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      ret

myMod2 ENDP

end start
```

`4dd466d0e1e0cc123afbe504da2462ca`
