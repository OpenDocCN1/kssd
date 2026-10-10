
`mov esp,TopOfStack`                 ;当然堆栈段也是段的选择子咯

;-----------显示缓冲--------

`mov ah, 0Ch`    ; 0000: 黑底    1100: 红字
`mov esi,OffsetPMMessage`                          ;这个是字符串相对于它的段偏移值
`mov edi,(80*10+0)*2`     ;第 10 行
`cld`
.1:
`lodsb`
`test al,al`
`jz   .2`
`mov [gs:edi],ax`       ;要用 ax 做为参数传进缓冲区
`add edi,2`
`jmp .1`
.2:                                      ;OffsetPMMessage   字符串显示完成:
;------------------------测试 5MB 空间的读------------------

`call DispReturn`    ;显示回车，也就是改变 edi 的位置(edi 是显缓冲区段的偏移值)
`call ReadTest`
`call WriteTest`
`call ReadTest`     ;当在执行 Call 命令的时候     会将 eip + 1 压栈,然后跳转
`jmp SelectorCode16:0`      ;跳到最后那个 16 位代码段去了 前面有 16 位段描述符的讲解。。

;--------------显示回车---------------
DispReturn:
`push ebx`                         ;临时用 ebx eax 所以先保存一下
`push eax`
`mov bl,160`
`mov eax,edi`
`div bl`
`inc eax`   ;得到回车后的行数
`mov bl,160`
`mul bl`
`mov edi,eax` ;取得当前的位置
`pop eax`
`pop ebx`
`ret`      ;ret 指令会恢复 eip + 1

;--------------读取我们定义的大地址段---------------
ReadTest:
`xor  esi,esi`
`mov  ecx,11`
`mov  ah, 0Ch`      ; 0000: 黑底    1100: 红字
.loop:
`mov  al,[es:esi]`          ;这个是 SelectorTest 的段,也就是我们测试的大地址段
`call  DispAL`                ;显示从 test 段取出来的字符
`inc  esi`
`loop  .loop`               ;cx--
`call    DispReturn`
`ret`

;--------------写入我们定义的大地址段---------------
WriteTest:
`push esi`                         ;借这两个寄存器来传字符串
`push edi`
`xor esi, esi`
`xor edi, edi`
`mov esi, OffsetStrTest` ; 数据段的字符串
`cld`
.1:
`lodsb`
`test al, al`
`jz .2`
`mov [es:edi], al`    ;把数据段的字符串传给测试段
`inc edi`
`jmp .1`
.2:
`pop edi`
`pop esi`
`ret`

;--------------显示 AL 的内容---------------
DispAL:                         ;对传来的 Al 字符进行处理
`test al,al`
`jnz   .next`
`mov al,'0'`
.next:
`mov ah, 0Ch`    ; 0000: 黑底    1100: 红字
`mov [gs:edi],ax`
`add edi,2`
`ret`

`SegCode32Len equ $ - LABEL_SEG_CODE32`

;-----------------------------------准备 8086 模式的 16 位段------------------------------------
`[SECTION .code16]`
`align 32`
`[BITS 16]`

`LABEL_SEG_CODE16:`
`mov ax,SelectorNormal`       ; 保护 8086 标准段属性的描述符选择子
`mov ds, ax`
`mov es, ax`
`mov fs, ax`
`mov gs, ax`
`mov ss, ax`       ;使所有的段选择子都达到 8086 标准
`mov eax,cr0`
`and al,0feh`
`mov cr0,eax`    ;CR0 PE 位为 0 回到 Real mode
`LABEL_GO_BACK_TO_REAL:`
`jmp 0:LABEL_REAL_ENTRY`            ;还记得上面的[LABEL_GO..]+3 吗，指的就是 0 这个段值
`Code16Len equ $ - LABEL_SEG_CODE16`

呵呵、那么就完工了代码虽然比较繁杂，但是还不是特别复杂：

有几个要注意的地方:

1、堆栈(新加了堆栈这个段，其寻址方式也是选择子。)

在入口的时候是实模式,应该把当前的 sp 值保存下来,因为在返回系统时我们需要基本恢复原来的样子。

就在保护模式的全局数据段定义一个变量用来保存 SP。在返回 DOS 时候要记得恢复。

2、保护模式到实模式

