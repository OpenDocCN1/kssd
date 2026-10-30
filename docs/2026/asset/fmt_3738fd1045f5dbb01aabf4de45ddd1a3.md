
### 1. A执行到调用B的位置的时候，就把`$PC`寄存器的值复制到`$RA`寄存器；此时`$RA`存的是执行完B之后的返回地址，然后去执行B。

### 2. 到B之后，如果B是非叶子函数，就会把`$RA`的值放入堆栈（因为自己待会儿也走第一步要用）

### 3. 执行完B之后，如果B是叶子函数，则直接`jr $ra`回到A就行；如果是非叶子函数，则从堆栈中取出返回地址存入`$ra`，然后再执行`jr $ra`回到A

### 4.1.2 函数调用执行的内存堆栈变化

## 3 web漏洞

## 4 缓冲区溢出漏洞

一般指的是栈的缓冲区溢出。

原理简述：MIPS架构下，函数A调用了`strcpy`函数，会将`strcpy`函数需要返回的地址存在`RA`寄存器，当执行完`strcpy`会回`RA`取回返回地址，并继续执行后续指令。但因为可能其他函数也会调用导致也要重写`RA`寄存器，所以在A刚开始运行的时候会把`RA`存到函数A的栈空间，最后从函数A中取出返回地址给`RA`，再跳转`RA`。

.......

函数的参数及局部变量是存储在函数栈空间的，如果函数中的局部变量内存溢出覆盖掉A函数栈空间高地址的`RA`地址，就会执行攻击者的恶意代码。

## 仿真工具

###### fat and firmadyne

在attifyos用的，自己的环境死活有问题；当前只运行起来README说的那个路由器。

```
下载WNAP320 Firmware Version 2.0.3.zip刀firmadyne文件夹内
```

访问即可。

## 渗透测试

#### 分为三层进行攻击：

- 物理感知层（固件提取，硬件攻击）

- 通信层（流量监听，中间人攻击）

- 管理控制层、应用层（web，rtsp，ssh）

#### 工具：

反汇编：IDA、Ghidra

流量分析：wireshark、tcpdump，burpsuite

固件分析：binwalk，qemu

#### 漏洞风险的切入点：

1. 基于web安全的漏洞风险

- 未授权访问

1. 认证绕过

- 如敏感路径不需要认证即可访问

- cookie伪造

- 案例

1. D-link 850L 伪造全局变量冒充admin

1. D-link DCS-2530L 可访问未认证的敏感路径

1. Tenda系列路由器前台未授权

1. Rtsp服务未授权访问

- 这类漏洞在智能IPC摄像头中最常见，往往不需要密码就能访问

- 可根据不同厂商常见网络摄像头的端口和RTSP地址测试

2. 弱密码

- 默认密码

- 任意密码

- 常见密码爆破

- 命令注入

2. 逻辑漏洞风险

- 任意文件读取

- 存在路径穿越，没有过滤../

- 符号链接漏洞

- 密码重置

3. 基于固件二进制的漏洞风险

- 研究步骤：固件获取--->固件解包--->提取待分析程序--->逆向分析

- 漏洞风险

- 堆栈溢出

- 用户输入验证不足，攻击者输入过长数据覆盖栈缓冲区，重写正常数据

- 案例

1. D-link DIR-815路由器cookie字段栈溢出

1. 思科 RC110w路由器栈溢出

- 拒绝服务

- 逻辑验证不足导致指针异常导致程序非正常崩溃，伤害有限

- 整数溢出

- 越界读写

- 命令注入

4. 后门漏洞风险

- 服务访问后门（web服务，rtsp服务）

- 调试后门（软、硬件）

5. 云端安全风险

- 传统web安全风险

- 越权访问

6. 基于通信、私有协议的漏洞风险

#### 常见漏洞挖掘方法

1. 固件逆向分析之分析输入点

- 寻找二进制中统一的cgi输入函数

1. `getenv("HTTP_HOST")`

1. `get_cgi("device")`

1. `websGetvar(wp,"device_id","")`

1. ......

2. 固件逆向分析之敏感函数、代码段回溯

- 用反汇编工具（IDA、Ghidra）脚本分析

3. 模糊测试（通信协议、固件）

- 端口fuzz

- burp suite

