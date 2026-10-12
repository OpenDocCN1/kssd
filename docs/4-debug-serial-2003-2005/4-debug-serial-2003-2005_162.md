
# Hidownload 注册方法总结

> 原文：[`www.pediy.com/kssd/pediy06/pediy6315.htm`](https://www.pediy.com/kssd/pediy06/pediy6315.htm)

[`www.hidownload.com`](http://www.hidownload.com)

*   下载工具
*   试用天数限制
*   `AsPack2.12`
*   `pe-scan` 脱壳
*   `Delphi6.0` 的作品

首先输入用户名和电子邮件地址，和随便一个序列号，比如：

```
用户名:dnpf
电子邮件:dnpf@sohu.com
注册码:695ec0ee7352c033530309d0ad3418cb
```

然后在 `0068A51A` 地址处下中断，拦截到之后，分别看 `EAX` 和 `EDX` 的内容：

```
EAX:b0351f56a39e0f21d9736d09278cd3c2
EDX:e7352c033530309d0ad3418cb
```

然后按照上面我的分析，关键运算只和输入的注册码的前面某部分有关，那我们就把它组合一下，看看 `EDX` 的内容和我们输入的注册码的差别了吧，我们就取前面的部分“`695ec0e`”，然后我们知道 `EAX` 的内容就是根据这个部分生成的，并且最后还要和 `EDX` 的内容比较，我们就把 `EAX` 的内容作为我们注册码的后面部分“`b0351f56a39e0f21d9736d09278cd3c2`”连接起来就是“`695ec0eb0351f56a39e0f21d9736d09278cd3c2`”

最后整理一下：

```
用户名:dnpf-------------------
电子邮件:dnpf@sohu.com----------/这两部分不参与运算，可以任意输入
注册码:695ec0eb0351f56a39e0f21d9736d09278cd3c2
```

***

`Dede` 反汇编，过程页面单元名里面找到 `URegistrationWin` ，这个就是注册窗口，在右边找到注册按钮对应的事件 `ButtRegClick` ，查看其代码:

```
005A3A70   55                     push    ebp
005A3A71   8BEC                   mov     ebp, esp
005A3A73   B909000000             mov     ecx, $00000009
005A3A78   6A00                   push    $00
005A3A7A   6A00                   push    $00
005A3A7C   49                     dec     ecx
005A3A7D   75F9                   jnz     005A3A78
005A3A7F   8945FC                 mov     [ebp-$04], eax
005A3A82   33C0                   xor     eax, eax
005A3A84   55                     push    ebp
005A3A85   68463D5A00             push    $005A3D46

***** TRY

005A3A8A   64FF30                 push    dword ptr fs:[eax]
005A3A8D   648920                 mov     fs:[eax], esp
005A3A90   8D55F4                 lea     edx, [ebp-$0C]---------------->保存注册码的新地址
005A3A93   8B45FC                 mov     eax, [ebp-$04]

* Reference to control Registrationkey : TLabel

005A3A96   8B8004030000           mov     eax, [eax+$0304]

* Reference to: controls.TControl.GetText(TControl):System.String;

005A3A9C   E83BC4EBFF             call    0045FEDC------------------------>取注册码
005A3AA1   8B45F4                 mov     eax, [ebp-$0C]------------------>注册码
005A3AA4   50                     push    eax\--------------------------->入栈
005A3AA5   8D55E8                 lea     edx, [ebp-$18]------------------>保存用户名的新地址
005A3AA8   8B45FC                 mov     eax, [ebp-$04]

* Reference to control RegisteredUser : TLabel

005A3AAB   8B8000030000           mov     eax, [eax+$0300]

* Reference to: controls.TControl.GetText(TControl):System.String;

005A3AB1   E826C4EBFF             call    0045FEDC---------------------->取用户名
005A3AB6   8B45E8                 mov     eax, [ebp-$18]---------------->用户名
005A3AB9   8D55EC                 lea     edx, [ebp-$14]

* Reference to: sysutils.Trim(System.AnsiString):System.AnsiString;

005A3ABC   E8275CE6FF             call    004096E8----------------------->再保存用户名
005A3AC1   FF75EC                 push    dword ptr [ebp-$14]
005A3AC4   685C3D5A00             push    $005A3D5C
005A3AC9   8D55E0                 lea     edx, [ebp-$20]----------------->保存电子邮件的新地址
005A3ACC   8B45FC                 mov     eax, [ebp-$04]

* Reference to control MechineCode : TEdit

005A3ACF   8B8018030000           mov     eax, [eax+$0318]

* Reference to: controls.TControl.GetText(TControl):System.String;

005A3AD5   E802C4EBFF             call    0045FEDC----------------------->取电子邮件地址
005A3ADA   8B45E0                 mov     eax, [ebp-$20]----------------->电子邮件地址
005A3ADD   8D55E4                 lea     edx, [ebp-$1C]

* Reference to: sysutils.Trim(System.AnsiString):System.AnsiString;

005A3AE0   E8035CE6FF             call    004096E8
005A3AE5   FF75E4                 push    dword ptr [ebp-$1C]
005A3AE8   8D45F0                 lea     eax, [ebp-$10]
005A3AEB   BA03000000             mov     edx, $00000003

005A3AF0   E80B14E6FF             call    00404F00------------------->用户名和电子邮件地址连接在一起(用:连接)
005A3AF5   8B45F0                 mov     eax, [ebp-$10]------------->连接后的形式
005A3AF8   5A                     pop     edx\------------------------>注册码

005A3AF9   E826690E00             call    0068A424------------------->验证
005A3AFE   84C0                   test    al, al\--------------------->作为标志
005A3B00   0F84BB010000            jz      005A3CC1------------------->为零失败
005A3B06   33C0                   xor     eax, eax
005A3B08   55                     push    ebp
005A3B09   687E3C5A00             push    $005A3C7E

{上面用户名和注册码各取了两次，因为用户名和电子邮件连接在一起是调用另外的 CALL 完成的，Delphi 参数传递默认是使用复制值}

……

中间代码省略

……

* Reference to : TApplication._PROC_0044DABC()

005A3CBC   E8FB9DEAFF             call    0044DABC
005A3CC1   33C0                   xor     eax, eax\------------------->跳到这里说明注册失败
005A3CC3   5A                     pop     edx
005A3CC4   59                     pop     ecx
005A3CC5   59                     pop     ecx
005A3CC6   648910                 mov     fs:[eax], edx

****** FINALLY

* Possible String Reference to: '嬪]?

005A3CC9   684D3D5A00             push    $005A3D4D
005A3CCE   8D45B8                 lea     eax, [ebp-$48]

……

之后代码省略

*********************************************************************
*********************************************************************

以下内容为验证 CALL,最终 EAX 为零表示失败

0068A424   $  55              PUSH EBP
0068A425   .  8BEC            MOV EBP,ESP
0068A427   .  83C4 D8         ADD ESP,-28
0068A42A   .  53              PUSH EBX
0068A42B   .  56              PUSH ESI
0068A42C   .  57              PUSH EDI
0068A42D   .  33C9            XOR ECX,ECX
0068A42F   .  894D F0        MOV DWORD PTR SS:[EBP-10],ECX
0068A432   .  894D D8        MOV DWORD PTR SS:[EBP-28],ECX
0068A435   .  894D DC        MOV DWORD PTR SS:[EBP-24],ECX
0068A438   .  894D F4        MOV DWORD PTR SS:[EBP-C],ECX
0068A43B   .  894D F8        MOV DWORD PTR SS:[EBP-8],ECX
0068A43E   .  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
0068A441   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0068A444   .  E8 DFABD7FF    CALL HIDOWNLO.00405028
0068A449   .  33C0           XOR EAX,EAX
0068A44B   .  55              PUSH EBP
0068A44C   .  68 65A56800    PUSH HIDOWNLO.0068A565
0068A451   .  64:FF30        PUSH DWORD PTR FS:[EAX]
0068A454   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
0068A457   .  33C0           XOR EAX,EAX
0068A459   .  55              PUSH EBP
0068A45A   .  68 31A56800    PUSH HIDOWNLO.0068A531
0068A45F   .  64:FF30        PUSH DWORD PTR FS:[EAX]
0068A462   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
0068A465   .  33C0           XOR EAX,EAX
0068A467   .  55              PUSH EBP
0068A468   .  68 9DA46800    PUSH HIDOWNLO.0068A49D
0068A46D   .  64:FF30        PUSH DWORD PTR FS:[EAX]
0068A470   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
0068A473   .  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
0068A476   .  50              PUSH EAX
0068A477   .  B9 01000000    MOV ECX,1
0068A47C   .  BA 01000000    MOV EDX,1
0068A481   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0068A484   .  E8 0FACD7FF    CALL HIDOWNLO.00405098--------------->取注册码首位
0068A489   .  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]--------->首位字符
0068A48C   .  E8 D7F5D7FF    CALL HIDOWNLO.00409A68--------------->转化为字符形式(39->9)
```



### 代码段 1

```
0068A491   .  8BD8            MOV EBX,EAX\-------------------------->保存   

0068A493   .  33C0            XOR EAX,EAX   

0068A495   .  5A             POP EDX   

0068A496   .  59             POP ECX   

0068A497   .  59             POP ECX   

0068A498   .  64:8910        MOV DWORD PTR FS:[EAX],EDX   

0068A49B   .  EB 1E          JMP SHORT HIDOWNLO.0068A4BB   

0068A49D   .^ E9 7E9DD7FF    JMP HIDOWNLO.00404220   

0068A4A2   .  33DB           XOR EBX,EBX   

0068A4A4   .  E8 DFA0D7FF    CALL HIDOWNLO.00404588   

0068A4A9   .  33C0           XOR EAX,EAX   

0068A4AB   .  5A            POP EDX   

0068A4AC   .  59            POP ECX   

0068A4AD   .  59            POP ECX   

0068A4AE   .  64:8910       MOV DWORD PTR FS:[EAX],EDX   

0068A4B1   .  E9 87000000   JMP HIDOWNLO.0068A53D   

0068A4B6   .  E8 CDA0D7FF   CALL HIDOWNLO.00404588   

0068A4BB   >  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]------------>新地址   

0068A4BE   .  50            PUSH EAX   

0068A4BF   .  8BCB          MOV ECX,EBX\----------------------------->首位字符   

0068A4C1   .  BA 02000000   MOV EDX,2   

0068A4C6   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]------------>注册码   

0068A4C9   .  E8 CAABD7FF   CALL HIDOWNLO.00405098---------------->取去掉首位后注册码的"首位值"位   

0068A4CE   .  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]----------->新地址   

0068A4D1   .  B9 80A56800   MOV ECX,HIDOWNLO.0068A580--------------->ASCII "1.16"   

0068A4D6   .  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]---------->取得的去掉首位后注册码的"首位值"位   

0068A4D9   .  E8 AEA9D7FF   CALL HIDOWNLO.00404E8C------------------>上面两部分字符连续连接到一起   

0068A4DE   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]----------->连接后的字符串   

0068A4E1   .  8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]----------->新地址   

0068A4E4   .  E8 CBDBEAFF   CALL HIDOWNLO.005380B4------------------>\*\*\*\*\*\*关键运算\*\*\*\*\*\*   

0068A4E9   .  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]----------->运算结果(以字符保存在内存)   

0068A4EC   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]----------->新地址   

0068A4EF   .  E8 34DCEAFF   CALL HIDOWNLO.00538128------------------>将关键运算的结果从内存读取到新地址   

0068A4F4   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]----------->运算结果   

0068A4F7   .  50            PUSH EAX\-------------------------------->保存   

0068A4F8   .  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]----------->新地址   

0068A4FB   .  50            PUSH EAX   

0068A4FC   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]------------>输入的注册码   

0068A4FF   .  E8 3CA9D7FF   CALL HIDOWNLO.00404E40------------------>取长度   

0068A504   .  8BC8          MOV ECX,EAX\----------------------------->保存长度值   

0068A506   .  8D43 01       LEA EAX,DWORD PTR DS:[EBX+1]------------>首位值+1   

0068A509   .  2BC8          SUB ECX,EAX\----------------------------->求差值   

0068A50B   .  8D53 02       LEA EDX,DWORD PTR DS:[EBX+2]------------>首位值+2   

0068A50E   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]------------>输入的注册码   

0068A511   .  E8 82ABD7FF   CALL HIDOWNLO.00405098------------------>从注册码的结尾开始取"差值"位注册码   

0068A516   .  8B55 D8       MOV EDX,DWORD PTR SS:[EBP-28]----------->取得的"差值"位注册码   

0068A519   .  58            POP EAX\--------------------------------->上面关键运算的结果   

0068A51A   .  E8 65AAD7FF   CALL HIDOWNLO.00404F84------------------>比较   

0068A51F   .  75 04         JNZ SHORT HIDOWNLO.0068A525------------->关键跳(不相等则跳,则错)\*\*\*\*\*\*   

0068A521   .  B3 01         MOV BL,1-------------------------------->如果上面不跳,BL\=1   

0068A523   .  EB 02         JMP SHORT HIDOWNLO.0068A527   

0068A525   >  33DB          XOR EBX,EBX\----------------------------->如果上面跳到这里,则 BL=0   

0068A527   >  33C0          XOR EAX,EAX   

0068A529   .  5A            POP EDX   

0068A52A   .  59            POP ECX   

0068A52B   .  59            POP ECX   

0068A52C   .  64:8910       MOV DWORD PTR FS:[EAX],EDX   

0068A52F   .  EB 0C         JMP SHORT HIDOWNLO.0068A53D   

0068A531   .^ E9 EA9CD7FF   JMP HIDOWNLO.00404220   

0068A536   .  33DB          XOR EBX,EBX   

0068A538   .  E8 4BA0D7FF   CALL HIDOWNLO.00404588   

0068A53D   >  33C0          XOR EAX,EAX   

0068A53F   .  5A            POP EDX   

0068A540   .  59            POP ECX   

0068A541   .  59            POP ECX   

0068A542   .  64:8910       MOV DWORD PTR FS:[EAX],EDX   

0068A545   .  68 6CA56800   PUSH HIDOWNLO.0068A56C   

0068A54A   >  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]   

0068A54D   .  BA 02000000   MOV EDX,2   

0068A552   .  E8 39A6D7FF   CALL HIDOWNLO.00404B90   

0068A557   .  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]   

0068A55A   .  BA 04000000   MOV EDX,4   

0068A55F   .  E8 2CA6D7FF   CALL HIDOWNLO.00404B90   

0068A564   .  C3            RETN   

0068A565   .^ E9 6A9FD7FF   JMP HIDOWNLO.004044D4   

0068A56A   .^ EB DE         JMP SHORT HIDOWNLO.0068A54A   

0068A56C   .  8BC3          MOV EAX,EBX\------------------------------>上面 EBX 的值给 EAX   

0068A56E   .  5F            POP EDI   

0068A56F   .  5E            POP ESI   

0068A570   .  5B            POP EBX   

0068A571   .  8BE5          MOV ESP,EBP   

0068A573   .  5D            POP EBP   

0068A574   .  C3            RETN   
```

### 本段内容总结

设：

*   `L` := 注册码的长度
*   `F` := 注册码的首位字符

则注册码的最后(`L`-`F`-1)个字符的值应当与关键运算的结果相同.

关键运算使用了（注册码－首位字符）的前面的“首位”个字符，并且用到了一个我不认识的加密方法。

另:

关于关键运算部分

*   运算结果(以字符保存在内存):比如内存中内容为"76 E1 08 B0 B0 FE 7F 46 61 9F 2B 49 FD D8 2F 8C"
*   将关键运算的结果从内存读取到新地址:内存中为"37 36 65 31 30 38 62 30 62 30 66 65 37 66 34 36   
    36 31 39 66 32 62 34 39 66 64 64 38 32 66 38 63"

{表达能力不好}

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

### 关键运算

`005380B4`

本段的关键是对开始时的`[EDX]`的处理,最终运算结果都保存在`[EDX]`

#### 005380B4

```
005380B4  /$  55             PUSH EBP   

005380B5  |.  8BEC           MOV EBP,ESP   

005380B7  |.  83C4 A4        ADD ESP,-5C   

005380BA  |.  53             PUSH EBX   

005380BB  |.  8BDA           MOV EBX,EDX\---------------------->[EDX]=用户名:电子邮件   

005380BD  |.  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX\----->截取的注册码+1.16   

005380C0  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]----->截取的注册码+1.16   

005380C3  |.  E8 60CFECFF    CALL HIDOWNLO.00405028----------->   

005380C8  |.  33C0           XOR EAX,EAX   

005380CA  |.  55             PUSH EBP   

005380CB  |.  68 1A815300    PUSH HIDOWNLO.0053811A   

005380D0  |.  64:FF30        PUSH DWORD PTR FS:[EAX]   

005380D3  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP   

005380D6  |.  8D45 A4        LEA EAX,DWORD PTR SS:[EBP-5C]---->新地址   

005380D9  |.  E8 AEFEFFFF    CALL HIDOWNLO.00537F8C----->上面的地址"01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10"   

005380DE  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]----->截取的注册码+1.16   

005380E1  |.  E8 5ACDECFF    CALL HIDOWNLO.00404E40----------->取长度   

005380E6  |.  50             PUSH EAX\------------------------->长度值   

005380E7  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]----->截取的注册码+1.16   

005380EA  |.  E8 49CFECFF    CALL HIDOWNLO.00405038----------->判断长度   

005380EF  |.  8BD0           MOV EDX,EAX\---------------------->截取的注册码+1.16   

005380F1  |.  8D45 A4        LEA EAX,DWORD PTR SS:[EBP-5C]---->内存"01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10"   

005380F4  |.  59             POP ECX\-------------------------->(截取的注册码+1.16)的长度   

005380F5  |.  E8 C6FEFFFF    CALL HIDOWNLO.00537FC0----------->   

005380FA  |.  8BD3           MOV EDX,EBX\---------------------->关键地址   

005380FC  |.  8D45 A4        LEA EAX,DWORD PTR SS:[EBP-5C]---->内存"01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10"   

005380FF  |.  E8 3CFFFFFF    CALL HIDOWNLO.00538040----------->\*\*\*\*关键运算二\*\*\*\*   

00538104  |.  33C0           XOR EAX,EAX   

00538106  |.  5A             POP EDX   
```



### 代码段 00538107 - 00538125

```
00538107  |.  59             POP ECX   
00538108  |.  59             POP ECX   
00538109  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX   
0053810C  |.  68 21815300   PUSH HIDOWNLO.00538121   
00538111  |>  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]---->恢复截取的注册码+1.16   
00538114  |.  E8 53CAECFF   CALL HIDOWNLO.00404B6C   
00538119  .  C3             RETN   
0053811A   .^ E9 B5C3ECFF   JMP HIDOWNLO.004044D4   
0053811F   .^ EB F0         JMP SHORT HIDOWNLO.00538111   
00538121   .  5B             POP EBX   
00538122   .  8BE5          MOV ESP,EBP   
00538124   .  5D             POP EBP   
00538125   .  C3             RETN   
```

***

### 关键运算二

```
00538040  /$  53             PUSH EBX   
00538041  |.  56             PUSH ESI   
00538042  |.  83C4 F8       ADD ESP,-8   
00538045  |.  8BF2          MOV ESI,EDX   
00538047  |.  8BD8          MOV EBX,EAX   
00538049  |.  8BD4          MOV EDX,ESP   
0053804B  |.  8D43 10      LEA EAX,DWORD PTR DS:[EBX+10]---------->   
0053804E  |.  B9 02000000  MOV ECX,2   
00538053  |.  E8 C8F7FFFF  CALL HIDOWNLO.00537820----------------->   
00538058  |.  8B43 10      MOV EAX,DWORD PTR DS:[EBX+10]---------->   
0053805B  |.  C1E8 03       SHR EAX,3   
0053805E  |.  83E0 3F       AND EAX,3F   
00538061  |.  83F8 38      CMP EAX,38----------------------------->EAX 的值和数字 8 比较   
00538064  |.  73 0B         JNB SHORT HIDOWNLO.00538071------------>不小于则跳   
00538066  |.  BA 38000000  MOV EDX,38   
0053806B  |.  2BD0          SUB EDX,EAX   
0053806D  |.  8BC2          MOV EAX,EDX   
0053806F  |.  EB 09         JMP SHORT HIDOWNLO.0053807A   
00538071  |>  BA 78000000  MOV EDX,78   
00538076  |.  2BD0          SUB EDX,EAX   
00538078  |.  8BC2          MOV EAX,EDX   
0053807A  |>  BA 4C126C00  MOV EDX,HIDOWNLO.006C124C   
0053807F  |.  8BCB          MOV ECX,EBX   
00538081  |.  91             XCHG EAX,ECX   
00538082  |.  E8 39FFFFFF  CALL HIDOWNLO.00537FC0----------------->   
00538087  |.  8BD4          MOV EDX,ESP  --------------------------->   
00538089  |.  8BC3          MOV EAX,EBX -->内存"01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10"(此地址生成关键内容)   
0053808B  |.  B9 08000000  MOV ECX,8  ----------------------------->8   
00538090  |.  E8 2BFFFFFF  CALL HIDOWNLO.00537FC0----------------->关键四(生成下面的关键内容)   
00538095  |.  8BD6          MOV EDX,ESI  --------------------------->关键地址   
00538097  |.  8BC3          MOV EAX,EBX  --------------------------->关键内容   
00538099  |.  B9 04000000  MOV ECX,4   
0053809E  |.  E8 7DF7FFFF  CALL HIDOWNLO.00537820 ---------------->关键三   
005380A3  |.  8BC3          MOV EAX,EBX   
005380A5  |.  BA 58000000  MOV EDX,58   
005380AA  |.  E8 E100EDFF  CALL HIDOWNLO.00408190   
005380AF  |.  59             POP ECX   
005380B0  |.  5A             POP EDX   
005380B1  |.  5E             POP ESI   
005380B2  |.  5B             POP EBX   
005380B3  .  C3             RETN   
```

***

### 关键三

本段在程序重启和输入注册码以后都有多次调用

```
00537820  /$  56             PUSH ESI   
00537821  |.  8BF0          MOV ESI,EAX  --------------------->关键内容   
00537823  |.  8BC2          MOV EAX,EDX  --------------------->关键地址   
00537825  |.  8BD1          MOV EDX,ECX  --------------------->计数   
00537827  |.  85D2          TEST EDX,EDX   
00537829  |.  76 2F         JBE SHORT HIDOWNLO.0053785A   
0053782B  |>  8A0E         /MOV CL,BYTE PTR DS:[ESI] ------>取关键内容的首位   
0053782D  |.  80E1 FF      |AND CL,0FF -------------------->屏蔽其他位(取最末位)   
00537830  |.  8808         |MOV BYTE PTR DS:[EAX],CL ------>保存第一位   
00537832  |.  40           |INC EAX ----------------------->指向下一位   
00537833  |.  8B0E         |MOV ECX,DWORD PTR DS:[ESI] ---->取关键内容的首位   
00537835  |.  C1E9 08      |SHR ECX,8 --------------------->逻辑右移 8 位   
00537838  |.  80E1 FF      |AND CL,0FF -------------------->屏蔽其他位(取第三位)   
0053783B  |.  8808         |MOV BYTE PTR DS:[EAX],CL ------>保存   
0053783D  |.  40           |INC EAX ----------------------->指向下一位   
0053783E  |.  8B0E         |MOV ECX,DWORD PTR DS:[ESI] ---->取关键内容的首位   
00537840  |.  C1E9 10      |SHR ECX,10 -------------------->逻辑右移 16 位   
00537843  |.  80E1 FF      |AND CL,0FF -------------------->屏蔽其他位(取第二位)   
00537846  |.  8808         |MOV BYTE PTR DS:[EAX],CL ------>保存   
00537848  |.  40           |INC EAX ----------------------->指向下一位   
00537849  |.  8B0E         |MOV ECX,DWORD PTR DS:[ESI] ---->取关键内容的首位   
0053784B  |.  C1E9 18      |SHR ECX,18 -------------------->逻辑右移 24 位   
0053784E  |.  80E1 FF      |AND CL,0FF -------------------->屏蔽其他位(取第一位)   
00537851  |.  8808         |MOV BYTE PTR DS:[EAX],CL ------>保存   
00537853  |.  40           |INC EAX ----------------------->指向下一位   
00537854  |.  83C6 04      |ADD ESI,4 --------------------->指向关键内容的后 4 位   
00537857  |.  4A           |DEC EDX ----------------------->计数器-1   
00537858  |.^ 75 D1        JNZ SHORT HIDOWNLO.0053782B -->判断是否结束   
0053785A  |>  5E             POP ESI   
0053785B  .  C3             RETN   
```

总结:将关键内容反向读取保存,一共 16 位。

由于本段执行时生成的比较内容在内存中是的形式是`"76 E1 08 B0 B0 FE 7F 46 61 9F 2B 49 FD D8 2F 8C"`所以本段的循环中每保存一位都是保存了两个字符,如开头的`"76"`,由本段的内容可知,需要找到关键内容的生成部分。于是从关键三继续向上找,得到关键四

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

### 关键四

作用:生成关键三使用的关键内容,`EBX` 的地址

```
00537FC0  /$  53             PUSH EBX   
00537FC1  |.  56             PUSH ESI  
00537FC2  |.  57             PUSH EDI   
00537FC3  |.  55             PUSH EBP   
00537FC4  |.  8BF9          MOV EDI,ECX  -------------------------->计数器   
00537FC6  |.  8BEA          MOV EBP,EDX   
00537FC8  |.  8BF0          MOV ESI,EAX  -------------------------->生成关键内容的地址   
00537FCA  |.  8B46 10      MOV EAX,DWORD PTR DS:[ESI+10]--------->紧接关键内容后的 4 字节   
00537FCD  |.  C1E8 03       SHR EAX,3  ---------------------------->逻辑右移 3 位   
00537FD0  |.  83E0 3F       AND EAX,3F --------------------------->(保留最末 6 位)屏蔽其余位   
00537FD3  |.  8BD7          MOV EDX,EDI  -------------------------->计数器   
00537FD5  |.  C1E2 03       SHL EDX,3  ---------------------------->逻辑左移 3 位   
00537FD8  |.  0156 10      ADD DWORD PTR DS:[ESI+10],EDX -------->结果加到上述地址   
00537FDB  |.  3B56 10      CMP EDX,DWORD PTR DS:[ESI+10] -------->比较   
00537FDE  |.  76 03         JBE SHORT HIDOWNLO.00537FE3 ---------->若小于等于则跳(即加法之后无溢出则跳)   
00537FE0  |.  FF46 14      INC DWORD PTR DS:[ESI+14]------------->有溢出则设置高位   
00537FE3  |>  8BD7          MOV EDX,EDI  -------------------------->计数器   
00537FE5  |.  C1EA 1D       SHR EDX,1D  --------------------------->逻辑右移 13 位   
00537FE8  |.  0156 14      ADD DWORD PTR DS:[ESI+14],EDX -------->结果加到高位上   
00537FEB  |.  BB 40000000  MOV EBX,40  --------------------------->EBX=40h   
00537FF0  |.  2BD8          SUB EBX,EAX  -------------------------->求差(EAX 为关键地址第一次运算后的结果)   
00537FF2  |.  3BDF          CMP EBX,EDI  -------------------------->比较(EDI 为计数器初始值)   
00537FF4  |.  77 32         JA SHORT HIDOWNLO.00538028  ----------->大于则跳   
00537FF6  |.  8D4406 18    LEA EAX,DWORD PTR DS:[ESI+EAX+18] ---->否则 EAX 取[]   
00537FFA  |.  8BCB          MOV ECX,EBX  -------------------------->差值   
00537FFC  |.  8BD5          MOV EDX,EBP   
00537FFE  |.  E8 8501EDFF  CALL HIDOWNLO.00408188   
```



```
00538003  |.  8BD6          MOV EDX,ESI ---->内存"01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10"(此地址生成关键内容)   

00538005  |.  8D46 18       LEA EAX,DWORD PTR DS:[ESI+18] -------->注册码   

00538008  |.  E8 4FF8FFFF   CALL HIDOWNLO.0053785C -------------->******   

0053800D  |.  EB 0E        JMP SHORT HIDOWNLO.0053801D   

0053800F  |>  8BD6         /MOV EDX,ESI   

00538011  |.  8D441D 00    |LEA EAX,DWORD PTR SS:[EBP+EBX]   

00538015  |.  E8 42F8FFFF   |CALL HIDOWNLO.0053785C   

0053801A  |.  83C3 40       |ADD EBX,40   

0053801D  |>  8D43 3F        LEA EAX,DWORD PTR DS:[EBX+3F]   

00538020  |.  3BF8         |CMP EDI,EAX   

00538022  |.^ 77 EB        JA SHORT HIDOWNLO.0053800F   

00538024  |.  33C0         XOR EAX,EAX   

00538026  |.  EB 02        JMP SHORT HIDOWNLO.0053802A   

00538028  |>  33DB         XOR EBX,EBX   

0053802A  |>  8D4406 18    LEA EAX,DWORD PTR DS:[ESI+EAX+18]   

0053802E  |.  8BCF         MOV ECX,EDI   

00538030  |.  2BCB         SUB ECX,EBX   

00538032  |.  8D541D 00    LEA EDX,DWORD PTR SS:[EBP+EBX]   

00538036  |.  E8 4D01EDFF  CALL HIDOWNLO.00408188   

0053803B  |.  5D           POP EBP   

0053803C  |.  5F           POP EDI   

0053803D  |.  5E           POP ESI   

0053803E  |.  5B           POP EBX   

0053803F  .  C3           RETN   

-------------------------------------------------------------------------------------------------   

0053785C  /$  53           PUSH EBX   

0053785D  |.  56           PUSH ESI   

0053785E  |.  57           PUSH EDI   

0053785F  |.  55           PUSH EBP   

00537860  |.  83C4 A8      ADD ESP,-58   

00537863  |.  895424 04    MOV DWORD PTR SS:[ESP+4],EDX   

00537867  |.  890424       MOV DWORD PTR SS:[ESP],EAX   

0053786A  |.  8D5C24 08    LEA EBX,DWORD PTR SS:[ESP+8]   

0053786E  |.  8D7424 0C    LEA ESI,DWORD PTR SS:[ESP+C]   

00537872  |.  8D7C24 10    LEA EDI,DWORD PTR SS:[ESP+10]   

00537876  |.  8D6C24 14    LEA EBP,DWORD PTR SS:[ESP+14]   

0053787A  |.  8D5424 18    LEA EDX,DWORD PTR SS:[ESP+18]   

0053787E  |.  B9 40000000  MOV ECX,40   

00537883  |.  8B0424       MOV EAX,DWORD PTR SS:[ESP]   

00537886  |.  E8 5DFFFFFF  CALL HIDOWNLO.005377E8   

0053788B  |.  8B4424 04    MOV EAX,DWORD PTR SS:[ESP+4]   

0053788F  |.  8B00         MOV EAX,DWORD PTR DS:[EAX]   

00537891  |.  8903         MOV DWORD PTR DS:[EBX],EAX   

00537893  |.  8B4424 04    MOV EAX,DWORD PTR SS:[ESP+4]   

00537897  |.  8B40 04      MOV EAX,DWORD PTR DS:[EAX+4]   

0053789A  |.  8906         MOV DWORD PTR DS:[ESI],EAX   

0053789C  |.  8B4424 04    MOV EAX,DWORD PTR SS:[ESP+4]   

005378A0  |.  8B40 08      MOV EAX,DWORD PTR DS:[EAX+8]   

005378A3  |.  8907         MOV DWORD PTR DS:[EDI],EAX   

005378A5  |.  8B4424 04    MOV EAX,DWORD PTR SS:[ESP+4]   

005378A9  |.  8B40 0C      MOV EAX,DWORD PTR DS:[EAX+C]   

005378AC  |.  8945 00      MOV DWORD PTR SS:[EBP],EAX   

005378AF  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]*****************************************************************①   

005378B2  |.  50           PUSH EAX                     ; /Arg4   

005378B3  |.  8B4424 1C    MOV EAX,DWORD PTR SS:[ESP+1C]    ; |   

005378B7  |.  50           PUSH EAX                     ; |Arg3   

005378B8  |.  6A 07        PUSH 7                       ; |Arg2 = 00000007   

005378BA  |.  68 78A46AD7  PUSH D76AA478                ; |Arg1 = D76AA478---------->◎◎◎   

005378BF  |.  8BC3         MOV EAX,EBX                  ; |   

005378C1  |.  8B0F         MOV ECX,DWORD PTR DS:[EDI]    ; |   

005378C3  |.  8B16         MOV EDX,DWORD PTR DS:[ESI]    ; |   

005378C5  |.  E8 4EFEFFFF  CALL HIDOWNLO.00537718       ; HIDOWNLO.00537718   

005378CA  |.  8B07         MOV EAX,DWORD PTR DS:[EDI]   

005378CC  |.  50           PUSH EAX                     ; /Arg4   

005378CD  |.  8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]    ; |   

005378D1  |.  50           PUSH EAX                     ; |Arg3   

005378D2  |.  6A 0C        PUSH 0C                      ; |Arg2 = 0000000C   

005378D4  |.  68 56B7C7E8  PUSH E8C7B756                ; |Arg1 = E8C7B756---------->◎◎◎   

005378D9  |.  8BC5         MOV EAX,EBP                  ; |   

005378DB  |.  8B0E         MOV ECX,DWORD PTR DS:[ESI]    ; |   

005378DD  |.  8B13         MOV EDX,DWORD PTR DS:[EBX]    ; |   

005378DF  |.  E8 34FEFFFF  CALL HIDOWNLO.00537718       ; HIDOWNLO.00537718   

005378E4  |.  8B06         MOV EAX,DWORD PTR DS:[ESI]   

005378E6  |.  50           PUSH EAX                     ; /Arg4   

005378E7  |.  8B4424 24    MOV EAX,DWORD PTR SS:[ESP+24]    ; |   

005378EB  |.  50           PUSH EAX                     ; |Arg3   

005378EC  |.  6A 11        PUSH 11                      ; |Arg2 = 00000011   

005378EE  |.  68 DB702024  PUSH 242070DB                ; |Arg1 = 242070DB---------->◎◎◎   

005378F3  |.  8BC7         MOV EAX,EDI                  ; |   

005378F5  |.  8B0B         MOV ECX,DWORD PTR DS:[EBX]    ; |   

005378F7  |.  8B55 00      MOV EDX,DWORD PTR SS:[EBP]    ; |   

005378FA  |.  E8 19FEFFFF  CALL HIDOWNLO.00537718       ; HIDOWNLO.00537718   

005378FF  |.  8B03         MOV EAX,DWORD PTR DS:[EBX]   

00537901  |.  50           PUSH EAX                     ; /Arg4   

00537902  |.  8B4424 28    MOV EAX,DWORD PTR SS:[ESP+28]    ; |   

00537906  |.  50           PUSH EAX                     ; |Arg3   

00537907  |.  6A 16        PUSH 16                      ; |Arg2 = 00000016   

00537909  |.  68 EECEBDC1  PUSH C1BDCEEE                ; |Arg1 = C1BDCEEE---------->◎◎◎   

0053790E  |.  8BC6         MOV EAX,ESI                  ; |   

00537910  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]    ; |   

00537913  |.  8B17         MOV EDX,DWORD PTR DS:[EDI]    ; |   

00537915  |.  E8 FEFDFFFF  CALL HIDOWNLO.00537718       ; HIDOWNLO.00537718   

0053791A  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]   

0053791D  |.  50           PUSH EAX                     ; /Arg4   

0053791E  |.  8B4424 2C    MOV EAX,DWORD PTR SS:[ESP+2C]    ; |   

00537922  |.  50           PUSH EAX                     ; |Arg3   

00537923  |.  6A 07        PUSH 7                       ; |Arg2 = 00000007   

00537925  |.  68 AF0F7CF5  PUSH F57C0FAF                ; |Arg1 = F57C0FAF---------->◎◎◎   

0053792A  |.  8BC3         MOV EAX,EBX                  ; |   

0053792C  |.  8B0F         MOV ECX,DWORD PTR DS:[EDI]    ; |   

0053792E  |.  8B16         MOV EDX,DWORD PTR DS:[ESI]    ; |   

00537930  |.  E8 E3FDFFFF  CALL HIDOWNLO.00537718       ; HIDOWNLO.00537718   

00537935  |.  8B07         MOV EAX,DWORD PTR DS:[EDI]   

00537937  |.  50           PUSH EAX                     ; /Arg4   

00537938  |.  8B4424 30    MOV EAX,DWORD PTR SS:[ESP+30]    ; |   

0053793C  |.  50           PUSH EAX                     ; |Arg3   

0053793D  |.  6A 0C        PUSH 0C                      ; |Arg2 = 0000000C   

0053793F  |.  68 2AC68747  PUSH 4787C62A                ; |Arg1 = 4787C62A---------->◎◎◎   

00537944  |.  8BC5         MOV EAX,EBP                  ; |   

00537946  |.  8B0E         MOV ECX,DWORD PTR DS:[ESI]    ; |   

00537948  |.  8B13         MOV EDX,DWORD PTR DS:[EBX]    ; |   
```



```
0053794A  |.  E8 C9FDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

0053794F  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

00537951  |.  50            PUSH EAX                                 ; /Arg4 = AA348623

00537952  |.  8B4424 34     MOV EAX,DWORD PTR SS:[ESP+34]            ; |

00537956  |.  50            PUSH EAX                                 ; |Arg3

00537957  |.  6A 11         PUSH 11                                  ; |Arg2 = 00000011

00537959  |.  68 134630A8   PUSH A8304613                            ; |Arg1 = A8304613---------->◎◎◎

0053795E  |.  8BC7          MOV EAX,EDI                              ; |

00537960  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]               ; |

00537962  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]               ; |

00537965  |.  E8 AEFDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

0053796A  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]

0053796C  |.  50            PUSH EAX                                 ; /Arg4

0053796D  |.  8B4424 38     MOV EAX,DWORD PTR SS:[ESP+38]            ; |

00537971  |.  50            PUSH EAX                                 ; |Arg3

00537972  |.  6A 16         PUSH 16                                  ; |Arg2 = 00000016

00537974  |.  68 019546FD   PUSH FD469501                            ; |Arg1 = FD469501---------->◎◎◎

00537979  |.  8BC6          MOV EAX,ESI                              ; |

0053797B  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]               ; |

0053797E  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]               ; |

00537980  |.  E8 93FDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

00537985  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]

00537988  |.  50            PUSH EAX                                 ; /Arg4

00537989  |.  8B4424 3C     MOV EAX,DWORD PTR SS:[ESP+3C]            ; |

0053798D  |.  50            PUSH EAX                                 ; |Arg3

0053798E  |.  6A 07         PUSH 7                                   ; |Arg2 = 00000007

00537990  |.  68 D8988069   PUSH 698098D8                            ; |Arg1 = 698098D8---------->◎◎◎

00537995  |.  8BC3          MOV EAX,EBX                              ; |

00537997  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]               ; |

00537999  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]               ; |

0053799B  |.  E8 78FDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

005379A0  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]

005379A2  |.  50            PUSH EAX                                 ; /Arg4

005379A3  |.  8B4424 40     MOV EAX,DWORD PTR SS:[ESP+40]            ; |

005379A7  |.  50            PUSH EAX                                 ; |Arg3

005379A8  |.  6A 0C         PUSH 0C                                  ; |Arg2 = 0000000C

005379AA  |.  68 AFF7448B   PUSH 8B44F7AF                            ; |Arg1 = 8B44F7AF---------->◎◎◎

005379AF  |.  8BC5          MOV EAX,EBP                              ; |

005379B1  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]               ; |

005379B3  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]               ; |

005379B5  |.  E8 5EFDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

005379BA  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

005379BC  |.  50            PUSH EAX                                 ; /Arg4

005379BD  |.  8B4424 44     MOV EAX,DWORD PTR SS:[ESP+44]            ; |

005379C1  |.  50            PUSH EAX                                 ; |Arg3

005379C2  |.  6A 11         PUSH 11                                  ; |Arg2 = 00000011

005379C4  |.  68 B15BFFFF   PUSH FFFF5BB1                            ; |Arg1 = FFFF5BB1---------->◎◎◎

005379C9  |.  8BC7          MOV EAX,EDI                              ; |

005379CB  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]               ; |

005379CD  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]               ; |

005379D0  |.  E8 43FDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

005379D5  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]

005379D7  |.  50            PUSH EAX                                 ; /Arg4

005379D8  |.  8B4424 48     MOV EAX,DWORD PTR SS:[ESP+48]            ; |

005379DC  |.  50            PUSH EAX                                 ; |Arg3

005379DD  |.  6A 16         PUSH 16                                  ; |Arg2 = 00000016

005379DF  |.  68 BED75C89   PUSH 895CD7BE                            ; |Arg1 = 895CD7BE---------->◎◎◎

005379E4  |.  8BC6          MOV EAX,ESI                              ; |

005379E6  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]               ; |

005379E9  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]               ; |

005379EB  |.  E8 28FDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

005379F0  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]

005379F3  |.  50            PUSH EAX                                 ; /Arg4

005379F4  |.  8B4424 4C     MOV EAX,DWORD PTR SS:[ESP+4C]            ; |

005379F8  |.  50            PUSH EAX                                 ; |Arg3

005379F9  |.  6A 07         PUSH 7                                   ; |Arg2 = 00000007

005379FB  |.  68 2211906B   PUSH 6B901122                            ; |Arg1 = 6B901122---------->◎◎◎

00537A00  |.  8BC3          MOV EAX,EBX                              ; |

00537A02  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]               ; |

00537A04  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]               ; |

00537A06  |.  E8 0DFDFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

00537A0B  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]

00537A0D  |.  50            PUSH EAX                                 ; /Arg4

00537A0E  |.  8B4424 50     MOV EAX,DWORD PTR SS:[ESP+50]            ; |

00537A12  |.  50            PUSH EAX                                 ; |Arg3

00537A13  |.  6A 0C         PUSH 0C                                  ; |Arg2 = 0000000C

00537A15  |.  68 937198FD   PUSH FD987193                            ; |Arg1 = FD987193---------->◎◎◎

00537A1A  |.  8BC5          MOV EAX,EBP                              ; |

00537A1C  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]               ; |

00537A1E  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]               ; |

00537A20  |.  E8 F3FCFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

00537A25  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

00537A27  |.  50            PUSH EAX                                 ; /Arg4

00537A28  |.  8B4424 54     MOV EAX,DWORD PTR SS:[ESP+54]            ; |

00537A2C  |.  50            PUSH EAX                                 ; |Arg3

00537A2D  |.  6A 11         PUSH 11                                  ; |Arg2 = 00000011

00537A2F  |.  68 8E4379A6   PUSH A679438E                            ; |Arg1 = A679438E---------->◎◎◎

00537A34  |.  8BC7          MOV EAX,EDI                              ; |

00537A36  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]               ; |

00537A38  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]               ; |

00537A3B  |.  E8 D8FCFFFF   CALL HIDOWNLO.00537718                   ; HIDOWNLO.00537718

00537A40  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]

00537A42  |.  50            PUSH EAX                                 ; /Arg4

00537A43  |.  8B4424 58     MOV EAX,DWORD PTR SS:[ESP+58]            ; |

00537A47  |.  50            PUSH EAX                                 ; |Arg3

00537A48  |.  6A 16         PUSH 16                                  ; |Arg2 = 00000016
```



```
00537A4A  |.  68 2108B449   PUSH 49B40821                ; |Arg1 = 49B40821---------->◎◎◎   

00537A4F  |.  8BC6         MOV EAX,ESI                    ; |   

00537A51  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]      ; |   

00537A54  |.  8B17         MOV EDX,DWORD PTR DS:[EDI]      ; |   

00537A56  |.  E8 BDFCFFFF  CALL HIDOWNLO.00537718           ; HIDOWNLO.00537718   

00537A5B  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]************②   

00537A5E  |.  50           PUSH EAX                         ; /Arg4   

00537A5F  |.  8B4424 20    MOV EAX,DWORD PTR SS:[ESP+20]    ; |   

00537A63  |.  50           PUSH EAX                         ; |Arg3   

00537A64  |.  6A 05        PUSH 5                           ; |Arg2 = 00000005   

00537A66  |.  68 62251EF6  PUSH F61E2562                    ; |Arg1 = F61E2562---------->◎◎◎   

00537A6B  |.  8BC3         MOV EAX,EBX                      ; |   

00537A6D  |.  8B0F         MOV ECX,DWORD PTR DS:[EDI]       ; |   

00537A6F  |.  8B16         MOV EDX,DWORD PTR DS:[ESI]       ; |   

00537A71  |.  E8 D6FCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537A76  |.  8B07         MOV EAX,DWORD PTR DS:[EDI]

00537A78  |.  50           PUSH EAX                         ; /Arg4   

00537A79  |.  8B4424 34    MOV EAX,DWORD PTR SS:[ESP+34]    ; |   

00537A7D  |.  50           PUSH EAX                         ; |Arg3   

00537A7E  |.  6A 09        PUSH 9                           ; |Arg2 = 00000009   

00537A80  |.  68 40B340C0  PUSH C040B340                    ; |Arg1 = C040B340---------->◎◎◎   

00537A85  |.  8BC5         MOV EAX,EBP                      ; |   

00537A87  |.  8B0E         MOV ECX,DWORD PTR DS:[ESI]       ; |   

00537A89  |.  8B13         MOV EDX,DWORD PTR DS:[EBX]       ; |   

00537A8B  |.  E8 BCFCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537A90  |.  8B06         MOV EAX,DWORD PTR DS:[ESI]

00537A92  |.  50           PUSH EAX                         ; /Arg4   

00537A93  |.  8B4424 48    MOV EAX,DWORD PTR SS:[ESP+48]    ; |   

00537A97  |.  50           PUSH EAX                         ; |Arg3   

00537A98  |.  6A 0E        PUSH 0E                          ; |Arg2 = 0000000E   

00537A9A  |.  68 515A5E26  PUSH 265E5A51                    ; |Arg1 = 265E5A51---------->◎◎◎   

00537A9F  |.  8BC7         MOV EAX,EDI                      ; |   

00537AA1  |.  8B0B         MOV ECX,DWORD PTR DS:[EBX]       ; |   

00537AA3  |.  8B55 00      MOV EDX,DWORD PTR SS:[EBP]      ; |   

00537AA6  |.  E8 A1FCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537AAB  |.  8B03         MOV EAX,DWORD PTR DS:[EBX]

00537AAD  |.  50           PUSH EAX                         ; /Arg4   

00537AAE  |.  8B4424 1C    MOV EAX,DWORD PTR SS:[ESP+1C]    ; |   

00537AB2  |.  50           PUSH EAX                         ; |Arg3   

00537AB3  |.  6A 14        PUSH 14                          ; |Arg2 = 00000014   

00537AB5  |.  68 AAC7B6E9  PUSH E9B6C7AA                    ; |Arg1 = E9B6C7AA---------->◎◎◎   

00537ABA  |.  8BC6         MOV EAX,ESI                      ; |   

00537ABC  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]      ; |   

00537ABF  |.  8B17         MOV EDX,DWORD PTR DS:[EDI]      ; |   

00537AC1  |.  E8 86FCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537AC6  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]

00537AC9  |.  50           PUSH EAX                         ; /Arg4   

00537ACA  |.  8B4424 30    MOV EAX,DWORD PTR SS:[ESP+30]    ; |   

00537ACE  |.  50           PUSH EAX                         ; |Arg3   

00537ACF  |.  6A 05        PUSH 5                           ; |Arg2 = 00000005   

00537AD1  |.  68 5D102FD6  PUSH D62F105D                    ; |Arg1 = D62F105D---------->◎◎◎   

00537AD6  |.  8BC3         MOV EAX,EBX                      ; |   

00537AD8  |.  8B0F         MOV ECX,DWORD PTR DS:[EDI]       ; |   

00537ADA  |.  8B16         MOV EDX,DWORD PTR DS:[ESI]       ; |   

00537ADC  |.  E8 6BFCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537AE1  |.  8B07         MOV EAX,DWORD PTR DS:[EDI]

00537AE3  |.  50           PUSH EAX                         ; /Arg4   

00537AE4  |.  8B4424 44    MOV EAX,DWORD PTR SS:[ESP+44]    ; |   

00537AE8  |.  50           PUSH EAX                         ; |Arg3   

00537AE9  |.  6A 09        PUSH 9                           ; |Arg2 = 00000009   

00537AEB  |.  68 53144402  PUSH 2441453                     ; |Arg1 = 02441453---------->◎◎◎   

00537AF0  |.  8BC5         MOV EAX,EBP                      ; |   

00537AF2  |.  8B0E         MOV ECX,DWORD PTR DS:[ESI]       ; |   

00537AF4  |.  8B13         MOV EDX,DWORD PTR DS:[EBX]       ; |   

00537AF6  |.  E8 51FCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537AFB  |.  8B06         MOV EAX,DWORD PTR DS:[ESI]

00537AFD  |.  50           PUSH EAX                         ; /Arg4   

00537AFE  |.  8B4424 58    MOV EAX,DWORD PTR SS:[ESP+58]    ; |   

00537B02  |.  50           PUSH EAX                         ; |Arg3   

00537B03  |.  6A 0E        PUSH 0E                          ; |Arg2 = 0000000E   

00537B05  |.  68 81E6A1D8  PUSH D8A1E681                    ; |Arg1 = D8A1E681---------->◎◎◎   

00537B0A  |.  8BC7         MOV EAX,EDI                      ; |   

00537B0C  |.  8B0B         MOV ECX,DWORD PTR DS:[EBX]       ; |   

00537B0E  |.  8B55 00      MOV EDX,DWORD PTR SS:[EBP]      ; |   

00537B11  |.  E8 36FCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537B16  |.  8B03         MOV EAX,DWORD PTR DS:[EBX]

00537B18  |.  50           PUSH EAX                         ; /Arg4   

00537B19  |.  8B4424 2C    MOV EAX,DWORD PTR SS:[ESP+2C]    ; |   

00537B1D  |.  50           PUSH EAX                         ; |Arg3   

00537B1E  |.  6A 14        PUSH 14                          ; |Arg2 = 00000014   

00537B20  |.  68 C8FBD3E7  PUSH E7D3FBC8                    ; |Arg1 = E7D3FBC8---------->◎◎◎   

00537B25  |.  8BC6         MOV EAX,ESI                      ; |   

00537B27  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]      ; |   

00537B2A  |.  8B17         MOV EDX,DWORD PTR DS:[EDI]      ; |   

00537B2C  |.  E8 1BFCFFFF  CALL HIDOWNLO.0053774C           ; HIDOWNLO.0053774C   

00537B31  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]

00537B34  |.  50           PUSH EAX                         ; /Arg4   

00537B35  |.  8B4424 40    MOV EAX,DWORD PTR SS:[ESP+40]    ; |   

00537B39  |.  50           PUSH EAX                         ; |Arg3   

00537B3A  |.  6A 05        PUSH 5                           ; |Arg2 = 00000005   

00537B3C  |.  68 E6CDE121  PUSH 21E1CDE6                    ; |Arg1 = 21E1CDE6---------->◎◎◎   

00537B41  |.  8BC3         MOV EAX,EBX                      ; |   

00537B43  |.  8B0F         MOV ECX,DWORD PTR DS:[EDI]       ; |   

00537B45  |.  8B16         MOV EDX,DWORD PTR DS:[ESI]       ; |   
```



```
00537B47  |.  E8 00FCFFFF   CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537B4C  |.  8B07         MOV EAX,DWORD PTR DS:[EDI]   

00537B4E  |.  50           PUSH EAX                                    ; /Arg4   

00537B4F  |.  8B4424 54    MOV EAX,DWORD PTR SS:[ESP+54]             ; |   

00537B53  |.  50           PUSH EAX                                    ; |Arg3   

00537B54  |.  6A 09        PUSH 9                                      ; |Arg2 = 00000009   

00537B56  |.  68 D60737C3  PUSH C33707D6                               ; |Arg1 = C33707D6---------->◎◎◎   

00537B5B  |.  8BC5        MOV EAX,EBP                                 ; |   

00537B5D  |.  8B0E        MOV ECX,DWORD PTR DS:[ESI]                  ; |   

00537B5F  |.  8B13        MOV EDX,DWORD PTR DS:[EBX]                  ; |   

00537B61  |.  E8 E6FBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537B66  |.  8B06        MOV EAX,DWORD PTR DS:[ESI]   

00537B68  |.  50           PUSH EAX                                    ; /Arg4   

00537B69  |.  8B4424 28    MOV EAX,DWORD PTR SS:[ESP+28]             ; |   

00537B6D  |.  50           PUSH EAX                                    ; |Arg3   

00537B6E  |.  6A 0E        PUSH 0E                                     ; |Arg2 = 0000000E   

00537B70  |.  68 870DD5F4  PUSH F4D50D87                               ; |Arg1 = F4D50D87---------->◎◎◎   

00537B75  |.  8BC7        MOV EAX,EDI                                 ; |   

00537B77  |.  8B0B        MOV ECX,DWORD PTR DS:[EBX]                  ; |   

00537B79  |.  8B55 00      MOV EDX,DWORD PTR SS:[EBP]                  ; |   

00537B7C  |.  E8 CBFBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537B81  |.  8B03        MOV EAX,DWORD PTR DS:[EBX]   

00537B83  |.  50           PUSH EAX                                    ; /Arg4   

00537B84  |.  8B4424 3C    MOV EAX,DWORD PTR SS:[ESP+3C]             ; |   

00537B88  |.  50           PUSH EAX                                    ; |Arg3   

00537B89  |.  6A 14        PUSH 14                                     ; |Arg2 = 00000014   

00537B8B  |.  68 ED145A45  PUSH 455A14ED                               ; |Arg1 = 455A14ED---------->◎◎◎   

00537B90  |.  8BC6        MOV EAX,ESI                                 ; |   

00537B92  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]                  ; |   

00537B95  |.  8B17        MOV EDX,DWORD PTR DS:[EDI]                  ; |   

00537B97  |.  E8 B0FBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537B9C  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]   

00537B9F  |.  50           PUSH EAX                                    ; /Arg4   

00537BA0  |.  8B4424 50    MOV EAX,DWORD PTR SS:[ESP+50]             ; |   

00537BA4  |.  50           PUSH EAX                                    ; |Arg3   

00537BA5  |.  6A 05        PUSH 5                                      ; |Arg2 = 00000005   

00537BA7  |.  68 05E9E3A9  PUSH A9E3E905                               ; |Arg1 = A9E3E905---------->◎◎◎   

00537BAC  |.  8BC3        MOV EAX,EBX                                 ; |   

00537BAE  |.  8B0F        MOV ECX,DWORD PTR DS:[EDI]                  ; |   

00537BB0  |.  8B16        MOV EDX,DWORD PTR DS:[ESI]                  ; |   

00537BB2  |.  E8 95FBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537BB7  |.  8B07        MOV EAX,DWORD PTR DS:[EDI]   

00537BB9  |.  50           PUSH EAX                                    ; /Arg4   

00537BBA  |.  8B4424 24    MOV EAX,DWORD PTR SS:[ESP+24]             ; |   

00537BBE  |.  50           PUSH EAX                                    ; |Arg3   

00537BBF  |.  6A 09        PUSH 9                                      ; |Arg2 = 00000009   

00537BC1  |.  68 F8A3EFFC  PUSH FCEFA3F8                               ; |Arg1 = FCEFA3F8---------->◎◎◎   

00537BC6  |.  8BC5        MOV EAX,EBP                                 ; |   

00537BC8  |.  8B0E        MOV ECX,DWORD PTR DS:[ESI]                  ; |   

00537BCA  |.  8B13        MOV EDX,DWORD PTR DS:[EBX]                  ; |   

00537BCC  |.  E8 7BFBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537BD1  |.  8B06        MOV EAX,DWORD PTR DS:[ESI]   

00537BD3  |.  50           PUSH EAX                                    ; /Arg4   

00537BD4  |.  8B4424 38    MOV EAX,DWORD PTR SS:[ESP+38]             ; |   

00537BD8  |.  50           PUSH EAX                                    ; |Arg3   

00537BD9  |.  6A 0E        PUSH 0E                                     ; |Arg2 = 0000000E   

00537BDB  |.  68 D9026F67  PUSH 676F02D9                               ; |Arg1 = 676F02D9---------->◎◎◎   

00537BE0  |.  8BC7        MOV EAX,EDI                                 ; |   

00537BE2  |.  8B0B        MOV ECX,DWORD PTR DS:[EBX]                  ; |   

00537BE4  |.  8B55 00      MOV EDX,DWORD PTR SS:[EBP]                  ; |   

00537BE7  |.  E8 60FBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537BEC  |.  8B03        MOV EAX,DWORD PTR DS:[EBX]   

00537BEE  |.  50           PUSH EAX                                    ; /Arg4   

00537BEF  |.  8B4424 4C    MOV EAX,DWORD PTR SS:[ESP+4C]             ; |   

00537BF3  |.  50           PUSH EAX                                    ; |Arg3   

00537BF4  |.  6A 14        PUSH 14                                     ; |Arg2 = 00000014   

00537BF6  |.  68 8A4C2A8D  PUSH 8D2A4C8A                               ; |Arg1 = 8D2A4C8A---------->◎◎◎   

00537BFB  |.  8BC6        MOV EAX,ESI                                 ; |   

00537BFD  |.  8B4D 00      MOV ECX,DWORD PTR SS:[EBP]                  ; |   

00537C00  |.  8B17        MOV EDX,DWORD PTR DS:[EDI]                  ; |   

00537C02  |.  E8 45FBFFFF  CALL HIDOWNLO.0053774C                   ; HIDOWNLO.0053774C   

00537C07  |.  8B45 00      MOV EAX,DWORD PTR SS:[EBP]***********************************************************************************************************************************③   

00537C0A  |.  50           PUSH EAX                                    ; /Arg4   

00537C0B  |.  8B4424 30    MOV EAX,DWORD PTR SS:[ESP+30]             ; |   

00537C0F  |.  50           PUSH EAX                                    ; |Arg3   

00537C10  |.  6A 04        PUSH 4                                      ; |Arg2 = 00000004   

00537C12  |.  68 4239FAFF  PUSH FFFA3942                               ; |Arg1 = FFFA3942---------->◎◎◎   

00537C17  |.  8BC3        MOV EAX,EBX                                 ; |   

00537C19  |.  8B0F        MOV ECX,DWORD PTR DS:[EDI]                  ; |   

00537C1B  |.  8B16        MOV EDX,DWORD PTR DS:[ESI]                  ; |   

00537C1D  |.  E8 5EFBFFFF  CALL HIDOWNLO.00537780                   ; HIDOWNLO.00537780   

00537C22  |.  8B07        MOV EAX,DWORD PTR DS:[EDI]   

00537C24  |.  50           PUSH EAX                                    ; /Arg4   

00537C25  |.  8B4424 3C    MOV EAX,DWORD PTR SS:[ESP+3C]             ; |   

00537C29  |.  50           PUSH EAX                                    ; |Arg3   

00537C2A  |.  6A 0B        PUSH 0B                                     ; |Arg2 = 0000000B   

00537C2C  |.  68 81F67187  PUSH 8771F681                               ; |Arg1 = 8771F681---------->◎◎◎   

00537C31  |.  8BC5        MOV EAX,EBP                                 ; |   

00537C33  |.  8B0E        MOV ECX,DWORD PTR DS:[ESI]                  ; |   

00537C35  |.  8B13        MOV EDX,DWORD PTR DS:[EBX]                  ; |   

00537C37  |.  E8 44FBFFFF  CALL HIDOWNLO.00537780                   ; HIDOWNLO.00537780   

00537C3C  |.  8B06        MOV EAX,DWORD PTR DS:[ESI]   

00537C3E  |.  50           PUSH EAX                                    ; /Arg4   

00537C3F  |.  8B4424 48    MOV EAX,DWORD PTR SS:[ESP+48]             ; |   

00537C43  |.  50           PUSH EAX                                    ; |Arg3
```



```
00537C44  |. 6A 10              PUSH 10                              ; |Arg2 = 00000010
00537C46  |. 68 22619D6D        PUSH 6D9D6122                        ; |Arg1 = 6D9D6122---------->◎◎◎
00537C4B  |. 8BC7               MOV EAX,EDI                          ; |
00537C4D  |. 8B0B               MOV ECX,DWORD PTR DS:[EBX]           ; |
00537C4F  |. 8B55 00            MOV EDX,DWORD PTR SS:[EBP]           ; |
00537C52  |. E8 29FBFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537C57  |. 8B03               MOV EAX,DWORD PTR DS:[EBX]
00537C59  |. 50                 PUSH EAX                             ; /Arg4
00537C5A  |. 8B4424 54          MOV EAX,DWORD PTR SS:[ESP+54]        ; |
00537C5E  |. 50                 PUSH EAX                             ; |Arg3
00537C5F  |. 6A 17              PUSH 17                              ; |Arg2 = 00000017
00537C61  |. 68 0C38E5FD        PUSH FDE5380C                        ; |Arg1 = FDE5380C---------->◎◎◎
00537C66  |. 8BC6               MOV EAX,ESI                          ; |
00537C68  |. 8B4D 00            MOV ECX,DWORD PTR SS:[EBP]           ; |
00537C6B  |. 8B17               MOV EDX,DWORD PTR DS:[EDI]           ; |
00537C6D  |. E8 0EFBFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537C72  |. 8B45 00            MOV EAX,DWORD PTR SS:[EBP]
00537C75  |. 50                 PUSH EAX                             ; /Arg4
00537C76  |. 8B4424 20          MOV EAX,DWORD PTR SS:[ESP+20]        ; |
00537C7A  |. 50                 PUSH EAX                             ; |Arg3
00537C7B  |. 6A 04              PUSH 4                               ; |Arg2 = 00000004
00537C7D  |. 68 44EABEA4        PUSH A4BEEA44                        ; |Arg1 = A4BEEA44---------->◎◎◎
00537C82  |. 8BC3               MOV EAX,EBX                          ; |
00537C84  |. 8B0F               MOV ECX,DWORD PTR DS:[EDI]           ; |
00537C86  |. 8B16               MOV EDX,DWORD PTR DS:[ESI]           ; |
00537C88  |. E8 F3FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537C8D  |. 8B07               MOV EAX,DWORD PTR DS:[EDI]
00537C8F  |. 50                 PUSH EAX                             ; /Arg4
00537C90  |. 8B4424 2C          MOV EAX,DWORD PTR SS:[ESP+2C]        ; |
00537C94  |. 50                 PUSH EAX                             ; |Arg3
00537C95  |. 6A 0B              PUSH 0B                              ; |Arg2 = 0000000B
00537C97  |. 68 A9CFDE4B        PUSH 4BDECFA9                        ; |Arg1 = 4BDECFA9---------->◎◎◎
00537C9C  |. 8BC5               MOV EAX,EBP                          ; |
00537C9E  |. 8B0E               MOV ECX,DWORD PTR DS:[ESI]           ; |
00537CA0  |. 8B13               MOV EDX,DWORD PTR DS:[EBX]           ; |
00537CA2  |. E8 D9FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537CA7  |. 8B06               MOV EAX,DWORD PTR DS:[ESI]
00537CA9  |. 50                 PUSH EAX                             ; /Arg4
00537CAA  |. 8B4424 38          MOV EAX,DWORD PTR SS:[ESP+38]        ; |
00537CAE  |. 50                 PUSH EAX                             ; |Arg3
00537CAF  |. 6A 10              PUSH 10                              ; |Arg2 = 00000010
00537CB1  |. 68 604BBBF6        PUSH F6BB4B60                        ; |Arg1 = F6BB4B60---------->◎◎◎
00537CB6  |. 8BC7               MOV EAX,EDI                          ; |
00537CB8  |. 8B0B               MOV ECX,DWORD PTR DS:[EBX]           ; |
00537CBA  |. 8B55 00            MOV EDX,DWORD PTR SS:[EBP]           ; |
00537CBD  |. E8 BEFAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537CC2  |. 8B03               MOV EAX,DWORD PTR DS:[EBX]
00537CC4  |. 50                 PUSH EAX                             ; /Arg4
00537CC5  |. 8B4424 44          MOV EAX,DWORD PTR SS:[ESP+44]        ; |
00537CC9  |. 50                 PUSH EAX                             ; |Arg3
00537CCA  |. 6A 17              PUSH 17                              ; |Arg2 = 00000017
00537CCC  |. 68 70BCBFBE        PUSH BEBFBC70                        ; |Arg1 = BEBFBC70---------->◎◎◎
00537CD1  |. 8BC6               MOV EAX,ESI                          ; |
00537CD3  |. 8B4D 00            MOV ECX,DWORD PTR SS:[EBP]           ; |
00537CD6  |. 8B17               MOV EDX,DWORD PTR DS:[EDI]           ; |
00537CD8  |. E8 A3FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537CDD  |. 8B45 00            MOV EAX,DWORD PTR SS:[EBP]
00537CE0  |. 50                 PUSH EAX                             ; /Arg4
00537CE1  |. 8B4424 50          MOV EAX,DWORD PTR SS:[ESP+50]        ; |
00537CE5  |. 50                 PUSH EAX                             ; |Arg3
00537CE6  |. 6A 04              PUSH 4                               ; |Arg2 = 00000004
00537CE8  |. 68 C67E9B28        PUSH 289B7EC6                        ; |Arg1 = 289B7EC6---------->◎◎◎
00537CED  |. 8BC3               MOV EAX,EBX                          ; |
00537CEF  |. 8B0F               MOV ECX,DWORD PTR DS:[EDI]           ; |
00537CF1  |. 8B16               MOV EDX,DWORD PTR DS:[ESI]           ; |
00537CF3  |. E8 88FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537CF8  |. 8B07               MOV EAX,DWORD PTR DS:[EDI]
00537CFA  |. 50                 PUSH EAX                             ; /Arg4
00537CFB  |. 8B4424 1C          MOV EAX,DWORD PTR SS:[ESP+1C]        ; |
00537CFF  |. 50                 PUSH EAX                             ; |Arg3
00537D00  |. 6A 0B              PUSH 0B                              ; |Arg2 = 0000000B
00537D02  |. 68 FA27A1EA        PUSH EAA127FA                        ; |Arg1 = EAA127FA---------->◎◎◎
00537D07  |. 8BC5               MOV EAX,EBP                          ; |
00537D09  |. 8B0E               MOV ECX,DWORD PTR DS:[ESI]           ; |
00537D0B  |. 8B13               MOV EDX,DWORD PTR DS:[EBX]           ; |
00537D0D  |. E8 6EFAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537D12  |. 8B06               MOV EAX,DWORD PTR DS:[ESI]
00537D14  |. 50                 PUSH EAX                             ; /Arg4
00537D15  |. 8B4424 28          MOV EAX,DWORD PTR SS:[ESP+28]        ; |
00537D19  |. 50                 PUSH EAX                             ; |Arg3
00537D1A  |. 6A 10              PUSH 10                              ; |Arg2 = 00000010
00537D1C  |. 68 8530EFD4        PUSH D4EF3085                        ; |Arg1 = D4EF3085---------->◎◎◎
00537D21  |. 8BC7               MOV EAX,EDI                          ; |
00537D23  |. 8B0B               MOV ECX,DWORD PTR DS:[EBX]           ; |
00537D25  |. 8B55 00            MOV EDX,DWORD PTR SS:[EBP]           ; |
00537D28  |. E8 53FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
00537D2D  |. 8B03               MOV EAX,DWORD PTR DS:[EBX]
00537D2F  |. 50                 PUSH EAX                             ; /Arg4
00537D30  |. 8B4424 34          MOV EAX,DWORD PTR SS:[ESP+34]        ; |
00537D34  |. 50                 PUSH EAX                             ; |Arg3
00537D35  |. 6A 17              PUSH 17                              ; |Arg2 = 00000017
00537D37  |. 68 051D8804        PUSH 4881D05                         ; |Arg1 = 04881D05---------->◎◎◎
00537D3C  |. 8BC6               MOV EAX,ESI                          ; |
00537D3E  |. 8B4D 00            MOV ECX,DWORD PTR SS:[EBP]           ; |
00537D41  |. 8B17               MOV EDX,DWORD PTR DS:[EDI]           ; |
00537D43  |. E8 38FAFFFF        CALL HIDOWNLO.00537780               ; HIDOWNLO.00537780
```



```
00537D48  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]   

00537D4B  |.  50            PUSH EAX                         ; /Arg4   

00537D4C  |.  8B4424 40     MOV EAX,DWORD PTR SS:[ESP+40]     ; |   

00537D50  |.  50            PUSH EAX                         ; |Arg3   

00537D51  |.  6A 04         PUSH 4                          ; |Arg2 = 00000004   

00537D53  |.  68 39D0D4D9   PUSH D9D4D039                    ; |Arg1 = D9D4D039---------->◎◎◎   

00537D58  |.  8BC3          MOV EAX,EBX                       ; |   

00537D5A  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]        ; |   

00537D5C  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]        ; |   

00537D5E  |.  E8 1DFAFFFF   CALL HIDOWNLO.00537780            ; HIDOWNLO.00537780   

00537D63  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]   

00537D65  |.  50            PUSH EAX                         ; /Arg4   

00537D66  |.  8B4424 4C     MOV EAX,DWORD PTR SS:[ESP+4C]     ; |   

00537D6A  |.  50            PUSH EAX                         ; |Arg3   

00537D6B  |.  6A 0B         PUSH 0B                          ; |Arg2 = 0000000B   

00537D6D  |.  68 E599DBE6   PUSH E6DB99E5                    ; |Arg1 = E6DB99E5---------->◎◎◎   

00537D72  |.  8BC5          MOV EAX,EBP                       ; |   

00537D74  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]        ; |   

00537D76  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]        ; |   

00537D78  |.  E8 03FAFFFF   CALL HIDOWNLO.00537780            ; HIDOWNLO.00537780   

00537D7D  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]   

00537D7F  |.  50            PUSH EAX                         ; /Arg4   

00537D80  |.  8B4424 58     MOV EAX,DWORD PTR SS:[ESP+58]     ; |   

00537D84  |.  50            PUSH EAX                         ; |Arg3   

00537D85  |.  6A 10         PUSH 10                          ; |Arg2 = 00000010   

00537D87  |.  68 F87CA21F   PUSH 1FA27CF8                    ; |Arg1 = 1FA27CF8---------->◎◎◎   

00537D8C  |.  8BC7          MOV EAX,EDI                       ; |   

00537D8E  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]        ; |   

00537D90  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]        ; |   

00537D93  |.  E8 E8F9FFFF   CALL HIDOWNLO.00537780            ; HIDOWNLO.00537780   

00537D98  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]   

00537D9A  |.  50            PUSH EAX                         ; /Arg4   

00537D9B  |.  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+24]     ; |   

00537D9F  |.  50            PUSH EAX                         ; |Arg3   

00537DA0  |.  6A 17         PUSH 17                          ; |Arg2 = 00000017   

00537DA2  |.  68 6556ACC4   PUSH C4AC5665                    ; |Arg1 = C4AC5665---------->◎◎◎   

00537DA7  |.  8BC6          MOV EAX,ESI                       ; |   

00537DA9  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]        ; |   

00537DAC  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]        ; |   

00537DAE  |.  E8 CDF9FFFF   CALL HIDOWNLO.00537780            ; HIDOWNLO.00537780   

00537DB3  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]************④   

00537DB6  |.  50            PUSH EAX                         ; /Arg4   

00537DB7  |.  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+1C]     ; |   

00537DBB  |.  50            PUSH EAX                         ; |Arg3   

00537DBC  |.  6A 06         PUSH 6                           ; |Arg2 = 00000006   

00537DBE  |.  68 442229F4   PUSH F4292244                    ; |Arg1 = F4292244---------->◎◎◎   

00537DC3  |.  8BC3          MOV EAX,EBX                       ; |   

00537DC5  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]        ; |   

00537DC7  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]        ; |   

00537DC9  |.  E8 E6F9FFFF   CALL HIDOWNLO.005377B4            ; HIDOWNLO.005377B4   

00537DCE  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]   

00537DD0  |.  50            PUSH EAX                         ; /Arg4   

00537DD1  |.  8B4424 38     MOV EAX,DWORD PTR SS:[ESP+38]     ; |   

00537DD5  |.  50            PUSH EAX                         ; |Arg3   

00537DD6  |.  6A 0A         PUSH 0A                          ; |Arg2 = 0000000A   

00537DD8  |.  68 97FF2A43   PUSH 432AFF97                    ; |Arg1 = 432AFF97---------->◎◎◎   

00537DDD  |.  8BC5          MOV EAX,EBP                       ; |   

00537DDF  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]        ; |   

00537DE1  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]        ; |   

00537DE3  |.  E8 CCF9FFFF   CALL HIDOWNLO.005377B4            ; HIDOWN   

00537DE8  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]   

00537DEA  |.  50            PUSH EAX                         ; /Arg4   

00537DEB  |.  8B4424 54     MOV EAX,DWORD PTR SS:[ESP+54]     ; |   

00537DEF  |.  50            PUSH EAX                         ; |Arg3   

00537DF0  |.  6A 0F         PUSH 0F                          ; |Arg2 = 0000000F   

00537DF2  |.  68 A72394AB   PUSH AB9423A7                    ; |Arg1 = AB9423A7---------->◎◎◎   

00537DF7  |.  8BC7          MOV EAX,EDI                       ; |   

00537DF9  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]        ; |   

00537DFB  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]        ; |   

00537DFE  |.  E8 B1F9FFFF   CALL HIDOWNLO.005377B4            ; HIDOWNLO.005377B4   

00537E03  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]   

00537E05  |.  50            PUSH EAX                         ; /Arg4   

00537E06  |.  8B4424 30     MOV EAX,DWORD PTR SS:[ESP+30]     ; |   

00537E0A  |.  50            PUSH EAX                         ; |Arg3   

00537E0B  |.  6A 15         PUSH 15                          ; |Arg2 = 00000015   

00537E0D  |.  68 39A093FC   PUSH FC93A039                    ; |Arg1 = FC93A039---------->◎◎◎   

00537E12  |.  8BC6          MOV EAX,ESI                       ; |   

00537E14  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]        ; |   

00537E17  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]        ; |   

00537E19  |.  E8 96F9FFFF   CALL HIDOWNLO.005377B4            ; HIDOWNLO.005377B4   

00537E1E  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]   

00537E21  |.  50            PUSH EAX                         ; /Arg4   

00537E22  |.  8B4424 4C     MOV EAX,DWORD PTR SS:[ESP+4C]     ; |   

00537E26  |.  50            PUSH EAX                         ; |Arg3   

00537E27  |.  6A 06         PUSH 6                           ; |Arg2 = 00000006   

00537E29  |.  68 C3595B65   PUSH 655B59C3                    ; |Arg1 = 655B59C3---------->◎◎◎   

00537E2E  |.  8BC3          MOV EAX,EBX                       ; |   

00537E30  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]        ; |   

00537E32  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]        ; |   

00537E34  |.  E8 7BF9FFFF   CALL HIDOWNLO.005377B4            ; HIDOWNLO.005377B4   

00537E39  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]   

00537E3B  |.  50            PUSH EAX                         ; /Arg4   

00537E3C  |.  8B4424 28     MOV EAX,DWORD PTR SS:[ESP+28]     ; |   

00537E40  |.  50            PUSH EAX                         ; |Arg3   

00537E41  |.  6A 0A         PUSH 0A                          ; |Arg2 = 0000000A   
```



```asm
00537E43  |.  68 92CC0C8F   PUSH 8F0CCC92                          ; |Arg1 = 8F0CCC92---------->◎◎◎

00537E48  |.  8BC5          MOV EAX,EBP                             ; |

00537E4A  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]              ; |

00537E4C  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]              ; |

00537E4E  |.  E8 61F9FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537E53  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

00537E55  |.  50            PUSH EAX                                  ; /Arg4

00537E56  |.  8B4424 44     MOV EAX,DWORD PTR SS:[ESP+44]           ; |

00537E5A  |.  50            PUSH EAX                                  ; |Arg3

00537E5B  |.  6A 0F         PUSH 0F                                   ; |Arg2 = 0000000F

00537E5D  |.  68 7DF4EFFF   PUSH FFEFF47D                             ; |Arg1 = FFEFF47D---------->◎◎◎

00537E62  |.  8BC7          MOV EAX,EDI                               ; |

00537E64  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]              ; |

00537E66  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]              ; |

00537E69  |.  E8 46F9FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537E6E  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]

00537E70  |.  50            PUSH EAX                                  ; /Arg4

00537E71  |.  8B4424 20     MOV EAX,DWORD PTR SS:[ESP+20]           ; |

00537E75  |.  50            PUSH EAX                                  ; |Arg3

00537E76  |.  6A 15         PUSH 15                                   ; |Arg2 = 00000015

00537E78  |.  68 D15D8485   PUSH 85845DD1                             ; |Arg1 = 85845DD1---------->◎◎◎

00537E7D  |.  8BC6          MOV EAX,ESI                               ; |

00537E7F  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]              ; |

00537E82  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]              ; |

00537E84  |.  E8 2BF9FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537E89  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]

00537E8C  |.  50            PUSH EAX                                  ; /Arg4

00537E8D  |.  8B4424 3C     MOV EAX,DWORD PTR SS:[ESP+3C]           ; |

00537E91  |.  50            PUSH EAX                                  ; |Arg3

00537E92  |.  6A 06         PUSH 6                                    ; |Arg2 = 00000006

00537E94  |.  68 4F7EA86F   PUSH 6FA87E4F                             ; |Arg1 = 6FA87E4F---------->◎◎◎

00537E99  |.  8BC3          MOV EAX,EBX                               ; |

00537E9B  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]              ; |

00537E9D  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]              ; |

00537E9F  |.  E8 10F9FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537EA4  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]

00537EA6  |.  50            PUSH EAX                                  ; /Arg4

00537EA7  |.  8B4424 58     MOV EAX,DWORD PTR SS:[ESP+58]           ; |

00537EAB  |.  50            PUSH EAX                                  ; |Arg3

00537EAC  |.  6A 0A         PUSH 0A                                   ; |Arg2 = 0000000A

00537EAE  |.  68 E0E62CFE   PUSH FE2CE6E0                             ; |Arg1 = FE2CE6E0---------->◎◎◎

00537EB3  |.  8BC5          MOV EAX,EBP                               ; |

00537EB5  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]              ; |

00537EB7  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]              ; |

00537EB9  |.  E8 F6F8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537EBE  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

00537EC0  |.  50            PUSH EAX                                  ; /Arg4

00537EC1  |.  8B4424 34     MOV EAX,DWORD PTR SS:[ESP+34]           ; |

00537EC5  |.  50            PUSH EAX                                  ; |Arg3

00537EC6  |.  6A 0F         PUSH 0F                                   ; |Arg2 = 0000000F

00537EC8  |.  68 144301A3   PUSH A3014314                             ; |Arg1 = A3014314---------->◎◎◎

00537ECD  |.  8BC7          MOV EAX,EDI                               ; |

00537ECF  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]              ; |

00537ED1  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]              ; |

00537ED4  |.  E8 DBF8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537ED9  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]

00537EDB  |.  50            PUSH EAX                                  ; /Arg4

00537EDC  |.  8B4424 50     MOV EAX,DWORD PTR SS:[ESP+50]           ; |

00537EE0  |.  50            PUSH EAX                                  ; |Arg3

00537EE1  |.  6A 15         PUSH 15                                   ; |Arg2 = 00000015

00537EE3  |.  68 A111084E   PUSH 4E0811A1                             ; |Arg1 = 4E0811A1---------->◎◎◎

00537EE8  |.  8BC6          MOV EAX,ESI                               ; |

00537EEA  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]              ; |

00537EED  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]              ; |

00537EEF  |.  E8 C0F8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537EF4  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]

00537EF7  |.  50            PUSH EAX                                  ; /Arg4

00537EF8  |.  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+2C]           ; |

00537EFC  |.  50            PUSH EAX                                  ; |Arg3

00537EFD  |.  6A 06         PUSH 6                                    ; |Arg2 = 00000006

00537EFF  |.  68 827E53F7   PUSH F7537E82                             ; |Arg1 = F7537E82---------->◎◎◎

00537F04  |.  8BC3          MOV EAX,EBX                               ; |

00537F06  |.  8B0F          MOV ECX,DWORD PTR DS:[EDI]              ; |

00537F08  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]              ; |

00537F0A  |.  E8 A5F8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537F0F  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]

00537F11  |.  50            PUSH EAX                                  ; /Arg4

00537F12  |.  8B4424 48     MOV EAX,DWORD PTR SS:[ESP+48]           ; |

00537F16  |.  50            PUSH EAX                                  ; |Arg3

00537F17  |.  6A 0A         PUSH 0A                                   ; |Arg2 = 0000000A

00537F19  |.  68 35F23ABD   PUSH BD3AF235                             ; |Arg1 = BD3AF235---------->◎◎◎

00537F1E  |.  8BC5          MOV EAX,EBP                               ; |

00537F20  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]              ; |

00537F22  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]              ; |

00537F24  |.  E8 8BF8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537F29  |.  8B06          MOV EAX,DWORD PTR DS:[ESI]

00537F2B  |.  50            PUSH EAX                                  ; /Arg4

00537F2C  |.  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+24]           ; |

00537F30  |.  50            PUSH EAX                                  ; |Arg3

00537F31  |.  6A 0F         PUSH 0F                                   ; |Arg2 = 0000000F

00537F33  |.  68 BBD2D72A   PUSH 2AD7D2BB                             ; |Arg1 = 2AD7D2BB---------->◎◎◎

00537F38  |.  8BC7          MOV EAX,EDI                               ; |

00537F3A  |.  8B0B          MOV ECX,DWORD PTR DS:[EBX]              ; |

00537F3C  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]              ; |

00537F3F  |.  E8 70F8FFFF   CALL HIDOWNLO.005377B4                 ; HIDOWNLO.005377B4

00537F44  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]
```



```
00537F46  |.  50            PUSH EAX                              ; /Arg4   
00537F47  |.  8B4424 40     MOV EAX,DWORD PTR SS:[ESP+40]        ; |   
00537F4B  |.  50            PUSH EAX                              ; |Arg3   
00537F4C  |.  6A 15         PUSH 15                               ; |Arg2 = 00000015   
00537F4E  |.  68 91D386EB   PUSH EB86D391                         ; |Arg1 = EB86D391---------->◎◎◎   
00537F53  |.  8BC6          MOV EAX,ESI                           ; |   
00537F55  |.  8B4D 00       MOV ECX,DWORD PTR SS:[EBP]           ; |   
00537F58  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]           ; |   
00537F5A  |.  E8 55F8FFFF   CALL HIDOWNLO.005377B4                ; HIDOWNLO.005377B4   
00537F5F  |.  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]   
00537F63  |.  8B13          MOV EDX,DWORD PTR DS:[EBX]   
00537F65  |.  0110          ADD DWORD PTR DS:[EAX],EDX   
00537F67  |.  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]   
00537F6B  |.  8B16          MOV EDX,DWORD PTR DS:[ESI]   
00537F6D  |.  0150 04       ADD DWORD PTR DS:[EAX+4],EDX   
00537F70  |.  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]   
00537F74  |.  8B17          MOV EDX,DWORD PTR DS:[EDI]   
00537F76  |.  0150 08       ADD DWORD PTR DS:[EAX+8],EDX   
00537F79  |.  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]   
00537F7D  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]   
00537F80  |.  0150 0C       ADD DWORD PTR DS:[EAX+C],EDX   
00537F83  |.  83C4 58       ADD ESP,58   
00537F86  |.  5D            POP EBP   
00537F87  |.  5F            POP EDI   
00537F88  |.  5E            POP ESI   
00537F89  |.  5B            POP EBX   
00537F8A  .  C3            RETN   
```

最后这一段一共用了 4 个 `CALL`，每个 `CALL` 分别调用了 16 次，各个 `CALL` 的各次调用的关键参数我用了 `◎◎◎` 表示，但是我的水平太低了，算法一点不会了，哪位大侠给讲讲这是什么算法吧

使用一个比较偷懒的办法：

首先输入用户名和电子邮件地址，和随便一个序列号，比如：

```
/用 户 名:dnpf
|电子邮件:dnpf@sohu.com
注 册 码:695ec0ee7352c033530309d0ad3418cb
```

然后在 `0068A51A` 地址处下中断，拦截到之后，分别看 `EAX` 和 `EDX` 的内容：

```
/EAX:b0351f56a39e0f21d9736d09278cd3c2
EDX:e7352c033530309d0ad3418cb
```

然后按照上面我的分析，关键运算只和输入的注册码的前面某部分有关，那我们就把它组合一下，看看 `EDX` 的内容和我们输入的注册码的差别了吧，我们就取前面的部分“`695ec0e`”，然后我们知道 `EAX` 的内容就是根据这个部分生成的，并且最后还要和 `EDX` 的内容比较，我们就把 `EAX` 的内容作为我们注册码的后面部分“`b0351f56a39e0f21d9736d09278cd3c2`”连接起来就是“`695ec0eb0351f56a39e0f21d9736d09278cd3c2`”

最后整理一下：

```
/用 户 名:dnpf-------------------
|电子邮件:dnpf@sohu.com----------/这两部分不参与运算，可以任意输入
注 册 码:695ec0eb0351f56a39e0f21d9736d09278cd3c2
```

怎么样，注册码应该有好多吧…… `4dd466d0e1e0cc123afbe504da2462ca`
