
# ASProtect 2.1X 之 VB 程序&OEP 修复

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-556.htm`](https://www.pediy.com/kssd/pediy08/pediy8-556.htm)

## 基本信息

- **文章标题**: ASProtect 2.1X 之 VB 程序&OEP 修复
- **文章作者**: wynney
- **软件名称**: Klasik
- **软件大小**: 979K
- **下载地址**: 见附件
- **加壳方式**: ASProtect 2.1X
- **保护方式**: 仅代码变形+Stolen Code
- **编写语言**: VB
- **使用工具**: `Shadow`, `LoadPE`, `ImportREC`
- **操作平台**: `XP SP1`
- **软件介绍**: 国外论坛上某人求脱的小东西。。。
- **作者声明**: 凑热闹~而已^\_^

--------------------------------------------------------------------------------

## 详细过程

`VolX` 大虾的脚本已经完美无比了，佩服 ing，但是，我这个人有个习惯，不喜欢用脚本，虽然很累~但是，充实^\_^这篇陋文也是按前人的思路来作~前人太多~就不列表了：）我就按我的流程写详细吧，突出点不一样~呵呵

### 1、突击敌后--探听虚实

第一步我们需要探听以下几点~为后面做准备呀~

i、`OD` 忽略除了 `int3` 和指定异常之外的所有异常，隐藏 `OD`(我这里隐藏不隐藏好像都一样)，`F9`，中断。搜索所有参考文本。

找到两个 85 吧，双击任何一个“85”，来到代码处~

```
0130EBF8    68 E8F43001     push 130F4E8                    ; ASCII "85"
0130EBFD    E8 2A62FEFF     call 012F4E2C
0130EC02    A1 1C373101     mov eax,dword ptr ds:[131371C]
0130EC07    8B00            mov eax,dword ptr ds:[eax]
0130EC09    E8 0A8CFFFF     call 01307818                    ;F2，Shift+F9 中断下来，继续 F7
0130EC0E    84C0            test al,al
0130EC10    75 0A           jnz short 0130EC1C
0130EC12    68 E8F43001     push 130F4E8                    ; ASCII "85"
```

```
01307818    53              push ebx                        ;来到这里，去段尾。。。
01307819    56              push esi
0130781A    57              push edi
.....
.....
.....
01307959    56              push esi
0130795A    E8 59FCFFFF     call 013075B8                     ;F4 下来，接着就是 F7
0130795F    0FB707          movzx eax,word ptr ds:[edi]
01307962    83C0 02         add eax,2
01307965    03F8            add edi,eax
01307967    8A1F            mov bl,byte ptr ds:[edi]
01307969    47              inc edi
0130796A    3A5E 34         cmp bl,byte ptr ds:[esi+34]
0130796D ^ 0F85 77FFFFFF    jnz 013078EA
01307973    8BDF            mov ebx,edi
01307975    8B03            mov eax,dword ptr ds:[ebx]
01307977    85C0            test eax,eax
01307979 ^ 0F85 0AFFFFFF    jnz 01307889
0130797F    8A0424          mov al,byte ptr ss:[esp]
01307982    83C4 0C         add esp,0C
01307985    5D              pop ebp
01307986    5F              pop edi
01307987    5E              pop esi
01307988    5B              pop ebx
01307989    C3              retn
```

段尾好像都长这个样子，一个 `CALL`，两个向上的跳，一个 `retn`(简称 121)

```
013075B8    55              push ebp
013075B9    8BEC            mov ebp,esp
013075BB    81C4 F8FEFFFF   add esp,-108
013075C1    53              push ebx
....
....
....
013075F2    3BF0            cmp esi,eax
013075F4    75 5E           jnz short 01307654                ；我们探听的内容之一，纪录下吧    
013075F6    EB 01           jmp short 013075F9                ；我们探听的内容之二，纪录下吧 
....
....
....
```

在 `013075F4` 下 `F2`，`Shift+F9` 运行中断下来

此时可以看到 `EAX=4F`,`ESI=98`

好我们先不要急着按 `F9`，下面还是进行我们探听的内容之三，`Ctrl+G`-->`00401000`-->`Ctrl+B`,查找 `FF 25`-->

这样一来我们就得到探听内容之三(`0040FCC0  - FF25 C4114000   jmp dword ptr ds:[4011C4]`)长得像这样的

东西-->右键数据窗口中跟随-->往上翻翻，往下翻翻,找到 `IAT Start`(`00401000`)&`IAT End`(`004014D4`),纪录下来吧

接下来就按 `F9` 吧~按了一下 `F9` 之后，在数据窗口中 `Ctrl+G`-->`00401000`（`IAT Start`），观察 `IAT` 的变化

同时眼睛还要尖锐点注意 `ESI` 的变化，`ESI` 变化的次序 98，4F，7E

按了 N 多下，转存窗口中的 `IAT` 一直都在变，而且都出来。。。。。哎~伤害感情~原来 `IAT` 没有加密：）

那些变化看够了没？看够了就取消断点吧，断点取消了吧？把 `OD` 异常设置换下吧~`INT3` 打上勾~内存访问

去掉勾，`F9` 几下来到最后一次异常，找段尾的倒数第一个 `CALL XXXXXXXX`

```
0130FBF5    E8 1A30FDFF     call 012E2C14
0130FBFA    E8 B5D0FFFF     call 0130CCB4    ；都长这个样~F2，F9 中断下来
0130FBFF    83C4 2C         add esp,2C
0130FC02    5D              pop ebp
0130FC03    5F              pop edi
0130FC04    5E              pop esi
0130FC05    5B              pop ebx
0130FC06    C3              retn
```

接下来不要急着 `F7`，先 `Ctrl+G`-->`00401000`-->`Ctrl+B`,查找 `FF 25`，和他们在一起的会有代码变形的那些东西

（`0040FCD2    E8 29031401     call 01550000`）这个 1550000 就是我们探听的内容之四纪录下来~

```
F7 进 call 0130CCB4 去吧~Ctrl+F9 返回~这样我们在反汇编窗口下面就会发现有

