
# ExeStealth V2.72

> 原文：[`www.pediy.com/kssd/pediy06/pediy6342.htm`](https://www.pediy.com/kssd/pediy06/pediy6342.htm)

ExeStealth 常用脱壳方法 + ExeStealth V2.72 主程序脱壳

下载页面： `http://www.webtoolmaster.com/`

软件语言 英文

软件类型 国外软件 / 共享软件 / 加密工具

应用平台 `Win9x/NT/ME/2000/XP`

软件大小 `342KB`

更新时间 `2003-8-19 14:57:39`

【软件简介】：`EXE Stealth` 是一个功能非常强大的 `EXE` 可执行文件加密工具,程序支持 `CRC` 保护,`API` 重定向,删除 `EXE` 文件头信息等功能,还可以为你的 `EXE` 可执行文件增加防止 `SmartCheck`,`Softice`,`IDag` 等跟踪调试工具跟踪破解的功能!

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

【脱壳过程】：

偶只是说 ExeStealth“常用”脱壳方法，没有说通用，呵呵，其实偶用这种方法脱掉了所有偶所碰到的 ExeStealth 加壳程序。下面举了三个例子：1 个单纯 ExeStealth V2.72 加壳、1 个伪装的 ExeStealth 壳、1 个 ExeStealth V2.72 + ASPack 2.12 加壳程序。有兴趣的兄弟再自己验证吧。:-) 第 4 个例子直捣黄龙：脱了 ExeStealth V2.72 的 ExeStealth2.exe 主程序。

设置 Ollydbg 忽略所有的异常选项。用 Ollydbg 手动脱壳,老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

—————————————————————————————————

## 一、例子 试炼品：用 ExeStealth V2.72 加壳的 Win98 的 Notepad

加壳选项：`CRC protect`、`API Redirection`、`Erase Import Information`、`Anti Process Dumping`、`Anti-SmartCheck`、`Anti-SoftIce`、`Anti-Idag`

```
0040D060     EB 00             jmp short Notepad.0040D062 //进入 OD 后停在这！
0040D062     EB 2F             jmp short Notepad.0040D093
```

下断：`BP CloseHandle`

```
77E57963     64:A1 18000000    mov eax,dword ptr fs:[18] //断在这！取消断点 Ctrl+F9 执行到返回
77E57969     8B48 30           mov ecx,dword ptr ds:[eax+30]
77E5796C     8B4424 04         mov eax,dword ptr ss:[esp+4]
77E57970     83F8 F4           cmp eax,-0C
77E57973     0F84 4CB4FFFF     je kernel32.77E52DC5
77E57979     83F8 F5           cmp eax,-0B
77E5797C     0F84 38B4FFFF     je kernel32.77E52DBA
77E57982     83F8 F6           cmp eax,-0A
77E57985     0F84 0F500200     je kernel32.77E7C99A
77E5798B     8BC8              mov ecx,eax
77E5798D     81E1 03000010     and ecx,10000003
77E57993     83F9 03           cmp ecx,3
77E57996     50                push eax
77E57997     0F84 26870000     je kernel32.77E600C3
77E5799D     FF15 3C10E477     call dword ptr ds:[<&ntdll.NtClose>]
77E579A3     85C0              test eax,eax
77E579A5     0F8C 02B4FFFF     jl kernel32.77E52DAD
77E579AB     33C0              xor eax,eax
77E579AD     40                inc eax
77E579AE     C2 0400           retn 4  //返回到 0040D3B0
```

```
0040D3B0     58                pop eax
0040D3B1     8B85 B62F4000     mov eax,dword ptr ss:[ebp+402FB6]
0040D3B7     BB 01000000       mov ebx,1
0040D3BC     E8 08000000       call Notepad.0040D3C9
0040D3C1     8D85 F52B4000     lea eax,dword ptr ss:[ebp+402BF5]
0040D3C7     50                push eax
0040D3C8     C3                retn  //返回到 0040D49F
```

