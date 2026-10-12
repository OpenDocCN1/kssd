
# 一个软件的另类注册，呵呵，可以说另类吗？

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-70.htm`](https://www.pediy.com/kssd/pediy07/pediy7-70.htm)

【破解作者】 落魄浪子  
【作者邮箱】 `zxy223_szb@21cn.net`  
【使用工具】 `FlyOD1.1`  
【破解平台】 `Win9x/NT/2000/XP`  
【软件名称】 图片猎人  
【下载地址】 [`www.enova-soft.com`](http://www.enova-soft.com)  
【软件简介】 《图片猎人》（`Pix Hunter`）是一个专门用于从 Internet 上搜索和下载、管理图片的工具软件。它保持了“网际快车”、“网络蚂蚁”等下载软件操作简便、下载迅速等优点，并扩展很多功能：如智能搜索、图片浏览、图片管理等等。这是一款集成了众多同类软件的优点，并增加了更多功能的图片下载专用工具软件。

简而言之，《图片猎人》就象是下载工具 `ACDSee` 和下载工具 `FlashGet` 的完美结合。

【加壳方式】 `Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks`

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

\--------------------------------------------------------------------------------

【破解内容】

脱文请看这里：[`bbs.pediy.com/showthread.php...;threadid=10131`](http://bbs.pediy.com/showthread.php?s=&threadid=10131)

这个软件加的是 `Armadillo` 的壳，我已经把她脱了，脱文发在脱壳板块，有兴趣可以去看看，现在我要让她自已告诉你注册码，并且自注册，以前没有这样做过，现在来试试，呵呵，错误之处望高手指正。

用 `OD` 载入脱壳之后的程序，运行，注册码不对时会弹出错误提示，找到出错字符串，向上看看哪里可以跳过。

```
00541B12      E8 99E8FFFF        call CR_dumpe.005403B0             关键 CALL，进入
00541B17      84C0               test al,al
00541B19      75 38              jnz short CR_dumpe.00541B53        这里不跳就出错
00541B1B      6A 00              push 0
00541B1D      66:8B0D B41B5400   mov cx,word ptr ds:[541BB4]
00541B24      B2 01              mov dl,1
00541B26      B8 081C5400        mov eax,CR_dumpe.00541C08                 ; ASCII "Invalid Release Code.  Please check your entry and try again."
00541B2B      E8 D0CBF1FF        call CR_dumpe.0045E700
00541B30      8B45 FC            mov eax,dword ptr ss:[ebp-4]

005403B0      55                 push ebp          进入后来到这里
005403B1      8BEC               mov ebp,esp
005403B3      83C4 E8            add esp,-18
005403B6      53                 push ebx
005403B7      33C9               xor ecx,ecx
005403B9      894D F0            mov dword ptr ss:[ebp-10],ecx
005403BC      894D EC            mov dword ptr ss:[ebp-14],ecx
005403BF      894D E8            mov dword ptr ss:[ebp-18],ecx
005403C2      8BDA               mov ebx,edx                 预置数 AD76C 即 710508
005403C4      8945 FC            mov dword ptr ss:[ebp-4],eax
005403C7      8B45 FC            mov eax,dword ptr ss:[ebp-4]
005403CA      E8 F13EECFF        call CR_dumpe.004042C0
```

\--------------------略过------------------------------------------------

```
005403FB      8B55 FC            mov edx,dword ptr ss:[ebp-4]        输入的假的注册码入 EDX
005403FE      B8 98045400        mov eax,CR_dumpe.00540498
00540403      E8 F03FECFF        call CR_dumpe.004043F8
00540408      85C0               test eax,eax
0054040A    ^ 7F D3              jg short CR_dumpe.005403DF
0054040C      FF35 7C725800      push dword ptr ds:[58727C]
00540412      FF35 78725800      push dword ptr ds:[587278]
00540418      8D4D F4            lea ecx,dword ptr ss:[ebp-C]
0054041B      B8 68725800        mov eax,CR_dumpe.00587268
00540420      8BD3               mov edx,ebx             预置数 AD76C 即 710508
00540422      E8 0963FFFF        call CR_dumpe.00536730      进入看看

00536730      55                 push ebp          进入后来到这里
00536731      8BEC               mov ebp,esp
00536733      53                 push ebx
00536734      56                 push esi
00536735      57                 push edi
00536736      8BD9               mov ebx,ecx
00536738      8BFA               mov edi,edx
0053673A      8BF0               mov esi,eax
0053673C      66:C703 693C       mov word ptr ds:[ebx],3C69    3C69 即 i<
00536741      FF75 0C            push dword ptr ss:[ebp+C]
00536744      FF75 08            push dword ptr ss:[ebp+8]
00536747      E8 8CFFFFFF        call CR_dumpe.005366D8
0053674C      66:8943 02         mov word ptr ds:[ebx+2],ax
00536750      897B 04            mov dword ptr ds:[ebx+4],edi
00536753      8BD3               mov edx,ebx
00536755      8BC6               mov eax,esi
00536757      B1 01              mov cl,1
00536759      E8 6EF9FFFF        call CR_dumpe.005360CC     进入看看

