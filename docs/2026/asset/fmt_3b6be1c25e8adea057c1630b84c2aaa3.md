
；送CS+10A

```
004682C3    8D340A             lea     esi,dword ptr ds:[edx+ecx]        ; 垃圾代码
004682C6    BA 926B7F56        mov     edx,567F6B92                      ; 垃圾代码
004682CB    BF 7B62BA5E        mov     edi,5EBA627B                      ; 垃圾代码
004682D0    81C5 3C34D76D      add     ebp,6DD7343C                      ; 垃圾代码
004682D6    B9 DEAFC704        mov     ecx,4C7AFDE                       ; 垃圾代码
004682DB    4B                 dec     ebx
004682DC    83EB FB            sub     ebx,-5                            ; CS+10E
004682DF    3BC1               cmp     eax,ecx                           ; 垃圾代码
004682E1    25 A981CB14        and     eax,14CB81A9                      ; 垃圾代码
004682E6    812B 75F8C422      sub     dword ptr ds:[ebx],22C4F875       ; 计算的结果是（1c758d73）
```
，送CS+10E

```
004682EC    8DA8 E3F5B334      lea     ebp,dword ptr ds:[eax+34B3F5E3]   ; 垃圾代码
004682F2    BA B802F97D        mov     edx,7DF902B8                      ; 垃圾代码
004682F7    B9 D7711D1D        mov     ecx,1D1D71D7                      ; 垃圾代码
004682FC    4B                 dec     ebx
004682FD    83EB FB            sub     ebx,-5                            ; CS+112
00468300    8DA8 1A4E0E7B      lea     ebp,dword ptr ds:[eax+7B0E4E1A]   ; 垃圾代码
00468306    B8 71037F33        mov     eax,337F0371                      ; 垃圾代码
0046830B    3BC1               cmp     eax,ecx                           ; 垃圾代码
0046830D    8D92 51A10971      lea     edx,dword ptr ds:[edx+7109A151]   ; 垃圾代码
00468313    51                 push    ecx                               ; 垃圾代码
00468314    59                 pop     ecx                               ; CS+EE，垃圾，清空前面假CALL入的堆栈
00468315    81F7 40079F60      xor     edi,609F0740                      ; 垃圾代码
0046831B    812B 9B1E105D      sub     dword ptr ds:[ebx],5D101E9B       ; 计算的结果是（55ff5056），送CS+112
00468321    BF AF102C2A        mov     edi,2A2C10AF                      ; 垃圾代码
00468326    8D8E BDEC0965      lea     ecx,dword ptr ds:[esi+6509ECBD]   ; 垃圾代码
0046832C    F7C2 A16EF328      test    edx,28F36EA1                      ; 垃圾代码
00468332    B8 4A11E162        mov     eax,62E1114A                      ; 垃圾代码
00468337    25 E2686047        and     eax,476068E2                      ; 垃圾代码
0046833C    83EB FC            sub     ebx,-4                            ; CS+116
0046833F    B9 E49E9030        mov     ecx,30909EE4                      ; 垃圾代码
00468344    8103 DF92742D      add     dword ptr ds:[ebx],2D7492DF       ; 计算的结果是（2c458974），送CS+116
0046834A    43                 inc     ebx
0046834B    43                 inc     ebx
0046834C    43                 inc     ebx
0046834D    43                 inc     ebx                               ; CS+11A
0046834E    BF 58506F78        mov     edi,786F5058                      ; 垃圾代码
00468353    81F7 DD57125B      xor     edi,5B1257DD                      ; 垃圾代码
00468359    3BC1               cmp     eax,ecx                           ; 垃圾代码
0046835B    81F7 B6D6FF2B      xor     edi,2BFFD6B6                      ; 垃圾代码
00468361    81CE FF4FA918      or      esi,18A94FFF                      ; 垃圾代码
00468367    8D85 D1616900      lea     eax,dword ptr ss:[ebp+6961D1]     ; 垃圾代码
0046836D    4F                 dec     edi                               ; 垃圾代码
0046836E    8103 33A7550A      add     dword ptr ds:[ebx],0A55A733       ; 计算的结果是（0068046a），送CS+11a
00468374    81F7 2A21832A      xor     edi,2A83212A                      ; 垃圾代码
0046837A    8D85 308D2C48      lea     eax,dword ptr ss:[ebp+482C8D30]   ; 垃圾代码
00468380    3BC1               cmp     eax,ecx                           ; 垃圾代码
00468382    B9 ECB5265A        mov     ecx,5A26B5EC                      ; 垃圾代码
00468387    81CA 67E14F73      or      edx,734FE167                      ; 垃圾代码
0046838D    4B                 dec     ebx
0046838E    83EB FB            sub     ebx,-5                            ; CS+11E
00468391    8D85 90B22541      lea     eax,dword ptr ss:[ebp+4125B290]   ; 垃圾代码
00468397    8DA8 EE62926D      lea     ebp,dword ptr ds:[eax+6D9262EE]   ; 垃圾代码
0046839D    25 BA289002        and     eax,29028BA                       ; 垃圾代码
004683A2    BE 2C267320        mov     esi,2073262C                      ; 垃圾代码
004683A7    812B 75F43E3B      sub     dword ptr ds:[ebx],3B3EF475       ; 计算的结果是（FF000010），送CS+11E
004683AD    25 56AC092D        and     eax,2D09AC56                      ; 垃圾代码
004683B2    8D340A             lea     esi,dword ptr ds:[edx+ecx]        ; 垃圾代码
004683B5    3BC1               cmp     eax,ecx                           ; 垃圾代码
004683B7    8D8E 31CE7824      lea     ecx,dword ptr ds:[esi+2478CE31]   ; 垃圾代码
004683BD    B9 04EA210A        mov     ecx,0A21EA04                      ; 垃圾代码
004683C2    B8 65563C31        mov     eax,313C5665                      ; 垃圾代码
004683C7    E8 02000000        call    jl.004683CE                       ; 虚跳CALL，花指令
004683CE    5E                 pop     esi                               ; 无意义
004683CF    4B                 dec     ebx
004683D0    83EB FB            sub     ebx,-5                            ; CS +122
004683D3    E8 02000000        call    jl.004683DA                       ; 虚call花指令,无意义
```

