
### 反汇编与 IAT 填充逻辑

```assembly
00424292     8BD8                     mov  ebx,eax                         ; ASCII "MSVBVM60.DLL"（VB 程序）
00424294     50                       push eax
00424295     FF95 4D0F0000            call dword ptr [ebp+F4D]            ; GetModuleHandleA 获取 dll 句柄
0042429B     85C0                     test eax,eax
0042429D     75 07                    jnz  short KillBox.004242A6          ; 成功则跳
0042429F     53                       push ebx                            ; 否则 LoadLibraryA 动态调入 dll 库
004242A0     FF95 510F0000            call dword ptr [ebp+F51]
004242A6     8985 45050000            mov  dword ptr [ebp+545],eax        ; 保存 dll 句柄
004242AC     C785 49050000 00000000  mov  dword ptr [ebp+549],0           ; 初始 IAT 首地址偏移量为 0，以后每填充完一个就+4
004242B6     8B95 22040000            mov  edx,dword ptr [ebp+422]        ; ImageBase
004242BC     8B06                     mov  eax,dword ptr [esi]            ; [esi]处是输入表 MSVBVM60.DLL 的 IID，这里取 OriginalFirstThunk
004242BE     85C0                     test eax,eax                        ; OriginalFirstThunk 是否为 0？
004242C0     75 03                    jnz  short KillBox.004242C5          ; 否，则跳去找函数名（本例非 0）
004242C2     8B46 10                  mov  eax,dword ptr [esi+10]         ; 是，则取 FirstThunk(＝00001000)
004242C5     03C2                     add  eax,edx                        ; OriginalFirstThunk+ImageBase 定位到 IMAGE_THUNK_DATA 数组首地址
004242C7     0385 49050000            add  eax,dword ptr [ebp+549]         ; IMAGE_THUNK_DATA 数组首地址+偏移量(N*4 字节)，定位到待填充函数的 IMAGE_THUNK_DATA 地址
004242CD     8B18                     mov  ebx,dword ptr [eax]            ; 取 IMAGE_THUNK_DATA
004242CF     8B7E 10                  mov  edi,dword ptr [esi+10]         ; 取 FirstThunk
004242D2     03FA                     add  edi,edx                        ; FirstThunk+ImageBase 定位到 IAT 首地址
004242D4     03BD 49050000            add  edi,dword ptr [ebp+549]         ; IAT 首地址+偏移量(N*4 字节)，定位到本次待填充的 IAT 目标地址
004242DA     85DB                     test ebx,ebx                        ; IMAGE_THUNK_DATA 数组是否已到末尾？
004242DC     0F84 A2000000            je   KillBox.00424384                ; 是，则结束
004242E2     F7C3 00000080            test ebx,80000000                   ; 否，则判断该函数 IMAGE_IMPORT_BY_NAME 还是 BY_INDEX 方式导入。（在填充了部分函数后，我们可以看到本例是 VB 语言写的程序，以"rtc"开头的函数都是以序号导入的方式，其他函数的则是 BY_NAME 方式）
004242E8     75 04                    jnz  short KillBox.004242EE          ; by_INDEX 方式则跳。
004242EA     03DA                     add  ebx,edx                        ; by_NAME 方式则+ImageBase 定位函数名
004242EC     43                       inc  ebx
004242ED     43                       inc  ebx                            ; 后移 2 位跳过成员 Hint 定位到函数名地址
004242EE     53                       push ebx                            ; by_NAME 方式时为函数名地址，by_INDEX 方式是时为 IMAGE_THUNK_DATA
004242EF     81E3 FFFFFFFF            and  ebx,7FFFFFFF                   ; 取序号导入的函数序号(按名称导入的不影响)
004242F5     53                       push ebx                            ; 函数名地址或函数序号
004242F6     FFB5 45050000            push dword ptr [ebp+545]            ; dll 句柄
004242FC     FF95 490F0000            call dword ptr [ebp+F49]            ; GetProcAddress 获取函数地址
00424302     85C0                     test eax,eax
00424304     5B                       pop  ebx
00424305     75 6F                    jnz  short KillBox.00424376          ; 成功则跳
00424376     8907                     mov  dword ptr [edi],eax           ; 跳到了这里，填充 IAT
00424378     8385 49050000 04         add  dword ptr [ebp+549],4          ; 距首地址偏移量+4，准备处理下一个函数
0042437F     E9 32FFFFFF              jmp  KillBox.004242B6               ; 循环处理
```

### 单个 DLL IAT 填充结束处理

一个 DLL 的 IAT 填充完毕后跳转到这里：