005360CC      53                 push ebx 进入后来到这里，压入 i<即 693C
005360CD      56                 push esi
005360CE      57                 push edi                  压入 AD76C 即 710508
005360CF      83C4 E8            add esp,-18
005360D2      884C24 08          mov byte ptr ss:[esp+8],cl
005360D6      895424 04          mov dword ptr ss:[esp+4],edx
005360DA      890424             mov dword ptr ss:[esp],eax
005360DD      8B4424 04          mov eax,dword ptr ss:[esp+4]
005360E1      8B00               mov eax,dword ptr ds:[eax]
005360E3      894424 0C          mov dword ptr ss:[esp+C],eax
005360E7      8B4424 04          mov eax,dword ptr ss:[esp+4]
005360EB      8B40 04            mov eax,dword ptr ds:[eax+4]
005360EE      894424 10          mov dword ptr ss:[esp+10],eax
005360F2      C74424 14 0400000>mov dword ptr ss:[esp+14],4

005360FA      BE 74D35700        mov esi,CR_dumpe.0057D374        查表数地址入 ESI
005360FF      8B5424 0C          mov edx,dword ptr ss:[esp+C]     693C 入 EDX
00536103      33C0               xor eax,eax
00536105      8A4424 08          mov al,byte ptr ss:[esp+8]         查表数 1 入 AL 记为 S1
00536109      8BD8               mov ebx,eax
0053610B      03DB               add ebx,ebx                         S1+S1 结果记为 N1
0053610D      8D1C5B             lea ebx,dword ptr ds:[ebx+ebx*2]      N1+N1*2 记为 N2
00536110      8B04DE             mov eax,dword ptr ds:[esi+ebx*8]       ESI＝0057D374+N2*8＝查表得 3 记为 N3
00536113      8B0C24             mov ecx,dword ptr ss:[esp] 查表数地址 00587268 入 ECX
00536116      8B0C81             mov ecx,dword ptr ds:[ecx+eax*4]  ECX＝00587268+N3*4＝查表得 F22C6843 记为 N4
00536119      8B44DE 04          mov eax,dword ptr ds:[esi+ebx*8+4] ESI＝0057D374+N2*8+4＝查表得 2 记为 N5
0053611D      8B3C24             mov edi,dword ptr ss:[esp] 查表数地址 00587268 入 EDI
00536120      8B0487             mov eax,dword ptr ds:[edi+eax*4] EDI＝00587268+N5*4＝查表得 0ABCE064 记为 N6
00536123      8B5CDE 08          mov ebx,dword ptr ds:[esi+ebx*8+8] ESI＝0057D374+N2*8+8＝查表得 0 记为 N7
00536127      8B3C24             mov edi,dword ptr ss:[esp]   查表数地址 00587268 入 EDI
0053612A      8B1C9F             mov ebx,dword ptr ds:[edi+ebx*4] EDI＝00587268+N7*4＝查表得 4025BBCA 记为 N8
0053612D      03D3               add edx,ebx   N8+3C69 结果 4025F833 记为 N9
0053612F      03DA               add ebx,edx   N9+N8 结果 804BB3FD 记为 N10
00536131      8BFA               mov edi,edx
00536133      C1EF 07            shr edi,7    N9 右移 7 次结果 00804BF0 记为 N11
00536136      33D7               xor edx,edi   N9 与 N11 作异或运算结果 40A5B3C3 记为 N12
00536138      03CA               add ecx,edx   N12+N4 结果 32D21C06 记为 N13
0053613A      03D1               add edx,ecx   N12+N13 结果 7377CFC9 记为 N14
0053613C      8BF9               mov edi,ecx   N13 入 EDI
0053613E      C1E7 0D            shl edi,0D    N13 左移 13 次结果 4380C000 记为 N15
00536141      33CF               xor ecx,edi   N13 与 N15 作异或运算结果 7152DC06 记为 N16
00536143      03C1               add eax,ecx   N6+N16 结果 7C0FBC6A 记为 N17
00536145      03C8               add ecx,eax   N16+N17 结果 ED629870 记为 N18
00536147      8BF8               mov edi,eax   N18 放入 EDI
00536149      C1EF 11            shr edi,11    N18 右移 17 次结果 00003E07 记为 N19
0053614C      33C7               xor eax,edi   N17 与 N19 作异或运算结果 7C0F826D 记为 N20
0053614E      03D8               add ebx,eax   N10+N20 结果 FC5B366A 记为 N21
00536150      03C3               add eax,ebx   N20+N21 结果 786AB8D7 记为 N22
00536152      8BFB               mov edi,ebx   N21 放入 EDI
00536154      C1E7 09            shl edi,9     N21 左移 9 次结果 B66CD400 记为 N23
00536157      33DF               xor ebx,edi   N21 与 N23 作异或运算结果 4A37E26A 记为 N24
00536159      03D3               add edx,ebx   N14+N24 结果 BDAFB233 记为 N25
0053615B      03DA               add ebx,edx   N24+N25 结果 07E7949D 记为 N26
```



```
0053615D     8BFA               mov edi,edx   N25 放入 EDI
0053615F     C1EF 03           shr edi,3     N25 右移 3 次结果 17B5F646 记为 N27
00536162     33D7              xor edx,edi   N25 与 N27 作异或运算结果 AA1A4475 记为 N28
00536164     03CA              add ecx,edx   N18+N28 结果 977CDCE5 记为 N29
00536166     8BD1              mov edx,ecx   N29 放入 EDX
00536168     C1E2 07          shl edx,7     N29 左移 7 次结果 BE6E7280 记为 N30
0053616B     33CA              xor ecx,edx   N29 与 N30 作异或运算结果 2912AE65 记为 N31
0053616D     03C1              add eax,ecx   N22+N31 结果 A17D673C 记为 N32
0053616F     8BD3              mov edx,ebx   N32 放入 EDX
00536171     C1EA 0F          shr edx,0F    N32 右移 15 次结果 00000FCF 记为 N33
00536174     33C2              xor eax,edx   N32 与 N33 作异或运算结果 A17D68F3 记为 N34
00536176     03D8              add ebx,eax   N26+N34 结果 A964FD90 记为 N35
00536178     8BC3              mov eax,ebx   N35 放入 EAX
0053617A     C1E0 0B          shl eax,0B    N35 左移 11 次结果 27EC8000 记为 N36
0053617D     33D8              xor ebx,eax   N35 与 N36 作异或运算结果 8E887D90 记为 N37
0053617F     8B4424 10        mov eax,dword ptr ss:[esp+10]     AD76C 放入 EAX
00536183     33C3              xor eax,ebx              AD76C 与 N37 作异或运算结果 8E82AAFC 记为 N38
00536185     8B5424 0C        mov edx,dword ptr ss:[esp+C]      3C69 放入 EDX
00536189     895424 10        mov dword ptr ss:[esp+10],edx     
0053618D     894424 0C        mov dword ptr ss:[esp+C],eax
00536191     83C6 0C          add esi,0C                    ESI＝0057D374+C
00536194     FF4C24 14        dec dword ptr ss:[esp+14]          计数器减 1
00536198     0F85 61FFFFFF    jnz CR_dumpe.005360FF           没有计算完则继续 ; 这样循环计算 4 次
0053619E     8B4424 04        mov eax,dword ptr ss:[esp+4]     存放注册码的地址入 EAX
005361A2     8B5424 10        mov edx,dword ptr ss:[esp+10]    注册码的前 8 位入 EDX  
005361A6     8910               mov dword ptr ds:[eax],edx      注册码的前 8 位放入 EAX 的内存地址
005361A8     8B4424 04        mov eax,dword ptr ss:[esp+4]     存放注册码的地址入 EAX
005361AC     8B5424 0C        mov edx,dword ptr ss:[esp+C]     注册码的后 8 位入 EDX
005361B0     8950 04          mov dword ptr ds:[eax+4],edx     注册码的后 8 位放入 EAX 的内存地址
005361B3     83C4 18          add esp,18
005361B6     5F                pop edi
005361B7     5E                pop esi
005361B8     5B                pop ebx
005361B9     C3                retn                 返回
0053675E     5F                pop edi 返回到这里
0053675F     5E                pop esi
00536760     5B                pop ebx
00536761     5D                pop ebp
00536762     C2 0800           retn 8         返回
00540427     8D55 F0           lea edx,dword ptr ss:[ebp-10]返回到这里
0054042A     8B45 FC            mov eax,dword ptr ss:[ebp-4]   输入的假注册码
0054042D     E8 D28DECFF       call CR_dumpe.00409204
00540432     8B45 F0            mov eax,dword ptr ss:[ebp-10]
00540435     50                push eax
00540436     8D4D E8           lea ecx,dword ptr ss:[ebp-18]
00540439     8D45 F4           lea eax,dword ptr ss:[ebp-C]
0054043C     BA 08000000       mov edx,8
00540441     E8 025AFFFF       call CR_dumpe.00535E48
00540446     8B45 E8            mov eax,dword ptr ss:[ebp-18] 真注册码入 EAX，这里可以做个内存注册机
00540449     8D55 EC           lea edx,dword ptr ss:[ebp-14]
0054044C     E8 B38DECFF       call CR_dumpe.00409204        这个 CALL 是检查注册的位数，16 位  
00540451     8B55 EC            mov edx,dword ptr ss:[ebp-14]
00540454     58                pop eax                 输入的假注册码
00540455     E8 C23DECFF       call CR_dumpe.0040421C      进入
0040421C     53                push ebx  进入后来到这里
0040421D     56                push esi
0040421E     57                push edi
0040421F     89C6               mov esi,eax   假注册码
00404221     89D7               mov edi,edx   真注册码
00404223     39D0               cmp eax,edx   真假注册码作比较
00404225     0F84 8F000000     je CR_dumpe.004042BA   
0040422B     85F6               test esi,esi
0040422D     74 68              je short CR_dumpe.00404297
0040422F     85FF               test edi,edi
00404231     74 6B              je short CR_dumpe.0040429E
00404233     8B46 FC            mov eax,dword ptr ds:[esi-4] 假注册码的长度入 EAX 记为 LEN1
00404236     8B57 FC            mov edx,dword ptr ds:[edi-4]   真注册码的长度入 EDX 记为 LEN2
00404239     29D0               sub eax,edx                 LEN1-LEN2
0040423B     77 02              ja short CR_dumpe.0040423F
0040423D     01C2               add edx,eax
0040423F     52                push edx
00404240     C1EA 02           shr edx,2        取注册码的位数
00404243     74 26              je short CR_dumpe.0040426B
00404245     8B0E               mov ecx,dword ptr ds:[esi]   假注册码的前四位
00404247     8B1F               mov ebx,dword ptr ds:[edi]   真注册码的前四位
00404249     39D9               cmp ecx,ebx               比较是否相等
0040424B     75 58              jnz short CR_dumpe.004042A5  不等则 GAME OVER
0040424D     4A                dec edx
0040424E     74 15              je short CR_dumpe.00404265
00404250     8B4E 04            mov ecx,dword ptr ds:[esi+4]
00404253     8B5F 04            mov ebx,dword ptr ds:[edi+4]
00404256     39D9               cmp ecx,ebx
00404258     75 4B              jnz short CR_dumpe.004042A5  不等则 GAME OVER
0040425A     83C6 08            add esi,8
0040425D     83C7 08            add edi,8
00404260     4A                dec edx
00404261     75 E2              jnz short CR_dumpe.00404245  是否比较完了，没有则继续
00404263     EB 06              jmp short CR_dumpe.0040426B

