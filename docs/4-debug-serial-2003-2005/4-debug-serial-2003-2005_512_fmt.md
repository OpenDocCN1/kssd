
# 网络警察 V2.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6320.htm`](https://www.pediy.com/kssd/pediy06/pediy6320.htm)

《网络警察》`V2.4`

**用途**：不良站点过滤、弹出广告屏蔽、`IE` 浏览器修复和操作系统加解锁工具

**软件功能特色如下**：

◆ 可以使含有关键字的网址自动转向到你设定的网站或 `HTML` 文件；
◆ 清空 `IE` 历史记录、临时文件，文档清空，删除收藏夹和 `IE` 地址栏网址；
◆ 限制浏览器窗口数量，自动杀死讨厌的各类弹出广告窗口；
◆ 修复浏览器首页、标题等被篡改，解除按钮变灰、选项消失等十余类症状；
◆ 对电脑操作系统进行限制如隐藏硬盘、禁止打开控制面板等 20 余类控制；
◆ 提供反动、色情、非法站点数据库并定期更新，让您上网浏览高枕无忧；
◆ 真正绿色软件，无需安装即可使用，操作简便，界面美观。

**主页**：`http://www.seekeasysoft.com`

**镜像**：
`http://seekeasy.yeah.net`
`http://seekeasy.cnnb.net`
`http://seekeasy.126.com`

`Delphi6`，无壳

反汇编，查找字符串参考"软件注册号错误"：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B202F(C)
|
:004B20A8 55                   push ebp
:004B20A9 8BEC                 mov ebp, esp
:004B20AB 83C4E0               add esp, FFFFFE0
:004B20AE 53                   push ebx
:004B20AF 33C9                 xor ecx, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B204A(C)
|
:004B20B1 894DE0               mov dword ptr [ebp-20], ecx
:004B20B4 894DF4               mov dword ptr [ebp-0C], ecx
:004B20B7 8BD8                 mov ebx, eax
:004B20B9 33C0                 xor eax, eax
:004B20BB 55                   push ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B2048(C)
|
:004B20BC 68BA214B00           push 004B21BA
:004B20C1 64FF30               push dword ptr fs:[eax]
:004B20C4 648920               mov dword ptr fs:[eax], esp
:004B20C7 8D45F4               lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"00000000"
                               |
:004B20CA BAD0214B00           mov edx, 004B21D0
:004B20CF E82825F5FF           call 004045FC
:004B20D4 8D4DF4               lea ecx, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"请输入您的软件注册码"
                               |
:004B20D7 BAE4214B00           mov edx, 004B21E4

* Possible StringData Ref from Code Obj ->"登记注册"
                               |
