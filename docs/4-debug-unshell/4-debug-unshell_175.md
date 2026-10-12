
# anti007 v2.5 脱壳【原创】

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-516.htm`](https://www.pediy.com/kssd/pediy09/pediy09-516.htm)

> 【 标题 】 anti007 v2.5 脱壳

> 【 作者 】 linxer

> 【 Q Q 】 3568599

> 【破解平台】 盗版 xp sp2

> 【脱壳工具】 OllyICE + OllyDump 插件

> 【待脱软件】 见附件

> 【 声明 】 初学脱壳，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

这款壳，脱法很简单，不过，在网上搜了下，貌似还没人写这个脱文，这里就粗略写一篇出来吧，并给出函数导出部分的详细分析

`OD` 加载程序，程序停留在入口

这片代码，基本都是顺序的，没啥说的，一直往下走

```assembly
0040F20C >  57               push    edi                     ; ntdll.7C930738
0040F20D     56               push    esi
0040F20E     33F7             xor     esi, edi
0040F210     5E               pop     esi
0040F211     46               inc     esi
0040F212     4E               dec     esi
0040F213     33FE             xor     edi, esi
0040F215     83C4 08          add     esp, 8
0040F218     83EC 04          sub     esp, 4
0040F21B     8B7C24 FC        mov     edi, dword ptr [esp-4]
0040F21F     60               pushad
0040F220     56               push    esi
0040F221     BE 5028122C      mov     esi, 2C122850
0040F226     81C6 B0D7EDD3    add     esi, D3EDD7B0
0040F22C     03FE             add     edi, esi
0040F22E     5E               pop     esi
0040F22F     57               push    edi
0040F230     68 00000000      push    0
0040F235     5F               pop     edi
0040F236     293C24           sub     dword ptr [esp], edi
0040F239     5F               pop     edi
0040F23A     E8 00000000      call    0040F23F
0040F23F     8BDC             mov     ebx, esp
0040F241     8B1B             mov     ebx, dword ptr [ebx]
0040F243     83C4 04          add     esp, 4
0040F246     56               push    esi
0040F247     56               push    esi
0040F248     83C4 04          add     esp, 4
0040F24B     891424           mov     dword ptr [esp], edx
0040F24E     56               push    esi
0040F24F     8BD4             mov     edx, esp
0040F251     C702 00000000    mov     dword ptr [edx], 0
0040F257     5A               pop     edx
0040F258     52               push    edx
0040F259     68 33000000      push    33
0040F25E     5A               pop     edx
0040F25F     311424           xor     dword ptr [esp], edx
0040F262     5A               pop     edx
0040F263     52               push    edx
0040F264     81D2 FA0F5745   adc     edx, 45570FFA
0040F26A     2B1C24           sub     ebx, dword ptr [esp]
0040F26D     5A               pop     edx
0040F26E     83C4 04          add     esp, 4
0040F271     8B5424 FC        mov     edx, dword ptr [esp-4]
0040F275     83EC 04          sub     esp, 4
0040F278     891C24           mov     dword ptr [esp], ebx
0040F27B     5A               pop     edx
0040F27C     51               push    ecx
0040F27D     891C24           mov     dword ptr [esp], ebx
0040F280     57               push    edi
0040F281     8BDC             mov     ebx, esp
0040F283     C703 00000000    mov     dword ptr [ebx], 0
0040F289     5B               pop     ebx
0040F28A     53               push    ebx
0040F28B     812C24 F04A0000  sub     dword ptr [esp], 4AF0
0040F292     5B               pop     ebx
0040F293     53               push    ebx
0040F294     03D3             add     edx, ebx
0040F296     03D8             add     ebx, eax
0040F298     5B               pop     ebx
0040F299     83C4 04          add     esp, 4
0040F29C     8B5C24 FC        mov     ebx, dword ptr [esp-4]
0040F2A0     51               push    ecx
0040F2A1     891424           mov     dword ptr [esp], edx
0040F2A4     81C7 D17AB04A   add     edi, 4AB07AD1
0040F2AA     83EC 04          sub     esp, 4
0040F2AD     81EF D17AB04A   sub     edi, 4AB07AD1
0040F2B3     893C24           mov     dword ptr [esp], edi
0040F2B6     47               inc     edi
0040F2B7     4F               dec     edi
0040F2B8     57               push    edi
0040F2B9     BF 0A423402      mov     edi, 234420A
0040F2BE     81F7 0A423402   xor     edi, 234420A
0040F2C4     83C4 04          add     esp, 4
0040F2C7     90               nop
0040F2C8     F7D7             not     edi
0040F2CA     68 00000000      push    0
0040F2CF     8B3C24           mov     edi, dword ptr [esp]
0040F2D2     83C4 04          add     esp, 4
0040F2D5     57               push    edi
0040F2D6     893C24           mov     dword ptr [esp], edi
0040F2D9     031424           add     edx, dword ptr [esp]
0040F2DC     83C4 04          add     esp, 4
0040F2DF     E8 00000000      call    0040F2E4
0040F2E4     83C4 04          add     esp, 4
0040F2E7     8B3C24           mov     edi, dword ptr [esp]
0040F2EA     83C4 04          add     esp, 4
0040F2ED     50               push    eax
0040F2EE     9B               wait
0040F2EF     DFE0             fstsw   ax
0040F2F1     58               pop     eax
0040F2F2     3BDA             cmp     ebx, edx
0040F2F4     F5               cmc
0040F2F5     F5               cmc
0040F2F6     0F85 0B000000   jnz     0040F307
0040F2FC     57               push    edi
0040F2FD     33FE             xor     edi, esi
0040F2FF     5F               pop     edi
0040F300     E9 50000000      jmp     0040F355  //到 0x0040F355 这个地方 F4 吧
```

以下代码还是顺序关系

```assembly
0040F355     8B0C24           mov     ecx, dword ptr [esp]
0040F358     83C4 04          add     esp, 4
0040F35B     8BDC             mov     ebx, esp
0040F35D     81C3 00000000   add     ebx, 0
0040F363     57               push    edi
0040F364     53               push    ebx
0040F365     5F               pop     edi
0040F366     81C7 00000000   add     edi, 0
0040F36C     8BDF             mov     ebx, edi
0040F36E     5F               pop     edi
0040F36F     53               push    ebx
0040F370     51               push    ecx
0040F371     B9 242A8410      mov     ecx, 10842A24
0040F376     81F1 382A8410   xor     ecx, 10842A38
0040F37C     49               dec     ecx
0040F37D     03D9             add     ebx, ecx
0040F37F     59               pop     ecx
0040F380     43               inc     ebx
0040F381     890B             mov     dword ptr [ebx], ecx
0040F383     5B               pop     ebx
0040F384     51               push    ecx
0040F385     57               push    edi
0040F386     5F               pop     edi
0040F387     56               push    esi
0040F388     D3CE             ror     esi, cl
0040F38A     5E               pop     esi
0040F38B     E8 00000000      call    0040F390
0040F390     EB 01            jmp     short 0040F393  //出现花指令
0040F392     E8 83EC0483      call    8345E01A
```

还是顺序的

```assembly
0040F393     83EC 04          sub     esp, 4
0040F396     83EC 04          sub     esp, 4
0040F399     8B4C24 04        mov     ecx, dword ptr [esp+4]
0040F39D     83C4 08          add     esp, 8
0040F3A0     83EC 04          sub     esp, 4
0040F3A3     59               pop     ecx
0040F3A4     59               pop     ecx
0040F3A5     81E9 FEFFC5FB   sub     ecx, FBC5FFFE
0040F3AB     81E9 02003A04   sub     ecx, 43A0002
0040F3B1     81E9 EAE77DDF   sub     ecx, DF7DE7EA
0040F3B7     81E9 FEEFCDF7   sub     ecx, F7CDEFFE
0040F3BD     81E9 02103208   sub     ecx, 8321002
0040F3C3     81E9 16188220   sub     ecx, 20821816
0040F3C9     57               push    edi
0040F3CA     BF 44788A0C      mov     edi, 0C8A7844
0040F3CF     81C7 558875F3   add     edi, F3758855
0040F3D5     2BCF             sub     ecx, edi
0040F3D7     5F               pop     edi
0040F3D8     C601 84          mov     byte ptr [ecx], 84
0040F3DB     83EC 04          sub     esp, 4
```



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



```
0040A8A0    68 000000C0     push    C0000000  

