
# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy64064.htm`](https://www.pediy.com/kssd/pediy06/pediy64064.htm)

## 有用的结构体

### `PSP`(`Program Segment Prefix`)

现在介绍我们已经讨论了很多的 `PSP`。

`PSP` 结构如下：

```text
__________________________________________________
|   INT 20h(CD 20)              |<------+0000h
|________________________________|         Size:1 WORD
|    Pointer to the next segment  |<------+0002h
|________________________________|         Size:1 WORD
|                  Reserved      |<------+0004h
|________________________________|         Size:1 BYTE
|      Far call to INT 21h       |<------+0005h
|________________________________|         Size:5 BYTES
|       Saved INT 22h vector     |<------+000Ah
|________________________________|         Size:1 DWORD
|       Saved INT 23h vector     |<------+000Eh
|________________________________|         Size:1 DWORD
|       Saved INT 24h vector     |<------+0012h
|________________________________|         Size:1 DWORD
|                  Reserved      |<------+0016h
|________________________________|         Size:22 BYTES
|   Offset to Environment Segment |<------+002Ch
|________________________________|         Size:1 WORD
|                  Reserved      |<------+002Eh
|________________________________|         Size:46 BYTES
|         First default FCB       |<------+005Ch
|________________________________|         Size:16 BYTES
|        Second default FCB       |<------+006Ch
|________________________________|         Size:16 BYTES
|    Command Tail and default DTA |<------+0080h
|________________________________|         Size:180 BYTES
                                 Total Size:256 BYTES
```

因为这个结构非常重要，让我一步一步地解释吧。

`offset` `0000h`:
`INT 20h` 是终止程序的过时的方法，现在我们使用 `INT 21h` 的 `4Ch` 号函数。

`offset` `0002h`:
这是 `pointer to the next segment`，这个指针指向我们程序的下一段。我们利用它可以知道 `DOS` 能给我们多少内存(它指向的偏移地址减去 `PSP` 的偏移地址 `0000`)。它将以段返回给我们内存，所以我们必须把它乘以 16 来得到字节数。

`offset` `0005h`:
这是调用 `INT 21h` 的一种奇特方式。而且，当然，我们可以利用它来达到我们的目的。这个函数用到 `CL` 而不是 `AH`，而且我们只能在低于 `24h` 时调用这个函数。我将在 `TUNNELING` 这一章里介绍更多。

`offset` `000Ah`:
这里存储的是原先的 `INT 22h` 中断向量。`INT 22h` 当程序使用如下方法终止执行时接受控制：
- `INT 20h`
- `INT 27h`
- `INT 21h`(`00h`, `21h`, `4Ch` 函数)

`offset` `000Eh`:
这里存储另一个中断的向量，`INT 23h`。这个中断处理 `CTRL+C` 按键组合。

`offset` `0012h`:
存储在这里的另外一个中断，`INT 24h`。这个中断处理严重错误。这种类型错误的例子：当你的软驱里面没有软盘，或者软盘被写保护了。

`offset` `002Ch`:
这里是环境块偏移地址的开始。

`offset` `005Ch`:
这里存储第一个缺省 `FCB`(`File Control Block` 文件控制块)。这种访问文件的方式通常情况下不会被程序使用(在这里是为了和低版本的 `DOS` 兼容)，但是病毒作者为了使病毒更隐蔽，经常使用它。你可以在 `FCB` 结构里看到更多的信息。

`offset` `006Ch`:
同上，这是第二个缺省 `FCB`。

`Offset` `0080h`:
这一段有两个功能：
- 存储命令尾(`command tail`)
- 存储 `DTA` 的缺省文件缓冲区

这两个功能不能同时使用，所以我们开始一个程序要考虑的第一件事就是命令尾(`command tail`)。如果我们需要它，我建议你把它保存到一个安全的地方(我们代码中的一个变量中)。命令尾的第一个字节(`80h`)约束了它的长度，而且这里，它保存了真正的参数。`DTA` 的结构将会在这一章中介绍。

### `FCB`(`File Control Block` 文件控制模块)

现在有两种类型的 `FCB`:普通和扩展类型。下面给出普通 `FCB`:

```text
________________________________________________
|  Drive Letter(0=actual,1=A...) |<----+0000h
|________________________________|         Size:1 BYTE
|    Blank padded file name       |<----+0001h
|________________________________|         Size:8 BYTES
|   Blank padded file extension   |<----+0009h
|________________________________|         Size:3 BYTES
|      Current block number       |<----+000Ch
|________________________________|         Size:1 WORD
|      Logical record size        |<----+000Eh
|________________________________|         Size:1 WORD
|            File size            |<----+0010h
|________________________________|         Size:1 DWORD
|            File date            |<----+0014h
|________________________________|         Size:1 WORD
|            File time            |<----+0016h
|________________________________|         Size:1 WORD
|            Reserved             |<----+0018h
|________________________________|         Size:8 BYTES
|  Record within current block    |<----+0020h
|________________________________|         Size:1 BYTE
|  Record access record number    |<----+0021h
|________________________________|         Size:1 DWORD
                                  Total Size:37 BYTES
```

而在一个扩展 `FCB` 中，上面所有的偏移地址会向后移 7 个字节，而开始的 7 个字节如下：

```text
________________________________________________
|  FF(Signature for extended FCB)  |<---- -0007h
|________________________________|          Size:1 BYTE
|             Reserved           |<---- -0006h
|________________________________|          Size:5 BYTES
|       File attribute           |<---- -0001h
|________________________________|          Size:1 BYTE
                                   Total Size:44 BYTES
```

检测 `FCB` 是普通的还是扩展的方法是看 `FCB` 的第一个字节是否为 `FFh`。如果是，那么就为扩展 `FCB`，否则就为普通 `FCB`。

有一种隐蔽病毒(`STEALTH`)方法就是改变 `FCB` 的某些值，我们将会在 `STEALTH` 一章里具体讨论。

### `MCB`(`Memory Control Block` 内存控制模块)

我们将在驻留内存病毒这一章介绍它(下一章)。下面给出 `MCB`:

```text
________________________________________________
|     ID(Z=last,M=there're more)   |<----+0000h
|________________________________|         Size:1 BYTE
|     Address of associated PSP    |<----+0001h
|________________________________|         Size:1 WORD
|  Number of paras in allocated mem|<----+0003h
|________________________________|         Size:1 BYTE
|               Unused             |<----+0005h
|________________________________|         Size:11 BYTES
|             Block Name           |<----+0008h
|________________________________|         Size:8 BYTES
|     Zone of allocated memory      |<----+0010h
|________________________________|         Size:?? PARAS
                                   Total Size: VARIABLE
```

### `DTA`(`Disk Transfer Area` 磁盘交换区)

在病毒写作中这个结构非常重要。让我们看：

```text
________________________________________________
|  Drive Letter(equal than above)  |<----+0000h
|________________________________|         Size: 1 BYTE
|           Search Template        |<----+0001h
```



## DTA 结构

```
|___________________________________________|      Size: 11 BYTES   

|                   Reserved                |<----+000Ch   

|___________________________________________|      Size: 9 BYTES   

|                 File attribute            |<----+0015h   

|___________________________________________|      Size: 1 BYTE   

|                  File time                |<----+0016h   

|___________________________________________|      Size: 1 WORD   

|                  File date                |<----+0018h   

|___________________________________________|      Size: 1 WORD   

|                  File size                |<----+001Ah   

|___________________________________________|      Size: 1 DWORD   

|    ASCIIZ Filename+extention              |<----+001Eh   

|___________________________________________|      Size: 13 BYTES   

                                            Total Size: 43 BYTES   
```

原始 `DTA` 保存在 `PSP` 的偏移地址 `80h` 处。我们可以利用 `INT 21h` 的 `1Ah` 功能保存它。

## IVT (Interrupt Vector Table 中断向量表)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个并不是“真正的”结构体。恩...让我解释一下吧...`IVT` 是保存所有中断向量的地方(哇，天才啊！)。所有的中断向量的位置为 `number_of_interrupt*4` (中断号*4)。假如我们想把 `INT 21h` 中断向量赋给 `DS:DX`...则：

```
xor ax,ax
mov ds,ax
lds dx,ds:[21h*4]
```

为什么我们清除 `DS`?因为 `IVT` 是从地址 `0000:0000` 开始到高地址内存的。这种操作(不通过使用 `DOS`)是获得/赋给一个中断的向量的直接方法。有关更多的介绍将在驻留内存病毒(`RESIDENT VIRUSES`)一章中介绍。嗨...我忘记了示意图了:)

```
  ___________________________________________________

|           INT 00h vector            |<-----+0000h

|_____________________________________
|            Size:1 DWORD

|           INT 01h vector            |<-----+0004h

|_____________________________________
|            Size:1 DWORD

|

//////////////////

|_____________________________________
|           INT FEh vector            |<-----+03FCh

|_____________________________________
|            Size:1 DWORD

|           INT FFh vector            |<-----+0400h

|_____________________________________
|            Size:1 DWROD

                                            Total Size:1024 BYTES
```

你能想象得到“断”行表示有 256 个中断，我不得不优化这篇教程(我可不想花 5 页来表示它！)

