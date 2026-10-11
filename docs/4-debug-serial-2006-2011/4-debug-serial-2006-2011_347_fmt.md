
# 是男人就上 120 层 破解

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-396.htm`](https://www.pediy.com/kssd/pediy09/pediy09-396.htm)

> 【文章标题】: 是男人就上 120 层 破解  
> 【文章作者】: Nukou.G  
> 【软件名称】: 是男人就上 120 层  
> 【软件大小】: 480K  
> 【下载地址】: 黑客 X 档案 4 月份附书光盘中  
> 【加壳方式】: 无  
> 【保护方式】: 序列号  
> 【编写语言】: `VC4.0`  
> 【使用工具】: `OllyDBG`  
> 【操作平台】: `WinXP`  
> 【软件介绍】: 这个游戏没玩过也该听过吧  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> ------------------------------------------------------------------------

> 【详细过程】

当输入的注册码不正确时弹出一个对话框提示错误。在 `MessageBoxA` 上下断点，输入注册码会断在这里：

```
00402FE1   .  8D85 FCFDFFFF lea     eax, dword ptr [ebp-204]
00402FE7   .  50         push    eax
00402FE8   .  E8 C0010000  call    004031AD                     ;  验证注册码的函数
00402FED   .  83C4 04     add     esp, 4
00402FF0   .  85C0       test    eax, eax
00402FF2   .  0F85 37000000 jnz     0040302F
00402FF8   .  68 00010000  push    100                            ; /Count = 100 (256.)
00402FFD   .  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]     ; |
00403003   .  50         push    eax                              ; |Buffer
00403004   .  6A 04     push    4                                ; |RsrcID = STRING "Registration number is incorrect."
00403006   .  A1 98D24000  mov     eax, dword ptr [40D298]       ; |
0040300B   .  50         push    eax                              ; |hInst => NULL
0040300C   .  FF15 BCF34000 call    dword ptr [<&USER32.LoadStringA>]    ; \LoadStringA
00403012   .  6A 10     push    10                               ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
00403014   .  68 6CC24000  push    0040C26C                      ; |Title = "NS-TOWER"
00403019   .  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]     ; |
0040301F   .  50         push    eax                              ; |Text
00403020   .  8B45 08   mov     eax, dword ptr [ebp+8]           ; |
00403023   .  50         push    eax                              ; |hOwner
00403024   .  FF15 B8F34000 call    dword ptr [<&USER32.MessageBoxA>]   ; \MessageBoxA
```

可见 `004031AD` 就是验证注册码的函数了

来到 `004031AD` 处:

```
004031AD   $  55         push    ebp
004031AE   .  8BEC       mov     ebp, esp
004031B0   .  83EC 04   sub     esp, 4
004031B3   .  53         push    ebx
004031B4   .  56         push    esi
004031B5   .  57         push    edi
004031B6   .  8B45 08   mov     eax, dword ptr [ebp+8]
004031B9   .  33C9       xor     ecx, ecx
004031BB   .  8A48 07   mov     cl, byte ptr [eax+7]           ;  注册码的长度要小于 8
004031BE   .  85C9       test    ecx, ecx
004031C0   .  0F84 07000000 je      004031CD
.......
00403315   .  C9         leave
00403316   .  C3         retn
```

中间的汇编代码略去，该段代码等价于：

```c
{
  ...
  int i = 0;
  while(i < 0x7)
  {
    if (fun(serial[i]) > 24)
    {
      return(0);
    }
    i++;
  }

  if ((( fun(serial[5])*2 +  fun(serial[2]) + 0x1C) % 0x24) == fun(serial[0]))
  {
    if ((( fun(serial[4])*2 +  fun(serial[1])+ 0x1C) % 0x24) == fun(serial[6]))
    {
      if ((( fun(serial[6])*2 +  fun(serial[0]) + 0x1C) % 0x24) == fun(serial[3]))
      {
        return(1);
      }
    }
  }
  
  return(0);
}
```

其中 `fun()` 函数是 `00403317` 处的函数，其代码等价于:

```c
char fun(char key)
{
  if (key >= 0x61)
  {
    key  -= 0x20;
  }
  else if (key >= 0x41)
  {
    key -= 0x7;
  }

  key -= 0x30;

  return(key);
}
```

一切都很明显了。

==========

注册机编写

当然是穷举，不过在这之前仔细观察一下 3 个条件：

```c
(( fun(serial[5])*2 +  fun(serial[2]) + 0x1C) % 0x24) == fun(serial[0])
(( fun(serial[4])*2 +  fun(serial[1])+ 0x1C) % 0x24) == fun(serial[6])
(( fun(serial[6])*2 +  fun(serial[0]) + 0x1C) % 0x24) == fun(serial[3])
```

看出来了吧？`serial[6]`和 `serial[0]`是两个关键位，先确定这两个位，剩下的就简单了:

```c
#include <stdio.h>
#include <stdlib.h>