-------------------------略过---------------------------------

00404293     01C0               add eax,eax   输入的注册码的长度不能超过计算出的注册码的长度，否则不能注册
00404295     EB 23              jmp short CR_dumpe.004042BA
00404297     8B57 FC            mov edx,dword ptr ds:[edi-4]
0040429A     29D0               sub eax,edx
0040429C     EB 1C              jmp short CR_dumpe.004042BA
0040429E     8B46 FC            mov eax,dword ptr ds:[esi-4]
004042A1     29D0               sub eax,edx
004042A3     EB 15              jmp short CR_dumpe.004042BA
004042A5     5A                pop edx
004042A6     38D9               cmp cl,bl
004042A8     75 10              jnz short CR_dumpe.004042BA
004042AA     38FD               cmp ch,bh
004042AC     75 0C              jnz short CR_dumpe.004042BA
004042AE     C1E9 10            shr ecx,10
004042B1     C1EB 10            shr ebx,10
004042B4     38D9               cmp cl,bl
004042B6     75 02              jnz short CR_dumpe.004042BA
004042B8     38FD               cmp ch,bh
004042BA     5F                pop edi
004042BB     5E                pop esi
004042BC     5B                pop ebx
004042BD     C3                retn        返回
0054045A     0F94C3             sete bl    返回到这里置注册标置位，0 为注册错误，1 为注册正确
0054045D     33C0               xor eax,eax
0054045F     5A                pop edx
00540460     59                pop ecx
00540461     59                pop ecx
00540462     64:8910            mov dword ptr fs:[eax],edx
00540465     68 87045400        push CR_dumpe.00540487
0054046A     8D45 E8            lea eax,dword ptr ss:[ebp-18]
0054046D     BA 03000000        mov edx,3
00540472     E8 1D3AECFF        call CR_dumpe.00403E94
00540477     8D45 FC            lea eax,dword ptr ss:[ebp-4]
0054047A     E8 F139ECFF        call CR_dumpe.00403E70
0054047F     C3                retn 返回
00540480     E9 2333ECFF        jmp CR_dumpe.004037A8
00540485     EB E3              jmp short CR_dumpe.0054046A
00540487     8BC3               mov eax,ebx
00540489     5B                pop ebx
0054048A     8BE5               mov esp,ebp
0054048C     5D                pop ebp
0054048D     C3                retn  返回
```



`00541B17` `84C0` `test al,al` 返回到这里

`00541B19` `75 38` `jnz short CR_dumpe.00541B53` 一定要跳，不跳就 `GAME OVER`

`00541B1B` `6A 00` `push 0`

`00541B1D` `66:8B0D B41B5400` `mov cx,word ptr ds:[541BB4]`

`00541B24` `B2 01` `mov dl,1`

`00541B26` `B8 081C5400` `mov eax,CR_dumpe.00541C08` ; `ASCII "Invalid Release Code.  Please check your entry and try again."`

`00541B2B` `E8 D0CBF1FF` `call CR_dumpe.0045E700`

`00541B30` `8B45 FC` `mov eax,dword ptr ss:[ebp-4]`

`00541B33` `8B80 E4020000` `mov eax,dword ptr ds:[eax+2E4]`

`00541B39` `8B10` `mov edx,dword ptr ds:[eax]`

`00541B3B` `FF92 B0000000` `call dword ptr ds:[edx+B0]`

`00541B41` `8B45 FC` `mov eax,dword ptr ss:[ebp-4]`

`00541B44` `8B80 E4020000` `mov eax,dword ptr ds:[eax+2E4]`

`00541B4A` `33D2` `xor edx,edx`

`00541B4C` `E8 2B17EFFF` `call CR_dumpe.0043327C`

`00541B51` `EB 38` `jmp short CR_dumpe.00541B8B`

`00541B53` `6A 00` `push 0`

`00541B55` `66:8B0D B41B5400` `mov cx,word ptr ds:[541BB4]`

`00541B5C` `B2 02` `mov dl,2`

`00541B5E` `B8 501C5400` `mov eax,CR_dumpe.00541C50` ; `ASCII "Registration complete."`

`00541B63` `E8 98CBF1FF` `call CR_dumpe.0045E700`

`0057D374` 处的数据：

`0057D374` `00 00 00 00 03 00 00 00 01 00 00 00 02 00 00 00` `.....`
`0057D384` `01 00 00 00 03 00 00 00 01 00 00 00 00 00 00 00` `.....`
`0057D394` `02 00 00 00 03 00 00 00 02 00 00 00 00 00 00 00` `.....`
`0057D3A4` `03 00 00 00 02 00 00 00 00 00 00 00 01 00 00 00` `.....`
`0057D3B4` `00 00 00 00 02 00 00 00 02 00 00 00 01 00 00 00` `.....`
`0057D3C4` `03 00 00 00 00 00 00 00 03 00 00 00 01 00 00 00` `.....`
`0057D3D4` `01 00 00 00 00 00 00 00` `...`

-------------------------------------------------------

`0057D448` 处的 4 组数据：

`00587268` `3E C9 18 78 7D AC AF D3 64 E0 BC 0A 43 68 2C F2` `?砘???桃?`

-----------------------------------------------------------

但 `00587268` 处的数据是因机器的不同而不同的，`OD` 载入程序看看是哪里生成的这些数据，先在转存窗口定位到 `00587268` 处，载入程序时，从 `00585000` 到 `00591FF0` 这里的数据是一片空白，`F8` 慢找，这样好看看是哪里生成这些数据的。

`00578BC8` `C>` `55` `push ebp` 载入程序后停在入口处
`00578BC9` `8BEC` `mov ebp,esp`
`00578BCB` `83C4 F4` `add esp,-0C`
`00578BCE` `53` `push ebx`
`00578BCF` `B8 28865700` `mov eax,CR_dumpe.00578628`
`00578BD4` `E8 57E7E8FF` `call CR_dumpe.00407330` 发现这个 `CALL` 一过之后那些数据就出来了，`F7` 进入
`00407330` `50` `push eax` 来到这里
`00407331` `6A 00` `push 0`
`00407333` `E8 F8FEFFFF` `call <jmp.&kernel32.GetModuleHandleA>`
`00407338` `BA 08915700` `mov edx,CR_dumpe.00579108`
`0040733D` `52` `push edx`
`0040733E` `8905 DC545800` `mov dword ptr ds:[5854DC],eax`
`00407344` `8942 04` `mov dword ptr ds:[edx+4],eax`
`00407347` `C742 08 00000000` `mov dword ptr ds:[edx+8],0`
`0040734E` `C742 0C 00000000` `mov dword ptr ds:[edx+C],0`
`00407355` `E8 8AFFFFFF` `call CR_dumpe.004072E4`
`0040735A` `5A` `pop edx`
`0040735B` `58` `pop eax`
`0040735C` `E8 47C8FFFF` `call CR_dumpe.00403BA8`

----------------------略过------------------------------------

`00403BD1` `E8 72FFFFFF` `call CR_dumpe.00403B48` 进入

----------------------略过-----------------------------------

`00403B48` `55` `push ebp` 来到这里
`00403B49` `8BEC` `mov ebp,esp`
`00403B4B` `53` `push ebx`
`00403B4C` `56` `push esi`
`00403B4D` `57` `push edi`
`00403B4E` `A1 AC545800` `mov eax,dword ptr ds:[5854AC]`
`00403B53` `85C0` `test eax,eax`
`00403B55` `74 4B` `je short CR_dumpe.00403BA2`
`00403B57` `8B30` `mov esi,dword ptr ds:[eax]` 计数长度 `B3`
`00403B59` `33DB` `xor ebx,ebx`
`00403B5B` `8B78 04` `mov edi,dword ptr ds:[eax+4]`
`00403B5E` `33D2` `xor edx,edx`
`00403B60` `55` `push ebp`
`00403B61` `68 8E3B4000` `push CR_dumpe.00403B8E`
`00403B66` `64:FF32` `push dword ptr fs:[edx]`
`00403B69` `64:8922` `mov dword ptr fs:[edx],esp`
`00403B6C` `3BF3` `cmp esi,ebx`
`00403B6E` `7E 14` `jle short CR_dumpe.00403B84`
`00403B70` `8B04DF` `mov eax,dword ptr ds:[edi+ebx*8]`
`00403B73` `43` `inc ebx`
`00403B74` `891D B0545800` `mov dword ptr ds:[5854B0],ebx`
`00403B7A` `85C0` `test eax,eax`
`00403B7C` `74 02` `je short CR_dumpe.00403B80`
`00403B7E` `FFD0` `call eax` 就是这个 `CALL` 动态生成那些代码的，
`00403B80` `3BF3` `cmp esi,ebx`
`00403B82` `^ 7F EC` `jg short CR_dumpe.00403B70` 当 `EBX＝A8～AB` 时上面的 `CALL` 生成 `00587268` 处的数据，等于 `AB` 时进入
`00403B84` `33C0` `xor eax,eax`

--------------------------略过-----------------------------

`00403BA2` `5F` `pop edi`
`00403BA3` `5E` `pop esi`
`00403BA4` `5B` `pop ebx`
`00403BA5` `5D` `pop ebp`
`00403BA6` `C3` `retn`

`00540800` `56` `push esi` 进入后来到这里
`00540801` `57` `push edi` ; `CR_dumpe.00578630`
`00540802` `832D 84725800 01` `sub dword ptr ds:[587284],1`
`00540809` `73 47` `jnb short CR_dumpe.00540852`
`0054080B` `33C0` `xor eax,eax`
`0054080D` `8905 78725800` `mov dword ptr ds:[587278],eax`
`00540813` `8905 7C725800` `mov dword ptr ds:[58727C],eax`
`00540819` `BE 48D45700` `mov esi,CR_dumpe.0057D448` *
`0054081E` `BF 68725800` `mov edi,CR_dumpe.00587268` *
`00540823` `B9 04000000` `mov ecx,4`
`00540828` `F3:A5` `rep movs dword ptr es:[edi],dword ptr d>` 这里生成
`0054082A` `A0 58085400` `mov al,byte ptr ds:[540858]`
`0054082F` `E8 345BFFFF` `call CR_dumpe.00536368` 进入
`00536368` `53` `push ebx` 来到这里
`00536369` `56` `push esi`
`0053636A` `57` `push edi`
`0053636B` `81C4 94FAFFFF` `add esp,-56C`
`00536371` `880424` `mov byte ptr ss:[esp],al`
`00536374` `8D4424 54` `lea eax,dword ptr ss:[esp+54]`
`00536378` `E8 3FFEFFFF` `call CR_dumpe.005361BC` 进入
`005361BC` `56` `push esi` ; `CR_dumpe.0057D458` 来到这里
`005361BD` `57` `push edi`
`005361BE` `33D2` `xor edx,edx`
`005361C0` `8910` `mov dword ptr ds:[eax],edx`
`005361C2` `8D78 04` `lea edi,dword ptr ds:[eax+4]`
`005361C5` `BE 74D25700` `mov esi,CR_dumpe.0057D274`
`005361CA` `B9 40000000` `mov ecx,40`
`005361CF` `F3:A5` `rep movs dword ptr es:[edi],dword ptr d>`
`005361D1` `33D2` `xor edx,edx`
`005361D3` `8990 04010000` `mov dword ptr ds:[eax+104],edx`
`005361D9` `C780 08010000 555>` `mov dword ptr ds:[eax+108],55555555`
`005361E3` `C780 0C010000 555>` `mov dword ptr ds:[eax+10C],55555555`
`005361ED` `C780 10010000 555>` `mov dword ptr ds:[eax+110],55555555`
`005361F7` `C780 14010000 555>` `mov dword ptr ds:[eax+114],55555555`
`00536201` `5F` `pop edi`
`00536202` `5E` `pop esi`
`00536203` `C3` `retn` 上面这段代码生成 `0012FA58` 处的数据
`0053637D` `F60424 01` `test byte ptr ss:[esp],1` 返回到这里
`00536381` `0F84 60010000` `je CR_dumpe.005364E7`

-------------------------略过----------------------------------------

`005365EA` `B9 04000000` `mov ecx,4`
`005365EF` `E8 14FDFFFF` `call CR_dumpe.00536308` 进入
`00536308` `53` `push ebx` ; `RPCRT4.77E50000` 来到这里
`00536309` `56` `push esi`
`0053630A` `57` `push edi`



```
0053630B     55                 push ebp

