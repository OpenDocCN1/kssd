
# 期待 OD 更新

> 原文：[`www.pediy.com/kssd/pediy10/67737.html`](https://www.pediy.com/kssd/pediy10/67737.html)

```
CC          int3  
9D          popfd  
```

好像先是 `TT`，然后就是 `VMP`![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

我是完全没搞懂，才出来咋呼的。不过好像是我系统环境好像有关系，没有 `K8 7` 和 `PhantOm` 的情况下似乎就没事，没搞懂![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

`q3` 的观察总是出人意料。

### OD 的 Bug?!

`int3`

`pushfd`

和

`int3`

`popfd`

一样的效果。

只要修改 `int3` 后面的 `popfd` 为其他值，`OD` 都能通过。老掉牙的技术又重新被用了。`SEH` 异常机制的运用而已。

原理：

在 `SEH` 异常处理中设置了硬件断点 `DR0=EIP+2`，并把 `EIP` 的值加 2，那么应该在 `int3, popfd` 后面的指令执行时会产生单步异常。但是 `OD` 遇到前面是 `popfd/pushfd` 时，`OD` 会自动在 `popfd` 后一指令处设置硬件断点，而 `VMP` 的 `seh` 异常处理会判断是否已经设置硬件断点，如果已经有硬件断点就不产生单步异常，所以不能正常执行。

`EIP` 停在 `9C ' 9D` 得另外处理，如下:

```
  00434E32   .  3D 9C000000   cmp     eax,9C  
  00434E37   .  74 0D         je      short 00434E46  
  00434E39   .  8B95 68FAFFFF mov     edx,[ebp-598]  
  00434E3F   .  3D 9D000000   cmp     eax,9D  
  00434E44   .  75 17         jnz     short 00434E5D  
```

若往下追，会得知 此时若按 `F4` 将需要 2 个硬断点. 若剩下的 `DR` 不足 2 个, 则用 `CC`

(所以若你 `DR` 用掉 3 个, 则不会有此困扰)

------------------------------------------------------------------

(下列... 请先清除所有硬断点)

```
  401000  9C    pushf          <- EIP  
  401001  90    nop  
  401002  90    nop  
  401003  90    nop  
  401004  90    nop            <- 在此按 F4 看看  
  401005  90    nop  
  401006  90    nop  
  401007  90    nop  
  401008  90    nop            <- 先在此按 F2 , 造成一个 CC, 以免待会跑飞  
```

`OD` 有个 `Bug`, 上列情况, 若你在 `401004` 按 `F4`, 将不会断下来

就算你有在途中强设硬断点也没用, 不会被断下来.

在 `401004` 按 `F4`, `OD` 会设一个 `Temp` 硬断点在 `401001` , 即亦 `pushf` 的下一条

以及一个 `one-shot` 硬断点在 `401004`

`Temp` 硬断点 有正常被断下来, 一个 `80000004`, 然后...所有的硬断点将都不会被激发

你会一直执行到遇到 `CC` 为止. 所以你的 `one-shot` 硬断点还留在那里.

(`Debug\Hardware breakpoints` 内可看到其存在着)

因为 `OD` 在调用 `SetThreadContext` 前, 会将 `DR7` 的低 10 个 bit 清为 0, 然后再

根据若 `DR0~3` 非零的话, 则将相对的 `DR7` 的相对 `bit` 设为 1, 使之 `Enable`.

可是这个步骤出了问题.

下列是一个 4 次的 `Loop` . 第一行 `cmp [edx],0` 是在看 `DRx` 有无设值.

这一行没问题. 有值就有值, 0 就是 0.

第三行的 `test byte ptr [4D5710], 1` 有问题

我发现 `[4D5710] = 01`

所以 4 次 `Loop` 结束后 `SI` 还是等于 `00`

`Loop` 的下一行 `0042EB17 mov [ebx+18],esi` 就是在设 `DR7`

```
0042EA28  |.  BE 00040000   |mov     esi,400                    //清掉 10 个 bit  
0042EA51  |> /833A 00       |/cmp     dword ptr [edx],0  
0042EA54  |. |0F84 B0000000 ||je      0042EB0A  
0042EA5A  |. |F605 10574D00>||test    byte ptr [4D5710],1  
0042EA61  |. |0F85 A3000000 ||jnz     0042EB0A  
0042EA67  |. |8BC8          ||mov     ecx,eax  
0042EA69  |. |03C9          ||add     ecx,ecx  
0042EA6B  |. |BF 01000000   ||mov     edi,1  
0042EA70  |. |D3E7          ||shl     edi,cl  
0042EA72  |. |0BF7          ||or      esi,edi  
0042EA74  |. |8B0A          ||mov     ecx,[edx]  
0042EA76  |. |83F9 07       ||cmp     ecx,7                   ;   Switch (cases 1..7)  
0042EA79  |. |77 62         ||ja      short 0042EADD  
0042EA7B  |. |FF248D 82EA42>||jmp     [ecx*4+42EA82]   
0042EA82  |. |DDEA4200      ||dd      ollydbg.0042EADD  
0042EA86  |. |A2EA4200      ||dd      ollydbg.0042EAA2  
0042EA8A  |. |B0EA4200      ||dd      ollydbg.0042EAB0  
0042EA8E  |. |BFEA4200      ||dd      ollydbg.0042EABF  
0042EA92  |. |CEEA4200      ||dd      ollydbg.0042EACE  
0042EA96  |. |A2EA4200      ||dd      ollydbg.0042EAA2  
0042EA9A  |. |A2EA4200      ||dd      ollydbg.0042EAA2  
0042EA9E  |. |A2EA4200      ||dd      ollydbg.0042EAA2  
0042EAA2  |> |33C9          ||xor     ecx,ecx  
0042EAA4  |. |894D F0       ||mov     [ebp-10],ecx  
0042EAA7  |. |C742 FC 01000>||mov     dword ptr [edx-4],1  
0042EAAE  |. |EB 32         ||jmp     short 0042EAE2  
0042EAB0  |> |C745 F0 03000>||mov     dword ptr [ebp-10],3  
0042EAB7  |. |81CE 00010000 ||or      esi,100  
0042EABD  |. |EB 23         ||jmp     short 0042EAE2  
0042EABF  |> |C745 F0 01000>||mov     dword ptr [ebp-10],1  
0042EAC6  |. |81CE 00010000 ||or      esi,100  
0042EACC  |. |EB 14         ||jmp     short 0042EAE2  
0042EACE  |> |C745 F0 02000>||mov     dword ptr [ebp-10],2  
0042EAD5  |. |81CE 00010000 ||or      esi,100  
0042EADB  |. |EB 05         ||jmp     short 0042EAE2  
0042EADD  |> |33C9          ||xor     ecx,ecx  
0042EADF  |. |894D F0       ||mov     [ebp-10],ecx  
0042EAE2  |> |8B4A FC       ||mov     ecx,[edx-4]  
0042EAE5  |. |83E9 02       ||sub     ecx,2  
0042EAE8  |. |74 07         ||je      short 0042EAF1  
0042EAEA  |. |83E9 02       ||sub     ecx,2  
0042EAED  |. |74 08         ||je      short 0042EAF7  
0042EAEF  |. |EB 0A         ||jmp     short 0042EAFB  
0042EAF1  |> |834D F0 04    ||or      dword ptr [ebp-10],4  
0042EAF5  |. |EB 04         ||jmp     short 0042EAFB  
0042EAF7  |> |834D F0 0C    ||or      dword ptr [ebp-10],0C  
0042EAFB  |> |8BC8          ||mov     ecx,eax  
0042EAFD  |. |C1E1 02       ||shl     ecx,2  
0042EB00  |. |83C1 10       ||add     ecx,10  
0042EB03  |. |8B7D F0       ||mov     edi,[ebp-10]  
0042EB06  |. |D3E7          ||shl     edi,cl  
0042EB08  |. |0BF7          ||or      esi,edi  
0042EB0A  |> |40            ||inc     eax  
0042EB0B  |. |83C2 1C       ||add     edx,1C  
0042EB0E  |. |83F8 04       ||cmp     eax,4  
0042EB11  |.^\\0F8C 3AFFFFFF |\\jl      0042EA51  
0042EB17  |.  8973 18       |mov     [ebx+18],esi  
0042EB1A  |>  53            |push    ebx                         ; /pContext = 04190020  
0042EB1B  |.  8B45 EC       |mov     eax,[ebp-14]                    ; |  
0042EB1E  |.  8B50 0C       |mov     edx,[eax+C]                     ; |  
0042EB21  |.  52            |push    edx                         ; |hThread = 004D8D78  
0042EB22  |.  E8 83060800   |call    <jmp.&KERNEL32.SetThreadContext>; \\SetThreadContext  
0042EB27  |.  85C0          |test    eax,eax  
0042EB29  |.  75 07         |jnz     short 0042EB32  
0042EB2B  |.  C745 F4 FFFFF>|mov     dword ptr [ebp-C],-1  
0042EB32  |>  FF45 FC       |inc     dword ptr [ebp-4]  
0042EB35  |.  8145 EC 6C060>|add     dword ptr [ebp-14],66C  
0042EB3C  |>  8B4D FC        mov     ecx,[ebp-4]  
0042EB3F  |.  3B0D 987D4D00 |cmp     ecx,[4D7D98]  
0042EB45  |.^ 0F8C BFFCFFFF \\jl      0042E80A  
```

各位说的壳我没遇过.

我认为 `popf` 前面的 `int 3` 主要是想让 `OD` 形成 `EIP` 停在 `popf` 的效果.

### 由 cc9d 引发的 pushfx/popfx 蝴蝶效应

引用: `最初由 sessiondiy 发布` ![查看帖子](http://bbs.pediy.com/showthread.php?p=476653#post476653) `EIP` 停在 `9C ' 9D` 得另外处理, 如下:

```
  00434E32   .  3D 9C000000   cmp     eax,9C  
  00434E37   .  74 0D         je      short 00434E46  
  00434E39   .  8B95 68FAFFFF mov     edx,[ebp-598]  
  00434E3F   .  3D 9D000000   cmp     eax,9D  
  00434E44   .  75 17         jnz     short 00434E5D  
```

...... 

学习了 另外有点错误就是 `"mov [ebx+18],esi 就是在设 DR6"` 这个应该是 `DR7`
