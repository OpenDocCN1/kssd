
# [原创]Virrut变种感染型病毒分析报告（Virut.ce）

# 1.样本概况

| 病毒名称 | Virus.Win32.Virut.ce |
| ------------- |:-------------:|
| MD5 | 6A500B42FC27CC5546079138370C492F |
| 文件大小 | 131 KB (134,144 字节) |
| 壳信息 | 无壳 |
| 文件名 | `9-29_vir.exe`（感染程序） |
| 数字签名 | NO |
| 时间戳 | 2000年1月10日, 12:00:00 |

## 1.1 样本行为
恶意代码为感染型病毒，母体病毒通过将正常的PE文件进行加密，然后每个受感染的文件又可以进行全盘传播，通过连接国外IP服务器发送上线报文。

## 1.2 测试环境及工具

| 操作系统 | Windows7（x86） |
| ------------- |:-------------:|
| 行为检测 | Wireshark、Api Monitor、Malware Defender |
| 文件检测 | 010 Editor、PEID |
| 虚拟机平台 | VMware10 |
| 调试工具 | OD、IDA |

## 1.3 分析目标
1）样本的网络行为、文件行为
2）感染传播的方式

# 2、具体行为分析

## 2.1 主要行为概述

![](upload/attach/202002/214974_MDGCUWM2H268CG4.jpg)

## 2.2 恶意程序对用户造成的危害

### 2.2.1 感染途径
- 通过母体文件，或被母体文件感染过的文件感染。
- 运行后会将恶意代码以及模块注入`explorer.exe`、`Conhost.exe`、`winlogo.exe`、`dwm.exe`等进程。

### 2.2.2 恶意行为
- 感染已经打开的进程和磁盘可执行文件，重写程序入口点的代码并指向病毒代码。
- 与国外IP发送数据包进行通讯连接。

## 2.3 恶意代码分析

### 2.3.1 恶意程序加固方式
1）加密处理：病毒代码处理经过多层的解密指令，运行过程中对后续代码进行解密，解密代码会变形。例如原反汇编指令代码如下：

```
00424772 imul edx,eax,0xCFA924B3
00424778 mov ecx,0x1A7DDAE2
0042477D sbb byte ptr ss:[ebp-0x223],dh; 带借位减法指令,这里会修改42479D位置的ret变成call
00424783 pop edi
00424784 lea edi,dword ptr ds:[edi-0x69B4>
0042478A test dl,bh ; dl = 4D('M');查找文件头
0042478C mov dl,ah
0042478E adc al,bl
00424790 adc al,cl
00424792 push dword ptr ss:[ebp-0x17D]
00424798 add edi,ebp ; 9-29_vir.004249C0
0042479A xchg cl,cl
0042479C pop edx
0042479D retn ; import - NoTips ---- By Jmp2 -
0042479E add al,0x3
004247A0 add byte ptr ds:[eax],al
004247A2 mov edi,edi
```

执行完【0042477D sbb byte ptr ss:[ebp-0x223],dh】这条语句后，机器码为C3的改成了E8，ret就变成了call。修改后的代码如下：

![](upload/attach/202002/214974_QDAWXQD6FV48YZP.jpg)

图：指令被修改

2）花指令：代码会在指令中增加00，在鼠标滑动时指令发生变化。

![](upload/attach/202002/214974_5WY45UW489JXUZ7.jpg)

图：花指令

### 2.3.2 恶意行为过程截获

- 恶意行为
（1）修改其他进程的内存：

![](upload/attach/202002/214974_43TV6BF5BY6TSZN.jpg)

图：修改其程序内存

（2）API行为监控：

![](upload/attach/202002/214974_Y9BR5DBEYDWNFXN.jpg)

图：API行为监控

（3）感染文件变化：

![](upload/attach/202002/214974_5DDRYE9TVY64398.jpg)

图：源文件与感染文件对比

- 网络行为
通过Malware Defender对被感染的文件进行监控，发现每个被感染的文件自成母体病毒文件向当前的系统进程`dwm.exe`进行了修改。通过`dwm.exe`文件访问国外IP。

![](upload/attach/202002/214974_GGQ3AMMBJFAKCYR.jpg)

图：注入进程，进程访问网络

