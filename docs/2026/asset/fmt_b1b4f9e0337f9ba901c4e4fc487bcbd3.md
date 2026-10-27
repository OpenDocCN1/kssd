
```
.text:0046932C                 call    ds:\_\_vbaGenerateBoundsError

.text:00469332                 mov     [ebp+var\_240], eax

.text:00469338

.text:00469338 loc\_469338:                             ; CODE XREF: sub\_468060+12CAj

.text:00469338                 movsx   edx, [ebp+var\_BC] ; 取当前索引

.text:0046933F                 mov     [ebp+var\_15C], edx ; 保存到这里

.text:00469345                 cmp     [ebp+var\_15C], 11h

.text:0046934C                 jnb     short loc\_46935A

.text:0046934E                 mov     [ebp+var\_244], 0

.text:00469358                 jmp     short loc\_469366

.text:0046935A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:0046935A

.text:0046935A loc\_46935A:                             ; CODE XREF: sub\_468060+12ECj

.text:0046935A                 call    ds:\_\_vbaGenerateBoundsError

.text:00469360                 mov     [ebp+var\_244], eax

.text:00469366

.text:00469366 loc\_469366:                             ; CODE XREF: sub\_468060+12F8j

.text:00469366                 mov     eax, [ebp+index\_158] ; 当前索引

.text:0046936C                 mov     ecx, [ebp+var\_D0] ; 数组5地址

.text:00469372                 mov     edx, [ebp+var\_15C] ; 当前索引

.text:00469378                 mov     esi, [ebp+var\_80] ; 数组3地址

.text:0046937B                 mov     ax, [ecx+eax\*2] ; 取数组5一个元素

.text:0046937F                 xor     ax, [esi+edx\*2] ; 和数组3一个元素异或

.text:00469383                 mov     cx, ax

.text:00469386                 and     cx, 1FFh        ; 模512

.text:0046938B                 jns     short loc\_469396

.text:0046938D                 dec     cx

.text:0046938F                 or      cx, 0FE00h

.text:00469394                 inc     cx

.text:00469396

.text:00469396 loc\_469396:                             ; CODE XREF: sub\_468060+132Bj

.text:00469396                 sub     cx, [ebp+mysum\_B4] ; 减去var\_b4

.text:0046939D                 jo      loc\_469788

.text:004693A3                 call    ds:\_\_vbaI2Abs   ; 取绝对值

.text:004693A9                 mov     word ptr [ebp+var\_E8], ax ; 保存到var\_e8

.text:004693B0                 mov     [ebp+var\_4], 63h

.text:004693B7                 cmp     [ebp+var\_EC], 3

.text:004693BF                 jnz     loc\_4695D5      ; var\_ec不等于3则跳

.text:004693C5                 mov     [ebp+var\_4], 64h

.text:004693CC                 cmp     word ptr [ebp+var\_E8], 10h

.text:004693D4                 jge     loc\_46948D      ; 绝对值大于等于16则跳

.text:004693DA                 mov     [ebp+var\_4], 65h

.text:004693E1                 mov     [ebp+var\_138], offset a0 ; "0"

.text:004693EB                 mov     [ebp+var\_140], 8 ; var\_140 = "0"

.text:004693F5                 lea     ecx, [ebp+var\_E8]

.text:004693FB                 mov     [ebp+var\_128], ecx

.text:00469401                 mov     [ebp+var\_130], 4002h

.text:0046940B                 lea     edx, [ebp+var\_130] ; 绝对值

.text:00469411                 push    edx

.text:00469412                 lea     eax, [ebp+var\_110]

.text:00469418                 push    eax

.text:00469419                 call    ds:rtcHexVarFromVar ; Hex(绝对值)

.text:0046941F                 lea     ecx, [ebp+var\_50] ; var\_50

.text:00469422                 push    ecx

.text:00469423                 movsx   edx, [ebp+var\_FC] ; 位置

.text:0046942A                 push    edx

.text:0046942B                 push    2               ; 长度

.text:0046942D                 lea     eax, [ebp+var\_140] ; "0"

.text:00469433                 push    eax

.text:00469434                 lea     ecx, [ebp+var\_110] ; 绝对值转成的十六进制字符串

.text:0046943A                 push    ecx

.text:0046943B                 lea     edx, [ebp+var\_120]

.text:00469441                 push    edx

.text:00469442                 call    ds:\_\_vbaVarCat

.text:00469448                 push    eax

.text:00469449                 call    ds:\_\_vbaStrVarMove

.text:0046944F                 mov     edx, eax

.text:00469451                 lea     ecx, [ebp+var\_100] ; 保存到这里

.text:00469457                 call    ds:\_\_vbaStrMove

.text:0046945D                 push    eax

.text:0046945E                 push    0

.text:00469460                 call    ds:\_\_vbaMidStmtBstr ; Mid(var50,var\_fc,2) = "0" & hex(绝对值)

.text:00469466                 lea     ecx, [ebp+var\_100]

.text:0046946C                 call    ds:\_\_vbaFreeStr

.text:00469472                 lea     eax, [ebp+var\_120]

.text:00469478                 push    eax

.text:00469479                 lea     ecx, [ebp+var\_110]

.text:0046947F                 push    ecx

.text:00469480                 push    2

.text:00469482                 call    ds:\_\_vbaFreeVarList

.text:00469488                 add     esp, 0Ch

.text:0046948B                 jmp     short loc\_469508

.text:0046948D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:0046948D

.text:0046948D loc\_46948D:                             ; CODE XREF: sub\_468060+1374j

.text:0046948D                 mov     [ebp+var\_4], 67h

.text:00469494                 lea     edx, [ebp+var\_E8] ; 取绝对值

.text:0046949A                 mov     [ebp+var\_128], edx

.text:004694A0                 mov     [ebp+var\_130], 4002h

.text:004694AA                 lea     eax, [ebp+var\_130]

.text:004694B0                 push    eax

.text:004694B1                 lea     ecx, [ebp+var\_110]

.text:004694B7                 push    ecx

.text:004694B8                 call    ds:rtcHexVarFromVar ; Hex(绝对值)

.text:004694BE                 lea     edx, [ebp+var\_50]

.text:004694C1                 push    edx

.text:004694C2                 movsx   eax, [ebp+var\_FC] ; 位置

.text:004694C9                 push    eax

.text:004694CA                 push    2               ; 长度

.text:004694CC                 lea     ecx, [ebp+var\_110]

.text:004694D2                 push    ecx

.text:004694D3                 call    ds:\_\_vbaStrVarMove

.text:004694D9                 mov     edx, eax

.text:004694DB                 lea     ecx, [ebp+var\_100]

.text:004694E1                 call    ds:\_\_vbaStrMove

.text:004694E7                 push    eax

.text:004694E8                 push    0

.text:004694EA                 call    ds:\_\_vbaMidStmtBstr ; mid(var50,var\_fc,2) = hex(绝对值)

.text:004694F0                 lea     ecx, [ebp+var\_100]

.text:004694F6                 call    ds:\_\_vbaFreeStr

.text:004694FC                 lea     ecx, [ebp+var\_110]

.text:00469502                 call    ds:\_\_vbaFreeVar

.text:00469508

.text:00469508 loc\_469508:                             ; CODE XREF: sub\_468060+142Bj

.text:00469508                 mov     [ebp+var\_4], 69h

.text:0046950F                 cmp     [ebp+var\_F4], 2

.text:00469517                 jnz     short loc\_469571 ; var\_f4不等于2则跳

.text:00469519                 cmp     [ebp+var\_FC], 12h

.text:00469521                 jge     short loc\_469571 ; var\_fc大于等于18则跳

.text:00469523                 mov     [ebp+var\_4], 6Ah

.text:0046952A                 mov     dx, [ebp+var\_FC]

.text:00469531                 add     dx, 1

.text:00469535                 jo      loc\_469788

.text:0046953B                 mov     [ebp+var\_FC], dx ; var\_fc加1

.text:00469542                 mov     [ebp+var\_4], 6Bh

.text:00469549                 lea     eax, [ebp+var\_50]

.text:0046954C                 push    eax

.text:0046954D                 mov     cx, [ebp+var\_FC] ; 取var\_fc

.text:00469554                 add     cx, 1           ; 加1

.text:00469558                 jo      loc\_469788

.text:0046955E                 movsx   edx, cx         ; 保存到edx

.text:00469561                 push    edx

.text:00469562                 push    1

.text:00469564                 push    offset asc\_40D030 ; "-"

.text:00469569                 push    0

.text:0046956B                 call    ds:\_\_vbaMidStmtBstr ; mid(var\_50,var\_fc+1,1) = "-"
```
