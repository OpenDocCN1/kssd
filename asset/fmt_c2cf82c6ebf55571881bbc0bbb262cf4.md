
# 1stOptv1.5 Demo

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-313.htm`](https://www.pediy.com/kssd/pediy08/pediy8-313.htm)

## 移花接木巧妙 Crack 1stOptv1.5 Demo

追本溯源轻松 Keygen 1stOptv1.0

【文章标题】: 移花接木巧妙 Crack 1stOptv1.5 Demo

追本溯源轻松 Keygen 1stOptv1.0

【文章作者】: `winndy`

【联系方式】: `CNwinndy@hotmail.com`

【目标】:

*   `1stOptv1.5` Demo : `http://www.7d-soft.com/cn/index.htm`
*   `1stOptv1.0`
*   `Auto2Fit3.0`

【使用工具】: `OllyICE`、`DEDE`、`OllyDump` 插件、`CodeHelper` 插件

【操作平台】: `Windows2003` Server

【作者声明】: 本文纯属技术交流，只作学习使用，不得用于商业用途。转载请注明作者并保持文章的完整。失误之处敬请诸位大侠赐教!

【文章结构】:

*   一、揭开迷雾：`1stOptv1.5` 是个 Demo
*   二、移花接木：`1stOptv1.0` 功能齐全
*   三、追本溯源：`Auto2Fit` v3.0 和 `THKStreams` Delphi Component
*   四、修复 Bug：帮作者修复，同时学习 Inline Patch

### 一、揭开迷雾：`1stOptv1.5` 是个 Demo

#### 1. 脱壳

`Aspack` 压缩，到达 OEP 后直接用 `OllyDump`(缺省参数)脱壳，可以运行。

#### 2. 未运行版本的限制

1.  关于窗口上显示“未注册版”。
2.  右键复制结果时，弹出 MessageBox“未注册版不能进结果复制操作！”。
3.  代码本上右键粘贴，弹出 MessageBox“未注册版不能进行粘贴操作！”。
4.  双击左侧文件浏览器中的 Example 文件，窗口标题后 `[]` 内是文件的全路径，修改 Example 文件后，点保存，弹出 MessageBox“未注册版不能保存为‘.mff’格式！”。
5.  若新建一个代码文件，则窗口标题后 `[]` 内是 `Untitled1`，点保存，弹出保存对话框，save 后不弹出任何对话框(1.0 中会弹出 MessageBox“未注册版不能保存为‘.mff’格式！”)，但实际上文件没有保存，而且窗口标题不含路径。
6.  在新建一个代码文件前，双击左侧文件浏览器中的文件，窗口标题随着改变，但在新建一个代码文件后，也就是窗口标题变为 `[Untitled1]` 后，再双击左侧文件浏览器中的文件，窗口标题不再改变。注意：在 1.0 版的使用版中不存在这个现象。
7.  点“编辑”，再“插入”，再点“文件数据”，弹出 MessageBox “未注册版不能进行数据插入操作！”。
8.  算出结果后，右键点击保存，可以看到可以保存为文本文件，选择文件名后，弹出 MessageBox “未注册版不能保存结果内容！”

#### 3. 载入 1stOpt\_unpacked

右键查找参考字符串，搜索“未注册”，在本段代码起始处设好断点：

**第一处：**

```
0068E23C    push   ebx            ; 下断
 [省略…]
0068E2C8  mov     eax, 0068E2E0    ;  未注册版不能进结果复制操作！
0068E2CD  call    0046E9B8        ;  D7.Dialogs.ShowMessage(AnsiString);
0068E2D2  pop     esi
0068E2D3  pop     ebx
0068E2D4  retn
```

**第二处：**

```
0068E300   mov    eax, 0068E314      ;  未注册版不能进行粘贴操作！
0068E305   call   0046E9B8          ;  D7.Dialogs.ShowMessage(AnsiString);
0068E30A   retn
```

**第三处：**

```
0068E330  push   ebp
 [省略…]
0068E341  mov   eax, 0068E36C     ;  未注册版不能保存为.mff 文件！
0068E346  call   0046E9B8        ;  D7.Dialogs.ShowMessage(AnsiString);
 [省略…]
```

**第四处：**

```
0069F404  mov    eax, 0069F418    ;  未注册版不能进行插入数据操作！
0069F409  call   0046E9B8        ;  D7.Dialogs.ShowMessage(AnsiString);
0069F40E  retn
```

