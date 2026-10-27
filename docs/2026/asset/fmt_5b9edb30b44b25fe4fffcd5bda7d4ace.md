
```
004040D6  |.  C645 FC 02    ||mov     byte ptr [ebp-4], 2

004040DA  |.  8D8D F0FBFFFF ||lea     ecx, dword ptr [ebp-410]

004040E0  |.  E8 8CDEFFFF   ||call    00401F71

004040E5  |.^ E9 97FEFFFF   |\jmp     00403F81

004040EA  |>  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]

004040F0  |.  51            |push    ecx

004040F1  |.  8D8D 64FFFFFF |lea     ecx, dword ptr [ebp-9C]

004040F7  |.  E8 0DDFFFFF   |call    00402009

004040FC  |.  8B95 60FFFFFF |mov     edx, dword ptr [ebp-A0]

00404102  |.  C1FA 05       |sar     edx, 5

00404105  |.  8B8D 60FFFFFF |mov     ecx, dword ptr [ebp-A0]

0040410B  |.  83E1 1F       |and     ecx, 1F

0040410E  |.  8B45 0C       |mov     eax, dword ptr [ebp+C]

00404111  |.  8B5490 04     |mov     edx, dword ptr [eax+edx*4+4]    ;  取短固定字符串

00404115  |.  D3EA          |shr     edx, cl                         ;  右移上面计算2的次方位

00404117  |.  83E2 01       |and     edx, 1                          ;  与1相与 作为不同大循环的路径的判断

0040411A  |.  85D2          |test    edx, edx

0040411C  |.  0F84 2B020000 |je      0040434D                        ;   跳就是到M循环，不跳就到N循环

00404122  |.  8B85 64FFFFFF |mov     eax, dword ptr [ebp-9C]

00404128  |.  8B8C85 64FFFF>|mov     ecx, dword ptr [ebp+eax*4-9C]

0040412F  |.  51            |push    ecx

00404130  |.  8D95 60FBFFFF |lea     edx, dword ptr [ebp-4A0]

00404136  |.  52            |push    edx

00404137  |.  8B8D 18F9FFFF |mov     ecx, dword ptr [ebp-6E8]

0040413D  |.  E8 3FE9FFFF   |call    00402A81                        ;  第二种循环N算法A

00404142  |.  8985 ECF8FFFF |mov     dword ptr [ebp-714], eax

00404148  |.  8B85 ECF8FFFF |mov     eax, dword ptr [ebp-714]

0040414E  |.  8985 E8F8FFFF |mov     dword ptr [ebp-718], eax

00404154  |.  C645 FC 08    |mov     byte ptr [ebp-4], 8

00404158  |.  8B8D E8F8FFFF |mov     ecx, dword ptr [ebp-718]

0040415E  |.  51            |push    ecx

0040415F  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]

00404165  |.  E8 9FDEFFFF   |call    00402009

0040416A  |.  C645 FC 02    |mov     byte ptr [ebp-4], 2

0040416E  |.  8D8D 60FBFFFF |lea     ecx, dword ptr [ebp-4A0]

00404174  |.  E8 F8DDFFFF   |call    00401F71

00404179  |.  8B55 10       |mov     edx, dword ptr [ebp+10]

0040417C  |.  52            |push    edx

0040417D  |.  8D85 D0FAFFFF |lea     eax, dword ptr [ebp-530]

00404183  |.  50            |push    eax

00404184  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]

0040418A  |.  E8 3BF0FFFF   |call    004031CA                        ;  第二种循环N的算法B

0040418F  |.  8985 E4F8FFFF |mov     dword ptr [ebp-71C], eax

00404195  |.  8B8D E4F8FFFF |mov     ecx, dword ptr [ebp-71C]

0040419B  |.  898D E0F8FFFF |mov     dword ptr [ebp-720], ecx

004041A1  |.  C645 FC 09    |mov     byte ptr [ebp-4], 9

004041A5  |.  8B95 E0F8FFFF |mov     edx, dword ptr [ebp-720]

004041AB  |.  52            |push    edx

004041AC  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]

004041B2  |.  E8 52DEFFFF   |call    00402009

004041B7  |.  C645 FC 02    |mov     byte ptr [ebp-4], 2

004041BB  |.  8D8D D0FAFFFF |lea     ecx, dword ptr [ebp-530]

004041C1  |.  E8 ABDDFFFF   |call    00401F71

004041C6  |.  C785 C0FEFFFF>|mov     dword ptr [ebp-140], 1

004041D0  |.  EB 0F         |jmp     short 004041E1

004041D2  |>  8B85 C0FEFFFF |/mov     eax, dword ptr [ebp-140]

004041D8  |.  83C0 01       ||add     eax, 1

004041DB  |.  8985 C0FEFFFF ||mov     dword ptr [ebp-140], eax

004041E1  |>  8B8D C0FEFFFF | mov     ecx, dword ptr [ebp-140]

004041E7  |.  3B8D 64FFFFFF ||cmp     ecx, dword ptr [ebp-9C]        ;  第二种循环N第二段循环M2的开始

004041ED  |.  0F83 48010000 ||jnb     0040433B

004041F3  |.  8B95 D0FEFFFF ||mov     edx, dword ptr [ebp-130]

004041F9  |.  8995 CCFEFFFF ||mov     dword ptr [ebp-134], edx

004041FF  |.  EB 0F         ||jmp     short 00404210

00404201  |>  8B85 CCFEFFFF ||/mov     eax, dword ptr [ebp-134]

00404207  |.  83E8 01       |||sub     eax, 1

0040420A  |.  8985 CCFEFFFF |||mov     dword ptr [ebp-134], eax

00404210  |>  83BD CCFEFFFF>|| cmp     dword ptr [ebp-134], 0

00404217  |.  7E 1C         |||jle     short 00404235

00404219  |.  8B8D CCFEFFFF |||mov     ecx, dword ptr [ebp-134]

0040421F  |.  8B95 CCFEFFFF |||mov     edx, dword ptr [ebp-134]

00404225  |.  8B8495 D0FEFF>|||mov     eax, dword ptr [ebp+edx*4-130>

0040422C  |.  89848D D4FEFF>|||mov     dword ptr [ebp+ecx*4-12C], ea>

00404233  |.^ EB CC         ||\jmp     short 00404201

00404235  |>  C785 D4FEFFFF>||mov     dword ptr [ebp-12C], 0

0040423F  |.  8B8D D0FEFFFF ||mov     ecx, dword ptr [ebp-130]

00404245  |.  83C1 01       ||add     ecx, 1

00404248  |.  898D D0FEFFFF ||mov     dword ptr [ebp-130], ecx

0040424E  |.  8B95 64FFFFFF ||mov     edx, dword ptr [ebp-9C]

00404254  |.  2B95 C0FEFFFF ||sub     edx, dword ptr [ebp-140]

0040425A  |.  8B8495 64FFFF>||mov     eax, dword ptr [ebp+edx*4-9C]

00404261  |.  50            ||push    eax

00404262  |.  8D8D 40FAFFFF ||lea     ecx, dword ptr [ebp-5C0]

00404268  |.  51            ||push    ecx

00404269  |.  8B8D 18F9FFFF ||mov     ecx, dword ptr [ebp-6E8]

0040426F  |.  E8 0DE8FFFF   ||call    00402A81                               ; 算法A

00404274  |.  8985 DCF8FFFF ||mov     dword ptr [ebp-724], eax

0040427A  |.  8B95 DCF8FFFF ||mov     edx, dword ptr [ebp-724]

00404280  |.  8995 D8F8FFFF ||mov     dword ptr [ebp-728], edx

00404286  |.  C645 FC 0A    ||mov     byte ptr [ebp-4], 0A

0040428A  |.  8B85 D8F8FFFF ||mov     eax, dword ptr [ebp-728]

00404290  |.  50            ||push    eax

00404291  |.  8D8D B0F9FFFF ||lea     ecx, dword ptr [ebp-650]

00404297  |.  51            ||push    ecx

00404298  |.  8D8D D0FEFFFF ||lea     ecx, dword ptr [ebp-130]

0040429E  |.  E8 2FDEFFFF   ||call    004020D2

004042A3  |.  8985 D4F8FFFF ||mov     dword ptr [ebp-72C], eax

004042A9  |.  8B95 D4F8FFFF ||mov     edx, dword ptr [ebp-72C]

004042AF  |.  8995 D0F8FFFF ||mov     dword ptr [ebp-730], edx

004042B5  |.  C645 FC 0B    ||mov     byte ptr [ebp-4], 0B

004042B9  |.  8B85 D0F8FFFF ||mov     eax, dword ptr [ebp-730]

004042BF  |.  50            ||push    eax

004042C0  |.  8D8D D0FEFFFF ||lea     ecx, dword ptr [ebp-130]

004042C6  |.  E8 3EDDFFFF   ||call    00402009

004042CB  |.  C645 FC 0A    ||mov     byte ptr [ebp-4], 0A

004042CF  |.  8D8D B0F9FFFF ||lea     ecx, dword ptr [ebp-650]

004042D5  |.  E8 97DCFFFF   ||call    00401F71

004042DA  |.  C645 FC 02    ||mov     byte ptr [ebp-4], 2

004042DE  |.  8D8D 40FAFFFF ||lea     ecx, dword ptr [ebp-5C0]

004042E4  |.  E8 88DCFFFF   ||call    00401F71

004042E9  |.  8B4D 10       ||mov     ecx, dword ptr [ebp+10]

004042EC  |.  51            ||push    ecx

004042ED  |.  8D95 20F9FFFF ||lea     edx, dword ptr [ebp-6E0]

004042F3  |.  52            ||push    edx

004042F4  |.  8D8D D0FEFFFF ||lea     ecx, dword ptr [ebp-130]

004042FA  |.  E8 CBEEFFFF   ||call    004031CA                              '算法B

004042FF  |.  8985 CCF8FFFF ||mov     dword ptr [ebp-734], eax

00404305  |.  8B85 CCF8FFFF ||mov     eax, dword ptr [ebp-734]

0040430B  |.  8985 C8F8FFFF ||mov     dword ptr [ebp-738], eax

00404311  |.  C645 FC 0C    ||mov     byte ptr [ebp-4], 0C

00404315  |.  8B8D C8F8FFFF ||mov     ecx, dword ptr [ebp-738]

0040431B  |.  51            ||push    ecx

0040431C  |.  8D8D D0FEFFFF ||lea     ecx, dword ptr [ebp-130]

00404322  |.  E8 E2DCFFFF   ||call    00402009

00404327  |.  C645 FC 02    ||mov     byte ptr [ebp-4], 2

0040432B  |.  8D8D 20F9FFFF ||lea     ecx, dword ptr [ebp-6E0]

00404331  |.  E8 3BDCFFFF   ||call    00401F71

00404336  |.^ E9 97FEFFFF   |\jmp     004041D2

0040433B  |>  8D95 D0FEFFFF |lea     edx, dword ptr [ebp-130]

00404341  |.  52            |push    edx
```
