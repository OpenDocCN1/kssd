
00450174  E5 1D 03 33 5F 4C 0A AA C9 7C 0D DD 3C 71 05 50  ????糉?焼倅  

00450184  AA 41 02 27 10 10 0B BE 86 20 0C C9 25 B5 68 57  ???????坨  

00450194  B3 85 6F 20 09 D4 66 B9 9F E4 61 CE 0E F9 DE 5E  薳???癩廞  

004501A4  98 C9 D9 29 22 98 D0 B0 B4 A8 D7 C7 17 3D B3 59  ??頢????妳  

004501B4  81 0D B4 2E 3B 5C BD B7 AD 6C BA C0 20 83 B8 ED  ??尻?沭?茠?  

004501C4  B6 B3 BF 9A 0C E2 B6 03 9A D2 B1 74 39 47 D5 EA  ?骿ζ?璱??  

004501D4  AF 77 D2 9D 15 26 DB 04 83 16 DC 73 12 0B 63 E3  瞯鷒???珜?  

004501E4  84 3B 64 94 3E 6A 6D 0D A8 5A 6A 7A 0B CF 0E E4  ?鑤樾??#123;穪?  

004501F4  9D FF 09 93 27 AE 00 0A B1 9E 07 7D 44 93 0F F0  ?鋉??麱紇鍄?  

00450204  D2 A3 08 87 68 F2 01 1E FE C2 06 69 5D 57 62 F7  ?蜈???椆坝?  

00450214  CB 67 65 80 71 36 6C 19 E7 06 6B 6E 76 1B D4 FE  柋聥???湫??  

00450224  E0 2B D3 89 5A 7A DA 10 CC 4A DD 67 6F DF B9 F9  ?觓穚??柝?惡  

00450234  F9 EF BE 8E 43 BE B7 17 D5 8E B0 60 E8 A3 D6 D6  ?躾??軕悰??  

00450244  7E 93 D1 A1 C4 C2 D8 38 52 F2 DF 4F F1 67 BB D1  鍾????俟柱?  

00450254  67 57 BC A6 DD 06 B5 3F 4B 36 B2 48 DA 2B 0D D8  坧???????  

00450264  4C 1B 0A AF F6 4A 03 36 60 7A 04 41 C3 EF 60 DF  ????穠???  

00450274  55 DF 67 A8 EF 8E 6E 31 79 BE 69 46 8C B3 61 CB  ??軯?????  

00450284  1A 83 66 BC A0 D2 6F 25 36 E2 68 52 95 77 0C CC  茚??╯剨瞕?  

00450294  03 47 0B BB B9 16 02 22 2F 26 05 55 BE 3B BA C5  ?????唅??  

004502A4  28 0B BD B2 92 5A B4 2B 04 6A B3 5C A7 FF D7 C2  ??媒?樄岳??  

004502B4  31 CF D0 B5 8B 9E D9 2C 1D AE DE 5B B0 C2 64 9B  ??麋??寞?魤  

004502C4  26 F2 63 EC 9C A3 6A 75 0A 93 6D 02 A9 06 09 9C  ???番錊??鰉  

004502D4  3F 36 0E EB 85 67 07 72 13 57 00 05 82 4A BF 95  ??枅\_k 圓??閿  

004502E4  14 7A B8 E2 AE 2B B1 7B 38 1B B6 0C 9B 8E D2 92  稔?箱??躛鋒  

004502F4  0D BE D5 E5 B7 EF DC 7C 21 DF DB 0B D4 D2 D3 86  ?糜???蛓  

00450304  42 E2 D4 F1 F8 B3 DD 68 6E 83 DA 1F CD 16 BE 81  ??棝荮??膾  

00450314  5B 26 B9 F6 E1 77 B0 6F 77 47 B7 18 E6 5A 08 88  ??矡澰??嫦蠈  

00450324  70 6A 0F FF CA 3B 06 66 5C 0B 01 11 FF 9E 65 8F  橰／?昆??黿轥  

00450334  69 AE 62 F8 D3 FF 6B 61 45 CF 6C 16 78 E2 0A A0  ???慫???  

00450344  EE D2 0D D7 54 83 04 4E C2 B3 03 39 61 26 67 A7  ??荔丄????  

00450354  F7 16 60 D0 4D 47 69 49 DB 77 6E 3E 4A 6A D1 AE  ????矛?橊?  

00450364  DC 5A D6 D9 66 0B DF 40 F0 3B D8 37 53 AE BC A9  嫜???????  

00450374  C5 9E BB DE 7F CF B2 47 E9 FF B5 30 1C F2 BD BD  黅????サ??  

00450384  8A C2 BA CA 30 93 B3 53 A6 A3 B4 24 05 36 D0 BA  ??錰厳????  

00450394  93 06 D7 CD 29 57 DE 54 BF 67 D9 23 2E 7A 66 B3  ??圩哞枿?種?  

004503A4  B8 4A 61 C4 02 1B 68 5D 94 2B 6F 2A 37 BE 0B B4  ???嵨????  

004503B4  A1 8E 0C C3 1B DF 05 5A 8D EF 02 2D             躡??娅??  

总结一下关键 `CALL1`：

- 初始值：`edi=-1`
- `EDX=注册码(i) xor edi and 0ff`，然后查表，`EDX=[edx*4+44FFC0]` 指向的内容
- `ECX=EDI shr 8`
- `edi=edx xor ecx`
- `i` 指向下一位，循环。最终的值 `xor FFFFFFFF(-1)--->eax`

