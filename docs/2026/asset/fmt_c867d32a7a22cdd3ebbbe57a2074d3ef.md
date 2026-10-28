
# Billy Belceb 病毒编写教程(DOS篇) 有用的结构体

## 有用的结构体

现在介绍我们已经讨论了很多的`PSP`。

`%PSP(Program Segment Prefix)%`

### `PSP`结构如下：

```
          INT 20h(CD 20)          |<------+0000h
                                   Size:1 WORD
     Pointer to the next segment    |<------+0002h
                                   Size:1 WORD
             Reserved              |<------+0004h
                                   Size:1 BYTE
        Far call to INT 21h        |<------+0005h
                                   Size:5 BYTES
       Saved INT 22h vector        |<------+000Ah
                                   Size:1 DWORD
       Saved INT 23h vector        |<------+000Eh
                                   Size:1 DWORD
       Saved INT 24h vector        |<------+0012h
                                   Size:1 DWORD
             Reserved              |<------+0016h
                                   Size:22 BYTES
   Offset to Environment Segment   |<------+002Ch
                                   Size:1 WORD
             Reserved              |<------+002Eh
                                   Size:46 BYTES
         First default FCB         |<------+005Ch
                                   Size:16 BYTES
        Second default FCB         |<------+006Ch
                                   Size:16 BYTES
    Command Tail and default DTA   |<------+0080h
                                   Size:180 BYTES
                                     Total Size:256 BYTES
```

因为这个结构非常重要，让我一步一步地解释吧。

`offset 0000h`:
`INT 20h`是终止程序的过时的方法，现在我们使用`INT 21h`的`4Ch`号函数。

`offset 0002h`:
这是`pointer to the next segment`，这个指针指向我们程序的下一段。我们利用它可以知道`DOS` 能给我们多少内存(它指向的偏移地址减去`PSP`的偏移地址`0000`)。它将以段返回给我们内存，所以我们必须把它乘以`16`来得到字节数。

`offset 0005h`:
这是调用`INT 21h`的一种奇特方式。而且，当然，我们可以利用它来达到我们的目的。这个函数用到`CL`而不是`AH`，而且我们只能在低于`24h`时调用这个函数。我将在`TUNNELING`这一章里介绍更多。

`offset 000Ah`:
这里存储的是原先的`INT 22h`中断向量。`INT 22h`当程序使用如下方法终止执行时接受控制：
- `INT 20h`
- `INT 27h`
- `INT 21h(00h,21h,4Ch`函数)

`offset 000Eh`:
这里存储另一个中断的向量，`INT 23h`。这个中断处理`CTRL+C`按键组合。

`offset 0012h`:
存储在这里的另外一个中断，`INT 24h`。这个中断处理严重错误。这种类型错误的例子：当你的软驱里面没有软盘，或者软盘被写保护了。

`offset 002Ch`:
这里是环境块偏移地址的开始。

`offset 005Ch`:
这里存储第一个缺省`FCB(File Control Block`文件控制块)。这种访问文件的方式通常情况下不会被程序使用(在这里是为了和低版本的`DOS`兼容)，但是病毒作者为了使病毒更隐蔽，经常使用它。你可以在`FCB`结构里看到更多的信息。

`offset 006Ch`:
同上，这是第二个缺省`FCB`。

`Offset 0080h`:
这一段有两个功能：
----存储命令尾`command tail`
---存储`DTA`的缺省文件缓冲区
这两个功能不能同时使用，所以我们开始一个程序要考虑的第一件事就是命令尾`command tail`。如果我们需要它，我建议你把它保存到一个安全的地方(我们代码中的一个变量中)。命令尾的第一个字节`80h`约束了它的长度，而且这里，它保存了真正的参数。`DTA`的结构将会在这一章中介绍。

`%FCB(File Control Block 文件控制模块)%`

现在有两种类型的`FCB`:普通和扩展类型。下面给出普通`FCB`:

```
  Drive Letter(0=actual,1=A...) |<----+0000h
                               Size:1 BYTE
    Blank padded file name      |<----+0001h
                               Size:8 BYTES
   Blank padded file extension  |<----+0009h
                               Size:3 BYTES
      Current block number      |<----+000Ch
                               Size:1 WORD
      Logical record size       |<----+000Eh
                               Size:1 WORD
            File size           |<----+0010h
                               Size:1 DWORD
            File date           |<----+0014h
                               Size:1 WORD
            File time           |<----+0016h
                               Size:1 WORD
            Reserved            |<----+0018h
                               Size:8 BYTES
   Record within current block  |<----+0020h
                               Size:1 BYTE
   Record access record number  |<----+0021h
                               Size:1 DWORD
                                  Total Size:37 BYTES
```

而在一个扩展`FCB`中，上面所有的偏移地址会向后移`7`个字节，而开始的`7`个字节如下：

```
   FF(Signature for extended FCB)    |<---- -0007h
                                     Size:1 BYTE
              Reserved               |<---- -0006h
                                     Size:5 BYTES
          File attribute             |<---- -0001h
                                     Size:1 BYTE
                                       Total Size:44 BYTES
```

检测`FCB`是普通的还是扩展的方法是看`FCB` 的第一个字节是否为`FFh`。如果是，那么就为扩展`FCB`，否则就为普通`FCB`。

有一种隐蔽病毒`STEALTH`方法就是改变`FCB`的某些值，我们将会在`STEALTH`一章里具体讨论。

`%MCB(Memory Control Block 内存控制模块)%`

我们将在驻留内存病毒这一章介绍它(下一章)。下面给出`MCB`:

```
     ID(Z=last,M=there're more)    |<----+0000h
                                   Size:1 BYTE
     Address of associated PSP     |<----+0001h
                                   Size:1 WORD
  Number of paras in allocated mem |<----+0003h
                                   Size:1 BYTE
               Unused              |<----+0005h
                                   Size:11 BYTES
             Block Name            |<----+0008h
                                   Size:8 BYTES
     Zone of allocated memory      |<----+0010h
                                   Size:?? PARAS
                                     Total Size: VARIABLE
```

`%DTA(Disk Transfer Area 磁盘交换区)%`

在病毒写作中这个结构非常重要。让我们看：

```
   Drive Letter(equal than above)  |<----+0000h
                                   Size: 1 BYTE
         Search Template           |<----+0001h
                                   Size: 11 BYTES
             Reserved              |<----+000Ch
                                   Size: 9 BYTES
          File attribute           |<----+0015h
                                   Size: 1 BYTE
             File time             |<----+0016h
                                   Size: 1 WORD
             File date             |<----+0018h
                                   Size: 1 WORD
```
