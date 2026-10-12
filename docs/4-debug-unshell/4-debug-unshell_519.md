
# ASProtect 2.1x SKE dll 文件脱壳+重定位表修复+优化【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-596.htm`](https://www.pediy.com/kssd/pediy08/pediy8-596.htm)

【文章标题】: ASProtect 2.1x SKE dll 文件脱壳+重定位表修复+优化  

【作 者】: machenglin  

【软件名称】: `TestDog.dll`  

【大 小】: 306K  

【加壳方式】: ASProtect 2.1x SKE->Alexey Solodovnikov  

【编写语言】: Borland Delphi  

【工 具】: `WinXP`、`OllyDbg`、`PEiD`、`LordPE`、`RecImport`、`Dll_LoadEx`、`ReloX`。  

【操作平台】: `WinXP sp2`  

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

--------------------------------------------------------------------------------  

## 详细过程  

很多的大侠已经做过 N 多的 ASProtect 2.1x SKE->Alexey Solodovnikov 的 exe 软件的教程，有关 ASProtect 2.1x SKE->Alexey Solodovnikov 的 dll 教程不是很多。通过对 dll 脱壳的学习练习，参考了 fly、Volx、syscom、linex 等很多的大侠的教程，菜鸟就将 dll 脱壳的笔记发出来，希望高手能指正错误，给初学者提供一些思路。  

用 `PEid 0.94` 显示为 ASProtect 2.1x SKE -> Alexey Solodovnikov  

用 flyDBG 载入，ALT+O， 设置内存访问异常，以及下面“同时忽略以下指定的异常或者异常范围”，其余选项全部忽略。  

```
  008D2001  >  60                pushad                                    ; OD 入口     
  
  008D2002    E8 03000000       call TestDogj.008D200A  

  008D2007  - E9 EB045D45       jmp 45EA24F7  

  008D200C    55                push ebp  

  008D200D    C3                retn  

  008D200E    E8 01000000      call TestDogj.008D2014  

  ---------------------------------------------------------------  

```

### 一、找到 OEP      

`Shift+F9`,一直到第二次出现 855CCAEF。  

`Alt+M`，在该 dll 的 code 段，F2 下断。  

`Shift+F9`，来到 OEP 了。  

```
  008BFF4C    55                push ebp  

  008BFF4D    8BEC             mov ebp,esp  

  008BFF4F    83C4 B4          add esp,-4C  

  008BFF52    B8 E4FD8B00      mov eax,TestDogj.008BFDE4  

  008BFF57    E8 E064FBFF      call TestDogj.0087643C  

  008BFF5C    E8 EF38FBFF      call TestDogj.00873850  

  008BFF61    8D40 00          lea eax,dword ptr ds:[eax]  

  008BFF64    0000             add byte ptr ds:[eax],al  

  ---------------------------------------------------------------  

```

### 二、发现 IAT、call xxxxxxxx。  

`Ctrl+B`：FF 25。  

```
  008711D8  - FF25 A8318C00    jmp dword ptr ds:[8C31A8]                ; kernel32.CloseHandle  

  008711DE    8BC0             mov eax,eax  

  008711E0  - FF25 A4318C00    jmp dword ptr ds:[8C31A4]                ; kernel32.CreateFileA  

  008711E6    8BC0             mov eax,eax  

  008711E8    E8 13EE2D00      call 00B50000                            ; call xxxxxxx①  

  008711ED    7B 8B            jpo short TestDog1.0087117A  

  008711EF    C0E8 0B          shr al,0B  

  008711F2    EE               out dx,al  

  008711F3    2D 00D48BC0      sub eax,C08BD400                         ; call xxxxxxx①  

  008711F8    E8 03EE2D00      call 00B50000  

```

右键--->数据窗口跟随--->内存地址。  

在数据窗口右键--->长型--->地址。  

向上滚动来到 IAT 开始。  

```
  008C3104  1FFC7DB3  

  008C3108  7C93188A  ntdll.RtlDeleteCriticalSection                    ; IAT 开始②  

  008C310C  7C9210ED  ntdll.RtlLeaveCriticalSection  

  008C3110  7C921005  ntdll.RtlEnterCriticalSection  

  008C3114  7C809FA1  kernel32.InitializeCriticalSection  

  .................  

  008C3134  7C80A0C7  kernel32.WideCharToMultiByte  

  008C3138  7C809CAD  kernel32.MultiByteToWideChar  

  008C313C  768C6426                                        ; IAT 加密  

  008C3140  7C810311  kernel32.lstrcpynA  

  008C3144  7C80C729  kernel32.lstrcpyA  

  008C3148  397191AA                                        ; IAT 加密  

  008C314C  4524D92E                                        ; IAT 加密  

  008C3150  5083EFAF                                        ; IAT 加密  

  008C3154  7C80AC28  kernel32.GetProcAddress  

  008C3158  82EE7A98                                        ; IAT 加密  

  ..................  

  008C36F0  5D17BD2E  comctl32.ImageList_Destroy  

  008C36F4  5D17BB5B  comctl32.ImageList_Create  

  008C36F8  00000000  

  008C36FC  77DC3238  ADVAPI32.StartServiceA                ; IAT 结束③  

  008C3700  BC9A88FE  

```