0040A8A5    57             push    edi  

0040A8A6    FF95 60FDFFFF  call    dword ptr [ebp-2A0]  ; kernel32.CreateFileA,释放文件 ya.dll 系统目录下  

0040A8AC    5F             pop     edi  

0040A8AD    5E             pop     esi  

0040A8AE    83F8 FF       cmp     eax, -1  

0040A8B1    74 29         je      short 0040A8DC  

0040A8B3    8BD8          mov     ebx, eax  

0040A8B5    57             push    edi  

0040A8B6    53             push    ebx  

0040A8B7    56             push    esi  

0040A8B8    8BC6          mov     eax, esi  

0040A8BA    8BD0          mov     edx, eax  

0040A8BC    83C2 44      add     edx, 44  

0040A8BF    8B0A          mov     ecx, dword ptr [edx]  

0040A8C1    6A 00         push    0  

0040A8C3    52             push    edx  

0040A8C4    51             push    ecx  

0040A8C5    50             push    eax  

0040A8C6    53             push    ebx  

0040A8C7    FF95 64FDFFFF  call    dword ptr [ebp-29C]  ; kernel32.WriteFile,随便写点数据到 ya.dll  

0040A8CD    5E             pop     esi  

0040A8CE    5B             pop     ebx  

0040A8CF    5F             pop     edi  

