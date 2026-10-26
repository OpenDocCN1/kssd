
四、Pre-Dip：用自己的名字注册

取消以前的所有断点，现在设置Ollydbg忽略除了“内存访问异常”之外的所有其他异常选项。

`Shift+F9`运行，注意看堆栈，当第2次在堆栈中看见“`8wh3JAAQjOI=`”硬盘指纹时，可以处理了。

```
00B90B36     893B                mov dword ptr ds:[ebx],edi
//异常，第2次看见硬盘指纹
0012FF20   0012FF2C   指针到下一个 SEH 记录
0012FF24   00B90CEF   SE 句柄
0012FF28   40E2AEE6
0012FF2C   0012FF80   指针到下一个 SEH 记录
0012FF30   00B9114C   SE 句柄
0012FF34   0012FF78
0012FF38   00B70000
0012FF3C   00B40000   ASCII "MZ?
0012FF40   00B9051C   ASCII "USVW′&?
0012FF44   00000000
0012FF48   00BAE59C   ASCII "8wh3JAAQjOI="//硬盘指纹
0012FF4C   00B8F0F6   返回到 00B8F0F6 来自 00B81868
```

`Alt+M`打开内存查看窗口，在`00401000`段下 内存访问断点，`Shift+F9`通过异常，断下

```
00535838     55                  push ebp
//中断在这里
00535839     8BEC                mov ebp,esp
0053583B     8B45 08             mov eax,dword ptr ss:[ebp+8]
0053583E     A3 0CCE5400         mov dword ptr ds:[54CE0C],eax
//保存注册名地址
00535843     5D                  pop ebp
00535844     C2 0400             retn 4
```

过了`0053583E`之后，`[54CE0C]`处应该是保存的注册用户名，找一段空地，在`0056AB00`处写入：`fly  [CUG]`，然后把`[54CE0C]`处的值改为`0056AB00`。OK，这个Pre-Dip处理完毕。

五、以壳解壳――ASProtect V1.3壳的Stolen Code简便解决方案

取消以前的所有断点。继续`Shift+F9` 运行，直至中断在最后一次典型异常。

```
00B8F3F3     893B                mov dword ptr ds:[ebx],edi
//ASProtect V1.31壳的最后一次异常
00B8F3F5     3085 5798C864       xor byte ptr ss:[ebp+64C89857],al
00B8F3FB     1B3F                sbb edi,dword ptr ds:[edi]
00B8F3FD     05 7B462835         add eax,3528467B
00B8F402     67:64:8F06 0000     pop dword ptr fs:[0]
00B8F408     26:EB 02            jmp short 00B8F40D
```

提供一个简便的方法快速达到使用“以壳解壳”的时机！

在最后一次异常时，`Ctrl+F`在“当前位置下”搜索命令：

`mov esp,dword ptr ss:[ebp-8]`

OK，找到在`00B8F65E` 处！下断，`Shift+F9`运行，中断

```
00B8F64A     E8 8B45FC8B         call 8CB53BDA
00B8F64F     0085 C0752B8B       add byte ptr ss:[ebp+8B2B75C0],al
00B8F655     0D 904BB900         or eax,0B94B90
00B8F65A     85C9                test ecx,ecx
00B8F65C     74 0B               je short 00B8F669
00B8F65E     8B65 F8             mov esp,dword ptr ss:[ebp-8]
//中断后取消所有断点
00B8F661     FF35 904BB900       push dword ptr ds:[B94B90]
00B8F667     C3                  retn
//返回00DE0303
00DE0303     55                  push ebp
//这里开始处理Stolen Code ★
//在这儿用PETools完全DUMP这个进程
00DE0304     EB 02               jmp short 00DE0308
00DE0308     8D6C5E 4A           lea ebp,dword ptr ds:[esi+ebx*2+4A]
00DE0310     8D6C05 B6           lea ebp,dword ptr ss:[ebp+eax-4A]
00DE0318     2BE8                sub ebp,eax
00DE031A     2BEB                sub ebp,ebx
```

六、Dump、修复输入表

如果仅仅脱壳出上面的壳代码部分是不行的，如下面的访问地址就丢失了：

```
02720167     FFD7                call edi  ;00B8BC98
00409C6C     68 10D6BA00         push 0BAD610
00409C71     C3                  retn
00DE1C99     E8 62E39301         call 02720000
00B8BC53     FF60 24             jmp dword ptr ds:[eax+24]; 02730000
```

所以区域脱壳出以下部分壳代码：

`00B70000-00B9B000.dmp`

`00BA8000-00BB0000.dmp`

`00DE0000-00DE4000.dmp`

`02720000-02721000.dmp`

`02730000-02731000.dmp`

“组装”一下`dumped.exe`，从磁盘载入刚才部分脱壳的五部分。

“组装”一下`dumped.exe`，从磁盘载入刚才部分脱壳的3部分。

减去基址后分别修改其虚拟地址为`00770000`、`007A8000`、`009E0000`、`02320000`、`02330000`

只保留PETools的“验证PE文件”选项，重建PE。

运行`ImportREC`，填入`RVA=0016A000`、`Size=00000850`

获取输入表，会发现有2个无效指针

`0016A078  ?  0000  00B846B8`

