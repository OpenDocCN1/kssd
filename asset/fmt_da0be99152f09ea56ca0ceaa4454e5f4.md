
我们需要在读完 `IMAGE_IMPORT_DESCRIPTOR` 还没有读 `Name` 的时候 patch 判断 `FirstThunk`。

就在 `0045D11A` 处补丁。我开始在代码段找了一块 0 串，结果改了不让保存，郁闷！我就自己加了一个区段 `VA:1AA000`。

`45D11A` 处改为：

```
0045D11A   .- E9 E1CE1400    jmp     005AA000
0045D11F      90                  nop
```

`5AA000` 处代码写：

```
005AA000  - 0F84 2936EBFF    je      0045D62F
005AA006     60              pushad
005AA007     6A 00         push    0
005AA009     6A 04         push    4
005AA00B     8B4D CC      mov     ecx, dword ptr [ebp-34]
005AA00E     038D 40F9FFFF add     ecx, dword ptr [ebp-6C0]
005AA014     68 50A05A00    push    005AA050
005AA019     51           push    ecx                 /////FirstThunk 的值 409C
005AA01A     A1 685A4D00   mov     eax, dword ptr [4D5A68]
005AA01F     50           push    eax
005AA020     E8 4951F0FF   call    <jmp.&KERNEL32.ReadProcessMemory>
005AA025     A1 50A05A00   mov     eax, dword ptr [5AA050]
005AA02A     85C0          test    eax, eax
005AA02C     61           popad
005AA02D  - 0F85 ED30EBFF jnz     0045D120        ///不为 0 继续执行
005AA033  - E9 DC35EBFF    jmp     0045D614          ///为 0 已读大小加上，再读下一条 IMAGE_IMPORT_DESCRIPTOR
005AA038     90           nop
```

然后保存。用这个修改的 `OllyICE.exe` 去打开 `antiod` 试试没问题，加载别的程序也没出错。

加点 `OD` 对这个 `anti` 溢出的说明：

有个朋友发帖分析了，我只稍微提下。

问题在这个格式化函数处，它把目标格式化成 `USER32.MessageBoxA` 的形式。用格式化成 `USER32.MessageBoxA` 作例子说明。

```
0045D51D   .  B9 00010000   mov     ecx, 100                 //////作者的期望是格式化的结果在 256 个字节以内。
0045D522   .  2BC8          sub     ecx, eax                  ///先减去 USER32 的长度，6 个大小
0045D524   .  8D85 78FCFFFF lea     eax, dword ptr [ebp-388]
0045D52A   .  83E9 02       sub     ecx, 2                    //再减去'.'+结束 00 2 个大小
0045D52D   .  51           push    ecx                       ///ecx 就是剩余空间大小
0045D52E   .  50           push    eax
0045D52F   .  68 92C04B00   push    004BC092                           ;  ASCII "%s.%.*s"
0045D534   .  52           push    edx                             /////USER32    这里实际大小只读了 255 个字节，既 DLL 名如果大于 255 个字节，256 位写 00
0045D535   .  E8 F2960400   call    004A6C2C
```

问题就在这里了，如果 `USER32` 名字大于等于 255，`ecx` 值将为 -1，我们看看它的内部处理：

```
004A6C2C  /$  55           push    ebp
004A6C2D  |.  8BEC         mov     ebp, esp
004A6C2F  |.  8B45 08      mov     eax, dword ptr [ebp+8]
004A6C32  |.  8D4D 08      lea     ecx, dword ptr [ebp+8]
004A6C35  |.  C600 00      mov     byte ptr [eax], 0
004A6C38  |.  8D45 10      lea     eax, dword ptr [ebp+10]
004A6C3B  |.  50           push    eax
004A6C3C  |.  6A 00        push    0
004A6C3E  |.  6A 00        push    0
004A6C40  |.  8B55 0C      mov     edx, dword ptr [ebp+C]
004A6C43  |.  52           push    edx
004A6C44  |.  51           push    ecx
004A6C45  |.  68 046C4A00  push    004A6C04
004A6C4A  |.  E8 6D020000  call    004A6EBC
004A6C4F  |.  83C4 18      add     esp, 18
004A6C52  |.  5D           pop     ebp
```

