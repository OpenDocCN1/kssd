
```
00AA5BB9    8BEC            mov     ebp, esp
00AA5BBB    83C4 CC         add     esp, -34
00AA5BBE    53              push    ebx
00AA5BBF    56              push    esi
00AA5BC0    57              push    edi
00AA5BC1    894D F8         mov     dword ptr [ebp-8], ecx
……
00AA5CFF    FFD7            call    edi
00AA5D01    84C0            test    al, al
00AA5D03    8B45 F4         mov     eax, dword ptr [ebp-C]
00AA5D06    8A40 4A         mov     al, byte ptr [eax+4A]
00AA5D09    3A45 EF         cmp     al, byte ptr [ebp-11]            ; al的值决定是FF15还是FF25
00AA5D0C    0F85 9C000000   jnz     00AA5DAE                         ; 不跳是FF15情况
```

`al`中的值，不同程序是不同的。本节实例：当`al=4c`时，对应的是`FF15`。

`FF15`分支来到：

```
00AA5D15    8B45 F4         mov     eax, dword ptr [ebp-C]
00AA5D18    8B80 E0000000   mov     eax, dword ptr [eax+E0]
00AA5D1E    0145 FC         add     dword ptr [ebp-4], eax           ;[ebp-4]是解密出来的API
00AA5D21   /EB 01           jmp     short 00AA5D24
```

`FF25`分支来到：

```
00AA5DC7    8B45 F4         mov     eax, dword ptr [ebp-C]
00AA5DCA    8B80 E0000000   mov     eax, dword ptr [eax+E0]
00AA5DD0    0145 FC         add     dword ptr [ebp-4], eax           ; [ebp-4] 是我们需要的导入函数的地址
00AA5DD3    8D45 0C         lea     eax, dword ptr [ebp+C]
```

> *最初由 **shoooo** 发布*

> `00AA5D1E`或`00AA5DD0` 执行完后`[ebp-4]` 是需要的输入函数的地址

> 再看看`[ebp-2c]`，如果它是`FFFFFFFF`，说明这个导入函数调用是干净的，如果它有值，表示它的下一行也偷了。

## 4.修复`call 01200000`

经过上面分析，现在可以写修复代码了，每个被加密的函数都从`call 01200000`进入，在计算过程中某个点，其调用函数名会显示出来，因此我们在这个出口设断取得函数地址。

`FF15`分支来到：

```
00AA5D15    8B45 F4         mov     eax, dword ptr [ebp-C]
00AA5D18    8B80 E0000000   mov     eax, dword ptr [eax+E0]
00AA5D1E    0145 FC         add     dword ptr [ebp-4], eax           ;[ebp-4]是解密出来的API
00AA5D21   /EB 01           jmp     short 00AA5D24                     //在这下硬件断点
```

`FF25`分支来到：

```
00AA5DC7    8B45 F4         mov     eax, dword ptr [ebp-C]
00AA5DCA    8B80 E0000000   mov     eax, dword ptr [eax+E0]
00AA5DD0    0145 FC         add     dword ptr [ebp-4], eax           ; [ebp-4] 是我们需要的导入函数的地址
00AA5DD3    8D45 0C         lea     eax, dword ptr [ebp+C]             //在这下硬件断点
```

再来看看实例的IAT：

```
00405000  CF C6 80 7C 78 2C 81 7C 80 A4 80 7C B9 8C 83 7C  掀?x,?????
00405010  C4 CE 80 7C 2B 2E 83 7C AD 9C 80 7C 77 1D 80 7C  奈?+.???w?
00405020  28 AC 80 7C 29 B5 80 7C EE 1E 80 7C 8D 2C 81 7C  (?|)?|????
00405030  AB 14 81 7C A2 CA 81 7C 16 1E 80 7C 0D E0 80 7C  ??⑹??.?|
00405040  8A 2B 86 7C 57 B3 80 7C 3F DC 81 7C 5F 48 81 7C  ??W?|??|\_H?
00405050  C7 A0 80 7C 23 CC 81 7C FD 79 93 7C A9 2C 81 7C  沁?#?|????
00405060  69 10 81 7C 10 11 81 7C 29 29 81 7C 14 9B 80 7C  i??))??|
00405070  3D 04 93 7C 40 7A 95 7C 9F 0F 81 7C E6 2B 81 7C  =?@z?????
00405080  43 99 80 7C 2A E8 81 7C D4 05 93 7C 81 9A 80 7C  C?|\*?|????
00405090  00 00 00 00 1E AC D6 77 4A C9 D3 77 AD A8 D1 77  ....?wJ捎w?痒
004050A0  11 12 D2 77 24 13 D2 77 9A F3 D2 77 85 CB D1 77  吟$吟?吟?痒
004050B0  50 62 D2 77 1C B1 D3 77 00 00 00 00 00 00 00 00  Pb吟庇w........
```

从上面的IAT可以看出，IAT地址在`405000～4050B8`之间，并且这些地址没加密。

还记得第2节获得的地址表吧，现在可以用上了，写个补丁程序，扫描这个地址表，依次从表里的每个地址进入，然后在`00AA5D21`或`00AA5DD3`两个出口处获得函数地址，扫描IAT表，将找到的IAT地址写入程序代码中。

用`HideOD`插件分配一个临时空间（`Alloc Memory`菜单命令）：