实模式到保护模式 比较简单就几个固定步骤不需要考虑太多，而从保护模式到实模式就比较复杂：

首先需要定义两个 8086 标准属性的段描述符.它们分别是 Normal 数据段与 16 位代码段描述符。

在 16 位代码段中 cs 的属性跟 8086 模式的 CS 段界限与段属性一致，并且把 Normal 的段属性分别付给全部的数据段寄存器.这样就让所有保护模式的数据段寄存器全部跟 8086 段界限与段属性一致！！！

在 cr0 的 PE 位为 0 后 就可以转换成实模式寻址了!!!

接下来继续学习:

引用: 补充一下.com 要在纯 DOS 下运行，，，因为用到了很对特权指令!!!

Day Day Up

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 5MB.rar  |

# 【LDT 描述符&&特权级&&门】

## 一、LDT（局部描述符）

GDT 是全局描述符，是整个系统的描述符，描述符着所有的段！！！

在前几章我们已经熟悉了 GDT 的一些基本功能，与运作机制。

对 GDT 描述符的定义与使用也就那么几项固定的步骤，接下来再了解 LDT.

LDT 是局部描述符。看字面 LDT 与 GDT 很相似.它们都是描述符。

只不过 GDT 是全局描述符、而 LDT 是局部描述符。

那么 LDT 该如何【定义】与【使用】呢？(与 GDT 非常类似，不过 LDT 是归属于 GDT 的)：

### @【定义】：

1、在 GDT 的描述集合中插入一条 `LABEL_DESC_LDT` Descriptor `0,LdtLen - 1,DA_LDT`
（这个段描述符描述的 LDT 数据段：是一个局部描述符段的集合,结构类似 GDT.）

2、在 GDT 选择子集合中照样也插入一条 `SelectorLdt equ ;LABEL_DESC_LDT - LABEL_GDT`
（看似跟普通的段描述符没啥两样哦,不过接下来就有点小区别了!）

3、建一个 LDT 数据段,这个数据段里的结构与 GDT 描述符段类似!

```
LABEL_LDT:         ;这里与 GDT 不同,LDT 的段描述符一开始就是实际的段
LABEL_DESC_LDT_CODEA:    Descriptor 0,LdtCodeALen - 1,DA_C | DA_32
LABEL_DESC_LDT_DATA:     Descriptor 0,LdtDataLen - 1,DA_DRW
LdtLen equ $ - LABEL_LDT;
;在加载 LDT 的时候并不是靠一个绝对结构体，而是通过 GDT 这个桥梁索引加载的。
;LDT 内部段的选择子
SelectorLdtCodeA   equ   LABEL_DESC_LDT_CODEA - LABEL_LDT | SA_TIL
SelectorLdtData   equ   LABEL_DESC_LDT_DATA - LABEL_LDT | SA_TIL
```

`;LDT 内部的选择子多了一个 SA_TIL 属性，他是段选择的 TI 位也就是第 2 位 如果是 1 代表当前选择子代表的段是 LDT 的内部段。 因为选择子的 0-2 位并不参与索引。所以选择子的索引都是 8 的倍数或 0，`

4、接着完成 LDT 里面段描述符描述的段。这里只定义了一个代码段: `LABEL_DESC_LDT_CODEA`
(这个跟普通的代码段一样，有执行、32 位等属性. 为了测试 LDT 的效果那么我们还是加一个 Data 段吧)

那么 LDT 的基本定义已经完成了！！！

### @【使用】：

1、LDT 描述符与其内部的局部段描述符信息、，我们在进入保护模式前初始化!!!

那么我们的进入保护模式之前的段在哪还知道吗？当然就是最最开始了！在 16 位段实模式入口那里!!!

我们在这里做完 GDT 描述符的初始化工作后：

`;--------------初始化 LDT 描述符---------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT`
`mov WORD [LABEL_DESC_LDT + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT + 4],al`
`mov BYTE [LABEL_DESC_LDT + 7],ah`

`;---------------初始化 LDT 内部的数据段描述符----------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT_DATA`
`mov WORD [LABEL_DESC_LDT_DATA + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT_DATA + 4],al`
`mov BYTE [LABEL_DESC_LDT_DATA + 7],ah`

