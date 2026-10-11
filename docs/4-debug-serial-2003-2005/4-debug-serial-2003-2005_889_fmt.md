
# 利康医药进销存系统算法的分析附注册机程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-295.htm`](https://www.pediy.com/kssd/pediy07/pediy7-295.htm)

## 基本信息

- 【破解作者】`jsliyangsj`
- 【作者邮箱】`sjcrack@yahoo.com.cn`
- 【使用工具】`peid`, `OllyDbg1.10`
- 【破解平台】`Winxp`
- 【软件名称】`利康医药进销存系统`
- 【软件地址】`http://www.topthinksoft.com/`
- 【编写语言】`Borland Delphi`

查找字符串“输入注册码不正确”来到关键点：

```
00662EDC   /.  55 push ebp  
00662EDD   |.  8B>mov ebp,esp  
00662EDF   |.  33>xor ecx,ecx  
00662EE1   |.  51 push ecx  
00662EE2   |.  51 push ecx  
00662EE3   |.  51 push ecx  
00662EE4   |.  51 push ecx  
00662EE5   |.  51 push ecx  
00662EE6   |.  51 push ecx  
00662EE7   |.  51 push ecx  
00662EE8   |.  53 push ebx  
00662EE9   |.  8B>mov ebx,eax  
00662EEB   |.  33>xor eax,eax  
00662EED   |.  55 push ebp  
00662EEE   |.  68>push LKYD.0066301C  
00662EF3   |.  64>push dword ptr fs:[eax]  
00662EF6   |.  64>mov dword ptr fs:[eax],esp  
00662EF9   |.  8D>lea edx,dword ptr ss:[ebp-C]  
00662EFC   |.  8B>mov eax,dword ptr ds:[ebx+31C]  
00662F02   |.  E8>call LKYD.0047EC60                 ;  得到注册码  
00662F07   |.  8B>mov eax,dword ptr ss:[ebp-C]  
00662F0A   |.  8D>lea edx,dword ptr ss:[ebp-8]  
00662F0D   |.  E8>call LKYD.00409968  
00662F12   |.  8B>mov eax,dword ptr ss:[ebp-8]  
00662F15   |.  50 push eax  
00662F16   |.  8D>lea edx,dword ptr ss:[ebp-14]  
00662F19   |.  8B>mov eax,dword ptr ds:[ebx+318]  
00662F1F   |.  E8>call LKYD.0047EC60                 ;  得到固定字符串 TOPTHINK  
00662F24   |.  8B>mov eax,dword ptr ss:[ebp-14]  
00662F27   |.  8D>lea edx,dword ptr ss:[ebp-10]  
00662F2A   |.  E8>call LKYD.00409968  
00662F2F   |.  8B>mov eax,dword ptr ss:[ebp-10]  
00662F32   |.  50 push eax  
00662F33   |.  8D>lea edx,dword ptr ss:[ebp-1C]  
00662F36   |.  8B>mov eax,dword ptr ds:[ebx+314]  
00662F3C   |.  E8>call LKYD.0047EC60  
00662F41   |.  8B>mov eax,dword ptr ss:[ebp-1C]  
00662F44   |.  8D>lea edx,dword ptr ss:[ebp-18]  
00662F47   |.  E8>call LKYD.00409968  
00662F4C   |.  8B>mov edx,dword ptr ss:[ebp-18]  
00662F4F   |.  8B>mov eax,dword ptr ds:[ebx+32C]  
00662F55   |.  59 pop ecx  
00662F56   |.  E8>call LKYD.005BBE98                 ;  关键点！！！进入  
00662F5B   |.  84>test al,al  
00662F5D   |.  75>jnz short LKYD.00662F8B  
00662F5F   |.  8D>lea eax,dword ptr ss:[ebp-4]  
00662F62   |.  BA>mov edx,LKYD.00663030              ;  输入注册码不正确  
00662F67   |.  E8>call LKYD.00404BCC  
00662F6C   |.  6A>push 40  
00662F6E   |.  8B>mov eax,dword ptr ss:[ebp-4]  
00662F71   |.  E8>call LKYD.00404FF4  
00662F76   |.  8B>mov edx,eax  
00662F78   |.  B9>mov ecx,LKYD.0066304C  
00662F7D   |.  A1>mov eax,dword ptr ds:[6A63B0]  
00662F82   |.  8B>mov eax,dword ptr ds:[eax]  
00662F84   |.  E8>call LKYD.004A04F0  
00662F89   |.  EB>jmp short LKYD.00662FD9  
00662F8B   |>  68>push LKYD.00663060                 ;  注册成功  
00662F90   |.  8B>mov eax,dword ptr ds:[ebx+32C]  
00662F96   |.  FF>push dword ptr ds:[eax+5C]  
………………………………………………………………………………………………………………………………
```