0130CD44    C3              retn
```

返回到 `01530211`

不要按 `F7` 了~把 `01530211` 记住先~这便是我们要探听的内容之五了

在 `00401000` 段下 `F2`，`F9` 运行~看到这样的代码了

```
004103F8  - FF25 D4134000   jmp dword ptr ds:[4013D4]             ; msvbvm60.ThunRTMain
004103FE    0000            add byte ptr ds:[eax],al
00410400  - E9 0CFE1101     jmp 01530211
00410405    9B              wait
00410406    A2 093EBF16     mov byte ptr ds:[16BF3E09],al
0041040B    0000            add byte ptr ds:[eax],al
```

恭喜恭喜，`OEP` 被抽了~但是，`VB` 程序抽 `OEP` 是小意思了~

注意看堆栈

```
0012FFBC   01530295  返回到 01530295 来自 01590000
0012FFC0   0041744C  ASCII "VB5!6&\*"
```

这个时候你可以随便找个 `VB` 程序看看入口~

```
0040116C  >/$  68 147C4000   push VB.00407C14             "VB5!6&\*"
00401171  |.   E8 F0FFFFFF   call <jmp.MSVBVM60.ThunRTMain>
00401176  |.   0000          add byte ptr ds:[eax],al
00401178  |.   0000          add byte ptr ds:[eax],al
```

`VB` 的入口都张这样

那么我们很清楚了吧？

这个被抽的代码就是这样的了

`push 0041744C`

`call 004103F8`

`68 4C 74 41 00 E8 EE FF FF FF`

这是我们探听的内容之六

这里我说明下选那个地址做 `OEP` 呢？

看看这个

`00410400  - E9 0CFE1101     jmp 01530211`

这里的 `01530211` 不正是我们 `Stolen Code` 的 `OEP` 吗？

哎~很明了了~我们就选 `00410400` 做 `OEP`！

这是我们探听的内容之七

好虚实探听好了~现在来修复变形代码吧

这个的 `IAT` 根本没有加密，我们还是有必要事先走个过场熟悉下^\_^

### 2、`IATFixing`(没有加密嘛~跳过~)

### 3、代码变形处理

这里前面有很多朋友介绍得相当多了，我还是赘述下吧~

`OD` 忽略除了内存访问之外的所有异常，到最后一次异常~

找段尾的倒数第一个 `CALL XXXXXXXX`

```
0130FBF5    E8 1A30FDFF     call 012E2C14
0130FBFA    E8 B5D0FFFF     call 0130CCB4    ；F2，F9 中断下来~用内存分配插件分配个 1640000，SIZE=10000
0130FBFF    83C4 2C         add esp,2C
0130FC02    5D              pop ebp
0130FC03    5F              pop edi
0130FC04    5E              pop esi
0130FC05    5B              pop ebx
0130FC06    C3              retn
```

`Ctrl+B`-->`89,45,F0,B8,00,07,00,00`

找到这里

```
01307188    8945 F0         mov dword ptr ss:[ebp-10],eax
0130718B    B8 00070000     mov eax,700
01307190    E8 B7B3FDFF     call 012E254C      ；改成 jmp 1640037
```

`Ctrl+G`-->`1640000`，写上这些代码~

```
01640000    BA 00104000     mov edx,KLASIK.<ModuleEntryPoint>   ；写完之后一定要新建 EIP，再运行~
01640005    803A E8         cmp byte ptr ds:[edx],0E8
01640008    75 12           jnz short 0164001C
0164000A    8B42 01         mov eax,dword ptr ds:[edx+1]
0164000D    03C2            add eax,edx
0164000F    83C0 05         add eax,5
01640012    3D 00005501     cmp eax,1550000                     ；变形的 IAT，也是上面纪录的内容
01640017    75 03           jnz short 0164001C
01640019    EB 0C           jmp short 01640027
0164001B    90              nop
0164001C    42              inc edx
0164001D    81FA 00F08200   cmp edx,82F000                      ；00401000 区段后面的那个区段的起始地址
01640023 ^ 72 E0           jb short 01640005
01640025 - EB FE           jmp short 01640025
01640027    8915 00016401   mov dword ptr ds:[1640100],edx      ；申请的地址+100
0164002D    60              pushad
0164002E    FFE2            jmp edx
01640030    90              nop
01640031    90              nop
01640032    90              nop
01640033    90              nop
01640034    90              nop
01640035    90              nop
01640036    90              nop
01640037    60              pushad
01640038    B8 00104000     mov eax,00401000                    ；IAT Start     
0164003D    90              nop
0164003E    3910            cmp dword ptr ds:[eax],edx
01640040    75 20           jnz short 01640062
01640042    8B0D 00016401   mov ecx,dword ptr ds:[1640100]      ；申请的地址+100
```



```
01640048    C701 FF250000   mov dword ptr ds:[ecx],25FF           ；JMP[XXXXXXXX]为 25FF，CALL[XXXXXXXX]为 15FF
0164004E    8941 02         mov dword ptr ds:[ecx+2],eax
01640051    61              popad
01640052    90              nop
01640053    8B15 00016401   mov edx,dword ptr ds:[1640100]        ；申请的地址+100
01640059    90              nop
0164005A    90              nop
0164005B    90              nop
0164005C  ^ EB BE          jmp short 0164001C
0164005E    90              nop
0164005F    90              nop
01640060    90              nop
01640061    90              nop
01640062    83C0 04         add eax,4
01640065    3D D4194000     cmp eax,4014D4                          ；IAT End
0164006A  ^ 7E D2          jle short 0164003E
0164006C  ^ EB E3          jmp short 01640051