`0016A1E0  ?  0000  00B846B8`

通过`AL=3`的分析，我们知道应该把这2个函数修复为：`GetProcAddress`

修改`OEP=009E0303`，`FixDump`！

七、暗桩修复

运行后会发现有异常，其实还是壳造成的问题。

1、004021FA处异常

```
00402184     55                  push ebp
00402185     8BEC                mov ebp,esp
00402187     83C4 F8             add esp,-8
0040218A     53                  push ebx
0040218B     56                  push esi
0040218C     57                  push edi
0040218D     8BD8                mov ebx,eax
0040218F     803D 30E45400 00    cmp byte ptr ds:[54E430],0
//把54E430处的01修改为00，让其重新申请内存
00402196     75 13               jnz short dumped\_.004021AB
00402198     E8 13F9FFFF         call dumped\_.00401AB0
0040219D     84C0                test al,al
0040219F     75 0A               jnz short dumped\_.004021AB
004021A1     33C0                xor eax,eax
004021A3     8945 FC             mov dword ptr ss:[ebp-4],eax
004021A6     E9 54010000         jmp dumped\_.004022FF
004021AB     33C9                xor ecx,ecx
004021AD     55                  push ebp
004021AE     68 F8224000         push dumped\_.004022F8
004021B3     64:FF31             push dword ptr fs:[ecx]
004021B6     64:8921             mov dword ptr fs:[ecx],esp
004021B9     803D 49E05400 00    cmp byte ptr ds:[54E049],0
004021C0     74 0A               je short dumped\_.004021CC
004021C2     68 38E45400         push dumped\_.0054E438
004021C7     E8 40F2FFFF         call <jmp.&kernel32.EnterCriticalSection>
004021CC     83C3 07             add ebx,7
004021CF     83E3 FC             and ebx,FFFFFFFC
004021D2     83FB 0C             cmp ebx,0C
004021D5     7D 05               jge short dumped\_.004021DC
004021D7     BB 0C000000         mov ebx,0C
004021DC     81FB 00100000       cmp ebx,1000
004021E2     0F8F 93000000       jg dumped\_.0040227B
004021E8     8BC3                mov eax,ebx
004021EA     85C0                test eax,eax
004021EC     79 03               jns short dumped\_.004021F1
004021EE     83C0 03             add eax,3
004021F1     C1F8 02             sar eax,2
004021F4     8B15 90E45400       mov edx,dword ptr ds:[54E490]
004021FA     8B5482 F4           mov edx,dword ptr ds:[edx+eax*4-C]
//异常
004021FE     85D2                test edx,edx
00402200     74 79               je short dumped\_.0040227B
```

2、0040146C处异常

```
00401426     BE 54E45400         mov esi,dumped\_.0054E454
0040142B     833E 00             cmp dword ptr ds:[esi],0
//把[0054E454]处的00153FF4修改为00  ★
0040142E     75 3A               jnz short dumped\_.0040146A
00401430     68 44060000         push 644
00401435     6A 00               push 0
00401437     E8 A8FFFFFF         call <jmp.&kernel32.LocalAlloc>
0040143C     8BC8                mov ecx,eax
0040143E     85C9                test ecx,ecx
00401440     75 05               jnz short dumped\_.00401447
00401442     33C0                xor eax,eax
00401444     5E                  pop esi
00401445     5B                  pop ebx
00401446     C3                  retn
00401447     A1 50E45400         mov eax,dword ptr ds:[54E450]
0040144C     8901                mov dword ptr ds:[ecx],eax
0040144E     890D 50E45400       mov dword ptr ds:[54E450],ecx
00401454     33D2                xor edx,edx
00401456     8BC2                mov eax,edx
00401458     03C0                add eax,eax
0040145A     8D44C1 04           lea eax,dword ptr ds:[ecx+eax*8+4]
0040145E     8B1E                mov ebx,dword ptr ds:[esi]
00401460     8918                mov dword ptr ds:[eax],ebx
```

```
00B83EAF     8D45 FC             lea eax,dword ptr ss:[ebp-4]
00B83EB2     8B4D 08             mov ecx,dword ptr ss:[ebp+8]
00B83EB5     BA 043FB800         mov edx,0B83F04  ; ASCII "Error: "
00B83EBA     E8 81F9FEFF         call 00B73840
00B83EBF     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00B83EC2     E8 F1FAFEFF         call 00B739B8
00B83EC7     50                  push eax
00B83EC8     68 0C3FB800         push 0B83F0C     ; ASCII "Protection Error"
00B83ECD     E8 4E000000         call 00B83F20
//Game Over
00B83ED2     33C0                xor eax,eax
00B83ED4     5A                  pop edx
00B83ED5     59                  pop ecx
00B83ED6     59                  pop ecx
00B83ED7     64:8910             mov dword ptr fs:[eax],edx
00B83EDA     68 F73EB800         push 0B83EF7
00B83EDF     8D45 FC             lea eax,dword ptr ss:[ebp-4]
00B83EE2     E8 B1F6FEFF         call 00B73598
00B83EE7     8D45 08             lea eax,dword ptr ss:[ebp+8]
00B83EEA     E8 A9F6FEFF         call 00B73598
00B83EEF     C3                  retn
```