.............
一路都是这样解码＋无意义的垃圾代码
.............

```
0046947A    FF0424             inc     dword ptr ss:[esp]                ; 解码结束，返回地址为4670ee
```
；开始从解码后的代码处执行代码
；以后看见这个壳入手就看第一个
CALL 中ESI 值，直接跳ESI这里往下分析

```
0046947D    C3                 retn
```

小结：这样一段是开始，一上来就可以看出来，这个壳利用动态生产代码＋大量的垃圾
代码来迷惑跟踪的分析，入手就给很多普通的DEBUG 者一个下马威，这才是万里长征的
开始，刚刚打好行囊，我们准备上路了 :)

```
004670EE   /EB 5D              jmp     short jl.0046714D
004670F0   |8B45 00            mov     eax,dword ptr ss:[ebp]
004670F3   |0BC0               or      eax,eax                           ;
004670F5   |74 04              je      short jl.004670FB
004670F7   |55                 push    ebp
004670F8   |FF65 0C            jmp     near dword ptr ss:[ebp+C]
004670FB   |FF45 00            inc     dword ptr ss:[ebp]
004670FE   |8B4424 24          mov     eax,dword ptr ss:[esp+24]         ;
00467102   |8945 04            mov     dword ptr ss:[ebp+4],eax          ;
00467105   |8DB5 84000000      lea     esi,dword ptr ss:[ebp+84]         ; kernel32.dll,把dll的名称保存地址放到esi中
0046710B   |56                 push    esi
0046710C   |FF55 78            call    near dword ptr ss:[ebp+78]        ; kernel32.GetModuleHandleA
0046710F   |8D75 1C            lea     esi,dword ptr ss:[ebp+1C]         ; 把VirtualAlloc名称所在的地址传到esi中
00467112   |56                 push    esi
00467113   |50                 push    eax                               ;
00467114   |FF55 74            call    near dword ptr ss:[ebp+74]        ;  kernel32.GetProcAddress
00467117   |8945 2C            mov     dword ptr ss:[ebp+2C],eax         ; VirtuAlloc地址保存
0046711A   |6A 04              push    4
0046711C   |68 00100000        push    1000
00467121   |FF75 10            push    dword ptr ss:[ebp+10]             ; 申请空间大小17951
00467124   |6A 00              push    0
00467126   |FF55 2C            call    near dword ptr ss:[ebp+2C]        ; 申请空间,VirtualAlloc
00467129 > |50                 push    eax                               ; 申请地址入栈(370000)
0046712A   |8945 0C            mov     dword ptr ss:[ebp+C],eax          ; 申请地址
0046712D   |8B5D 08            mov     ebx,dword ptr ss:[ebp+8]          ; 解码代码相对偏移2499
00467130   |03DD               add     ebx,ebp                           ; 把RVA转换成VA（469499)
```
