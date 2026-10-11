
# 汇编代码分析

## 1. 前置验证片段

```
004C6AEE   |.  4F                 dec edi  
004C6AEF   |.^ 75 EC             jnz short XXXXXXXX.004C6ADD
```

紧接着，验证前 3 位：

## 2. 核心验证循环

```
004D1D1A    > /BE 01000000      mov esi,1                       ;  启动计算  
004D1D1F    . |A1 BC4B4D00      mov eax,[dword ds:4D4BBC]  
004D1D24    . |8B00             mov eax,[dword ds:eax]          ;  用户名 5 位  
004D1D26    . |E8 AD2EF3FF      call XXXXXXXX.00404BD8  
004D1D2B    . |8BC8             mov ecx,eax  
004D1D2D    . |85C9             test ecx,ecx  
004D1D2F    . |7E 29            jle short XXXXXXXX.004D1D5A  
004D1D31    . |BB 01000000      mov ebx,1  
004D1D36    > |A1 BC4B4D00      mov eax,[dword ds:4D4BBC]       ;  循环乘,除  
004D1D3B    . |8B00             mov eax,[dword ds:eax]          ;  ASCII "scxtb"  
004D1D3D    . |0FB64418 FF      movzx eax,[byte ds:eax+ebx-1]   ;  取第 i 位  
004D1D42    . |F7EE             imul esi                        ;  *esi,上轮结果,初为 1  
004D1D44    . |0305 E4604D00   add eax,[dword ds:4D60E4]       ;  +1  
004D1D4A    . |03C3             add eax,ebx                     ;  +ebx,轮次!  
004D1D4C    . |BE 65010000      mov esi,165  
004D1D51    . |99               cdq  
004D1D52    . |F7FE             idiv esi                        ;  /165  
004D1D54    . |8BF2             mov esi,edx                     ;  结果取余数  
004D1D56    . |43               inc ebx  
004D1D57    . |49               dec ecx  
004D1D58    . ^|75 DC            jnz short XXXXXXXX.004D1D36  
004D1D5A    > |8BC6             mov eax,esi                     ;  循环结果  
004D1D5C    . |B9 24000000      mov ecx,24  
004D1D61    . |99               cdq  
004D1D62    . |F7F9             idiv ecx                        ;  /24  
004D1D64    . |B8 B8244D00      mov eax,XXXXXXXX.004D24B8       ;  ASCII "0123456789abcdefghijklmnopqrstuvwxyz"  
004D1D69    . |8A0410           mov al,[byte ds:eax+edx]        ;  按余数取相应位!  
004D1D6C    . |8B15 B8484D00   mov edx,[dword ds:4D48B8]       ;  XXXXXXXX.004D4700  
004D1D72    . |8B12             mov edx,[dword ds:edx]  
004D1D74    . |8B0D E4604D00   mov ecx,[dword ds:4D60E4]  
004D1D7A    . |3A440A FF        cmp al,[byte ds:edx+ecx-1]      ;  与注册码比较  
004D1D7E    . |74 08            je short XXXXXXXX.004D1D88  
004D1D80    . |A1 B44B4D00      mov eax,[dword ds:4D4BB4]  
004D1D85    . |C600 01          mov [byte ds:eax],1  
004D1D88    > |FF05 E4604D00    inc [dword ds:4D60E4]  
004D1D8E    . |833D E4604D00   cmp [dword ds:4D60E4],4         ;  验证前 3 位?  
004D1D95    . ^\75 83            jnz short XXXXXXXX.004D1D1A
```

我们可以看出是与注册过程是相同的。不同是在下面，它验证了前 4--9 位：

## 3. 后续验证与比较

```
004D1E31    .  8D45 C8         lea eax,[dword ss:ebp-38]  
004D1E34    .  50             push eax                    ;  注册码!  
004D1E35    .  A1 B8484D00    mov eax,[dword ds:4D48B8]  
004D1E3A    .  8B00           mov eax,[dword ds:eax]      ;  ASCII "h6s015lphi00000448"  
004D1E3C    .  B9 09000000    mov ecx,9  
004D1E41    .  BA 04000000    mov edx,4  
004D1E46    .  E8 6DC0FEFF    call XXXXXXXX.004BDEB8      ;  取 4-9 位?  
004D1E4B    .  8B55 C8        mov edx,[dword ss:ebp-38]  
004D1E4E    .  B8 E8604D00    mov eax,XXXXXXXX.004D60E8  
004D1E53    .  E8 1C2BF3FF    call XXXXXXXX.00404974      ;  是否为 0?  
004D1E58    .  33C0           xor eax,eax  
004D1E5A    .  55             push ebp  
004D1E5B    .  68 7C1E4D00    push XXXXXXXX.004D1E7C  
004D1E60    .  64:FF30        push [dword fs:eax]  
004D1E63    .  64:8920        mov [dword fs:eax],esp  
004D1E66    .  A1 E8604D00    mov eax,[dword ds:4D60E8]   ;  4-9 位  
004D1E6B    .  E8 E075F3FF    call XXXXXXXX.00409450      ； 验证！  
004D1E70    .  8BD8           mov ebx,eax  
004D1E72    .  33C0           xor eax,eax
```

这里计算验证出来后,在下面还有个计算比较：

```
004D1E8C    .  E8 E724F3FF    call unpacked.00404378  
004D1E91    >  8BC3            mov eax,ebx  
004D1E93    .  B9 10270000    mov ecx,2710  
004D1E98    .  99             cdq  
004D1E99    .  F7F9           idiv ecx                     ;   
004D1E9B    .  83F8 0A        cmp eax,0A  
004D1E9E    .  74 08          je short unpacked.004D1EA8   ;   
004D1EA0    .  A1 B44B4D00    mov eax,[dword ds:4D4BB4]  
004D1EA5    .  C600 01        mov [byte ds:eax],1  
004D1EA8    >  E8 770AFFFF    call unpacked.004C2924
```

（由于某种原因，我不能说明这里的关系了）

如果你有兴趣看到这，是否也觉得它的注册验证比较有意思?它这个样子可有不少麻烦的，我一共用了 `7` 天来搞它的，回头想想，也就知道我为什么是莱鸟了！

祝元旦快乐！
![image](img/710b5589904d0d2bf58a67748fca40e1.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