`;---------------初始化 LDT 内部的代码段描述符----------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT_CODEA`
`mov WORD [LABEL_DESC_LDT_CODEA + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT_CODEA + 4],al`
`mov BYTE [LABEL_DESC_LDT_CODEA + 7],ah`

上面几个初始化段基地址的代码相信大家已经很熟悉了!!!!!!!!

2、 先前就说过了 LDT 跟 GDT 类似都是描述符集合，那么他们在使用起来也有点类似!!!

需要用 `lldt` 指令：它的参数并不是一个结构体的指针,因为 LDT 是属于 GDT 的。不能独立的加载。必需依靠 GDT 这个桥梁。那么这个参数也就是 LDT 在 GDT 当中的索引值.,

在进入保护模式后我们添加如下代码进行 LDT 的加载：

`mov ax,SelectorLdt`
`lldt ax`      ;lldt 加载时是从 GDT 当中的 ax 索引位置的段做 LDT 段加载到 LDTR 寄存器的.\\
`jmp SelectorLdtCodeA:0` ;首先检查选择子的 TI 位，当是 LDT 选择子的时候。马上去 LDTR 寄存器取 LDT 在 GDT 当中的索引，找到 LDT 的位置。最后根据 SelectorLdtCodeA 在 LDT 局部描述符集合中查找。

## 二、特权级：

我们在 WIindows 里使用 Ring3 程序也好、Ring0 函数也好，在执行他们的时候或者调用他们的时候是按一定规则执行的? 当然 Windows 自己包装的规则不在讨论之类,.我们现在需要了解 CPU 最铁最硬的规则!!!

Ring0 -Ring3 并不是属于 WINDOWS 的东西，而是属于 CPU 本身的!!! 那么当我们在使用程序的时候或者模块的时候 cpu 是如何鉴别程序特权级的呢?

1、`CPL`（Current Privilege Level）

`CPL` 是指当前执行的程序或者任务的特权级，它存储在 CS 和 SS 的 0 到 1 位。CS、SS 在这里都是段选择子了, (第 2 位是 TI 位代表 LDT），这 0-1 位合起来可以表示 Ring0-Ring3 个级别.

通常情况下 CPL 等于代码本身所在段的特权级(DPL), 当程序跳转到不同特权级的代码段时,处理器将改变 CPL, 以上指得是非一致代码段。 如果是一致代码段，一致代码段可以被同等级的或者低特权级访问，并且跳转过去后 CPL 也不会改变。

2、`DPL`(Descriptor Privilege Level)

`DPL` 表示段或者门的特权级. 门其实也是一个段描述符来描述的。所以他们的 DPL 值就是段描述符属性里面的 DPL 位。下面来列举一些访问这些段时所需要的 CPL 值。

×数据段 ：

一个代码段要想访问这个数据段，那么必需 `CPL`(代码段) <= `DPL`(数据段)。

这样代码段才能正常访问此数据段。

×非一致代码段 ：(不使用调用门的情况下)

访问非一致代码段的条件是 `CPL == DPL`(非一致代码段)

×调用门 ：

访问调用门的条件: 跟数据一样 `CPL`(代码段) <= `DPL`(调用门段）

×一致代码段和通过调用门访问的非一致代码段：

访问一致代码段的条件： 必需 `CPL >= DPL`

通过调用门访问非一致代码段条件: 必需 `CPL >= DPL`

×TSS(任务状态段)：

`Tss` 是一个 32 位的结构体，它包含了 100 个字段。其中偏移量为 4 -27 里有 3 个 ss 和 3 个 esp 信息。它们记录着不同特权级任务切换时的堆栈信息。 由于只有特权级低到高切换任务时，新堆栈才会从 TSS 中取得。所以 TSS 里没有 Ring3 的 SS ESP 信息。

访问 TSS 的条件： 跟数据一样 `CPL`(代码段) <= `DPL`(TSS）

3、`RPL`(Request Privilege Level)：

它存放在段选择子的 0-1 这两位上面。 在访问目标段时 CPU 会做一系列检查。

其中 RPL 是访问目标段的关键:如果当前代码段 CPL 为 0，要访问对方数据段的 DPL 值是 1,而 RPL 值却是 2。那么访问是不成功的。数据段的 RPL 值一定要<=DPL 才能正常访问。。
