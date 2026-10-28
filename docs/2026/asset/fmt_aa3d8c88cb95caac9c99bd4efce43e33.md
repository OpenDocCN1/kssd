
```text
.text:10003691                 push    0               ; dwCreationFlags

.text:10003693                 push    0               ; lpParameter

.text:10003695                 push    offset sub\_10005ACB ; lpStartAddress --->

.text:1000369A                 push    0               ; dwStackSize

.text:1000369C                 push    0               ; lpThreadAttributes

.text:1000369E                 call    ds:CreateThread

.text:100036A4

.text:100036A4 loc\_100036A4:                           ; CODE XREF: sub\_1000332A+34Fj

.text:100036A4                                         ; sub\_1000332A+363j

.text:100036A4                 cmp     dword\_1000846C, 0

.text:100036AB                 jz      short loc\_10003728

.text:100036AD                 lea     eax, [ebp+SystemTime]

.text:100036B3                 push    eax             ; lpSystemTime

.text:100036B4                 call    ds:GetLocalTime

.text:100036BA

.text:100036BA loc\_100036BA:                           ; CODE XREF: sub\_1000332A+3FCj

.text:100036BA                 xor     eax, eax

.text:100036BC                 inc     eax

.text:100036BD                 jz      short loc\_10003728

.text:100036BF                 lea     eax, [ebp+var\_14C]

.text:100036C5                 push    eax             ; lpSystemTime

.text:100036C6                 call    ds:GetLocalTime

.text:100036CC                 movzx   eax, [ebp+var\_144]

.text:100036D3                 movzx   ecx, [ebp+SystemTime.wHour]

.text:100036DA                 sub     eax, ecx

.text:100036DC                 cmp     eax, dword\_1000846C

.text:100036E2                 jl      short loc\_1000371B

.text:100036E4                 mov     ax, [ebp+var\_144]

.text:100036EB                 mov     [ebp+SystemTime.wHour], ax

.text:100036F2                 push    1

.text:100036F4                 movzx   eax, word\_10008450

.text:100036FB                 push    eax

.text:100036FC                 call    sub\_1000116C

.text:10003701                 cmp     eax, 1

.text:10003704                 jnz     short loc\_1000371B

.text:10003706                 push    0               ; lpThreadId

.text:10003708                 push    0               ; dwCreationFlags

.text:1000370A                 push    0               ; lpParameter

.text:1000370C                 push    offset sub\_10005ACB ; lpStartAddress  --->运行临时目录下的Newtemp\_387.exe

.text:10003711                 push    0               ; dwStackSize

.text:10003713                 push    0               ; lpThreadAttributes

.text:10003715                 call    ds:CreateThread

.text:1000371B

.text:1000371B loc\_1000371B:                           ; CODE XREF: sub\_1000332A+3B8j

.text:1000371B                                         ; sub\_1000332A+3DAj

.text:1000371B                 push    0EA60h          ; dwMilliseconds

.text:10003720                 call    ds:Sleep

.text:10003726                 jmp     short loc\_100036BA

.text:10003728 ; ---------------------------------------------------------------------------

.text:10003728

.text:10003728 loc\_10003728:                           ; CODE XREF: sub\_1000332A+381j

.text:10003728                                         ; sub\_1000332A+393j ...

.text:10003728                 xor     eax, eax

.text:1000372A                 inc     eax

.text:1000372B                 jz      short loc\_1000373A

.text:1000372D                 push    2710h           ; dwMilliseconds

.text:10003732                 call    ds:Sleep

.text:10003738                 jmp     short loc\_10003728

.text:1000373A ; ---------------------------------------------------------------------------

.text:1000373A

.text:1000373A loc\_1000373A:                           ; CODE XREF: sub\_1000332A+401j

.text:1000373A                 xor     eax, eax

.text:1000373C                 mov     ecx, [ebp+var\_10]

.text:1000373F                 mov     large fs:0, ecx

.text:10003746                 pop     edi

.text:10003747                 pop     esi

.text:10003748                 pop     ebx

.text:10003749                 leave

.text:1000374A                 retn    4

.text:1000374A sub\_1000332A    endp
```

上面我只做了简单的分析，大家可以根据我的思路，慢慢分析一下这个DLL，我这里就不详解了~~有不清楚的在讨论吧~~

当我在写完这篇文章时，我无间之间搜索一个函数的时候，发现原来这个病毒09年8月份被UPX上的一个人分析过~~嘿嘿~~我就当补充一下了~~`0e5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1M7r3q4U0K9#2)9J5k6h3y4F1i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1x3K6R3&6y4K6k6Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`。

要玩病毒：思想有多远，你就可以走多远~~~

睡了~~写一篇给新手的文章，很累，好了，以后有时间再写给大家吧~~![](https://bbs.kanxue.com/view/img/face/28.gif)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)

  （43.16kb，1313次下载）

- [2.jpg](javascript:void(0);)

  （43.47kb，1309次下载）

- [3.jpg](javascript:void(0);)

  （13.77kb，1302次下载）

- [4.jpg](javascript:void(0);)

  （5.96kb，1304次下载）

- [5.jpg](javascript:void(0);)

  （34.64kb，1304次下载）

- [6.jpg](javascript:void(0);)

  （67.85kb，1305次下载）

- [7.jpg](javascript:void(0);)

  （19.44kb，1300次下载）

- [分析.rar](javascript:void(0);)

  （496.61kb，276次下载）
