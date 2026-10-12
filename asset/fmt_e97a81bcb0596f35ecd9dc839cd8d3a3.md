
:0040E9AD B901000000                `mov ecx, 00000001`

:0040E9B2 8B5590                  `mov edx, dword ptr [ebp-70]`

:0040E9B5 E85A142B00              `call 006BFE14` <====取第 V4 位的硬盘序列号

:0040E9BA 8D45E4                  `lea eax, dword ptr [ebp-1C]`

:0040E9BD 50                      `push eax`

:0040E9BE 8D45E0                  `lea eax, dword ptr [ebp-20]`

:0040E9C1 E8D236FFFF              `call 00402098`

:0040E9C6 8BC8                    `mov ecx, eax`

:0040E9C8 FF45C4                  `inc [ebp-3C]`

:0040E9CB 8D45F4                  `lea eax, dword ptr [ebp-0C]`

:0040E9CE 5A                      `pop edx`

:0040E9CF E8CC0F2B00              `call 006BF9A0`

:0040E9D4 8D55E0                  `lea edx, dword ptr [ebp-20]`

:0040E9D7 8D45F4                  `lea eax, dword ptr [ebp-0C]`

:0040E9DA E8990F2B00              `call 006BF978`

:0040E9DF FF4DC4                  `dec [ebp-3C]`

:0040E9E2 8D45E0                  `lea eax, dword ptr [ebp-20]`

:0040E9E5 BA02000000              `mov edx, 00000002`

:0040E9EA E8590F2B00              `call 006BF948`

:0040E9EF FF4DC4                  `dec [ebp-3C]`

:0040E9F2 8D45E4                  `lea eax, dword ptr [ebp-1C]`

:0040E9F5 BA02000000              `mov edx, 00000002`

:0040E9FA E8490F2B00              `call 006BF948`

:0040E9FF FF45A0                  `inc [ebp-60]`

:0040EA02 8B4DA0                  `mov ecx, dword ptr [ebp-60]`

:0040EA05 8B4594                  `mov eax, dword ptr [ebp-6C]`

:0040EA08 3BC8                    `cmp ecx, eax`

:0040EA0A 0F8C69FFFFFF            `jl 0040E979` <====取够 V3 位没有?

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0040E973(C)

|

:0040EA10 8D45F4                  `lea eax, dword ptr [ebp-0C]`

:0040EA13 E8C436FFFF              `call 004020DC` <====取上面的运算结果 NEWBIT

:0040EA18 50                      `push eax`

:0040EA19 E82A912A00              `call 006B7B48` <====转为十六进制

:0040EA1E 59                      `pop ecx`

:0040EA1F 898570FFFFFF            `mov dword ptr [ebp+FFFFFF70], eax` <====保存 NEWBIT

:0040EA25 C7458485000000          `mov [ebp-7C], 00000085` <====变量 V5

:0040EA2C C745806F000000          `mov [ebp-80], 0000006F` <====变量 V6

:0040EA33 C7857CFFFFFF42000000    `mov dword ptr [ebp+FFFFFF7C], 00000042` <====变量 V7

:0040EA3D C78578FFFFFFA6000000    `mov dword ptr [ebp+FFFFFF78], 000000A6` <====变量 V8

:0040EA47 33D2                    `xor edx, edx`

