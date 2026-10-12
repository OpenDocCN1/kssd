
### “潜行者”原理与影响

“潜行者”以感染 `Windows` 系统文件 `Dsound.dll`、`Ddraw.dll` 作为跳板，绕过杀毒软件及网游保护系统。一旦受感染的系统文件被网络游戏加载到内存，“潜行者”便会加载各种流行网游盗号木马（特征是扩展名为 `drv`），盗取《天龙八部》、《剑网三》、《QQ 地下城勇士》等流行网游的账号。同时，也会使游戏过程中频繁卡机。

多数杀毒软件对“潜行者”病毒无法查杀，或查杀后造成系统找不到。

现在有很多杀毒软件，都可以直接修复被替换的 `dsound.dll` 了，这里我就不多说了，我还是进入主题吧!

下面我们重点分析一下被替换的 `dsound.dll` 与原来的 `dsound.dll` 有什么不同吧~~

先用 `PEID` 分别查看两个 `DLL` 有什么不同吧~~

![](img/4a7a1c145af5715da3489faaee748392.jpg)

很多明显，被感染的 `dsound.dll` 多了一个节 `.text8`，正是我们前面所分析的，我们就用 `UE` 分别打开这两个 `DLL` 来比较有哪些不同吧

（由于 `UE` 不能双开，这个搞起来很不方便，不能对照分析，于是我分别用两个软件 `UE` 和 `C32ASM`）来对照起来分析吧

要想替换系统中的某个 `DLL`，一定不能把原文件破坏了，只能在其基础上进行相应修改!

我们就来复习一下 `PE` 结构吧~~先来看看 `PE` 区块数，如图原 `DLL` 只有四个区块，感染版 `DLL` 有五个

![](img/9cba29eed56948b4426b05964ba0de14.jpg)

![](img/24f40bbe8d3c95689fe2f27484c18871.jpg)

然后我们再用 `PEID` 分析一下，如图所示：

![](img/9a7361f3e14f342c9159573ffded0f5e.jpg)

左边的是感染了的，右边的是没有被感染的

从上面很明显有，三个地方不同：

(1) `DLL` 入口点：

原版 `DLL` 入口点：`00001788`----->在 `.text` 节中

感染 `DLL` 入口点：`0005C000`----->在 `.text8` 节中

(2) 节数目：

原版 `DLL`: `0004`

感染 `DLL`: `0005`

(3) 镜像大小

原版 `DLL` 镜像大小: `0005C000`

感染 `DLL` 镜像大小: `0005D000`

可以看出被替换的 `DLL` 入口被改成了我们增加的代码的入口处，先执行我们的某些功能，然后在把控制权给真实的系统 `DLL`

在用上面两个工具进行对比分析时，发现在文件偏移 `00000280h` 处

原版的数据是这样的

![](img/589dc1ca36a2649c4455d0700065b8dc.jpg)

感染版的数据变成了

![](img/f229d12027187dd5c0acced78a2bfb27.jpg)

根据上面的图中我们可以看到，感染片中的 `DLL` 最后文件偏移 `59C00` 处加入了 `800` 大小的代码~~~

我们就来重点看下，这段代码在做什么吧~~~

现在我们用 `IDA` 打开被感染的 `DLL`~~~

`DLL` 入口代码被改成了如下内容:

```
.text8:73ECC000  ; BOOL __stdcall DllEntryPoint(HINSTANCE hinstDLL,DWORD fdwReason,LPVOID lpReserved)
.text8:73ECC000                 public DllEntryPoint
.text8:73ECC000 DllEntryPoint   proc near
.text8:73ECC000
.text8:73ECC000 arg_4            dword ptr  8
.text8:73ECC000
.text8:73ECC000 ; FUNCTION CHUNK AT .text8:73ECC13C SIZE 00000011 BYTES
.text8:73ECC000
.text8:73ECC000                 cmp     [esp+arg_4], 1
.text8:73ECC005                 jnz     short loc_73ECC014
.text8:73ECC005
.text8:73ECC007                 nop
.text8:73ECC008                 pusha
.text8:73ECC009                 nop
.text8:73ECC00A                 add     edx, 0
.text8:73ECC00D                 call    sub_73ECC01E
.text8:73ECC00D
.text8:73ECC012                 nop
.text8:73ECC013                 popa
.text8:73ECC013
.text8:73ECC014
.text8:73ECC014 loc_73ECC014:                           ; CODE XREF: DllEntryPoint+5j
.text8:73ECC014                 nop
.text8:73ECC015                 add     ebp, 0
.text8:73ECC018                 jmp     loc_73ECC13C
.text8:73ECC018
.text8:73ECC018 DllEntryPoint   endp
```

很明显入口地址变成 `73E70000+5C000 = 73ECC000`，而原来的入口地址应该是 `73E70000+1788 = 73E71788` 代码如下:

```
.text:73E71788 ; BOOL __stdcall DllEntryPoint(HINSTANCE hinstDLL,DWORD fdwReason,LPVOID lpReserved)
.text:73E71788                 public DllEntryPoint
.text:73E71788 DllEntryPoint   proc near
.text:73E71788
.text:73E71788 hinstDLL         dword ptr  8
.text:73E71788 fdwReason        dword ptr  0Ch
.text:73E71788 lpReserved       dword ptr  10h
.text:73E71788
.text:73E71788 ; FUNCTION CHUNK AT .text:73E82EC3 SIZE 00000054 BYTES
.text:73E71788 ; FUNCTION CHUNK AT .text:73E8627E SIZE 00000036 BYTES
.text:73E71788
.text:73E71788                 mov     edi, edi
.text:73E7178A                 push    ebp
.text:73E7178B                 mov     ebp, esp
.text:73E7178D                 push    ebx
.text:73E7178E                 mov     ebx, [ebp+hinstDLL]
.text:73E71791                 push    esi
.text:73E71792                 mov     esi, [ebp+fdwReason]
.text:73E71795                 test    esi, esi
.text:73E71797                 push    edi
.text:73E71798                 mov     edi, [ebp+lpReserved]
.text:73E7179B                 jz      loc_73E82EC3
.text:73E7179B
.text:73E717A1                 cmp     esi, 1
.text:73E717A4                 jnz     loc_73E82ECF
.text:73E717A4
.text:73E717AA
.text:73E717AA loc_73E717AA:                           ; CODE XREF: DllEntryPoint+1174Aj
.text:73E717AA                 mov     eax, __pRawDllMain
.text:73E717AF                 test    eax, eax
.text:73E717B1                 jnz     loc_73E8627E
.text:73E717B1
.text:73E717B7
.text:73E717B7 loc_73E717B7:                           ; CODE XREF: DllEntryPoint+14AFDj
.text:73E717B7                 push    edi
.text:73E717B8                 push    esi
.text:73E717B9                 push    ebx
.text:73E717BA                 call    _CRT_INIT(x,x,x)
.text:73E717BA
.text:73E717BF                 test    eax, eax
.text:73E717BF
.text:73E717C1
.text:73E717C1 loc_73E717C1:                           ; CODE XREF: DllEntryPoint+11742j
.text:73E717C1                 jz      loc_73E8628B
.text:73E717C1
.text:73E717C7
.text:73E717C7 loc_73E717C7:                           ; CODE XREF: DllEntryPoint+11750j
.text:73E717C7                 push    edi
.text:73E717C8                 push    esi
.text:73E717C9                 push    ebx
.text:73E717CA                 call    DllMain(x,x,x)
.text:73E717CA
.text:73E717CF                 cmp     esi, 1
.text:73E717D2                 mov     [ebp+fdwReason], eax
.text:73E717D5                 jnz     loc_73E82EE7
.text:73E717D5
.text:73E717DB                 test    eax, eax
.text:73E717DD                 jz      loc_73E86292
.text:73E717DD
.text:73E717E3
.text:73E717E3 loc_73E717E3:                           ; CODE XREF: DllEntryPoint+1175Aj
.text:73E717E3                                        ; DllEntryPoint+11777j
.text:73E717E3                                        ; DllEntryPoint+11784j
.text:73E717E3                                        ; DllEntryPoint+14B27j
.text:73E717E3                 mov     eax, [ebp+fdwReason]
.text:73E717E3
.text:73E717E6
.text:73E717E6 loc_73E717E6:                           ; CODE XREF: DllEntryPoint+14B05j
.text:73E717E6                 pop     edi
.text:73E717E7                 pop     esi
.text:73E717E8                 pop     ebx
.text:73E717E9                 pop     ebp
.text:73E717EA                 retn    0Ch
.text:73E717EA
.text:73E717EA DllEntryPoint   endp
```

很明显两个入口地址的代码完全不同的，我们来重点分析一下，被感染的入口地址入码吧!

```
.text8:73ECC000                 cmp     [esp+arg_4], 1       ;比较参数是否为 1
.text8:73ECC005                 jnz     short loc_73ECC014    ;如果不为 1，则跳转到 loc_73ECC014 处----
.text8:73ECC005
.text8:73ECC007                 nop
.text8:73ECC008                 pusha
.text8:73ECC009                 nop
.text8:73ECC00A                 add     edx, 0
.text8:73ECC00D                 call    sub_73ECC01E    ----->重要的 CALL
.text8:73ECC00D
.text8:73ECC012                 nop
.text8:73ECC013                 popa
-----------------------------------------------------------------------------------
.text8:73ECC014 loc_73ECC014:                           ; CODE XREF: DllEntryPoint+5j
.text8:73ECC014                 nop
.text8:73ECC015                 add     ebp, 0
.text8:73ECC018                 jmp     loc_73ECC13C  -------->直接 jmp loc_73ECC13C
.text8:73ECC018
.text8:73ECC018 DllEntryPoint   endp
-----------------------------------------------------------------------------------
```

我们来看看 `loc_73ECC13C` 处的代码吧

```
.text8:73ECC13C loc_73ECC13C:                           ; CODE XREF: DllEntryPoint+18j
.text8:73ECC13C                 nop
.text8:73ECC13D                 nop
.text8:73ECC13E                 call    sub_73ECC14D
.text8:73ECC13E
.text8:73ECC143                 push    esi
.text8:73ECC144                 xor     esi, esi
.text8:73ECC146                 pop     esi
.text8:73ECC147                 mov     eax, 0
.text8:73ECC14C                 nop
```
