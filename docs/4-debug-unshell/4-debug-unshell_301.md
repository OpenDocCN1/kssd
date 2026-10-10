

# Authorware 4 中 U32

> 原文：[`www.pediy.com/kssd/pediy02/forum256.htm`](https://www.pediy.com/kssd/pediy02/forum256.htm)

 对 Authorware 4 中 U32 类型文件的脱壳测试    ----- 告别前的最后一篇脱文   
  
【声明】   
我写文章以交流为主，希望大家在转载时能保持文章的完整性。   
  
【前言】   
由于 Authorware 4 中的 U32 类型文件其实是动态链接库文件，因此对这些 U32 文件的脱壳其实和对 DLL 脱壳的方法相似。   
对 Authorware 我实在不熟悉，花了半天时间，才弄清楚怎样在 Authorware 中加载 U32 类型文件。   
由于上次我已经用 PECompact 加壳一个 DLL 脱壳测试过。这次就换另一种加壳程序试试。我选中了 Aspack v2.1。   
  
样例文件：    ODBC.U32    （Authorware v4.0 所带文件）   
加壳方式：    Aspack v2.1 加壳   
调试工具：    SoftICE v4.05,PEditor v1.5,Hex WorkShop 2.10   
目标：        脱壳   
作者：        ljttt   
写作日期：    2000-08-16   
  
（简称说明：EP: Entry Point，DLL: 动态链接库文件）   
  
1、首先当然要分析基本信息了。用 PEditor 打开动态链接库文件，得到如下信息   
Entry Point:    0000C001   
Image Base:    10000000   
Size of Image:    0000F000   
  
Section        Virtual Size    Virtual Offset   
.idata        00001000    00009000   
  
2、现在我们要在动态链接库的入口点处中断，这次我用《对 PECompact 加壳的 DLL 脱壳的一点分析》一文中的第二种方法。   
  
一、首先，用 PEditor 打开 ODBC.U32 文件，然后单击 FLC 按钮（这个功能是帮你计算 Virtual Address 转换为 Offset 的）   
输入 1000C001 ，单击 DO! 按钮，得到 Offset\[hex\] 为 4001。   
二、然后，用 Hex WorkShop 打开 ODBC.U32 文件，定位到 4001，记下此处字节的值 60。   
三、然后，用 Hex WorkShop 把 4001 处的字节 60 改为 CC。   
四、Ctrl-D 中断进入 SoftICE，设断点 bpint 3。   
五、F5 回到 Windows，运行 Authorware 4.exe。（主程序文件）   
  
这时，程序还没有加载 ODBC.U32 文件，单击菜单 Insert / Load Model，选择主程序目录下的 TEMPLATE/CONSUMER 子目录下的 QSADCONS.A4D 文件，然后单击菜单 Insert / Paste Model，选择 QS:Administration - Cons.。程序开始加载 ODBC.U32，此时将中断进入 SoftICE。   
  
3、下指令，修改程序入口的指令 CC 为 60。   
eb eip 60   
修改后代码窗口显示如下：   
015F:03C6C001  60                  PUSHAD                <--我们中断在此   
015F:03C6C002  E872050000          CALL      03C6C579   
015F:03C6C007  EB33                JMP      03C6C03C   
015F:03C6C009  87DB                XCHG      EBX,EBX   
015F:03C6C00B  90                  NOP   
015F:03C6C00C  0000                ADD      \[EAX\],AL   
015F:03C6C00E  0000                ADD      \[EAX\],AL   
  
注意我们中断时的 IP 为 03C6C001，而不是 10000000+C001=1000C001，这说明 Windows 加载 ODBC.U32 的基地址不是 10000000，我们来计算一下实际加载的基地址： 3C6C001-C001=3C60000。（我跟踪发现每次加载的基地址都不一样）   
  
4、继续跟踪，到如下（也可用 bpm 3C60000+9000 来跟踪 .idata 的变化情况来快速跟踪）   
015F:03C6C2C9  F3A4                REPZ MOVSB   
015F:03C6C2CB  5E                  POP      ESI   
015F:03C6C2CC  6800800000          PUSH      00008000   
015F:03C6C2D1  6A00                PUSH      00   
015F:03C6C2D3  FFB5E1394400        PUSH      DWORD PTR \[EBP+004439E1\]   
015F:03C6C2D9  FF95ED394400        CALL      \[EBP+004439ED\]   
015F:03C6C2DF  83C608              ADD      ESI,08   
015F:03C6C2E2  833E00              CMP      DWORD PTR \[ESI\],00   
015F:03C6C2E5  0F8526FFFFFF        JNZ      03C6C211   
015F:03C6C2EB  6800800000          PUSH      00008000   
015F:03C6C2F0  6A00                PUSH      00   
015F:03C6C2F2  FFB5E5394400        PUSH      DWORD PTR \[EBP+004439E5\]   
015F:03C6C2F8  FF95ED394400        CALL      \[EBP+004439ED\]        <---在此处停下来   
015F:03C6C2FE  8B9D60394400        MOV      EBX,\[EBP+00443960\]   
015F:03C6C304  0BDB                OR        EBX,EBX   
015F:03C6C306  7408                JZ        03C6C310   
  
此时 3C69000 （.idata 所在的内存地址） 处的内容如下   
015F:03C69000 00009064  00000000  00000000  000092A2      d...............   
015F:03C69010 00009150  00009148  00000000  00000000      P...H...........   
015F:03C69020 000092C0  00009234  00009108  00000000      ....4...........   
  
5、在此处我们要得到还原好的 import table。下指令   
/dump 3C69000 1000 c:\\temp\\dump.bin   
  
6、继续跟踪，到如下   
015F:03C6C4EA  59                  POP      ECX   
015F:03C6C4EB  0BC9                OR        ECX,ECX   
015F:03C6C4ED  89852F3E4400        MOV      \[EBP+00443E2F\],EAX   
015F:03C6C4F3  61                  POPAD                                          
015F:03C6C4F4  7508                JNZ      03C6C4FE                (JUMP )    
015F:03C6C4F6  B801000000          MOV      EAX,00000001   
015F:03C6C4FB  C20C00              RET      000C   
\==> 03C6C4FE  68001CC603          PUSH      03C61C00            <---3C61C00 就是我们要找的 OEP 了   
015F:03C6C503  C3                  RET                    <---在此处停下来   
  
7、这里到了我们得到整个程序内存映象的时候了。   
/dump 3C60000 F000 c:\\temp\\dump.u32   
记下此处我们得到 OEP RVA = 3C61C00 - 3C60000 = 1C00。（也就是要减去实际基地址 3C60000）   
  
8、F5 回到 Windows，结束主程序。开始修补 dump.u32。用 PEditor 打开 dump.u32 文件，选择 sections，右键打开菜单，选择 dumpfixer。OK!一次完成所有 Section 的 RVA、Size 的转换工作。再修改 Entry Point 为 1C00，单击 apply changes 保存，选择 directory，修改其中的 Import Table 的 RVA 为 9000，Size 为 1000。单击保存。   
  
9、用 Hex WorkShop 打开 dump.u32 和 dump.bin，定位 dump.dll 位置到 9000，选择 1000 个字节。删除。   
选择 dump.bin 的 1000 个字节，复制到 dump.dll 中。保存。   
  
10、把 dump.u32 复制到主程序目录下，换名为 ODBC.U32。重新运行主程序 Authorware 4.exe。测试成功。   
  
【后记】   
从测试的几个 DLL 脱壳情况来看，加壳程序一般没有对 Export table 做处理。不知道 Export table 象 import table 一样加密后会怎么样。我还不太了解 Windows 对 Export table 的加载处理机制。万一遇到这种情况，我一定死翘翘了。不过还好，实验测试中没发现此类事件。不过真正的商业软件加的壳就不一定了。   
但愿等我把 export table 的加载处理机制弄清楚了，再遇到这类软件。到那时，（俺手拿牛二尖刀，一脸阴笑）嘿嘿...................   
  
【本月小结】   
不知不觉，一个月就过去了。算下来俺也写十余篇脱文。感觉进步不少，这里要多谢看雪教学中各位高手精彩的文章给了我很大的帮助，投桃报李，因此俺也把自己的一点点所得写了出来。无奈俺用隔月上网的方式来减少网费，所以如果哪位朋友给我写信，可能只有等到我 9 月 20 日再次上网以后才能回复了。Email:ljtt@yeah.net   4dd466d0e1e0cc123afbe504da2462ca