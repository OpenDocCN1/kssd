
# Themida 1.9.1.0 版的通法破解

> 原文：[`www.pediy.com/kssd/pediy10/73257.html`](https://www.pediy.com/kssd/pediy10/73257.html)

## 文章信息

【文章标题】: Themida 1.9.1.0 版的通法破解

【下载地址】: 自己搜索下载

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

感谢 kanxue 坛主和 okdodo 大师提供的插件。

上一篇文章《Themida 的另类破解》只适用于 Themida v1.8.5.5 版本。那时我还以为它是 Themida 的最高版本，因为我用的 Themida 是从 `www.pediy.com` 的工具页上下载的，标签是 new(现在也如此)！直到文章发表后网友问能否用于 Themida v1.9.1.0 时，我才在网上看到了它，下载后加密了几个软件后，才知道它的利害！经过十多天的努力，终于也给破了。

### 一、请下载 okdodo 大师的插件“invisible”

用 Themida1.9.1.0 版加密的软件，有很强的反 anti 能力，在 OD v1.10 版上不能运行。请在 `bbc.pediy.com` 的精华篇中下载 okdodo 的插件 `invisible.dll`，添加到 OD 文件夹的 plugin 中，并删除 `HideOD.dll`(或改名为 `HideOD.dll.bak`)。这样 Themida v1.9.1.0 加密的软件就可以在 OD 中运行了。

### 二、Themida v1.9.1.0 的通法破解

因为 1.9.1.0 版本是 1.8.5.5 版本的升级，在重要的代码段中随机插入了大量的垃圾代码和花指令。不同的软件，甚至同一软件第二次加密其代码都是不一样的！这跟破解的“通用性”带来极大困难。但我终于搞出了一个通用的破解方法！但要使用三个脚本，反复用 OD 加载要解密的软件。三个脚本分别叫：脚本 1(断点侦测)、脚本 2(API 修复)、脚本 3(OEP 修复)。

#### 1. 脚本 1(断点侦测)的使用

脚本如下(已改进为全自动了,用它替换附件中的脚本 1)

```
//Themida 1.9.1.0 的断点设置脚本(改进版)
data:
  var memory
  var csize
  var tmp
  var temp
  var tmpbp
  var mem
  var mem0
      var mem1
  var mem2
  var mem3
   var mem4
  var mem5
  var Str0
  var Str3
  var Str4
  var Str5
  var Str1
      var Str2

star:
  gmi eip,CODEBASE
      mov memory,$RESULT
      gmi eip,CODESIZE
  mov csize,$RESULT
      add csize,memory
  
  find memory,#0000000000000000000000000000000000000000000000#
  mov tmp,$RESULT
  bphws tmp,"w"
      esto
  esto
  esto
  bphwc tmp
  sto
  find memory,#0000000000000000000000000000000000000000000000#
  mov tmp,$RESULT
      sub csize,tmp
      find memory,#909090909090909090909090909090909090#
  mov temp,$RESULT+8
  bphws temp,"w"
  mov tmpbp,eip
  run
  cmp eip,tmpbp+2
      jnz next
      run
  next:
  bphwc
      mov Str0,"断点地址："
      add Str0,#0D0A#
  mov mem,eip
  mov mem4,eip         //eip=Addr_4
  itoa eip
  mov Str4,"Addr_4="
      add Str4,$RESULT
      add Str4,#0A0D#
      find mem4,#83BD????????000F8497000000#
      mov mem5,$RESULT       //Addr_5
      itoa mem5  
      mov Str5,"Addr_5="
      add Str5,$RESULT  
      sub mem,4000  
  find mem,#83BD????????640F82??020000#
      mov mem0,$RESULT+7    //Addr_0
      itoa mem0
      add Str0,"Addr_0="
      add Str0,$RESULT  
      add Str0,#0D0A#
      find mem0,#C1C003#
      find $RESULT,#0385????7409#,40
      mov mem3,$RESULT+6   //Addr_3
      itoa mem3
      mov Str3,"Addr_3="
      add Str3,$RESULT  
      add Str3,#0D0A#

findon:
  find mem0,#AD#       //从 Addr_0 开始查找
      cmp $RESULT,0
      jz stop
      mov mem0,$RESULT
  find mem0,#01C8#,40
      cmp $RESULT,0     
  jnz findoff
      add mem0,40
      cmp mem0,mem5
      ja stop            //超过 Addr_5 后停止
      jmp findon

findoff:
      bp $RESULT
      add mem0,40
      itoa $RESULT
      jmp findon

stop: 
  run
@1:
      sti
      gn eax
      cmp $RESULT,0
      jz @1       
  itoa eip
      mov Str1,"Addr_1="
      add Str1,$RESULT
      add Str1,#0D0A#
  bpwm tmp,csize
      run
@3:
  sti
      cmp [eax],ecx
      jnz @3
  itoa eip
  mov Str2,"Addr_2="
  add Str2,$RESULT
  add Str2,#0A0D#
      add Str0,Str1
      add Str0,Str2
      add Str0,Str3
      add Str0,Str4
      add Str0,Str5
      msg Str0
      pause
```

脚本 1 的使用非常简单，用 OD 打开要解密的软件，直接运行脚本 1，第一次弹出的对话框给出了 6 个确定的断点.

Themida 每次加密代码都有变化，特征码很难捕获。比如要捕获的特征代码如果是 `mov [eax],ecx`：

那么，代码 `mov [eax],ecx` 可以有多种变化，如：

```
0068475C    51                 push    ecx
0068475D    813424 1F15FD3E    xor     dword ptr [esp], 3EFD151F
00684764    8F00               pop     dword ptr [eax]
00684766    52                 push    edx
00684767    BA 00000000        mov     edx, 0
0068476C    01C2               add     edx, eax
0068476E    8132 1F15FD3E      xor     dword ptr [edx], 3EFD151F
00684774    5A                 pop     edx
```

或者

```
00679700    51                 push    ecx
00679701    53                 push    ebx
00679702    BB AF61DB7F        mov     ebx, 7FDB61AF
00679707    315C24 04          xor     dword ptr [esp+4], ebx
0067970B    5B                 pop     ebx
0067970C    8F00               pop     dword ptr [eax]
0067970E    8130 AF61DB7F      xor     dword ptr [eax], 7FDB61AF     
```

(新改进的脚本 1 已经成为全自动了)

当你把这 6 个断点地址写入了脚本 2 后(可以打开脚本 2 直接修改)就可以关掉 OD 了。

#### 2. 脚本 2(API 修复)的使用

```
//本脚本适用于 Thmida 1.9.1.0 版本
bphwc
bc
data:
  var mem
  var mem1
  var temIAT
  var temESI
  var temAPI
  var APIstr
  var Addr_0
  var Addr_1
  var Addr_2
  var Addr_3
  var Addr_4
  var Addr_5

Init:
    //用对话框中的断点值修改下列各断点，Addr_1,Addr_2 的值必须单步跟踪后确定
    //只须修改这 6 个地址
  mov Addr_0,676d13      
  mov Addr_1,67763b       
  mov Addr_2,679714     
  mov Addr_3,679889           
  mov Addr_4,679cd6      
  mov Addr_5,67cc30     

start:  
  esto
  esto
  bphws Addr_0,"x"  
  esto   
  bphwc Addr_0
  mov [Addr_0],#90E9#  //将 jb 改为 jmp,去除代码扫描
  bp Addr_1     
  bp Addr_2     
  bp Addr_3     
  bp Addr_4  
  bp Addr_5     

First:
  run
  cmp eip,Addr_1
  jz A1
  cmp eip,Addr_2         
  jz A2
  cmp eip,Addr_3
  jz A3
  cmp eip,Addr_4
  jz A4
  cmp eip,Addr_5
  jz A5
  jmp First

A1:     
  mov temAPI,eax     //eax 是 API 函数地址
  gn temAPI         //显示函数名
  log $RESULT  
  add APIstr,"  "
  add APIstr,$RESULT_2  //恢复 API 函数名(INT 表)
  jmp First

A2:     
  mov temIAT,eax
  cmp edx,10000
  ja next
  xor edx,80000000
  mov [eax],edx        //修复 IAT 表(写入序列号)
  jmp First

next: 
  mov [eax],temAPI
  jmp First

A3:
  mov mem1,eax          //eax 是内存中呼叫 API 地址
  mov temESI,[esi]      //获取转跳标记
  jmp First

A4:
  cmp temESI,AAAAAAAA
  jnz step3
  mov [mem1],#FF25#      //修复代码中转跳地址
  mov [mem1+2],temIAT    
  jmp First

step3:
  mov [mem1],#FF15#      //修复代码中呼叫地址
  mov [mem1+2],temIAT
  jmp First

A5:
  bc
//---下面位置在运行脚本 3 后,写入 OEP 修复代码，若要再次用于新的程序，则应该将添加的代码删除 
  pause
```

脚本 2 的使用更简单，当用脚本 1 的 6 个地址把本脚本中对应地址修改好后存盘，用 OD 再次打开该软件，运行脚本 2。注意：这 6 个地址不能有丝毫差错！程序运行如飞，很快停止在断点 `Addr_5` 上。

注意到这时堆栈项是：13FF64，而刚打开 OD 时栈项是 13FFC4，差值是 60，后面平衡堆栈(脚本 3 中)要用。

将 OD 代码段转到 401000 开始的位置，用 OD 分析代码一次，你将看到所有的秘密都显示出来了。以提供的 FindFile 为例，转到 API 函数表位置(API 集中转跳位置 402646402736)，选出代码前面没有“$-”符号的部分(本例只有 3 个)：

（若全部都没有“$-”符号，再用 OD 分析一次）

```
00402652   .- FF25 A4304000 jmp     dword ptr [4030A4]   ;  USER32.DialogBoxParamA
004026B2   .- FF25 48304000 jmp     dword ptr [403048]   ;  kernel32.ExitProcess
004026DC   .- FF25 24304000 jmp     dword ptr [403024]   ;  kernel32.GetModuleHandleA
0040272A   $- FF25 04304000 jmp     dword ptr [403004]
00402730   $- FF25 00304000 jmp     dword ptr [403000]
```



# 4. 脚本 2 的最后完善与后续分析

## 1. 入口代码分析

注意到，`40272A`、`402730` 后面没有出现函数名称，但`[403004]`和`[403000]`中都有确定的值。用`alt-M`打开 Memory map，发现`comdlg32.dll`库是红色显示，我什么也没做，关掉 Memory map 后，居然正常显示了如下：（不能解释其原因）

```
0040272A   $- FF25 04304000 jmp     dword ptr [403004]    ;  comdlg32.GetOpenFileNameA

00402730   $- FF25 00304000 jmp     dword ptr [403000]    ;  comdlg32.GetSaveFileNameA
```

现在打开脚本 3(OEP 修复)，将没有“$-”符号的那 3 个函数的相应的代码(照原样写)，如下添加在脚本 3 中：

```
    mov [402652],#FF25A4304000#  //<--对应 USER32.DialogBoxParamA

    mov [4026B2],#FF2548304000#  //<--对应 kernel32.ExitProcess

    mov [4026DC],#ff2524304000#  //<--对应 kernel32.GetModuleHandleA

    mov [4030A4],77D3B144        //<--在该地址中查到的值(注意倒着写)，下同

    mov [403048],7C81CAFA

    mov [403024],7C80B731

    bp  402652                   //设置对应的断点，下同

    bp  4026B2

    bp  4026DC
```

### 1.1 为什么要写入脚本 3 中？

原来程序在运行了脚本 2 后，`Themida`早已侦查到了系统内存中出现了“调试器”，若继续运行，会在运行一段时间后中止，得不到最后结果。当你在脚本 3 中写入了相应代码后存盘，关闭 `OD`。

## 2. 脚本 3 (OEP 修复) 的使用

```
---------------------------------------------

data:

    var Addr_5

start:

      esto

      esto

    mov Addr_5,67cc30  // <----写入由脚本 1 得到的 Addr_5

      bphws Addr_5,"x"

      esto

      bphwc

      //----下面位置在运行脚本 2 后,添加与 OEP 有关的断点处代码，若要再次用于新的程序，则应该将全部代码删除

    mov [402652],#FF25A4304000#

    mov [4026B2],#FF2548304000#

    mov [4026DC],#ff2524304000#

    mov [4030A4],77D3B144

    mov [403048],7C81CAFA

    mov [403024],7C80B731

      bp  402652

      bp  4026B2

      bp  4026DC

      bpwm 404000,1000               //404000 的确定是根据 402000 是代码区，403000 是资源数据区

pause

------------------------------------------------------
```

`脚本 3`的作用实际上就是将 `OD` 运行到底(`Addr_5`的值一定要根据脚本 1 得到的修改)。按 `F9`，由于在脚本中添加了如上代码，程序将中断在上面 4 个断点之一上。添加这些代码的作用就是恢复被 VM 虚拟的 API（在入口代码段中），且运行后在将在堆栈中揭示其秘密。

### 2.1 执行流程分析

核对无错误后，用 `OD` 再次打开软件，加载脚本 3，程序停在 `Addr_5` 上。按 `F9`，`OD` 中断在

```
004026DC   .- FF25 24304000 jmp     dword ptr [403024]   ;  kernel32.GetModuleHandleA
```

在堆栈中看到：

```
0013FF7C   0066E2F0  /CALL 到 GetModuleHandleA
0013FF80   00000000  \pModule = NULL
```

这就是入口的第 1 个 API 调用。记下后，再 `F9`，中断在 VM 虚拟机代码段中：

```
00B9CD7E    F3:A4          rep     movs byte ptr es:[edi], byte ptr [esi]
```

下面的状态条中显示：

```
ecx = 00000004

ds:[esi]=[003E1FF8]=00

es:[edi]=[00404000]=00
```

原来是向`[00404000]`中写入代码，代码值就是当前堆栈中的 `003E1FF8` 中的值。`404000`就是 `GetModuleHandleA` 句柄地址。

继续 `F9`，中断在

```
00402652    - FF25 A4304000 jmp     dword ptr [4030A4]               ;  USER32.DialogBoxParamA
```

堆栈中显示：

```
0013FF6C   0066E2FC  /CALL 到 DialogBoxParamA
0013FF70   00400000  |hInst = 00400000
0013FF74   00000064  |pTemplate = 64
0013FF78   00000000  |hOwner = NULL
0013FF7C   00401B92  |DlgProc = FindFile.00401B92
0013FF80   00000000  \lParam = NULL
```

这就是入口的第 2 个 API 调用，其参数堆栈中显示非常清楚。因为程序将立即进入 `DialogBoxParamA`，直到关闭窗口后才退出，下面就交替 `F9` 或 `alt-F9`，直到窗口出现。（会反复多次按 `F9` 或 `alt-F9`，这是因为脚本中设置了“`bpwm 404000,1000`”的原因），我不知道怎样在运行中清除它？

关闭程序窗口，程序中断在

```
004026B2    - FF25 48304000 jmp     dword ptr [403048]               ;  kernel32.ExitProcess
```

堆栈中出现

```
0013FF7C   0066E309  /CALL 到 ExitProcess
0013FF80   00000000  \ExitCode = 0
```

请暂时不要关闭 `OD`，整个入口代码都清楚了。将 `OD` 代码转到 `40261C` 处（乱码开始位置），逐条汇编如下：

| 汇编语言 | 对应代码 |
| :--- | :--- |
| `push 0` | `6A00` |
| `call 4026DC` | `E8B9000000` |
| `mov [404000],eax` | `A300404000` |
| `push 0` | `6A00` |
| `push 401B92` | `68921B4000` |
| `push 0` | `6A00` |
| `push 64` | `6A64` |
| `push [404000]` | `FF3500404000` |
| `call 402652` | `E814000000` |
| `push 0` | `6A00` |
| `call 4026B2` | `E86D000000` |
| `int3` | `CC` `<--最后添上一个 int3，让其返回系统` |

## 3. 脚本 2 的最终完善步骤

现在打开脚本 2，在标签 `A5`：的 `bc` 后面写入（添加）如下代码：

### 3.1 修复 OEP

完整抄录 `OD` 上汇编后出的 2 进制码：

```
mov  [40261C],#6A00E8B9000000A3004040006A0068921B40006A006A64FF3500404000E8140000006A00E86D000000CC#
```

### 3.2 平衡堆栈与转跳

为平衡堆栈，实现对 OEP 的转跳，继续写入：（前面记录过栈项的差值=60）

```
mov  [Addr_5],#81C460000000B81C26400050C3#
```

即对应下列汇编的二进制代码：

```
add   esp,60
mov   eax,40261C
push  eax
retn
```

### 3.3 写入 API 名称

再写入如下代码，作用是将 API 函数名称写入适当空白处（如 `403600`）减轻 dump 后的录入文字工作。

```
mov  [403600],APIstr

sti
sti
sti
sti      //4 个单步
```

### 3.4 验证

写完后存盘，关闭 `OD`，再次打开 `OD`，加载脚本 2，程序停在 OEP 处。若按 `F9`，程序运行良好。

至此，`Thmida 1.9.1.0` 加密的软件完美解密！

# 三、破解思路整理

三个脚本，翻来复去地运行，有点头晕，整理思路：

运行脚本 1，得到 6 个断点地址--->写入脚本 2，运行脚本 2 得到几个“无关联函数”将其信息写入脚本 3，并设置相应断点--->运行脚本 3，从堆栈中获取 OEP 入口代码信息并回写入脚本 2---->运行脚本 2，实现完美解密。

# 四、后记

## 1. 适用范围

本文的方法适用于 `Thmida 1.8---1.9` 的各种版本。`1.8` 以前的版本我没有看过，`1.9.1.0` 以后的也没有看过，很难说通吃各种版本，但对于大多数被 `Themida` 加密的程序应该是没有问题的。（试验得不多，若有例外，研究后改进）其实我毫无保留地给出了我破解的详细思路和作法，让有兴趣者可以在此基础上加以发展，写出 `Themida` 脱壳机。（网上有个 `Themida` 脱壳机，不知什么版本，也根本不能运行）

## 2. 针对 VC++ 程序的修改

对于用 `VC++`编写的程序，运行脚本 3 不会有太好的结果，原因是没有“$-”符号的函数太多，大多是无用的（`VC-6` 编辑器几乎将 `MFC42D.DLL`、`MSVCRTD.dll` 等库中函数全部装载，不管你调用不调用）而 `windows` 下的 API 又很少使用，但 `VC++`的入口代码又具有相对固定的模式，甚至可以 `nop` 掉一堆 `SEH` 的设置代码，程序对入口的堆栈长度设置(如：`add esp,-6C`)也不敏感（参考我《对 themida1855 加密 VC++程序的完美脱壳》一文）。

对于用 `VC++`编写的程序在运行脚本 1 后，运行脚本 2 一次，获取入口地址后，在脚本 2 的 `A5`：标签后直接添加如下代码：

```
mov  [4xxxxx],APIstr                             //4xxxxx 表示找一段空白，写入函数名称

mov  [Addr_5],#81C460000000B8xxxx4x0050C3#       //xxxx4x 入口地址反向写入

mov  [4xxxxx],#558BEC6AFF90909090909090909090909090909090909090909090909083C4945356578965E8C745FC000000006A02#          //代码长度由现场决定，4xxxxx 表示 OEP 入口地址(乱码开始地址)，添加的代码几乎无须改动

sti
sti
sti
sti
```

将脚本 2 存盘一次，再次运行它，当你具有一定经验后，可能会得到满意的结果。

## 3. 注意事项

请注意，当须要把三个脚本用于新的程序时，请一定将前一个程序在脚本 2、脚本 3 中添加的代码删除，否则可能有不希望的结果发生。

## 4. Dump 后处理

通过三个脚本解密的程序，可以 `dump` 了，但 dump 后的程序还要加工才能运行。如修改 `IID` 导入表，恢复 `INT` 表，还原 `IAT` 表中数据，瘦身等，这些是 `PE` 文件基本知识，不涉及破解知识，不在此讨论。

谢谢你耐心地读完本文！希望能对你有所帮助。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 09 月 22 日 23:10:50

# 附件：改进版断点设置脚本

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || temp.rar [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=73257`](http://bbs.pediy.com/showthread.php?t=73257)] |

附件中的脚本 1 是个手动版，现已经改进为自动版了，即运行后就出现 6 个准确断点，脚本如下：

```
//Themida 1.9.1.0 的断点设置脚本（改进版）

data:

            var memory
            var csize
            var tmp
            var temp
    var tmpbp
    var mem
    var mem0
            var mem1
    var mem2
    var mem3
     var mem4
    var mem5
    var Str0
    var Str3
    var Str4
    var Str5
    var Str1
            var Str2
star:
            gmi eip,CODEBASE
            mov  memory,$RESULT
            gmi eip,CODESIZE
    mov csize,$RESULT
            add csize,memory

    find memory,#0000000000000000000000000000000000000000000000#
    mov tmp,$RESULT
    bphws tmp,"w"

        esto
    esto
    esto
    bphwc tmp
    sto
    find memory,#0000000000000000000000000000000000000000000000#
    mov tmp,$RESULT

            sub csize,tmp
            find memory,#909090909090909090909090909090909090#
    mov temp,$RESULT+8
    bphws temp,"w"
    mov  tmpbp,eip
    run
    cmp  eip,tmpbp+2
            jnz  next
            run
    next:
    bphwc
            mov  Str0,"断点地址："
            add  Str0,#0D0A#
    mov  mem,eip
    mov  mem4,eip          //eip=Addr_4
    itoa eip
    mov  Str4,"Addr_4="
            add  Str4,$RESULT
            add  Str4,#0A0D#
```



```
find mem4,#83BD????????000F8497000000#  

mov  mem5,$RESULT       //Addr_5  

itoa mem5  

mov  Str5,"Addr_5="  

add  Str5,$RESULT  

sub  mem,4000  

find mem,#83BD????????640F82??020000#  

mov  mem0,$RESULT+7    //Addr_0  

itoa mem0  

add  Str0,"Addr_0="  

add  Str0,$RESULT  

add  Str0,#0D0A#  

find mem0,#C1C003#  

find $RESULT,#0385????7409#,40  

mov  mem3,$RESULT+6   //Addr_3  

itoa mem3  

mov  Str3,"Addr_3="  

add  Str3,$RESULT  

add  Str3,#0D0A#  

findon:  

find mem0,#AD#       //从 Addr_0 开始查找  

cmp  $RESULT,0  

jz   stop  

mov  mem0,$RESULT  

find mem0,#01C8#,40  

cmp  $RESULT,0    

jnz  findoff  

add  mem0,40  

cmp  mem0,mem5  

ja   stop            //超过 Addr_5 后停止  

jmp  findon  

findoff:  

bp   $RESULT  

add  mem0,40  

itoa $RESULT  

jmp  findon  

stop:  

run  

@1:  

sti  

gn   eax  

cmp  $RESULT,0  

jz  @1     

itoa eip  

mov  Str1,"Addr_1="  

add  Str1,$RESULT  

add  Str1,#0D0A#  

bpwm tmp,csize  

run  

@3:  

sti  

cmp \[eax\],ecx  

jnz @3  

itoa eip  

mov  Str2,"Addr_2="  

add  Str2,$RESULT  

add  Str2,#0A0D#  

add  Str0,Str1  

add  Str0,Str2  

add  Str0,Str3  

add  Str0,Str4  

add  Str0,Str5  

msg  Str0  

pause 
```

附件中的脚本 1 是个手动版，现已经改进为自动版了，即运行后就出现 6 个准确断点，脚本如下：  

//Themida 1.9.1.0 的断点设置脚本（改进版）  

```
data:  

var memory  

var csize  

var tmp  

var temp  

var tmpbp  

var mem  

var mem0  

var mem1  

var mem2  

var mem3  

var mem4  

var mem5  

var Str0  

var Str3  

var Str4  

var Str5  

var Str1  

var Str2  

star:  

gmi eip,CODEBASE  

mov  memory,$RESULT  

gmi eip,CODESIZE  

mov csize,$RESULT  

add csize,memory   

find memory,#0000000000000000000000000000000000000000000000#  

mov tmp,$RESULT  

bphws tmp,"w"  

esto  

esto  

esto  

bphwc tmp  

sto  

find memory,#0000000000000000000000000000000000000000000000#  

mov tmp,$RESULT  

sub csize,tmp  

find memory,#909090909090909090909090909090909090#  

mov temp,$RESULT+8  

bphws temp,"w"  

mov  tmpbp,eip  

run  

cmp  eip,tmpbp+2  

jnz  next  

run  

next:  

bphwc  

mov  Str0,"断点地址："  

add  Str0,#0D0A#  

mov  mem,eip  

mov  mem4,eip           //eip=Addr_4  

itoa eip  

mov  Str4,"Addr_4="  

add  Str4,$RESULT  

add  Str4,#0A0D#  

find mem4,#83BD????????000F8497000000#  

mov  mem5,$RESULT       //Addr_5  

itoa mem5  

mov  Str5,"Addr_5="  

add  Str5,$RESULT  

sub  mem,4000  

find mem,#83BD????????640F82??020000#  

mov  mem0,$RESULT+7    //Addr_0  

itoa mem0  

add  Str0,"Addr_0="  

add  Str0,$RESULT  

add  Str0,#0D0A#  

find mem0,#C1C003#  

find $RESULT,#0385????7409#,40  

mov  mem3,$RESULT+6   //Addr_3  

itoa mem3  

mov  Str3,"Addr_3="  

add  Str3,$RESULT  

add  Str3,#0D0A#  

findon:  

find mem0,#AD#       //从 Addr_0 开始查找  

cmp  $RESULT,0  

jz   stop  

mov  mem0,$RESULT  

find mem0,#01C8#,40  

cmp  $RESULT,0    

jnz  findoff  

add  mem0,40  

cmp  mem0,mem5  

ja   stop            //超过 Addr_5 后停止  

jmp  findon  

findoff:  

bp   $RESULT  

add  mem0,40  

itoa $RESULT  

jmp  findon  

stop:  

run  

@1:  

sti  

gn   eax  

cmp  $RESULT,0  

jz  @1     

itoa eip  

mov  Str1,"Addr_1="  

add  Str1,$RESULT  

add  Str1,#0D0A#  

bpwm tmp,csize  

run  

@3:  

sti  

cmp \[eax\],ecx  

jnz @3  

itoa eip  

mov  Str2,"Addr_2="  

add  Str2,$RESULT  

add  Str2,#0A0D#  

add  Str0,Str1  

add  Str0,Str2  

add  Str0,Str3  

add  Str0,Str4  

add  Str0,Str5  

msg  Str0  

pause
```
