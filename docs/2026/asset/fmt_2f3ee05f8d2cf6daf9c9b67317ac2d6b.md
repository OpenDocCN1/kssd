
or (C)onditional Jump at Address:
|:00494016(C)

|
:00494021 83BD68FFFFFF00 `cmp` dword ptr [ebp+FFFFFF68],00000000
:00494028 0F8444010000 `je` 00494172
           ====>跳则OVER！
:0049402E DF6DE0
             `fild` qword ptr [ebp-20]
:00494031 DD9D60FFFFFF            `fstp` qword ptr [ebp+FFFFFF60]
:00494037 DD8560FFFFFF            `fld` qword ptr [ebp+FFFFFF60]
:0049403D DF6DF8                  `fild` qword ptr [ebp-08]
:00494040 DD9D60FFFFFF            `fstp` qword ptr [ebp+FFFFFF60]
:00494046 DC8D60FFFFFF            `fmul` qword ptr [ebp+FFFFFF60]
====>ST=633.00000000000000000 \* 561.0000000000000=355113.00000000000000
:0049404C DD9D60FFFFFF
       `fstp` qword ptr [ebp+FFFFFF60]
:00494052 DF6DF8                  `fild` qword ptr [ebp-08]
:00494055 DD9D58FFFFFF            `fstp` qword ptr [ebp+FFFFFF58]
:0049405B DD8558FFFFFF            `fld` qword ptr [ebp+FFFFFF58]
:00494061 DC0D39010000            `fmul` qword ptr [00000139]
====>ST=561.00000000000000000 \* 5.000000000000000=2805.0000000000000000
:00494067 DD9D58FFFFFF
       `fstp` qword ptr [ebp+FFFFFF58]
:0049406D DD8560FFFFFF            `fld` qword ptr [ebp+FFFFFF60]
:00494073 DC8558FFFFFF            `fadd` qword ptr [ebp+FFFFFF58]
====>ST=355113.00000000000000 + 2805.000000000000=357918.00000000000000
:00494079 DD9D50FFFFFF
       `fstp` qword ptr [ebp+FFFFFF50]
:0049407F 6801060080              `push` 80000601
:00494084 FFB554FFFFFF            `push` dword ptr [ebp+FFFFFF54]
:0049408A FFB550FFFFFF            `push` dword ptr [ebp+FFFFFF50]
:00494090 6801000000              `push` 00000001
:00494095 BB68010000              `mov` ebx,00000168
:0049409A E87DF20100              `call` 004B331C
:0049409F 83C410                  `add` esp,00000010
:004940A2 89854CFFFFFF            `mov` dword ptr [ebp+FFFFFF4C],eax
           ====>[ebp+FFFFFF4C]=357918
:004940A8 6801030080
         `push` 80000301
:004940AD 6A00
 `push` 00000000
:004940AF 6804000000              `push` 00000004
:004940B4 6804000080              `push` 80000004
:004940B9 6A00
 `push` 00000000
:004940BB 8B854CFFFFFF            `mov` eax,dword ptr [ebp+FFFFFF4C]
:004940C1 85C0
 `test` eax,eax
:004940C3 7505
 `jne` 004940CA
:004940C5 B8F3000000              `mov` eax,000000F3
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004940C3(C)
|
:004940CA 50
   `push` eax
:004940CB 6802000000              `push` 00000002
:004940D0 BB38010000              `mov` ebx,00000138
:004940D5 E842F20100              `call` 004B331C
           ====>取357918的后4位
:004940DA 83C41C
             `add` esp,0000001C
:004940DD 898548FFFFFF            `mov` dword ptr [ebp+FFFFFF48],eax
           ====>[ebp+FFFFFF78]=7918
:004940E3 8B9D4CFFFFFF
       `mov` ebx,dword ptr [ebp+FFFFFF4C]
:004940E9 85DB
 `test` ebx,ebx
:004940EB 7409
 `je` 004940F6
:004940ED 53
   `push` ebx
:004940EE E811F20100              `call` 004B3304
:004940F3 83C404                  `add` esp,00000004
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004940EB(C)
|
:004941F6 6AFF
 `push` FFFFFFFF
:004940F8 6A08
 `push` 00000008
:004940FA 6807070116              `push` 16010707
:004940FF 6800070152              `push` 52010700
:00494104 E819F20100              `call` 004B3322
:00494109 83C410                  `add` esp,00000010
:0049410C 898544FFFFFF            `mov` dword ptr [ebp+FFFFFF44],eax
           ====>[ebp+FFFFFF44]=3456
       试炼码的第四段
:00494112 8B8548FFFFFF
       `mov` eax,dword ptr [ebp+FFFFFF48]
           ====>EAX=[ebp+FFFFFF48]=7918
   注册码的第四段
:00494118 50
               `push` eax
:00494119 FFB544FFFFFF            `push` dword ptr [ebp+FFFFFF44]
:0049411F E88E48FDFF              `call` 004689B2
           ====>比较CALL！④
:00494124 83C408
             `add` esp,00000008
