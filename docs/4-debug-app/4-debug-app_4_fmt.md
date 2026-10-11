
# 天堂 1 窗口化的方式

> 原文：[`www.pediy.com/kssd/pediy10/73996.html`](https://www.pediy.com/kssd/pediy10/73996.html)

【文章标题】: 天堂 1 窗口化的方式  

【文章作者】: dengkeng  

【作者邮箱】: poppig#sohu.com  

【软件名称】: 天堂 1  

【下载地址】: 自己搜索下载  

【加壳方式】: Themida  

【保护方式】: Themida  

【使用工具】: `IDA`, `OD`, `StrongOD.dll`  

【操作平台】: xp, sp2  

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

【详细过程】  

天堂 1 窗口化的方式  

这个游戏也算是老游戏了，可能因为过于太老吧，没有一些设置选项的东西。

(现在的游戏大部分都有)所以要想着怎么样窗口化这个游戏。通用的工具大家  
都知道 `D3DWindower` 这个程序，但是使用起来还是有点问题，游戏虽然给窗口化  
了，但是鼠标的操作确实怪怪的，貌似现在的 `Gua` 提供的窗口化的功能要缴费以  
后才能够使用。尽量在网上 `google`，找到几个关于传奇的窗口的代码，主要部分  
我摘录了一下。  

1. Hook 的时机，Hook 时首先截获 `DirectDrawCreat` 函数，从中得到产生的设备  
   对象的指针的指针。这个指针是放在堆栈中的，它是一个指向 com 虚表指针的指针。  
2. 截获这个指针后，将虚表中的地址 `SetCooperatelever` 和 `SetDisplayMode`  
   函数替换成我的 `SetCooperatelever` 和 `SetDisplayMode` 函数地址。  
3. 下面就是写 `SetCooperatelever` 和 `SetDisplayMode` 两个涵数了  
   `SetDisplayMode` 屏掉，因为 DirectX 中窗口化方式下显示模式不能设置。  
   在 `SetCooperatelever` 中设置显示分鞭率为 `1024\*768`  

   当让这个是别人总结的，我们就拿来主义了。经过后面的实验，发现 3 还是有点  
   问题。(虽然屏幕给窗口化了，但是鼠标的操作还是有点问题，实际还是全屏幕，  
   只是游戏给窗口化了，有点别扭吧，^\_^，就是如果你操作游戏，实际上鼠标是点不  
   了游戏窗口以外的东西，要运行其他的东西还是需要 `Alt+Tab` 进行切换。  

   好了，下面就说说方法吧。  

方法 1: 分析别人的外挂程序。挂的名字不说了。开启外挂，然后定位到游戏代码  
`DirectDrawCreate` 的部分。`IDA` 分析字符串。  

```
___:00423660 sub_423660      proc near                  ; CODE XREF: sub_4234A0p
___:00423660
___:00423660 var_120          = byte ptr -120h
___:00423660 var_20           = dword ptr -20h
___:00423660 var_1C           = dword ptr -1Ch
___:00423660 var_10           = dword ptr -10h
___:00423660 var_C            = dword ptr -0Ch
___:00423660 var_8            = dword ptr -8
___:00423660 var_4            = dword ptr -4
___:00423660
___:00423660                   push    ebp
___:00423661                   mov     ebp, esp
___:00423663                   sub     esp, 120h
___:00423669                   call    sub_423450
___:0042366E                   cmp     eax, 10h
___:00423671                   jz      short loc_42367A
___:00423673                   mov     g_IsFullWindow, 1
___:0042367A
___:0042367A loc_42367A:                ; CODE XREF: sub_423660+11j
___:0042367A                   mov     eax, dword_67F9F8
___:0042367F                   test    eax, eax
___:00423681                   jnz     short loc_4236B3
___:00423683                   push    eax
___:00423684                   push    offset dword_67F9F8
___:00423689                   push    eax
___:0042368A                   call    sub_4D53B8
___:0042368F ; ---------------------------------------------------------------------------
___:0042368F                   test    eax, eax
___:00423691                   jz      short loc_4236AE
___:00423693                   push    eax
___:00423694                   push    offset aDirectdrawcrea ; "DirectDrawCreate failed err=%d"
___:00423699
___:00423699 loc_423699:                ; CODE XREF: sub_423660+7Aj
___:00423699                   lea     eax, [ebp+var_120]
___:0042369F                   push    eax
___:004236A0                   nop
___:004236A1                   call    near ptr 77C0F931h
___:004236A6                   add     esp, 0Ch
___:004236A9                   jmp     loc_423752
___:004236AE ; ---------------------------------------------------------------------------
___:004236AE
___:004236AE loc_4236AE:                ; CODE XREF: sub_423660+31j
___:004236AE                   mov     eax, dword_67F9F8
___:004236B3
___:004236B3 loc_4236B3:                ; CODE XREF: sub_423660+21j
___:004236B3                   mov     cl, g_IsFullWindow
___:004236B9                   test    cl, cl
___:004236BB                   mov     ecx, [eax]
___:004236BD                   jz      short loc_4236C3
___:004236BF                   push    11h
___:004236C1                   jmp     short loc_4236C5
___:004236C3 ; ---------------------------------------------------------------------------
___:004236C3
___:004236C3 loc_4236C3:                ; CODE XREF: sub_423660+5Dj
___:004236C3                   push    8
___:004236C5
___:004236C5 loc_4236C5:                ; CODE XREF: sub_423660+61j
___:004236C5                   mov     edx, g_GamehWnd
___:004236CB                   push    edx
___:004236CC                   push    eax
___:004236CD                   call    dword ptr [ecx+50h]
___:004236D0                   test    eax, eax
___:004236D2                   jz      short loc_4236DC
___:004236D4                   push    eax
___:004236D5                   push    offset aSetcooperative ; "SetCooperativeLevel failed err=%d"
___:004236DA                   jmp     short loc_423699
___:004236DC ; ---------------------------------------------------------------------------
___:004236DC
___:004236DC loc_4236DC:                ; CODE XREF: sub_423660+72j
___:004236DC                   mov     al, g_IsFullWindow
___:004236E1                   test    al, al
___:004236E3                   jz      loc_42376A
___:004236E9                   mov     ecx, g_GamehWnd
___:004236EF                   push    0FFFFFFF0h
___:004236F1                   push    ecx
___:004236F2                   call    near ptr 2F30387h ; GetWindowLongA
___:004236F7                   nop
___:004236F8                   mov     edx, g_GamehWnd
___:004236FE                   and     eax, 0FF3DFFFFh
___:00423703                   or      eax, 80000000h
___:00423708                   push    eax
___:00423709                   push    0FFFFFFF0h
___:0042370B                   push    edx
___:0042370C                   call    near ptr 2F305BDh ; SetWindowLongA
___:00423711                   nop
___:00423712                   mov     eax, dword_67F9F8
___:00423717                   push    10h
___:00423719                   push    1E0h
___:0042371E                   mov     ecx, [eax]
___:00423720                   push    280h
___:00423725                   push    eax
___:00423726                   call    dword ptr [ecx+54h]
___:00423729                   test    eax, eax
___:0042372B                   jge     loc_423885
___:00423731                   push    eax
___:00423732                   lea     edx, [ebp+var_120]
___:00423738                   push    offset aSetmodeFailedE ; "SetMode failed err=%x"
```

正常的流程，也就是全屏幕的过程，`g_IsFullWindow`，这个地方的地址，是为 1(当然这  
个后面分析出的结果，我用 `IDA` 给它重新命名了)，所以 `4236BD`, `4236E3` 这 2 个地方它不会  
跳走，所以就执行了  

```
call    dword ptr [ecx+50h]
call    dword ptr [ecx+54h]
```

这 2 个函数，通过后面的提示，以及网上的一些搜索。`ecx+0x50` 应该就是 `SetCooperativeLevel`  
的函数的地址，而 `ecx+0x54` 就是 `SetDisplayMode`，现在看看挂是怎么给窗口化的？重新再来，开游戏  
你会发现如果用挂，这个地方的流程就变化了 `g_IsFullWindow` 这个地方的值给改变了为 0 了，而不是  
原来的 1。所以方法 1 很简单 `Hook` `DirectDrawCreat` 这个函数，然后在里面修改 `g_IsFullWindow` 这个  
地址的值(因为紧跟着就是判断调用，所以我们在 `Hook` 的 `DirectDrawCreat` 里面做下手脚)  

```c
HRESULT WINAPI MyDirectDrawCreate(GUID FAR* lpGUID, LPDIRECTDRAW FAR* lplpDD, IUnknown FAR* pUnkOuter)
{
  HRESULT hResult;
  __asm
  {
    push pUnkOuter
    push lplpDD
    push lpGUID
    Call [g_DirectDrawCreate_Call]
    mov hResult,eax
  }
  *(byte *)g_IsWindow = 0;
  return hResult;
}
```



方法 1 的特点,简单,但是缺点就是需要随着游戏的更新而更新这个全局变量的地址,来适应游戏的升级.为了达到通用版本(针对这个游戏),所以我们想到了方法 2,实际上这个游戏也不是不能够自己窗口化,只是他自己没有给出让玩家自己设定而已(想不通,为什么呢?难道非要一心一意的玩游戏?)

好了,接下来我们在上面 `MyDirectDrawCreate` 里面返回 `4236BD` 这个地址跳走了,原来是不会跳走的

`push 11` 现在跳走了然后 `push 8`,这个是个什么意思呢???google 一下这个函数,需要对 `Dircet` 编程有点了解,你会发现 `SetCooperativeLevel` 的调用方式

`HRESULT SetCooperativeLevel(HWND hwnd,DWORD dwLevel);`

第一个是游戏的 `HWND`,创建窗口得来的(这个也很重要,后面会讲到),第 2 个就是我们前面看到的 2 次跳转的不同的值,默认的是 `0x11`,窗口化的这个地方传入 `0x08`

`0x11 = DDSCL_EXCLUSIVE | DDSCL_FULLSCREEN`
`0x08 = DDSCL_NORMAL`

看到了吗?这个就是全屏幕的调用,窗口则传入 `DDSCL_NORMAL` 的方式,接下来我们继续走 `4236E3` 这个地方,因为已经修改掉了,所以就是 0 了,它就跳到了 `42376A`,代码见下面,其实就处理一些窗口的东西了.

```
__:0042376A loc_42376A:                              ; CODE XREF: sub_423660+83j
__:0042376A                  mov     eax, g_GamehWnd
__:0042376F                  push    esi
__:00423770                  push    12CA0000h
__:00423775                  push    0FFFFFFF0h
__:00423777                  push    eax
__:00423778                  call    near ptr 2F305BDh ; SetWindowLongA
__:0042377D                  nop
__:0042377E                  lea     ecx, [ebp+var_10]
__:00423781                  push    1E0h
__:00423786                  push    280h
__:0042378B                  push    0
__:0042378D                  push    0
__:0042378F                  push    ecx
__:00423790                  call    near ptr 2F30737h ; SetRect
__:00423795                  nop
__:00423796                  mov     edx, g_GamehWnd
__:0042379C                  mov     esi, g_GetWindowLongA
__:004237A2                  push    -20             ; GWL_EXSTYLE
__:004237A4                  push    edx
__:004237A5                  call    esi ; g_GetWindowLongA
__:004237A7                  push    eax             ; eax = 0x40100
__:004237A8                  mov     eax, g_GamehWnd
__:004237AD                  push    eax
__:004237AE                  call    near ptr 2F30965h ; GetMenu
__:004237B3                  nop
__:004237B4                  mov     ecx, g_GamehWnd
__:004237BA                  neg     eax
__:004237BC                  sbb     eax, eax
__:004237BE                  neg     eax
__:004237C0                  push    eax
__:004237C1                  push    -16             ; GWL_STYLE
__:004237C3                  push    ecx
__:004237C4                  call    esi ; g_GetWindowLongA
__:004237C6                  lea     edx, [ebp+var_10]
__:004237C9                  push    eax
__:004237CA                  push    edx
__:004237CB                  call    near ptr 2F30D21h ; AdjustWindowRectEx
__:004237D0                  nop
__:004237D1                  mov     eax, [ebp+var_4]
__:004237D4                  mov     edx, [ebp+var_C]
__:004237D7                  mov     ecx, [ebp+var_8]
__:004237DA                  sub     eax, edx
__:004237DC                  push    16h             ; _DWORD
__:004237DE                  mov     edx, g_GamehWnd
__:004237E4                  push    eax             ; _DWORD
__:004237E5                  mov     eax, [ebp+var_10]
__:004237E8                  mov     esi, g_SetWindowPos
__:004237EE                  sub     ecx, eax
__:004237F0                  push    ecx             ; _DWORD
__:004237F1                  push    0               ; _DWORD
__:004237F3                  push    0               ; _DWORD
__:004237F5                  push    0               ; _DWORD
__:004237F7                  push    edx             ; _DWORD
__:004237F8                  call    esi ; g_SetWindowPos
__:004237FA                  mov     eax, g_GamehWnd
__:004237FF                  push    13h             ; _DWORD
__:00423801                  push    0               ; _DWORD
__:00423803                  push    0               ; _DWORD
__:00423805                  push    0               ; _DWORD
__:00423807                  push    0               ; _DWORD
__:00423809                  push    0FFFFFFFEh      ; _DWORD
__:0042380B                  push    eax             ; _DWORD
__:0042380C                  call    esi ; g_SetWindowPos
__:0042380E                  lea     ecx, [ebp+var_20]
__:00423811                  push    0
__:00423813                  push    ecx
__:00423814                  push    0
__:00423816                  push    30h
__:00423818                  call    near ptr 2F40000h ; SystemParametersInfoA
__:0042381D                  nop
__:0042381E                  mov     eax, g_GamehWnd
__:00423823                  lea     edx, [ebp+var_10]
__:00423826                  push    edx
__:00423827                  push    eax
__:00423828                  call    near ptr 2F40402h ; GetWindowRect
__:0042382D                  nop
__:0042382E                  mov     eax, [ebp+var_10]
__:00423831                  mov     ecx, [ebp+var_20]
__:00423834                  cmp     eax, ecx
__:00423836                  jge     short loc_42383D
__:00423838                  mov     eax, ecx
__:0042383A                  mov     [ebp+var_10], eax
__:0042383D
__:0042383D loc_42383D:                              ; CODE XREF: sub_423660+1D6j
__:0042383D                  mov     ecx, [ebp+var_C]
__:00423840                  mov     edx, [ebp+var_1C]
__:00423843                  cmp     ecx, edx
__:00423845                  jge     short loc_42384C
__:00423847                  mov     ecx, edx
__:00423849                  mov     [ebp+var_C], ecx
__:0042384C
__:0042384C loc_42384C:                              ; CODE XREF: sub_423660+1E5j
__:0042384C                  push    15h             ; _DWORD
__:0042384E                  push    0               ; _DWORD
__:00423850                  push    0               ; _DWORD
__:00423852                  push    ecx             ; _DWORD
__:00423853                  mov     ecx, g_GamehWnd
__:00423859                  push    eax             ; _DWORD
__:0042385A                  push    0               ; _DWORD
__:0042385C                  push    ecx             ; _DWORD
__:0042385D                  call    esi ; g_SetWindowPos
__:0042385F                  mov     edx, g_GamehWnd
__:00423865                  mov     dword_682088, 0
__:0042386F                  push    1
__:00423871                  push    0
__:00423873                  push    edx
__:00423874                  mov     dword_68208C, 0
__:0042387E                  call    near ptr 2F71B9Fh ; InvalidateRect
```

好了,既然知道了游戏的处理流程,我们就可以自己实现了.Hook `DirectDrawCreate` 是肯定的了,接下来就是要 Hook `SetCooperativeLevel` 跟 `SetDisplayMode` 了

```
HRESULT WINAPI MyDirectDrawCreate(GUID FAR* lpGUID, LPDIRECTDRAW FAR* lplpDD,IUnknown FAR* pUnkOuter)
{
    HRESULT hResult;
    __asm
    {
        push pUnkOuter
        push lplpDD
        push lpGUID
        Call [g_DirectDrawCreate_Call]
        mov hResult,eax
    }
    //这个看开头的说明
    DWORD dwDD = (DWORD)*lplpDD;
    dwDD = *(DWORD *)dwDD;
    Hook_Api(*(DWORD *)(dwDD+0x50)
        ,(DWORD)MySetCooperativeLevel,
}
```



&g\_str\_inline\_hook\_SetCooperativeLevel,&g\_SetCooperativeLevel\_Call);

Hook\_Api(\*(DWORD \*)(dwDD+0x54)

,(DWORD)MySetDisplayMode,

&g\_str\_inline\_hook\_SetDisplayMode,&g\_SetDisplayMode\_Call);

return hResult;

}