:004B20DC B804224B00           mov eax, 004B2204
:004B20E1 E84EB3F8FF           call 0043D434----------------------->出注册框
:004B20E6 3C01                 cmp al, 01-------------------------->是否点击"确定"
:004B20E8 0F85AE000000         jne 004B219C------------------------>不是点击"确定"则跳走
:004B20EE 8D55E0               lea edx, dword ptr [ebp-20]--------->新地址
:004B20F1 8B45F4               mov eax, dword ptr [ebp-0C]--------->输入的注册码
:004B20F4 E80369F5FF           call 004089FC----------------------->保存于新地址
:004B20F9 8B45E0               mov eax, dword ptr [ebp-20]--------->输入的注册码
:004B20FC E8CF6BF5FF           call 00408CD0----------------------->输入的注册码转换为 16 进制
:004B2101 8945F8               mov dword ptr [ebp-08], eax\--------->转换为 16 进制的注册码
:004B2104 8955FC               mov dword ptr [ebp-04], edx\--------->高位
:004B2107 6A00                 push 00000000
:004B2109 6A44                 push 00000044
:004B210B 8B45F8               mov eax, dword ptr [ebp-08]------->输入注册码的 16 进制
:004B210E 8B55FC               mov edx, dword ptr [ebp-04]------->高位------->这里要注意,如果高位为零,在运算一中会出错
:004B2111 E83E34F5FF           call 00405554--------------------->******关键运算一******
:004B2116 8945F8               mov dword ptr [ebp-08], eax\------->保存运算一的结果
:004B2119 8955FC               mov dword ptr [ebp-04], edx\------->保存高位
:004B211C 8B45F8               mov eax, dword ptr [ebp-08]
:004B211F 8B55FC               mov edx, dword ptr [ebp-04]
:004B2122 2DDA070900           sub eax, 000907DA----------------->减法（关键数字）◎◎◎◎◎
:004B2127 83DA00               sbb edx, 00000000----------------->借位
:004B212A 8945F8               mov dword ptr [ebp-08], eax\------->保存回原地址
:004B212D 8955FC               mov dword ptr [ebp-04], edx\------->高位保存回去
:004B2130 8D45E4               lea eax, dword ptr [ebp-1C]------->新地址
:004B2133 E8B075FFFF           call 004A96E8---------------------->******关键运算二******(CPUID)
:004B2138 8B4DE4               mov ecx, dword ptr [ebp-1C]-------->关键运算二的结果(与硬件相关)
:004B213B 8BC1                 mov eax, ecx\---------------------->运算二的结果到 EAX
:004B213D 99                   cdq\-------------------------------->EAX 高位扩展
:004B213E 3B55FC               cmp edx, dword ptr [ebp-04]-------->与关键运算一的高位结果比较
:004B2141 7544                 jne 004B2187----------------------->不相等则跳到错误
:004B2143 3B45F8               cmp eax, dword ptr [ebp-08]-------->关键运算二的结果与运算一的低位结果比较
:004B2146 753F                 jne 004B2187----------------------->不相等则跳到错误
:004B2148 8B8320050000         mov eax, dword ptr [ebx+00000520]
:004B214E E85D5FFFFF           call 004A80B0
:004B2153 33D2                 xor edx, edx
:004B2155 8B83D8040000         mov eax, dword ptr [ebx+000004D8]
:004B215B 8B08                 mov ecx, dword ptr [eax]
:004B215D FF5164               call [ecx+64]

* Possible StringData Ref from Code Obj ->"已注册登记版本"
                               |
:004B2160 BA18224B00           mov edx, 004B2218
:004B2165 8B8324050000         mov eax, dword ptr [ebx+00000524]
:004B216B E8241EF9FF           call 00443F94
:004B2170 6A00                 push 00000000
:004B2172 668B0D28224B00       mov cx, word ptr [004B2228]
:004B2179 B202                 mov dl, 02

* Possible StringData Ref from Code Obj ->"软件登记注册成功"
                               |
:004B217B B834224B00           mov eax, 004B2234
:004B2180 E893B1F8FF           call 0043D318
:004B2185 EB15                 jmp 004B219C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004B2141(C), :004B2146(C)
|
:004B2187 6A00                 push 00000000
:004B2189 668B0D28224B00       mov cx, word ptr [004B2228]
:004B2190 B201                 mov dl, 01

* Possible StringData Ref from Code Obj ->"软件注册号错误"
                               |
:004B2192 B850224B00           mov eax, 004B2250
:004B2197 E87CB1F8FF           call 0043D318

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004B20E8(C), :004B2185(U)
|
:004B219C 33C0                 xor eax, eax
:004B219E 5A                   pop edx
:004B219F 59                   pop ecx
:004B21A0 59                   pop ecx
:004B21A1 648910               mov dword ptr fs:[eax], edx
:004B21A4 68C1214B00           push 004B21C1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B21BF(U)
|
:004B21A9 8D45E0               lea eax, dword ptr [ebp-20]
:004B21AC E8B323F5FF           call 00404564
:004B21B1 8D45F4               lea eax, dword ptr [ebp-0C]
:004B21B4 E8AB23F5FF           call 00404564
:004B21B9 C3                   ret

****************************************************************************************************
*****关键运算一*****

* Referenced by a CALL at Addresses:
|:0040E884   , :004B2111
|
:00405554 55                   push ebp
:00405555 53                   push ebx
:00405556 56                   push esi
:00405557 57                   push edi
:00405558 31FF                 xor edi, edi
:0040555A 8B5C2414             mov ebx, dword ptr [esp+14]
```



```
:0040555E 8B4C2418                mov ecx, dword ptr [esp+18]   