0053630C     51                 push ecx
```

\-------------------------略过----------------------------------------

```
00536333     8BD3               mov edx,ebx

00536335     8D87 08010000      lea eax,dword ptr ds:[edi+108]

0053633B     B1 01              mov cl,1

0053633D     E8 8AFDFFFF        call CR_dumpe.005360CC   又调用注册算法 CALL，有很多地方都调用了它，进入

005360CC     53                 push ebx    来到这里

005360CD     56                 push esi

005360CE     57                 push edi

005360CF     83C4 E8            add esp,-18

005360D2     884C24 08          mov byte ptr ss:[esp+8],cl

005360D6     895424 04          mov dword ptr ss:[esp+4],edx

005360DA     890424             mov dword ptr ss:[esp],eax

005360DD     8B4424 04          mov eax,dword ptr ss:[esp+4]

005360E1     8B00               mov eax,dword ptr ds:[eax]

005360E3     894424 0C          mov dword ptr ss:[esp+C],eax

005360E7     8B4424 04          mov eax,dword ptr ss:[esp+4]

005360EB     8B40 04            mov eax,dword ptr ds:[eax+4]

005360EE     894424 10          mov dword ptr ss:[esp+10],eax

005360F2     C74424 14 0400000>mov dword ptr ss:[esp+14],4

