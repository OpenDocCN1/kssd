
# Assistant

> 原文：[`www.pediy.com/kssd/pediy06/pediy6058.htm`](https://www.pediy.com/kssd/pediy06/pediy6058.htm)

【下载页面】：`http://www.wbj2000.com/`

【软件大小】：364 KB

【应用平台】：`WIN9X`/`WINNT`/`WIN2K`/`WINXP`

【软件简介】： 一款很好的电脑辅助功能软件， 包含如下功能：提醒（定时、每年、每月、每日、西方节日）、透明桌面时钟、中国日历（含农历）、模拟科学计算器、`MP3` 播放器。 本软件尚未正式发布，作者正在使之逐步完善中.

【软件限制】：30 天试用期!

【作者声明】：本人发表这篇文章只是为了学习!!!请不用于商业用途或是将本文方法制作的注册机任意传播,读者看了文章后所做的事情与我无关,我也不会负责,请读者看了文章后三思而后行!最后希望大家在经济基础好的时候,支持共享软件!

【破解工具】：`TRW2K`  `w32Dasm`  `PEiD`

【过 程】：

`PEiD` 侦察出主程序 `Assistant.exe` 没有加壳!

用 `w32Dasm` 反汇编后,查找参考字串很快就找到关键!

运行主程序 `Assistant.exe`-->提示注册-->输入用户名(用户名要大于 5 位)Yock[DFCG]-->注册码 48484848

启动 `TRW2K` 动态跟踪!`ctrl+n` 呼出-->下断点 `bpx 41BD29`-->`F5` 返回-->点注册来到下面!

```
:0041BD29 FFD5                   call ebp                       \\取得用户名的长度!
:0041BD2B 8B4E14                mov ecx, dword ptr [esi+14]
:0041BD2E 53                    push ebx
:0041BD2F 6A10                  push 00000010
:0041BD31 6A0D                  push 0000000D
:0041BD33 682C040000            push 0000042C
:0041BD38 51                    push ecx
:0041BD39 FFD5                   call ebp                       \\取得注册码的长度!
:0041BD3B 8BBC2418010000        mov edi, dword ptr [esp+00000118]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041BCF9(C)`

```
:0041BD42 8D9E80010000          lea ebx, dword ptr [esi+00000180]
:0041BD48 53                    push ebx                       \\DEBX 可以看到我输入的注册码!
:0041BD49 E81F0F0000            call 0041CC6D                  \\不算太重要,我个人觉得完全是多余的!
:0041BD4E 8B150C8A4200          mov edx, dword ptr [00428A0C]
:0041BD54 8DAE80000000          lea ebp, dword ptr [esi+00000080]
:0041BD5A 53                    push ebx                       \\DEBP 可以看见我输入的用户名!
:0041BD5B 55                    push ebp
:0041BD5C 6A08                  push 00000008
:0041BD5E 52                    push edx
:0041BD5F E80CEAFFFF            call 0041A770                  \\关键 CALL,F8 追进去!!!
:0041BD64 A38C2E4300            mov dword ptr [00432E8C], eax
:0041BD69 8B4E18                mov ecx, dword ptr [esi+18]
:0041BD6C 83C414                add esp, 00000014
:0041BD6F 83F903                cmp ecx, 00000003
:0041BD72 7525                  jne 0041BD99                   \\跳向成功处!
:0041BD74 85C0                  test eax, eax
:0041BD76 7525                  jne 0041BD9D                   \\跳向成功处!不跳就死得好惨!
:0041BD78 50                    push eax
```

\* Possible StringData Ref from Data Obj ->"用户注册"

```
:0041BD79 68BC194300            push 004319BC
```

\* Possible StringData Ref from Data Obj ->"注册码错误！请重新输入！"

```
:0041BD7E 68A0194300            push 004319A0
:0041BD83 57                    push edi
```

\* Reference To: `USER32.MessageBoxA`, Ord:01BEh

```
:0041BD84 FF15FC724200          Call dword ptr [004272FC]      \\不知道这个 CALL 有什么用的话,就自己去刨书!
\------------------------------------------------------------------
```

上面 `0041BD5F` 的 `CALL` 来到这里!!这里可是关键哦!

\* Referenced by a `CALL` at Addresses:

`:0041AAA7 , :0041BD5F`

```
:0041A770 81EC00020000          sub esp, 00000200
:0041A776 B940000000            mov ecx, 00000040
:0041A77B 33C0                  xor eax, eax
:0041A77D 53                    push ebx
:0041A77E 56                    push esi
:0041A77F 57                    push edi
:0041A780 8D7C240C              lea edi, dword ptr [esp+0C]
:0041A784 F3                    repz
:0041A785 AB                    stosd
:0041A786 8B842414020000        mov eax, dword ptr [esp+00000214]\\我输入注册码的位数!
                                  \\下面是比较注册码位数!
                                  \\如果注册码等于 6、8、10、12、14、16 位
                                  \\就跳到 0041A7AB
                                  \\如果不是的话就跳到 0041A7B0
:0041A78D 83F806                cmp eax, 00000006
:0041A790 7419                  je 0041A7AB
:0041A792 83F808                cmp eax, 00000008
:0041A795 7414                  je 0041A7AB
:0041A797 83F80A                cmp eax, 0000000A
:0041A79A 740F                  je 0041A7AB
:0041A79C 83F80C                cmp eax, 0000000C
:0041A79F 740A                  je 0041A7AB
:0041A7A1 83F80E                cmp eax, 0000000E
:0041A7A4 7405                  je 0041A7AB
:0041A7A6 83F810                cmp eax, 00000010
:0041A7A9 7505                  jne 0041A7B0
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0041A790(C), :0041A795(C), :0041A79A(C), :0041A79F(C), :0041A7A4(C)`

```
:0041A7AB A3C0AF4300            mov dword ptr [0043AFC0], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041A7A9(C)`

```
:0041A7B0 8BBC2418020000        mov edi, dword ptr [esp+00000218]
:0041A7B7 83C9FF                or ecx, FFFFFFFF
:0041A7BA 33C0                  xor eax, eax
:0041A7BC 8D54240C              lea edx, dword ptr [esp+0C]
:0041A7C0 F2                    repnz
:0041A7C1 AE                    scasb
:0041A7C2 F7D1                  not ecx
:0041A7C4 2BF9                  sub edi, ecx
:0041A7C6 8BC1                  mov eax, ecx
:0041A7C8 8BF7                  mov esi, edi
:0041A7CA 8BFA                  mov edi, edx
:0041A7CC C1E902                shr ecx, 02
:0041A7CF F3                    repz
:0041A7D0 A5                    movsd
:0041A7D1 8BC8                  mov ecx, eax
:0041A7D3 8B842410020000        mov eax, dword ptr [esp+00000210]
:0041A7DA 83E103                and ecx, 00000003
:0041A7DD 85C0                  test eax, eax
:0041A7DF F3                    repz                         \\假如注册码不大于 6 位,就死在这里!但我不知道为什么!!!
:0041A7E0 A4                    movsb
:0041A7E1 7405                  je 0041A7E8
:0041A7E3 A3C8AF4300            mov dword ptr [0043AFC8], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041A7E1(C)`

```
:0041A7E8 8D4C240C              lea ecx, dword ptr [esp+0C]
:0041A7EC 51                    push ecx                     \\DECX 可以看见用户名!
:0041A7ED E8DEF9FFFF            call 0041A1D0                 \\注册码 A 是怎么样炼成的,F8 追!
:0041A7F2 68F84CBA01            push 01BA4CF8
:0041A7F7 8D942414010000        lea edx, dword ptr [esp+00000114]
```

\* Possible StringData Ref from Data Obj ->`%lu`

```
:0041A7FE 68C8194300            push 004319C8                 \\D 4319C8 可以看见一些数字!
                                                                 \\我看见的是 3397255503
:0041A803 52                    push edx
```

\* Reference To: `USER32.wsprintfA`, Ord:02ACh

```
:0041A804 FF15F8714200          Call dword ptr [004271F8]     \\这个 CALL 是生成注册码 B
:0041A80A 8BBC242C020000        mov edi, dword ptr [esp+0000022C]\\DEDI 可以看见我输入的注册码!
:0041A811 83C410                add esp, 00000010
:0041A814 8BF7                  mov esi, edi                   \\EDI=ESI
:0041A816 8D4C240C              lea ecx, dword ptr [esp+0C]     \\DECX 可以看见真注册码 A!
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041A83C(C)                   \\从这里开始到 0041A83C 是比较真假注册码每一位的 ASCII 码是否相等,不等就跳下去再比较!`

```
:0041A81A 8A01                  mov al, byte ptr [ecx]
:0041A81C 8A1E                  mov bl, byte ptr [esi]
:0041A81E 8AD0                  mov dl, al
```



```
:0041A820 3AC3                 cmp al, bl
:0041A822 751E                 jne 0041A842
:0041A824 84D2                 test dl, dl
:0041A826 7416                 je 0041A83E
:0041A828 8A4101               mov al, byte ptr [ecx+01]
:0041A82B 8A5E01               mov bl, byte ptr [esi+01]
:0041A82E 8AD0                 mov dl, al
:0041A830 3AC3                 cmp al, bl
:0041A832 750E                 jne 0041A842
:0041A834 83C102               add ecx, 00000002
:0041A837 83C602               add esi, 00000002
:0041A83A 84D2                 test dl, dl
:0041A83C 75DC                 jne 0041A81A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A826(C)
|

:0041A83E 33C9                 xor ecx, ecx
:0041A840 EB05                 jmp 0041A847

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A822(C), :0041A832(C)
|

:0041A842 1BC9                 sbb ecx, ecx
:0041A844 83D9FF               sbb ecx, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A840(U)
|

:0041A847 85C9                 test ecx, ecx
:0041A849 7446                 je 0041A891
:0041A84B 8DB4240C010000       lea esi, dword ptr [esp+0000010C]     \\DESI 可以看见真的注册码 B
:0041A852 8BC7                 mov eax, edi                          \\这里是我输入的注册码!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A876(C)                   \\从这里开始到 0041A876 是比较真假注册码每一位的 ASCII 码是否相等!
|

:0041A854 8A10                 mov dl, byte ptr [eax]
:0041A856 8A1E                 mov bl, byte ptr [esi]
:0041A858 8ACA                 mov cl, dl
:0041A85A 3AD3                 cmp dl, bl
:0041A85C 751E                 jne 0041A87C
:0041A85E 84C9                 test cl, cl
:0041A860 7416                 je 0041A878
:0041A862 8A5001               mov dl, byte ptr [eax+01]
:0041A865 8A5E01               mov bl, byte ptr [esi+01]
:0041A868 8ACA                 mov cl, dl
:0041A86A 3AD3                 cmp dl, bl
:0041A86C 750E                 jne 0041A87C
:0041A86E 83C002               add eax, 00000002
:0041A871 83C602               add esi, 00000002
:0041A874 84C9                 test cl, cl                          \\是否取完
:0041A876 75DC                 jne 0041A854                        \\没有取完就跳回去!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A860(C)
|

:0041A878 33C0                 xor eax, eax
:0041A87A EB05                 jmp 0041A881

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A85C(C), :0041A86C(C)
|

:0041A87C 1BC0                 sbb eax, eax
:0041A87E 83D8FF               sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A87A(U)
|

:0041A881 85C0                 test eax, eax
:0041A883 740C                 je 0041A891
:0041A885 5F                   pop edi
:0041A886 5E                   pop esi
:0041A887 33C0                 xor eax, eax                          \\清零
:0041A889 5B                   pop ebx
:0041A88A 81C400020000         add esp, 00000200
:0041A890 C3                   ret                                   \\返回

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041A849(C), :0041A883(C)
|

:0041A891 5F                   pop edi
:0041A892 5E                   pop esi
:0041A893 B801000000           mov eax, 00000001                   \\赋值
:0041A898 5B                   pop ebx
:0041A899 81C400020000         add esp, 00000200
:0041A89F C3                   ret                                   \\返回

------------------------------------------------------------------
上面 0041A7ED 的 CALL 来到这里!注册码的生成和比较都再这里!关键!!!!!

* Referenced by a CALL at Addresses:
|:0041A575  , :0041A7ED   

|

:0041A1D0 53                   push ebx
:0041A1D1 56                   push esi
:0041A1D2 57                   push edi
:0041A1D3 8B7C2410             mov edi, dword ptr [esp+10]    \\DEDI 看见我输入的注册码!
:0041A1D7 32DB                 xor bl, bl
:0041A1D9 8BCF                 mov ecx, edi                  \\ECX=EDI
:0041A1DB 8A07                 mov al, byte ptr [edi]        \\我输入注册码的第一位数的 ASCII 码入 EAX 低位!
:0041A1DD 84C0                 test al, al                   \\是否为空!
:0041A1DF 740A                 je 0041A1EB                   \\为空的话就跳走!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1E9(C)                  \\从这里到 0041A1E9 是把我输入注册码的 ASCII 码加再一起放进 EBX 低位!
|

:0041A1E1 02D8                 add bl, al
:0041A1E3 8A4101               mov al, byte ptr [ecx+01]
:0041A1E6 41                   inc ecx                       \\计数器
:0041A1E7 84C0                 test al, al
:0041A1E9 75F6                 jne 0041A1E1                  \\循环!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1DF(C)
|

:0041A1EB A1C8AF4300           mov eax, dword ptr [0043AFC8]
:0041A1F0 33F6                 xor esi, esi
:0041A1F2 A3D8AF4300           mov dword ptr [0043AFD8], eax
:0041A1F7 A1C0AF4300           mov eax, dword ptr [0043AFC0]
:0041A1FC 85C0                 test eax, eax
:0041A1FE 7E2D                 jle 0041A22D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A22B(C)                  \\从这里开始到 0041A22B 就把用户名的 ASCII 码循环计算出正确注册码的!
                              \\你把内存区的地址指向 6AF20C 就可以看见注册码的变化了!
|

:0041A200 8A0C3E               mov cl, byte ptr [esi+edi]
:0041A203 32CB                 xor cl, bl
:0041A205 51                   push ecx
:0041A206 E895FFFFFF           call 0041A1A0                  \\这个是运算 CALL,F8 追进去!
:0041A20B 83C404               add esp, 00000004
:0041A20E 88043E               mov byte ptr [esi+edi], al
:0041A211 3C0A                 cmp al, 0A
:0041A213 0FBEC0               movsx eax, al
:0041A216 7D05                 jge 0041A21D
:0041A218 83C030               add eax, 00000030
:0041A21B EB03                 jmp 0041A220

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A216(C)
|

:0041A21D 83C041               add eax, 00000041

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A21B(U)
|

:0041A220 88043E               mov byte ptr [esi+edi], al
:0041A223 A1C0AF4300           mov eax, dword ptr [0043AFC0]
:0041A228 46                   inc esi                         \\计数器!
:0041A229 3BF0                 cmp esi, eax                    \\比较用户名是否取完!
:0041A22B 7CD3                 jl 0041A200                     \\没有就跳回去!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A1FE(C)
|

:0041A22D C6043800             mov byte ptr [eax+edi], 00
:0041A231 5F                  pop edi
:0041A232 5E                  pop esi
:0041A233 5B                  pop ebx
:0041A234 C3                  ret

------------------------------------------------------------------
上面 0041A206 的 CALL 来到这里!

* Referenced by a CALL at Address:
|:0041A206
|

:0041A1A0 0FBE442404          movsx eax, byte ptr [esp+04]
:0041A1A5 0305D8AF4300        add eax, dword ptr [0043AFD8]
:0041A1AB 69C0697DAE42        imul eax, 42AE7D69
:0041A1B1 0531D40000          add eax, 0000D431
:0041A1B6 A3D8AF4300          mov dword ptr [0043AFD8], eax
:0041A1BB C1F810              sar eax, 10
```



```
:0041A1BE 83E00F                 and eax, 0000000F  
:0041A1C1 C3                     ret
```

`------------------------------------------------------------------`

注册码 `A` 的内存注册机:  

中断地址:`0041A81A`  

中断次数:`1`  

第一字节:`BA`  

字节长度:`2`  

注册码:内存方式 --> 寄存器 `ECX` 

注册码 `B` 的内存注册机:  

中断地址:`0041A852`  

中断次数:`1`  

第一字节:`8B`  

字节长度:`2`  

注册码:内存方式 --> 寄存器 `ESI`  

`------------------------------------------------------------------`

【总结】：

这个程序一共有两个注册码! 

分别是注册码 `A` 和注册码 `B` 

注册码 `A` 是根据用户名注册的!(注册的用户名要大于 5 位) 

注册码 `B` 是一机一码!用户名随便!(注册的用户名要大于 5 位) 

一组可以用的注册码:

用户名:`Yock[DFCG]`  

注册码:`63532K04`  

`-------------------------------------------------------------------` 

算法:(大家觉得是不是很熟悉啊?!) 

我语文水平差,表达得不要请您帮我补充!谢谢! 

把我输入注册码的 `ASCII` 码相加结果的最后两位!(假如注册码的 `ASCII` 码的和是 123,那么最后两位是 23) 

再把注册码的每一位数的 `ASCII` 码 `XOR` 后的结果就到下面了!  

(这里不知道怎么用文字表达,所以大家看汇编指令吧!这里要多谢师傅兔子,我也是看了它的文章才会这么表达的!)  

```
:0041A1A0 0FBE442404             movsx eax, byte ptr [esp+04]  
:0041A1A5 0305D8AF4300           add eax, dword ptr [0043AFD8]  
:0041A1AB 69C0697DAE42           imul eax, 42AE7D69  
:0041A1B1 0531D40000             add eax, 0000D431  
:0041A1B6 A3D8AF4300             mov dword ptr [0043AFD8], eax  
:0041A1BB C1F810                 sar eax, 10  
:0041A1BE 83E00F                 and eax, 0000000F  
:0041A1C1 C3                     ret
```

最后的结果保存在 `EDI` 

我个人觉得可以用 `keymake` 做一个算法注册机,可惜我不会!555555 

哪个大哥会的话可以做一个,做好后如果方便的话可以把怎么做的过程给我看看吗??期待... 

最后在这里真心感谢你花了那么多时间看这篇文章!谢谢了... 

 2003.05.17 凌晨于清远

`4dd466d0e1e0cc123afbe504da2462ca`
