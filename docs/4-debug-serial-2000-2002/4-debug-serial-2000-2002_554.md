
# winhex9.9

> 原文：[`www.pediy.com/kssd/pediy03/forum1167.htm`](https://www.pediy.com/kssd/pediy03/forum1167.htm)

我以 `winhex9.9` 为例。请使用 `TRW`，因为要用 `suspend`。

启动 `winhex` 前，伪造 `user.txt`。要留意伪造的 `user.txt` 的格式，`User:` `ADDR1:` `ADDR2:` 英文、中文可任意输入。但是，`code1`、`code2` 的数必须是 `0-f` 且必须有 `32` 个。`code1` 的第 `21` 到 `24` 位数，假设它为 `A(1)A(2)A(3)A(4)`，那么 `A(3)A(4)A(1)A(2)` 应小于 `0370`，或者大于 `03A2`。

打 `bpx` `417dbd`

启动 `winhex`，打开内存修改，存盘。中断在下面：

```
:00417DBD B9BC060000                 mov ecx, 000006BC
:00417DC2 8B1578ED4500             mov edx, dword ptr [0045ED78]
:00417DC8 E823D3FEFF               call 004050F0
:00417DCD E826FEFFFF               call 00417BF8                     <---进入
:00417DD2 BAB8654600               mov edx, 004665B8
:00417DD7 8D442428                 lea eax, dword ptr [esp+28]

* Possible Ref to Menu: MenuID_0001, Item: "Sicherung laden..."
```

进入 `417dcd`: `call` `417bf8` 中：

```
:00417D27 E8CCDCFEFF               call 004059F8
:00417D2C 8BD8                     mov ebx, eax
:00417D2E 6A00                     push 00000000
:00417D30 BAB8654600               mov edx, 004665B8
:00417D35 8BC3                     mov eax, ebx

* Possible Ref to Menu: MenuID_0001, Item: "Sicherung laden..."

:00417D37 B910000000               mov ecx, 00000010
:00417D3C E8A3DDFEFF               call 00405AE4              <----进入

* Possible Ref to Menu: MenuID_0001, Item: "Hex-Werte  Strg+Shift+C"

:00417D41 BA2C000000               mov edx, 0000002C
:00417D46 8BC3                     mov eax, ebx
```

进入 `417d3c`: `call` `405ae4` 中：

```
:00405B2A 8B45FC                   mov eax, dword ptr [ebp-04]
:00405B2D E8AAFDFFFF               call 004058DC
:00405B32 0FB7C8                   movzx ecx, ax
:00405B35 C1E908                   shr ecx, 08
:00405B38 24FF                     and al, FF
:00405B3A 32C8                     xor cl, al
:00405B3C 8B45F8                   mov eax, dword ptr [ebp-08]
:00405B3F 03C6                     add eax, esi
:00405B41 3208                     xor cl, byte ptr [eax]     <---此处打 bpx,每次停在这里，cl 中的数据和下面的一组 16 个字节数据依次 xor 后，存入[eax]中，这个数一定要记下来，最后记下来的一组 16 个字节数即为 code2.
```

即：`cl` `xor` `5b` == `?`，将 `?` 存入 `[eax]` 中，同时记下 `?`，这是 `code2` 的第一个数据。

第二次中断在这里，将 `cl` 与 `0e` `xor`，结果存入 `[eax]`，并记下，此为 `code2` 的第二个数据，依此类推。一共中断 `16` 次，得到 `16` 个数据，即为 `code2`。

**这一组 16 个字节数据如下：**

```
5b 0e 44 2c 80 f2 0c d3 b4 ad 22 97 da ce 3f 6d
```

这样找到的注册码适用 `winhex9.74` 到 `9.92`。

`winhex` 的加密做的真的很好，连 `\\.\SICE` 字符串也加了密。

***

`user.txt` 如下，它适合 `winhex9.74` 到 `9.92` 的任何版本：

```
User: yourname
Addr1: aaaa
Addr2: aa
Code1: 0123456789abcdef0123456789abcdef
Code2: 9b8be3627af62ab02fb7c4b793766410
```

```
User: 中国四川
Addr1: 看雪论坛
Addr2: 看雪学院
Code1: 88888888888888888888888888888888
Code2: 93e853c64441aeaf93c4b1c805b90ca9
```

`4dd466d0e1e0cc123afbe504da2462ca`
