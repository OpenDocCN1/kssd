
# Armadillo V3.6 双进程

> 原文：[`www.pediy.com/kssd/pediy06/pediy6496.htm`](https://www.pediy.com/kssd/pediy06/pediy6496.htm)

下载页面：`http://www.superrsoft.com/cn/srcom.htm`   

软件大小：`5794 KB`   

软件语言：`中文`   

软件类别：`国产软件 / 共享版 / 网络辅助`   

应用平台：`Win9x/NT/2000/XP`   

【说明】：此破文结合了 `Fly`、`mysqladm`、`pyzpyz`、`jwh51`、`骨灰 C (MM)` 大虾们所写的破文，在此谢谢他们了，这是我第一次学 `OLLYDBG` 和脱 `Armadillo` 的壳，请各位大侠多多指教！   

【作者声明】：只作为学习技术用，没有其他任何商业目的，失误之处敬请诸位大侠赐教！   

【调试环境】：`Win2000`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC`   

—————————————————————————————————   

## 一、使程序把自己当成子进程运行

### 脱壳过程

设置 `Ollydbg` 忽略所有的异常选项。用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。   

```
00567000  >  $ 60             PUSHAD  //进入 OD 后停在这！
00567001   .  E8 00000000     CALL iepro.00567006
00567006   $  5D             POP EBP
00567007   .  50             PUSH EAX
00567008   .  51             PUSH ECX
00567009   .  EB 0F          JMP SHORT iepro.0056701A
```

下断：`BP OpenMutexA`   

```
77E89CE9 >  55              PUSH EBP //断在这，看看堆栈
77E89CEA   8BEC             MOV EBP,ESP
77E89CEC   51              PUSH ECX
77E89CED   51              PUSH ECX
77E89CEE   837D 10 00       CMP DWORD PTR SS:[EBP+10],0
77E89CF2   56              PUSH ESI
77E89CF3   74 50            JE SHORT KERNEL32.77E89D45
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   

`BP OpenMutexA` 时的堆栈：   

```
0012F574   0053E141  /CALL 到 OpenMutexA
0012F578   001F0001  |Access = 1F0001
0012F57C   00000000  |Inheritable = FALSE
0012F580   0012FBB4  MutexName = "20C::DA3A38CB6B"
0012F584   0012FF04
0012F588   00000000
0012F58C   0055C5D9  iepro.0055C5D9
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   

`Ctrl+G`：`401000`  键入以下代码   

```
00401000     60                pushad
00401001     9C                pushfd
00401002     68 B4FB1200       push 12FBB4 ★ 堆栈里看到的值
00401007     33C0              xor eax,eax
00401009     50                push eax
0040100A     50                push eax
0040100B     E8 B4B2A577       call kernel32.CreateMutexA
00401010     9D                popfd
00401011     61                popad
00401012     E9 33F7A577       jmp kernel32.OpenMutexA
```

在 `401000` 处新建起源，`F9` 运行，再次中断在 `OpenMutexA` 处。   

—————————————————————————————————   

## 二、Magic Jump，避开 IAT 加密

取消以前断点，下断：`BP GetModuleHandleA`（为什么不下 `BP GetModuleHandleA+5` 呢？因为我试了很多次，用这个断点断不下来，所以改用 `BP GetModuleHandleA`）   

```
77E80978 >  55              PUSH EBP //断在这里
77E80979   8BEC             MOV EBP,ESP
77E8097B   837D 08 00       CMP DWORD PTR SS:[EBP+8],0
77E8097F   75 0E            JNZ SHORT KERNEL32.77E8098F //重新设置断点在这里
77E80981   64:A1 18000000   MOV EAX,DWORD PTR FS:[18]
77E80987   8B40 30          MOV EAX,DWORD PTR DS:[EAX+30]
77E8098A   8B40 08          MOV EAX,DWORD PTR DS:[EAX+8]
77E8098D   EB 14            JMP SHORT KERNEL32.77E809A3
77E8098F   FF75 08          PUSH DWORD PTR SS:[EBP+8]
77E80992   E8 2E890000      CALL KERNEL32.77E892C5
77E80997   85C0             TEST EAX,EAX
77E80999   74 08            JE SHORT KERNEL32.77E809A3
77E8099B   FF70 04          PUSH DWORD PTR DS:[EAX+4]
77E8099E   E8 E2000000      CALL KERNEL32.GetModuleHandleW
77E809A3   5D              POP EBP
77E809A4   C2 0400          RETN 4
```

取消断点，在 `77E8097F` 处重新下断点，在这里中断十几次，中间会出现无法处理异常的对话框，用 `SHIFT+F9` 跳过就行了；`Alt+F9` 返回程序。判断返回程序的时机：   

☆  ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   

注意看 `BP GetModuleHandleA` 时的堆栈变化：   

```
0012BEF8   00BCE3B7  /CALL 到 GetModuleHandleA 来自 00BCE3B1
0012BEFC   00BDF700  pModule = "kernel32.dll"
0012BF00   00BE0710  ASCII "VirtualAlloc"

0012BEF8   00BCE3D4  /CALL 到 GetModuleHandleA 来自 00BCE3CE
0012BEFC   00BDF700  pModule = "kernel32.dll"
0012BF00   00BE0704  ASCII "VirtualFree"

//再 F9 下去就会出现无法处理异常的对话框
0012BC70  |00BB912E  返回到 00BB912E 来自 KERNEL32.GetModuleHandleA
0012BC74  |0012BDAC  ASCII "kernel32.dll"
```

到这里时就用 `CTRL+F9` 返回主程序：   

```
00BB912E   8B0D 0838BE00   MOV ECX,DWORD PTR DS:[BE3808]
00BB9134   89040E           MOV DWORD PTR DS:[ESI+ECX],EAX
00BB9137   A1 0838BE00      MOV EAX,DWORD PTR DS:[BE3808]
00BB913C   393C06           CMP DWORD PTR DS:[ESI+EAX],EDI
00BB913F   75 16            JNZ SHORT 00BB9157
00BB9141   8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:[EBP-14C]
00BB9147   50              PUSH EAX
00BB9148   FF15 CCA0BD00   CALL DWORD PTR DS:[BDA0CC]   KERNEL32.LoadLibraryA
00BB914E   8B0D 0838BE00   MOV ECX,DWORD PTR DS:[BE3808]
00BB9154   89040E           MOV DWORD PTR DS:[ESI+ECX],EAX
00BB9157   A1 0838BE00      MOV EAX,DWORD PTR DS:[BE3808]
00BB915C   393C06           CMP DWORD PTR DS:[ESI+EAX],EDI
00BB915F   0F84 AD000000    JE 00BB9212    //这里就是我们常说的 Magic Jump，把它改为 JMP
00BB9165   33C9             XOR ECX,ECX
00BB9167   8B03             MOV EAX,DWORD PTR DS:[EBX]
00BB9169   3938             CMP DWORD PTR DS:[EAX],EDI
00BB916B   74 06            JE SHORT 00BB9173
```

这里所作的修改，可以不需要在 `IAT` 处理结束后再还原代码了。   

—————————————————————————————————   

## 三、查找入口点

清除上面所有的断点，然后下 `bp GetCurrentThreadId`，继续按 `F9`，中断后注意堆栈的变化：   

```
0012B830   6A281E36  /CALL 到 GetCurrentThreadId 来自 6A281E30   //注意这一行
0012B834   00000001
0012B838   6A281C1A  返回到 6A281C1A 来自 6A281DE2
0012B83C   6A281B60  返回到 6A281B60 来自 6A281B8C

0012B810   6A28353F  /CALL 到 GetCurrentThreadId 来自 6A283539
0012B814   00000000
0012B818   6A390470
0012B81C   6A3904D8
0012B820  /0012B838

0012F56C   00BD61D4  /CALL 到 GetCurrentThreadId 来自 00BD61CE //关键
0012F570   0012FF04
0012F574   00000000
0012F578   0055C5D9  iepro.0055C5D9
0012F57C   0053E8C4  iepro.0053E8C4
0012F580   0057D100  iepro.0057D100
```

当堆栈的地址出现的是主程序的地址时，这时要用 `CTRL+F9` 返回主程序：   

```
00BD61D4   A3 D47CBE00     MOV DWORD PTR DS:[BE7CD4],EAX
00BD61D9   E8 4032FEFF     CALL 00BB941E
00BD61DE   6A 00           PUSH 0
00BD61E0   E8 8F8AFEFF     CALL 00BBEC74
00BD61E5   6A 00           PUSH 0
00BD61E7   C705 0CFCBD00 7C>MOV DWORD PTR DS:[BDFC0C],0BE077C        ; ASCII "RC"
00BD61F1   E8 9D2CFEFF     CALL 00BB8E93
00BD61F6   59             POP ECX
00BD61F7   59             POP ECX
00BD61F8   E8 8F10FFFF     CALL 00BC728C
00BD61FD   8BF8           MOV EDI,EAX
00BD61FF   A1 BC7BBE00     MOV EAX,DWORD PTR DS:[BE7BBC]
00BD6204   8B48 3C        MOV ECX,DWORD PTR DS:[EAX+3C]
00BD6207   3348 14        XOR ECX,DWORD PTR DS:[EAX+14]
00BD620A   3348 10        XOR ECX,DWORD PTR DS:[EAX+10]
00BD620D   03F9           ADD EDI,ECX
00BD620F   8B0E           MOV ECX,DWORD PTR DS:[ESI]
00BD6211   85C9           TEST ECX,ECX
00BD6213   75 2F          JNZ SHORT 00BD6244
00BD6215   8B78 3C        MOV EDI,DWORD PTR DS:[EAX+3C]
00BD6218   E8 6F10FFFF     CALL 00BC728C
00BD621D   8B0D BC7BBE00   MOV ECX,DWORD PTR DS:[BE7BBC]            ; iepro.00577260
00BD6223   FF76 14        PUSH DWORD PTR DS:[ESI+14]
00BD6226   8B51 14        MOV EDX,DWORD PTR DS:[ECX+14]
00BD6229   FF76 10        PUSH DWORD PTR DS:[ESI+10]
00BD622C   3351 10        XOR EDX,DWORD PTR DS:[ECX+10]
00BD622F   FF76 0C        PUSH DWORD PTR DS:[ESI+C]
00BD6232   33D7           XOR EDX,EDI
00BD6234   03C2           ADD EAX,EDX
00BD6236   8B51 24        MOV EDX,DWORD PTR DS:[ECX+24]
00BD6239   3351 1C        XOR EDX,DWORD PTR DS:[ECX+1C]
00BD623C   33D7           XOR EDX,EDI
00BD623E   2BC2           SUB EAX,EDX
00BD6240   FFD0           CALL EAX
00BD6242   EB 25          JMP SHORT 00BD6269
00BD6244   83F9 01        CMP ECX,1
00BD6247   75 22          JNZ SHORT 00BD626B
00BD6249   FF76 04        PUSH DWORD PTR DS:[ESI+4]
00BD624C   FF76 08        PUSH DWORD PTR DS:[ESI+8]
00BD624F   6A 00          PUSH 0
```



```
00BD6251   E8 3610FFFF      CALL 00BC728C   

00BD6256   50              PUSH EAX   

00BD6257   A1 BC7BBE00     MOV EAX,DWORD PTR DS:[BE7BBC]   

00BD625C   8B48 3C         MOV ECX,DWORD PTR DS:[EAX+3C]   

00BD625F   3348 24         XOR ECX,DWORD PTR DS:[EAX+24]   

00BD6262   3348 1C         XOR ECX,DWORD PTR DS:[EAX+1C]   

00BD6265   2BF9           SUB EDI,ECX   

00BD6267   FFD7           CALL EDI    //这里 F7，飞向光明之颠。    

00BD6269   8BD8           MOV EBX,EAX   

00BD626B   5F             POP EDI   

00BD626C   8BC3           MOV EAX,EBX   

00BD626E   5E             POP ESI   

00BD626F   5B             POP EBX   

00BD6270   C3             RETN
```

在那个 `CALL EDI` 处 F7 我们就到入口了，这时可以用 `LORDPE` 把主程序 `DUMP` 出来了。

四、修复 IAT

好了，运行 `Import1.6`，OEP 填 `145C`，所有的的函数都完好无缺，`FixDump` 它。好，运行一下，没有问题，然后再拿 `Lordpe` `rebuild` 一下，减小体积。

【后语】自退出破界那么久，再次回来时已经是今非昔比了，如今是长江后浪推前浪，技术不断的发展，我们这些老 `CRACKER` 也该歇歇了。

`4dd466d0e1e0cc123afbe504da2462ca`
