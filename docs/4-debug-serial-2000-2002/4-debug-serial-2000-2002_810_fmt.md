
# GetRight 4.5b

> 原文：[`www.pediy.com/kssd/pediy04/kanxue279.htm`](https://www.pediy.com/kssd/pediy04/kanxue279.htm)

`GetRight` 4.5b 注册分析

by `Fpc`/`CCG`&`BCG` @2001/12

tools: `trw1.23`, `Wdasm` 8.93

软件名称：`GetRight`

整理日期：2001.12.8

最新版本：4.5b

文件大小：2136KB

软件授权：共享软件

使用平台：Win9x/Me/NT/2000

本地下载：`hXtp://sd.onlinedown.net/down/getrt45b.exe`

软件简介：

`GetRight` 是目前最好用的浏览器下载工具，可让你用浏览器下载文件时有续传功能 (状态视窗会显示对方站台支持续传的情形)，免除临时有事需暂停下载或突然断线必须从头传起的困扰。附有时间排程工具，可设定时间来下载文件或是中断 Modem 拨接。可设定文件下载完毕时自动中断 Modem 拨接或是关机。可设定若文件下载中途发生断线时自动拨号上线并继续完成下载。有 Mirror Site List 功能，可在 Server timeout 时自动转到其他 Server 来下载文件。

安装时可以选择输入注册码，但跟了半天也找不到。安装后运行，关于，~~只需输入注册码：123654，##$$，注册码不合法。~~用 `trw` 设断点：`bpx hmemcpy`，再输入后，拦下来，慢慢跟，你可以到这里。注意到在此之前它会与一个假码比较，两者一致则立即失败

```assembly
:00401986 68F8675900  push 005967F8

* Possible StringData Ref from Data Obj ->"RegistrationCode"

                    |
:0040198B 68A04B5800  push 00584BA0
:00401990 8D45E8       lea eax, dword ptr [ebp-18]

* Possible StringData Ref from Data Obj ->"Config"

                    |
:00401993 686C4B5800  push 00584B6C
:00401998 50           push eax
:00401999 B9A0695900   mov ecx, 005969A0
:0040199E E879C71100   call 0051E11C
:004019A3 FF30         push dword ptr [eax]
:004019A5 8D45EC       lea eax, dword ptr [ebp-14]
:004019A8 C645FC03     mov [ebp-04], 03

* Possible StringData Ref from Data Obj ->"%s"

                    |
:004019AC 689C4B5800  push 00584B9C
:004019B1 50           push eax
:004019B2 E8CCAE1000   call 0050C883
:004019B7 83C40C       add esp, 0000000C
:004019BA 8D4DE8       lea ecx, dword ptr [ebp-18]
:004019BD C645FC02     mov [ebp-04], 02
:004019C1 E872161100   call 00513038
:004019C6 8D4DEC       lea ecx, dword ptr [ebp-14]  <- 输入的注册码
:004019C9 E8E21A0A00   call 004A34B0               <- 这里是关键，追进去
:004019CE 85C0         test eax, eax               <- 如果 eax==0 就失败了
:004019D0 7458         je 00401A2A                 <- 这里不能跳

* Possible Reference to String Resource ID=00327: "You Have Registered GetRight.  Thank You!"

                    |
:004019D2 6847010000  push 00000147
:004019D7 8D4DF0       lea ecx, dword ptr [ebp-10]
:004019DA E8801C1100   call 0051365F
:004019DF FF75F0       push [ebp-10]
:004019E2 8D8E94040000 lea ecx, dword ptr [esi+00000494]
:004019E8 E8F80C1100   call 005126E5
:004019ED 53           push ebx
:004019EE 8D8EC4010000 lea ecx, dword ptr [esi+000001C4]
:004019F4 E8080E1100   call 00512801
:004019F9 53           push ebx
... ...
```

进入那个 `call`，我们来看看，计算过程倒很长，不过不复杂：

```assembly
* Referenced by a CALL at Addresses:                      <- 很多处调用这个注册判断
|:00401760  , :004019C9  , :0041238B  , :004125E9  , :00415445
|:004228F2  , :00426C44  , :004337E9  , :004350A3  , :0043A9D5
|:0043DB00  , :00452CD2  , :004633E4  , :004638F7  , :004A9FE3
|:004AA2AE  , :004ABD2D  , :004C7EF7  , :004C8481  , :004CC0AD
|:004D38B0  , :004EF549  , :004F2DC6
|

:004A34B0 55          push ebp
:004A34B1 8BEC        mov ebp, esp
:004A34B3 83EC14      sub esp, 00000014
:004A34B6 53          push ebx
:004A34B7 57          push edi

* Possible Reference to Menu: MenuID_0001

                    |

* Possible Ref to Menu: MenuID_00B4, Item: "OK"

                    |
:004A34B8 6A01        push 00000001
:004A34BA 8BD9        mov ebx, ecx
:004A34BC 5F          pop edi                     <- edi=1
:004A34BD 897DF8      mov dword ptr [ebp-08], edi <- [ebp-8]是个注册标志，为 1 表示注册成功
:004A34C0 E863940600  call 0050C928
:004A34C5 8BCB        mov ecx, ebx
:004A34C7 E813940600  call 0050C8DF
:004A34CC 8B03        mov eax, dword ptr [ebx]
:004A34CE 8378F80C    cmp dword ptr [eax-08], 0000000C <- 注册码是否为 C 个字符
:004A34D2 7407        je 004A34DB
:004A34D4 33C0        xor eax, eax
:004A34D6 E9D9040000  jmp 004A39B4                <- 不是则 eax 清零去死

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A34D2(C)
|

:004A34DB 56          push esi
:004A34DC 57          push edi
:004A34DD 8D45F4      lea eax, dword ptr [ebp-0C]
:004A34E0 6A05        push 00000005             <- 取第 5 个字符，注意位置从 0 开始
:004A34E2 50          push eax                  <- 保存到这里
:004A34E3 8BCB        mov ecx, ebx              <- 输入的注册码
:004A34E5 E8348F0600  call 0050C41E             <- 这个 call 反复用到，作用是取一个字符
:004A34EA FF30        push dword ptr [eax]      <- 取出的一个字符
:004A34EC E8AF950500  call 004FCAA0             <- char->int 到 eax
:004A34F1 59          pop ecx
:004A34F2 8BF0        mov esi, eax              <- 保存
:004A34F4 8D4DF4      lea ecx, dword ptr [ebp-0C]
:004A34F7 8975F0      mov dword ptr [ebp-10], esi <- 保存
:004A34FA E839FB0600  call 00513038
:004A34FF 57          push edi
:004A3500 8D45EC      lea eax, dword ptr [ebp-14]
:004A3503 6A08        push 00000008             <- 第 8 个
:004A3505 50          push eax
:004A3506 8BCB        mov ecx, ebx
:004A3508 E8118F0600  call 0050C41E
:004A350D FF30        push dword ptr [eax]
:004A350F E88C950500  call 004FCAA0             <- char to int
:004A3514 59          pop ecx
:004A3515 8945F4      mov dword ptr [ebp-0C], eax <- 保存
:004A3518 8D4DEC      lea ecx, dword ptr [ebp-14]
:004A351B E818FB0600  call 00513038
:004A3520 8B45F4      mov eax, dword ptr [ebp-0C]
:004A3523 6A0A        push 0000000A
:004A3525 03C6        add eax, esi              <- eax=int(Irc[5])+int(Irc[8])
:004A3527 59          pop ecx                   <- ecx=10
:004A3528 99          cdq
:004A3529 F7F9        idiv ecx
:004A352B 83FA09      cmp edx, 00000009         <- 余数是否为 9
:004A352E 7404        je 004A3534               <- 正确则跳
:004A3530 8365F800    and dword ptr [ebp-08], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A352E(C)
|

:004A3534 837DF000    cmp dword ptr [ebp-10], 00000000 <- 注册码相应位置是否为 0

* Possible StringData Ref from Data Obj ->"00"

                    |
:004A3538 BEFCC35800  mov esi, 0058C3FC
:004A353D 7533        jne 004A3572              <- 不为 0 则正确，跳走
:004A353F 57          push edi                  <- 否则会有麻烦，这下面我没细看
:004A3540 8D45EC      lea eax, dword ptr [ebp-14]

* Possible Reference to Menu: MenuID_0005

                    |
:004A3543 6A05        push 00000005
:004A3545 50          push eax
:004A3546 8BCB        mov ecx, ebx
```



:004A3548 `E8D18E0600` call `0050C41E`

:004A354D `56` push `esi`

:004A354E `FF30` push `dword ptr [eax]`

:004A3550 `E8EB830500` call `004FB940`

:004A3555 `F7D8` neg `eax`

:004A3557 `59` pop `ecx`

:004A3558 `1BC0` sbb `eax, eax`

:004A355A `59` pop `ecx`

:004A355B `F7D8` neg `eax`

:004A355D `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A3560 `8845FF` mov `byte ptr [ebp-01], al`

:004A3563 `E8D0FA0600` call `00513038`

:004A3568 `807DFF00` cmp `byte ptr [ebp-01], 00`

:004A356C `7404` je `004A3572`

:004A356E `8365F800` and `dword ptr [ebp-08], 00000000`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

| `:004A353D(C), :004A356C(C)`
|

:004A3572 `837DF400` cmp `dword ptr [ebp-0C], 00000000` `<- 另一位置是否不为 0`

:004A3576 `7533` jne `004A35AB` `<- 正确则跳走`

简单说明上面这一段：

假设输入了注册码：`912345678912`，第 5 个是 5，第 8 个是 8，相加 13，只要这个和对 10 取模为 9 即可，比如 8 改为 4 满足要求。

继续：

:004A3578 `57` push `edi`

:004A3579 `8D45EC` lea `eax, dword ptr [ebp-14]`

:004A357C `6A08` push `00000008`

:004A357E `50` push `eax`

:004A357F `8BCB` mov `ecx, ebx`

:004A3581 `E8988E0600` call `0050C41E`

:004A3586 `56` push `esi`

:004A3587 `FF30` push `dword ptr [eax]`

:004A3589 `E8B2830500` call `004FB940`

:004A358E `F7D8` neg `eax`

:004A3590 `59` pop `ecx`

:004A3591 `1BC0` sbb `eax, eax`

:004A3593 `59` pop `ecx`

:004A3594 `F7D8` neg `eax`

:004A3596 `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A3599 `8845FF` mov `byte ptr [ebp-01], al`

:004A359C `E897FA0600` call `00513038`

:004A35A1 `807DFF00` cmp `byte ptr [ebp-01], 00`

:004A35A5 `7404` je `004A35AB`

:004A35A7 `8365F800` and `dword ptr [ebp-08], 00000000`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

| `:004A3576(C), :004A35A5(C)`
|

:004A35AB `57` push `edi`

:004A35AC `8D45EC` lea `eax, dword ptr [ebp-14]`

:004A35AF `57` push `edi` `<- 第 1 个`

:004A35B0 `50` push `eax`

:004A35B1 `8BCB` mov `ecx, ebx`

:004A35B3 `E8668E0600` call `0050C41E`

:004A35B8 `FF30` push `dword ptr [eax]`

:004A35BA `E8E1940500` call `004FCAA0`

:004A35BF `59` pop `ecx`

:004A35C0 `8945F0` mov `dword ptr [ebp-10], eax`

:004A35C3 `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A35C6 `E86DFA0600` call `00513038`

:004A35CB `57` push `edi`

:004A35CC `8D45EC` lea `eax, dword ptr [ebp-14]`

`* Possible Reference to Dialog: DialogID_010C, CONTROL_ID:0006, "Yes"`

|

:004A35CF `6A06` push `00000006` `<- 第 6 个`

:004A35D1 `50` push `eax`

:004A35D2 `8BCB` mov `ecx, ebx`

:004A35D4 `E8458E0600` call `0050C41E`

:004A35D9 `FF30` push `dword ptr [eax]`

:004A35DB `E8C0940500` call `004FCAA0`

:004A35E0 `59` pop `ecx`

:004A35E1 `8945F4` mov `dword ptr [ebp-0C], eax`

:004A35E4 `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A35E7 `E84CFA0600` call `00513038`

:004A35EC `837DF000` cmp `dword ptr [ebp-10], 00000000`

:004A35F0 `7532` jne `004A3624` `<- 跳走`

:004A35F2 `57` push `edi`

:004A35F3 `8D45EC` lea `eax, dword ptr [ebp-14]`

:004A35F6 `57` push `edi`

:004A35F7 `50` push `eax`

:004A35F8 `8BCB` mov `ecx, ebx`

:004A35FA `E81F8E0600` call `0050C41E`

:004A35FF `56` push `esi`

:004A3600 `FF30` push `dword ptr [eax]`

:004A3602 `E839830500` call `004FB940`

:004A3607 `F7D8` neg `eax`

:004A3609 `59` pop `ecx`

:004A360A `1BC0` sbb `eax, eax`

:004A360C `59` pop `ecx`

:004A360D `F7D8` neg `eax`

:004A360F `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A3612 `8845FF` mov `byte ptr [ebp-01], al`

:004A3615 `E81EFA0600` call `00513038`

:004A361A `807DFF00` cmp `byte ptr [ebp-01], 00`

:004A361E `7404` je `004A3624`

:004A3620 `8365F800` and `dword ptr [ebp-08], 00000000`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

| `:004A35F0(C), :004A361E(C)`
|

:004A3624 `837DF400` cmp `dword ptr [ebp-0C], 00000000`

:004A3628 `7533` jne `004A365D` `<- 跳走`

:004A362A `57` push `edi`

:004A362B `8D45EC` lea `eax, dword ptr [ebp-14]`

`* Possible Reference to Dialog: DialogID_010C, CONTROL_ID:0006, "Yes"`

|

:004A362E `6A06` push `00000006`

:004A3630 `50` push `eax`

:004A3631 `8BCB` mov `ecx, ebx`

:004A3633 `E8E68D0600` call `0050C41E`

:004A3638 `56` push `esi`

:004A3639 `FF30` push `dword ptr [eax]`

:004A363B `E800830500` call `004FB940`

:004A3640 `F7D8` neg `eax`

:004A3642 `59` pop `ecx`

:004A3643 `1BC0` sbb `eax, eax`

:004A3645 `59` pop `ecx`

:004A3646 `F7D8` neg `eax`

:004A3648 `8D4DEC` lea `ecx, dword ptr [ebp-14]`

:004A364B `8845FF` mov `byte ptr [ebp-01], al`

:004A364E `E8E5F90600` call `00513038`

:004A3653 `807DFF00` cmp `byte ptr [ebp-01], 00`

:004A3657 `7404` je `004A365D`

:004A3659 `8365F800` and `dword ptr [ebp-08], 00000000`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

| `:004A3628(C), :004A3657(C)`
|

:004A365D `8B45F0` mov `eax, dword ptr [ebp-10]`

:004A3660 `8B4DF4` mov `ecx, dword ptr [ebp-0C]`

:004A3663 `03C1` add `eax, ecx`

:004A3665 `6A0A` push `0000000A`

:004A3667 `99` cdq

:004A3668 `59` pop `ecx`

:004A3669 `F7F9` idiv `ecx`

:004A366B `83FA03` cmp `edx, 00000003` `<- 余数应为 3`

:004A366E `7404` je `004A3674`

:004A3670 `8365F800` and `dword ptr [ebp-08], 00000000`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

| `:004A366E(C)`
|

:004A3674 `57` push `edi`

:004A3675 `8D45EC` lea `eax, dword ptr [ebp-14]`

:004A3678 `6A02` push `00000002` `<- 第 2 个`

:004A367A `50` push `eax`

:004A367B `8BCB` mov `ecx, ebx`

:004A367D `E89C8D0600` call `0050C41E`

:004A3682 `FF30` push `dword ptr [eax]`

:004A3684 `E817940500` call `004FCAA0`

:004A3689 `59` pop `ecx`

:004A368A `8945F0` mov `dword ptr [ebp-10], eax`



:004A368D 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A3690 E8A3F90600              call 00513038   

:004A3695 57                      push edi   

:004A3696 8D45EC                  lea eax, dword ptr [ebp-14]   

:004A3699 6A0B                    push 0000000B              <- 第 11 个(最后一个)   

:004A369B 50                      push eax   

:004A369C 8BCB                    mov ecx, ebx   

:004A369E E87B8D0600              call 0050C41E   

:004A36A3 FF30                    push dword ptr [eax]   

:004A36A5 E8F6930500              call 004FCAA0   

:004A36AA 59                      pop ecx   

:004A36AB 8945F4                  mov dword ptr [ebp-0C], eax   

:004A36AE 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A36B1 E882F90600              call 00513038   

:004A36B6 8B45F0                  mov eax, dword ptr [ebp-10]   

:004A36B9 8B4DF4                  mov ecx, dword ptr [ebp-0C]   

:004A36BC 03C1                    add eax, ecx   

:004A36BE 6A0A                    push 0000000A   

:004A36C0 99                      cdq   

:004A36C1 59                      pop ecx   

:004A36C2 F7F9                    idiv ecx   

:004A36C4 83FA08                  cmp edx, 00000008        <- 余数为 8   

:004A36C7 7404                    je 004A36CD   

:004A36C9 8365F800                and dword ptr [ebp-08], 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004A36C7(C)   

|   

:004A36CD 837DF000                cmp dword ptr [ebp-10], 00000000   

:004A36D1 7533                    jne 004A3706             <- 跳走   

:004A36D3 57                      push edi   

:004A36D4 8D45EC                  lea eax, dword ptr [ebp-14]   

:004A36D7 6A02                    push 00000002   

:004A36D9 50                      push eax   

:004A36DA 8BCB                    mov ecx, ebx   

:004A36DC E83D8D0600              call 0050C41E   

:004A36E1 56                      push esi   

:004A36E2 FF30                    push dword ptr [eax]   

:004A36E4 E857820500              call 004FB940   

:004A36E9 F7D8                    neg eax   

:004A36EB 59                      pop ecx   

:004A36EC 1BC0                    sbb eax, eax   

:004A36EE 59                      pop ecx   

:004A36EF F7D8                    neg eax   

:004A36F1 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A36F4 8845FF                  mov byte ptr [ebp-01], al   

:004A36F7 E83CF90600              call 00513038   

:004A36FC 807DFF00                cmp byte ptr [ebp-01], 00   

:004A3700 7404                    je 004A3706   

:004A3702 8365F800                and dword ptr [ebp-08], 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:004A36D1(C), :004A3700(C)   

|   

:004A3706 837DF400                cmp dword ptr [ebp-0C], 00000000   

:004A370A 7533                    jne 004A373F             <- 跳走   

:004A370C 57                      push edi   

:004A370D 8D45EC                  lea eax, dword ptr [ebp-14]   

:004A3710 6A0B                    push 0000000B   

:004A3712 50                      push eax   

:004A3713 8BCB                    mov ecx, ebx   

:004A3715 E8048D0600              call 0050C41E   

:004A371A 56                      push esi   

:004A371B FF30                    push dword ptr [eax]   

:004A371D E81E820500              call 004FB940   

:004A3722 F7D8                    neg eax   

:004A3724 59                      pop ecx   

:004A3725 1BC0                    sbb eax, eax   

:004A3727 59                      pop ecx   

:004A3728 F7D8                    neg eax   

:004A372A 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A372D 8845FF                  mov byte ptr [ebp-01], al   

:004A3730 E803F90600              call 00513038   

:004A3735 807DFF00                cmp byte ptr [ebp-01], 00   

:004A3739 7404                    je 004A373F   

:004A373B 8365F800                and dword ptr [ebp-08], 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:004A370A(C), :004A3739(C)   

|   

:004A373F 57                      push edi   

:004A3740 8D45EC                  lea eax, dword ptr [ebp-14]   

* Possible Reference to Dialog: DialogID_0099, CONTROL_ID:0003, "OK && Remember"   

                                       |   

:004A3743 6A03                    push 00000003              <- 第 3 个   

:004A3745 50                      push eax   

:004A3746 8BCB                    mov ecx, ebx   

:004A3748 E8D18C0600              call 0050C41E   

:004A374D FF30                    push dword ptr [eax]   

:004A374F E84C930500              call 004FCAA0   

:004A3754 59                      pop ecx   

:004A3755 8945F0                  mov dword ptr [ebp-10], eax   

:004A3758 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A375B E8D8F80600              call 00513038   

:004A3760 57                      push edi   

:004A3761 8D45EC                  lea eax, dword ptr [ebp-14]   

* Possible Reference to Menu: MenuID_0004   

                                       |   

* Possible Reference to Dialog: DialogID_014E, CONTROL_ID:0004, "Switch and use the Full options for   

"   

                                       |   

:004A3764 6A04                    push 00000004              <- 第 4 个   

:004A3766 50                      push eax   

:004A3767 8BCB                    mov ecx, ebx   

:004A3769 E8B08C0600              call 0050C41E   

:004A376E FF30                    push dword ptr [eax]   

:004A3770 E82B930500              call 004FCAA0   

:004A3775 59                      pop ecx   

:004A3776 8945F4                  mov dword ptr [ebp-0C], eax   

:004A3779 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A377C E8B7F80600              call 00513038   

:004A3781 8B45F0                  mov eax, dword ptr [ebp-10]   

:004A3784 8B4DF4                  mov ecx, dword ptr [ebp-0C]   

:004A3787 03C1                    add eax, ecx   

:004A3789 6A0A                    push 0000000A   

:004A378B 99                      cdq   

:004A378C 59                      pop ecx   

:004A378D F7F9                    idiv ecx   

:004A378F 85D2                    test edx, edx   

:004A3791 7404                    je 004A3797              <-余数应为 0   

:004A3793 8365F800                and dword ptr [ebp-08], 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004A3791(C)   

|   

:004A3797 837DF000                cmp dword ptr [ebp-10], 00000000   

:004A379B 7533                    jne 004A37D0             <- 跳走   

:004A379D 57                      push edi   

:004A379E 8D45EC                  lea eax, dword ptr [ebp-14]   

* Possible Reference to Dialog: DialogID_0099, CONTROL_ID:0003, "OK && Remember"   

                                       |   

:004A37A1 6A03                    push 00000003   

:004A37A3 50                      push eax   

:004A37A4 8BCB                    mov ecx, ebx   

:004A37A6 E8738C0600              call 0050C41E   

:004A37AB 56                      push esi   

:004A37AC FF30                    push dword ptr [eax]   

:004A37AE E88D810500              call 004FB940   

:004A37B3 F7D8                    neg eax   

:004A37B5 59                      pop ecx   

:004A37B6 1BC0                    sbb eax, eax   

:004A37B8 59                      pop ecx   

:004A37B9 F7D8                    neg eax   

:004A37BB 8D4DEC                  lea ecx, dword ptr [ebp-14]   

:004A37BE 8845FF                  mov byte ptr [ebp-01], al   

:004A37C1 E872F80600              call 00513038   

:004A37C6 807DFF00                cmp byte ptr [ebp-01], 00   

:004A37CA 7404                    je 004A37D0   

:004A37CC 8365F800                and dword ptr [ebp-08], 00000000



# 汇编代码分析

## 跳转引用与代码段

### 地址 004A37D0 至 004A3805

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:004A379B(C), :004A37CA(C)

:004A37D0 837DF400  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp dword ptr [ebp-0C], 00000000
:004A37D4 7533 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jne 004A3809 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 跳走
:004A37D6 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A37D7 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]

* Possible Reference to Menu: `MenuID_0004`
|
* Possible Reference to Dialog: `DialogID_014E`, `CONTROL_ID:0004`, "Switch and use the Full options for "
|

:004A37DA 6A04 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000004
:004A37DC 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A37DD 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A37DF E83A8C0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A37E4 56 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push esi
:004A37E5 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A37E7 E854810500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FB940
:004A37EC F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A37EE 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A37EF 1BC0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sbb eax, eax
:004A37F1 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A37F2 F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A37F4 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A37F7 8845FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [ebp-01], al
:004A37FA E839F80600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A37FF 807DFF00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp byte ptr [ebp-01], 00
:004A3803 7404 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; je 004A3809
:004A3805 8365F800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; and dword ptr [ebp-08], 00000000

### 地址 004A3809 至 004A385E

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:004A37D4(C), :004A3803(C)

:004A3809 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A380A 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]

* Possible Reference to Dialog: `DialogID_010C`, `CONTROL_ID:0007`, "No"
|

:004A380D 6A07 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000007 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 第 7 个
:004A380F 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A3810 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A3812 E8078C0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A3817 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A3819 E882920500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FCAA0
:004A381E 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A381F 8945F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov dword ptr [ebp-10], eax
:004A3822 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A3825 E80EF80600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A382A 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A382B 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]
:004A382E 6A0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 0000000A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 第 10 个
:004A3830 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A3831 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A3833 E8E68B0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A3838 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A383A E861920500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FCAA0
:004A383F 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A3840 8945F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov dword ptr [ebp-0C], eax
:004A3843 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A3846 E8EDF70600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A384B 8B45F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov eax, dword ptr [ebp-10]
:004A384E 8B4DF4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, dword ptr [ebp-0C]
:004A3851 03C1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add eax, ecx
:004A3853 6A0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 0000000A
:004A3855 99 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cdq
:004A3856 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A3857 F7F9 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; idiv ecx
:004A3859 83FA05 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp edx, 00000005 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 余数为 5
:004A385C 7404 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; je 004A3862
:004A385E 8365F800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; and dword ptr [ebp-08], 00000000

### 地址 004A3862 至 004A3897

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004A385C(C)

:004A3862 837DF000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp dword ptr [ebp-10], 00000000
:004A3866 7533 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jne 004A389B &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 跳走
:004A3868 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A3869 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]

* Possible Reference to Dialog: `DialogID_010C`, `CONTROL_ID:0007`, "No"
|

:004A386C 6A07 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000007
:004A386E 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A386F 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A3871 E8A88B0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A3876 56 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push esi
:004A3877 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A3879 E8C2800500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FB940
:004A387E F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A3880 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A3881 1BC0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sbb eax, eax
:004A3883 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A3884 F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A3886 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A3889 8845FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [ebp-01], al
:004A388C E8A7F70600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A3891 807DFF00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp byte ptr [ebp-01], 00
:004A3895 7404 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; je 004A389B
:004A3897 8365F800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; and dword ptr [ebp-08], 00000000

### 地址 004A389B 至 004A38D0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:004A3866(C), :004A3895(C)

:004A389B 837DF400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp dword ptr [ebp-0C], 00000000
:004A389F 7533 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jne 004A38D4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 跳走
:004A38A1 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A38A2 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]
:004A38A5 6A0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 0000000A
:004A38A7 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A38A8 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A38AA E86F8B0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A38AF 56 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push esi
:004A38B0 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A38B2 E889800500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FB940
:004A38B7 F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A38B9 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A38BA 1BC0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sbb eax, eax
:004A38BC 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A38BD F7D8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; neg eax
:004A38BF 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A38C2 8845FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [ebp-01], al
:004A38C5 E86EF70600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A38CA 807DFF00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp byte ptr [ebp-01], 00
:004A38CE 7404 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; je 004A38D4
:004A38D0 8365F800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; and dword ptr [ebp-08], 00000000

### 地址 004A38D4 至 004A38F6

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:004A389F(C), :004A38CE(C)

:004A38D4 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A38D5 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]
:004A38D8 6A00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 第 0 个
:004A38DA 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A38DB 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A38DD E83C8B0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A38E2 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A38E4 E8B7910500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FCAA0
:004A38E9 59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pop ecx
:004A38EA 8945F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov dword ptr [ebp-10], eax
:004A38ED 8D4DEC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [ebp-14]
:004A38F0 E843F70600 &nbsp;&nbsp;&nbsp;&nbsp; call 00513038
:004A38F5 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:004A38F6 8D45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ebp-14]

### 地址 004A38F9 至 004A3905

* Possible Reference to Dialog: `DialogID_00A0`, `CONTROL_ID:0009`, "Help"
|

```assembly
:004A38F9 6A09 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000009 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;- 第 9 个
:004A38FB 50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push eax
:004A38FC 8BCB &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebx
:004A38FE E81B8B0600 &nbsp;&nbsp;&nbsp;&nbsp; call 0050C41E
:004A3903 FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push dword ptr [eax]
:004A3905 E896910500 &nbsp;&nbsp;&nbsp;&nbsp; call 004FCAA0
```



```
:004A390A 59                               pop ecx
:004A390B 8945F4                           mov dword ptr [ebp-0C], eax
:004A390E 8D4DEC                           lea ecx, dword ptr [ebp-14]
:004A3911 E822F70600                       call 00513038
:004A3916 8B45F0                           mov eax, dword ptr [ebp-10]
:004A3919 8B4DF4                           mov ecx, dword ptr [ebp-0C]
:004A391C 03C1                             add eax, ecx
:004A391E 6A0A                             push 0000000A
:004A3920 99                               cdq
:004A3921 59                               pop ecx
:004A3922 F7F9                             idiv ecx
:004A3924 83FA02                           cmp edx, 00000002             <- 余数为 2
:004A3927 7404                             je 004A392D

:004A3929 8365F800                         and dword ptr [ebp-08], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004A3927(C)

:004A392D 837DF000                         cmp dword ptr [ebp-10], 00000000
:004A3931 7533                             jne 004A3966                 <- 跳走

:004A3933 57                               push edi
:004A3934 8D45EC                           lea eax, dword ptr [ebp-14]
:004A3937 6A00                             push 00000000
:004A3939 50                               push eax
:004A393A 8BCB                             mov ecx, ebx
:004A393C E8DD8A0600                       call 0050C41E
:004A3941 56                               push esi
:004A3942 FF30                             push dword ptr [eax]
:004A3944 E8F77F0500                       call 004FB940
:004A3949 F7D8                             neg eax
:004A394B 59                               pop ecx
:004A394C 1BC0                             sbb eax, eax
:004A394E 59                               pop ecx
:004A394F F7D8                             neg eax
:004A3951 8D4DEC                           lea ecx, dword ptr [ebp-14]
:004A3954 8845FF                           mov byte ptr [ebp-01], al
:004A3957 E8DCF60600                       call 00513038
:004A395C 807DFF00                         cmp byte ptr [ebp-01], 00
:004A3960 7404                             je 004A3966
:004A3962 8365F800                         and dword ptr [ebp-08], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004A3931(C), :004A3960(C)

:004A3966 837DF400                         cmp dword ptr [ebp-0C], 00000000
:004A396A 7533                             jne 004A399F                 <- 跳走

:004A396C 57                               push edi
:004A396D 8D45EC                           lea eax, dword ptr [ebp-14]

* Possible Reference to Dialog: DialogID_00A0, CONTROL_ID:0009, "Help"

:004A3970 6A09                             push 00000009
:004A3972 50                               push eax
:004A3973 8BCB                             mov ecx, ebx
:004A3975 E8A48A0600                       call 0050C41E
:004A397A 56                               push esi
:004A397B FF30                             push dword ptr [eax]
:004A397D E8BE7F0500                       call 004FB940
:004A3982 F7D8                             neg eax
:004A3984 59                               pop ecx
:004A3985 1BC0                             sbb eax, eax
:004A3987 59                               pop ecx
:004A3988 F7D8                             neg eax
:004A398A 8D4DEC                           lea ecx, dword ptr [ebp-14]
:004A398D 8845FF                           mov byte ptr [ebp-01], al
:004A3990 E8A3F60600                       call 00513038
:004A3995 807DFF00                         cmp byte ptr [ebp-01], 00
:004A3999 7404                             je 004A399F
:004A399B 8365F800                         and dword ptr [ebp-08], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004A396A(C), :004A3999(C)

:004A399F 57                               push edi                     <- 到这里
:004A39A0 57                               push edi
:004A39A1 8BCB                             mov ecx, ebx
:004A39A3 E810000000                       call 004A39B8
:004A39A8 85C0                             test eax, eax
:004A39AA 7404                             je 004A39B0                <- 这里会跳走

:004A39AC 8365F800                         and dword ptr [ebp-08], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004A39AA(C)

:004A39B0 8B45F8                           mov eax, dword ptr [ebp-08] <- eax=[ebp-8]，如果注册码符合条件，eax=1，否则 eax=0
:004A39B3 5E                               pop esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004A34D6(U)

:004A39B4 5F                               pop edi
:004A39B5 5B                               pop ebx
:004A39B6 C9                               leave
:004A39B7 C3                               ret                          <- 返回
```

总结一下：

设输入的注册码为 `Irc[]`，那么
```
strlen(Irc) 应为 0xC;
(Irc[5]+Irc[8])%10==9;
(Irc[1]+Irc[6])%10==3;
(Irc[2]+Irc[11])%10==8;
(Irc[3]+Irc[4])%10==0;
(Irc[7]+Irc[10])%10==5;
(Irc[0]+Irc[9])%10==2;
同时最好没有 0 出现;
```

可用的注册码：`127645134121`

写注册机用随机数，找出满足条件的即可。 `4dd466d0e1e0cc123afbe504da2462ca`
