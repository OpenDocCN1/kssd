
# 屏蔽 QQ 的系统广播消息

> 原文：[www.pediy.com/kssd/pediy05/pediy50516.htm](https://www.pediy.com/kssd/pediy05/pediy50516.htm)

在 `qq.exe` 的主程序里：（build1230final）

```
.0045C535: 8B4628              mov        eax,[esi][28]
.0045C538: 8B0CD8              mov        ecx,[eax][ebx]*8
.0045C53B: 8D04D8              lea        eax,[eax][ebx]*8
.0045C53E: 3B4DF0              cmp        ecx,[ebp][-10]                 ;这里是判断收到了什么消息
.0045C541: 7511                jne        .00045C554  -----? (1)         ；比方说如果[ebp][-10]=2 那就是有人向你发送请求加入的消息
.0045C543: 8B4004              mov        eax,[eax][04]                  =3 就是你好友给你发消息了
.0045C546: 57                  push       edi                             =6 就是我们要的，系统广播消息！
.0045C547: 50                  push       eax                             所以我们用 SMC 做一个判断，如果是 6，
.0045C548: C7450801000000      mov        d,[ebp][08],000000001 ;"       就直接跳过 45C551 的 call 即可：）
.0045C54F: 8B08                mov        ecx,[eax]
.0045C551: FF510C              call       d,[ecx][0C]
.0045C554: 43                  inc        ebx
.0045C555: 3B5E2C              cmp        ebx,[esi][2C]
.0045C558: 7CDB                jl         .00045C535  -----? (2)
.0045C55A: 837D0800            cmp        d,[ebp][08],000 ;" "
.0045C55E: 7534                jne        .00045C594  -----? (3)
.0045C560: 8D4D08              lea        ecx,[ebp][08]
```

一开始因为收不到系统广播消息而走了很多弯路，后来不停的注册 5 个 qq 号码才搞定：（

一开始跟的时候用 `bpx sndplaysounda`（需要 export `winmm.dll`）因为有消息来了会先响，直到你去看什么消息。

又注册了 3 个号码测试过了，不会再收到系统广播了，其它的毫无影响，eNjoy!

回去睡觉了，困，明天还要给国军买命:(

by dRag0nMa 03.27.03 4dd466d0e1e0cc123afbe504da2462ca
