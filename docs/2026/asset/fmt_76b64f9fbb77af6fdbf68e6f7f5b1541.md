
# 梦幻Ollydbg之寻找OEP篇――Divx Avi Asf Wmv Wma Rm Rmvb修复器 V3.21

标 题:梦幻Ollydbg之寻找OEP篇——Divx Avi Asf Wmv Wma Rm Rmvb修复器 V3.21

发信人:fly

时 间:2003年11月03日 11:05

详细信息:

梦幻Ollydbg之寻找OEP篇――Divx Avi Asf Wmv Wma Rm Rmvb修复器 V3.21

下载页面： `http://61.151.251.199/soft/fixvideo_v3.21.rar`

软件大小： 1322 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 文件修复

应用平台： Win9x/NT/2000/XP

加入时间： 2003-08-28 11:11:24

下载次数： 146813

推荐等级： ***

开 发 商： `http://www.cnghost.com/`

## 【软件简介】

只需轻松的一次点击就可以修复不能拖动的或者不能播放的divx avi asf wmv wma rm rmvb文件。Divx Avi Asf Wmv Wma Rm Rmvb 修复器可以修复你通过http,ftp,mms,rtsp方式由于某些原因没有下载完全的divx avi asf wmv wma rm rmvb文件。修复后的文件可以流畅的播放，自由的拖动。Divx Avi Asf Wmv Wma Rm Rmvb 修复器也可以修复在播放过程中不能拖动的divx avi asf wmv wma rm rmvb文件。修复后的文件可以随意的拖动。Divx Avi Asf Wmv Wma Rm Rmvb 修复器还有另一个功能，他可以强行修复部分损坏的divx avi asf wmv wma rm rmvb文件。修复后的文件可以跳过坏的数据块，继续播放。如果一些播放器，例如Mediaplayer,realplayer提示dvix avi asf wmv wma rm rmvb文件不能播放或文件损坏，都可以尝试用Divx Avi Asf Wmv Wma Rm Rmvb 修复器来修复。

## 【作者声明】

初学Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 【调试环境】

WinXP、Ollydbg1.09修改版、PEiD、LordPE、ImportREC

―――――――――――――――――――――――――――――――――

## 【过    程】

这个V3.21版的`videofixer.exe`是`ACProtect V1.09`加壳的，可能是所知道的较早使用ACProtect加壳的软件。众所周知，123112 大侠的《妖幻TRW and videofixer的脱壳方法之我之拙见》可谓是网上见到的最早的一篇脱ACProtect的经典脱文。他是采用TRW分析的，有朋友要求偶再用Ollydbg演示一下如何走到OEP的，于是偶无聊的时候就跟了一下，聊作游戏。省了不少事，最起码知道程序返回的入口呀。感谢众位前辈无私的奉献！

1、偶使用的调试器：Ollydbg Anti ACProtect For XP 修改版 ―― `flyODBG`

下载页面：`http://tongtian.net/pediybbs/viewtopic.php?t=1532`

2、反跟踪可以参考：梦幻Ollydbg ―― 浅谈 ACProtect V1.09 Pro 的反跟踪And脱壳

相关页面：`http://tongtian.net/pediybbs/viewtopic.php?t=1525`

OK，设置 `flyODBG` 隐藏（自动隐藏，只是偶还要提醒一下朋友），设置 忽略 除了“内存访问异常”之外的所有其他异常选项。打开 `WinAmp` :-)  分析壳是一件比较费时且紧张的事，有音乐相伴会轻松点啦。

――――――――――――――――――――――――

```
77F7F571     C3                   retn

                                  ====>进入OD后断在这！
```

`F9`运行，弹出数十个“入口点预警”的对话框，一一确定之。偶们就来到 `INT 1` 异常处啦。

```
0070DE96     CD 01                int 1

                                  ====>ACProtect典型异常  不用看堆栈的第2条地址啦

0070DE98     40                   inc eax

                                  ====>直接此处下断！Shift+F9断在这里！
```

```
0070DE99     40                   inc eax
0070DE9A     0BC0                 or eax, eax
0070DE9C     0F85 B6000000        jnz videofix.0070DF58
0070DF58     33C0                 xor eax,eax
0070DF5A     64:8F00              pop dword ptr fs:[eax]
0070DF5D     58                   pop eax
0070DF5E     60                   pushad
0070DF5F     E8 00000000          call videofix.0070DF64
0070DF64     5E                   pop esi
0070DF65     83EE 06              sub esi,6
0070DF68     B9 0C010000          mov ecx,10C
0070DF6D     29CE                 sub esi,ecx
0070DF6F     BA 916FCDD0          mov edx,D0CD6F91
0070DF74     C1E9 02              shr ecx,2
0070DF77     83E9 02              sub ecx,2
0070DF7A     83F9 00              cmp ecx,0
0070DF7D     7C 1A                jl short videofix.0070DF99
0070DF7F     8B048E               mov eax,dword ptr ds:[esi+ecx*4]
0070DF82     8B5C8E 04            mov ebx,dword ptr ds:[esi+ecx*4+4]
0070DF86     03C3                 add eax,ebx
0070DF88     C1C0 18              rol eax,18
0070DF8B     33C2                 xor eax,edx
0070DF8D     81EA B962CB32        sub edx,32CB62B9
0070DF93     89048E               mov dword ptr ds:[esi+ecx*4],eax
0070DF96     49                   dec ecx
0070DF97     EB E1                jmp short videofix.0070DF7A

                                  ====>F4下去跳出循环！
```