在进行调试的时候，根本找不到一个可以跳到实际执行程序功能的地方，即使是向上面的 call 追，也还是毫无头绪。开始怀疑这就是 Demo 版。干脆把 1.0 的拿来看看，对比一下，看有什么值得借鉴的参考的地方。

在网眼天下，有篇爆破的文章：

完美爆破 1stOpt1.0 数学优化分析综合工具软件包

【破文作者】`luzhmu`

跟了一下，关键点在于

```
0062A954    mov     eax, ebx
```

前面则有：

```
0062A8E9    xor     ebx, ebx     //改为 mov  bl, 1 就可以了
0062A8EB    jmp     short 0062A8EF
0062A8ED    mov     bl, 1
0062A8EF    xor     eax, eax
```

想要爆破的话，将 `xor ebx, ebx` 改为 `mov bl, 1` 就可以了，都是 2 个 bytes。

爆破后，发现一个奇怪的现象，运行完爆破后的 exe 文件，`1stOpt.dll` 就变成正式注册文件了，即再运行原文件，仍显示注册成功。看来，程序在校验完 `1stOpt.dll` 后，重新写了注册文件。这就激起了我写注册机的想法。别急。

在跟踪 1.0 的时候，我看到解码出很奇怪的字符串：

```
0062A7E0    call    [ecx+5C]    
```

这句执行之后，`d [edx]`，可以在内存中看到：

```
00CB8030  €?0€?DCHTTVVUPNWORWQVDAFGM
00CB8050  G..BHASQMMPQMJOMYAFKATXV..SYUWVC
00CB8070  TWHIHIUDABLMKWP..TFLDDBEQKSXWVLJ
 [省略后面的…]
```

仔细看看这些字符串，夹杂着：用户名(`xycheng`)，`CPUID`，`HardDiskID`，1.0(谁都猜得到是版本号).

转念一想，要是我把 1.5 的 `1stOpt.dll` 拿来，把版本号改为 1.5，那不就可以得到一个 1.5 的正式版注册文件吗？会出现 MessageBox“版本号不同”，很容易找到 `lstrcmp`，把它跳过，得到 1.5 版的正式注册文件。

满怀激动的心情，把 `1stOpt.dll` 放到 1.5 目录下去，仍然是未注册。难道算法改变了？那你至少也得读这个 `1stOpt.dll` 文件吧。搜索字符串，未找到；干脆删了这个 `1stOpt.dll`，看你报错不，结果它不理睬。在 1.0 下，则会报“缺少库文件”的错。

再比较一下 1.0 和 1.5 弹出“未注册版不能…”的代码，发现 1.5 根本就不包含正式功能的代码。

晕死了！就无法破解了吗？

### 二、移花接木：`1stOptv1.0` 功能齐全

哈哈，幸好 1.0 的代码功能是齐全的，何不把 1.0 的代码拿到 1.5 中来使用呢？这两个版本差别不大，应该是没问题的。最后是移植成功，幸亏不要我去修改 Import Table。

上面，已经得到注册文件了，有了正式版 1.0。分别用 OD 打开 1.0 和 1.5，找到相对应的地方，用你最喜欢的十六进制编辑工具(我用 `winhex`，`shooo` 教我用的)把 1.0 的代码复制到 1.5 里面去。1.5 的代码很短，空间不够，跳到一个空白地方在进行复制。复制完一个功能，就打开 OD 进行调试，知道这个功能调试好，再复制下一个功能。

#### 第一处

**1.5 修改前**

```
0068E2C8  mov     eax, 0068E2E0        ;  未注册版不能进结果复制操作！
0068E2CD  call  0046E9B8               ;  D7.Dialogs.ShowMessage(AnsiString);
0068E2D2  pop     esi
0068E2D3  pop     ebx
0068E2D4  retn
```

**修改后：**

```
0068E2C8    mov     eax, [esi+394]
0068E2CE    call    005262A0
0068E2D3    pop     esi
0068E2D4    pop     ebx
0068E2D5    retn
```

**1.0 中是：**

```
00683530   mov   eax, [esi+394]
00683536   call    0050D820
0068353B   pop     esi
0068353C   pop     ebx
0068353D   retn
```

#### 第二处

**1.5 修改后：**

```
0068E300   push   ebx
0068E301   push   esi
0068E302   mov    ebx, eax
 [省略…]
0068E329   jmp    006D0156    ;空间不够，跳到其他地方去
006D0156   jge    short 006D0173
 [省略…]
006D017B   pop    ebx
006D017C   retn
```

