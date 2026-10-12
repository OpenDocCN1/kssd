
# ACProtect V1.41 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6922.htm`](https://www.pediy.com/kssd/pediy06/pediy6922.htm)

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`WinXP`、`flyODBG`、`PEiD`、`LordPE`

**【实例下载】**：点击此处下载

**【脱壳过程】**

`ACProtect` 是非常优秀的中国强壳。至今没有脱 `ACProtect` 加壳 `DLL` 的教程，所以我来写个简单的过程。

`DLL` 例子虽然是用现今最新的 `ACProtect V1.41` 专业版加壳，但是没有 `SDK`、`Stolen Code`，也不是 `Delphi` 程序，不必生搬硬套。

本教程提供了寻找 `ACProtect` 加壳 `DLL` 重定位表信息、不用 `ImportREC` 来恢复其输入表的方法。

放个 `PEiD` 的 `Sign`：

`\[ACProtect V1.3X-1.4X DLL -> risco\]`

`signature = 80 7C 24 08 01 0F 85`

`ep_only = true`

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug V1.4` 插件去掉 `Ollydbg` 的调试器标志。

## 一、EP 和 OEP

```
003D7000     807C24 08 01       cmp byte ptr ss:[esp+8],1

//进入 Ollydbg 后暂停在这

003D7005     0F85 FE700100      jnz EdrLib.003EE109

//DLL 退出时会跳转

003D700B     60                 pushad
003D700C     4B                 dec ebx
003D700D     03D1               add edx,ecx
003D700F     40                 inc eax
003D7010     85D8               test eax,ebx
003D7012     E8 01000000        call EdrLib.003D7018
```

`DLL` 退出时会再次中断在 `EP` 处，找 `OEP` 就非常简单了。

退出时会从 `003D7005` 处跳转，而这下面就是跳向 `OEP` 的地方！

```
003EE109     FF25 4BE13E00      jmp dword ptr ds:[3EE14B]; EdrLib.003D11C9

//飞向光明之巅！ ^O^
```

## 二、寻找重定位表信息

一个有趣的现象，很多保护壳都没有加密重定位表，如 `ASProtect`、`Armadillo`、`ACProtect`、`幻影` 等；而一些压缩壳却加密了重定位表，如 `Neolite`、`Petite` 等。

下断：`HE GlobalFree` `Shift+F9` 运行，中断后取消断点。`Alt+F9` 返回

```
003E9598     FF95 03FD4000      call dword ptr ss:[ebp+40FD03]; kernel32.GlobalFree
003E959E     5E                 pop esi

//返回这里

003E959F     83C6 08            add esi,8
003E95A2     EB B1              jmp short EdrLib.003E9555
```

`Ctrl+F` 在当前位置下搜索命令：

```
sub ebx,eax
```

找到在 `003EBB58` 处，下断，`F9` 运行，中断。

```
003EBB58     2BD8               sub ebx,eax ; LOADDLL.00400000

//这里其实就是检测与映像基址是否相符，和 ASProtect 的方法一样。不符则重定位处理！★

003EBB5A     899D 055B4100      mov dword ptr ss:[ebp+415B05],ebx
003EBB60     0BDB               or ebx,ebx
003EBB62     0F84 BB000000      je EdrLib.003EBC23

//可以在这里改标志位 Z=1，使其跳转，这样脱壳后就不需要修改基址了 ★

003EBB68     8BB5 46F84000      mov esi,dword ptr ss:[ebp+40F846]
003EBB6E     03B5 F75A4100      add esi,dword ptr ss:[ebp+415AF7]

//[ebp+415AF7]＝[003EBAF7]＝00006000  这里保存的是重定位表的 RVA  ★

003EBB74     8B8D FB5A4100      mov ecx,dword ptr ss:[ebp+415AFB]

