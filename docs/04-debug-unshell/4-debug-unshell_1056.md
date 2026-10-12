
# TELock 加壳的 Import Table

> 原文：[`www.pediy.com/kssd/pediy04/kanxue486.htm`](https://www.pediy.com/kssd/pediy04/kanxue486.htm)

目的：在内存中将 `FlashFXP` 未被破坏的 `IT` 完整地提取出来

（至于找 `TELock` 的 `OEP`、修复的方法等，`zombieys`、`hying`、`liotta` 已经讲得很详细了）

## 第一步：找到 IT 的位置

找 `IT` 自然要依赖于 `IT` 的结构特征。`IT` 是个数组，数组中的每个元素是如下的一个结构(`#include <winnt.h>`)，对应着引入的一个 `DLL`：

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD  Characteristics;               // 0 for terminating null import descriptor
        DWORD  OriginalFirstThunk;            // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    };
    DWORD  TimeDateStamp;                     // 0 if not bound,
                                               // -1 if bound, and real date\ttime stamp
                                               //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                               // O.W. date/time stamp of DLL bound to (Old BIND)
    DWORD  ForwarderChain;                    // -1 if no forwarders
    DWORD  Name;                              //DLL name
    DWORD  FirstThunk;                        // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
```

`PE Loader/unpacker` 在加载 `DLL` 时，过程大致如下：

1、先根据 `Name` 用 `LoadLibraryA` 加载一个 `DLL`；

2、根据 `FirstThunk` 找到 `IMAGE_IMPORT_BY_NAME` 数组，用 `GetProcAddress` 取得该 `DLL` 中的相应函数的地址填入 `IAT`。`FirstThunk` 就是指向 `IAT`。

```c
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD   Hint;
    BYTE   Name[1];
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

找 `IT` 的位置的过程和上面的过程有些相反。利用调试器设 `bpx LoadLibraryA`、`bpx GetProcAddress` 断点，我们能够直接得到的条件是 `DLL` 文件名

字符串及该串的内存地址、每个函数的 `Hint` 和函数名的内存地址、`IAT` 的位置（即 `GetProcAddress` 的返回值所要写入的位置），这样就至少有 3 条途

径可以找到 `IT` 的位置，一是根据 `DLL Name`（在看雪教学中已有详细描述），第二条途径是根据 `Hint` 和函数名的地址，第三条则是根据 `IAT` 的位置

，这 3 条途径用的是同一种方法（这是找 `IT` 的一种通用方法，并不限于 `TELock` 加的壳）。

### 1、用 BPMB LoadLibraryA X DO "d *(esp+4)"设断点

看见加载 `oleaut32.dll`：

```
0023:00533DBC 61656C6F  32337475  6C6C642E  00000000       oleaut32.dll....
0023:00533DCC 69726156  43746E61  676E6168  70795465       VariantChangeTyp
0023:00533DDC 00784565  61560000  6E616972  706F4374       eEx...VariantCop
0023:00533DEC 646E4979  00000000  69726156  43746E61       yInd....VariantC
0023:00533DFC 7261656C  00000000  53737953  6E697274       lear....SysStrin
0023:00533E0C 6E654C67  00000000  46737953  53656572       gLen....SysFreeS
0023:00533E1C 6E697274  00000067  52737953  6C6C4165       tring...SysReAll
0023:00533E2C 7453636F  676E6972  006E654C  79530000       ocStringLen...Sy
```

### 2、先尝试第一条途径

字符串 `"oleaut32.dll"` 的 `RVA` 为 `00133DBC`，于是在内存中搜索这个 `RVA`，希望能够找到存放有该 `RVA` 值的某个

`IMAGE_IMPORT_DESCRIPTOR` 结构，也就等于是找到了 `IT`。

```
:s cs:400000 l 178000 bc,3d,13,00

Pattern not found
```

没有找到。因为 `TELock` 已经把相应位置上的值清 0 了。（实际上也可以用下一个 `DLL` 的文件名来进行搜索，因为 `TELock` 此时尚未处理下一个 `DLL`，

它不可能提前把下一个 `DLL` 的 `IMAGE_IMPORT_DESCRIPTOR` 结构清掉。）

### 3、再尝试第 2 条途径

注意到第一个函数 `VariantChangeTypeEx` 的 `Hint` 的地址是 `00533DCA`，于是搜索 `RVA` 值 `00133DCA`，希望能够找到 `FirstThunk`

所指向的那个数组（其实该数组也就是 `IAT`），进而再找到 `FirstThunk` 的位置。

```
:s cs:400000 l 178000 ca,3d,13,00

Pattern found at 001B:0053328C (0013328C)
```

找到之后看一下 `IAT`。由于此时还未把 `GetProcAddress` 的返回值写入 `IAT`，所以此时 `IAT` 中存放的都是指向 `IMAGE_IMPORT_BY_NAME` 的指针（`RVA`）

：

```
:dd 53328c

001B:0053328C 00133DCA  00133DE0  00133DF2  00133E02       .=...=...=...>..
001B:0053329C 00133E12  00133E22  00133E38  00000000       .>..">..8>......
001B:005332AC 00133E5A  00133E68  00133E76  00133E84       Z>..h>..v>...>..
001B:005332BC 00133E98  00000000  00133EBC  00133ECE       .>.......>...>..
001B:005332CC 00133EE2  00133EF6  00133F06  00133F14       .>...>...?...?..
001B:005332DC 00133F24  00133F36  00133F44  00133F58       $?..6?..D?..X?..
001B:005332EC 00133F70  00000000  00133F96  00133FA2       p?.......?...?..
001B:005332FC 00133FAE  00133FBA  00133FC6  00133FDC       .?...?...?...?..
```

### 4、再看究竟是哪个 FirstThunk 指向了上面的 IAT

由于该 `IAT` 的 `RVA` 为 `0013328C`，故

```
:s cs:400000 l 178000 8c,32,13,00

Pattern found at 001B:0053304C (0013304C)
```

### 5、这样就找到了 oleaut32.dll 所对应的 FirstThunk 的位置

位置是 `0053304C`。在此位置附近看一下，就可以找到 `IT` 的起始位置为 `00533000`

，长度为 `0x1CC`（问号表示未被映射的内存空间；最初的 4 个 `IMAGE_IMPORT_DESCRIPTOR` 结构已经被清除了一部分）：

```
001B:00532FE0 ????????  ????????  ????????  ????????       ................
001B:00532FF0 ????????  ????????  ????????  ????????       ................
001B:00533000 00000000  00000000  00000000  00000000       ................
001B:00533010 001331CC  00000000  00000000  00000000       .1..............
001B:00533020 00000000  0013326C  00000000  00000000       ....l2..........
001B:00533030 00000000  00000000  0013327C  00000000       ........|2......
001B:00533040 00000000  00000000  00000000  0013328C       .............2..
001B:00533050 00000000  00000000  00000000  00133E4C       ............L>..
001B:00533060 001332AC  00000000  00000000  00000000       .2..............
001B:00533070 00133EAE  001332C4  00000000  00000000       .>...2..........
001B:00533080 00000000  00133F88  001332F4  00000000       .....?...2......
001B:00533090 00000000  00000000  001346D4  0013349C       .........F...4..
```

## 第二步：找到破坏 IT 的代码

由于 `TELock` 清除了 `IT` 中指向 `DLL` 文件名的 `RVA`，故只要设个 `bpm` 断点，看它什么时候清这个文件名的，就可以找到破坏 `IT` 的代码。

第一个 `DLL` 文件名的 `RVA` 存放在 `0053300C` 处，故 `BPM 00533000+0C W do "dd 533000"`，

```
001B:00576C16  8B95BEAF4000        MOV       EDX,[EBP+0040AFBE]
001B:00576C1C  8BB5AEAF4000        MOV       ESI,[EBP+0040AFAE]
001B:00576C22  85F6                TEST      ESI,ESI
001B:00576C24  0F849D030000        JZ        00576FC7
001B:00576C2A  03F2                ADD       ESI,EDX
001B:00576C2C  83A5AAB0400000      AND       DWORD PTR [EBP+0040B0AA],00
001B:00576C33  8B460C              MOV       EAX,[ESI+0C]
001B:00576C36  83660C00            AND       DWORD PTR [ESI+0C],00      //清除指向每个 DLL 的 Name 的 RVA 值
001B:00576C3A  85C0                TEST      EAX,EAX
001B:00576C3C  0F8485030000        JZ        00576FC7
001B:00576C42  03C2                ADD       EAX,EDX
001B:00576C44  8BD8                MOV       EBX,EAX
001B:00576C46  50                  PUSH      EAX
001B:00576C47  FF9518AF4000        CALL      [EBP+0040AF18]
001B:00576C4D  85C0                TEST      EAX,EAX
001B:00576C4F  0F858C000000        JNZ       00576CE1
001B:00576C55  53                  PUSH      EBX
```

在第一次执行上面 `CS:00576C36` 处的 `AND` 之前将 `533000`～`535FFF` 之间的内容（共 `0x3000` 字节，不仅包含 `IT`，还包含 `IAT` 等）`dump` 出来就行了。

所以觉得 `TELock` 目前还没有 `ASProtect` 强 `4dd466d0e1e0cc123afbe504da2462ca`
