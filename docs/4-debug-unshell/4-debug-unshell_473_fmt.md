
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



下面根据 AL 值分别进行处理，AL=4 则不加密函数

```
代码:
00957692     331D 64E79500       xor ebx,dword ptr ds:[95E764]
00957698     031C24             add ebx,dword ptr ss:[esp]; UnPackMe.00400000
0095769B     EB 01             jmp short 0095769E

0095769E     8B07              mov eax,dword ptr ds:[edi]
009576A0     8A00              mov al,byte ptr ds:[eax]
009576A2     FF07              inc dword ptr ds:[edi]
009576A4     33D2              xor edx,edx
009576A6     8AD0              mov dl,al
009576A8     8B4424 2C         mov eax,dword ptr ss:[esp+2C]
009576AC     E8 E3F3FFFF       call 00956A94
009576B1     894424 30         mov dword ptr ss:[esp+30],eax
009576B5     8B07              mov eax,dword ptr ds:[edi]
009576B7     8A00              mov al,byte ptr ds:[eax]
009576B9     FF07              inc dword ptr ds:[edi]
009576BB     84C0              test al,al//根据 AL 值分别进行处理
009576BD     75 44             jnz short 00957703
009576BF     EB 01             jmp short 009576C2
```

```
代码:
00957703     3C 02             cmp al,2//AL=2  ?
00957705     0F85 43010000     jnz 0095784E
0095770B     EB 01             jmp short 0095770E
0095770E     33ED              xor ebp,ebp
00957710     8B07              mov eax,dword ptr ds:[edi]
00957712     8A00              mov al,byte ptr ds:[eax]
00957714     FF07              inc dword ptr ds:[edi]
00957716     2C 01             sub al,1
00957718     73 30             jnb short 0095774A
0095771A     8B07              mov eax,dword ptr ds:[edi]
0095771C     8A00              mov al,byte ptr ds:[eax]
0095771E     FF07              inc dword ptr ds:[edi]
00957720     8B17              mov edx,dword ptr ds:[edi]
00957722     8B12              mov edx,dword ptr ds:[edx]
00957724     8307 04           add dword ptr ds:[edi],4
00957727     807C24 1E 01      cmp byte ptr ss:[esp+1E],1
0095772C     75 0F             jnz short 0095773D
0095772E     6A 01             push 1
00957730     8D4C24 44         lea ecx,dword ptr ss:[esp+44]
00957734     E8 EFFCFFFF       call 00957428
00957739     8BE8              mov ebp,eax
0095773B     EB 0D             jmp short 0095774A
0095773D     6A 00             push 0
0095773F     8D4C24 44         lea ecx,dword ptr ss:[esp+44]
00957743     E8 E0FCFFFF       call 00957428
00957748     8BE8              mov ebp,eax
0095774A     8B07              mov eax,dword ptr ds:[edi]
0095774C     8B00              mov eax,dword ptr ds:[eax]
0095774E     894424 10         mov dword ptr ss:[esp+10],eax
00957752     8307 04           add dword ptr ds:[edi],4
00957755     8B4424 24         mov eax,dword ptr ss:[esp+24]
00957759     E8 E6ADFEFF       call 00942544
0095775E     8BF0              mov esi,eax
00957760     8B5424 10         mov edx,dword ptr ss:[esp+10]
00957764     8B4424 30         mov eax,dword ptr ss:[esp+30]
00957768     E8 AFF0FFFF       call 0095681C//Alexey's GetProcAddress
0095776D     894424 38         mov dword ptr ss:[esp+38],eax
00957771     EB 01             jmp short 00957774
00957774     85ED              test ebp,ebp
00957776     0F85 9B000000     jnz 00957817
0095777C     6A 01             push 1
0095777E     8D5424 38         lea edx,dword ptr ss:[esp+38]
00957782     8D4424 3C         lea eax,dword ptr ss:[esp+3C]
00957786     33C9              xor ecx,ecx
00957788     E8 3BFAFFFF       call 009571C8//加密 CALL！进去修改 ★
0095778D     2B05 B8C49500     sub eax,dword ptr ds:[95C4B8]
00957793     8906              mov dword ptr ds:[esi],eax
00957795     807C24 1E 01      cmp byte ptr ss:[esp+1E],1
0095779A     0F85 A7000000     jnz 00957847
009577A0     8B4424 24         mov eax,dword ptr ss:[esp+24]
009577A4     83C0 32           add eax,32
009577A7     E8 98ADFEFF       call 00942544
009577AC     8BE8              mov ebp,eax
009577AE     55                push ebp
009577AF     A1 5CE79500       mov eax,dword ptr ds:[95E75C]
009577B4     50                push eax
009577B5     E8 76C8FFFF       call 00954030
009577BA     A1 5CE79500       mov eax,dword ptr ds:[95E75C]
009577BF     50                push eax
009577C0     E8 7FD4FFFF       call 00954C44
009577C5     A1 5CE79500       mov eax,dword ptr ds:[95E75C]
009577CA     E8 81C8FFFF       call 00954050
009577CF     03C5              add eax,ebp
009577D1     894424 28         mov dword ptr ss:[esp+28],eax
009577D5     B8 02000000       mov eax,2
009577DA     E8 E5AFFEFF       call 009427C4
009577DF     48                dec eax
009577E0     75 17             jnz short 009577F9
009577E2     8B16              mov edx,dword ptr ds:[esi]
009577E4     2B5424 28         sub edx,dword ptr ss:[esp+28]
009577E8     83EA 05           sub edx,5
009577EB     8B4424 28         mov eax,dword ptr ss:[esp+28]
009577EF     E8 2CEBFFFF       call 00956320
009577F4     EB 1D             jmp short 00957813
009577F6     EB 01             jmp short 009577F9
009577F8     698B442428C60068800 imul ecx,dword ptr ds:[ebx+C6282444],68B6800
00957802     8B5424 28         mov edx,dword ptr ss:[esp+28]
00957806     42                inc edx
00957807     8902              mov dword ptr ds:[edx],eax
00957809     8B4424 28         mov eax,dword ptr ss:[esp+28]
0095780D     83C0 05           add eax,5
00957810     C600 C3           mov byte ptr ds:[eax],0C3
00957813     892E              mov dword ptr ds:[esi],ebp
00957815     EB 30             jmp short 00957847
00957817     892E              mov dword ptr ds:[esi],ebp
00957819     8BC5              mov eax,ebp
0095781B     034424 40         add eax,dword ptr ss:[esp+40]
0095781F     C600 68           mov byte ptr ds:[eax],68
00957822     6A 01             push 1
00957824     8D5424 38         lea edx,dword ptr ss:[esp+38]
00957828     8D4424 3C         lea eax,dword ptr ss:[esp+3C]
0095782C     33C9              xor ecx,ecx
0095782E     E8 95F9FFFF       call 009571C8//同样的加密 CALL！ ★
00957833     8BD5              mov edx,ebp
00957835     035424 40         add edx,dword ptr ss:[esp+40]
00957839     42                inc edx
0095783A     8902              mov dword ptr ds:[edx],eax
0095783C     036C24 40         add ebp,dword ptr ss:[esp+40]
00957840     83C5 05           add ebp,5
00957843     C645 00 C3        mov byte ptr ss:[ebp],0C3
00957847     8933              mov dword ptr ds:[ebx],esi//加密地址写入程序！NOP 掉 ★
00957849     E9 92FDFFFF       jmp 009575E0//循环
```



