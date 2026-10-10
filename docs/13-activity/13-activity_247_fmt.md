
# 第一题做法-7byte

> 原文：[`www.pediy.com/kssd/pediy10/73971.html`](https://www.pediy.com/kssd/pediy10/73971.html)

目的：想从初级会员升为普通会员:P

思路：

`OpenUrlA`长度为`7byte`.主要是用的壳里的常用手法“堆栈切换”来实现的。

在全局内存中构造：

内存1：

低    `WinExecIAT`

      `OpenUrlA`地址

      字符串地址

高    `0x5` --- `WM_SHOW`

内存2：

      {内存1的地址}

`OpenUrlA`:

`xchg [内存2], esp`

`retn`

或

`OpenUrlA`:

`mov eax, 内存2`

`xchg [eax], esp`

`retn`

由于 CPU 对 `eax` 做了优化，以上两组指令均为 `7byte`;

流程为：`client` ---call---> `OpenUrlA` ---retn----> `winexec` ----retn---> `client`

在第二和第三个箭头过程中分别做了一次堆栈切换。

这里注意的是我们的 PE 内存可能不够堆栈的使用，这里的办法是把内存1写入 `reloc` 节，然后把前面节`.data`的虚拟内存大小加大为 PE header 中的

`SizeOfStackReserve = 100000 (1048576.)`。然后调整一下后面的节（还有属性要可写）。

## 1）

`import table`:

首先修改导入表在导入表加入 `kernel32.dll` 中的 `WinExec` 函数.

`import table` 的 RVA 为：`2014`,这个 PE 的 DLL 中先把具体文件和函数名写入注册表后面。

由于导入表的 `Original First Thunk` 没有太严格的要求，可以为 `NULL`。所以可以将 `kernel32.dll` 和 `user32.dll` 的 `Original First Thunk` 清为 `0`，

现在再加入一个 `kernel32.dll` 的 `WinExec` 的导入信息这里注意了，这个导入信息的 `Original First Thunk` 是要有的(`WinExec` 的第一个参数不为

`NULL`)

## 2）

找空间把数据和代码写入 PE 文件中。

先写入数据到最后

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00000BC0   A9 20 00 00 D0 10 40 00  D0 31 50 00 05 00 00 00   ?..?@.?P.....
00000BD0   65 78 70 6C 6F 72 65 20 68 74 74 70 3A 2F 2F   explorer http://
00000BE0   62 62 73 2E 70 65 64 69 79 2E 63 6F 6D 00 00 00   bbs.pediy.com...
00000BF0   00 00 00 00 00 00 00 00  C0 31 50 00 00 00 00 00   ........?P.....
```

调整`.data`段的虚拟内存大小为`100000`，把`rdata,data`和`reloc`都修改为可写可读。

将代码写入`.text`段

```
008710D0  > .  8725 F8319700  xchg    dword ptr [9731F8], esp
008710D6   .  C3            retn
```

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
000004D0   87 25 F8 31 50 00 C3 00                        ??P.?
```

## 3）修改重定位表

把代码和数据中用到的全局变量的地址，按格式写入重定位表中。

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00000A30   00 30 10 00 10 00 00 00  C4 31 C8 31 F8 31 00 00   .0......???..
```

## 4）加入导出表

在导入表后面加入导出表和 `OpenUrlA` 函数：

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
000006D0   00 00 00 00 00 00 00 00  00 00 00 00 11 21 00 00   ...............!..
000006E0   00 00 00 00 01 00 00 00  01 00 00 00 00 21 00 00   ...............!..
000006F0   04 21 00 00 E1 20 00 00  00 00 00 00 00 00 00 00   .!..?..........
00000700   D0 10 00 00 08 21 00 00  4F 70 65 6E 55 72 6C 41   ?...!..OpenUrlA
00000710   00 70 65 64 69 79 2E 64  6C 6C                  .pediy.dll
```

要调整一下`.rdata`的虚拟内存

OK!
