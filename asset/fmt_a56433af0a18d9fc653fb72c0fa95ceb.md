
# 汇编指令序列整理

## 入口点 `004DAAFC`

```
004DAAFC     58                      pop eax
```

## 执行序列 `004DAACA` 至 `004DAAD7`

```
004DAACA     53                      push ebx
004DAACB     56                      push esi
004DAACC     33C9                    xor ecx,ecx
004DAACE     894D D8                 mov dword ptr ss:[ebp-28],ecx
004DAAD1     894D D0                 mov dword ptr ss:[ebp-30],ecx
004DAAD4     894D CC                 mov dword ptr ss:[ebp-34],ecx
004DAAD7     894D D4                 mov dword ptr ss:[ebp-2C],ecx
```

## 执行序列 `004DAADA` 至 `004DAAEC`

```
004DAADA     8BD8                    mov ebx,eax
004DAADC     33C0                    xor eax,eax
004DAADE     55                      push ebp
004DAADF     68 9FAE4D00             push Dump3.004DAE9F
004DAAE4     64:FF30                 push dword ptr fs:[eax]
004DAAE7     64:8920                 mov dword ptr fs:[eax],esp
004DAAEA     60                      pushad
004DAAEB     FC                      cld
004DAAEC     1BCE                    sbb ecx,esi
```

## 执行序列 `004DAAEE` 至 `004DAAF3`

```
004DAAEE     C1E9 EA                 shr ecx,0EA
004DAAF1     72 01                   jb short Dump3.004DAAF4
004DAAF3     43                      inc ebx
```

## 执行序列 `004DAAF4` 至 `004DAAFC`

```
004DAAF4     90                      nop
004DAAF5     90                      nop
004DAAF6     90                      nop
004DAAF7     90                      nop
004DAAF8     90                      nop
004DAAF9     90                      nop
004DAAFA     90                      nop
004DAAFB     90                      nop
004DAAFC     90                      nop
```

## 执行序列 `004DAAFD` 至 `004DAB0C`

```
004DAAFD     0F88 02000000          js Dump3.004DAB05
004DAB03     0BDD                    or ebx,ebp
004DAB05     90                      nop
004DAB06     90                      nop
004DAB07     90                      nop
004DAB08     90                      nop
004DAB09     90                      nop
004DAB0A     90                      nop
004DAB0B     90                      nop
004DAB0C     90                      nop
```

## 执行序列 `004DAB0D` 至 `004DAB1F`

```
004DAB0D     90                      nop
004DAB0E     F9                      stc
004DAB0F     90                      nop
004DAB10     90                      nop
004DAB11     90                      nop
004DAB12     90                      nop
004DAB13     90                      nop
004DAB14     F9                      stc
004DAB15     90                      nop
004DAB16     90                      nop
004DAB17     90                      nop
004DAB18     90                      nop
004DAB19     90                      nop
004DAB1A     90                      nop
004DAB1B     90                      nop
004DAB1C     90                      nop
004DAB1D     90                      nop
004DAB1E     90                      nop
004DAB1F     90                      nop
```

## 执行序列 `004DAB20` 至 `004DAB2D`

```
004DAB20     66:BD 2B85              mov bp,852B
004DAB24     90                      nop
004DAB25     90                      nop
004DAB26     90                      nop
004DAB27     90                      nop
004DAB28     90                      nop
004DAB29     90                      nop
004DAB2A     90                      nop
004DAB2B     90                      nop
004DAB2C     90                      nop
004DAB2D     74 02                   je short Dump3.004DAB31
```

## 执行序列 `004DAB2F` 至 `004DAB36`

```
004DAB2F     85E9                    test ecx,ebp
004DAB31     E8 00000000             call Dump3.004DAB36
004DAB36     5D                      pop ebp
```

## 执行序列 `004DAB37` 至 `004DAB44`

```
004DAB37     8BC5                    mov eax,ebp
004DAB39     3B45 26                 cmp eax,dword ptr ss:[ebp+26]
004DAB3C     7C 06                   jl short Dump3.004DAB44
004DAB3E     0345 26                 add eax,dword ptr ss:[ebp+26]
004DAB41     8945 26                 mov dword ptr ss:[ebp+26],eax ; Dump3.004DAC9A
004DAB44     90                      nop
```

## 执行序列 `004DAB45` 至 `004DAB49`

```
004DAB45     90                      nop
004DAB46     90                      nop
004DAB47     90                      nop
004DAB48     90                      nop
004DAB49     85CB                    test ebx,ecx
```

## 执行序列 `004DAB4B` 至 `004DAB53`

```
004DAB4B     0F89 02000000          jns Dump3.004DAB53
004DAB51     85CD                    test ebp,ecx
004DAB53     0F89 02000000          jns Dump3.004DAB5B
```