```
代码:
0095784E     3C 01                 cmp al,1//AL=1  ?
00957850     0F85 9C000000        jnz 009578F2
00957856     EB 01                jmp short 00957859
00957858     9A 895C2418 8B07     call far 078B:18245C89
0095785F     8B28                 mov ebp,dword ptr ds:\[eax\]
00957861     8307 04              add dword ptr ds:\[edi\],4
00957864     8BC5                 mov eax,ebp
00957866     FF5424 04            call dword ptr ss:\[esp+4\]
0095786A     8BE8                 mov ebp,eax
0095786C     8B07                 mov eax,dword ptr ds:\[edi\]
0095786E     8A00                 mov al,byte ptr ds:\[eax\]
00957870     884424 1D            mov byte ptr ss:\[esp+1D\],al
00957874     FF07                 inc dword ptr ds:\[edi\]
00957876     807C24 3C 00         cmp byte ptr ss:\[esp+3C\],0
0095787B     74 11                je short 0095788E
0095787D     A1 64E79500          mov eax,dword ptr ds:\[95E764\]
00957882     A3 68E79500          mov dword ptr ds:\[95E768\],eax
00957887     C64424 3C 00         mov byte ptr ss:\[esp+3C\],0
0095788C     EB 0B                jmp short 00957899
0095788E     8B4424 34            mov eax,dword ptr ss:\[esp+34\]
00957892     8B00                 mov eax,dword ptr ds:\[eax\]
00957894     A3 68E79500          mov dword ptr ds:\[95E768\],eax
00957899     8B0D 68E79500        mov ecx,dword ptr ds:\[95E768\]
0095789F     8A5424 1D            mov dl,byte ptr ss:\[esp+1D\]
009578A3     8B07                 mov eax,dword ptr ds:\[edi\]
009578A5     E8 1AD5FFFF          call 00954DC4
009578AA     8B4424 24            mov eax,dword ptr ss:\[esp+24\]
009578AE     E8 91ACFEFF          call 00942544
009578B3     8BF0                 mov esi,eax
009578B5     8B07                 mov eax,dword ptr ds:\[edi\]
009578B7     50                   push eax
009578B8     8B4424 34            mov eax,dword ptr ss:\[esp+34\]
009578BC     50                   push eax
009578BD     A1 1CC69500          mov eax,dword ptr ds:\[95C61C\]
009578C2     8B00                 mov eax,dword ptr ds:\[eax\]
009578C4     FFD0                 call eax
009578C6     894424 38            mov dword ptr ss:\[esp+38\],eax
009578CA     6A 00                push 0
009578CC     8D5424 38            lea edx,dword ptr ss:\[esp+38\]
009578D0     8D4424 3C            lea eax,dword ptr ss:\[esp+3C\]
009578D4     8BCD                 mov ecx,ebp
009578D6     E8 EDF8FFFF          call 009571C8//同样的加密 CALL！ ★
009578DB     0305 B8C49500        add eax,dword ptr ds:\[95C4B8\]
009578E1     8906                 mov dword ptr ds:\[esi\],eax
009578E3     8933                 mov dword ptr ds:\[ebx\],esi//加密地址写入程序！NOP 掉 ★
009578E5     33C0                 xor eax,eax
009578E7     8A4424 1D            mov al,byte ptr ss:\[esp+1D\]
009578EB     0107                 add dword ptr ds:\[edi\],eax
009578ED     E9 EEFCFFFF          jmp 009575E0//循环
```

