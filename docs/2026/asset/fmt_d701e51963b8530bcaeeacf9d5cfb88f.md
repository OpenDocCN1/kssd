
```
.text:005CD347 l_cmp:                                  ; CODE XREF: sub_5CD150+11D

.text:005CD347                 mov     ecx, [ebp+arg_pPtInputedLicNum]

.text:005CD34A                 mov     edx, [ecx]

.text:005CD34C                 push    edx

.text:005CD34D                 mov     eax, [ebp+var_pCurrLicNum]

.text:005CD350                 push    eax

.text:005CD351                 call    ds:__vbaStrCmp

.text:005CD357                 test    eax, eax

.text:005CD359                 jz      short loc_5CD372

.text:005CD35B                 mov     eax, 1

.text:005CD360                 add     ax, word ptr [ebp+var_nCount]

.text:005CD364                 jo      l_exception

.text:005CD36A                 mov     [ebp+var_nCount], eax

.text:005CD36D                 jmp     loc_5CD243
```

没有太大的意思。见MSDN对`Randomize`的解释：若想得到重复的随机数序列，在使用具有数值参数的`Randomize`之前直接调用具有负参数值的`Rnd`。这里先用`-1`去调`Rnd`，得到固定的"随机数"序列，用来在`'a'-'z'`间取值。可以照葫芦画瓢，做出300个`license number`，一网打尽:-)。

```
Rnd (-1)

Randomize (25)

For i = 1 To 300   '300个license number

    CurrLicNum = ""

    For j = 1 To 16    '每个16字符

        CurrLicNum = CurrLicNum + Chr(&H61 + Int(26 * Rnd))

    Next j

Next i
```

这里只是明文，还要逆回去。难道厂商只打算卖300份?或许这300个`license number`用完了，就要用`keyfile`了:-)
