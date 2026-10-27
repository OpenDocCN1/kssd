
# [原创]一个比较另类的ECC stuff

这个[`crackme`](https://bbs.kanxue.com/elink@508K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4J5j5h3y4C8L8h3g2K6i4K6u0W2k6r3g2Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4E0k6h3&6W2M7$3W2S2i4K6u0r3N6r3E0E0i4K6g2X3N6s2u0A6j5h3I4Q4y4h3j5J5x3o6l9@1i4K6u0r3k6r3!0%4L8X3I4G2j5h3b7`.) 是tkm 2004年的strainer,用ECC保护的。虽然ECC大家玩得比较熟了，不过这个东东的保护方法还是有点意思。

用PEiD查壳并脱壳，unpack不是目的，算法才是关键。unpack后载入ida,用miracl的sig，或

f0cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4W2j5%4u0W2N6r3q4K6K9r3g2D9L8q4)9J5k6h3y4G2L8g2)9J5c8X3y4J5P5i4m8@1L8$3y4J5K9#2)9J5c8Y4c8#2N6r3!0J5K9h3q4D9M7#2)9J5c8X3#2A6M7X3q4U0L8q4)9J5k6h3y4S2L8r3I4K6i4K6u0W2P5X3W2H3i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1&6i4K6R3%4i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1$3i4@1t1K6i4K6V1#2i4@1f1^5i4@1q4r3i4K6R3$3i4@1f1#2i4K6R3^5i4@1q4n7L8h3W2J5j5h3y4D9i4@1f1#2i4@1u0m8i4K6V1K6i4@1f1#2i4K6R3%4i4@1u0p5i4@1f1$3i4K6V1#2i4@1t1H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4@1p5K6i4K6R3H3i4@1f1$3i4K6W2r3i4@1p5#2i4@1f1#2i4@1u0m8i4K6S2r3i4@1f1#2i4K6R3^5i4K6V1%4i4@1f1#2i4K6S2r3i4@1t1%4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4@1t1#2i4K6R3I4i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1%4i4K6W2n7i4@1q4q4i4@1f1@1i4@1u0m8i4K6R3$3i4@1f1%4i4K6R3@1i4@1t1$3i4@1f1K6i4K6R3H3i4K6R3J5

## 1. 获取`name`,`sn1`,`sn2`

```
Un_FSG__:0040194C loc_40194C:                   

Un_FSG__:0040194C                 push    50h

Un_FSG__:0040194E                 push    offset name

Un_FSG__:00401953                 push    2

Un_FSG__:00401955                 push    edi

Un_FSG__:00401956                 call    GetDlgItemText ; name>=4

Un_FSG__:0040195B                 mov     esi, eax

Un_FSG__:0040195D                 push    50h

Un_FSG__:0040195F                 push    offset sn1

Un_FSG__:00401964                 push    3

Un_FSG__:00401966                 push    edi

Un_FSG__:00401967                 call    GetDlgItemText ; sn1>=4

Un_FSG__:0040196C                 mov     ebx, eax

Un_FSG__:0040196E                 push    50h

Un_FSG__:00401970                 push    offset sn2

Un_FSG__:00401975                 push    4

Un_FSG__:00401977                 push    edi

Un_FSG__:00401978                 call    GetDlgItemText ; sn2>=1
```

## 2. 检查序列号

### 1) 椭圆曲线E(`a`,`b`,`p`)、点(`pt1`,`pt2`,`pt3`,`pt4`,`pt5`)初始化

