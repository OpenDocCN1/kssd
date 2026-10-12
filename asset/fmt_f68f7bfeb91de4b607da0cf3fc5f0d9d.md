
```
0067   02       011F72A6         mov       [addr_context.register], addr_context.eax
0068   11 00    011F72B4         push      [context.register]
0069   07 02    011F72DE         mov       [addr_context.register], addr_context.eax
0070   13 01    011F72C2         push      [context.register]
0071   3B 01    011F72D0         Test      [esp+4], [esp] (4 bytes)       ; test eax, eax
0072   3A 00    011F72EC         jnz       011F7500

0007   3C 02    011F72FA         mov       [addr_context.register], context.ebp
0008   3B 00    011F7316         Add       [addr_context.register], 09AC933B
0009   32 01    011F7340         push      [addr_context.register]
0010   36 00    011F7324         mov       [addr_context.register], addr_context.eax
0011   30 01    011F7308         pop       [context.register] (4 bytes)     ; lea eax, dword ptr [ebp+09AC933B]

0012   1C 02    011F7332         load      context.register, 0AAE8C69
0013   1A 01    011F734E         mov       [addr_context.register], addr_context.eax
0014   11 00    011F7386         push      [context.register]
0015   1A 01    011F735C         mov       [addr_context.register], context.esp
0016   08 00    011F7378         Sub       [addr_context.register], 00000004
0017   2D 00    011F736A         pop       [context.register] (4 bytes)
0018   20 00    011F7394         push      [addr_context.register]
0019   30 01    011F73A2         pop       [addr_context.esp] (4 bytes)     ; push eax

0020   34 02    011F73B0         push      00000000
0021   07 02    011F73BE         mov       [addr_context.register], context.esp
0022   23 01    011F73DA         Sub       [addr_context.register], 00000004
0023   0A 02    011F73E8         pop       [context.register] (4 bytes)
0024   13 01    011F73CC         push      [addr_context.register]
0025   14 02    011F73F6         pop       [addr_context.esp] (4 bytes)     ; push 0

0026   07 02    011F7404         mov       [addr_context.register], context.esp
0027   23 01    011F742E         Sub       [addr_context.register], 00000004
0028   34 02    011F7412         push      [addr_context.register]
0029   2D 00    011F744A         pop       [addr_context.esp] (4 bytes)
0030   11 00    011F7466         push      011F7490
0031   07 02    011F7482         mov       [addr_context.register], context.esp
0032   30 01    011F7458         pop       [context.register] (4 bytes)
0033   07 02    011F7474         mov       [addr_context.register], context.ebp
0034   3B 00    011F7420         Add       [addr_context.register], 099E0055
0035   25 00    011F743C         call      [context.register]        ; call FindWindowA
```

// 堆栈  

0006FF70  00000000  

0006FF74  011F635E  ASCII "File Monitor - Sysinternals: www.sysinternals.com"  

---------------------------------------------------------------------  

011F860B  20 AB A6 C3 1C FF BE 65 52 65 67 6D 6F 6E 43 6C   ?RegmonCl  

011F861B  61 73 73 00 52 65 67 69 73 74 72 79 20 4D 6F 6E  ass.Registry Mon  

011F862B  69 74 6F 72 20 2D 20 53 79 73 69 6E 74 65 72 6E  itor - Sysintern  

011F863B  61 6C 73 3A 20 77 77 77 2E 73 79 73 69 6E 74 65  als: www.sysinte  

011F864B  72 6E 61 6C 73 2E 63 6F 6D 00 F8 D1 36 37 A4 0D  rnals.com.67?  

代码:

```
0112   00       011F939D         push      00000000
0113   36 00    011F93C7         mov       [addr_context.register], context.esp
0114   23 01    011F93AB         Sub       [addr_context.register], 00000004
0115   30 01    011F93D5         pop       [context.register] (4 bytes)
0116   20 00    011F93B9         push      [addr_context.register]
0117   14 02    011F93E3         pop       [addr_context.esp] (4 bytes)
0118   39 01    011F93F1         mov       [addr_context.register], context.ebp
0119   35 01    011F941B         Add       [addr_context.register], 09ACB5F0
0120   11 00    011F940D         push      [addr_context.register]
0121   1A 01    011F9437         mov       [addr_context.register], addr_context.eax
0122   0A 02    011F9429         pop       [context.register] (4 bytes)
0123   13 00    011F93FF         load      context.register, 006D7A4E
0124   39 01    011F9445         mov       [addr_context.register], addr_context.eax
0125   32 01    011F9453         push      [context.register]
0126   39 01    011F946F         mov       [addr_context.register], context.esp
0127   23 01    011F9461         Sub       [addr_context.register], 00000004
0128   14 02    011F9499         pop       [context.register] (4 bytes)
0129   13 01    011F948B         push      [addr_context.register]
0130   2D 00    011F947D         pop       [addr_context.esp] (4 bytes)
0131   13 00    011F94A7         load      context.register, 0BA7DD87
0132   07 02    011F94B5         mov       [addr_context.register], context.esp
0133   17 00    011F9533         Sub       [addr_context.register], 00000004
0134   1F 01    011F94C3         push      [addr_context.register]
0135   14 02    011F94ED         pop       [addr_context.esp] (4 bytes)
0136   11 00    011F94DF         push      011F9541
0137   1A 01    011F94D1         mov       [addr_context.register], context.esp
0138   0A 02    011F9517         pop       [context.register] (4 bytes)
0139   1A 01    011F9509         mov       [addr_context.register], context.ebp
0140   3B 00    011F9525         Add       [addr_context.register], 099E0055     
0141   29 00    011F94FB         call      [context.register]        ; call FindWindowA
```

// 堆栈  

0006FF70  011F8613  ASCII "RegmonClass"  

0006FF74  00000000  

代码:
