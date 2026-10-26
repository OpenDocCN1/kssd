
```
CODE:006A01A1                 mov     [ebp+var\_C], eax ; 保存

CODE:006A01A4                 lea     ecx, [ebp+var\_20]

CODE:006A01A7                 mov     edx, 3          ; 要取得的注册码的段索引

CODE:006A01AC                 mov     eax, [ebp+var\_8]

CODE:006A01AF                 mov     edi, [eax]

CODE:006A01B1                 call    dword ptr [edi+0Ch]

CODE:006A01B4                 mov     eax, [ebp+var\_20] ; 指向注册码最后5位

CODE:006A01B7                 or      edx, 0FFFFFFFFh

CODE:006A01BA                 call    sub\_409E40      ; 把eax指向的转化为16进制表示

CODE:006A01BF                 cmp     [ebp+var\_C], 0FFFFFFFFh ; 返回是-1?

CODE:006A01C3                 jz      loc\_6A03B8

CODE:006A01C9                 cmp     eax, 0FFFFFFFFh ; 返回是-1?

CODE:006A01CC                 jz      loc\_6A03B8

CODE:006A01D2                 mov     edi, [ebp+var\_C] ; a

CODE:006A01D5                 add     edi, eax        ; a+=b

CODE:006A01D7                 lea     edx, [ebp+var\_38]

CODE:006A01DA                 mov     eax, edi

CODE:006A01DC                 call    sub\_409C98      ; a转化为10进制

CODE:006A01E1                 mov     eax, [ebp+var\_38] ; a转化为10进制的结果记为c

CODE:006A01E4                 lea     edx, [ebp+var\_34]

CODE:006A01E7                 call    sub\_49F2B4

CODE:006A01EC                 lea     eax, [ebp+var\_34]

CODE:006A01EF                 lea     edx, [ebp+var\_24]

CODE:006A01F2                 call    sub\_49F240

CODE:006A01F7                 mov     eax, [ebp+var\_24]

CODE:006A01FA                 mov     edx, offset a1c395a8dce13\_0 ; "1c395a8dce135849bd73c6dba3b54809"

CODE:006A01FF                 call    sub\_4095D4      ; MD5(c)与内置密文比较

CODE:006A0204                 test    eax, eax

CODE:006A0206                 jnz     loc\_6A03B8      ; 不等就注册失败

CODE:006A020C                 lea     ecx, [ebp+var\_3C]

CODE:006A020F                 mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A0214                 mov     eax, [ebp+var\_8]

CODE:006A0217                 mov     edi, [eax]

CODE:006A0219                 call    dword ptr [edi+0Ch]

CODE:006A021C                 mov     eax, [ebp+var\_3C] ; 指向注册码第2段

CODE:006A021F                 or      edx, 0FFFFFFFFh

CODE:006A0222                 call    sub\_409E40      ; 把eax指向的转化为16进制表示记为d

CODE:006A0227                 mov     [ebp+var\_C], eax ; 保存返回值

CODE:006A022A                 lea     ecx, [ebp+var\_40]

CODE:006A022D                 mov     edx, 2          ; 要取得的注册码的段索引

CODE:006A0232                 mov     eax, [ebp+var\_8]

CODE:006A0235                 mov     edi, [eax]

CODE:006A0237                 call    dword ptr [edi+0Ch]

CODE:006A023A                 mov     eax, [ebp+var\_40] ; 指向注册码的第三段

CODE:006A023D                 or      edx, 0FFFFFFFFh

CODE:006A0240                 call    sub\_409E40      ; 把eax指向的转化为16进制表示记为e

CODE:006A0245                 cmp     [ebp+var\_C], 0FFFFFFFFh

CODE:006A0249                 jz      loc\_6A03B8

CODE:006A024F                 inc     eax

CODE:006A0250                 jz      loc\_6A03B8

CODE:006A0256                 lea     eax, [ebp+var\_44]

CODE:006A0259                 push    eax

CODE:006A025A                 lea     ecx, [ebp+var\_48]

CODE:006A025D                 mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A0262                 mov     eax, [ebp+var\_8]

CODE:006A0265                 mov     edi, [eax]

CODE:006A0267                 call    dword ptr [edi+0Ch]

CODE:006A026A                 mov     eax, [ebp+var\_48] ; 指向注册码的第2段

CODE:006A026D                 mov     ecx, 7FFFFFFFh

CODE:006A0272                 mov     edx, 3          ; 在第2段取得后三位

CODE:006A0277                 call    sub\_4050FC

CODE:006A027C                 mov     eax, [ebp+var\_44] ; 指向第2段后3位

CODE:006A027F                 xor     edx, edx

CODE:006A0281                 call    sub\_409E40      ; 转化为16进制表示

CODE:006A0286                 mov     [ebp+var\_C], eax ; 保存

CODE:006A0289                 lea     eax, [ebp+var\_4C]

CODE:006A028C                 push    eax

CODE:006A028D                 lea     ecx, [ebp+var\_50]

CODE:006A0290                 mov     edx, 2          ; 要取得的注册码的段索引

CODE:006A0295                 mov     eax, [ebp+var\_8]

CODE:006A0298                 mov     edi, [eax]

CODE:006A029A                 call    dword ptr [edi+0Ch]

CODE:006A029D                 mov     eax, [ebp+var\_50] ; 指向注册玛第3段

CODE:006A02A0                 mov     ecx, 7FFFFFFFh

CODE:006A02A5                 mov     edx, 3          ; 指向第3段后3位

CODE:006A02AA                 call    sub\_4050FC

CODE:006A02AF                 mov     eax, [ebp+var\_4C] ; 指向第3段后3位

CODE:006A02B2                 xor     edx, edx

CODE:006A02B4                 call    sub\_409E40      ; 转化为16进制表示

CODE:006A02B9                 mov     edi, [ebp+var\_C] ; 第2段注册码后3位的16进制

CODE:006A02BC                 add     edi, eax        ; 加上第3段后3位16进制表示

CODE:006A02BE                 lea     edx, [ebp+var\_58]

CODE:006A02C1                 mov     eax, edi

CODE:006A02C3                 call    sub\_409C98      ; 转化为10进制表示记为a1

CODE:006A02C8                 mov     eax, [ebp+var\_58]

CODE:006A02CB                 lea     edx, [ebp+var\_34]

CODE:006A02CE                 call    sub\_49F2B4      ; MD5(a1)

CODE:006A02D3                 lea     eax, [ebp+var\_34]

CODE:006A02D6                 lea     edx, [ebp+var\_54]

CODE:006A02D9                 call    sub\_49F240      ; MD5(a1)结果字符化

CODE:006A02DE                 mov     eax, [ebp+var\_54]

CODE:006A02E1                 mov     edx, offset a65cc2c8205a0\_0 ; "65cc2c8205a05d7379fa3a6386f710e1"

CODE:006A02E6                 call    sub\_4095D4      ; MD5(a1)与内置密文比较

CODE:006A02EB                 test    eax, eax

CODE:006A02ED                 jnz     loc\_6A03B8      ; 不等就注册失败

CODE:006A02F3                 lea     eax, [ebp+var\_5C]

CODE:006A02F6                 push    eax

CODE:006A02F7                 lea     ecx, [ebp+var\_60]

CODE:006A02FA                 mov     edx, 1          ; 要取得的注册码的段索引

CODE:006A02FF                 mov     eax, [ebp+var\_8]

CODE:006A0302                 mov     edi, [eax]

CODE:006A0304                 call    dword ptr [edi+0Ch]

CODE:006A0307                 mov     eax, [ebp+var\_60] ; 指向注册码第2段

CODE:006A030A                 mov     ecx, 2          ; 取第2段的前2位

CODE:006A030F                 xor     edx, edx

CODE:006A0311                 call    sub\_4050FC

CODE:006A0316                 mov     eax, [ebp+var\_5C] ; 指向第2段的前2位

CODE:006A0319                 xor     edx, edx

CODE:006A031B                 call    sub\_409E40      ; 转化为16进制表示

CODE:006A0320                 mov     [ebp+var\_C], eax ; 保存

CODE:006A0323                 lea     eax, [ebp+var\_64]

CODE:006A0326                 push    eax

CODE:006A0327                 lea     ecx, [ebp+var\_68]

CODE:006A032A                 mov     edx, 2          ; 要取得的注册码的段索引

CODE:006A032F                 mov     eax, [ebp+var\_8]

CODE:006A0332                 mov     edi, [eax]

CODE:006A0334                 call    dword ptr [edi+0Ch]

CODE:006A0337                 mov     eax, [ebp+var\_68] ; 指向注册码第3段

CODE:006A033A                 mov     ecx, 2          ; 指向第3段的前2位

CODE:006A033F                 xor     edx, edx

CODE:006A0341                 call    sub\_4050FC

CODE:006A0346                 mov     eax, [ebp+var\_64] ; 指向第3段的前2位

CODE:006A0349                 xor     edx, edx

CODE:006A034B                 call    sub\_409E40      ; 转化为16进制表示

CODE:006A0350                 mov     edi, [ebp+var\_C] ; 转化后的result1

CODE:006A0353                 add     edi, eax        ; result1+=result2

CODE:006A0355                 lea     edx, [ebp+var\_70]

CODE:006A0358                 mov     eax, edi

CODE:006A035A                 call    sub\_409C98      ; 将result1转化为10进表示

CODE:006A035F                 mov     eax, [ebp+var\_70]

CODE:006A0362                 lea     edx, [ebp+var\_34]
```
