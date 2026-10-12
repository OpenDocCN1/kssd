
# win7 自动扫雷

> 原文：[`www.pediy.com/kssd/pediy11/120152.html`](https://www.pediy.com/kssd/pediy11/120152.html)

**【文章标题】**: win7 自动扫雷

**【作者声明】**: 参考了看雪的一篇帖子 `http://bbs.pediy.com/showthread.php?t=40295`，感谢该帖作者！

打开 win7 扫雷，感觉无从下手，来看雪搜索了一下，找到一篇帖子《Vista 的扫雷》，看完之后豁然开朗。该帖子分析得很清楚了，我就不多说了，直接开干了。

首先下断 `bp rand`，直接就断下来了，执行到返回，发现是一个小函数，很多地方都会调用这个函数，而且会不停中断。

```
008DD7F3  /$  8BFF          mov     edi, edi  ；鼠标点到这一行的时候，OD 会在代码窗口下方提示“本地调用来自。。。”

008DD7F5  |.  55           push    ebp

008DD7F6  |.  8BEC         mov     ebp, esp

008DD7F8  |.  FF15 B8168B00 call    dword ptr [<&msvcrt.rand>]     ; [rand

008DD7FE  |.  8B4D 0C      mov     ecx, dword ptr [ebp+C]

008DD801  |.  2B4D 08      sub     ecx, dword ptr [ebp+8]

008DD804  |.  99           cdq

008DD805  |.  41           inc     ecx

008DD806  |.  F7F9         idiv    ecx

008DD808  |.  8BC2         mov     eax, edx

008DD80A  |.  0345 08      add     eax, dword ptr [ebp+8]

008DD80D  |.  5D           pop     ebp

008DD80E  \.  C2 0800      retn    8
```

在每个调用该函数的 `call` 都下断，然后清除掉一直中断的一个 `call`（`rand` 的中断也要清除掉），然后回到游戏，点击鼠标发现完全不会中断下来，于是开始新游戏，点击一个格子，断下来了。

```
008D0163  |. /EB 31        jmp     short 008D0196

008D0165  |> |33FF         xor     edi, edi

008D0167  |. |EB 2D        jmp     short 008D0196

008D0169  |> |8B45 FC      /mov    eax, dword ptr [ebp-4]

008D016C  |. |8B00         |mov    eax, dword ptr [eax]

008D016E  |. |85C0         |test   eax, eax

008D0170  |. |76 2B        |jbe    short 008D019D

008D0172  |. |48           |dec    eax

008D0173  |. |50          |push   eax

008D0174  |. |6A 00       |push   0

008D0176  |. |E8 78D60000  |call   008DD7F3

008D017B  |. |8BD8        |mov    ebx, eax

008D017D  |. |8B45 FC     |mov    eax, dword ptr [ebp-4]

008D0180  |. |8B40 0C     |mov    eax, dword ptr [eax+C]

008D0183  |. |FF3498      |push   dword ptr [eax+ebx*4]

008D0186  |. |8BCF        |mov    ecx, edi

008D0188  |. |E8 C4580100  |call   008E5A51

008D018D  |. |8B4D FC     |mov    ecx, dword ptr [ebp-4]

008D0190  |. |53          |push   ebx

008D0191  |. |E8 08EB0000  |call   008DEC9E

008D0196  |> \8B07         mov    eax, dword ptr [edi]

008D0198  |.  3B46 04      |cmp    eax, dword ptr [esi+4]

008D019B  |.^ 75 CC        \jnz    short 008D0169
```

此处代码跟《Vista 的扫雷》完全一样，我就不献丑了，直接 `Ctrl+F9`，回到这里：

```
008D0C7F  |.  A1 B4689300  mov     eax, dword ptr [9368B4]  ；这里就是我们要的了

008D0C84  |.  3848 18      cmp     byte ptr [eax+18], cl

008D0C87  |.  74 5C       je      short 008D0CE5

008D0C89  |.  51          push    ecx

008D0C8A  |.  51          push    ecx

008D0C8B  |.  51          push    ecx

008D0C8C  |.  E8 58F90000 call    008E05E9

008D0C91  |.  33C9        xor     ecx, ecx

008D0C93  |.  EB 50       jmp     short 008D0CE5

008D0C95  |>  394E 18     cmp     dword ptr [esi+18], ecx

008D0C98  |.  75 20      jnz     short 008D0CBA

008D0C9A  |.  57          push    edi

008D0C9B  |.  53          push    ebx

008D0C9C  |.  8BCE        mov     ecx, esi

008D0C9E  |.  E8 18F4FFFF call    008D00BB

008D0CA3  |.  6A 00       push    0      ；Ctrl+F9 后出现在这里

008D0CA5  |.  57          push    edi

008D0CA6  |.  53          push    ebx

008D0CA7  |.  6A 00       push    0

008D0CA9  |.  57          push    edi

008D0CAA  |.  53          push    ebx

008D0CAB  |.  8BCE        mov     ecx, esi

008D0CAD  |.  E8 90FDFFFF call    008D0A42
```

现在我们得到一个稳定的地址：`minesweeper.exe+868B4`，从这里可以知道某个格子有没有雷。公式是这样的：`[[[minesweeper.exe+868B4]+10]+44]+c]+4*(x*高度+y)`，这个（`x`，`y`）的设定是假设扫雷的左上角是（0，0），往右 `x` 增加，往下 `y` 增加。

那么现在知道哪里有雷哪里没雷，想做手脚就很容易了，比如可以透视出哪里有雷，或者直接自动扫雷。《Vista 的扫雷》中是模拟鼠标的移动和点击来实现的，本文则是使用 `CE`（`Cheat Engine`）注入代码，创建远程线程直接调用扫雷程序的 `call` 来扫雷，优点是不用计算客户区和格子的坐标，缺点是不稳定，容易使扫雷程序崩溃。。。

下面我们来找一下程序用来翻开格子的 `call`，先在 `minesweeper.exe+868B4` 下硬件访问断点，发现一直断，只能清除掉，试试 `[minesweeper.exe+868B4]+10`，还是一直断，继续试 `[[minesweeper.exe+868B4]+10]+44`，不会断了，回到游戏，翻开一个格子，中断了。清除硬件断点，按 `Ctrl+F9` 四次之后来到这里，就是关键的 `call` 了：

```
008EA3F3  |> \8B0D 54749300 mov     ecx, dword ptr [937454]

008EA3F9  |.  85C9          test    ecx, ecx

008EA3FB  |.  0F84 5E030000 je      008EA75F

008EA401  |.  6A 00         push    0

008EA403  |.  FF75 10       push    dword ptr [ebp+10]

008EA406  |.  53            push    ebx

008EA407  |.  E8 635DFFFF   call    008E016F    ；刚才我们就在这个 call 里

008EA40C  |. /E9 4E030000   jmp     008EA75F
```

为什么要按四次呢？为什么判定这里就是关键 `call` 呢？因为我已经跟过了所以我知道。。。

现在马后炮分析一下，在刚才硬件断点中断的地方按一次 `Ctrl+F9`，出来之后是一个 `switch` 的分支，在每个分支上下断点，回到游戏继续点格子，发现 `case 7` 是鼠标按下，`case 14` 是鼠标弹起，我们不关心鼠标按下，所以只要分析 `case 14` 就好了。

看看 `case 14`，发现这里除了用到 `minesweeper.exe+868B4` 之外，还有一个 `ESI` 的参数也参与了重要计算，追随一下 `ESI`，发现在函数开头有一个 `mov esi, ecx`，于是继续 `Ctrl+F9`，来到这里：

```
008E6835  /$  8BFF          mov     edi, edi

008E6837  |.  55            push    ebp

008E6838  |.  8BEC         mov     ebp, esp

008E683A  |.  53            push    ebx

008E683B  |.  8B5D 08      mov     ebx, dword ptr [ebp+8]

008E683E  |.  56            push    esi

008E683F  |.  8BF1         mov     esi, ecx

008E6841  |.  57            push    edi

008E6842  |.  33FF         xor     edi, edi

008E6844  |.  8973 04      mov     dword ptr [ebx+4], esi

008E6847  |.  39BE 90000000 cmp     dword ptr [esi+90], edi

008E684D  |.  76 17        jbe     short 008E6866

008E684F  |>  8B86 9C000000 /mov    eax, dword ptr [esi+9C]

008E6855  |.  8B0CB8       |mov    ecx, dword ptr [eax+edi*4]

008E6858  |.  8B01        |mov    eax, dword ptr [ecx]

008E685A  |.  53          |push   ebx

008E685B  |.  FF10        |call   dword ptr [eax]

008E685D  |.  47          |inc    edi      ；出来的时候在这里

008E685E  |.  3BBE 90000000 |cmp    edi, dword ptr [esi+90]

008E6864  |.^ 72 E9       \jb     short 008E684F

008E6866  |>  5F           pop     edi

008E6867  |.  5E          pop     esi

008E6868  |.  5B          pop     ebx

008E6869  |.  5D          pop     ebp

008E686A  \.  C2 0400     retn    4
```

可以看出，`call` 的时候 `ecx` 是这么来的：`[[esi+9C]+edi*4]`，`edi` 是计数的不管他，`esi` 呢？又一个 `mov esi, ecx`，那么再一次 `Ctrl+F9`，来到这里：

```
008E048B  |.  8B4D 08      mov     ecx, dword ptr [ebp+8]

008E048E  |.  8B01        mov     eax, dword ptr [ecx]

008E0490  |.  57          push    edi

008E0491  |.  FF50 08     call    dword ptr [eax+8]

008E0494  |>  838E 88000000>or     dword ptr [esi+88], FFFFFFFF  ；出来的时候在这里
```

看到一个 `mov ecx, dword ptr [ebp+8]`，感觉好像是参数传进来的，要不就是个临时变量，我们来分析一下这个 `[ebp+8]` 怎么来的。再一次 `Ctrl+F9`（就是关键 `call` 啦），下个断点，回到游戏，翻开一个格子，中断在这里：

```
008EA407  |.  E8 635DFFFF   call    008E016F
```

`F7` 跟进去，一路只注意 `[ebp+8]` 的内存变化，发现在执行完这个 `call` 之后，`[ebp+8]` 就被赋值了：

```
008E0470  |.  8BCE         mov     ecx, esi

008E0472  |.  0F94C0      sete    al

008E0475  |.  8847 0C     mov     byte ptr [edi+C], al

008E0478  |.  8D45 08     lea     eax, dword ptr [ebp+8]

008E047B  |.  50          push    eax

008E047C  |.  FFB6 88000000 push    dword ptr [esi+88]

008E0482  |.  E8 BAF40000 call    008EF941
```

那么在这个 `call` 下断，回到游戏，翻开一个格子，中断到这个 `call` 的时候，`F7` 跟进去：

```
008EF941  /$  8BFF          mov     edi, edi

008EF943  |.  55            push    ebp

008EF944  |.  8BEC         mov     ebp, esp

008EF946  |.  56            push    esi

008EF947  |.  8BF1         mov     esi, ecx

008EF949  |.  FF76 08      push    dword ptr [esi+8]

008EF94C  |.  6A 00        push    0

008EF94E  |.  FF75 08      push    dword ptr [ebp+8]

008EF951  |.  E8 06F3FeFF call    008DEC5C

008EF956  |.  85C0         test    eax, eax

008EF958  |.  7D 04       jge     short 008EF95E

008EF95A  |.  32C0        xor     al, al

008EF95C  |.  EB 0D       jmp     short 008EF96B

008EF95E  |>  8B4E 04     mov     ecx, dword ptr [esi+4]
```



### 汇编代码段分析

```asm
008EF961  |.  8B0481        mov     eax, dword ptr [ecx+eax*4]  
008EF964  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]  
008EF967  |.  8901          mov     dword ptr [ecx], eax  
008EF969  |.  B0 01         mov     al, 1  
008EF96B  |>  5E            pop     esi  
008EF96C  |.  5D            pop     ebp  
008EF96D  \.  C2 0800       retn    8  
```

啥也不说了，还是一个 `mov esi, ecx`，那这个函数就不看了，回到上层，找一下 `ECX` 吧。。。往上找了几行，就看到了：

```asm
008E0470  |.  8BCE          mov     ecx, esi  
```

好，现在任务又变成找 `ESI` 了，往上找，正如你想到的，函数的开头有个 `mov esi, ecx`，绕的好晕啊，`Ctrl+F9` 回上层找吧：

```asm
008EA3F3  |>  8B0D 54749300 mov     ecx, dword ptr [937454]  
008EA3F9  |.  85C9          test    ecx, ecx  
008EA3FB  |.  0F84 5E030000 je      008EA75F  
008EA401  |.  6A 00         push    0  
008EA403  |.  FF75 10       push    dword ptr [ebp+10]  
008EA406  |.  53            push    ebx  
008EA407  |.  E8 635DFFFF   call    008E016F  
```

看到了，`mov ecx, dword ptr [937454]`，真不容易啊。。。`ecx` 可以找到了，那么这个 `call` 的几个参数分别是干啥的呢？跟了几次，发现全是 0。。。那么就当作全是 `push 0` 好了。

接下来分析 `ecx` 指向的结构，经过几次实验和观察，发现 `call` 内部使用到的地方也不多，而有意义的就这么几个：

*   `[ecx+b8]`：0 表示鼠标按下，1 表示鼠标弹起
*   `[ebx+88]`，`[ebx+10]`：这两个地址的值都是格子的 `index=x*高度+y+1`，因为这个 `index` 是从 1 开始计数的，所以后面有个 `+1`。
*   `[ebx+7c]`：值为 `[[ecx+4]+4*(x*高度+y)]`，一个指针，指向的地方比较有意思，记录了翻开的格子上面显示的数字，也记录了没翻开的和有雷的格子，具体内容大家自己跟一下看就知道了。

大家可以试一下再 `Ctrl+F9`，发现已经出了程序的领空了，来到 `USER32` 里面了，所以不能再往上找了，就在这里折腾就好了。

那么现在要这么做：

循环每个格子，遇到没有雷的格子，就 `mov ecx, dword ptr [937454]`，然后设置一下几个重要的参数，调用 `call`，继续循环。

用 `CE` 的 `Auto assemble`，输入以下内容：

```asm
alloc(myscript,1024)

globalalloc(mydata,256)

define(adr1,minesweeper.exe+868B4) 

define(adr2,minesweeper.exe+87454) 

define(adrcall,minesweeper.exe+3016F) 

define(height,mydata+0) 

define(width,mydata+4) 

label(end) 

label(in_width) 

label(in_height) 

myscript: 

//Call this code to execute the script from assembler

mov ecx, dword ptr [adr2] 

test ecx, ecx 

je end 

mov edi, dword ptr [adr1] 

mov edi,[edi+10] 

mov ebx,[edi+8] 

mov [height],ebx 

mov ebx,[edi+c] 

mov [width],ebx 

mov edi,[edi+44] 

mov edi,[edi+c] 

mov ecx,[width] 

in_width: 

test ecx,ecx 

je end 

dec ecx 

mov eax,[edi+ecx*4] 

mov eax,[eax+c] 

mov edx,[height] 

in_height: 

test edx,edx 

je in_width 

dec edx 

xor ebx,ebx 

mov bl,byte ptr [eax+edx] 

cmp bl,0 

jnz in_height 

pushad 

mov ebx, dword ptr [adr2] 

inc esi 

mov [ebx+b8],esi 

mov eax,[height] 

push edx 

mul ecx 

pop edx 

add eax,edx 

push eax 

shl eax,2 

mov edx,[ebx+4] 

mov edx,[edx+eax] 

mov [ebx+7c],edx 

pop eax 

inc eax 

mov [ebx+88],eax 

mov [ebx+10],eax 

mov ecx,ebx 

push 0 

push 0 

push 0 

call adrcall 

push 32 

call Sleep 

popad 

jmp in_height 

end: 

ret 
```

`execute` 一下，`CE` 会告诉你代码注入到什么位置了，记下来，然后开多一个 `Auto assemble` 的窗口，输入这个：

```asm
createthread(地址) 
```

这个地址就是刚才代码注入的地址了，然后 `execute`，启动远程线程，就开始刷刷刷的自动扫雷了。

脚本写的不好请大家不吝赐教！还有就是这样注入经常会导致程序崩溃，哪位高人知道的请指点一下，多谢！

下面是我的实现，似乎没有进程 `CRASH` 的情况，这里只贴下代码

我 `BLOG` 里有更多说明，这里不直接贴了

[使用 `CALL` 调用实现扫雷 ---- 原理](http://cnbeta2004.blog.163.com/blog/static/6023134020108209252801/)

[使用 `CALL` 调用实现扫雷 ---- 编码](http://cnbeta2004.blog.163.com/blog/static/602313402010820101425385/)

```cpp
void GameForm::OnBnClickedBtnAuto2()

{

  HMODULE hMine = GetModuleHandle(_T("Minesweeper.exe")); 

  if(hMine==NULL)

  {

    AfxMessageBox(_T("无法找到模块 mineswpeer"));

    return;

  }

  //模块句柄即为基址

  DWORD mineBaseAddr = (DWORD)hMine; 

  const int RVA_top = 0x868B4;  //数据基址 RVA

  const int RVA_fun = 0x21418;  //扫雷 CALL RVA

  const int RVA_fun2 = 0x26BCD;  //后续函数 RVA

  const int RVA_fun3 = 0x200BB;  //布雷函数 RVA

  m_VATOP = mineBaseAddr + (DWORD)RVA_top;

  m_VAFUN = mineBaseAddr + (DWORD)RVA_fun;

  m_VAFUN2 = mineBaseAddr + (DWORD)RVA_fun2;

  m_VAFUN3 = mineBaseAddr + (DWORD)RVA_fun3;

  HitBlock2(0,0);

}

void GameForm::HitBlock2(int arow, int acol)

{

  int va_top = m_VATOP;

  int va_fun = m_VAFUN;

  int va_fun2 = m_VAFUN2;

  int va_fun3 = m_VAFUN3;

  _asm

  {

    mov esi,va_top

    mov esi,[esi]

    mov eax,[esi+0x10]

    mov edi,arow    //行

    mov ebx,acol    //列

//    mov [eax+0x18],0    //edi+18=0,zf=1,第一次

    mov [esi+0xC5],1

    mov ecx,esi

    push edi

    push ebx

    xor bl,bl

    mov eax,va_fun

    call eax        //第一次点击方块

    mov edi,0

    mov ebx,0

  /*  mov eax,[esi+0x10]*/

  /*  mov [eax+0x18],1*/

row_:

    push edi

column_:

    push ebx

    mov     eax, dword ptr [esi+0x10]

    mov     eax, dword ptr [eax+0x44]

    mov     eax, dword ptr [eax+0x0C]

    mov     eax, dword ptr [eax+ebx*4]

    mov     eax, dword ptr [eax+0x0C]

    xor     ecx, ecx

    cmp     byte ptr [edi+eax], cl      //  判断是不是雷

    jnz     end_        //  雷跳转到最后

    mov [esi+0xC5],1

    mov ecx,esi

    push edi

    push ebx

    xor bl,bl

    mov eax,va_fun

    call eax        //调用点击方块 CALL

    test eax,eax

    jg here_

    jmp end_

here_:

    push eax

    mov eax,va_fun2

    call eax        //后续处理 CALL

end_:

    pop ebx

    inc ebx

    mov eax,[esi+0x10]

    cmp ebx,[eax+0x0C]

    jb column_

    pop edi

    mov ebx,0

    inc edi

    cmp edi,[eax+0x08]

    jb row_

  }

}
```

汇编码也写的不好。
