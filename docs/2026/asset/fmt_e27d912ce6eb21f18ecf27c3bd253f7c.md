
```
0040D199    C645 E1 6C      mov     byte ptr [ebp-1F], 6C
0040D19D    C645 E2 6F      mov     byte ptr [ebp-1E], 6F
0040D1A1    C645 E3 63      mov     byte ptr [ebp-1D], 63            ; VirtualAlloc
0040D1A5    897D E4         mov     dword ptr [ebp-1C], edi
0040D1A8    FFD0            call    eax                              ; GetProcAddress
0040D1AA    3BC6            cmp     eax, esi                         ; eax = 7C809AE1 也就是VirtualAlloc函数地址
0040D1AC    74 46           je      short 0040D1F4
0040D1AE    8945 F0         mov     dword ptr [ebp-10], eax          ; 将函数VirtualAlloc地址存入ebp-10
0040D1B1    8B4D FC         mov     ecx, dword ptr [ebp-4]
0040D1B4    8B45 F8         mov     eax, dword ptr [ebp-8]
0040D1B7    8D55 D8         lea     edx, dword ptr [ebp-28]
0040D1BA    52              push    edx
0040D1BB    51              push    ecx
0040D1BC    C645 D8 56      mov     byte ptr [ebp-28], 56
0040D1C0    C645 D9 69      mov     byte ptr [ebp-27], 69
0040D1C4    C645 DA 72      mov     byte ptr [ebp-26], 72
0040D1C8    C645 DB 74      mov     byte ptr [ebp-25], 74
0040D1CC    C645 DC 75      mov     byte ptr [ebp-24], 75
0040D1D0    C645 DD 61      mov     byte ptr [ebp-23], 61
0040D1D4    C645 DE 6C      mov     byte ptr [ebp-22], 6C
0040D1D8    C645 DF 46      mov     byte ptr [ebp-21], 46
0040D1DC    C645 E0 72      mov     byte ptr [ebp-20], 72
0040D1E0    C645 E1 65      mov     byte ptr [ebp-1F], 65
0040D1E4    C645 E2 65      mov     byte ptr [ebp-1E], 65            ; VirtualFree
0040D1E8    897D E3         mov     dword ptr [ebp-1D], edi
0040D1EB    FFD0            call    eax                              ; GetProcAddress
0040D1ED    3BC6            cmp     eax, esi                         ; eax = 7C809B74也就是VirtualAlloc函数的地址
0040D1EF    74 03           je      short 0040D1F4
0040D1F1    8945 EC         mov     dword ptr [ebp-14], eax          ; 将函数VirtualAlloc地址存入ebp-14
```

上面代码是得到三个非常重要的函数的地址`LoadLibraryA`, `VirtualAlloc`, `VirtualFree`，并将它们分别存入`ebp-c`, `ebp-10`, `ebp-14`之中，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201101/288797_y374sj1ttw00pfy.jpg)

然后是一些花指令吧~~

```
0040D1F4    90              nop
0040D1F5    90              nop
0040D1F6    8BC0            mov     eax, eax
0040D1F8    90              nop
0040D1F9    90              nop
```

接着看到一个`Call`如下，进去看看：

