
# 代码分析

## 初始化与调用
```
0000:00404724                 call    sub_403F94
```
```
0000:00404729                 mov     ds:word_4060D8, ax
```
// 字符串里的换行数 + 1。注意这里的初始值为 1。就是说，即使是空字符串，也应该在末尾加个换行。但是这里用了 `mov`，联系到 `sub_403F94` 里有 `SEH`，很容易想到在异常情况下，初始值起作用的。

```
0000:0040472F                 push    0               ; lpParam
0000:00404731                 movsx   eax, ds:word_4060CC
0000:00404738                 push    eax             ; hInstance
0000:00404739                 movsx   eax, ds:word_4060C8
0000:00404740                 push    eax             ; hMenu
0000:00404741                 movsx   eax, ds:word_4060D8
0000:00404748                 shl     eax, 2
0000:0040474B                 lea     eax, [eax+eax*2]
0000:0040474E                 push    eax             ; hWndParent
0000:0040474F                 mov     eax, ds:hWnd
0000:00404754                 push    eax             ; nHeight
0000:00404755                 push    0               ; nWidth
0000:00404757                 mov     eax, ds:hInstance
0000:0040475C                 push    eax             ; Y
0000:0040475D                 push    0               ; X
```
留意一下，IDA 已经第三次发飚了。而且这次跟前两次不一样。如果真按照注释对参数是要死人的！反正…… `push` 的参数顺序全部倒了……

```
0000:0040475F                 mov     edx, offset dword_404A28 ; lpWindowName
0000:00404764                 mov     eax, offset dword_404A2C ; lpClassName
```
象上面这样只有一个引用的 `lpstr` 就直接写字符串吧。

```
0000:00404769                 mov     ecx, 50000001h  ; dwStyle
```
这里的值是 3 个值的组合。由于该窗口是 `Static`，所以那个 1 是 `SS_CENTER`

```
0000:0040476E                 call    sub_403C98
```
调用 `CreateWindowEx`

```
0000:00404773                 mov     ds:dword_4060D4, eax
0000:00404778                 push    offset sub_404484 ; dwNewLong
```
专用的 `callback`

```
0000:0040477D                 push    0FFFFFFFCh      ; nIndex
0000:0040477F                 mov     eax, ds:dword_4060D4
0000:00404784                 push    eax             ; hWnd
0000:00404785                 call    SetWindowLongA
0000:0040478A                 push    eax             ; dwNewLong
0000:0040478B                 push    0FFFFFFEBh      ; nIndex
0000:0040478D                 mov     eax, ds:dword_4060D4
0000:00404792                 push    eax             ; hWnd
0000:00404793                 call    SetWindowLongA
0000:00404798                 mov     eax, esi        ; hWnd
0000:0040479A                 call    sub_404358
```
还好，是那个窗口特效……

```
0000:0040479F                 push    0               ; lpTimerFunc
0000:004047A1                 push    3Ch             ; uElapse
0000:004047A3                 push    0A8h            ; nIDEvent
0000:004047A8                 push    esi             ; hWnd
0000:004047A9                 call    SetTimer
0000:004047AE                 mov     ebx, 1
0000:004047B3                 jmp     loc_404927
```

`sub_403F94` 实际是统计 `$0A` 数目的。

## `sub_403F94` 实现
```
0000:00403F94 sub_403F94      proc near
0000:00403F94
0000:00403F94 var_4           = dword ptr -4
0000:00403F94
0000:00403F94                 push    ebp
0000:00403F95                 mov     ebp, esp
0000:00403F97                 push    ecx
0000:00403F98                 push    ebx
0000:00403F99                 mov     [ebp+var_4], eax
0000:00403F9C                 mov     eax, [ebp+var_4]
```
`var_4` 里是地址

```
0000:00403F9F                 call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)
```
增加引用计数

```
0000:00403FA4                 xor     eax, eax
0000:00403FA6                 push    ebp
0000:00403FA7                 push    offset loc_403FEB
0000:00403FAC                 push    dword ptr fs:[eax]
0000:00403FAF                 mov     fs:[eax], esp
```
设置 `SEH`

```
0000:00403FB2                 mov     bx, 1
0000:00403FB6                 mov     eax, [ebp+var_4]
```
仍然是地址

```
0000:00403FB9                 call    @System@_16823  ; System::_16823
```
取字符串的净长度。通过查看 `dword_40497C` 附近的数据可以发现，`dword_40497C` 前面的 4 个字节，内容是 `$A8`。`dword_40497C` 的最后部分，有一个 `dword` 的 0，加上这个 0，总长度是 `$AC`。`$AC-$A8=4`，恰好是一个 `dword`。

