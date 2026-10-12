
# Windows 系统切换工具

> 原文：[`www.pediy.com/kssd/pediy05/pediy50481.htm`](https://www.pediy.com/kssd/pediy05/pediy50481.htm)

《Windows 系统切换工具》包含了简体中文/繁体中文/英语三种语言包，并且在启动的过程中会自适应系统语言。相对来说《Windows 系统切换工具》的操作非常简单，用户可以通过操作向导很容易完成系统切换任务。启动《Windows 系统切换工具》，在“常规设定”中已经列出了程序已经检测出的安装的系统名称、目录、系统类型等等信息，用户可以通过右侧的按钮来进行添加（把安装了但没有检测到的系统添加到系统列表中去，注意只能添加 `Win9x`/`98`/`Me` 系统）、修改、默认等操作。“`NT`/`2000`/`Xp` 启动菜单栏”这个主要是管理 `NT`/`2000`/`Xp` 启动菜单栏的，它可以用来管理 `NT`/`2000`/`Xp` 启动菜单的默认系统和菜单停留时间。除此之外，《Windows 系统切换工具》还有设置系统和恢复 `IE` 设定的功能。

《Windows 系统切换工具》未注册版本在功能和时间上没有任何限制，只是在启动时会显示一个评估版本的对话框。

软件性质: `[免费版]`

操作系统: `WIN 9X`/`WIN NT`/`2000`/`XP`

语言界面: 简体中文

文件大小：`1176KB`

源文件用 `ASPack v2.11` 加壳，脱壳后，由原来的 `50k` 变成 `136k`。

破解工具：`TRW2000 v1.22` 娃娃修正版

程序运行后，点“注册”，要求输入，用户名、注册名和注册码，用户名和注册名都至少大于 3 个字符。（破解后得知，用户名并不参与运算，只是通过注册名算出相应的注册码，典型的 `name/serial` 型）

输入：

用户名：`xqlk`

注册名：`ricemaster`

注册码：`1122334455`

设断点：`bpx hmemcpy`，程序一共断下来 3 次，分别是在读取上述 3 个信息，3 次后，来到：

```
:00407226 E847980000             Call 00410A72
:0040722B 8BCD                  mov ecx, ebp
:0040722D E80E030000             call 00407540   ===》标准形式!!!!我喜欢!!!!!
:00407232 85C0                  test eax, eax
:00407234 0F842B020000           je 00407465      ===》不能跳，跳就 game over 了。
:0040723A 81EEB8040000           sub esi, 000004B8   ===》注册码保存到文件前减了 4B8，可能是出于安全方面的考虑吧，不然，注册名及相应的注册码就直接暴露了。
:00407240 8D4C241C               lea ecx, dword ptr [esp+1C]
:00407244 56                    push esi
```

`F8` 进去：

```
:00407540 6AFF                  push FFFFFFFF
:00407542 68581D4100             push 00411D58
:00407547 64A100000000           mov eax, dword ptr fs:[00000000]
:0040754D 50                    push eax
:0040754E 64892500000000         mov dword ptr fs:[00000000], esp
:00407555 83EC10                sub esp, 00000010
:00407558 53                    push ebx
:00407559 55                    push ebp
:0040755A 56                    push esi
:0040755B 57                    push edi
:0040755C 8BF9                  mov edi, ecx
:0040755E 51                    push ecx
:0040755F 8D442434               lea eax, dword ptr [esp+34]
:00407563 8BCC                  mov ecx, esp
:00407565 8964241C               mov dword ptr [esp+1C], esp
:00407569 50                    push eax
:0040756A C744243000000000       mov [esp+30], 00000000

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                   |
:00407572 E8FB940000             Call 00410A72
:00407577 8BCF                  mov ecx, edi
:00407579 E822010000             call 004076A0  ****传入的是用户名 ricemaster
:0040757E 8BF0                  mov esi, eax
:00407580 85F6                  test esi, esi
:00407582 0F84F0000000           je 00407678
:00407588 51                    push ecx
:00407589 8BCC                  mov ecx, esp
:0040758B 8964241C               mov dword ptr [esp+1C], esp

* Possible StringData Ref from Data Obj ->"EasunLee"
                   |
:0040758F 68F4A64100             push 0041A6F4

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:00407594 E8BF930000             Call 00410958
:00407599 8BCF                  mov ecx, edi
:0040759B E800010000             call 004076A0  ****传入的是 EasunLee
:004075A0 51                    push ecx
:004075A1 8BD8                  mov ebx, eax
:004075A3 8BCC                  mov ecx, esp
:004075A5 8964241C               mov dword ptr [esp+1C], esp

* Possible StringData Ref from Data Obj ->"EasunLee"
                   |
:004075A9 68F4A64100             push 0041A6F4

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:004075AE E8A5930000             Call 00410958
:004075B3 8BCF                  mov ecx, edi
:004075B5 E8E6000000             call 004076A0  ****传入的是 EasunLee
:004075BA 51                    push ecx
:004075BB 8BE8                  mov ebp, eax
:004075BD 8BCC                  mov ecx, esp
:004075BF 8964241C               mov dword ptr [esp+1C], esp

* Possible StringData Ref from Data Obj ->"easunlee98meiosys"
                   |
:004075C3 68E0A64100             push 0041A6E0

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:004075C8 E88B930000             Call 00410958
:004075CD 8BCF                  mov ecx, edi
:004075CF E8CC000000             call 004076A0  ****传入的是 easunlee98meiosys
:004075D4 51                    push ecx
:004075D5 89442418               mov dword ptr [esp+18], eax
:004075D9 8BCC                  mov ecx, esp
:004075DB 8964241C               mov dword ptr [esp+1C], esp

* Possible StringData Ref from Data Obj ->"Luyanghs&&Tsai&&bluebird"
                   |
:004075DF 68C4A64100             push 0041A6C4

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:004075E4 E86F930000             Call 00410958
:004075E9 8BCF                  mov ecx, edi
:004075EB E8B0000000             call 004076A0  ****传入的是
                                  Luyanghs&&Tsai&&bluebird
:004075F0 51                    push ecx
:004075F1 89442414               mov dword ptr [esp+14], eax
:004075F5 8BCC                  mov ecx, esp
:004075F7 8964241C               mov dword ptr [esp+1C], esp

* Possible StringData Ref from Data Obj ->"heshengwssu1091119"
                   |
:004075FB 68B0A64100             push 0041A6B0

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:00407600 E853930000             Call 00410958
:00407605 8BCF                  mov ecx, edi
:00407607 E894000000             call 004076A0  ****传入的是 heshengwssu1091119
:0040760C 51                    push ecx
:0040760D 8944241C               mov dword ptr [esp+1C], eax
:00407611 8BCC                  mov ecx, esp
:00407613 89642420               mov dword ptr [esp+20], esp

* Possible StringData Ref from Data Obj ->"200970878"
                   |
:00407617 68A4A64100             push 0041A6A4

* Reference To: MFC42.Ordinal:0219, Ord:0219h
                   |
:0040761C E837930000             Call 00410958
:00407621 8BCF                  mov ecx, edi
:00407623 E878000000             call 004076A0  ****传入的是 200970878
```

这个 `call` 还没完，先插入讲一下下面这个函数：

大家可以看到，上面的程序里 7 次用到了 `call 004076A0` 这个函数，它的具体算法是把传入字符串的每一位的 `ASCII` 值乘以 2 的（位置-1）次方，再加到一起。比如：`ricemaster`，就是 `ASCII(r)*1 + ASCII(i)*2 + ASCII(c)*4 + ASCII(e)*8 + ASCII(m)*16 …………`（我怎么觉得说得这么罗嗦啊，大家先凑合着看吧，后面有代码。）

```
:004076A0 64A100000000           mov eax, dword ptr fs:[00000000]
:004076A6 6AFF                  push FFFFFFFF
:004076A8 68781D4100             push 00411D78
:004076AD 50                    push eax
:004076AE 64892500000000         mov dword ptr fs:[00000000], esp
:004076B5 56                    push esi
:004076B6 57                    push edi
:004076B7 8B7C2418               mov edi, dword ptr [esp+18]
:004076BB 8B57F8                 mov edx, dword ptr [edi-08]
:004076BE 83FA03                 cmp edx, 00000003    ===》所输入的字符串必须大
                                      于 3 位
:004076C1 7D26                   jge 004076E9          ===》所以如果输入正确的
                                      话，这里是肯定跳的
:004076C3 8D4C2418               lea ecx, dword ptr [esp+18]
:004076C7 C7442410FFFFFFFF       mov [esp+10], FFFFFFFF

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                   |
:004076CF E836920000             Call 0041090A
:004076D4 33C0                  xor eax, eax
```



:004076D6 8B4C2408 mov ecx, dword ptr [esp+08]  

:004076DA 64890D00000000 mov dword ptr fs:[00000000], ecx  

:004076E1 5F pop edi  

:004076E2 5E pop esi  

:004076E3 83C40C add esp, 0000000C  

:004076E6 C20400 ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :004076C1(C)  

|

:004076E9 33F6 xor esi, esi  

:004076EB 33C9 xor ecx, ecx  

:004076ED 85D2 test edx, edx  

:004076EF 7E0D jle 004076FE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :004076FC(C) ===》 这个 call 的核心运算部分!!!  

|

:004076F1 0FBE0439 movsx eax, byte ptr [ecx+edi]  

:004076F5 D3E0 shl eax, cl  

:004076F7 03F0 add esi, eax  

:004076F9 41 inc ecx  

:004076FA 3BCA cmp ecx, edx  

:004076FC 7CF3 jl 004076F1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :004076EF(C)  

|

:004076FE 8D4C2418 lea ecx, dword ptr [esp+18]  

:00407702 C7442410FFFFFFFF mov [esp+10], FFFFFFFF

* Reference To: MFC42.Ordinal:0320, Ord:0320h  

|

:0040770A E8FB910000 Call 0041090A  

:0040770F 8B4C2408 mov ecx, dword ptr [esp+08]  

:00407713 8BC6 mov eax, esi  

:00407715 5F pop edi  

:00407716 64890D00000000 mov dword ptr fs:[00000000], ecx  

:0040771D 5E pop esi  

:0040771E 83C40C add esp, 0000000C  

:00407721 C20400 ret 0004

这个 call 的细微算法我看明白了，可是从宏观上讲，是仅仅是计算，还是别的什么意思，还请高手指点一下。

好，继续刚才的 call，希望大家还没有被我说迷糊：P  

此时，上述的 7 个字符串（1 个是用户名，6 个是程序自己的）都计算出了相应的值。

:00407628 81F678EE0220 xor esi, 2002EE78 ===》esi 里是根据用户名算出的值  

:0040762E 8B7C2414 mov edi, dword ptr [esp+14]  

:00407632 81EE21050E20 sub esi, 200E0521  

:00407638 8B542418 mov edx, dword ptr [esp+18]  

:0040763C 81F678563472 xor esi, 72345678  

:00407642 81EE88F76877 sub esi, 7768F788  

:00407648 33F3 xor esi, ebx ===》ebx 里是根据 Easunlee 算出的值  

:0040764A 8B5C2410 mov ebx, dword ptr [esp+10]  

:0040764E 03F5 add esi, ebp ===》ebp 里是根据 Easunlee 算出的值  

:00407650 33F3 xor esi, ebx ===》ebx 里是根据 Luyanghs&&Tsai&&bluebird 算出的值  

:00407652 33F7 xor esi, edi ===》edi 里是根据 easunlee98meiosys 算出的值  

:00407654 2BF2 sub esi, edx ===》edx 里是根据 heshengwssu1091119 算出的值  

:00407656 03F0 add esi, eax ===》eax 里是根据 200970878 算出的值  

:00407658 8B442434 mov eax, dword ptr [esp+34] ===》eax 里是注册码的 16 进制，把 esi 里的值传成 10 进制就是注册码了。   

:0040765C 3BF0 cmp esi, eax  

:0040765E 7518 jne 00407678 ===》跳就 over  

:00407660 8D4C2430 lea ecx, dword ptr [esp+30]  

:00407664 C7442428FFFFFFFF mov [esp+28], FFFFFFFF

* Reference To: MFC42.Ordinal:0320, Ord:0320h  

|

:0040766C E899920000 Call 0041090A  

:00407671 B801000000 mov eax, 00000001  

:00407676 EB13 jmp 0040768B

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

| :00407582(C), :0040765E(C)  

|

:00407678 8D4C2430 lea ecx, dword ptr [esp+30]  

:0040767C C7442428FFFFFFFF mov [esp+28], FFFFFFFF

* Reference To: MFC42.Ordinal:0320, Ord:0320h  

|

:00407684 E881920000 Call 0041090A  

:00407689 33C0 xor eax, eax ===》罪魁祸首!也是爆破的一种选择，至于其他爆破的选择，呵呵，显而易见的吧，我也没具体做。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

| :00407676(U)  

|

:0040768B 8B4C2420 mov ecx, dword ptr [esp+20]  

:0040768F 5F pop edi  

:00407690 5E pop esi  

:00407691 5D pop ebp  

:00407692 64890D00000000 mov dword ptr fs:[00000000], ecx  

:00407699 5B pop ebx  

:0040769A 83C41C add esp, 0000001C  

:0040769D C20800 ret 0008

算法到此结束，注册机也很容易写，毕竟那 6 个串都是固定的。

注册后程序会在其安装目录生成一个 `key.dll` 的文件，可是隐藏属性的哦，其内容如下：  

```ini
[Easun]
UserName = xqlk
User = ricemaster
code = 2109644410
```

这个程序总的说来挺简单的，从安装到成文一共花了 3 个小时吧，没走什么弯路，也没遇到什么陷阱，很适合我这种新手做练习用。不过里面的 `MFC` 我可没深入研究，看不懂：（

第一次写 `crack` 的文章，不足之处还请众位高手指教。  

饿了，吃饭去了………………………… 4dd466d0e1e0cc123afbe504da2462ca
