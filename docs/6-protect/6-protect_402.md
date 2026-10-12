
# fuck vmp1.64 anti

> 原文：[`www.pediy.com/kssd/pediy10/67872.html`](https://www.pediy.com/kssd/pediy10/67872.html)

```
004B3DF8    CC             int3  
004B3DF9    9D             popfd  
004B3DFA    68 A1597920    push   207959A1  
```

改成

```
004B3DF8    CC             int3  
004B3DF9    CC             int3  
004B3DFA    68 A1597920    push   207959A1  
```

装个 `HideOD` 把前面 2 个 ANTI 过了就行了，不需要装其他插件以及驱动

我的系统有问题，海风月影的插件用不了，就只能自己看了，谢谢海风月影的指导，膜拜一下

顺便膜拜一下 `shoooo` 和 `hexer` 两位偶像～～

引用: *最初由 **海风月影**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=476454#post476454) 这种 anti 都是小儿科，还是几千个 `int 3+rdtsc` 才够强悍

`FakestRdtsc`,抄袭 `Deroko` 的![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷") 俯卧撑我能做仨

代码:

```
volatile __declspec(naked) void NewInt0D()
{
  // 
  //  - Interrupt 0D Handler -
  //
  //  offset   | contains
  //  ---------+-----------------------------
  //  esp      : Error Code
  //  esp + 4  : EIP Context
  //  esp + 8  : CS  Context
  //  esp + C  : EFLAGS
  //
  
  __asm 
  {
    pushad
    mov  eax, [esp+24h]    ; eip
    cmp  eax, 80000000h    ; skip kernel stuff
    ja  __oldint0d

    push  eax
    call   PrefixedRdtsc    ; get prefix size

    mov  esi, [esp+24h]
    add  esi, eax
    cmp  word ptr [esi], 310Fh  ; rdtsc
    jne  __oldint0d
       
    add  eax, 2      ; prefix+opcode
    add  [esp+24h], eax    ; adjust eip   

    lea  esi, TimeStamp    ; fake rdtsc

    // Fuck the nasty rdtsc ;-) 

    mov  ecx, [esi]    ; lower 32b
    mov  ebx, [esi+4]    ; higher 32b
    rdtsc
    sub  eax, ecx
    sbb  edx, ebx    ; edx:eax = real delta
       
    mov  ecx, 05h    ; delta>>5, modify this if needed
    call   __allshr
       
    mov  ecx, [esi]
    mov  ebx, [esi+4]
    add  ecx, eax
    mov  [esp+1Ch], ecx    ; update eax
    adc  ebx, edx
    mov  [esp+14h], ebx    ; update edx
     

    test   dword ptr [esp+2Ch], 100h ; eflags
    jnz     __kitrap01
       
    popad
    add  esp, 4      ; discard error code
    iretd

__kitrap01: 
    push  esi
    sidt  fword ptr[esp-2]
    pop  esi 
    lea  esi, [esi+8]
    mov  ax, word ptr[esi+6]
    shl  eax, 10h
    mov  ax, word ptr[esi]
    mov  [esp+20h], eax    ; replace ErrorCode with int1 handler
    popad
    retn         ; go to KiTrap01   
       
__oldint0d:
    popad
    jmp   OldHandler0D
  }
}

```
