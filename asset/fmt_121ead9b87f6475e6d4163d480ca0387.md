
# CAXA V2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50111.htm`](https://www.pediy.com/kssd/pediy05/pediy50111.htm)

软件：`CAXA`  
版本：`V2`  
破解者: 火翼`[CCG]`  
组织 : `[CCG]` (China Cracking Group)  
使用软件: `pedump` + `superbpm` + `importrec` + `trw2000`

本来 `CAXA` 已经被破解过了，但破解者给程序加了一个 NAG 窗口，并且用 `asprotect1.2` 加了壳，最近正在研究脱壳，正好练习一下。

加载 `superbpm`,用 `trw2000` 载入，按 3 次 `F10` 来到下面：

```assembly
019F:005F3002  CALL    005F3022    //注意，这里的 Call 就是 asprotect 的
                                    //经典，不能按 F10,只能按 F8，否则就
                                    //飞了，下面还有很多处，多注意以下
019F:005F3007  JMP     1014BE08
019F:005F300C  PUSH    ES
019F:005F300D  ADD     [EAX],AL
```

继续跟下去，会有几处循环，看明白后用 `G` 跳过去就可以了，循环都过去后在执行一会就会来到：

```assembly
019F:01F713AB  ADD     ESP,BYTE -0C
019F:01F713AE  CALL    01F63130
019F:01F713B3  JNZ     NEAR 01F63E0C
019F:01F713B9  CALL    01F642C8     //也是 asprotect 的经典，最后一个要
019F:01F713BE  CALL    01F691CC     //跟进去
019F:01F713C3  CALL    01F69B5C
019F:01F713C8  CALL    01F6BEBC
019F:01F713CD  CALL    01F63E0C
019F:01F713D2  MOV     ESP,EBP
019F:01F713D4  POP     EBP
019F:01F713D5  RET     0C
```

跟进去后就会遇到最大的麻烦 `RET` 了。

对于 `asprotect` 很多 `ret` 的作用是跳到 `esp` 所指向的位置，遇到 `ret` 时要 `d esp` 一下如果显示的数字(要反着读，如看到 `d5 14 f6 01` 就是 `01 f6 14 d5`)和原来的地址差不多就可以按 `F10`，如果是 `BFXXXXX` 之类的就要用些特殊的方法了。

一般都用找 `ret` 法，即找下面最近的 `ret`，并在那里设断，并按 `G`, 就会断下来。但有时没有 `RET` 或不好找，也可以使用另一种办法，例如类似以下代码：

```assembly
019F:01F6FEA1  XOR     EAX,EAX
019F:01F6FEA3  RET            //此处即为上述的 ret,下面附近没有 ret
019F:01F6FEA4  JMP     SHORT 01F6FEA7
019F:01F6FEA6  CALL    04E2BEDC
019F:01F6FEAB  INT     20
019F:01F6FEAD  PUSH    DWORD [FS:EAX]
019F:01F6FEB0  JMP     SHORT 01F6FEB3
019F:01F6FEB2  JMP     3317881B
019F:01F6FEB7  ADD     BL,CH
019F:01F6FEB9  ADD     [EAX+64],EBP
019F:01F6FEBC  POP     DWORD [00]
019F:01F6FEC2  JMP     SHORT 01F6FEC6 //在这里设断，可以断下来
019F:01F6FEC4  CALL    C22A56CA
019F:01F6FEC9  POP     EDX
```

类似的代码很多，都可用类似的设断方法断下来。

继续跟，如果遇到按 `F10` 时 `CALL` 使程序运行了，就用 `F8` 进去。

运行一段时间后(有点长，一定要有耐心)，来到下面：

```assembly
019F:01F6FBFE  MOV     EDX,04
019F:01F6FC03  CALL    01F6C39C              //也是看雪的书里提
                                              //到过的代码
019F:01F6FC08  CALL    01F6FC22
019F:01F6FC0D  MOV     EAX,[ESP+0C]
019F:01F6FC11  ADD     DWORD [EAX+B8],BYTE +02
019F:01F6FC18  MOV     DWORD [EAX+18],00
019F:01F6FC1F  XOR     EAX,EAX
019F:01F6FC21  RET                          //很特别的跳转
019F:01F6FC22  XOR     EAX,EAX
019F:01F6FC24  PUSH    DWORD [FS:EAX]
019F:01F6FC27  MOV     [FS:EAX],ESP
019F:01F6FC2A  XOR     [EAX],EAX
019F:01F6FC2C  POP     DWORD [FS:00]
019F:01F6FC33  POP     EAX
019F:01F6FC34  CMP     DWORD [01F73934],BYTE +00
019F:01F6FC3B  JZ      01F6FC51
019F:01F6FC3D  PUSH    BYTE +0C
019F:01F6FC3F  MOV     ECX,01F73934
019F:01F6FC44  LEA     EAX,[EBP-08]
019F:01F6FC47  MOV     EDX,04
019F:01F6FC4C  CALL    01F6C39C
019F:01F6FC51  PUSH    DWORD [EBP-04]
019F:01F6FC54  PUSH    DWORD [EBP-08]
```

在那个 `ret` 处以下实在找不到什么有特点的设断处，所以就把下面的 10 来句全设上断，按 `G` 后，断在 `01f6fc2c` 处，继续运行，不一会就看到：