进入关键点：`00662F56   |.  E8>call LKYD.005BBE98`

………………………………………………………………………………………………………………………………

```
005BBE98   /$  55 push ebp  
005BBE99   |.  8B>mov ebp,esp  
005BBE9B   |.  83>add esp,-10  
005BBE9E   |.  53 push ebx  
005BBE9F   |.  33>xor ebx,ebx  
005BBEA1   |.  89>mov dword ptr ss:[ebp-10],ebx  
005BBEA4   |.  89>mov dword ptr ss:[ebp-C],ebx  
005BBEA7   |.  89>mov dword ptr ss:[ebp-8],ecx  
005BBEAA   |.  89>mov dword ptr ss:[ebp-4],edx  
005BBEAD   |.  8B>mov ebx,eax  
005BBEAF   |.  8B>mov eax,dword ptr ss:[ebp-4]  
005BBEB2   |.  E8>call LKYD.00404FE4  
005BBEB7   |.  8B>mov eax,dword ptr ss:[ebp-8]  
005BBEBA   |.  E8>call LKYD.00404FE4  
005BBEBF   |.  8B>mov eax,dword ptr ss:[ebp+8]  
005BBEC2   |.  E8>call LKYD.00404FE4  
005BBEC7   |.  33>xor eax,eax  
005BBEC9   |.  55 push ebp  
005BBECA   |.  68>push LKYD.005BBF82  
005BBECF   |.  64>push dword ptr fs:[eax]  
005BBED2   |.  64>mov dword ptr fs:[eax],esp  
005BBED5   |.  8B>mov eax,dword ptr ss:[ebp-4]  
005BBED8   |.  E8>call LKYD.00404DF4  
005BBEDD   |.  3B>cmp eax,dword ptr ds:[ebx+4C]  
005BBEE0   |.  7F>jg short LKYD.005BBEFB             ;  位数大于 100  
005BBEE2   |.  8B>mov eax,dword ptr ss:[ebp-4]  
005BBEE5   |.  E8>call LKYD.00404DF4                 ;  得到位数  
005BBEEA   |.  3B>cmp eax,dword ptr ds:[ebx+50]  
005BBEED   |.  7C>jl short LKYD.005BBEFB             ;  位数小于 3  
005BBEEF   |.  8B>mov eax,dword ptr ss:[ebp+8]  
005BBEF2   |.  E8>call LKYD.00404DF4                 ;  我的输入码的位数  
005BBEF7   |.  85>test eax,eax                       ;  输入码不等于 0  
005BBEF9   |.  75>jnz short LKYD.005BBEFF  
005BBEFB   |>  33>xor ebx,ebx  
005BBEFD   |.  EB>jmp short LKYD.005BBF5F  
005BBEFF   |>  8D>lea edx,dword ptr ss:[ebp-C]  
005BBF02   |.  8B>mov eax,dword ptr ss:[ebp+8]  
005BBF05   |.  E8>call LKYD.004096F8                 ;  拷贝注册码  
005BBF0A   |.  8B>mov edx,dword ptr ss:[ebp-C]  
005BBF0D   |.  8D>lea eax,dword ptr ss:[ebp+8]  
005BBF10   |.  E8>call LKYD.00404BCC  
005BBF15   |.  8D>lea ecx,dword ptr ss:[ebp-10]  
005BBF18   |.  8B>mov edx,dword ptr ss:[ebp-4]  
005BBF1B   |.  8B>mov eax,ebx  
005BBF1D   |.  E8>call LKYD.005BBA68                 ;  根据机器码计算注册吗的地方  
005BBF22   |.  8B>mov eax,dword ptr ss:[ebp-10]  
005BBF25   |.  8B>mov edx,dword ptr ss:[ebp+8]  
005BBF28   |.  E8>call LKYD.00409770                 ;  比较  
005BBF2D   |.  85>test eax,eax  
005BBF2F   |.  74>je short LKYD.005BBF35             ;  一定要等于  
005BBF31   |.  33>xor ebx,ebx  
005BBF33   |.  EB>jmp short LKYD.005BBF5F  
005BBF35   |>  8D>lea eax,dword ptr ds:[ebx+48]  
005BBF38   |.  8B>mov edx,dword ptr ss:[ebp-4]  
005BBF3B   |.  E8>call LKYD.00404B88  
005BBF40   |.  8D>lea eax,dword ptr ds:[ebx+54]  
005BBF43   |.  8B>mov edx,dword ptr ss:[ebp-8]  
005BBF46   |.  E8>call LKYD.00404B88  
005BBF4B   |.  8D>lea eax,dword ptr ds:[ebx+5C]  
005BBF4E   |.  8B>mov edx,dword ptr ss:[ebp+8]  
005BBF51   |.  E8>call LKYD.00404B88  
005BBF56   |.  8B>mov eax,ebx  
005BBF58   |.  E8>call LKYD.005BC1B8                 ;  又计算一遍  
005BBF5D   |.  B3>mov bl,1                             ;  一定要到这里  
………………………………………………………………………………………………………………………………
```

