
# 汇编代码片段

```
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
005BBB05   |.  E8>call LKYD.00405054                   ;  得到机器码后6位
005BBB0A   |.  FF>push dword ptr ss:[ebp-18]           ;  前6
005BBB0D   |.  FF>push dword ptr ss:[ebp-4]            ;  固定值TOPTHINK
005BBB10   |.  FF>push dword ptr ss:[ebp-1C]           ;  后6
005BBB13   |.  8D>lea eax,dword ptr ss:[ebp-20]
005BBB16   |.  BA>mov edx,3
005BBB1B   |.  E8>call LKYD.00404EB4                   ;  组合起来机器码的前6位加上固定字符窜加上机器码的后6位
005BBB20   |>  C7>mov dword ptr ss:[ebp-10],0          ;  组合后00003ATOPTHINKDE67D5
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
005BBB63   |.  8B>|mov edx,dword ptr ss:[ebp-C]        ;  固定值0
005BBB66   |.  03>|add eax,dword ptr ds:[esi+68]       ;  上次的结果值加上0x75Bcd15
005BBB69   |.  13>|adc edx,dword ptr ds:[esi+6C]       ;  进位加法上面有没有进位
005BBB6C   |.  52 |push edx
005BBB6D   |.  50 |push eax                            ;  压入准备相减
005BBB6E   |.  8B>|mov eax,dword ptr ss:[ebp-20]
005BBB71   |.  0F>|movzx eax,byte ptr ds:[eax+ebx-1]   ;  从最后一位得到数据
005BBB76   |.  50 |push eax
005BBB77   |.  B8>|mov eax,459
005BBB7C   |.  5A |pop edx
005BBB7D   |.  8B>|mov ecx,edx
005BBB7F   |.  33>|xor edx,edx
005BBB81   |.  F7>|div ecx                             ;  用固定值459除以每一位的ASCII（从最后一位取）
005BBB83   |.  8B>|mov eax,edx                         ;  得到余数
005BBB85   |.  33>|xor edx,edx
005BBB87   |.  29>|sub dword ptr ss:[esp],eax          ;  用固定值减去余数
005BBB8A   |.  19>|sbb dword ptr ss:[esp+4],edx        ;   进位减法，上面有没有借位
005BBB8E   |.  58 |pop eax
005BBB8F   |.  5A |pop edx
005BBB90   |.  89>|mov dword ptr ss:[ebp-10],eax
005BBB93   |.  89>|mov dword ptr ss:[ebp-C],edx
005BBB96   |.  4B |dec ebx                             ;  位数
005BBB97   |>  8B> mov eax,dword ptr ss:[ebp-20]
005BBB9A   |.  E8>|call LKYD.00404DF4                  ;  得到位数
005BBB9F   |.  3B>|cmp ebx,eax
005BBBA1   |.  7F>|jg short LKYD.005BBBA7
005BBBA3   |.  85>|test ebx,ebx
005BBBA5   |.^ 7F>\jg short LKYD.005BBB60
005BBBA7   |>  8B>mov ebx,dword ptr ds:[esi+60]
005BBBAA   |.  85>test ebx,ebx
005BBBAC   |.  7F>jg short LKYD.005BBBBF
005BBBAE   |.  FF>push dword ptr ss:[ebp-C]            ; /Arg2
005BBBB1   |.  FF>push dword ptr ss:[ebp-10]           ; |Arg1
005BBBB4   |.  8B>mov edx,edi                          ; |
005BBBB6   |.  33>xor eax,eax                          ; |
005BBBB8   |.  E8>call LKYD.00409F98                   ; \LKYD.00409F98
005BBBBD   |.  EB>jmp short LKYD.005BBBE5
005BBBBF   |>  FF>push dword ptr ss:[ebp-C]            ; /Arg2
005BBBC2   |.  FF>push dword ptr ss:[ebp-10]           ; |Arg1
005BBBC5   |.  8B>mov edx,edi                          ; |
005BBBC7   |.  8B>mov eax,ebx                          ; |
005BBBC9   |.  E8>call LKYD.00409F98                   ; \处理成字符
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
005BBC07   \.  C3 retn
```

…………………………………………………………………………………………＋………………………………

## 算法说明

算法：机器码共12位，前面有4个0，拆成前6位与后6位，中间加上固定的`TOPTHINK`，组合。一共20位

对它进行计20次计算，每一次结果加上123456789（16进制`75BCD15`）其结果减去，用16进制的459除以每一位

机器码与固定字符串的组合的ASCII（从最后前面取值）。

