
# [原创]新手补给之脱壳1——DBPE

## 文章信息
【文章标题】: 新手补给之DBPE
【文章作者】: rocktx
【软件名称】: 雪狐提醒簿 v3.0beta1
【软件大小】: 2.74MB
【下载地址】: 狗狗一下
【加壳方式】: DBPE 2.33
【编写语言】: Delphi
【使用工具】: Ollydbg、LordPE、ImportRec
【操作平台】: Win2K/XP SP2
【软件介绍】: 老版本就不作介绍了
【作者声明】: 只为学习，失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

一直没碰到过幻影的壳，今天终于让我给逮住了，也算是补补课吧，新手万一也碰到了，可以拿来主义一下！

高手们只管超光速飞过...

幻影DBPE v2.33 加壳程序的入口特征：

```
008C9000 R> /EB 20               jmp short RemindBo.008C9022
008C9002    |0000                add byte ptr ds:[eax],al
008C9004    |40                  inc eax
008C9005    |0000                add byte ptr ds:[eax],al
008C9007    |0040 00             add byte ptr ds:[eax],al
008C900A    |0000                add byte ptr ds:[eax],al
008C900C    |0000                add byte ptr ds:[eax],al
008C900E    |0000                add byte ptr ds:[eax],al
008C9010    |0090 4C000B00       add byte ptr ds:[eax+B004C],dl
008C9016    |0000                add byte ptr ds:[eax],al
008C9018    |0230                add dh,byte ptr ds:[eax]
008C901A    |0000                add byte ptr ds:[eax],al
008C901C    |0000                add byte ptr ds:[eax],al
008C901E    |0000                add byte ptr ds:[eax],al
008C9020    |0000                add byte ptr ds:[eax],al
008C9022    \9C                  pushfd
008C9023     55                  push ebp
008C9024     57                  push edi
008C9025     56                  push esi
008C9026     52                  push edx
008C9027     51                  push ecx
008C9028     53                  push ebx
008C9029     9C                  pushfd
008C902A     E8 00000000         call RemindBo.008C902F
```

往下`F7`单步跟踪会发现两个函数调用 `VirtualAlloc`、`VirtualFree`；

如果用断点 `bp VirtualAlloc` 跟踪，运行两次就挂了，这是DBPE壳的AntiDebug在作祟；

主要是通过修改`IDT`(`Interrupt Descriptor Table` 中断向量表)来阻止调试，表现为：

1、OD重新载入加壳程序后，无法停在入口，程序直接跑飞；
2、严重一点，直接重启计算机；
3、再严重一点，直接蓝屏；
4、再再严重一点，首先（同时省去286字...）

对抗DBPE的步骤（前提是曾经运行过幻影加壳的程序）：

1、用`Trial-Reset`将DBPE的注册表信息清除（一般没什么东西）；
2、加壳程序运行后，会在`c:\windows\system32`生成`cdcd.sys`文件，并注册一个叫做`Cdsys`的服务；  
这里要用`Wsyscheck`、`Syscheck2`等高级安全工具，停止并删除该服务以及`cdcd.sys`文件；
3、如果软件过期，可能会影响脱壳，所以最好先修改系统时间；
4、重启计算机，用`Xidt.exe`保存当前的`idt`；
5、每次调试DBPE加壳的程序时，如果出错（也可能不明显），就要用`Xidt.exe`恢复保存的`idt`；

废话不多，开始脱壳：

### 1、寻找OEP

下断点 `he VirtualAlloc`，`Shift+F9`三次，查看堆栈：

```
0013FF94   008E6234  /CALL 到 VirtualAlloc 来自 RemindBo.008E622E
0013FF98   7FF60000  |Address = 7FF60000                              ；注意这个地址
0013FF9C   0003078D  |Size = 3078D (198541.)
0013FFA0   00101000  |AllocationType = MEM_COMMIT|MEM_TOP_DOWN
0013FFA4   00000040  \Protect = PAGE_EXECUTE_READWRITE
```

DBPE壳代码的地址一般在7F打头的高位区段，这里申请的是`7FF60000`；

然后数据窗口跟随寄存器窗口中`ESP`的地址，下`WORD`硬件访问断点，`Shift+F9`两次后，来到壳代码领空：

```
7FF7D504     83C4 04         add esp,4
7FF7D507     EB 02           jmp short 7FF7D50B
7FF7D509     99              cdq
7FF7D50A     EB FF           jmp short 7FF7D50B
7FF7D50C     0C 24           or al,24
7FF7D50E     71 01           jno short 7FF7D511
7FF7D510     E8 79E07A01     call 8172B58E
7FF7D515   ^ 75 83           jnz short 7FF7D49A
```

查找命令`jmp eax`，发现有两个，选择当前地址以后的，来到

```
7FF8AACB     6A 00           push 0
7FF8AACD     FF96 EA664300   call dword ptr ds:[esi+4366EA]
7FF8AAD3     61              popad
7FF8AAD4     58              pop eax                                  ；OEP地址pop到eax
7FF8AAD5     83F8 FF         cmp eax,-1
7FF8AAD8     75 05           jnz short 7FF8AADF
7FF8AADA     33C0            xor eax,eax
7FF8AADC     C2 0C00         retn 0C
7FF8AADF     FFE0            jmp eax                                  ；这里跳到OEP
```

在`jmp eax` 处下内存访问断点，`F9`运行后来到`OEP`：

