
## 反汇编记录

`ECX` 00000001

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F760

`EBP` 0012F798

`ESI` 00FC5138

`EDI` 0012F84D

`EIP` 00402C42 `AspAppEn.00402C42`

```
00402C42    89C8            mov     eax, ecx
```

`EAX` 00000001

`ECX` 00000001

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F760

`EBP` 0012F798

`ESI` 00FC5138

`EDI` 0012F84D

`EIP` 00402C44 `AspAppEn.00402C44`

```
00402C44    39F7            cmp     edi, esi     ；比较
00402C46    77 13           ja      short 00402C5B     ；跳转不实现
00402C48    74 2F           je      short 00402C79     ；跳不了
00402C4A    C1F9 02         sar     ecx, 2     ；ECX右移2
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F760

`EBP` 0012F798

`ESI` 00FC5138

`EDI` 0012F84D

`EIP` 00402C4D `AspAppEn.00402C4D`

```
00402C4D    78 2A           js      short 00402C79     ；跳转不了
00402C4F    F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]     ；重复前缀，不知道是什么意思
00402C51    89C1            mov     ecx, eax
```

`EAX` 00000001

`ECX` 00000001

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F760

`EBP` 0012F798

`ESI` 00FC5138

`EDI` 0012F84D

`EIP` 00402C53 `AspAppEn.00402C53`

```
00402C53    83E1 03         and     ecx, 3     ；与运算
00402C56    F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]      ；重复前缀，不知道是什么意思
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F760

`EBP` 0012F798

`ESI` 00FC5139

`EDI` 0012F84E

`EIP` 00402C58 `AspAppEn.00402C58`

```
00402C58    5F              pop     edi    ；出栈
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F764

`EBP` 0012F798

`ESI` 00FC5139

`EDI` 0012F84C

`EIP` 00402C59 `AspAppEn.00402C59`

```
00402C59    5E              pop     esi    ；出栈
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000001

`ESP` 0012F768

`EBP` 0012F798

`ESI` 00F66460

`EDI` 0012F84C

`EIP` 00402C5A `AspAppEn.00402C5A`

```
00402C5A    C3              retn     返回到 00404F47
```

```
00404F47    5B              pop     ebx     ；出栈
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000014

`ESP` 0012F770

`EBP` 0012F798

`ESI` 00F66460

`EDI` 0012F84C

`EIP` 00404F48 `AspAppEn.00404F48`

```
00404F48    C3              retn     ；返回到 004FB4A2
```

```
004FB4A2    4B    dec     ebx     ；EBX减1
```

`EAX` 00000001

`ECX` 00000000

`EDX` 0012F84D

`EBX` 00000013

`ESP` 0012F774

`EBP` 0012F798

`ESI` 00F66460

`EDI` 0012F84C

`EIP` 004FB4A3 `AspAppEn.004FB4A3`

```
004FB4A3  ^ 75 AA jnz     short 004FB44F 向上跳，继续跳转到 004FB44F处理第二个字符。
```

只分析到这里,迫不急待的想发点东西上来(其实也进行不下去了),

想请高手帮忙.想传附件,可是权限不够

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
