
# ASProtect 1.23 RC4 - Win Web Crawler

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-619.htm`](https://www.pediy.com/kssd/pediy07/pediy7-619.htm)

## 软件信息

*   **软件名称**：`Win Web Crawler 2.0`
*   **应用平台**：`Win9x/NT/2000/XP`
*   **软件类别**：试验版
*   **软件大小**：`591KB`

## 软件功能

A powerful web crawler utility to Extract:
*   `URL`
*   `meta tag` (`title`, `description`, `keyword`)
*   `plain text between <body> to </body> tag`
*   `page size`
*   `last modified date value`

Sources:
*   `Web Site`
*   `Web Directories`
*   `Search Results`
*   `List of URLs from file`

**功能限制**：`trial version` 只能使用 15 天，并且只能 `SAVE` 几条记录。

## 调试环境

*   `Win2000`
*   `OllyDbg` `1.10`
*   `LordPE`
*   `ImportREC` `1.6`
*   `PEid` `v0.93`

**实例下载**：`http://www.winwebcrawler.com/wwc.exe`

## 1. Dump 文件，寻找 OEP 和 Stolen Code

首先用 `PEiD` `v0.93` 识别：

```
ASProtect 1.23 RC4 Registered -> Alexey Solodovnikov
```

打开 `OllyDbg`，忽略内存以外的所有异常，载入 `WinWebCrawler.exe`，然后用 `IsDebugPresent` 插件隐藏自身。

```
00401000 >  68 01A04A00      PUSH WinWebCr.004AA001  ; 载入后停在这里
00401005    E8 01000000      CALL WinWebCr.0040100B
0040100A    C3               RETN
```

按 `F9` 运行，程序出现内存访问异常，按 `Shift+F9` 跳过。按了 17 次之后，程序运行。

`Ctrl-F2` 重来，按 16 次 `Shift+F9`，停在这里：

```
00D73D03    3100             XOR DWORD PTR DS:[EAX],EAX  ; 最后一次内存异常
00D73D05    64:8F05 0000000> POP DWORD PTR FS:[0]
00D73D0C    58               POP EAX
...
```

在这里用 `LordPE` `Dump` 整个进程。

按 `Alt+M` 窗口打开内存镜像窗口，在 `Code` 段下内存访问断点：

```
Address    Size       Owner          Section    Contains      Type   Access    Initial   Mapped as
00400000   00001000   WinWebCr 0040          PE header     Imag   R          RWE
00401000   00083000   WinWebCr 0040          code          Imag   R          RWE   // 下内存访问断点
...
```

再按 1 次 `Shift+F9` 会断在 `004054e8`：

```
004054DA    0000             ADD BYTE PTR DS:[EAX],AL
004054DC    0000             ADD BYTE PTR DS:[EAX],AL
004054DE    0000             ADD BYTE PTR DS:[EAX],AL
004054E0  - FF25 18724800    JMP DWORD PTR DS:[487218]
004054E6    8BC0             MOV EAX,EAX
004054E8  - FF25 14724800    JMP DWORD PTR DS:[487214]              ; 断在这里
```

这是代码段被执行的第一句代码，这不可能是 `OEP`。

`====>F7` 跟进，进入壳中：

```
00D71500    55               PUSH EBP
00D71501    8BEC             MOV EBP,ESP
00D71503    8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]
00D71506    85C0             TEST EAX,EAX
00D71508    75 13            JNZ SHORT 00D7151D
00D7150A    813D 7079D700 0> CMP DWORD PTR DS:[D77970],400000       ; ASCII "MZP"
00D71514    75 07            JNZ SHORT 00D7151D
00D71516    A1 7079D700      MOV EAX,DWORD PTR DS:[D77970]
00D7151B    EB 06            JMP SHORT 00D71523
00D7151D    50               PUSH EAX
00D7151E    E8 893FFFFF      CALL 00D654AC                           ; JMP to kernel32.GetModuleHandleA
00D71523    5D               POP EBP
00D71524    C2 0400          RETN 4
```

这段代码实际上是等于 `kernel32.GetModuleHandleA`。

`F8` 单步走几下，最后返回 `004055F0`：

