
```
006186A5    B1 19             mov     cl, 19
006186A7    32CE            xor     cl, dh
006186A9    85F3            test    ebx, esi
006186AB    FB              sti
006186AC    8CFD            mov     bp, seg?                 ; 未定义的段寄存器
006186AE    98              cwde
006186AF    52              push    edx
006186B0    8B91 BB7D6B9E   mov     edx, dword ptr [ecx+9E6B7DBB]
006186B6    4E              dec     esi
006186B7    2A30            sub     dh, byte ptr [eax]
006186B9    9E              sahf
006186BA    65:B7 2B        mov     bh, 2B
006186BD    6389 507206CA   arpl    word ptr [ecx+CA067250], cx
006186C3    7A 95           jpe     short 0061865A
006186C5    A3 08927200     mov     dword ptr [729208], eax    ; (5)=(7)+08
006186CA    8B56 08         mov     edx, dword ptr [esi+8]
006186CD    8915 0C927200   mov     dword ptr [72920C], edx    ; (6)=(7)+0C
006186D3    8B76 0C         mov     esi, dword ptr [esi+C]
006186D6    81E6 FF7F0000   and     esi, 7FFF
006186DC    8935 00927200   mov     dword ptr [729200], esi    ; (7)
006186E2    83F9 02         cmp     ecx, 2
006186E5    74 0C           je      short 006186F3
006186E7    81CE 00800000   or      esi, 8000
006186ED    8935 00927200   mov     dword ptr [729200], esi
006186F3    C1E0 08         shl     eax, 8
006186F6    03C2            add     eax, edx
006186F8    A3 04927200     mov     dword ptr [729204], eax
006186FD    33F6            xor     esi, esi
006186FF    56              push    esi
00618700    8B3D 10D16900   mov     edi, dword ptr [69D110]    ; kernel32.GetModuleHandleA
00618706    FFD7            call    edi
00618708    66:8138 4D5A    cmp     word ptr [eax], 5A4D
0061870D    75 1F           jnz     short 0061872E
0061870F    8B48 3C         mov     ecx, dword ptr [eax+3C]
00618712    03C8            add     ecx, eax
00618714    8139 50450000   cmp     dword ptr [ecx], 4550
0061871A    75 12           jnz     short 0061872E
0061871C    0FB741 18       movzx   eax, word ptr [ecx+18]
00618720    3D 0B010000     cmp     eax, 10B
00618725    74 1F           je      short 00618746
00618727    3D 0B020000     cmp     eax, 20B
0061872C    74 05           je      short 00618733
0061872E    8975 E4         mov     dword ptr [ebp-1C], esi
00618731    EB 27           jmp     short 0061875A
00618733    83B9 84000000   cmp     dword ptr [ecx+84], 0E
0061873A    76 F2           jbe     short 0061872E
0061873C    33C0            xor     eax, eax
0061873E    39B1 F8000000   cmp     dword ptr [ecx+F8], esi
00618744    EB 0E           jmp     short 00618754
00618746    8379 74 0E      cmp     dword ptr [ecx+74], 0E
0061874A    76 E2           jbe     short 0061872E
0061874C    33C0            xor     eax, eax                 我们搜索的位置
0061874E    39B1 E8000000   cmp     dword ptr [ecx+E8], esi
00618754    0F95C0          setne   al
00618757    8945 E4         mov     dword ptr [ebp-1C], eax
0061875A    6A 01           push    1
```

好了，现在根据 `wynney` 兄所提到的各地址之间的关系

引用：

- 以 `⑺` 为基点有以下关系
  - `OEP` 位置 = `③`
  - `⑴` = `②`，`⑵` = `①`，`⑶` = `④`
  - `⑷` = `⑺` - `04`，`⑸` = `⑺` + `08`，`⑹` = `⑺` + `0C`
- 那么其他的都一样的啦
- 对 `VC++ 7.0` 而言，以上关系是永远存在的

找到我们标记的 `③` 的地方，改汇编后接着改 `⑴`＝`②` 的地方，接着 `⑵`=`①` 的地方，下面两段代码就用原来没加壳前的代码，接着是 `⑶`=`④` 的地方，以下四段代码照原来没加壳的代码改写，接着

`call dword ptr [69D088]` 这里 `[69D088]` 是从上面来的

接着有一段代码用原来没加壳的就可以，代码改好了现在要注意的是 `⑷`=`⑺`-`04`，`729200`-`04`=`7291FC` 得来;