```assembly
019F:01FAC904  CLD
019F:01FAC905  SUB     EAX,ESP
019F:01FAC907  CALL    01FAC924
019F:01FAC90C  POP     EBX
019F:01FAC90D  POP     EAX
019F:01FAC90E  ADD     EAX,7C6060E4
019F:01FAC913  POP     ESP
019F:01FAC914  OR      ECX,ECX
019F:01FAC916  JZ      01FAC8FB
019F:01FAC918  MOV     [ECX],EAX
019F:01FAC91A  ADD     EAX,EBX
019F:01FAC91C  MOV     [ESP+1C],EAX
019F:01FAC920  POPA
019F:01FAC921  JMP     EAX    //从这里转到入口点，在这里 d eax 就可以
                               //看到入口点 48d154
019F:01FAC923  ADD     [EBX],DH
```

按 `f10` 运行到 `019f:00048d154` 处 suspend。

运行 `pedump`,把程序 dump 出来,然后运行没脱壳的程序，用 `importrec` 重建引入表 (具体操作参见看雪的书或者论坛精华里的相关文章)。

然后就只剩下去掉 NAG 窗口了，用 `trw2000` 载入脱好壳的程序，`bpx messageboxa` 断下后，`pmodule`, 然后在前一句设断，再次载入，断下后把 `call messageboxa` 改为 6 个 `nop`，执行，竟然非法操作，跟进下一个 `call` 发现是堆栈的问题，把前面的 `call messageboxa` 改为 `add esp ,10`(为什么是 10？因为不去掉 `call` 时执行 `call` 前后 `esp` 相差 10)，执行，成功。

记下修改前后的机器码，ok。

### 整理

先脱壳。

然后用 16 进制编辑工具打开脱壳后的文件。

查找：
`FF 15 80 c3 56 00`
改为：
`81 C4 10 00 00 00`

1. 这个软件是用 `VC 6.0` 编的，可以用 `BW2000` 直接找出入口点。
2. 这个软件可以用 `caspr1.1` 直接脱壳。
3. 原来的破解里还包括一个 `ebkey.dll`,其实和破解无关，只是把原来的 `ebgui.dll` 改了个名字，并且修改了一些资源，还用 `Asprotect 1.2` 加了壳。
4. 软件引入表比较简单，不用手动修改，用 `importREC` 安全自动搞定。
5. 这个软件本来使用 `rockey` 的狗加密，加密用的 Dll 为 `windows\system\keypro.dll`,搞定也比较简单。

## CAXA V2_4 升级版

软件：`CAXA`  
版本：`V2_4` 升级版  
破解者: 火翼`[CCG]`  
组织 : `[CCG]` (China Cracking Group)  
使用软件: `trw2000`

最近一直在研究 `CAXA`，这回是破解升级补丁的加密狗。

因为最近一直在研究 `CAXA`,而且我又对 `Rockey` 的狗比较熟悉(有同学在那里工作,hehe),所以知道读狗的 dll 为 `windows\system\keypro.dll`。

其实不知道也没关系，仔细看一下 `eb.exe` 的引入表就能找到，很明显的。

把 `keypro.dll` 拷贝到 `trw2000` 下的 dll 目录里，载入 `eb.exe` 后 `bpx keypro` 断下后 `bd *` ,`pmodule` ,返回 `44013c` 处，向上看发现有一个跳转可以避开读狗，于是在哪里下断点，再次载入，在 `440115` 处 `r fl z`,继续，可以正常使用所有功能，但会弹出一个授权使用的对话框，到这里才明白了原来上次解的 NAG 窗口不是原破解者加的，他只是改变了程序的资源，让那里显示他的网址，上次理解错了，不好意思。

```assembly
019F:00440105  MOV     DWORD [ESI],00589E64
019F:0044010B  CALL    0043FB50
019F:00440110  ADD     ESP,BYTE +04
019F:00440113  CMP     EAX,EBX
019F:00440115  JNZ     NEAR 004401FD             (NO JUMP)
019F:0044011B  PUSH    DWORD 005B9A90
019F:00440120  MOV     [005B9A96],BX
019F:00440127  MOV     [005B9A98],BX
019F:0044012E  MOV     [005B9A94],BX
019F:00440135  MOV     [005B9A92],BX
019F:0044013C  CALL    `KEYPRO!KEYPRO`   //第一次读狗的地方
019F:00440141  MOV     EDI,03
019F:00440146  MOV     ESI,63
019F:0044014B  CMP     [005B9A90],DI
```

因为有了上次经验，直接 `bpx messageboxa` 断下后 `bd*`,`pmodule`，返回到 `44027a` 处：

```assembly
019F:00440279  PUSH    EBX
019F:0044027A  CALL    `USER32!MessageBoxA` //  NAG 窗口
019F:00440280  LEA     ECX,[ESP+10]
019F:00440284  MOV     BYTE [ESP+E0],05
019F:0044028C  CALL    `MFC42!ord_00000320`
```

把 `call USER32!MessageBoxA` 改为 `add esp,10` ，全部搞定，具体原因参见上次的文章。

### 总结

其实这次这么容易主要是运气，北航海尔的软件工程师留了后门，可以不用读狗。升级补丁 [`www.caxa.com`](http://www.caxa.com) 有下载。

### 整理

用 16 进制编辑工具打开 `eb.exe`。

查找：
`0f 85 e2 00 00 00 68 90`
改为：
`e9 e3 00 00 00 90 68 90`

查找：
`ff 15 a0 52 58 00 8d`
改为：
`81 c4 10 00 00 00 8d`

***

# BeginLibraryItem "/Library/copyright.lbi" CLASS="p9z"  Copyright © 2000-2003 [看雪学院(www.pediy.com)](http://www.pediy.com) All Rights Reserved. # EndLibraryItem