```
004055E8    50               PUSH EAX                                 ;   ??
004055E9    6A 00            PUSH 0                                   ;  ??
004055EB    E8 F8FEFFFF      CALL WinWebCr.004054E8                   ;  ??
004055F0    BA A8404800      MOV EDX,WinWebCr.004840A8                ;
004055F5    52               PUSH EDX
004055F6    8905 BC644800    MOV DWORD PTR DS:[4864BC],EAX            ;  EAX=00400000
004055FC    8942 04          MOV DWORD PTR DS:[EDX+4],EAX
004055FF    E8 98FFFFFF      CALL WinWebCr.0040559C
00405604    5A               POP EDX
00405605    58               POP EAX                                  ;  EAX = 00483214
00405606    E8 09E0FFFF      CALL WinWebCr.00403614
0040560B    C3               RETN
```

// 返回 `004833E5`
// 返回后，往上滚几行看一下，是这个样子：

```
004833D1    3148 00          XOR DWORD PTR DS:[EAX],ECX
004833D4    0000             ADD BYTE PTR DS:[EAX],AL
004833D6    0000             ADD BYTE PTR DS:[EAX],AL
004833D8    0000             ADD BYTE PTR DS:[EAX],AL
004833DA    0000             ADD BYTE PTR DS:[EAX],AL
004833DC    0000             ADD BYTE PTR DS:[EAX],AL
004833DE    0000             ADD BYTE PTR DS:[EAX],AL
004833E0    E8 0322F8FF      CALL WinWebCr.004055E8
004833E5    8B1D E05B4800    MOV EBX,DWORD PTR DS:[485BE0]            ; WinWebCr.004866BC // 从 0040560B 返回到这
004833EB    8B03             MOV EAX,DWORD PTR DS:[EBX]
004833ED    E8 FAC7FAFF      CALL WinWebCr.0042FBEC
004833F2    8B03             MOV EAX,DWORD PTR DS:[EBX]
004833F4    BA C0344800      MOV EDX,WinWebCr.004834C0                ; ASCII "Win Web Crawler 2.0"
004833F9    E8 06C5FAFF      CALL WinWebCr.0042F904
004833FE    8B0D 9C5A4800    MOV ECX,DWORD PTR DS:[485A9C]            ; WinWebCr.00486A30
00483404    8B03             MOV EAX,DWORD PTR DS:[EBX]
00483406    8B15 30314700    MOV EDX,DWORD PTR DS:[473130]            ; WinWebCr.00473170
0048340C    E8 F3C7FAFF      CALL WinWebCr.0042FC04
```

从这儿起明显是在执行原程序的 `code`。

这时在 `CPU` 窗口点右键，`Search for -> All referenced text strings`。

`PageUp` 到最上面，会看到 `"SOFTWARE\\Borland\\Delphi\\RTL"`，很明显，程序是用 `Delphi` 写的，入口代码应该是：

```
push ebp
mov  ebp, esp
add  esp, -10
mov  eax, ####
```

让我们重新分析一下，下面是应该执行却没有执行的 `code`，实际上是壳已经执行了：

```
004833E0    E8 0322F8FF      CALL WinWebCr.004055E8                   ; 这句从没执行过
004055E8    50               PUSH EAX                                 ; 这句从没执行过
004055E9    6A 00            PUSH 0                                   ; 这句从没执行过
004055EB    E8 F8FEFFFF      CALL WinWebCr.004054E8                   ; 这句从没执行过, 等于 Call kernel32.GetModuleHandleA
004055F0    BA A8404800      MOV EDX,WinWebCr.004840A8                ; 从壳中返回到这
004055F5    52               PUSH EDX
004055F6    8905 BC644800    MOV DWORD PTR DS:[4864BC],EAX            ;  EAX=00400000
004055FC    8942 04          MOV DWORD PTR DS:[EDX+4],EAX
004055FF    E8 98FFFFFF      CALL WinWebCr.0040559C
00405604    5A               POP EDX
00405605    58               POP EAX                                  ;  EAX = 00483214, 对应第一句 PUSH EAX
00405606    E8 09E0FFFF      CALL WinWebCr.00403614
0040560B    C3               RETN
```

所以原程序开始的 `code` 应该是这样的：

```
     push ebp
     mov  ebp, esp
     add  esp, -10
     mov  eax, ####
```

```
004833E0    E8 0322F8FF      CALL WinWebCr.004055E8
004833E5    8B1D E05B4800    MOV EBX,DWORD PTR DS:[485BE0]
004833EB    8B03             MOV EAX,DWORD PTR DS:[EBX]
```

