
# Patch 注册 Ultra Tag Editor V2.0.8

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-529.htm`](https://www.pediy.com/kssd/pediy07/pediy7-529.htm)

Patch 注册 Ultra Tag Editor V2.0.8(ASProtect V1.X 壳)的方法

下载页面： `http://yncnc.onlinedown.net/soft/25235.htm`

软件大小： 1264KB

软件语言： 英文

软件类别： 共享版/MP3 制作

应用平台： `Win9x/NT/2000/XP`

加入时间： 2005-3-19 19:54:57

下载次数： 1153

推荐等级： *****

软件介绍： 标签编辑器

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`OllyDbg`、`LordPE`、`UltraEdit`

【脱壳过程】：

看了 fly 老大的"Patch 注册 ASProtect V1.X 壳保护程序的方法"一文，受益非潜，看懂原理后就可以"举一反三"啦，这里只是 fly 老大的翻版，响应 fly 教给我的任务，大家就凑活着看吧，再次感谢 fly 的无私奉献，真是我的偶像，顺便拍两下。

设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其它异常选项。下面的各步除了特殊说明的，皆为这个异常选项设置。

老规矩：用 `IsDebug` V1.4 插件去掉 `Ollydbg` 的调试器标志。

## 一、获得 ASProtect 注册名 Pre-Dip 处理的地址

```
00401000 T> 68 01606400       push TagEdito.00646001

//进入 Ollydbg 后暂停在这

00401005     E8 01000000      call TagEdito.0040100B
0040100A     C3               retn
```

Shift+F9 运行 N 次（我这里是 23 次），直至堆栈中第 2 次看见硬盘指纹:

```
0012FF38       0012FF40  指针到下一个 SEH 记录
0012FF3C       00FF4902  SE 句柄
0012FF40       0012FFE0  指针到下一个 SEH 记录
0012FF44       00FF4EF4  SE 句柄
0012FF48       0012FF90
0012FF4C       00FE0000
0012FF50       00FC0000
0012FF54       00FF4400
0012FF58       00000000
0012FF5C       0100AF20  ASCII "dXlooADwifU="
```

Alt+M 打开内存查看窗口，在 `00401000` 段“设置内存访问断点”，Shift+F9 通过异常，断下

```
0047398C     55               push ebp
0047398D     8BEC             mov ebp,esp
0047398F     8B45 08          mov eax,dword ptr ss:[ebp+8]

//Stack ss:[ebp+8]]=00FE3951 注册名保存地址 ★

//这个地址知道就可以拉，patch 把注册名地址指向了自己的大名。

00473992     85C0             test eax,eax

//地址"00473992"要记下的，patch 后要从这里继续的，即 jmp 00473992

00473994     74 0C            je short TagEdito.004739A2
00473996     8038 00          cmp byte ptr ds:[eax],0
00473999     74 07            je short TagEdito.004739A2
0047399B     C605 40005900 01 mov byte ptr ds:[590040],1
004739A2     8B15 34005900    mov edx,dword ptr ds:[590034]
004739A8     8915 30005900    mov dword ptr ds:[590030],edx
004739AE     A3 34005900      mov dword ptr ds:[590034],eax
004739B3     5D               pop ebp
004739B4     C2 0400          ret 4

00FF4AE5     A1 2066FF00      mov eax,dword ptr ds:[FF6620]
00FF4AEA     8B40 04          mov eax,dword ptr ds:[eax+4]
00FF4AED     FFD0             call eax
00FF4AEF     8B15 0865FF00    mov edx,dword ptr ds:[FF6508]

//执行返回到这里，上面的 call eax 就是读取注册名

eax=[00FF6620]+4=00FF7984==>8C 39 47 00,所以 call eax 等价为

call 0047398C 记住：0047398C 和 00FF7984
```

—————————————————————————————————

## 二、第一次回溯：00FF7984

重新载入程序。忽略除了“内存访问异常”之外的所有其它异常选项，Shift+F9 运行 1 次