005360FA     BE 74D35700        mov esi,CR_dumpe.0057D374

005360FF     8B5424 0C          mov edx,dword ptr ss:[esp+C]

00536103     33C0               xor eax,eax

00536105     8A4424 08          mov al,byte ptr ss:[esp+8]

00536109     8BD8               mov ebx,eax

0053610B     03DB               add ebx,ebx

0053610D     8D1C5B             lea ebx,dword ptr ds:[ebx+ebx*2]

00536110     8B04DE             mov eax,dword ptr ds:[esi+ebx*8]

00536113     8B0C24             mov ecx,dword ptr ss:[esp]   这里的算法和上面的相同，只是 ESP 的数据不同

00536116     8B0C81             mov ecx,dword ptr ds:[ecx+eax*4]

0053618D     894424 0C          mov dword ptr ss:[esp+C],eax

00536191     83C6 0C            add esi,0C

00536194     FF4C24 14          dec dword ptr ss:[esp+14]

00536198    ^ 0F85 61FFFFFF      jnz CR_dumpe.005360FF
```

\-----------------------------------------------------------------------------

ESP 用到的 4 组数据：

```
0012FB5C  27 BD FE D3 C7 7C F8 D3 97 3A 27 50 86 24 C8 76  ??糇??倧⒆盈
```

\------------------------------------------------------------------------------

```
00536342     83C3 08            add ebx,8   返回到这里