`004833E0` 前面有 13 个 `00`，根据前面的分析就可以写出 `stolen code` `11 bytes`。

```
004833D5     55              PUSH EBP
004833D6     8BEC            MOV EBP,ESP
004833D8     83C4 F0         ADD ESP,-10
004833DB     B8 14324800    MOV EAX, 00483214
```

`OEP` 为 `000833D5`。

## 2. 修复 IAT

关掉 `OllyDbg`，直接运行 `WinWebCrawler.exe`。

打开 `ImportREC`，选中进程，`OEP` 处填入 `000833D5`，点击 `IAT AutoSearch`。

```
RVA:00087140, size 00000080
```

`size` 太小，改成 `2000` 然后点 `Get Imports`。

看到有许多无效的，明显的，从 `rva 00087760` 到 `0008913C` 根本就不可能是 `valid address`。

点 `clear imports`，然后重新设置 `RVA / size`。

```
RVA:00087140, size 00000620
```

然后再点 `Get Imports`，`392` 个函数，`133` 个无效。

```
188 (decimal:392) imported function(s). (added: +188 (decimal:+392))
(85 (decimal:133) unresolved pointer(s)) (added: +85 (decimal:+133))
```

点 `Show Invalid` 全部选中，然后右键 `==>Trace Level1(Disasm)` 修复大部分。

```
188 (decimal:392) imported function(s). (added: +188 (decimal:+392))
(16 (decimal:22) unresolved pointer(s)) (added: -6F (decimal:-111))
```

还剩下 `22` 个：`14` 个无效地址，还有 `8` 个指向壳中，`3` 个可手工修复。

```
0  0008718C  ?  0000  00D71574
0  00087214  ?  0000  00D71500 // kernel32.GetModuleHandleA
0  00087284  ?  0000  00D71564
0  000872BC  ?  0000  00D71528
0  000872CC  ?  0000  00D710AC // kernel32.GetProcAddress
0  000872D0  ?  0000  00D71500 // kernel32.GetModuleHandleA
0  000872F0  ?  0000  00D7155C
0  000872FC  ?  0000  00D7158C
```

剩下的 `5` 个，让我们来看看都是什么，右键 `==>Disassemble/HexView`：

```
00D71574    6A 00            PUSH 0
```



## 1. 初步脱壳与 IAT 修复

以下汇编代码片段涉及 `CALL` 指令及数据引用：

```assembly
00D71576    E8 313FFFFF     CALL 00D654AC                // = kernel32.dll/013F/GetModuleHandleA  
00D7157B    FF35 E07CD700   PUSH DWORD PTR DS:[D77CE0]   
00D71581    58              POP EAX  
00D71582    8B05 F07CD700   MOV EAX,DWORD PTR DS:[D77CF0]   // DWORD value:00132D88  
00D71588    C3              RETN  
```

上述代码没有 `CALL` 任何 API，逻辑等价于：

```assembly
  mov eax, 00132D88  
  retn  
```

类似地，观察另外 4 个相关位置：

```assembly
00D71564    55              PUSH EBP  
00D71565    8BEC            MOV EBP,ESP  
00D71567    8B05 F07CD700   MOV EAX,DWORD PTR DS:[D77CF0]  
00D7156D    8B45 08         MOV EAX,DWORD PTR SS:[EBP+8]  
00D71570    5D              POP EBP  
00D71571    C2 0400         RETN 4  
```

这几行等价于：

```assembly
  mov eax,[esp+4]  
  retn 4  
```

```assembly
00D71528    FF35 E07CD700   PUSH DWORD PTR DS:[D77CE0]    // DWORD value:08930005  
00D7152E    58              POP EAX  
00D7152F    C3              RETN  
```

这几行等价于：

```assembly
  mov eax, 08930005  
  ret  
```

```assembly
00D7155C    A1 EC7CD700     MOV EAX,DWORD PTR DS:[D77CEC]  // DWORD value:0000047C  
00D71561    C3              RETN  
```

这几行等价于：

```assembly
  mov eax, 0000047C  
  retn  
```

```assembly
00D7158C    55              PUSH EBP  
00D7158D    8BEC            MOV EBP,ESP  
00D7158F    8B05 F07CD700   MOV EAX,DWORD PTR DS:[D77CF0]  // DWORD value:00132D88  
00D71595    5D              POP EBP  
00D71596    C2 0400         RETN 4  
```

