
# IrfanView32 破解指南

> 原文：[`www.pediy.com/kssd/pediy03/forum697.htm`](https://www.pediy.com/kssd/pediy03/forum697.htm)

**说明**：这是我以前写的心得，贴在这里，请大家指正。

**破解对象**：`IrfanView32`
**破解工具**：
- `W32Dasm` 8.5 (或更高版本)
- `Hacker View` 5.6 (或更高版本)
**破解者**：`chcw`

`IrfanView32` 是一个免费软件，程序作者没有花什么力气来对付 Cracker，故较适合初学 Crack 的人练习用。

## 步骤详解

1. 运行 `IrfanView32`。
2. 选择 `About/Registration`，随便输一个 `name` 和 `code`，如 `name=chcw`, `code=9530109`。
3. 一般来说，你的注册码当然是错误的（除非你的运气特别好，刚好给你碰对了）。程序将显示一个消息框，内容是 `"Incorrect registration!"`。记下该信息后，退出 `IrfanView32`。
4. 运行 `W32Dasm`，将 `I_view32.exe` 反汇编。
5. 反汇编结束后，选择 `W32Dasm` 菜单 `Refs/String Data References`，从列表框中找出字符串 `"Incorrect registration!"`，选中后用鼠标双击，`W32Dasm` 将把亮条定位在反汇编代码中访问到该字符串的代码行上。
6. 关闭字符串列表对话框，我们来看一下亮条处的代码：

```
...
:0041A733 E8C868FEFF      call 00401000
:0041A738 83C408          add esp, 00000008
:0041A73B 85C0            test eax, eax
:0041A73D 751B            jne 0041A75A
...

* Possible StringData Ref from Data Obj -> "Incorrect registration !"
                                 |
:0041A74F 68A0E24700      push 0047E2A0   <-亮条所在处
:0041A754 51              push ecx
:0041A755 E9FEFDFFFF      jmp 0041A558

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041A73D(C)
:0041A75A 8B354C754900    mov esi, dword ptr [0049754C]
```

从亮条所在处往上找，可以发现在 `0041A73B` 处有一条 `test eax,eax` 语句，它后面的 `jne` 语句是 `eax` 为非零值时，跳转到 `0041A75A` 处，正好跳过了对字符串 `"Incorrect registration !"` 的引用。而在 `test` 语句前有一个 `call 00401000` 调用。因此，`00401000` 很可能就是一个对注册码进行检查的函数，当注册码合法时返回非零值，否则返回零，并显示错误信息。将亮条移到 `:0041A733 call 00401000` 语句处，亮条呈亮绿色。按工具栏上的 `"Call"` 按钮进入该子程序。在子程序 `ret` 语句前有下面几条语句：

```
:00401280 3BE8            cmp ebp, eax
...
:00401284 0F94C1          sete cl
...
:00401288 8BC1            mov eax, ecx
...
:0040128E C3              ret
```

可见，如果猜想正确，那么我们只需要将 `00401284` 处的 `sete cl` 语句改为 `setne cl`，就可以使注册成功。

7. 在 `W32Dasm` 中，将亮条移到 `00401284` 的 `jne` 语句处（注：原文此处可能有误，实际为 `sete` 相关位置，依据上下文修正逻辑），此时在 `W32Dasm` 反汇编窗口的状态行中将显示 `@Offset 00000684h in File:I_view32.exe`，记下这个偏移地址。
8. 将 `I_view32.exe` 备份，运行 `Hiew` 打开 `I_view32.exe`，按 `F4` 键 (Mode)，选择 `Decode` 模式，再按 `F5` 键 (Goto)，输入偏移地址 `684`，`Hiew` 将定位到偏移 `684` 处：

```
.00401284 0F94C1          sete cl
```

9. 现在我们将这条语句改为 `setne cl`，按 `F3` 键 (Edit)，输入 `0F95` (`setne` 的机器码) 将原先的 `0F94` (`sete` 的机器码) 覆盖，此时我们可以看到机器码右部的汇编代码也发生了相应的变化。再按 `F9` 键 (Update)，保存所作的修改，退出 `Hiew`。
10. 运行 `I_view32.exe`，随便输入一个注册码（如 `Name="chcw"`, `Code="9530109"`）选择 `"OK"`，程序显示 `"Register Successful"`，再到 `About/About IrfanView` 中，可以看到原先的 `registered to You` 字样改成了 `registerd to chcw`。注册成功了！
    在修改过 `I_view32.exe` 之后，如果你输入了正确的注册码，程序反而会提示 `"Incorrect registeration"`。
11. 有人也许会想，能不能将调用检测函数 `00401000` 之后的语句 `0041A73D jne 0041A75A` 改为 `je` 或 `jmps` 语句，从而达到相同的效果呢？回答是否定的。的确，很多软件都可以在检测函数之外的跳转处 Patch，而且这样做一般来说更为安全，但是 `IrfanView32` 却是例外。`IrfanView32` 相当狡猾，它在程序启动时、注册时和显示 `About IrfanView32` 对话框时都会反复调用 `00401000` 函数检验注册的正确性。所以直接改动检测函数也许更为合理一些。

## 编写 Patcher 工具

**破解对象**：`IrfanView32`
**破解者**：`chcw`

在上一节中，我们已经知道，只要执行程序 `I_view32.exe` 在 `Offset=0x685` 处的字节从 `0x94` 改为 `0x95` 即可破解 `IrfanView32`。在这一节，我们将介绍如何为 `IrfanView32` 写一个 `Patcher`，它在执行时自动完成上述的修改任务。

### Patcher.c

```c
/*--------------------- Patcher.c -----------------------*/
#include <stdio.h>
#include <stdlib.h>

void main()
{
    unsigned char patch=0x95;
    long offset=0x00000685;
    FILE *fp;

    printf("Crack for IrfanView32\n");
    printf("Written by Mr. Chcw\n");

    if ((fp=fopen("I_view32.exe", "r+"))==NULL) {
        printf("Error: Cannot find file I_view32.exe\n");
        exit(1);
    }

    fseek(fp,offset,SEEK_SET);
    fputc(patch,fp);
    printf("Patch successful\n");
}
/*--------------------- End of Patcher.c ------------------*/
```

`Patcher.c` 程序先将可执行文件 `I_view32.exe` 读入，然后通过偏移地址定位到要 Patch 的地方，再用机器码 `0x95` 改写该处的字节，就完成了 Patch 过程。

当然，在实际应用中，待 Patch 的程序因版本的不同可能会产生 Patch 地址的偏差或 Patch 代码的偏差，我们需要在 Patch 之前检查待 Patch 的程序的版本（如根据程序的大小）和 Patch 处原机器码是否正确，以保证不会 Patch 错误。

### Patcher1.c（改进版）

下面是一个改进的 `Patcher1.c` 程序：

```c
/*----------------------- Patcher1.c ------------------------*/
#include <stdio.h>
#include <io.h>
#include <stdlib.h>

void main()
{
    const long filesize=614912;  //614,912  bytes
    const unsigned char oldcode=0x94;
    unsigned char patchcode=0x95;
    long offset=0x00000685;
    FILE *fp;

    printf("Crack for IrfanView32\n");
    printf("Written by Mr. Chcw\n");

    if ((fp=fopen("I_view32.exe", "r+"))==NULL) {
        printf("Error: Cannot find file I_view32.exe\n");
        exit(1);
    }
    else if (filelength(fileno(fp))!=filesize) {
        printf("Error: Incorrect IrfanView32 version, cannot patch it\n");
        exit(2);
    }

    fseek(fp,offset,SEEK_SET);
    if (fgetc(fp)!=oldcode) {
        printf("Error: The file I_view32.exe is corrupted\n");
        exit(3);
    }

    fseek(fp,offset,SEEK_SET);
    fputc(patchcode,fp);
    fclose(fp);
    printf("Patch successful\n");
}
/*-------------------- End of Patcher1.c --------------------*/
```

## 关于 Patch 方法的局限性与其他思路

**破解对象**：`IrfanView32`
**破解者**：`chcw`

在上两节中，我们介绍了通过修补 `IrfanView32` 程序来破解其注册检验的方法。修补程序（或称 Patch）是 Crack 中常用的一种方法，它的缺点也是显而易见的，主要有以下几点：

1.  不少程序在运行时会检查自身的完整性（如 `Win Command32`），如果发现自身被修改过，就会立即报错退出，Patch 方法对这类程序就无计可施了（除非把程序检查完整性的部分也给 Patch 掉）。
2.  有些程序在通过编译优化后，可能不只在一处调用 Patch 过的代码，因此 Patch 此类程序一般要相当小心，对 Patch 点的选取是关键。如果 Patch 不好的化，在程序运行时，可能会产生许多莫名其妙的错误。
3.  有些查毒程序（如 `MSAV`），在扫描硬盘上的程序文件时，同时会生成这些文件的校验码，以备日后检查文件完整性用。因此，如果我们 Patch 了一个受查毒程序校验和保护的程序，查毒程序可能会认为该程序被病毒感染。
4.  Patch 程序常以执行文件方式提供，容易传播病毒。
5.  Patch 程序对错误的执行文件版本进行 Patch 后，会导致无法预料的结果。

另一种常用的 Crack 方法是直接获取程序的合法注册码或是序列号，然后通过正常的注册过程完成注册，这种方法可以避免对执行程序本身进行修改，因而不会有 Patch 方法的那些问题；它的缺点是工作量较大，破解者往往需要理解整个注册检验函数的工作机理。

还是以 `IrfanView32` 为例，在上一节中，我们已经知道 `:0041A733 call 00401000` 语句很可能就是程序检验注册码的子程序，在这一节中，我们将分析子程序 `00401000`，并从中获取 `IrfanView32` 的注册码。

1.  用 `WinICE` 载入 `I_view32.exe`。
2.  按 `Ctrl D`，回到 `IrfanView32` 窗口。选择 `About/Registration`，打开注册窗口。
3.  按 `Ctrl D`，回到 `WinICE` 窗口，键入命令 `BPX MessageBoxA`，对消息框设置断点。再键入 `X` 命令，回到注册窗口。
4.  在注册窗口中输入注册码 `Name=chcw,Code=9530109`，按 `OK` 命令键，触发我们刚才设置的断点，进入 `WinICE` 窗口。
5.  用 `BC *` 命令清除原先的断点。然后键入命令 `BPX CS:0041A733`，在调用 `00401000` 子程序的语句处设置新的断点。键入 `X` 命令，回到 `IrfanView32` 窗口。
6.  再次输入注册码，按 `OK` 后，触发断点。执行亮带停留在 `0137:0041A733 call 00401000` 语句处，检查该程序的调用，发现参数 1 为串 `"chcw"` 的地址，参数 2 为串 `"9530109"` 的地址，这就进一步证实了 `00401000` 子程序是注册检验函数的猜想。
7.  按 `F8`，进入该子程序调用。其中有不少子程序调用语句和循环，先不去管它们，不断按 `F10`，跳过 (Step Out) 这些语句，最后到达 `00401000` 子程序的返回语句附近：

```
:00401280 cmp ebp, eax
...
:00401284 sete cl
...
:00401288 mov eax, ecx
...
:0040128E ret
```

这几条语句是处理子程序 `00401000` 的返回值用的（大部分函数的返回值都放在 `eax` 中），其效果相当于 `c` 语言中的 `return (ebp==eax);`。在上两节中，我们知道，当子程序 `00401000` 返回值 `eax` 不为零时，注册成功。因此，在对用户输入的姓名和注册码进行一定的运算之后，只要满足 `ebp==eax`，注册就算成功了。记下在语句 `cmp ebp, eax` 处的 `ebp` 和 `eax` 的值，`ebp=00916AFD`, `eax=10E1B31A`。

8.  键入 `X` 命令后，回到 `IrfanView32` 窗口，再次输入注册码，进入子程序 `00401000`，设置断点：
    ```
    EPR cs:00401000 cs:00401280 R if ((ebp==00916AFD) || (eax==10E1B31A))
    ```
    该断点是用来查找从 `cs:00401000` 到 `00401280` 的语句中，将 `ebp` 设置为 `00916AFD` 或将 `eax` 设置为 `10E1B31A` 的第一条语句。键入 `X` 命令后，中断被触发，相继找到 `ebp=00916AFD` 是 `:0040100E` 处 `call 0045F770` 子程序调用的返回值。`eax=10E1B31A` 是 `:00401276` 处 `call 0045F770` 子程序调用的返回值。在这两处分别设置一次性断点。发现对子程序 `0045F770` 的两次调用的参数分别为字符串 `"9530109"` 和 `"283226906"`。既然两次调用的函数都是 `0045F770`，那么只要我们将原先的注册码 `"9530109"` 改为 `"283226906"`。