00536345     4E                 dec esi

00536346    ^ 75 EB              jnz short CR_dumpe.00536333  循环 32 次生成 0012FA58 处的数据

00536348     8B1424             mov edx,dword ptr ss:[esp]

0053634B     B8 00010000        mov eax,100

00536350     E8 5FFBFFFF        call CR_dumpe.00535EB4

00536355     8BC8               mov ecx,eax

00536357     8BD5               mov edx,ebp

00536359     8D47 04            lea eax,dword ptr ds:[edi+4]

0053635C     E8 33C5ECFF        call CR_dumpe.00402894     

00536361     5A                 pop edx

00536362     5D                 pop ebp

00536363     5F                 pop edi

00536364     5E                 pop esi

00536365     5B                 pop ebx

00536366     C3                 retn          返回

005365F4     8B4424 04          mov eax,dword ptr ss:[esp+4] 返回到这里，0012FA58 处的一组数入 EAX

005365F8     81C4 6C050000      add esp,56C

005365FE     5F                 pop edi

005365FF     5E                 pop esi

00536600     5B                 pop ebx

00536601     C3                 retn   返回
```

写了一大段费话，还是没能弄懂她是怎样算出 `00587268` 处的 4 组数据的，有时间再来研究，本想删了，只是为了给能弄懂的朋友少花一点时间，还是写出来吧，呵呵，希望大师们不要笑我。

做内存注册机是没有问题了，现在来让她自已告诉你注册码，就不用搞算法了。

\-------------------------------------------------------------

我在 `00540446` 处把算出来的注册码保存到自已的地址，因为我在其他地方找到存放注册码的地址是动态的，也可能因为太菜没能找到，呵呵。我在程序尾找一空位写上自已的代码：

\-------------------------------------------------------------

```
00541B12     E8 99E8FFFF        call CR_dumpe.005403B0