进入根据机器码计算注册码的地方 `005BBF1D   |.  E8>call LKYD.005BBA68`

………………………………………………………………………………………………………………………………

```
005BBA83   |.  8B>mov esi,eax  
005BBA85   |.  8B>mov eax,dword ptr ss:[ebp-4]  
005BBA88   |.  E8>call LKYD.00404FE4  
005BBA8D   |.  33>xor eax,eax  
005BBA8F   |.  55 push ebp  
005BBA90   |.  68>push LKYD.005BBC08  
005BBA95   |.  64>push dword ptr fs:[eax]  
005BBA98   |.  64>mov dword ptr fs:[eax],esp  
005BBA9B   |.  8D>lea edx,dword ptr ss:[ebp-24]  
005BBA9E   |.  8B>mov eax,esi  
005BBAA0   |.  E8>call LKYD.005BC9AC                 ;  得到机器码！！！00003ADE67D5  
005BBAA5   |.  8B>mov eax,dword ptr ss:[ebp-24]  
005BBAA8   |.  8D>lea edx,dword ptr ss:[ebp-14]  
005BBAAB   |.  E8>call LKYD.00409968  
005BBAB0   |.  83>cmp dword ptr ss:[ebp-14],0         ;  机器码是否为 0  
005BBAB4   |.  75>jnz short LKYD.005BBAC3  
005BBAB6   |.  8D>lea eax,dword ptr ss:[ebp-20]  
005BBAB9   |.  8B>mov edx,dword ptr ss:[ebp-4]  
005BBABC   |.  E8>call LKYD.00404BCC  
005BBAC1   |.  EB>jmp short LKYD.005BBB20  
005BBAC3   |>  8B>mov eax,dword ptr ss:[ebp-14]  
005BBAC6   |.  E8>call LKYD.00404DF4                 ;  得到机器码位数  
005BBACB   |.  8B>mov ebx,eax  
005BBACD   |.  8D>lea eax,dword ptr ss:[ebp-18]  
005BBAD0   |.  50 push eax  
005BBAD1   |.  8B>mov ecx,ebx  
005BBAD3   |.  D1>sar ecx,1  
005BBAD5   |.  79>jns short LKYD.005BBADA  
005BBAD7   |.  83>adc ecx,0  
005BBADA   |>  BA>mov edx,1  
005BBADF   |.  8B>mov eax,dword ptr ss:[ebp-14]  
005BBAE2   |.  E8>call LKYD.00405054                 ;  得到机器码前 6 位  
005BBAE7   |.  8D>lea eax,dword ptr ss:[ebp-1C]  
```



## 反汇编代码段 1

