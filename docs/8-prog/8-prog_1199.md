
# 我的学习的脚印 2011.6.11 日--中断 IDT

> 原文：[`www.pediy.com/kssd/pediy12/135256.html`](https://www.pediy.com/kssd/pediy12/135256.html)

声明：  

内容都是书中或 baidu 的，很多在论坛都有出现，只是觉得有阅读和收藏整理以方便自己使用的必要。另外就是方便自己以后回头看看自己曾经走过的坎坷之路，仅此而已。  

## 实模式

### IVT： 中断向量表

从 0 地址开始，共 256 个，结构就是个地址表，每个占 4 字节，保存的是中断向量处理函数，存储方式仍为高高低低。

2-3 Seg  
0-1 offset `jmp Seg:offset`

`Seg:offset` 中断处理函数

#### 处理步骤：

1. 把 `EFLAGS` 压栈
2. `CS` 栈
3. `IP` 压栈(指向中断处理结束后要执行的下一条 `OpCode`)
4. 清楚 `IF` 和 `TF` 标志位.
5. 根据中断向量(索引)找到对应的 `IVT` 数据项
6. 报 `IVT` 中的段地址和偏移地址分别加载到 `CS` 和 `IP` 寄存器

#### IRET 返回过程

1. 栈顶 16 位数值 pop 到 `IP` 前面压栈对应的弹出
2. 栈顶 16 位数值 pop 到 `CS`
3. 栈顶 16 位数值 pop 到 `EFLAGS`

### 读中断信息

`Int 21`  
`AH` = Dos 中断功能号 `0x35`  
`AL` = 中断向量  
返回值放在 `ES`：段，`BX`：偏移

### 写中断向量表

`Int 21`  
`AH` = Dos 中断功能号 `0x25`  
`Al` = 中断向量  
`Push DS`  
`DS` = 要改写的段地址  
`DX` = 要改写的 `Offset`  
`Pop DS`

## 保护模式

`Idtr` 中断描述表基值为的基地址  
`Idtrl` 界限(长度)

```
kd> r idtr
idtr=8003f400
```

中断描述符表，每个表大小是 8byte，因为兼容的关系处理函数地址被分割在低 2byte 和高 2byte，共 32 位，16 位应该就是用低 16 位就好了吧，哈哈。

代码:

```
kd> dw 8003f400
8003f400  e36c 0008 8e00 8053    e4e4 0008 8e00 8053
8003f410  112e 0058 8500 0000    e8b4 0008 ee00 8053 
```

第三个中断描述符为 `8053 ee00 0008 e8b4`

中断处理函数地址为：`8053e8b4` 选择子为 `8` 属性为 `ee00`

代码:

```
kd> u 8053e8b4 
nt!KiTrap03:
8053e8b4 6a00             push    0
8053e8b6 66c74424020000  mov     word ptr [esp+2],0
8053e8bd 55               push    ebp
```

可以通过修改中断描述表内容或修改描述符表对应函数地址中的某些东东来实现 hook

[`bbs.pediy.com/showthread.php?p=417908#post417908`](http://bbs.pediy.com/showthread.php?p=417908#post417908)

这篇对 `IDT` 的描述太详细了，以至于没看明白，hook 的的思路也是修改描述表的 `offset` 来实现修改中断处理函数。

代码:

```
typedef struct
{
  unsigned short LowOffset;
  unsigned short selector;
  unsigned char unused_lo;
  unsigned char segment_type:4;  //0x0E is an interrupt gate
  unsigned char system_segment_flag:1;
  unsigned char DPL:2;  // descriptor privilege level 
  unsigned char P:1; /* present */
  unsigned short HiOffset;
} IDTENTRY;
  // load idt_info
  __asm  sidt  idt_info      //不知道 s 是那个单词缩写，set 还是...   而 r-read  如：rdmsr
  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);   //取高 2 和低 2byte
```

关键代码

修改 IDT 处理函数

代码:

```
  // remember we disable interrupts while we patch the table
  __asm cli
  idt_entries[NT_INT_TIMER].LowOffset = (unsigned short)my_interrupt_hook;
  idt_entries[NT_INT_TIMER].HiOffset = (unsigned short)((unsigned long)my_interrupt_hook >> 16);
  __asm sti
```

恢复 IDT 代码:

```
   __asm cli
  idt_entries[NT_INT_TIMER].LowOffset = (unsigned short) old_ISR_pointer;
  idt_entries[NT_INT_TIMER].HiOffset = (unsigned short)((unsigned long)old_ISR_pointer >> 16);
  __asm sti
处理时感觉需要增加修改 Cr0 内存属性操作。
```

## 全局和局部描述符表格式

GDT 和 IDT 的格式类似

`GDT` ： 长度(界限)  
`BaseAddr` GDT 描述符表的地址

代码:

```
kd> r gdtr
gdtr=8003f000
kd> dw 8003f000
8003f000  0000 0000 0000 0000 ffff 0000 9a00 00cf
8003f010  ffff 0000 9200 00cf ffff 0000 fa00 00cf
8003f020  ffff 0000 f200 00cf 20ab 2000 8b04 8000
kd> r ldtr
ldtr=00000000
```