```cpp
HRESULT WINAPI MySetCooperativeLevel(DWORD dwThis,HWND hwnd,DWORD dwLevel)
{
  HRESULT hResult;
  __asm
  {
    push 0x00000008
    push hwnd
    push dwThis
    Call [g_SetCooperativeLevel_Call]
    mov hResult,eax
  }
  return hResult;
}
```

`MySetCooperativeLevel`，我们需要把参数人为的修改为 `0x08`

```cpp
HRESULT WINAPI MySetDisplayMode(DWORD dwThis,DWORD dwWidth,DWORD dwHeight,DWORD dwBit)
{
  HRESULT hResult = 1;
//   __asm
//   {
//     push dwBit
//     push dwHeight
//     push dwWidth
//     push dwThis
//     Call [g_SetDisplayMode_Call]
//     mov hResult,eax
//   }
  return hResult;
}
```

`MySetDisplayMode` 函数里面,我们什么都不做,OK,让他直接返回,就像开头说的一样

回忆一下,`SetDisplayMode` 屏掉,因为 DirectX 中窗口化方式下显示模式不能设置.然后

再开游戏,dll 注入进去,看看是否已经窗口化掉了???但是这个只是个假象,游戏虽然

窗口化掉了,但是点了窗口外面的东西没有反应,这个就是窗口化后,后面的部分没有处理