```
00FF446C     3100             xor dword ptr ds:[eax],eax

//第 1 次内存异常
```

在转存中 Ctrl+G：`00FF7984`，或者下命令：`DB 00FF7984`

选中 `00FF7984` 处 4 个字节，下“内存写入”断点。忽略所有异常，Shift+F9

```
00FE27B8     F3:AB            rep stos dword ptr es:[edi]

//第一次中断

00FE2663     F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi]

//第二次中断

ds:[esi]=[0100128E]=0007398C ★

es:[edi]=[00FF7984]=00000000

0047398C＝0007398C－基址 0040000
```

记住：`0100128E`

—————————————————————————————————

## 三、得到壳重定位后的基址 + 第二次回溯：0100128E

重新载入程序。忽略所有异常，BP `VirtualAlloc` 然后把断点挪到 `VirtualAlloc` 的段尾

Shift+F9 运行，观察壳所申请到的内存地址

————————————————————————

### 1、得到壳重定位后的基址

```
7C809A81 8BFF mov edi,edi

//VirtualAlloc

7C809A83 55 push ebp
7C809A84 8BEC mov ebp,esp
7C809A86 FF75 14 push dword ptr ss:[ebp+14]
7C809A89 FF75 10 push dword ptr ss:[ebp+10]
7C809A8C FF75 0C push dword ptr ss:[ebp+C]
7C809A8F FF75 08 push dword ptr ss:[ebp+8]
7C809A92 6A FF push -1
7C809A94 E8 09000000 call kernel32.VirtualAllocEx
7C809A99 5D pop ebp
7C809A9A C2 1000 ret 10

//这里下断

//因为 Stack ss:[ebp+8]]=00FE3951 注册名保存地址 ★,可知壳重定位地址为 00FE0000
```

当我们在 `VirtualAlloc` 中断第 2 次，会发现申请的 `EAX` 返回值是 `00FE0000`

`00FE0000` 就是这个程序的壳重定位后的基址，F7，需要跟踪一下

```
00646588     FF95 DF030000    call dword ptr ss:[ebp+3DF]
0064658E     8985 BF010000    mov dword ptr ss:[ebp+1BF],eax

//[00646668]=00FE0000 ★

00646594     8B55 5B          mov edx,dword ptr ss:[ebp+5B]
00646597     8B85 BF010000    mov eax,dword ptr ss:[ebp+1BF]
0064659D     8902             mov dword ptr ds:[edx],eax
```

记住：`[00646668]`，保存壳重定位后的基址。

————————————————————————

### 2、第二次回溯：0100128E

OK，我们继续 Shift+F9 运行，观察壳所申请到的内存地址

这个需要不少次才看到 `EAX=01000000`，某些时候需要中断 2 次 `EAX=01000000`

取消 `7C809A9A` 处断点。在转存中 Ctrl+G：`0100128E`

选中 `0100128E` 处 4 个字节，下“内存写入”断点。Shift+F9

```
00FE267D     F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi]

//中断在这里

ds:[esi]=[006572E4]=1AEF4F1F

es:[edi]=[0100128F]=00000000
```

`006572E3`＝＝>`0100128E` `91 1F 4F EF` ★

你可以用 `LordPE` 的 `FLC` 功能查看 `TagEditor.exe` 的 `006572E3`（偏移 `000BBCE3`）处数据：`91 1F 4F EF`

Good！ ASProtect 壳下面会对这些数据进行简单的运算，变换成 `0007398C`

—————————————————————————————————

## 四、得到运算数据 + 求逆

选择 `0100128E` 处的 4 个字节，设置“内存访问”断点。

Shift+F9，中断在 `00FD9936` 处，开始运算

