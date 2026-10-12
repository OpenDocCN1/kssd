
# JDPE 2.01 (铁甲)

> 原文：[`www.pediy.com/kssd/pediy03/forum1071.htm`](https://www.pediy.com/kssd/pediy03/forum1071.htm)

`JDPE` 2.01 (铁甲)的脱壳

`by 6767 [BCG]`

工具：`SoftIce`，`IceDump`

地址：[`http://go5.163.com/hmjd/xz/jdpe201.zip`](http://go5.163.com/hmjd/xz/jdpe201.zip) (52K)

简述：`PE`文件的加壳软件，与 `Ding Boy` 的 `DUPE`类似，但水平要差的多。

目标：脱壳

难度：`1/5`

先把我遇到的问题写出来：如何写`JDPE`的脱壳脚本？

它没有对 `Import Table` 加密，不知道有没有反跟踪手段，解压缩比较容易。

研究脱壳，我们要选用记事本(感谢 `MICRO$OFT`)，至少入口点我们知道(RVA)：`10CC`。

用 `JDPE` 对记事本加壳，选项里选择“设注册码”和“到期时间”，随便填入些信息，加壳。用 `ProcDump` 的 `PeEdit`功能看一下：`Image Base=400000`, `Image Size=E000`

用 `Symble Loader` 载入，接下来用 `SI`来跟，小心的设断点以跳出循环，过程很容易，这里就不详述了。

`* Referenced by a CALL at Address:`

`:0040D001`
`<- Entry Point`

```
:0040D006 5D                     pop ebp
:0040D007 8BD5                   mov edx, ebp
:0040D009 81EDE21C4000           sub ebp, 00401CE2
:0040D00F 2B9501244000           sub edx, dword ptr [ebp+00402401]
:0040D015 81EA06000000           sub edx, 00000006
:0040D01B 899505244000           mov dword ptr [ebp+00402405], edx
:0040D021 83BD0924400000         cmp dword ptr [ebp+00402409], 00000000
:0040D028 0F8538020000           jne 0040D266
:0040D02E C7850924400001000000   mov dword ptr [ebp+00402409], 00000001
:0040D038 B90C060000             mov ecx, 0000060C
:0040D03D 8DB5341D4000           lea esi, dword ptr [ebp+00401D34]
```

`>>>>` 下面这个循环对紧随其后的 `40D058`到 `40D058+60C` 之间的代码解码

```
   :0040D043 8A8500244000          mov al, byte ptr [ebp+00402400]
   :0040D049 8A1E                  mov bl, byte ptr [esi]
   :0040D04B 32C3                  xor al, bl
   :0040D04D 8806                  mov byte ptr [esi], al
   :0040D04F 889D00244000          mov byte ptr [ebp+00402400], bl
   :0040D055 46                    inc esi
   :0040D056 E2EB                  loop 0040D043
```

`>>>>` 下面的原来都是乱码，现在是正常的：

```
:0040D058 9C                       pushfd
:0040D059 58                       pop eax
:0040D05A F6C401                   test ah, 01
:0040D05D 7406                     je 0040D065
:0040D05F FFA535224000             jmp dword ptr [ebp+00402235]
.....
:0040D30B 8BC8                     mov ecx, eax
:0040D30D 8B8595224000             mov eax, dword ptr [ebp+00402295]
:0040D313 8B9D91224000             mov ebx, dword ptr [ebp+00402291]
:0040D319 8DB5C1224000             lea esi, dword ptr [ebp+004022C1]
:0040D31F FFD1                     call ecx     <- 这里是对运行状态的检查，如：是否过期、注册码是否正确
                                          <- 如果出口 EAX=0 则表示失败
:0040D321 85C0                     test eax, eax    <- 若跟踪到此 EAX=0，则用 r EAX 1 命令
:0040D323 0F846AFFFFFF             je 0040D293
:0040D329 EB47                     jmp 0040D372    <- 向下跳是我们所需要的
:0040D32B EB45                     jmp 0040D372
.....
:0040D352 8BC8                     mov ecx, eax
:0040D354 8B8595224000             mov eax, dword ptr [ebp+00402295]
:0040D35A 8B9D99224000             mov ebx, dword ptr [ebp+00402299]
:0040D360 8DB5C1224000             lea esi, dword ptr [ebp+004022C1]
:0040D366 FFD1                     call ecx     <- 同上，也是检查
:0040D368 85C0                     test eax, eax
:0040D36A 0F8423FFFFFF             je 0040D293
:0040D370 EB00                     jmp 0040D372
```

`* Referenced by Jump at Addresses:`

```
:0040D372 8B9505244000             mov edx, dword ptr [ebp+00402405]   <- ImageBase
:0040D378 8B8535224000             mov eax, dword ptr [ebp+00402235]   <- OEip
:0040D37E 03C2                     add eax, edx
:0040D380 5D                       pop ebp
:0040D381 FFE0                     jmp eax      <- 跳到 OEip 去
```

我们只要跟踪到 `0040D381`，按一次 `F10`到原程序的入口，就能 dump 得到脱壳文件：

```
/pedump 400000 10cc d:dumped.exe
```

运行这个文件，没有任何干扰窗口，成功。

总结：

与 `UPX` 差不多，用 `Symble Loader` 载入后，先跟踪到一开始的那个 `Loop` 循环结束，然后用 `Ctrl+PageDown` 向下翻页，找到后对其偏移址下 `G` 命令，再是一、二次 `F10`到 `OEip`，下 `Pedump`就可以了。

问题：

我写了 `ProcDump` 的脚本，如下，但不成功：

```
[JDPE]

L1=LOOK 46,E2
L2=ADD 3
L3=BP
L4=OBJR
L5=LOOK 5D,FF
L6=BP
L7=WALK
L8=WALK
L9=STEP
OPTL1=00000000
OPTL2=01010001
OPTL3=01010001
OPTL4=00020000
OPTL5=00000000
```

不知道该怎样写，哪位有时间看一下。 `4dd466d0e1e0cc123afbe504da2462ca`
