
```
00AA6127    8BC3            mov     eax, ebx  
00AA6129    FFD2            call    edx            // (X) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
00AA612B    8845 EB         mov     [ebp-15], al  
00AA612E    33C0            xor     eax, eax  
00AA6130    8A46 06         mov     al, [esi+6]  
00AA6133    8D0440          lea     eax, [eax+eax*2]  
00AA6136    8B55 F8         mov     edx, [ebp-8]  
00AA6139    8B5482 68       mov     edx, [edx+eax*4+68]  
00AA613D    8BC3            mov     eax, ebx  
00AA613F    FFD2            call    edx            // (Y) 0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
00AA6141    8845 EA         mov     [ebp-16], al  
00AA6144    EB 01           jmp     short 00AA6147  
00AA6146  - E9 33C08A46     jmp     4735217E  
00AA614B    07              pop     es  
00AA614C    8D0440          lea     eax, [eax+eax*2]  
00AA614F    8B55 F8         mov     edx, [ebp-8]  
00AA6152    8B5482 68       mov     edx, [edx+eax*4+68]  
00AA6156    8BC3            mov     eax, ebx  
00AA6158    FFD2            call    edx            // 常数 n  
00AA615A    8BD8            mov     ebx, eax  
00AA615C    8A4D EB         mov     cl, [ebp-15]  
00AA615F    8B55 10         mov     edx, [ebp+10]  
00AA6162    8B45 F8         mov     eax, [ebp-8]  
00AA6165    E8 4E010000     call    00AA62B8  
00AA616A    8BF0            mov     esi, eax  
00AA616C    8A4D EA         mov     cl, [ebp-16]  
00AA616F    8B55 10         mov     edx, [ebp+10]  
00AA6172    8B45 F8         mov     eax, [ebp-8]  
00AA6175    E8 3E010000     call    00AA62B8  
00AA617A    03F3            add     esi, ebx  
00AA617C    8906            mov     [esi], eax  
00AA617E    EB 01           jmp     short 00AA6181  
00AA6180    9A 8B5DF003 5DF> call    far F45D:03F05D8B  
00AA6187    8B45 F8         mov     eax, [ebp-8]  
00AA618A    0398 E0000000   add     ebx, [eax+E0]  
00AA6190    E9 B5000000     jmp     00AA624A  
```

7====================================================================

`//` `edx`+4C 的情况，模拟：

`CMP x,y`

`jxx n`

（特别感谢 jwh51 在这情况给与的技术指点）

