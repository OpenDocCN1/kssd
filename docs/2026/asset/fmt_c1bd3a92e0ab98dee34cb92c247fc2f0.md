
```
01003512  /$ 8B4424 08      MOV EAX,DWORD PTR SS:[ESP+8]

01003516  |. 53             PUSH EBX

01003517  |. 55             PUSH EBP

01003518  |. 56             PUSH ESI

01003519  |. 8B7424 10      MOV ESI,DWORD PTR SS:[ESP+10]

0100351D  |. 8BC8           MOV ECX,EAX

0100351F  |. C1E1 05        SHL ECX,5

01003522  |. 8D9431 4053000>LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]  // 分析后，发现这里是 winmine 地图的数据结构所在

01003529  |. F602 80        TEST BYTE PTR DS:[EDX],80

0100352C  |. 57             PUSH EDI

0100352D  |. 74 66          JE SHORT winmine.01003595         // 检查了是否踏中地雷

0100352F  |. 833D A4570001 >CMP DWORD PTR DS:[10057A4],0      // 踏中了，来到这里

01003536  |. 75 50          JNZ SHORT winmine.01003588
```

我们看了一下`LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]`，在 dump 中看到一段有规律的资料，很明显这些就是 winmine 地图的真正资料。

我们踏中了地雷，`0100352D` 不跳，到达下面。我们开始 F8 跟踪，看看有甚么发现。

```
0100352F  |. 833D A4570001 >CMP DWORD PTR DS:[10057A4],0

01003536  |. 75 50          JNZ SHORT winmine.01003588     // <-  跳了

01003538  |. 8B2D 38530001  MOV EBP,DWORD PTR DS:[1005338]

0100353E  |. 33C0           XOR EAX,EAX

...

01003586  |. EB 28          JMP SHORT winmine.010035B0
```

```
01003588  |> 6A 4C          PUSH 4C                           // <- 到了这里

0100358A  |. 50             PUSH EAX

0100358B  |. 56             PUSH ESI

0100358C  |. E8 1AF9FFFF    CALL winmine.01002EAB

01003591  |. 6A 00          PUSH 0

01003593  |. EB 16          JMP SHORT winmine.010035AB        // 强行跳了

01003595  |> 50             PUSH EAX                                 ; /Arg2

01003596  |. 56             PUSH ESI                                 ; |Arg1

01003597  |. E8 E8FAFFFF    CALL winmine.01003084                    ; \winmine.01003084

0100359C  |. A1 A4570001    MOV EAX,DWORD PTR DS:[10057A4]

010035A1  |. 3B05 A0570001  CMP EAX,DWORD PTR DS:[10057A0]

010035A7  |. 75 07          JNZ SHORT winmine.010035B0

010035A9  |. 6A 01          PUSH 1

010035AB  |> E8 CCFEFFFF    CALL winmine.0100347C             //  来到这里，很值得注意，因为上一次成功游戏，也是到这里

010035B0  |> 5F             POP EDI

010035B1  |. 5E             POP ESI

010035B2  |. 5D             POP EBP

010035B3  |. 5B             POP EBX

010035B4  \. C2 0800        RETN 8
```

很奇怪，最后我们再次到达`010035AB`的`CALL winmine.0100347C`，我们上一次成功了游戏，不是也到了这里吗？看清楚一点，今次的参数，是 push 了`0`，上一次是 push`1`。

意思是，`CALL winmine.0100347C`这个呼叫，如果传入了`1`，代表游戏成功，如果是`0`代表你踏中地雷。

到了现在的阶段，我们可以任意修改它了。

按照我们当初的目标，是使它在踏中了地雷时不会爆炸。

我们知道，一般的点按 (不是完成游戏，也没有踏中地雷)，是会跳过`010035AB`，直接来到`010035B0`的。我们现在只需要把踏中地雷的跳转修改，指向这里便行了。

```
01003586  |. EB 28          JMP SHORT winmine.010035B0

01003588  |> 6A 4C          PUSH 4C                           // <- 到了这里

0100358A  |. 50             PUSH EAX

0100358B  |. 56             PUSH ESI

0100358C  |. E8 1AF9FFFF    CALL winmine.01002EAB

01003591  |. 6A 00          PUSH 0                            //  <- 改这里

01003593  |. EB 16          JMP SHORT winmine.010035AB        // 强行跳了    <--  改这里
```

我们把`01003591` nop 了，再把`01003593`的 jmp 改成`JMP 010035B0`。

清除其它断点，重新开始游戏， F9 运行。

现在任意打开格子。

果然，即使踏中了地雷，它是显示了，但不会爆炸  :-)

## Part 3.   总结

近年软件安全的发展，走到了一个很复杂的地步，双方都花了大量 "气力"，不论破解或保护，都很费精神，花的体力巨大。我们对于一些简单的破解所带来的趣味，是很怀念的。这篇文章希望给新手们一次比较有趣的学习，只懂得使用简单 OD，不懂脱壳或算法的兄弟，也可以参与。

2007 年 5 月9 日

转载请保持内容完整

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