```
代码:
009578F2     3C 04                cmp al,4//AL=4  ?  则不加密函数，下面单独处理
009578F4     75 5C                jnz short 00957952
009578F6     EB 01                jmp short 009578F9
009578F8     E9 8B078B28          jmp 29208088
009578FD     8307 04              add dword ptr ds:\[edi\],4
00957900     8B07                 mov eax,dword ptr ds:\[edi\]
00957902     8A00                 mov al,byte ptr ds:\[eax\]
00957904     884424 1D            mov byte ptr ss:\[esp+1D\],al
00957908     FF07                 inc dword ptr ds:\[edi\]
0095790A     8BCD                 mov ecx,ebp
0095790C     8A5424 1D            mov dl,byte ptr ss:\[esp+1D\]
00957910     8B07                 mov eax,dword ptr ds:\[edi\]
00957912     E8 ADD4FFFF          call 00954DC4
00957917     8B4424 24            mov eax,dword ptr ss:\[esp+24\]
0095791B     E8 24ACFEFF          call 00942544
00957920     8BF0                 mov esi,eax
00957922     8B07                 mov eax,dword ptr ds:\[edi\]
00957924     50                   push eax
00957925     8B4424 34            mov eax,dword ptr ss:\[esp+34\]
00957929     50                   push eax
0095792A     A1 1CC69500          mov eax,dword ptr ds:\[95C61C\]
0095792F     8B00                 mov eax,dword ptr ds:\[eax\]
00957931     FFD0                 call eax
00957933     894424 38            mov dword ptr ss:\[esp+38\],eax
00957937     8B4424 38            mov eax,dword ptr ss:\[esp+38\]
0095793B     2B05 B8C49500        sub eax,dword ptr ds:\[95C4B8\]
00957941     8906                 mov dword ptr ds:\[esi\],eax
00957943     8933                 mov dword ptr ds:\[ebx\],esi//修改：jmp 00964030 ★
00957945     33C0                 xor eax,eax
00957947     8A4424 1D            mov al,byte ptr ss:\[esp+1D\]
0095794B     0107                 add dword ptr ds:\[edi\],eax
0095794D     E9 8EFCFFFF          jmp 009575E0//循环
```