```asm
005BBAEA   |.  50 push eax
005BBAEB   |.  8B>mov eax,ebx
005BBAED   |.  D1>sar eax,1
005BBAEF   |.  79>jns short LKYD.005BBAF4
005BBAF1   |.  83>adc eax,0
005BBAF4   |>  8B>mov ecx,ebx
005BBAF6   |.  2B>sub ecx,eax
005BBAF8   |.  8B>mov edx,ebx
005BBAFA   |.  D1>sar edx,1
005BBAFC   |.  79>jns short LKYD.005BBB01
005BBAFE   |.  83>adc edx,0
005BBB01   |>  42 inc edx
005BBB02   |.  8B>mov eax,dword ptr ss:[ebp-14]
005BBB05   |.  E8>call LKYD.00405054                   ;  得到机器码后 6 位
005BBB0A   |.  FF>push dword ptr ss:[ebp-18]           ;  前 6
005BBB0D   |.  FF>push dword ptr ss:[ebp-4]            ;  固定值 TOPTHINK
005BBB10   |.  FF>push dword ptr ss:[ebp-1C]           ;  后 6
005BBB13   |.  8D>lea eax,dword ptr ss:[ebp-20]
005BBB16   |.  BA>mov edx,3
005BBB1B   |.  E8>call LKYD.00404EB4                   ;  组合起来机器码的前 6 位加上固定字符窜加上机器码的后 6 位
005BBB20   |>  C7>mov dword ptr ss:[ebp-10],0          ;  组合后 00003ATOPTHINKDE67D5
005BBB27   |.  C7>mov dword ptr ss:[ebp-C],0
005BBB2E   |.  8B>mov eax,dword ptr ss:[ebp-4]
005BBB31   |.  E8>call LKYD.00404DF4
005BBB36   |.  3B>cmp eax,dword ptr ds:[esi+4C]
005BBB39   |.  7F>jg short LKYD.005BBB48
005BBB3B   |.  8B>mov eax,dword ptr ss:[ebp-4]
005BBB3E   |.  E8>call LKYD.00404DF4
005BBB43   |.  3B>cmp eax,dword ptr ds:[esi+50]
005BBB46   |.  7D>jge short LKYD.005BBB54
005BBB48   |>  8B>mov eax,edi
005BBB4A   |.  E8>call LKYD.00404B34
005BBB4F   |.  E9>jmp LKYD.005BBBE5
005BBB54   |>  8B>mov eax,dword ptr ss:[ebp-20]        ;  组合后的
005BBB57   |.  E8>call LKYD.00404DF4                   ;  得到组合后的位数
005BBB5C   |.  8B>mov ebx,eax
005BBB5E   |.  EB>jmp short LKYD.005BBB97
005BBB60   |>  8B>/mov eax,dword ptr ss:[ebp-10]       ;  取上次计算结果
005BBB63   |.  8B>|mov edx,dword ptr ss:[ebp-C]        ;  固定值 0
005BBB66   |.  03>|add eax,dword ptr ds:[esi+68]       ;  上次的结果值加上 0x75Bcd15
005BBB69   |.  13>|adc edx,dword ptr ds:[esi+6C]       ;  进位加法上面有没有进位
005BBB6C   |.  52 |push edx
005BBB6D   |.  50 |push eax                             ;  压入准备相减
005BBB6E   |.  8B>|mov eax,dword ptr ss:[ebp-20]
005BBB71   |.  0F>|movzx eax,byte ptr ds:[eax+ebx-1]   ;  从最后一位得到数据
005BBB76   |.  50 |push eax
005BBB77   |.  B8>|mov eax,459
005BBB7C   |.  5A |pop edx
005BBB7D   |.  8B>|mov ecx,edx
005BBB7F   |.  33>|xor edx,edx
005BBB81   |.  F7>|div ecx                               ;  用固定值 459 除以每一位的 ASCII（从最后一位取）
005BBB83   |.  8B>|mov eax,edx                           ;  得到余数
005BBB85   |.  33>|xor edx,edx
005BBB87   |.  29>|sub dword ptr ss:[esp],eax            ;  用固定值减去余数
005BBB8A   |.  19>|sbb dword ptr ss:[esp+4],edx          ;   进位减法，上面有没有借位
005BBB8E   |.  58 |pop eax
005BBB8F   |.  5A |pop edx
005BBB90   |.  89>|mov dword ptr ss:[ebp-10],eax
005BBB93   |.  89>|mov dword ptr ss:[ebp-C],edx
005BBB96   |.  4B |dec ebx                               ;  位数
005BBB97   |>  8B> mov eax,dword ptr ss:[ebp-20]
005BBB9A   |.  E8>|call LKYD.00404DF4                    ;  得到位数
005BBB9F   |.  3B>|cmp ebx,eax
005BBBA1   |.  7F>|jg short LKYD.005BBBA7
005BBBA3   |.  85>|test ebx,ebx
005BBBA5   |.^ 7F>\\jg short LKYD.005BBB60
005BBBA7   |>  8B>mov ebx,dword ptr ds:[esi+60]
005BBBAA   |.  85>test ebx,ebx
005BBBAC   |.  7F>jg short LKYD.005BBBBF
005BBBAE   |.  FF>push dword ptr ss:[ebp-C]              ; /Arg2
005BBBB1   |.  FF>push dword ptr ss:[ebp-10]             ; |Arg1
005BBBB4   |.  8B>mov edx,edi                            ; |
005BBBB6   |.  33>xor eax,eax                            ; |
005BBBB8   |.  E8>call LKYD.00409F98                     ; \\LKYD.00409F98
005BBBBD   |.  EB>jmp short LKYD.005BBBE5
005BBBBF   |>  FF>push dword ptr ss:[ebp-C]              ; /Arg2
005BBBC2   |.  FF>push dword ptr ss:[ebp-10]             ; |Arg1
005BBBC5   |.  8B>mov edx,edi                            ; |
005BBBC7   |.  8B>mov eax,ebx                            ; |
005BBBC9   |.  E8>call LKYD.00409F98                     ; \\处理成字符
005BBBCE   |.  8B>mov eax,dword ptr ds:[edi]
005BBBD0   |.  E8>call LKYD.00404DF4
005BBBD5   |.  8B>mov ecx,eax
005BBBD7   |.  2B>sub ecx,dword ptr ds:[esi+60]
005BBBDA   |.  8B>mov edx,dword ptr ds:[esi+60]
005BBBDD   |.  42 inc edx
005BBBDE   |.  8B>mov eax,edi
005BBBE0   |.  E8>call LKYD.00405094
005BBBE5   |>  33>xor eax,eax
005BBBE7   |.  5A pop edx
005BBBE8   |.  59 pop ecx
005BBBE9   |.  59 pop ecx
005BBBEA   |.  64>mov dword ptr fs:[eax],edx
005BBBED   |.  68>push LKYD.005BBC0F
005BBBF2   |>  8D>lea eax,dword ptr ss:[ebp-24]
005BBBF5   |.  BA>mov edx,5
005BBBFA   |.  E8>call LKYD.00404B58
005BBBFF   |.  8D>lea eax,dword ptr ss:[ebp-4]
005BBC02   |.  E8>call LKYD.00404B34
005BBC07   \\.  C3 retn
```