这几行等价于：

```assembly
  mov eax, 00132D88  
  retn 4  
```

暂且忽略这些细节，将未修复的部分全部 `CUT` 掉，修复 `dumped.exe`，得到 `dumped_.exe`。

## 2. 初步去暗桩

打开 `OllyDbg`，`Load` `dumped_.exe`，按 `F9` `RUN`，触发非法操作。

查看右侧的堆栈窗口：

```text
0012FF70   004054B4  dumped_.004054B4
```

点击 CPU 窗口，按 `Ctrl+G`，`goto` `004054B4`：

```assembly
004054AF      E8 D8BDFFFF   CALL dumped_.0040128C  
004054B4   .  A3 2C604800   MOV DWORD PTR DS:[48602C],EAX  
```

上一句调用的是 `CALL` `dumped_.0040128C`。

按 `Ctrl+G`，`goto` `0040128C`：

```assembly
0040128C      FF25 8C714800 JMP DWORD PTR DS:[48718C]  
```

再查看数据窗口：

`DS:[48718C]` = `00000000`

原来如此！

回去再看一下修复 IAT 时漏掉的 5 个 `imported function`。

`JMP` `DWORD` `PTR` `DS:[48718C]` 本应跳转到壳的 `00D71574`，但脱壳后这些指令已不存在。

很明显，这是加壳程序的暗桩，将原来的：

```assembly
  mov eax, xxxxx
```

改成了一个：

```assembly
  call ....
```

因此：

```assembly
004054AF      E8 D8BDFFFF   CALL dumped_.0040128C  
```

应该改回到：

```assembly
004054AF      B8 882D1300   mov eax, 00132D88  
```

仅做此修改可能会遗漏其他位置，且如果有很多地方 `CALL` `dumped_.0040128C`，处理起来会很麻烦。因此建议在 `dumped_.exe` 的 `CODE` 段中寻找空闲空间，写入 `mov eax xxx` 指令，然后将 `JMP` `DWORD` `PTR` `DS:[48718C]` 改为 `jmp` 到新添加的代码位置。

`CODE` 段末尾有大量空闲空间，随意选择位置写入以下代码：

```assembly
00483F00   > \B8 80251300   MOV EAX,132D88  
00483F05   .  C3            RETN  
00483F06      00            DB 00  
00483F07      00            DB 00  
00483F08   >  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]  
00483F0C   .  C2 0400       RETN 4  
00483F0F      90            NOP  
00483F10   >  B8 05009308   MOV EAX,8930005  
00483F15   .  C3            RETN  
00483F16      00            DB 00  
00483F17      00            DB 00  
00483F18   >  B8 00040000   MOV EAX,407C  
00483F1D   .  C3            RETN  
00483F1E      90            NOP  
00483F1F      90            NOP  
00483F20   >  B8 80251300   MOV EAX,132D88  
00483F25   .  C2 0400       RETN 4  
00483F28      00            DB 00  
```

再将以下 5 处跳转指令：

```assembly
0040128C   $  FF25 8C714800       JMP DWORD PTR DS:[48718C]  
004057EC   $  FF25 84724800       JMP DWORD PTR DS:[487284]  
0040577C   $  FF25 BC724800       JMP DWORD PTR DS:[4872BC]  
00405714   $  FF25 F0724800       JMP DWORD PTR DS:[4872F0]  
004056FC   $  FF25 FC724800       JMP DWORD PTR DS:[4872FC]  
```

修改为：

```assembly
0040128C     /E9 6F2C0800          JMP dumped_.00483F00  
004057EC     /E9 17E70700          JMP dumped_.00483F08  
0040577C     /E9 8FE70700          JMP dumped_.00483F10  
00405714     /E9 FFE70700          JMP dumped_.00483F18  
004056FC     /E9 1FE80700          JMP dumped_.00483F20  
```

然后将修改写回 `dumped_.exe`。

按 `Ctrl-F2` 重新开始，按 `F9` `RUN`，此时出现 `MessageBox` 对话框：

> This copy of "Win Web Crawler" is corrupted.  
> (Initial file size do not match)  
> You should download a new copy from web site  
> `http://www.winwebcrawler.com/`

太好了，程序终于能运行了，说明脱壳是正确的。

此时在 CPU 窗口右键，选择 `Search for -> All referenced text strings`。

很容易定位到以下代码段：

