
:0050741D `E80EDBEFFF` `call 00404F30` 将余数"7"与余数"0"连接起来----->"70"

:00507422 `8B85FCFEFFFF` `mov eax, dword ptr [ebp+FFFFFEFC]`

:00507428 `8D9500FFFFFF` `lea edx, dword ptr [ebp+FFFFFF00]`

:0050742E `E8B9030000` `call 005077EC`

:00507433 `8B9500FFFFFF` `mov edx, dword ptr [ebp+FFFFFF00]`

:00507439 `8D45EC` `lea eax, dword ptr [ebp-14]`

:0050743C `E8BFD8EFFF` `call 00404D00`

:00507441 `33F6` `xor esi, esi` `esi` 清零

:00507443 `BB01000000` `mov ebx, 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00507480(C)`
|

:00507448 `8D85F4FEFFFF` `lea eax, dword ptr [ebp+FFFFFEF4]`

:0050744E `50` `push eax`

:0050744F `8D95F0FEFFFF` `lea edx, dword ptr [ebp+FFFFFEF0]`

:00507455 `8B45D4` `mov eax, dword ptr [ebp-2C]`

:00507458 `E88F030000` `call 005077EC`

:0050745D `8B85F0FEFFFF` `mov eax, dword ptr [ebp+FFFFFEF0]` "60708"(见上)--->`eax`

:00507463 `B901000000` `mov ecx, 00000001`

:00507468 `8BD3` `mov edx, ebx`

:0050746A `E87D42F3FF` `call 0043B6EC`

:0050746F `8B85F4FEFFFF` `mov eax, dword ptr [ebp+FFFFFEF4]`

:00507475 `E8E224F0FF` `call 0040995C`

:0050747A `03F0` `add esi, eax`

:0050747C `43` `inc ebx`

:0050747D `83FB06` `cmp ebx, 00000006`

:00507480 `75C6` `jne 00507448` 又一个循环,将 60708 各位相加，即 6+0+7+0+8=0x15----->`esi`

:00507482 `8BC6` `mov eax, esi`

:00507484 `B90A000000` `mov ecx, 0000000A`

:00507489 `99` `cdq`

:0050748A `F7F9` `idiv ecx` 0x15/0xA

:0050748C `8BF2` `mov esi, edx` 余数为"1"--->`esi`

:0050748E `8D95E4FEFFFF` `lea edx, dword ptr [ebp+FFFFFEE4]`

:00507494 `8B45EC` `mov eax, dword ptr [ebp-14]`

:00507497 `E850030000` `call 005077EC`

:0050749C `FFB5E4FEFFFF` `push dword ptr [ebp+FFFFFEE4]`

:005074A2 `8D95E0FEFFFF` `lea edx, dword ptr [ebp+FFFFFEE0]`

:005074A8 `8BC6` `mov eax, esi`

:005074AA `E84123F0FF` `call 004097F0`

:005074AF `FFB5E0FEFFFF` `push dword ptr [ebp+FFFFFEE0]`

:005074B5 `FF75E0` `push [ebp-20]`

:005074B8 `8D85E8FEFFFF` `lea eax, dword ptr [ebp+FFFFFEE8]`

:005074BE `BA03000000` `mov edx, 00000003`

:005074C3 `E820DBEFFF` `call 00404FE8` 将以上得到的余数连接起来得到数"7014"

:005074C8 `8B85E8FEFFFF` `mov eax, dword ptr [ebp+FFFFFEE8]` "7014"--->`eax`

:005074CE `8D95ECFEFFFF` `lea edx, dword ptr [ebp+FFFFFEEC]`

:005074D4 `E813030000` `call 005077EC`

:005074D9 `8B95ECFEFFFF` `mov edx, dword ptr [ebp+FFFFFEEC]`

:005074DF `8D45EC` `lea eax, dword ptr [ebp-14]`

:005074E2 `E819D8EFFF` `call 00404D00`

:005074E7 `33F6` `xor esi, esi` `esi` 清零

:005074E9 `BB01000000` `mov ebx, 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00507526(C)`
|

:005074EE `8D85DCFEFFFF` `lea eax, dword ptr [ebp+FFFFFEDC]`

:005074F4 `50` `push eax`