**1.5 修改前：**

```
0068E300  mov   eax, 0068E314      ;  未注册版不能进行粘贴操作！
0068E305  call  0046E9B8          ;  D7.Dialogs.ShowMessage(AnsiString);
0068E30A  retn
```

**1.0 中：**

```
00683540   .  53         push    ebx                          ;  Paste
00683541   .  56         push    esi
 [省略…]
006835A5   .  5B         pop     ebx
006835A6   .  C3         retn
```

#### 第三处

**1.5 修改后：**

```
0068E330   jmp    006D0180           ;  空间不够，jmp to save .mff
006D0180   push   ebp             ;  save .mff file fixed code
006D0181   .mov   ebp, esp
006D0183    add    esp, -28
 [省略…]
006D01AD   push   ebp
006D01AE   push   006D03FF
006D01B3    push   dword ptr fs:[eax]
006D01B6    mov    fs:[eax], esp
006D01B9    mov    eax, [6D91D8]
 [省略…]
006D01D9  mov  eax, [41712C]
006D01DE  call  004036EC      ;  D7.System.TObject.Create(TObject;Boolean);
 [省略…]
006D01F3  mov  edx, 006D0418    ;  ASCII "1stOpt File"
006D0202  mov  edx, [6D88CC]    ;    1.5
006D0208   .  E8 7B45D3FF    call    00404788
 [省略…]
006D03CE  mov    eax, [6D91D8]
006D03D3  mov    eax, [eax]
006D03D5  xor    edx, edx
006D03D7  call   004629AC
006D03DC  xor    eax, eax
006D03DE  pop    edx
006D03DF  pop    ecx
006D03E0  pop    ecx
006D03E1   mov    fs:[eax], edx
006D03E4   push   006D0406
006D03E9   lea    eax, [ebp-28]
006D03EC   mov    edx, 4
006D03F1   call   00404588
006D03F6   lea    eax, [ebp-8]
006D03F9   call   00404564
006D03FE   retn

006D03FF   jmp    00403E9C  ；跳到==D7.System.@HandleFinally;
006D0404   jmp    short 006D03E9
006D0406   pop    edi
006D0407   pop    esi
006D0408   pop    ebx
006D0409   mov    esp, ebp
006D040B   pop    ebp
006D040C   retn   0C
```

**1.5 修改前：**

```
0068E330    push   ebp
0068E331    mov    ebp, esp
0068E333    xor    eax, eax
0068E335    push   ebp
0068E336    push   0068E359
0068E33B    push   dword ptr fs:[eax]
0068E33E    mov    fs:[eax], esp
0068E341    mov    eax, 0068E36C         ;  未注册版不能保存为.mff 文件！
 [省略…]
```

**1.0 中：**

```
006835A8   push   ebp             ;  保存.mff 文件
006835A9   mov    ebp, esp
006835AB   add    esp, -28
 [省略…]
```

#### 第四处

**在 1.5 中修改后：**

```
0069F404  jmp    006D04C0        ；空间不够，跳到别处  
006D04C0   push   ebp            ;  InsertFileData
006D04C1   mov    ebp, esp
006D04C3   xor    ecx, ecx
 [省略…]
006D053D  |.  90           nop   ；在 1.0 中校验注册没，nop 掉
 [省略…]
006D054A  |.  90           nop
 [省略…]
```

**1.5 修改前：**

```
0069F404  mov   eax, 0069F418      ;  未注册版不能进行插入数据操作！
0069F409  call  0046E9B8          ;  D7.Dialogs.ShowMessage(AnsiString);
0069F40E   retn
```

**在 1.0 中：**

```
00693E7C   push   ebp            ;  InsertFileData
00693E7D   mov    ebp, esp
00693E7F   xor    ecx, ecx
 [省略…]
00693EF9   mov    eax, 2
00693EFE   call   0062A538       ；校验注册没有
00693F03   test   al, al
00693F05   je     short 00693F76
```

比较 1.0 和 1.5 的代码，复制过来后，要注意修改 call，还有一些变量，push 的常量。

修改的方法为：

1.  找到在 1.5 中与 1.0 中相同的函数，进入 1.0 的 call，复制几行能唯一标识这个函数的代码，`Ctrl+S`，在 1.5 中搜索，找到后，找到函数起始地址，修正 call。
2.  举个例子\[例子举的都是第三处的\]：

```
006D01AE   push   006D03FF
006D03FF   jmp    00403E9C  ；跳到==D7.System.@HandleFinally;
```
