
```
053A4CBF  |>  8D0449        LEA EAX,DWORD PTR DS:[ECX+ECX*2]
053A4CC2  |.  8D0483        LEA EAX,DWORD PTR DS:[EBX+EAX*4]               ; EAX = X + 0ch*ECX
053A4CC5  |.  8945 0C       MOV DWORD PTR SS:[EBP+C],EAX                   ; 匹配的结构地址
053A4CC8  |>  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]
053A4CCB  |.  5F            POP EDI
053A4CCC  |.  5E            POP ESI
053A4CCD  |.  5B            POP EBX
053A4CCE  |.  5D            POP EBP
053A4CCF  \.  C3            RETN
```

`Proc 53A2301(pStructure)`

`ECX = 53B8C48`, 很重要的一个全局变量

```
053B8C48  ;第一个结构地址
053B8C4C  ;最后一个结构地址
053B8C50  ;结构个数
```

`Structure` // 0c byte ; 这个结构很重要 `**********`

```
{
  DWORD DLLNameHash;
  DWORD APINameHash;
  DWORD APIAddress( 被 53A102C 处理过的);
}
```

```
053A2301  /$  55            PUSH EBP
053A2302  |.  8BEC          MOV EBP,ESP
053A2304  |.  51            PUSH ECX
053A2305  |.  53            PUSH EBX
053A2306  |.  8BD9          MOV EBX,ECX                                   ; 53B8C48
053A2308  |.  56            PUSH ESI
053A2309  |.  57            PUSH EDI
053A230A  |.  8B43 08       MOV EAX,DWORD PTR DS:[EBX+8]                 ; 结构个数
053A230D  |.  85C0          TEST EAX,EAX
053A230F  |.  75 30         JNZ SHORT EXERe.053A2341
053A2311  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:[53BA0B4]               ; 原来为 0, 新生成一个
053A2316  |.  6A 0C         PUSH 0C
053A2318  |.  6A 08         PUSH 8
053A231A  |.  FF35 D4A03B05 PUSH DWORD PTR DS:[53BA0D4]
053A2320  |.  F7D0          NOT EAX
053A2322  |.  FFD0          CALL EAX                                      ; HeapAlloc(0ch byte)
053A2324  |.  85C0          TEST EAX,EAX
053A2326  |.  0F84 80000000 JE EXERe.053A23AC
053A232C  |.  8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]                 ; pStructure
053A232F  |.  8BF8          MOV EDI,EAX
053A2331  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]    ; 复制结构
053A2332  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2333  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2334  |.  8903          MOV DWORD PTR DS:[EBX],EAX                   ; [53B8C48]
053A2336  |.  83C0 0C       ADD EAX,0C
053A2339  |.  FF43 08       INC DWORD PTR DS:[EBX+8]                     ; 结构个数 ++
053A233C  |.  8943 04       MOV DWORD PTR DS:[EBX+4],EAX
053A233F  |.  EB 6B         JMP SHORT EXERe.053A23AC
053A2341  |>  8D4440 03     LEA EAX,DWORD PTR DS:[EAX+EAX*2+3]           ; 原来不为 0, 扩展一个
053A2345  |.  C1E0 02       SHL EAX,2                                     ; EAX = (EAX+1)*12
053A2348  |.  50            PUSH EAX
053A2349  |.  A1 B4A03B05   MOV EAX,DWORD PTR DS:[53BA0B4]
053A234E  |.  6A 08         PUSH 8
053A2350  |.  FF35 D4A03B05 PUSH DWORD PTR DS:[53BA0D4]
053A2356  |.  F7D0          NOT EAX
053A2358  |.  FFD0          CALL EAX                                      ; HeapAlloc(新分配空间, 比原来大 0c 字节)
053A235A  |.  33C9          XOR ECX,ECX
053A235C  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
053A235F  |.  394B 08       CMP DWORD PTR DS:[EBX+8],ECX
053A2362  |.  7E 15         JLE SHORT EXERe.053A2379
053A2364  |.  33D2          XOR EDX,EDX                                   ; 把老的内容复制到新空间
053A2366  |>  8BF2          /MOV ESI,EDX
053A2368  |.  8D3C02        |LEA EDI,DWORD PTR DS:[EDX+EAX]
053A236B  |.  0333          |ADD ESI,DWORD PTR DS:[EBX]
053A236D  |.  41            |INC ECX
053A236E  |.  83C2 0C       |ADD EDX,0C
053A2371  |.  A5            |MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2372  |.  A5            |MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2373  |.  A5            |MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2374  |.  3B4B 08       |CMP ECX,DWORD PTR DS:[EBX+8]
053A2377  |.^ 7C ED         \JL SHORT EXERe.053A2366
053A2379  |>  8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]                 ; pStructure
053A237C  |.  8D0C49        LEA ECX,DWORD PTR DS:[ECX+ECX*2]
053A237F  |.  8D3C88        LEA EDI,DWORD PTR DS:[EAX+ECX*4]
053A2382  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]    ; append 到最后
053A2383  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2384  |.  A5            MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
053A2385  |.  FF33          PUSH DWORD PTR DS:[EBX]                       ; 释放老的空间
053A2387  |.  A1 90A03B05   MOV EAX,DWORD PTR DS:[53BA090]
053A238C  |.  F7D0          NOT EAX
053A238E  |.  6A 00         PUSH 0
053A2390  |.  FF35 D4A03B05 PUSH DWORD PTR DS:[53BA0D4]
053A2396  |.  FFD0          CALL EAX                                      ; HeapFree
053A2398  |.  FF43 08       INC DWORD PTR DS:[EBX+8]                      ; 结构个数 ++
053A239B  |.  8B43 08       MOV EAX,DWORD PTR DS:[EBX+8]
053A239E  |.  8B4D FC       MOV ECX,DWORD PTR SS:[EBP-4]
053A23A1  |.  8D0440        LEA EAX,DWORD PTR DS:[EAX+EAX*2]
053A23A4  |.  890B          MOV DWORD PTR DS:[EBX],ECX                    ; 新空间的开始地址
053A23A6  |.  8D0481        LEA EAX,DWORD PTR DS:[ECX+EAX*4]
053A23A9  |.  8943 04       MOV DWORD PTR DS:[EBX+4],EAX                  ; 新空间的结束地址
053A23AC  |>  5F            POP EDI
053A23AD  |.  5E            POP ESI
053A23AE  |.  5B            POP EBX
053A23AF  |.  C9            LEAVE
053A23B0  \.  C2 0400       RETN 4
```

