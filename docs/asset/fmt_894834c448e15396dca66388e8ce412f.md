
# Tag&Rename v.1.9.6

> 原文：[`www.pediy.com/kssd/pediy04/kanxue002.htm`](https://www.pediy.com/kssd/pediy04/kanxue002.htm)

注册 `Tag&Rename` `v.1.9.6`

这个东西发布已经很长时间了，而破解也早就做出来了，由于脱壳部分一直没有搞定所以到现在才放出(直到现在还没搞定，谁来指点我一下呀)，当初是找 `hying` 大哥脱壳的，在这里说声多谢啦:)。

## 1.unpack(2001.9.8)

首先是脱壳部分，这里介绍一种高速寻找 `aspr` 入口的方法(寻找时间应该小于 1 分钟,不过只对 `aspr` 有效，对其他的壳无效)，这个方法的可靠性还有待进一步验证。

**方法：**

执行 `tag&rename.exe`，用 `prodump` 选 `dump(full)` 脱壳,存为 `dump.exe`。接着用 `winhex` 打开 `dump.exe`，选择搜索文本，填 `runtime`,执行搜索，搜到后,向前找到离 `runtime` 最近的机器码为 `55 8B EC` 的地方就是程序的 `oep`，而在 `tag&rename` 里这个位置在 `offset:00156000` 处，用 `peditor` 的 `flc` 功能将它转换为 `virtual address` 就是 `556a00`----`oep`。

接着修复 `import table`

以下操作必须在 `tag&rename` 的地盘里

修复 `import table` 部分

1. `0015C230` `KERNEL32.dll` `0151` `GetCommandLineA`
2. `0015C380` `KERNEL32.dll` `036E` `FreeResource`
3. `0015C3B8` `KERNEL32.dll` `01E9` `GetVersion`
4. `0015C410` `KERNEL32.dll` `0163` `GetCurrentProcessId`
5. `0015C41C` `KERNEL32.dll` `00E2` `LockRescouce`

还是那句话，`ImportREC` 一般只有这几个函数认不出来，以后根据情况照抄就可以了，一定要参考周围的，以免抄错。

修复 `it` 后，运行 `t&r` 报错 `Initialization Error`，所以就载入看看。发现错误在这个地方：

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:

|:005569BD(C)

:00556A0E E86904EBFF                call 00406E7C

:00556A13 B8A85F5500                mov eax, 00555FA8

:00556A18 E8FBE7FFFF                call 00555218

:00556A1D 33C0                      xor eax, eax

:00556A1F 55                        push ebp

:00556A20 68706A5500                push 00556A70

:00556A25 64FF30                    push dword ptr fs:[eax]

:00556A28 648920                    mov dword ptr fs:[eax], esp

:00556A2B A1F89D5500                mov eax, dword ptr [00559DF8]

:00556A30 8B00                      mov eax, dword ptr [eax]

:00556A32 E879C3EEFF                call 00442DB0

:00556A37 A108A05500                mov eax, dword ptr [0055A008]

:00556A3C 8B00                      mov eax, dword ptr [eax]

:00556A3E E81DFBF1FF                call 00476560

:00556A43 FF15CCBC5500              call dword ptr [0055BCCC]      -->call 556120

这个只能用原版跟进去看看啦，发现这里最后 `call` 的是 `556120`，在对应的位置改一下就好了。`/12fc8ac`

:00556A49 A1F89D5500                mov eax, dword ptr [00559DF8]

:00556A4E 8B00                      mov eax, dword ptr [eax]

最后把 `dump.exe` 的 `ep` 改成 `156a00`。

修复 `it` 后，运行 `t&r` 报错 `Initialization Error`，所以就载入看看啦。发现错误在这个地方：

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:

|:005569BD(C)

|

:00556A0E E86904EBFF                call 00406E7C

:00556A13 B8A85F5500                mov eax, 00555FA8

:00556A18 E8FBE7FFFF                call 00555218

:00556A1D 33C0                      xor eax, eax

:00556A1F 55                        push ebp

:00556A20 68706A5500                push 00556A70

:00556A25 64FF30                    push dword ptr fs:[eax]

:00556A28 648920                    mov dword ptr fs:[eax], esp

:00556A2B A1F89D5500                mov eax, dword ptr [00559DF8]

:00556A30 8B00                      mov eax, dword ptr [eax]

:00556A32 E879C3EEFF                call 00442DB0

:00556A37 A108A05500                mov eax, dword ptr [0055A008]

:00556A3C 8B00                      mov eax, dword ptr [eax]

:00556A3E E81DFBF1FF                call 00476560

:00556A43 FF15CCBC5500              call dword ptr [0055BCCC]      -->call 5562b8/556120?

