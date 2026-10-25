
# System Cleaner 4.89 Build 110

> 原文：[`www.pediy.com/kssd/pediy05/pediy50563.htm`](https://www.pediy.com/kssd/pediy05/pediy50563.htm)

用 `OllyDbg` 脱 `ASPR` 1.3x 的壳

作者：`LaBBa`

翻译：`gmh001`

软件：`System Cleaner` 4.89 Build 110

Url : [`www.allerasoft.com/products/systemcleaner/`](http://www.allerasoft.com/products/systemcleaner/)

## 前言

ＯＫ，贴上来！！这是我第三次写这篇破解文章，第一次正在写的时候机死机了，第二次停电了。

（呵呵，运气有点背啊…）

现在… 这篇文章讲述如何真正快速简单的脱掉 `ASPR` 1.3x 的壳，并且找出被抽掉的字节… 并且不使用 `SoftIce`、`IceDump` 和 `/tracex`！

用什么？

使用 `OllyDbg` ！

### 所需工具

1. `OllyDbg` 1.09b2 or newer
2. `ProcDump`(G-rom)/`Pe-edit`(y0da)
3. `imprec` 1.3 (MackT/UCF)(protools.cjb.net)
4. `HVIEW` / Hex Editor

## 脱壳步骤

1. 找到 `OEP`＋从内存中抓取程序
2. 找到抽掉的字节
3. 重建 `IAT`
4. 修正 `OEP`
5. 结束

### 第一步—找到 `OEP`＋从内存中抓取程序

1. 运行 `Olly` 并载入应用程序（如果有提示就按 `YES`！）

2. 现在按 `F9` 就会在这儿中断：

```
017E3414  3100           XOR DWORD PTR DS:[EAX],EAX  〈－我们在这儿中断！
017E3416  EB 01          JMP SHORT 017E3419
017E3418  68 648F0500    PUSH 58F64
```

为什么 `Olly` 会中断？我们不用设断点吗？！ 呵呵，`Olly` 在每次进入一个新模块时都会中断…我们将继续使用它！！

3. 按 `Shift+F9`，这样 `Olly` 会继续运行直到遇到一个新模块。

4. 按 `Shift+F9` 26次后我们将到达这里：

```
017E2D7A  3100           XOR DWORD PTR DS:[EAX],EAX〈－我们停在这里
017E2D7C  64:8F05 00000000 POP DWORD PTR FS:[0]    〈在这里设断点
017E2D83  58             POP EAX
017E2D84  833D 806D7E01 00 CMP DWORD PTR DS:[17E6D80],0
017E2D8B  74 14          JE SHORT 017E2DA1
017E2D8D  6A 0C          PUSH 0C
017E2D8F  B9 806D7E01    MOV ECX,17E6D80
017E2D94  8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
017E2D97  BA 04000000    MOV EDX,4
017E2D9C  E8 EFE0FFFF    CALL 017E0E90
017E2DA1  FF75 FC        PUSH DWORD PTR SS:[EBP-4]
017E2DA4  FF75 F8        PUSH DWORD PTR SS:[EBP-8]
017E2DA7  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
017E2DAA  8338 00        CMP DWORD PTR DS:[EAX],0
017E2DAD  74 02          JE SHORT 017E2DB1
017E2DAF  FF30           PUSH DWORD PTR DS:[EAX]
017E2DB1  FF75 F0        PUSH DWORD PTR SS:[EBP-10]
017E2DB4  FF75 EC        PUSH DWORD PTR SS:[EBP-14]
017E2DB7  C3             RETN
```

如果我们再按一次 `Shift+F9`，软件就运行了…千万别这么做！！

5. 现在我们向下移光标，到这里：

```
017E2D7C  64:8F05 00000000 POP DWORD PTR FS:[0]
```

按 `F2`（设断点）。 再按 `Shift+F9`，`Olly` 就会在断点中断。

6. 现在按 `F8` 单步追踪，执行 `RETN` 将会到达这里：

```
017F4EC8  E9 080A0000      JMP 017F58D5 〈－在这里
```

按 `F8` 执行这个跳转… 现在我们来到这里：

```
017F58D5  D3DE            RCR ESI,CL          〈－停在这里
017F58D7  B9 7D966271     MOV ECX,7162967D
017F58DC  81C1 38F10A23   ADD ECX,230AF138
017F58E2  D3EE            SHR ESI,CL
017F58E4  BA 9ECC7376     MOV EDX,7673CC9E
017F58E9  81EA C56EFFD4   SUB EDX,D4FF6EC5
017F58EF  81F2 B7104902   XOR EDX,24910B7
017F58F5  C1CA 94         ROR EDX,94
017F58F8  8BF2            MOV ESI,EDX
017F58FA  81EE 87D851D2   SUB ESI,D251D887
017F5900  C1CE B0         ROR ESI,0B0
017F5903  8BC6            MOV EAX,ESI
017F5905  C1C8 28         ROR EAX,28
017F5908  53              PUSH EBX
017F5909  5E              POP ESI
017F590A  81C5 974FAF73   ADD EBP,73AF4F97
017F5910  BA 048A9178     MOV EDX,78918A04
017F5915  F7D2            NOT EDX
017F5917  F7D2            NOT EDX
017F5919  81C2 FC756E87   ADD EDX,876E75FC
017F591F  8BDA            MOV EBX,EDX
017F5921  41              INC ECX -------------------------
017F5922  8BD9            MOV EBX,ECX                   |
017F5924  D1C3            ROL EBX,1                   | a Realy Long
017F5926  81F3 A38FD7AC   XOR EBX,ACD78FA3        |    Loop
017F592C  3BD8            CMP EBX,EAX                   |
017F592E  0F85 EDFFFFFF   JNZ 017F5921 --------------------
017F5934  8BC1            MOV EAX,ECX  〈〈－在这里设断点！！
```

好了，我们可以不用通过按 `F8` 来追踪整个循环！！我们在下面的指令上设一个断点：

```
017F5934  8BC1            MOV EAX,ECX
```

设置断点后（`F2`），按 `F9`（运行），将会在断点上停下来。

7. 为什么？？总是用 `F8` / `F7` 追踪？？！！ 好吧…不！！ `Olly` 里有一个很好的功能：调用命令行。 从菜单里选择：`Plugins->Command line->Command line` 现在我们将写一个条件追踪命令！ `YES` ! `Olly` 有一个追踪命令！它调用－`TC`－条件追踪 它将会在条件表达式为真时中断！ 这样…在文本框里写入：`TC EIP<900000` 按 `Enter` 键。 在窗口的右下会出现：“正在追踪” 我的机子慢，花了大约8分钟(350MHz) 在另一台较快的机子上花了2分钟多一点(900MHz `Celeron`)

8. 追踪完成后中断在这里：

```
0057EA5B   E8            DB E8
0057EA5C   00            DB 00
0057EA5D   8A            DB 8A
0057EA5E   E8            DB E8
0057EA5F   FF            DB FF
```

嘿！！ i Dont See Shit !!这是为什么？！ 噢，`Olly` 没有分析这段代码，实际上我们并没有真提停在 `OEP` 上！ `OEP=57EA5B`

9. 重点！！ 在 `OEP` 处看盾 `EAX` 值（`CPU` 窗口的右上面） `EAX=57E318` 记住它，以后会用到它的。

10. 打开 `ProcDump/Pe-Edit`，完全抓取进程。

### 第三步－找出抽掉的字节＝

找出抽掉的字节有三种方法：

1. `ASPR` 会执行抽掉的字节，象下面这样跳转到 `OEP`：

```
Stolen_Bytes_1
Stolen_Bytes_2
Stolen_Bytes_3
.....
.....
PUSH_THE_ADDRESS_OF_OEP
RET_TO_OEP
```

2. `ASPR` 首先保存抽掉的字节，然后从 `OEP` 处还原，再象下面这样跳转到 `OEP`：

```
Stolen_Bytes_1
Stolen_Bytes_2
Stolen_Bytes_3
PUSH EBX
PUSH ESI
PUSH EDI
......
......
......
REPZ STOSB 〈－还原字节
     POPFD
     POPAD
JMP_TO_OEP
```

3. 没有抽掉的字节时，只执行 `PopAD` 指令和跳转到 `OEP`。

那么我们怎么处理它呢？ 看看 `ASPR` 在跳转到 `OEP` 前的最后几步是什么。这需要查看跟踪日记。 太好了！`Olly` 可以使用"Run Trace"功能来得到一个 跟踪日记。 这样…选择菜单： `View->Run trace` 现在我们看见一个新窗口…跳转到最后一行的前一行，你会看到 `CPU` 窗口也变为同样的地址，象下面这样：

```
017F5779  F3:AA          REP STOS BYTE PTR ES:[EDI]
017F577B  9D             POPFD
017F577C  61             POPAD
017F577D  -E9 D992D8FE   JMP SYSTEMCL.0057EA5B  〈－OEP 的地址
```

我们用第二种方法对付它！！ 上面的内容我是在 `winXP` 下发现的，不是 `win98`:: 在 `CPU` 窗口里按 `Ctrl+S`（搜索命令的次序） 写入下面的：

```
PUSH EBX
PUSH ESI
PUSH EDI
```

你会发现：

```
01029227  0055 8B        ADD BYTE PTR SS:[EBP-75],DL
0102922A  EC             IN AL,DX                              ; I/O 命令
0102922B  83EC 54        SUB ESP,54
0102922E  53             PUSH EBX
0102922F  56             PUSH ESI
01029230  57             PUSH EDI
01029231  6A 11          PUSH 11
```

忽略 "00"，你会找到抽掉的字节：`55,8b,ec,83,ec,54` 记下它们，以后会有用的… 关闭 `OLLY`… 目前是不需要它啦…

### 第三步－重建 `IAT`＝

1. 首先运行软件，等候它被载入。
2. 运行 `Imprec`，从列表中选择进程。
3. 按" `IAT AutoSearch` "
4. 把大小从" `BC` "修改为 `1000`（ `BC` 太小了！！）
5. 按" `Get Imports` "
6. 按" `Show Invalid` "
7. 在显示无效的项目上右击鼠标，选择" `Trace level 1` "
8. 再次按" `Show Invalid` "…现在我们应该得到下面的东西：

```
（从保存的树中剪切）

FThunk: 0019E258     NbFunc: 00000400

1     0019E258     kernel32.dll     00D6     DeleteCriticalSection
1     0019E25C     kernel32.dll     0228     LeaveCriticalSection
................
省略－省略
................
1     0019E2A4     kernel32.dll     01D1     GetThreadLocale
1     0019E2A8     kernel32.dll     01B9     GetStartupInfoA
0     0019E2AC     ?                0000     017E0F2C            <-- good
0     0019E2B0     ?                0000     017E139C            <-- good
1     0019E2B4     kernel32.dll     018B     GetModuleFileNameA
1     0019E2B8     kernel32.dll     0183     GetLocaleInfoA
1     0019E2BC     kernel32.dll     0181     GetLastError
1     0019E2C0     kernel32.dll     0158     GetCurrentDirectoryA
0     0019E2C4     ?                0000     017E1408            <-- good
1     0019E2C8     kernel32.dll     0133     FreeLibrary
1     0019E2CC     kernel32.dll     011C     FindFirstFileA
..................
省略－省略
..................
```



`kernel32.dll` `CloseHandle()`

`user32.dll` `GetKeyboardType()`

`user32.dll` `LoadStringA()`

`user32.dll` `MessageBoxA()`

`user32.dll` `CharNextA()`

`advapi32.dll` `RegQueryvalueExA()`

`advapi32.dll` `RegOpenKeyExA()`

`advapi32.dll` `RegCloseKey()`

`oleaut32.dll` `SysFreeString()`

`oleaut32.dll` `SysReAllocStringLen()`

`oleaut32.dll` `SysAllocStringLen()`

`kernel32.dll` `TlsSetvalue()`

`kernel32.dll` `TlsGetvalue()`

`kernel32.dll` `GlobalAlloc()`

好了，我们继续重复这样做…

我们需要手工选择所有坏地址（不在压缩代码内），然后在它们上面右击鼠标，从菜单里选择 "Cut Thunk(s)"。

观察窗口在最下…那儿有好的 Thunk(s)…

现在我们需要修正这些好字节…你可能从 `fraviamb.cjb.net` 下载过一些插件

或者看我其他关于如何重建的文章（`New2Cracking.cjb.net` 或者 `Fraviamb.cjb.net`）

这是我修正的：

`0019E2AC-> 017E0F2C ->GetProcAddress()`

`0019E2B0-> 017E139C ->GetModuleHandleA()`

`0019E2C4-> 017E1408 ->GetCommandLineA()`

`0019E35C-> 017E139C ->GetModuleHandleA()`

`0019E428-> 017E1420 ->LockResource()`

`0019E47C-> 017E13C4 ->GetVersion()`

`0019E4AC-> 017E0F2C ->GetProcAddress()`

`0019E4B8-> 017E139C ->GetModuleHandleA()`

`0019E4FC-> 017E13F8 ->GetCurrentProcessId()`

`0019E500-> 017E13F0 ->GetCurrentProcess()`

`0019E50C-> 017E1430 ->FreeResource()`

按 "Fix Dump"，选择我们抓取的文件…

`imprec` 将会以在文件名前加 `"\_"` 的方式保存抓取的文件。

========================================

### 第四步－修正 OEP＝

========================================

1. 打开 `HVIEW` / Hex 编辑器，跳转到 OEP 的偏移地址：`57ea5b` 处，

   向后移 6 字节到 `57ea55` 处，用 `"00"` 替换写入 stolen bytes. 保存为新文件。

2. 打开 `ProcDump`/`Pe-Edit`，点击 “Pe-Edit” 按纽载入重建的新文件，修改入口点为：`57ea55` （因为抽掉的字节）

3. 如果你现在试图运行程序，它会死掉的！！

4. 打开 `Olly` 并载入我们的修正文件。

5. 停在 OEP：`57ea55` 处，`EAX` 寄存器的值是什么？还记得我告诉过你在抓取之前保存的 `EAX` 的值吗？它是 `EAX=57E318`，两个值的差别意味着有两个以上的命令： that is stolen:

   ```
   MOV EAX, 0157E318
   ```

   这条指令占用 5 个字节（你可自己在 `HVIEW` 里试一下）

   ```
   B818E35700      MOV EAX, 0157E318
   ```

   这意味着有 `6+5=11`（十进制）个抽掉的字节

   那么，真正的 OEP 是：

   ```
   11=B(hex)
   OLD_OEP-STOLEN_BYTES=57EA5B-B=57EA50
   OEP = 57EA50
   ```

   用 `HVIEW`/Hex 打开，跳转到新的 OEP，写入：

   ```
   55,8B,EC,83,EC,54,B8,18,E3,57,00
   ```

   用 `ProcDump`/`Pe-Edit` 修改入口点为：`57EA50`

   哈哈，这下程序能运行了！！！

   收工！！！

翻译后记：俺一直用 `Softice`，没用过 `OllyDbg`，所以有些内容不知道是否准确，再者水平有限，如有错误，请各位大虾指点！！

以下附原文：

## Unpacking ASPR 1.3x With OllyDbg

================================================

Writer: LaBBa

App: System Cleaner 4.89 Build 110

Url: [`www.allerasoft.com/products/systemcleaner/`](http://www.allerasoft.com/products/systemcleaner/)

### Pre to this tut

================

ok listen up ! this is the 3rd time i write this tut !!

the first time my computer crash while writing it

the second time there was a power fail in my house !!

now..

this tut is going to show us how to unpack real easy and fast ASPR 1.3x

and finding the stolen bytes real easy and fast too...

and all that without using `SoftIce` and `IceDump` and the `/tracex` !

how u ask ?

by using `OllyDbg` !

### Needed Tools

================

1. `OllyDbg` 1.09b2 or newer
2. `ProcDump`(G-rom)/`Pe-edit`(y0da)
3. `imprec` 1.3 (MackT/UCF)(`protools.cjb.net`)
4. `HVIEW` / Hex Editor

### Steps of Unpacking

=======================

1. Finding the OEP + Dumping
2. Finding the Stolen Bytes
3. Rebuilding the IAT
4. Fixing the OEP
5. The End.

====================================

#### Step 1 - Finding the OEP + Dumping =

====================================

1. load `Olly` and load the App to it (if it ask somthing just say YES!)

2. now press F9 and the prog will break here :

   ```
   017E3414  3100            XOR DWORD PTR DS:[EAX],EAX  <- we break Here!
   017E3416  EB 01           JMP SHORT 017E3419
   017E3418  68 648F0500     PUSH 58F64
   ```

   why `Olly` Break ? we didn't ask for break points ?!

   well `Olly` Break every time that it enter a new module ... AND WE ARE GOING TO USE IT !!

3. press Shift+F9 so `Olly` continue to run till new module.

4. after 26 times of Shift+F9 we will be here :

   ```
   017E2D7A  3100            XOR DWORD PTR DS:[EAX],EAX  <-we land here
   017E2D7C  64:8F05 00000000 POP DWORD PTR FS:[0]     <-here to set a Break Point
   017E2D83  58              POP EAX
   017E2D84  833D 806D7E01 00 CMP DWORD PTR DS:[17E6D80],0
   017E2D8B  74 14           JE SHORT 017E2DA1
   017E2D8D  6A 0C           PUSH 0C
   017E2D8F  B9 806D7E01     MOV ECX,17E6D80
   017E2D94  8D45 F8         LEA EAX,DWORD PTR SS:[EBP-8]
   017E2D97  BA 04000000     MOV EDX,4
   017E2D9C  E8 EFE0FFFF     CALL 017E0E90
   017E2DA1  FF75 FC         PUSH DWORD PTR SS:[EBP-4]
   017E2DA4  FF75 F8         PUSH DWORD PTR SS:[EBP-8]
   017E2DA7  8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
   017E2DAA  8338 00         CMP DWORD PTR DS:[EAX],0
   017E2DAD  74 02           JE SHORT 017E2DB1
   017E2DAF  FF30            PUSH DWORD PTR DS:[EAX]
   017E2DB1  FF75 F0         PUSH DWORD PTR SS:[EBP-10]
   017E2DB4  FF75 EC         PUSH DWORD PTR SS:[EBP-14]
   017E2DB7  C3              RETN
   ```

   if we do another Shift+F9 the App will strat runing .. so DON'T!!

5. now move the marker down to :

   ```
   017E2D7C  64:8F05 00000000 POP DWORD PTR FS:[0]
   ```

   and now press F2 (set Break Point).

   now press Shift+F9 and `Olly` will Break at our break point.

6. now trace with F8 (Trace Over) and execute the `RETN` and you will be here:

   ```
   017F4EC8  E9 080A0000     JMP 017F58D5 <- we are here
   ```

   execute with F8 this jump too..

   now we will be here:

   ```
   017F58D5  D3DE            RCR ESI,CL           <--we Land Here
   017F58D7  B9 7D966271     MOV ECX,7162967D
   017F58DC  81C1 38F10A23   ADD ECX,230AF138
   017F58E2  D3EE            SHR ESI,CL
   017F58E4  BA 9ECC7376     MOV EDX,7673CC9E
   017F58E9  81EA C56EFFD4   SUB EDX,D4FF6EC5
   017F58EF  81F2 B7104902   XOR EDX,24910B7
   017F58F5  C1CA 94         ROR EDX,94
   017F58F8  8BF2            MOV ESI,EDX
   017F58FA  81EE 87D851D2   SUB ESI,D251D887
   017F5900  C1CE B0         ROR ESI,0B0
   017F5903  8BC6            MOV EAX,ESI
   017F5905  C1C8 28         ROR EAX,28  
   017F5908  53              PUSH EBX
   017F5909  5E              POP ESI
   017F590A  81C5 974FAF73   ADD EBP,73AF4F97
   017F5910  BA 048A9178     MOV EDX,78918A04
   017F5915  F7D2            NOT EDX
   017F5917  F7D2            NOT EDX
   017F5919  81C2 FC756E87   ADD EDX,876E75FC
   017F591F  8BDA            MOV EBX,EDX
   017F5921  41              INC ECX-------------------------
   017F5922  8BD9            MOV EBX,ECX                |
   017F5924  D1C3            ROL EBX,1                | a Realy Long
   017F5926  81F3 A38FD7AC   XOR EBX,ACD78FA3          |   Loop
   017F592C  3BD8            CMP EBX,EAX                |
   017F592E  F85 EDFFFFFF    JNZ 017F5921--------------------
   017F5934  8BC1            MOV EAX,ECX  <<-- Here To Set A Break Point!
   ```

   well we will not trace the hole loop by pressing F8 !! we will put a Break Point

   on :

   ```
   017F5934  8BC1            MOV EAX,ECX
   ```

   now after setting the Break Point (F2) press F9 (Run) and we will break at our BP



7. Now what?? We trace all with `F8` / `F7` (Trace Into) all the way??!!

Well.. NO!!

We have a nice feature in Olly called Command Line

Choose from the menu `Plugins->Command line->Command line`

Now we are going to write a trace command with condition!

YES! Olly does have a Trace command! It's called - `TC` - Trace Condition

It will trace till the condition is True!

So.. Write in the text Box: `TC EIP<900000`

And now press `Enter`.

At the window right Down you will see: "Tracing"

At my Slow computer it took me about 8+ min (350MHz)

At another computer that i tried it took 2+ min (900MHz Celeron)

So if anyone would like to buy me a computer u know where to find me and msg me.

8. After the trace is done we break here:

```
0057EA5B    E8            DB E8
0057EA5C    00            DB 00
0057EA5D    8A            DB 8A
0057EA5E    E8            DB E8
0057EA5F    FF            DB FF
```

Hey!! I Dont See Shit!! What is this?!

Well this code has not been analyse yet by Olly

It doesn't really matter WE ARE AT THE OEP!

`OEP=57EA5B`

9. Important! At the OEP look at the value of `EAX` (top right - CPU Window)

`EAX=57E318`

Write it down and Keep it we could need it!

10. Now open `ProcDump`/`Pe-Edit` and Dump fully our process.

===================================
Step 2 - Finding the Stolen Bytes
===================================

Well to the Stolen Bytes there is currently 3 ways:

1. `ASPR` will Execute the stolen bytes and then go to the OEP like this:

```
Stolen_Bytes_1
Stolen_Bytes_2
Stolen_Bytes_3
.....
.....
PUSH_THE_ADDRESS_OF_OEP
RET_TO_OEP
```

2. `ASPR` will first Save the Stolen Bytes then it will Erase the bytes From the OEP

Then will `Jmp` to the OEP like this:

```
Stolen_Bytes_1
Stolen_Bytes_2
Stolen_Bytes_3
PUSH EBX
PUSH ESI
PUSH EDI
......
......
......
REPZ STOSB <- Erase the bytes
    POPFD
    POPAD
JMP_TO_OEP
```

3. With NO Stolen Bytes it just `PopAD` and jumps to the OEP (No Stolen Bytes)

So with what we are dealing with??

To view the last steps of `ASPR` before it got to our OEP we need to See the Trace Log

YES! Olly got a Trace code log that called "Run Trace"

So.. choose the menu `View->Run trace`

Now we will See a new Window.. go to the line one before the last and u will see that the CPU window also changes to the same address and it will look like this:

```
017F5779  F3:AA           REP STOS BYTE PTR ES:[EDI]
017F577B  9D              POPFD
017F577C  61              POPAD
017F577D  -E9 D992D8FE    JMP SYSTEMCL.0057EA5B  <-- the address of the OEP
```

So we are dealing with #2 way! ;)

This i found that is working on `winXP` NOT `win98`:

So now in the CPU window Press `Ctrl+S` (Search for Sequence of Commands)

And write this there:

```
PUSH EBX
PUSH ESI
PUSH EDI
```

You will find:

```
01029227  0055 8B         ADD BYTE PTR SS:[EBP-75],DL
0102922A  EC              IN AL,DX                                      ; I/O command
0102922B  83EC 54         SUB ESP,54
0102922E  53              PUSH EBX
0102922F  56              PUSH ESI
01029230  57              PUSH EDI
01029231  6A 11           PUSH 11
```

Ignore the "00" and you will have:

Our stolen bytes: `55`, `8b`, `ec`, `83`, `ec`, `54`

Write them down too we will need them later...

NOW CLOSE OLLY AND EXIT IT...

WE DONT NEED IT FOR NOW...

=============================
Step 3 - Rebuilding the IAT
=============================

1. First run our app and wait till its loaded.
2. Open `Imprec` and choose our process from the list
3. Press on "IAT AutoSearch"
4. Change the size from "BC" to `1000` (BC is too small!!)
5. Press on "Get Imports"
6. Press on "Show Invalid"
7. Right click on the invalid that was shown and choose "Trace level 1"
8. Press on "Show Invalid" again.. and now we should get somthing like this:

(cut from the saved tree)

```
FThunk: 0019E258    NbFunc: 00000400
1     0019E258      kernel32.dll    00D6      DeleteCriticalSection
1     0019E25C      kernel32.dll    0228      LeaveCriticalSection
................
snip-snip
................
1     0019E2A4      kernel32.dll    01D1      GetThreadLocale
1     0019E2A8      kernel32.dll    01B9      GetStartupInfoA
0     0019E2AC      ?               0000      017E0F2C      <-- good
0     0019E2B0      ?               0000      017E139C      <-- good
1     0019E2B4      kernel32.dll    018B      GetModuleFileNameA
1     0019E2B8      kernel32.dll    0183      GetLocaleInfoA
1     0019E2BC      kernel32.dll    0181      GetLastError
1     0019E2C0      kernel32.dll    0158      GetCurrentDirectoryA
0     0019E2C4      ?               0000      017E1408      <-- good
1     0019E2C8      kernel32.dll    0133      FreeLibrary
1     0019E2CC      kernel32.dll    011C      FindFirstFileA
..................
snip-snip
..................
1     0019E314      kernel32.dll    00A0      CloseHandle
0     0019E318      ?               0000      0255A00E      <-- BAD
1     0019E31C      user32.dll      0112      GetKeyboardType
1     0019E320      user32.dll      019F      LoadStringA
1     0019E324      user32.dll      01AD      MessageBoxA
1     0019E328      user32.dll      0026      CharNextA
0     0019E32C      ?               0000      70F7D832      <-- BAD
1     0019E330      advapi32.dll    00F7      RegQueryvalueExA
1     0019E334      advapi32.dll    00EF      RegOpenKeyExA
1     0019E338      advapi32.dll    00D8      RegCloseKey
0     0019E33C      ?               0000      F37514C2      <-- BAD
1     0019E340      oleaut32.dll    0006      SysFreeString
1     0019E344      oleaut32.dll    0005      SysReAllocStringLen
1     0019E348      oleaut32.dll    0004      SysAllocStringLen
0     0019E34C      ?               0000      4007F56E      <-- BAD
1     0019E350      kernel32.dll    0307      TlsSetvalue
1         0019E354    kernel32.dll    0306      TlsGetvalue
1     0019E358      kernel32.dll    01E6      GlobalAlloc
0     0019E35C      ?               0000      017E139C      <-- good
0     0019E360      ?               0000      BF57C0D8      <-- BAD
.......................
snip-snip
.......................
```

Well it continue like this on and on..

We need to manually choose all the BAD address (Not in the packer code) and then choose "Cut Thunk(s)" from the right click of the mouse on them.

* Watch out at the end.. there is a good thunk in there..

Now we need to fix those other Good bytes.. u can download from `fraviamb.cjb.net` some plugs

I saw there or to read my other tuts about how to rebuild those (`New2Cracking.cjb.net` or `Fraviamb.cjb.net`)

Here is my fix:

```
0019E2AC-> 017E0F2C ->GetProcAddress
0019E2B0-> 017E139C ->GetModuleHandleA
0019E2C4-> 017E1408 ->GetCommandLineA
0019E35C-> 017E139C ->GetModuleHandleA
0019E428-> 017E1420 ->LockResource
0019E47C-> 017E13C4 ->GetVersion
0019E4AC-> 017E0F2C ->GetProcAddress
0019E4B8-> 017E139C ->GetModuleHandleA
0019E4FC-> 017E13F8 ->GetCurrentProcessId
0019E500-> 017E13F0 ->GetCurrentProcess
0019E50C-> 017E1430 ->FreeResource
```

9. Now press on "Fix Dump" and choose our Dumped file..

`Imprec` will save the file with "_" before the name of the dump file.

=========================
Step 4 - Fixing the OEP
=========================

1. Now open `HVIEW` /Hex Editor and go to the OEP offset: `57ea5b`
   Go up (back) to `57ea55` (6 bytes back) and write instead of the "00" our stolen bytes. Now save our new file.



2. 现在打开`ProcDump`/`Pe-Edit`，加载我们重建的文件到`Pe-Edit`中并更改

入口点为：`57ea55`（因为被偷取的字节）

3. 如果你现在尝试运行应用程序，它会崩溃！

4. 打开`Olly`并加载我们的修复后的文件

5. 在OEP：`57ea55`，`EAX`的值是什么？还记得我告诉过你在dump之前要保存OEP处`EAX`的值吗？它是`EAX=57E318`。这个值与我们当前的值不同，这意味着我们还有一条被偷取的指令：

`MOV EAX, 0157E318`

在字节中它占用5个字节（你可以在`HVIEW`中尝试自己验证）

`B818E35700` `MOV EAX, 0157E318`

这意味着我们有`6+5=11`(十进制)个被偷取的字节

这意味着真实的OEP是：

`11`=`B`(十六进制)

`OLD_OEP-STOLEN_BYTES=57EA5B-B=57EA50`

`OEP = 57EA50`

现在去`HVIEW`/`Hex Edit`中的新OEP并写入：

`55,8B,EC,83,EC,54,B8,18,E3,57,00`

现在使用`ProcDump`/`Pe-Edit`修复入口点为：`57EA50`

现在程序可以运行了！！！

The End! `4dd466d0e1e0cc123afbe504da2462ca`