- 协议fuzz工具（boofuzz、peach）

> 主要是路由器，自己学习的一个小总结

> @zunlong copyright

# 0 前置准备

## 1 漏洞分类

路由器是所有上网流量的管控设备，是网络的公共出入口。若路由器被黑客控制，就意味着与网络有关的所有应用都可能被黑客控制。

#### 1. Wi-Fi密码暴力破解

密码常见加密认证方式：WEP、WPA、WPA2、WPA3；常用的是WPA2

同时大部分路由器引入了WPS（一键加密键，一般是实体按键没在我家路由器上找到），便于用户完成无限加密设置。WPS可以通过web页面启用或停用，但它的PIN码有一定规律容易被破解。

#### 2. Web漏洞

家用路由器一般会带web管理页面；会存在通用型web漏洞

#### 3. 后门漏洞

#### 4. 缓冲区溢出漏洞

## 2 基础知识

路由器一般是基于Linux开发的，其系统有一些特点：

1. 一般采用MIPS或者ARM两种指令架构

2. 路由器shell基于busybox

#### 2.1 MIPS Linux

MIPS指令架构属于RISC（精简指令集）体系。

其文件系统一般会有usr sys proc lib etc bin var tmp sbin mnt include home dev目录。

其中：

- bin、sbin和usr下的bin、sbin都是用于存放路由器中的应用程序的目录

- lib以及usr下的lib用于存放程序运行需要的动态库文件

- etc目录用于存放路由器配置文件

  - 比如程序自启动配置文件

  - 脚本文件

  - 各种服务程序的配置文件

#### 2.2 BusyBox

考虑到路由器的存储空间小，不可能整个完整的Linux命令系统，所以使用的shell一般是一个剪裁过的BusyBox文件。

路由器的命令其实都是指向BusyBox的符号链接，不同的路由器剪裁不同，可以`busybox --help`查看。

## 3 MIPS汇编基础

#### 3.1 寄存器

精简指令集RISC一大特点就是大量使用寄存器。

##### 3.1.1通用寄存器

MISP有32个通用寄存器，在汇编中可以用编号`$0~$31`表示，也可以用寄存器名字代替，比如`$sp`, `$t1`, `$ra`；堆栈是从高地址向低地址增长。

| 编号 | 寄存器名称 | 寄存器描述 |
| --- | --- | --- |
| 0 | zero | 第0号寄存器，值始终为0；提供常数0的编码 |
| 1 | `$at` | 保留寄存器 |
| 2~3 | `$v0 ~ $v1` | values，保留表达式或函数返回结果，如果俩不够，编译器就用内存 |
| 4~7 | `$a0 ~ $a3` | aruments，作为函数的前四个参数，不够用堆栈处理。 |
| 8~15 | `$t0 ~ $t7` | temporaries，供汇编程序使用的临时寄存器；调用子函数时，这个临时变量的值可能被子函数改变 |
| 16~23 | `$s0 ~ $s7` | saved values，子函数使用时先保存原寄存器的值 |
| 24~25 | `$t8 ~ $t9` | temporaries |
| 26~27 | `$k0 ~ $k1` | 保留，中断处理函数使用，保存一些系统参数 |
| 28 | `$gp` | global pointer，全局指针 |
| 29 | `$sp` | stack pointer，堆栈指针，指向堆栈的栈顶 |
| 30 | `$fp` | frame pointer，保存栈指针 |
| 31 | `$ra` | return address，返回地址 |

Tips：

- 子函数调用过程：`$a0 ~ $a3`、`$v0 ~ $v1`、`$ra`共同完成，分别用于传递参数，返回结果、存放返回地址。

- MIPS硬件并不直接支持堆栈，不像x86有专门的PUSH/POP指令，所有堆栈使用都是统一的内存访问方式。发生函数调用时，调用者把之后要用的寄存器入栈，被调用者把返回地址和保留寄存器入栈，然后调整栈顶指针。

  - 因此寄存器出入栈都需要指定偏移量

- MIPS有一个`jal`（jump-and-link，跳转并链接）指令，在跳转某地址时可以把下一条指令的地址放到`$ra`中，用于支持子程序。

##### 3.1.2 特殊寄存器

- `PC`：程序计数器

- `HI`：乘除结果高位寄存器