`⑸`=`⑺`+`08`，`⑹`=`⑺`+`0C`，这两个的能看出来了，我就不写了

还原 `Stolen Code` 后代码如下

```
/////////////////////////////////////////////////////////////////////////////
00618693    6A 60             push    60                       ; 真 OEP;=③
00618695    68 E8156C00       push    006C15E8                   ; (1)=②
0061869A    E8 81420000       call    0061C920                   ; (2)=①
0061869F    BF 94000000       mov     edi, 94
006186A4    8BC7              mov     eax, edi
006186A6    E8 35E3FFFF       call    006169E0                   ; (3)=④
006186AB    8965 E8           mov     dword ptr [ebp-18], esp
006186AE    8BF4              mov     esi, esp
006186B0    893E              mov     dword ptr [esi], edi
006186B2    56                push    esi
006186B3    FF15 88D06900     call    dword ptr [69D088]     ; kernel32.GetVersionExA
006186B9    8B4E 10           mov     ecx, dword ptr [esi+10]
006186BC    890D FC917200     mov     dword ptr [7291FC], ecx   ; (4)=(7)-04
006186C2    8B46 04           mov     eax, dword ptr [esi+4]
```

机器码如下：在这里也可以用机器码改写，我刚才一步步是为了给大家能看懂

```
6A 60 68 E8 15 6C 00 E8 81 42 00 00 BF 94 00 00 00 8B C7 E8 35 E3 FF FF 89 65 E8 8B F4 89
3E 56 FF 15 88 D0 69 00 8B 4E 10 89 0D FC 91 72 00 8B 46 04
```

```
///////////////////////////////////////////////////////////
006186C5    A3 08927200       mov     dword ptr [729208], eax    ; (5)=(7)+08
006186CA    8B56 08           mov     edx, dword ptr [esi+8]
006186CD    8915 0C927200     mov     dword ptr [72920C], edx    ; (6)=(7)+0C
006186D3    8B76 0C           mov     esi, dword ptr [esi+C]
006186D6    81E6 FF7F0000     and     esi, 7FFF
006186DC    8935 00927200     mov     dword ptr [729200], esi    ; (7)
006186E2    83F9 02           cmp     ecx, 2
006186E5    74 0C             je      short 006186F3
006186E7    81CE 00800000     or      esi, 8000
006186ED    8935 00927200     mov     dword ptr [729200], esi
006186F3    C1E0 08           shl     eax, 8
006186F6    03C2              add     eax, edx
006186F8    A3 04927200       mov     dword ptr [729204], eax
006186FD    33F6              xor     esi, esi
006186FF    56                push    esi
00618700    8B3D 10D16900     mov     edi, dword ptr [69D110]    ; kernel32.GetModuleHandleA
00618706    FFD7              call    edi
00618708    66:8138 4D5A      cmp     word ptr [eax], 5A4D
0061870D    75 1F             jnz     short 0061872E
0061870F    8B48 3C           mov     ecx, dword ptr [eax+3C]
00618712    03C8              add     ecx, eax
00618714    8139 50450000     cmp     dword ptr [ecx], 4550
0061871A    75 12             jnz     short 0061872E
0061871C    0FB741 18         movzx   eax, word ptr [ecx+18]
00618720    3D 0B010000       cmp     eax, 10B
00618725    74 1F             je      short 00618746
00618727    3D 0B020000       cmp     eax, 20B
0061872C    74 05             je      short 00618733
0061872E    8975 E4           mov     dword ptr [ebp-1C], esi
00618731    EB 27             jmp     short 0061875A
00618733    83B9 84000000     cmp     dword ptr [ecx+84], 0E
0061873A    76 F2             jbe     short 0061872E
0061873C    33C0              xor     eax, eax
0061873E    39B1 F8000000     cmp     dword ptr [ecx+F8], esi
00618744    EB 0E             jmp     short 00618754
00618746    8379 74 0E        cmp     dword ptr [ecx+74], 0E
0061874A    76 E2             jbe     short 0061872E
0061874C    33C0              xor     eax, eax
0061874E    39B1 E8000000     cmp     dword ptr [ecx+E8], esi
00618754    0F95C0            setne   al
00618757    8945 E4           mov     dword ptr [ebp-1C], eax
0061875A    6A 01             push    1
```

上面代码还原完成。

第五步：脱壳和修复。