病毒运行后会访问国外IP：`148.81.111.121`，通过VT查询该IP与多个恶意代码HASH绑定，Wireshark抓包后的数据包：

![](upload/attach/202002/214974_HXD8RSCJUJHKU76.jpg)

图：访问IP

通过Windows系统自带的DNS记录跟踪大量域名后，通过系统命令确定病毒感染后访问的国外IP绑定的域名为`ant.trenz.pl`。

![](upload/attach/202002/214974_TA4SZPAXPKWZRM9.jpg)

图：Windwos自带DNS记录访问的域名

![](upload/attach/202002/214974_5948XWDKKHHT5CC.jpg)

图：系统命令关联域名的IP地址

### 2.3.3 恶意程序的代码分析记录

#### 2.3.3.1 调用病毒代码
病毒代码执行前，首先获取`kernel32.dll`入口循环遍历代码所需函数，通过IDA静态可查看代码运行的结构是在不断进行循环。

![](upload/attach/202002/214974_MNDFD2KNZXZ6HRA.jpg)

图：OD-kernel32入口地址

![](upload/attach/202002/214974_QQNR5QQ6DGTGM7D.jpg)

图：IDA-kernel32函数地址遍历

当面对这类感染类病毒，很多使用寄存器进行传值的指令是无法用IDA轻轻松松F5查看源码的。

所以分析过程中我采用通过行为监控+动态调试的思路分析！打开Malware Defender，借助吾爱破解OD进行F8单层步过直到程序结束。当Malware Defender出现关键行为时，对走过的call进行记录，然后深入这个call再逐层分析。

跟踪到第6个函数时，程序出现了修改其他进程内存、其他进程中创建线程的行为。

![](upload/attach/202002/214974_TAZPFX7WWXRTGD2.jpg)

图：第6个call为重点分析函数

通过反复还原虚拟机快照对比，发现该病毒在感染后 `cmp al,bl`执行的分支是不一样的，如果没有被感染的系统`JNZ`不跳转程序继续向下执行，否则`JNZ`则跳转到下一个分支处。对比值是`eax`与`ebx`的寄存器的低位值。此时的AL与BL的值分别为：Al=B7，Bl=00。

为了验证这个【B7】的值是否为随机值，切换到另外一台虚拟机再次单步跟踪有操作EAX的的位置，发现在00424A38地址处执行完【`mov eax,dword ptr ds:[eax+0x34];`】，B7的值就没有发生过改变，由此得出B7这个值就是病毒判断系统有没有被感染的标识，对比反汇编代码如下：

```
00424A38 mov eax,dword ptr ds:[eax+0x34]; 获取感染标志位
00424A3B jmp short 9-29_vir.00424A5D
00424A3D and al,0xFF
00424A3F cmc
00424A40 mov edx,0x8B090FE1
00424A45 sete dh
00424A48 inc ch
00424A4A push 0xC0FCDCDC
00424A4F call 9-29_vir.00424845 ; ----
00424A54 and al,al
00424A56 xor ecx,ebp ; 9-29_vir.004249C0
00424A58 call 9-29_vir.00424841 ; -----
00424A5D cmp al,bl ; 判断是否被感染
00424A5F jnz short 9-29_vir.00424A6A; 已感染则跳转
00424A61 xchg dh,al ; 该分支1在未感染时,执行
00424A63 xchg esi,edx
00424A65 call 9-29_vir.00424702 ; ;重点分析函数;import
00424A6A push 0xBC8257D8 ; 感染后执行的分支2
00424A6F jmp short 9-29_vir.00424A74
00424A71 mov ebx,ebx ; kernel32.75EC0000
00424A73 clc
00424A74 xchg ch,cl
```

所有寄存器当前的值如下：