```
00AA6195    8B55 F8         mov     edx, [ebp-8]  
00AA6198    3A42 4C         cmp     al, [edx+4C]  
00AA619B    75 67           jnz     short 00AA6204  
00AA619D    EB 01           jmp     short 00AA61A0  
00AA61A0    8BCB            mov     ecx, ebx  
00AA61A2    8B55 10         mov     edx, [ebp+10]  
00AA61A5    8B45 F8         mov     eax, [ebp-8]  
00AA61A8    E8 6FF7FFFF     call    00AA591C   // 进去  
{  
    00AA5967    8D0440          lea     eax, [eax+eax*2]  
    00AA596A    8B5483 68       mov     edx, [ebx+eax*4+68]  
    00AA596E    8BC7            mov     eax, edi  
    00AA5970    FFD2            call    edx                              ; (x)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
    00AA5972    8BD0            mov     edx, eax  
    00AA5974    80EA 08         sub     dl, 8  
    00AA5977    0F92C2          setb    dl  
    00AA597A    80FA 01         cmp     dl, 1  
    00AA597D    75 0F           jnz     short 00AA598E  
    00AA597F    8BC8            mov     ecx, eax  
    00AA5981    8B55 FC         mov     edx, [ebp-4]  
    00AA5984    8BC3            mov     eax, ebx  
    00AA5986    E8 2D090000     call    00AA62B8  
    00AA598B    8945 EC         mov     [ebp-14], eax  
    00AA598E    33C0            xor     eax, eax  
    00AA5990    8A46 09         mov     al, [esi+9]  
    00AA5993    8D0440          lea     eax, [eax+eax*2]  
    00AA5996    8B5483 68       mov     edx, [ebx+eax*4+68]  
    00AA599A    8BC7            mov     eax, edi  
    00AA599C    FFD2            call    edx  
    00AA599E    8845 F3         mov     [ebp-D], al  
    00AA59A1    EB 01           jmp     short 00AA59A4  
    00AA59A3    6933 C08A4608   imul    esi, [ebx], 8468AC0  
    00AA59A9    8D0440          lea     eax, [eax+eax*2]  
    00AA59AC    8B5483 68       mov     edx, [ebx+eax*4+68]  
    00AA59B0    8BC7            mov     eax, edi  
    00AA59B2    FFD2            call    edx                              ;  如果是 cmp x,n  情况，此处返回常数 n。否则返回 0  
    00AA59B4    8945 F4         mov     [ebp-C], eax  
    00AA59B7    33C0            xor     eax, eax  
    00AA59B9    8A46 06         mov     al, [esi+6]  
    00AA59BC    8D0440          lea     eax, [eax+eax*2]  
    00AA59BF    8B5483 68       mov     edx, [ebx+eax*4+68]  
    00AA59C3    8BC7            mov     eax, edi  
    00AA59C5    FFD2            call    edx                              ; (y)0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi  
    00AA59C7    8BD0            mov     edx, eax  
    00AA59C9    80EA 08         sub     dl, 8  
    00AA59CC    0F92C2          setb    dl  
    00AA59CF    80FA 01         cmp     dl, 1  
    00AA59D2    75 12           jnz     short 00AA59E6  
}  
00AA61B3    33C0            xor     eax, eax  
00AA61B5    8A46 04         mov     al, [esi+4]  
00AA61B8    8D0440          lea     eax, [eax+eax*2]  
00AA61BB    8B55 F8         mov     edx, [ebp-8]  
00AA61BE    8B5482 68       mov     edx, [edx+eax*4+68]  
00AA61C2    8BC3            mov     eax, ebx  
00AA61C4    FFD2            call    edx                              ;  跳转类型，如是机器码是 74 则返回 4,如是 75 则返回 5，依次类推  
00AA61C6    8BD8            mov     ebx, eax  
00AA61C8    8B4D 14         mov     ecx, [ebp+14]  
00AA61CB    8BD3            mov     edx, ebx  
00AA61CD    8B45 F8         mov     eax, [ebp-8]  
00AA61D0    E8 87F4FFFF     call    00AA565C  
00AA61D5    84C0            test    al, al  
00AA61D7    74 1A           je      short 00AA61F3  
00AA61D9    EB 01           jmp     short 00AA61DC  
00AA61DC    8B45 F8         mov     eax, [ebp-8]  
00AA61DF    8B98 E0000000   mov     ebx, [eax+E0]  
00AA61E5    035D EC         add     ebx, [ebp-14]  
00AA61E8    8B45 F8         mov     eax, [ebp-8]  
00AA61EB    0358 14         add     ebx, [eax+14]                     ;  跳转的地址  
00AA61EE    EB 5A           jmp     short 00AA624A  
00AA61F3    8B45 F8         mov     eax, [ebp-8]  
00AA61F6    8B98 E0000000   mov     ebx, [eax+E0]  
00AA61FC    035D F0         add     ebx, [ebp-10]  
00AA61FF    035D F4         add     ebx, [ebp-C]                       ;  不跳的地址  
00AA6202    EB 46           jmp     short 00AA624A  
```

8====================================================================

`//` `edx`+4D 的情况，模拟: `CMP x,y`

```
00AA6204    8B55 F8         mov     edx, [ebp-8]  
00AA6207    3A42 4D         cmp     al, [edx+4D]  
00AA620A    75 27           jnz     short 00AA6233  
00AA620C    EB 01           jmp     short 00AA620F  
00AA620F    8BCB            mov     ecx, ebx  
00AA6211    8B55 10         mov     edx, [ebp+10]  
00AA6214    8B45 F8         mov     eax, [ebp-8]  
00AA6217    E8 00F7FFFF     call    00AA591C   // 里面和 edx+4C 情况处理一样  
```

【版权声明】本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

【参考资料】[nspack3.5 主程序脱壳分析(Aspr SKE 2.X)  ](http://bbs.pediy.com/showthread.php?s=&threadid=19313) 作者：shoooo

看雪技术论坛(http://bbs.pediy.com)

2006 年 04 月 07 日

```
009ABE00    53              push    ebx  
009ABE01    56              push    esi  
009ABE02    57              push    edi  
009ABE03    55              push    ebp  
009ABE04    8BFA            mov     edi,edx  
009ABE06    B3 01           mov     bl,1  
009ABE08    8BF0            mov     esi,eax  
009ABE0A    83C6 08         add     esi,8  
009ABE0D    8BEE            mov     ebp,esi  
009ABE0F    EB 15           jmp     short 009ABE26  
009ABE11    55              push    ebp  
009ABE12    FFD7            call    edi   //处理部分 CALL，变成 PUSH XXXXXXXX，RETN 或 JUMP XXXXXXXX  
009ABE14    84C0            test    al,al  
009ABE16    75 04           jnz     short 009ABE1C  
009ABE18    33DB            xor     ebx,ebx  
```
