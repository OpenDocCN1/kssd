
```
0056A66C  |.  50            push    eax                         ; |hOwner
0056A66D  |.  E8 CADCE9FF   call    <jmp.&user32.MessageBoxA> ; \\MessageBoxA
0056A672  |.  E9 01030000   jmp     0056A978
0056A677  |>  8D55 CC       lea     edx, dword ptr [ebp-34]
0056A67A  |.  8B83 B4030000 mov     eax, dword ptr [ebx+3B4]
0056A680  |.  E8 97C6EEFF   call    00456D1C
0056A685  |.  8B45 CC       mov     eax, dword ptr [ebp-34]
0056A688  |.  8945 DC       mov     dword ptr [ebp-24], eax
0056A68B  |.  8B45 DC       mov     eax, dword ptr [ebp-24]
0056A68E  |.  85C0          test    eax, eax
0056A690  |.  74 05         je      short 0056A697
0056A692  |.  83E8 04       sub     eax, 4
    0056A695  |.  8B00          mov     eax, dword ptr [eax]      ;  取第 5 组注册码的位数
    0056A697  |>  83F8 04       cmp     eax, 4                    ;  比较位数等于 4 吗？
    0056A69A  |.  74 2E         je      short 0056A6CA            ;  是，跳到进行下一步的验证
0056A69C  |.  6A 40         push    40
0056A69E  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A6A3  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A6A9  |.  E8 AEAFE9FF   call    0040565C
0056A6AE  |.  50            push    eax
0056A6AF  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A6B2  |.  E8 A5AFE9FF   call    0040565C
0056A6B7  |.  50            push    eax
0056A6B8  |.  8BC3          mov     eax, ebx
0056A6BA  |.  E8 5D41EFFF   call    0045E81C
0056A6BF  |.  50            push    eax                         ; |hOwner
0056A6C0  |.  E8 77DCE9FF   call    <jmp.&user32.MessageBoxA> ; \\MessageBoxA
0056A6C5  |.  E9 AE020000   jmp     0056A978
0056A6CA  |>  8D55 C8       lea     edx, dword ptr [ebp-38]
0056A6CD  |.  8B83 A4030000 mov     eax, dword ptr [ebx+3A4]
    0056A6D3  |.  E8 44C6EEFF   call    00456D1C                  ;  取第一组注册码
0056A6D8  |.  8B45 C8       mov     eax, dword ptr [ebp-38]
    0056A6DB  |.  E8 D4F7E9FF   call    00409EB4                  ;  将第一组注册码转成对应 16 进制,设为 A
    0056A6E0  |.  8BF0          mov     esi, eax                  ;  ESI 保存转换的结果
0056A6E2  |.  8D55 C4       lea     edx, dword ptr [ebp-3C]
0056A6E5  |.  8B83 A8030000 mov     eax, dword ptr [ebx+3A8]
    0056A6EB  |.  E8 2CC6EEFF   call    00456D1C                  ;  取第 2 组注册码
0056A6F0  |.  8B45 C4       mov     eax, dword ptr [ebp-3C]
    0056A6F3  |.  E8 BCF7E9FF   call    00409EB4                  ;  将第 2 组注册码转成对应 16 进制，设为 B
    0056A6F8  |.  8BF8          mov     edi, eax                  ;  EDI 保存转换的结果
0056A6FA  |.  8D55 C0       lea     edx, dword ptr [ebp-40]
0056A6FD  |.  8B83 AC030000 mov     eax, dword ptr [ebx+3AC]
    0056A703  |.  E8 14C6EEFF   call    00456D1C                  ;  取第 3 组注册码
0056A708  |.  8B45 C0       mov     eax, dword ptr [ebp-40]
    0056A70B  |.  E8 A4F7E9FF   call    00409EB4                  ;  将第 3 组注册码转成对应 16 进制，设为 C
     0056A710  |.  8945 F8       mov     dword ptr [ebp-8], eax
    0056A713  |.  0FAFF7        imul    esi, edi                  ;  ESI 乘以 EDI,A*B
    0056A716  |.  81EE 2B060000 sub     esi, 62B                  ;  ESI*EDI 的结果减 0x62B(A*B-62B)
    0056A71C  |.  81FE 10270000 cmp     esi, 2710                 ;  计算的结果与 0x2710 比较
    0056A722      7D 2E         jge     short 0056A752            ;  大于 0x2710 继续计算，小于错误提示
0056A724  |.  6A 40         push    40
0056A726  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A72B  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A731  |.  E8 26AFE9FF   call    0040565C
0056A736  |.  50            push    eax
0056A737  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A73A  |.  E8 1DAFE9FF   call    0040565C
0056A73F  |.  50            push    eax
0056A740  |.  8BC3          mov     eax, ebx
0056A742  |.  E8 D540EFFF   call    0045E81C
0056A747  |.  50            push    eax                         ; |hOwner
0056A748  |.  E8 EFDBE9FF   call    <jmp.&user32.MessageBoxA> ; \\MessageBoxA
0056A74D  |.  E9 26020000   jmp     0056A978
0056A752  |>  8D55 B8       lea     edx, dword ptr [ebp-48]
0056A755  |.  8BC6          mov     eax, esi
    0056A757  |.  E8 1CF6E9FF   call    00409D78                  ;  上面 A*B-62B 结果转换 10 进数,设为 D
0056A75C  |.  8B45 B8       mov     eax, dword ptr [ebp-48]
0056A75F  |.  8D4D BC       lea     ecx, dword ptr [ebp-44]
0056A762  |.  BA 04000000   mov     edx, 4
    0056A767  |.  E8 24DDEDFF   call    00448490                  ;  取 D 的后 4 位
    0056A76C  |.  8B45 BC       mov     eax, dword ptr [ebp-44]   ;  保存下来
0056A76F  |.  50            push    eax
0056A770  |.  8D55 B4       lea     edx, dword ptr [ebp-4C]
0056A773  |.  8B83 B0030000 mov     eax, dword ptr [ebx+3B0]
    0056A779  |.  E8 9EC5EEFF   call    00456D1C                  ;  取第 4 组注册码
0056A77E  |.  8B55 B4       mov     edx, dword ptr [ebp-4C]
0056A781  |.  58            pop     eax
    0056A782  |.  E8 21AEE9FF   call    004055A8                  ;  D 的后 4 位与输入的第 4 组注册码相比较
    0056A787      74 2E         je      short 0056A7B7            ;  相等继续计算，否就到错误提示
0056A789  |.  6A 40         push    40
0056A78B  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
0056A790  |.  8B80 6C080000 mov     eax, dword ptr [eax+86C]
0056A796  |.  E8 C1AEE9FF   call    0040565C
0056A79B  |.  50            push    eax
0056A79C  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056A79F  |.  E8 B8AEE9FF   call    0040565C
0056A7A4  |.  50            push    eax
0056A7A5  |.  8BC3          mov     eax, ebx
0056A7A7  |.  E8 7040EFFF   call    0045E81C
0056A7AC  |.  50            push    eax                         ; |hOwner
0056A7AD  |.  E8 8ADBE9FF   call    <jmp.&user32.MessageBoxA> ; \\MessageBoxA
0056A7B2  |.  E9 C1010000   jmp     0056A978
0056A7B7  |>  8B75 F8       mov     esi, dword ptr [ebp-8]
0056A7BA  |.  81C6 01020000 add     esi, 201                  ;  C 加上 0x201
0056A7C0  |.  0FAFF7        imul    esi, edi                  ;  结果乘上 B
    0056A7C3  |.  81EE F50D0000 sub     esi, 0DF5                ;  得到的积减去 0x0DF5,设为 E
0056A7C9  |.  8D55 AC       lea     edx, dword ptr [ebp-54]
0056A7CC  |.  8BC6          mov     eax, esi
    0056A7CE  |.  E8 A5F5E9FF   call    00409D78                  ;  将 E 转成对应的 10 进制数
0056A7D3  |.  8B45 AC       mov     eax, dword ptr [ebp-54]
0056A7D6  |.  8D4D B0       lea     ecx, dword ptr [ebp-50]
0056A7D9  |.  BA 04000000   mov     edx, 4
    0056A7DE  |.  E8 ADDCEDFF   call    00448490                  ;  取结果 E 十进数后 4 位
    0056A7E3  |.  8B45 B0       mov     eax, dword ptr [ebp-50]   ;  保存到 EAX 中
0056A7E6  |.  50            push    eax
0056A7E7  |.  8D55 A8       lea     edx, dword ptr [ebp-58]
0056A7EA  |.  8B83 B4030000 mov     eax, dword ptr [ebx+3B4]
    0056A7F0  |.  E8 27C5EEFF   call    00456D1C                  ;  取第 5 组注册码
0056A7F5  |.  8B55 A8       mov     edx, dword ptr [ebp-58]
0056A7F8  |.  58            pop     eax
    0056A7F9  |.  E8 AAADE9FF   call    004055A8                  ;  E 十进数后 4 位与第 5 组注册码比较
    0056A7FE  |.  74 2E         je      short 0056A82E            ;  相等就注册成功
0056A800  |.  6A 40         push    40
0056A802  |.  A1 84055A00   mov     eax, dword ptr [5A0584]
```

### 总结
`第一组转成 16 进数 A` 乘上 `第二组转成 16 进数 B` 减去 `0x62B`; (`A * B - 0x62B`) > `0x2710`