```
0040D49F     8B9D B62F4000     mov ebx,dword ptr ss:[ebp+402FB6] ; Notepad.00400000
0040D4A5     039D BA2F4000     add ebx,dword ptr ss:[ebp+402FBA]
                                //EBX=00400000 + 000010CC=004010CC  这就是 OEP 值  :-)
0040D4AB     C1CB 07           ror ebx,7
```

OK，在 4010CC 内存处下 内存访问断点，F9 运行，断在 OEP 处！

```
004010CC     55                push ebp //在这儿用 OllyDump 插件直接 DUMP
004010CD     8BEC              mov ebp,esp
004010CF     83EC 44           sub esp,44
004010D2     56                push esi
004010D3     FF15 E4634000     call dword ptr ds:[4063E4]
004010D9     8BF0              mov esi,eax
004010DB     8A00              mov al,byte ptr ds:[eax]
004010DD     3C 22             cmp al,22
004010DF     75 1B             jnz short 004010FC
```

————————————————————————

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000010CC`，点 `IT AutoSearch`，点“Get Import”，用“追踪层次 1”全部修复，`FixDump`，正常运行！

—————————————————————————————————

## 二、例子 试炼品：linson 兄弟的伪装壳 LiNSoN's Unpack Me 0.1

下载：`http://tongtian.net/pediybbs/download.php?id=386`

用 `PEID` 或 `FI` 侦测会报出是 `Microsoft Visual C++ 6.0`，呵呵，上了 linson 的当啦 :-) 这就是 linson 伪装代码的效果，相信这个方法会被一些新壳所采用以蒙蔽侦壳工具的！

```
00405060     55                push ebp //进入 OD 后停在这！这段代码是 linson 伪装的 VC 的入口方式
00405061     8BEC              mov ebp,esp
00405063     6A FF             push -1
00405065     68 45230100       push 12345
0040506A     68 90780600       push 67890
0040506F     64:A1 00000000    mov eax,dword ptr fs:[0]
00405075     50                push eax
00405076     64:8925 00000000  mov dword ptr fs:[0],esp
0040507D     83EC 44           sub esp,44
00405080     53                push ebx
00405081     56                push esi
00405082     57                push edi
00405083     73 15             jnb short LZSunpac.0040509A
00405085     72 13             jb short LZSunpac.0040509A
0040509A     5F                pop edi
0040509B     5E                pop esi
0040509C     5B                pop ebx
0040509D     83C4 44           add esp,44
004050A0     64:8B25 00000000  mov esp,dword ptr fs:[0]
004050A7     83C4 0C           add esp,0C
004050AA     8BE5              mov esp,ebp
004050AC     5D                pop ebp
004050AD     2BC0              sub eax,eax
004050AF     60                pushad
004050B0     E8 00000000       call LZSunpac.004050B5
004050B5     5D                pop ebp
004050B6     81ED 421E4000     sub ebp,LZSunpac.00401E42
004050BC     B9 7B090000       mov ecx,97B
004050C1     8DBD 8A1E4000     lea edi,dword ptr ss:[ebp+401E8A]
004050C7     8BF7              mov esi,edi
004050C9     AC                lods byte ptr ds:[esi]
004050CA     F9                stc
004050CB     90                nop
004050CC     C0C0 C5           rol al,0C5
004050CF     02C1              add al,cl
004050D1     EB 01             jmp short LZSunpac.004050D4
004050D4     FEC8              dec al
004050D6     34 F3             xor al,0F3
004050D8     EB 01             jmp short LZSunpac.004050DB //这里应是 ExeStealth 壳的入口
004050DB     EB 01             jmp short LZSunpac.004050DE
004050DE     2AC1              sub al,cl
004050E0     2C 99             sub al,99
004050E2     2AC1              sub al,cl
004050E4     C0C0 01           rol al,1
004050E7     2C 98             sub al,98
004050E9     04 E2             add al,0E2
004050EB     04 19             add al,19
004050ED     C0C8 16           ror al,16
004050F0     EB 01             jmp short LZSunpac.004050F3
```

————————————————————————

障眼法对于手动脱壳是没有太大影响的。直接下断：`BP CloseHandle`

