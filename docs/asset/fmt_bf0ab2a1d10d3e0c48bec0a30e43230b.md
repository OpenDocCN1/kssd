
```text
0040F3DE    8B4C24 04        mov     ecx, dword ptr [esp+4]
0040F3E2    83C4 08          add     esp, 8
0040F3E5    61             popad
0040F3E6    53             push    ebx
0040F3E7    890424         mov     dword ptr [esp], eax
0040F3EA    8BC4           mov     eax, esp
0040F3EC    8B00           mov     eax, dword ptr [eax]
0040F3EE    83C4 04        add     esp, 4
0040F3F1    83EC 04        sub     esp, 4
0040F3F4    C3             retn  //这个地方终于出现了大跳转,转到 0x0040A71C 处指令
```

```text
0040A71C    1BC5           sbb     eax, ebp
0040A71E    40             inc     eax
0040A71F    E8 E4490000    call    0040F108  //这里用 F8，这个函数修改 0x0040A724 以下的代码
```

```text
0040A724    9C             pushfd
0040A725    60             pushad
0040A726    33D2           xor     edx, edx
0040A728    66:8CE2        mov     dx, fs
0040A72B    0F03D2         lsl     edx, edx
0040A72E    66:8CE3        mov     bx, fs
0040A731    66:3BD3        cmp     dx, bx
0040A734    75 03          jnz     short 0040A739
0040A736    6A 00          push    0
0040A738    C3             retn
0040A739    E8 00000000    call    0040A73E
0040A73E    5D             pop     ebp
0040A73F    B8 F0110000    mov     eax, 11F0
0040A744    2D D6110000    sub     eax, 11D6
0040A749    2BE8           sub     ebp, eax
0040A74B    8DB5 24FDFFFF  lea     esi, dword ptr [ebp-2DC]
0040A751    8B06           mov     eax, dword ptr [esi]
0040A753    83F8 00        cmp     eax, 0
0040A756    74 17         je      short 0040A76F
0040A758    8DB5 30FDFFFF  lea     esi, dword ptr [ebp-2D0]
0040A75E    8B06           mov     eax, dword ptr [esi]
0040A760    83F8 01        cmp     eax, 1
0040A763    C706 01000000  mov     dword ptr [esi], 1
0040A769    0F84 4A030000  je      0040AAB9
0040A76F    60             pushad
0040A770    E8 00000000    call    0040A775
0040A775    5A             pop     edx
0040A776    B8 59120000    mov     eax, 1259
0040A77B    2D 27120000    sub     eax, 1227
0040A780    03C2           add     eax, edx
0040A782    50             push    eax      //安装 SEH 异常处理项，回调函数地址是 0x0040a7a7
0040A783    33D2           xor     edx, edx
0040A785    64:FF32        push    dword ptr fs:[edx]
0040A788    64:8922        mov     dword ptr fs:[edx], esp
0040A78B    F7F2           div     edx      //除 0 异常，这里 bp 0x0040a7a7(在回调函数入口处下断点),注意不能单步到这里,单步到它上面一个指令吧，因为 SEH 出来后第一条指令就在卸载 SEH 项了，如果这里单步了会使 TF=1，异常处理后会单步异常，但这个时候已经没有 SEH 项了,这是跟其它壳不同的
```

除 0 异常后，来到这里，这里获取异常时上下文环境的方法，有点与众不同

```text
0040A7A7    53             push    ebx
0040A7A8    E8 00000000    call    0040A7AD
0040A7AD    5B             pop     ebx
0040A7AE    B8 5F120000    mov     eax, 125F
0040A7B3    2D 4C120000    sub     eax, 124C
0040A7B8    2BD8           sub     ebx, eax
0040A7BA    8B45 10        mov     eax, dword ptr [ebp+10]  //通过 ebp + 10 来获取上下文，这里没有用回调函数的参数
0040A7BD    8998 B8000000  mov     dword ptr [eax+B8], ebx  //异常处理结束后要执行的地址，这里 bp ebx(0x0040a79a)吧
0040A7C3    33DB           xor     ebx, ebx
0040A7C5    8958 04        mov     dword ptr [eax+4], ebx   //清除硬件断点
0040A7C8    8958 08        mov     dword ptr [eax+8], ebx
0040A7CB    8958 0C        mov     dword ptr [eax+C], ebx
0040A7CE    8958 10        mov     dword ptr [eax+10], ebx
0040A7D1    C740 18 5501000> mov    dword ptr [eax+18], 155
0040A7D8    B8 00000000    mov     eax, 0
0040A7DD    5B             pop     ebx
0040A7DE    C2 0A00        retn    0A     //这里返回也与众不同，不是用 retn,而用了带操作数的 retn
```

