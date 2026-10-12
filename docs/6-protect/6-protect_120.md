
# 蒙泰 5.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue109.htm`](https://www.pediy.com/kssd/pediy04/kanxue109.htm)

又到杀狗的时间了，呵呵，这次的疯狗的是什么呢？小弟是广告界的，当然先拿喷绘软件来开刀了，`蒙泰 5.0`应该是国内用得最多的一个喷绘软件吧，好，这次就拿它来试刀了。

工具：`trw2000`
`w32dasm8.93` 黄金版
`hview`

`蒙泰`在运行时如果没有加密狗，就会弹出一个对话框，好，我们就从这个对话框入手。运行 `trw2000`，然后运行 `蒙泰`，会出现对话框，切入 `trw2000`(`ctrl+N`)，下断点 `bpx enddialog`，返回主程序，按下“确定”按钮，Boom，被拦下来的，暂停断点(`BD *`)，接着就一直按 `F12` 和 `F10`，直到返回到下面的代码处：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00490D53(C)
|
:00490D5D 8B1B                  mov ebx, dword ptr [ebx]
:00490D5F 85DB                  test ebx, ebx
:00490D61 75EC                  jne 00490D4F

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00490D47(C), :00490D4D(C)
|
:00490D63 833D80AC630007        cmp dword ptr [0063AC80], 00000007  <=====我们想办法让[0063AC80]不等于 7
:00490D6A 750E                  jne 00490D7A
:00490D6C 833D20C6650000        cmp dword ptr [0065C620], 00000000
:00490D73 7505                  jne 00490D7A
:00490D75 E836FFFFFF            call 00490CB0  <=====出错对话框

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00490D6A(C), :00490D73(C)
|
:00490D7A B801000000            mov eax, 00000001  <=====返回到这里
```

我们往上看，有两条跳转指令，程序是通过地址 `63ac80` 和 `65c620` 的内容来决定是否显示出错对话框的，好，退出 `蒙泰`，我们再下断点 `bpm 63ac80`，看会断在什么地方：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B67AC(C)
|
:004B67CD C743109A010000        mov [ebx+10], 0000019A
:004B67D4 C743145C000000        mov [ebx+14], 0000005C
:004B67DB 33D2                  xor edx, edx
:004B67DD 895318                mov dword ptr [ebx+18], edx
:004B67E0 C7431C07000000        mov [ebx+1C], 00000007  <=====这里就是给 63ac80 赋值的地方,当走到这里的时候就 GAME OVER 了,所以我们往上看什么地方可以跳过这里
:004B67E7 68606D0000            push 00006D60  <======中断在此
:004B67EC E85EF10E00            call 005A594F
```

我们往上看，是 `4B67AC` 这个地址调用的：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B6792(C)
|
:004B67A4 83BC240402000004     cmp dword ptr [esp+00000204], 00000004  <=====当[esp+00000204]小于 4 时,就会跳到 4B67CD,如果没有狗,这个地址的内容为 0,同时这里也是判断版本号的地方,当大于等于 4 时,就是通用版
:004B67AC 7C1F                  jl 004B67CD
:004B67AE C705D443630001000000  mov dword ptr [006343D4], 00000001
:004B67B8 33C0                  xor eax, eax
:004B67BA C743148C000000        mov [ebx+14], 0000008C
:004B67C1 894318                mov dword ptr [ebx+18], eax
:004B67C4 C7431C05000000        mov [ebx+1C], 00000005  <====[63AC80]=5
:004B67CB EB55                  jmp 004B6822
```

再往上看地址 `4B6792`：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B66EB(U)
|
:004B6757 83BC24040200000C      cmp dword ptr [esp+00000204], 0000000C  <=====大于等于 0C 时,是专业版
:004B675F 7C0C                  jl 004B676D
:004B6761 C7431C01000000        mov [ebx+1C], 00000001
:004B6768 E9B5000000            jmp 004B6822

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B675F(C)
|
:004B676D 83BC24040200000A      cmp dword ptr [esp+00000204], 0000000A  <=====大于等于 0A 时,是专业版 S(哪位朋友知道专业版 S 和专业版有什么区别)
:004B6775 7C13                  jl 004B678A
:004B6777 C74314C2010000        mov [ebx+14], 000001C2
:004B677E C7431C02000000        mov [ebx+1C], 00000002
:004B6785 E998000000            jmp 004B6822

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B6775(C)
|
:004B678A 83BC240402000006      cmp dword ptr [esp+00000204], 00000006  <=====当大于等于 6 时,是标准版
:004B6792 7C10                  jl 004B67A4
:004B6794 C743142C010000        mov [ebx+14], 0000012C
:004B679B C7431C04000000        mov [ebx+1C], 00000004
:004B67A2 EB7E                  jmp 004B6822
```

好了，我们快要接近目标了，再往上看：

```
:004B6722 51                   push ecx
:004B6723 E8EAE11000            call 005C4912
:004B6728 85C0                  test eax, eax  <=====注意了,这里就是关键的地方,当没有狗时,EAX=0
:004B672A 740E                  je 004B673A
:004B672C 8B842408020000        mov eax, dword ptr [esp+00000208]
:004B6733 A320C66500            mov dword ptr [0065C620], eax
:004B6738 EB09                  jmp 004B6743

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004B6702(C), :004B672A(C)
|
:004B673A 33D2                  xor edx, edx  <=====EDX=0
:004B673C 89942404020000        mov dword ptr [esp+00000204], edx  <=====放标志了,[ESP+00000204]=0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B6738(U)
|
:004B6743 6800010000            push 00000100
:004B6748 6A00                  push 00000000
:004B674A 8D4C240C              lea ecx, dword ptr [esp+0C]
:004B674E 51                   push ecx
:004B674F E8A8EF1000            call 005C56FC
:004B6754 83C40C                add esp, 0000000C
```

好了，我们找到关键的地方了，我们要让 `[ESP+00000204]` 不为 0，我的改法是：

```
004B6722 51                   push ecx
:004B6723 E8EAE11000            call 005C4912
:004B6728 85C0                  test eax, eax  <=====改为 push 0c(我需要专业版嘛,如果你要通用版,就用 04 好了,我想你不会那么笨吧,呵呵)
:004B672A 740E                  je 004B673A   <=====改为 JMPS 004B673A
:004B672C 8B842408020000        mov eax, dword ptr [esp+00000208]
:004B6733 A320C66500            mov dword ptr [0065C620], eax
:004B6738 EB09                  jmp 004B6743

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004B6702(C), :004B672A(C)
|
:004B673A 33D2                  xor edx, edx  <=====改为 POP EDX; NOP(POP 指令只有一个字节,所以要加 NOP 补足)
:004B673C 89942404020000        mov dword ptr [esp+00000204], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B6738(U)
|
:004B6743 6800010000            push 00000100
:004B6748 6A00                  push 00000000
:004B674A 8D4C240C              lea ecx, dword ptr [esp+0C]
:004B674E 51                   push ecx
:004B674F E8A8EF1000            call 005C56FC
:004B6754 83C40C                add esp, 0000000C
```

好长啊，终于写完了。哪位朋友对解 `HASP` 外壳狗有经验的，能否给我一份破解过程呢？小弟很需要这方面的知识的，谢谢了。

本文章可以自由张贴，但请保持此文章的完整性，谢谢 `4dd466d0e1e0cc123afbe504da2462ca`
