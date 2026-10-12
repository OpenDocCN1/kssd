
# SoftDefender

> 原文：[`www.pediy.com/kssd/pediy06/pediy6358.htm`](https://www.pediy.com/kssd/pediy06/pediy6358.htm)

呵呵，昨天朋友过生日，玩的尽兴，今天早上 6 点才回来 :P ，休息好了，把关于 `SoftDefender` 主程序脱壳手记附上

借 `Fly` 兄的格式用用，`Fly` 兄的破文生成器能不能给我一份？？

PS: 这个壳实际是国产软件，我最近才知道的……

`SoftDefender` V1.12 DEMO 主程序脱壳

下载地址： `http://www.softdefender.com/setup.exe`

应用平台： Win 98/ME/NT/2000/XP/2003

软件大小： 600 K

**【软件简介】**：`Soft Defender` is a powerful tool securing products against crackers. It was designed with ease of use and high speed as a priority without sacrificing high levels of protection. With `Soft Defender`, your application can have anticrack, antidebug, antidisassemble, antidump, anti-apihook, file integrity checking functions in seconds. It requires no source code editing or your registration algorithm changing. In addition, `Soft Defender` is a perfect `exe` file compressor, which can reduce the file size of 32-bit Windows programs by as much as 50%.

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：WinXP、`Ollydbg`1.09、`PEiD`、`LordPE`、`ImportREC` 1.6.0 Final

---

## 脱壳过程

废话不说，直接进入主题，我这里的 A 和 B 都是必须的哟

★★★★★★★★★★★★

### A、对于父进程的跟踪分析

★★★★★★★★★★★★

用 `OD` 载入 `SoftDefender.exe`，隐藏好 `IsDebuggerPresent` 标记，断在入口点：

```
<ModuleEn> /74 07          je short SoftDefe.004D1009      ;这实际就是个 Jmp 嘛～，这个壳里大量用到这样的变形 Jmp
004D1002   |75 05          jnz short SoftDefe.004D1009     ;
004D1004   |1932           sbb dword ptr ds:\[edx\], esi
004D1006   |67:E8 E8741F75  call 756C84F4                   ; Superfluous prefix
004D100C    1D E8683944    sbb eax, 443968E8
```

直接下断 `bp ZwQueryInformationProcess`（嘿嘿，这个特殊，壳不检查前 5 个字节），`F9`，`OD` 断下，这个我们不需要，再 `F9`，`OD` 又断下

```
ZwQueryIn>/$  B8 9A000000   mov eax, 9A                 ;断在这
77F7603A  |.  BA 0003FE7F   mov edx, 7FFE0300
77F7603F  |.  FFD2          call edx
77F76041  .  C2 1400       retn 14
```

执行到返回,再 `F8` 到这

```
004D8FAE    85C0            test eax, eax          ;返回到这,F8
004D8FB0    75 08          jnz short SoftDefe.004D8FBA    ;这里一定要跳,在 004D8FBA 用 New Origin here，或者用 Fly 兄的修改标志寄存器也可以，最好的方法自然是前面一步修改 eax 的值为非 0
004D8FB2    8B4424 08      mov eax, dword ptr ss:\[esp+8\]
004D8FB6    85C0            test eax, eax
004D8FB8    75 0E          jnz short SoftDefe.004D8FC8
004D8FBA    E8 4BFFFFFF    call SoftDefe.004D8F0A   ;Jmp to here
004D8FBF    85C0            test eax, eax
004D8FC1    75 05          jnz short SoftDefe.004D8FC8
```

现在我们在命令栏里输入 `follow GetTickCount`，反汇编窗口显示

```
GetTickCo>  BA 0000FE7F     mov edx, 7FFE0000             
77E5A2A0    8B02          mov eax, dword ptr ds:\[edx\]
77E5A2A2    F762 04       mul dword ptr ds:\[edx+4\]
77E5A2A5    0FACD0 18     shrd eax, edx, 18
77E5A2A9    C3            retn          ;这里按 F4，当然你也可以在这里做点小手脚 :twisted:
```

`F8` 返回

```
004D8171    894424 0C     mov dword ptr ss:\[esp+C\], eax
004D8175    8B4424 10     mov eax, dword ptr ss:\[esp+10\]
004D8179    83F8 04       cmp eax, 4
```

在`[esp+C]`处下内存访问断点， 注意这里开始有分岔路咯～`F9` `Go`，OK，`OD` 断在这

```
004D6F27    8A0411          mov al, byte ptr ds:\[ecx+edx\]
004D6F2A    8801            mov byte ptr ds:\[ecx\], al
004D6F2C    41              inc ecx
004D6F2D    4E              dec esi
004D6F2E  ^ 75 F7           jnz short SoftDefe.004D6F27
004D6F30    8BC7            mov eax, edi
```

此时去掉内存断点。

呵呵，看到这个表示我们是在调试父进程，下面父进程要做的事是建立临时文件，写入前面 `GetTickCount` 所得值的加密值，再 `CreateProcessA` 建立子进程，随后自己退出，`CreateProcessA` 我们没办法跟进去，不过主导思想是既然 `CreateProcessA` 是执行同一个文件，那么保存好现在的现场，用 `OD` 再次载入 `SoftDefender` 模拟子进程，OK，我们可以用 `follow CloseHandle`

```
CloseHandl>  64:A1 18000000  mov eax, dword ptr fs:\[18\]   ;No BreakPoint here :wink:
77E5A6F6     8B48 30       mov ecx, dword ptr ds:\[eax+30\]
77E5A6F9     8B4424 04     mov eax, dword ptr ss:\[esp+4\]
77E5A6FD     83F8 F4       cmp eax, -0C
77E5A700   ^ 0F84 7E93FFFF  je kernel32.77E53A84
```

只要你不在第一句下断点就没问题,`F9` `Go`,断在你下断的地方，这个 `CloseHandle` 实际是关闭临时文件的句柄，如果父进程始终不 `CloseHandle` 的话系统是不会取消这个内核的，这样也方便我们后面对子进程的调试嘛 :P

到此，父进程的工作已经完成了，我们留着这个 `OD`

★★★★★★★★★★★★★

### B、对于子进程的跟踪分析

★★★★★★★★★★★★★

新开一个 `OD`，再次载入 `SoftDefender.exe`

前面的过程和父进程一样，绕过 `ZwQueryInformationProcess` 的 Anti，到前面我说的分岔路口那，当你再`[esp+C]`处下内存访问断点后，`F9` `Go`，稍等一会，`OD` 会断在这

```
004D81E2    8B4C24 0C     mov ecx, dword ptr ss:\[esp+C\]   ;Stop here
004D81E6    8B5424 08     mov edx, dword ptr ss:\[esp+8\]
004D81EA    2BCA          sub ecx, edx
004D81EC    81F9 401F0000  cmp ecx, 1F40
004D81F2    72 66         jb short SoftDefe.004D825A   ;一定要跳！
```

呵呵，这个父子进程的时间差，过长的话子进程就执行父进程的作用，写入新值，`CreateProcessA` 另一个新的进程…………

```
004D81F4    83FF 03       cmp edi, 3
004D81F7    74 25         je short SoftDefe.004D821E
004D81F9    6A 00         push 0
```

跳到这里，怎么修改程序运行轨道我就不再说了

```
004D825A    5F            pop edi
004D825B    B8 08000000   mov eax, 8
004D8260    5E            pop esi
004D8261    81C4 40010000  add esp, 140
004D8267    C3            retn
```

OK,`F9` 吧，程序运行了，我们成功模拟了子进程，兴奋不？？

我们千辛万苦绕过那些 Anti，这么一个 `F9` 就全完了？？？？`OEP`，`Import` 都还没搞定那…………

是的，全完了，不过我们也可以获得一个重要的情况，外壳会自己清除关键代码，向上翻，会发现有很长一段都是

```
004D137B    0000          add byte ptr ds:\[eax\], al
```

其实就是 00，被清除了的代码，这里是我们找 `OEP` 的关键

重复子进程前面的过程，最后不要 `F9` 了，而是在其中某一个地方，比如

`004D1487` 处下一个硬写断点（这个位置我没有试，我每次都是随机选的）

`F9` `Go`，直到断在下面这个位置为止

```
004D5E78    F3:AA         rep stos byte ptr es:\[edi\]
004D5E7A    C3            retn
```

`F8` 执行到 `retn`，下面就是要返回到 `OEP` 了，如果你要是一高兴用 `F8` 按下去，那将是一个恶梦，因为我们这个时候看看栈堆

```
0012EB90  0044D3DD  SoftDefe.0044D3DD
0012EB94  0044D3A3  SoftDefe.0044D3A3
0012EB98  0044D399  SoftDefe.0044D399
0012EB9C  0044D374  RETURN to SoftDefe.0044D374 from SoftDefe.00447CF1
0012EBA0  0044D31C  RETURN to SoftDefe.0044D31C from SoftDefe.00403480
…… …… …
0012FFB4  00429790  SoftDefe.00429790
0012FFB8  00429760  SoftDefe.00429760
0012FFBC  004296CA  SoftDefe.004296CA
0012FFC0  0042966D  SoftDefe.0042966D    ;这里才是目的地，飞向光明之颠？？ :lol:
0012FFC4  004D5748  SoftDefe.004D5748
0012FFC8  77D64F70  user32.77D64F70
```

OK，`G 0042966D` 吧

```
0042966D    FF15 688F1400   call dword ptr ds:\[148F68\]   ;注意这个 call，进去看看
00429673    64:A1 00000000  mov eax, dword ptr fs:\[0\]
00429679    50              push eax
0042967A    64:8925 0000000>mov dword ptr fs:\[0\], esp
00429681    83EC 58         sub esp, 58
```

进入 `call [148F68]`后

```
00169BB0    58              pop eax\---------------------------------------
00169BB1    55              push ebp
00169BB2    8BEC            mov ebp, esp
00169BB4    6A FF           push -1               Stolen Byte!!!
00169BB6    68 E02C4500     push 452CE0
00169BBB    68 48C74200     push 42C748
00169BC0    FFE0            jmp eax   -------------------------------------
```

而 `pop eax` 和 `jmp eax` 实际就是 `retn` 的变形，将 `Stolen Byte` 还原

```
00429664    55              push ebp
00429665    8BEC            mov ebp, esp
00429667    6A FF           push -1
00429669    68 E02C4500     push 00452CE0
0042966E    68 48C74200     push 0042C748
00429673    64:A1 00000000  mov eax, dword ptr fs:\[0\]
00429679    50              push eax
0042967A    64:8925 0000000>mov dword ptr fs:\[0\], esp
```

好了，这个时候可以用 `LordPE` 来 `Dump` 了

下面让我们看看输入表，基本都被修改了，除了几个只有一个引用函数的 `Dll` 没有被加密，我们的目标就是————修复它 :lol:

用 `ImportRec` 的 `trace Level 3` 还有 17 个没有修复，自己模拟跟踪一下就知道了 :wink:

我是通过修改程序里的一个内存值达到防止 `ImportTable` 被破坏的

这里就是恢复并修改 `ImportTable` 的代码了

```
004D88E7    52              push edx
004D88E8    50              push eax
004D88E9    57              push edi
004D88EA    E8 6DF5FFFF     call SoftDefe.004D7E5C
004D88EF    8B8424 40040000 mov eax, dword ptr ss:\[esp+440\]   ;我要改这个地址的值为 0 :o  :o
004D88F6    85C0            test eax, eax
```