```assembly
0047A765  |.  55              PUSH EBP  
0047A766  |.  68 06A84700    PUSH dumped_.0047A806  
0047A76B  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
0047A76E  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP  
0047A771  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]  
0047A774  |.  A1 E05B4800   MOV EAX,DWORD PTR DS:[485BE0]  
0047A779  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0047A77B  |.  E8 F458FBFF   CALL dumped_.00430074  
0047A780  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  
0047A783  |.  E8 DC7DFEFF   CALL dumped_.00462564  
0047A788  |.  3D 60CC0500   CMP EAX,5CC60                  <==== filesize  3800000?  
0047A78D  |.  7E 21         JLE SHORT dumped_.0047A7B0       <==== eax <= 380000 OK, 否则 GAMEOVER, 必须跳  
0047A78F  |.  6A 30         PUSH 30                    ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL  
0047A791  |.  68 14A84700   PUSH dumped_.0047A814          ; |Title = "File corrupted!"  
0047A796  |.  68 24A84700   PUSH dumped_.0047A824          ; |Text = "This copy of "Win Web Crawler" is corrupted.  
0047A79B  |.  6A 00         PUSH 0                     ; |hOwner = NULL  
0047A79D  |.  E8 3AB6F8FF   CALL <JMP.&user32.MessageBoxA>       ; \MessageBoxA  
0047A7A2  |.  A1 E05B4800   MOV EAX,DWORD PTR DS:[485BE0]  
0047A7A7  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]  
0047A7A9  |.  E8 9655FBFF   CALL dumped_.0042FD44  
0047A7AE  |.  EB 3B         JMP SHORT dumped_.0047A7EB  
0047A7B0  |>  833D D0594800 >CMP DWORD PTR DS:[4859D0],0     <===== 剩余天数 > 0 ?  
0047A7B7  |.  7F 32         JG SHORT dumped_.0047A7EB        <===== 大于 0, OK, 否则 GAMEOVER, 必须跳  
0047A7B9  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]  
0047A7BC  |.  8B15 DC594800 MOV EDX,DWORD PTR DS:[4859DC]  
0047A7C2  |.  E8 3D92F8FF   CALL dumped_.00403A04  
0047A7C7  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]  
0047A7CA  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]  
0047A7CD  |.  E8 2EC6F8FF   CALL dumped_1.00406E00  
0047A7D2  |.  837D FC 00    CMP DWORD PTR SS:[EBP-4],0  
0047A7D6  |.  75 13         JNZ SHORT dumped_1.0047A7EB  
0047A7D8  |.  6A 30         PUSH 30                    ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL  
0047A7DA  |.  68 C0A84700   PUSH dumped_1.0047A8C0         ; |Title = "Trial Period Expired!"  
0047A7DF  |.  68 D8A84700   PUSH dumped_1.0047A8D8         ; |Text = "License key not found!  
0047A7E4  |.  6A 00         PUSH 0                     ; |hOwner = NULL  
0047A7E6  |.  E8 F1B5F8FF   CALL <JMP.&user32.MessageBoxA>       ; \MessageBoxA  
0047A7EB  |>  33C0          XOR EAX,EAX  
0047A7ED  |.  5A            POP EDX  
0047A7EE  |.  59            POP ECX  
0047A7EF  |.  59            POP ECX  
0047A7F0  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX  
0047A7F3  |.  68 0DA84700   PUSH dumped_1.0047A80D  
0047A7F8  |>  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]  
0047A7FB  |.  BA 02000000   MOV EDX,2  
0047A800  |.  E8 6F90F8FF   CALL dumped_1.00403874  
0047A805  \\.  C3            RETN  
```

将：

```assembly
0047A78D  |.  7E 21         JLE SHORT dumped_.0047A7B0  
```

修改为：

```assembly
0047A78D  |.  EB 21         JMP SHORT dumped_.0047A7B0  
```

然后再次运行 `dumped_.exe`，一切正常，与脱壳前一致，至此脱壳完成。

## 3. 去除功能限制

`trial version` 只能使用 15 天，且只能 `SAVE` 少量记录，并在文件末尾附加如下内容：

```text
********************************************************************
In trial version, you can save only few data
To purchase full version, please visit: http://www.winwebcrawler.com/
```

将：

```assembly
0047A7B7  |.  7F 32         JG SHORT dumped_.0047A7EB  
```

修改为：

```assembly
0047A7B7  |.  EB 32         JMP SHORT dumped_.0047A7EB  
```

