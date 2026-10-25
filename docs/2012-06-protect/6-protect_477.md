
# hidetoolz v2.1 在 win2k3 下重启解决办法

> 原文：`www.pediy.com/kssd/pediy09/pediy09-314.htm`

> 用 16 进制打开 `hidetoolz.exe` 找到 `0x15e71` 位置把 `0x14` 改成 `0x24`

> 在我自己机器测试成功，不保证这个方法的完美性

> 请期待作者修改这个 bug

> ps: 求人不如求己

代码:

```assembly
.text:000129A9    mov dword_1327C, 11E1h
.text:000129B3    mov dword_13284, 1137h
.text:000129BD    mov dword_13274, 1179h
.text:000129C7    mov dword_13268, 42h
.text:000129D1    mov dword_13280, 1205h
.text:000129DB    mov dword_13264, 228h
.text:000129E5    mov dword_13290, 154h
.text:000129EF    mov dword_13278, 22Ch
.text:000129F9    mov dword_13270, 0C4h
.text:00012A03    mov dword_1328C, 114h
```

这个地方是判断操作系统是 2003 后初始化的地方，这些数值应该都是未公开的数据。根据操作系统不同数值也会不同，我们注意到把 `dword_1328C` 初始化为 `114h`。

代码:

```assembly
.text:00011C17    mov eax, dword_1328C
.text:00011C1C    mov eax, [eax+esi]
.text:00011C1F    mov [ebp+Object], eax
```

这个地方取 `esi + dword_1328C` 即 `esi + 114h` 的地方的数据保存到 `ebp + Object`。

代码:

```assembly
.text:00011C93    mov eax, [ebp+Object]
.text:00011C96    cmp eax, ebx
.text:00011C98    jz short loc_11CDB
.text:00011C9A    mov eax, [eax+14h]
.text:00011C9D    mov eax, [eax]
.text:00011C9F    mov esi, [eax+24h]
```

这个地方取上面保存的数值为 `eax`，然后再取 `[eax+14h]` 的值。而当 `dword_1328C` 等于 `114h` 的时候，`[esi+114h]` = 3，这样 `mov eax, [eax+14h]` 就是 `mov eax, [3+14h]` 造成错误，导致系统重启。所以说这个 `114h` 数值不适合 2003，更确切的说不适合 2003sp2 吧，可能这个数适合某个版本的 2003。经过调试 2003sp2 下的这个数值应该是 `124h`。

所以我们要做的就是把 `mov dword_1328C, 114h` 改成 `mov dword_1328C, 124h`。

刚看了一下，原来是 `EPROCESS->SectionObject` 的偏移。

`4dd466d0e1e0cc123afbe504da2462ca`
