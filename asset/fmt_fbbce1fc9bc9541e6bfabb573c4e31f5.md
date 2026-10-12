
### 汇编代码段

```
0094A37C D1E8             shr eax,1
0094A37E 8BFA             mov edi,edx
0094A380 037C24 04        add edi,dword ptr ss:[esp+4]
0094A384 83C3 08          add ebx,8
0094A387 8BF0             mov esi,eax
0094A389 85F6             test esi,esi
0094A38B 76 38            jbe short 0094A3C5
0094A38D 66:8B13          mov dx,word ptr ds:[ebx]
0094A390 0FB7C2           movzx eax,dx
0094A393 C1E8 0C          shr eax,0C
0094A396 66:83E8 01       sub ax,1
0094A39A 72 23            jb short 0094A3BF
0094A39C 66:83E8 02       sub ax,2
0094A3A0 74 02            je short 0094A3A4
0094A3A2 EB 11            jmp short 0094A3B5
0094A3A4 66:81E2 FF0F     and dx,0FFF
0094A3A9 0FB7C2           movzx eax,dx
0094A3AC 03C7             add eax,edi
0094A3AE 8B1424           mov edx,dword ptr ss:[esp]
0094A3B1 0110             add dword ptr ds:[eax],edx
0094A3B3 EB 0A            jmp short 0094A3BF
0094A3B5 68 ECA39400      push 94A3EC                       ; ASCII "34 "
0094A3BA E8 5DACFFFF      call 0094501C
0094A3BF 83C3 02          add ebx,2
0094A3C2 4E               dec esi
0094A3C3 75 C8            jnz short 0094A38D
0094A3C5 8B13             mov edx,dword ptr ds:[ebx]
0094A3C7 85D2             test edx,edx
0094A3C9 75 A9            jnz short 0094A374
0094A3CB 83C4 14          add esp,14
0094A3CE 5D               pop ebp                            ; F4 到这儿，再看 ebx 值，bx 减去上面的 bx 就是 size 了
0094A3CF 5F               pop edi
0094A3D0 5E               pop esi
0094A3D1 5B               pop ebx
0094A3D2 C3               retn
```

最后得：`RVA:D000`，`Size:86C`

用`loadpe`编辑`3-1.dll`文件，填入新的重定位表地址和大小（在`Directories`里面），重建`PE`，OK了。。。呵呵

用`Peid`查，再也不是讨厌的的`ASProtect 2.1x SKE -> Alexey Solodovnikov`了，而是：

而是`Microsoft Visual C++ 6.0 DLL`了，呵呵。用`loaddll`运行`OK`，用主程序运行，好象有些问题，估计是主程序里面有检测`dll`文件`size`之类的，爆破的问题了。

### 再表 exe 文件
`exe`文件这步主要处理`code`段里面调用到壳里的函数或`jmp`

#### 1.exe
`call 011A0000`，就这一个，总体思路参照`shoooo`的"nspack3.5 主程序脱壳分析"，具体操作参考`kanxue`老大的"Asprotect SKE 2.2 的 Advanced Import protection 保护技术"，程序我是用的`kanxue`老大的现成的程序，先记录处理了哪些函数，后面再修复。代码我就不写了，看`kanxue`老大的文章，我的是里面说的`a`情况，出来之后的类型全部是`FF15`，即`call *`，没有`jmp *`的情况（我就说原来用搜索改`jmp`怎么不行。。）。

分两步进行的，1、`patch1`先记录哪些函数处理了；2、再次运行，到`OEP`后，再调用`patch2`把这些`call 011A0000`替换过来,

完成后，`dump`为`1-1.exe`

#### 2.exe
先在`OEP`处`dump`为`2-1.exe`。

对于`call 01020004;call 01040004;...`的情况，我搜索了一下，一共有`12`处，在`wak`没放文章之前，我是手工修复的，过程很简单，跟踪原程序的这些函数进去一个一个找出来。这种比较简单，如：

```
004010AC FF15 3C504000   call dword ptr ds:[40503C]             ; kernel32.SetEvent
004010B2 8B0D E47B4000   mov ecx,dword ptr ds:[407BE4]
004010B8 6A FF           push -1
004010BA 51               push ecx
004010BB E8 44EFC100      call 01020004                          ; 回车进去
004010C0 848B 15E47B40   test byte ptr ds:[ebx+407BE415],cl
004010C6 008B 35445040   add byte ptr ds:[ebx+40504435],cl
004010CC 0052 FF          add byte ptr ds:[edx-1],dl
004010CF D6               salc
004010D0 A1 E07B4000      mov eax,dword ptr ds:[407BE0]

01020004 F2:              prefix repne:
01020005 EB 01            jmp short 01020008                     ; 回车进去
01020007 - E9 FF0424E9    jmp EA26050B
0102000C F0:FFFE          ???                                    ; 未知命令
0102000F FF00             inc dword ptr ds:[eax]
01020011 0000             add byte ptr ds:[eax],al
01020013 0000             add byte ptr ds:[eax],al

01020008 FF0424           inc dword ptr ss:[esp]
0102000B - E9 F0FFFEFF    jmp 01010000                           ; 回车进去
01020010 0000             add byte ptr ds:[eax],al
01020012 0000             add byte ptr ds:[eax],al
01020014 0000             add byte ptr ds:[eax],al

01010000 8BFF             mov edi,edi
01010002 55               push ebp
01010003 8BEC             mov ebp,esp
01010005 6A 00            push 0
01010007 FF75 0C          push dword ptr ss:[ebp+C]
0101000A FF75 08          push dword ptr ss:[ebp+8]
0101000D 68 4225807C      push 7C802542                         ; ctrl+g 到这个地址去
01010012 68 5025807C      push kernel32.WaitForSingleObjectEx
01010017 C3               retn

7C802530 > 8BFF           mov edi,edi
7C802532 55               push ebp
7C802533 8BEC             mov ebp,esp
7C802535 6A 00            push 0
7C802537 FF75 0C          push dword ptr ss:[ebp+C]
7C80253A FF75 08          push dword ptr ss:[ebp+8]
7C80253D E8 0E000000      call kernel32.WaitForSingleObjectEx
7C802542 5D               pop ebp                                ; 到这儿
7C802543 C2 0800          retn 8
```

