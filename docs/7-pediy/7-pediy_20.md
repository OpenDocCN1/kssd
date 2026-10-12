
# 是男人就上 120 层 Loader

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-247.htm`](https://www.pediy.com/kssd/pediy09/pediy09-247.htm)

> 不是我残忍，老是拿这一个游戏开刀，而是稍微大一点的游戏我就啃不动了...

> 这次用 Loader 来...![尴尬](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")

> 功能：

> 1. 注册
> 2. 阻止错误窗口的显示
> 3. 快捷键（用 1－9 键分别对应 12 个弹跳力度，不用按住空格键蓄力了）
> 4. 无敌（在屏幕最底部增加一个浮动的木板，这样玩家永远不会死～）

> 因为我这是继续以前的工作，中间用到的部分地址来的可能有点莫名其妙，这个时候你需要看我之前的分析：

> [<是男人就上 120 层 破解>](http://bbs.pediy.com/showthread.php?t=43309)
>
> [<是男人就上 120 层 Patch>](http://bbs.pediy.com/showthread.php?t=43383)

> ＝＝＝＝＝

> 1、注册
>
> 最直接的解决方法莫过于把检查注册码的函数 `004031AD` 返回值固定为 1。
>
> 做法有两种：
>
> 1. 把 `004031B6` 处的命令改为 `jmp 004032FC`
> 2. 把 `0040310B` 处的命令改为 `mov eax, 1`
>
> 采用方法 1，既将 `004031B6` 处的 `8B 45 08 33 C9` 改为 `E9 41 01 00 00`

> ```
> ==============================
> 
> const LPVOID dwRegPatch = (LPVOID)0x004031B6;
> 
> BYTE RegPatcho[] = {0x8B, 0x45, 0x08, 0x33, 0xC9};//注册-004031B6 原始数据
> 
> BYTE RegPatchp[] = {0xE9, 0x41, 0x01, 0x00, 0x00};//注册-004031B6 补丁数据
> 
> 
> 
> ReadProcessMemory(pinfo.hProcess, dwRegPatch, Buffer, sizeof(RegPatcho), &bytesRead);
> 
> if (memcmp(Buffer, RegPatcho, bytesRead) == false)
> 
> WriteProcessMemory(pinfo.hProcess, (LPVOID)dwRegPatch, RegPatchp, sizeof(RegPatchp), &bytesRead);
> 
> ==============================
> ```

> 2、阻止错误窗口的显示
>
> 把 `0040495F` 处的 `jnz` 改成 `jmp` 就可以，既将 `0F 85 4E 00 00 00` 改为 `EB 52 90 90 90 90`

> ```
> ==============================
> 
> const LPVOID dwNagPatch = (LPVOID)0x0040495F;
> 
> BYTE NagPatcho[] = {0x0F, 0x85, 0x4E, 0x00, 0x00, 0x00};//错误窗口-0040495F 原始数据
> 
> BYTE NagPatchp[] = {0xEB, 0x52, 0x90, 0x90, 0x90, 0x90};//错误窗口-0040495F 补丁数据
> 
> 
> 
> //Nag 补丁
> 
> ReadProcessMemory(pinfo.hProcess, dwNagPatch, Buffer, sizeof(NagPatcho), &bytesRead);
> 
> if (memcmp(Buffer, NagPatcho, bytesRead) == false)
> 
> WriteProcessMemory(pinfo.hProcess, dwNagPatch, NagPatchp, sizeof(NagPatchp), &bytesRead);
> 
> ==============================
> ```

> 3、快捷键
>
> 用<是男人就上 120 层 Patch>里的结果，我又把代码改进了一下：

> ```
> ==============================
> 
> 0040ACB9   > \837D 0C 31    cmp     dword ptr [ebp+C], 31
> 
> 0040ACBD   .^ 0F82 C796FFFF jb      0040438A
> 
> 0040ACC3   .  837D 0C 39    cmp     dword ptr [ebp+C], 39
> 
> 0040ACC7   .^ 0F87 BD96FFFF ja      0040438A
> 
> 0040ACCD   .  8B45 0C       mov     eax, dword ptr [ebp+C]
> 
> 0040ACD0   .  83E8 30       sub     eax, 30
> 
> 0040ACD3   .  83C0 03       add     eax, 3
> 
> 0040ACD6   .  8B5D 08       mov     ebx, dword ptr [ebp+8]
> 
> 0040ACD9   .  837D 10 01    cmp     dword ptr [ebp+10], 1
> 
> 0040ACDD   .  75 00         jnz     short 0040ACDF
> 
> 0040ACDF   >  8983 A8110000 mov     dword ptr [ebx+11A8], eax
> 
> 0040ACE5   .  C783 08130000>mov     dword ptr [ebx+1308], 0
> 
> 0040ACEF   .  C783 B0110000>mov     dword ptr [ebx+11B0], 1
> 
> 0040ACF9   .  8983 A0110000 mov     dword ptr [ebx+11A0], eax
> 
> 0040ACFF   .^ E9 C696FFFF   jmp     004043CA
> 
> ==============================
> ```

> 将 `0040433F` 处的 `jnz 0040438A` 改为 `jnz 0040ACB9`，既将 `0F 85 45 00 00 00` 改为 `0F 85 74 69 00 00`
>
> `0040ACB9` 起的代码：
>
> `83 7D 0C 31 0F 82 C7 96 FF FF 83 7D 0C 39 0F 87 BD 96 FF FF 8B 45 0C 83 E8 30 83 C0 03 8B 5D 08`
>
> `83 7D 10 01 75 00 89 83 A8 11 00 00 C7 83 08 13 00 00 00 00 00 00 C7 83 B0 11 00 00 01 00 00 00`
>
> `89 83 A0 11 00 00 E9 C6 96 FF FF`

> ```
> ==============================
> 
> const LPVOID dwCheckKey = (LPVOID)0x0040433F;
> 
> BYTE CheckKeyo[] = {0x0F, 0x85, 0x45, 0x00, 0x00, 0x00};//CheckKey-0040433F 原始数据
> 
> BYTE CheckKeyp[] = {0x0F, 0x85, 0x74, 0x69, 0x00, 0x00};//CheckKey-0040433F 补丁数据
> 
> const LPVOID dwHotKey = (LPVOID)0x0040ACB9;
> 
> BYTE Hotkeys[] = {0x83, 0x7D, 0x0C, 0x31, 0x0F, 0x82, 0xC7, 0x96, 0xFF, 0xFF, 0x83, 0x7D, 0x0C, 0x39, 0x0F,
> 
>                   0x87, 0xBD, 0x96, 0xFF, 0xFF, 0x8B, 0x45, 0x0C, 0x83, 0xE8, 0x30, 0x83, 0xC0, 0x03, 0x8B,
> 
>                   0x5D, 0x08, 0x83, 0x7D, 0x10, 0x01, 0x75, 0x00, 0x89, 0x83, 0xA8, 0x11, 0x00, 0x00, 0xC7,
> 
>                   0x83, 0x08, 0x13, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xC7, 0x83, 0xB0, 0x11, 0x00, 0x00,
> 
>                   0x01, 0x00, 0x00, 0x00, 0x89, 0x83, 0xA0, 0x11, 0x00, 0x00, 0xE9, 0xC6, 0x96, 0xFF, 0xFF};//快捷键功能实现代码-0040ACB9
> 
> 
> 
> //Hotkey 补丁
> 
> WriteProcessMemory(pinfo.hProcess, dwHotKey, Hotkeys, sizeof(Hotkeys), &bytesRead);
> 
> 
> 
> SuspendThread(pinfo.hThread);
> 
> if (sto == false)
> 
> {//打开开关
> 
> ReadProcessMemory(pinfo.hProcess, dwCheckKey, Buffer, sizeof(CheckKeyo), &bytesRead);
> 
> if (memcmp(Buffer, CheckKeyo, bytesRead) == false)
> 
> WriteProcessMemory(pinfo.hProcess, dwCheckKey, CheckKeyp, sizeof(CheckKeyp), &bytesRead);
> 
> sto = true;
> 
> }
> 
> else
> 
> {//关闭开关
> 
> ReadProcessMemory(pinfo.hProcess, dwCheckKey, Buffer, sizeof(CheckKeyp), &bytesRead);
> 
> if (memcmp(Buffer, CheckKeyp, bytesRead) == false)
> 
> WriteProcessMemory(pinfo.hProcess, dwCheckKey, CheckKeyo, sizeof(CheckKeyo), &bytesRead);
> 
> sto = false;
> 
> }
> 
> ResumeThread(pinfo.hThread);
> 
> ==============================
> ```

> 4、无敌
>
> 这个功能就没有现成的东西用了，要自己去找了。我们要找的是生成木板的函数。有了生成木板的函数，我们就能在屏幕底部给加一道保险了。
>
> 现在我们有什么线索帮助我们找到这个函数呢？小人跳起来后的落地判断应该会涉及到木板的相关数据，然后我们再根据这个数据顺藤摸瓜找到生成木板的函数。首先想到的是 `Jump` 函数。
>
> `Jump` 函数的分析：这个函数里调用了 `SetRect`、`IntersectRect` 两个 API,一个是创建矩形一个是求两个矩形的相交部分。猜测程序是用小人的矩形和其他各个木板的矩形求相交的部分，若没有任何一个木板和它相交则说明小人处于悬空状态。经过分析，发现最初创建的那个矩形就是小人。而和它求相交的矩形一共有 `0xB` 个，这些矩形是在函数 `004010F3` 里创建的。几乎可以肯定这些矩形就是木板，现在要把存放这些木板坐标的内存找出来。
>
> 注意到这一段：

> ```
> 00406035   > /6A 01         push    1
> 
> 00406037   . |6A 60         push    60
> 
> 00406039   . |8B45 D8      mov     eax, dword ptr [ebp-28]
> 
> 0040603C   . |8D0440       lea     eax, dword ptr [eax+eax*2]
> 
> 0040603F   . |8B4D 08      mov     ecx, dword ptr [ebp+8]
> 
> 00406042   . |8B84C1 BC1100>mov    eax, dword ptr [ecx+eax*8+11BC]
> 
> 00406049   . |50          push    eax
> 
> 0040604A   . |8B45 D8      mov     eax, dword ptr [ebp-28]
> 
> 0040604D   . |8D0440       lea     eax, dword ptr [eax+eax*2]
> 
> 00406050   . |8B4D 08      mov     ecx, dword ptr [ebp+8]
> 
> 00406053   . |BB 0A000000  mov     ebx, 0A
> 
> 00406058   . |8B84C1 C01100>mov    eax, dword ptr [ecx+eax*8+11C0]
> 
> 0040605F   . |99          cdq
> 
> 00406060   . |F7FB        idiv    ebx
> 
> 00406062   . |8B4D D8      mov     ecx, dword ptr [ebp-28]
> 
> 00406065   . |8D0C49       lea     ecx, dword ptr [ecx+ecx*2]
> 
> 00406068   . |8B55 08      mov     edx, dword ptr [ebp+8]
> 
> 0040606B   . |8B8CCA B81100>mov    ecx, dword ptr [edx+ecx*8+11B8]
> 
> 00406072   . |03C8        add     ecx, eax
> 
> 00406074   . |51          push    ecx
> 
> 00406075   . |8D45 E0      lea     eax, dword ptr [ebp-20]
> 
> 00406078   . |50          push    eax
> 
> 00406079   . |E8 75B0FFFF  call    004010F3
> ```
>
> 函数 `004010F3` 中调用了 `SetRect`，实际上这个函数是起了个适配器的作用：这个函数的参数是高、宽和左下角点坐标，而 `SetRect` 的参数是左下角坐标和右上角坐标。
>
> ```
> 0040606B   . |8B8CCA B81100>mov    ecx, dword ptr [edx+ecx*8+11B8]
> ```
>
> 中的`[edx+ecx*8+11B8]`应该就是几个木板中的一个的左坐标。在`[edx+ecx*8+11B8]`所指向的内存下写断点，重新开始游戏会断在 `00404DBD` 函数中。中断 11 次后游戏正常运行，向上跳几层后游戏中断在 `MoveScreen` 函数中。现在有两处线索了。分析两处，发现 `00404DBD` 这个函数只是初始化，`MoveScreen` 里才是游戏进行时生成木板的地方。但是我在分析 `00404DBD` 的时候发现了有趣的东西，木板的信息是
>
> ```
> struct Board
> {
>  DWORD type;
>  POINT pt;
>  DWORD unkown1;
> 
>  DWORD unkown2;
> 
>  DWORD unkown3;
> 
> };
> ```
>
> 结构体成员 `type` 是木板的类型，而类型为 5 的时候，生成的是地板...快要摸到瓜了，现在需要做的工作是分析 `MovScreen` 函数，找找下手的地方……
>
> `MoveScreen` 大致流程是这样的：对于所有的木板，加上屏幕要上移的距离，当有哪块木板的 y 坐标加上要上移的距离后超出 `0x160`，既要移动到屏幕之外的时候，就生成一个新的木板（新木板的数据覆盖掉那块要移动到屏幕之外的木板的数据）。我们想弄一个跟着屏幕上升的地板，在这里做手脚就可以了。下面这段代码是判断一块木板是否会移动到屏幕之外：

> ```
> 00406345  |> \8B45 FC      |mov     eax, dword ptr [ebp-4]
> 
> 00406348  |.  8D0440      |lea     eax, dword ptr [eax+eax*2]
> 
> 0040634B  |.  8B4D 08     |mov     ecx, dword ptr [ebp+8]
> 
> 0040634E  |.  8B84C1 BC1100>|mov    eax, dword ptr [ecx+eax*8+11BC]
> ```



## 汇编代码分析

```
00406355  |.  0345 0C       |add     eax, dword ptr [ebp+C]  
00406358  |.  3D 60010000   |cmp     eax, 160  
0040635D  |.  0F8C D0000000 |jl      00406433  
```

而 `00406433` 处的代码是更新那些不会被移动到屏幕外面的木板的坐标的：

```
00406433  |>  \8B45 0C      |mov     eax, dword ptr [ebp+C]  
00406436  |.  8B4D FC      |mov     ecx, dword ptr [ebp-4]  
00406439  |.  8D0C49       |lea     ecx, dword ptr [ecx+ecx*2]  
0040643C  |.  8B55 08      |mov     edx, dword ptr [ebp+8]  
0040643F  |.  0184CA BC1100>|add    dword ptr [edx+ecx*8+11BC], eax  
```

### 逻辑修改思路

我们想弄一个跟着屏幕上升的地板，只需要在 `0040635D` 处跳转之前判断当前处理的是不是地板，如果是地板的话就跳过 `00406433` 处代码，阻止程序修改地板的位置，这样地板的位置就会始终保持在屏幕最底部。

先用脚本试试这个方法是否行的通：

```
==============脚本分割线==============
var bak
var addr
eob break