即可去除 15 天限制。

搜索字符串 `"In trial version, you can save only few data"`，很容易找到各个对应的 `CODE` 位置。实际上找到了十几处，但它们的结构大致相同。



## 修改汇编指令

```asm
inc dword ptr ds:[xxxx]  
cmp dword ptr ds:[xxxx], 0A  
jl short dumped_1.lable1  
mov edx, dumped_1.errorMsg     // Error Message  
...  
dumped1.label1
```

将 `jl short` 改成 `jmp short`，可去掉只能 `SAVE` 几条记录的限制。

十几个地方，没一个统一改的地方，还得一处一处的改，在此略过，改的真烦人。

全部改过之后，没了功能限制，一切运行正常。

## 5、再次去暗桩

把 `CRACKED` 的程序在 `Win98` 下试了一下，有一个 `ACCESS` 异常，不过程序可以自己处理，还能运行。

无意间又发现另一个问题：

在用 `ImportREC` 修复 `IAT` 的时候，`[D77CF0]` 中是 `00132D88`，但用 `OLLYDBG` 跟的时候，`[D77CF0]` 却是 `00132C88`，而且程序运行后也不再改写。

看来，把下面的这句：

```asm
00D71582    8B05 F07CD700   MOV EAX,DWORD PTR DS:[D77CF0]   // DWORD value:00132D88
```

直接翻译成：

```asm
mov eax 00132D88
```

是不行的。

那么 `[00D77CF0]` 中是什么？没办法，只好又用 `OLLYDBG` 跟。

载入 `WinWebCrawler.exe`，`00D77CF0` 是无效地址。

`Shift+F9`，地址有效，但无内容。

再次 `Shift+F9`，`00D77CF0` 中出现了 `00132C88`，而且直到程序运行也不再变。

那就看第 1 次内存异常与第 2 次内存异常之间，壳都干了什么：

