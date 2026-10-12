
`004332BB |. >xor ecx, dword ptr ss:[ebp-8]`

`004332BE |. >mov dword ptr ss:[ebp-14], ecx`

`004332C1 |. >mov ecx, dword ptr ss:[ebp-8]`

`004332C4 |. >mov edx, dword ptr ss:[ebp-18]`

`004332C7 |. >rol edx, cl`

`004332C9 |. >mov eax, dword ptr ss:[ebp-C]`

`004332CC |. >add eax, edx`

`004332CE |. >mov dword ptr ss:[ebp-C], eax`

`/////////////////////////////////////////////////`

`004332D1 |. >mov ecx, dword ptr ss:[ebp-20]`

`004332D4 |. >mov edx, dword ptr ss:[ebp-14]`

`004332D7 |. >add edx, dword ptr ds:[ecx+8C]`

`004332DD |. >mov dword ptr ss:[ebp-1C], edx`

`004332E0 |. >mov eax, dword ptr ss:[ebp-14]`

`004332E3 |. >rol eax, 0D`

`004332E6 |. >mov dword ptr ss:[ebp-14], eax`

`004332E9 |. >mov ecx, dword ptr ss:[ebp-20]`

`004332EC |. >mov edx, dword ptr ss:[ebp-14]`

`004332EF |. >imul edx, dword ptr ds:[ecx+90]`

`004332F6 |. >mov dword ptr ss:[ebp-8], edx`

`004332F9 |. >mov eax, dword ptr ss:[ebp-1C]`

`004332FC |. >and eax, 1FF`

`00433301 |. >mov ecx, dword ptr ds:[eax*4+45B3C0]`

`00433308 |. >mov dword ptr ss:[ebp-18], ecx`

`0043330B |. >mov edx, dword ptr ss:[ebp-8]`

`0043330E |. >rol edx, 5`

`00433311 |. >mov dword ptr ss:[ebp-8], edx`

`00433314 |. >mov ecx, dword ptr ss:[ebp-8]`

`00433317 |. >mov eax, dword ptr ss:[ebp-1C]`

`0043331A |. >rol eax, cl`

`0043331C |. >mov ecx, dword ptr ss:[ebp-C]`

`0043331F |. >add ecx, eax`

`00433321 |. >mov dword ptr ss:[ebp-C], ecx`

`00433324 |. >mov edx, dword ptr ss:[ebp-18]`

`00433327 |. >xor edx, dword ptr ss:[ebp-8]`

`0043332A |. >mov dword ptr ss:[ebp-18], edx`

`0043332D |. >mov eax, dword ptr ss:[ebp-8]`

`00433330 |. >rol eax, 5`

`00433333 |. >mov dword ptr ss:[ebp-8], eax`

`00433336 |. >mov ecx, dword ptr ss:[ebp-18]`

`00433339 |. >xor ecx, dword ptr ss:[ebp-8]`

`0043333C |. >mov dword ptr ss:[ebp-18], ecx`

`0043333F |. >mov edx, dword ptr ss:[ebp-4]`

`00433342 |. >xor edx, dword ptr ss:[ebp-8]`

`00433345 |. >mov dword ptr ss:[ebp-4], edx`

`00433348 |. >mov ecx, dword ptr ss:[ebp-8]`

`0043334B |. >mov eax, dword ptr ss:[ebp-18]`

`0043334E |. >rol eax, cl`

`00433350 |. >mov ecx, dword ptr ss:[ebp-10]`

`00433353 |. >add ecx, eax`

`00433355 |. >mov dword ptr ss:[ebp-10], ecx`

`/////////////////////////////////////////////////////////////////////1`