…………………………………………………………………………………………＋………………………………

### 算法说明

机器码共 12 位，前面有 4 个 0，拆成前 6 位与后 6 位，中间加上固定的 `TOPTHINK`，组合。一共 20 位。

对它进行计 20 次计算，每一次结果加上 123456789（16 进制 75BCD15）其结果减去，用 16 进制的 459 除以每一位。

机器码与固定字符串的组合的 ASCII（从最后前面取值）。

最后得到的结果 8 位 16 进制数据，把这些数据当作字符，并前面加上 4 个 0 即可！！

…………………………………………………………………………………………………………………………

机器码的计算也和上面一样

………………………………………………………………………………………………………………

## 反汇编代码段 2

```asm
005BC9DD   |.  8B>mov edx,dword ptr ds:[esi+94]          ;  准备计算机器码
005BC9E3   |.  E8>call LKYD.00404BCC
005BC9E8   |.  8B>mov eax,dword ptr ss:[ebp-4]
005BC9EB   |.  E8>call LKYD.00404DF4
005BC9F0   |.  8B>mov ebx,eax
005BC9F2   |.  EB>jmp short LKYD.005BCA2B
005BC9F4   |>  8B>/mov eax,dword ptr ds:[esi+68]          ;  取固定值 0x75bcd15
005BC9F7   |.  8B>|mov edx,dword ptr ds:[esi+6C]          ;  固定值 0
005BC9FA   |.  03>|add eax,dword ptr ss:[ebp-10]          ;  上次的结果值加上 0x75ncd15
005BC9FD   |.  13>|adc edx,dword ptr ss:[ebp-C]           ;  进位加法上面有没有进位
005BCA00   |.  52 |push edx
005BCA01   |.  50 |push eax                                ;  压入准备相减
005BCA02   |.  8B>|mov eax,dword ptr ss:[ebp-4]
005BCA05   |.  0F>|movzx eax,byte ptr ds:[eax+ebx-1]      ;  从最后一位得到数据
005BCA0A   |.  50 |push eax
005BCA0B   |.  B8>|mov eax,459
005BCA10   |.  5A |pop edx
005BCA11   |.  8B>|mov ecx,edx
005BCA13   |.  33>|xor edx,edx
005BCA15   |.  F7>|div ecx                                  ;  用固定值 459 除以每一位的 ASCII（从最后一位取）
005BCA17   |.  8B>|mov eax,edx                              ;  得到余数
005BCA19   |.  33>|xor edx,edx
005BCA1B   |.  29>|sub dword ptr ss:[esp],eax               ;  用固定值减去余数
005BCA1E   |.  19>|sbb dword ptr ss:[esp+4],edx             ;  进位减法，上面有没有借位
005BCA22   |.  58 |pop eax
005BCA23   |.  5A |pop edx
005BCA24   |.  89>|mov dword ptr ss:[ebp-10],eax
005BCA27   |.  89>|mov dword ptr ss:[ebp-C],edx
005BCA2A   |.  4B |dec ebx                                  ;  位数
005BCA2B   |>  8B> mov eax,dword ptr ss:[ebp-4]
005BCA2E   |.  E8>|call LKYD.00404DF4
005BCA33   |.  3B>|cmp ebx,eax
005BCA35   |.  7F>|jg short LKYD.005BCA3B
005BCA37   |.  85>|test ebx,ebx
005BCA39   |.^ 7F>\\jg short LKYD.005BC9F4
005BCA3B   |>  8B>mov ebx,dword ptr ds:[esi+60]
005BCA3E   |.  85>test ebx,ebx
005BCA40   |.  7F>jg short LKYD.005BCA53
005BCA42   |.  FF>push dword ptr ss:[ebp-C]                 ; /Arg2
005BCA45   |.  FF>push dword ptr ss:[ebp-10]                ; |Arg1
005BCA48   |.  8B>mov edx,edi                               ; |
005BCA4A   |.  33>xor eax,eax                               ; |
005BCA4C   |.  E8>call LKYD.00409F98                        ; \\LKYD.00409F98
005BCA51   |.  EB>jmp short LKYD.005BCA79
005BCA53   |>  FF>push dword ptr ss:[ebp-C]                 ; /Arg2
005BCA56   |.  FF>push dword ptr ss:[ebp-10]                ; |Arg1
005BCA59   |.  8B>mov edx,edi                               ; |
005BCA5B   |.  8B>mov eax,ebx                               ; |
005BCA5D   |.  E8>call LKYD.00409F98                        ; \\把上面计算出来的前面加 0 以凑足 12 位
```