:00494127 83F800                  `cmp` eax,00000000
:0049412A B800000000              `mov` eax,00000000
:0049412F 0F94C0                  `sete` al
           ====>正确则AL=1
:00494132 898540FFFFFF
       `mov` dword ptr [ebp+FFFFFF40],eax
:00494138 8B9D44FFFFFF            `mov` ebx,dword ptr [ebp+FFFFFF44]
:0049413E 85DB
 `test` ebx,ebx
:00494140 7409
 `je` 0049414B
:00494142 53
   `push` ebx
:00494143 E8BCF10100              `call` 004B3304
:00494148 83C404                  `add` esp,00000004
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00494140(C)
|
:0049414B 8B9D48FFFFFF            `mov` ebx,dword ptr [ebp+FFFFFF48]
:00494151 85DB
 `test` ebx,ebx
:00494153 7409
 `je` 0049415E
:00494155 53
   `push` ebx
:00494156 E8A9F10100              `call` 004B3304
:0049415B 83C404                  `add` esp,00000004
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00494153(C)
|
:0049415E 83BD40FFFFFF00          `cmp` dword ptr [ebp+FFFFFF40],00000000
:00494165 0F8407000000            `je` 00494172
           ====>跳则OVER！
:0049416B B801000000
         `mov` eax,00000001
:00494170 EB02
 `jmp` 00494174
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00493E14(C), :00493F09(C), :00494028(C), :00494165(C)
|
:00494172 33C0
 `xor` eax,eax
           ====>跳到这里清0就OVER了！
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00494170(U)
|
:00494174 85C0
 `test` eax,eax
:00494176 0F84782C0000            `je` 00496DF4
…… ……省略…… ……  唉，很长一段代码都是保存注册信息的。有必要如此吗？:-(
:00495C14 E803D70100              `call` 004B331C
           ====>呵呵，胜利女神！
:00497273 E8A4C00100              `call` 004B331C
           ====>BAD BOY！
―――――――――――――――――――――――――――――――――
进入关键CALL：00493D9F  `call` 004B331C   取后4位字符
\* Referenced by a CALL at Addresses:
|:0046873A   , :00468846   , :004688FB   , :0046891F , :004689A6
…… …… 省 略 …… ……  如此多的地方来这里！
再进入：004B331C  `jmp` dword ptr [0000000C]    这里面是易语言的代码了  ^O^
再进入：100257ED  `call` dword ptr ds:[ebx]
```
10052730       56
    `push` esi
          ; KRNLN.100D6BD0
10052731       8B7424 10          `mov` esi,dword ptr ss:[esp+10]
10052735       57
    `push` edi
10052736       83C9 FF            `or` ecx,FFFFFFFF
10052739       8B16
  `mov` edx,dword ptr ds:[esi]
           ====>EDX=328587
1005273B       33C0
              `xor` eax,eax
1005273D       8BFA
  `mov` edi,edx
1005273F       F2:AE
 `repne` `scas` byte ptr es:[edi]
10052741       8B46 0C            `mov` eax,dword ptr ds:[esi+C]
           ====>EAX=[esi+C]=4
     这个是“法定长度”了
10052744       F7D1
              `not` ecx
10052746       49
    `dec` ecx
           ====>和1005273F处取328587长度  ECX=6
10052747       3BC8
              `cmp` ecx,eax
           ====>和4比较
10052749       8BF1
              `mov` esi,ecx
1005274B       7C 02
 `jl` short KRNLN.1005274F
1005274D       8BF0
  `mov` esi,eax
1005274F       5F
    `pop` edi
10052750       85F6
  `test` esi,esi
10052752       5E
    `pop` esi
10052753       7D 16
 `jge` short KRNLN.1005276B
10052755       33C0
  `xor` eax,eax
10052757       2BD0
  `sub` edx,eax
10052759       03D1
  `add` edx,ecx
1005275B       52
    `push` edx
1005275C       E8 6F97FFFF        `call` KRNLN.1004BED0
10052761       8B4C24 08          `mov` ecx,dword ptr ss:[esp+8]
10052765       83C4 04            `add` esp,4
10052768       8901
  `mov` dword ptr ds:[ecx],eax
1005276A       C3
    `retn`
1005276B       3BC8
              `cmp` ecx,eax
1005276D       7D 02
 `jge` short KRNLN.10052771
1005276F       8BC1
  `mov` eax,ecx
10052771       2BD0
  `sub` edx,eax
10052773       03D1
  `add` edx,ecx
           ====>呵呵，这2步就简单的完成取后4位了
10052775       52
                `push` edx
           ====>EDX=8587
10052776       E8
5597FFFF        `call` KRNLN.1004BED0
1005277B       8B4C24 08          `mov` ecx,dword ptr ss:[esp+8]
1005277F       83C4 04            `add` esp,4
10052782       8901
  `mov` dword ptr ds:[ecx],eax
10052784       C3
    `retn`
```
其他几部分取后4位的步骤是相同的。
―――――――――――――――――――――――――――――――――

### 【算 法 总 结】：

1、参考码：561584633321  分为4段：A=561  B=584  C=633  D=321

2、注册码1=A \* B + D \* 3=328587  的后4为数字：8587