//[ebp+415AFB]＝[003EBAFB]＝000003B0  这里保存的是重定位表的 Size ★
```

下面处理重定位表部分就不看了。在 `003EBB62` 处，转存中跟随 `ebp+415AF7` 和 `ebp+415AFB`，得到重定位表 `RVA` 和 `Size` 后修改标志位 `Z=1`，使其跳转，免得脱壳后要修改基址。

## 三、搞定输入表，不用 ImportREC 了

`ACProtect` 的输入表加密一直是比较弱的，作者以后肯定会加强这方面保护的。

`Go Go Go`！呵呵，我们继续下断：`HE GetModuleHandleA`

`Shift+F9` 运行，中断后取消断点。`Alt+F9` 返回。 注释如下

```
003EB479     8BB5 07F94000      mov esi,dword ptr ss:[ebp+40F907]
003EB47F     03F2               add esi,edx
003EB481     8B46 0C            mov eax,dword ptr ds:[esi+C]
003EB484     0BC0               or eax,eax
003EB486     0F84 25020000      je EdrLib.003EB6B1
003EB48C     8366 0C 00         and dword ptr ds:[esi+C],0

//这里清空 ImageImportDescriptor 的 Name！ NOP 掉 ①  ★

要注意的是，当我们中断后返回 003EB4AF 时，这里已经运行过一次了，清除了第一个 Name 指针。

可以根据当时的寄存器情况来恢复这个指针。在 003EB4AF 时 ESI＝003D442C，[esi+C]＝[003D4438]=00 00，而 EBX＝003D454C ASCII "KERNEL32.dll" 是第一个处理的 DLL 名，所以可以确定[003D4438]=4C 45，修改之，否则 DLLName 会有错误 ★

003EB490     03C2               add eax,edx
003EB492     8BD8               mov ebx,eax
003EB494     56                 push esi
003EB495     57                 push edi
003EB496     50                 push eax
003EB497     8BF3               mov esi,ebx
003EB499     8BFB               mov edi,ebx
003EB49B     AC                 lods byte ptr ds:[esi]
003EB49C     C0C0 03            rol al,3

//解码出 DLL 名 ★

003EB49F     AA                 stos byte ptr es:[edi]
003EB4A0     803F 00            cmp byte ptr ds:[edi],0
003EB4A3     75 F6              jnz short EdrLib.003EB49B
003EB4A5     58                 pop eax
003EB4A6     5F                 pop edi
003EB4A7     5E                 pop esi
003EB4A8     50                 push eax
003EB4A9     FF95 20854100      call dword ptr ss:[ebp+418520]; kernel32.GetModuleHandleA
003EB4AF     0BC0               or eax,eax

//返回这里

//此时 ESI＝003D442C-003D0000＝442C   输入表的 RVA  ★

003EB4B1     75 43              jnz short EdrLib.003EB4F6
003EB4F6     60                 pushad
003EB4F7     2BC0               sub eax,eax
003EB4F9     8803               mov byte ptr ds:[ebx],al

//用完之后清空 DLL 名   NOP 掉 ②！ ★

003EB4FB     43                 inc ebx
003EB4FC     3803               cmp byte ptr ds:[ebx],al
003EB4FE     75 F9              jnz short EdrLib.003EB4F9
003EB500     61                 popad
003EB501     8985 3EF84000      mov dword ptr ss:[ebp+40F83E],eax

//保存 DLL 基址

003EB507     C785 42F84000 000>mov dword ptr ss:[ebp+40F842],0
003EB511     8B95 46F84000      mov edx,dword ptr ss:[ebp+40F846]
003EB517     8B06               mov eax,dword ptr ds:[esi]
003EB519     0BC0               or eax,eax
003EB51B     75 07              jnz short EdrLib.003EB524
003EB51D     90                 nop
003EB51E     90                 nop
003EB51F     90                 nop
003EB520     90                 nop
003EB521     8B46 10            mov eax,dword ptr ds:[esi+10]
003EB524     03C2               add eax,edx
003EB526     0385 42F84000      add eax,dword ptr ss:[ebp+40F842]
003EB52C     8B18               mov ebx,dword ptr ds:[eax]
003EB52E     8B7E 10            mov edi,dword ptr ds:[esi+10]
003EB531     03FA               add edi,edx
003EB533     03BD 42F84000      add edi,dword ptr ss:[ebp+40F842]
003EB539     85DB               test ebx,ebx
003EB53B     0F84 62010000      je EdrLib.003EB6A3
003EB541     F7C3 00000080      test ebx,80000000
003EB547     75 1D              jnz short EdrLib.003EB566
003EB549     90                 nop
003EB54A     90                 nop
003EB54B     90                 nop
003EB54C     90                 nop
003EB54D     03DA               add ebx,edx
003EB54F     83C3 02            add ebx,2
003EB552     56                 push esi
003EB553     57                 push edi
003EB554     50                 push eax
003EB555     8BF3               mov esi,ebx
003EB557     8BFB               mov edi,ebx
003EB559     AC                 lods byte ptr ds:[esi]
003EB55A     C0C0 03            rol al,3