0040A8D0    83F8 00      cmp     eax, 0  

0040A8D3    74 0E         je      short 0040A8E3  

0040A8D5    53             push    ebx  

0040A8D6    FF95 68FDFFFF  call    dword ptr [ebp-298]  //调用 CloseHandle  

0040A8DC    57             push    edi  

0040A8DD    FF95 48FDFFFF  call    dword ptr [ebp-2B8]  //LoadLibraryA ya.dll,这里肯定是失败的,这个地方不能单步调试,这里利用了 LoadLibraryA 加载 ya.dll 在单步时,异常无法调试  

0040A8E3    58             pop     eax  

0040A8E4    61             popad  

0040A8E5    5F             pop     edi  

0040A8E6    5D             pop     ebp  

0040A8E7    5A             pop     edx  

0040A8E8    03BD DCFCFFFF  add     edi, dword ptr [ebp-324]  

0040A8EE    60             pushad  

0040A8EF    8DB5 FCFCFFFF  lea     esi, dword ptr [ebp-304]  

0040A8F5    E8 00000000    call    0040A8FA    //F8  

0040A8FA    5B             pop     ebx  

0040A8FB    B9 D6130000    mov     ecx, 13D6  

0040A900    81E9 AC130000  sub     ecx, 13AC  

0040A906    03D9          add     ebx, ecx  

0040A908    8B0E          mov     ecx, dword ptr [esi]  

0040A90A    83C6 04      add     esi, 4  

0040A90D    833E 00      cmp     dword ptr [esi], 0  

0040A910    74 1C         je      short 0040A92E  

0040A912    0306          add     eax, dword ptr [esi]  

0040A914    56             push    esi  

0040A915    51             push    ecx  

0040A916    50             push    eax  

0040A917    53             push    ebx  

0040A918    57             push    edi  

0040A919    53             push    ebx  

0040A91A    51             push    ecx  

0040A91B    57             push    edi  

0040A91C    57             push    edi  

0040A91D    68 08000000    push    8  

0040A922    FFD0          call    eax     //F8  

0040A924    5F             pop     edi  

0040A925    5B             pop     ebx  

0040A926    58             pop     eax  

0040A927    59             pop     ecx  

0040A928    5E             pop     esi  

0040A929    83C6 04      add     esi, 4  

0040A92C   ^ EB DF        jmp     short 0040A90D  

0040A92E    61             popad     //这里 F4 吧  

0040A92F    8BDF          mov     ebx, edi  

0040A931    833F 00      cmp     dword ptr [edi], 0  

0040A934    75 0A         jnz     short 0040A940  

0040A936    83C7 04      add     edi, 4  

0040A939    B9 00000000    mov     ecx, 0  

0040A93E    EB 16         jmp     short 0040A956  

0040A940    B9 01000000    mov     ecx, 1  

0040A945    033B          add     edi, dword ptr [ebx]  

0040A947    83C3 04      add     ebx, 4  

0040A94A    833B 00      cmp     dword ptr [ebx], 0  

