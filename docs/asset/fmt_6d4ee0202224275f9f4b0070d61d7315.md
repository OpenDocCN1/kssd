
# SVK Protector V1.32 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6651.htm`](https://www.pediy.com/kssd/pediy06/pediy6651.htm)

**【目标程序】**：`SVK Protector V1.32` 破解版加壳的某个 `dll`。附件中含有输入表和 `UnPacked` 以供参考。

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`WinXP`、`Ollydbg1.10`、`LordPE`、`DLL_Loader`、`ImportREC`

**【实例下载】**：点击此处下载（或右键另存为）

---

### 脱壳过程

`DLL` 脱壳系列已写了几篇了，虽然没有什么太有价值的东东，也算是留点资料吧。

前些日子有朋友提出看看 `SVK` 加壳的 `DLL`，正好看雪论坛恢复，所以写了这篇贺之。

手里只有 `SVK Protector V1.32 Demo` 破解版，和注册版加壳的 `dll` 有些方面是不同的。

关于 `SVK` 的脱壳可以参看我以前发的《伪 `SVK Protector 1.32` 脱壳+修复——`SVK Protector DEMO 1.32` 主程序》和《浅谈 `SVKP 1.3X` 壳的输入表修复——注册表医生 `V2.96` 脱壳+破解》。

### 一、轻松寻找 SVK 1.32 破解版加壳程序的 OEP

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
003C1000     60                 pushad//进入 OD 后停在这
003C1001     E8 00000000        call SVKTest.003C1006
003C1006     5D                 pop ebp
003C1007     81ED 06000000      sub ebp,6
003C100D     EB 05              jmp short SVKTest.003C1014
```

`F9` 运行，程序中断在 `SVK Protector V1.32` 破解版的典型异常处！

```
0425137F     6285 0E0B0000      bound eax,qword ptr ss:[ebp+B0E]
//SVK Protector V1.32 破解版的典型异常！
04251385     EB 02              jmp short 04251389
```

`CTR+F` 在当前位置下查找命令：`add edx, dword ptr ss:[ebp]`

找到在 `0426BC7A` 处，`F2` 下断。`Shift+F9` 运行，中断在 `0426BC7A` 处！

```
0426BC7A     0355 00            add edx,dword ptr ss:[ebp]; SVKTest.003B0000
//EDX=0000108C + 003B0000=003B108C    ★  这就是 OEP 值  :-)
0426BC7D     E8 32000000        call 0426BCB4
0426BC82     C3                 retn
```

直接下断：`HE 003B108C`    `F9` 运行，中断在 `OEP` 处！

```
003B108C     EB 10              jmp short SVKTest.003B109E//OEP
003B108E     66:623A            bound di,dword ptr ds:[edx]
003B1091     43                 inc ebx
003B1092     2B2B               sub ebp,dword ptr ds:[ebx]
003B1094     48                 dec eax
003B1095     4F                 dec edi
003B1096     4F                 dec edi
003B1097     4B                 dec ebx
003B1098     90                 nop
003B1099     E9 F8903B00        jmp 0076A196
003B109E     A1 7F903B00        mov eax,dword ptr ds:[3B907F]
003B10A3     C1E0 02            shl eax,2
003B10A6     A3 83903B00        mov dword ptr ds:[3B9083],eax
003B10AB     8B4424 08          mov eax,dword ptr ss:[esp+8]
003B10AF     A3 F1903B00        mov dword ptr ds:[3B90F1],eax
003B10B4     FF1485 E1903B00    call dword ptr ds:[eax*4+3B90E1]
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `SVKTest.dll`，然后完整脱壳，得到 `dumped.dll`。

以上寻找 `SVK 1.32` 破解版加壳程序的 `OEP` 的方法，既适用于 `EXE`，也适用于 `DLL`！

---

### 二、输入表

因为有 `SvkpIAT` 的 `ImportREC` 插件，所以就偷点懒了，呵呵。