break:
//获得当前处理的木板的 type 属性
mov addr, eax
shl addr, 3
add addr, ecx
add addr, 11B4
mov bak, [addr]

//若当前处理的木板是地板，则直接跳到 406446 处
cmp bak, 5
jne continue

bp634E:
mov eip, 406446

continue:
run
==============脚本分割线==============
```

但是这里还有一个问题：我要无敌这个功能是可以打开关闭的，而上面这个实现方法只能从游戏一开始就使用才能起作用。因为游戏进行到一半的时候才打开开关的话，很可能就没有地板了，更不要说保持地板的坐标了。

很明显，这个问题的意思是：我的算法是在有地板这个前提之下的，我怎么通过保证这个前提来保证我的算法工作正常呢？

答案就是刚打开开关的时候检查是否有地板，没有的话就找一个要被移出屏幕底端的木板并把它修改成地板（随便找一个的话会产生一个木板凭空消失的现象，显得不自然）。

### 实现方案

```
==============汇编代码割线==============
0040634E     /FF25 F0EF4000        jmp     dword ptr [40EFF0]               ;  是男人就.0040AD10  

//维持地板上升的代码段
0040AD10   .  50                  push    eax  
0040AD11   .  8B84C1 B4110000     mov     eax, dword ptr [ecx+eax*8+11B4]  
0040AD18   .  83F8 05            cmp     eax, 5  
0040AD1B   .  58                  pop     eax  
0040AD1C   .^ 0F84 24B7FFFF       je      00406446  
0040AD22   .  8B84C1 BC110000     mov     eax, dword ptr [ecx+eax*8+11BC]  
0040AD29   .^ E9 27B6FFFF         jmp     00406355  

