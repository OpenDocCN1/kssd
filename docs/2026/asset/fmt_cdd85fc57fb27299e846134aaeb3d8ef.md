
## 汇编代码片段

```
004302FE  |.  51         |push    ecx                       ; |Arg1

004302FF  |.  E8 2869070>|call    004A6C2C                  ; \\_sprintf

00430304  |.  83C4 0C    |add     esp, 0C

00430307  |.  0145 F4    |add     dword ptr [ebp-C], eax

0043030A  |>  833D 50814>|cmp     dword ptr [4D8150], 0

00430311  |.  74 1D      |je      short 00430330

00430313  |.  A1 443B4E0>|mov     eax, dword ptr [4E3B44]

00430318  |.  8D95 98FDF>|lea     edx, dword ptr [ebp-268]

0043031E  |.  50         |push    eax                       ; /Arg3 => 00000000

0043031F  |.  68 79624B0>|push    004B6279                  ; | write=%08x

00430324  |.  0355 F4    |add     edx, dword ptr [ebp-C]    ; |

00430327  |.  52         |push    edx                       ; |Arg1

00430328  |.  E8 FF68070>|call    004A6C2C                  ; \\_sprintf

0043032D  |.  83C4 0C    |add     esp, 0C

00430330  |>  8D8D 98FDF>|lea     ecx, dword ptr [ebp-268]

00430336  |.  51         |push    ecx                       ; /Arg1

00430337  |.  E8 2C14000>|call    \_Infoline                 ; \\_Infoline

0043033C  |.  59         |pop     ecx

0043033D  |.  A1 403B4E0>|mov     eax, dword ptr [4E3B40]

00430342  |.  A3 483B4E0>|mov     dword ptr [4E3B48], eax

00430347  |.  8B15 443B4>|mov     edx, dword ptr [4E3B44]

0043034D  |.  8915 4C3B4>|mov     dword ptr [4E3B4C], edx
```

### 检查模块信息结构体是否正确

```
00430353  |>  833D 648D4>|cmp     dword ptr [4D8D64], 2

0043035A  |.  74 43      |je      short 0043039F

0043035C  |.  833D 648D4>|cmp     dword ptr [4D8D64], 1

00430363  |.  75 49      |jnz     short 004303AE

00430365  |.  833D 688D4>|cmp     dword ptr [4D8D68], 0

0043036C  |.  74 40      |je      short 004303AE

0043036E  |.  837D AC 00 |cmp     dword ptr [ebp-54], 0

00430372  |.  74 3A      |je      short 004303AE

00430374  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]

00430377  |.  8B55 AC    |mov     edx, dword ptr [ebp-54]

0043037A  |.  8B81 E1030>|mov     eax, dword ptr [ecx+3E1]

00430380  |.  3B42 0C    |cmp     eax, dword ptr [edx+C]  ; 检查入口点是否在模块首前面

00430383  |.  72 29      |jb      short 004303AE

00430385  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]

00430388  |.  8B55 AC    |mov     edx, dword ptr [ebp-54]

0043038B  |.  8B41 0C    |mov     eax, dword ptr [ecx+C]

0043038E  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]

00430391  |.  0382 E9030>|add     eax, dword ptr [edx+3E9]

00430397  |.  3B81 E1030>|cmp     eax, dword ptr [ecx+3E1] ; 检查入口点是否在模块尾的后面

0043039D  |.  76 0F      |jbe     short 004303AE
```

### 将临时变量置空

```
0043039F  |>  33C0       |xor     eax, eax

004303A1  |.  33D2       |xor     edx, edx

004303A3  |.  A3 54814D0>|mov     dword ptr [4D8154], eax

004303A8  |.  8915 50814>|mov     dword ptr [4D8150], edx
```

### 错误处理的部分

```
004303AE  |>  B8 0200000>|mov     eax, 2      ; 返回2

004303B3  |.  E9 6D10000>|jmp     00431425    ; 跳转到结束处
```

### 设置内存断点的提示信息

```
004303B8  |>  8B55 A4    |mov     edx, dword ptr [ebp-5C]

004303BB  |.  837A 14 00 |cmp     dword ptr [edx+14], 0

004303BF  |.  75 14      |jnz     short 004303D5

004303C1  |.  57         |push    edi                       ; /Arg3

004303C2  |.  68 85624B0>|push    004B6285              ; |读取 [%08lx] 时设置内存断点

004303C7  |.  8B4D D8    |mov     ecx, dword ptr [ebp-28]   ; |

004303CA  |.  51         |push    ecx                       ; |Arg1

004303CB  |.  E8 30DDFFF>|call    0042E100                  ; \SetInfoMsg

004303D0  |.  83C4 0C    |add     esp, 0C

004303D3  |.  EB 12      |jmp     short 004303E7

004303D5  |>  57         |push    edi                       ; /Arg3

004303D6  |.  68 AC624B0>|push    004B62AC             ; |写到 [%08lx] 时设置内存断点

004303DB  |.  8B45 D8    |mov     eax, dword ptr [ebp-28]   ; |

004303DE  |.  50         |push    eax                       ; |Arg1

004303DF  |.  E8 1CDDFFF>|call    0042E100                  ; \ SetInfoMsg

004303E4  |.  83C4 0C    |add     esp, 0C
```

### 调整优先级