```
77E57963     64:A1 18000000    mov eax,dword ptr fs:[18] //断在这！取消断点 Ctrl+F9 执行到返回
  … …  省 略  … …
77E579AE     C2 0400           retn 4 //返回到 00405383
```

```
00405383     58                pop eax
00405384     8B85 B3254000     mov eax,dword ptr ss:[ebp+4025B3]
0040538A     BB 01000000       mov ebx,1
0040538F     E8 08000000       call LZSunpac.0040539C
00405394     8D85 F2214000     lea eax,dword ptr ss:[ebp+4021F2]
0040539A     50                push eax
0040539B     C3                retn //返回到 00405465
```

```
00405465     8B9D B3254000     mov ebx,dword ptr ss:[ebp+4025B3] ; LZSunpac.00400000
0040546B     039D B7254000     add ebx,dword ptr ss:[ebp+4025B7]
                                //EBX=00400000 + 00001000=00401000  这就是 OEP 值  :-)
00405471     C1CB 07           ror ebx,7
```

OK，在 401000 内存处下 内存访问断点，F9 运行，几秒后断在 OEP 处！

```
00401000     6A 00             push 0 //在这儿用 OllyDump 插件直接 DUMP
00401002     68 00304000       push 403000
00401007     68 05304000       push 403005
```



### 0040100C

```
0040100C     6A 00                  push 0   
0040100E     E8 07000000            call 0040101A   
00401013     6A 00                  push 0   
00401015     E8 06000000            call 00401020   
0040101A     FF25 08204000          jmp dword ptr ds:[402008]   
00401020     FF25 00204000          jmp dword ptr ds:[402000]   
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00001000`，点 `IT AutoSearch`，点“Get Import”，用“追踪层次 1”全部修复，`FixDump`。

运行修复后的程序出错！用 `LordPE` 删除 `XJ` 和 `newIID` 区段，然后重建 `PE`，正常运行！`7.15->4.61K`

### 三、例子 试炼品：大嘴日语 V1.0

