
# 美萍电脑安全卫士 V7.52

> 原文：[`www.pediy.com/kssd/pediy03/forum647.htm`](https://www.pediy.com/kssd/pediy03/forum647.htm)

美萍电脑安全卫士(V7.52 标准版)终极破解（注册法 & 暴力破解法）

作者：PaulYoung

时间：二○○一年二月二十三日凌晨

首先感谢《看雪论坛》几名知名不知名的朋友提供技术指导，参考 `WIND[CCG]美萍电脑安全卫士 V5.2` 的破解法。（论坛精华 2）

美萍电脑安全卫士可以防 `TRW`、`SOFTICE` 等多种常用破解软件，一旦检测到相关关键词，即时关机没商量，惨……惨……惨……所以，破解前必须先干掉它的反破解。

先用 `unaspack` 或其它工具脱掉 `Aspack` 的壳，再用 `UEDIT` 或 `HEXWORKSHOP` 等编辑工具查找 `TRW`、`WINICE`...你会在那里同时发现有多种常用的破解软件的关键词，只要把那些关键词替换成任意一个词即可，如 `TRW` 换成 `TRR` 即可，其它同样道理。

好了，现在用 `W32DASM` 反汇编脱壳后的文件。

关键比较在下面，具体跟踪过程我懒得写了。（别扔鸡蛋！！！）

```
* Possible StringData Ref from Code Obj ->"RegNum"

              |
:0046EE99 B840EF4600         mov eax, 0046EF40
:0046EE9E E895F6FFFF         call 0046E538
:0046EEA3 3BD8               cmp ebx, eax
:0046EEA5 750A               jne 0046EEB1
:0046EEA7 B801000000         mov eax, 00000001
:0046EEAC E983000000         jmp 0046EF34

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046EEA5(C)
|
:0046EEB1 33C0               xor eax, eax
:0046EEB3 5B                pop ebx
:0046EEB4 C3                ret
```

用 `trw2000` 下 `bpx 0046eea3`，再运行美萍安全卫士，立即被拦截，下 `? ebx`，看到的 `DEC` 值（十进制）就是你的注册码。

或将 `:0046EEA5 750A jne 0046EEB1` 中的 `jne` 改为 `je`，那填什么注册码都行了！ 4dd466d0e1e0cc123afbe504da2462ca
