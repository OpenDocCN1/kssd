
# PIMOne V2.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6926.htm`](https://www.pediy.com/kssd/pediy06/pediy6926.htm)

软件名称： `PIMOne V2.3`  
下载页面： [`www.onlinedown.net/soft/30544.htm`](http://www.onlinedown.net/soft/30544.htm)  
软件大小： 4875KB  
软件语言： 英文  
软件类别： 国外软件/共享版/记事管理  
运行环境： `Win9x/Me/NT/2000/XP`  
加入时间： 2004-11-9 8:09:03  
软件评级： `*****`  

软件介绍： PIM（个人信息管理）软件，为用户提供了联系地址手册、计划任务定时提醒等功能。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`flyODBG`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

## 脱壳过程

2004 远去了，逝者如斯，不必感叹了。写了篇简单的教程，作为迎接 2005 的小礼物吧。

感谢好友 `fxyang` 和 `stephenteh`。

`Armadillo` 的输入表乱序有点麻烦，不少朋友采用变换的方法来修复，其实用 `ImportREC` 就可以修复了。

`PIMOne V2.3` 使用了双进程、IAT 乱序、远程跳转，但是没有 CC。

—————————————————————————————————

### 一、把双进程切换成单进程

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug V1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
006013F3     55               push ebp

//进入 Ollydbg 后暂停在这

006013F4     8BEC             mov ebp,esp

006013F6     6A FF            push -1

006013F8     68 20BB6200      push PIMOne.0062BB20

006013FD     68 30116000      push PIMOne.00601130

00601402     64:A1 00000000   mov eax,dword ptr fs:[0]

00601408     50               push eax

00601409     64:8925 00000000 mov dword ptr fs:[0],esp

00601410     83EC 58          sub esp,58

00601413     53               push ebx

00601414     56               push esi

00601415     57               push edi

00601416     8965 E8          mov dword ptr ss:[ebp-18],esp

00601419     FF15 88616200    call dword ptr ds:[<&KERNEL32.GetVersion>]
```

下断：`BP OpenMutexA`

`Shift+F9`，中断后看堆栈：

```
0012D7AC    005ECFD1  /CALL 到 OpenMutexA 来自 PIMOne.005ECFCB
0012D7B0    001F0001  |Access = 1F0001
0012D7B4    00000000  |Inheritable = FALSE
0012D7B8    0012DDEC  \MutexName = "22C::DA2D80EF7F" ★ 注意 0012DDEC
```

`Ctrl+G`：`401000`  键入以下代码

```
00401000     60               pushad

00401001     9C               pushfd

00401002     68 ECDD1200      push 12DDEC ★ 堆栈里看到的值

00401007     33C0             xor eax,eax

00401009     50               push eax

0040100A     50               push eax

0040100B     E8 2FDB407C      call kernel32.CreateMutexA

00401010     9D               popfd

00401011     61               popad

00401012     E9 04DC407C      jmp kernel32.OpenMutexA

60 9C 68 EC DD 12 00 33 C0 50 50 E8 2F DB 40 7C 9D 61 E9 04 DC 40 7C
```

在 `401000` 处新建起源，`F9` 运行，再次中断在 `OpenMutexA` 处。

返回 `401000` 处，“撤销选择”，清除掉写入的代码。`BC OpenMutexA`，取消这个断点。

—————————————————————————————————

### 二、Armadillo V4.0 新增的反跟踪手段

`OllyDbg` 在处理调式包含格式串的消息时存在问题，被跟踪的应用程序可以使 `OllyDbg` 崩溃，或可能以进程权限执行任意指令。`OutputDebugString` 函数可发送字符串到调试器上，然后 `OllyDbg` 会在底端显示相关状态消息，但是如果包含格式串消息，就可能使 `OllyDbg` 崩溃。

`Armadillo` 以前的版本没有此种 Anti，自 `V4.0` 始才有。

`HE OutputDebugStringA`

`Shift+F9` 运行，中断下来。看堆栈：

```
0012CCE4    00DF1F59  /CALL 到 OutputDebugStringA 来自 00DF1F53
0012CCE8    0012D60C  \String = "%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%>
```

转存跟随 `12D60C`，选中`%s`之类的字符，点右键－>二进制－>使用 `00` 填充

下面的同样处理：

```
0012CCE4    00DF251F  /CALL 到 OutputDebugStringA 来自 00DF2519
0012CCE8    0012D60C  \String = "%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%>
```

保留这个 `OutputDebugStringA` 断点，后面还有用。

—————————————————————————————————

### 三、Magic Jump，避开 IAT 加密

下断：`HE GetModuleHandleA`

`Shift+F9`，弹出 `Reminder` 界面，点 `OK`

中断下来，看堆栈：

```
001265F4    00DDABDD  /CALL 到 GetModuleHandleA 来自 00DDABD7
001265F8    00126738  \pModule = "kernel32.dll"
```

`HD GetModuleHandleA` 取消这个断点，`Alt+F9` 返回

```
00DDABD7     FF15 D820E000    call dword ptr ds:[E020D8]    ; kernel32.GetModuleHandleA
00DDABDD     8B0D E4C9E000    mov ecx,dword ptr ds:[E0C9E4]

//返回这里

00DDABE3     89040E           mov dword ptr ds:[esi+ecx],eax
00DDABE6     A1 E4C9E000      mov eax,dword ptr ds:[E0C9E4]
00DDABEB     391C06           cmp dword ptr ds:[esi+eax],ebx
00DDABEE     75 16            jnz short 00DDAC06
00DDABF0     8D85 B4FEFFFF    lea eax,dword ptr ss:[ebp-14C]
00DDABF6     50               push eax
00DDABF7     FF15 E020E000    call dword ptr ds:[E020E0]     ; kernel32.LoadLibraryA
00DDABFD     8B0D E4C9E000    mov ecx,dword ptr ds:[E0C9E4]
00DDAC03     89040E           mov dword ptr ds:[esi+ecx],eax
00DDAC06     A1 E4C9E000      mov eax,dword ptr ds:[E0C9E4]
00DDAC0B     391C06           cmp dword ptr ds:[esi+eax],ebx
00DDAC0E     0F84 32010000    je 00DDAD46
```

`//Magic Jump！ 修改为：jmp 00DDAD46 ★`

```
00DDAC14     33C9             xor ecx,ecx
00DDAC16     8B07             mov eax,dword ptr ds:[edi]
00DDAC18     3918             cmp dword ptr ds:[eax],ebx
00DDAC1A     74 06            je short 00DDAC22
00DDAC1C     41               inc ecx
00DDAC1D     83C0 0C          add eax,0C
00DDAC20     EB F6            jmp short 00DDAC18
```

—————————————————————————————————

### 四、飞向光明之巅

`Shift+F9` 运行，再次中断在 `OutputDebugStringA` 处

```
00126888    00DF6B53  /CALL 到 OutputDebugStringA 来自 00DF6B4D
0012688C    0012920C  \String = "%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%>
```

和第二步的处理相同

下断：`BP GetCurrentThreadId`

`Shift+F9` 运行，中断在 `GetCurrentThreadId` 处，堆栈：

```
0012D59C    00DFC3DD   CALL 到 GetCurrentThreadId 来自 00DFC3D7
```

取消这个断点。`Alt+F9` 返回，`Ctrl+S` 在当前位置下搜索命令序列：

```
sub edi, ecx
call edi
```

找到在 `00DFC48C` 处，下断。`Shift+F9` 运行，中断下来

```
00DFC47D     50               push eax
00DFC47E     A1 F81EE100      mov eax,dword ptr ds:[E11EF8]
00DFC483     8B48 5C          mov ecx,dword ptr ds:[eax+5C]
00DFC486     3348 40          xor ecx,dword ptr ds:[eax+40]
00DFC489     3348 08          xor ecx,dword ptr ds:[eax+8]
00DFC48C     2BF9             sub edi,ecx

//EDI=5EA37C21-5E4965CD=005A1654

00DFC48E     FFD7             call edi  ; PIMOne.005A1654

//飞向光明之点巅！

005A1654     55               push ebp

//OEP   ★   用 LordPE 完全 Dump 这个进程

005A1655     8BEC             mov ebp,esp
005A1657     83C4 F0          add esp,-10
005A165A     B8 AC0D5A00      mov eax,PIMOne.005A0DAC
005A165F     E8 B85EE6FF      call PIMOne.0040751C
005A1664     E8 BB97FFFF      call PIMOne.0059AE24
005A1669     E8 6634E6FF      call PIMOne.00404AD4
```

程序内部还有远程跳转，如：

```
00407598     E9 E6BC9902      jmp 02DA3283
```

用 `LordPE` 部分脱壳出：`Region02DA0000-02DC0000.dmp`

注意：这个地址并不是固定的

组装一下，用 `LordPE` 从磁盘载入部分脱壳的文件

修正其 `Virtual Address=02DA0000-00400000=029A0000`

只保留 `LordPE` 的“`Validate PE`”选项，重建 `PE`。

—————————————————————————————————

### 五、输入表乱序的简便修复方法

从程序找一个 API 调用，如：

```
00407458     FF25 78D7EF00    jmp dword ptr ds:[EFD778]; kernel32.GetModuleHandleA
```

在转存中跟随 `0EFD778`，上下看到许多函数地址，可以找到 IAT 开始和结束的地址：

```
00EFD4D8   00 00 00 00 00 00 00 00 5C 01 07 00 F1 07 1C 01   .\\.?
00EFD4E8   EF 01 D3 77 BD BC D1 77 49 EF D1 77 BF 5E D5 77   ?觲郊褀 I 镅 w 縙誻
…… ……
00EFDFA8   37 97 80 7C AB AB AB AB AB AB AB AB EE FE EE FE   7 梹|铪铪
00EFDFB8   00 00 00 00 00 00 00 00 06 00 5C 01 95 07 1C 01   .\\.?
```

开始地址＝`00EFD4E8`

结束地址＝`00EFDFAC`

运行 `ImportREC`。注意：去掉“使用来自磁盘的 PE 部首”的选项，选择“创建新的 IAT”选项！

选中 `Ollydbg` 调试的 `PIMOne.exe` 进程，填入 `RVA＝00AFD4E8`、大小＝`00000AC4`，点“`Get Import`”。

可以看到函数乱序了，`ImportREC` 显示是无效函数。如：

```
1  00AFD634  user32.dll  0001  ActivateKeyboardLayout
1  00AFD638  kernel32.dll  0032  CloseHandle //乱序
0  00AFD63C  ?  0000  00DDBFAE          //垃圾指针
1  00AFD640  gdi32.dll  00DE  ExtTextOutA //乱序
```

先不管乱序，现在需要把填充在里面的垃圾指针全部 `CUT` 掉，有不少，细心点。

为了能够跨平台运行，需要修改某些函数，如：

```
1  00AFD554  ntdll.dll  02B6  RtlLeaveCriticalSection

//修改为：kernel32.dll LeaveCriticalSection
```

修改 `OEP=001A1654`，`FixDump`！

`ImportREC` 自动新建了一个输入表。

可以删除 `text` 和其下的 `adata`、`data1`、`.reloc1`、`pdata` 共 5 个区段，然后再用 `LordPE` 或者 `FileScanner` 优化一下脱壳后的文件。

—————————————————————————————————

### 六、破解

脱壳之后时间限制就没有了。那些字符提示就随意修改吧。

脱壳后的文件放在`[FCG]`、赢政论坛，想参考的朋友可以去下载。



```
005773C8     BA BC745700       mov edx,dumped\_.005774BC ; ASCII "Unregistered version"  

005773CD     8B83 F8020000     mov eax,dword ptr ds:\[ebx+2F8\]  

005773D3     E8 A0E7EDFF       call dumped\_.00455B78  

005773D8     E8 3F9BFCFF       call dumped\_.00540F1C  

005773DD     50               push eax  

005773DE     B8 14000000       mov eax,14  

005773E3     5A               pop edx  

005773E4     2BC2             sub eax,edx  

005773E6     8D55 EC          lea edx,dword ptr ss:\[ebp-14\]  

005773E9     E8 4A29E9FF       call dumped\_.00409D38  

005773EE     8D45 EC          lea eax,dword ptr ss:\[ebp-14\]  

005773F1     BA DC745700       mov edx,dumped\_.005774DC ; ASCII " day(s) left."  

005773F6     E8 1DDBE8FF       call dumped\_.00404F18  
```

这篇教程几天前就开始写了，直至今天才修改完毕，《看雪论坛精华六》收集文章的截止日期 2004 年 12 月 31 日，此篇可算是最后一篇了。祝福朋友们在新的岁月里好运！

—————————————————————————————————     
     
     ,      \_/   
       /| \_.-~/            \\_      ,        青春都一晌  
      ( /~   /               ~.\_ |\\  
       \\\\   \_/                \\   ~\\ )            忍把浮名  
   _-~~~-.)   )\_\_/;;,.            \\_  //  
  /'\_,\\   --~   \\ ~~~-  ,;;\\__\_\_\_(  (.-~~~-.         换了破解轻狂  
 \`~ _( ,_..--\\  (   ,;'' /    ~--   /.\\_\`\\  
   /~~//'   /` \\           ) /--..\_, )_  \`~  
   "  \`~"  "       `"      /~'\`\\    \`\\~~\\      
                  "    "   "~'  ""  
  
          UnPacked By :  fly  
            2004-12-31 零点

`4dd466d0e1e0cc123afbe504da2462ca`