```
EAX 000000B7
ECX 0012FF70
EDX 00000000
EBX 75EC0000 kernel32.75EC0000
ESP 0012FF38
EBP 004249C0 9-29_vir.004249C0
ESI E2FCE64A
EDI 00000506
EIP 00424A5D 9-29_vir.00424A5D
C 1 ES 0023 32位 0(FFFFFFFF)
P 1 CS 001B 32位 0(FFFFFFFF)
A 0 SS 0023 32位 0(FFFFFFFF)
Z 1 DS 0023 32位 0(FFFFFFFF)
S 0 FS 003B 32位 7FFDE000(4000)
T 0 GS 0000 NULL
D 0
O 0 LastErr ERROR_ALREADY_EXISTS (000000B7)
EFL 00000247 (NO,B,E,BE,NS,PE,GE,LE)
ST0 empty 0.0
ST1 empty 0.0
ST2 empty 0.0
ST3 empty 0.0
ST4 empty 0.0
ST5 empty 0.0
ST6 empty 0.0
ST7 empty 0.0
3 2 1 0 E S P U O Z D I
FST 0000 Cond 0 0 0 0 Err 0 0 0 0 0 0 0 0 (GT)
FCW 027F Prec NEAR,53 掩码 1 1 1 1 1 1
```

当前已知关键感染代码都在00424A65地址的call内，那么按照原有思路继续根据行为找关键call，F8单步跟踪所有的call后借助监控行为软件的结果细化分析各层call内的反汇编指令。

#### 2.3.3.2 代码混淆
病毒代码处理存在两种方式，一种是对后续代码进行解密，解密代码会变形。另一种是后续代码是明文未加密直接调用明文。例如原反汇编指令代码如下：

```
00424772 imul edx,eax,0xCFA924B3
00424778 mov ecx,0x1A7DDAE2
0042477D sbb byte ptr ss:[ebp-0x223],dh ; 带借位减法指令,这里会修改42479D位置的ret变成call
00424783 pop edi
00424784 lea edi,dword ptr ds:[edi-0x69B4>
0042478A test dl,bh ; dl = 4D('M');查找文件头
0042478C mov dl,ah
0042478E adc al,bl
00424790 adc al,cl
00424792 push dword ptr ss:[ebp-0x17D]
00424798 add edi,ebp ; 9-29_vir.004249C0
0042479A xchg cl,cl
0042479C pop edx
0042479D retn ; import - NoTips ---- By Jmp2 -
0042479E add al,0x3
004247A0 add byte ptr ds:[eax],al
004247A2 mov edi,edi
```

执行完【0042477D sbb byte ptr ss:[ebp-0x223],dh】这条语句后，机器码为C3的改成了E8，ret就变成了call。修改后的代码如下：

![](upload/attach/202002/214974_GBDDJ5XQARNKTCG.jpg)

图：指令被修改

在跟随中遇到获取`kernel32.dll`解密代码的算法。反汇编代码如下：

```
00424896 xor eax,eax ; kernel32.BaseThreadInitThunk
00424898 imul eax,eax,0xF ; eax == eax*0xF
0042489B lea ebx,dword ptr ds:[esi-0x18]; ebx = esi - 0x18
0042489E inc ebx ; ebx = ebx + 1
0042489F lea ebx,dword ptr ds:[eax-0x3] ; ebx = eax-0x3
004248A2 movzx ecx,byte ptr ds:[esi+0x3]; Kernel32函数存放
004248A6 sbb bh,0xCB ; bh = bh - bh - CF
004248A9 dec ebx ; ebx = ebx - 1
004248AA adc bl,0x25 ; bl =bl + 0x25 + CF
004248AD not ebx ; 位翻转指令，将操作数中的每一位翻转
004248AF sub eax,ecx ; eax = eax - ecx
004248B1 sub esi,-0x12
004248B4 lea esi,dword ptr ds:[esi-0x11]; 取有效地址
004248B7 cmp byte ptr ds:[esi+0x3],0xA ; 获取所有kernel32.dll函数
004248BB jnb short 9-29_vir.00424898 ;大于或等于转移，因为kernel32.dll函数后的下一个是0xA(换行符)
```

![](upload/attach/202002/214974_8N4KBBHJ8VTBV42.jpg)

图：OD-反汇编代码

指令块通过`jmp`连接，跳转后有指令变形的红色提示。

![](upload/attach/202002/214974_7FDSJXBZ6WJSFTA.jpg)

图：通过JMP连接

#### 2.3.3.3 挂钩底层API函数
进入代码变形后的指令继续跟踪，发现挂钩了底层API函数。对应的七个函数分别为`ZwCreateFile`、`ZwOpenFile`、`ZwCreateProcess`、`ZwCreateProcessEx`、`ZwCreateUserProcess`、`ZwQueryInformationProcess`、`ZwDeviceIoControlFile`；反汇编指令如下：
