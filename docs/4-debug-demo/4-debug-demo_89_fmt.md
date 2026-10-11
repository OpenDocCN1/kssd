
# Readbook v1.4

> 原文：[`www.pediy.com/kssd/pediy03/forum316.htm`](https://www.pediy.com/kssd/pediy03/forum316.htm)

`Readbook` v1.4 破解实战 (天意 II + `W32dasm` 破解并用 `crackcode2000` 来写注册机)

本文作者: `wind[CCG]`

作者主页: `http://biggow.8u8.com`

1、这里仍然要应用 `w32dasm` 来辅助分析，然后用天意 II，来下断点！就算用 `trw2000` 或者是 `soft-ice` 下 `bpx hmemcpy` 或其他断点，也未必会找到比较核心！这里我们可以用 `w32dasm` 来分析！

2、这个程序没有加壳，所以，直接使用 `w32dasm` 来反汇编程序，将它保存成文本！不要退出，我们可以 string data ref，这里我们找到了"您已经成功地注册了!"

然后，来到如下程序段：

```
:0040A13D 6864384600      push 00463864
:0040A142 8BCE            mov ecx, esi
:0040A144 E8C4F20200      call 0043940D
:0040A149 EB38            jmp 0040A183
```

\-->这里没有我们需要的东西，往上找，来到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409F9C(C)
|
:0040A00B 8D8544FFFFFF    lea eax, dword ptr [ebp+FFFFFF44]
:0040A011 50              push eax
:0040A012 E8AF960200      call 004336C6
:0040A017 8B854CFFFFFF    mov eax, dword ptr [ebp+FFFFFF4C]
:0040A01D 8B8D48FFFFFF    mov ecx, dword ptr [ebp+FFFFFF48]
:0040A023 8BB544FFFFFF    mov esi, dword ptr [ebp+FFFFFF44]
:0040A029 03C8            add ecx, eax
:0040A02B 038D50FFFFFF    add ecx, dword ptr [ebp+FFFFFF50]
:0040A031 69F631750000    imul esi, 00007531
:0040A037 69C931750000    imul ecx, 00007531
```

\-->这个片段就是计算注册码的部分了！我们就用天意 II 在 `40a00b` 下断点！看看！

(1) 先启动 `Readbook` v1.4，来到输入注册码的地方，输入信息！

```
Name:wind[CCG]
Code:8989898989
```

(2) 然后 `alt+z` 呼出天意 II，下 `bpx 40a00b`! 用 `g` 命令返回到 `readbook`

(3) 按"填写完毕"按钮，来到天意 II 调试环境！

(4) 来到 `push eax`，就可以知道 `eax` 里是我们输入的名字，那么下面那个 `call` 是检查你输入的 `Name` 是否符合要求！

\-->OK，我们在往下看一些程序片段！

我们来到这里：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040A0FD(C)
|
:0040A0E2 3975FC          cmp dword ptr [ebp-04], esi
:0040A0E5 741D            je 0040A104
:0040A0E7 8B55EC          mov edx, dword ptr [ebp-14]
:0040A0EA 8BC8            mov ecx, eax
:0040A0EC 83E17F          and ecx, 0000007F
:0040A0EF 03348A          add esi, dword ptr [edx+4*ecx]
:0040A0F2 40              inc eax -->在这里，看 esi 寄存器的值，就是注册码的 16 进制形式!
:0040A0F3 3DFF0F0000      cmp eax, 00000FFF
:0040A0F8 A3B8544600      mov dword ptr [004654B8], eax
:0040A0FD 72E3            jb 0040A0E2
:0040A0FF 3975FC          cmp dword ptr [ebp-04], esi
:0040A102 7547            jne 0040A14B
```

\-->这一端就是比对注册码的地方，我们同样可以使用天意 II 来下断点获得我们的注册码，这个程序在这里可以生成很多注册码，所以，我们就开始捕捉行动！

(1) 在 `40a0e2` 处下断点，`bpx 40a0e2`！

(2) 从天意 II 的调试环境中，可以发现这段程序会生成很多个注册码，太多了！我们只要其中一个!就可以了!

(3) 我们用 `crackcode2000` 来制作一个注册机就可以给任何想要自己注册码朋友使用了!

我的注册码:

```
Name:wind[CCG]
Code:634315452
```

附上 `crackcode2000` 配置文件:

```
[Options]
CommandLine=readbook.exe
Mode=1
First_Break_Address=40A0F2
First_Break_Address_Code=40
First_Break_Address_Code_Lenth=1
Save_Code_Address=ESI

/*这是 wind 安全网斑竹制作的注册器!http://biggow.8u8.com*/  4dd466d0e1e0cc123afbe504da2462ca
```