格式化都在 `4A6EBC`，这个函数太长了，只贴关键点说明下。

用一个临时变量，我们用 `mark` 标记，用 `IDA` 的标记功能比较爽，这里还贴 `OD` 的吧。

根据参数 `;  ASCII "%s.%.*s"` 给 `mark` 赋值。

走 `USER32` 时 `mark = -1`;

```
004A6F5B  |.  33D2         |xor     edx, edx
004A6F5D  |.  8955 F0      |mov     dword ptr [ebp-10], edx
004A6F60  |.  83CA FF      |or      edx, FFFFFFFF
004A6F63  |.  C645 F7 00   |mov     byte ptr [ebp-9], 0
004A6F67  |.  8955 F8      |mov     dword ptr [ebp-8], edx           ///mark
```

走 `MessageBoxA` 时 `mark` 值为传入的参数 `ecx` 剩余空间大小，中间有个处理参数为 0 `null` 出去的；

关键在这里了：

```
004A74A8  |> \837D F8 00   |cmp     dword ptr [ebp-8], 0
004A74AC  |.  7C 05       |jl      short 004A74B3
004A74AE  |.  8B45 F8     |mov     eax, dword ptr [ebp-8]
004A74B1  |.  EB 05       |jmp     short 004A74B8
004A74B3  |>  B8 FFFFFF7F |mov     eax, 7FFFFFFF
004A74B8  |>  8B55 E8     |mov     edx, dword ptr [ebp-18]
004A74BB  |.  33C9        |xor     ecx, ecx
004A74BD  |.  894D C0     |mov     dword ptr [ebp-40], ecx
004A74C0  |.  EB 05       |jmp     short 004A74C7
004A74C2  |>  48          |/dec    eax
004A74C3  |.  FF45 C0     ||inc     dword ptr [ebp-40]
004A74C6  |.  42          ||inc     edx
004A74C7  |>  85C0        | test    eax, eax
004A74C9  |.  0F84 CF000000 ||je     004A759E
004A74CF  |.  803A 00     ||cmp     byte ptr [edx], 0
004A74D2  |.^ 75 EE       |\jnz     short 004A74C2
004A74D4  |.  E9 C5000000  |jmp     004A759E
```

/// `mark < 0` 就记下 `dword ptr [ebp-18]` 字符串的长度保存在 `dword ptr [ebp-40]` 中，后面用这个长度格式化；不小于 0 取 `mark` 值和 `dword ptr [ebp-18]` 字符串的长度的最小值。

如果 `DLL` 名字大于等于 255 字符，分配的栈空间是按格式化 256 字符长分配的，函数名超长的再格式化自然溢出了。如果 `DLL` 名字小于 255 字节，`.` 后的函数名取前面字节填充 256 字节剩余空间。这样顶多出现函数名不全的情况，不会溢出。

`Windows` `DLL` 文件名好像只能取到 240 多位（我随便重命名了一个文件，数了下）。保证程序能运行的情况下用我上面的 patch 方法不会出现溢出的情况。

实际上这样 patch 以后也只能保证加载的时候不溢出。如果程序在运行的时候把 `IMAGE_IMPORT_DESCRIPTOR` 改的面目全非，程序运行的时候 `IAT` 填充完了，不影响程序运行。`OD` 这时附加即使不崩溃也读的内容乱七八糟。

读输入表 `0045D0FB` 这句所在函数过程需要说明下，它的作用如下：

```
0045D0FB   . |E8 0C420000   call    _Readmemory                 ////此处为读一项 IMAGE_IMPORT_DESCRIPTOR 的
```

在 `OD` 的 `CPU` 窗口里 `call dword ptr [402198]` 显示成 `00401770 call dword ptr <&MSVCRT.__p__commode>`。针对它的解决方法是 `NOP` 掉读原始输入表的过程，但是这样似乎不太好。我的方法就是用海风的插件，哈哈！！！ `海风` 的插件真是好东西，支持 `海风`!!!!!

附上我修改的 `OllyICE.exe`

上传的附件

![OllyICE.rar](http://bbs.pediy.com/showthread.php?t=80224)