//解码出函数名 ★

003EB55D     AA                 stos byte ptr es:[edi]
003EB55E     803F 00            cmp byte ptr ds:[edi],0
003EB561     75 F6              jnz short EdrLib.003EB559
003EB563     58                 pop eax
003EB564     5F                 pop edi
003EB565     5E                 pop esi
003EB566     3B9D 46F84000      cmp ebx,dword ptr ss:[ebp+40F846]
003EB56C     7C 11              jl short EdrLib.003EB57F
003EB56E     90                 nop
003EB56F     90                 nop
003EB570     90                 nop
003EB571     90                 nop
003EB572     83BD 1A204000 00   cmp dword ptr ss:[ebp+40201A],0
003EB579     75 0A              jnz short EdrLib.003EB585
003EB57B     90                 nop
003EB57C     90                 nop
003EB57D     90                 nop
003EB57E     90                 nop
003EB57F     81E3 FFFFFFF0F     and ebx,0FFFFFFF
003EB585     53                 push ebx
003EB586     FFB5 3EF84000      push dword ptr ss:[ebp+40F83E]
003EB58C     FF95 1C854100      call dword ptr ss:[ebp+41851C]; kernel32.GetProcAddress
003EB592     3B9D 46F84000      cmp ebx,dword ptr ss:[ebp+40F846]
003EB598     7C 0F              jl short EdrLib.003EB5A9
003EB59A     90                 nop
003EB59B     90                 nop
003EB59C     90                 nop
```



四、第 2 区段内存断点法，飞向光明之巅

`Alt+M` 打开内存查看窗口：

```
003D0000   00001000   EdrLib   003D0000 (itself)       PE header  
003D1000   00003000   EdrLib   003D0000    .text       code//第二区段  
003D4000   00001000   EdrLib   003D0000    .rdata      code,data,exports  
003D5000   00001000   EdrLib   003D0000    .data       code  
003D6000   00001000   EdrLib   003D0000    .reloc      code  
003D7000   00018000   EdrLib   003D0000    .perplex    code,imports  
```

在 `003D1000` 第二区段上设置内存访问断点，`Shift+F9` 运行

确定 `ACProtect` 的试用版保护标记，`Ollydbg` 中断在 `OEP`！

```
003D11C9     55            push ebp
//OEP ^O^
003D11CA     8BEC          mov ebp,esp
003D11CC     53            push ebx
003D11CD     8B5D 08       mov ebx,dword ptr ss:[ebp+8]
003D11D0     56            push esi
003D11D1     8B75 0C       mov esi,dword ptr ss:[ebp+C]
003D11D4     57            push edi
003D11D5     8B7D 10       mov edi,dword ptr ss:[ebp+10]
003D11D8     85F6          test esi,esi
003D11DA     75 09         jnz short EdrLib.003D11E5
```

五、PE 修正

用 `LordPE` 修正 `UnPacked.dll` 的 `OEP` `RVA=000011C9`，重定位表 `RVA=00006000`、大小＝`000003B0`，输入表 `RVA=0000442C`。保存之。`ACProtect` 没有对输出表动手脚，因此不需要处理。

用 `LordPE` 删除 `.perplex` 壳区段，然后重建 `PE` 优化一下。

`OK`，脱壳完成了！

2004 年的最后一天，昨夜落了微雪，今天天晴了。

```
         ,     \_/  
       /| \_.-~/          \\\_     ,        青春都一晌 
      ( /~   /             \~-.\_ |\\
      \`\\\\  \_/               \   ~\\ )          忍把浮名  
  _-~~~-.)  )\_\_/;;,.         \\_  //'
 /'\_,\  --~   \ ~~~-  ,;;\\__\_( (.-~~~-.       换了破解轻狂 
 \~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_\`\\  
  /~~//'  /' \`~\         ) /--..\_, )\_  \`~
  "  \`~"  "      \`"       /~'\`\\    \`\\\\~~\\    
```

附：汇编分析参考

```
003EB59D     90            nop  
003EB59E     60            pushad  
003EB59F     2BC0          sub eax,eax  
003EB5A1     8803          mov byte ptr ds:[ebx],al  
//用完之后清空函数名 NOP 掉 ③！ ★  
003EB5A3     43            inc ebx  
003EB5A4     3803          cmp byte ptr ds:[ebx],al  
003EB5A6     75 F9         jnz short EdrLib.003EB5A1  
003EB5A8     61            popad  
003EB5A9     0BC0          or eax,eax  
003EB5AB     0F84 15FFFFFF je EdrLib.003EB4C6  
003EB5B1     3B85 2C854100 cmp eax,dword ptr ss:[ebp+41852C]; USER32.MessageBoxA  
//是否是 MessageBoxA ？ EMbedded Protector 专用 APT 接口  
003EB5B7     74 20         je short EdrLib.003EB5D9  
003EB5B9     90            nop  
003EB5BA     90            nop  
003EB5BB     90            nop  
003EB5BC     90            nop  
003EB5BD     3B85 C4FD4000 cmp eax,dword ptr ss:[ebp+40FDC4]; USER32.RegisterHotKey  
//是否是 RegisterHotKey ？  
003EB5C3     74 09         je short EdrLib.003EB5CE  
003EB5C5     90            nop  
003EB5C6     90            nop  
003EB5C7     90            nop  
003EB5C8     90            nop  
003EB5C9     EB 14         jmp short EdrLib.003EB5DF  
003EB5CB     90            nop  
003EB5CC     90            nop  
003EB5CD     90            nop  
003EB5CE     8D85 31FE4000 lea eax,dword ptr ss:[ebp+40FE31]  
003EB5D4     EB 09         jmp short EdrLib.003EB5DF  
003EB5D6     90            nop  
003EB5D7     90            nop  
003EB5D8     90            nop  
003EB5D9     8D85 4BFE4000 lea eax,dword ptr ss:[ebp+40FE4B]  
003EB5DF     56            push esi  
003EB5E0     FFB5 3EF84000 push dword ptr ss:[ebp+40F83E]  
003EB5E6     5E            pop esi  
003EB5E7     39B5 12204000 cmp dword ptr ss:[ebp+402012],esi; kernel32.7C800000  
//比较是否是 Kernel32.DLL 基址  
003EB5ED     74 15         je short EdrLib.003EB604  
003EB5EF     90            nop  
003EB5F0     90            nop  
003EB5F1     90            nop  
003EB5F2     90            nop  
003EB5F3     39B5 16204000 cmp dword ptr ss:[ebp+402016],esi  
//比较是否是 User32.DLL 基址  
003EB5F9     74 09         je short EdrLib.003EB604  
003EB5FB     90            nop  
003EB5FC     90            nop  
003EB5FD     90            nop  
003EB5FE     90            nop  
003EB5FF     EB 63         jmp short EdrLib.003EB664  
003EB601     90            nop  
003EB602     90            nop  
003EB603     90            nop  
003EB604     80BD 16564100 00  cmp byte ptr ss:[ebp+415616],0  
003EB60B     74 57         je short EdrLib.003EB664  
//Magic Jump！ 如果用 ImportREC 修复输入表，则可以修改这里为：jmp 003EB664 ★  
//这样就可以得到未被加密的函数了。但是我这次不用 ImportREC，所以不修改这里  ^O^  
003EB60D     90            nop  
003EB60E     90            nop  
003EB60F     90            nop  
003EB610     90            nop  
003EB611     EB 07         jmp short EdrLib.003EB61A  
//下面就是加密了  
003EB613     90            nop  
003EB614     90            nop  
003EB615     90            nop  
003EB616     0100          add dword ptr ds:[eax],eax  
003EB618     0000          add byte ptr ds:[eax],al  
003EB61A     8BB5 0BF94000 mov esi,dword ptr ss:[ebp+40F90B]  
003EB620     83C6 0D       add esi,0D  
003EB623     81EE 02184000 sub esi,401802  
003EB629     2BF5          sub esi,ebp  
003EB62B     83FE 00       cmp esi,0  
003EB62E     7F 34         jg short EdrLib.003EB664  
003EB630     90            nop  
003EB631     90            nop  
003EB632     90            nop  
003EB633     90            nop  
003EB634     8BB5 0BF94000 mov esi,dword ptr ss:[ebp+40F90B]  
003EB63A     53            push ebx  
003EB63B     50            push eax  
003EB63C     E8 8DB2FFFF   call EdrLib.003E68CE  
003EB641     8BD8          mov ebx,eax  
003EB643     58            pop eax  
003EB644     33C3          xor eax,ebx  
003EB646     C606 68       mov byte ptr ds:[esi],68  
003EB649     8946 01       mov dword ptr ds:[esi+1],eax  
003EB64C     C746 05 81342400 mov dword ptr ds:[esi+5],243481  
003EB653     895E 08       mov dword ptr ds:[esi+8],ebx  
003EB656     C646 0C C3    mov byte ptr ds:[esi+C],0C3  
003EB65A     5B            pop ebx  
003EB65B     8BC6          mov eax,esi  
003EB65D     8385 0BF94000 0D  add dword ptr ss:[ebp+40F90B],0D  
003EB664     5E            pop esi  
003EB665     60            pushad  
003EB666     8BD0          mov edx,eax  
003EB668     2BBD 46F84000 sub edi,dword ptr ss:[ebp+40F846]  
003EB66E     8BC7          mov eax,edi  
003EB670     B9 01010000   mov ecx,101  
003EB675     8DBD EBE C4000 lea edi,dword ptr ss:[ebp+40ECEB]  
003EB67B     F2:AF         repne scas dword ptr es:[edi]  
003EB67D     0BC9          or ecx,ecx  
003EB67F     74 13         je short EdrLib.003EB694  
003EB681     90            nop  
003EB682     90            nop  
003EB683     90            nop  
003EB684     90            nop  
003EB685     81E9 01010000 sub ecx,101  
003EB68B     F7D1          not ecx  
003EB68D     89948D EBE84000  mov dword ptr ss:[ebp+ecx*4+40E8EB],edx  
003EB694     61            popad  
003EB695     8907          mov dword ptr ds:[edi],eax  
//API 函数的系统地址（或者加密地址）填充到 IAT 中    NOP 掉 ④！★  
003EB697     8385 42F84000 04  add dword ptr ss:[ebp+40F842],4  
003EB69E     E9 6EFEFFFF   jmp EdrLib.003EB511  
003EB6A3     83C6 14       add esi,14  
003EB6A6     8B95 46F84000 mov edx,dword ptr ss:[ebp+40F846]  
003EB6AC     E9 D0FDFFFF   jmp EdrLib.003EB481  
//循环处理  
003EB6B1     8DBD EBE C4000 lea edi,dword ptr ss:[ebp+40ECEB]  
//修改上面 4 处后直接 F4 到这里    输入表处理完毕了  
运行 LordPE 完全 Dump 出这个 dll，保存为 UnPacked.dll  
```



`"     "` `~'  ` `  `

`UnPacked By :  fly`

`2004-12-31 12:00`

`4dd466d0e1e0cc123afbe504da2462ca`
