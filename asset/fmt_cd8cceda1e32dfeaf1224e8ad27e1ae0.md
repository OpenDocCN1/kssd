
## 行号表

这是基于文件的行号表的偏移量，一个行号表使源文件的行号和一个给定的行所产生的代码地址相关联。在现代的调试格式如 `CodeView` 格式中，行号信息是作为调试信息的一部分存储的。在 `COFF` 调试格式中，然而，行号信息是和符号名/符号类型分开存储的。通常，只有 `code` 节(如 `.text`)有行号。在 `EXE` 文件中，行号是在节的 `raw data`(原始数据)之后向文件尾累加的。在 `OBJ` 文件中，一个节的行号表是在原始节数据和这个节的重定位表之后开始的。

`Number Of Relocations:`

在节的行号表中的行号的数值(上面的 `PointerToLinenumbers` 域)。

`Characteristics:`

大多数程序员叫做标志(`flag`)，在 `COFF/PE` 格式中叫做特征(`characterstic`)。这个域是一些表面节属性(如代码/数据，可读，或可写)的标志。要看所有可能的节属性的列表，看看定义在 `WINNT.H` 中的 `IMAGE_SCN_XXX_XXX`。下面给出一些比较重要的标志：

*   `0x00000020` 这个节包含代码。通常和可执行标志(`0x80000000`)联合设置。
*   `0x00000040` 这个节包含了初始化了的数据(`initialized data`)。除了可执行和 `.bss` 节之外几乎所有的节都有这个标志。
*   `0x00000080` 这个节包含了未初始化的数据(`uninitialized data`)，如 `.bss` 节。
*   `0x00000200` 这个节包含了一些注释或者一些其它类型的信息。这个节的一个典型利用是由编译器所设置的 `.drectve` 节，这个节包含了连接器的命令。
*   `0x00000800` 这个节的内容是不应该放在最终的 `EXE` 文件中的。这些节被编译器/汇编器用来传递信息给连接器。
*   `0x02000000` 在它被装载之后，进程就不再需要它了，这个节就可以被丢弃。最普通的可丢弃的节是基址重定位节(`.reloc`)。
*   `0x10000000` 这个节是可共享的。当使用一个 `DLL` 时，这个节中的数据将会通过 `DLL` 来给所有的进程共享。数据节的默认是不共享的。用更专业的术语，一个共享节告诉内存管理器设置这个节的页映射使得所有使用这个 `DLL` 的进程指向内存中的同一个物理页。要使一个节可共享的，在连接的时候使用共享(`SHARED`)属性。如：
    ```
    LINK /SECTION:MYDATA,RWS...
    ```
    就告诉了连接器一个叫做 `MYDATA` 的节是可读的，可写的，而且是共享的。
*   `0x20000000` 这个节是可执行的。这个标志通常在"包含代码"的标志(`0x00000020`)被设置后设置。
*   `0x40000000` 这个节是可读的。这个标志几乎在 `EXE` 文件的所有节中都被设置。
*   `0x80000000` 这个节是可写的。如果这个标志在一个 `EXE` 文件的节中没有被设置，装载器就会标志内存映射页为只读的或只能执行的。有这个属性的典型的节是 `.data` 和 `.bss`。有趣的是，`.idata` 节也设置了这个属性。

## 要改变的东西

下面，我将介绍在编写一个普通的 `PE` 病毒时的一些改变。假设你要编写一个会增加 `PE` 文件的最后一个节内容的病毒，这个在我们看来更容易成功的技术，然而添加一个节更容易。让我们看看一个病毒是怎么来改变一个可执行文件的头。我使用了 `Lord Julus[SLAM]` 的 `INFO-PE` 程序。