## SFT (System File Table 系统文件表)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个结构真得很酷。它能帮助你使你的代码更健壮，更优化。它类似于 `FCB`，但是，正如你看到的，它更强大。利用这个表，我们可以使病毒更隐蔽，改变文件指针的打开模式，属性...这里你看到的使 `DOS 4` 以上版本的结构(我相信在这个世界上已经没有人用 `DOS 3` 或更低了吧)。好了，如果你想为 `DOS 3` 编写代码，可以参考 `Ralph Brown` 的中断列表。但是 `DOS 3` 的 `SFT` 是和下面的非常类似的。重要的值在相同的地方:)

```
 ===================================== <----+0000h

‖    Pointer to next file table     ‖          Size:1 DWORD

‖===================================‖<----+0004h

‖  Number of files in this table    ‖-----------Size:1 WORD----------

‖===================================‖<----+0000h  [3Bh bytes per file]

 |  Number of file handles of file   |             Size:1 WORD

 |___________________________________

 |      File open mode(AH=3Dh)       |<-----+0002h

 |___________________________________
 |             Size:1 WORD

 |          File attribute           |<-----+0004h

 |___________________________________
 |             Size:1 BYTE

 |    Device info block(AX=4400h)    |<-----+0005h

 |___________________________________
 |             Size:1 WORD

 | If char device points next dev h. |<-----+0007h

 |       else point to DOS DPB       |             Size:1 DWORD

 |___________________________________

 |     Starting cluster of file      |<-----+000Bh

 |___________________________________
 |             Size:1 WORD

 |              File time             |<-----+000Dh

 |___________________________________
 |             Size:1 WORD

 |              File date             |<-----+000Fh

 |___________________________________
 |             Size:1 WORD

 |              File size             |<-----+0011h

 |___________________________________
 |             Size:1 DWORD

 |      Current offset in file        |<-----+0015h

 |___________________________________
 |             Size:1 DWORD

 |  Relative cluster within file of  |<-----+0019h----------[If Local File]

 |      last cluster accessed        |             Size:1 WORD

 |___________________________________

 |  Number of sector with dir entry  |<-----+001Bh

 |___________________________________
 |             Size:1 DWORD

 | Number of dir entry within sector |<-----+001Fh

 |___________________________________
 |             Size:1 BYTE

 |    Pointer to REDIRIFS records    |<-----+0019h----[Network redirector]

 |___________________________________
 |             Size:1 DWORD

 |                ???                |<-----+001Dh

 |___________________________________
 |-----------Size:3 BYTES-----------

 |      Filename in FCB format       |<-----+0020h

 |___________________________________
 |             Size:11 BYTES

 | Pointer to prev SFT sharing file* |<-----+002Bh

 |___________________________________
 |             Size:1 DWORD

 | Network machine num opened file*  |<-----+002Fh

 |___________________________________
 |             Size:1 WORD

 |      PSP segment of file owner     |<-----+0031h

 |___________________________________
 |             Size:1 WORD

 |  Offset to code segment of rec*   |<-----+0033h

 |___________________________________
 |             Size:1 WORD

 | Absolute clust num of last access |<-----+0035h

 |___________________________________
 |             Size:1 WORD

 |  Pointer to IFS driver for file   |<-----+0037h

 |___________________________________
 |             Size:1 DWORD

                                            Total Size:61 BYTES
```

Uhm...我忘记访问 `SFT` 的方法了...下面给出的程序把 `SFT` 赋给 `ES:DI`，把文件句柄保存到 `BX` 中。

```
GetSFT:
 mov  ax,1220h
 int  2Fh
 jc  BadSFT
 xor  bx,bx
 mov  ax,1216h
 mov  bl,byte ptr es:[di]
 int  2Fh
BadSFT:
 ret
```

我强烈地建议你把返回值保存到 `AX`/`BX` 中(`BX` 非常重要：这里我们用来保存文件句柄)。

标志(\*)的域将会被 `SHARE.EXE` 使用

## DIB (DOS Info Block DOS 信息模块)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

利用 `DIB`，我们能够访问那些重要的不能利用其它方法访问结构。这个结构并不是固定的在内存中的，我们必须利用 `INT 21h` 的 `52h` 功能。在 `DOS` 文档里没有这个函数的介绍。当我们调用这个函数的时候，我们将会在 `ES:BX` 里得到 `DIB` 的地址。你将会得到：

