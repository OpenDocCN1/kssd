
```
     ┃                                  *---*

     ┃

     ▽                                      off_401090

jmp ds:off_401090[ecx*4]              *-----------------*
                                   000| 401019h | cases_0   |
                                   004| 40102Ch | cases_1   |
                                   008| 40103Fh | cases_6   |
                                   012| 401052h | cases_7   |
                                   016| 401065h | cases_199 |
                          ecx*4--> 020| 40107Bh | default   |
                                      *-----------------*
```

通过上面这幅流程图，相信各位读者已经明白间接表是怎么回事了，那么下面我们就开始来点有挑战的。

# 1.6.1、switch-case分支结构与平衡二叉树

虽然我们前面所学的知识已经足够对付大多数情况，但是毕竟大多数不代表所有，而当我们学完本小节之后，那么各位读者的`switch-case`分支结构学习之旅才算是真正的告一段落。

通过上面的中转表的数据类型我们就能判断出他所能应付的只限于分支小于等于256的情况，如果超过256之后这种与跳转表相配合的中转表肯定会随之失效，取而代之的便是我们著名的二叉树了（准确的说应该叫平衡二叉树），有关于什么是二叉树或平衡二叉树笔者在这里很难以一个小节的容量将其介绍清楚，因此希望各位读者参考一下百度百科相关的内容，毕竟数据结构不是本文讨论的主题。

为了是各位读者可是顺利的阅读下去，笔者在这里为各位简单的为大家介绍一下二叉树。

所谓的二叉树查找法我们也可以暂时将其理解为折半查找法，例如我们想快速在1-7几个数字中找到某个数值，那么我们肯定是现将其与1-7的中间数4作比对看看它是比4小还是比4大，如果比4大的话那么就在4-7之间取一个中间数6与其比对，如果大于6那么这个数就是7，如果小于6那么这个数就是5了。如果将所有的可能性化成一个流程的话，那么这幅图大概就是以下这个样子：

```
    4

   / \
  2   6

/ \ / \
1 3 5 7
```

由此可知当比对次数（深度）为k时，则最多可以查找2^(k)-1个结点，例如如果用此算法查找419万条数据的某一条的话，那么其比对次数不会超过20次。

大致了解了二叉树的原理与优势之后，我们先看一段代码：

```c
int _tmain(int argc, _TCHAR* argv[])
{
    switch (argc)
    {
        case 1:
                printf("argc1=0",argc);
                break;
        case 92:
                printf("argc12=%d",argc);
                break;
        case 262:
                printf("argc1=%d",argc);
                break;
        case 118:
                printf("argc118=%d",argc);
                break;
        case 25:
                printf("argc25=%d",argc);
                break;
        case 456:
                printf("argc456=%d",argc);
                break;
        case 588:
                printf("argc588=%d",argc);
                break;
        case 896:
                printf("argc896=0",argc);
                break;
        case 1000:
                printf("argc1000=%d",argc);
                break;
        case 1090:
                printf("argc1090=%d",argc);
                break;
        case 2100:
                printf("argc2100=%d",argc);
                break;
    default:
        printf("default nNum=%d,error!",argc);
    }
    return 0;
}
```

下面是Release版的反汇编代码，有部分删截……