:0040EA49 8955A0                  `mov dword ptr [ebp-60], edx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address: <====循环 6 次计算注册码

| :0040EBEE(C)

|

:0040EA4C 8B4D84                  `mov ecx, dword ptr [ebp-7C]` <====取 V5

:0040EA4F 41                      `inc ecx` <====V5+1

:0040EA50 8B8570FFFFFF            `mov eax, dword ptr [ebp+FFFFFF70]` 取 NEWBIT

:0040EA56 33D2                    `xor edx, edx`

:0040EA58 F7F1                    `div ecx` <====NEWBIT MOD (V5+1)

:0040EA5A 8BCA                    `mov ecx, edx`

:0040EA5C 8B4580                  `mov eax, dword ptr [ebp-80]` <====取 V6

:0040EA5F 40                      `inc eax` <====V6+1

:0040EA60 50                      `push eax`

:0040EA61 8B8570FFFFFF            `mov eax, dword ptr [ebp+FFFFFF70]` <====取 NEWBIT

:0040EA67 5A                      `pop edx`

:0040EA68 8BDA                    `mov ebx, edx`

:0040EA6A 33D2                    `xor edx, edx`

:0040EA6C F7F3                    `div ebx` <====NEWBIT MOD (V6+1)

:0040EA6E 03CA                    `add ecx, edx` <====(NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))

:0040EA70 8B857CFFFFFF            `mov eax, dword ptr [ebp+FFFFFF7C]` <====取 V7

:0040EA76 40                      `inc eax` <====V7+1

:0040EA77 50                      `push eax`

:0040EA78 8B8570FFFFFF            `mov eax, dword ptr [ebp+FFFFFF70]` <====取 NEWBIT

:0040EA7E 5A                      `pop edx`

:0040EA7F 8BDA                    `mov ebx, edx`

:0040EA81 33D2                    `xor edx, edx`

:0040EA83 F7F3                    `div ebx` <====NEWBIT MOD (V7+1)

:0040EA85 03CA                    `add ecx, edx` <====(NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))+(NEWBIT MOD (V7+1))

:0040EA87 8BC1                    `mov eax, ecx`

:0040EA89 8B9578FFFFFF            `mov edx, dword ptr [ebp+FFFFFF78]` <====取 V8

:0040EA8F 42                      `inc edx` <====V8+1

:0040EA90 8BCA                    `mov ecx, edx`

:0040EA92 33D2                    `xor edx, edx`

:0040EA94 F7F1                    `div ecx` <====((NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))+(NEWBIT MOD (V7+1))) MOD (V8+1)

:0040EA96 895588                  `mov dword ptr [ebp-78], edx` <====保存结果(即计算出来的注册码)

:0040EA99 83458402                `add dword ptr [ebp-7C], 00000002` <====V5:=V5+2

:0040EA9D 836D8007                `sub dword ptr [ebp-80], 00000007` <====V6:=V6-7

:0040EAA1 83857CFFFFFF05          `add dword ptr [ebp+FFFFFF7C], 00000005` <====V7:=V7+5

:0040EAA8 83AD78FFFFFF03          `sub dword ptr [ebp+FFFFFF78], 00000003` <====V8:=V8-3

:0040EAAF 8B4588                  `mov eax, dword ptr [ebp-78]`

:0040EAB2 83F830                  `cmp eax, 00000030`

:0040EAB5 7C08                    `jl 0040EABF`

:0040EAB7 8B4588                  `mov eax, dword ptr [ebp-78]`

:0040EABA 83F839                  `cmp eax, 00000039`

:0040EABD 7E20                    `jle 0040EADF` <====注册码是不是数字?

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0040EAB5(C)

|

:0040EABF 8B5588                  `mov edx, dword ptr [ebp-78]`

:0040EAC2 83FA41                  `cmp edx, 00000041`

:0040EAC5 7C08                    `jl 0040EACF`

:0040EAC7 8B4D88                  `mov ecx, dword ptr [ebp-78]`

:0040EACA 83F95A                  `cmp ecx, 0000005A`

:0040EACD 7E10                    `jle 0040EADF` <====注册码是不是大写字母?

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0040EAC5(C)

|

:0040EACF 8B4588                  `mov eax, dword ptr [ebp-78]`

:0040EAD2 83F861                  `cmp eax, 00000061`

:0040EAD5 7C63                    `jl 0040EB3A`

:0040EAD7 8B5588                  `mov edx, dword ptr [ebp-78]`

:0040EADA 83FA7A                  `cmp edx, 0000007A`

:0040EADD 7F5B                    `jg 0040EB3A` <====注册码是不是小写

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :0040EABD(C), :0040EACD(C)

|

:0040EADF 66C745B86800            `mov [ebp-48], 0068`

:0040EAE5 8D45D8                  `lea eax, dword ptr [ebp-28]`

:0040EAE8 E8AB35FFFF              `call 00402098`

:0040EAED 50                      `push eax`

:0040EAEE FF45C4                  `inc [ebp-3C]`

:0040EAF1 8A5588                  `mov dl, byte ptr [ebp-78]`

:0040EAF4 8D45DC                  `lea eax, dword ptr [ebp-24]`

:0040EAF7 E8800C2B00              `call 006BF77C`

:0040EAFC 8BD0                    `mov edx, eax`

:0040EAFE FF45C4                  `inc [ebp-3C]`

:0040EB01 8D45F8                  `lea eax, dword ptr [ebp-08]`

:0040EB04 59                      `pop ecx`

:0040EB05 E8960E2B00              `call 006BF9A0`

:0040EB0A 8D55D8                  `lea edx, dword ptr [ebp-28]`

:0040EB0D 8D45F8                  `lea eax, dword ptr [ebp-08]`

:0040EB10 E8630E2B00              `call 006BF978`

:0040EB15 FF4DC4                  `dec [ebp-3C]`

:0040EB18 8D45D8                  `lea eax, dword ptr [ebp-28]`

:0040EB1B BA02000000              `mov edx, 00000002`

:0040EB20 E8230E2B00              `call 006BF948`

:0040EB25 FF4DC4                  `dec [ebp-3C]`

:0040EB28 8D45DC                  `lea eax, dword ptr [ebp-24]`