0040A94D    74 2D         je      short 0040A97C  

0040A94F    0113          add     dword ptr [ebx], edx  

0040A951    8B33          mov     esi, dword ptr [ebx]  

0040A953    037B 04      add     edi, dword ptr [ebx+4]  

0040A956    57             push    edi  

0040A957    51             push    ecx  

0040A958    52             push    edx  

0040A959    53             push    ebx  

0040A95A    FFB5 58FDFFFF  push    dword ptr [ebp-2A8]  

0040A960    FFB5 54FDFFFF  push    dword ptr [ebp-2AC]  

0040A966    56             push    esi  

0040A967    57             push    edi  

0040A968    FF95 F8FCFFFF  call    dword ptr [ebp-308]  //这里 F8 吧，这里是漫长的解码过程  

0040A96E    5B             pop     ebx  

0040A96F    5A             pop     edx  

0040A970    59             pop     ecx  

0040A971    5F             pop     edi  

0040A972    83F9 00      cmp     ecx, 0  

0040A975    74 05         je      short 0040A97C  

0040A977    83C3 08      add     ebx, 8  

0040A97A   ^ EB CE        jmp     short 0040A94A  

0040A97C    68 00800000    push    8000  

0040A981    6A 00         push    0  

0040A983    FFB5 F8FCFFFF  push    dword ptr [ebp-308]  

0040A989    FF95 58FDFFFF  call    dword ptr [ebp-2A8]  //调用 VirtualFree  

0040A98F    8DB5 18FDFFFF  lea     esi, dword ptr [ebp-2E8]  

0040A995    8B4E 04      mov     ecx, dword ptr [esi+4]  

0040A998    8D56 08      lea     edx, dword ptr [esi+8]  

0040A99B    8B36          mov     esi, dword ptr [esi]  

0040A99D    8BFE          mov     edi, esi  

0040A99F    83F9 00      cmp     ecx, 0  

0040A9A2    74 3F         je      short 0040A9E3  

0040A9A4    8A07          mov     al, byte ptr [edi]  

0040A9A6    47             inc     edi  

0040A9A7    2C E8         sub     al, 0E8  

0040A9A9    3C 01         cmp     al, 1  

0040A9AB   ^ 77 F7        ja      short 0040A9A4  

0040A9AD    8B07          mov     eax, dword ptr [edi]  

0040A9AF    807A 01 00    cmp     byte ptr [edx+1], 0  

0040A9B3    74 14         je      short 0040A9C9  

0040A9B5    8A1A          mov     bl, byte ptr [edx]  

0040A9B7    381F          cmp     byte ptr [edi], bl  

0040A9B9   ^ 75 E9        jnz     short 0040A9A4  

0040A9BB    8A5F 04      mov     bl, byte ptr [edi+4]  

0040A9BE    66:C1E8 08    shr     ax, 8  

0040A9C2    C1C0 10      rol     eax, 10  

0040A9C5    86C4          xchg    ah, al  

0040A9C7    EB 0A         jmp     short 0040A9D3  

0040A9C9    8A5F 04      mov     bl, byte ptr [edi+4]  

0040A9CC    86C4          xchg    ah, al  

0040A9CE    C1C0 10      rol     eax, 10  

0040A9D1    86C4          xchg    ah, al  

0040A9D3    2BC7          sub     eax, edi  

0040A9D5    03C6          add     eax, esi  

0040A9D7    8907          mov     dword ptr [edi], eax  

0040A9D9    83C7 05      add     edi, 5  

0040A9DC    80EB E8      sub     bl, 0E8  

0040A9DF    8BC3          mov     eax, ebx  

0040A9E1   ^ E2 C6        loopd   short 0040A9A9  

0040A9E3    E8 D8000000    call    0040AAC0    //到这里 F4，这个函数在处理 IAT，当然这里可以 F8 后到 0040A9E8  
```

IAT 处理函数，并加密函数名称

```
0040AAC0    8BB5 E0FCFFFF  mov     esi, dword ptr [ebp-320]  

0040AAC6    0BF6          or      esi, esi  

0040AAC8    0F84 A4000000 je      0040AB72  

0040AACE    8B95 E4FCFFFF  mov     edx, dword ptr [ebp-31C]  

0040AAD4    03F2          add     esi, edx  