下载页面：[http://www.skycn.com/soft/12090.html](http://www.skycn.com/soft/12090.html)

软件大小：6628 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 外语学习

应用平台：`Win9x/NT/2000/XP`

加入时间：2003-05-13 16:32:19

下载次数：2696

推荐等级：***

开发商：[http://bmenglish.3322.net/](http://bmenglish.3322.net/)

**【软件简介】**：大嘴日语集日语的听、说、读、写、背功能于一身，解决了困扰国人的学习日语难的问题。是一款集日语口语、日语背单词综合教育软件。大嘴日语精选了常用日语、基础日语、实用日语、日语情景会话等四大类共 1300 多句常用日语对话和情景对话内容以及大学日语第一册、大学日语第二册、大学日语第三册、大学日语第四册、大学日语补充词汇、标准日语初级上册、标准日语初级下册、标准日语中级上册、标准日语中级下册、标准日语补充词汇等 8700 多个单词的全部内容并以可爱的卡通人物，运用先进的日语语音朗读技术，同时大嘴日语的日语五十音采用真人发音，以语音复读方式学习五十音，纠正发音，使发音更准确。富于娱乐性；使您在使用电脑进行工作、学习或娱乐的同时，就能不知不觉、轻轻松松地掌握各类日常日语会话及常用日语单词。该方式既摆脱了枯燥的传统学习方法，又充分利用了在电脑前的各种冗余和等待时间，有效地提高了学习效率，在不知不觉中轻轻松松、迅速提高日语的听力、口语水平及日语单词数量。

这是偶以前下载的，现在肯定不是 V1.0 版啦，不知加壳方式变了没有。

`BmJapanese.exe` 用 `PEID` 看是 `EXEStealth 2.7` 壳，其实是 `EXEStealth 2.7 + ASPack 2.12` 双层壳。

```
00718060     EB 00                  jmp short BmJapane.00718062//进入 OD 后停在这！    
00718062     60                   pushad   
00718063     EB 00                  jmp short BmJapane.00718065   
```

下断：`BP CloseHandle`

```
77E57963     64:A1 18000000       mov eax,dword ptr fs:[18] //断在这！取消断点 Ctrl+F9 执行到返回    
  …… …… 省 略 …… ……   

77E579AE     C2 0400              retn 4  //返回到 0071836C    
0071836C     58                   pop eax   
0071836D     8B85 852E4000        mov eax,dword ptr ss:[ebp+402E85]   
00718373     BB 01000000          mov ebx,1   
00718378     E8 08000000          call BmJapane.00718385   
0071837D     8D85 C42A4000        lea eax,dword ptr ss:[ebp+402AC4]   
00718383     50                   push eax   
00718384     C3                   retn  //返回到 0071845B    
0071845B     8B9D 852E4000        mov ebx,dword ptr ss:[ebp+402E85]; BmJapane.00400000   
00718461     039D 892E4000        add ebx,dword ptr ss:[ebp+402E89]   
                                  //EBX=00400000 + 00315001=00715001  这就是第 2 层壳的入口    
00718467     C1CB 07              ror ebx,7   
```

`OK`，在 `00715001` 内存处下内存访问断点，`F9` 运行，断在 `00715001` 处！

```
00715001     60                   pushad //断在第 2 层 ASPack 壳的入口处 :-)    
00715002     E8 03000000          call 0071500A   
```

偶们再来脱掉第 2 层 `ASPack` 马甲 `:-)` 下断：`BP GetModuleHandleA`

```
77E59F93     837C24 04 00         cmp dword ptr ss:[esp+4],0 //断在这！取消断点 Ctrl+F9 执行到返回    
77E59F98     0F84 23060000        je kernel32.77E5A5C1   
77E59F9E     FF7424 04           push dword ptr ss:[esp+4]   
77E59FA2     E8 55080000          call kernel32.77E5A7FC   
77E59FA7     85C0                 test eax,eax   
77E59FA9     74 08                je short kernel32.77E59FB3   
77E59FAB     FF70 04             push dword ptr ds:[eax+4]   
77E59FAE     E8 B0060000          call kernel32.GetModuleHandleW   
77E59FB3     C2 0400              retn 4  //返回到 0071529B    
00715295     FF95 4D0F0000        call dword ptr ss:[ebp+F4D]   
0071529B     85C0                 test eax,eax  //向下找 POPAD    
0071529D     75 07                jnz short 007152A6   
```

`Ctrl+F` 在当前位置下查找命令：`POPAD`

```
0071539A     B8 18D91900          mov eax,19D918   
0071539F     50                   push eax  //此处下断，F9 运行断在这！    
007153A0     0385 22040000        add eax,dword ptr ss:[ebp+422]   
                                  //EAX=0019D918 + 00400000=0059D918  这就是 OEP 值  :-)    
007153A6     59                   pop ecx   
007153A7     0BC9                 or ecx,ecx   
007153A9     8985 A8030000        mov dword ptr ss:[ebp+3A8],eax   
007153AF     61                   popad  //找到这里！    
007153B0     75 08                jnz short 007153BA   
007153BA     68 18D95900          push 59D918   
007153BF     C3                   retn   //飞向光明之巅！:-) 返回到 0059D918    
```

```
0059D918     55                   push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程    
0059D919     8BEC                 mov ebp,esp   
0059D91B     83C4 F0              add esp,-10   
0059D91E     53                   push ebx   
0059D91F     B8 68D55900          mov eax,59D568   
0059D924     E8 1394E6FF          call 00406D3C   
0059D929     8B1D A83F5A00        mov ebx,dword ptr ds:[5A3FA8]   
0059D92F     8B0B                 mov ecx,dword ptr ds:[ebx]   
0059D931     B2 01                mov dl,1   
0059D933     A1 90945400          mov eax,dword ptr ds:[549490]   
0059D938     E8 8FB8EDFF          call 004791CC   
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `0019D918`，点 `IT AutoSearch`，点“Get Import”，用“追踪层次 1”全部修复，`FixDump`，正常运行！

### 四、ExeStealth V2.72 主程序 ExeStealth2.exe 脱壳

虽然 `ExeStealth2.exe` 用 `PEID` 看是 `EXEStealth 2.72-2.73 -> WebToolMaster`，但是正如 `飞速` 兄所言：`ExeStealth2.exe` 是 `ASProtect` 加壳的，或者说是 `ASProtect` 的 `Hanspeter Imp` 修改版。

用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有异常选项。

```
00418060     EB 00                  jmp short ExeSteal.00418062//进入 OD 后停在这！    
00418062     EB 2F                  jmp short ExeSteal.00418093   
```

用 `ASProtect` 的脱壳方法。`F9` 运行，程序中断在异常处，`Shift+F9` 通过 20 次异常，在 `RET` 处下断！

```
00418766     CD 68                  int 68                 //第 1 次异常    
004187F2     0000                   add byte ptr ds:[eax],al  //第 2 次异常    
00A744C8     3100                   xor dword ptr ds:[eax],eax//第 3 次异常    
00A73DD5     3100                   xor dword ptr ds:[eax],eax//第 4 次异常    
00A73E1D     3100                   xor dword ptr ds:[eax],eax//第 5 次异常    
00A741F2     3100                   xor dword ptr ds:[eax],eax//第 6 次异常    
00A7212E     CD 01                  int 1                   //第 7 次异常    
00A72A9A     3100                   xor dword ptr ds:[eax],eax//第 8 次异常    
00A72B71     3100                   xor dword ptr ds:[eax],eax//第 9 次异常    
00A7309F     3100                   xor dword ptr ds:[eax],eax//第 10 次异常    
00A731B3     3100                   xor dword ptr ds:[eax],eax//第 11 次异常    
00A736BE     3100                   xor dword ptr ds:[eax],eax//第 12 次异常    
00A738B6     3100                   xor dword ptr ds:[eax],eax//第 13 次异常    
00A74603     3100                   xor dword ptr ds:[eax],eax//第 14 次异常    
00A74677     3100                   xor dword ptr ds:[eax],eax//第 15 次异常    
00A749CB     3100                   xor dword ptr ds:[eax],eax//第 16 次异常    
00A74C0D     3100                   xor dword ptr ds:[eax],eax//第 17 次异常    
00A74D3F     3100                   xor dword ptr ds:[eax],eax//第 18 次异常    
        弹出 30 天试用的警告，确定后返回   
00A74F48     3100                   xor dword ptr ds:[eax],eax//第 19 次异常    
00A73D03     3100                   xor dword ptr ds:[eax],eax//第 20 次异常  停！    
00A73D03     3100                   xor dword ptr ds:[eax],eax   
00A73D05     64:8F05 00000000     pop dword ptr fs:[0]   
00A73D0C     58                   pop eax   
00A73D0D     833D BC7EA700 00     cmp dword ptr ds:[A77EBC],0   
00A73D14     74 14                je short 00A73D2A   
00A73D16     6A 0C                push 0C   
00A73D18     B9 BC7EA700          mov ecx,0A77EBC   
00A73D1D     8D45 F8              lea eax,dword ptr ss:[ebp-8]   
00A73D20     BA 04000000          mov edx,4   
00A73D25     E8 E6D2FFFF          call 00A71010   
00A73D2A     FF75 FC              push dword ptr ss:[ebp-4]   
00A73D2D     FF75 F8              push dword ptr ss:[ebp-8]   
00A73D30     8B45 F4              mov eax,dword ptr ss:[ebp-C]   
00A73D33     8338 00              cmp dword ptr ds:[eax],0   
```



```
00A73D36     74 02                je short 00A73D3A   

00A73D38     FF30                 push dword ptr ds:[eax]   

00A73D3A     FF75 F0              push dword ptr ss:[ebp-10]   

00A73D3D     FF75 EC              push dword ptr ss:[ebp-14]   

00A73D40     C3                   retn //此处下断！Shift+F9 断下，返回到 00A841F0    

00A841F0     68 7891283F          push 3F289178   

00A841F5     8AF3                 mov dh,bl   

00A841F7     5A                   pop edx   

00A841F8     E8 05000000          call 00A84202   

00A84202     66:8BF9              mov di,cx   

00A84205     5B                   pop ebx   

00A84206     68 5447F463          push 63F44754   

00A8420B     66:BA C084           mov dx,84C0   

00A8420F     5A                   pop edx   

00A84210     81C3 48150000        add ebx,1548   

00A84216     56                   push esi   

00A84217     66:B9 31AB           mov cx,0AB31   

00A8421B     5F                   pop edi   

00A8421C     BE 28050000          mov esi,528   

00A84221     66:8BCA              mov cx,dx   

00A84224     FF33                 push dword ptr ds:[ebx]   

00A84226     0FB7CB               movzx ecx,bx   

00A84229     58                   pop eax   

00A8422A     BA B4FFED44          mov edx,44EDFFB4   

00A8422F     81C0 63DA2D7C        add eax,7C2DDA63   

00A84235     66:81E7 D924         and di,24D9   

00A8423A     81C0 6056A263        add eax,63A25660   

00A84240     66:81E1 AA66         and cx,66AA   

00A84245     81C0 1937B010        add eax,10B03719   

00A8424B     8BCA                 mov ecx,edx   

00A8424D     50                   push eax   

00A8424E     80CE 49              or dh,49   

00A84251     8F03                 pop dword ptr ds:[ebx]   

00A84253     81EB 8B78BA3A        sub ebx,3ABA788B   

00A84259     66:81D9 1468         sbb cx,6814   

00A8425E     81C3 8778BA3A        add ebx,3ABA7887   

00A84264     80F2 B9              xor dl,0B9   

00A84267     81EE 01000000        sub esi,1   

00A8426D     0F85 1A000000        jnz 00A8428D   

00A84273     0F82 03000000        jb 00A8427C   

00A84279     66:8BCB              mov cx,bx   

00A8427C     E9 28000000          jmp 00A842A9 //此处下断，F9 断下，跳出循环    

00A8428D     BA 74800474          mov edx,74048074   

00A84292     E9 8DFFFFFF          jmp 00A84224 //循环，发现在 00A8427C 处可跳过    

00A842A9     5B                   pop ebx   

00A842AA     58                   pop eax   

00A842AB     05 06FAF05B          add eax,5BF0FA06   

00A842B0     5C                   pop esp   

00A842B1     EB 3F                jmp short 00A842F2   

00A842F2     03C3                 add eax,ebx   

                              //EAX=00003607 + 00400000=00403607 这就是 OEP 值  :-)    

00A842F4     BB 00000000          mov ebx,0   

00A842F9     0BDB                 or ebx,ebx   

00A842FB     75 02                jnz short 00A842FF   

00A842FD     50                   push eax   

00A842FE     C3                   retn    //飞向光明之巅！:-) 返回到 00403607    

————————————————————————   

00403607     6A 00                push 0 //在这儿用 OllyDump 插件直接 DUMP    

00403609     E8 E2040000          call 00403AF0   

0040360E     A3 02184000          mov dword ptr ds:[401802],eax   

00403613     6A 00                push 0   

00403615     68 2B364000          push 40362B   

0040361A     6A 00                push 0   

0040361C     6A 64                push 64   

0040361E     50                   push eax   

0040361F     E8 08050000          call 00403B2C   

———————————————————————   
```

重新运行，运行`ImportREC`，选择这个进程。把`OEP`改为`00003607`，点`IT AutoSearch`，点`Get Import`，用`追踪层次 1`和`ASProtect 1.3 calculated imports`插件全部修复无效函数。`FixDump`，正常运行！`82.4K->88.1K`

—————————————————————————————————

```
         ,     \_/ 
       /| \_.-~/         \_     ,        青春都一饷
      ( /~   /            ~-.\ |
      \`\\  \_/             ~ )           忍把浮名  
  \_-~~~-.)  )\_\_/;;,.       \_  //'    
  '/\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`  
  /~~//'   /' \`~        ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~    
    
            "     "   "~'  ""
  
  Cracked By 巢水工作坊——fly [OCN][FCG][NUKE]
  
            2003-12-05  14:14
```

`4dd466d0e1e0cc123afbe504da2462ca`
