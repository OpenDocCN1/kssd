
# Armadillo 2.50

> 原文：[`www.pediy.com/kssd/pediy05/pediy50081.htm`](https://www.pediy.com/kssd/pediy05/pediy50081.htm)

`Armadillo.exe` 的脱壳(2.50)

在 `hying` 和 `zombieys` 两位大虾的指教下，初试脱 `armadillo.exe`，没有想到意外成功。

以下不妥的地方还请诸位大侠多多指教。

我用 `TRW` 装载 `Armadillo.exe`（2.50）。按照 `zombieys` 大虾的方法，`bpx virtualprotect`。共中断 27 次后，按 `F12` 进入 `armadillo` 的领空。在下面：

```
0167:004C7DB3 MOV CL,[EBP+0C]
0167:004C7DB6 PUSH ECX
0167:004C7DB7 CALL NEAR [EBP-3C] 《＝＝＝＝进入
0167:004C7DBA ADD ESP,BYTE +08
0167:004C7DBD MOV [EBP-10],EAX
0167:004C7DC0 PUSH DWORD 004D01B4
0167:004C7DC5 CALL 004C9798
0167:004C7DCA ADD ESP,BYTE +04
0167:004C7DCD CALL NEAR [004D2EF0]
0167:004C7DD3 PUSH BYTE +00
0167:004C7DD5 PUSH BYTE +00
```

进入后，来到下面：

```
0167:00C9C827 MOV ECX,[EAX+20]
0167:00C9C82A XOR ECX,[EAX+1C]
0167:00C9C82D XOR ECX,[EAX+10]
0167:00C9C830 ADD EDI,ECX
0167:00C9C832 CALL KERNEL32!GetCommandLineA
0167:00C9C838 PUSH EAX
0167:00C9C839 PUSH EBX
0167:00C9C83A PUSH EBX
0167:00C9C83B CALL ESI
0167:00C9C83D PUSH EAX
0167:00C9C83E CALL EDI 《＝＝＝＝＝edi=41dc2c.(oep)
0167:00C9C840 MOV ESI,EAX
0167:00C9C842 POP EDI
0167:00C9C843 MOV EAX,ESI
0167:00C9C845 POP ESI
0167:00C9C846 POP EBX
0167:00C9C847 RET
```

进入 `41dc2c` 后，用 `makepe` 命令，dump 出一个文件 `a.exe`。

用 `LoadPE` 分析原文件 `Armadillo.exe`，选择“PE Editor”。点击“Sections”看看“.rsrc”部分。`Voffset=38000`，`VSize=8BE00`。它的尾部是 `c3e00`。加 `imagebase` `400000` 应该是 `4c3e00`。我选择 `4c3f00` 为 `hying` 大虾的那段代码的起始点。同时记下 `SizeOfImage`。

重新运行原文件 `Armadillo.exe`，下 `bpx virtualalloc`。断下后，记下该函数的指针。取消断点。下断在入口点处：`bpx 41dc2c`。断下后，将 `eip` 改为我们选定的地方：`4c3f00`。在这里，我们将 `hying` 大虾的那段代码写入。

```
6804000000 push PAGE_READWRITE
6800100000 push MEM_COMMIT
6800001100 push 110000h <-申请内存的大小，可大一点。
6800000000 push 0
e8xxxxxxxx call VirtualAlloc <-申请内存
mov edi,eax <--记下 eax 的值，假如为 fa0000
mov ecx,？？？？ <-文件映象大小减 1000h，假如为 xxxxxx
mov esi,401000h <-基址加 1000h，有的程序不是这个值
rep movsb
```

当 `rep movsb` 指令运行完后，从 `fa0000` 开始的 `xxxxxx` 字节数据，用 `W` 命令保存为文件 `1.bin`。

将 `eip` 改为原入口点。退出原程序。

用 `winhex` 打开 dump 出的文件 `a.exe`，在 `offset 1000` 处将 `1.bin` 文件写入。存盘。

我们运行 `a.exe` 文件，出错！记下出错地址。再运行原 `Armadillo.exe`，在出错地址设断。

断下后看看出错地址的内存，按 `PageUp` 键，上下看看内存，把上下 `？？` 之间内存数据，用 `W` 命令写入文件 `2.bin` 中，记下 `RVA`（开始地址－`400000`）和 `size`。用 `LoadPE` 打开 dump 文件 `a.exe`，增加一个 sections，记住新增的 offset。用 `winhex` 打开 `a.exe`，将 `2.bin` 文件写入。

运行 `a.exe` 文件，呵呵，又出错了。记下出错地址，仿照上述办法，保存为 bin 文件，写入 `a.exe` 文件中。再运行 dump 文件，呵呵，成功了。

`armadillo2.5` 后的版本也可仿照此办法。

不正确、不清楚的地方，请诸位大侠多多指教。
