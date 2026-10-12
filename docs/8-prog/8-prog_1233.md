
# 我的学习的脚印 2011 年 24 周 花指令/调用门/R3<-->R0 切换

> 原文：[`www.pediy.com/kssd/pediy12/135375.html`](https://www.pediy.com/kssd/pediy12/135375.html)

## 声明

内容都是书中或 `baidu` 的，很多在论坛都有出现，只是觉得有阅读和收藏整理以方便自己使用的必要。另外就是方便自己以后回头看看自己曾经走过的坎坷之路，仅此而已。

非常颓废的一个周末

`2011.6.12` 电视消灭 30 集，OY！

迷糊中突然想到花指令，之前看过但从来没用过，也忘记了目的。

只知道是在代码中插入数据来混淆反汇编引擎解析的代码。反汇编多数都是顺序解析的，在指令中插入其他数据后，导致将数据当 `Opcode` 解析导致后面的不对齐。

`_asm _emit 0xEB` 编译后数据就混在代码里，执行是怎么分辨呢？(唉，自己想多了)

其实是类似如下的方式，花指令是不可达的。(学了不用==没学)

### 代码

```asm
_asm
{ 
JMP MyCode
  _emit xxx
MyCode:
}
```

## 调用门

`2011.6.13`

对应的结构体是：

### 代码

```c
    typedef struct
    {
        unsigned short  offset_0_15;        //偏移
        unsigned short  selector;      //选择子，此门要调用哪个代码段的对应选择子
        unsigned char    param_count : 4;
        unsigned char    some_bits   : 4;         
unsigned char    type        : 4;
        unsigned char    app_system  : 1;
        unsigned char    dpl         : 2;
        unsigned char    present     : 1;
        unsigned short  offset_16_31;
    } CALLGATE_DESCRIPTOR; 
```

`call SelectorCode32:0`

`call SelectorGate:0`    ;门的选择子为 `Selector32`

则他们实质是调用同一块代码，实现跳转，主要用来完成低权限代码调用搞权限代码操作。

调用门可以理解为其他代码段描述符的经纪人，代码段都很拽，我们没发直接和他进行操作，但是通过它的代理人 `Gate` 来传达我们的需求。

下面的代码主要实现的功能为：在 `3E0` 处构造一个调用门调用 `3E8` 处描述符对应的代码，`3E8` 描述符的代码范围：`0-0xFFFF`，从内存的 `0` 执行，在 `0` 地址是 `retn` 指令

- `0` : `retn`
- `3E0` 调用门 `Select 3E8`
- `3E8` `BaseLimit`   `0-0xFFFF`

### 别人给的学习代码

```asm
.586p
.model flat,stdcall
option casemap:none

CALLGATE struct
  OffsetL dw ?
  Selector dw ?
  DCount db ?
  Gtype db ?
  OffsetH dw ?
CALLGATE ends

DESCRIPTOR struct
  LimitL dw ?
  BaseL dw ?
  BaseM db ?
  Attributes dw ?
  BaseH db ?
DESCRIPTOR ends

CODES_SEL equ 3e8h
CODE_TYPE equ 0cf9ah
GATE_TYPE equ 0ech

.code

DriverEntry proc pDriverObject,pRegisterPath
  pushfd
  pushad

  push eax
  sgdt [esp-2]    ;获取 GDTR，edx 是描述符的 BaseAddr
  pop edx
  mov eax,edx
  mov byte ptr[edx],0c3h  ;retn 的机器码
  mov ecx,3e0h            ;eE0/8 = 7C
  add edx,ecx
  assume edx:ptr CALLGATE
  cmp [edx].Selector,CODES_SEL;判断是否已处理
  jz _end

  mov [edx].OffsetL,ax
  mov [edx].Selector,CODES_SEL
  mov [edx].DCount,0
  mov [edx].Gtype,GATE_TYPE   ;这个调用门来调用下面段描述符对应的代码
  shr eax,16
  mov [edx].OffsetH,ax

  add edx,8
  assume edx:ptr DESCRIPTOR
  mov [edx].LimitL,0ffffh
  mov [edx].BaseL,0
  mov [edx].BaseM,0
  mov [edx].Attributes,CODE_TYPE
  mov [edx].BaseH,0
_end:
  popad
  popfd
  mov eax,0c0000182h  ;错误代码
  ret
DriverEntry endp
end DriverEntry
```

[`bbs.pediy.com/showthread.php?t=62263`](http://bbs.pediy.com/showthread.php?t=62263) 关于调用门的讲解这里引用下方便以后查询

### 代码

```text
！PCR   （Processor Control Region）
kd> !pcr
KPCR for Processor 0 at ffdff000:
    Major 1 Minor 1
  NtTib.ExceptionList: b276d654
      NtTib.StackBase: b276ddf0
      NtTib.StackLimit: b276b000
   NtTib.SubSystemTib: 00000000
        NtTib.Version: 00000000
    NtTib.UserPointer: 00000000
        NtTib.SelfTib: 7ff7d000
              SelfPcr: ffdff000
                Prcb: ffdff120
                Irql: 00000000
                  IRR: 00000000
                  IDR: ffffffff
        InterruptMode: 00000000
                  IDT: 8003f400
                  GDT: 8003f000    //GDT 地址
                  TSS: 80042000
        CurrentThread: 8267cc30
           NextThread: 00000000
           IdleThread: 80552920
kd> r gdtr
gdtr=8003f000
kd> dw /c4 8003f000
8003f000  0000 0000 0000 0000
8003f008  ffff 0000 9a00 00cf
8003f010  ffff 0000 9200 00cf

选择子 8 的描述符为 00 CF9A  000000  FFFF
段基址为：0
CF9A  ==>  1100  1111  1001  1010 
段界限为 0xFFFFF
DPL：0
P：1
G：1
S：1 数据描述符,0 是系统描述符
TYPE：1010     执行/读
```

### 关键代码

```asm
  push esi
  sgdt [esp-2]
  pop esi
  
  mov eax,8
  .while eax < GDT_LIMIT    ; GDT_LIMIT :03ffh
       lea edx,[esi+eax]
       assume edx:ptr CALLGATE
       test [edx].GTYPE,80h    ;通过判断 present 是否为 1 来判断是否是空闲
      .if ZERO?
          mov ebx,FuncAddr
          mov [edx].OFFSETL,bx
          mov [edx].SELECTOR,08h
          mov [edx].DCOUNT,0
          mov [edx].GTYPE,GATE_TYPE
          shr ebx,16
          mov [edx].OFFSETH,bx     
          .break
      .endif    
       assume edx:nothing
       add eax,8
  .endw
```

目标是一致代码段   `CPL <= DPL(门), RPL <= DPL(门) DPL(目标) <= CPL`

非一致 `CALL`    `CPL <= DPL(门)， RPL<= DPL(门), DPL(目标) <= CPL`

     `JMP`      `CPL <= DPL(门)， RPL <= DPL(门)， DPL(目标) = CPL`

## R3->R0

`2011.6.14`

摘自一个操作系统的实现

1. 根据目标代码段的 `DPL`(新的 `CPL`)从 `TSS` 中选择相应的要切换的 `ss` 和 `esp`(切到 `R1`,就需要 `ss1` 和 `esp1`)
2. 从 `TSS` 中读取新的 `ss` 和 `esp`，在这过程中如发现 `ss/esp` 或者 `TSS` 界限错误都会导致无效 `TSS` 异常(`#TS`)
3. 对 `SS` 描述符进行检验，如果哦发生错误，同样产生`#TS` 异常
4. 暂时性保存当前的 `ss` 和 `esp` 的值
5. 加载信息的 `ss` 和 `esp`
6. 将刚刚保存的 `ss` 和 `esp` 值压栈
7. 从调用者堆栈中将参数复制到被调用者堆栈(当前的新堆栈)，复制参数数目由调用门中的 `Param Count` 来决定。如果是 `0` 不复制参数，。
8. 将当前 `cs` 和 `eip` 压栈
9. 加载调用门中指定的新的 `cs` 和 `eip`，开始执行被调用者过程

简单的描述主要就是如下操作：

- 根据切换目标是那个层的(只能低跳高)，获取相应的 `ss` 和 `esp` 来构建当前的新栈
- 将之前调用过来的 `ss` 和 `esp` 压栈
- 根据 `Param Count` 来复制参数
- 将调用过来的 `cs` 和 `eip` 压栈 ， 压栈的信息被用来回调
- 这里保存环境的工作完毕就加载调用门指定的段描述符的代码段的代码执行之

`TSS` 中切换层的 `ss` 和 `esp` 构建的新栈，类似正常的函数调用栈，不同的是多保存的原来栈信息，因为这里涉及到栈切换，而函数调用栈是不需要切换的，使用的相同的栈。

```text
    Eip
    
    Cs
    
    参数
    
    esp
    
新栈    ss        高地址  
```

在处理 `TSS` 时，根据调用的目标代码段的 `DPL` 来使用相应的 `ss`(栈段选择子)和 `esp`(`TopStack`)

## R0 -> R3

1. 检查保存的 `cs` 中的 `RPL` 以判断返回时是否需要切换特权级
2. 加载被调用者堆栈上的 `cs` 和 `eip`(此时会进行代码段描述符和选择子类型和特权级检查)
3. 如果 `ret` 指令含有参数，则增加 `esp` 的值跳过参数，然后 `esp` 将执行被保存的调用者 `ss` 和 `esp`。注意，`ret` 的参数必须对应调用门中的 `Param Count` 值。
4. 加载 `ss` 和 `esp`，切换到调用者堆栈，如果被调用的 `ss` 和 `esp` 被丢弃。这里会进行 `ss` 描述符/`esp` 以及 `ss` 段描述符的检查
5. 如果 `ret` 指令含有参数，增加 `esp` 的值以跳过参数(此时已在调用者堆栈)
6. 检查 `ds`，`es`，`fs`，`gs` 的值，如果其中哪一个寄存器指向的段的 `DPL` 小于 `CPL`(不适用于一致代码段)，那么一个空描述符会被加载到该寄存器。

简单描述主要过程是：

- 加载新栈中的调用者的 `cs`，`eip`，`ss`，`esp` 切换到调用者环境中。
- 调用环境和 `R3` 正常的函数调用 `ret` 过程一样。

```text
    Eip                             cs:eip  <== 返回地址
    
    Cs                              参数
    
    参数       ==>  
    
    esp
    
    ss
    
新栈     高地址        调用者栈
```

在练习时出现问题记录下以后要注意：

1. 调用门不需要跟段描述符一样再初始化(段描述符中段寄存器是选择子，各种标识是 offset，注意先后顺序)
