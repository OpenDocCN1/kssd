
```
00488EFA  |.  C1E2 02       ||shl edx,2  

00488EFD  |.  8D1492        ||lea edx,dword ptr ds:\[edx+edx\*4\]  

00488F00  |.  8B0D BC655600 ||mov ecx,dword ptr ds:\[5665BC\]  

00488F06  |.  8D14D1        ||lea edx,dword ptr ds:\[ecx+edx\*8\]  

00488F09  |.  89849A 5CFFFF>||mov dword ptr ds:\[edx+ebx\*4-A4\],eax  

00488F10  |>  43            ||inc ebx                               ;  Default case of switch 00488C2A  

00488F11  |.  8345 EC 04    ||add dword ptr ss:\[ebp-14\],4  

00488F15  |.  83C7 04       ||add edi,4  

00488F18  |.  4E            ||dec esi  

00488F19  |.^ 0F85 08FDFFFF |\jnz dumped.00488C27  

00488F1F  |>  FF45 F4       |inc dword ptr ss:\[ebp-C\]  

00488F22  |.  837D F4 0B    |cmp dword ptr ss:\[ebp-C\],0B  

00488F26  |.^ 0F85 DCFCFFFF \|jnz dumped.00488C08  

00488F2C  |.  8D45 F0       lea eax,dword ptr ss:\[ebp-10\]  

00488F2F  |.  E8 E8ACF7FF   call dumped.00403C1C  

00488F34  |.  8B35 C4655600 mov esi,dword ptr ds:\[5665C4\]                 ;  取注册码长度  

00488F3A  |.  85F6          test esi,esi  

00488F3C  |.  7E 62         jle short dumped.00488FA0  

00488F3E  |.  BB 01000000   mov ebx,1  

00488F43  |.  BF DC635600   mov edi,dumped.005663DC  

00488F48  |>  8BC3          /mov eax,ebx  

00488F4A  |.  B9 0A000000   |mov ecx,0A  

00488F4F  |.  99            |cdq  

00488F50  |.  F7F9          |idiv ecx  

00488F52  |.  8BCA          |mov ecx,edx  

00488F54  |.  85C9          |test ecx,ecx                                ;  计算 1-9 的数  

00488F56  |.  7E 05         |jle short dumped.00488F5D  

00488F58  |.  894D F4       |mov dword ptr ss:\[ebp-C\],ecx             ;  保存当前上面算得的数  

00488F5B  |.  EB 07         |jmp short dumped.00488F64  

00488F5D  |>  C745 F4 0A000>|mov dword ptr ss:\[ebp-C\],0A  

00488F64  |>  8B45 F4       |mov eax,dword ptr ss:\[ebp-C\]  

00488F67  |.  C1E0 02       |shl eax,2  

00488F6A  |.  8D0480        |lea eax,dword ptr ds:\[eax+eax\*4\]         ;  相当于 i\*14H  

00488F6D  |.  8B15 BC655600 |mov edx,dword ptr ds:\[5665BC\]            ;  取得上面计算的数的保存地址  

00488F73  |.  8D04C2        |lea eax,dword ptr ds:\[edx+eax\*8\]         ;  当\[ebp-c\]为 1 时取第二个表格，3 时取第三个，4->5...  

00488F76  |.  8B8498 5CFFFF>|mov eax,dword ptr ds:\[eax+ebx\*4-A4\]     ;  等效于取第一个表格中的第一个数据，第二个表格中的第二个数据，第三个表格中的第三个数据...第一个表格中的最后一个数据  

00488F7D  |.  8907          |mov dword ptr ds:\[edi\],eax               ;  写入到 5663dc 开始的地方  

00488F7F  |.  8D45 E0       |lea eax,dword ptr ss:\[ebp-20\]  

00488F82  |.  8B55 F8       |mov edx,dword ptr ss:\[ebp-8\]  

00488F85  |.  8A541A FF     |mov dl,byte ptr ds:\[edx+ebx-1\]  

00488F89  |.  E8 36AEF7FF   |call dumped.00403DC4  

00488F8E  |.  8B55 E0       |mov edx,dword ptr ss:\[ebp-20\]  

00488F91  |.  8D45 F0       |lea eax,dword ptr ss:\[ebp-10\]  

00488F94  |.  E8 0BAFF7FF   |call dumped.00403EA4  

00488F99  |.  43            |inc ebx  

00488F9A  |.  83C7 04       |add edi,4  

00488F9D  |.  4E            |dec esi  

00488F9E  |.^ 75 A8         \|jnz short dumped.00488F48  

00488FA0  |>  8B35 C4655600 mov esi,dword ptr ds:\[5665C4\]                 ;  取注册码长度  

00488FA6  |.  83EE 03       sub esi,3                                       ;  只循环 长度-3 次  

00488FA9  |.  85F6          test esi,esi  

00488FAB  |.  0F8E 80000000 jle dumped.00489031  

00488FB1  |.  BB DC635600   mov ebx,dumped.005663DC  

00488FB6  |.  BF CC655600   mov edi,dumped.005665CC  

00488FBB  |.  C745 E8 6C665>mov dword ptr ss:\[ebp-18\],dumped.0056666C  

00488FC2  |>  8B03          /mov eax,dword ptr ds:\[ebx\]               ;  取一个上面变换的注册码  

00488FC4  |.  F7E8          |imul eax  

00488FC6  |.  8BC8          |mov ecx,eax  

00488FC8  |.  8B43 04       |mov eax,dword ptr ds:\[ebx+4\]             ;  取一个上面变换的下一个注册码  

00488FCB  |.  F7E8          |imul eax  

00488FCD  |.  0FAFC8        |imul ecx,eax  

00488FD0  |.  894D E4       |mov dword ptr ss:\[ebp-1C\],ecx  

00488FD3  |.  DB45 E4       |fild dword ptr ss:\[ebp-1C\]  

00488FD6  |.  D9FA          |fsqrt  

00488FD8  |.  DB7D D4       |fstp tbyte ptr ss:\[ebp-2C\]  

00488FDB  |.  9B            |wait  

00488FDC  |.  8B43 08       |mov eax,dword ptr ds:\[ebx+8\]             ;  取一个上面变换的下下个注册码  

00488FDF  |.  F7E8          |imul eax  

00488FE1  |.  8BC8          |mov ecx,eax  

00488FE3  |.  8B03          |mov eax,dword ptr ds:\[ebx\]  

00488FE5  |.  F7E8          |imul eax  

00488FE7  |.  03C8          |add ecx,eax  

00488FE9  |.  894D D0       |mov dword ptr ss:\[ebp-30\],ecx  

00488FEC  |.  DB45 D0       |fild dword ptr ss:\[ebp-30\]  

00488FEF  |.  D9FA          |fsqrt  

00488FF1  |.  DB6D D4       |fld tbyte ptr ss:\[ebp-2C\]  

00488FF4  |.  DEC9          |fmulp st(1),st  

00488FF6  |.  E8 D59AF7FF   |call dumped.00402AD0                      ;  EAX=sqrt(code2(i)\*code2(i)\*code2(i+1)\*code2(i+1))\*sqrt(code2(i+2)\*code2(i+2)+code2(i)\*code2(i))  

00488FFB  |.  8907          |mov dword ptr ds:\[edi\],eax               ;  变换后的码写入 5665cc  

00488FFD  |.  8B43 04       |mov eax,dword ptr ds:\[ebx+4\]             ;  下一个变换后的  

00489000  |.  2B03          |sub eax,dword ptr ds:\[ebx\]  

00489002  |.  F7E8          |imul eax  

00489004  |.  8BC8          |mov ecx,eax  

00489006  |.  8B43 08       |mov eax,dword ptr ds:\[ebx+8\]             ;  取下下个变换后的  

00489009  |.  2B43 04       |sub eax,dword ptr ds:\[ebx+4\]             ;  取下个变换后的  

0048900C  |.  F7E8          |imul eax  

0048900E  |.  03C8          |add ecx,eax  

00489010  |.  8BC1          |mov eax,ecx  

00489012  |.  F7E8          |imul eax  

00489014  |.  8945 E4       |mov dword ptr ss:\[ebp-1C\],eax  

00489017  |.  DB45 E4       |fild dword ptr ss:\[ebp-1C\]  

0048901A  |.  E8 B19AF7FF   |call dumped.00402AD0                      ;  EAX=((code2(i+1)-code2(i))²+(code2(i+2)-code2(i+1))²)²  

0048901F  |.  8B55 E8       |mov edx,dword ptr ss:\[ebp-18\]  

00489022  |.  8902          |mov dword ptr ds:\[edx\],eax               ;  保存到 56666c 开始的地方  

00489024  |.  8345 E8 04    |add dword ptr ss:\[ebp-18\],4  

00489028  |.  83C7 04       |add edi,4  

0048902B  |.  83C3 04       |add ebx,4  

0048902E  |.  4E            |dec esi  

0048902F  |.^ 75 91        \|jnz short dumped.00488FC2  

00489031  |>  33C0          xor eax,eax  

00489033  |.  5A            pop edx  

00489034  |.  59            pop ecx  

00489035  |.  59            pop ecx  

00489036  |.  64:8910      mov dword ptr fs:\[eax\],edx  

00489039  |.  68 63904800   push dumped.00489063  

0048903E  |>  8D45 E0       lea eax,dword ptr ss:\[ebp-20\]  

00489041  |.  E8 D6ABF7FF   call dumped.00403C1C  

00489046  |.  8D45 F0       lea eax,dword ptr ss:\[ebp-10\]  

00489049  |.  E8 CEABF7FF   call dumped.00403C1C  

0048904E  |.  8D45 F8       lea eax,dword ptr ss:\[ebp-8\]  

00489051  |.  BA 02000000   mov edx,2  

00489056  |.  E8 E5ABF7FF   call dumped.00403C40  

0048905B  \\.  C3           retn  

0048905C    .^ E9 53A6F7FF  jmp dumped.004036B4  

00489061    .^ EB DB        jmp short dumped.0048903E  

00489063     .  5F          pop edi  

00489064     .  5E          pop esi  

00489065     .  5B          pop ebx  

00489066     .  8BE5        mov esp,ebp  

00489068     .  5D          pop ebp  

00489069     .  C3          retn  

0048906C     $  55          push ebp  

0048906D     .  8BEC        mov ebp,esp  

0048906F     .  53          push ebx  

00489070     .  BB 39635600 mov ebx,dumped.00566339  

00489075     .  33C0        xor eax,eax  

00489077     .  55          push ebp  

00489078     .  68 2B914800 push dumped.0048912B  

0048907D     .  64:FF30     push dword ptr fs:\[eax\]  

00489080     .  64:8920     mov dword ptr fs:\[eax\],esp  

00489083     .  B8 EB250300 mov eax,325EB  

00489088     .  E8 0F020000 call dumped.0048929C  

0048908D     .  803B 00     cmp byte ptr ds:\[ebx\],0                   ;  与成功标志 566339 比较  

00489090     .  0F84 87000000 je dumped.0048911D  
```