```asm
00D744C8    3100            XOR DWORD PTR DS:[EAX],EAX  // 第 1 次内存异常  
00D744CA    EB 01           JMP SHORT 00D744CD    // F2 下断点, Shift+F9, 再 F2 取消断点, 然后 F7  
00D744CD    64:8F05 0000000>POP DWORD PTR FS:[0]                     ; 0012FFE0  
00D744D4    EB 02           JMP SHORT 00D744D8    // 下面有花指令, 全用 F7  
00D744D8    58              POP EAX                           ; 00D7447F  
00D744D9    68 30B4D600     PUSH 0D6B430  
00D744DE    68 3845D700     PUSH 0D74538  
00D744E3    68 A039D700     PUSH 0D739A0  
00D744E8    68 6836D700     PUSH 0D73668  
00D744ED    68 1430D700     PUSH 0D73014  
00D744F2    68 FC29D700     PUSH 0D729FC  
00D744F7    68 603DD700     PUSH 0D73D60  
00D744FC    C3              RETN  
  
// 上面返回这  
00D73D60    55              PUSH EBP  
00D73D61    8BEC            MOV EBP,ESP  
00D73D63    81C4 F4FEFFFF   ADD ESP,-10C  
00D73D69    53              PUSH EBX  
00D73D6A    56              PUSH ESI  
00D73D6B    57              PUSH EDI  
00D73D6C    33C0            XOR EAX,EAX  
00D73D6E    8985 F4FEFFFF   MOV DWORD PTR SS:[EBP-10C],EAX  
00D73D74    33C0            XOR EAX,EAX  
00D73D76    55              PUSH EBP  
00D73D77    68 7442D700     PUSH 0D74274  
00D73D7C    64:FF30         PUSH DWORD PTR FS:[EAX]  
00D73D7F    64:8920         MOV DWORD PTR FS:[EAX],ESP  
00D73D82    E8 A1D5FFFF     CALL 00D71328                   // F7 跟进  
00D73D87    E8 3A000000     CALL 00D73DC6  
00D73D8C    68 953DD700     PUSH 0D73D95  
00D73D91    FF0424          INC DWORD PTR SS:[ESP]  
00D73D94    C3              RETN  
00D71328    60              PUSHAD  
00D71329    68 F27CD700     PUSH 0D77CF2  
00D7132E    6A 00           PUSH 0  
00D71330    68 3F13D700     PUSH 0D7133F  
00D71335    8B05 AE54D600   MOV EAX,DWORD PTR DS:[D654AE]  
00D7133B  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetModuleHandleA  
  
// GetModuleHandleA 返回这  
00D7133F    68 4A13D700     PUSH 0D7134A  
00D71344    E9 85000000     JMP 00D713CE  
00D713CE    5A              POP EDX                           ; 00D7134A  
00D713CF    5B              POP EBX  
00D713D0    68 D713D700     PUSH 0D713D7  
00D713D5    C3              RETN  
  
// 重要, GetModuleHandleA 的返回值 ==> [00D77CE8]  
00D713D7    8943 F6         MOV DWORD PTR DS:[EBX-A],EAX          ; GetModuleHandleA => [00D77CE8]  
00D713DA    FFE2            JMP EDX  
  
// 跳到这  
00D7134A    68 EA7CD700     PUSH 0D77CEA  
00D7134F    68 5D13D700     PUSH 0D7135D  
00D71354    8B05 C654D600   MOV EAX,DWORD PTR DS:[D654C6]  
00D7135A  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetVersion  
  
// GetVersion 返回这  
00D7135D    68 6513D700     PUSH 0D71365  
00D71362    EB 6A           JMP SHORT 00D713CE  
00D713CE    5A              POP EDX                           ; 00D71365  
00D713CF    5B              POP EBX  
00D713D0    68 D713D700     PUSH 0D713D7  
00D713D5    C3              RETN  
  
// GerVersion 的返回值 ==> [00D77CE0]  
00D713D7    8943 F6         MOV DWORD PTR DS:[EBX-A],EAX      ; GerVersion() => [00D77CE0]  
00D713DA    FFE2            JMP EDX  
  
// 跳到这  
00D71365    68 567CD700     PUSH 0D77C56  
00D7136A    68 7913D700     PUSH 0D71379  
00D7136F    8B05 CE54D600   MOV EAX,DWORD PTR DS:[D654CE]  
00D71375  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetVersionExA  
  
// GetVersionExA 返回这  
00D71379    68 EE7CD700     PUSH 0D77CEE  
00D7137E    68 8D13D700     PUSH 0D7138D  
00D71383    8B05 6654D600   MOV EAX,DWORD PTR DS:[D65466]  
00D71389  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetCurrentProcess  
  
// GetCurrentProcess 返回这  
00D7138D    68 9513D700     PUSH 0D71395  
00D71392    EB 3A           JMP SHORT 00D713CE  
00D713CE    5A              POP EDX                           ; 00D71395  
00D713CF    5B              POP EBX  
00D713D0    68 D713D700     PUSH 0D713D7  
00D713D5    C3              RETN  
  
// GetCurrentProcess 的返回值 ==> [00D77CE4]  
00D713D7    8943 F6         MOV DWORD PTR DS:[EBX-A],EAX          ; GetCurrentProcess() => [00D77CE4]  
00D713DA    FFE2            JMP EDX  
00D71395    68 F67CD700     PUSH 0D77CF6  
00D7139A    68 A813D700     PUSH 0D713A8  
00D7139F    8B05 6E54D600   MOV EAX,DWORD PTR DS:[D6546E]  
00D713A5  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetCurrentProcessId  
  
// GetCurrentProcessId 返回这  
00D713A8    68 B013D700     PUSH 0D713B0  
00D713AD    EB 1F           JMP SHORT 00D713CE  
00D713CE    5A              POP EDX                           ; 00D713B0  
00D713CF    5B              POP EBX  
00D713D0    68 D713D700     PUSH 0D713D7  
00D713D5    C3              RETN  
  
// GetCurrentProcessId 的返回值 ==> [00D77CEC]  
00D713D7    8943 F6         MOV DWORD PTR DS:[EBX-A],EAX          ; GetCurrentProcessId => [00D77CEC]  
00D713DA    FFE2            JMP EDX  
00D713B0    68 FA7CD700     PUSH 0D77CFA  
00D713B5    68 C413D700     PUSH 0D713C4  
00D713BA    8B05 5E54D600   MOV EAX,DWORD PTR DS:[D6545E]  
00D713C0  - FF20            JMP DWORD PTR DS:[EAX]                ; kernel32.GetCommandLineA  
  
// GetCommandLineA 返回这  
00D713C4    68 DC13D700     PUSH 0D713DC  
00D713C9    EB 03           JMP SHORT 00D713CE  
00D713CE    5A              POP EDX                           ; 00D713DC  
00D713CF    5B              POP EBX  
00D713D0    68 D713D700     PUSH 0D713D7  
00D713D5    C3              RETN  
  
// GetCommandLineA 的返回值 ==> [00D77CF0]  
00D713D7    8943 F6         MOV DWORD PTR DS:[EBX-A],EAX          ; GetCommandLineA => [00D77CF0]  
00D713DA    FFE2            JMP EDX  
  
// 上面跳到这  
00D713DC    61              POPAD  
00D713DD    C3              RETN  
  
// 返回这, 再跟的话马上就有第 2 次内存异常  
00D73D87    E8 3A000000     CALL 00D73DC6
```

