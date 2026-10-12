
# 明伦五笔高手 1.2A

> 原文：[`www.pediy.com/kssd/pediy03/forum696.htm`](https://www.pediy.com/kssd/pediy03/forum696.htm)

**破解程序：**

明伦五笔高手 1.2A

**破解工具：**

SoftICE 3.0 For Windows 95 (Util16 Dldr)
W32Dasm 8.5
UltraEdit 5.0

**破解者：**

chcw

明伦五笔高手是 DOS 下的一个应用程序，该程序在初启时和选择菜单命令时，都会去访问软驱，并显示‘请插入 Key 盘’的对话框，但在软驱中无盘或无钥匙盘的情况下，并不会出错。由于频繁的访问软驱对软驱损害较大，所以我们要把该程序访问软驱的部分破解掉。

1.  用 `Dldr` 载入 `WT.EXE` 程序。
2.  在 DOS 下常用的磁盘 IO 中断是 `INT 13H`，我们就对该中断设置断点：

    ```
    BPINT 13
    ```

3.  键入 `X`，运行程序。立即就会被 `SoftICE` 拦截到：

    ```
    1DB3:13B3    XOR AX, AX
    1DB3:13B5    INT 13       ; Reset Controler      <----亮带所在点
    ```

4.  键入 `BC *`，清除原有的断点。然后从亮带处向下，找到子程序的返回点处：

    ```
    1DB3:1458    RETF
    ```

    将光标移到 `1458` 处，键入 `'HERE'`。程序将执行至此处。

5.  按 `F10` 返回到调用的语句处：

    ```
    1DB3:01C3    CALL 13A8
    1DB3:01C6    POP CX
    1DB3:01C7    POP CX
    ```

    键入 `r ax`，查得返回值 `ax=0`。仔细分析堆栈，发现执行调用 `CALL 13A8` 后，`SP` 指针增加了 2。因此我们可以用语句:

    ```
    XOR AX, AX   ; 将 ax 置为 0
    POP CX       ; 将 SP 指针增加 2
    ```

    代替 `CALL 13A8` 语句来获得相同的效果。

6.  用 `W32Dasm` 反汇编 `WT.EXE` 程序，并将反汇编代码存为文件 `WT.ALF`，在文件 `WT.ALF` 中查找字符串 `'int 13'`，找到以下地址：

    ```
    :0002.2B95 CD13                int 13
    ```

7.  从该地址向上找，可以找到该语句所在子程序的入口点，并得到调用该程序的语句地址：

    ```
    * Referenced by a CALL at Addresses:
    |:0002.19A3, :0002.2C42
    |
    :0002.2B88 55                 push bp      <---子程序入口点
    ```

8.  下面我们将把 `WT.EXE` 程序中所有调用子函数 `0002.2B88` 的语句改为语句 `xor ax,ax`/`pop cx`。找到地址 `0002.19A3` 处:

    ```
    :0002.19A3 E8E211              call 2B88
    ```

    该函数调用语句的机器码为 `'E8E211'`，用 `UltraEdit` 打开 `WT.EXE` 程序，查找字符串 `'E8E211'` 并将其替换为 `'33C059'`（`'33C0'`是 `xor ax, ax` 的机器码，`'59'`是 `pop cx` 的机器码）。同理可以将 `0002.2C42` 地址处的机器码进行修改。

9.  执行 `WT.EXE` 程序，程序不会再对软驱进行访问了，但仍会显示‘请插入 Key 盘’的对话框。

**注:** 程序在调用 `2B88` 之后，若返回值 `ax==0`，会显示‘请插入 Key 盘’的对话框。因此如果在第 8 步中将替换的字符串改为 `'B00159'`（`'B001'`是 `mov al, 1` 的机器码，`'59'`是 `pop cx` 的机器码），那么连对话框也不会显示了。

**附:** `WT.EXE` 的 Patch 程序 `Patcher.c`

```c
--------------------------------Patcher.c---------------------------------
#include <stdio.h>
#include <io.h>
#include <stdlib.h>
#include <string.h>

void main()
{
    const long filesize=101334;    //101,334 bytes
    const unsigned char oldcode[][4]={{0xE8, 0xE2, 0x11, 0x00}, {0xE8, 0x43, 0xFF, 0x00}};
    unsigned char patchcode[4]={0xB0, 0x01, 0x59, 0x00};
    unsigned char buffer[4];
    long offset[]={0x000135A3, 0x00014842};
    FILE *fp;
    int i;
    printf("Crack for WT\n");
    printf("Written by Mr. Chcw\n");
    if ((fp=fopen("wt.exe", "r+"))==NULL) {
        printf("Error: Cannot open file wt.exe\n");
        exit(1);
    }
    else if (filelength(fileno(fp))!=filesize) {
        printf("Error: Incorrect WT version, cannot patch it\n");
        exit(2);
    }

    for (i=0; i<2; i++) {
        fseek(fp,offset[i],SEEK_SET);
        fread(buffer, 3, 1, fp);
        buffer[3]='\0';
        if (strcmp(buffer, oldcode[i]) != 0) {
            printf("Error: The file wt.exe is corrupted\n");
            exit(3);
        }
    }

    for (i=0; i<2; i++) {
        fseek(fp,offset[i],SEEK_SET);
        fwrite(patchcode, 3, 1, fp);
    }
    fclose(fp);
    printf("Patch successful\n");
}
----------------------- End of File Patcher.c --------------------------
```
