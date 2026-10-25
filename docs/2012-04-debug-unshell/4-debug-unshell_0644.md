
# 影音传送带(Net Transport) V1.90

> 原文：[`www.pediy.com/kssd/pediy06/pediy6669.htm`](https://www.pediy.com/kssd/pediy06/pediy6669.htm)

下载页面： [`www.skycn.com/soft/9541.html`](http://www.skycn.com/soft/9541.html)

软件大小： 1949 KB

软件语言： 简体中文

软件类别： 国产软件 / 免费版 / 下载工具

应用平台： `Win9x`/`NT`/`2000`/`XP`

加入时间： 2004-06-22 17:58:43

下载次数： 3819145

推荐等级： *****

软件介绍： 影音传送带是一个高效稳定功能强大的下载工具，下载速度一流，CPU 占用率低，尤其在宽带上特别明显；内建易于使用的文件管理器，轻松实现按类别存放下载的文件。而且只要站点支持，这些协议均支持多线程断点续传。`MMS` 协议流每秒的字节数差不多是固定的，这就决定了 `MMS` 流本身不能充分利用用户的带宽资源，所以多线程技术能有效地缩短 `MMS` 流下载的时间。`RTSP` 同样支持多线程，另外选中“流/最大速度”能明显提速，效果立竿见影。影音传送带支持 `HTTPS`（加密套接字协议层），并且支持经过 `HTTP`、`SOCKS4`&`5` 等代理下载；`FTP` 同样支持 `SSL`，可以有效地保护传输的数据安全。

**【作者声明】**： 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**： `WinXP`、`Ollydbg`1.10、`PEiD`、`LordPE`、`ImportREC`

---

## 脱壳过程

`DosKey` 兄弟提出这个程序的 `eXe32Pack` 壳，无法用 `PE-Scan` 自动脱壳，手脱后还有校验无法运行。所以抽点时间看看，恰好 `DLL` 脱壳系列中还没有 `eXe32Pack`，呵呵。目标程序可以去天空下载，主程序 `NetTransport.exe` 和 `libcrypto.dll` 等几个 `DLL` 都是采用 `eXe32Pack` 加壳的。

---

### 一、主程序 `NetTransport.exe` 脱壳

```
004E400C      3BC0              cmp eax,eax//进入 OD 后停在这
004E400E      74 02             je short NetTrans.004E4012
004E4010      8185 553BC074 02818>add dword ptr ss:[ebp+74C03B55],53808102
004E401A      3BD2              cmp edx,edx
004E401C      74 01             je short NetTrans.004E401F
```

直接下断： `BP IsDebuggerPresent`

`F9` 运行，中断后 `Alt+F9` 返回 `00536ECE` 处

```
00536ECC      FFD0              call eax ;IsDebuggerPresent
00536ECE      8BBD D13C4000      mov edi,dword ptr ss:[ebp+403CD1]//返回这里！
00536ED4      03BD F73C4000      add edi,dword ptr ss:[ebp+403CF7]
//EDI=00088EAA + 00400000=00488EAA   这个就是 OEP 值啦 ★
```

直接下命令：`G 00488EAA` 直达 `OEP`！

```
00488EAA      55                push ebp//在这儿用 LordPE 完全 DUMP 这个进程
00488EAB      8BEC              mov ebp,esp
00488EAD      6A FF             push -1
00488EAF      68 B8864900       push NetTrans.004986B8
00488EB4      68 36904800       push NetTrans.00489036 ; jmp to msvcrt._except_handler3
00488EB9      64:A1 00000000     mov eax,dword ptr fs:[0]
00488EBF      50                push eax
00488EC0      64:8925 00000000   mov dword ptr fs:[0],esp
00488EC7      83EC 68            sub esp,68
00488ECA      53                push ebx
00488ECB      56                push esi
00488ECC      57                push edi
00488ECD      8965 E8           mov dword ptr ss:[ebp-18],esp
00488ED0      33DB              xor ebx,ebx
00488ED2      895D FC           mov dword ptr ss:[ebp-4],ebx
00488ED5      6A 02             push 2
00488ED7      FF15 B4EA4800      call dword ptr ds:[48EAB4]; msvcrt.__set_app_type
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00088EAA`，点 `IT AutoSearch`，点“`Get Import`”，`FixDump`！

删除 `eXe32Pack` 壳“”的区段，然后用 `LordPE` 重建 `PE`，简单优化一下脱壳后的文件。`EXE` 脱壳完成啦。

---

### 二、`eXe32Pack` V1.38 加壳的 `DLL` 脱壳

这个才是写这点东西的只要目的啦。

以 `libcrypto.dll` 为例子吧，因为这个 `dll` 对主程序有个校验啦。

---

#### 1、`OEP`

```
1003300C      3BC9              cmp ecx,ecx//进入 OD 后停在这
1003300E      74 02             je short libcrypt.10033012
10033010      8181 553BC974 02818>add dword ptr ds:[ecx+74C93B55],53848102
1003301A      3BC9              cmp ecx,ecx
1003301C      74 01             je short libcrypt.1003301F
```

直接下断： `BP IsDebuggerPresent`

`F9` 运行，中断后 `Alt+F9` 返回 `10045A67` 处

```
10045A65      FFD0              call eax
10045A67      8BBD D13C4000      mov edi,dword ptr ss:[ebp+403CD1]//返回这里！
10045A6D      03BD F73C4000      add edi,dword ptr ss:[ebp+403CF7]
//EDI=0001C7AA + 10000000=1001C7AA   这个就是 OEP 值啦 ★
```

现在 `Ctrl+B` 在“整个段块”搜索 16 进制值：`0F84F7000000`

```
1004573B      74 01             je short libcrypt.1004573E
1004573D      BE 2BBDD53C       mov esi,3CD5BD2B
10045742      40                inc eax
10045743      000F              add byte ptr ds:[edi],cl//找到这里
10045745      84F7              test bh,dh
```

因为有花指令，其实这段代码是：

```
1004573E      2BBD D53C4000      sub edi,dword ptr ss:[ebp+403CD5]
10045744      0F84 F7000000      je libcrypt.10045841
```

在 `1004573E` 处下个“硬件执行”断点，为将要寻找重定位表提供方便啦。

现在去 `OEP` 吧。`G 1001C7AA`

```
1001C7AA      55                push ebp //OEP ★
1001C7AB      8BEC              mov ebp,esp
1001C7AD      53                push ebx
1001C7AE      8B5D 08           mov ebx,dword ptr ss:[ebp+8]
1001C7B1      56                push esi
1001C7B2      8B75 0C           mov esi,dword ptr ss:[ebp+C]
1001C7B5      57                push edi
1001C7B6      8B7D 10           mov edi,dword ptr ss:[ebp+10]
1001C7B9      85F6              test esi,esi
1001C7BB      75 09             jnz short libcrypt.1001C7C6
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `libcrypto.dll`，然后完整脱壳，得到 `dumped.dll`。

---

#### 2、输入表

还是借用 `ImportREC` 吧。

随便从程序找一个 `API` 调用，如：

```
1001C72D      FF15 24F10110      call dword ptr ds:[1001F124];77BFAC46
```

在转存中跟随 `1001F124`，上下看到许多函数地址，很明显的可以找到 `IAT` 开始和结束的地址：

```
1001F000   58 B2 0E 78 09 2D 0C 78 EC 1A 0C 78 B7 1A 0C 78   X?x.-.x?.x?.x
…… ……
1001F120   DB 79 C0 77 46 AC BF 77 88 D3 C2 77 00 00 00 00   踶纖 Fw 堄聎....
```

开始地址＝`1001F000`

结束地址＝`1001F12C`

运行 `ImportREC`，选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取 `DLL`”，选择 `libcrypto.dll`，填入 `RVA`=0001F000、大小＝12C，点“`Get Import`”，改 `OEP`=0001C7AA，`FixDump`！

---

#### 3、重定位表修复

`eXe32Pack` 壳没有加密重定位表，我们所要做的只是找到 `DLL` 原来的重定位表的 `RVA` 和大小就行了！

还记得在 `1004573E` 处下个“硬件执行”断点吧？`OK`，重新载入这个 `DLL`，`F9` 运行，中断在 `1004573E` 处

```
1004573E      2BBD D53C4000      sub edi,dword ptr ss:[ebp+403CD5]; libcrypt.10000000
//中断在这里！
10045744      0F84 F7000000      je libcrypt.10045841
//如与映像基址不符则重定位处理！★ 
//因为这个 DLL 不进行重定位，无论得到重定位表信息，所以我们在这里可以改标志位 Z=0，使其不跳转！
           

1004574A      3BE4              cmp esp,esp
1004574C      74 01             je short libcrypt.1004574F
           

1004574F      8B53 04           mov edx,dword ptr ds:[ebx+4]
10045752      83EA 08           sub edx,8
10045755      3BDB              cmp ebx,ebx; libcrypt.1002C000
//EBX=1002C000             ★ 重定位表的开始地址！
10045757      74 02             je short libcrypt.1004575B
           

1004575B      0BD2              or edx,edx
1004575D      0F8E DE000000      jle libcrypt.10045841
//重定位处理结束则这里跳转！    ★
```

所以我们直接 `F4` 至 `10045841`

```
10045841      80BD F53C4000 00   cmp byte ptr ss:[ebp+403CF5],0
//此时 EBX=1002E470           ★ 这是重定位表的结束地址
10045848      0F84 CB010000      je libcrypt.10045A19
```

得到重定位表信息：

`RVA`=0002C000，大小＝1002E470－1002C000＝2470

用 `LordPE` 修正 `dumped\_.dll` 的重定位表 `RVA`=0002C000、大小＝00002470，保存之。`DLL` 脱壳完成啦！

---

### 三、解除校验

重命名原文件。把 `dumped\_.exe` 改名为 `NetTransport.exe`，把 `dumped\_.dll` 改名为 `libcrypto.dll`

为何重命名 `NetTransport.exe`？呵呵，发现脱壳前的原程序改名后都会异常出错，`NetTransport.exe.manifest` 的缘故吧。为了省点事，就用原来的“名字”了。根据出错时的堆栈情况，判断、寻找程序中出错的地方。直接说修改的地方了。



因为主程序中还有几处调用 `libcrypt.wx_CheckFile` 检测，所以进入 DLL 中修改。

OK，现在脱壳后的 `NetTransport.exe` 和 `libcrypto.dll` 都可以正常运行了。

—————————————————————————————————

```
,     _/ 
      /| \_.-~/            \\\_     ,        青春都一晌
    ( /~   /              \\~-.\_ |\
    \`\\\\  \_/                \   ~\\           忍把浮名 
  _-~~~-.)  \_\_/;;,.           \\\_  //'
 /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂
 \`~ _( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\
  /~~//'   /' \`~\\            ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) /--..\_,  ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)\_  \`~
  "  \`~"  "       \`"      /~'\`\\   \`\\\\~~\

        "     "   "~'  ""
```

Cracked By 巢水工作坊——fly `[OCN]` `[FCG]` `[NUKE]` `[DCM]`

2004-06-24 02:00 4dd466d0e1e0cc123afbe504da2462ca