向上滚动来到 IAT 结束。  

现在，我们得到这些必要的数据，开始解决 IAT 加密的问题。  

----------------------------------------------------

### 三、测试 IAT 加密  

`Ctrl+F2` 重起 OD，在 OD 入口处 Shift+F9，一直到第一次出现 0B6。 ; 未知命令  

```
  0097EA55    C601 B6        mov byte ptr ds:[ecx],0B6      ; 第一次出现 0B6。     
  
  0097EA58    6323          arpl word ptr ds:[ebx],sp  

  0097EA5A    DEAA B86B9C6B fisubr word ptr ds:[edx+6B9C6BB8]  

  0097EA60    75 11         jnz short 0097EA73  

  0097EA62    E5 8F         in eax,8F  

  0097EA64    FE            ???                              ; 未知命令  

```

右键--->搜索--->文本字符串，在第一个 85 下双击，来到这里。  

```
  0097EBF6   /EB 0A        jmp short 0097EC02  

  0097EBF8   |68 E8F49700  push 97F4E8                    ; ASCII "85"  

  0097EBFD   |E8 2A62FEFF  call 00964E2C  

  0097EC02   \A1 1C379800  mov eax,dword ptr ds:[98371C]  

  0097EC07    8B00        mov eax,dword ptr ds:[eax]  

  0097EC09    E8 0A8CFFFF call 00977818                    ; 在这里 F2 下断。  

  0097EC0E    84C0        test al,al  

  0097EC10    75 0A       jnz short 0097EC1C  

  0097EC12    68 E8F49700 push 97F4E8                     ; ASCII "85"  

  0097EC17    E8 1062FEFF call 00964E2C  

  0097EC1C    A1 EC379800 mov eax,dword ptr ds:[9837EC]  

```

在 0097EC09 这个 call 下 F2 断点，Shift+F9，中断在这里，F2 取消断点，F7 进入这个 call。  

向下拉动，在 0097795A 这个 call 下 F2 断点。F9，中断，F2 取消断点，F7 进入这个 call。  

```
  00977939   /75 0A         jnz short 00977945  

  0097793B   |68 94799700  push 977994                   ; ASCII "180"  

  00977940   |E8 E7D4FEFF  call 00964E2C  

  00977945   \834424 08 04 add dword ptr ss:[esp+8],4  

  0097794A    47           inc edi  

  0097794B    EB 1A        jmp short 00977967  

  0097794D    83C7 02     add edi,2  

  00977950    8BC7         mov eax,edi  

  00977952    50           push eax  

  00977953    55           push ebp  

  00977954    8D4424 10    lea eax,dword ptr ss:[esp+10]  

  00977958    50           push eax  

  00977959    56           push esi  

  0097795A    E8 59FCFFFF  call 009775B8                 ; 在 0097795A 下 F2 断点。  

  0097795F    0FB707      movzx eax,word ptr ds:[edi]  

```

F9，中断，F2 取消断点，F7 进入这个 call。  

```
  009775F2    3BF0        cmp esi,eax                    ; 下 F2 断点。  

  009775F4    75 5E       jnz short 00977654      

  009775F6    EB 01       jmp short 009775F9  

  009775F8    C7          ???                          ; 未知命令  

```

在数据窗口定位 IAT 区段，Ctrl+G：008C3108 (IAT 头)。  

在 009775F2 地址连续 F9，注意数据窗口第一个出现了，看积存器 ESI 为 6D，继续 F9。  

====================================================

```
  008C3100  00000000  

  008C3104  59A77327  

  008C3108  7C93188A  ntdll.RtlDeleteCriticalSection  

  008C310C  29026569  

  008C3110  2023122F  

====================================================

```

通过测试，6D、1C 是没有加密，A8 是加密，最好能测试完全部的 IAT。  

----------------------------------------------------

### 四、测试 IAT 加密  

`Ctrl+F2` 重起，重复进行到 009775F2 这个比较，F2 断点，F9，中断。  

用 OD 的插件 HideOD 申请内存，我这里分配给 00B40000。  

修改 009775F4：  

```
  009775F4    75 5E      jnz short 00977654         ; jnz short 00977654-->jmp 00B40000  

```

F8 步进，来到申请内存 00B40000，键入以下补丁代码。  

===============================

```
  je  009775F9  

  com esi,1C  

  je  00977654 

  mov esi,1C  

  jmp 00977654  

================================

```

