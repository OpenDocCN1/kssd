
```
mov  sp,offset   DGROUP:RTN_ADD
jmp      RTN_NAME

RTN_ADD:  dw      DGROUP:$+2
          ENDM
```

例子：

```
Address    Hex               Mnemonic

F000:6000                    F000_6000_read_pci_byte proc near

F000:6000   66 B8 00 00 00 80   mov   eax, 80000000h
F000:6006   8B C1               mov   ax, cx          ; copy offset addr to ax
F000:6008   24 FC               and   al, 0FCh        ; mask it
F000:600A   BA F8 0C            mov   dx, 0CF8h
F000:600D   66 EF              out   dx, eax
F000:600F   B2 FC               mov   dl, 0FCh
F000:6011   0A D1               or    dl, cl          ; get the byte addr
F000:6013   EC                   in    al, dx          ; read the byte
F000:6014   C3                   retn                  ; Return Near from Procedure
F000:6014                    F000_6000_read_pci_byte endp
......

F000:6043 18 00               GDTR_F000_6043 dw 18h   ; limit of GDTR (3 valid desc entry)
F000:6045 49 60 0F 00         dd 0F6049h              ; GDT physical addr (below)
F000:6049 00 00 00 00 00 00 00 00  dq 0                 ; null descriptor
F000:6051 FF FF 00 00 0F 9F 00 00  dq 9F0F0000FFFFh     ; code descriptor:
                                             ; base addr = F 0000h; limit=FFFFh; DPL=0;
                                             ; exec/ReadOnly, conforming, accessed;
                                             ; granularity=byte; Present; 16-bit segment
F000:6059 FF FF 00 00 00 93 8F 00  dq 8F93000000FFFFh    ; data descriptor:
                                             ; base addr = 00h; seg_limit=F FFFFh; DPL=0;
                                             ; Present; read-write, accessed;
                                             ; granularity = 4 KByte; 16-bit segment
......

F000:619B 0F 01 16 43 60     lgdt  qword ptr GDTR_F000_6043 ; Load Global Descriptor Table Register
F000:61A0 0F 20 C0            mov   eax, cr0
F000:61A3 0C 01               or    al, 1           ; set PMode flag
F000:61A5 0F 22 C0            mov   cr0, eax
F000:61A8 EA AD 61 08 00     jmp   far ptr 8:61ADh ; jmp below in 16-bit PMode (abs addr F 61ADh)
                                             ; (code segment with base addr = F 0000h)
F000:61AD                    ; -----------------------------------------------------
F000:61AD B8 10 00            mov   ax, 10h         ; load ds with valid data descriptor
F000:61B0 8E D8               mov   ds, ax          ; ds = data descriptor (GDT 3rd entry)
......

F000:61BC  B9 6B 00           mov   cx, 6Bh         ; DRAM arbitration control
F000:61BF  BC C5 61           mov   sp, 61C5h
F000:61C2  E9 3B FE           jmp   F000_6000_read_pci_byte ; Jump
F000:61C2                    ; --------------------------------------------------
F000:61C5  C7 61              dw 61C7h
F000:61C7                    ; --------------------------------------------------
F000:61C7  0C 02              or    al, 2           ; enable VC-DRAM
```

你看到的，必需要考虑`retn`指令被当前`ss:sp`寄存器值影响，`ss`寄存器还没有加载到正确的 16-bit 保护模式使用！这些代码怎么会执行？答案有点复杂。让我们看看`ss`寄存器的值，它在上述调用之前就巧妙的处理了。

```
Address    Hex               Mnemonic

F000:E060 8C C8               mov   ax, cs
F000:E062 8E D0               mov   ss, ax          ; ss = cs (ss = F000h a.k.a F_segment)
F000:E064                    assume ss:F000
```

Note: this routine is executed in real-mode

就如你看到的，`ss`寄存器装入了`f000h`（当前 bios 代码 16-bit 段在实模式）。这段代码说明隐藏的描述缓存寄存器（存在为每一个选择/段寄存器）被加载入`ss*16`或`F_0000h`的物理地址值。并且这个值会返回，尽管机器转变成了上述的 16-bit 保护模式，因为`ss`寄存器没有重载。`Intel Software Developer Manual Vol.3` 片断：

### 8.1.4. First Instruction Executed