//刚打开开关的时候的操作：寻找是否有地板，
//有的话则把维持地板上升的代码段补丁到程序中;
//如果没有地板则把改造一个木板的代码段补丁到程序中。
0040AD30   .  53                  push    ebx  
0040AD31   .  52                  push    edx  
0040AD32   .  BA 00000000         mov     edx, 0  
0040AD37   >  8D0452             lea     eax, dword ptr [edx+edx*2]  
0040AD3A   .  8B9CC1 B4110000     mov     ebx, dword ptr [ecx+eax*8+11B4]  
0040AD41   .  83FB 05            cmp     ebx, 5  
0040AD44      74 48              je      short 0040AD8E  
0040AD46   .  42                  inc     edx  
0040AD47   .  83FA 0E            cmp     edx, 0E  
0040AD4A   .^ 76 EB              jbe     short 0040AD37  
0040AD4C   .  C705 F0EF4000 60AD4000     mov     dword ptr [40EFF0], 0040AD60  
0040AD56   .  5A                  pop     edx  
0040AD57   .  5B                  pop     ebx  
0040AD58   .^ E9 E9B6FFFF         jmp     00406446  

//发现一个要移出屏幕底部的木块后把它改造成地板，然后把维持地板上升的代码段补丁到程序中
0040AD60   .  8B45 FC            mov     eax, dword ptr [ebp-4]  
0040AD63   .  8D0440             lea     eax, dword ptr [eax+eax*2]  
0040AD66   .  C784C1 B4110000 05000000   mov     dword ptr [ecx+eax*8+11B4], 5  
0040AD71      C784C1 B8110000 10000000   mov     dword ptr [ecx+eax*8+11B8], 10  
0040AD7C   .  C784C1 BC110000 50010000   mov     dword ptr [ecx+eax*8+11BC], 150  
0040AD87   .  EB 07              jmp     short 0040AD90  

