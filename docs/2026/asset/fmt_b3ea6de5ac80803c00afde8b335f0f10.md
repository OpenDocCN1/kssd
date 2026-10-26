
```
.text:004085A5                 cmp     eax, ebx

.text:004085A7                 jz      loc_4086ED     ;相等(eax=0),跳出,不等,继续

.text:004085AD                 mov     ebp, off_470DB8     ;特征码1

.text:004085B3                 mov     eax, offset aVncEnterpriseE ; "VNC Enterprise Edition for Windows"

.text:004085B8                 mov     [esp+40h+var_10], edi

.text:004085BC                 mov     dword ptr [esp+40h+Args], eax

.text:004085C0                 mov     [esp+40h+var_20], eax

.text:004085C4                 mov     [esp+40h+var_14], esi

.text:004085C8                 xor     edi, edi

.text:004085CA                 lea     ebx, [ebx+0]

.text:004085D0

.text:004085D0 loc_4085D0:                             ; CODE XREF: sub_4084B0+217j

.text:004085D0                 mov     esi, [esp+edi+40h+var_14]

.text:004085D4                 mov     ecx, ebp

.text:004085D6                 mov     eax, esi

.text:004085D8

.text:004085D8 loc_4085D8:                             ; CODE XREF: sub_4084B0+142j

.text:004085D8                 mov     dl, [eax]       ; 比较特征码

.text:004085DA                 cmp     dl, [ecx]

.text:004085DC                 jnz     short loc_4085F8

.text:004085DE                 cmp     dl, bl

.text:004085E0                 jz      short loc_4085F4

.text:004085E2                 mov     dl, [eax+1]

.text:004085E5                 cmp     dl, [ecx+1]

.text:004085E8                 jnz     short loc_4085F8

.text:004085EA                 add     eax, 2

.text:004085ED                 add     ecx, 2

.text:004085F0                 cmp     dl, bl

.text:004085F2                 jnz     short loc_4085D8

.text:004085F4

.text:004085F4 loc_4085F4:                             ; CODE XREF: sub_4084B0+130j

.text:004085F4                 xor     eax, eax

.text:004085F6                 jmp     short loc_4085FD

.text:004085F8 ; ---------------------------------------------------------------------------

.text:004085F8

.text:004085F8 loc_4085F8:                             ; CODE XREF: sub_4084B0+12Cj

.text:004085F8                                         ; sub_4084B0+138j

.text:004085F8                 sbb     eax, eax

.text:004085FA                 sbb     eax, 0FFFFFFFFh

.text:004085FD

.text:004085FD loc_4085FD:                             ; CODE XREF: sub_4084B0+146j

.text:004085FD                 cmp     eax, ebx

.text:004085FF                 jnz     short loc_40860E     ;相等(eax=0),继续,不等,跳到下个比较

.text:00408601                 mov     dword ptr [esp+edi+40h+Args], offset aVncPersonalEdi ; "VNC Personal Edition for Windows"

.text:00408609                 jmp     loc_4086C1           ;转到提示版本不同信息

.text:0040860E ; ---------------------------------------------------------------------------

.text:0040860E

.text:0040860E loc_40860E:                             ; CODE XREF: sub_4084B0+14Fj

.text:0040860E                 mov     ecx, off_470DB4      ;特征码2

.text:00408614                 mov     eax, esi

.text:00408616

.text:00408616 loc_408616:                             ; CODE XREF: sub_4084B0+180j

.text:00408616                 mov     dl, [eax]       ; 比较特征码

.text:00408618                 cmp     dl, [ecx]

.text:0040861A                 jnz     short loc_408636

.text:0040861C                 cmp     dl, bl

.text:0040861E                 jz      short loc_408632

.text:00408620                 mov     dl, [eax+1]

.text:00408623                 cmp     dl, [ecx+1]

.text:00408626                 jnz     short loc_408636

.text:00408628                 add     eax, 2

.text:0040862B                 add     ecx, 2

.text:0040862E                 cmp     dl, bl

.text:00408630                 jnz     short loc_408616

.text:00408632

.text:00408632 loc_408632:                             ; CODE XREF: sub_4084B0+16Ej

.text:00408632                 xor     eax, eax

.text:00408634                 jmp     short loc_40863B

.text:00408636 ; ---------------------------------------------------------------------------

.text:00408636

.text:00408636 loc_408636:                             ; CODE XREF: sub_4084B0+16Aj

.text:00408636                                         ; sub_4084B0+176j

.text:00408636                 sbb     eax, eax

.text:00408638                 sbb     eax, 0FFFFFFFFh

.text:0040863B

.text:0040863B loc_40863B:                             ; CODE XREF: sub_4084B0+184j

.text:0040863B                 cmp     eax, ebx

.text:0040863D                 jnz     short loc_408649

.text:0040863F                 mov     dword ptr [esp+edi+40h+Args], offset aVncEnterpris_0 ; "VNC Enterprise Edition for UNIX"

.text:00408647                 jmp     short loc_4086C1

.text:00408649 ; ---------------------------------------------------------------------------

.text:00408649

.text:00408649 loc_408649:                             ; CODE XREF: sub_4084B0+18Dj

.text:00408649                 mov     ecx, off_470DB0    ;特征码3

.text:0040864F                 mov     eax, esi

.text:00408651

.text:00408651 loc_408651:                             ; CODE XREF: sub_4084B0+1BBj

.text:00408651                 mov     dl, [eax]       ; 比较特征码

.text:00408653                 cmp     dl, [ecx]

.text:00408655                 jnz     short loc_408671

.text:00408657                 cmp     dl, bl

.text:00408659                 jz      short loc_40866D

.text:0040865B                 mov     dl, [eax+1]

.text:0040865E                 cmp     dl, [ecx+1]

.text:00408661                 jnz     short loc_408671

.text:00408663                 add     eax, 2

.text:00408666                 add     ecx, 2

.text:00408669                 cmp     dl, bl

.text:0040866B                 jnz     short loc_408651

.text:0040866D

.text:0040866D loc_40866D:                             ; CODE XREF: sub_4084B0+1A9j

.text:0040866D                 xor     eax, eax

.text:0040866F                 jmp     short loc_408676

.text:00408671 ; ---------------------------------------------------------------------------

.text:00408671

.text:00408671 loc_408671:                             ; CODE XREF: sub_4084B0+1A5j

.text:00408671                                         ; sub_4084B0+1B1j

.text:00408671                 sbb     eax, eax

.text:00408673                 sbb     eax, 0FFFFFFFFh

.text:00408676

.text:00408676 loc_408676:                             ; CODE XREF: sub_4084B0+1BFj

.text:00408676                 cmp     eax, ebx

.text:00408678                 jnz     short loc_408684

.text:0040867A                 mov     dword ptr [esp+edi+40h+Args], offset aVncEnterpris_1 ; "VNC Enterprise Edition for Mac OS X"

.text:00408682                 jmp     short loc_4086C1

.text:00408684 ; ---------------------------------------------------------------------------

.text:00408684

.text:00408684 loc_408684:                             ; CODE XREF: sub_4084B0+1C8j

.text:00408684                 mov     ecx, off_470DAC    ;特征码4

.text:0040868A                 mov     eax, esi

.text:0040868C                 lea     esp, [esp+0]

.text:00408690

.text:00408690 loc_408690:                             ; CODE XREF: sub_4084B0+1FAj

.text:00408690                 mov     dl, [eax]       ; 比较特征码

.text:00408692                 cmp     dl, [ecx]

.text:00408694                 jnz     short loc_4086B0

.text:00408696                 cmp     dl, bl

.text:00408698                 jz      short loc_4086AC

.text:0040869A                 mov     dl, [eax+1]

.text:0040869D                 cmp     dl, [ecx+1]

.text:004086A0                 jnz     short loc_4086B0

.text:004086A2                 add     eax, 2
```