```
0070DF99     61                   popad
0070DF9A     61                   popad
0070DF9B     C3                   retn

                                  ====>返回到 00716C21
```

――――――――――――――――――――――――

下面就是硬“功夫”啦，呵呵，只是磨练你的耐心罢了，只要你能坐的住冷板凳就行了。用`F7`单步走，注意`F8`带过几个CALL，用`F4`跳过明显的循环。下面这段偶走了约两个小时，可怜偶的F7键呀，自从用OD脱壳它就日渐“憔悴”了 :-(

```
00716C21     E9 09000000          jmp videofix.00716C2F
00716C21     E9 09000000          jmp videofix.00716C2F
00716C2F     0F84 02000000        je videofix.00716C37
00716C35     13F8                 adc edi,eax
00716C37     81C5 1DB373AF        add ebp,AF73B31D
00716C3D     50                   push eax
00716C3E     E8 01000000          call videofix.00716C44
00716C44     58                   pop eax
00716C45     58                   pop eax
00716C46     F9                   stc
00716C47     7C 02                jl short videofix.00716C4B
00716C4B     E9 06000000          jmp videofix.00716C56
00716C56     83C0 04              add eax,4
00716C59     50                   push eax
00716C5A     E8 01000000          call videofix.00716C60
00716C60     58                   pop eax
00716C61     58                   pop eax
00716C62     E9 03000000          jmp videofix.00716C6A
00716C6A     46                   inc esi
00716C6B     4F                   dec edi
00716C6C     83C2 FF              add edx,-1
00716C6F     0F85 37FFFFFF        jnz videofix.00716BAC

                                  ====>F4下去
```

```
00716C75     E8 01000000          call videofix.00716C7B
00716C7B     830424 06            add dword ptr ss:[esp],6
00716C7F     C3                   retn
00716C80     E9 08000000          jmp videofix.00716C8D
00716C8D     E9 0D000000          jmp videofix.00716C9F
00716C9F     EB 01                jmp short videofix.00716CA2
00716CA2     E9 05000000          jmp videofix.00716CAC
00716CAC     E9 09000000          jmp videofix.00716CBA
00716CBA     68 E56D7100          push videofix.00716DE5
00716CBF     41                   inc ecx
00716CC0     5F                   pop edi
00716CC1     EB 01                jmp short videofix.00716CC4
00716CC4     E9 03000000          jmp videofix.00716CCC
00716CCC     8BD7                 mov edx,edi
00716CCE     0F8C 02000000        jl videofix.00716CD6
00716CD4     85CA                 test edx,ecx
00716CD6     68 11801D9B          push 9B1D8011
00716CDB     0F83 01000000        jnb videofix.00716CE2
00716CE2     58                   pop eax
00716CE3     EB 01                jmp short videofix.00716CE6
```

…… …… 省 略 …… ……       否则有灌水“重大嫌疑”  :-)  同样的代码类型啦

```
00716DCA     83ED 01              sub ebp,1
00716DCD     0F85 47FFFFFF        jnz videofix.00716D1A

                                  ====>F4下去
```

```
00716DD3     E8 01000000          call videofix.00716DD9
00716E80     83C4 04              add esp,4
00716E83     E8 7E9EFFFF          call videofix.00710D06

                                  ====>F8带过
```

```
00716E88     0F87 04000000        ja videofix.00716E92
00716F1D     E8 B06AFFFF          call videofix.0070D9D2

                                  ====>F8带过
```

```
00716F22     E9 0A000000          jmp videofix.00716F31
00716F4C     4F                   dec edi
00716F4D     0F85 25FFFFFF        jnz videofix.00716E78

                                  ====>F4下去
```

```
00716F53     E8 01000000          call videofix.00716F59
0071701B     E8 AB65FFFF          call videofix.0070D5CB

                                  ====>F8带过
```

```
00717020     E9 09000000          jmp videofix.0071702E
0071709A     46                   inc esi
0071709B     83C7 FF              add edi,-1
0071709E     0F85 4AFFFFFF        jnz videofix.00716FEE

                                  ====>F4下去
```

```
007170A4     EB 01                jmp short videofix.007170A7
007171F7     83C6 FF              add esi,-1
007171FA     0F85 3FFFFFFF        jnz videofix.0071713F

                                  ====>F4下去
```

```
00717200     50                   push eax
00717201     E8 01000000          call videofix.00717207
0071726B     E8 2A5EFFFF          call videofix.0070D09A

                                  ====>F8带过
```

```
00717338     4E                   dec esi
00717339     0F85 3BFFFFFF        jnz videofix.0071727A

                                  ====>F4下去
```

```
0071733F     7C 03                jl short videofix.00717344
00717341     7D 01                jge short videofix.00717344
0071747D     48                   dec eax
0071747E     0F85 44FFFFFF        jnz videofix.007173C8

                                  ====>F4下去
```
