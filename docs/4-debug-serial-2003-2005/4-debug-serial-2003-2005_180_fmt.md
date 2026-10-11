
# L00P's SmartBrainForce

> 原文：[`www.pediy.com/kssd/pediy05/pediy50277.htm`](https://www.pediy.com/kssd/pediy05/pediy50277.htm)

相关附件可以到此下载:`http://www.dlking.com/perl/topic_show.cgi?id=20202&h=1&bpg=1&age=30`

```
===================================

= Target:  SmartBrainForce.exe    =

= Coder:  ^L00P                   =

= Date:    13.09.2002            =

===================================
```

```
=====================================================

验证过程很简单

=====================================================
```

1.
```
//条件 1:msum=4AA=(DEAD-377)/2F; include FF
{
    msum=buf[0]+buf[1]+...+buf[lken-1]=4AA
    buf[0]+buf[1]+...+buf[lken-2]=4AA-FF=3AB
}
```

2.
```
//条件 2:mxor=E1=(C0DE-63)/DB; include FF
{
    buf[0] xor buf[1] xor .... xor buf[len-1] xor 57=E1
}
```

3.
```
//条件 3:mmul=1EBB2=FACE*2+0-9EA
//条件 3:mmul=1EBB3=FACE*2+1-9EA
{
    mmul=buf[0]*(buf[1]+buf[2]+....+buf[len-2]);not include FF
}
```

4. 分析

```
{
    很容易知道,buf[len-1]=FF
    从条件 3 我们可以看出
        mmul 一定是第一个自己的倍数
    于是我们得到,
        1*1EBB2;2*F5D9;3*A3E6;6*51F3;7*463E;9*36A2;E*231F
        12*1B51;15*176A;1B*1236;25*D4A;2A*BB5;36*91B;3F*7CE
        4A*6A5;51*612;6F*46E;7E*3E7;A2*309;BD*29A;DE*237
        F3*206;//=========>103*1E6;14D*17A;
        1*1EBB3;5*6257;13*19E1;19*13AB;35*947;5F*52D;7D*3EF
        //================>109*1DB
    我们从条件 1 可以知道, 只有
        1EBB2=A2*309 是有效的
    于是我们得到方程:
    3AB=buf[0]+buf[1]+...+buf[lken-2];
    mmul=buf[0]*(buf[1]+buf[2]+....+buf[len-2]);
    这样我们就得到:
        buf[0]=A2;
        buf[1]+buf[2]+....+buf[len-2]=309;
        buf[len-1]=FF;
        buf[0] xor buf[1] xor .... xor buf[len-1] xor 57=E1
    也就是
        buf[1] xor buf [2] xor ... xor buf[len-2]=E1 xor 57 xor A2 xor FF=EB
    最后, 我们得到最终条件:
        buf[1]+buf[2]+....+buf[len-2]=309;
        buf[1] xor buf [2] xor ... xor buf[len-2]=EB;
        最简单的一种情况,
            根据异或的性质,
            len-2 是一个奇数, 让 buf[1]..buf[len-2], 其中一个是 EB, 其他的是成对数字
            309-EB=21E=2*10F
            这样, 我们把 10F 可以分成同样的部分, 不能超过 3 个部分, 也不能包含 FF
            10F=FE+11=...=87+88,
            10F=FE+11=...=87+88,
            10F=5A+5A+5B=...
            10F=5A+5A+5B=...
        (在 A2 和 FF 之间的数字顺序可以打乱)
        最后我们可以得到答案(当然, 我们还可以得到更多的答案):
        A2 EB FE 11 FE 11 FF 44 69 4B 65 4E
        A2 EB 54 5A 61 54 5A 61 FF 44 69 4B 65 4E
=====================================================
```

`DiKeN/iPB` `4dd466d0e1e0cc123afbe504da2462ca`
