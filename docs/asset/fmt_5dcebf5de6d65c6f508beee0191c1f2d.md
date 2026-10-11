
```assembly
00409701   |.  48                dec eax                       *****EAX=C-1=B
00409702   |.  85C0             test eax,eax
00409704   |.  7E 0E            jle short DISKdata.00409714
00409706   |.  B9 01000000      mov ecx,1                     *****ECX=1
0040970B   |>  66:035C4E FE     /add bx,word ptr ds:[esi+ecx*2-2]  *****开始累加一个数
00409710   |.  41               |inc ecx
00409711   |.  48               |dec eax                      *****EAX 是要累加的次数 B，得到累加值是 14F，存于 BX 中
00409712   |.^ 75 F7            \jnz short DISKdata.0040970B
00409714   |>  0FB74D FE        movzx ecx,word ptr ss:[ebp-2] *****ECX=7F1
00409718   |.  49               dec ecx                       *****ECX=7F1-1=7F0
00409719   |.  8BC1             mov eax,ecx                   *****EAX=ECX=7F0
0040971B   |.  BE 64000000      mov esi,64                    *****ESI=64
00409720   |.  99               cdq
00409721   |.  F7FE             idiv esi                      *****IDIV:7F0 IDIV 64=00000014 r 00000020
00409723   |.  69F1 6D010000    imul esi,ecx,16D              *****IMUL:ESI=7F0 * 16D=000B5130
00409729   |.  8BD1             mov edx,ecx                   *****EDX=ECX=7F0
0040972B   |.  85D2             test edx,edx                  *****测试一下
0040972D   |.  79 03            jns short DISKdata.00409732   没有符号就跳到 00409732
0040972F   |.  83C2 03          add edx,3
00409732   |>  C1FA 02          sar edx,2                     *****Sar:7F0/2²=1FC
00409735   |.  03F2             add esi,edx                   *****ADD:B5130+1FC=B532C
00409737   |.  2BF0             sub esi,eax                   *****Sub:B532C-14=B5318
00409739   |.  8BC1             mov eax,ecx                   *****EAX=ECX=7F0
0040973B   |.  B9 9000000       mov ecx,190                   *****ECX=190
00409740   |.  99               cdq
00409741   |.  F7F9             idiv ecx                      *****idiv:7F0 idiv 190=00000005 r 00000020
00409743   |.  03F0             add esi,eax                   *****ADD:ESI=B5318+5=B531D
00409745   |.  0FB7C3           movzx eax,bx                  *****BX=14F
00409748   |.  03F0             add esi,eax                   *****ADD:ESI=B531D+14F=B546C
0040974A   |.  81EE 5A950A00    sub esi,0A955A                *****Sub:ESI=B546C-0A955A=BF12
00409750   |.  8975 F8          mov dword ptr ss:[ebp-8],esi
00409753   |.  DB45 F8          fild dword ptr ss:[ebp-8]     *****浮点，装入整数
```

Stack ss:`[0012F5A8]`=0000BF12 (十进制 48914.)

```assembly
00409756   |.  8B45 08          mov eax,dword ptr ss:[ebp+8]
00409759   |.  DD18             fstp qword ptr ds:[eax]       *****上托出栈
```

st=48914.000000000000000                       ××××××××全靠它了！！！！！！！！！！

Stack ds:`[0012F5D8]`=1.486724818696458e-304

```assembly
0040975B   |.  9B               wait
0040975C   |.  C645 FD 01       mov byte ptr ss:[ebp-3],1
00409760   |>  8A45 FD          mov al,byte ptr ss:[ebp-3]
00409763   |.  5F               pop edi
00409764   |.  5E               pop esi
00409765   |.  5B               pop ebx
00409766   |.  59               pop ecx
00409767   |.  59               pop ecx
00409768   |.  5D               pop ebp
00409769   \.  C2 0400          retn 4
```

××××××××××××××××××××××××××××××××××××

