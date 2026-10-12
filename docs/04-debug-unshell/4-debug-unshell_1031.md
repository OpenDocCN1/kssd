
# 手动修复 SVKP1.11 加壳的 IAT

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-467.htm`](https://www.pediy.com/kssd/pediy09/pediy09-467.htm)
>
> 【破解作者】 `h_f22`  
> 【使用工具】 `FI4.0`、`PEID0.94`、`OllyDBG`、`ImportREC1.6f`  
> 【破解平台】 `WinXPSP2`  
> 【软件名称】 `Auth.exe`  
> 【软件简介】 一个游戏  
> 【加壳方式】 `SVKP1.11`  
> 【破解声明】 水平很菜，一点感受，希望对大家有用

本文应用一个简单的 `SVKP` 脱壳实例来说明手工修复 `IAT` 的几种方法和技巧，因为重点在手工修复 `IAT` 所以挑了一个简单的壳--`SVKP1.11`，这个壳难度不大，但是有几个非常典型的函数需要手工修复，虽然早就有了相关的脚本，但是对于真正的破解者来说，知其然，还要知其所以然。

这是一个游戏，由 `SVKP1.11` 加壳，可以看出 `svkp` 的比较早期的壳强度并不大。没有 `StolenCode`，`AntiDebug` 里面的 `rdtsc` 命令还是比较烦的，但他只影响单步，单步运行时注意一下即可，其余的用 `HIDEOD` 就对付了。解码代码段的函数用内存断点即可断下，很容易找到解码的地方。入口也不难，不过这次偷个懒用 `QuickUnpack` 的寻找 `OEP` 的插件找到：

`OEP:00414FA7`

知道入口后到达入口就很容易了。在 `00414FA7` 上下个内存访问的断点，忽略所有异常，`F9` 运行程序第一次断在一个异常处，继续会断在解码函数中，有兴趣可以自己看看其实并不复杂，完全可以自己写出来，但是要注意的是解码是分几个段的，不同段解码方式不一样，有几段花指令挺烦人。解码函数过去了就到入口了，看到熟悉的 `push ebp`：

```
00414FA7    55               push    ebp
00414FA8    8BEC             mov     ebp, esp
00414FAA    6A FF            push    -1
00414FAC    68 B0A84400      push    0044A8B0
00414FB1    68 40984100      push    00419840
00414FB6    64:A1 00000000   mov     eax, dword ptr fs:\[0\]
00414FBC    50               push    eax
00414FBD    64:8925 0000000> mov     dword ptr fs:\[0\], esp
......
```

在这里 `Dump` 出来，注意不用重建 `IAT`，重建也没用浪费地方，等会用 `ImportREC` 修复。

然后是 `IAT` 修复，这就是 `ImportREC` 的活了，基本上他的 `IAT` 乱序没什么难度。大部分都可以用 `ImportREC` 的 `Trace1` 找到，自己找的话也不难，但是有 6 个函数是找不到的：

```
0005D95C  0C9E1F48
0005D960  0C9E28DC
0005DA40  0C9E32B5
0005DA44  0C9EB57F
0005DA64  0C9E2F30
0005DA88  0C9EB1FA
```

根据以往经验 `SVKP` 特殊处理的函数不外乎以下六个：

```
ExitProcess
GetCommandLineA
GetModuleHandleA
GetVersionExA
GetCurrentProcess
GetVersion
```

关键是哪个是哪个这个不太好办，有几点经验：

1. 先用 `ImportREC` 把程序修复，剩下的几个函数看调用情况和顺序一个一个修复。
2. 如果是 `VC` 的程序，入口开始的头两个函数肯定是 `GetVersion` 和 `GetCommandLineA` 所以这两个好说。

比如本程序中：

```
00414FA7    55               push    ebp
00414FA8    8BEC             mov     ebp, esp
00414FAA    6A FF            push    -1
00414FAC    68 B0A84400      push    0044A8B0
00414FB1    68 40984100      push    00419840
00414FB6    64:A1 00000000   mov     eax, dword ptr fs:\[0\]
00414FBC    50               push    eax
00414FBD    64:8925 0000000> mov     dword ptr fs:\[0\], esp
00414FC4    83EC 58          sub     esp, 58
00414FC7    53               push    ebx
00414FC8    56               push    esi
00414FC9    57               push    edi
00414FCA    8965 E8          mov     dword ptr [ebp-18], esp
00414FCD    FF15 88DA4500    call    dword ptr [45DA88]      //这是第一个 API 调用，显然是 GetVersion
00414FD3    33D2             xor     edx, edx
00414FD5    8AD4             mov     dl, ah
00414FD7    8915 EC9B4500    mov     dword ptr [459BEC], edx
00414FDD    8BC8             mov     ecx, eax
00414FDF    81E1 FF000000    and     ecx, 0FF
00414FE5    890D E89B4500    mov     dword ptr [459BE8], ecx
00414FEB    C1E1 08          shl     ecx, 8
00414FEE    03CA             add     ecx, edx
00414FF0    890D E49B4500    mov     dword ptr [459BE4], ecx
00414FF6    C1E8 10          shr     eax, 10
00414FF9    A3 E09B4500      mov     dword ptr [459BE0], eax
00414FFE    6A 01            push    1
00415000    E8 D4470000      call    004197D9
00415005    59               pop     ecx
00415006    85C0             test    eax, eax
00415008    75 08            jnz     short 00415012
0041500A    6A 1C            push    1C
0041500C    E8 C3000000      call    004150D4
00415011    59               pop     ecx
00415012    E8 653C0000      call    00418C7C
00415017    85C0             test    eax, eax
00415019    75 08            jnz     short 00415023
0041501B    6A 10            push    10
0041501D    E8 B2000000      call    004150D4
00415022    59               pop     ecx
00415023    33F6             xor     esi, esi
00415025    8975 FC          mov     dword ptr [ebp-4], esi
00415028    E8 7B440000      call    004194A8
0041502D    FF15 60D94500    call    dword ptr [45D960]   //这是第二个 API 调用 是 GetCommandLineA
00415033    A3 88B74500      mov     dword ptr [45B788], eax
......
```

3. `GetModuleHandleA` 一般出现在 `GetProcAddress` 前面，另外 `VC` 的程序在入口开始后找到 `GetStartupInfoA` 之后的一个函数一般也是 `GetModuleHandleA`，也是这个函数的最后一个 `API` 调用，他在这里当作主函数的一个参数。

情况 1：

```
0040D09F    0F95C0          setne   al
0040D0A2    E9 B7000000     jmp     0040D15E
0040D0A7    68 6C794400     push    0044796C                     ; ASCII "USER32"
0040D0AC    FF15 40DA4500   call    dword ptr [45DA40]    //看看这里的形式 显然是 GetModileHandleA
0040D0B2    8BF8            mov     edi, eax
0040D0B4    3BFB            cmp     edi, ebx
0040D0B6    74 76           je      short 0040D12E
0040D0B8    8B35 6CDB4500   mov     esi, dword ptr [45DB6C]
0040D0BE    68 58794400     push    00447958                     ; ASCII "GetSystemMetrics"
0040D0C3    57              push    edi
0040D0C4    FFD6            call    esi              //这里显然就是 GetProcAddress
0040D0C6    3BC3            cmp     eax, ebx
0040D0C8    A3 88954500     mov     dword ptr [459588], eax
0040D0CD    74 5F           je      short 0040D12E
0040D0CF    68 44794400     push    00447944                     ; ASCII "MonitorFromWindow"
0040D0D4    57              push    edi
0040D0D5    FFD6            call    esi
0040D0D7    3BC3            cmp     eax, ebx
0040D0D9    A3 8C954500     mov     dword ptr [45958C], eax
0040D0DE    74 4E           je      short 0040D12E
0040D0E0    68 34794400     push    00447934                     ; ASCII "MonitorFromRect"
....
```

情况 2：

```
00415051    8975 D0         mov     dword ptr [ebp-30], esi
00415054    8D45 A4         lea     eax, dword ptr [ebp-5C]
00415057    50              push    eax
00415058    FF15 68D94500   call    dword ptr [45D968]     //这个就是 GetStartupInfoA
0041505E    E8 B53F0000     call    00419018
00415063    8945 9C         mov     dword ptr [ebp-64], eax
00415066    F645 D0 01      test    byte ptr [ebp-30], 1
0041506A    74 06           je      short 00415072
0041506C    0FB745 D4       movzx   eax, word ptr [ebp-2C]
00415070    EB 03           jmp     short 00415075
00415072    6A 0A           push    0A
00415074    58              pop     eax
00415075    50              push    eax
00415076    FF75 9C         push    dword ptr [ebp-64]
00415079    56              push    esi
0041507A    56              push    esi
0041507B    FF15 40DA4500   call    dword ptr [45DA40]   //这个就是 GetModileHandleA 了
00415081    50              push    eax
00415082    E8 400C0100     call    00425CC7            //到这里程序是程序的主函数
00415087    8945 A0         mov     dword ptr [ebp-60], eax
0041508A    50              push    eax
0041508B    E8 0DFDFFFF     call    00414D9D
.....
```

4. 关于 `GetCurrentProcess` 这个函数不是很好确定，在这里提供一个参考，在 `VC` 中他往往与 `TerminateProcess` 合用，达到 `ExitProcess` 的目的，如下：

```
00414DCE   /75 11           jnz     short 00414DE1
00414DD0   |FF7424 08       push    dword ptr [esp+8]
00414DD4   |FF15 64DA4500   call    dword ptr [45DA64]   //这里是 GetCurrentProcess
00414DDA   |50              push    eax
00414DDB   |FF15 3CDB4500   call    dword ptr [45DB3C]  //这个是 TerminateProcess
00414DE1   \\837C24 0C 00    cmp     dword ptr [esp+C], 0
....
```

5. `GetVersionExA` 函数也不是很好确定，可以把它留在最后，如果只剩下两三个函数，可以挨个试直到不出错为止，当然出错了也不见得是因为他，所以有必要的话一点点跟出来。

6. `ExitProcess` 这个好说，不管它也没事，只不过程序退出的时候会出错，不影响大局。如果还剩一两个没确定可以挨个试一下，如果 `ExitProcess` 位置正确程序不会一运行就退出。

7. 当然还有一些其他的方法确定，一步步跟踪永远是最后的方法，这里介绍一个简单的方法，适用于任何函数：



# 逆向工程技巧：API 函数定位与修复

## 方法概述

在调用 API 函数的地方下断点，运行程序等待断下。断下后在相应的 DLL 的代码段中下内存断点，并在返回处下断（防止继续运行调用了其他 API）。按 `F9` 运行，正常的话会断在调用的 API 函数入口处，或者是在壳里。在壳里这时往往能看到要调用的 API 地址，不正常会断在返回处，一般是调用的下一行。当然，这招对于壳自己完全模拟系统 API 就失效，比如在这个壳中 `GetVersion` 等函数用这招是没用的。

### 第一步：下断点

代码：

```
0041BC87    68 D8694400     push    004469D8                      ; ASCII "KERNEL32"
0041BC8C    FF15 40DA4500   call    dword ptr [45DA40]   //不知道这个 API 是什么 在这里下断
0041BC92    85C0            test    eax, eax                  //返回处也下断
0041BC94    74 15           je      short 0041BCAB
0041BC96    68 F0AE4400     push    0044AEF0                      ; ASCII "IsProcessorFeaturePresent"
0041BC9B    50             push    eax
0041BC9C    FF15 6CDB4500   call    dword ptr [45DB6C]
0041BCA2    85C0            test    eax, eax
0041BCA4    74 05           je      short 0041BCAB
0041BCA6    6A 00           push    0
....
```

### 第二步：运行

会断在 `0041507B` 处。

### 第三步：下内存断点

知道是 `Kernel32.dll` 的函数，在他的代码段上下内存断点。就是这个段：

* 地址=`7C801000`
* 大小=`00082000` (`532480`)
* 属主=`kernel32` `7C800000`
* 区段=`.text`
* 包含=`代码,输入表,输出表`
* 类型=`Imag` `01001040`
* 访问=`RWE`
* 初始访问=`RWE`

### 第四步：运行并断在壳中

断在这里：

代码：

```
0C9E6248    AC              lods    byte ptr [esi] //在 ESI 中可以看到 API 的地址 GetModuleHandleA
0C9E6249    EB 02           jmp     short 0C9E624D
0C9E624B    0FE850 52       psubsb  mm2, qword ptr [eax+52]
0C9E624F    EB 02           jmp     short 0C9E6253
0C9E6251    CD20 EB020FE9   vxdcall E90F02EB
0C9E6257    0F31            rdtsc
....
```

这时基本可以断定是 `GetModuleHandleA`，但是如果不确定可以单步越过这行后继续下内存断点运行，注意每次断下都要下内存断点。这样反复几次，终于来到 `kernel32.dll` 的空间中：

代码：

```
7C80B6A1 >  8BFF            mov     edi, edi
7C80B6A3    55             push    ebp
7C80B6A4    8BEC            mov     ebp, esp
7C80B6A6    837D 08 00      cmp     dword ptr [ebp+8], 0
7C80B6AA    74 18           je      short 7C80B6C4
7C80B6AC    FF75 08         push    dword ptr [ebp+8]
7C80B6AF    E8 C0290000     call    7C80E074
7C80B6B4    85C0            test    eax, eax
....
```

这时看看 `EIP`，就是 `GetModuleHandleA`，可以确定 `0041BC8C` 处的 API 调用就是 `GetModuleHandleA`。即 `IAT` 中地址 `045DA40` 处应该是 `GetModuleHandleA` 的地址 `7C80B6A1`。手工写上或者用 `ImportREC` 指定都可以。到此我们便手工修复了一个函数。

## 通用方法与注意事项

8. 上面的方法是一个通用的方法，虽然能够确定大部分函数，但是对于没被调用的函数就无能为力。这时候我们可以在将 `EIP` 指定在这样的未知函数上，然后下 `DLL` 的内存断点和返回出的断点，原理相同，也可以得到函数是哪个 `API`。有人可能会问这样不会出错吗？实际上大部分情况下都不会，除非壳检测压入堆栈中的参数，但这一般很难检测。因为壳在进行这样的 `API` 保护的时候并不知道程序什么时候会调用 `API`，为什么会调用 `API`。所以壳进行 `API` 保护的时候与程序是无关的，它不关心你在什么时候调用，只关心你调用了后他能给你正确的 `API` 入口或结果。所以无论怎样调用，都是一样会把程序正确转入到 `API` 中，有的时候有点变化，壳可能完全模拟了 `API` 或者模拟一部分代码。至于壳是怎么做到的这一点，有兴趣的可以自己跟踪一下，方法就那么几种。

9. 最后无论用什么方法确定了一个 `API`，都要知道一个简单原则，就是 `import` 表中一般不会有重复出现的 `API` 地址，如果你确定一个函数后发现 `IAT` 中已经有了这个函数，那肯定有问题。另外熟悉各种编译器的翻译习惯是非常重要的，有丰富的经验可让你少走很多弯路。不过无论怎样手工修复 `IAT` 都是件非常痛苦的事情。

经过以上几步后一般就可以确定这些未知的 `API` 函数了，如下：

代码：

```
0005D95C  0C9E1F48  ExitProcess
0005D960  0C9E28DC  GetCommandLineA
0005DA40  0C9E32B5  GetModuleHandleA
0005DA44  0C9EB57F  GetVersionExA   
0005DA64  0C9E2F30  GetCurrentProcess   
0005DA88  0C9EB1FA  GetVersion
```

用 `ImportREC` 修复运行... `OK`。

## 总结

本文的这些方法都是经验，实际中还要靠读者自己灵活运用。单步跟踪是解决一切问题的方法，但不是最优的方法，这里提供的几点经验仅供大家参考，希望能够对你有所帮助。由于作者水平有限，难免有疏漏和错误，还望大家指教。

`4dd466d0e1e0cc123afbe504da2462ca`