`0F 84 F3 75 E3 FF 83 FE 10 0F 84 45 76 E3 FF BE 10 00 00 00 E9 3B 76 E3 FF 90`  

定位数据窗口在 IAT 区段，Shift+F9，现在 IAT 已经全部解密了。



但注意的一点是要检查所有的 `IAT` 是否完全解密，有时候它是和解密的参数数据是对应的，不一定用一个没加密的参数解密所有的参数，也许有些地址只响应所对应的解密参数。

撤消申请内存和 `009775F4` 的修改，准备下面再用。

```
00977188    8945 F0           mov dword ptr ss:[ebp-10],eax
0097718B    B8 00070000       mov eax,700
00977190    E8 B7B3FDFF       call 0095254C                              ; 记住这个地址，一会用。
00977195    8945 E4           mov dword ptr ss:[ebp-1C],eax
00977198    C645 EF 00        mov byte ptr ss:[ebp-11],0
0097719C    8B7D F8           mov edi,dword ptr ss:[ebp-8]
0097719F    8B75 E4           mov esi,dword ptr ss:[ebp-1C]
```

继续 `Shift+F9`，到达最后一次的 `855CCAEF`。

`Alt+M`，在该 `dll` 的 `code` 段，`F2` 下断，`Ctrl+F9`，来到 `OEP`。

`Ctrl+G`：`00977190`

修改：

```
00977190    E8 B7B3FDFF       call 0095254C                              ; call 0095254C-->jmp 00B40043
```

`Ctrl+G`：`00B40000`。

```
00B40000    0000              add byte ptr ds:[eax],al
00B40002    0000              add byte ptr ds:[eax],al
00B40004    0000              add byte ptr ds:[eax],al
00B40006    0000              add byte ptr ds:[eax],al
00B40008    0000              add byte ptr ds:[eax],al
00B4000A    BA 00108700       mov edx,871000                             ; code 段头④
00B4000F    803A E8           cmp byte ptr ds:[edx],0E8
00B40012    75 14             jnz short 00B40028
00B40014    8B42 01           mov eax,dword ptr ds:[edx+1]
00B40017    03C2              add eax,edx
00B40019    05 05000000       add eax,5
00B4001E    3D 0000B600       cmp eax,0B60000                            ; call xxxxxxxx①
00B40023    75 03             jnz short 00B40028
00B40025    EB 0C             jmp short 00B40033
00B40027    90                nop
00B40028    42                inc edx
00B40029    81FA 00008C00     cmp edx,8C0000                             ; data 段头⑤
00B4002F    ^ 72 DE           jb short 00B4000F
00B40031    - EB FE           jmp short 00B40031
00B40033    8915 0001B400     mov dword ptr ds:[B40100],edx
00B40039    60                pushad
00B4003A    FFE2              jmp edx
00B4003C    90                nop
00B4003D    90                nop
00B4003E    90                nop
00B4003F    90                nop
00B40040    90                nop
00B40041    90                nop
00B40042    90                nop
00B40043    60                pushad
00B40044    B8 08318C00       mov eax,8C3108                             ; IAT 开始②
00B40049    90                nop
00B4004A    3910              cmp dword ptr ds:[eax],edx
00B4004C    75 23             jnz short 00B40071
00B4004E    8B0D 0001B400     mov ecx,dword ptr ds:[B40100]
00B40054    C701 FF250000     mov dword ptr ds:[ecx],25FF
00B4005A    8941 02           mov dword ptr ds:[ecx+2],eax
00B4005D    61                popad
00B4005E    90                nop
00B4005F    8B15 0001B400     mov edx,dword ptr ds:[B40100]
00B40065    90                nop
00B40066    90                nop
00B40067    90                nop
00B40068    ^ EB BE           jmp short 00B40028
00B4006A    90                nop
00B4006B    90                nop
00B4006C    90                nop
00B4006D    90                nop
00B4006E    90                nop
00B4006F    90                nop
00B40070    90                nop
00B40071    83C0 04           add eax,4
00B40074    3D FC368C00       cmp eax,8C36FC                             ; IAT 结束③
00B40079    ^ 7E CF           jle short 00B4004A
00B4007B    ^ EB E0           jmp short 00B4005D
00B4007D    90                nop
00B4007E    90                nop
00B4007F    90                nop
00B40080    0000              add byte ptr ds:[eax],al
00B40082    0000              add byte ptr ds:[eax],al
```

```
00 00 00 00 00 00 00 00 00 00 BA 00 10 87 00 80 3A E8 75 14 8B 42 01 03 C2 05 05 00 00 00 3D 00 00 B6 00 75 03 EB
0C 90 42 81 FA 00 00 8C 00 72 DE EB FE 89 15 00 01 B4 00 60 FF E2 90 90 90 90 90 90 90 60 B8 08 31 8C 00 90 39 10
75 23 8B 0D 00 01 B4 00 C7 01 FF 25 00 00 89 41 02 61 90 8B 15 00 01 B4 00 90 90 90 EB BE 90 90 90 90 90 90 90 83
C0 04 3D FC 36 8C 00 7E CF EB E0 90 90 90
```