```
053B8C18 ; 开始结构地址
053B8C1C ; 结束结构地址
053B8C20 ; hThread 结构个数
```

`structure hThread`

```
{
  Dword ThreadHandle;
}
```

```
053B8C24 ; 开始结构地址
053B8C28 ; 结束结构地址
053B8C2C ; DllNameHash 结构个数
```

`structure DllNameHash`

```
{
   Dword DllNameHash;
}
```

```
053B8C38    ; 开始结构地址
053B8C3C    ; 结束结构地址
053B8C40    ; Dll UniName Path 结构个数
```

`structure UniName`

```
{
   Dword BaseAddress;
   Dword UnicodeString; 指向 Unicode Name
}
```

```
053B8C58    ; 开始结构地址
053B8C5C    ; 结束结构地址
053B8C60    ; DLL API Address 结构个数
```

`structure Export`

```
{
   Dword BaseAddress;
   Dword pFirstThunk;  (Dll 所有的 API address, 按原始循序, 0 结束)
   Dword TlsSize
}
```

`MyLoadLibrary(pDllName)`

`ECX = 053B8C18` ; 关键全局变量

```
053A5514  /.  55            PUSH EBP
053A5515  |.  8BEC          MOV EBP,ESP
053A5517  |.  81EC 180A0000 SUB ESP,0A18
053A551D  |.  53            PUSH EBX
053A551E  |.  56            PUSH ESI
053A551F  |.  57            PUSH EDI
053A5520  |.  8B7D 08       MOV EDI,DWORD PTR SS:[EBP+8]                ; pDllName
053A5523  |.  33DB          XOR EBX,EBX
053A5525  |.  8BF1          MOV ESI,ECX                                 ; 053B8C18
053A5527  |.  3BFB          CMP EDI,EBX
053A5529  |.  8975 F0       MOV DWORD PTR SS:[EBP-10],ESI
053A552C  |.  0F84 75010000 JE EXERe.053A56A7                           ; pDllName == 0 则结束
053A5532  |.  53            PUSH EBX
053A5533  |.  57            PUSH EDI
053A5534  |.  E8 6EF1FFFF   CALL EXERe.053A46A7                         ; uppercase
053A5539  |.  59            POP ECX
053A553A  |.  50            PUSH EAX
053A553B  |.  E8 C91F0000   CALL EXERe.053A7509                         ; 对 Dll Name 做 Hash 变换
053A5540  |.  59            POP ECX
053A5541  |.  33D2          XOR EDX,EDX
053A5543  |.  59            POP ECX
053A5544  |.  8945 F4       MOV DWORD PTR SS:[EBP-C],EAX                ; 保存 Dll Name Hash
053A5547  |.  8B4E 14       MOV ECX,DWORD PTR DS:[ESI+14]               ; 已处理的 DLL 个数
053A554A  |.  3BCB          CMP ECX,EBX
```