```
0000:00403FBE                 dec     eax
0000:00403FBF                 test    eax, eax
0000:00403FC1                 jl      short loc_403FD5
```
判断空字符串

```
0000:00403FC3                 inc     eax
```
加回来

```
0000:00403FC4                 xor     edx, edx
```
此时 `bx=1`，`edx=0`

```
0000:00403FC6 loc_403FC6:
0000:00403FC6                 mov     ecx, [ebp+var_4]
```
字符串的开头地址

```
0000:00403FC9                 cmp     byte ptr [ecx+edx-1], 0Ah
```
应该是用 `edx` 做索引，但是开始时 `edx=0`，`ecx+edw-1=(Var_4)-1`，这样就到了长度字节。

```
0000:00403FCE                 jnz     short loc_403FD1
0000:00403FD0                 inc     ebx
0000:00403FD1 loc_403FD1:
0000:00403FD1                 inc     edx
0000:00403FD2                 dec     eax
0000:00403FD3                 jnz     short loc_403FC6
0000:00403FD5 loc_403FD5:
```
`ebx` 里应该是换行的数目 + 1

```
0000:00403FD5                 xor     eax, eax
0000:00403FD7                 pop     edx
0000:00403FD8                 pop     ecx
0000:00403FD9                 pop     ecx
0000:00403FDA                 mov     fs:[eax], edx
```
恢复 `SEH`

```
0000:00403FDD                 push    offset loc_403FF2
```
实际和 `retn` 配套
不得不说，虽然开始看到 `403FC6` 的循环就猜到这个是计算 `$0A` 的，但是因为是分析模板，开始时没想到里面居然会有 `push/ret` 这个变形的 `jmp`。怀疑这个东西不是 `Delphi` 生成的。但是看 `403FE2` 又发现不像是手写的，真是!@#$

```
0000:00403FE2 loc_403FE2:
0000:00403FE2                 lea     eax, [ebp+var_4]
0000:00403FE5                 call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)
```
这里没有改变 `eax`，`eax` 还是那个地址
清除字符串?

```
0000:00403FEA                 retn
```
```
0000:00402EE8 @System@_16823  proc near
0000:00402EE8                 test    eax, eax
0000:00402EEA                 jz      short locret_402EEF
0000:00402EEC                 mov     eax, [eax-4]
0000:00402EEF locret_402EEF:
0000:00402EEF                 retn
0000:00402EEF @System@_16823  endp
```
```
0000:00403FF2 loc_403FF2:
0000:00403FF2                 mov     eax, ebx
```
返回值就是 `ebx`

```
0000:00403FF4                 pop     ebx
0000:00403FF5                 pop     ecx
0000:00403FF6                 pop     ebp
0000:00403FF7                 retn
0000:00403FF7 sub_403F94      endp
```

## `CreateWindowEx` 外套
```
0000:00403C98 sub_403C98      proc near
0000:00403C98
0000:00403C98 lpParam         = dword ptr  8
0000:00403C98 hInstance       = dword ptr  0Ch
0000:00403C98 hMenu           = dword ptr  10h
0000:00403C98 hWndParent      = dword ptr  14h
0000:00403C98 nHeight         = dword ptr  18h
0000:00403C98 nWidth          = dword ptr  1Ch
0000:00403C98 Y               = dword ptr  20h
0000:00403C98 X               = dword ptr  24h
0000:00403C98
0000:00403C98                 push    ebp
0000:00403C99                 mov     ebp, esp
0000:00403C9B                 push    ebx
0000:00403C9C                 mov     ebx, [ebp+lpParam]
0000:00403C9F                 push    ebx             ; lpParam
0000:00403CA0                 mov     ebx, [ebp+hInstance]
0000:00403CA3                 push    ebx             ; hInstance
0000:00403CA4                 mov     ebx, [ebp+hMenu]
0000:00403CA7                 push    ebx             ; hMenu
0000:00403CA8                 mov     ebx, [ebp+hWndParent]
0000:00403CAB                 push    ebx             ; hWndParent
0000:00403CAC                 mov     ebx, [ebp+nHeight]
0000:00403CAF                 push    ebx             ; nHeight
0000:00403CB0                 mov     ebx, [ebp+nWidth]
0000:00403CB3                 push    ebx             ; nWidth
0000:00403CB4                 mov     ebx, [ebp+Y]
0000:00403CB7                 push    ebx             ; Y
0000:00403CB8                 mov     ebx, [ebp+X]
0000:00403CBB                 push    ebx             ; X
0000:00403CBC                 push    ecx             ; dwStyle
0000:00403CBD                 push    edx             ; lpWindowName
0000:00403CBE                 push    eax             ; lpClassName
```