:005074F5 `8D95D8FEFFFF` `lea edx, dword ptr [ebp+FFFFFED8]`

:005074FB `8B45EC` `mov eax, dword ptr [ebp-14]`

:005074FE `E8E9020000` `call 005077EC`

:00507503 `8B85D8FEFFFF` `mov eax, dword ptr [ebp+FFFFFED8]` "7014"--->`eax`

:00507509 `B901000000` `mov ecx, 00000001`

:0050750E `8BD3` `mov edx, ebx`

:00507510 `E8D741F3FF` `call 0043B6EC`

:00507515 `8B85DCFEFFFF` `mov eax, dword ptr [ebp+FFFFFEDC]`

:0050751B `E83C24F0FF` `call 0040995C`

:00507520 `03F0` `add esi, eax`

:00507522 `43` `inc ebx`

:00507523 `83FB05` `cmp ebx, 00000005`

:00507526 `75C6` `jne 005074EE` 又一个循环,将 7014 各位相加，即 7+0+1+4=0xC----->`esi`

:00507528 `8BC6` `mov eax, esi`

:0050752A `B90A000000` `mov ecx, 0000000A`

:0050752F `99` `cdq`

:00507530 `F7F9` `idiv ecx` 0xC/0xA

:00507532 `8BF2` `mov esi, edx` 余数为"2"--->`esi`

:00507534 `8D95D0FEFFFF` `lea edx, dword ptr [ebp+FFFFFED0]`

:0050753A `8B45EC` `mov eax, dword ptr [ebp-14]`

:0050753D `E8AA020000` `call 005077EC`

:00507542 `8D85D0FEFFFF` `lea eax, dword ptr [ebp+FFFFFED0]`

:00507548 `50` `push eax`

:00507549 `8D95CCFEFFFF` `lea edx, dword ptr [ebp+FFFFFECC]`

:0050754F `8BC6` `mov eax, esi`

:00507551 `E89A22F0FF` `call 004097F0`

:00507556 `8B95CCFEFFFF` `mov edx, dword ptr [ebp+FFFFFECC]`

:0050755C `58` `pop eax`

:0050755D `E8CED9EFFF` `call 00404F30` 将"2"与"7014"连接起来，得到第五部分真注册码，即"70142"

:00507562 `8B85D0FEFFFF` `mov eax, dword ptr [ebp+FFFFFED0]`

:00507568 `8D95D4FEFFFF` `lea edx, dword ptr [ebp+FFFFFED4]`

:0050756E `E879020000` `call 005077EC` 真码各位取反

:00507573 `8B95D4FEFFFF` `mov edx, dword ptr [ebp+FFFFFED4]`

:00507579 `8D45EC` `lea eax, dword ptr [ebp-14]`

:0050757C `E87FD7EFFF` `call 00404D00`

:00507581 `8D85C4FEFFFF` `lea eax, dword ptr [ebp+FFFFFEC4]`

:00507587 `50` `push eax`

:00507588 `B905000000` `mov ecx, 00000005`

:0050758D `BA19000000` `mov edx, 00000019`

:00507592 `8B45F4` `mov eax, dword ptr [ebp-0C]` 取第五部分的假码

:00507595 `E85241F3FF` `call 0043B6EC`

:0050759A `8B85C4FEFFFF` `mov eax, dword ptr [ebp+FFFFFEC4]`

:005075A0 `8D95C8FEFFFF` `lea edx, dword ptr [ebp+FFFFFEC8]`

:005075A6 `E841020000` `call 005077EC` 第五部分的假码取反

:005075AB `8B95C8FEFFFF` `mov edx, dword ptr [ebp+FFFFFEC8]`

:005075B1 `8B45EC` `mov eax, dword ptr [ebp-14]`

:005075B4 `E8BBDAEFFF` `call 00405074` 第五部分的经过变换的真假注册码相比

:005075B9 `7404` `je 005075BF` 相等就跳，注册成功，否则去死(爆破点)

:005075BB `C645F300` `mov [ebp-0D], 00`

所以注册信息为：

序列号：`3781489924572`

注册名：`wzh123`

注册码：`H1287-29685-60708-81168-70142`

由于注册算法用到了序列号，所以，一个注册码只对应一台机器，你只好自己算算了^-^

`4dd466d0e1e0cc123afbe504da2462ca`
