
`intdos(&inregs, &outregs)`;

`printf("\n This program PSP segment is %xh\n", outregs.x.bx)`;

{
  ...
}

### 五、一个简单的例子

这里我们一起看一下经 `BITSHELL` 加密后的程序的情况。

采用一个极简单的例子，用 `DEBUG` 生成一个 `TEST.COM` 文件：

```text
C:\>DEBUG
-A
876B:0100 MOV AL,0
876B:0102 MOV DX,260
876B:0105 OUT DX,AL
876B:0106 INT 20
876B:0108
-RCX
CX 0000
:8
-N TEST.COM
-W
Writing 00008 bytes
-Q
```

用 `BITSHELL` 对 `TEST.COM` 进行加密：

```text
C:\>BITSHELL TEST.COM
BITSHELL(tm) Version 2.01
Copyrighe(c) 1989-1995 by Yellow Rose SoftWare Workgroup
Scheme 6 (CodeKey shift repeated)
9K
Total encryption size is 9K
Encryption completed successfully.
```

再用 `DEBUG` 来看一下加密后的 `TEST.COM`：

```text
C:\>DEBUG TEST.COM
-R
AX=0000 BX=0000 CX=4CBC DX=0000 SP=00C0 BP=0000 SI=0000 DI=0000
DS=877C ES=877C SS=878C CS=878C IP=021C NV UP EI PL NZ NA PO NC
878C:021C FA        CLI
-U
878C:021C FA        CLI
878C:021D 8CCC      MOV SP,CS
878C:021F 8ED4      MOV SS,SP
878C:0221 BC4D25    MOV SP,254D
878C:0224 9C        PUSHF
878C:0225 51        PUSH CX
878C:0226 52        PUSH DX
878C:0227 56        PUSH SI
878C:0228 57        PUSH DI
878C:0229 55        PUSH BP
878C:022A 0E        PUSH CS
878C:022B 1F        POP DS
878C:022C 2E        CS:
878C:022D 8C061000  MOV [0010],ES
878C:0231 2E        CS:
878C:0232 C606480080 MOV BYTE PTR [0048],80
878C:0237 0E        PUSH CS
878C:0238 07        POP ES
878C:0239 BEAD24    MOV SI,24AD
-U
...
-Q
```

可以看到经加密后 `TEST.COM` 文件变长了很多，这是因为在程序外面加了一层外壳。同时，您也发现了加密后的程序没法反编译；如果您再跟踪一下，您又会发现加密后的程序已经无法跟踪了。如果您手头有 `Turbo Debugger` 或 `Soft-ICE` 等等，它们也是无效的。

看起来，这是一个比较好的加密工具，尤其是它提供的函数可以嵌入源代码中，跟软件狗配合起来，达到内外结合的加密效果，大家不妨一试。

虽然加密方法那么多，但是道高一尺，魔高一丈，随着加密技术的发展，解密技术也进一步蓬勃发展起来，目前可用于解密的工具有 `SOURCE`、`DEBUG`、`SYMDEB`、`TURBO DEBUG`、`SOFT-ICE`、`TRW2000` 等等。这些工具的功能一个比一个强，更何况还有那么多“专业”解密高手，似乎任何加密技术到了他们手里都会迎刃而解。说来说去，您可能会想，你仍然没有最终解决软件的加密问题吗。是的，本来吗，加密和解密就是一对矛盾的统一体，某一阶段的优劣只能看是哪一方先亮出了新招，谁都不敢说他的加密方法别人破译不了，或者说他能破译任何加密方法。如此看来，我们所做的工作只在于尽量减少被解密的可能性，以赢得时间进行下一步的开发。

软件解密就是要想办法把加密代码攻破，具体到软件狗上，即是找到检查程序，然后干掉它。因此，如果我们在应用程序的不同地方多做几次检查，就可以让解密者浪费更多的时间。这对 I/O 速率很快的软件狗来说是很方便的，不像软盘加密那样检查起来既费时又不方便。

一个好的程序员会把程序设计得简单易懂，具有结构化，但这也给解密者带来了方便，所以程序不要写得太规则，另外还要加上一些“废话”，以干扰解密者。

另外一种不破坏程序结构化，也不需要写“废话”就可以有效干扰解密者，提高解密难度和复杂性的方法是，把子程序全部用宏改写。这样汇编出的程序是一串串很难看出结构的指令，进出堆栈的数据和各种传递的参数以及累加器暂存数的存取相距很远，嵌套很深，分析起来很伤脑筋。第三代软件狗的存取程序就是全部用这种方法写的。

很多软件加密技术的研制者对 `Soft-ICE` 很头疼，因为它的解密功能太强了，但是您只要在程序中调用一下 `INT 7`，就能防止它的跟踪，您不妨试试看。

在用解密工具跟踪程序时，执行时间显然要比平常的长，另外一般都用键盘来操作，用显示器或打印机作输出，所以采用执行时间检查或者在关键程序部分禁止键盘中断，以及禁止显示器或打印机输出（修改 `INT 10H` 或 `INT 17H` 中断）等都是较好的方法。例如在第三代软件狗编程中用到的方法：

