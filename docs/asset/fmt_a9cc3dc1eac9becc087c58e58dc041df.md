
//销毁 `AnsiString1`

```
RtlFreeAnsiString(&AnsiString1);
```

//(2)将 `ANSI_STRING` 字符串转换成 `UNICODE_STRING` 字符串

//初始化 `AnsiString2`

```
ANSI_STRING AnsiString2;
RtlInitString(&AnsiString2, "Hello World");
UNICODE_STRING UnicodeString2;
nStatus = RtlAnsiStringToUnicodeString(&UnicodeString2, &AnsiString2, TRUE);

if (NT_SUCCESS(nStatus))
{
    KdPrint(("Conver succussfully!\n"));
    KdPrint(("Result:%wZ\n", &UnicodeString2));
}
else
{
    KdPrint(("Conver unsuccessfully!\n"));
}
```

//销毁 `UnicodeString2`

```
RtlFreeUnicodeString(&UnicodeString2);
```

`///CPU表///`

*   **全局描述符表** `Global Descriptor Table`, `GDT` 用于映射地址
*   **本地描述符表** `Local Descriptor Table`, `LDT` 用于映射地址
*   **页目录** `Page Directory` 用于映射地址
*   **中断描述符表** `Interrupt Descriptor Table`, `IDT` 用于寻找中断处理程序

`///OS表///`

*   **系统服务调度表** `System Service Dispatch Table`, `SSDT`, `Windows OS` 用于处理系统调用

一个地址空间是否可以被写入，可以被多少优先级的代码写入，是不是允许执行等涉及保护，涉及属性和保护模式下段的其他参数，要表示的信息太多了，要用 64 位长的数据才能表示。我们把这 64 位的属性数据叫做段描述符（`Segment Descriptor`）。

`80386` 中引入了两个新的寄存器来管理段描述符表。一个是 48 位的全局描述符表寄存器 `GDTR`，一个是 16 位的局部描述符表寄存器 `LDTR`。

### GDTR [全局描述符表寄存器 48 位]

`GDTR` 指向的描述符表为全局描述符表 `GDT`, `Global Descriptor Table`。

它包含系统中所有任务都可用的段描述符，通常包含描述操作系统所使用的代码段、数据段和堆栈段的描述符及各任务的 `LDT` 段等；全局描述符表只有一个。

`GDT` 不为系统提供任何安全能力。

### LDTR [局部描述符表寄存器 16 位]

`LDTR` 则指向局部描述符表 `LDT`（`Local Descriptor Table`）。`80386` 处理器设计成每个任务都有一个独立的 `LDT`。

它包含有每个任务私有的代码段、数据段和堆栈段的描述符，也包含该任务所使用的一些门描述符，如任务门和调用门描述符等。

`CS`, `DS`...段寄存器中的信息不再是段地址了，而是段选择器（`Segment Selector`）。可以通过它在段描述符表中“选择”一个项目以得到段的全部信息

*   16 位的段选择器中只有高 13 位表示索引值。剩下的 3 个数据位中，第 0，1 位表示程序的当前优先级 `RPL`；第 2 位 `TI` 位用来表示在段描述符的位置；`TI=0` 表示在 `GDT` 中，`TI=1` 表示在 `LDT` 中。
*   `xxxx:yyyyyyyy` 格式表示一个虚拟地址，经过段选择器及局部描述符表从全局描述符表得到段的起始地址。

### 地址转换

```
test dd 100 ;若 test 的 rva 值是 2000, imagebast 是 400000,则物理地址是 402000 处
...
mov eax, test ;读入到 eax 中的值是存在物理地址上的值
```

| 转换过程 | 结果 |
| :--- | :--- |
| `______>` 页表转换 `------------->` `2000______>` `402000` `----------->` `eax` | `402000` |
