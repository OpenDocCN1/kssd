
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