```assembly
004099E3   |.  50               push eax                           ; /pLocaltime
004099E4   |.  E8 8FCFFFFF      call <jmp.&kernel32.GetLocalTime>   ; \GetLocalTime  *****取得日期，今天是 3 号
004099E9   |.  66:8B4C24 0E     mov cx,word ptr ss:[esp+E]         *****word ptr ss:[esp+E]=3（今天是 3 号）
004099EE   |.  66:8B5424 0A     mov dx,word ptr ss:[esp+A]         *****word ptr ss:[esp+A]=6 （6 月）
004099F3   |.  66:8B4424 08     mov ax,word ptr ss:[esp+8]         *****word ptr ss:[esp+8]=07D5（2005）（今年是 2005 年）
004099F8   |.  E8 6FFDFFFF      call DISKdata.0040976C             ****************************通过日期算出一个浮点数
004099FD   |.  DD1C24           fstp qword ptr ss:[esp]
00409A00   |.  9B               wait
00409A01   |.  DD0424           fld qword ptr ss:[esp]
00409A04   |.  83C4 18          add esp,18
00409A07   \.  C3               retn
```

```assembly
004096CF   |.  66:837D FE 01    cmp word ptr ss:[ebp-2],1          *****word ptr ss:[ebp-2]=7D5
004096D4   |.  0F82 86000000    jb DISKdata.00409760
004096DA   |.  66:817D FE 0F27  cmp word ptr ss:[ebp-2],270F
004096E0   |.  77 7E            ja short DISKdata.00409760
004096E2   |.  66:83FF 01       cmp di,1                            *****DI=6
004096E6   |.  72 78            jb short DISKdata.00409760
004096E8   |.  66:83FF 0C       cmp di,0C                           *****现在知道为什么极限数是 C(12),有效数是 1--12（因为这是一个月份值呀！）
004096EC   |.  77 72            ja short DISKdata.00409760
004096EE   |.  66:83FB 01       cmp bx,1                            *****为什么 BX 要与 1 比较？因为没有人见过日期是 0 的呀！！！！
004096F2   |.  72 6C            jb short DISKdata.00409760
004096F4   |.  0FB7C7           movzx eax,di
004096F7   |.  66:3B5C46 FE     cmp bx,word ptr ds:[esi+eax*2-2]   *****这里查表的数据全是 1E（30）,1F（31）这都是月份有关的数！
004096FC   |.  77 62            ja short DISKdata.00409760
004096FE   |.  0FB7C7           movzx eax,di
00409701   |.  48               dec eax
00409702   |.  85C0             test eax,eax
00409704   |.  7E 0E            jle short DISKdata.00409714
00409706   |.  B9 01000000      mov ecx,1
0040970B   |>  66:035C4E FE     /add bx,word ptr ds:[esi+ecx*2-2]
00409710   |.  41               |inc ecx
00409711   |.  48               |dec eax
00409712   |.^ 75 F7            \jnz short DISKdata.0040970B
00409714   |>  0FB74D FE        movzx ecx,word ptr ss:[ebp-2]
00409718   |.  49               dec ecx
00409719   |.  8BC1             mov eax,ecx
0040971B   |.  BE 64000000      mov esi,64
00409720   |.  99               cdq
00409721   |.  F7FE             idiv esi
00409723   |.  69F1 6D010000    imul esi,ecx,16D
00409729   |.  8BD1             mov edx,ecx
0040972B   |.  85D2             test edx,edx
0040972D   |.  79 03            jns short DISKdata.00409732
0040972F   |.  83C2 03          add edx,3
00409732   |>  C1FA 02          sar edx,2
00409735   |.  03F2             add esi,edx
00409737   |.  2BF0             sub esi,eax
00409739   |.  8BC1             mov eax,ecx
0040973B   |.  B9 9000000       mov ecx,190
00409740   |.  99               cdq
00409741   |.  F7F9             idiv ecx
00409743   |.  03F0             add esi,eax
00409745   |.  0FB7C3           movzx eax,bx
00409748   |.  03F0             add esi,eax
0040974A   |.  81EE 5A950A00    sub esi,0A955A
00409750   |.  8975 F8          mov dword ptr ss:[ebp-8],esi
00409753   |.  DB45 F8          fild dword ptr ss:[ebp-8]
00409756   |.  8B45 08          mov eax,dword ptr ss:[ebp+8]
00409759   |.  DD18             fstp qword ptr ds:[eax]
0040975B   |.  9B               wait
```