```asm
.text:00401000 ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:00401000 _main           proc near               ; CODE XREF: __tmainCRTStartup+10A p
.text:00401000
.text:00401000 argc            = dword ptr  4
.text:00401000 argv            = dword ptr  8
.text:00401000 envp            = dword ptr  0Ch
.text:00401000
.text:00401000                 mov     eax, [esp+argc]
.text:00401004                 cmp     eax, 1C8h
.text:00401009                 jg      loc_4010B0
.text:0040100F                 jz      loc_40109A
.text:00401015                 cmp     eax, 5Ch
.text:00401018                 jg      short loc_401065
.text:0040101A                 jz      short loc_401052
.text:0040101C                 mov     ecx, eax
.text:0040101E                 sub     ecx, 1
.text:00401021                 jz      short loc_40103F
.text:00401023                 sub     ecx, 18h
.text:00401026                 jnz     loc_401113
.text:0040102C                 push    19h
.text:0040102E                 push    offset Format   ; "argc25=%d"
......  ......                 ......
.text:0040103F ; ---------------------------------------------------------------------------
.text:0040103F
.text:0040103F loc_40103F:                             ; CODE XREF: _main+21j
.text:0040103F                 push    1
.text:00401041                 push    offset aArgc10  ; "argc1=0"
......  ......                 ......
.text:00401052 ; ---------------------------------------------------------------------------
.text:00401052
.text:00401052 loc_401052:                             ; CODE XREF: _main+1Aj
.text:00401052                 push    5Ch
.text:00401054                 push    offset aArgc12D ; "argc12=%d"
......  ......                 ......
.text:00401065 ; ---------------------------------------------------------------------------
.text:00401065
.text:00401065 loc_401065:                             ; CODE XREF: _main+18j
.text:00401065                 cmp     eax, 76h
.text:00401068                 jz      short loc_401087
.text:0040106A                 cmp     eax, 106h
.text:0040106F                 jnz     loc_401113
.text:00401075                 push    eax
.text:00401076                 push    offset aArgc1D  ; "argc1=%d"
......  ......                 ......
.text:00401087 ; ---------------------------------------------------------------------------
.text:00401087
.text:00401087 loc_401087:                             ; CODE XREF: _main+68j
.text:00401087                 push    76h
.text:00401089                 push    offset aArgc118D ; "argc118=%d"
......  ......                 ......
.text:0040109A ; ---------------------------------------------------------------------------
.text:0040109A
.text:0040109A loc_40109A:                             ; CODE XREF: _main+Fj
.text:0040109A                 push    1C8h
.text:0040109F                 push    offset aArgc456D ; "argc456=%d"
......  ......                 ......
.text:004010B0 ; ---------------------------------------------------------------------------
.text:004010B0
.text:004010B0 loc_4010B0:                             ; CODE XREF: _main+9j
.text:004010B0                 cmp     eax, 3E8h
.text:004010B5                 jg      short loc_401105
.text:004010B7                 jz      short loc_4010EF
.text:004010B9                 cmp     eax, 24Ch
.text:004010BE                 jz      short loc_4010D9
.text:004010C0                 cmp     eax, 380h
.text:004010C5                 jnz     short loc_401113
.text:004010C7                 push    eax
.text:004010C8                 push    offset aArgc8960 ; "argc896=0"
......  ......                 ......
.text:004010D9 ; ---------------------------------------------------------------------------
.text:004010D9
.text:004010D9 loc_4010D9:                             ; CODE XREF: _main+BEj
.text:004010D9                 push    24Ch
.text:004010DE                 push    offset aArgc588D ; "argc588=%d"
......  ......                 ......
.text:004010EF ; ---------------------------------------------------------------------------
.text:004010EF
.text:004010EF loc_4010EF:                             ; CODE XREF: _main+B7j
.text:004010EF                 push    3E8h
.text:004010F4                 push    offset aArgc1000D ; "argc1000=%d"
......  ......                 ......
.text:00401105 ; ---------------------------------------------------------------------------
.text:00401105
.text:00401105 loc_401105:                             ; CODE XREF: _main+B5j
.text:00401105                 cmp     eax, 442h
.text:0040110A                 jz      short loc_40113B
.text:0040110C                 cmp     eax, 834h
.text:00401111                 jz      short loc_401125
.text:00401113
.text:00401113 loc_401113:                             ; CODE XREF: _main+26j
.text:00401113                                         ; _main+6Fj ...
.text:00401113                 push    eax
.text:00401114                 push    offset aDefaultNnumDEr ; "default nNum=%d,error!"
......  ......                 ......
.text:00401125 ; ---------------------------------------------------------------------------
.text:00401125
.text:00401125 loc_401125:                             ; CODE XREF: _main+111j
.text:00401125                 push    834h
```