```
代码:
00957952     3C 03                cmp al,3//AL=3  ?  这个 UnPackMe 没有 AL=3 的情况
00957954     75 7D                jnz short 009579D3//AL=不等于上面几个值，这里跳就出错了
00957956     EB 01                jmp short 00957959
00957959     8B07                 mov eax,dword ptr ds:\[edi\]
0095795B     8A00                 mov al,byte ptr ds:\[eax\]
0095795D     FF07                 inc dword ptr ds:\[edi\]
0095795F     FEC8                 dec al
00957961     74 09                je short 0095796C
00957963     FEC8                 dec al
00957965     74 30                je short 00957997
00957967     E9 74FCFFFF          jmp 009575E0
0095796C     8B07                 mov eax,dword ptr ds:\[edi\]
0095796E     8A00                 mov al,byte ptr ds:\[eax\]
00957970     884424 1C            mov byte ptr ss:\[esp+1C\],al
00957974     FF07                 inc dword ptr ds:\[edi\]
00957976     8B4424 24            mov eax,dword ptr ss:\[esp+24\]
0095797A     E8 C5ABFEFF          call 00942544
0095797F     8BF0                 mov esi,eax
00957981     8933                 mov dword ptr ds:\[ebx\],esi
00957983     8A4C24 1C            mov cl,byte ptr ss:\[esp+1C\]
00957987     8B5424 30            mov edx,dword ptr ss:\[esp+30\]
0095798B     8BC6                 mov eax,esi
0095798D     E8 B2F4FFFF          call 00956E44
00957992     E9 49FCFFFF          jmp 009575E0
00957997     8B07                 mov eax,dword ptr ds:\[edi\]
00957999     8B30                 mov esi,dword ptr ds:\[eax\]
0095799B     8307 04              add dword ptr ds:\[edi\],4
0095799E     8BC6                 mov eax,esi
009579A0     E8 9FABFEFF          call 00942544
009579A5     894424 20            mov dword ptr ss:\[esp+20\],eax
009579A9     8BCE                 mov ecx,esi
009579AB     8B17                 mov edx,dword ptr ds:\[edi\]
009579AD     8B4424 20            mov eax,dword ptr ss:\[esp+20\]
009579B1     E8 76DCFEFF          call 0094562C
009579B6     0137                 add dword ptr ds:\[edi\],esi
009579B8     8B4424 24            mov eax,dword ptr ss:\[esp+24\]
009579BC     E8 83ABFEFF          call 00942544
009579C1     8BF0                 mov esi,eax
009579C3     8B4424 20            mov eax,dword ptr ss:\[esp+20\]
009579C7     8906                 mov dword ptr ds:\[esi\],eax
009579C9     8933                 mov dword ptr ds:\[ebx\],esi
009579CB     E9 10FCFFFF          jmp 009575E0//循环
```