- `LO`：乘除结果低位寄存器

乘法时`HI`存高32位，`LO`存低32位。

触发时`HI`存余数，`LO`存商。

#### 3.2 字节序

当读一个字即4字节，可能出现两种情况：

- `{ mem[n], mem[n+1], mem[n+2], mem[n+3] }`：大端模式（Big-Endian），也称MSB（Most Significant Byte)，数据的高位存在存储器的低地址中

- `{ mem[n+3], mem[n+2], mem[n+1], mem[n] }`：小端模式（Little-Endian），也称LSB（Least Significant Byte）

#### 3.3 MIPS指令集

##### 3.3.1 MIPS指令特点

- MIPS固定4字节

- MIPS指令长度固定32位

- 内存的数据访问必须严格对齐（至少4字节对齐）

- 跳转指令目标地址26位，加上2位对齐位，可寻址28位共256MB

- 条件分支目标地址16位，加上2位对齐位，可寻址18位共256KB

- MIPS默认不把子函数返回地址放到栈中，而是放到`$ra`中，这对叶子函数有利

- 流水线效应。分支跳转语句的后面一条语句叫做分支延迟槽，会提前执行。

|  |  |
| --- | --- |
| 1  2  3 | `mov $a0,$s2`  `jalr strchr`  `mov $a0,$s0` |

在执行第二句之前的时候，第三句也执行完了。

##### 3.3.2 常用的汇编指令

- 寄存器前面都用`$`标识

- `imm` 表示立即数

- `MEM[]` 表示RAM中的一段内存

- `offset`表示偏移量

**（1）LOAD/STORE 指令**

这些 `l` 开头的都是加载指令， `s` 开头的都是存储指令；共有如下指令`lb \ lbu \ lh \ lhu \ li \ lw \ lwl \ lwr \ sb \ sc \ sh \ sw \ swl \ swr`

| `la $t0,val_1` | Load Address。复制val_1表示的地址到`$t0`寄存器中，其中val_1是一个Label |
| --- | --- |
| `li $t1 40` | Load Immediate。将寄存器`$t1`赋值40 |
| `lw $s0,0($sp)` | Load Word。`$s0=MEM[$sp+0]`，取堆栈偏移0内存word长度的值到`$s0` |
| `sw $a0,0($sp)` | Store Word。`MEM[$sp+0]=$a0`,将`$a0`寄存器中word大小的值存入堆栈 |
| `move $t5,$t1` | `$t5=$t1` |

**（2）算数运算指令**

- 算术运算指令操作数只能是寄存器，不能用RAM地址或者间接寻址

- 操作数大小都是word

`add addi addiu addu sub subu clo clz slt slti sltiu sltu mul mult multu madd maddu msub msubu div divu`

| `add $t0,$t1,$t2` | `$t0=$t1+$t2` |
| --- | --- |
| `sub $t0,$t1,$t2` | `$t0=$t1-$t2` |
| `addi $t0,$t1,5` |  |
| `addu` | 无符号数 |
| `mult $t3,$t4` | （Hi，Lo）=`$t3 * $t4` |
| `div $t5,$t6` | `$t5/$t6`,LO存商，HI存余数 |
| `mfhi $t0` | `$t0=$HI` |
| `mflo $t1` | `$t1=$LO` |

**（3）类比较指令**

