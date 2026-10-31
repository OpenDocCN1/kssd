
# 虚拟地址和偏移量转换

## 基础知识

### 一、虚拟地址和偏移量转换

由于Windows程序是运行在386保护模式下，在保护模式下，程序访问存储器所使用的逻辑地址称为虚拟地址（`Virual Address`，`VA`）。与实地址模式下的分段地址类似，虚拟地址也可写成"段：偏移量"的形式，这里的段是指段选择器。

文件执行时将被映像到指定内存地址中，这个初始内存地址称为基址（`imageBase`）。在Windows NT中，缺省的值是`10000h`；对于DLLs，缺省值为`400000h`。在Windows 9x中，`10000h`不能用来装入32位的执行文件，因为该地址处于所有进程共享的线性地址区域，因此Microsoft将Win32可执行文件的缺省基地址改变为`400000h`。

相对虚拟地址（Relative `Virual Address`，`RVA`）表示此段代码在内存中相对于基地址的偏移。即：相对虚拟地址(`RVA`)=虚拟地址（`VA`）-基址（`imageBase`）。

文件中的地址与内存中表示不同，它是用偏移量(`File offset`)来表示的。

在SoftICE和W32Dasm下显示的地址值是内存地址（memory offset），或称之为虚拟地址（`Virual Address`，`VA`）。而十六进制工具里，如：Hiew、Hex Workshop等显示的地址就是文件地址，称之为偏移量（`File offset`) 或物理地址(`RAW offset`)。

在实际操作时，使用 `RVA-Offset`之类的转换器很容易查出字串`RVA`和偏移量的值。

以`Pc_offset`（光盘\tools\offset）此工具为例说说如何将SoftICE下看到的虚拟地址(`Virual Address`, `VA`)转换成十六进制工具里的偏移量(`File offset`)，运行该软件打开记事本程序，在Memory Address（内存地址）输入虚拟地址（`Virual Address`, `VA`）的值：`40117D`，点击"Do it"按钮将显示己转换成文件偏移量(`File offset`)，如图一所示。

![](https://www.kanxue.com/image/chap411.gif)

（图一）内存虚拟地址转换成文件偏移量
