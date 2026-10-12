
# 投机取巧，用内存注册机方式破解 `X-NetStat`

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-310.htm`](https://www.pediy.com/kssd/pediy07/pediy7-310.htm)

## 程序说明

`X-NetStat` `Professional` `5.42` crack tutorial

`X-NetStat` `Professional` shows your current Internet and network connections in realtime.

These connections are established each time you visit a web page, send an instant message, check your email, or anytime you do network activity that requires you to connect to another computer. They also appear when outside computers attempt to connect to your machine, authorized or not.

## 破解说明

程序加壳 - 无壳
保护方式 - 试用期 + 注册码
注册保存 - 注册表
保护算法 - `MD5`

破解目的：

使用内存注册机方式，跳过算法分析的鸿沟 ^\_^

破解结果：

`Name` `MrBeer[CCG]`
`Email` `middlebeer@sohu.com`
`Key` `27490-2FFF4-2A4FE-CC645-8291B`

## 破解分析

`47E9EC` `CALL` `xchange`
`5981FC` `CALL` `MD5`

1. 第一步是对程序进行查壳/脱壳，这个对于任何破解应该都是不可免除的一步。
   用 `PEiD` 检测主程序 `xns5.exe`，无壳，`BORLAND` `DELPHI` 开发。

2. 第二步是侦测软件的加密算法，使用 `Kanal`，发现 `SHA1`、`MD5` 等算法。
   但好像通过后来的跟踪分析，发现 `MD5` 与注册算法无关。

3. 第三步是运行程序，进行注册，观察程序反应。
   在 `Register` 界面输入 `FADE` 用户名和注册码，分别为：
   `Name` `MrBeer[CCG]`
   `Email` `middlebeer@sohu.com`
   `Key` `12345678900987654321`
   注册后，程序提示"You registration key is invalid....."

4. 第四步是静态分析，有助于把握软件的整体构架，理清思路。
   启用 `PLL621` 大侠的 `C32ASM`，反汇编，查看字符串。
   搜索"You registration key is invalid....."，BINGO，找到。

5. 第五步动态分析。XP 系统下，当然是 `OllyDbg` 了。
   对访问该字符串的 `CALL` 设断，很容易跟出 Key 的判断跳转语句。
   通过分析，验证过程是（逻辑过程，有程序实际的实现过程有出入)：

   a. 处理 `Name`/`Email` 字符串，得到字符串 `M1`。对 `M1` 进行 `MD5` 运算，运算结果转换为字符串。取该字符串前 5 位作为 `Key` 的第 1 段。
   b. 处理 `Name`/`Email` 字符串，得到字符串 `M2`。对 `M2` 进行 `MD5` 运算，运算结果转换为字符串。取该字符串前 5 位作为 `Key` 的第 2 段。
   c. 处理 `Name`/`Email` 字符串，得到字符串 `M3`。对 `M3` 进行 `MD5` 运算，运算结果转换为字符串。取该字符串前 5 位作为 `Key` 的第 3 段。
   d. 处理 `Name`/`Email` 字符串，得到字符串 `M4`。对 `M4` 进行 `MD5` 运算，运算结果转换为字符串。取该字符串前 5 位作为 `Key` 的第 4 段。
   e. 处理 `Name`/`Email` 字符串，得到字符串 `M5`。对 `M5` 进行 `MD5` 运算，运算结果转换为字符串。取该字符串前 5 位作为 `Key` 的第 5 段。
   f. 将计算得到 5 段 `Key` 与输入的 `Key` 比较，各段相同，则注册成功。
   g. 相关信息写入 `HKEY_CURRENT_USER\Software\X-NetStat` `Professional` `5\Registration`

   因为该程序的注册算法不存在单向的问题，写出注册机是不成问题的。
   不过该程序生成各段 `Key` 的所用的对 `Name`/`Email` 的字符串运算相当复杂，还原出来比较费劲。
   而且该程序才用的 `MD5` 算法与以前遇见的标准 `MD5` 字符串运算都不相同，比较怪异。
   因此，决定偷懒了，采用内存注册机的办法来获得注册码，呵呵。
   总结：该软件的虽然使用强力的 `MD5` 算法，对 `Name`/`Email` 进行 `HASH` 运算，但最后竟然使用明码比较的方式认证注册码，功亏一篑啊。
   只有有动态调试工具，2 分钟就可以跟到自己的注册码。

## 源码分析

通过对字符串"You registration key is invalid....."时段，很容易跟踪到这里：

```
0055B0EC     /.   55                     push ebp                   ; --->整个注册算法开始
0055B0ED     |.   8BEC                   mov ebp,esp
0055B0EF     |.   83C4 E8                add esp,-18
0055B0F2     |.   33C9                   xor ecx,ecx
0055B0F4     |.   894D F8                mov dword ptr ss:[ebp-8],ecx
0055B0F7     |.   894D F4                mov dword ptr ss:[ebp-C],ecx
0055B0FA     |.   894D F0                mov dword ptr ss:[ebp-10],ecx
0055B0FD     |.   8955 E8                mov dword ptr ss:[ebp-18],edx
0055B100     |.   8945 FC                mov dword ptr ss:[ebp-4],eax
0055B103     |.   33C0                   xor eax,eax
0055B105     |.   55                     push ebp
0055B106     |.   68 88B25500            push xns5.0055B288
0055B10B     |.   64:FF30                push dword ptr fs:[eax]
0055B10E     |.   64:8920               mov dword ptr fs:[eax],esp
0055B111     |.   8D55 F8                lea edx,dword ptr ss:[ebp-8]
0055B114     |.   8B45 FC                mov eax,dword ptr ss:[ebp-4]
0055B117     |.   8B80 0C030000          mov eax,dword ptr ds:[eax+30C]
0055B11D     |.   E8 FE74EFFF            call xns5.00452620        ;  --->得到输入的 Name
0055B122     |.   8D55 F4                lea edx,dword ptr ss:[ebp-C]
0055B125     |.   8B45 FC                mov eax,dword ptr ss:[ebp-4]
0055B128     |.   8B80 1C030000          mov eax,dword ptr ds:[eax+31C]
0055B12E     |.   E8 ED74EFFF            call xns5.00452620        ;  --->得到输入的 Email
0055B133     |.   8D55 F0                lea edx,dword ptr ss:[ebp-10]
0055B136     |.   8B45 FC                mov eax,dword ptr ss:[ebp-4]
0055B139     |.   8B80 10030000          mov eax,dword ptr ds:[eax+310]
0055B13F     |.   E8 DC74EFFF            call xns5.00452620        ;  --->得到输入的 Key
0055B144     |.   6A 02                  push 2
0055B146     |.   8B4D F0                mov ecx,dword ptr ss:[ebp-10]
0055B149     |.   8B55 F4                mov edx,dword ptr ss:[ebp-C]
0055B14C     |.   8B45 F8                mov eax,dword ptr ss:[ebp-8]
0055B14F     |.   E8 BC020000            call xns5.0055B410        ; --->关键运算/判断 CALL
0055B154     |.   8845 EF                mov byte ptr ss:[ebp-11],al
0055B157     |.   807D EF 00             cmp byte ptr ss:[ebp-11],0
0055B15B     |.   0F84 C3000000          je xns5.0055B224           ; --->不跳！注册成功，将相关信息写入注册表
0055B161     |.   E8 A2FDFFFF            call xns5.0055AF08
```

关键运算/判断 `CALL`：

```
0055B410      $   55                     push ebp
0055B411      .   8BEC                   mov ebp,esp
0055B413      .   51                     push ecx
0055B414      .   B9 0C000000             mov ecx,0C
0055B419      >   6A 00                  push 0
0055B41B      .   6A 00                  push 0
0055B41D      .   49                     dec ecx
0055B41E      .^  75 F9                  jnz short xns5.0055B419
0055B420      .   51                     push ecx
0055B421      .   874D FC                xchg dword ptr ss:[ebp-4],ecx
0055B424      .   53                     push ebx
0055B425      .   894D F4                mov dword ptr ss:[ebp-C],ecx
0055B428      .   8955 F8                mov dword ptr ss:[ebp-8],edx
0055B42B      .   8945 FC                mov dword ptr ss:[ebp-4],eax
0055B42E      .   8B45 FC                mov eax,dword ptr ss:[ebp-4]
0055B431      .   E8 9A9BEAFF            call xns5.00404FD0
0055B436      .   8B45 F8                mov eax,dword ptr ss:[ebp-8]
0055B439      .   E8 929BEAFF            call xns5.00404FD0
0055B43E      .   8B45 F4                mov eax,dword ptr ss:[ebp-C]
0055B441      .   E8 8A9BEAFF            call xns5.00404FD0
0055B446      .   33C0                   xor eax,eax
0055B448      .   55                     push ebp
0055B449      .   68 9BB75500            push xns5.0055B79B
0055B44E      .   64:FF30                push dword ptr fs:[eax]
0055B451      .   64:8920               mov dword ptr fs:[eax],esp
0055B454      .   C645 E3 00             mov byte ptr ss:[ebp-1D],0
0055B458      .   8D4D EC                lea ecx,dword ptr ss:[ebp-14]
0055B45B      .   8B55 F8                mov edx,dword ptr ss:[ebp-8]
0055B45E      .   8B45 FC                mov eax,dword ptr ss:[ebp-4]
0055B461      .   E8 F2F3FFFF            call xns5.0055A858        ; --->运算 Name & Email 得到 Key
0055B466      .   8D45 E4                lea eax,dword ptr ss:[ebp-1C]
0055B469      .   8B55 EC                mov edx,dword ptr ss:[ebp-14] ; --->EDX=计算得到 Key 地址
0055B46C      .   E8 5797EAFF            call xns5.00404BC8
0055B471      .   8D45 E8                lea eax,dword ptr ss:[ebp-18]
0055B474      .   8B55 F4                mov edx,dword ptr ss:[ebp-C]
0055B477      .   E8 4C97EAFF            call xns5.00404BC8
0055B47C      .   0FB745 08             movzx eax,word ptr ss:[ebp+8]
0055B480      .   83F8 1F                cmp eax,1F                 ;  Switch (cases 0..1F)
0055B483      .   0F87 A6020000          ja xns5.0055B72F
0055B489      .   8A80 96B45500          mov al,byte ptr ds:[eax+55B496>
```



`0055B48F . FF2485 B6B45500 jmp dword ptr ds:[eax*4+55B4B6>]`

`Name&Email`的详细算法不再详细分析，有兴趣的可以自己做。反正很繁琐。

运行过`55B469`，`EDX`中即为正确`Key`的地址。

让我们来内存注册之吧 ^\_^

=============================

=============================

### KeyGen

使用`KeyMake2.0`。

启动内存注册机，添加：

*   中断地址`55B46C`
*   中断次数`1`
*   第一字节`E8`
*   指令长度`5`
*   保存下列信息为注册码 - 内存方式
*   寄存器`EDX`

=============================

=============================

### Greeting

感谢`PEiD`/`C32ASM`/`OllyDbg`及其插件的开发、修改者，没有他们的杰出工作也就没有本文。

=============================

`MrBeer[CCG]`

16/Jul/2005

=============================

`4dd466d0e1e0cc123afbe504da2462ca`