BA 00 10 40 00 80 3A E8 75 12 8B 42 01 03 C2 83 C0 05 3D 00 00 55 01 75 03 EB 0C 90 42 81 FA 00
F0 82 00 72 E0 EB FE 89 15 00 01 64 01 60 FF E2 90 90 90 90 90 90 90 60 B8 00 10 40 00 90 39 10
75 20 8B 0D 00 01 64 01 C7 01 FF 25 00 00 89 41 02 61 90 8B 15 00 01 64 01 90 90 90 EB BE 90 90
90 90 83 C0 04 3D D4 14 40 00 7E D2 EB E3
```

这里没什么技巧，不过一定得注意我上面标注好的位置！在你按 `F9` 之前先确认一遍~不然~嘿嘿~

`F9` 运行之后~判断成功如否的一个标志就是左上角 `OD` 提示“运行”~机器卡卡卡~哈哈~

`OK`，此时 `LoadPE` 完全 `Dump`~打开 `ImportREC`，`OEP` 填 `10400` 看看`~点“自动查找 IAT”提示“在此 OEP 处。。。”

晕糊糊~~呵呵~不急~因为这个地方的代码被抽取了嘛`~当然。。。我们填 `103EF`~再来看看~啊。。晕糊糊~有 4 个无效哦

仔细看看~后面 3 个没用~剪掉吧~但是，在 `MSVBVM60.DLL` 里面却有一个无效~~右键反汇编查看。。。有个 `// = msvbvm60.dll/00BB//DllFunctionCall`，就把它改成 `DllFunctionCall` 吧~`OK`，此时把 `OEP` 换成 `10400`，`FixDump`！

用 `OD` 打开修复后的程序~二进制粘贴 `68 4C 74 41 00 E8 EE FF FF FF` 保存下`~OK~运行正常~体积 4.44M：（懒得减肥了

太累了~~

`PS`：如果你想使用区段填补的方法~可以以 `01530211` 做 `OEP`，把 `012E0000`~`016B0000` 中所有的区段都补回去~不过等着你的

是 `Route Check`，这个用 `SYSCOM` 斑竹的代码。。。~`OK`，当然你要知道怎么跟到那个地方去~很简单~自己去跟下~

--------------------------------------------------------------------------------

### 【经验总结】

`VB` 程序的加壳一直都是软件作者很头痛的问题吧~呵呵~看这个程序就知道了，软柿子一个！没什么技术含量，大虾就飘过吧

~

--------------------------------------------------------------------------------

### 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 04 月 27 日 8:16:21

`4dd466d0e1e0cc123afbe504da2462ca`
