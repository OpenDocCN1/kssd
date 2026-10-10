

# 360SelfProtection.sys 无法使用 Driver Monitor 加载

> 原文：[`www.pediy.com/kssd/pediy12/128515.html`](https://www.pediy.com/kssd/pediy12/128515.html)

虚拟机里使用 Driver Monitor 加载 360SelfProtection.sys，外面使用 windbg 下了延迟断点。断点下在了 DriverEntry 第一句。结果。。。  
使用 Driver Monitor 加载时没有命中断点。。。  
之后 Driver Monitor 还提示连到系统上的设备没有发挥作用。。。。  
![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  
求高人指点。。。 

windbg 连上 VM,在虚拟机启动的时候 break 一下,然后输入:  
**sxe ld 360SelfProtection **  
**g**  
   
不一会就会断下,如下  
kd> **lmvm 360SelfProtection **  
   
start end module name  
**f67b4000** f67d1980 360SelfProtection (no symbols)   
Loaded symbol image file: 360SelfProtection.sys  
Image path: 360SelfProtection.sys  
Image name: 360SelfProtection.sys  
Timestamp: Tue Jan 11 19:36:54 2011 (4D2C40D6)  
CheckSum: 00021EF4  
ImageSize: 0001D980  
File version: 1.0.0.1054  
Product version: 1.0.0.1054  
File flags: 0 (Mask 3F)  
File OS: 40004 NT Win32  
File type: 3.8 Driver  
File date: 00000000.00000000  
Translations: 0804.04b0  
CompanyName: 360 安全中心  
ProductName: 360 安全中心  
InternalName: SelfProtection  
OriginalFilename: SelfProtection.sys  
ProductVersion: 1, 0, 0, 1054  
FileVersion: 1, 0, 0, 1054  
PrivateBuild: 1, 0, 0, 1054  
SpecialBuild: 1, 0, 0, 1054  
FileDescription: 360 安全卫士 - SelfProtection  
LegalCopyright: 版权所有 (C) 2006-2010 360 安全中心  
LegalTrademarks: 版权所有 (C) 2006-2010 360 安全中心  
Comments: 版权所有 (C) 2006-2010 360 安全中心  
   
kd> kvn 100  
# ChildEBP RetAddr Args to Child   
00 f819c398 80527fce f819c430 f819c3ac 00000003 nt!DebugService2+0x10 (FPO: \[3,0,0\])  
01 f819c3bc 805a3cea f819c430 f67b4000 ffffffff nt!DbgLoadImageSymbols+0x42 (FPO: \[3,4,0\])  
02 f819c560 80576254 f819c5e4 00000000 00000000 nt!MmLoadSystemImage+0xa34 (FPO: \[Non-Fpo\])  
03 f819c640 80689770 000006c8 00000001 00000000 nt!IopLoadDriver+0x370 (FPO: \[4,45,0\])  
04 f819c69c 80686ad9 00043000 00000000 00000000 nt!IopInitializeSystemDrivers+0x16c (FPO: \[0,14,4\])  
05 f819c83c 80684edd 80087000 00000000 81c4f3e8 nt!IoInitSystem+0x7a3 (FPO: \[1,99,4\])  
06 f819cdac 805c5a28 80087000 00000000 00000000 nt!Phase1Initialization+0x9b5 (FPO: \[1,342,4\])  
07 f819cddc 80541fa2 80684528 80087000 00000000 nt!PspSystemThreadStartup+0x34 (FPO: \[Non-Fpo\])  
08 00000000 00000000 00000000 00000000 00000000 nt!KiThreadStartup+0x16  
   
   
kd> **bp f67b4000+poi(poi(f67b4000+3c)+f67b4000+28)**  
kd> bl  
0 e f67cee85 0001 (0001) 360SelfProtection+0x1ae85  
   
kd> u f67cee85  
360SelfProtection+0x1ae85:  
f67cee85 a188ad7cf6 mov eax,dword ptr \[360SelfProtection+0x16d88 (f67cad88)\]  
f67cee8a 85c0 test eax,eax  
f67cee8c b94ee640bb mov ecx,0BB40E64Eh  
f67cee91 7404 je 360SelfProtection+0x1ae97 (f67cee97)  
f67cee93 3bc1 cmp eax,ecx  
f67cee95 7519 jne 360SelfProtection+0x1aeb0 (f67ceeb0)  
f67cee97 a1a48a7cf6 mov eax,dword ptr \[360SelfProtection+0x14aa4 (f67c8aa4)\]  
f67cee9c 8b00 mov eax,dword ptr \[eax\]  
   
kd> g  
Breakpoint 0 hit  
360SelfProtection+0x1ae85:  
f67cee85 a188ad7cf6 mov eax,dword ptr \[360SelfProtection+0x16d88 (f67cad88)\]  
   
kd> **kvn 100**  
# ChildEBP RetAddr Args to Child   
WARNING: Stack unwind information not available. Following frames may be wrong.  
00 f819c640 80689770 000006c8 00000001 00000000 **360SelfProtection+0x1ae85**  
01 f819c69c 80686ad9 00043000 00000000 00000000 nt!IopInitializeSystemDrivers+0x16c (FPO: \[0,14,4\])  
02 f819c83c 80684edd 80087000 00000000 81c4f3e8 nt!IoInitSystem+0x7a3 (FPO: \[1,99,4\])  
03 f819cdac 805c5a28 80087000 00000000 00000000 nt!Phase1Initialization+0x9b5 (FPO: \[1,342,4\])  
04 f819cddc 80541fa2 80684528 80087000 00000000 nt!PspSystemThreadStartup+0x34 (FPO: \[Non-Fpo\])  
05 00000000 00000000 00000000 00000000 00000000 nt!KiThreadStartup+0x16  
   
ok,进入此驱动内部,调试吧.