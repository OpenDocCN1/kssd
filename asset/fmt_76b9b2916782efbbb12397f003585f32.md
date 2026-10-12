
# HelpBuilder 1.00

> 原文：[`www.pediy.com/kssd/pediy03/forum887.htm`](https://www.pediy.com/kssd/pediy03/forum887.htm)

## HelpBuilder 1.00

> 启动时判断注册码（`cover.pas`）：

> \* Possible String Reference to: `Software\\Iridium Systems\\HelpBuilde`
>
> | r\\1.0\\LicUser
>
> 00101C2D BA601E1000 mov edx, $00101E60
> 00101C32 B802000080 mov eax, $80000002
>
> 00101C37 E81487F9FF call 0009A350
> 00101C3C 8D45BC lea eax, \[ebp\-$44\]
> 00101C3F 8B55F8 mov edx, \[ebp\-$08\]
>
> 00101C42 E87522F1FF call 00013EBC
> 00101C47 8B45BC mov eax, \[ebp\-$44\]
> 00101C4A 8D4DC0 lea ecx, \[ebp\-$40\]
> 00101C4D 8B55F8 mov edx, \[ebp\-$08\]
>
> 00101C50 E85FFBFFFF call 001017B4
> 00101C55 59 pop ecx
> 00101C56 8B55C0 mov edx, \[ebp\-$40\]
> 00101C59 58 pop eax
>
> 00101C5A E86523F1FF call 00013FC4 //注册码第一部分
> 00101C5F 0F8537010000 jnz 00101D9C
> 00101C65 8D55C4 lea edx, \[ebp\-$3C\]
> 00101C68 8B45EC mov eax, \[ebp\-$14\]
>
> 00101C6B E804FCFFFF call 00101874
> 00101C70 8B45C4 mov eax, \[ebp\-$3C\]
> 00101C73 50 push eax
> 00101C74 55 push ebp
> 00101C75 8D4DBC lea ecx, \[ebp\-$44\]
>
> \* Possible String Reference to: `Software\\Iridium Systems\\HelpBuilde`
>
> | r\\1.0\\LicUser
>
> 00101C78 BA601E1000 mov edx, $00101E60
> 00101C7D B802000080 mov eax, $80000002
>
> 00101C82 E8C986F9FF call 0009A350
> 00101C87 8B45BC mov eax, \[ebp\-$44\]
> 00101C8A 8D4DC0 lea ecx, \[ebp\-$40\]
> 00101C8D 8B55F0 mov edx, \[ebp\-$10\]
>
> 00101C90 E81FFBFFFF call 001017B4
> 00101C95 59 pop ecx
> 00101C96 8B55C0 mov edx, \[ebp\-$40\]
> 00101C99 58 pop eax
>
> 00101C9A E82523F1FF call 00013FC4 //注册码第二部分
> 00101C9F 0F85F7000000 jnz 00101D9C
> 00101CA5 8D55C4 lea edx, \[ebp\-$3C\]
> 00101CA8 8B45E8 mov eax, \[ebp\-$18\]
>
> 00101CAB E8C4FBFFFF call 00101874
> 00101CB0 8B45C4 mov eax, \[ebp\-$3C\]
> 00101CB3 50 push eax
> 00101CB4 55 push ebp
> 00101CB5 8D4DC0 lea ecx, \[ebp\-$40\]
> 00101CB8 8B55EC mov edx, \[ebp\-$14\]
> 00101CBB 8B45F8 mov eax, \[ebp\-$08\]
>
> 00101CBE E8F1FAFFFF call 001017B4
> 00101CC3 59 pop ecx
> 00101CC4 8B55C0 mov edx, \[ebp\-$40\]
> 00101CC7 58 pop eax
>
> 00101CC8 E8F722F1FF call 00013FC4 //注册码第三部分
> 00101CCD 0F85C9000000 jnz 00101D9C
> 00101CD3 8D55C4 lea edx, \[ebp\-$3C\]
> 00101CD6 8B45E4 mov eax, \[ebp\-$1C\]
>
> 00101CD9 E896FBFFFF call 00101874
> 00101CDE 8B45C4 mov eax, \[ebp\-$3C\]
> 00101CE1 50 push eax
> 00101CE2 55 push ebp
> 00101CE3 8D45BC lea eax, \[ebp\-$44\]
> 00101CE6 8D55C8 lea edx, \[ebp\-$38\]
> 00101CE9 B90C000000 mov ecx, $0000000C
>
> 00101CEE E85521F1FF call 00013E48
> 00101CF3 8B55BC mov edx, \[ebp\-$44\]
> 00101CF6 8D4DC0 lea ecx, \[ebp\-$40\]
> 00101CF9 8B45E8 mov eax, \[ebp\-$18\]
>
> 00101CFC E8B3FAFFFF call 001017B4
> 00101D01 59 pop ecx
> 00101D02 8B55C0 mov edx, \[ebp\-$40\]
> 00101D05 58 pop eax
>
> 00101D06 E8B922F1FF call 00013FC4 //注册码第四部分
> 00101D0B 0F858B000000 jnz 00101D9C

> 作者大概是从哪里找到一个免费的 Hash 库，计算注册码的时候派上用场了 ;-)
>
> 用 `UltraEdit` 打开 exe 文件可以看到大堆 `THash_xxxx`，所以要写注册机的话还得把 hash 算法分析出来。要暴力改的话除了上面几个地方要改，还有其它的判断，如 `About.pas`、`main.pas`。只要 `*.pas` 中有字符串“`\\1.0\\SerialNumber`”就有注册码判断。 `4dd466d0e1e0cc123afbe504da2462ca`