The first instruction that is fetched and executed following a hardware reset is located at physical address `FFFFFFF0H`. This address is 16 bytes below the processor uppermost physical address. The `EPROM` containing the software-initialization code must be located at this address. The address `FFFFFFF0H` is beyond the 1-MByte addressable range of the processor while in real-address mode. The processor is initialized to this starting address as follows. The `CS` register has two parts: the visible segment selector part and the hidden base address part. In real address mode, the base address is normally formed by shifting the 16-bit segment selector value 4 bits to the left to produce a 20-bit base address. However, during a hardware reset, the segment selector in the `CS` register is loaded with `F000H` and the base address is loaded with `FFFF0000H`. The starting address is thus formed by adding the base address to the value in the `EIP` register (that is, `FFFF0000 + FFF0H = FFFFFFF0H`).

The first time the `CS` register is loaded with a new value after a hardware reset, the processor will follow the normal rule for address translation in real-address mode (that is, `[CS base address = CS segment selector * 16]`). To insure that the base address in the `CS` register remains unchanged until the `EPROM` based software-initialization code is completed, the code must not contain a far jump or far call or allow an interrupt to occur (which would cause the `CS` selector value to be changed).

`Ddj (Doctor Dobbs Journal)`的一个小片断：

At power-up, the descriptor cache registers are loaded with fixed, default values, the CPU is in real mode, and all segments are marked as read/write data segments, including the code segment (`CS`). According to `Intel`, each time the CPU loads a segment register in real mode, the base address is 16 times the segment value, while the access rights and size limit attributes are given fixed, "real-mode compatible" values. This is not true. In fact, only the `CS` descriptor cache access rights get loaded with fixed values each time the segment register is loaded - and even then only when a far jump is encountered. Loading any other segment register in real mode does not change the access rights or the segment size limit attributes stored in the descriptor cache registers. For these segments, the access rights and segment size limit attributes are honored from any previous setting (see `Figure 3`). Thus it is possible to have a four giga-byte, read-only data segment in real mode on the `80386`, but `Intel` will not acknowledge, or support this mode of operation.

现在，你知道重点在于描述缓存寄存器，特别是它的基地址部分。`Ss`可见部分只是一个“place holder”和“register-in-charge”，对于真实地址计算/变换是一个隐藏的描述缓存。无论你对这个描述缓存做什么，当任何代码、栈或者数据值地址被转换计算的时候，它都要受到影响。在我们看来，我们不得不在 16-bit 保护模式使用基址是`F_0000h`的物理地址的“堆栈段”。这不是问题，因为`ss`描述缓存寄存器的基址已经在上面的代码中赋予了`F_0000h`值。这就解释了为什么上面的代码能够正确执行，下面是一个例子：

```
Address    Hex               Mnemonic

F000:61BF  BC C5 61           mov   sp, 61C5h
F000:61C2  E9 3B FE           jmp   F000_6000_read_pci_byte ; Jump
F000:61C2                    ; --------------------------------------------------
F000:61C5  C7 61              dw 61C7h
```

这段代码里面我们已经给`ss:sp`指向`F_61C5h`，为`retn`指令服务。实际上，我们已经做了，因为`ss`包含了`F_0000h`（它的描述缓存基址部分）和你看到（`sp contains 61C5h`）的物理地址，`ss:sp`是`F_0000h+61C5h`，物理地址是`F_61C5h`。

## 5. 用到的工具

本节介绍逆向工程分析所需的工具。将有一节单独解释`IDA Pro`反汇编工具。

### 5.1. 我们的需求

开始进行之前，我们需要以下工具：

1、`IDA Pro`反汇编工具。我使用`IDA Pro V4.3`。你可以使用你喜欢的交互式反汇编工具。我觉得`IDA Pro`最适合我。我们之所以需要交互式反汇编工具，因为我们要反汇编的`BIOS`代码并不是普通的代码。当驻留在`ROM`中执行的一些时候并没有栈可用，而是使用了一些栈的技巧来进行过程/例程调用。

2、一个好的二进制编辑器。我使用`HexWorkshop ver3.0b`。该二进制编辑器最大的一个好处是它可以计算打开文件的所选范围内的校验和。

3、`LHA2.55`，用来修改`BIOS`二进制。如果你仅想解压缩并分析压缩的`BIOS`组件，也可使用`winzip`或其他可以处理`LZH/LHA`文件的压缩/解压缩工具。
