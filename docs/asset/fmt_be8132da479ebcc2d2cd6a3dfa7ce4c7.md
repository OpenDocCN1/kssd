
# ASProtect V1.3B

> 原文：[`www.pediy.com/kssd/pediy06/pediy6733.htm`](https://www.pediy.com/kssd/pediy06/pediy6733.htm)

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：WinXP、Ollydbg1.10B、PEiD、LordPE、ImportREC

【实例下载】点击此处下载（或鼠标右键另存为）

【原文链接】[`bbs.pediy.com/showthread.php?s=&threadid=642`](http://bbs.pediy.com/showthread.php?s=&threadid=642)

---

## 【脱壳过程】

`ASProtect` 无疑是越来越厉害了，自 `ASProtect V1.23` 以后就不再公开发布，只对注册用户发送了，这样使得 `ASProtect` 有点神秘的味道。前几天据说 `ASProtect` 已经升级为 `V1.4` 了，可惜即使是 `ASProtect V1.3B` 网上也没有公开的。

飞速 兄用 `ASProtect V1.3B` 加壳的这个 Win98 的记事本很早就放到坛子上了，jingulong 兄曾成功脱壳过。我在学习 jingulong 兄的脱壳文件基础上写下这点脱壳笔记，其中有些是猜测的，欢迎各位指点。

为何选用记事本程序作为 UnPackMe？呵呵，当然是为了简单啦！`NotePad.EXE` 比较熟悉，没用 SDK 没有暗桩，大大降低了脱壳难度；所以这个 UnPackMe 可以算是最简单的 `ASProtect V1.3B` 壳保护程序。

---

## 一、Stolen Code & OEP

老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。忽略除了“内存访问异常”之外的所有其他异常选项。

```
00401000     68 01D04000          push Notepad.0040D001//进入 OD 后停在这
00401005     E8 01000000          call Notepad.0040100B
0040100A     C3                   retn
```

`F9` 运行，程序会中断在异常处，`Shift+F9` 通过异常，直至来到 `ASProtect` 最后 1 次典型异常处：

```
00959CB5     3100                 xor dword ptr ds:[eax],eax//ASProtect 最后 1 次典型异常
00959CB7     64:8F05 00000000     pop dword ptr fs:[0]
00959CBE     58                   pop eax
00959CBF     833D C4E79500 00     cmp dword ptr ds:[95E7C4],0
00959CC6     74 14                je short 00959CDC
00959CC8     6A 0C                push 0C
00959CCA     B9 C4E79500          mov ecx,95E7C4
00959CCF     8D45 F8              lea eax,dword ptr ss:[ebp-8]
00959CD2     BA 04000000          mov edx,4
00959CD7     E8 6075FFFF          call 0095123C
00959CDC     FF75 FC              push dword ptr ss:[ebp-4]
00959CDF     FF75 F8              push dword ptr ss:[ebp-8]
00959CE2     8B45 F4              mov eax,dword ptr ss:[ebp-C]
00959CE5     8338 00              cmp dword ptr ds:[eax],0
00959CE8     74 02                je short 00959CEC
00959CEA     FF30                 push dword ptr ds:[eax]
00959CEC     FF75 F0              push dword ptr ss:[ebp-10]
00959CEF     FF75 EC              push dword ptr ss:[ebp-14]
00959CF2     C3                   retn//此处下断，Shift+F9，断在这！返回 0097724C
```

```
0097724C     B8 533E765B          mov eax,5B763E53
00977251     E8 05000000          call 0097725B
```

此时 `ESP=0012FF3C`，看看堆栈：

```
0012FF3C     00400000   UnPackMe.00400000
0012FF40     C51510FB
0012FF44     0012FFA4
```

`0012FF44＝0012FFA4`，选中 `0012FF44` 转存处的 4 个字节，下“硬件访问－>Word”断点。`F9` 运行，中断下来：

```
0097734A     EB 44                jmp short 00977390//中断在这
```

`F7` 向下走，来到处理 Stolen Code 的地方：

```
009773ED     61                   popad//Stolen Code 开始
009773EE     F3:                  prefix rep:
009773EF     EB 02                jmp short 009773F3
```

```
00977492     892C24               mov dword ptr ss:[esp],ebp//push EBP ★
009774A3     8BEC                 mov ebp,esp ★
009774B3     83EC 44              sub esp,44 ★
0097750E     893424               mov dword ptr ss:[esp],esi
00977511     FF35 D5104000        push dword ptr ds:[4010D5]//Push ESI ★
```

```
00977517     8F05 1F759700        pop dword ptr ds:[97751F]; 004063E4
0097751D     FF15 48119700        call dword ptr ds:[971148]//Call dword ptr ds:[<&KERNEL32.GetCommandLineA>]★
跟进这个 CALL 看看：
00971154     A1 1466EB77          mov eax,dword ptr ds:[77EB6614]
00971159     F2:                  prefix repne:
0097115A     EB 01               jmp short 0097115D
0097115D     68 3DC9E577         push 77E5C93D
00971162     C3                   retn//返回 77E5C93D
77E5C938     A1 1466EB77          mov eax,dword ptr ds:[77EB6614]
77E5C93D     C3                   retn
```

`77E5C938` 处正是 `KERNEL32.GetCommandLineA` 函数地址，所以 `call dword ptr ds:[971148]` 即是 `call dword ptr ds:[<&KERNEL32.GetCommandLineA>]`。

```
00977531     8BF0                 mov esi,eax★
00977541     8A00                 mov al,byte ptr ds:[eax]★
00977551     3C 22                cmp al,22★
00977556     9C                   pushfd//Stolen Code 完毕
00977557     65:EB 01             jmp short 0097755B
```

…… 省略 …… 再向下走就是返回伪 OEP 了：

```
0097771C     8947 00              mov dword ptr ds:[edi],eax; UnPackMe.004010DF
0097771F     58                   pop eax
00977720     5F                   pop edi
00977721     9D                   popfd
00977722     36:EB 01             jmp short 00977726
00977726     FF35 7B759700        push dword ptr ds:[97757B]; UnPackMe.004010DF
0097772C     68 4C739700          push 97734C
00977731     C3                   retn
0097734F     51                   push ecx
00977350     57                   push edi
00977351     9C                   pushfd
00977352     FC                   cld
00977353     BF 90739700          mov edi,977390
00977358     B9 5E280000          mov ecx,285E
0097735D     F3:AA                rep stos byte ptr es:[edi]//轻松战场
0097735F     9D                   popfd
00977360     5F                   pop edi
00977361     59                   pop ecx
00977362     C3                   retn//飞向光明之巅！:-)   返回 004010DF
```

---

```
004010DF     75 1B                jnz short UnPackMe.004010FC//伪 OEP
004010E1     56                   push esi
004010E2     FF15 9C119700        call dword ptr ds:[97119C]
004010E8     8BF0                 mov esi,eax
004010EA     8A00                 mov al,byte ptr ds:[eax]
004010EC     84C0                 test al,al
004010EE     74 04                je short UnPackMe.004010F4
004010F0     3C 22                cmp al,22
004010F2     75 ED                jnz short UnPackMe.004010E1
004010F4     803E 22              cmp byte ptr ds:[esi],22
004010F7     75 15                jnz short UnPackMe.0040110E
004010F9     46                   inc esi
004010FA     EB 12                jmp short UnPackMe.0040110E
004010FC     3C 20                cmp al,20
004010FE     7E 0E                jle short UnPackMe.0040110E
00401100     56                   push esi
00401101     FF15 54129700        call dword ptr ds:[971254]
```

郁闷，`ASProtect V1.3B` 的输入表加密更变态了，`ImportREC` 插件无能为力啦。

用记事本原程序来参考很占便宜呀，呵呵。分析以上代码得出 Stolen Code 如下：

```
push ebp
mov ebp,esp
sub esp,44
push esi
call dword ptr ds:[<&KERNEL32.GetCommandLineA>]
mov esi,eax
mov al,byte ptr ds:[eax]
cmp al,22
```

---

## 二、输入表处理

`ASProtect V1.3B` 的输入表加密让人头痛，不但将跳转地址放人壳中，而且混乱了 IAT，函数不再是像以前那样整齐排列了。看 jingulong 兄的脱壳文件，发现他采用了另外的改造输入表的方法，jingulong 不愧为脱神！

经过数次跟踪，通过对输入表地址下内存访问断点，也可以下断 `BP GetModuleHandleA+5`，确定以下 IAT 加密的地方。

```
009575E0     8B07                 mov eax,dword ptr ds:[edi]
009575E2     8B18                 mov ebx,dword ptr ds:[eax]
009575E4     8307 04              add dword ptr ds:[edi],4
009575E7     8B07                 mov eax,dword ptr ds:[edi]
009575E9     8A00                 mov al,byte ptr ds:[eax]
009575EB     884424 1E            mov byte ptr ss:[esp+1E],al
009575EF     FF07                 inc dword ptr ds:[edi]
009575F1     85DB                 test ebx,ebx
009575F3     0F85 99000000        jnz 00957692
009575F9     EB 01                jmp short 009575FC//IAT 处理完毕，在 009575FC 处下断
```