0040AAD6    833E 00      cmp     dword ptr [esi], 0  

0040AAD9    75 11         jnz     short 0040AAEC  

0040AADB    837E 04 00    cmp     dword ptr [esi+4], 0  

0040AADF    75 0B         jnz     short 0040AAEC  

0040AAE1    837E 08 00    cmp     dword ptr [esi+8], 0  
```



# 汇编代码段 1

```
0040AAE5    75 05              jnz     short 0040AAEC  

0040AAE7    E9 84000000        jmp     0040AB70  

0040AAEC    8B5E 08            mov     ebx, dword ptr [esi+8]  

0040AAEF    03DA               add     ebx, edx  

0040AAF1    53                 push    ebx  

0040AAF2    52                 push    edx  

0040AAF3    56                 push    esi  

0040AAF4    8DBD F0FDFFFF      lea     edi, dword ptr [ebp-210]  

0040AAFA    037E 04            add     edi, dword ptr [esi+4]  

0040AAFD    83C6 0C            add     esi, 0C  

0040AB00    57                 push    edi  

0040AB01    FF95 48FDFFFF      call    dword ptr [ebp-2B8]   //调用 LoadLibraryA  

0040AB07    5F                 pop     edi  

0040AB08    5A                 pop     edx  

0040AB09    5B                 pop     ebx  

0040AB0A    83F8 00            cmp     eax, 0  

0040AB0D    74 63              je      short 0040AB72  

0040AB0F    8985 E8FCFFFF      mov     dword ptr [ebp-318], eax  

0040AB15    033E               add     edi, dword ptr [esi]  

0040AB17    83C6 04            add     esi, 4  

0040AB1A    33C9               xor     ecx, ecx  

0040AB1C    8A0E               mov     cl, byte ptr [esi]  

0040AB1E    83F9 00            cmp     ecx, 0  

0040AB21    75 03              jnz     short 0040AB26  

0040AB23    46                 inc     esi  

0040AB24    ^ EB B0            jmp     short 0040AAD6  

0040AB26    8BC7               mov     eax, edi  

0040AB28    03F9               add     edi, ecx  

0040AB2A    52                 push    edx  

0040AB2B    53                 push    ebx  

0040AB2C    50                 push    eax  

0040AB2D    8038 FF            cmp     byte ptr [eax], 0FF  

0040AB30    75 03              jnz     short 0040AB35  

0040AB32    40                 inc     eax  

0040AB33    8B00               mov     eax, dword ptr [eax]  

0040AB35    8A0F               mov     cl, byte ptr [edi]  

0040AB37    C607 00            mov     byte ptr [edi], 0  

0040AB3A    51                 push    ecx  

0040AB3B    FFB5 4CFDFFFF      push    dword ptr [ebp-2B4]  

0040AB41    50                 push    eax  

0040AB42    FFB5 E8FCFFFF      push    dword ptr [ebp-318]  

0040AB48    E8 2A000000        call    0040AB77     //导出函数，详细分析见下面  

0040AB4D    83C4 0C            add     esp, 0C  

0040AB50    59                 pop     ecx  

0040AB51    5A                 pop     edx  

0040AB52    92                 xchg    eax, edx  

0040AB53    E8 17010000        call    0040AC6F     //这里加密刚导出的函数的函数名，异或 0x4c  

0040AB58    92                 xchg    eax, edx  

0040AB59    5B                 pop     ebx  

0040AB5A    5A                 pop     edx  

0040AB5B    83F8 00            cmp     eax, 0  

0040AB5E    74 12              je      short 0040AB72  

0040AB60    880F               mov     byte ptr [edi], cl  

0040AB62    8946 FC            mov     dword ptr [esi-4], eax  

0040AB65    FF76 FC            push    dword ptr [esi-4]  

0040AB68    8F03               pop     dword ptr [ebx]  

0040AB6A    83C3 04            add     ebx, 4  

0040AB6D    46                 inc     esi  

0040AB6E    ^ EB AA            jmp     short 0040AB1A  

0040AB70    F8                 clc  

0040AB71    C3                 retn      //这个函数从这里返回  
```

## 说明

IAT 处理完后，到这里，这里马上就要到达 `OEP` 了，一路 `F7` 下去吧

# 汇编代码段 2

```
0040A9E8    8D8D 24FDFFFF      lea     ecx, dword ptr [ebp-2DC]  