char defun(char key)
{
  key += 0x30;
  
  if (key >= 0x41-0x7)
  {
    key += 0x7;
  }
  else if (key >= 0x61-0x20)
  {
    key += 0x20;
  }
  
  return(key);
}

void main()
{
  char serial[7];
  int i;

  serial[5] = 0;
  serial[4] = 0;
  for (serial[6] = 0; serial[6] < 0x24; serial[6]++)
  {
    for (serial[0] = 0; serial[0] < 0x24; serial[0]++)
    {
      serial[3] = ((serial[6]*2 + serial[0])+ 0x1C) % 0x24;
      for (serial[2] = 0; serial[2] < 0x24; serial[2]++)
      {
        if (serial[6] == (serial[2]+ 0x1C) % 0x24)
          for (serial[1] = 0; serial[1] < 0x24; serial[1]++)
          {
            if (serial[4] == (serial[1]+ 0x1C) % 0x24)
            {
              for (i = 0; i < 7; i++)
              {
                serial[i] = defun(serial[i]);
                printf("%c", serial[i]);
              }
              system("pause");
              printf("\n");
            }
          }
      }
    }
  }
}
```

------------------------------------------------------------------------

> 【经验总结】

以前没耐心看汇编代码，很少能独立解决一个软件。经过一段时间的锻炼，阅读汇编不是那么痛苦的事情了，能独立解决的问题也多了起来了。

------------------------------------------------------------------------

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

> 【文章标题】: 是男人就上 120 层 Patch  
> 【文章作者】: Nukou.G  
> 【软件名称】: 是男人就上 120 层  
> 【软件大小】: 480K  
> 【下载地址】: 黑客 X 档案 4 月份附书光盘中  
> 【加壳方式】: 无  
> 【保护方式】: 序列号  
> 【编写语言】: `VC4.0`  
> 【使用工具】: `OllyDBG`  
> 【操作平台】: `WinXP`  
> 【软件介绍】: 这个游戏没玩过也该听过吧  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> ------------------------------------------------------------------------

> 【详细过程】

每次这个程序启动都会显示一个对话框：“`BGM1.MID` was not found. Please put it in the same folder as application file.”很烦人，没有这个文件程序一样运行，所以把这个对话框跳过去。顺便再给游戏加一个功能:`1`-`=`键分别对应 12 个弹跳力度，不用按住空格键蓄力了（当然这样难度降低了）。

先是去掉那恼人的对话框：在 `MessageBoxA` 函数上下断点，运行会断在

```
0040495F  .  /0F85 4E000000 jnz     004049B3
00404965  .  |68 00010000   push    100                    ; /Count = 100 (256.)
0040496A  .  |8D85 00FFFFFF lea     eax, dword ptr [ebp-100]       ; |
00404970  .  |50         push    eax                    ; |Buffer
00404971  .  |6A 05     push    5                      ; |RsrcID = STRING "NS-TOWER Error"
00404973  .  |A1 98D24000  mov     eax, dword ptr [40D298]     ; |
00404978  .  |50         push    eax                    ; |hInst => 00400000
00404979  .  |FF15 BCF34000 call    dword ptr [<&USER32.LoadStringA>] ; \LoadStringA
0040497F  .  |68 00010000   push    100                    ; /Count = 100 (256.)
00404984  .  |8D85 00FEFFFF lea     eax, dword ptr [ebp-200]       ; |
0040498A  .  |50         push    eax                    ; |Buffer
0040498B  .  |6A 06     push    6                      ; |RsrcID = STRING "BGM1.MID was not found.  Please put it in the same folder as application file."
```



```asm
0040498D  .  |A1 98D24000   mov     eax, dword ptr [40D298]          ; |
00404992  .  |50           push    eax                                ; |hInst => 00400000
00404993  .  |FF15 BCF34000 call    dword ptr [<&USER32.LoadStringA>>; \LoadStringA
00404999  .  |6A 30        push    30                                 ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
0040499B  .  |8D85 00FFFFFF lea     eax, dword ptr [ebp-100]         ; |
004049A1  .  |50           push    eax                                ; |Title
004049A2  .  |8D85 00FEFFFF lea     eax, dword ptr [ebp-200]         ; |
004049A8  .  |50           push    eax                                ; |Text
004049A9  .  |8B45 08      mov     eax, dword ptr [ebp+8]            ; |
004049AC  .  |50           push    eax                                ; |hOwner
004049AD  .  |FF15 B8F34000 call    dword ptr [<&USER32.MessageBoxA>>; \MessageBoxA
```

将地址 `0040495F` 处的指令 `jnz` 修改为 `jmp` 即可。

接下来是给程序添加快捷键：首先要找到游戏的过程函数。起初我想在 `NewGame` 按钮上寻找，结果什么也没找到。然后我注意到 `WINMM.sndPlaySoundA` 函数，蓄力时是否是用该函数发出声音的？在该函数上下断点后，断点命中在：

```asm
00406E47  .   FF15 58F44000 call    dword ptr [<&WINMM.sndPlaySoundA>;  WINMM.sndPlaySoundA
00406E4D  .   817D 0C 9D000>cmp     dword ptr [ebp+C], 9D
00406E54  .   0F87 1B000000 ja      00406E75
```

我们进入了游戏的核心逻辑！将该函数重命名为 `MakeSnd`（在函数起始地址 `00406E3D` 上右键 > 编辑标签 > 输入名称），并将包含该函数的大函数（起始地址 `004067DA`）命名为 `Fun`。

随后跟随该函数，发现它返回到一个起始于 `004067DA`、结束于 `00406DA2` 的函数。这个函数做了什么？它调用的其他函数又做了什么？肯定没人愿意阅读这么长的汇编代码，且不确定是否就是目标函数。

这里有一个小技巧：将 `OllyDBG` 窗口与游戏窗口分离，避免 `OllyDBG` 遮挡游戏窗口。然后在每个 `CALL` 指令前下断点（`MakeSnd` 函数除外，因为已知其功能），多次按 `F9`，发现游戏总是在以下 4 个位置中断：

```asm
0040681B  .   E8 44FCFFFF   call    00406464
00406837  .   E8 6FF7FFFF   call    00405FAB
00406D24  .   E8 EAE3FFFF   call    00405113
00406D30  .   E8 DAEEFFFF   call    00405C0F
```

每次执行以下指令后，小人会向前走一步，因此将该函数命名为 `Walk`，并移除所有 `call Walk` 处的断点。

```asm
00406D24  .   E8 EAE3FFFF   call    00405113
```

但程序仍在剩余 3 个位置中断，这是因为未对游戏窗口进行输入。为了收集更多信息，将这 3 个函数分别命名为 `Fun1`、`Fun2`、`Fun3`，并移除这些断点。按 `F9` 后，游戏不再中断。

切换至游戏窗口，按下空格键，此时程序中断在：

```asm
004068F1  .   E8 9DF1FFFF   call    00405A93
```

按 `F8` 单步执行，若游戏窗口位置合适，会发现 `POWER` 槽增加一格！将该函数命名为 `ChgPow`。存储力度的内存地址很可能在此函数中。查看函数参数（此处为 `0x144108`）：

`00144108 C2 05 01 00`

看起来不像目标数据。必须进一步跟进，所幸该函数不长。立即发现 `BitBlt` 具有可疑的参数：

```asm
00405ABA  .   8B45 08      mov     eax, dword ptr [ebp+8]          ; |
00405ABD  .   8B80 A8110000 mov     eax, dword ptr [eax+11A8]       ; |<在此处观察 EAX 值
00405AC3  .   C1E0 04      shl     eax, 4                           ; |
00405AC6  .   05 A0000000   add     eax, 0A0                         ; |
00405ACB  .   50           push    eax                              ; |YSrc
```

跟踪时注意寄存器：执行 `mov eax, dword ptr [eax+11A8]` 后观察 `EAX` 值及力度槽，即可找到存储力度的内存位置。该位置位于 `ChgPow` 函数唯一参数指向的地址加 `11A8` 处。回到之前的调用者，查看传给 `ChgPow` 的参数：

```asm
004068ED  .   8B45 08      mov     eax, dword ptr [ebp+8]
004068F0  .   50           push    eax
004068F1  .   E8 9DF1FFFF   call    <ChgPow>
```

发现是第一个参数。

继续分析，移除 `call ChgPow` 处的断点，回到游戏按空格键，程序再次中断：

```asm
00406AE7  .   E8 BFF4FFFF   call    <Fun2>
```

再次遇到 `Fun2`，但除了寄存器变化外，画面和力度均未改变。回想之前：小人能跑能跳，运行函数已找到，跳跃函数在哪里？调试过程中小人已跳跃多次，除 `Fun2` 外的中断函数均已识别，推测 `Fun2` 即为跳跃函数。验证方法：在游戏中按住空格蓄力，蓄力过程无中断，松开空格时中断于 `Fun2` 位置，即小人起跳时刻。

多玩一段时间后，可识别出：
- 地址 `00406A47` 处的 `call 004062BE` 是移动屏幕的 `MovScr` 函数。
- 地址 `00406CB2` 处的 `call 0040490E` 是玩家死亡的 `Die` 函数。

由于它们与目标无关，此处不再赘述。

现在需查找检查玩家按键的位置。根据前文分析，调用 `ChgPow` 必然基于蓄力键按下。因此，未按键时应存在跳转跳过 `ChgPow`。从 `004068F1 call <ChgPow>` 向上检查跳转指令。幸运的是，第一个遇到的跳转指令非常可疑：

```asm
004068B3  |> \8B45 08      mov     eax, dword ptr [ebp+8]
004068B6  .   83B8 08130000>cmp     dword ptr [eax+1308], 1
004068BD  .   0F85 66000000 jnz     00406929
```

可见 `[ebp+8]+1308` 即为“按键状态”标志。找到该内存地址并设置内存写断点。按 `F9` 让游戏正常运行，切换至游戏窗口按空格键，断点命中在：

```asm
00404352  .   C780 08130000>mov     dword ptr [eax+1308], 1
```

至此找到判断按键的位置。将该语句所在的函数命名为 `CheKey`。附近可能有修改力度的语句。力度存储在 `ChgPow` 参数指向地址加 `11A8` 处，有趣的是 `CheKey` 的第一个参数也指向相同位置。

为修改 `CheKey`，需理解其流程。该函数等价于：

```c
void CheKey(PDWORD pPower, DWORD key, DWORD flag, DWORD, DWORD)
{
    if (key == ' ')
    {
        if (flag == 0)
        {
            KeyIsPressed = 0;
        }
        else
        {
            KeyIsPressed = 1;
            
            if (*(pPower+0x1190) == 0x3)
            {
                Pause(pPower);
            }
        }
    }
    else if (key == 0x1B)
    {//0x1B 是哪个键...
        if (flag != 0)
        {
            if (*(pPower+0x1190) == 0x3)
            {
                Pause(pPower);
            }
        }
    }
}
```

意外发现该函数未修改力度。需在其他位置查找力度修改逻辑（必须了解原始逻辑才能安全修改）。在力度内存位置设置内存写断点，按 `F9` 进入游戏蓄力，断点命中在：

```asm
004068B6  .   83B8 08130000>cmp     dword ptr [eax+1308], 1
004068BD  .   0F85 66000000 jnz     00406929
004068C3  .   8B45 08      mov     eax, dword ptr [ebp+8]
004068C6  .   C780 AC110000>mov     dword ptr [eax+11AC], 1
004068D0  .   8B45 08      mov     eax, dword ptr [ebp+8]
004068D3  .   B9 0C000000   mov     ecx, 0C
004068D8  .   8B80 A8110000 mov     eax, dword ptr [eax+11A8]
004068DE  .   99           cdq
004068DF  .   F7F9          idiv    ecx
004068E1  .   8D42 01      lea     eax, dword ptr [edx+1]
004068E4  .   8B4D 08      mov     ecx, dword ptr [ebp+8]
004068E7  .   8981 A8110000 mov     dword ptr [ecx+11A8], eax <断在此处
```

推测程序通过 `KeyIsPressed` 标志使力度加一。

已知所有必要信息，只需编写代码。策略为：在每个新功能键的事件处理中，将力度修改为原计划值减 1（例如 1 对应 0，2 对应 1... - 对应 10，= 对应 11），并将 `KeyIsPressed` 置 1。这样执行 `004068E7` 处的加法即可达成原目标。

将 `CheKey` 修改为：

```c
void CheKey(PDWORD pPower, DWORD key, DWORD flag, DWORD, DWORD)
{
    if (key == ' ')
    {
        if (flag == 0)
        {
            KeyIsPressed = 0;
        }
        else
        {
            KeyIsPressed = 1;
            
            if (*(pPower+0x1190) == 0x3)
            {
                00407157(pPower);
            }
        }
    }
    else if (key == '1')
    {
        POWER = 1;
        if (flag == 0)
        {
            KeyIsPressed = 0;
        }
        else
        {
            KeyIsPressed = 1;
            
            if (*(pPower+0x1190) == 0x3)
            {
                Pause(pPower);
            }
        }
    }
    else if (key == '2')
    ...
    else if (key == 0x1B)
    {//0x1B 是哪个键...
        if (flag != 0)
        {
            if (*(pPower+0x1190) == 0x3)
            {
                Pause(pPower);
            }
        }
    }
}
```

回游戏测试，发现未按预期在按键时跳跃，仍是停止蓄力才跳。需查找跳跃触发条件。从 `Jump` 函数入手，有两处调用：

```asm
00406837  .   E8 6FF7FFFF   call    <Jump>
00406AE7  .   E8 BFF4FFFF   call    <Jump>
```

其中 `00406837` 始终被触发，故不考虑。需研究哪个跳转决定了 `00406AE7` 是否执行。从 `Fun` 函数入口开始跟踪，发现一处关键指令：

```asm
00406823  .   8B45 08      mov     eax, dword ptr [ebp+8]
00406826  .   83B8 B0110000>cmp     dword ptr [eax+11B0], 0
0040682D  .   0F85 E4010000 jnz     00406A17
```

该指令距 `Fun` 入口不远。此处引入新变量 `pPower+0x11B0`，可视为 `WanaJump` 标志。

添加修改 `WanaJump` 的语句并测试功能键。结果仍令人失望：能听到跳跃音效，但小人无动作。观察正常跳跃时的程序状态：

```asm
00406AC9  |> \33C0          xor     eax, eax
```



# 汇编代码分析

```
00406ACB   |.  8B4D 08       mov     ecx, dword ptr [ebp+8]  
00406ACE   |.  8B89 A0110000 mov     ecx, dword ptr [ecx+11A0]<看看 ECX 的值  
00406AD4   |.  03C9          add     ecx, ecx  
00406AD6   |.  2BC1          sub     eax, ecx  
00406AD8   |.  F7D8          neg     eax  
00406ADA   |.  8B4D 08       mov     ecx, dword ptr [ebp+8]  
00406ADD   |.  2981 98110000 sub     dword ptr [ecx+1198], eax  
00406AE3   |>  8B45 08       mov     eax, dword ptr [ebp+8]  
00406AE6   |.  50            push    eax  
00406AE7   |.  E8 BFF4FFFF   call    <Jump>  
```

在 `00406AC9` 处下断点（在这里下断点的原因是我怀疑它之后的语句对 `Jump` 的参数有影响），回游戏按空格正常蓄力，注意到 `pPower+11A0` 处的值与力度相同。然后让游戏运行，按一下我们的功能键：`pPower+11A0` 的值是 `0`?

再加上修改 `pPower+11A0` 处数据的语句。回到游戏，测试一下，成了！

这里是各数据的偏移量（相对于 `pPower` 也就是 `[ebp+8]`）：

- `11A8` `POWER`
- `11B0` `WanaJump`
- `1308` `IsKerPressed`
- `11A0` `NoName`

这里是 OllyDBG 中的代码:

```
0040433F   . /0F85 74690000 jnz     0040ACB9  

=======  

0040ACB9   > \\33C0          xor     eax, eax  
0040ACBB   .  837D 0C 31    cmp     dword ptr [ebp+C], 31  
0040ACBF   .  75 05         jnz     short 0040ACC6  
0040ACC1   .  B8 01000000   mov     eax, 1  
0040ACC6   >  837D 0C 32    cmp     dword ptr [ebp+C], 32  
0040ACCA   .  75 05         jnz     short 0040ACD1  
0040ACCC   .  B8 02000000   mov     eax, 2  
0040ACD1   >  837D 0C 33    cmp     dword ptr [ebp+C], 33  
0040ACD5   .  75 05         jnz     short 0040ACDC  
0040ACD7   .  B8 03000000   mov     eax, 3  
0040ACDC   >  837D 0C 34    cmp     dword ptr [ebp+C], 34  
0040ACE0   .  75 05         jnz     short 0040ACE7  
0040ACE2   .  B8 04000000   mov     eax, 4  
0040ACE7   >  837D 0C 35    cmp     dword ptr [ebp+C], 35  
0040ACEB   .  75 05         jnz     short 0040ACF2  
0040ACED   .  B8 05000000   mov     eax, 5  
0040ACF2   >  837D 0C 36    cmp     dword ptr [ebp+C], 36  
0040ACF6   .  75 05         jnz     short 0040ACFD  
0040ACF8   .  B8 06000000   mov     eax, 6  
0040ACFD   >  837D 0C 37    cmp     dword ptr [ebp+C], 37  
0040AD01   .  75 05         jnz     short 0040AD08  
0040AD03   .  B8 07000000   mov     eax, 7  
0040AD08   >  837D 0C 38    cmp     dword ptr [ebp+C], 38  
0040AD0C   .  75 05         jnz     short 0040AD13  
0040AD0E   .  B8 08000000   mov     eax, 8  
0040AD13   >  837D 0C 39    cmp     dword ptr [ebp+C], 39  
0040AD17   .  75 05         jnz     short 0040AD1E  
0040AD19   .  B8 09000000   mov     eax, 9  
0040AD1E   >  837D 0C 30    cmp     dword ptr [ebp+C], 30  
0040AD22   .  75 05         jnz     short 0040AD29  
0040AD24   .  B8 0A000000   mov     eax, 0A  
0040AD29   >  817D 0C BD000>cmp     dword ptr [ebp+C], 0BD  
0040AD30   .  75 05         jnz     short 0040AD37  
0040AD32   .  B8 0B000000   mov     eax, 0B  
0040AD37   >  817D 0C BB000>cmp     dword ptr [ebp+C], 0BB  
0040AD3E   .  75 05         jnz     short 0040AD45  
0040AD40   .  B8 0C000000   mov     eax, 0C  
0040AD45   >  85C0          test    eax, eax  
0040AD47   .^ 0F84 4196FFFF je      0040438E  
0040AD4D   .  8B5D 08       mov     ebx, dword ptr [ebp+8]  
0040AD50   .  837D 10 01    cmp     dword ptr [ebp+10], 1  
0040AD54   .  75 00         jnz     short 0040AD56  
0040AD56   >  8983 A8110000 mov     dword ptr [ebx+11A8], eax  
0040AD5C   .  C783 08130000>mov     dword ptr [ebx+1308], 0  
0040AD66   .  C783 B0110000>mov     dword ptr [ebx+11B0], 1  
0040AD70   .  8983 A0110000 mov     dword ptr [ebx+11A0], eax  
0040AD76   .^ E9 4F96FFFF   jmp     004043CA  
```

玩了一下发现：用功能键的时候不是跳，是飞！不踩地板都往上升....

大家还是把新加的功能键当作救命用的吧，不然就跟我一样不想玩它了.....

---

## 经验总结

看来动手之前的观察是很重要的，要通过观察程序的表现推测程序的运行原理，进而判断我们需要收集的信息，这样动手的时候就有了方向，可以少走不少弯路。而我这次就走了几次弯路。

---

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 04 月 26 日 13:07:33

`4dd466d0e1e0cc123afbe504da2462ca`