```
  ___________________________________________________

|       Pointer to first MCB         |<---- -0004h

|_____________________________________
|            Size:1 DWORD

|       Pointer to first DPB         |<-----+0000h

|_____________________________________
|            Size:1 DWORD

|    Pointer to DOS last buffer      |<-----+0004h

|_____________________________________
|            Size:1 DWORD

|        Pointer to $CLOCK           |<-----+0008h

|_____________________________________
|            Size:1 DWORD
```



### Pointer to CON
|   Pointer to CON               | <-----+000Ch   
|------------------------------|--------------|
|                              | Size:1 DWORD |

### Maximum sector length
|   Maximum sector length        | <-----+0010h   
|------------------------------|--------------|
|                              | Size:1 WORD  |

### Pointer to DOS first buffer
|   Pointer to DOS first buffer  | <-----+0012h   
|------------------------------|--------------|
|                              | Size:1 DWORD |

### Pointer to array of cur dir struc
|   Pointer to array of cur dir struc | <-----+0016h   
|------------------------------|--------------|
|                              | Size:1 DWORD |

### Pointer to SFT
|   Pointer to SFT              | <-----+001Ah   
|------------------------------|--------------|
|                              | Size:1 DWORD |

Total Size:34 BYTES

### DPB (Drive Parameter Block 驱动器参数模块)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个结构为我们提供了达到我们的目的的非常有用的信息。通过使用 `DIB` (看上文) 里的第二个指针，我们能够知道它的位置。下面给出：

__________________________________________________________

|   Drive Letter (0=A,1=B...)  | <----+0000h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Unit number within device driver | <----+0001h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Bytes per sector             | <----+0002h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Highest sect num within a cluster | <----+0004h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Shift count for clust to sectors  | <----+0005h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Number of reserved clusters    | <----+0006h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of FATS               | <----+0008h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Number of root directory entries  | <----+0009h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of first sector with data  | <----+000Bh   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of last sector with data   | <----+000Dh   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of sector per FAT     | <----+000Fh   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Sector number of first dir sector | <----+0010h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Address of device driver header  | <----+0012h   
|------------------------------|---------------|
|                              | Size:1 DWORD  |

|   Media ID byte               | <----+0016h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   00h if disk accessed, else FFh   | <----+0017h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Pointer to next DPB            | <----+0018h   
|------------------------------|---------------|
|                              | Size:1 DWORD  |

Total Size:28 BYTES

### 分区表

~~~~~~~~~~~~~~~~~~~~~~~~~~

所有编写启动病毒的人都知道这个结构，它是硬盘上的第一块。它总是在第一块的，无论在软盘还是在硬盘上。如果是硬盘，我们就叫它 `MBR` (Master Boot Record 主启动记录)，如果是软盘，就叫它启动扇区。

分区表是一个有着四个入口的数组，在偏移地址 `01BEh` 处。下面给出每个入口的格式：

__________________________________________________________

|   Boot indicator (Bootable=80h,   | <----+0000h   
|        Non bootable 00h)          |             |
|                                  | Size:1 BYTE |
|------------------------------|--------------|

|   Head where the partition begins  | <----+0001h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Sector where the partition begins | <----+0002h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Cylinder where the part.begins   | <----+0003h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   System indicator* (What OS?)     | <----+0004h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Head where partition ends        | <----+0005h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Sector where the partition ends  | <----+0006h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Cylinder where the partition ends | <----+0007h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Total blocks preceding partition | <----+0008h   
|------------------------------|---------------|
|                              | Size:1 DWORD  |

|   Total blocks in the partition    | <----+000Ch   
|------------------------------|---------------|
|                              | Size:1 DWORD  |

Total Size:16 BYTES

(*) `01`=12-bit `FAT`
    `04`=16-bit `FAT`

### BPB (Bios Parameter Block Bios 参数模块)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在以 `DOS` 为基础的系统中，启动记录以一个跳转(jump)作为开始，后面跟这一个结构体——`BPB`。

__________________________________________________________

|   OEM name and version (ASCII)  | <----+0000h   
|------------------------------|---------------|
|                              | Size:8 BYTES  |

|   Bytes per sector             | <----+0008h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Sectors per cluster          | <----+000Dh   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Reserved sector (starting at 0) | <----+000Eh   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of FATs               | <----+0010h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Total sectors in partition   | <----+0011h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Media descriptor             | <----+0013h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Sectors per FAT              | <----+0015h   
|------------------------------|---------------|
|                              | Size:1 BYTE   |

|   Sectors per track            | <----+0017h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of heads              | <----+0019h   
|------------------------------|---------------|
|                              | Size:1 WORD   |

|   Number of hidden sectors     | <----+001Dh   
|------------------------------|---------------|
|                              | Size:1 WORD   |

Total Size:29 BYTES

`4dd466d0e1e0cc123afbe504da2462ca`
