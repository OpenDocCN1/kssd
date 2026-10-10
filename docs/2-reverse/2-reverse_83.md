

# CMDBAR v3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6436.htm`](https://www.pediy.com/kssd/pediy06/pediy6436.htm)

CMDBAR v3.0 命令列表及宏功能相关介绍   
     Author:    Loveboom\[DFCG\]   
？   
? 表达式 计算表达式和 CALC 一样的   
\* 和 ollydbg 的\*功能一样,转到 eip 处   
A   
 命令    参数        功能介绍   
 A      addr,string     在 addr 处进行汇编   
 At     addr             反汇编 addr   
 Asm    String           把汇编代码 string 转换成相应的机器码   
 AC                      分析代码   
B   
 Bp    addr,string       在 addr 处下条件断点，条件为 string，string 为空时就相当于 bpx   
 Bpx   string            下所有调用 string 的断点   
 Bpd   string            删除 BPX 的断点   
 BC    addr               删除 bp 断点   
 BRK                     打开断点窗口   
C   
 C      addr,string       在 addr 处添加注释 string   
 Cpu                     打开 CPU 窗口   
 CS                      打开堆栈调用窗口   
 Close                   关闭加载的可执行程序，相当于 ALT+F2   
 Calc    表达式           和?一样的功能   
D   
 D       addr             在 dump 栏 dump addr 处,和 dump 功能一样   
 Dump    addr             同上   
 Da     \[addr\]            在 dump 栏显示反汇编代码,如果 addr 写上的话就在 dump 栏反汇编地址 addr   
 Db     \[addr\]            在 dump 栏显示为 HEX byte 格式   
 Dc     \[addr\]            以 ascii 方式显示 dump 栏   
 Dd     \[addr\]            以堆栈方式显示 dump 栏   
 Du     \[addr\]            以 Unicode 方式显示 dump 栏   
 Dw     \[addr\]            以 Hex word 格式显示 dump 栏   
 Dasm   string            把机器码转换成相应的汇编代码，和 asm 相反   
E   
 Exit                     退出 ollydbg 和 quit 一样的功能   
F   
 Fr                       查找所选命令/地址的所有调用参考.   
 Follow  addr             和 CTRL+G 类似，跟随到 addr   
G   
 G       \[addr\]           运行到 addr 处   
 Ge      \[addr\]           忽略异常运行到 addr 处   
H   
 H       string           调用相关 api 函数 string 的帮助，前提是你已经选择了 win32 sdk 所在路径.   
 Hd      \[addr\]           删除硬件断点 addr,没有写 addr 就调出硬件断点框   
 He      addr             设置硬件执行断点   
 Help    string           显示 api 函数 string 的帮助，如果没有 string 就显示 cmdbar 的帮助   
 Hr      addr             设置硬件访问断点   
 Hw      addr             设置硬件写断点   
L   
 L       addr,string      在 addr 处插入标签 string   
 Log                      打开 log 窗口   
M   
 Mr      addr,\[表达式\]    内存访问断点，表达式为中断条件   
 Mw      addr,\[表达式\]    内存写断点，表达式为中断条件   
 Md                       删除全部内存断点   
 Mod                      打开模块窗口相当于 ALT+E   
 Mem                      打开内存页窗口相当于 ALT+M   
 Mac     string           执行宏命令,只在新版的 cmdbar 插件中有。   
O   
 Orig                     相当于\*，回到 eip 处   
 Opt                      打开选项框   
 Open    string           打开可执行文件 string，相当于 F3   
P   
 Pause                    暂停执行，相当于 F12   
Q   
 Quit                     退出 ollydbg   
R   
 Run                      运行程序相当于 F9   
 Rst                      重新加载调试的程序,相当于 ctrl+f2   
S   
 Stk    addr              转到地址 ADDR 指在堆栈栏中   
 Stop                     相当于 F12、pause   
 S                        步进相当于 F7   
 Si                       同上   
 So                       步过,相当于 F8   
 Sob                      扫描 obj 文件   
 Sn                       打开当调试模块的所有输入输出列表   
T   
 T       \[addr\]           跟踪步进直到地址 addr 处   
 Ti      \[addr\]           同上   
 To      \[addr\]           跟踪步过直到地址 addr 处   
 Toc     string           跟踪步过直到满足条件 string   
 Tc      string           跟踪步进直到满足条件 string   
 Tr                       执行到返回相当于 ctrl+f9   
 Tu                       执行返回到用户代码相当于 ALT+F9   
W   
 W        string          设置监视表达式 string   
 Watch    string          同上    
  
注：\[xxxx\]的表示可选项.   
  
Cmdbar v3.0 新功能宏的相关介绍   
简单的说，它的宏功能就相当于 N 久以前的在 DOS 下流行的批处理文件.有了它我们可以少做很多重复的事（个人认为脱壳是做重复事最多的了），当然它是没有 ollyscripts 功能那么强大了(Thanks ShaG)。宏的保存在 cmdbar 插件的同一目录中的 macro.def 文件中，宏格式如下(其实就是 ini 文件的"改装版"):   
\[DebugPlugin\]    //这里就是宏名,调用时为 mac DebugPlugin(宏名)   
Line0=BP 492503   //这里的 line(x)就是每一次你要做的事,   
Line2=G   
Line3=AT EAX   
Line4=BC 492503   
这个宏的意思就是先在 492503 处下断,然后运行，中断后反汇编 eax,取消 492503 断点.   
相关问题：   
问题一：宏只会在打开 ollydbg 时加载，如果加载后你再修改宏文件的话，它是不会认的.   
问题二：宏名分大小   
附上我的测试 unpetite v2.2 unpack macro(mac unpetite,测试 OS:win2k adv)和测试程序 macro.def、cmdbar.ini 及 CMDBAR。DLL 放在插件目录中.测试程序为我自己写的 vb6 程序.   
献给所有曾经关心支持和帮助过我的朋友，献给所有 crack 初学者及 DFCG 愿它能够蒸蒸日上!   
  
by loveboom\[DFCG\]   
date:2004-3-1  
    
 4dd466d0e1e0cc123afbe504da2462ca