```
00433358  |>mov edx,dword ptr ss:[ebp-4]             ;  取 c1
0043335B  |>rol edx,8                               ;  循环左移 0x8
0043335E  |>mov dword ptr ss:[ebp-8],edx            ;  替换 c2
00433361  |>mov eax,dword ptr ss:[ebp-4]            ;  取 c1
00433364  |>and eax,0FF                             ;  取低 16 位
00433369  |>mov ecx,dword ptr ss:[ebp-C]            ;  取 c3
0043336C  |>xor ecx,dword ptr ds:[eax*4+45B7C0]     ;  与一数据异或
00433373  |>mov dword ptr ss:[ebp-C],ecx            ;  替换 c3
00433376  |>mov edx,dword ptr ss:[ebp-8]            ;  取 c2
00433379  |>and edx,0FF                             ;  取低 16 位
0043337F  |>mov eax,dword ptr ss:[ebp-10]           ;  取 c4
00433382  |>sub eax,dword ptr ds:[edx*4+45B3C0]     ;  与一数据减
00433389  |>mov dword ptr ss:[ebp-10],eax           ;  替换 c4
0043338C  |>mov ecx,dword ptr ss:[ebp-4]            ;  取 c1
0043338F  |>rol ecx,10                              ;  循环左移 0x10
00433392  |>mov dword ptr ss:[ebp-8],ecx            ;  替换 c2
00433395  |>mov edx,dword ptr ss:[ebp-4]            ;  取 c1
00433398  |>rol edx,18                              ;  循环左移 0x18
0043339B  |>mov dword ptr ss:[ebp-4],edx            ;  替换 c1
0043339E  |>mov eax,dword ptr ss:[ebp-8]            ;  取 c2
004333A1  |>and eax,0FF                             ;  取低 16 位
004333A6  |>mov ecx,dword ptr ss:[ebp-14]           ;  取 c5
004333A9  |>sub ecx,dword ptr ds:[eax*4+45B7C0]     ;  与一数据减
004333B0  |>mov dword ptr ss:[ebp-14],ecx           ;  替换 c5
004333B3  |>mov edx,dword ptr ss:[ebp-4]            ;  取 c1
004333B6  |>and edx,0FF                             ;  取低 16 位
004333BC  |>mov eax,dword ptr ss:[ebp-14]           ;  取 c5
004333BF  |>xor eax,dword ptr ds:[edx*4+45B3C0]     ;  与一数据异或
004333C6  |>mov dword ptr ss:[ebp-14],eax           ;  替换 c5
004333C9  |>mov ecx,dword ptr ss:[ebp-C]            ;  /////////////////2
```

`///////////////////////////////////////////////////////////////////`

下面 7 个变换几乎都是差不多的分析，不做过多的说明了

反正觉得这个软件就是很无聊

```
004333CC  |>rol ecx,8
004333CF  |>mov dword ptr ss:[ebp-8],ecx
004333D2  |>mov edx,dword ptr ss:[ebp-C]
004333D5  |>and edx,0FF
004333DB  |>mov eax,dword ptr ss:[ebp-10]
004333DE  |>xor eax,dword ptr ds:[edx*4+45B7C0]
004333E5  |>mov dword ptr ss:[ebp-10],eax
004333E8  |>mov ecx,dword ptr ss:[ebp-8]
004333EB  |>and ecx,0FF
004333F1  |>mov edx,dword ptr ss:[ebp-14]
004333F4  |>sub edx,dword ptr ds:[ecx*4+45B3C0]
004333FB  |>mov dword ptr ss:[ebp-14],edx
004333FE  |>mov eax,dword ptr ss:[ebp-C]
00433401  |>rol eax,10
00433404  |>mov dword ptr ss:[ebp-8],eax
00433407  |>mov ecx,dword ptr ss:[ebp-C]
0043340A  |>rol ecx,18
0043340D  |>mov dword ptr ss:[ebp-C],ecx
00433410  |>mov edx,dword ptr ss:[ebp-8]
00433413  |>and edx,0FF
00433419  |>mov eax,dword ptr ss:[ebp-4]
0043341C  |>sub eax,dword ptr ds:[edx*4+45B7C0]
00433423  |>mov dword ptr ss:[ebp-4],eax
00433426  |>mov ecx,dword ptr ss:[ebp-C]
00433429  |>and ecx,0FF
0043342F  |>mov edx,dword ptr ss:[ebp-4]
00433432  |>xor edx,dword ptr ds:[ecx*4+45B3C0]
00433439  |>mov dword ptr ss:[ebp-4],edx////////////////////////3
```