`slti`(立即数）；`sltu`（无符号）

| `slt $v0,$a0,$s0` | `$v0=$a0<$s0?1:0` |
| --- | --- |
|  |  |

**（4）SYSCALL**

产生软中断，实现系统调用

**（5）分支跳转**

| `b target` | 无条件跳转到target |
| --- | --- |
| `beq $t0,$t1,target` | `$t0=$t1`则跳转 |
| `blt $t0,$t1,target` | `$t0<$t1`则跳转 |

**（6）跳转指令**

- `j target`：无条件跳转

- `jr $t3`：跳转到`$t3`寄存器指向地址

- `jal target`：跳转到target，并保存返回地址到`$ra`

# 1 搭建环境

这个有很多坑.......主要可能出现的坑点：软件包太老无法安装、软件包深度依赖、idaPro的32位Linux版本在64位ubuntu无法使用（安装wine失败）、部分软件包需要翻墙但虚拟机nat连接给宿主机的代理有问题、虚拟机必须内存3G以上否则buildroot的menu config会出错。

[环境搭建](https://bbs.kanxue.com/elink@6c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6N6i4q4#2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4A6#2L8X3I4G2L8X3N6Q4x3X3c8^5L8o6N6C8K9q4)9J5c8X3S2#2K9h3V1#2N6q4)9J5c8Y4c8Y4K9o6S2J5N6W2)9K6c8Y4k6A6k6i4N6Q4x3@1c8V1L8$3y4Q4y4h3k6W2L8h3u0W2k6l9`.`.)

- binwalk：用于分析固件、提取系统；比如可以分析固件中的文件名、路径名、版本信息、符号表地址、CPU架构；都展示在description中，但我的binwalk可能少装了什么插件，没这些信息。

- sasquatch：用来解压squashfs的工具

- squashfs：常用于嵌入式系统中**只读压缩文件系统**，可以减少成本。

- qemu：QEMU是一个处理器模拟软件，可以用来在PC中模拟ARM、MIPS等多种架构的软硬件运行环境。主要有两种模式：

  1. User Mode：使用者模式，运行程序的；两种模拟程序：大端的qemu-mips和小端的qemu-mipsel；可以直接qemu-mips编译程序并运行。

  2. System Mode：系统模式，模拟整个计算机系统（相当于vmware）；`sudo qemu-system-mips -kernel 某1 -hda 某2 -append 某3`，即用qemu-system-mips启动MIPS虚拟机，内核是某1，磁盘镜像为某2。

- 大端和小端：大端（Big-endian）和小端（Little-endian），指程序存放的字节顺序区别；大端是高位存低地址，小端相反。x86系列CPU都是`Little-endian`字节序，PowerPC通常是`Big-endian`字节序。

  > file命令看到的，MSB是大端，LSB是小端

- buildroot：交叉编译环境，就是用来把嵌入式开发写的代码编译成系统的。

  `buildroot是Linux平台上一个构建嵌入式Linux系统的框架。整个Buildroot是由Makefile脚本和Kconfig配置文件构成的。你可以和编译Linux内核一样，通过buildroot配置，menuconfig修改，编译出一个完整的可以直接烧写到机器上运行的Linux系统软件(包含boot、kernel、rootfs以及rootfs中的各种库和应用程序)。`

- IDA：逆向软件分析的。

- mips：是一种采取精简指令集（RISC）的处理器架构，有32位64位之分。

- 主流芯片架构：ARM、MIPS、X86

  1. ARM：16/32位精简指令集，**体积小、低功耗、低成本、高性能**，广泛用于嵌入式系统。

  2. MIPS：32/64位精简指令集，**有大量寄存器、指令数和字符，可视管道延时时隙**，SoC（系统级芯片）设计中最低的能耗。

  3. X86：intel设计，如8086/80286，大部分的PC用的都是X86，采用复杂指令集；控制简单效率慢。

- busybox：路由器系统存储空间受到限制，用的shell都是基于busybox，busybox打包好了很多命令的符号链接。

---

经常看到的命令：

|  |  |
| --- | --- |
| 1 | `sudo chroot . .``/``qemu``-``mips``-``static  .``/``bin``/``ls` |

就是将系统重定向的，先把系统的根地址重定向到当前位置，然后用qemu-mips-static运行ls命令。

---

qemu的使用者模式用的比较多，但它无法模拟一个真正的mips-linux内核环境，所以需要用qemu的system模式模拟一个mips架构的debian环境。

|  |  |
| --- | --- |
| 1 | `sudo qemu``-``system``-``mips` `-``M malta` `-``kernel vmlinux``-``2.6``.``32``-``5``-``4kc``-``malta` `-``hda debian_squeeze_mips_standard.qcow2` `-``append` `"root=/dev/sda1 console=tty0"` `-``net nic,macaddr``=``00``:``16``:``3e``:``00``:``00``:``01` `-``net tap` |

安装环境测试的这段命令，就是模拟一个debian系统。

---

# 2 漏洞分析技能

### IDA 动态调试

这里我们先调试./bin/ls命令，该路由器的shell是基于BusyBox的，可以看到ls指向busybox

|  |  |
| --- | --- |