`AL=1`，`AL=2` 这两个分支处理都使用了 `call 009571C8` 进行加密，所以进去动动手脚



# 脱壳分析与修复

## 一、替换 IAT 处理代码

在原程序中找一段可用的空间放 `IAT`，取代跳转到壳中的地址，这样就可以 `Dump` 下来了。`jingulong` 选择存放 `IAT` 的地方是从 `40C000` 开始的，我也选择这里吧。我们可以找在最下面找空白地址写入一段代码来恢复 `IAT`

```
009571C8      55                 push ebp
009571C9      8BEC               mov ebp,esp
009571CB      81C4 50FDFFFF      add esp,-2B0
009571D1      53                 push ebx
009571D2      56                 push esi
009571D3      57                 push edi
009571D4      894D F8            mov dword ptr ss:[ebp-8],ecx
009571D7      8955 FC            mov dword ptr ss:[ebp-4],edx
009571DA      8BF8               mov edi,eax
009571DC      33C0               xor eax,eax//此时[EDI]=正确的函数地址 ★
009571DE      8D95 93FDFFFF      lea edx,dword ptr ss:[ebp-26D]
009571E4      8955 EC            mov dword ptr ss:[ebp-14],edx
009571E7      33F6               xor esi,esi//这里动手脚吧
009571E9      33DB               xor ebx,ebx
009571EB      8B17               mov edx,dword ptr ds:[edi]
009571ED      8955 E4            mov dword ptr ss:[ebp-1C],edx
009571F0      33D2               xor edx,edx
009571F2      8955 F0            mov dword ptr ss:[ebp-10],edx
009571F5      833F 00            cmp dword ptr ds:[edi],0
009571F8      0F84 01020000      je 009573FF
```

把 `009571E7` `xor esi,esi` 改为：`JMP 00964000`

在 `00964000` 处汇编：

```
00964000      50                 push eax
00964001      56                 push esi
00964002      8B35 F03F9600      mov esi,dword ptr ds:[963FF0] ; UnPackMe.0040C000//初始地址
00964008      8B07               mov eax,dword ptr ds:[edi]    ; kernel32.GetCommandLineA
0096400A      8906               mov dword ptr ds:[esi],eax//正确的函数写入
0096400C      8933               mov dword ptr ds:[ebx],esi//取代原先入壳的跳转地址
0096400E      83C6 08            add esi,8//没办法
00964011      8935 F03F9600      mov dword ptr ds:[963FF0],esi//保存地址
00964017      58                 pop eax
00964018      5E                 pop esi//恢复堆栈
00964019      33F6               xor esi,esi//原先 009571E7 处代码挪到这里来运行
0096401B      33DB               xor ebx,ebx
0096401D      8B17               mov edx,dword ptr ds:[edi]
0096401F      E9 C931FFFF        jmp 009571ED//跳回去继续流程
```

别忘了在 `[963FF0]` 处写入初始地址：`0040C000`

我的 `Code` 能力几乎等于零，以上蹩脚的代码各位兄弟见笑了。

## 二、处理 AL=4

把 `00957943` `mov dword ptr ds:[ebx],esi` 修改为：`jmp 00964030`

在 `00964030` 处汇编：

```
00964030      56                 push esi
00964031      8B35 F03F9600      mov esi,dword ptr ds:[963FF0]
00964037      8906               mov dword ptr ds:[esi],eax; kernel32.ExitProcess
00964039      8933               mov dword ptr ds:[ebx],esi//取代原先入壳的跳转地址
0096403B      83C6 08            add esi,8
0096403E      8935 F03F9600      mov dword ptr ds:[963FF0],esi//保存地址
00964044      5E                 pop esi
00964045      33C0               xor eax,eax//原先 00957945 处代码挪到这里来运行
00964047      8A4424 1D          mov al,byte ptr ss:[esp+1D]
0096404B      E9 FB38FFFF        jmp 0095794B//跳回去继续流程吧
```