```
00693770     55              push ebp
00693771     8BEC            mov ebp,esp
00693773     83C4 DC         add esp,-24
00693776     33C0            xor eax,eax
00693778     8945 DC         mov dword ptr ss:[ebp-24],eax
0069377B     B8 B02D6900     mov eax,RemindBo.00692DB0
00693780     E8 AB41D7FF     call RemindBo.00407930
00693785     33C0            xor eax,eax
00693787     55              push ebp
00693788     68 07396900     push RemindBo.00693907
0069378D     64:FF30         push dword ptr fs:[eax]
00693790     64:8920         mov dword ptr fs:[eax],esp
```

### 2、处理IAT

右键-查找-所有模块间的调用，发现有很多`IAT`跳转跑到壳里去了：

```
0040143C   - FF25 49E0F87F   jmp dword ptr ds:[7FF8E049]
00401442     8BC0            mov eax,eax
00401444   - FF25 4DE0F87F   jmp dword ptr ds:[7FF8E04D]
0040144A     8BC0            mov eax,eax
0040144C   - FF25 51E0F87F   jmp dword ptr ds:[7FF8E051]
00401452     8BC0            mov eax,eax
00401454   - FF25 55E0F87F   jmp dword ptr ds:[7FF8E055]
0040145A     8BC0            mov eax,eax
0040145C   - FF25 59E0F87F   jmp dword ptr ds:[7FF8E059]
00401462     8BC0            mov eax,eax
00401464   - FF25 5DE0F87F   jmp dword ptr ds:[7FF8E05D]
0040146A     8BC0            mov eax,eax
0040146C   - FF25 61E0F87F   jmp dword ptr ds:[7FF8E061]
```

当然也有一部分`IAT`没有被修改

```
004087DC   - FF25 18686B00   jmp dword ptr ds:[6B6818]
004087E2     8BC0            mov eax,eax
004087E4   - FF25 14686B00   jmp dword ptr ds:[6B6814]
004087EA     8BC0            mov eax,eax
004087EC   - FF25 10686B00   jmp dword ptr ds:[6B6810]
004087F2     8BC0            mov eax,eax
004087F4   - FF25 0C686B00   jmp dword ptr ds:[6B680C]
004087FA     8BC0            mov eax,eax
004087FC   - FF25 08686B00   jmp dword ptr ds:[6B6808]
00408802     8BC0            mov eax,eax
00408804   - FF25 04686B00   jmp dword ptr ds:[6B6804]
0040880A     8BC0            mov eax,eax
0040880C   - FF25 00686B00   jmp dword ptr ds:[6B6800]
```

可以用patch代码修复，但比较麻烦；

程序中`idata`的区段是`006B6000`，查看发现：

```
006B636C  77DDEAF4  絷輜    advapi32.RegCreateKeyExA
006B6370  77DD6BF0  餶輜    advapi32.RegCloseKey
006B6374  77DD7753  Sw輜   advapi32.OpenProcessToken
006B6378  77DFD11B  堰w   advapi32.LookupPrivilegeValueA
006B637C  77DFC534  4胚w   advapi32.AdjustTokenPrivileges
006B6380  00000000  ....
006B6384  7FF87F15  ?   ASCII "h6"
006B6388  7FF87F20   ?   ASCII "h7"
006B638C  7FF87F2B  +?   ASCII "h8"
006B6390  7FF87F36  6?   ASCII "h9"
006B6394  7FF87F41  A?   ASCII "h:"
006B6398  7FF87F4C  L?   ASCII "h;"
006B639C  7FF87F57  W?   ASCII "h<"
```

这下知道该怎么办了，关闭`OD`（不想出意外的话），然后用`Xidt.exe`写入之前保存的`idt`，再用`OD`载入程序；

数据窗口跟随地址：`006B6390`，下内存访问断点后运行，停在

```
7FF7EBE4     66:3103         xor word ptr ds:[ebx],ax               ；停在这里
7FF7EBE7     7A 03           jpe short 7FF7EBEC
7FF7EBE9     7B 01           jpo short 7FF7EBEC
7FF7EBEB     75 66           jnz short 7FF7EC53
7FF7EBED     48              dec eax
7FF7EBEE     9C              pushfd
7FF7EBEF     6A 10           push 10
7FF7EBF1     73 0B           jnb short 7FF7EBFE
7FF7EBF3     EB 02           jmp short 7FF7EBF7
7FF7EBF5     C151 E8 06      rcl dword ptr ds:[ecx-18],6
```

清除内存断点，用花指令去除器插件去掉花指令，效果如下

```
7FF7EBE4     66:3103         xor word ptr ds:[ebx],ax
7FF7EBE7     90              nop
7FF7EBE8     90              nop
7FF7EBE9     90              nop
7FF7EBEA     90              nop
7FF7EBEB     90              nop
7FF7EBEC     66:48           dec ax
7FF7EBEE     90              nop
7FF7EBEF     90              nop
7FF7EBF0     90              nop
7FF7EBF1     90              nop
```

往下找`popad`指令

```
7FF7F085     68 00800000     push 8000
7FF7F08A     6A 00           push 0
7FF7F08C     50              push eax
7FF7F08D     FF95 43C64200   call dword ptr ss:[ebp+42C643]         ；这个是VirtualFree
7FF7F093     90              nop
7FF7F094     90              nop
7FF7F095     90              nop
7FF7F096     90              nop
7FF7F097     90              nop
```