```
0040D1FA    E8 10000000     call    0040D20F----->这是一个近CALL，F7进入，到下面的地址处：
------------------------------------------------------------------------------------------
0040D20F    5E              pop     esi                              ; 0040D1FF
0040D210    8BFE            mov     edi, esi
0040D212    81EF 9B010000   sub     edi, 19B
0040D218    57              push    edi
0040D219    6A 04           push    4
0040D21B    BF 00100000     mov     edi, 1000
0040D220    57              push    edi
0040D221    57              push    edi                                     ; size = 1000
0040D222    6A 00           push    0
0040D224    FF55 F0         call    dword ptr [ebp-10]               ; VirtualAlloc
0040D227    59              pop     ecx                              ; 40D064
0040D228    BB 00800000     mov     ebx, 8000
0040D22D    53              push    ebx
0040D22E    6A 00           push    0
0040D230    50              push    eax                              ; VirtualAlloc得到的分配的空间基址380000
0040D231    51              push    ecx                              ; 40D064
0040D232    46              inc     esi
0040D233    8B16            mov     edx, dword ptr [esi]             ; edx = 00400000
0040D235    52              push    edx
0040D236    50              push    eax
0040D237    50              push    eax
0040D238    8B46 06         mov     eax, dword ptr [esi+6]           ; eax = 15E2
0040D23B    03C2            add     eax, edx
0040D23D    8BF0            mov     esi, eax                         ; esi = eax = 004015E2
0040D23F    8B00            mov     eax, dword ptr [eax]             ; eax = 00004794
0040D241    03C2            add     eax, edx
0040D243    8941 05         mov     dword ptr [ecx+5], eax           ; [0040D069] = 00404794
0040D246    58              pop     eax
0040D247    8946 04         mov     dword ptr [esi+4], eax           ; [004015E6] = 00380000
0040D24A    8B46 0C         mov     eax, dword ptr [esi+C]
0040D24D    03C2            add     eax, edx
0040D24F    50              push    eax                              ; eax = 004010E8
0040D250    8B46 10         mov     eax, dword ptr [esi+10]
0040D253    03C2            add     eax, edx
0040D255    FFD0            call    eax                              ; call 0040101F----->这里进入看看~~
--------------------------------------------------------------------------------------------------------------
```

```
0040101C   .  15 24906050   adc     eax, 50609024
00401021   .  52            push    edx
00401022   .  8B4424 2C     mov     eax, dword ptr [esp+2C]             ;  eax = 004010E8
00401026   .  8BF0          mov     esi, eax                            ;  esi = eax = 004010E8
00401028   .  8B5424 30     mov     edx, dword ptr [esp+30]
0040102C   .  8BFA          mov     edi, edx                            ;  edi = edx = 00380000
0040102E   .  5A            pop     edx                                 ;  edx = 00400000
0040102F   .  58            pop     eax                                 ;  eax = 0040101F
00401030   .  FC            cld                                         ;  DF = 0
00401031   .  60            pushad
00401032   >  8A16          mov     dl, byte ptr [esi]                  ;  逐个取esi = 004010E8地址每位数
00401034   .  84D2          test    dl, dl                              ;  判断是否为零
00401036   .  74 08         je      short 00401040
00401038   .  80F2 15       xor     dl, 15                              ;  不为空就与15做异或操作
0040103B   .  8816          mov     byte ptr [esi], dl                  ;  异或之后，重新填入到004010E8相应位置
0040103D   .  46            inc     esi                                 ;  逐个取出
0040103E   .^ EB F2         jmp     short 00401032                      ;  循环
00401040   >  61            popad
00401041   .  33D2          xor     edx, edx                            ;  edx = 0
00401043   .  B3 80         mov     bl, 80                              ;  bl = 80
00401045   >  A4            movs    byte ptr es:[edi], byte ptr [esi]   ;  将刚才加密的esi(004010E8)中第一个数据拷贝到edi(00380000)中
00401046   .  B2 02         mov     dl, 2
00401048   > /E8 71000000   call    004010BE                                ------->由于后面有几处都调用了这个函数，先分析一下吧~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
004010BE  /$  02DB          add     bl, bl                       ;  bl+bl看是否为零
004010C0  |.  75 05         jnz     short 004010C7               ;  不为零就跳走
004010C2  |.  8A1E          mov     bl, byte ptr [esi]           ;  为零就将esi中的下一个数据取出一个传入给bl
004010C4  |.  46            inc     esi                          ;  esi++
004010C5  |.  10DB          adc     bl, bl                       ;  bl+bl带进位相加
004010C7  \>  C3            retn                                 ;  返回
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
0040104D   .  8BC0          mov     eax, eax
0040104F   .^ 73 F4         jnb     short 00401045                              ;  循环，根据前面的规则将esi中的一个数据传入到edi中
00401051   .  33C9          xor     ecx, ecx
00401053   .  E8 66000000   call    004010BE
00401058   .  73 1E         jnb     short 00401078
0040105A   .  33C0          xor     eax, eax
0040105C   .  E8 5D000000   call    004010BE
00401061   .  8BC0          mov     eax, eax
00401063   .  73 23         jnb     short 00401088
00401065   .  B2 02         mov     dl, 2
00401067   .  41            inc     ecx
00401068   .  B0 10         mov     al, 10                                      ;  进行四次循环，使al = 10
```