异常处理结束后，到这里

```text
0040A79A    64:8F05 0000000> pop     dword ptr fs:[0]
0040A7A1    83C4 04        add     esp, 4
0040A7A4    61             popad
0040A7A5    EB 3A          jmp     short 0040A7E1
```

```text
0040A7E1    8BD5           mov     edx, ebp
0040A7E3    8B85 E4FCFFFF  mov     eax, dword ptr [ebp-31C]
0040A7E9    2BD0           sub     edx, eax
0040A7EB    8995 E4FCFFFF  mov     dword ptr [ebp-31C], edx
0040A7F1    0195 18FDFFFF  add     dword ptr [ebp-2E8], edx
0040A7F7    8DB5 34FDFFFF  lea     esi, dword ptr [ebp-2CC]
0040A7FD    0116           add     dword ptr [esi], edx
0040A7FF    8B36           mov     esi, dword ptr [esi]
0040A801    8BFD           mov     edi, ebp
0040A803    52             push    edx
0040A804    55             push    ebp
0040A805    57             push    edi
0040A806    6A 40          push    40
0040A808    68 00100000    push    1000
0040A80D    68 00A00000    push    0A000
0040A812    6A 00          push    0
0040A814    FF95 54FDFFFF  call    dword ptr [ebp-2AC]
0040A81A    85C0           test    eax, eax
0040A81C    0F84 0D050000  je      0040AD2F
0040A822    8985 F8FCFFFF  mov     dword ptr [ebp-308], eax
0040A828    E8 00000000    call    0040A82D    //这里用 F8
0040A82D    5B             pop     ebx
0040A82E    B9 04180000    mov     ecx, 1804
0040A833    81E9 DF120000  sub     ecx, 12DF
0040A839    03D9           add     ebx, ecx
0040A83B    50             push    eax
0040A83C    50             push    eax
0040A83D    53             push    ebx
0040A83E    E8 43040000    call    0040AC86
0040A843    58             pop     eax
0040A844    E9 00000000    jmp     0040A849
0040A849    60             pushad
0040A84A    05 A50A0000    add     eax, 0AA5
0040A84F    50             push    eax
0040A850    8BF0           mov     esi, eax
0040A852    83C6 48        add     esi, 48
0040A855    8B36           mov     esi, dword ptr [esi]
0040A857    03F0           add     esi, eax
0040A859    8BC8           mov     ecx, eax
0040A85B    83C1 4C        add     ecx, 4C
0040A85E    8B09           mov     ecx, dword ptr [ecx]
0040A860    50             push    eax
0040A861    56             push    esi
0040A862    51             push    ecx
0040A863    56             push    esi
0040A864    FF95 5CFDFFFF  call    dword ptr [ebp-2A4]         ; kernel32.GetSystemDirectoryA
0040A86A    5F             pop     edi
0040A86B    58             pop     eax
0040A86C    50             push    eax
0040A86D    57             push    edi
0040A86E    8BD7           mov     edx, edi
0040A870    E8 C3040000    call    0040AD38  //计算系统目录字符串长度
0040A875    03F8           add     edi, eax
0040A877    5E             pop     esi
0040A878    8BC6           mov     eax, esi
0040A87A    83C6 40        add     esi, 40
0040A87D    8B36           mov     esi, dword ptr [esi]
0040A87F    03F0           add     esi, eax
0040A881    50             push    eax
0040A882    56             push    esi
0040A883    56             push    esi
0040A884    E8 AF040000    call    0040AD38  //计算"ya.dll"长度
0040A889    8BC8           mov     ecx, eax
0040A88B    5E             pop     esi
0040A88C    58             pop     eax
0040A88D    F3:A4          rep     movs byte ptr es:[edi], byte ptr>
0040A88F    C607 00        mov     byte ptr [edi], 0
0040A892    8BFA           mov     edi, edx
0040A894    50             push    eax
0040A895    57             push    edi
0040A896    6A 00          push    0
0040A898    6A 02          push    2
0040A89A    6A 02          push    2
0040A89C    6A 00          push    0
0040A89E    6A 03          push    3
```
