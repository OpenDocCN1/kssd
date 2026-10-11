
> `0055764A  .  8846 0D     mov     byte ptr [esi+D], al`
>
> `/////////////////////`
>
> `0055764D  .  F645 01 10  test    byte ptr [ebp+1], 10`
>
> `00557651  .  0F97C0      seta    al`
>
> `00557654  .  8847 0E     mov     byte ptr [edi+E], al`
>
>
> `00557657  .  F645 01 20  test    byte ptr [ebp+1], 20`
>
> `0055765B  .  0F97C0      seta    al`
>
> `0055765E  .  8846 0E     mov     byte ptr [esi+E], al`
>
> `/////////////////////`
>
> `00557661  .  F645 00 10  test    byte ptr [ebp], 10`
>
> `00557665  .  0F97C0      seta    al`
>
> `00557668  .  8847 0F     mov     byte ptr [edi+F], al`
>
>
> `0055766B  .  F645 00 20  test    byte ptr [ebp], 20`
>
> `0055766F  .  0F97C0      seta    al`
>
> `00557672  .  8846 0F     mov     byte ptr [esi+F], al`
>
>
> 上面的代码就是把输入的用户名的前 8 个字符的 ASCII 依次和`$10`、`$20`比较 改变两个序列 `si`、`di` 的值
>
> `////////////////////////>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>`
>
> `00557675  .  F645 07 04  test    byte ptr [ebp+7], 4`
>
> `00557679  .  0F97C0      seta    al`
>
> `0055767C  .  8847 10     mov     byte ptr [edi+10], al`
>
>
> `0055767F  .  F645 07 08  test    byte ptr [ebp+7], 8`
>
> `00557683  .  0F97C0      seta    al`
>
> `00557686  .  8846 10     mov     byte ptr [esi+10], al`
>
> `//////////////////////`
>
> `00557689  .  F645 06 04  test    byte ptr [ebp+6], 4`
>
> `0055768D  .  0F97C0      seta    al`
>
> `00557690  .  8847 11     mov     byte ptr [edi+11], al`
>
>
> `00557693  .  F645 06 08  test    byte ptr [ebp+6], 8`
>
> `00557697  .  0F97C0      seta    al`
>
> `0055769A  .  8846 11     mov     byte ptr [esi+11], al`
>
> `///////////////////`
>
> `0055769D  .  F645 05 04  test    byte ptr [ebp+5], 4`
>
> `005576A1  .  0F97C0      seta    al`
>
> `005576A4  .  8847 12     mov     byte ptr [edi+12], al`
>
>
> `005576A7  .  F645 05 08  test    byte ptr [ebp+5], 8`
>
> `005576AB  .  0F97C0      seta    al`
>
> `005576AE  .  8846 12     mov     byte ptr [esi+12], al`
>
> `///////////////////////////`
>
> `005576B1  .  F645 04 04  test    byte ptr [ebp+4], 4`
>
> `005576B5  .  0F97C0      seta    al`
>
> `005576B8  .  8847 13     mov     byte ptr [edi+13], al`
>
>
> `005576BB  .  F645 04 08  test    byte ptr [ebp+4], 8`
>
> `005576BF  .  0F97C0      seta    al`
>
> `005576C2  .  8846 13     mov     byte ptr [esi+13], al`
>
> `////////////////////////////`
>
> `005576C5  .  F645 03 04  test    byte ptr [ebp+3], 4`
>
> `005576C9  .  0F97C0      seta    al`
>
> `005576CC  .  8847 14     mov     byte ptr [edi+14], al`
>
>
> `005576CF  .  F645 03 08  test    byte ptr [ebp+3], 8`
>
> `005576D3  .  0F97C0      seta    al`
>
> `005576D6  .  8846 14     mov     byte ptr [esi+14], al`
>
> `//////////////////////`
>
> `005576D9  .  F645 02 04  test    byte ptr [ebp+2], 4`
>
> `005576DD  .  0F97C0      seta    al`
>
> `005576E0  .  8847 15     mov     byte ptr [edi+15], al`
>
>
> `005576E3  .  F645 02 08  test    byte ptr [ebp+2], 8`
>
> `005576E7  .  0F97C0      seta    al`
>
> `005576EA  .  8846 15     mov     byte ptr [esi+15], al`
>
> `//////////////////////`
>
> `005576ED  .  F645 01 04  test    byte ptr [ebp+1], 4`
>
> `005576F1  .  0F97C0      seta    al`
>
> `005576F4  .  8847 16     mov     byte ptr [edi+16], al`
>
>
> `005576F7  .  F645 01 08  test    byte ptr [ebp+1], 8`
>
> `005576FB  .  0F97C0      seta    al`
>
> `005576FE  .  8846 16     mov     byte ptr [esi+16], al`
>
> `////////////////////`
>
> `00557701  .  F645 00 04  test    byte ptr [ebp], 4`
>
> `00557705  .  0F97C0      seta    al`
>
> `00557708  .  8847 17     mov     byte ptr [edi+17], al`
>
>
> `0055770B  .  F645 00 08  test    byte ptr [ebp], 8`
>
> `0055770F  .  0F97C0      seta    al`
>
> `00557712  .  8846 17     mov     byte ptr [esi+17], al`
>
> `/////////////////////////////`
>
>
> 上面的代码就是把输入的用户名的前 8 个字符的 ASCII 依次和`$4`、`$8`比较 改变两个序列 `si`、`di` 的值
>
> `00557715  .  8A45 07     mov     al, byte ptr [ebp+7]           \`%%%%%%%%%%%%%%%%
>
> `00557718  .  24 01       and     al, 1`
>
> `0055771A  .  8847 18     mov     byte ptr [edi+18], al`
>
>
> `0055771D  .  F645 07 02  test    byte ptr [ebp+7], 2`
>
> `00557721  .  0F97C0      seta    al`
>
> `00557724  .  8846 18     mov     byte ptr [esi+18], al`
>
> `///////////////////`
>
> `00557727  .  8A45 06     mov     al, byte ptr [ebp+6]`
>
> `0055772A  .  24 01       and     al, 1`
>
> `0055772C  .  8847 19     mov     byte ptr [edi+19], al`
>
>
> `0055772F  .  F645 06 02  test    byte ptr [ebp+6], 2`
>
> `00557733  .  0F97C0      seta    al`
>
> `00557736  .  8846 19     mov     byte ptr [esi+19], al`
>
> `/////////////////////`
>
> `00557739  .  8A45 05     mov     al, byte ptr [ebp+5]`
>
> `0055773C  .  24 01       and     al, 1`
>
> `0055773E  .  8847 1A     mov     byte ptr [edi+1A], al`
>
>
> `00557741  .  F645 05 02  test    byte ptr [ebp+5], 2`
>
> `00557745  .  0F97C0      seta    al`
>
> `00557748  .  8846 1A     mov     byte ptr [esi+1A], al`
>
> `/////////////////////`
>
> `0055774B  .  8A45 04     mov     al, byte ptr [ebp+4]`
>
> `0055774E  .  24 01       and     al, 1`
>
> `00557750  .  8847 1B     mov     byte ptr [edi+1B], al`
>
>
> `00557753  .  F645 04 02  test    byte ptr [ebp+4], 2`
>
> `00557757  .  0F97C0      seta    al`
>
> `0055775A  .  8846 1B     mov     byte ptr [esi+1B], al`
>
> `//////////////////////`
>
> `0055775D  .  8A45 03     mov     al, byte ptr [ebp+3]`
>
> `00557760  .  24 01       and     al, 1`
>
> `00557762  .  8847 1C     mov     byte ptr [edi+1C], al`
>
>
> `00557765  .  F645 03 02  test    byte ptr [ebp+3], 2`
>
> `00557769  .  0F97C0      seta    al`
>
> `0055776C  .  8846 1C     mov     byte ptr [esi+1C], al`
>
> `////////////////////////////////`
>
> `0055776F  .  8A45 02     mov     al, byte ptr [ebp+2]`
>
> `00557772  .  24 01       and     al, 1`
>
> `00557774  .  8847 1D     mov     byte ptr [edi+1D], al`
>
>
> `00557777  .  F645 02 02  test    byte ptr [ebp+2], 2`
>
> `0055777B  .  0F97C0      seta    al`
>
> `0055777E  .  8846 1D     mov     byte ptr [esi+1D], al`
>
> `///////////////////////////`
>
> `00557781  .  8A45 01     mov     al, byte ptr [ebp+1]`
>
> `00557784  .  24 01       and     al, 1`
>
> `00557786  .  8847 1E     mov     byte ptr [edi+1E], al`
>
>
> `00557789  .  F645 01 02  test    byte ptr [ebp+1], 2`
>
> `0055778D  .  0F97C0      seta    al`
>
> `00557790  .  8846 1E     mov     byte ptr [esi+1E], al`
>
> `////////////////////////`
>
> `00557793  .  8A45 00     mov     al, byte ptr [ebp]`
>
> `00557796  .  24 01       and     al, 1`
>
> `00557798  .  8847 1F     mov     byte ptr [edi+1F], al`
>
>
> `0055779B  .  F645 00 02  test    byte ptr [ebp], 2`
>
> `0055779F  .  0F97C0      seta    al`
>
> `005577A2  .  8846 1F     mov     byte ptr [esi+1F], al`
>
>
> 上面的代码就是把输入的用户名的前 8 个字符的 ASCII 依次和`$4`、`$8`比较 改变两个序列 `si`、`di` 的值
>
> `///////////////////////`
>
> ```
> 005577A5  .  B2 10         mov     dl, 10 //循环 16 次
> 005577A7  .  B8 1C9E5800   mov     eax, 00589E1C         //另一个己预先生成的加密序列称为 b[]
> 005577AC  |>  8A4E 1F     /mov     cl, byte ptr [esi+1F]
> 005577AF  |.  3208        |xor     cl, byte ptr [eax]
> 005577B1  |.  81E1 FF000000 |and    ecx, 0FF
> 005577B7  |.  C1E1 05     |shl     ecx, 5
>
> 005577BA  |.  8A5E 04     |mov     bl, byte ptr [esi+4]
> 005577BD  |.  3258 05     |xor     bl, byte ptr [eax+5]
> 005577C0  |.  81E3 FF000000 |and    ebx, 0FF
> 005577C6  |.  C1E3 04     |shl     ebx, 4
> 005577C9  |.  66:0BCB     |or      cx, bx
> ```