同上面比较，它把`7C802530`这个函数的前面几句给偷了，在`IAT`中查找`7C802530`，为：

```
00405000  7C802530  kernel32.WaitForSingleObject
```

将`2-1.exe`中本`call`改为`call dword ptr ds:[405000]` //注意：这个为`6`个字节，原来为`5`个字节，改的时候不要选择填充`nop`

依次改完所有的这种`call`，`aspr`处理这理很怪，全是`01020004,01040004,01060004...,01160004`这样的，居然有规律？

修改完后,保存。

后面`wak`放了一篇文章，可以用程序把这些函数记录下来，我试了一下，文章`1`里面的函数是没问题的，文章`2`里面的函数找不到地方，郁闷。这里理论上应该也可以用`kanxue`的方法记录这些函数，我没试过，不知道灵不灵。

### 四、修复 OEP 处的 stolen code

--只对`1.exe`和`2.exe`有用，两个基本一样，就只表一个了

我试过将它所需要的区段全部帖上去的方法，但区段太多，根本就帖不上去，`loadpe`友情提示我，“区段太多了，不能再帖。。。”

大侠说这是体力活，其实不单是体力活，也是脑力活，要去排除`aspr`的垃圾代码也很难啊。。。

#### 1.exe OEP:

```
011802F3 55                push ebp                                ; OEP
011802F4 EB 02             jmp short 011802F8
011802F6 CD20 1BEA8BEC    vxdjump EC8BEA1B
011802FC 6A FF             push -1
011802FE 50                push eax
011802FF 66:9C             pushfw
01180301 53                push ebx
01180302 81DB E8A90959    sbb ebx,5909A9E8
01180308 81F3 4060C549    xor ebx,49C56040
0118030E 8D5C24 1D        lea ebx,dword ptr ss:[esp+1D]
```

`ctrl+g`输入`00401000`到代码段，`ctrl+f`搜索`jmp 011802F3`，找到原`OEP`，修复之旅就从这儿开始。

```
004B0014 - E9 DA02CD00     jmp 011802F3                            ; 原 OEP
004B0019 D7                xlat byte ptr ds:[ebx+al]               ; 这些都是垃圾
004B001A 25 CFD1DD87      and eax,87DDD1CF。。。。。。。。。。。。。。。
004B001F 8C0F              mov word ptr ds:[edi],cs。。。。。。。。。。。。
004B0021 2A75 E2           sub dh,byte ptr ss:[ebp-1E]。。。。。。。。。
```

先用`IR`修复`dump`出的`1-1.exe`，填`OEP`为`004B0014`，转存为`1-1\_.exe`

用`OD`载入`1-1\_.exe`到`004B0014`处,另外`OD`再载入原程序`1.exe`到`011802F3`处，两边同时进行：

再`1-1\_.exe`里面填`1.exe`里面的有用代码。。。主要是去除垃圾代码。下面是去除垃圾后的部分`OEP`

```
004B0014 > 55               push ebp
004B0015 8BEC               mov ebp,esp
004B0017 6A FF              push -1
004B0019 68 003A4E00        push XXXXXXXX.004E3A00
004B001E 68 C2FD4A00        push <jmp.&msvcrt._except_handler3>
004B0023 64:A1 00000000     mov eax,dword ptr fs:[0]
004B0029 50                 push eax
004B002A 64:8925 0000000>   mov dword ptr fs:[0],esp
...
```

我发现我这边程序中垃圾代码主要有这几个地方，

1)、胡乱`jmp`
象`OEP`处，才`push ebp`就`jmp`了，这种情况，把代码去除`jmp`后复制到记事本再分析

2)、代码变形
如`push 404134`
它会这样处理：

```
00C20785 66:9C              pushfw
00C20787 57                 push edi
00C20788 13FB               adc edi,ebx
00C2078A F2:                prefix repne:
00C2078B EB 01              jmp short 00C2078E
00C2078E 8D7C51 07          lea edi,dword ptr ds:[ecx+edx*2+7]//N
00C20792 8D7C24 12          lea edi,dword ptr ss:[esp+12]//
00C20796 8D7F EE            lea edi,dword ptr ds:[edi-12]//
00C20799 8D7F 06            lea edi,dword ptr ds:[edi+6]//edi = esp + 6
00C2079C 56                 push esi
00C2079D 64:EB 02           jmp short 00C207A2
00C207A2 81CE CEE6ACC5      or esi,C5ACE6CE//N
00C207A8 1BF5               sbb esi,ebp//N
00C207AA 8DB40B 34414000   lea esi,dword ptr ds:[ebx+ecx+404134]//
```