```
0043343C  |>mov eax,dword ptr ss:[ebp-10]
0043343F  |>sub eax,dword ptr ss:[ebp-C]
00433442  |>mov dword ptr ss:[ebp-10],eax
00433445  |>mov ecx,dword ptr ss:[ebp-10]
00433448  |>rol ecx,8
0043344B  |>mov dword ptr ss:[ebp-8],ecx
0043344E  |>mov edx,dword ptr ss:[ebp-10]
00433451  |>and edx,0FF
00433457  |>mov eax,dword ptr ss:[ebp-14]
0043345A  |>xor eax,dword ptr ds:[edx*4+45B7C0]
00433461  |>mov dword ptr ss:[ebp-14],eax
00433464  |>mov ecx,dword ptr ss:[ebp-8]
00433467  |>and ecx,0FF
0043346D  |>mov edx,dword ptr ss:[ebp-4]
00433470  |>sub edx,dword ptr ds:[ecx*4+45B3C0]
00433477  |>mov dword ptr ss:[ebp-4],edx
0043347A  |>mov eax,dword ptr ss:[ebp-10]
0043347D  |>rol eax,10
00433480  |>mov dword ptr ss:[ebp-8],eax
00433483  |>mov ecx,dword ptr ss:[ebp-10]
00433486  |>rol ecx,18
00433489  |>mov dword ptr ss:[ebp-10],ecx
0043348C  |>mov edx,dword ptr ss:[ebp-8]
0043348F  |>and edx,0FF
00433495  |>mov eax,dword ptr ss:[ebp-C]
00433498  |>sub eax,dword ptr ds:[edx*4+45B7C0]
0043349F  |>mov dword ptr ss:[ebp-C],eax
004334A2  |>mov ecx,dword ptr ss:[ebp-10]
004334A5  |>and ecx,0FF
004334AB  |>mov edx,dword ptr ss:[ebp-C]
004334AE  |>xor edx,dword ptr ds:[ecx*4+45B3C0]
004334B5  |>mov dword ptr ss:[ebp-C],edx//////////////////////////4
```

```
004334B8  |>mov eax,dword ptr ss:[ebp-14]
004334BB  |>sub eax,dword ptr ss:[ebp-4]
004334BE  |>mov dword ptr ss:[ebp-14],eax
004334C1  |>mov ecx,dword ptr ss:[ebp-14]
004334C4  |>rol ecx,8
004334C7  |>mov dword ptr ss:[ebp-8],ecx
004334CA  |>mov edx,dword ptr ss:[ebp-14]
004334CD  |>and edx,0FF
004334D3  |>mov eax,dword ptr ss:[ebp-4]
004334D6  |>xor eax,dword ptr ds:[edx*4+45B7C0]
004334DD  |>mov dword ptr ss:[ebp-4],eax
004334E0  |>mov ecx,dword ptr ss:[ebp-8]
004334E3  |>and ecx,0FF
004334E9  |>mov edx,dword ptr ss:[ebp-C]
004334EC  |>sub edx,dword ptr ds:[ecx*4+45B3C0]
004334F3  |>mov dword ptr ss:[ebp-C],edx
004334F6  |>mov eax,dword ptr ss:[ebp-14]
004334F9  |>rol eax,10
004334FC  |>mov dword ptr ss:[ebp-8],eax
004334FF  |>mov ecx,dword ptr ss:[ebp-14]
00433502  |>rol ecx,18
00433505  |>mov dword ptr ss:[ebp-14],ecx
00433508  |>mov edx,dword ptr ss:[ebp-8]
```
