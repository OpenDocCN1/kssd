
# OllyScript0.92

> 原文：[`www.pediy.com/kssd/pediy06/pediy6602.htm`](https://www.pediy.com/kssd/pediy06/pediy6602.htm)

怪 bug，大家注意：  

平常偶 Dump 内存映像（PE）数据时，总是用 `LordPE`，大家都很熟了，近来认真试用脚本的 `DM`  

大家有否使用 `OlllyScript0.92` 功能命令：`dm addr,size,"path&filename"`

用法：  

例：  

```
dm 405000,280,"e:\redone.bin"
ret
```

结果：内存中大小为 `280h` 的数据，变成了 `29Ah` 的数据，真是哭笑不得。  

我直觉上找到它的 bug 了（98 下发现的，其它的不知道有没有）：  

它会善作主张，在为我 Dump 取处理（不处理）的文本（非文本）数据的换行`0D0A`的`0A`前添加`0D`，真是莫明其妙，它为什么那么爱追`0A`。  

bugs:  

```
the DM command has a bug,there is it dump a data file will be insert in a hex "0D" before hex  is "0A".that is unclearly.
```

我吾知上面写的英文对不对，英文好的同志可以 send bug 给作者！  

好了，等人家作者来 fixed，不知什么等到时候（人家手头好像有大工程、大买卖），不如自己看看能不能解决算了，本着“摶一搏”，“有排我挥霍”的侥幸精神，将 `OlllyScript0.92` 的 `dll` 复制一个副本到别的路径，用 `Ollydbg` 的 `loaddll.exe` 加载它，搜索一下字符串，找到 `dm` 的双击其地址，一闪之下就到位了，可是，没发现什么可疑有用的信息。难到要放弃吗，当然不能，（现在我入手有点走歪了）既然它对 `0A` 那么感兴趣，那来个搜索常数`0A`，搜了一会，找直接比较的很多呀，有点盲目，于是转向搜 `mov byte [xxx], 0D` 的，找到了好几个标上记号，经分析最可疑是下面这里了：  

```
1001F2F6  /mov ecx,dword ptr ss:[ebp-74]
1001F2F9  |sub ecx,dword ptr ss:[ebp+3AC]
1001F2FF  |lea eax,dword ptr ss:[ebp-68]
1001F302  |mov dword ptr ss:[ebp-70],edi
1001F305  |/cmp ecx,dword ptr ss:[ebp+3B0]
1001F30B  ||jnb short OLLYSCRI.1001F334
1001F30D  ||mov edx,dword ptr ss:[ebp-74]
1001F310  ||inc dword ptr ss:[ebp-74]
1001F313  ||mov dl,byte ptr ds:[edx]
1001F315  ||inc ecx
1001F316  ||cmp dl,0A////////////////////////////////////
1001F319  ||jnz short OLLYSCRI.1001F325
1001F31B  ||inc dword ptr ss:[ebp-7C]
1001F31E  ||mov byte ptr ds:[eax],0D                 ;  // 呵呵，这里对我要求输出的内容加料了，标上多个双斜杆的代码要 nop 掉！
1001F321  ||inc eax
1001F322  ||inc dword ptr ss:[ebp-70]/////////////////
1001F325  ||mov byte ptr ds:[eax],dl
1001F327  ||inc eax
1001F328  ||inc dword ptr ss:[ebp-70]
1001F32B  ||cmp dword ptr ss:[ebp-70],400
1001F332  |\jl short OLLYSCRI.1001F305
1001F334  |mov edi,eax
1001F336  |lea eax,dword ptr ss:[ebp-68]
1001F339  |sub edi,eax
1001F33B  |push 0                      ; /pOverlapped = NULL
1001F33D  |lea eax,dword ptr ss:[ebp-80]       ; |
1001F340  |push eax                     ; |pBytesWritten
1001F341  |push edi                     ; |nBytesToWrite
1001F342  |lea eax,dword ptr ss:[ebp-68]       ; |
1001F345  |push eax                     ; |Buffer
1001F346  |mov eax,dword ptr ds:[ebx]      ; |
1001F348  |push dword ptr ds:[eax+esi]     ; |hFile
1001F34B  |call dword ptr ds:<KERNEL32.WriteFile>; \WriteFile  //写文件，我开头怎么不想找这个（dm 就是写文件嘛）
1001F351  |test eax,eax
1001F353  |je short OLLYSCRI.1001F374
```

修改后存为个正本复盖到原插件目录，再次测试同样的目标任务（长度正好）,没问题了，Enjoy!!

`4dd466d0e1e0cc123afbe504da2462ca`