```
00FE98E6     41               inc ecx
00FE98E7     80E1 FF          and cl,0FF
00FE98EA     33DB             xor ebx,ebx
00FE98EC     8AD9             mov bl,cl
00FE98EE     021418           add dl,byte ptr ds:[eax+ebx]
00FE98F1     80E2 FF          and dl,0FF
00FE98F4     33DB             xor ebx,ebx
00FE98F6     8AD9             mov bl,cl
00FE98F8     8A1C18           mov bl,byte ptr ds:[eax+ebx]
00FE98FB     885D F7          mov byte ptr ss:[ebp-9],bl
00FE98FE     8BF2             mov esi,edx
00FE9900     81E6 FF000000    and esi,0FF
00FE9906     33DB             xor ebx,ebx
00FE9908     8AD9             mov bl,cl
00FE990A     8D3C18           lea edi,dword ptr ds:[eax+ebx]
00FE990D     8A1C30           mov bl,byte ptr ds:[eax+esi]
00FE9910     881F             mov byte ptr ds:[edi],bl
00FE9912     8A5D F7          mov bl,byte ptr ss:[ebp-9]
00FE9915     881C30           mov byte ptr ds:[eax+esi],bl
00FE9918     33DB             xor ebx,ebx
00FE991A     8AD9             mov bl,cl
00FE991C     8A1C18           mov bl,byte ptr ds:[eax+ebx]
00FE991F     8BF2             mov esi,edx
00FE9921     81E6 FF000000    and esi,0FF
00FE9927     021C30           add bl,byte ptr ds:[eax+esi]
00FE992A     80E3 FF          and bl,0FF
00FE992D     885D F7          mov byte ptr ss:[ebp-9],bl
00FE9930     8B5D FC          mov ebx,dword ptr ss:[ebp-4]
00FE9933     8B75 F0          mov esi,dword ptr ss:[ebp-10]
00FE9936     8A1C33           mov bl,byte ptr ds:[ebx+esi]

//中断在这里

00FE9939     0FB675 F7        movzx esi,byte ptr ss:[ebp-9]
00FE993D     321C30           xor bl,byte ptr ds:[eax+esi]
```

//异或 注意：记住运算的数据！可是异或的结果都为 0，直到 `0100129E` 才恢复我们想要的

数据 `8C 39 07 00`，`LordPE` 的 `FLC` 功能查看 `TagEditor.exe` 的 `006572E3+10`（偏移 `000BBCE3+10`）处数据

为：`7B 2B D9 AA`，与预想的偏差 10。

`//①、BL=7B XOR F7=8C`

`//②、BL=2B XOR 12=39`

`//③、BL=D9 XOR DE=07`

`//④、BL=AA XOR AA=00`

运算结束得到： ★

`0100129E` `8C 39 07 00`

`TagEditor.exe` 的 `000BBCF3` 偏移地址处是 `7B2BD9AA`，这个值正是上面进行运算的数据。

我们要把这里的 `0047398C`（`0007398C+0040000`）改为运算后跳向可以写 Patch 代码的地址！

一般在 PE 文件偏移 80－100 间有段空白地址可以给我们利用

所以要修改这个运算后的 Patch 地址为 `00400080`

求逆 `00000080`（`00400080－00400000`）如下：

`①、BL=80 XOR F7=77`

`②、BL=00 XOR 12=12`

`③、BL=00 XOR DE=DE`

`④、BL=00 XOR AA=AA`

使用 `UltraEdit` 修改 `TagEditor.exe` 偏移 `000BBCF3` 处改为：`7712DEAA`

另存为 `daxia2002.eXe`，第六步时需要在里面进行最终的修改

—————————————————————————————————

## 五、RegOpenKeyExA

得到上面的地址运算数据后，我们还要得到壳通过读取注册表中的信息判断是否过期的地方。

Ctrl+F 在“整个段块”搜索命令：`push 20019`

`00FD961D` `68 19000200` `push 20019`

//找到这里

```
00FE9A99     68 19000200      push 20019
00FE9A9E     6A 00            push 0
00FE9AA0     8BC7             mov eax,edi
00FE9AA2     E8 A59EFFFF      call 00FE394C
00FE9AA7     50               push eax
00FE9AA8     56               push esi
```



# 逆向分析笔记

