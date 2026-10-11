
```
CODE:006A017B             mov     edx, [eax]

CODE:006A017D             call    dword ptr [edx+14h]

CODE:006A0180             cmp     eax, 4

CODE:006A0183             jnz     loc_6A03B8

CODE:006A0189             lea     ecx, [ebp+var_1C]

CODE:006A018C             xor     edx, edx        ; 要取得的注册码的段索引

CODE:006A018E             mov     eax, [ebp+var_8]

CODE:006A0191             mov     edi, [eax]

CODE:006A0193             call    dword ptr [edi+0Ch]

CODE:006A0196             mov     eax, [ebp+var_1C] ; eax 指向注册码前 5 位

CODE:006A0199             or      edx, 0FFFFFFFFh

CODE:006A019C             call    sub_409E40      ; 把 eax 指向的转化为 16 进制表示

CODE:006A01A1             mov     [ebp+var_C], eax ; 保存

CODE:006A01A4             lea     ecx, [ebp+var_20]

CODE:006A01A7             mov     edx, 3          ; 要取得的注册码的段索引

CODE:006A01AC             mov     eax, [ebp+var_8]

CODE:006A01AF             mov     edi, [eax]

CODE:006A01B1             call    dword ptr [edi+0Ch]

CODE:006A01B4             mov     eax, [ebp+var_20] ; 指向注册码最后 5 位

CODE:006A01B7             or      edx, 0FFFFFFFFh

CODE:006A01BA             call    sub_409E40      ; 把 eax 指向的转化为 16 进制表示

CODE:006A01BF             cmp     [ebp+var_C], 0FFFFFFFFh ; 返回是-1?

CODE:006A01C3             jz      loc_6A03B8

CODE:006A01C9             cmp     eax, 0FFFFFFFFh ; 返回是-1?

CODE:006A01CC             jz      loc_6A03B8

CODE:006A01D2             mov     edi, [ebp+var_C] ; a

CODE:006A01D5             add     edi, eax        ; a+=b

CODE:006A01D7             lea     edx, [ebp+var_38]

CODE:006A01DA             mov     eax, edi

CODE:006A01DC             call    sub_409C98      ; a 转化为 10 进制

CODE:006A01E1             mov     eax, [ebp+var_38] ; a 转化为 10 进制的结果记为 c

CODE:006A01E4             lea     edx, [ebp+var_34]

CODE:006A01E7             call    sub_49F2B4

CODE:006A01EC             lea     eax, [ebp+var_34]

CODE:006A01EF             lea     edx, [ebp+var_24]

CODE:006A01F2             call    sub_49F240

CODE:006A01F7             mov     eax, [ebp+var_24]

CODE:006A01FA             mov     edx, offset a1c395a8dce13_0 ; "1c395a8dce135849bd73c6dba3b54809"

CODE:006A01FF             call    sub_4095D4      ; MD5(c)与内置密文比较

CODE:006A0204             test    eax, eax

CODE:006A0206             jnz     loc_6A03B8      ; 不等就注册失败

CODE:006A020C             lea     ecx, [ebp+var_3C]

CODE:006A020F             mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A0214             mov     eax, [ebp+var_8]

CODE:006A0217             mov     edi, [eax]

CODE:006A0219             call    dword ptr [edi+0Ch]

CODE:006A021C             mov     eax, [ebp+var_3C] ; 指向注册码第 2 段

CODE:006A021F             or      edx, 0FFFFFFFFh

CODE:006A0222             call    sub_409E40      ; 把 eax 指向的转化为 16 进制表示记为 d

CODE:006A0227             mov     [ebp+var_C], eax ; 保存返回值

CODE:006A022A             lea     ecx, [ebp+var_40]

CODE:006A022D             mov     edx, 2          ; 要取得的注册码的段索引

CODE:006A0232             mov     eax, [ebp+var_8]

CODE:006A0235             mov     edi, [eax]

CODE:006A0237             call    dword ptr [edi+0Ch]

CODE:006A023A             mov     eax, [ebp+var_40] ; 指向注册码的第三段

CODE:006A023D             or      edx, 0FFFFFFFFh

CODE:006A0240             call    sub_409E40      ; 把 eax 指向的转化为 16 进制表示记为 e

CODE:006A0245             cmp     [ebp+var_C], 0FFFFFFFFh

CODE:006A0249             jz      loc_6A03B8

CODE:006A024F             inc     eax

CODE:006A0250             jz      loc_6A03B8

CODE:006A0256             lea     eax, [ebp+var_44]

CODE:006A0259             push    eax

CODE:006A025A             lea     ecx, [ebp+var_48]

CODE:006A025D             mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A0262             mov     eax, [ebp+var_8]

CODE:006A0265             mov     edi, [eax]

CODE:006A0267             call    dword ptr [edi+0Ch]

CODE:006A026A             mov     eax, [ebp+var_48] ; 指向注册码的第 2 段

CODE:006A026D             mov     ecx, 7FFFFFFFh

CODE:006A0272             mov     edx, 3          ; 在第 2 段取得后三位

CODE:006A0277             call    sub_4050FC

CODE:006A027C             mov     eax, [ebp+var_44] ; 指向第 2 段后 3 位

CODE:006A027F             xor     edx, edx

CODE:006A0281             call    sub_409E40      ; 转化为 16 进制表示

CODE:006A0286             mov     [ebp+var_C], eax ; 保存

CODE:006A0289             lea     eax, [ebp+var_4C]

CODE:006A028C             push    eax

CODE:006A028D             lea     ecx, [ebp+var_50]

CODE:006A0290             mov     edx, 2          ; 要取得的注册码的段索引

CODE:006A0295             mov     eax, [ebp+var_8]

CODE:006A0298             mov     edi, [eax]

CODE:006A029A             call    dword ptr [edi+0Ch]

CODE:006A029D             mov     eax, [ebp+var_50] ; 指向注册玛第 3 段

CODE:006A02A0             mov     ecx, 7FFFFFFFh

CODE:006A02A5             mov     edx, 3          ; 指向第 3 段后 3 位

CODE:006A02AA             call    sub_4050FC

CODE:006A02AF             mov     eax, [ebp+var_4C] ; 指向第 3 段后 3 位

CODE:006A02B2             xor     edx, edx

CODE:006A02B4             call    sub_409E40      ; 转化为 16 进制表示

CODE:006A02B9             mov     edi, [ebp+var_C] ; 第 2 段注册码后 3 位的 16 进制

CODE:006A02BC             add     edi, eax        ; 加上第 3 段后 3 位 16 进制表示

CODE:006A02BE             lea     edx, [ebp+var_58]

CODE:006A02C1             mov     eax, edi

CODE:006A02C3             call    sub_409C98      ; 转化为 10 进制表示记为 a1

CODE:006A02C8             mov     eax, [ebp+var_58]

CODE:006A02CB             lea     edx, [ebp+var_34]

CODE:006A02CE             call    sub_49F2B4      ; MD5(a1)

CODE:006A02D3             lea     eax, [ebp+var_34]

CODE:006A02D6             lea     edx, [ebp+var_54]

CODE:006A02D9             call    sub_49F240      ; MD5(a1)结果字符化

CODE:006A02DE             mov     eax, [ebp+var_54]

CODE:006A02E1             mov     edx, offset a65cc2c8205a0_0 ; "65cc2c8205a05d7379fa3a6386f710e1"

CODE:006A02E6             call    sub_4095D4      ; MD5(a1)与内置密文比较

CODE:006A02EB             test    eax, eax

CODE:006A02ED             jnz     loc_6A03B8      ; 不等就注册失败

CODE:006A02F3             lea     eax, [ebp+var_5C]

CODE:006A02F6             push    eax

CODE:006A02F7             lea     ecx, [ebp+var_60]

CODE:006A02FA             mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A02FF             mov     eax, [ebp+var_8]

CODE:006A0302             mov     edi, [eax]

CODE:006A0304             call    dword ptr [edi+0Ch]

CODE:006A0307             mov     eax, [ebp+var_60] ; 指向注册码第 2 段

CODE:006A030A             mov     ecx, 2          ; 取第 2 段的前 2 位

CODE:006A030F             xor     edx, edx

CODE:006A0311             call    sub_4050FC

CODE:006A0316             mov     eax, [ebp+var_5C] ; 指向第 2 段的前 2 位

CODE:006A0319             xor     edx, edx

CODE:006A031B             call    sub_409E40      ; 转化为 16 进制表示

CODE:006A0320             mov     [ebp+var_C], eax ; 保存

CODE:006A0323             lea     eax, [ebp+var_64]

CODE:006A0326             push    eax

CODE:006A0327             lea     ecx, [ebp+var_68]
```
