
# HIEW6.70

> 原文：[`www.pediy.com/kssd/pediy03/forum1115.htm`](https://www.pediy.com/kssd/pediy03/forum1115.htm)

破解 `HIEW6.70` 完全版软件的读 `Hiew.key` 文件的功能。

- **破解人**：4
- **日期**：七月一日

写得太差了，请不要扔石头。啊！那个高手扔得的蕃茄？又一个土豆？啊，有人扔石头了！救命啊...

今天收到一个朋友寄来的 `HIEW6.70` 完全版软件。不过我不想使用他的 `Hiew.key` 文件，但是如果没有这个文件，`HIEW32.EXE` 就会提醒你去注册。

还是破解一下这个软件吧。

使用 `fileinfo` 检查到 `HIEW32.EXE` 是用 `ASPACK211` 加密。使用 `ASPR` 脱壳它的壳吧。

用 `w32dasm` 打开 `HIEW32`，反编译后，按 `String Data References` 键。看这里 `HIEW.KEY`，按两下这个英文单词来这里：

## 办法一

`Possible StringData Ref from Data Obj ->"HIEW.KEY"`

```assembly
:00403573 689C264300           push 0043269C
:00403578 50                   push eax
:00403579 51                   push ecx
:0040357A E811FDFFFF           call 00403290
:0040357F 83C40C               add esp, 0000000C
:00403582 50                   push eax
:00403583 E888010000           call 00403710
:00403588 83C408               add esp, 00000008
:0040358B 85C0                 test eax, eax
:0040358D 743C                 je 004035CB    < = 如果 eax=0,就跳
:0040358F 68402B4400           push 00442B40
:00403594 E8E7020000           call 00403880
:00403599 83C404               add esp, 00000004
:0040359C 85C0                 test eax, eax
:0040359E 742B                 je 004035CB    < = 如果 eax=0,就跳
:004035A0 57                   push edi
:004035A1 56                   push esi
:004035A2 6A65                 push 00000065
:004035A4 C7054C0B440001020000 mov dword ptr [00440B4C], 00000201
:004035AE E87DE6FFFF           call 00401C30
:004035B3 B919000000           mov ecx, 00000019
:004035B8 BE582B4400           mov esi, 00442B58
:004035BD 8BF8                 mov edi, eax
:004035BF 83C404               add esp, 00000004
:004035C2 A3480B4400           mov dword ptr [00440B48], eax
:004035C7 F3                   repz
:004035C8 A5                   movsd
:004035C9 5E                   pop esi
:004035CA 5F                   pop edi
```

容易啦，修改两个 `je 004035CB` 为 `nop`：

```assembly
nop
```

或者

在 `w32dasm` 查出 `:0040358D 743C je 004035CB` 的 offset address 等于 `298Dh`，利用 `Hiew`, `Hex Workshop`, `Winhex` 等，修改 `743C` 为 `9090`。

然后找出 `:0040359E 742B je 004035CB` 的 offset address 是 `299Eh`，修改 `742B` 为 `9090`。

## 办法二

用 `w32dasm` 打开 `HIEW32`，反编译后，按 `String Data References` 键。找那个要你注册的信息 `Unregistered version, please registeryour copy`，按两下来这里：

`Referenced by a CALL at Addresses: :0040119E, :004011E0`

```assembly
:004036E0 A14C0B4400           mov eax, dword ptr [00440B4C]
:004036E5 85C0                 test eax, eax
:004036E7 751B                 jne 00403704   < = 一定要跳，修改 jne 为 jmp
:004036E9 6A00                 push 00000000
:004036EB E8C0590200           call 004290B0
:004036F0 83C404               add esp, 00000004
:004036F3 A807                 test al, 07
:004036F5 750D                 jne 00403704
```

`Possible StringData Ref from Data Obj ->"Unregistered version, please register " ->"your copy !"`

```assembly
:004036F7 68D4264300           push 004326D4
:004036FC E8EF060200           call 00423DF0
:00403701 83C404               add esp, 00000004
```

在 `w32dasm` 查出 `jne 00403704` 的 offset address: `2AE7`。利用 `Hiew`, `Hex Workshop`, `Winhex` 等，修改 `751B` 为 `EB1B`。

办法一要比办法二要好一点儿啦。

- 用办法一，按 `Ctrl+Alt`，就可以看到注册窗口。
- 用办法二，就看到不同信息窗口，但是办法二仅要修改一处地方。