00541B17     84C0               test al,al

00541B19     75 38              jnz short CR_dumpe.00541B53 把这里改成 74 38
```

\-------------------------------------------------------------

```
00540446   - E9 B5DB1E00        jmp CR_dumpe.0072E000    跳到自已的地址来保存注册码

0054044B     90                 nop                     
```

\-------------我自已加的代码----------------------------------

```
0072E000    A3 70F27200     mov dword ptr ds:[72F270],eax \\

0072E005    890D 84F27200   mov dword ptr ds:[72F284],ecx |  保存寄存器的样子，因为很多

0072E00B    8915 96F27200   mov dword ptr ds:[72F296],edx |  地方会调用 00540446 处，下面

0072E011    891D ACF27200   mov dword ptr ds:[72F2AC],ebx |  会破坏它，如不然会非法操作

0072E017    893D C2F27200   mov dword ptr ds:[72F2C2],edi /

0072E01D    8B45 E8         mov eax,dword ptr ss:[ebp-18]    注册码入 EAX

0072E020    8B78 FC         mov edi,dword ptr ds:[eax-4]

0072E023    893D C7FF7200   mov dword ptr ds:[72FFC7],edi    把注册码的长度保存到一个地址

0072E029    BB 261C5400     mov ebx,CR_dumpe.00541C26        显示出注册码的地址

0072E02E    33C9            xor ecx,ecx            初绐计数器

0072E030    0FB61408        movzx edx,byte ptr ds:[eax+ecx] 取注册码的一位入 EDX