```
00C50000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00C50010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00C50020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

将如下补丁代码放上（由于水平有限，代码写的比较乱，但将a、b两种情况都考虑了，因此比较通用）：

```
00C50000    A1 C000C500        mov     eax, dword ptr [C500C0]                 ;  指向待处理的地址列表
00C50005    8B18               mov     ebx, dword ptr [eax]
00C50007    81E3 FFFFFF7F      and     ebx, 7FFFFFFF
00C5000D    FFE3               jmp     ebx
00C5000F    0000               add     byte ptr [eax], al
00C50011    0000               add     byte ptr [eax], al
00C50013    0000               add     byte ptr [eax], al
00C50015    0000               add     byte ptr [eax], al
00C50017    0000               add     byte ptr [eax], al
00C50019    0000               add     byte ptr [eax], al
00C5001B    0000               add     byte ptr [eax], al
00C5001D    0000               add     byte ptr [eax], al
00C5001F    0000               add     byte ptr [eax], al                      ; 0c500c0得手工填上，0c500c0＋4处也得填上（存放那些下一句被抽的API的CALL）
00C50021    BF C000C500        mov     edi, 0C500C0                            ; EDI存放的是指向需要处理的API地址表
00C50026    8B07               mov     eax, dword ptr [edi]
00C50028    8B18               mov     ebx, dword ptr [eax]
00C5002A    81FB FFFFFF7F      cmp     ebx, 7FFFFFFF
00C50030    79 49              jns     short 00C5007B                          ; 高位是1的走这条路线（跳）,即处理a方式
00C50032    837D D4 FF         cmp     dword ptr [ebp-2C], -1                  ; 如是FFFFFFFF，说明这个导入函数调用是干净的
00C50036    74 0F              je      short 00C50047
00C50038    8B47 04            mov     eax, dword ptr [edi+4]                  ; EDI＋4处的值得手工填上，我填的是c50160（存放那些下一句被抽的API的CALL）
00C5003B    8B1F               mov     ebx, dword ptr [edi]                    ; 指向待处理的API列表
00C5003D    8B1B               mov     ebx, dword ptr [ebx]
00C5003F    8918               mov     dword ptr [eax], ebx
00C50041    83C0 04            add     eax, 4
00C50044    8947 04            mov     dword ptr [edi+4], eax
00C50047    8B5D FC            mov     ebx, dword ptr [ebp-4]                  ; ebp-4是获得的API函数地址
00C5004A    E8 46000000        call    00C50095                                ; 在IAT里搜索,正确的IAT项从ESI中返回
00C5004F    B0 4C              mov     al, 4C                                  ; 决定是ff15还是ff25,不同软件AL的值不同，本实例是4c
00C50051    66:B9 FF15         mov     cx, 15FF
00C50055    3A45 EF            cmp     al, byte ptr [ebp-11]
00C50058    74 05              je      short 00C5005F
00C5005A    66:81C1 0010       add     cx, 1000
00C5005F    8B07               mov     eax, dword ptr [edi]                    ; 指向待处理的API列表
00C50061    8B18               mov     ebx, dword ptr [eax]
00C50063    81E3 FFFFFF7F      and     ebx, 7FFFFFFF                           ; 将高1位清除
00C50069    83C0 04            add     eax, 4                                  ; 指向下一个地址
00C5006C    8907               mov     dword ptr [edi], eax
00C5006E    66:890B            mov     word ptr [ebx], cx
00C50071    83C3 02            add     ebx, 2
00C50074    8933               mov     dword ptr [ebx], esi
00C50076  ^ EB 88              jmp     short 00C50000
00C50078    90                 nop
00C50079    90                 nop
00C5007A    90                 nop                                             ; 下面是处理a情况（本实例无这情况，但为了大家以后方便，故放出）
00C5007B    8B5D B4            mov     ebx, dword ptr [ebp-4C]                 ; ebp-4c是获得的API函数地址
00C5007E    E8 12000000        call    00C50095                                ; 在IAT里搜索,正确的IAT项从ESI中返回
00C50083    B0 4C              mov     al, 4C                                  ; 这个4c的值在不同的程序是随机
00C50085    66:B9 FF15         mov     cx, 15FF
00C50089    3AC2               cmp     al, dl                                  ; dl中的值决定了是call(ff15)还是jmp(ff25)
00C5008B  ^ 74 D2              je      short 00C5005F
00C5008D  ^ EB CB              jmp     short 00C5005A
00C5008F    0000               add     byte ptr [eax], al
00C50091    0000               add     byte ptr [eax], al
00C50093    0000               add     byte ptr [eax], al                      ; 这个CALL搜索IAT中的指定项
00C50095    BE 00504000        mov     esi, 405000                             ; IAT起始地址
00C5009A    391E               cmp     dword ptr [esi], ebx
00C5009C    74 0D              je      short 00C500AB
00C5009E    83C6 04            add     esi, 4
00C500A1    81FE B8504000      cmp     esi, 4050B8                             ; IAT结束地址，判断是否越界
00C500A7    77 03              ja      short 00C500AC
00C500A9  ^ EB EF              jmp     short 00C5009A                          ; 如果在IAT里没找到就死循环,所以，如果死循环，你按F12暂停程序
00C500AB    C3                 retn
```