从程序找几个 `API` 调用：

```
003B8B68     FF25 5CD03B00      jmp dword ptr ds:[3BD05C]
003B8B6E     FF25 60D03B00      jmp dword ptr ds:[3BD060]
…… ……
003B8C5E     FF25 B4D13B00      jmp dword ptr ds:[3BD1B4]
003B8C64     FF25 B8D13B00      jmp dword ptr ds:[3BD1B8]
```

开始地址＝`3BD05C`

结束地址＝`3BD1BC`

可以看出其中有一些填充的无效数据。

如果用 `ImportREC` 的插件来修复在 `Ollydbg` 中运行的 `DLL` 会导致 `ImportREC` 失去响应，所以我们用 `DLL_Loader.exe` 加载这个 `DLL`，然后运行 `ImportREC 1.42+` 来修复，`ImportREC 1.6` 对某些插件支持不佳。注意：去掉 `ImportREC` 的“使用来自磁盘的 `PE` 部首”的选项！程序属性中选择“启用调试权限”！

选中 `DLL_Loader.exe` 的进程，然后点“选取 `DLL`”，选择 `SVKTest.dll`，填入 `RVA=0000D05C`、大小＝`00000160` ，点“Get Import”，先用“追踪层次 1”修复了部分函数。通过后面的值可以看出 `FThunk: 0000D068 NbFunc: 00000026` 和 `FThunk: 0000D1A0 NbFunc: 00000003` 这些是无法数据，可以删除这 2 组指针。

再用“SvkpIAT”插件依次修复剩下的 4 个无效函数。通过以前的经验知道 `SVK Protector` 加壳的程序有几个特殊函数，如：`GetVersionExA`、`GetModuleHandleA`、`GetProcAddress`、`ExitProcess` 等，特别是 `ExitProcess`，用插件修复到这个函数时加壳程序会自动退出！几个函数依次修复如下：

```
0000D10C  kernel32.dll  00AC  ExitProcess
//修复到这个函数时 SVKTest.dll 会自动退出，通过跟踪原加壳 dll 也可以判定这个是 ExitProcess
0000D134  kernel32.dll  0168  GetModuleHandleA
0000D13C  kernel32.dll  018A  GetProcAddress
0000D150  kernel32.dll  01C8  GetVersion
0000D154  kernel32.dll  01C9  GetVersionExA
```

改 `OEP=0000108C`，可以保存这个树文件了。

---

### 三、重定位表

`SVK Protector V1.32` 破解版加壳 `dll` 没有加密重定位表，这省了我们很多工夫。从而使得这个壳的 `DLL` 脱壳相对来说简单了许多。

以前已经说过了，得到重定位信息的基本思路是对程序中需要重定位的代码地址处下“内存写入”断点，最终跟踪得到是哪里对这些数据进行处理的。比如这个 `dll` 可以从程序中一处需要重定位的地方：

```
003B109E     A1 7F903B00        mov eax,dword ptr ds:[3B907F]
```

这里 `003B1010` 处的 `3B` 就是重定位后的地址，所以我们可以在适当的时机对 `003B1010` 处下“内存写入”断点，从而得到 `SVK` 壳对 `dll` 重定位表的处理代码部分。

不断的实践+不断的总结，寻找相对最简便的方法，这或许是脱壳和 `Crack` 的乐趣吧？ :-)

现在我们重新用 `Ollydbg` 载入 `SVKTest.dll`，按照第一部分的步骤来到 `SVK Protector V1.32` 破解版的典型异常处！

```
0425137F     6285 0E0B0000      bound eax,qword ptr ss:[ebp+B0E]
//SVK Protector V1.32 破解版的典型异常！
04251385     EB 02              jmp short 04251389
```

第一部分时我们是从这里开始寻找 `OEP` 的，现在我们同样可以在这里开始寻找重定位表信息！

提供一个简便寻找 `SVK Protector V1.32` 破解版加壳 `DLL` 的重定位处理代码段的方法！