0072E034    881419          mov byte ptr ds:[ecx+ebx],DL  把注册码放到显示地址

0072E037    41              inc ecx      计数器加一

0072E038    4F              dec edi      注册码长度减一

0072E039   ^ 75 F5           jnz short CR_dumpe.0072E02A 没有取完则继续

0072E03B    A1 70F27200     mov eax,dword ptr ds:[72F270]  \\

0072E040    8B0D 84F27200   mov ecx,dword ptr ds:[72F284]  |

0072E046    8B15 96F27200   mov edx,dword ptr ds:[72F296]  | 恢复寄存器的样子

0072E04C    8B1D ACF27200   mov ebx,dword ptr ds:[72F2AC]  |

0072E052    8B3D C2F27200   mov edi,dword ptr ds:[72F2C2]  /

0072E058    8B45 E8         mov eax,dword ptr ss:[ebp-18] \\00540446 处的原程序

0072E05B    8D55 EC         lea edx,dword ptr ss:[ebp-14] /

0072E05E   - E9 EF23E1FF     jmp CR_dumpe.0054044C           跳回去执行
```

\-----------------------------------------------------------

```
00541B5E     B8 501C5400        mov eax,CR_dumpe.00541C50 把这里改成 00541C08          

00541B63     E8 98CBF1FF        call CR_dumpe.0045E700
```

\---------------------------------------------------------

注册码已经保存到我们的地址，现在让她显示出来，下面是出错的信息，`3D` 是所取字符串的长度。

```
00541BF8                   FF FF FF FF 3D 00 00 00      ..=.

00541C08  49 6E 76 61 6C 69 64 20 52 65 6C 65 61 73 65 20   湉慶楬?敒敬獡?

00541C18  43 6F 64 65 2E 20 20 50 6C 65 61 73 65 20 63 68   潃敤?倠敬獡?档

00541C28  65 63 6B 20 79 6F 75 72 20 65 6E 74 72 79 20 61   捥?潹牵攠瑮祲愠

00541C38  6E 64 20 74 72 79 20 61 67 61 69 6E 2E 00 00 00   摮琠祲愠慧湩..

00541C48  FF FF FF FF                                    ..
```

我把上面的信息改成这样：

```
00541BF8                   FF FF FF FF 3D 00 00 00      ..=.

00541C08  BA C7 BA C7 A3 AC CE D2 B2 BB D2 AA C4 E3 B5 C4  ??????

00541C18  C7 AE B8 F8 C4 E3 D7 A2 B2 E1 C2 EB A3 BA 00 00  ??

00541C28  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ??????休.

00541C38  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ........

00541C48  FF FF FF FF                                    ..
```

\---------------------------------------------------------

显示出了注册码，如果你连抄都不愿抄一下，那就让她自已用正确的注册码来帮你生成注册文件吧：

```
00541B79     8B45 F0            mov eax,dword ptr ss:[ebp-10]

00541B7C     8BD3               mov edx,ebx

00541B7E     E8 05EAFFFF        call CR_dumpe.00540588
```

我把上面改成这样-----------------------------------------

```
00541B79   - E9 90DE1E00        jmp CR_dumpe.0072FA0E     跳到自加的代码处执行

00541B7E     E8 05EAFFFF        call CR_dumpe.00540588
```

我加的代码-----------------------------------------------

```
0072FA0E    8915 01FE7200   mov dword ptr ds:[72FE01],edx  保存寄存器 EDX 的样子

0072FA14    8B45 F0         mov eax,dword ptr ss:[ebp-10]  存放注册码的地址

0072FA17    8B15 261C5400   mov edx,dword ptr ds:[541C26]   

0072FA1D    8910            mov dword ptr ds:[eax],edx

0072FA1F    8B15 2A1C5400   mov edx,dword ptr ds:[541C2A]

0072FA25    8950 04         mov dword ptr ds:[eax+4],edx
```



```
0072FA28    8B15 2E1C5400   mov edx,dword ptr ds:[541C2E]  

0072FA2E    8950 08         mov dword ptr ds:[eax+8],edx  

0072FA31    8B15 321C5400   mov edx,dword ptr ds:[541C32]  

0072FA37    8950 0C         mov dword ptr ds:[eax+C],edx  

0072FA3A    8B15 C7FF7200   mov edx,dword ptr ds:[72FFC7]  

0072FA40    8850 FC         mov byte ptr ds:[eax-4],dl  

0072FA43    8B15 01FE7200   mov edx,dword ptr ds:[72FE01]  恢复寄存器 EDX 的样子  

0072FA49    8BD3            mov edx,ebx  

0072FA4B  - E9 2E21E1FF     jmp CR\_dumpe.00541B7E   跳回去执行
```

## 破解总结

这个软件的算法还没有完全搞明白，有兴趣的朋友可以研究一下，这个程序我在三台机上测试通过，都可以正确显示

注册码，注册成功之后会在根目录下生成`REGISTRATION.DAT`文件。想来一个软件给我改成这样？？？？惨啊？错误之处敬请指教。

---

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