```
-------- DOS INFORMATION --------------------------------------------------- 

Analyzed File: GOAT002.EXE 

DOS Reports: 

            ?File Size  - 2000H      (08192d) 

            ?File Time  - 17:19:46   (hh:mm:ss) 

            ?File Date  - 11/06/1999 (dd/mm/yy) 

            ?Attributes : Archive 

[...] 

-------- PE Header ---------------------------------------------------------- 

  --------------- 

 ‖O_DOS |O_PE  ‖(Offset from Dos Header / PE Header 

 ‖------|------‖ 

 |0100H  |0000H | PE Header Signature - PE/0/0 

 |0104H  |0004H | The machine for this EXE is Intel 386 (value = 014CH) 

 |0106H  |0006H | Number of sections in the file - 0004H 

 |0108H  |0008H | File was linked at : 23/03/2049 

 |010CH  |000CH | Pointer to Symbol Table : 00000000H 

 |0110H  |0010H | Number of Symbols : 00000000H 

 |0114H  |0014H | Size of the Optional Header : 00E0H 

 |       |       | 

 |0116H  |0016H | File Characteristics - 818EH : 

 |       |       | ?File is executable 

 |       |       | ?Line numbers stripped from file 

 |       |       | ?Local symbols stripped from file 

 |       |       | ?Bytes of machine word are reversed 

 |       |       | ?32 bit word machine 

 |       |       | ?Bytes of machine word are reversed 

 ‖______|_____‖ 

-------- PE Optional Header ----------------------------------- 

  --------------- 

 ‖O_DOS |O_PE  ‖(Offset from Dos Header / PE Header 

 ‖------|------‖ 

 |0118H  |0018H | Magic Value              : 010BH (`Θ`) 

 |011AH  |001AH | Major Linker Version     : 2 

 |011BH  |001BH | Minor Linker Version     : 25 

 |       |       | Linker Version           : 2.25 

 |011CH  |001CH | Size of Code             : 00001200H 

 |0120H  |0020H | Size of Initialized Data : 00000600H 

 |0124H  |0024H | Size of Uninitialized Data : 00000000H 

 |0128H  |0028H | Address of Entry Point   : 00001000H 

 |012CH  |002CH | Base of Code (.text ofs.)  : 00001000H 

 |0130H  |0030H | Base of Data (.bss ofs.)   : 00003000H 

 |0134H  |0034H | Image Base               : 00400000H 

 |0138H  |0038H | Section Alignment        : 00001000H 

 |013CH  |003CH | File Alignment           : 00000200H 

 |0140H  |0040H | Major Operating System Version : 1 

 |0142H  |0042H | Minor Operating System Version : 0 

 |0144H  |0044H | Major Image Version      : 0 

 |0146H  |0046H | Minor Image Version      : 0 

 |0148H  |0048H | Major SubSystem Version  : 3 

 |014AH  |004AH | Minor SubSystem Version  : 10 

 |014CH  |004CH | Reserved Long            : 00000000H 

 |0150H  |0050H | Size of Image            : 00006000H 

 |0154H  |0054H | Size of Headers          : 00000400H 

 |0158H  |0058H | File Checksum            : 00000000H 

 |015CH  |005CH | SubSystem                : 2 

 |       |       |  Image runs in the Windows GUI subsystem 

 |015EH  |005EH | DLL Characteristics      : 0000H 

 |0160H  |0060H | Size of Stack Reserve    : 00100000H 

 |0164H  |0064H | Size of Stack Commit   : 00002000H 

 |0168H  |0068H | Size of Heap Reserve   : 00100000H 

 |016CH  |006CH | Size of Heap Commit    : 00001000H 

 |0170H  |0070H | Loader Flags           : 00000000H 

 |0174H  |0074H | Number Directories     : 00000010H 

[...] 

------- PE Section Headers --------------------------------- 

  --------------------------- 

 ‖O_DOS |O_PE  ‖(Offset from Dos Header / PE Header 

 ‖------|------‖[...] 

 |0270H  |0170H | Section name          : .reloc 

 |0278H  |0178H | Physical Address      : 00001000H 

 |027CH  |017CH | Virtual Address       : 00005000H 

 |0280H  |0180H | Size of RAW data      : 00000200H 

 |0284H  |0184H | Pointer to RAW data   : 00001C00H 

 |0288H  |0188H | Pointer to relocations : 00000000H 

 |028CH  |018CH | Pointer to line numbers : 00000000H 

 |0290H  |0190H | Number of Relocations  : 0000H 

 |0292H  |0192H | Number of line numbers : 0000H 

 |0294H  |0194H | Characteristics       : 50000040H 

 |       |       | ?Section contains initialized data. 

 |       |       | ?Section is shareable. 

 |       |       | ?Section is readable. 

 |       |       | 

 ‖______|_____‖ 
```

这是一个正常文件，没有被感染。下面是同一个文件，但是被我的 `Aztec` 病毒(一个 `Ring-3` 病毒例子，看下面的)感染了。

```
------------ DOS INFORMATION ----------------------------------------------------------------- 

Analyzed File: GOAT002.EXE 

DOS Reports: 

           ?File Size  - 2600H      (09728d) 

           ?File Time  - 23:20:58   (hh:mm:ss) 

           ?File Date  - 22/06/1999 (dd/mm/yy) 

           ?Attributes : Archive 

[...] 

-------------- PE Header ----------------------------------------------------------------- 

--------------- 

‖O_DOS |O_PE  ‖(Offset from Dos Header / PE Header 

‖------|------‖[...] 

 |0100H |0000H  | PE Header Signature - PE/0/0 

 |0104H |0004H  | The machine for this EXE is Intel 386 (value = 014CH) 

 |0106H |0006H  | Number of sections in the file - 0004H 

 |0108H |0008H  | File was linked at : 23/03/2049 

 |010CH |000CH  | Pointer to Symbol Table : 00000000H 

 |0110H |0010H  | Number of Symbols : 00000000H 

 |0114H |0014H  | Size of the Optional Header : 00E0H 

 |      |        | 

 |0116H |0016H  | File Characteristics - 818EH : 

 |      |        | ?File is executable 

 |      |        | ?Line numbers stripped from file 

 |      |        | ?Local symbols stripped from file 

 |      |        | ?Bytes of machine word are reversed 

 |      |        | ?32 bit word machine 

 |      |        | ?Bytes of machine word are reversed 

 ‖_____|_______‖ 

  --------- PE Optional Header ------------------------------------------------------ 

  --------------- 

‖O_DOS |O_PE  ‖(Offset from Dos Header / PE Header 

‖------|------‖ 

 |0118H |0018H | Magic Value              : 010BH 

 |      |       | 

 |011AH |001AH | Major Linker Version     : 2 

 |011BH |001BH | Minor Linker Version     : 25 

 |      |       | Linker Version           : 2.25 

 |011CH |001CH | Size of Code             : 00001200H 

 |0120H |0020H | Size of Initialized Data : 00000600H 

 |0124H |0024H | Size of Uninitialized Data : 00000000H 

 |0128H |0028H | Address of Entry Point   : 00005200H 

 |012CH |002CH | Base of Code (.text ofs.)  : 00001000H 
```