//把维持地板上升的代码段补丁到程序中
0040AD8E      5A                  pop     edx  
0040AD8F      5B                  pop     ebx  
0040AD90   >  C705 F0EF4000 10AD4000     mov     dword ptr [40EFF0], 0040AD10  
0040AD9A     ^ E9 A7B6FFFF        jmp     00406446  
==============汇编代码割线==============
```

程序流程就是先让程序执行 `0040AD30` 处代码，寻找是否有地板，然后执行相应操作，这是初始化操作。完成初始化后，就可以把 `0040AD10` 处代码“连接”到程序里了。

### 程序代码实现

```
==============程序代码割线==============
const LPVOID dwMoveFloor = (LPVOID)0x0040634E;//补丁的位置
const LPVOID dwJMPAddr = (LPVOID)0x0040EFF0;//存放跳转地址的指针
const DWORD dwFirstJMP = 0x0040AD30;//初始化跳转地址
BYTE MoveFlooro[] = {0x8B, 0x84, 0xC1, 0xBC, 0x11, 0x00, 0x00};//原始数据
BYTE MoveFloorp[] = {0xFF, 0x25, 0xF0, 0xEF, 0x40, 0x00, 0x90};//补丁数据

const LPVOID dwMoveFloors = (LPVOID)0x0040AD10;
BYTE MoveFloors[] = {0x50, 0x8B, 0x84, 0xC1, 0xB4, 0x11, 0x00, 0x00, 0x83, 0xF8, 0x05, 0x58, 0x0F, 0x84, 0x24, 0xB7, 0xFF, 
          0xFF, 0x8B, 0x84, 0xC1, 0xBC, 0x11, 0x00, 0x00, 0xE9, 0x27, 0xB6, 0xFF, 0xFF, 0x00, 0x00, 0x53, 0x52, 
          0xBA, 0x00, 0x00, 0x00, 0x00, 0x8D, 0x04, 0x52, 0x8B, 0x9C, 0xC1, 0xB4, 0x11, 0x00, 0x00, 0x83, 0xFB, 
          0x05, 0x74, 0x48, 0x42, 0x83, 0xFA, 0x0E, 0x76, 0xEB, 0xC7, 0x05, 0xF0, 0xEF, 0x40, 0x00, 0x60, 0xAD, 
          0x40, 0x00, 0x5A, 0x5B, 0xE9, 0xE9, 0xB6, 0xFF, 0xFF, 0x90, 0x90, 0x90, 0x8B, 0x45, 0xFC, 0x8D, 0x04, 
          0x40, 0xC7, 0x84, 0xC1, 0xB4, 0x11, 0x00, 0x00, 0x05, 0x00, 0x00, 0x00, 0xC7, 0x84, 0xC1, 0xB8, 0x11, 
          0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0xC7, 0x84, 0xC1, 0xBC, 0x11, 0x00, 0x00, 0x50, 0x01, 0x00, 0x00, 
          0xEB, 0x07, 0x00, 0x00, 0x00, 0x00, 0x00, 0x5A, 0x5B, 0xC7, 0x05, 0xF0, 0xEF, 0x40, 0x00, 0x10, 0xAD, 
          0x40, 0x00, 0xE9, 0xA7, 0xB6, 0xFF, 0xFF};//MoveFloor 功能代码

