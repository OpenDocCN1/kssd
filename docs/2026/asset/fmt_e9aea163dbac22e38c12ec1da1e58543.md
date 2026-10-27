
```text
0041A936     90                   nop
0041A937     90                   nop
0041A938     90                   nop
0041A939     90                   nop
0041A93A     90                   nop
0041A93B     90                   nop
0041A93C     90                   nop
0041A93D     90                   nop
0041A93E     90                   nop
0041A93F     90                   nop
0041A940     90                   nop
0041A941     A1 BD384200          mov eax, dword ptr ds:[4238BD]
0041A946     05 48BC0400          add eax,4BC48
0041A94B     8987 A0000000        mov dword ptr ds:[edi+A0],eax
0041A951     C787 A4000000 0A0000>mov dword ptr ds:[edi+A4],0A
0041A95B     803D 7C114B00 01     cmp byte ptr ds:[4B117C],1
0041A962     75 21                jnz short 0041A985
0041A964     A1 40A74400          mov eax,dword ptr ds:[44A740]
0041A969     A3 48F24600          mov dword ptr ds:[46F248],eax
0041A96E     C705 4CF24600 0A0000>mov dword ptr ds:[46F24C],0A
0041A978     B8 0F000000          mov eax,0F
0041A97D     B4 30                mov ah,30
0041A97F     66:A3 50F24600       mov word ptr ds:[46F250],ax
```

## 16、第十六处被抽取的代码

```text
0041A985     E8 00000000          call 0041A98A
0041A98A     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：→ → →

```text
0041A985     90                   nop
0041A986     90                   nop
0041A987     90                   nop
0041A988     90                   nop
0041A989     90                   nop
0041A98A     90                   nop
0041A98B     90                   nop
0041A98C     90                   nop
0041A98D     90                   nop
0041A98E     90                   nop
0041A98F     90                   nop
0041A990     90                   nop
0041A991     90                   nop
0041A992     90                   nop
0041A993     90                   nop
0041A994     90                   nop
0041A995     C3                   retn
```

---

## 17、第十七处被抽取的代码

```text
0041A8F1     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                                  ====>入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]

                                  ====>这里对0041A901至0041A91B之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`0041A901` 大小：`0041A91B-0041A901=1A` 存为：`0041A901.dmp`

用`WinHex`把这段代码写入脱壳后程序的相应位置！改`0041A8F1`后面全部`NOP`

```text
0041A8F1     90                   nop
0041A8F2     90                   nop
0041A8F3     90                   nop
0041A8F4     90                   nop
0041A8F5     90                   nop
0041A8F6     90                   nop
0041A8F7     90                   nop
0041A8F8     90                   nop
0041A8F9     90                   nop
0041A8FA     90                   nop
0041A8FB     90                   nop
0041A8FC     90                   nop
0041A8FD     90                   nop
0041A8FE     90                   nop
0041A8FF     90                   nop
0041A900     90                   nop
0041A901     A1 BD384200          mov eax, dword ptr ds:[4238BD]
0041A906     05 7C600000          add eax, 607C
0041A90B     8987 80000000        mov dword ptr ds:[edi+80], eax
0041A911     C787 84000000 E40000>mov dword ptr ds:[edi+84], 0E4
0041A91B     E8 00000000          call 0041A920
```

---

## 18、第十八处被抽取的代码

```text
0041A920     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：→ → →

```text
0041A920     90                   nop
0041A921     90                   nop
0041A922     90                   nop
0041A923     90                   nop
0041A924     90                   nop
0041A925     90                   nop
0041A926     90                   nop
0041A927     90                   nop
0041A928     90                   nop
0041A929     90                   nop
0041A92A     90                   nop
0041A92B     C3                   retn
```

---

## 19、第十九处被抽取的代码

```text
0041C9EB     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                                  ====>入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]

                                  ====>这里对0041C9FB至0041CA4B之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`0041C9FB` 大小：`0041CA4B-0041C9FB=50` 存为：`0041C9FB.dmp`

用`WinHex`把这段代码写入脱壳后程序的相应位置！改`0041C9EB`后面全部`NOP`

```text
0041C9EB     90                   nop
0041C9EC     90                   nop
0041C9ED     90                   nop
0041C9EE     90                   nop
0041C9EF     90                   nop
0041C9F0     90                   nop
0041C9F1     90                   nop
0041C9F2     90                   nop
0041C9F3     90                   nop
0041C9F4     90                   nop
0041C9F5     90                   nop
0041C9F6     90                   nop
0041C9F7     90                   nop
0041C9F8     90                   nop
0041C9F9     90                   nop
0041C9FA     90                   nop
0041C9FB     E8 60000000          call 0041CA60
0041CA00     E8 6B010000          call 0041CB70
0041CA05     E8 15010000          call 0041CB1F
0041CA0A     E8 4F000000          call 0041CA5E
0041CA0F     E8 48000000          call 0041CA5C
0041CA14     E8 F7680000          call 00423310
0041CA19     E8 39690000          call 00423357
0041CA1E     E8 A3690000          call 004233C6
0041CA23     E8 466A0000          call 0042346E
0041CA28     E8 516A0000          call 0042347E
0041CA2D     E8 936A0000          call 004234C5
0041CA32     E8 E96A0000          call 00423520
0041CA37     E8 BE030000          call 0041CDFA
0041CA3C     E8 306B0000          call 00423571
0041CA41     E8 786B0000          call 004235BE
0041CA46     E8 39000000          call 0041CA84
0041CA4B     E8 00000000          call 0041CA50
```

---

## 20、第二十处被抽取的代码

```text
0041CA50     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：→ → →

```text
0041CA50     90                   nop
0041CA51     90                   nop
0041CA52     90                   nop
0041CA53     90                   nop
0041CA54     90                   nop
0041CA55     90                   nop
0041CA56     90                   nop
0041CA57     90                   nop
0041CA58     90                   nop
0041CA59     90                   nop
0041CA5A     90                   nop
0041CA5B     90                   nop
0041CA5C     90                   nop
0041CA5D     C3                   retn
```

---

## 21、第廿一处被抽取的代码

```text
00423315     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                                  ====>入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]

                                  ====>这里对00423325至00423346之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`0041C9FB` 大小：`00423346-00423325=21` 存为：`00423325.dmp`

用`WinHex`把这段代码写入脱壳后程序的相应位置！改`00423315`后面全部`NOP`

```text
00423315     90                   nop
00423316     90                   nop
00423317     90                   nop
00423318     90                   nop
00423319     90                   nop
0042331A     90                   nop
0042331B     90                   nop
0042331C     90                   nop
0042331D     90                   nop
0042331E     90                   nop
0042331F     90                   nop
00423320     90                   nop
00423321     90                   nop
00423322     90                   nop
00423323     90                   nop
00423324     90                   nop
00423325     E8 0F95FFFF          call 0041C839
0042332A     A2 ACC34200          mov byte ptr ds:[42C3AC], al
0042332F     8D35 0BC84200        lea esi, dword ptr ds:[42C80B]
00423335     8BFE                 mov edi, esi
00423337     B9 AE050000          mov ecx, 5AE
0042333C     AC                   lods byte ptr ds:[esi]
0042333D     2A05 ACC34200        sub al, byte ptr ds:[42C3AC]
00423343     AA                   stos byte ptr es:[edi]
00423344     E2 F6                loopd short 0042333C
00423346     E8 00000000          call 0042334B
```