## 汇编代码片段
```
00FE9AA9     E8 3EB9FFFF      call 00FE53EC                              ; jmp to advapi32.RegOpenKeyExA  

//Call RegOpenKeyExA  

00FE9AAE     85C0             test eax,eax  

00FE9AB0     0F85 84000000    jnz 00FE9B3A  
```

`call 00FE53EC` 就是 `call RegOpenKeyExA`

点右键，搜索全部命令：`call 00FE53EC`

### 查找命令
- 地址
- 反汇编
- 注释
- `jmp to advapi32.RegOpenKeyExA`
- `jmp to advapi32.RegOpenKeyExA`
- `jmp to advapi32.RegOpenKeyExA`
- `jmp to advapi32.RegOpenKeyExA`

一般修改后面 3 个 `CALL` 后的跳转就行了：

```
00FE9BF2     E8 F5B7FFFF      call 00FE53EC                              ; jmp to advapi32.RegOpenKeyExA  

00FE9BF7     85C0             test eax,eax  

00FE9BF9     75 66            jnz short 00FE9C61  

//修改①、 jmp short 00FE9C61  

00FE9CA5     E8 42B7FFFF      call 00FE53EC                              ; jmp to advapi32.RegOpenKeyExA  

00FE9CAA     85C0             test eax,eax  

00FE9CAC     0F85 93000000    jnz 00FE9D45  

//修改②、 jmp 00FE9D45  

00FE9D8D     E8 5AB6FFFF      call 00FE53EC                              ; jmp to advapi32.RegOpenKeyExA  

00FE9D92     85C0             test eax,eax  

00FE9D94     75 30            jnz short 00FE9DC6  

//修改③、 jmp 00FE9DC6  
```

记住这三个需要修改的地方：`00FE9BF9`、`00FE9CAC`、`00FE9D94`

---

## 六、Patch 最终处理

用 `Ollydbg` 载入第四步保存的 `fly.eXe`，忽略所有异常

直接下断：`BP 400080`，`Shift+F9` 后中断下来

```
00400080 0000 add byte ptr ds:[eax],al  

//中断  
```

OK，`ASProtect` 成功到达我们指定的地点！现在写入 `Patch` 代码：

```
00400080     A1 68666400      mov eax,dword ptr ds:[646668]  

//[00646668]中是第三步得到的壳重定位后的基址  

00400085     C680 F99B0000 EB mov byte ptr ds:[eax+9BF9],0EB  

//这三句是修改壳通过读取注册表中的信息判断是否过期的地方  

//修改①、 jmp short 00FE9C61  

0040008C     C780 AC9C0000 E9>mov dword ptr ds:[eax+9CAC],94E9  

//修改②、 jmp 00FE9D45  

00400096     C680 949D0000 EB mov byte ptr ds:[eax+9D94],0EB  

//修改③、 jmp 00FE9DC6  

0040009D     55               push ebp  

0040009E     8BEC             mov ebp,esp  

//上面 2 句是把 0047398C 和 0047398D 代码挪到这里执行  

004000A0     B8 C0004000      mov eax,TagEdito.004000C0 ; ASCII "daxia2002[DFCG]"  

//直接在 004000C0 处写入注册名：daxia2002[DFCG]  

004000A5    - E9 E8380700    jmp TagEdito.00473992  

//跳回去继续流程  
```

从 `Ollydbg` 中“二进制复制”如下：

```
A1 68 66 64 00 C6 80 F9 9B 00 00 EB C7 80 AC 9C 00 00 E9 94 00 00 C6 80 94 9D 00 00 EB 55 8B EC

B8 C0 00 40 00 E9 E8 38 07 00
```

前面所有的工作只为了这里写入的几行代码，而这几行代码则使得这个使用 `ASProtect` 保护的程序变成了注册版！

至于制作 `Patch`，可以随意选择补丁工具，看你的喜好了。我选择 `keymaker` 制作了通用的文件补丁。

照猫划虎总算完成了 `fly` 老大教给的任务，再一次感谢偶的偶像 `fly`，谢谢他的无私奉献。

`4dd466d0e1e0cc123afbe504da2462ca`