:00405562 09C9                    or ecx, ecx   

:00405564 7508                    jne 0040556E   

:00405566 09D2                    or edx, edx---------------->注册码的高位是否为零(是否大于 4294967295)   

:00405568 745C                    je 004055C6---------------->小于则跳   

:0040556A 09DB                    or ebx, ebx   

:0040556C 7458                    je 004055C6   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00405564(C)   

|   

:0040556E 09D2                    or edx, edx   

:00405570 790A                    jns 0040557C   

:00405572 F7DA                    neg edx   

:00405574 F7D8                    neg eax   

:00405576 83DA00                  sbb edx, 00000000   

:00405579 83CF01                  or edi, 00000001   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00405570(C)   

|   

:0040557C 09C9                    or ecx, ecx   

:0040557E 790A                    jns 0040558A   

:00405580 F7D9                    neg ecx   

:00405582 F7DB                    neg ebx   

:00405584 83D900                  sbb ecx, 00000000   

:00405587 83F701                  xor edi, 00000001   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040557E(C)   

|   

:0040558A 89CD                    mov ebp, ecx   

:0040558C B940000000              mov ecx, 00000040   

:00405591 57                      push edi   

:00405592 31FF                    xor edi, edi   

:00405594 31F6                    xor esi, esi   

:00405596 D1E0                    shl eax, 1   

:00405598 D1D2                    rcl edx, 1   

:0040559A D1D6                    rcl esi, 1   

:0040559C D1D7                    rcl edi, 1   

:0040559E 39EF                    cmp edi, ebp   

:004055A0 720B                    jb 004055AD   

:004055A2 7704                    ja 004055A8   

:004055A4 39DE                    cmp esi, ebx   

:004055A6 7205                    jb 004055AD   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004055A2(C)   

|   

:004055A8 29DE                    sub esi, ebx   

:004055AA 19EF                    sbb edi, ebp   

:004055AC 40                      inc eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:004055A0(C), :004055A6(C)   

|   

:004055AD E2E7                    loop 00405596   

:004055AF 5B                      pop ebx   

:004055B0 F7C301000000            test ebx, 00000001   

:004055B6 7407                    je 004055BF   

:004055B8 F7DA                    neg edx   

:004055BA F7D8                    neg eax   

:004055BC 83DA00                  sbb edx, 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:004055B6(C), :004055CA(U)   

|   

:004055BF 5F                      pop edi   

:004055C0 5E                      pop esi   

:004055C1 5B                      pop ebx   

:004055C2 5D                      pop ebp   

:004055C3 C20800                  ret 0008   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00405568(C), :0040556C(C)   

|   

:004055C6 F7F3                    div ebx--------------------->EAX/EBX   

:004055C8 31D2                    xor edx, edx   

:004055CA EBF3                    jmp 004055BF   

:004055CC C3                      ret   

--------------------------------------------------------------------------------
```

### 总结

在关键二根据硬件取得`CPUID`之后，根据反向计算，得：

关键二的结果`+ 591834`（关键数字）＝关键运算一的结果（我的是`3859 + 591834 = 595693`）

最简单的形式是注册码小于`4294967295`，则运算形式为：`注册码/44h = 运算一的结果`

所以以上两处相等即可，我的注册码计算如下：

`595693 * 68 = 40507124`

注册成功！

但是注册成功以后更改自动转向的网址时已然出现要求注册的对话框，但是被过滤的网址的列表却没有问题（帮助里面说未注册版本只能有`10`个网址），不知道这个是软件的问题还是注册过程有陷阱……

`◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎`

### 其他

这个网站还有另外几个软件，我下载的还有《极速传真》`V2.7`、《搜易办公监管王》`V2.0` 他们的关键数字分别是：`0EC70h` 和 `11C0Eh` 两个，也都是使用相同的方法注册。

`4dd466d0e1e0cc123afbe504da2462ca`
