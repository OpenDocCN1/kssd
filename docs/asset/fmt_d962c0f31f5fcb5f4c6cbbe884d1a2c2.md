
# 单步执行逻辑分析

## 硬件断点检查

比较是否是单步，如果不是则继续比较下个线程的硬件断点：

```
0042E9D4  |.  EB 3F         |jmp     short 0042EA15
```

0042E9D6  |>  8B45 EC       |mov     eax, dword ptr [ebp-14]

0042E9D9  |.  F680 F4020000>|test    byte ptr [eax+2F4], 1

0042E9E0  |.  0F84 4C010000 |je      0042EB32    ;跳转到上面继续比较

## 设置单步执行

```
0042E9E6  |.  8B5D EC       |mov     ebx, dword ptr [ebp-14]

0042E9E9  |.  83C3 20       |add     ebx, 20

0042E9EC  |.  C703 11000100 |mov     dword ptr [ebx], 10011

0042E9F2  |.  8B55 EC       |mov     edx, dword ptr [ebp-14]

0042E9F5  |.  81C2 EC020000 |add     edx, 2EC

0042E9FB  |.  8B42 2C       |mov     eax, dword ptr [edx+2C]

0042E9FE  |.  8983 B8000000 |mov     dword ptr [ebx+B8], eax

0042EA04  |.  818B C0000000>|or      dword ptr [ebx+C0], 100    ;设置单步标识

0042EA0E  |.  83A2 E6000000>|and     dword ptr [edx+E6], FFFFFFF0
```

设置完寄存器后对应其标识处理 `context.dr7` 并设置线程环境，这部分与硬件断点表删除中的代码一样，可以参考从 `00408AF6` 到 `00408BCB` 部分的代码解释，就不再重复分析了：

## 后续处理与线程上下文设置

```
0042EA51  |>  833A 00       |/cmp    dword ptr [edx], 0

0042EA54  |.  0F84 B0000000 ||je     0042EB0A

0042EA5A  |.  F605 10574D00>||test   byte ptr [4D5710], 1

0042EA61  |.  0F85 A3000000 ||jnz    0042EB0A
```

……

……

```
0042EB17  |.  8973 18       |mov     dword ptr [ebx+18], esi

0042EB1A  |>  53           |push    ebx                   ; /pContext

0042EB1B  |.  8B45 EC       |mov     eax, dword ptr [ebp-14]       ; |

0042EB1E  |.  8B50 0C       |mov     edx, dword ptr [eax+C]        ; |

0042EB21  |.  52           |push    edx                   ; |hThread

0042EB22  |.  E8 83060800   |call <jmp.&KERNEL32.SetThreadContext>  ; \\SetThreadContext

0042EB27  |.  85C0         |test    eax, eax

0042EB29  |.  75 07         |jnz     short 0042EB32

0042EB2B  |.  C745 F4 FFFFF>|mov    dword ptr [ebp-C], -1

0042EB32  |>  FF45 FC       |inc     dword ptr [ebp-4]

0042EB35  |.  8145 EC 6C060>|add    dword ptr [ebp-14], 66C
```

这里检查设置过的线程数量，如果没有全部设置，继续跳转到设置函数：

```
0042EB3C  |>  8B4D FC       mov     ecx, dword ptr [ebp-4]

0042EB3F  |.  3B0D 987D4D00 |cmp     ecx, dword ptr [4D7D98]

0042EB45  |.^ 0F8C BFFCFFFF \\jl     0042E80A    ;跳转到设置 CONTEXT 函数体中
```

## 错误处理与返回值

查看返回值是否为 0，不是则弹出报错窗口：

```
0042EB4B  |.  837D F4 00    cmp     dword ptr [ebp-C], 0

0042EB4F  |.  74 1A         je      short 0042EB6B

0042EB51  |.  68 195F4B00   push    004B5F19                ; /无法写回被修改的寄存器

0042EB56  |.  6A 00         push    0                   ; |Arg1 = 00000000

0042EB58  |.  E8 D32A0000   call    _Message             ; \\_Message

0042EB5D  |.  83C4 08       add     esp, 8

0042EB60  |.  68 195F4B00   push    004B5F19            ;  无法写回被修改的寄存器

0042EB65  |.  E8 B2540200   call    _Error

0042EB6A  |.  59           pop     ecx
```

## 函数结尾

设置返回值：

```
0042EB6B  |>  8B45 F4       mov     eax, dword ptr [ebp-C]
```

恢复现场：

```
0042EB6E  |>  5F           pop     edi

0042EB6F  |.  5E           pop     esi

0042EB70  |.  5B           pop     ebx

0042EB71  |.  8BE5         mov     esp, ebp

0042EB73  |.  5D           pop     ebp

0042EB74  \\.  C3          retn
```

武汉科锐学员: `angelqkm`
2008-5-23