这个只能用原版跟进去看看啦，发现这里最后 `call` 的是 `5562b8`，在对应的位置改一下就好了。

:00556A49 A1F89D5500                mov eax, dword ptr [00559DF8]

:00556A4E 8B00                      mov eax, dword ptr [eax]

至此运行正常，脱壳任务完成了，继续 `crack` 工作。

## 2.crack(07/01/01)

这个判断的模式一直没变过。选注册，名字任意，注册码 `78787878`，`ctrl+d` 下 `s 30:0 l ffffffff '78787878'`，`bpm` 搜到的地址。跟踪到如下：

**(1)**

:0045C8D3 A154034100                mov eax, dword ptr [00410354]

:0045C8D8 E81B66FAFF                call 00402EF8

:0045C8DD 8945F8                    mov dword ptr [ebp-08], eax

:0045C8E0 8D45F8                    lea eax, dword ptr [ebp-08]

:0045C8E3 E87CFEFFFF                call 0045C764

:0045C8E8 8B55FC                    mov edx, dword ptr [ebp-04]

:0045C8EB 8B45F8                    mov eax, dword ptr [ebp-08]

:0045C8EE 8B08                      mov ecx, dword ptr [eax]

:0045C8F0 FF5150                    call [ecx+50]

:0045C8F3 40                        inc eax

:0045C8F4 7402->9090                je 0045C8F8->nop      //offset:0005BEF4

:0045C8F6 B301                      mov bl, 01

最难的还是无提示的 `crc` 校验，这个是什么时候加上的我还真没注意过，花了我不少时间，这个没法细说，花时间跟吧。起初选择 `nop` 掉 `55609f-5560A0`，没有成功，后来就接着跟踪看他如何处理这段代码，没多久就迷失在代码迷宫里:(。干脆直接打补丁吧，最后的出路了。用 `topo1.2` 找出可用空间的地址 `0066750A/offset:0015569A`，把那段用来做 `crc` 的数据直接复制过去好啦

**(2)**

:0055609A B904000000->E96B141100  mov ecx, 00000004->jmp 0066750A

:0055609F F3                        repz

:005560A0 A5                        movsd

:005560A1 33C0                      xor eax, eax

:005560A3 5A                        pop edx

:005560A4 59                        pop ecx

:005560A5 59                        pop ecx

:005560A6 648910                    mov dword ptr fs:[eax], edx

:005560A9 68BE605500                push 005560BE

```
66750a c705bcbc5500879ec943 mov dword ptr [55bcbc],43c99e87  //offset:0015569A

667514 c705c0bc55006b0c9ee5 mov dword ptr [55bcc0],e59e0c6b

66751e c705c4bc55008d9c5435 mov dword ptr [55bcc4],35549c8d

667528 c705c8bc5500b0212994 mov dword ptr [55bcc8],942921b0

667532 e96aebeeff                 jmp 5560a1
```

完活，好像能正常运行了，有错误的话请指出:)

最后控制 `trg&rename` 时间的东西就是这个，用 `norton register tracker` 监测到的

```
[HKEY_LOCAL_MACHINE\SOFTWARE\CLASSES\CLSID\{D5BB95E5-1303-615A-7237-C9FA207D1E21}]

@="YWNPK6ipEx/4yXKh"

删掉就可以重新计时

REGEDIT4

[HKEY_LOCAL_MACHINE\SOFTWARE\CLASSES\CLSID\{D5BB95E5-1303-615A-7237-C9FA207D1E21}]

@=""
```

或用这个补丁，把它存成 `reset.reg`

汉化已拜托宏伟搞定了 `http://hanhua.on.net.cn/default.asp` 在这下

顺便贴个 `TagRename` `Ver1.9-1.9.51` 的 `sn`

`51soft` 放出的，对 `1.9.6` 无效

```
name: ENFUSiA
serial: TR-O1tCAYu5-NzL90WVT
```

2001.9.11

`zombieys[CCG]`

———————————————————————————————>

               .-"      "-.      Cracked by zombieys[CCG]  >
               /          \\      qq:1789655                  >
                |          |    http://zombieys.yeah.net  >
                |,  .-.  .-.  ,|    http://zombieys.126.com  >
                  |)\\__/   \\__)(|       zombieys.cn.hongnet.com  >
                  |/   /\\   \\|                             >
        (@_@)    (\_    ^^    \_)      Thanks for your supports  >
      _  )\____________|IIIIII|__/\__________                       >
_)\_@8@8{<____________|-__IIIIII/-|______China Crack Group_zombieys____>

`4dd466d0e1e0cc123afbe504da2462ca`