```assembly
00424384     8906                     mov  dword ptr [esi],eax           ; OriginalFirstThunk 成员指向 00000000
00424386     8946 0C                  mov  dword ptr [esi+C],eax          ; NAME 成员指向 00000000
00424389     8946 10                  mov  dword ptr [esi+10],eax         ; FirstThunk 成员指向 00000000
0042438C     83C6 14                  add  esi,14                         ; 指针移到下一个 IID
0042438F     8B95 22040000            mov  edx,dword ptr [ebp+422]        ; ImageBase
00424395     E9 EBFEFFFF              jmp  KillBox.00424285
```

### OEP 处理

全部 IAT 填充完毕后跳到这里，下面开始处理 OEP：

```assembly
0042439A     B8 60230000              mov  eax,2360                       ; OEP 的 RVA
0042439F     50                       push eax
004243A0     0385 22040000            add  eax,dword ptr [ebp+422]        ; RVA+ImageBase = OEP
004243A6     59                       pop  ecx                            ; 弹出 OEP 的 RVA
004243A7     0BC9                     or   ecx,ecx                       ; OEP 的 RVA 是否为 0
004243A9     8985 A8030000            mov  dword ptr [ebp+3A8],eax        ; 修改程序代码，将 OEP 写入压栈指令
```

修改程序代码之前下面的几行代码是这样的：

```assembly
004243AF     61                       popad
004243B0     75 08                    jnz  short KillBox.004243BA          ; OEP 的 RVA 不为 0 则跳，否则 game over
004243B2     B8 01000000              mov  eax,1
004243B7     C2 0C00                  retn 0C
004243BA     68 00000000              push 0
004243BF     C3                       retn
```

修改程序代码后是这样的：

```assembly
004243AF     61                       popad
004243B0     75 08                    jnz  short KillBox.004243BA          ; OEP 的 RVA 不为 0 则跳，否则 game over
004243B2     B8 01000000              mov  eax,1
004243B7     C2 0C00                  retn 0C
004243BA     68 60234000              push KillBox.00402360               ; 这里被修改成了 push 00402360
004243BF     C3                       retn                                 ; 前往 OEP
```

### 到达 OEP

```assembly
00402360     68 2C4D4000              push KillBox.00404D2C               ; ASCII "VB5!6&*vb6chs.dll"
00402365     E8 EEFFFFFF              call KillBox.00402358
0040236A     0000                     add  byte ptr [eax],al
0040236C     0000                     add  byte ptr [eax],al
0040236E     0000                     add  byte ptr [eax],al
00402370     3000                     xor  byte ptr [eax],al
00402372     0000                     add  byte ptr [eax],al
00402374     3800                     cmp  byte ptr [eax],al
00402376     0000                     add  byte ptr [eax],al
00402378     0000                     add  byte ptr [eax],al
0040237A     0000                     add  byte ptr [eax],al
0040237C     4F                       dec  edi
0040237D     C2 F150                  retn 50F1
```

现在可以用 `OD` 的 `OllyDump` 插件脱壳了，用方式一脱后可运行；用方式二脱后不能运行，原因是方式二没能正确识别以序号方式导入的函数，均成为无效函数，需要修复输入表（用 `ImportREC` 即可）。

### 小结

#### 一、壳的工作流程

1. 根据壳中的伪输入表，动态调入壳运行所需的函数。
2. 解压。将原程序的各个段依次解压到缓冲区，再从缓冲区中复制到目标区段。如果是代码段，则在缓冲区中将所有 `E804` 和 `E904` 开头的 `call` 和 `jmp` 指令解密出正确的目标地址，再行复制。
3. 填充 `IAT`。保留了输入表，根据输入表动态加载 `dll` 库，再根据 `OriginalFirstThunk` 迭代搜索函数名，动态获取函数地址再填入 `IAT`，填充完一个 `dll` 的函数后，却又把相应的 `IID` 的 `OriginalFirstThunk`、`Name`、`FirstThunk` 三个成员全指向了 0。
4. 转往 `OEP`。壳通过自修改代码，以 `push`+`retn` 的方式跳往 `OEP`。

#### 二、程序保护方面

这个壳也是一个早期的压缩壳，所以在保护方法也没采取什么措施。下面是一些简单的措施：

1. 采用了一些花指令干扰静态调试。
2. 对部分 `call` 和 `jmp` 的目标地址似乎是加密了，不过解压后在缓冲区中又解密并复制到代码段中。
3. 两次用了 `SMC` 技术，第一次却不是出于保护目的，而是为了控制正常流程（将 `mov bl,0` 改为 `mov bl,1`），使程序除代码段之外不去执行额外的修改 `call` 和 `jmp` 目标地址的工作；第二次倒是起到干扰静态调试的作用（将 `push 0` 改为 `push 00402360`）。

### 附件

| 文件 | 说明 |
| :--- | :--- |
| `KillBox.rar` | 样本文件 |
![图片](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)
