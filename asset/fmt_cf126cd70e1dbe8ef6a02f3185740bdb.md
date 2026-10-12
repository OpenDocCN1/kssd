
### 跟踪函数 `004938D0(GenLv1_MacID_RegCode)`

返回跟踪函数 `004938D0(GenLv1_MacID_RegCode)`，同样通过搜索内存，找到很长的数字串是在中间一个循环之后(`00493A85`)生成的，这个循环里有 `sprintf` 和 `CString::+=` 的调用；跟踪发现此循环的作用是把一个 `24byte` 的数组的每个 `byte` 的十进制表示连接在一起，生成长数字串。

搜索那个 `24byte` 的数组，是在 `call 004055C0` 后生成的。进入分析这个函数 `004055C0`，此函数调用前 `push` 的其中一个参数正是已知的计算机标识码。函数里有个大循环，循环里有一处函数调用。经过跟踪，在执行循环之前，`15byte` 的计算机标识码加结束符`\0` 共 `16byte` 被扩展成 `24byte` 的数组。循环每次对 `8` 个 `byte` 进行处理，结果覆盖该 `24byte` 数组并对下 `8byte` 处理有所影响。

#### 代码

```
//函数 004055C0(GenLv2_MacID_24ByteArray)
004055C0  /$  51            push    ecx
004055C1  |.  53            push    ebx
004055C2  |.  8B5C24 14     mov     ebx, dword ptr [esp+14]
004055C6  |.  55            push    ebp
004055C7  |.  56            push    esi
004055C8  |.  8B7424 14     mov     esi, dword ptr [esp+14]
004055CC  |.  57            push    edi
004055CD  |.  8B7C24 1C     mov     edi, dword ptr [esp+1C]
004055D1  |.  8BC1          mov     eax, ecx
004055D3  |.  3BF7          cmp     esi, edi
004055D5  |.  894424 10     mov     dword ptr [esp+10], eax
004055D9  |.  74 10         je      short 004055EB
004055DB  |.  8BCB          mov     ecx, ebx
004055DD  |.  8BD1          mov     edx, ecx
004055DF  |.  C1E9 02       shr     ecx, 2
004055E2  |.  F3:A5         rep     movs dword ptr es:[edi], dword p>
004055E4  |.  8BCA          mov     ecx, edx
004055E6  |.  83E1 03      and     ecx, 3
004055E9  |.  F3:A4         rep     movs byte ptr es:[edi], byte ptr>
004055EB  |>  8B4C24 24     mov     ecx, dword ptr [esp+24]
004055EF  |.  51            push    ecx
004055F0  |.  53            push    ebx
004055F1  |.  8BC8          mov     ecx, eax
004055F3  |.  E8 A8FFFFFF   call    004055A0
004055F8  |.  8BE8          mov     ebp, eax
004055FA  |.  8A4424 24     mov     al, byte ptr [esp+24]
004055FE  |.  A8 20         test    al, 20
00405600  |.  74 06         je      short 00405608
00405602  |.  8BC5          mov     eax, ebp
00405604  |.  2BC3          sub     eax, ebx
00405606  |.  EB 02         jmp     short 0040560A
00405608  |>  33C0         xor     eax, eax
0040560A  |>  8B7424 1C     mov     esi, dword ptr [esp+1C]
0040560E  |.  8BCD          mov     ecx, ebp
00405610  |.  2BCB          sub     ecx, ebx
00405612  |.  8D3C1E        lea     edi, dword ptr [esi+ebx]
00405615  |.  8AD8          mov     bl, al
00405617  |.  8AFB          mov     bh, bl
00405619  |.  8BD1          mov     edx, ecx
0040561B  |.  8BC3          mov     eax, ebx
0040561D  |.  C1E0 10      shl     eax, 10
00405620  |.  66:8BC3      mov     ax, bx
00405623  |.  C1E9 02      shr     ecx, 2
00405626  |.  F3:AB        rep     stos dword ptr es:[edi]
00405628  |.  8BCA          mov     ecx, edx
0040562A  |.  83E1 03      and     ecx, 3
0040562D  |.  F3:AA        rep     stos byte ptr es:[edi]
0040562F  |.  33C0         xor     eax, eax
00405631  |.  85ED          test    ebp, ebp
00405633  |.  0F86 D8000000 jbe     00405711
00405639  |.  8B4424 24     mov     eax, dword ptr [esp+24]
0040563D  |.  83C5 07      add     ebp, 7
00405640  |.  83E0 02      and     eax, 2
00405643  |.  83C6 06      add     esi, 6
00405646  |.  C1ED 03      shr     ebp, 3
00405649  |.  894424 1C     mov     dword ptr [esp+1C], eax
0040564D  |.  8D0CED 000000>lea     ecx, dword ptr [ebp*8]
00405654  |.  894C24 18     mov     dword ptr [esp+18], ecx
00405658  |.  EB 04         jmp     short 0040565E
0040565A  |>  8B4424 1C     /mov    eax, dword ptr [esp+1C]
0040565E  |>  85C0         test    eax, eax
00405660  |.  74 18         |je     short 0040567A
00405662  |.  8B5424 28     |mov    edx, dword ptr [esp+28]
00405666  |.  8B7E FA      |mov    edi, dword ptr [esi-6]
00405669  |.  8B4424 2C     |mov    eax, dword ptr [esp+2C]
0040566D  |.  33FA         |xor    edi, edx
0040566F  |.  897E FA      |mov    dword ptr [esi-6], edi
00405672  |.  8B4E FE      |mov    ecx, dword ptr [esi-2]
00405675  |.  33C8         |xor    ecx, eax
00405677  |.  894E FE      |mov    dword ptr [esi-2], ecx
0040567A  |>  8B46 FA      |mov    eax, dword ptr [esi-6]
0040567D  |.  8D7E FA      |lea    edi, dword ptr [esi-6]
00405680  |.  894424 24     |mov    dword ptr [esp+24], eax         ;  JASS: 32bit 交换高低位
00405684  |.  8D5E FE      |lea    ebx, dword ptr [esi-2]
00405687  |.  8A4C24 26     |mov    cl, byte ptr [esp+26]
0040568B  |.  8A5424 27     |mov    dl, byte ptr [esp+27]
0040568F  |.  8846 FD      |mov    byte ptr [esi-3], al
00405692  |.  8866 FC      |mov    byte ptr [esi-4], ah
00405695  |.  884E FB      |mov    byte ptr [esi-5], cl
00405698  |.  8817         |mov    byte ptr [edi], dl
0040569A  |.  8B03         |mov    eax, dword ptr [ebx]
0040569C  |.  53            |push   ebx
0040569D  |.  894424 28     |mov    dword ptr [esp+28], eax
004056A1  |.  8846 01      |mov    byte ptr [esi+1], al
004056A4  |.  8A4C24 2B     |mov    cl, byte ptr [esp+2B]
004056A8  |.  8A4424 2A     |mov    al, byte ptr [esp+2A]
004056AC  |.  880B         |mov    byte ptr [ebx], cl
004056AE  |.  8B4C24 14     |mov    ecx, dword ptr [esp+14]
004056B2  |.  57            |push   edi
004056B3  |.  8826         |mov    byte ptr [esi], ah
004056B5  |.  8846 FF      |mov    byte ptr [esi-1], al
004056B8  |.  E8 33F4FFFF   |call   00404AF0                        ;  JASS:把 8 字节“字符串”转换成二进制串
004056BD  |.  8B07         |mov    eax, dword ptr [edi]
004056BF  |.  894424 24     |mov    dword ptr [esp+24], eax
004056C3  |.  8846 FD      |mov    byte ptr [esi-3], al
004056C6  |.  8A5424 26     |mov    dl, byte ptr [esp+26]
004056CA  |.  8A4424 27     |mov    al, byte ptr [esp+27]
004056CE  |.  8866 FC      |mov    byte ptr [esi-4], ah
004056D1  |.  8856 FB      |mov    byte ptr [esi-5], dl
004056D4  |.  8807         |mov    byte ptr [edi], al
004056D6  |.  8B03         |mov    eax, dword ptr [ebx]
004056D8  |.  894424 24     |mov    dword ptr [esp+24], eax
004056DC  |.  8846 01      |mov    byte ptr [esi+1], al
004056DF  |.  8A4C24 26     |mov    cl, byte ptr [esp+26]
004056E3  |.  8A5424 27     |mov    dl, byte ptr [esp+27]
004056E7  |.  8826         |mov    byte ptr [esi], ah
004056E9  |.  8B4424 1C     |mov    eax, dword ptr [esp+1C]
004056ED  |.  85C0         |test   eax, eax
004056EF  |.  884E FF      |mov    byte ptr [esi-1], cl
004056F2  |.  8813         |mov    byte ptr [ebx], dl
004056F4  |.  74 0D         |je     short 00405703
004056F6  |.  8B07         |mov    eax, dword ptr [edi]
004056F8  |.  8B4F 04      |mov    ecx, dword ptr [edi+4]
004056FB  |.  894424 28     |mov    dword ptr [esp+28], eax
004056FF  |.  894C24 2C     |mov    dword ptr [esp+2C], ecx
00405703  |>  83C6 08      |add    esi, 8
00405706  |.  4D            |dec    ebp
00405707  |.^ 0F85 4DFFFFFF \jnz    0040565A
0040570D  |.  8B4424 18     mov    eax, dword ptr [esp+18]
00405711  |>  5F            pop     edi
00405712  |.  5E            pop     esi
00405713  |.  5D            pop     ebp
00405714  |.  5B            pop     ebx
00405715  |.  59            pop     ecx
00405716  \\.  C2 1800       retn    18
```

循环中间的函数 `00404A4F` 很长但很有规律，里面没有更深层的函数调用，有很多异或运算。主要的运算就是分别对那 `8` 个 `byte` 的前 `4byte` 和后 `4byte` 进行异或迭代，其中还用到了一些“常量”数组加入异或运算。

#### 代码