`///////////////////////////////////////////////////////////////////////////////////////////////////////`

关键 `CALL2`：

```
00423D68  /$  55            push    ebp  
00423D69  |.  89E5          mov     ebp, esp  
00423D6B  |.  83EC 10       sub     esp, 10  
00423D6E  |.  8365 FC 00    and     dword ptr [ebp-4], 0  
00423D72  |.  8365 F8 00    and     dword ptr [ebp-8], 0  
00423D76  |.  8365 F0 00    and     dword ptr [ebp-10], 0  
00423D7A  |.  EB 03         jmp     short 00423D7F  
00423D7C  |>  FF45 08       /inc    dword ptr [ebp+8]  
00423D7F  |>  8B45 08       mov     eax, dword ptr [ebp+8]  
00423D82  |.  8A08          |mov    cl, byte ptr [eax]  
00423D84  |.  80F9 20       |cmp    cl, 20  
00423D87  |.^ 74 F3         |je     short 00423D7C  
00423D89  |.  80F9 09       |cmp    cl, 9  
00423D8C  |.^ 74 EE         \je     short 00423D7C  
00423D8E  |.  8038 2B       cmp     byte ptr [eax], 2B  
00423D91  |.  75 05         jnz     short 00423D98  
00423D93  |.  FF45 08       inc     dword ptr [ebp+8]  
00423D96  |.  EB 50         jmp     short 00423DE8  
00423D98  |>  8B45 08       mov     eax, dword ptr [ebp+8]  
00423D9B  |.  8038 2D       cmp     byte ptr [eax], 2D  
00423D9E  |.  75 48         jnz     short 00423DE8  
00423DA0  |.  C745 F0 01000>mov    dword ptr [ebp-10], 1           ;[ebp-10]=1  
00423DA7  |.  FF45 08       inc     dword ptr [ebp+8]  
00423DAA  |.  EB 3C         jmp     short 00423DE8           ；跳  
00423DAC  |>  8B45 08       /mov    eax, dword ptr [ebp+8]         ;  注册码到 EAX  
00423DAF  |.  FF45 08       |inc    dword ptr [ebp+8]  
00423DB2  |.  0FBE00        |movsx   eax, byte ptr [eax]          ;  注册码依次到 EAX  
00423DB5  |.  83E8 30       |sub     eax, 30                      ;  -0  
00423DB8  |.  8945 F4       |mov     dword ptr [ebp-C], eax       ;  结果保存到[EBP-C]  
00423DBB  |.  BA 00000000   |mov     edx, 0                       ；edx=0  
00423DC0  |.  B8 0A000000   |mov     eax, 0A                      ;eax=0a  
00423DC5  |.  83EC 08       |sub     esp, 8  
00423DC8  |.  DF6D F8       |fild    qword ptr [ebp-8]            ;  [EBP-8]-->ST(0)  
00423DCB  |.  DF3C24        |fistp   qword ptr [esp]              ;  ST(0)-->[ESP]  
00423DCE  |.  52            |push    edx                          ;作用应该是[EBP-8] <==>[esp]  
00423DCF  |.  50            |push    eax  
00423DD0  |.  E8 5BF8FFFF   |call    00423630  
    {  
       00423630  /$  8B4424 08     mov     eax, dword ptr [esp+8]     ;不知道程序在什么地方对[ESP+8]赋值，我这里总是 0  
       00423634  |.  8B4C24 10     mov     ecx, dword ptr [esp+10]    ;不知道程序在什么地方对[ESP+10]赋值，我这里总是 0  
       00423638  |.  09C1          or      ecx, eax  
       0042363A  |.  8B4C24 0C     mov     ecx, dword ptr [esp+C]      ;观察发现[esp+c]=[esp]  
       0042363E  |.  75 09         jnz     short 00423649  
       00423640  |.  8B4424 04     mov     eax, dword ptr [esp+4]      ；观察发现[ESP+4]总是等于 0A  
       00423644  |.  F7E1          mul     ecx                        ；[esp+c]*0A  
       00423646  |.  C2 1000       retn    10                          ；所以每次到这里就返回了  
       00423649  |>  53            push    ebx  
       0042364A  |.  F7E1          mul     ecx  
       0042364C  |.  89C3          mov     ebx, eax  
       0042364E  |.  8B4424 08     mov     eax, dword ptr [esp+8]  
       00423652  |.  F76424 14     mul     dword ptr [esp+14]  
       00423656  |.  01C3          add     ebx, eax  
       00423658  |.  8B4424 08     mov     eax, dword ptr [esp+8]  
       0042365C  |.  F7E1          mul     ecx  
       0042365E  |.  01DA          add     edx, ebx  
       00423660  |.  5B            pop     ebx  
       00423661  \.  C2 1000       retn    10  
      }  
  00423DD5  |.  52            |push    edx  
  00423DD6  |.  50            |push    eax  
  00423DD7  |.  8B45 F4       |mov     eax, dword ptr [ebp-C]         ;EAX<--[EBP-C]  
  00423DDA  |.  99            |cdq  
  00423DDB  |.  010424        |add     dword ptr [esp], eax           ;[ESP]=[ESP]+EAX  
  00423DDE  |.  115424 04     |adc     dword ptr [esp+4], edx  
  00423DE2  |.  8F45 F8       |pop     dword ptr [ebp-8]              ;[EBP-8]出栈  
  00423DE5  |.  8F45 FC       |pop     dword ptr [ebp-4]  
  00423DE8  |>  8B45 08        mov     eax, dword ptr [ebp+8]         ;  注册码到 eax  
```
