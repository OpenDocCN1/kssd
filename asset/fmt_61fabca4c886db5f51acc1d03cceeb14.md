
# Net Vampire v3.3

> 原文：[`www.pediy.com/kssd/pediy03/forum1265.htm`](https://www.pediy.com/kssd/pediy03/forum1265.htm)

这个软件已经出来很久了(最新版 `v4.0` 还没出来),`keygen` 满天飞，不知以前有没有人写过它的破解，反正我没看到

既然要学 `crack` 就不想用别人的 `keygen`, 就把它当作加入`[FCG]`后的第一个作业吧。

首先用 `filemonitor` 得知软件运行时会调用 `system` 目录下的 `advert.dll` (估计是广告条)

用 `W32Dasm` 载入，选上 `DLL Load Brk` 的选项，然后一直 `Run` 到开始载入 `advert.dll`, 记下地址 `00471605`

```
* Reference To: kernel32.LoadLibraryA, Ord:0000h
  |
:00471600 E86348F9FF                  Call 00405E68
:00471605 8B55FC                      mov edx, dword ptr [ebp-04]<-----------停在这里
:00471608 8982D8000000                mov dword ptr [edx+000000D8], eax
:0047160E 8B45FC                      mov eax, dword ptr [ebp-04]
:00471611 83B8D800000000              cmp dword ptr [eax+000000D8], 00000000
:00471618 7524                        jne 0047163E
:0047161A 6A00                        push 00000000
:0047161C 668B0DB8184700              mov cx, word ptr [004718B8]
:00471623 B201                        mov dl, 01
```

往上看，哪里可以跳过这个 call，这里省略几个 jump, 来到这里

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049AF47(U)
|
:0049AF5D E87A0FFFFF                  call 0048BEDC<------------------估计是比较，跟进去
:0049AF62 A1A0C94A00                  mov eax, dword ptr [004AC9A0]
:0049AF67 833800                      cmp dword ptr [eax], 00000000
:0049AF6A 7521                        jne 0049AF8D<--------------------这里一定要跳，否则就到刚才的地方了
:0049AF6C 8B45FC                      mov eax, dword ptr [ebp-04]
:0049AF6F 8B800C050000                mov eax, dword ptr [eax+0000050C]
:0049AF75 E89271FDFF                  call 0047210C
:0049AF7A 8B45FC                      mov eax, dword ptr [ebp-04]
:0049AF7D 8B800C050000                mov eax, dword ptr [eax+0000050C]
:0049AF83 E89C6CFDFF                  call 00471C24
:0049AF88 E980000000                  jmp 0049B00D
```

记下地址 `0049af5d`, 取消断点，运行程序，输入用户名和注册码，关闭程序。

启用 `trw2k` , 下 `bpx ****:0049af5d` , 运行 `Net Vampire v3.3` 来到这里

```
* Referenced by a CALL at Address:
|:0049AF5D 
|
:0048BEDC 55                          push ebp
:0048BEDD 8BEC                        mov ebp, esp
:0048BEDF 6A00                        push 00000000
:0048BEE1 6A00                        push 00000000
:0048BEE3 6A00                        push 00000000
:0048BEE5 53                          push ebx
:0048BEE6 56                          push esi
:0048BEE7 57                          push edi
:0048BEE8 33C0                        xor eax, eax
:0048BEEA 55                          push ebp
:0048BEEB 68B7BF4800                  push 0048BFB7
:0048BEF0 64FF30                      push dword ptr fs:[eax]
:0048BEF3 648920                      mov dword ptr fs:[eax], esp
:0048BEF6 33C0                        xor eax, eax
:0048BEF8 55                          push ebp
:0048BEF9 6860BF4800                  push 0048BF60
:0048BEFE 64FF30                      push dword ptr fs:[eax]
:0048BF01 648920                      mov dword ptr fs:[eax], esp
:0048BF04 8D45FC                      lea eax, dword ptr [ebp-04]
:0048BF07 50                          push eax
:0048BF08 33C9                        xor ecx, ecx

* Possible StringData Ref from Code Obj ->"vE9nT$Ma."<------------什么东西？
  |
:0048BF0A BAD0BF4800                  mov edx, 0048BFD0
:0048BF0F A164DA4A00                  mov eax, dword ptr [004ADA64]<--假注册码
:0048BF14 E81B57FDFF                  call 00461634<-----------------核心计算，要跟进去(1)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048BEC7(C)
|
:0048BF19 8B45FC                      mov eax, dword ptr [ebp-04]<-----由假注册码算出的
:0048BF1C 50                          push eax                          用户名 2
:0048BF1D 8D45F8                      lea eax, dword ptr [ebp-08]
:0048BF20 50                          push eax
:0048BF21 8D45F4                      lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"        "
  |
:0048BF24 B9E4BF4800                  mov ecx, 0048BFE4<---------------9 个空格
:0048BF29 8B1560DA4A00                mov edx, dword ptr [004ADA60]<---输入的用户名，用户名 1
:0048BF2F E8187EF7FF                  call 00403D4C
:0048BF34 8B45F4                      mov eax, dword ptr [ebp-0C]
:0048BF37 B909000000                  mov ecx, 00000009
:0048BF3C BA01000000                  mov edx, 00000001
:0048BF41 E8BE7FF7FF                  call 00403F04
:0048BF46 8B55F8                      mov edx, dword ptr [ebp-08]<---用户名 1 的前 9 位
:0048BF49 58                          pop eax
:0048BF4A E8C17EF7FF                  call 00403E10<--------比较用户名 1 和用户名 2
:0048BF4F 7405                        je 0048BF56<-----------跳？一定要跳!(此处改 jmp 则爆破)
:0048BF51 E89EEDF7FF                  call 0040ACF4
```

```
* Referenced by a CALL at Addresses:
|:00467EB2  , :00468192  , :00488EBF  , :0048BB84  , :0048BBDF 
|:0048BC16  , :0048BD4E  , :0048BD8D  , :0048BDCC  , :0048BF14 
|:004A2A4A  , :004A2E42  , :004A52CA  , :004A591E 
|
:00461634 55                          push ebp<--------------------------(1)
:00461635 8BEC                        mov ebp, esp
:00461637 83C4D8                      add esp, FFFFFFD8
:0046163A 53                          push ebx
:0046163B 33DB                        xor ebx, ebx
:0046163D 895DD8                      mov dword ptr [ebp-28], ebx<---0
:00461640 884DF7                      mov byte ptr [ebp-09], cl<-----0
:00461643 8955F8                      mov dword ptr [ebp-08], edx<---"vE9nT$Ma."
:00461646 8945FC                      mov dword ptr [ebp-04], eax<---假注册码
:00461649 8B45FC                      mov eax, dword ptr [ebp-04]
:0046164C E86328FAFF                  call 00403EB4
:00461651 8B45F8                      mov eax, dword ptr [ebp-08]
:00461654 E85B28FAFF                  call 00403EB4
:00461659 33C0                        xor eax, eax
:0046165B 55                          push ebp
:0046165C 6820184600                  push 00461820
:00461661 64FF30                      push dword ptr fs:[eax]
:00461664 648920                      mov dword ptr fs:[eax], esp
:00461667 8B4508                      mov eax, dword ptr [ebp+08]
:0046166A E81524FAFF                  call 00403A84
:0046166F 837DFC00                    cmp dword ptr [ebp-04], 00000000
:00461673 0F8484010000                je 004617FD
:00461679 837DF800                    cmp dword ptr [ebp-08], 00000000
:0046167D 0F847A010000                je 004617FD
:00461683 33C0                        xor eax, eax
:00461685 8945F0                      mov dword ptr [ebp-10], eax
:00461688 807DF700                    cmp byte ptr [ebp-09], 00
:0046168C 0F84AD000000                je 0046173F<----------------------跳，核心(2)
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046168C(C)
|
:0046173F 55                          push ebp<---------------------------(2)
:00461740 8D45D8                      lea eax, dword ptr [ebp-28]
:00461743 50                          push eax
:00461744 B902000000                  mov ecx, 00000002<--------------------取 2 位
:00461749 BA01000000                  mov edx, 00000001<--------------------从第 1 位开始
:0046174E 8B45FC                      mov eax, dword ptr [ebp-04]<---假注册码
:00461751 E8AE27FAFF                  call 00403F04
:00461756 8B45D8                      mov eax, dword ptr [ebp-28]<---第 1、2 位
```