```
Un_FSG__:0040149C                 call    epoint_init

Un_FSG__:004014A1                 mov     edi, eax

Un_FSG__:004014A3                 mov     [esp+1F8h+var_1F8], 0

Un_FSG__:004014AA                 call    _mirvar

Un_FSG__:004014AF                 mov     [ebp+pt2_x], eax

Un_FSG__:004014B5                 call    epoint_init

Un_FSG__:004014BA                 mov     [esp+1F8h+var_1F8], 0

Un_FSG__:004014C1                 mov     [ebp+pt2], eax

Un_FSG__:004014C7                 call    _mirvar

Un_FSG__:004014CC                 mov     [ebp+big_t], eax

Un_FSG__:004014D2                 call    epoint_init

Un_FSG__:004014D7                 mov     [ebp+pt3], eax

Un_FSG__:004014DD                 call    epoint_init

Un_FSG__:004014E2                 mov     [ebp+pt4], eax

Un_FSG__:004014E8                 call    epoint_init

Un_FSG__:004014ED                 mov     dword ptr [esi+220h], 10h

Un_FSG__:004014F7                 mov     [ebp+pt5], eax

...

Un_FSG__:00401512                 mov     ebx, [ebp+a]

Un_FSG__:00401518                 push    offset a416d656e657369 ; "416D656E65736961"

Un_FSG__:0040151D                 push    ebx

Un_FSG__:0040151E                 call    _cinstr

Un_FSG__:00401523                 pop     eax

Un_FSG__:00401524                 pop     edx

Un_FSG__:00401525                 mov     eax, [ebp+b]

Un_FSG__:0040152B                 push    offset a1b35b7093fee5a ; "1B35B7093FEE5AE601A"

Un_FSG__:00401530                 push    eax

Un_FSG__:00401531                 call    _cinstr

Un_FSG__:00401536                 pop     ebx

Un_FSG__:00401537                 pop     esi

Un_FSG__:00401538                 mov     esi, [ebp+p]

Un_FSG__:0040153E                 push    offset aFffffffdffffff ; "FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF"

Un_FSG__:00401543                 push    esi

Un_FSG__:00401544                 call    _cinstr

Un_FSG__:00401549                 pop     eax

Un_FSG__:0040154A                 pop     edx

Un_FSG__:0040154B                 mov     edx, [ebp+pt1_x]

Un_FSG__:00401551                 push    offset a71263a72c2fdfb ; "71263A72C2FDFB8FE851182B408210A4"

Un_FSG__:00401556                 push    edx

Un_FSG__:00401557                 call    _cinstr

Un_FSG__:0040155C                 mov     eax, [ebp+pt2_x]

Un_FSG__:00401562                 pop     ecx

Un_FSG__:00401563                 pop     ebx

Un_FSG__:00401564                 push    offset a2c052124bdd94e ; "2C052124BDD94E5645E99B01DCECA28D"

Un_FSG__:00401569                 push    eax

Un_FSG__:0040156A                 call    _cinstr

Un_FSG__:0040156F                 mov     eax, [ebp+p]

Un_FSG__:00401575                 push    1

Un_FSG__:00401577                 push    eax

Un_FSG__:00401578                 mov     eax, [ebp+b]

Un_FSG__:0040157E                 push    eax

Un_FSG__:0040157F                 mov     eax, [ebp+a]

Un_FSG__:00401585                 push    eax

Un_FSG__:00401586                 call    _ecurve_init

Un_FSG__:0040158B                 mov     eax, [ebp+pt1_x]

Un_FSG__:00401591                 add     esp, 20h

Un_FSG__:00401594                 push    edi             ; pt1

Un_FSG__:00401595                 push    0

Un_FSG__:00401597                 push    eax

Un_FSG__:00401598                 mov     eax, [ebp+pt1_x]

Un_FSG__:0040159E                 push    eax

Un_FSG__:0040159F                 call    _epoint_set

Un_FSG__:004015A4                 mov     ecx, [ebp+pt2_x]

Un_FSG__:004015AA                 mov     eax, [ebp+pt2]

Un_FSG__:004015B0                 mov     ebx, [ebp+pt2_x]

Un_FSG__:004015B6                 push    eax

Un_FSG__:004015B7                 push    1

Un_FSG__:004015B9                 push    ebx

Un_FSG__:004015BA                 push    ecx

Un_FSG__:004015BB                 call    _epoint_set
```

其中`E`的参数为:

`a`:416D656E65736961

`b`:1B35B7093FEE5AE601A

`p`:FFFFFFFDFFFFFFFFFFFFFFFFFFFFFFFF

`pt1`的`x`坐标为:71263A72C2FDFB8FE851182B408210A4

`pt2`的`x`坐标为:2C052124BDD94E5645E99B01DCECA28D

### 2) `sn1`,`sn2`转变为大数,并变形

```
Un_FSG__:004015C0                 mov     esi, [ebp+big_sn1]

Un_FSG__:004015C6                 add     esp, 18h

Un_FSG__:004015C9                 movsx   ebx, ds:Serial_sign

Un_FSG__:004015D0                 push    offset sn1

Un_FSG__:004015D5                 sub     ebx, 30h

Un_FSG__:004015D8                 push    esi

Un_FSG__:004015D9                 lea     esi, [ebp+var_1A8]

Un_FSG__:004015DF                 call    _cinstr

Un_FSG__:004015E4                 pop     eax

Un_FSG__:004015E5                 pop     edx

Un_FSG__:004015E6                 mov     eax, [ebp+big_sn2]

Un_FSG__:004015EC                 push    offset sn2

Un_FSG__:004015F1                 push    eax

Un_FSG__:004015F2                 call    _cinstr
```

其中`sn1`的第一个`byte`是正负标志位，剩下的部分转化为大数