## 执行序列 `004DAB59` 至 `004DAB5B`

```
004DAB59     D3E5                    shl ebp,cl
004DAB5B     B8 64010000             mov eax,164
```

## 执行序列 `004DAB60` 至 `004DAB6D`

```
004DAB60     90                      nop
004DAB61     90                      nop
004DAB62     90                      nop
004DAB63     90                      nop
004DAB64     90                      nop
004DAB65     90                      nop
004DAB66     90                      nop
004DAB67     90                      nop
004DAB68     90                      nop
004DAB69     90                      nop
004DAB6A     90                      nop
004DAB6B     90                      nop
004DAB6C     90                      nop
004DAB6D     E9 04000000             jmp Dump3.004DAB76
```

## 执行序列 `004DAB72` 至 `004DAB76`

```
004DAB72     66:8BCF                 mov cx,di
004DAB75     F9                      stc
004DAB76     BE F35C587D             mov esi,7D585CF3
```

## 执行序列 `004DAB7B` 至 `004DAB8A`

```
004DAB7B     90                      nop
004DAB7C     90                      nop
004DAB7D     90                      nop
004DAB7E     90                      nop
004DAB7F     90                      nop
004DAB80     90                      nop
004DAB81     90                      nop
004DAB82     90                      nop
004DAB83     90                      nop
004DAB84     90                      nop
004DAB85     90                      nop
004DAB86     90                      nop
004DAB87     90                      nop
004DAB88     90                      nop
004DAB89     90                      nop
004DAB8A     75 02                   jnz short Dump3.004DAB8E
```

## 执行序列 `004DAB8C` 至 `004DAB8E`

```
004DAB8C     87DD                    xchg ebp,ebx
004DAB8E     C1C5 50                 rol ebp,50
```

## 执行序列 `004DAB91` 至 `004DAB9D`

```
004DAB91     68 6A000000             push 6A
004DAB96     0F88 01000000          js Dump3.004DAB9D
004DAB9C     4B                      dec ebx
004DAB9D     5F                      pop edi
```

## 执行序列 `004DAB9E` 至 `004DABBA`

```
004DAB9E     90                      nop
004DAB9F     90                      nop
004DABA0     90                      nop
004DABA1     F9                      stc
004DABA2     87EB                    xchg ebx,ebp
004DABA4     66:BB 2F05              mov bx,52F
004DABA8     8B10                    mov edx,dword ptr ds:[eax] // 修改这里，因为下面的循环到这里。
004DABA8     /E9 EA000000            jmp Dump3.004DAC97 // 直接到保护地点。
004DABAD     |90                     nop
004DABAE     |90                     nop
```

## 执行序列 `004DABAA` 至 `004DABBE`

```
004DABAA     90                      nop
004DABAB     90                      nop
004DABAC     90                      nop
004DABAD     90                      nop
004DABAE     90                      nop
004DABAF     90                      nop
004DABB0     90                      nop
004DABB1     90                      nop
004DABB2     90                      nop
004DABB3     90                      nop
004DABB4     90                      nop
004DABB5     E9 04000000             jmp Dump3.004DABBE
004DABBA     66:D3D1                 rcl cx,cl
004DABBD     F8                      clc
004DABBE     0F85 01000000          jnz Dump3.004DABC5
```

## 执行序列 `004DABC4` 至 `004DABCD`

```
004DABC4     F9                      stc
004DABC5     81E1 6E082B94           and ecx,942B086E
004DABCB     33D6                    xor edx,esi
004DABCD     90                      nop
```

## 执行序列 `004DABCE` 至 `004DABE4`

```
004DABCE     90                      nop
004DABCF     90                      nop
004DABD0     90                      nop
004DABD1     90                      nop
004DABD2     90                      nop
004DABD3     90                      nop
004DABD4     90                      nop
004DABD5     90                      nop
004DABD6     90                      nop
004DABD7     90                      nop
004DABD8     66:D3F9                 sar cx,cl
004DABDB     0F86 03000000          jbe Dump3.004DABE4
004DABE1     66:23E9                 and bp,cx
004DABE4     E9 0A000000             jmp Dump3.004DABF3
```

## 执行序列 `004DABE9` 至 `004DABF3`

```
004DABE9     81F1 0D64113C           xor ecx,3C11640D
004DABEF     66:B9 F8AA              mov cx,0AAF8
004DABF3     C1C2 17                 rol edx,17
```

## 执行序列 `004DABF6` 至 `004DABFA`

```
004DABF6     90                      nop
004DABF7     90                      nop
004DABF8     90                      nop
004DABF9     F9                      stc
004DABFA     41                      inc ecx
```