`Ctrl+S` 在当前位置下搜索命令序列：

```
add ecx,-8
push ebp
push ebx
push edx
shr ecx,1
```

找到在 `0426F3EE` 处，我们在其上 `sub ebx,esi` 的 `0426F3E0` 处下断，`Shift+F9` 运行，中断在 `0426F3E0` 处！

```
0426F3D8     8BB5 73E10100      mov esi,dword ptr ss:[ebp+1E173]
0426F3DE     8BDF               mov ebx,edi
0426F3E0     2BDE               sub ebx,esi 
//这里其实就是检测与映像基址是否相符。不符则重定位处理！ ★
//可以在这里改寄存器值 EBX=ESI，这样脱壳后就不需要修改基址了。 ★
0426F3E2     8B48 04            mov ecx,dword ptr ds:[eax+4]
//注意 EAX=003C0000
//重定位表 RVA=003C0000－003B0000＝00010000  ★
0426F3E5     8D50 08            lea edx,dword ptr ds:[eax+8]
0426F3E8     298D 97E10100     sub dword ptr ss:[ebp+1E197],ecx
0426F3EE     83C1 F8            add ecx,-8
0426F3F1     55                 push ebp
0426F3F2     53                 push ebx
0426F3F3     52                 push edx
0426F3F4     D1E9               shr ecx,1
0426F3F6     51                 push ecx
0426F3F7     8BCF               mov ecx,edi
0426F3F9     0308               add ecx,dword ptr ds:[eax]
0426F3FB     51                 push ecx
0426F3FC     E8 0C000000        call 0426F40D
0426F401     5D                 pop ebp
0426F402     83BD 97E10100 00   cmp dword ptr ss:[ebp+1E197],0
0426F409     75 D7              jnz short 0426F3E2
//重定位处理完毕后这里则不跳转     在下面下断
0426F40B     61                 popad
//中断在这里时 EAX=003C0814        这就是重定位表的结束地址啦 ★
0426F40C     C3                 retn
```

我们得到重定位表信息：

```
RVA=00010000 ，大小＝003C0814－003C0000＝00000814   ★
```

现在 `CTR+F` 在整个区段查找命令：`add edx, dword ptr ss:[ebp]`

```
0426BC7A     0355 00            add edx,dword ptr ss:[ebp]; SVKTest.003B0000
//EDX=0000108C + 003B0000=003B108C    ★  这就是 OEP 值  :-)
```

可以按照第一部分所示的方法走到 `OEP` 了。熟悉之后当然可以一次完成脱壳了，为了方便大家理解，所以写的详细点啦。

`Dump` 之后就可以用 `ImportREC` 载入第二部分得到的树文件，`FixDump`！

---

### 四、PE 修正、完成脱壳

用 `LordPE` 修正 `dumped_.dll` 的重定位表 `RVA=00010000`、大小＝`00000814`，保存之。

再用 `LordPE` 修正 `dumped_.dll` 的基址为 `003B0000`，`OK`，脱壳完成啦。

如果在 `0426F3E0` 做了修改，避开了重定位处理，则不需要修改基址。

---

```
                 ,     \_/ 
              /| \_.-~/            \\\_      ,        青春都一晌
             ( /~   /               \\~-.\_ |\\
             \`\\  \_/                ~\                忍把浮名 
  _-~~~-.)  \__/;;,.          \\_  //'
 /'_ ,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.       换了破解轻狂
 \`~ _( ,_..--\ (     ,;''/   ~--   /.\_\`\\
 /~~//'   /\`~\        /--..\_, \  \`~
 " \`~"  "    \`"   /~'\`\\   \`\\~~\
                  " "  "~' ""
```

    `Cracked By` 巢水工作坊——`fly [OCN][FCG][NUKE][DCM]`

          `2004-06-13  09:00 4dd466d0e1e0cc123afbe504da2462ca`
