
# 模拟器 Nebula 的脱壳和保护研究

> 原文：[`www.pediy.com/kssd/pediy05/pediy50399.htm`](https://www.pediy.com/kssd/pediy05/pediy50399.htm)

**标题：** 模拟器 Nebula 的脱壳和保护研究  
**软件：** `Nebula`  
**版本：** 2.21  
**下载网站：** `www.hotbest.com`  
**破解者：** 火翼\[CCG\]  
**组织：** \[CCG\] (China Cracking Group)  
**使用软件：** `trw2000`+`HexWorkshop`+`OllyDbg`

这个软件的保护确实很好玩，很折磨人。

直接用 `trw2000` 加载后会无法执行(后来发现出错的原因不是那个异常而是因为没装 `Directx 8.0` 造成读取 `DirectInput` 设备出错)，用 `OllyDbg` 载入执行到：

```
0712D0BA  0F0B             UD2  //触发异常，会跳到异常处理
                                 //程序
0712D0BC  E8 2BD2648B      CALL 9277A2EC
0712D0C1  028B 20648F02    ADD CL,BYTE PTR DS:[EBX+28F6420]
0712D0C7  58               POP EAX
0712D0C8  5D               POP EBP
0712D0C9  C3               RETN
0712D0CA  9A 8B95E723 4000 CALL FAR 0040:23E7958B               ; 远距呼叫 FAR
0712D0D1  E8 F9000000      CALL nebula.0712D1CF
0712D0D6  E8 01000000      CALL nebula.0712D0DC
0712D0DB  C783 C404BB27 44>MOV DWORD PTR DS:[EBX+27BB04C4],6A001E44
0712D0E5  04 68            ADD AL,68
0712D0E7  0030            ADD BYTE PTR DS:[EAX],DH
```

结束异常处理后，会返回：

```
0712D0BD  2BD2             SUB EDX,EDX
0712D0BF  64:8B02          MOV EAX,DWORD PTR FS:[EDX]
0712D0C2  8B20             MOV ESP,DWORD PTR DS:[EAX]
0712D0C4  64:8F02          POP DWORD PTR FS:[EDX]
0712D0C7  58               POP EAX
0712D0C8  5D               POP EBP
0712D0C9  C3               RETN
```

比较后会发现，这一段会把堆栈返回异常前，另外把 `ECX` 置为 `7120bd`。

只要这样改便可跳过异常：

```
0712D0BA  EB 01            JMP SHORT nebula.0712D0BD

.
.
.

0712D0BF  B9 BDD01207      MOV ECX,nebula.0712D0BD
```

继续跟，跳出一个复杂的循环后会到达：

```
0712D1BE  2B7C24 28        SUB EDI,DWORD PTR SS:[ESP+28]
0712D1C2  897C24 1C        MOV DWORD PTR SS:[ESP+1C],EDI
0712D1C6  61               POPAD
0712D1C7  C3               RETN //跳到 712D11A
```

然后执行到：

```
0712D120  83C4 04          ADD ESP,4
0712D123  5A               POP EDX
0712D124  5E               POP ESI
0712D125  0E               PUSH CS
0712D126  56               PUSH ESI
0712D127  CB               RETF  //跳到 071d0000
```

按 `F7` 跟到 `071d0000`，执行到：

```
071D0029  C1               ???       ; 未知命令
071D002A  F1               ???       ;未知命令
```

会无法继续。只要把这两句改为 `NOP` 就可以了过去了。

循环会在：

```
071D035F  6A 3D            PUSH 3D
071D0361  59               POP ECX
071D0362  F3:A4  REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
071D0364  C3               RETN //跳到 712D001
```

跳回 `712d001` 处。之后是个小循环，只要在 `0712D03D  C3  RETN` 设断，按几次 `F9`，观察 `esp` 指向为：

```
0012FFC0  004E355C  nebula.004E355C
0012FFC4  77E614C7  返回到 kernel32.77E614C7
0012FFC8  0012D51C
```

`004e355c` 处就是 `OEP`。

用 `trw2000` 载入程序，`G 4E355C`，`Pedump` 或者 `Makepe` 就可以了。

### 总结

程序的入口点在 `import` 段，执行时会完成还原 `import` 表和解压程序的过程。

有一次循环是还原 `import` 表引用的函数名称，代码如下：

```
0712D083  8A07             MOV AL,BYTE PTR DS:[EDI]
0712D085  32C2             XOR AL,DL
0712D087  32C6             XOR AL,DH
0712D089  D2C0             ROL AL,CL
0712D08B  02C1             ADD AL,CL
0712D08D  02C5             ADD AL,CH
0712D08F  02C2             ADD AL,DL
0712D091  02C6             ADD AL,DH
0712D093  D2C8             ROR AL,CL
0712D095  2AC1             SUB AL,CL
0712D097  2AC5             SUB AL,CH
0712D099  F6D0             NOT AL
0712D09B  2AC2             SUB AL,DL
0712D09D  2AC6             SUB AL,DH
0712D09F  D2C0             ROL AL,CL
0712D0A1  D2C8             ROR AL,CL
0712D0A3  32C1             XOR AL,CL
0712D0A5  F6D0             NOT AL
0712D0A7  32C5             XOR AL,CH
0712D0A9  D3C2             ROL EDX,CL
0712D0AB  8807             MOV BYTE PTR DS:[EDI],AL
0712D0AD  47               INC EDI
0712D0AE  49               DEC ECX
0712D0AF  75 D2            JNZ SHORT nebula.0712D083
```

开始地址为 `712d229`，结束地址为 `71c5018`。

还有一次循环是用这还原出的名字，在申请的一块内存中构建一段代码（就是 `071D0000` 处），这一段代码会把 `import` 表对应的地址建好。

另外要注意，代码很多地方都用 `call` 代替 `jmp`，不能用 `F8` 跳过去，`ret` 时也要注意 `esp` 指向的地址。

### 问题

1. 程序只能在 `98` 下运行，`XP` 下 `dump` 之后修复引入表不成功。
2. `dump` 出来的程序有 `110M` 大，有 `100` 多 `M` 是 `00`。但我 `PE` 结构还学的不透，修正总是失败，还好，可以再用别的压缩工具再压一下。

### 修正

#### 1）文件大小修正

仔细研究了一下段的结构和加载时的内存映射关系，发现修正文件确实很简单。只要保持虚拟地址和大小不变，改变段的实际大小和在文件中的启始地质就可以了去掉没用的 `0` 了。`makepe` 产生的文件有 `100` 多 `M`，其中最大的短是 `124000` 开始的，这段的内容在 `1DA900` 后就全是 `0` 了，`1da900-124000=B6900`，这段的实际大小我取 `B8000`，`124000+B8000=1DC000`。然后用 `16` 进制文件工具打开脱壳后的文件，删除 `1DC000` 到下一段开始的地址之间的 `0`，这里是 `6D23000`，所以删除 `1DC000-6D22FFF` 之间的所有字节，然后修正剩下段在文件中的启始地址，后面所有段在文件中的启始地址都要减去 `6B47000`（`6D23000-1DC000`）。修正后文件大小为 `2.5M`。

#### 2）XP 下无法运行的修正

`98` 下去壳的文件无法在 `XP` 下执行，点击会弹出无法找到 `TlsAllocInternal`。这个函数对应的 `XP` 下函数为 `TlsAlloc`，改过后执行会出现 `runtime` 错误，只好手工修正引入表。原加壳程序的 `IAT` 填写由程序本身完成，经过比较发现在 `XP` 下运行 `98` 下生成的去壳文件时系统分配的 `IAT` 和 `XP` 下运行原加壳程序的 `IAT` 有 `5` 处不同。

原程序运行时生成 `IAT`：

```
77E5A6F0  ;CloseHandle
77E5569E  ;GlobalAlloc
77E5E595  ;kernel32.FreeEnvironmentStringsW
77E5DE7E  ;kernel32.LCMapStringW
77E5E0C8  ;kernel32.GetStringTypeW
```

在 `XP` 下运行 `98` 下脱壳程序生成的 `IAT`：

```
77E4DB14  ;FindCloseChangeNotification
77E5A682  ;LocalAlloc
77E51660  ;kernel32.FreeEnvironmentStringsA
77E5D995  ;kernel32.LCMapStringA
77E548CE  ;kernel32.GetStringTypeA
```

这些不同是因为 `98` 和 `XP` 系统对程序的引入表中的函数名作了不同的解释，而 `Trw2000` 在 `makepe` 时只是根据 `IAT` 重新建立了 `import` 表，这就造成了程序无法直接在 `XP` 下运行。

把 `98` 下脱壳文件的引入表里的函数名称改为对应的原程序 `IAT` 运算中使用的函数就可以修正在 `XP` 下不能执行的问题。

另外如果使用 `ProcDump` 来 `Dump` 文件就不会出现文件很大的问题，但引入表要自己修正。

到此，`Nebula` 已完美脱壳。

`4dd466d0e1e0cc123afbe504da2462ca`
