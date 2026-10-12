
# 扫雷之我析

> 原文：[`www.pediy.com/kssd/pediy12/139183.html`](https://www.pediy.com/kssd/pediy12/139183.html)

## 一、通过 `paimei` 获取程序初始化时的函数调用关系

`paimei` 的具体使用方法请参照《0day 软件漏洞分析技术》相关章节。如没有 `paimei`，请 email 给我，我可以发给您一份。

下图（图一和图二）是获得的程序函数调用关系，`_StartGame` 函数引起了我的注意，而且它调用了函数 `Rnd`。我们知道初始化雷区是需要随机数的，所以 `_StartGame` 函数是一个突破口，只要知道了该函数的细节流程，就可以破解扫雷游戏了。

先说明一点，只要知道游戏数据区的数据存放地址和格式，我们就能破解该游戏。

## 二、通过分析 `_StartGame` 函数获得的信息

游戏数据区首地址：`0x01005360`

游戏数据区行数存放地址：`0x01005338`

游戏数据区列数存放地址：`0x01005334`

游戏的每个单元格信息用一个字节来表示；数据区用二维数组的方式来描述单元格信息，每一行最多 30 列（两列是边界信息），最多有 30 行；不管游戏开始时有几列，数据区总是用 32 字节的信息表示一行。

![`test2.jiusheng.org/test/4.jpg`](http://test2.jiusheng.org/test/4.jpg)

图一：`StartGame` 函数调用了 `rand` 函数来获得地雷的随机行列位置

![`test2.jiusheng.org/test/5.jpg`](http://test2.jiusheng.org/test/5.jpg)

图二：程序在启动后调用 `_StartGame` 函数来初始化游戏数据区

## 三、对 `_StartGame` 函数的详细分析

### 1. 获得行列值并保存到全局变量

```assembly
MOV EAX,DWORD PTR DS:[10056AC] ;获得用户定义的行列值
MOV ECX,DWORD PTR DS:[10056A8]
PUSH EBX
PUSH ESI
PUSH EDI
XOR EDI,EDI
CMP EAX,DWORD PTR DS:[1005334] ;和全局变量中的列值比较
MOV DWORD PTR DS:[1005164],EDI
JNZ SHORT winmine.010036A4
CMP ECX,DWORD PTR DS:[1005338] ;和全局变量中的行值比较
JNZ SHORT winmine.010036A4
PUSH 4
JMP SHORT winmine.010036A6
PUSH 6
POP EBX
```

### 2. 调用 `rand` 函数生成雷的随机位置

```assembly
MOV DWORD PTR DS:[1005334],EAX ;保存当前的行列值
MOV DWORD PTR DS:[1005338],ECX
CALL winmine.01002ED5 ;调用 ClearField 函数部分初始化数据区
MOV EAX,DWORD PTR DS:[10056A4] ;获得初始化雷数
MOV DWORD PTR DS:[1005160],EDI
MOV DWORD PTR DS:[1005330],EAX
PUSH DWORD PTR DS:[1005334] ;用列数做 rand 参数，生成雷的列位置
CALL winmine.01003940
PUSH DWORD PTR DS:[1005338]
MOV ESI,EAX
INC ESI
CALL winmine.01003940 ;用行数做 rand 参数，生成雷的行位置
INC EAX
MOV ECX,EAX
SHL ECX,5 ;行数乘以 32，因为一行用 32 字节表示
TEST BYTE PTR DS:[ECX+ESI+1005340],80 ;字节的最高位表示是否为雷
JNZ SHORT winmine.010036C7 ;如果是雷，则继续寻找雷位置
```

以下处理该位置不是雷的情况：

```assembly
SHL EAX,5
LEA EAX,DWORD PTR DS:[EAX+ESI+1005340]
OR BYTE PTR DS:[EAX],80 ;将该字节最高位置 1，表示该位置是雷
DEC DWORD PTR DS:[1005330]
JNZ SHORT winmine.010036C7
```

### 3. 善后工作

```assembly
MOV ECX,DWORD PTR DS:[1005338]
IMUL ECX,DWORD PTR DS:[1005334]
MOV EAX,DWORD PTR DS:[10056A4]
SUB ECX,EAX
PUSH EDI
MOV DWORD PTR DS:[100579C],EDI
MOV DWORD PTR DS:[1005330],EAX
MOV DWORD PTR DS:[1005194],EAX
MOV DWORD PTR DS:[10057A4],EDI
MOV DWORD PTR DS:[10057A0],ECX
MOV DWORD PTR DS:[1005000],1
CALL winmine.0100346A
PUSH EBX ; /Arg1
CALL winmine.01001950 ; \winmine.01001950
POP EDI
POP ESI
POP EBX
RETN
```

## 四、对该游戏的破解

在文章的开始我就讲过，只要知道了游戏数据区的存放地址和格式，我们就能破解该游戏。下图是数据区在内存中的情况：

![`test2.jiusheng.org/test/6.jpg`](http://test2.jiusheng.org/test/6.jpg)

图三：数据区的内存情况

在上图中可以看到，从地址 `0x01005360` 开始的 32 个字节存放的是游戏第一行单元格的信息，以此类推，从地址 `0x01005380` 开始的 32 个字节存放的是游戏第二行单元格的信息……

每个单元格用一个字节表示，含义如下：

- `0F`：初始化时的状态
- `10`：代表边线
- `40`：代表无雷，无数字，无标记
- `4X`：`X` 表示该格子周围有 `X` 颗雷
- `8F`：代表雷
- `8E`：代表雷标记，低四位为 `E`

走到这步了，我相信你一定能写出破解程序了，呵呵。
