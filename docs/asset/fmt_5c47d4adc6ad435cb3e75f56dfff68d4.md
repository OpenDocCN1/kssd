
:00506ACC E8CFF1FFFF `call 00505CA0`

:00506AD1 8B45F0 `mov eax, dword ptr [ebp-10]`

:00506AD4 E84FF9FFFF `call 00506428`

:00506AD9 83F8FF `cmp eax, FFFFFFFF`

:00506ADC 741A `je 00506AF8`

:00506ADE BB91080000 `mov ebx, 00000891`

* Possible StringData Ref from Data Obj ->"?"

:00506AE3 BA10525200 `mov edx, 00525210`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00506AF6(C)

:00506AE8 3B02 `cmp eax, dword ptr [edx]`

:00506AEA 7506 `jne 00506AF2`

:00506AEC C645FF01 `mov [ebp-01], 01`

:00506AF0 EB06 `jmp 00506AF8`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00506AEA(C)

:00506AF2 83C204 `add edx, 00000004`

:00506AF5 4B `dec ebx`

:00506AF6 75F0 `jne 00506AE8`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :00506537(U), :00506548(C), :0050655A(C), :0050656C(C), :0050657E(C)
| :00506590(C), :005065A2(C), :005065B4(C), :005065C6(C), :005065D8(C)
| :005065EA(C), :005065FC(C), :0050660E(C), :00506620(C), :00506632(C)
| :00506644(C), :00506656(C), :00506668(C), :0050667A(C), :0050668C(C)
| :0050669E(C), :005066B0(C), :005066C2(C), :005066D4(C), :005066E6(C)
| :005066F8(C), :0050670A(C), :0050671C(C), :0050672E(C), :00506740(C)
| :00506752(C), :00506764(C), :00506776(C), :00506788(C), :0050679A(C)
| :005067AC(C), :005067BE(C), :005067D0(C), :005067E2(C), :00506801(C)
| :00506816(C), :0050682B(C), :0050683E(C), :00506853(C), :00506868(C)
| :0050687D(C), :00506892(C), :005068A7(C), :005068BC(C), :005068D1(C)
| :005068E6(C), :005068FB(C), :00506910(C), :00506925(C), :0050693A(C)
| :0050694F(C), :00506964(C), :00506979(C), :0050698E(C), :005069A3(C)
| :005069B8(C), :005069CD(C), :005069E2(C), :005069F7(C), :00506AB6(C)
| :00506ADC(C), :00506AF0(U)

:00506AF8 33C0 `xor eax, eax` //失败的话，则 `eax=0`

:00506AFA 5A `pop edx`

:00506AFB 59 `pop ecx`

:00506AFC 59 `pop ecx`

:00506AFD 648910 `mov dword ptr fs:[eax], edx`

:00506B00 681A6B5000 `push 00506B1A`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00506B18(U)

:00506B05 8D45F0 `lea eax, dword ptr [ebp-10]`

:00506B08 BA03000000 `mov edx, 00000003`

:00506B0D E882D3EFFF `call 00403E94`

:00506B12 C3 `ret`

程序的修改很简单：在 `call 005064f8` 开始处即：

:005064F8 55 `push ebp`

:005064F9 8BEC `mov ebp, esp`

:005064FB 6A00 `push 00000000`

:005064FD 6A00 `push 00000000`

直接改成 2 条语句：① `mov eax, 1`

② `retn`

至此程序完全破解，无论你用什么名注册均成功，时间如何变化也不会过期了。此法对 `v1.9 pro` 同样适用。

以上有不妥之处请各位一定要指出。

在此谢谢 `blowfish` 的提示。 4dd466d0e1e0cc123afbe504da2462ca
