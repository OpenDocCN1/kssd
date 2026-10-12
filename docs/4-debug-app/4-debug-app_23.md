
# 3 句代码，在星际争霸屏幕上写字(详细教程，有图，附源码)

> 原文：[`www.pediy.com/kssd/pediy10/80016.html`](https://www.pediy.com/kssd/pediy10/80016.html)

我们来看看在游戏里怎么写字！

游戏版本：星际争霸 1.16.0

工具: `OllyDBG`, `VS2005`

如下图：

![](img/f5c82aec4b30e27eb707c48a66b7575e.jpg)

**找 `CALL` 思路：游戏里写完字后，它应该会在内存里留下字符的痕迹，我们就从这个思路开始找写屏的函数。**

## 第一步：先让游戏自已写几个字！

运行游戏，选择局网 `IPX`, 和电脑对手。开始游戏后选择设置网速（玩星际的不用说怎么设置了吧？），完了它会出现以下字符：

![](img/7595d30552a69a5e7d02393b66a99eee.jpg)

## 第二步：在内存里找关键字，按下图所示，找到为止。

1. 按图所示，在 `OD` 里先从 `D 00400000` 开始，回车。
2. 按 `Ctrl+B` 输入关键字，再确认，如果没找到，再从 1 步骤开始（找不到再从 `D 00500000`, `D 00600000`）。

![](img/76b14eda1ea2734c8467548a8edc53eb.jpg)

## 第三步：找到后用你的大脑分析，怎么下断。

可以看得出，这是个动态分配的字符串，但是也有规律，每间隔 `N` 行就是一串。

![](img/24f561429a9829b8d39b5cd249ea0895.jpg)

## 第四步：找到规律后，我们试试在最后一次显示的字符串下面，选择一大片空白区下个"内存写入断点"，看看能不能找到是哪个程序在写字符？

下完断点后，在 `OD` 里按 `F9` 运行，再切换回游戏设置网速，让游戏自已再写一遍字。

![](img/e7e52eae1607d482b4027374ab5967ba.jpg)

## 第五步：运气真好，居然断到了。

1. 如果断点正确(看下图 1 的注释)，就按下图中所标的 1, 2, 3 步骤操作，之后，在 `OD` 按 `F9` 运行程序。

说明：按这三个步骤操作很重要，它可以帮你带到写屏函数那里去。因为找到写字符的内存了，我们就可以找到显示字符的函数了，因为游戏要读这段字符内存来进行显示。

![](img/d73d819fc030acbc162f0a253b2e7df7.jpg)

## 第六步：离胜利不远了。

它把你带到写屏函数附近了。如下图所示，为什么要在 2 那里下个断再走一步？因为这还不是写屏的主函数，因为它 `RETN 0C`，就算你用远程进程 `Call` 这个地址，程序也会出错，不会正常返回，所以我们还要往回找，看这里哪里 `CALL` 进来的，按下图的说法操作。

![](img/d5ed13b23dca2a7a45763e14306c28ce.jpg)

## 第七步：搞定！

![](img/872a9a9a7d601b20f3ced094bc9f2e54.jpg)

## 第八步：通过远程调用游戏的 `CALL`，来实现游戏写屏：

这一步骤就不详说了，`VS2005`, 我用的是全局钩子。你们爱怎么就怎么的，反正关键代码是：

```c
if(X 键按下)

{

DWORD addr = 0x0048CFB0;//这个地址就是我们刚才找到的写屏主函数的起点(星际 1.16 版本)。

char *buf = "Hello Word!";

__asm

{

  xor eax,eax;

  mov edi,buf;

  call addr;

}

}
```

星际每个版本都可能这个地址有变化，但是步骤都差不多。下面是 1.08 版本的:

```c
    DWORD addr = 0x0046DE10;//（星际 1.08 版本的）

    char *buf = "Hello Word!";

    __asm

    {

      xor edx,edx;

      mov ecx,buf;

      call addr;

    }
```

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `sc1.16.rar` |

## 顺便再给出一段星际联网时可地图全开的代码(1.08 版本)

//16 进制字符串转数字并写入指定地址

```c
void write_ma(DWORD addr,char *ma)

{

  __asm

  {

    PUSH EAX;

    PUSH EBX;

    PUSH ECX;

    PUSH EDX;

    PUSH ESI;

    MOV DL,16;

    MOV EBX,DWORD PTR DS:[ma];

    MOV ESI,addr;

    CALL Write;

    POP ESI;

    POP EDX;

    POP ECX;

    POP EBX;

    POP EAX;

    JMP OVER;

  Write:

    XOR EAX,EAX;

    XOR ECX,ECX;

    CALL GetAnsi;

    MUL DL;

    MOV CL,AL;

    INC EBX;

    CALL GetAnsi;

    OR AL,CL;

    MOV BYTE PTR DS:[ESI],AL;

    INC EBX;

    INC ESI;

    MOV AL,BYTE PTR DS:[EBX];

    CMP AL,0;

    JNZ Write;

    RET;

  GetAnsi:

    MOV AL,BYTE PTR DS:[EBX];

    CMP AL,57;

    JA ZiMu;

    SUB AL,48;

  L1:

    RET;

  ZiMu:

    SUB AL,55;

    JMP L1;

  OVER:

  }

}
```

//按 `F12` 地图全开

```c
==============================================

  if(VK_F12 == wParam && 1 == ((lParam>>30) & 1))

  {

//游戏屏幕上显示“Open Map”

    DWORD addr = 0x0046DE10;

    char *buf = "Open Map";

    __asm

    {

      xor edx,edx;

      mov ecx,buf;

      call addr;

    }

//开地图的代码

    write_ma(0x00404BA8,"7500");

    write_ma(0x00410C81,"BDFFFFFFFF");

    write_ma(0x00414046,"7400");

    write_ma(0x0046EA70,"909090909090");

    write_ma(0x0046EA7D,"EB29");

    write_ma(0x0046EA98,"7500");

    write_ma(0x0046F7EE,"310578A16500850500A86500750AFF0D90A6650090909090");

    write_ma(0x0046F815,"7400");

    write_ma(0x0046FA6B,"310578A16500850500A86500750AFF0D90A6650090909090");

    write_ma(0x0046FA92,"7400");

    write_ma(0x00470F2B,"7400");

    write_ma(0x00470F43,"EB04");

    write_ma(0x00470F4B,"EB09");

    write_ma(0x00471067,"7400");

    write_ma(0x0047107F,"EB04");

    write_ma(0x00471087,"EB09");

    write_ma(0x004C9541,"0F8400000000");

  }

}
```