```
004303E7  |>  6A 00      |push    0                         ; /Arg1 = 00000000

004303E9  |.  E8 EA16000>|call    \_Animate                  ; \\_Animate

004303EE  |.  59         |pop     ecx

004303EF  |.  33C0       |xor     eax, eax    ; 返回0

004303F1  |.  E9 2F10000>|jmp     00431425  ; 跳转到结束处
```

### 循环体判断条件以及自加部分，小于三的时候继续比较

```
004303F6  |>  43         |inc     ebx

004303F7  |.  83C0 04    |add     eax, 4

004303FA  |>  83FB 03     cmp     ebx, 3

004303FD  |.  7D 0A      |jge     short 00430409
```

### 检查读取机器码的长度，如果大于0则继续跳转到上面判断，否则就结束程序，返回2

```
004303FF  |.  837D F4 00 |cmp     dword ptr [ebp-C], 0

00430403  |.^ 0F8F E5FDF>\jg      004301EE
```

### 设置错误返回值，并退出

```
00430409  |>  FF05 303B4>inc     dword ptr [4E3B30]

0043040F  |.  B8 0200000>mov     eax, 2    ; 设置返回值为2

00430414  |.  E9 0C10000>jmp     00431425  ; 跳转到结束处
```

### 设置错误信息到全局变量，返回2并退出

```
00430419  |>  833D 5C8D4>cmp     dword ptr [4D8D5C], 0

00430420  |.  74 19      je      short 0043043B

00430422  |.  830D 74574>or      dword ptr [4D5774], 20

00430429  |.  33D2       xor     edx, edx

0043042B  |.  8915 5C8D4>mov     dword ptr [4D8D5C], edx

00430431  |.  B8 0200000>mov     eax, 2      ; 返回2

00430436  |.  E9 EA0F000>jmp     00431425    ; 跳转到结束处
```

### 检查内存断点是否在kernel32模块中，若是则弹出错误提示窗口

```
0043043B  |>  810D 74574>or      dword ptr [4D5774], 204

00430445  |.  833D A0574>cmp     dword ptr [4D57A0], 0

0043044C  |.  74 52      je      short 004304A0

0043044E  |.  8B55 D8    mov     edx, dword ptr [ebp-28]

00430451  |.  52         push    edx                        ; /Arg1

00430452  |.  E8 C1D9020>call    \_Findmodule                ; \\_Findmodule

00430457  |.  59         pop     ecx

00430458  |.  8945 AC    mov     dword ptr [ebp-54], eax

0043045B  |.  837D AC 00 cmp     dword ptr [ebp-54], 0

0043045F  |.  74 3F      je      short 004304A0

00430461  |.  6A 08      push    8                          ; /Arg3 = 00000008

00430463  |.  68 D6624B0>push    004B62D6                   ; |kernel32

00430468  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]    ; |

0043046B  |.  83C1 48    add     ecx, 48                    ; |

0043046E  |.  51         push    ecx                        ; |Arg1

0043046F  |.  E8 E034070>call    004A3954                   ; \Ollydbg.004A3954

00430474  |.  83C4 0C    add     esp, 0C

00430477  |.  85C0       test    eax, eax

00430479  |.  75 25      jnz     short 004304A0

0043047B  |.  68 DF624B0>push    004B62DF      ; /访问违反在 kernel32 中，根据请求已忽略

00430480  |.  8B45 D8    mov     eax, dword ptr [ebp-28]    ; |

00430483  |.  50         push    eax                        ; |Arg1

00430484  |.  E8 A711000>call    \_Message                   ; \\_Message

00430489  |.  83C4 08    add     esp, 8

0043048C  |.  C705 FC564>mov     dword ptr [4D56FC], 2

00430496  |.  B8 0100000>mov     eax, 1    ; 返回1

0043049B  |.  E9 850F000>jmp     00431425  ; 跳到程序结束处
```

### 根据上面设置的变量，设置相关的错误信息以及弹出窗口

```
004304A0  |>  8B55 A4    mov     edx, dword ptr [ebp-5C]

004304A3  |.  837A 10 02 cmp     dword ptr [edx+10], 2

004304A7  |.  72 3E      jb      short 004304E7

004304A9  |.  68 C6614B0>push   004B61C6 ; $使用“shift+f7/f8/f9”键跳过异常以继续执行程序

004304AE  |.  57         push    edi

004304AF  |.  8B4D A4    mov     ecx, dword ptr [ebp-5C]

004304B2  |.  8379 14 01 cmp     dword ptr [ecx+14], 1

004304B6  |.  75 07      jnz     short 004304BF

004304B8  |.  B8 32634B0>mov     eax, 004B6332              ;  写入到

004304BD  |.  EB 11      jmp     short 004304D0

004304BF  |>  3B7D D8    cmp     edi, dword ptr [ebp-28]

004304C2  |.  75 07      jnz     short 004304CB

004304C4  |.  B8 3D634B0>mov     eax, 004B633D              ;  正在执行

004304C9  |.  EB 05      jmp     short 004304D0
```

### 设置错误提示

```
004304CB  |>  B8 47634B0>mov     eax, 004B6347              ;  读取

004304D0  |>  50         push    eax                        ; |Arg3

004304D1  |.  68 0F634B0>push    004B630F                   ; |访问违反： %s [%08lx]%s

004304D6  |.  8D95 98FDF>lea     edx, dword ptr [ebp-268]       ; |

004304DC  |.  52         push    edx                        ; |Arg1
```