```asm
...
...

;...
;  计时反跟踪  ...
;...
;---[+]---
;保存时间
;---[+]---
TimeSave    macro
    ifndef debug_Time
        push ds
        push cs:[_const_word_0_]
        pop ds
        push ds:[046ch]    ;把时钟计数数值
        pop cs:start_time   ;保存到 start_time
        pop ds
    endif
    endm

;
;---[+]---
;检查程序执行时间，超过 5 秒则主机被挂起
;---[+]---
TimeTest    macro
    ifndef debug_Time
        push es
        push ax
        push cs:[_const_word_0_]
        pop es
        mov ax,es:[46ch]   ;得到新的时间计数数值
        sub ax,cs:start_time
        cmp ax,18*5         ;执行时间超过 5 秒吗？
        ja $+4              ;是，则主机被挂起
        pop ax
        pop es
    endif
    endm

;
;...
;  键盘反跟踪  ...
;...

;---[+]---
;禁止键盘中断
;---[+]---
KbdOff    macro
    ifndef debug_Kbd
        mov cs:_tmpB_,al    ;保存 al
        mov al,02h          ;禁止键盘中断
        out 21h,al
        mov al,cs:_tmpB_    ;恢复 al
    endif
    endm

;
;---[+]---
;
;检查键盘中断，若被开放则主机被挂起
;---[+]---
KbdTest_jmp macro
    ifndef debug_Kbd
        push ax
        in al,21h
        test al,02h         ;键盘中断被开放？
        jz $+3              ;是，则挂起
        pop ax
    endif
    endm

...
...
```

另外，如果我们把关键部分的程序加以编码，运行时再译码出来，这也增加了解密的难度，因为解密者必须懂得编码、译码规则才能修改可执行文件，而这些规则加密者一般是不会泄露的。

多种软件加密方法的综合，以及几种加密技术交叉使用，足以让许多解密者知难而退。因为必须了解所有的加密技术原理才有可能解密，这就增加了难度，毕竟能够精通各种加密方法的人不多。例如有就把软件狗加密技术和磁盘加密技术结合起来做。

## 二、硬件加密

软件狗的电路不要做得太直接，否则不利于加密。这里我们就来讲讲如何增进软件狗的复杂性，加强硬件电路的机密功能。

在讲第二代软件狗时，我们说它其实就是几个简单的与或逻辑门，目前已经被淘汰。现在我们将要把它跟 `EEPROM` 结合起来，经过变化，组成比较复杂的第三代软件狗，从而增加解密难度。

`CAL` 芯片由于本身就有加密功能，所以也常被用来与 `EEPROM` 结合做成复杂的软件狗，更由于其内部逻辑配置的灵活性，使得解密的难度上涨了很多倍。

解密者要仿制加密盒，首先必须搞到同型号的 `EEPROM`，所以，我们尽量采用市场上一般购买不到的 `EEPROM` 型号，这样也能起到保密的作用。另外，如果采用特殊的 `93CX6` 系列，就可以利用不同厂商出品的特殊 `93CX6`，其特性不同这一点来加强硬件的加密功能。

下面我们给出两个具体例子来说明（以 `93C46` 型为例）

**例 1** 在电路中加入一个非门，以改变 `93C46` 某一管脚的极性，电路中要尽量采用低功耗的芯片，所以这里用了 `CMOS` 型的 `74HC04` 非门来实现。

这是在前面例子的基础上把 `CS` 信号反了相，读者还可以适当加入一些无用的电阻，电容等器件，以迷惑解密者。同时，我们还需要把相应的程序修改一下，把原来对 `CS` 信号的控制也反相才行。

```asm
...
...

;---[+]---
;设置 EEPROM 的片选 CS 为高电平，即选中 EEPROM(-AUTO FEED 线反相
;后接 CS)
;硬件电路变化时，本部分程序应作相应改动
;---[+]---
SetCS     macro
    push ax
    CtrlPort
    in al,dx
    or al,02h       ;令 -AUTO FEED 线输出为低电平
    KbdTest_Add_AL  ;检查键盘中断，若被开放，则改变 al 值
    out dx,al
    _CtrlPort       ;选择数据口
    pop ax
    endm

;---[+]---
;设置 EEPROM 的片选 CS 为低电平，即不选中 EEPROM（-AUTO FEED 线反相
;后接 CS)
;硬件电路改变时，本部分程序应作相应改动
;---[+]---
ClearCS   macro
    push ax
    CtrlPort
    in al,dx
    and al,not 02h  ;令 -AUTO FEED 线输出为高电平
    KbdTest_Add_0w  ;检查键盘中断，若被开放，则改变 _const_word_0_ 值
    out dx,al
    _CtrlPort       ;选择数据口
    pop ax
    endm

...
...
```

**例 2** 在电路中加入一个或非门，使得必须用两根并行口的信号线来得到一根 `93CX` 的控制线，这里采用了 `COMS` 型的 `74HC02` 或非门。

这是在前面例子的基础上把 `CS` 信号用两根口线(`D2` 和 `D3`)来控制，还可以适当加入一些无用的电阻、电容等器件，以迷惑解密者。同时，我们还需要把相应的程序修改成只有当 `D2` 和 `D3` 都为低电平时 `CS` 才为高电平。

```asm
...
...

;---[+]---
;设置 EEPROM 的片选 CS 为高电平，即选中 EEPROM（D2D3 线或非后接 CS)
;硬件电路改变时，本部分程序应作相应改动
;---[+]---
SetCS     macro
    push ax
    in al,dx        ;取数据口数据
    and al,not 0ch  ;令 D2D3 线都输出低电平
```
