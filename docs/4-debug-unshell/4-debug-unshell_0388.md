
# Tag&Rename v2.0release

> 原文：`www.pediy.com/kssd/pediy04/kanxue203.htm`

注册 `Tag&Rename v2.0release`

以前写过，这次只改原来的一个小错误。

## 1. unpack

首先是脱壳部分，方法：执行 `tag&rename.exe`，用 `winhex` 搜索 `tag&rename` 内存，填 `runtime` 执行搜索，搜到后,向前找到离 `runtime` 最近的机器码为 `55 8B EC` 的地方就是程序的 `oep`，`2.0` 这个位置在 `593060`。然后打开 `superbpm`，点 `erase`，运行 `trw2k`，`load t&r2.0`，下 `g 593060`，下 `suspend`，用 `prodump` 选 `dump full` 保存。

接着修复 `import table`(就这几个，没变过)

1. `00198208` `KERNEL32.dll` `0151` `GetCommandLineA`
2. `00198354` `KERNEL32.dll` `00E2` `LockRescouce` (我以前写的那个错了,改成这个)
3. `00198390` `KERNEL32.dll` `01E9` `GetVersion`
4. `001983ec` `KERNEL32.dll` `0163` `GetCurrentProcessId`
5. `001983f8` `KERNEL32.dll` `00E2` `LockRescouce`

修复 `it` 后，运行 `t&r` 报错 `Initialization Error`，所以就载入看看啦。发现错误在这个地方：

```
/** Program Entry Point **
:00593060 55                push ebp
:00593061 8BEC              mov ebp, esp
:00593063 83C4F4            add esp, FFFFFFF4
:00593066 53                push ebx
:00593067 56                push esi
:00593068 57                push edi
:00593069 B8602A5900        mov eax, 00592A60
:0059306E E82542E7FF        call 00407298
:00593073 33C0              xor eax, eax
:00593075 55                push ebp
:00593076 68BA305900        push 005930BA
:0059307B 64FF30            push dword ptr fs:[eax]
:0059307E 648920            mov dword ptr fs:[eax], esp
:00593081 A194635900        mov eax, dword ptr [00596394]
:00593086 8B00              mov eax, dword ptr [eax]
:00593088 E8CBE3EAFF        call 00441458
:0059308D FF15307F5900      call dword ptr [00597F30]
```

这个只能用原版跟进去看看啦，发现这里最后 `call` 的是 `59277c`，在对应的位置改一下就好了。`e8c83201`

```
:00593093 A194635900        mov eax, dword ptr [00596394]
:00593098 8B00              mov eax, dword ptr [eax]
```

至此运行正常，脱壳任务完成了，继续 `crack` 工作。

## 2. crack

这个判断的模式一直没变过。选注册，名字任意，注册码 `78787878`，`ctrl+d` 下 `s 30:0 l ffffffff '78787878'`，`bpm` 搜到的地址。跟踪到如下：

`* Possible StringData Ref from Data Obj ->"?A"`

```
|
:0045B03B A188044100        mov eax, dword ptr [00410488]
:0045B040 E88F7FFAFF        call 00402FD4
:0045B045 8945F8            mov dword ptr [ebp-08], eax
:0045B048 8D45F8            lea eax, dword ptr [ebp-08]
:0045B04B E87CFEFFFF        call 0045AECC
:0045B050 8B55FC            mov edx, dword ptr [ebp-04]
:0045B053 8B45F8            mov eax, dword ptr [ebp-08]
:0045B056 8B08              mov ecx, dword ptr [eax]
:0045B058 FF5150            call [ecx+50]
:0045B05B 40                inc eax
:0045B05C 7402->9090        je 0045B060->nop
:0045B05E B301              mov bl, 01
```

这次没用 `crc` 校验??? lucky;-)

不知道有没有暗桩？？？

完活，好像能正常运行了，有错误的话请指出:)

另外 `AWMAWorkshop V1.5release` 今天也已搞定(`aspr` 保护)

下载：`http://zombieys.cn.hongnet.com/cr-AWMAWorkshop15.rar`

下载：`http://zombieys.cn.hongnet.com/cr-tag&rename2.rar`

仅供参考研究

2001.11.4

`zombieys[CCG]`

———————————————————————————————>

```
           .-"     "-.     Cracked by zombieys[CCG]  >
           /          \     qq:1789655                 >
           |           |  http://zombieys.yeah.net  >
           |,  .-.  .-.  ,|  http://zombieys.126.com  >
             |)(\_/  \__)(|      zombieys.cn.hongnet.com  >
             |/   /\   \\|                          >
       (@_)   (\   ^^   \_)      Thanks for your supports  >
       _   )\__________________|IIIIII|/_/_/          >
_)@8@8{}<__________________|-\\IIIIII/-|______China Crack Group_zombieys____>
4dd466d0e1e0cc123afbe504da2462ca
```