//MovingFloor 补丁
WriteProcessMemory(pinfo.hProcess, dwMoveFloors, MoveFloors, sizeof (MoveFloors), &bytesRead);

SuspendThread(pinfo.hThread);
if (inv == false)
{//打开开关
  ReadProcessMemory(pinfo.hProcess, dwMoveFloor, Buffer, sizeof (MoveFlooro), &bytesRead);
  if (memcmp(Buffer, MoveFlooro, bytesRead) == false)
  {
    WriteProcessMemory(pinfo.hProcess, dwMoveFloor, MoveFloorp, sizeof (MoveFloorp), &bytesRead);
    WriteProcessMemory(pinfo.hProcess, dwJMPAddr, &dwFirstJMP, sizeof (dwFirstJMP), &bytesRead);
  }
  inv = true;
}
else
{//关闭开关
  ReadProcessMemory(pinfo.hProcess, dwMoveFloor, Buffer, sizeof (MoveFloorp), &bytesRead);
  if (memcmp(Buffer, MoveFloorp, bytesRead) == false)
    WriteProcessMemory(pinfo.hProcess, dwMoveFloor, MoveFlooro, sizeof (MoveFlooro), &bytesRead);
  inv = false;
}
ResumeThread(pinfo.hThread);
==============程序代码割线==============
```