最后得到的结果8位16进制数据，把这些数据当作字符，并前面加上4个0即可！！

…………………………………………………………………………………………………………………………

机器码的计算也和上面一样

………………………………………………………………………………………………………………

## 机器码计算相关汇编

```
005BC9DD   |.  8B>mov edx,dword ptr ds:[esi+94]        ;  准备计算机器码
005BC9E3   |.  E8>call LKYD.00404BCC
005BC9E8   |.  8B>mov eax,dword ptr ss:[ebp-4]
005BC9EB   |.  E8>call LKYD.00404DF4
005BC9F0   |.  8B>mov ebx,eax
005BC9F2   |.  EB>jmp short LKYD.005BCA2B
005BC9F4   |>  8B>/mov eax,dword ptr ds:[esi+68]       ;  取固定值0x75bcd15
005BC9F7   |.  8B>|mov edx,dword ptr ds:[esi+6C]       ;  固定值0
005BC9FA   |.  03>|add eax,dword ptr ss:[ebp-10]       ;  上次的结果值加上0x75ncd15
005BC9FD   |.  13>|adc edx,dword ptr ss:[ebp-C]        ;  进位加法上面有没有进位
005BCA00   |.  52 |push edx
005BCA01   |.  50 |push eax                            ;  压入准备相减
005BCA02   |.  8B>|mov eax,dword ptr ss:[ebp-4]
005BCA05   |.  0F>|movzx eax,byte ptr ds:[eax+ebx-1]   ;  从最后一位得到数据
005BCA0A   |.  50 |push eax
005BCA0B   |.  B8>|mov eax,459
005BCA10   |.  5A |pop edx
005BCA11   |.  8B>|mov ecx,edx
005BCA13   |.  33>|xor edx,edx
005BCA15   |.  F7>|div ecx                             ;  用固定值459除以每一位的ASCII（从最后一位取）
005BCA17   |.  8B>|mov eax,edx                         ;  得到余数
005BCA19   |.  33>|xor edx,edx
005BCA1B   |.  29>|sub dword ptr ss:[esp],eax          ;  用固定值减去余数
005BCA1E   |.  19>|sbb dword ptr ss:[esp+4],edx        ;  进位减法，上面有没有借位
005BCA22   |.  58 |pop eax
005BCA23   |.  5A |pop edx
005BCA24   |.  89>|mov dword ptr ss:[ebp-10],eax
005BCA27   |.  89>|mov dword ptr ss:[ebp-C],edx
005BCA2A   |.  4B |dec ebx                             ;  位数
005BCA2B   |>  8B> mov eax,dword ptr ss:[ebp-4]
005BCA2E   |.  E8>|call LKYD.00404DF4
005BCA33   |.  3B>|cmp ebx,eax
005BCA35   |.  7F>|jg short LKYD.005BCA3B
005BCA37   |.  85>|test ebx,ebx
005BCA39   |.^ 7F>\jg short LKYD.005BC9F4
005BCA3B   |>  8B>mov ebx,dword ptr ds:[esi+60]
005BCA3E   |.  85>test ebx,ebx
005BCA40   |.  7F>jg short LKYD.005BCA53
005BCA42   |.  FF>push dword ptr ss:[ebp-C]            ; /Arg2
005BCA45   |.  FF>push dword ptr ss:[ebp-10]           ; |Arg1
005BCA48   |.  8B>mov edx,edi                          ; |
005BCA4A   |.  33>xor eax,eax                          ; |
005BCA4C   |.  E8>call LKYD.00409F98                   ; \LKYD.00409F98
005BCA51   |.  EB>jmp short LKYD.005BCA79
005BCA53   |>  FF>push dword ptr ss:[ebp-C]            ; /Arg2
005BCA56   |.  FF>push dword ptr ss:[ebp-10]           ; |Arg1
005BCA59   |.  8B>mov edx,edi                          ; |
005BCA5B   |.  8B>mov eax,ebx                          ; |
005BCA5D   |.  E8>call LKYD.00409F98                   ; \把上面计算出来的前面加0以凑足12位
005BCA62   |.  8B>mov eax,dword ptr ds:[edi]
```

…………………………………………………………………………………………………………………………

## 附：参考实现

```c
main()
{
unsigned long int guding=0x75bcd15,jieguo=0,zj;
int i,yushu,b;
static char zuhe[21]="000000TOPTHINK000000",jiqima[12],zhucema[13]="000000000000";
printf("shuru jiqima\n");
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
```