0040A9EE    8B41 04            mov     eax, dword ptr [ecx+4]  

0040A9F1    83F8 00            cmp     eax, 0  

0040A9F4    0F84 81000000      je      0040AA7B  

0040A9FA    8BF2               mov     esi, edx  

0040A9FC    2B71 08            sub     esi, dword ptr [ecx+8]  

0040A9FF    74 7A              je      short 0040AA7B  

0040AA01    8971 08            mov     dword ptr [ecx+8], esi  

0040AA04    8B01               mov     eax, dword ptr [ecx]  

0040AA06    8DB5 34FDFFFF      lea     esi, dword ptr [ebp-2CC]  

0040AA0C    8B36               mov     esi, dword ptr [esi]  

0040AA0E    8D5E FC            lea     ebx, dword ptr [esi-4]  

0040AA11    83F8 01            cmp     eax, 1  

0040AA14    74 0A              je      short 0040AA20  

0040AA16    8BFA               mov     edi, edx  

0040AA18    0379 04            add     edi, dword ptr [ecx+4]  

0040AA1B    8B49 08            mov     ecx, dword ptr [ecx+8]  

0040AA1E    EB 08              jmp     short 0040AA28  

0040AA20    8BFE               mov     edi, esi  

0040AA22    0379 04            add     edi, dword ptr [ecx+4]  

0040AA25    8B49 08            mov     ecx, dword ptr [ecx+8]  

0040AA28    33C0               xor     eax, eax  

0040AA2A    8A07               mov     al, byte ptr [edi]  

0040AA2C    47                 inc     edi  

0040AA2D    0BC0               or      eax, eax  

0040AA2F    74 20              je      short 0040AA51  

0040AA31    3C EF              cmp     al, 0EF  

0040AA33    77 06              ja      short 0040AA3B  

0040AA35    03D8               add     ebx, eax  

0040AA37    010B               add     dword ptr [ebx], ecx  

0040AA39    ^ EB ED            jmp     short 0040AA28  

0040AA3B    24 0F              and     al, 0F  

0040AA3D    C1E0 10            shl     eax, 10  

0040AA40    66:8B07            mov     ax, word ptr [edi]  

0040AA43    83C7 02            add     edi, 2  

0040AA46    0BC0               or      eax, eax  

0040AA48    ^ 75 EB            jnz     short 0040AA35  

0040AA4A    8B07               mov     eax, dword ptr [edi]  

0040AA4C    83C7 04            add     edi, 4  

0040AA4F    ^ EB E4            jmp     short 0040AA35  

0040AA51    33DB               xor     ebx, ebx  

0040AA53    87FE               xchg    esi, edi  

0040AA55    8B06               mov     eax, dword ptr [esi]  

0040AA57    83F8 00            cmp     eax, 0  

0040AA5A    74 1F              je      short 0040AA7B  

0040AA5C    AD                 lods    dword ptr [esi]  

0040AA5D    0BC0               or      eax, eax  

0040AA5F    74 08              je      short 0040AA69  

0040AA61    03D8               add     ebx, eax  

0040AA63    66:010C3B          add     word ptr [ebx+edi], cx  

0040AA67    ^ EB F3            jmp     short 0040AA5C  

0040AA69    33DB               xor     ebx, ebx  

0040AA6B    C1E9 10            shr     ecx, 10  

0040AA6E    AD                 lods    dword ptr [esi]  

0040AA6F    0BC0               or      eax, eax  

0040AA71    74 08              je      short 0040AA7B  

0040AA73    03D8               add     ebx, eax  

0040AA75    66:010C3B          add     word ptr [ebx+edi], cx  

0040AA79    ^ EB F3            jmp     short 0040AA6E  

0040AA7B    8BDD               mov     ebx, ebp  

0040AA7D    81EB 21010000      sub     ebx, 121  

0040AA83    33C9               xor     ecx, ecx  

0040AA85    8A0B               mov     cl, byte ptr [ebx]  

0040AA87    83F9 00            cmp     ecx, 0  

0040AA8A    74 2D              je      short 0040AAB9  

0040AA8C    43                 inc     ebx  

0040AA8D    8DB5 E4FCFFFF      lea     esi, dword ptr [ebp-31C]  

