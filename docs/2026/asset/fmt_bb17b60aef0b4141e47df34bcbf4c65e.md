
```
00413EAB   |.  C64424 14 00      mov byte ptr ss:[esp+14], 0
00413EB0       E8 CBB00000       call Phone.0041EF80         这里
00413EB5       8BC8              mov ecx, eax                EAX重点。
00413EB7       E8 64040100       call Phone.00424320        无狗此CALL出错
00413EBC   |>  8D4C24 00         lea ecx, dword ptr ss:[esp]
00413EC0   |.  C74424 10 FFFFFFF>mov dword ptr ss:[esp+10], -1
00413EC8   |.  E8 C3E90100       call <jmp.&MFC42.#800>
00413ECD   |.  8B4C24 08         mov ecx, dword ptr ss:[esp+8]
00413ED1   |.  64:890D 00000000  mov dword ptr fs:[0], ecx
00413ED8       83C4 14           add esp, 14
00413EDB   .  C3                retn
```

无狗`CALL`出错，那我们就带上狗看看，哦。正常，转存一下`EAX`的值看看

```
0034CB40  18 C2 43 00 01 00 00 00 00 00 00 00 00 00 00 00  .C.............
0034CB50  00 00 00 00 01 00 00 00 00 00 00 00 C8 3B 13 00  ............?..
0034CB60  44 02 3E 00 00 00 00 00 00 00 00 00 41 6D DF 77  D.>.........Amw
0034CB70  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0034CB80  38 98 34 00 00 00 00 00 00 00 00 00 00 00 00 00  8.4.............
0034CB90  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0034CBA0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0034CBB0  60 00 00 00 60 00 00 00 00 00 00 00 00 00 00 00  `...`...
```

下个硬件写入断点看看什么时候会生成

```
00420225   |.  E8 1C280100       call <jmp.&MFC42.#541>
0042022A   |.  8D8E 84010000     lea ecx, dword ptr ds:[esi+184]
00420230   |.  C64424 24 06      mov byte ptr ss:[esp+24], 6
00420235   |.  E8 74260100       call <jmp.&MFC42.#540>
0042023A   |.  8D8E 94010000     lea ecx, dword ptr ds:[esi+194]
00420240   |.  C64424 24 07      mov byte ptr ss:[esp+24], 7
00420245   |.  E8 F22B0100       call <jmp.&MFC42.#500>
0042024A   |.  8D8E B0010000     lea ecx, dword ptr ds:[esi+1B0]
00420250   |.  C64424 24 08      mov byte ptr ss:[esp+24], 8
00420255   |.  E8 36C90000       call Phone.0042CB90
0042025A   |.  8D8E 88030000     lea ecx, dword ptr ds:[esi+388]
00420260   |.  C64424 24 09      mov byte ptr ss:[esp+24], 9
00420265   |.  E8 44260100       call <jmp.&MFC42.#540>
0042026A       C706 18C24300     mov dword ptr ds:[esi], 0043C218  原来这里附值啊。我靠。
00420270   |.  B9 07000000       mov ecx, 7
00420275   |.  33C0              xor eax, eax
00420277   |.  8DBE 30020000     lea edi, dword ptr ds:[esi+230]
0042027D   |.  8935 BCEF4400     mov dword ptr ds:[44EFBC], esi
00420283   |.  C64424 24 0A      mov byte ptr ss:[esp+24], 0A
00420288   |.  F3:AB             rep stos dword ptr es:[edi]
```

运行到`0042026A`处的`ESI`的值就是我们需要的地址了，用SMC打个补就行了

软件还有多处同样地方地方需要这个值，搜索全部调`0041EF80`不会遗漏。全部打上

OK，已经可以不要狗运行了
