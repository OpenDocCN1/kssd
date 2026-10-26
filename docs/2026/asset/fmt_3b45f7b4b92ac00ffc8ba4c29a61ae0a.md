
```
0044B870  |.  E8 B6F90A00   call    004FB22B

0044B875  |.  8945 94       mov     [ebp-6C], eax                    ; |

0044B878  |.  8D4D E4       lea     ecx, [ebp-1C]                    ; |

0044B87B  |.  51            push    ecx                              ; |Arg1

0044B87C  |.  8B4D F0       mov     ecx, [ebp-10]                    ; |

0044B87F  |.  E8 5C080000   call    0044C0E0                         ; \算法call（2），跟进

0044B884  |.  8945 90       mov     [ebp-70], eax

0044B887  |.  8B55 90       mov     edx, [ebp-70]

0044B88A  |.  8955 8C       mov     [ebp-74], edx

0044B88D  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

0044B891  |.  8B45 8C       mov     eax, [ebp-74]

0044B894  |.  50            push    eax

0044B895  |.  8D4D EC       lea     ecx, [ebp-14]

0044B898  |.  E8 52FD0A00   call    004FB5EF

0044B89D  |.  C645 FC 02    mov     byte ptr [ebp-4], 2

0044B8A1  |.  8D4D E4       lea     ecx, [ebp-1C]

0044B8A4  |.  E8 0DFC0A00   call    004FB4B6

0044B8A9  |.  68 94075A00   push    005A0794                         ; /Arg2 = 005A0794 ASCII "8<?

0044B8AE  |.  8D45 EC       lea     eax, [ebp-14]                    ; |

0044B8B1  |.  50            push    eax                              ; |Arg1

0044B8B2  |.  E8 B9CCFFFF   call    00448570                         ; \真假码比较call

0044B8B7  |.  33C9          xor     ecx, ecx

0044B8B9  |.  8AC8          mov     cl, al

0044B8BB  |.  85C9          test    ecx, ecx

0044B8BD  |.  0F84 77050000 je      0044BE3A

0044B8C3  |.  833D 98075A00>cmp     dword ptr [5A0798], 1E

0044B8CA  |.  7D 55         jge     short 0044B921

0044B8CC  |.  833D C4F95900>cmp     dword ptr [59F9C4], 0

0044B8D3  |.  75 17         jnz     short 0044B8EC

0044B8D5  |.  C705 C4F95900>mov     dword ptr [59F9C4], 1

0044B8DF  |.  A1 98075A00   mov     eax, [5A0798]

0044B8E4  |.  83C0 01       add     eax, 1

0044B8E7  |.  A3 98075A00   mov     [5A0798], eax

0044B8EC  |>  8D45 EC       lea     eax, [ebp-14]

0044B8EF  |.  50            push    eax

0044B8F0  |.  8B4D 08       mov     ecx, [ebp+8]

0044B8F3  |.  E8 33F90A00   call    004FB22B

0044B8F8  |.  8B4D D8       mov     ecx, [ebp-28]

0044B8FB  |.  83C9 01       or      ecx, 1

0044B8FE  |.  894D D8       mov     [ebp-28], ecx

0044B901  |.  C645 FC 01    mov     byte ptr [ebp-4], 1

0044B905  |.  8D4D EC       lea     ecx, [ebp-14]

0044B908  |.  E8 A9FB0A00   call    004FB4B6

0044B90D  |.  C645 FC 00    mov     byte ptr [ebp-4], 0

0044B911  |.  8D4D 0C       lea     ecx, [ebp+C]

0044B914  |.  E8 9DFB0A00   call    004FB4B6

0044B919  |.  8B45 08       mov     eax, [ebp+8]

0044B91C  |.  E9 8C050000   jmp     0044BEAD

0044B921  |>  833D C4F95900>cmp     dword ptr [59F9C4], 0

0044B928  |.  0F85 D9040000 jnz     0044BE07

0044B92E  |.  C705 C4F95900>mov     dword ptr [59F9C4], 1

0044B938  |.  A1 98075A00   mov     eax, [5A0798]

0044B93D  |.  83C0 01       add     eax, 1

0044B940  |.  A3 98075A00   mov     [5A0798], eax

0044B945  |.  6A 54         push    54

0044B947  |.  8D4D EC       lea     ecx, [ebp-14]

0044B94A  |.  E8 18970A00   call    004F5067

0044B94F  |.  6A 68         push    68

0044B951  |.  8D4D EC       lea     ecx, [ebp-14]

0044B954  |.  E8 60FF0A00   call    004FB8B9

0044B959  |.  6A 69         push    69

0044B95B  |.  8D4D EC       lea     ecx, [ebp-14]

0044B95E  |.  E8 56FF0A00   call    004FB8B9

0044B963  |.  6A 73         push    73

0044B965  |.  8D4D EC       lea     ecx, [ebp-14]

0044B968  |.  E8 4CFF0A00   call    004FB8B9

0044B96D  |.  6A 20         push    20

0044B96F  |.  8D4D EC       lea     ecx, [ebp-14]

0044B972  |.  E8 42FF0A00   call    004FB8B9

0044B977  |.  6A 76         push    76

0044B979  |.  8D4D EC       lea     ecx, [ebp-14]

0044B97C  |.  E8 38FF0A00   call    004FB8B9

0044B981  |.  6A 65         push    65

0044B983  |.  8D4D EC       lea     ecx, [ebp-14]

0044B986  |.  E8 2EFF0A00   call    004FB8B9

0044B98B  |.  6A 72         push    72

0044B98D  |.  8D4D EC       lea     ecx, [ebp-14]

0044B990  |.  E8 24FF0A00   call    004FB8B9

0044B995  |.  6A 73         push    73

0044B997  |.  8D4D EC       lea     ecx, [ebp-14]

0044B99A  |.  E8 1AFF0A00   call    004FB8B9

0044B99F  |.  6A 69         push    69

0044B9A1  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9A4  |.  E8 10FF0A00   call    004FB8B9

0044B9A9  |.  6A 6F         push    6F

0044B9AB  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9AE  |.  E8 06FF0A00   call    004FB8B9

0044B9B3  |.  6A 6E         push    6E

0044B9B5  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9B8  |.  E8 FCFE0A00   call    004FB8B9

0044B9BD  |.  6A 20         push    20

0044B9BF  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9C2  |.  E8 F2FE0A00   call    004FB8B9

0044B9C7  |.  6A 72         push    72

0044B9C9  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9CC  |.  E8 E8FE0A00   call    004FB8B9

0044B9D1  |.  6A 65         push    65

0044B9D3  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9D6  |.  E8 DEFE0A00   call    004FB8B9

0044B9DB  |.  6A 71         push    71

0044B9DD  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9E0  |.  E8 D4FE0A00   call    004FB8B9

0044B9E5  |.  6A 75         push    75

0044B9E7  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9EA  |.  E8 CAFE0A00   call    004FB8B9

0044B9EF  |.  6A 69         push    69

0044B9F1  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9F4  |.  E8 C0FE0A00   call    004FB8B9

0044B9F9  |.  6A 72         push    72

0044B9FB  |.  8D4D EC       lea     ecx, [ebp-14]

0044B9FE  |.  E8 B6FE0A00   call    004FB8B9

0044BA03  |.  6A 65         push    65

0044BA05  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA08  |.  E8 ACFE0A00   call    004FB8B9

0044BA0D  |.  6A 73         push    73

0044BA0F  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA12  |.  E8 A2FE0A00   call    004FB8B9

0044BA17  |.  6A 20         push    20

0044BA19  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA1C  |.  E8 98FE0A00   call    004FB8B9

0044BA21  |.  6A 61         push    61

0044BA23  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA26  |.  E8 8EFE0A00   call    004FB8B9

0044BA2B  |.  6A 20         push    20

0044BA2D  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA30  |.  E8 84FE0A00   call    004FB8B9

0044BA35  |.  6A 6E         push    6E

0044BA37  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA3A  |.  E8 7AFE0A00   call    004FB8B9

0044BA3F  |.  6A 65         push    65

0044BA41  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA44  |.  E8 70FE0A00   call    004FB8B9

0044BA49  |.  6A 77         push    77

0044BA4B  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA4E  |.  E8 66FE0A00   call    004FB8B9

0044BA53  |.  6A 20         push    20

0044BA55  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA58  |.  E8 5CFE0A00   call    004FB8B9

0044BA5D  |.  6A 72         push    72

0044BA5F  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA62  |.  E8 52FE0A00   call    004FB8B9

0044BA67  |.  6A 65         push    65

0044BA69  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA6C  |.  E8 48FE0A00   call    004FB8B9

0044BA71  |.  6A 67         push    67

0044BA73  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA76  |.  E8 3EFE0A00   call    004FB8B9

0044BA7B  |.  6A 69         push    69

0044BA7D  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA80  |.  E8 34FE0A00   call    004FB8B9

0044BA85  |.  6A 73         push    73

0044BA87  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA8A  |.  E8 2AFE0A00   call    004FB8B9

0044BA8F  |.  6A 74         push    74

0044BA91  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA94  |.  E8 20FE0A00   call    004FB8B9

0044BA99  |.  6A 72         push    72

0044BA9B  |.  8D4D EC       lea     ecx, [ebp-14]

0044BA9E  |.  E8 16FE0A00   call    004FB8B9

0044BAA3  |.  6A 61         push    61

0044BAA5  |.  8D4D EC       lea     ecx, [ebp-14]

0044BAA8  |.  E8 0CFE0A00   call    004FB8B9

0044BAAD  |.  6A 74         push    74

0044BAAF  |.  8D4D EC       lea     ecx, [ebp-14]
```
