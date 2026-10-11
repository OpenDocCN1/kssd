
### 内存中的序列数据
`"YD2,ZHHFLEV3,FEECYFXZ,2GFLDZAA,L2HADH2F,FCV3F2"`
`"AX,8C9X4CHA,FFZXV2EE,LXGGFH9F,ZVEEFCEF,V2VG5AH"`
`"X,ZEY2F25C,2ZDXEA32,LX9CYCH3,2XZCFY9D,ZZE2AEAX"`
`",VHG9DFFZ,VE232GYC,YGE2"`

### 汇编代码分析与执行流程
```
:0042174C BFB8794900                mov edi, 004979B8
:00421751 83C9FF                    or ecx, FFFFFFFF
:00421754 33C0                      xor eax, eax
:00421756 8D942470220000           lea edx, dword ptr [esp+00002270]
:0042175D F2                        repnz
:0042175E AE                        scasb
:0042175F F7D1                      not ecx
                                <===ecx=7D7
:00421761 2BF9                      sub edi, ecx
:00421763 8BC1                      mov eax, ecx
:00421765 8BF7                      mov esi, edi
:00421767 8BFA                      mov edi, edx
:00421769 8D9424D8190000           lea edx, dword ptr [esp+000019D8]
:00421770 C1E902                    shr ecx, 02
                                <===ecx=7D7 div 04=1F5
:00421773 F3                        repz
:00421774 A5                        movsd
:00421775 8BC8                      mov ecx, eax
:00421777 83E103                    and ecx, 00000003
:0042177A F3                        repz
:0042177B A4                        movsb
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
:0042177C 8D4C240E                 lea ecx, dword ptr [esp+0E]
:00421780 51                        push ecx
                                <===ecx 指向在内存中表示为:
0079544A  2C 00 45 4C 34 49 43 47 45 34 2C 47 46 41 34 43  ,.EL4ICGE4,GFA4C
0079545A  47 47 42 2C 45 36 47 4C 47 47 41 43 2C 44 46 47  GGB,E6GLGGAC,DFG
0079546A  36 44 36 4C 46 2C 45 34 43 47 45 43 41 41 2C 43  6M6LF,E4CGECAA,C
0079547A  41 44 43 47 47 47 34 2C 46 44 4C 44 47 44 44 47  ADCGGG4,FDLDGDDG
0079548A  2C 42 36 47 41 35 46 42 42 2C 47 41 46 43 4C 47  ,B6GA5FBB,GAFCLG
0079549A  36 41 2C 47 46 41 4C 36 4C 4C 43 2C 37 35 54 34  6A,GFAL6LLC,75T4
.......................(略,好多哦!)
:00421781 52                        push edx
                                <===edx 指向在内存中表示为:
00796E14  39 47 44 44 43 43 47 39 2C 33 39 44 47 43 46 47  9GDDCCG9,39DGCFG
00796E24  39 2C 39 46 39 44 47 32 39 43 2C 44 39 47 32 47  9,9F9DG29C,D9G2G
00796E34  34 46 45 2C 45 34 45 43 44 43 47 45 2C 36 34 33  4FE,E4ECDCGE,643
00796E44  43 32 45 33 39 2C 43 43 47 39 44 34 34 45 2C 58  C2E39,CCG9D44E,X
00796E54  34 47 39 36 33 34 57 2C 47 47 32 44 39 47 45 58  4G9634W,GG2D9GEX
00796E64  2C 45 45 32 39 39 39 44 43 2C 39 41 33 32 46 45  ,EE2999DC,9A32FE
................(略,也是很多哦!就是正确的注册码随便挑一个吧!)
:00421782 E851EE0200               call 004505D8
                                <===这个 Call 我也跟进去,可是看了半天也没看
                                出与我们输入的信息有何关系!
★★★★★★★★★★★★★★★★★★★★★★★★
上面这么多代码看起来很复杂,但以我的菜鸟水平实在看不出与我们所输入的用户名
或注册码有何关系,猜想是故意迷惑我们 cracker 的视线,我认为最后的结果在每台机
器上应该是一样的.还请高手指正!
:00421787 8BBC24142B0000           mov edi, dword ptr [esp+00002B14]
                                <===edi="22072008912",试验码
:0042178E 83C408                   add esp, 00000008
:00421791 85C0                     test eax, eax
:00421793 744A                     je 004217DF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004217DD(C)
|
:00421795 8BF7                      mov esi, edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004217B9(C)
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
:00421797 8A10                      mov dl, byte ptr [eax]
                                <===[eax]="9GDDCCG9",eax 指向上面的内存
                                地址 00796E14
:00421799 8A1E                      mov bl, byte ptr [esi]
                                <===[esi]="22072008912",试验码
:0042179D 3AD3                      cmp dl, bl
:0042179F 751E                      jne 004217BF
:004217A1 84C9                      test cl, cl
:004217A3 7416                      je 004217BB
:004217A5 8A5001                    mov dl, byte ptr [eax+01]
:004217A8 8A5E01                    mov bl, byte ptr [esi+01]
:004217AB 8ACA                      mov cl, dl
:004217AD 3AD3                      cmp dl, bl
:004217AF 750E                      jne 004217BF
:004217B1 83C002                    add eax, 00000002
:004217B4 83C602                    add esi, 00000002
:004217B7 84C9                      test cl, cl
:004217B9 75DC                      jne 00421797
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
又是一个经典的比较句式!下面还有很多这种经典的比较,就是说注册码不止一个,有很
多哦!任君挑选! : )

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004217A3(C)
|
:004217BB 33C0                       xor eax, eax
                                        <===eax=0
:004217BD EB05                       jmp 004217C4

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0042179F(C), :004217AF(C)
|
:004217BF 1BC0                       sbb eax, eax
:004217C1 83D8FF                     sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004217BD(U)
|
:004217C4 85C0                       test eax, eax
:004217C6 0F84A6010000               je 00421972
:004217CC 8D44240E                   lea eax, dword ptr [esp+0E]
:004217D0 50                         push eax
:004217D1 6A00                       push 00000000
:004217D3 E800EE0200                 call 004505D8
:004217D8 83C408                     add esp, 00000008
:004217DB 85C0                       test eax, eax
:004217DD 75B6                       jne 00421795

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421793(C)
|
:004217DF 8D4C240E                   lea ecx, dword ptr [esp+0E]
:004217E3 8D542410                   lea edx, dword ptr [esp+10]
:004217E7 51                         push ecx
:004217E8 52                         push edx
:004217E9 E8EAED0200                 call 004505D8
:004217EE 83C408                     add esp, 00000008
:004217F1 85C0                       test eax, eax
:004217F3 744A                       je 0042183F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042183D(C)
|
:004217F5 8BF7                       mov esi, edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421819(C)
|
:004217F7 8A10                       mov dl, byte ptr [eax]
:004217F9 8A1E                       mov bl, byte ptr [esi]
:004217FB 8ACA                       mov cl, dl
:004217FD 3AD3                       cmp dl, bl
:004217FF 751E                       jne 0042181F
:00421801 84C9                       test cl, cl
:00421803 7416                       je 0042181B
:00421805 8A5001                    mov dl, byte ptr [eax+01]
:00421808 8A5E01                    mov bl, byte ptr [esi+01]
:0042180B 8ACA                      mov cl, dl
:0042180D 3AD3                      cmp dl, bl
:0042180F 750E                      jne 0042181F
:00421811 83C002                    add eax, 00000002
:00421814 83C602                    add esi, 00000002
:00421817 84C9                      test cl, cl
:00421819 75DC                      jne 004217F7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421803(C)
|
:0042181B 33C0                       xor eax, eax
:0042181D EB05                       jmp 00421824

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004217FF(C), :0042180F(C)
|
:0042181F 1BC0                       sbb eax, eax
```