结果：

`[00D77CE0]` = `GerVersion()`
`[00D77CE4]` = `GetCurrentProcess()`
`[00D77CE8]` = `GetModuleHandleA(0)`
`[00D77CEC]` = `GetCurrentProcessId()`
`[00D77CF0]` = `GetCommandLineA()`

这样前面的 5 个没有修复的 `IAT` 就有 3 个可修复了：
`RVA 0008718C (00D71574)` // `GerVersion`
`RVA 000872BC (00D71528)` // `GerVersion`
`RVA 000872F0 (00D7155C)` // `GetCurrentProcessId`

还省两个，先看第一个：

```asm
00D71564    55              PUSH EBP  
00D71565    8BEC            MOV EBP,ESP  
00D71567    8B05 F07CD700   MOV EAX,DWORD PTR DS:[D77CF0]  
00D7156D    8B45 08         MOV EAX,DWORD PTR SS:[EBP+8]  
00D71570    5D              POP EBP  
00D71571    C2 0400         RETN 4
```

如果它是一个 `API`，用 `C` 写应该是：

```c
ULONG xxxAPI(ULONG param)
{
  return parma;
}
```

这是什么？实在是想不出来，再跟了一下是哪里在调它，刚返回到调用的地方，就看见前面有一句：

```asm
00410C73  |.  E8 D44BFFFF   CALL <JMP.&kernel32.SizeofResource>       ; \SizeofResource
```

突然想到 `kernle32.LockResource`，先看看 `LockResource` 是怎么实现的。

`Alt+E` (`Executable Modules`)，选 `kernel32.dll`，然后 `Ctrl+N` (`ViewName`)。

怎么没有 `LockResource`？刚才在 `ImportREC` 中明明看到 `LockResouce` 应该是在 `kernel32.dll` 中的....?!

先在 `ImportREC` 中手工将它修复成 `kernle32.LockResource`，用 `OllyDbg` 跟。



点数据窗，`goto 00487284`

`00487284` > `7C591733` `kernel32.SetHandleCount`

怎么是 `SetHandleCount`，先不管，点 CPU 窗口，`goto 7C591733`

`7C591733` > `8B4424 04` `MOV EAX,DWORD PTR SS:[ESP+4]`

`7C591737` `C2 0400` `RETN 4`

这不就与壳中 `00D71574` 的代码一样吗!!!

原来在 `win2000` 下，`kernel32.LockResource` 就是 `kernel32.SetHandleCount`

再想想用 `ImportREC` 修复 `IAT` 的时候，`RVA 00087284` 上下都是到 `kernel32.dll` 中去的，

而且函数还是按字母顺序排的，能符合条件的只有 `kernle32.LockResource` 了，

最后还剩下 `00D7158C` 处的 `CODE` 还没弄清楚是什么.

`00D7158C` `55` `PUSH EBP`

`00D7158D` `8BEC` `MOV EBP,ESP`

`00D7158F` `8B05 F07CD700` `MOV EAX,DWORD PTR DS:[D77CF0]`

`00D71595` `5D` `POP EBP`

`00D71596` `C2 0400` `RETN 4`

带的参数没有用，调用的地方也没有用返回值，等于是

```
VOID xxxAPI(ULONG param)

{
  return;
}
```

跟了一下前后的 `CODE`，并根据 `ImportREC` 函数字母顺序，只可能是 `kernel32.FreeResouce`

再看看 `kernel32.FreeResouce` 的实现

`7C59172E` > `33C0` `XOR EAX,EAX`

`7C591730` `C2 0400` `RETN 4`

也对的上，就是它了.

总算是全搞定了.

`4dd466d0e1e0cc123afbe504da2462ca`
