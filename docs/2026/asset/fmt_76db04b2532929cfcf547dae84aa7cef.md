
```text
006904A6    E8 698ED7F>call EEV.00409314
006904AB    8BC6       mov eax,esi
006904AD    E8 0E44D7F>call EEV.004048C0
006904B2    33DB       xor ebx,ebx                     ; ebx清零
006904B4    85DB       test ebx,ebx
006904B6    75 15      jnz short EEV.006904CD
006904B8    56         push esi
006904B9    8BD3       mov edx,ebx
006904BB    C1E2 02    shl edx,2                       ; edx左移2
006904BE    B9 0400000>mov ecx,4                       ; 4送ecx
006904C3    8B45 FC    mov eax,dword ptr ss:[ebp-4]    ; ebp-4=A2B1334C5BF63029FA17728B232466BC，送eax
006904C6    E8 2549D7F>call EEV.00404DF0
006904CB    EB 2C      jmp short EEV.006904F9
006904CD    FF36       push dword ptr ds:[esi]         ; ds:[esi]=A2B1，即第一部分注册码
006904CF    68 4C05690>push EEV.0069054C
006904D4    8D45 D8    lea eax,dword ptr ss:[ebp-28]
006904D7    50         push eax
006904D8    8BD3       mov edx,ebx
006904DA    C1E2 02    shl edx,2
006904DD    B9 0400000>mov ecx,4
006904E2    8B45 FC    mov eax,dword ptr ss:[ebp-4]
006904E5    E8 0649D7F>call EEV.00404DF0
006904EA    FF75 D8    push dword ptr ss:[ebp-28]      ; ss:[ebp-28]=1344，即第二部分注册码
006904ED    8BC6       mov eax,esi
006904EF    BA 0300000>mov edx,3
006904F4    E8 5747D7F>call EEV.00404C50
006904F9    43         inc ebx                         ; ebx+1
006904FA    83FB 02    cmp ebx,2                       ; ebx与2比较
006904FD  ^ 75 B5      jnz short EEV.006904B4          ; 循环2次
006904FF    33C0       xor eax,eax
00690501    5A         pop edx
00690502    59         pop ecx
00690503    59         pop ecx
00690504    64:8910    mov dword ptr fs:[eax],edx
00690507    68 3105690>push EEV.00690531
0069050C    8D45 D8    lea eax,dword ptr ss:[ebp-28]
0069050F    BA 0200000>mov edx,2
00690514    E8 CB43D7F>call EEV.004048E4
00690519    8D45 E8    lea eax,dword ptr ss:[ebp-18]
0069051C    E8 9F43D7F>call EEV.004048C0
00690521    8D45 FC    lea eax,dword ptr ss:[ebp-4]
00690524    E8 9743D7F>call EEV.004048C0
00690529    C3         retn
```

---

跟进`call EEV.00404ED4`，来到这里

```text
00404ED4    85C0       test eax,eax
00404ED6    74 40      je short EEV.00404F18
00404ED8    85D2       test edx,edx
00404EDA    74 31      je short EEV.00404F0D
00404EDC    53         push ebx
00404EDD    56         push esi
00404EDE    57         push edi
00404EDF    89C6       mov esi,eax                     ; 注册码送esi
00404EE1    89D7       mov edi,edx                     ; 试练码送edi
00404EE3    8B4F FC    mov ecx,dword ptr ds:[edi-4]    ; ds:[edi-4]=9，送ecx
00404EE6    57         push edi
00404EE7    8B56 FC    mov edx,dword ptr ds:[esi-4]    ; ds:[esi-4]=9，送edx
00404EEA    4A         dec edx                         ; edx-1
00404EEB    78 1B      js short EEV.00404F08
00404EED    8A06       mov al,byte ptr ds:[esi]        ; 机器码ascii码逐位送al
00404EEF    46         inc esi                         ; esi+1
00404EF0    29D1       sub ecx,edx                     ; ecx-edx
00404EF2    7E 14      jle short EEV.00404F08
00404EF4    F2:AE      repne scas byte ptr es:[edi]    ; 循环比较注册码与试练码是否一致
00404EF6    75 10      jnz short EEV.00404F08          ; 不等则跳出
00404EF8    89CB       mov ebx,ecx
00404EFA    56         push esi
00404EFB    57         push edi
00404EFC    89D1       mov ecx,edx
00404EFE    F3:A6      repe cmps byte ptr es:[edi],byt>
00404F00    5F         pop edi
00404F01    5E         pop esi
00404F02    74 0C      je short EEV.00404F10
00404F04    89D9       mov ecx,ebx
00404F06  ^ EB EC      jmp short EEV.00404EF4
00404F08    5A         pop edx
00404F09    31C0       xor eax,eax
00404F0B    EB 08      jmp short EEV.00404F15
00404F0D    31C0       xor eax,eax
00404F0F    C3         retn
```

---

## 算法总结

软件采用了MD5算法，但也是很简单的，主要思路如下：

将机器码进行MD5运算，结果为`a2b1334c5bf63029fa17728b232466bc`（记做`codeA`），转成大写为`A2B1334C5BF63029FA17728B232466BC`（记做`codeB`），注册码为两部分：第一部分取`codeB`的1-4位；第二部分取注册码的4-7位；两部分合起来中间加上“-”，即为真正注册码。

我这里是：`MD5(886EA-AEF73-EB5BB-904FD)=a2b1334c5bf63029fa17728b232466bc`，注册码为`A2B1-1334`。

特别说明：本文仅是一些破解的心得和思路，完全是个人对程序的研究，无其他目的。![](https://bbs.kanxue.com/view/img/face/50.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