,也就是跳到 `42376A` 开始的往下执行部分,没有执行.因为默认的 Hook 调以后,执行完

`SetDisplayMode` 以后,在 `42372B` 处会跳到结尾结束.所以我们需要自己补齐那部分的

处理代码,位置嘛,当然是在 `MySetDisplayMode` 里面,毕竟在它里面什么都不做,那我们

就来做点东东了.也就是帮他执行 `42376A` 后面的代码.代码如下:

```cpp
HRESULT WINAPI MySetDisplayMode(DWORD dwThis,DWORD dwWidth,DWORD dwHeight,DWORD dwBit)
{
  HRESULT hResult = 1;
  RECT rcSect;
  SetWindowLong(g_GamehWnd,GWL_STYLE,0x12CA0000);
  SetRect(&rcSect,0,0,0x280,0x1E0);
  DWORD dwExstyle = GetWindowLong(g_GamehWnd,GWL_EXSTYLE);
  HMENU hMenu = GetMenu(g_GamehWnd);
  DWORD dwstyle = GetWindowLong(g_GamehWnd,GWL_STYLE);
  AdjustWindowRectEx(&rcSect,dwstyle,(BOOL)hMenu,dwExstyle);
  long cy = rcSect.bottom - rcSect.top;
  long cx = rcSect.right - rcSect.left;
  SetWindowPos(g_GamehWnd,0,0,0,cx,cy,0x16);
  SetWindowPos(g_GamehWnd,(HWND)0x0FFFFFFFE,0,0,0,0,0x13);
  RECT rcWorkArea;
  SystemParametersInfo(SPI_GETWORKAREA,0,&rcWorkArea,0);
  GetWindowRect(g_GamehWnd,&rcSect);
  if(rcSect.left < rcWorkArea.left)
  {
    rcSect.left = rcWorkArea.left;
  }
  if(rcSect.top < rcWorkArea.top)
  {
    rcSect.top = rcWorkArea.top;
  }
  SetWindowPos(g_GamehWnd,(HWND)0,rcSect.left,rcSect.top,0,0,0x15);
  InvalidateRect(g_GamehWnd,NULL,TRUE);
  return hResult;
}
```

在进游戏看看,是不是已经能够操作游戏窗口外的东西了,当然还有一点要说明,以为

后面的代码处理部分用到了 `g_GamehWnd`,所以为了达到通用,你可以自己处理下或者在

`MySetCooperativeLevel` 里面保存一下 `hwnd` 到 `g_GamehWnd`,也可以 Hook `CreateWindowExA`

来得到 `g_GamehWnd`,当然这个游戏用 Themida 处理过了,一些 Api 的调用都不在系统空间执

行了当然你可以继续在 `CreateWindowExA` 往下 Hook.

好了方法 2 也说完了,方法 2 的特点就是通用天 1 的所有版本,以后游戏更新也不怕了,

辛苦一次,幸福以后啊.:)至于传奇的是否可行,这个没有下游戏我就不知道了,我想应该

也应该可以吧.

在此感谢 fengyue 的 StrongOD,仅以此文献给,在网上找不到处理游戏窗口化的方式跟

`MySetDisplayMode` 代码的 XDJM

---

### 【经验总结】

有效合理运用已知外 Gua 跟 Google,:

---

### 【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!