`005BCA62` `|` `.` `8B>mov eax,dword ptr ds:[edi]`

…………………………………………………………………………………………………………………………

附：

```c
main()  

{  

unsigned long int guding=0x75bcd15,jieguo=0,zj;  

int i,yushu,b;  

static char zuhe[21]="000000TOPTHINK000000",jiqima[12],zhucema[13]="000000000000";  

printf("shuru jiqima\\n");  

scanf("%s",jiqima);  

for(i=0;i<12;i++)  

{  

if((jiqima[i]>='a')&(jiqima[i]<='z'))  

jiqima[i]=jiqima[i]-0x20;  

}  

for(i=0;i<6;i++)  

{zuhe[i]=jiqima[i];}  

for(i=14;i<20;i++)  

{zuhe[i]=jiqima[i-8];}  

for(i=19;i>=0;i--)  

{  

jieguo=jieguo+guding;  

yushu=0x459%zuhe[i];  

jieguo=jieguo-yushu;  

}  

for(i=11,b=1;i>3;i--,b++)  

{  

zj=jieguo<<(32-b*4);  

zj=zj>>28;  

if((zj>=0)&(zj<=9))  

zj=zj+0x30;  

else if((zj>=0xa)&(zj<=0xf))  

zj=zj+0x37;  

zhucema[i]=zj;  

}  

printf("zhucema:\\n");  

printf("%s",zhucema);  

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
