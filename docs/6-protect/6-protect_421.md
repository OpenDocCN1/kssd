

# hying 的单步陷阱

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-834.htm`](https://www.pediy.com/kssd/pediy08/pediy8-834.htm)

Opcode 很乱,具体我也不清楚,但可以用  
  
.586P  
.Model Flat,StdCall  
Option CaseMap:None  
Assume Fs:Nothing  
.Code  
  
Start:  
    call  \_push\_handler  
    mov  eax, \[esp+4\]  
    mov  eax, \[eax\]  
    cmp  eax, 80000004h  
    jnz  short \_live\_or\_dead+1 ; to see my god...  
  
    mov  esp, \[esp+8\]  
    jmp  short \_restore\_stack  
  
\_junk\_1:  
    pop  eax  
  
\_live\_or\_dead:  
    jmp  short \_trap\_end  
  
    db 0E9h  
  
\_restore\_stack:  
    pop  fs:0  
    jz    short \_junk\_1  
    jnz  short \_junk\_1  
  
\_trap\_end:  
    jmp  short \_out  
  
  
\_push\_handler:  
    push  fs:0  
    jmp  short \_set\_seh\_1  
  
    db 0FFh  
  
\_make\_exception:  
    pushf  
    jz    short \_junk\_2  
    jnz  short \_junk\_2  
  
\_bad\_address:  
    db 0E9h  
  
\_junk\_2:  
    or  dword ptr \[esp\], 100h  
    popf  
    nop  
    jmp  short \_bad\_address  
  
\_set\_seh\_1:  
    mov  fs:0, esp  
    jmp  short \_make\_exception  
\_out:  
    ret  
  
End  Start  
  
宏,随便放  
  
\_Trap  MACRO  Junkcode  
LOCAL  \_push\_handler, \_live\_or\_dead, \_restore\_stack, \_junk\_1,\_junk\_2,\_trap\_end, \_make\_exception, \_bad\_address,\_set\_seh\_1,\_out  
    call  \_push\_handler  
    mov  eax, \[esp+4\]  
    mov  eax, \[eax\]  
    cmp  eax, 80000004h  
    jnz  short \_live\_or\_dead+1 ; skip  
  
    mov  esp, \[esp+8\]  
    jmp  short \_restore\_stack  
  
\_junk\_1:  
    pop  eax  
  
\_live\_or\_dead:  
    jmp  short \_trap\_end  
  
    db 0E9h  ; jmp ->  
  
\_restore\_stack:  
    pop  fs:0  
    jz    short \_junk\_1  
    jnz  short \_junk\_1  
  
\_trap\_end:  
    jmp  short \_out  
  
  
\_push\_handler:  
    push  fs:0  
    jmp  short \_set\_seh\_1  
  
    db Junkcode  
  
\_make\_exception:  
    pushfd  
    jz    short \_junk\_2  
    jnz  short \_junk\_2  
  
\_bad\_address:  
    db 0E9h  
  
\_junk\_2:  
    or  dword ptr \[esp\], 100h  
    popfd  
    nop  
    jmp  short \_bad\_address  
  
\_set\_seh\_1:  
    mov  fs:0, esp  
    jmp  short \_make\_exception  
\_out:  
    ret  
ENDM 

 4dd466d0e1e0cc123afbe504da2462ca