0040AA93    8B16               mov     edx, dword ptr [esi]  

0040AA95    56                 push    esi  

0040AA96    51                 push    ecx  

0040AA97    53                 push    ebx  

0040AA98    52                 push    edx  

0040AA99    56                 push    esi  

0040AA9A    FF33               push    dword ptr [ebx]  

0040AA9C    FF73 04            push    dword ptr [ebx+4]  

0040AA9F    8B43 08            mov     eax, dword ptr [ebx+8]  

0040AAA2    03C2               add     eax, edx  

0040AAA4    50                 push    eax  

0040AAA5    FF95 50FDFFFF      call    dword ptr [ebp-2B0]  

0040AAAB    5A                 pop     edx  

0040AAAC    5B                 pop     ebx  

0040AAAD    59                 pop     ecx  

0040AAAE    5E                 pop     esi  

0040AAAF    83C3 0C            add     ebx, 0C  

0040AAB2    ^ E2 E1            loopd   short 0040AA95  
```



```asm
0040AAB4    E8 7E020000     call    0040AD37  
0040AAB9    61              popad  
0040AABA    9D              popfd  
0040AABB  - E9 3071FFFF     jmp     00401BF0  //这里到达 OEP 了
```

到这里，这个壳就脱完了

总结下这个壳的脱法：
1. `bp VirtualFree`, F9, `bc VirtualFree`
2. `hr 0x0012ffc0`, F9, `hd 0x0012ffc0`,这里就可以用硬件断点了，因为除 0 异常在 `VirtualFree` 前
3. F7
4. dump 收工

闲来给出取 `kernel32.dll` 中函数地址函数详细分析，加深对导出表的理解，呵呵

```asm
0040AB77    55              push    ebp  
0040AB78    8BEC            mov     ebp, esp  
0040AB7A    83EC 14         sub     esp, 14  
0040AB7D    8B45 0C         mov     eax, dword ptr [ebp+C]  
0040AB80    A9 00000080     test    eax, 80000000  
0040AB85    74 0D          je      short 0040AB94  
0040AB87    8BD0            mov     edx, eax  
0040AB89    81E2 FFFFFFFF  and     edx, 7FFFFFFF    //如果用序号来导出的 edx=函数序号  
0040AB8F    8955 F4         mov     dword ptr [ebp-C], edx  
0040AB92    EB 05          jmp     short 0040AB99  
0040AB94    33D2            xor     edx, edx        //如果用函数名来导出的 edx=0  
0040AB96    8945 F4         mov     dword ptr [ebp-C], eax  
0040AB99    53              push    ebx  
0040AB9A    8B5D 08         mov     ebx, dword ptr [ebp+8]  //ebx=kernel32.dll 基地  
0040AB9D    56              push    esi  
0040AB9E    8365 FC 00      and     dword ptr [ebp-4], 0  
0040ABA2    8B43 3C         mov     eax, dword ptr [ebx+3C]  //eax=PE 头标志偏移  
0040ABA5    57              push    edi  
0040ABA6    8B7418 78       mov     esi, dword ptr [eax+ebx+78]  //esi=导出表偏移  
0040ABAA    8D4418 18       lea     eax, dword ptr [eax+ebx+18]  //eax=OptionalHeader 偏移  
0040ABAE    03F3            add     esi, ebx  
0040ABB0    8B40 64         mov     eax, dword ptr [eax+64]  //eax=导出表大小  
0040ABB3    8B7E 1C         mov     edi, dword ptr [esi+1C]  //edi=导出函数地址表偏移  
0040ABB6    8B4E 20         mov     ecx, dword ptr [esi+20]  //ecx=导出函数名地址表偏移  
0040ABB9    8945 EC         mov     dword ptr [ebp-14], eax  
0040ABBC    8B46 24         mov     eax, dword ptr [esi+24]  //eax=导出函数序号表偏移  
0040ABBF    03FB            add     edi, ebx  
0040ABC1    03CB            add     ecx, ebx  
0040ABC3    03C3            add     eax, ebx        //计算真实地址  
0040ABC5    85D2            test    edx, edx  
0040ABC7    8945 F0         mov     dword ptr [ebp-10], eax  
0040ABCA    74 0D          je      short 0040ABD9  
0040ABCC    8B46 10         mov     eax, dword ptr [esi+10]  //导出函数的起始序号  
0040ABCF    69C0 FFFFFFF3F  imul    eax, eax, 3FFFFFFF    //这条语句，不解？？？  
0040ABD5    03C2            add     eax, edx  
0040ABD7    EB 3E          jmp     short 0040AC17  
0040ABD9    8365 08 00      and     dword ptr [ebp+8], 0  
0040ABDD    837E 18 00      cmp     dword ptr [esi+18], 0  //以函数名导出函数总是为 0 否？  
0040ABE1    76 3C          jbe     short 0040AC1F    //无函数名导出函数则跳，这里不可能跳的  
0040ABE3    894D F8         mov     dword ptr [ebp-8], ecx  
0040ABE6    8B45 F8         mov     eax, dword ptr [ebp-8]  
0040ABE9    FF75 0C         push    dword ptr [ebp+C]    //要导出的函数名地址  
0040ABEC    8B00            mov     eax, dword ptr [eax]  
0040ABEE    03C3            add     eax, ebx  
0040ABF0    50              push    eax  
0040ABF1    E8 4F000000     call    0040AC45       //两个函数名比较函数，思路有点意思，有兴趣可以看一看:)  
0040ABF6    59              pop     ecx  
0040ABF7    85C0            test    eax, eax  
0040ABF9    59              pop     ecx  
0040ABFA    75 11          jnz     short 0040AC0D  
0040ABFC    FF45 08         inc     dword ptr [ebp+8]    //函数名不匹配，则继续  
0040ABFF    8345 F8 04      add     dword ptr [ebp-8], 4  //函数名地址表偏移+4  
0040AC03    8B45 08         mov     eax, dword ptr [ebp+8]  //[ebp+8]是已经匹配过函数名的数量  
0040AC06    3B46 18         cmp     eax, dword ptr [esi+18]  //函数名地址表中所有都匹配完？  
0040AC09  ^ 72 DB          jb      short 0040ABE6    //小于则继续匹配  
0040AC0B    EB 12          jmp     short 0040AC1F  
0040AC0D    8B45 F0         mov     eax, dword ptr [ebp-10]  //如果找到要找的函数名后，跳到这里  
0040AC10    8B4D 08         mov     ecx, dword ptr [ebp+8]  //要找函数名在函数名表中的索引  
0040AC13    0FB70448        movzx   eax, word ptr [eax+ecx*2]  //到函数序号表中取出要导出函数的序号  
0040AC17    8B3C87          mov     edi, dword ptr [edi+eax*4]  //到函数地址表中取要导出函数地址 RVA  
0040AC1A    03FB            add     edi, ebx        //唉，千辛万苦，终于得到了我们所有的函数地址  
0040AC1C    897D FC         mov     dword ptr [ebp-4], edi  
0040AC1F    837D 10 00      cmp     dword ptr [ebp+10], 0  //如果[ebp+10]是 GetProcAddress 地址，则效验刚才找到的函数地址是否合法，不合法则 call [ebp + 10]  
0040AC23    74 18          je      short 0040AC3D  
0040AC25    3975 FC         cmp     dword ptr [ebp-4], esi  //是否大于导出表地址，应该要大于  
0040AC28    76 13          jbe     short 0040AC3D  
0040AC2A    8B45 EC         mov     eax, dword ptr [ebp-14]  
0040AC2D    03C6            add     eax, esi  
0040AC2F    3945 FC         cmp     dword ptr [ebp-4], eax  //是否大于(导出表地址 + 导出表大小)，也应该大于  
0040AC32    73 09          jnb     short 0040AC3D  
0040AC34    FF75 F4         push    dword ptr [ebp-C]  
0040AC37    53              push    ebx  
0040AC38    FF55 10         call    dword ptr [ebp+10]    //效验失败，则调用 GetProcAddress 获取函数地址  
0040AC3B    EB 03          jmp     short 0040AC40  
0040AC3D    8B45 FC         mov     eax, dword ptr [ebp-4]  
0040AC40    5F              pop     edi  
0040AC41    5E              pop     esi  
0040AC42    5B              pop     ebx  
0040AC43    C9              leave  
0040AC44    C3              retn 
```

4dd466d0e1e0cc123afbe504da2462ca
