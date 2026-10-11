
# Asproctect1.2 加壳

> 原文：[`www.pediy.com/kssd/pediy05/pediy50076.htm`](https://www.pediy.com/kssd/pediy05/pediy50076.htm)

用 `ollydbg` 跟踪 `asproctect1.2` 加壳的软件

作者：`zczc`

写文章只是为交流技术，转载请保持完整。

打开 `ollydbg`，将 `options` 菜单下的 `debugging options` 中的 `Exceptions` 的 `int3 break`

和 `single break` 勾选上，好了，开始吧。

这次以 `XXXXXX` 为目标，加载 `XXXXX`，按 `F9` 嘭.....

中断在第一个异常处：

```
003D009D  3100              XOR DWORD PTR DS:[EAX],EAX  <---
003D009F  EB 01             JMP SHORT 003D00A2
003D00A1  68 648F0500       PUSH 58F64
003D00A6  0000              ADD BYTE PTR DS:[EAX],AL
```

这次要按 `shift+F9` 忽略异常，继续执行 又断下，前后要按大约 19 次（呵呵，没仔细数），来到：

```
00901CA4  FE06              INC BYTE PTR DS:[ESI]  <---
00901CA6  EB E8             JMP SHORT 00901C90
00901CA8  83E0 72           AND EAX,72
00901CAB  33D2              XOR EDX,EDX
00901CAD  64:8F02           POP DWORD PTR FS:[EDX]
```

看一下右下角的 `esp` 数据窗，在代码窗点右键，`go to` --> `Expression`，把 `esp+4` 的内容填入，`F2` 设个断点，按 `shift+F9`

安全落地。呵呵，接下来一路 `F7`(别按太快，你看仔细）

```
00901C77  5B                POP EBX
00901C78  58                POP EAX
00901C79  05 F7AA26E5       ADD EAX,E526AAF7
00901C7E  5C                POP ESP
00901C7F  0BC9              OR ECX,ECX
00901C81  74 E3             JE SHORT 00901C66
00901C83  8901              MOV DWORD PTR DS:[ECX],EAX
00901C85  03C3              ADD EAX,EBX
00901C87  894424 1C         MOV DWORD PTR SS:[ESP+1C],EAX
00901C8B  61                POPAD
00901C8C  FFE0              JMP EAX  <---到这里下车
```

不会吧，你不会用 `procdump`,(喂，我要看只用 `ollydbg` 脱壳）

`s-ice` 还要调用 `Bhrama` 服务，何必那样强..呢？精通一样，就已经很强了

`trw2000` 好是好，可我用的是 `XP`。 `4dd466d0e1e0cc123afbe504da2462ca`
