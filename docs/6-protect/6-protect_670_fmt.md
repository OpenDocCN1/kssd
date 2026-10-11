
# Private exe Protector 主程序的 VM 分析

> 原文：[`www.pediy.com/kssd/pediy10/59803.html`](https://www.pediy.com/kssd/pediy10/59803.html)

转载注明看雪

我看的这个版本是 2.40 的版本，别的版本的 VM 没有研究过。

它的 VM 是基于 `PCode` 的了，每条 `PCode` 是 `0x20` 个 `BYTE`，格式如下:

```c
#pragma pack(1)

/*
reg_info
0 eax
1 ecx
2 edx
3 ebx
4 esp
5 ebp
6 esi
7 edi
*/

typedef struct _P_Code_
{
  BYTE x86code[0x10];    //这个是用来在 stack 里面运行的原始 x86 代码 最多 0x10 个字节
  BYTE flag;        //0 代表是一般情况 4 代表是 stack 最后加 ret 情况    38h
  BYTE disptach_type;    //PCode 分发号                39h
  BYTE regdest_type;    //目标寄存器标识              3Ah
  DWORD dwDestValue_Addr_Size;  //用来计算 regdest 相关信息的        3Bh
  BYTE regsrc_type;    //源寄存器的标识              3Fh
  DWORD dwSrcValue_Addr_Size;    //用来计算 regsrc 相关信息的        40h
  DWORD PCode_Addr;    //和 PCode 相关                44h
}P_CODE;

#pragma pack()
```

最后表明的 `38h`,`39h`... 是基于运行时候的偏移。

如果你在跟踪 `Private` 主程序时候看到正常的语句，后面猛然接个大跳，很可能就开始进入 VM 了，比如:

```asm
RYOMA:2C8B3593 64 FF 30                  push    dword ptr fs:[eax]
RYOMA:2C8B3596 64 89 20                  mov     fs:[eax], esp
RYOMA:2C8B3599 E9 F0 9F 08 00            jmp     near ptr 2C93D58Eh ; 进入虚拟机
```

进入 VM 前会做一些初始化工作：

首先会 `push VM_Data`，然后再 `push ret_addr`，例如这里

```asm
.rdata:2C93D754 9C                        pushf
.rdata:2C93D755 C7 04 24 12 D7 93+        mov     dword ptr [esp], offset VM_Data
.rdata:2C93D75C 7D 00                     jge     short $+2
.rdata:2C93D75E E9 03 00 00 00            jmp     loc_2C93D766
.rdata:2C93D75E                 ; ---------------------------
.rdata:2C93D763 67                        db  67h ; g
.rdata:2C93D764 3E                        db  3Eh ; >
.rdata:2C93D765 09                        db    9
.rdata:2C93D766                 ; ---------------------------
.rdata:2C93D766
.rdata:2C93D766                 loc_2C93D766:               ; CODE XREF: .rdata:2C93D75Ej
.rdata:2C93D766 68 8E D7 93 2C            push    offset loc_2C93D78E
```

`loc_2C93D78E` 就是执行完 vm 后的返回地址，这里是个 `call`

然后跳到一个固定的 `VM_Fucking_In` 上面，如下

```asm
.rdata:2C93534E 9C                        pushf
.rdata:2C93534F 89 2C 24                  mov     [esp], ebp    ; push ebp
.rdata:2C935352 89 F6                     mov     esi, esi
.rdata:2C935354 E9 03 00 00 00            jmp     loc_2C93535C
.rdata:2C935354                 ; ---------------------------
.rdata:2C935359 41                        db  41h ; A
.rdata:2C93535A BF                        db 0BFh ; ?
.rdata:2C93535B 44                        db  44h ; D
.rdata:2C93535C                 ; ---------------------------
.rdata:2C93535C
.rdata:2C93535C                 loc_2C93535C:               ; CODE XREF: .rdata:2C935354j
.rdata:2C93535C 7D 00                     jge     short $+2
.rdata:2C93535E 8B EC                     mov     ebp, esp
.rdata:2C935360 75 00                     jnz     short $+2
.rdata:2C935362 E9 00 00 00 00            jmp     $+5
.rdata:2C935367 75 00                     jnz     short $+2
.rdata:2C935369 9C                        pushf
.rdata:2C93536A EB 00                     jmp     short $+2
.rdata:2C93536C E9 03 00 00 00            jmp     loc_2C935374
.rdata:2C93536C                 ; ---------------------------
.rdata:2C935371 74                        db  74h ; t
.rdata:2C935372 85                        db  85h ; ?
.rdata:2C935373 DD                        db 0DDh ; ?
.rdata:2C935374                 ; ---------------------------
.rdata:2C935374
.rdata:2C935374                 loc_2C935374:               ; CODE XREF: .rdata:2C93536Cj
.rdata:2C935374 7D 00                     jge     short $+2
.rdata:2C935376 9C                        pushf
.rdata:2C935377 89 2C 24                  mov     [esp], ebp    ; push ebp
.rdata:2C93537A 7D 00                     jge     short $+2
.rdata:2C93537C E9 04 00 00 00            jmp     loc_2C935385
........
```

上面的代码去掉花以后就是:

```asm
push ebp
mov ebp,esp
pushfd        //保存一些寄存器
push ebp
push edi
push esi
push ecx
push edx
push ebx
push eax
push esp      //代表下面保存 reg 的开始位置，可以理解成一个结构
push [ebp+8]      //VM_Data
call VM_
```

真正的地方是在 `VM_` 里面，可以看出，这里的 `esp`,`ebp` 都不是最早进入 VM 时候的 `esp`,`ebp` 了。

下面我们再来看 `VM_`，选几个重点地方分析一下。

```asm
RYOMA:2C8ADDF0                 VM_       proc near
RYOMA:2C8ADDF0
RYOMA:2C8ADDF0                 var_10038_VM_Str= byte ptr -10038h
RYOMA:2C8ADDF0                 var_1074        = dword ptr -1074h
RYOMA:2C8ADDF0                 var_38_regtype  = dword ptr -38h
RYOMA:2C8ADDF0                 var_34_reg_type = dword ptr -34h
RYOMA:2C8ADDF0                 var_30_pVM_Struct= dword ptr -30h
RYOMA:2C8ADDF0                 var_2C_p        = dword ptr -2Ch
RYOMA:2C8ADDF0                 var_28_pVM_Struct= dword ptr -28h
RYOMA:2C8ADDF0                 var_24_VM_Regs  = dword ptr -24h
RYOMA:2C8ADDF0                 var_1D          = byte ptr -1Dh
RYOMA:2C8ADDF0                 var_1C          = dword ptr -1Ch
RYOMA:2C8ADDF0                 var_18          = dword ptr -18h
RYOMA:2C8ADDF0                 var_12          = byte ptr -12h
RYOMA:2C8ADDF0                 var_11          = byte ptr -11h
RYOMA:2C8ADDF0                 var_10_regs     = dword ptr -10h
RYOMA:2C8ADDF0                 var_C_PVM_regEAX= dword ptr -0Ch
RYOMA:2C8ADDF0                 var_8_count     = dword ptr -8
RYOMA:2C8ADDF0                 var_4_i_regs    = dword ptr -4
RYOMA:2C8ADDF0                 arg_0_VM_Data   = dword ptr  8
RYOMA:2C8ADDF0                 arg_4_regs      = dword ptr  0Ch
RYOMA:2C8ADDF0
RYOMA:2C8ADDF0 55                     push    ebp
RYOMA:2C8ADDF1 8B EC                   mov     ebp, esp
RYOMA:2C8ADDF3 50                     push    eax
RYOMA:2C8ADDF4 B8 10 00 00 00          mov     eax, 10h
RYOMA:2C8ADDF9
RYOMA:2C8ADDF9                 loop_:                 ; CODE XREF: VM_+11j
RYOMA:2C8ADDF9 81 C4 04 F0 FF FF        add     esp, -0FFCh
RYOMA:2C8ADDFF 50                     push    eax
RYOMA:2C8ADE00 48                     dec     eax
RYOMA:2C8ADE01 75 F6                   jnz     short loop_
            //这里 vm 分配自己的堆栈空间。
RYOMA:2C8ADE03 8B 45 FC                 mov     eax, [ebp+var_4_i_regs] ; 没有意义这里
RYOMA:2C8ADE06 83 C4 CC                 add     esp, -34h
RYOMA:2C8ADE09 53                     push    ebx
RYOMA:2C8ADE0A 56                     push    esi
RYOMA:2C8ADE0B 57                     push    edi
RYOMA:2C8ADE0C 8B 75 0C                 mov     esi, [ebp+arg_4_regs]
RYOMA:2C8ADE0F E9 E6 05 00 00            jmp     set_vm_struct
RYOMA:2C8ADE14                 ; ---------------------------
RYOMA:2C8ADE14
RYOMA:2C8ADE14                 loop_fucking_vm:        ; CODE XREF: VM_+5Aj
RYOMA:2C8ADE14                                          ; VM_+130j ...
```



```
RYOMA:2C8ADE14 0F B6 43 38                     movzx   eax, byte ptr [ebx+38h]

RYOMA:2C8ADE18 2C 01                           sub     al, 1               ; 0 4

RYOMA:2C8ADE1A 72 30                           jb      short less_0

RYOMA:2C8ADE1C 2C 03                           sub     al, 3

RYOMA:2C8ADE1E 0F 85 6C 05 00 00               jnz     get_next_pcode

RYOMA:2C8ADE24 C6 43 38 00                      mov     byte ptr [ebx+38h], 0

RYOMA:2C8ADE28 C6 43 39 0D                      mov     byte ptr [ebx+39h], 0Dh

RYOMA:2C8ADE2C 8B 43 3B                         mov     eax, [ebx+3Bh]

RYOMA:2C8ADE2F 40                               inc     eax

RYOMA:2C8ADE30 89 43 40                         mov     [ebx+40h], eax      ; size

RYOMA:2C8ADE33 8B 43 58                         mov     eax, [ebx+VM_Struct.VM_regESP]

RYOMA:2C8ADE36 2B 43 40                         sub     eax, [ebx+40h]

RYOMA:2C8ADE39 89 43 3B                         mov     [ebx+3Bh], eax

RYOMA:2C8ADE3C 8B 43 40                         mov     eax, [ebx+40h]

RYOMA:2C8ADE3F C6 44 03 27 C3                   mov     byte ptr [ebx+eax+27h], 0C3h ; ret

RYOMA:2C8ADE44 8D 43 28                         lea     eax, [ebx+VM_Struct.Curr_PCode]

RYOMA:2C8ADE47 89 43 44                         mov     [ebx+44h], eax      ; Curr_PCode

RYOMA:2C8ADE4A EB C8                            jmp     short loop_fucking_vm

RYOMA:2C8ADE4C                                   ; ---------------------------------------------------------------------------

RYOMA:2C8ADE4C

RYOMA:2C8ADE4C                                   less_0:                    ; CODE XREF: VM_+2Aj

RYOMA:2C8ADE4C 0F B6 43 39                      movzx   eax, byte ptr [ebx+39h]

RYOMA:2C8ADE50 34 00                             xor     al, 0               ; //没意义的

RYOMA:2C8ADE52 0F BE C0                          movsx   eax, al

RYOMA:2C8ADE55 89 45 FC                          mov     [ebp+var_4_i_regs], eax

RYOMA:2C8ADE58 8B 45 FC                          mov     eax, [ebp+var_4_i_regs]

RYOMA:2C8ADE5B 83 F8 23                          cmp     eax, 23h            ; switch 36 cases

RYOMA:2C8ADE5E 0F 87 29 05 00 00                 ja      case_default_next_pcode ; default

RYOMA:2C8ADE5E                                     ; jumptable 2C8ADE64 cases 0,1

RYOMA:2C8ADE64 FF 24 85 6B DE 8A+                jmp     dword ptr VM_Dispatch_Table[eax*4] ; switch jump

RYOMA:2C8ADE64 2C                                ; ---------------------------------------------------------------------------

RYOMA:2C8ADE6B                                   VM_Dispatch_Table:                ; DATA XREF: VM_+74r

RYOMA:2C8ADE6B 8D E3 8A 2C 8D E3+                dd offset case_default_next_pcode ; jump table for switch statement

RYOMA:2C8ADE6B 8A 2C AC E3 8A 2C+                dd offset case_default_next_pcode

RYOMA:2C8ADE6B 00 DF 8A 2C 11 DF+                dd offset case_disptach_2_vm_end

RYOMA:2C8ADE6B 8A 2C 1E E0 8A 2C+                dd offset case_disptach_3_VM_Push_

RYOMA:2C8ADE6B B1 E0 8A 2C A6 DF+                dd offset case_disptach_4_VM_GetRegValue

RYOMA:2C8ADE6B 8A 2C 9B E0 8A 2C+                dd offset case_disptach_5_movs

RYOMA:2C8ADE6B 25 DF 8A 2C C7 E0+                dd offset case_disptach_6_VM_Pop_Reg

RYOMA:2C8ADE6B 8A 2C D0 E0 8A 2C+                dd offset case_disptach_7_vm_popad

RYOMA:2C8ADE6B DD E0 8A 2C EA E0+                dd offset case_disptach_8_vm_pushad

RYOMA:2C8ADE6B 8A 2C 4B E2 8A 2C+                dd offset case_disptach_9_movs

RYOMA:2C8ADE6B 5F E2 8A 2C 6F E2+                dd offset case_disptach_10_VM_Add_esp_4

RYOMA:2C8ADE6B 8A 2C 8D E2 8A 2C+                dd offset case_disptach_11_VM_Pushfd

RYOMA:2C8ADE6B 80 E2 8A 2C BC DF+                dd offset case_disptach_12_VM_Popfd

RYOMA:2C8ADE6B 8A 2C E8 DF 8A 2C+                dd offset case_disptach_13_run_in_stack

RYOMA:2C8ADE6B CE DF 8A 2C 10 E0+                dd offset case_disptach_14_set_run_in_stack_size

RYOMA:2C8ADE6B 8A 2C F6 DF 8A 2C+                dd offset case_disptach_15_SetRegValue

RYOMA:2C8ADE6B A1 E2 8A 2C 6B E3+                dd offset case_disptach_16_GetMemValue

RYOMA:2C8ADE6B 8A 2C 7E E3 8A 2C+                dd offset case_disptach_17_GetRegValueSrc

RYOMA:2C8ADE6B C3 E2 8A 2C 09 E3+                dd offset case_disptach_18_mov_mem_value

RYOMA:2C8ADE6B 8A 2C 16 E3 8A 2C+                dd offset case_disptach_19_VM_MovMemWithReg

RYOMA:2C8ADE6B 2A E3 8A 2C 3B E3+                dd offset case_disptach_20_set_offset_0

RYOMA:2C8ADE6B 8A 2C 4A E3 8A 2C+                dd offset case_disptach_21_mov_

RYOMA:2C8ADE6B 57 E3 8A 2C E5 E2+                dd offset case_disptach_22_set_dest_offset_0

RYOMA:2C8ADE6B 8A 2C F5 E2 8A 2C                 dd offset case_disptach_23_mov_mem_offset_reg

RYOMA:2C8ADE6B                                   dd offset case_disptach_24_run_in_stack_get_cupid

RYOMA:2C8ADE6B                                   dd offset case_disptach_25_VM_idiv

RYOMA:2C8ADE6B                                   dd offset case_disptach_26_VM_imul

RYOMA:2C8ADE6B                                   dd offset case_disptach_27_run_in_stack_cc

RYOMA:2C8ADE6B                                   dd offset case_disptach_28_add_value

RYOMA:2C8ADE6B                                   dd offset case_disptach_29_get_reg_value

RYOMA:2C8ADE6B                                   dd offset case_disptach_30_GetMemValue

RYOMA:2C8ADE6B                                   dd offset case_disptach_31_add_mem

RYOMA:2C8ADE6B                                   dd offset case_disptach_32_VM_xor

RYOMA:2C8ADE6B                                   dd offset case_disptach_33_get_reg_value

RYOMA:2C8ADE6B                                   dd offset case_disptach_34_sub_

RYOMA:2C8ADE6B                                   dd offset case_disptach_35_get_reg_value
```

进入 `VM_` 后，分配完 `vm_stack`，马上就跳到 `set_vm_struct` 上面去了

```
RYOMA:2C8AE3FA                                   set_vm_struct:                  ; CODE XREF: VM_+1Fj

RYOMA:2C8AE3FA 8D 9D C8 FF FE FF                lea     ebx, [ebp+var_10038_VM_Str]

RYOMA:2C8AE400 33 C0                             xor     eax, eax

RYOMA:2C8AE402 89 45 FC                          mov     [ebp+var_4_i_regs], eax

RYOMA:2C8AE405 B8 74 A2 8B 2C                    mov     eax, offset reg_type

RYOMA:2C8AE40A

RYOMA:2C8AE40A                                   set_regs:                       ; CODE XREF: VM_+633j

RYOMA:2C8AE40A 8B 55 FC                          mov     edx, [ebp+var_4_i_regs]

RYOMA:2C8AE40D 03 D2                             add     edx, edx

RYOMA:2C8AE40F 03 D2                             add     edx, edx

RYOMA:2C8AE411 03 D6                             add     edx, esi

RYOMA:2C8AE413 8B 12                             mov     edx, [edx]

RYOMA:2C8AE415 0F B6 08                         movzx   ecx, byte ptr [eax]

RYOMA:2C8AE418 89 14 8B                         mov     [ebx+ecx*4], edx

RYOMA:2C8AE41B FF 45 FC                         inc     [ebp+var_4_i_regs]

RYOMA:2C8AE41E 40                               inc     eax

RYOMA:2C8AE41F 83 7D FC 08                      cmp     [ebp+var_4_i_regs], 8

RYOMA:2C8AE423 75 E5                             jnz     short set_regs

RYOMA:2C8AE425 8B 45 08                         mov     eax, [ebp+arg_0_VM_Data]

RYOMA:2C8AE428 89 43 70                         mov     [ebx+70h], eax        //保存 VM_Data

RYOMA:2C8AE42B 56                               push    esi

RYOMA:2C8AE42C 8B F0                             mov     esi, eax
```



# 汇编代码分析：VM 寄存器设置与 PCode 解析

以下代码段展示了虚拟机（VM）初始化过程中对寄存器的设置，包括复制寄存器状态、清零密钥数据以及调整栈指针。

```
RYOMA:2C8AE42E 8D 7B 28                 lea     edi, [ebx+VM_Struct.Curr_PCode]  
RYOMA:2C8AE431 B9 08 00 00 00           mov     ecx, 8  
RYOMA:2C8AE436 F3 A5                   rep movsd  
RYOMA:2C8AE438 5E                      pop     esi  
RYOMA:2C8AE439 8D 43 48                lea     eax, [ebx+VM_Struct.VM_regEAX]  
RYOMA:2C8AE43C 89 45 D4                mov     [ebp+var_2C_p], eax  
RYOMA:2C8AE43F 89 5D D0                mov     [ebp+var_30_pVM_Struct], ebx  
RYOMA:2C8AE442 33 C0                  xor     eax, eax  
RYOMA:2C8AE444                        
RYOMA:2C8AE444                       loop_set_vm_regs:              ; CODE XREF: VM_+66Bj  
RYOMA:2C8AE444 3C 05                   cmp     al, 5  
RYOMA:2C8AE446 74 10                  jz      short loc_2C8AE458  
RYOMA:2C8AE448 0F B6 D0               movzx   edx, al  
RYOMA:2C8AE44B 8B 4D D0               mov     ecx, [ebp+var_30_pVM_Struct]  
RYOMA:2C8AE44E FF 34 91               push    dword ptr [ecx+edx*4]  
RYOMA:2C8AE451 8B 4D D4               mov     ecx, [ebp+var_2C_p]  
RYOMA:2C8AE454 5F                      pop     edi                 ; mov edi,[ecx+edx*4]  
RYOMA:2C8AE455 89 3C 91               mov     [ecx+edx*4], edi  
RYOMA:2C8AE458                        
RYOMA:2C8AE458                       loc_2C8AE458:                 ; CODE XREF: VM_+656j  
RYOMA:2C8AE458 40                      inc     eax  
RYOMA:2C8AE459 3C 0A                   cmp     al, 0Ah  
RYOMA:2C8AE45B 75 E7                  jnz     short loop_set_vm_regs  
RYOMA:2C8AE45D 8B 43 6C               mov     eax, [ebx+VM_Struct.VM_keydata]  
RYOMA:2C8AE460 31 43 6C               xor     [ebx+VM_Struct.VM_keydata], eax ; 清 0  
RYOMA:2C8AE463 8B C3                  mov     eax, ebx  
RYOMA:2C8AE465 05 00 00 01 00         add     eax, 10000h  
RYOMA:2C8AE46A 83 E8 74               sub     eax, 74h  
RYOMA:2C8AE46D 89 43 58               mov     [ebx+VM_Struct.VM_regESP], eax ; 设置 VM_ESP  
RYOMA:2C8AE470 8B 43 14               mov     eax, [ebx+VM_Struct.regEBP] ; VM_EBP  
RYOMA:2C8AE473 83 C0 04               add     eax, 4  
RYOMA:2C8AE476 89 43 5C               mov     [ebx+VM_Struct.VM_regEBP], eax ; 指向返回地址  
RYOMA:2C8AE479 E9 96 F9 FF FF         jmp     loop_fucking_vm  
```

## VM_Struct 定义

上述逻辑涉及结构体 `VM_Struct`，其定义如下：

```
00000000 VM_Struct       struc   ; (sizeof=0x74)  
00000000 regEAX          dd ?  
00000004 regECX          dd ?  
00000008 regEDX          dd ?  
0000000C regEBX          dd ?  
00000010 regESP          dd ?  
00000014 regEBP          dd ?  
00000018 regESI          dd ?  
0000001C regEDI          dd ?  
00000020 eflags          dd ?  
00000024 count           dd ?  
00000028 Curr_PCode      db 32 dup(?)  
00000048 VM_regEAX       dd ?  
0000004C VM_regECX       dd ?  
00000050 VM_regEDX       dd ?  
00000054 VM_regEBX       dd ?  
00000058 VM_regESP       dd ?  
0000005C VM_regEBP       dd ?  
00000060 VM_regESI       dd ?  
00000064 VM_regEDI       dd ?  
00000068 VM_eflags       dd ?  
0000006C VM_count        dd ?  
00000070 VM_Data         dd ?  
00000074 VM_Struct       ends  
```

## 初始化流程解析

在函数 `set_vm_struct` 开始时，首先将作为参数传入的 `regs` 保存到 `VM_Struct` 的偏移 `0h-20h` 处，然后将参数 `VM_Data` 保存到偏移 `70h`。接着，复制一份当前的 `PCode_data` 到偏移 `28h`，总共 `20h` 字节（即 32 字节），这就是前面提到的 `PCode`。这一步也解释了为何结构体中存在 `38h`, `39h` 等偏移。

随后设置 VM 寄存器。`VM_Struct` 中包含两套寄存器：
1. 真实的物理寄存器映射，位于 `0-20h`。
2. 参与 VM 内部运算的寄存器，位于 `48h-68h`。

这套 VM 寄存器在 `set_vm_struct` 后期通过真实的寄存器进行赋值。需要注意的是，`VM_regESP` 和 `VM_regEBP` 是通过其他途径赋值的（见上文代码注释）。VM 执行的 `push` 和 `pop` 操作都是相对于 `vm_stack` 的，这一点至关重要。

## PCode 解析与 Dispatch 逻辑

准备工作完成后，开始解析 `PCode`。系统共有 `0x23` 个 dispatch 分支，这些在我的示例程序中均有体现，此处不再赘述。主要关注的逻辑如下：

### case_dispatch_13_run_in_stack

该分支用于将一些 x86 代码通过 `PCode` 的 `x86code[0x10]` 传递过来，并放入 `vm_stack` 中执行。具体实现可参考 `idb` 文件。

### 已知 Bug：case_dispatch_5_m 与 case_dispatch_9_m

该 VM 存在一个 bug，具体体现在 `case_dispatch_5_m` 和 `case_dispatch_9_m` 两个分支上。
- `case_dispatch_5_m` 的作用是将内存中的一段内容通过 `movs` 移动到 `vm_stack` 中。
- `case_dispatch_9_m` 的方向正好相反。

```
RYOMA:2C8AE01E                       case_dispatch_5_m:          ; CODE XREF: VM_+74j  
RYOMA:2C8AE01E                                                     ; DATA XREF: VM_:VM_Dispatch_Tableo  
RYOMA:2C8AE01E 8B 43 40               mov     eax, [ebx+40h]  ; jumptable 2C8ADE64 case 5  
RYOMA:2C8AE021 85 C0                  test    eax, eax        ; size  
RYOMA:2C8AE023 79 03                  jns     short loc_2C8AE028  
RYOMA:2C8AE025 83 C0 03               add     eax, 3  
RYOMA:2C8AE028                        
RYOMA:2C8AE028                       loc_2C8AE028:               ; CODE XREF: VM_+233j  
RYOMA:2C8AE028 C1 F8 02               sar     eax, 2    //开始时候是按照 DWORD 对其传送的  
RYOMA:2C8AE02B 89 45 F8               mov     [ebp+var_8_count], eax  
RYOMA:2C8AE02E 33 C0                  xor     eax, eax  
RYOMA:2C8AE030 89 45 FC               mov     [ebp+var_4_i_regs], eax  
RYOMA:2C8AE033 83 7D F8 00            cmp     [ebp+var_8_count], 0  
RYOMA:2C8AE037 74 1D                  jz      short loc_2C8AE056  
RYOMA:2C8AE039                        
RYOMA:2C8AE039                       loc_2C8AE039:               ; CODE XREF: VM_+264j  
RYOMA:2C8AE039 8B 43 58               mov     eax, [ebx+VM_Struct.VM_regESP]  
RYOMA:2C8AE03C 03 45 FC               add     eax, [ebp+var_4_i_regs]  
RYOMA:2C8AE03F 8B 00                  mov     eax, [eax]  
RYOMA:2C8AE041 8B 53 3B               mov     edx, [ebx+3Bh]  
RYOMA:2C8AE044 03 55 FC               add     edx, [ebp+var_4_i_regs]  
RYOMA:2C8AE047 89 02                  mov     [edx], eax  
RYOMA:2C8AE049 83 45 FC 04            add     [ebp+var_4_i_regs], 4  
RYOMA:2C8AE04D FF 4D F8               dec     [ebp+var_8_count]  
RYOMA:2C8AE050 83 7D F8 00            cmp     [ebp+var_8_count], 0  
RYOMA:2C8AE054 75 E3                  jnz     short loc_2C8AE039  
RYOMA:2C8AE056                        
RYOMA:2C8AE056                       loc_2C8AE056:               ; CODE XREF: VM_+247j  
RYOMA:2C8AE056 8B 43 40               mov     eax, [ebx+40h]  
RYOMA:2C8AE059 25 03 00 00 80         and     eax, 80000003h    //这里是传送多余出来的 BYTE  
RYOMA:2C8AE05E 79 05                  jns     short loc_2C8AE065  
RYOMA:2C8AE060 48                      dec     eax  
RYOMA:2C8AE061 83 C8 FC               or      eax, 0FFFFFFFCh  
RYOMA:2C8AE064 40                      inc     eax  
RYOMA:2C8AE065                        
RYOMA:2C8AE065                       loc_2C8AE065:               ; CODE XREF: VM_+26Ej  
RYOMA:2C8AE065 89 45 F8               mov     [ebp+var_8_count], eax  
RYOMA:2C8AE068 33 C0                  xor     eax, eax  
RYOMA:2C8AE06A 89 45 FC               mov     [ebp+var_4_i_regs], eax  //出错的地方在这里，var_4_i_regs 清 0 了，又从开始地方传起来了  
RYOMA:2C8AE06D 83 7D F8 00            cmp     [ebp+var_8_count], 0  
RYOMA:2C8AE071 74 1D                  jz      short loc_2C8AE090  
RYOMA:2C8AE073                        
RYOMA:2C8AE073                       loc_2C8AE073:               ; CODE XREF: VM_+29Ej  
RYOMA:2C8AE073 8B 43 58               mov     eax, [ebx+VM_Struct.VM_regESP]  
RYOMA:2C8AE076 03 45 FC               add     eax, [ebp+var_4_i_regs]  
RYOMA:2C8AE079 0F B6 00               movzx   eax, byte ptr [eax]  
RYOMA:2C8AE07C 8B 53 3B               mov     edx, [ebx+3Bh]  
```



```
RYOMA:2C8AE07F 03 55 FC                     add     edx, [ebp+var_4_i_regs]  

RYOMA:2C8AE082 88 02                        mov     [edx], al  

RYOMA:2C8AE084 FF 45 FC                      inc     [ebp+var_4_i_regs]  

RYOMA:2C8AE087 FF 4D F8                       dec     [ebp+var_8_count]  

RYOMA:2C8AE08A 83 7D F8 00                   cmp     [ebp+var_8_count], 0  

RYOMA:2C8AE08E 75 E3                         jnz     short loc_2C8AE073  
```

`case_disptach_9_movs` 也是这个问题。

差不多就这么多，不懂的可以看程序，和 `idb`。

执行完 `vm` 后，会

```
pop eax

pop ebx

pop edx

pop ecx

pop esi

pop edi

pop ebp

popfd

ret 4
```

返回到上面提到的那个返回地址。

`Private` 执行是会释放一个 `dll` 到自身空间 `2C48C000` 里面，这个 `dll` 就是这个 `vm` 引擎所在地方，我把它剥离出来，`fix` 一下，可以反汇编大概看看。

`^_^`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `分析.rar` |
|