## 三、补上 Stolen Code、修复输入表，完成脱壳

修改完以上代码后，`F9` 运行，中断在 `009575FC` 处，`IAT` 处理完毕。

```
009575FC      837C24 2C 00       cmp dword ptr ss:[esp+2C],0//此处下断，IAT 处理完毕
00957601      74 0C              je short 0095760F
```

注意 `009575FC` 的 `[esp+2C]` 处保存的是自校验值！如果自校验失败的话在走到 `OEP` 之前会出错！所以跟踪未修改的壳程序，得知这里的校验值等于 `0098FAF4`。把 `[esp+2C]` 处的值修改为 `0098FAF4`，就可以通过 `ASProtect` 最后 1 次典型异常分析 `Stolen Code` 然后走至 `OEP` 啦。

其实在 `009575FC` 处所有的代码均已解开，可以直接 `Dump` 了。通过第一次的跟踪分析，我们在 `OEP` 处补上 `Stolen Code`：

```
004010CC      55                 push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
004010CD      8BEC               mov ebp,esp
004010CF      83EC 44            sub esp,44
004010D2      56                 push esi
004010D3      FF15 00C04000      call dword ptr ds:[40C000]; kernel32.GetCommandLineA
004010D9      8BF0               mov esi,eax
004010DB      8A00               mov al,byte ptr ds:[eax]
004010DD      3C 22              cmp al,22//上面是补好的 Stolen Code
004010DF      75 1B              jnz short 004010FC
004010E1      56                 push esi
004010E2      FF15 08C04000      call dword ptr ds:[40C008]; user32.CharNextA
004010E8      8BF0               mov esi,eax
004010EA      8A00               mov al,byte ptr ds:[eax]
004010EC      84C0               test al,al
004010EE      74 04              je short 004010F4
004010F0      3C 22              cmp al,22
004010F2      75 ED              jnz short dumped_.004010E1
004010F4      803E 22            cmp byte ptr ds:[esi],22
004010F7      75 15              jnz short 0040110E
004010F9      46                 inc esi
004010FA      EB 12              jmp short 0040110E
004010FC      3C 20              cmp al,20
004010FE      7E 0E              jle short 0040110E
00401100      56                 push esi
00401101      FF15 10C04000      call dword ptr ds:[40C010]; user32.CharNextA
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000010CC`，`RVA=0000C000`，大小＝`CB0`

点“`Get Import`”，函数排列的比较“难看”，但全部是有效的。`FixDump`，正常运行！

修改以下几个函数，脱壳后的程序就也可以在 `Win98` 上运行了：

```
0000C148  ntdll.dll   0255   RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C3B8  ntdll.dll   02B3   RtlMoveMemory       //改为：kernel32.dll RtlMoveMemory
0000C450  ntdll.dll   0255   RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C4F0  ntdll.dll   0255   RtlGetLastWin32Error//改为：kernel32.dll GetLastError
0000C970  ntdll.dll   0255   RtlGetLastWin32Error//改为：kernel32.dll GetLastError
```

另外发现脱壳后的记事本当进行“保存”和“另存为”操作时会非法操作。把以下 2 处修改一下就正常了。

```
00404FB0      FF15 80CC4000      call dword ptr ds:[40CC80];comdlg32.GetSaveFileNameA
//修改为：jmp dword ptr ds:[40CCA0]
00404FC8      FF15 A0CC4000      call dword ptr ds:[40CCA0];comdlg32.GetOpenFileNameA
//修改为：jmp dword ptr ds:[40CCA0]
```

## 结尾信息

Cracked By 巢水工作坊——fly `[OCN]` `[FCG]` `[NUKE]` `[DCM]`

2004-05-09  00:00  `4dd466d0e1e0cc123afbe504da2462ca`
