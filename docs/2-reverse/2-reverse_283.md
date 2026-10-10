

# TRW2000 操作手册

> 原文：[`www.pediy.com/kssd/tutorial/append-c.htm`](https://www.pediy.com/kssd/tutorial/append-c.htm)

 

**TRW2000 操作手册**

|   || CODE || HWND || PRET || TRTCB |
| --- | --- | --- | --- | --- |
| ? || D || IDT || PROC || U |
| ADD || E || LDT || R || VCALL |
| ADDR || EC || LINES || RS || VER |
| BC || EXP || MakePE || S || VM |
| BD || EXP16 || MOD16 || SRC || VXD |
| BE || EXP32 || MOD32 || SRCLINES || VXDSYM |
| BL || FKEY  || NAME || SS || W  |
| BP || FILE  || P || Suspend || WC |
| BPE || FONT || PAGE || SYM || WD |
| BPIO || G || PageIn || T || WMSG |
| BPM || GDT || PDLL32  || TABLE || WR |
| BPMSG || H || PEDump  || task  || WS |
| BPMX || HBOOT || PHYS || thread  || X |
| BPX || HELP || PMODULE || TRNEWDOS ||   |
| CLEARDR || HotKey || PNEWSEC || TRNEWTCB ||   |

  

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || .　 |
| 作用: || 代表当前值（如当前进程，当内存区域等） |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || ？ |
| 作用: || 帮助，相当于“HELP”或“H”命令 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || ADD STACK|DATA|DASM |
| 作用: || 增加一个堆栈/数据/反汇编窗口 |
| 用法: || 例：ADD　STACK  
 　　ADD　DATA   
　　ADD　DASM |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || ADDR　内存句柄｜过程名 |
| 作用: || 显示或是切换内存区域 |
| 用法: || 用来察看某一个任务的私有内存区域, 或是加参数\[内存区域句柄|过程名\]在 TRW2000 中切换某一任务的私有内存区域为当前可寻址的内存区域。  
一些参数的显示:  
.HANDLE 内存区域控制块的地址  
.PGTPTR 每个任务私有页表的起址  
.TABLES 每个私有页表中的表项数目  
.MINADDR 每个任务的线性地址的起址  
.MAXADDR 每个任务的线性地址的终址  
.MUTEX VMM 用于页表管理的句柄  
.OWNER 使用这块内存区域的实例的名字。如果有多个相同名字的实例运行，ADDR 带 OWNER 名字切换的话，切到表中的第一个有此 NAME 的实例所占的内存区域。所以带 HANDLE 切换的话比较精确。  
当用 ADDR 加参数后，可以用上面提到过的"**.**"命令来回到 TRW2000 弹出时所属的任务内存区域。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BC list|\*  |
| 作用: || 清除一个或多个断点 |
| 用法: || 清除断点后，用 BL 命令就看不到断点列表，且被清除的断点不再起作用。  
参数: list: 可以是将要清除的一系列断点，中间用空格或逗号隔开。  
　　　　 \*: 清除所有的断点。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BD　list|\* |
| 作用: || 使一个或多个断点失效 |
| 用法: || 使某个断点失效与 BC 命令的清除不同，暂时失效的断点可以用 BE 命令来恢复。而 BC 命令是彻底清除  
参数:list: 可以是单个，也可以是一系列断点，中间用空格或逗号隔开。  
　　 　 \*: 禁止所有的断点。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BE　list|\* |
| 作用: || 恢复被 BD 命令使之失效的断点。(每当新定义断点或编辑断点时，系统自动将其置为有效) |
| 用法: || 参数:list: 可以是单个，也可以是一系列断点，中间以空格或逗号隔开。  
　　 　\* : 恢复所有的断点。 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BL　 |
| 作用: || 显示当前所设的断点 |
| 用法: || 用 BL 命令显示当前所有断点的序号(这个序号就是使用 BC BD BD 等命令时要指定的)、类型、是否被禁止等信息。如果是被 BD 禁止的断点，会在序号后出现一个"\*"号。 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BP　　\[address\]\[if (condition)\] |
| 作用: || 设置通过执行地址或所设条件激发的断点。 |
| 用法: || 例：BP if (eip>10000) |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BPE　number |
| 作用: || 编辑一个已存在的断点。 |
| 用法: || 用 BPE 命令可以很方便地修改一个已经存在的断点。  
但要注意一点: BPE 在执行时,会先将你所要修改的断点清除,然后再将改过的使能。  
如果你在修改时按 ESC 键退出修改,那么原先的断点也就不存在了, 修改错误的结果也是一样的，会造成原先的断点消失。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BPIO port |
| 作用: || 设置一个由 I/O 端口触发调试寄存器断点。 |
| 用法: || 相关内容: BreakPoint Overall |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BMSG hwnd \[msg\]　　　　　　hwnd 为句柄　　msg 为用于触发的消息 |
| 作用: || 设置由 Windows 消息触发的断点 |
| 用法: || BPMSG hwnd \[msg\]  
相关内容:  
 　　　　WMSG  
　　　　BreakPoint Overall   
例:BMSG 12c wm\_destroy  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BPM address R  
BPM address W  
BPM address X  |
| 作用: || 通过 DRx 设置一个硬件断点;相关内容: BreakPoint Overall |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BPMX 地址 |
| 作用: || 通过 DRx 设置一个可执行的断点 |
| 用法: || 等同于命令 'BPM address X' |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BPX \[\[seg:\]address\]  
BP \[\[seg:\]address\]  |
| 作用: || 执行时断点;TRW 会在代码中相应位置插入一条 int3(0xcc) 指令。 |
| 用法: || 相关内容:  
 　　　　BreakPoint Overall   
　　　　BPMX  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || CLEARDR　清除 DRx 寄存器 |
| 作用: ||   |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || CODE \[ON|OFF|number\] |
| 作用: || 设置代码窗口的显示方式 |
| 用法: || 设为 ON 则显示 16 进制的机器码，设为 OFF 则不显示 16 进制的机器码  
number 决定显示代码的风格（方式），可以以各种方式显示  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || D \[address\] |
| 作用: || 将内存某处的内容映像到数据窗口或者将内存某处的内容映像到文件中。 |
| 用法: || D range >filename  
例：  
　d 401000  
　d cs:402000  
　d 401000,402000 >myfile  
　d 401000 L 100 >myfile  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || E 　　\[address \[partern\]\] |
| 作用: || 编辑内存 |
| 用法: || 例： E ds:edi 'nothing',0 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || EC |
| 作用: || 打开或关闭子窗口 |
| 用法: || 如果当前子窗口不可见，那么 EC 命令将打开它，反过来，EC 命令将关闭它。 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || EXP |
| 作用: || 显示一个模块所有的 API。 |
| 用法: || EXP 　!  
Exp 　module-name!  
Exp 　partial\_export\_name  
搜索所引用的所有模块输出的 API 函数名  
　　EXP !  
　　Exp kernel32!  
　　Exp \*  
　　Exp Get\*  
　　Exp \*window\*  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || EXP16　　　 |
| 作用: || 显示 16 位的 exp |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || EXP32　 |
| 作用: || 显示 32 位的 exp |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || FILE　 \[source-filename\] |
| 作用: || 选择/显示源文件。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || FONT　　1｜2 |
| 作用: || 设定 TRW2000 用于显示的字体，默认为 2。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || FKEY \[function-key strings\] |
| 作用: || 显示/设置功能键 |
| 用法: || 例:  
　　FKEY  
　　FKEY f10 "d 2;U 3;"  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || G |
| 作用: || 运行 |
| 用法: || G address ;运行到由 address 指出的地址, 等同于 'gox address'命令  
G if conditions ;如果 conditions 满足则运行  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || GDT　　\[Selecter\] |
| 作用: ||  显示指定全局选择子 GDT 的详细信息。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || H \[command\] |
| 作用: ||   |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || HELP \[command\] |
| 作用: || 显示所有命令的帮助，后面跟命令名则显示该命令的详细说明。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || HOTKEY |
| 作用: || 显示当前有效的热键，也可以显示新按键。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || HBOOT　 |
| 作用: || 重新热启动计算机 |
| 用法: || HBOOT 将重新启动计算机，等同于按 Ctrl+Alt+Del 组合键。  
HBOOT 一般都能成功,只有特殊情况下(某些插卡需要重加电)才用机器上的 RESET 或 POWER 键  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || HWND \[HWND\] |
| 作用: || 显示 Winodws 句柄信息。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || IDT　　number |
| 作用: || 显示 IDT 的信息 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || LDT　　number |
| 作用: || 显示 LDT 的信息 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || Page　　\[address \[L length\]\]  |
| 作用: || 显示页表信息。 |
| 用法: || address : 段:偏移量 或 选择符:偏移量 格式的地址  
length : 要显示页的数量.PAGE 命令用来列出当前页目录和各个页表的情况.  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PageIn <address> |
| 作用: || 将非当前页调入内存。 |
| 用法: || 例：PageIn cs:401000 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || LINES \[25 | 43 | 50 | 60\] |
| 作用: || 设置/显示当前显示的屏幕行数。 |
| 用法: || 例: lines 43 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || MOD16 \[hmod|mod\_name\] |
| 作用: || 显示 16 位模块列表，如果后跟 mod\_name 则显示指定模块的详细信息。 |
| 用法: || MOD16  
MOD16 1cf  
MOD16 KERNEL  
MOD16 . ; .是指当前模块  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || MOD32 \[hmod|mod\_name\] |
| 作用: || 显示 32 位模块列表，如果后跟 mod\_name 则显示指定模块的详细信息。 |
| 用法: || MOD32  
MOD32 1cf  
MOD32 KERNEL32  
MOD32 . ; .是指当前模块  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || MAKEPE |
| 作用: || 从内存中整理出一个名为 'newpe.exe' 的 PE 格式的 exe 文件。 |
| 用法: || 每次使用该命令前应先删除当前目录下的 'newpe.exe' ,否则 TRW2000 会将新的文件附加在原文件的后面。当前的 EIP 将成为新的程序入口。  
该命令将在当前目录下生成文件，但生成文件的 Import table 已经重新生成过了。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || NAME　　address newname |
| 作用: || 定义指定地址的新符号名 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PDLL32 |
| 作用: || 运行到 32 位的 DLL 的入口。 |
| 用法: || PDLL32 mydll32.dll |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PEDUMP |
| 作用: || 将 PE 文件的内存映像直接映像到文件'DUMP1.EXE'， |
| 用法: || 您只要使用 MakePE 命令就可以重新整理出一个可用的 PE 可执行文件来。 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || BP if condition |
| 作用: || 条件断点，当条件满足时产生中断 |
| 用法: || 例：　bp if (eax>=3456787)  
bp if (dx<543)  
bp if (ch==23)  
go if (ah!=34)  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || P 单步跟踪指令 |
| 作用: ||  P 命令将单步执行程序。在汇编模式中，当遇到 CALL,INT,LOOP,REP 指令时，P 将不跟踪进去,直到这些指令执行完毕，控制才返回 TRW2000。换句话说，P 命令是"跨"过这些指令的。   
P 后加 RET 参数，SoftICE 将一直单步执行直到它找到一条返回语句(RET,RETF)。在源程序模式中，P 命令将执行一个源程序表达式，但也不跟踪到子例程中去。  
P 命令实际上是利用了单步标志。 大多数情况下是如此.但碰到 CALL,INT,LOOP,REP 指令时，就用 INT 3(一次性)在这些指令的后面设一下。 　 |
| 用法: ||  P 命令对应的有快捷键 F10 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PNEWSEC |
| 作用: || 运行直到进入一个 PE 程序内存的新的 section （这个词不知如何描述:D）时产生断点 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PMODULE |
| 作用: || 'pret'（相当于按＜F12＞） 直至 CS:EIP 位于模块中。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PRET |
| 作用: || 运行到遇到 RET,RETF,IRET 指令时停下。 |
| 用法: || 快捷键: F12 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || PROC |
| 作用: || 显示进程列表 |
| 用法: || output:

```
 :proc Process pProcess ID Threads Context -------- -------- 
\-------- ------- -------- Trw2k 81641D9C FFFB0415 1 C8070F70 Iexplore 81638644 
FFFC9FCD 4 C80EA130 WINOLDAP 81636FE0 FFFC7669 1 C17D9E80 Spool32 8162C990 FFFDD019 
2 C80711A0 KERNEL32 81702204 FFEF3B8D 8 C151CEA0 Mdm 8161B708 FFFEAE81 2 C17E4DC0 
Osa 81615458 FFFE4DD1 1 C17DC0D0 RUNDLL 816239D8 FFFD2051 1 C807C010 Internat 
8161A9A4 FFFEB02D 1 C17D5140 Systray 81616B5C FFFE72D5 1 C17C1E10 Taskmon 81616080 
FFFE7909 1 C17B9790 Explorer 8160E6A0 FFFFFF29 4 C153FCC0 Mstask 8160D390 FFFFCA19 
2 C152C9E0 Mprexe 8160720C FFFF6B85 1 C15274F0 MSGSRV32 8160603C FFFF79B5 1 C1442F90
```

 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || R 　　　\[-d | register-name | register-name \[=\] value\]  |
| 作用: || 显示或更改寄存器的内容。 |
| 用法: || 如果 R 命令不加参数，光标将移到寄存器窗口中，进行实时修改。如果当前寄存器窗口不可见， 那么这个命令将自动显示它。  
另外,修改 FL 寄存器时,参数不必按照顺序,如: R  fl=o+a-c 可以一次修改 3 个标志位。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || RS |
| 作用: || 显示用户屏幕。 (快捷键为＜F4＞). |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || S |
| 作用: || 在内存中搜索指定内容 |
| 用法: || 例：　S 0 L -1 'window'  
　　　S 100,200 'bug12',34  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || SRC |
| 作用: || 切换源代码模式，可以是源代码｜混合代码｜单纯的可执行代码 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || SUSPEND |
| 作用: || 暂停当前的线程，并退出 TRW2000，按热键返回。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || SYM |
| 作用: || 显示所有的调试符号列表。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || T　　　 \[startaddress\]\[count\] |
| 作用: || 单步跟踪指令。　T 命令是利用 CPU 的单步标志来进行单步跟踪的。如果指定 startaddress，将从指定的地址处开始单步跟踪。如果寄存器窗口可见，则寄存器窗口将高亮显示哪些改变的寄存器的值。  |
| 用法: || 快捷键为＜F7＞ |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || TABLE \[table name\] |
| 作用: || 选择/显示调试符号表 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || TASK |
| 作用: || 显示任务列表 |
| 用法: || output:

```
 :task W16TDB TaskName THCB Ring0TCB TDBX ------ -------- -------- 
\-------- -------- 39E6 Trw2k 81643454 C8095D30 C8092650 3EAE WINOLDAP 816371E4 
C80BC9F0 C17DC9E0 0FCE Spool32 8162CB7C C152A970 C8081730 00CF KERNEL32 817022DC 
C29202B0 C1415CF0 2FCE Mdm 8161B930 C17E4EF0 C17DEE40 2A16 Osa 81625884 C17DC290 
C17DC170 2E56 RUNDLL 8161FD80 C807C1D0 C807C0A0 305E Internat 8161ABA0 C17D5320 
C17D51F0 21DE Systray 81616D00 C17C1BC0 C17C1EC0 226E Taskmon 81616294 C17B9970 
C17B9840 2466 Explorer 8160E8B0 C153FEA0 C153FD70 265E Mstask 8160D5A8 C152AE60 
C152A150 1D8E Mprexe 81607434 C1528AB0 C1526C40 1FC6 MSGSRV32 8160620C C144B0A0 
C14D1390
```

 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || THREAD |
| 作用: || 显示线程的有关信息。 |
| 用法: || 例：  
　THREAD \[TCB\]  
　THREAD . ; .代表当前线程   
　THREAD c1547690   
output:

```
 :thread Ring0TCB ID Context Ring3TCB ThreadID 
Process TaskDB PDB SZ Owner c1574d40 7f c1574b60 81641d88 fffb069b 81641b68 1BEE 
1BE7 32 \*Trw2k c152cbb0 2a c152ca60 8162a678 fffdbd6b 8160fee0 2CAE 2C7F 16 \*WINOLDAP 
c1547690 21 c1548cc0 81623a94 fffd2187 8162383c 307E 3077 16 \*RUNDLL c15245b0 
8 c1431fc0 8160b2b4 ffffa9a7 81709204 00C7 00D7 32 KERNEL32 c14167d0 2 c29202b0 
1 c1431fc0 817092dc ffef89cf 81709204 00C7 00D7 32 VM 1 c65202b0 2b VM 2 :thread 
c1547690 Ring0TCB ID Context Ring3TCB ThreadID Process TaskDB PDB SZ Owner c1547690 
21 c1548cc0 81623a94 fffd2187 8162383c 307E 3077 16 \*RUNDLL Thread Flag = 00511401 
00000001 ???? 00000400 THFLAG\_THREAD\_BLOCKED 00001000 ???? 00010000 THFLAG\_OEM 
00100000 ???? 00400000 ???? ClientPtr = C5BF3F70 TDBX = C1548DD0
```

 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || TRNEWDOS |
| 作用: || 捕获另外的 DOS 程序。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || TRNEWTCB |
| 作用: || 捕获一个新的线程的入口操作码  
支持 32 位和 16 位程序  
不要在 DOS 提示符下运行 Windows 程序，否则 TRW2000 会捕获 START.EXE。  |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || TRTCB <thread\_handle> |
| 作用: || 跟踪一个正在运行的线程。 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || U \[address\] |
| 作用: || 在代码窗口反汇编程序或将反汇编的代码输出到一个文件。 |
| 用法: || U range >filename  
例:　 u 401000  
　　　u cs:402000  
　　　u 401000,402000 >myfile  
　　　u 401000 L 100 >myfile  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || VER |
| 作用: || 显示 TRW2000 的版本信息 |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || VCALL |
| 作用: || 显示 VxD 调用 |
| 用法: || Vcall \* -显示所有的 VxD 调用  
Vcall num -如果指定的 num<10000h, 显示所有的 VxD 调用  
\-如果指定的 num>10000h, 则显示所指定的 VxD 调用  
例:   
 　Vcall partial-name   
　Vcall 0d  
　Vcall 100001  
 　Vcall \*sys\*  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || VM　　　\[VMID\]  |
| 作用: ||  显示虚拟机的信息。 |
| 用法: || 如果 VM 命令不加参数，将显示系统中所有虚拟机的信息。如果加 VMID 参数,该 VM 的寄存器值将被显示。  
这里的寄存器值是在 VM 控制块用户区中的寄存器值。所以，这些寄存器表示的是最后当有内存地址切换时存入该控制块时的值。  
当 TRW2000 弹出时正好某个 VM 正在运行的话，在寄存器窗口中的值才是真正当前的值，而不是用 VM 命令在命令窗口中看到的值。  
有一点要注意:如果你在某个中断例程中的头几句指令处，而此时这个 VM 的寄存器值正在被存入控制块，那么可能当前只有 CS:IP 的值(由 VM 显示)是真的。其余的有可能还没有存入控制块。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || VXD　　 \[VXDNAME\] |
| 作用: || 显示 Windows VxD 映象。  
VXD 命令将显示指定的某个 VxD 的信息。如果不加参数，VXD 命令将显示系统中所有的虚拟设备驱动程序的映象。动态装载的 VxD 将显示在静态装载的 VxD 后面。  |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || W range filename |
| 作用: || 将内存内容写入指定文件 |
| 用法: || 例:　 w ds:401000,402000 myfile  
　　　w 401000 L 100 c:\\myfile.bin  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || WC \[codewindow\_lines\] |
| 作用: || 设置代码窗口的行数，如果不带参数则切换其开/关状态。 |
| 用法: || 例:  
　wc 25  
　wc |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || WD \[datawindow\_lines\] |
| 作用: || 设置数据窗口的行数，如果不带参数则切换其开/关状态。 |
| 用法: || 例:  
　wd 25  
　wd |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || WR　　　 |
| 作用: || 打开或关闭寄存器窗口  
如果当前寄存器窗口不可见，那么 WR 命令将使之可见，反过来，WR 命令将使寄存器窗口不可见。  
寄存器窗口显示 80386 寄存器集。  |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || WS　　　 |
| 作用: || 打开或关闭堆栈窗口  
如果当前堆栈窗口不可见，那么 WS 命令将使之可见，反过来，WS 命令将使堆栈窗口不可见。  |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || WMSG |
| 作用: || 显示 Windows 消息 |
| 用法: || WMSG \[partial-name\] \[WMSG-number\]  
例:  
　　WMSG 12  
　　WMSG wm\_destroy  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 命令: || X　　 |
| 作用: || 退出当前调试状态，返回 Windows |
| 用法: ||   |之一 之一 之一 之一 之一 之一 之一 

 　　　 　　　 　　　　　　　　　　　　 

* * *

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
| 条件断点语法 |
| 几乎所有的断点都可以通过如下的语法：  
　　BP?? <parameters> \IF ([conditions)\] \DO "[statement"\]  
而且开始的 2 个字母 'BP' 可以代换成 'GO'，即:   
　　GO?? <parameters> \IF ([conditions)\] \DO "[statement"\]  
来设置条件断点。 TRW2000 能够根据要求设置、使用和清除这些断点。  |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
|   || conditions: |
| 用法: || 条件必须用闭括符括起来:  
 

> ```
> Operator Meaning == Logical Equal != Logical Not equal 
> \> Logical Greater than >= Logical Greater than or equal to < Logical 
> Less than <= Logical Less than or equal to & Bit and | Bit or ^ Bit xor 
> \>> Bit shift right << Bit shift left ;not support ! Logical NOT && 
> Logical AND || Logical OR + add - sub \* mul / division % mod () parenthesis (byte) 
> (word) (dword) \* get value from point + positive - negative
> ```

```
Ex:
```

> ```
> g if ((byte)\*eip==c2) g if (\*esp==ebx) go if (eip>401000&&eip<<10000000)
> ```

```
未注册版本，条件表达示长度有限制.
```

 |之一 之一 之一 之一 之一 之一 之一 

| ![](img/93ee8f4c6034afd7fa4624e57bd959ec.jpg) |
| --- |之一 
|   || Do statement: |
| 用法: || <statement> 可以是任何有效的 TRW2000 的命令，但必须用双引号""引起来。  
例:  
BPX cs:401000 if (eax>200) do "d eax"   
GOX 401000 if (cs!=28) do "d ss:esp"  
G if (eip<1000)  |之一 之一 之一 之一 之一 之一 之一 

 4dd466d0e1e0cc123afbe504da2462ca