在 `00B4000A` 新建 `EIP`，`F9` 运行，OK。

在数据窗口，`Ctrl+G`：`008711E8`，检查 `call xxxxxxxx` 修复。

```
008711E8    - FF25 A0318C00     jmp dword ptr ds:[8C31A0]                  ; kernel32.GetFileType
008711EE    8BC0                mov eax,eax
008711F0    - FF25 9C318C00     jmp dword ptr ds:[8C319C]                  ; kernel32.GetFileSize
008711F6    8BC0                mov eax,eax
008711F8    - FF25 98318C00     jmp dword ptr ds:[8C3198]                  ; kernel32.GetStdHandle
008711FE    8BC0                mov eax,eax
00871200    - FF25 94318C00     jmp dword ptr ds:[8C3194]                  ; kernel32.RaiseException
00871206    8BC0                mov eax,eax
00871208    - FF25 90318C00     jmp dword ptr ds:[8C3190]                  ; kernel32.ReadFile
0087120E    8BC0                mov eax,eax
00871210    - FF25 8C318C00     jmp dword ptr ds:[8C318C]                  ; ntdll.RtlUnwind
00871216    8BC0                mov eax,eax
00871218    - FF25 88318C00     jmp dword ptr ds:[8C3188]                  ; kernel32.SetEndOfFile
0087121E    8BC0                mov eax,eax
00871220    - FF25 84318C00     jmp dword ptr ds:[8C3184]                  ; kernel32.SetFilePointer
00871226    8BC0                mov eax,eax
00871228    - FF25 80318C00     jmp dword ptr ds:[8C3180]                  ; kernel32.UnhandledExceptionFilter
0087122E    8BC0                mov eax,eax
00871230    - FF25 7C318C00     jmp dword ptr ds:[8C317C]                  ; kernel32.WriteFile
00871236    8BC0                mov eax,eax
00871238    - FF25 BC318C00     jmp dword ptr ds:[8C31BC]                  ; USER32.CharNextA
0087123E    8BC0                mov eax,eax
00871240    - FF25 78318C00     jmp dword ptr ds:[8C3178]                  ; kernel32.ExitProcess
```

看来已经全部修复了。

### 五、修复重定位表

好了，现在用 `LordPE` -> `dump`，选择下面的脱壳 `dll` 进程，完整转存为 `dump.dll`。

打开 `RecImport`，选 `loaddll` 进程，选 `dll`，填入 `OEP`：`008BFF4C-870000=4FF4C`，`RVA`：`008C3108-400000=4C3108`，获取输入表，剪切无效指针，修复抓取文件，存为 `dump\_.dll`。

重新启动 `OD`，载入 `dump\_.dll`，`F9` 运行，单独运行正常。修改文件名为原版 `TestDog.dll`，启动主程序，出现错误，看来是重定位表引起。

运行 `Dll_LoadEx`，复制一个原版的 `dll`，`load` 原版的 `dll` 和复制原版的 `dll`。

运行 `LordPE`，注意下面镜象地址不同，镜象大小一样的二个 `dll`。完整转存为 `1.dll`、`2.dll`。

运行 `ReloX`。

1. 分别载入 `1.dll`、`2.dll`；
2. 按 `elect Section`，勾选 `CODE`、`DATA`、`BBS`、`idata` 四个区段，`OK`；
3. `Compare`，稍等即完成；
4. 按 `Fix PE Module`，选择需要修复重定位表的 `dll`，修复成功。

修改镜象地址，原来的镜象地址为 `00400000`，修改为 `00870000`。

启动主程序，跑的很顺畅。

### 六、优化脱壳文件

原版 `dll` 文件字节是 `306K`，脱壳后文件为 `568K`，还是大概优化一下。

把段 `.mackt`、`.reloc` 保存到磁盘，记住 `.data` 的 `Voffset=62000`。

清除 `.rsrc` 后的所有区块。

打开 `WinHex`，载入文件-->位置-->偏移地址：`62000`，选块开始，到文件尾选块结束，编辑-->移除。

定位到 `PE` 头，将 `.rsrc` 后到 `PE` 尾块选择-->编辑-->用 `00` 填充，保存文件。

打开 `LordPE`，从磁盘载入段 `.mackt`、修改为 `Voffset=00850000`，修改 `.rsrc` 的 `Vsize` 为 `85000-5D000=28000`。

再从磁盘载入段 `.reloc`。

优化结束，文件字节从 `568K`-->`428K`。

测试结果，`OK`。

附练习文件。



---

【版权声明】：本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
