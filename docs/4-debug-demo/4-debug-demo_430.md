
# 光盘加密大师爆破--我的一篇垃圾破文

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-377.htm`](https://www.pediy.com/kssd/pediy09/pediy09-377.htm)

## 光盘加密大师 4.0.9 暴破

### 1. 找爆破点

`od` 载入后，用 `esp` 定律到 `oep`，然后用 `od` 插件查找相关字符串，找到“`user id`”相关字符串，双击来到代码处。

```asm
00486B08     BA E0714800    mov edx,cdloc_a.004871E0     ; ASCII "key"
00486B0D     8B45 FC        mov eax,dword ptr ss:[ebp-4]
00486B10     E8 0FF7FAFF    call cdloc_a.00436224
00486B15     8D4D C0        lea ecx,dword ptr ss:[ebp-40]
00486B18     BA EC714800    mov edx,cdloc_a.004871EC     ; ASCII "UserID"  //来到这里，f2，删除硬件断点，f9，在此断下
00486B1D     8B45 FC        mov eax,dword ptr ss:[ebp-4]            //往下 f8
00486B20     E8 FFF6FAFF    call cdloc_a.00436224
00486B25     8B55 C0        mov edx,dword ptr ss:[ebp-40]
00486B28     A1 E8584900    mov eax,dword ptr ds:[4958E8]
00486B2D     E8 F6D5F7FF    call cdloc_a.00404128
00486B32     8D4D F4        lea ecx,dword ptr ss:[ebp-C]
00486B35     BA FC714800    mov edx,cdloc_a.004871FC     ; ASCII "CDLOCK4020050529"
00486B3A     8B45 F8        mov eax,dword ptr ss:[ebp-8]
00486B3D     E8 A6ECFFFF    call cdloc_a.004857E8
00486B42     A1 945A4900    mov eax,dword ptr ds:[495A94]
00486B47     8B00           mov eax,dword ptr ds:[eax]
00486B49     8B55 F4        mov edx,dword ptr ss:[ebp-C]        //到这里
00486B4C     E8 9FD9F7FF    call cdloc_a.004044F0              //这是验证的 call
00486B51     75 26          jnz short cdloc_a.00486B79          //跳向未通过验证代码
00486B53     837D F8 00     cmp dword ptr ss:[ebp-8],0            //继续比较试炼码
00486B57     74 20          je short cdloc_a.00486B79           //跳向未通过验证代码
00486B59     837D F4 00     cmp dword ptr ss:[ebp-C],0            //再比较
00486B5D     74 1A          je short cdloc_a.00486B79           //还是跳死
00486B5F     A1 FC564900    mov eax,dword ptr ds:[4956FC]        //验证通过会到这里
00486B64     C600 01        mov byte ptr ds:[eax],1               //往下
00486B67     A1 BC564900    mov eax,dword ptr ds:[4956BC]
00486B6C     C600 00        mov byte ptr ds:[eax],0
00486B6F     E8 48D0F7FF    call cdloc_a.00403BBC
00486B74     E9 72050000    jmp cdloc_a.004870EB
```

在经过 `486b51`---`486b5d` 的三个跳转时，通过改变标志位，让程序一直来到 `486b5f` 处往下走就注册成功了。

那么可以直接将：

```asm
00486B51     75 26          jnz short cdloc_a.00486B79          //跳向未通过验证代码
```

这一句改为：

```asm
00486B51     EB 0C          jmp short cdloc_a.00486B5F          //直接跳向通过验证代码
```

用内存补丁改可以用这句代码实现：`mov word prt ds:[00486b51],0ceb` （后面会用到此句）。

### 2. 解除自校验

由于文件脱壳后运行一闪而过，考虑是有自校验，分析一下，一般会对文件取校验值与原文件的比较，那样就会调用到打开文件的函数 `CreateFileA`，那么就用此断点来试一下。

重新运行后，还是 `esp` 定律到 `oep` 处，命令栏下断点：`bpx CreateFileA`。

在 `Intermodular calls` 中可以看到：

```asm
00408742   call cdloc_a.004063CC  kernel32.CreateFileA      //第一次调用
00408769   call cdloc_a.004063CC  kernel32.CreateFileA
00486311   call cdloc_a.004063CC  kernel32.CreateFileA       //第二次
00486559   call cdloc_a.004063CC  kernel32.CreateFileA
00486577   call cdloc_a.004063CC  kernel32.CreateFileA
```

共有 5 处调用这个函数，`f9` 运行，第二次调用 `CreateFileA` 在 `00486311` 处经过后，命令行，下 `d ebx` 命令，发现在这个地址的上面 `0013fc44` 处有一字符串很刺眼：

```text
0013FC44  61 38 35 31 34 61 36 32        a8514a62
0013FC4C  30 35 37 64 32 39 34 35        057d2945
0013FC54  37 61 36 34 39 30 35 33        7a649053
0013FC5C  30 65 31 65 33 32 32 35        0e1e3225
```

于是就用 `DAMN Hash Calculator` 取 `cdlock` 的原文件计算了下，发现这个原文件的 `MD5` 值为：`A8514A62057D29457A6490530E1E3225`。

就是这个字符串转换为了小写。

终于找到是这个 `md5` 自校验在作怪，`f8` 往下走，经过两个 `closehandle` 函数后，一直返回到 `0048690D`：

```asm
00486908     E8 BFF9FFFF    call cdloc_a.004862CC
0048690D     8D45 B8        lea eax,dword ptr ss:[ebp-48]   //到这里
00486910     8D55 F4        lea edx,dword ptr ss:[ebp-C]
00486913     E8 10FBFFFF    call cdloc_a.00486428
```

```asm
00486918     8B45 F8        mov eax,dword ptr ss:[ebp-8]   //这里取文件原来 md5 校验值到 eax
//Stack ss:[0013FC68]=00E037AC, (ASCII "a8514a62057d29457a6490530e1e3225")
eax=00000000
```

```asm
0048691B     8B55 F4        mov edx,dword ptr ss:[ebp-C]  //这里取所运行文件的 md5 校验值到 edx
//Stack ss:[0013FC64]=00E0A220, (ASCII "a8514a62057d29457a6490530e1e3225")
edx=0013FC10
```

```asm
0048691E     E8 CDDFBF7FF    call cdloc_a.004044F0        //用这个 call 来判断是否一致
00486923     75 04          jnz short cdloc_a.00486929   //不一致跳走，退出
00486925     B3 01          mov bl,1                       //验证通过，bl 置 1
00486927     EB 02          jmp short cdloc_a.0048692B
```

因为这里并未对文件修改所以两个值一样，我用脱壳文件做这个 `md5` 校验值 `0048691B` 处为：

```text
Stack ss:[0013FC64]=00E0A220, (ASCII "d6c64732f76190daa0ec9c8cb61fdfc7")
edx=0013FC10
```

那么当然是让 `eax` 和 `edx` 中的校验值相等就好，我这里把 `00486918` 处：

```asm
00486918     8B45 F8      mov eax,dword ptr ss:[ebp-8]
```

改为：

```asm
00486918     8B45 F4      mov eax,dword ptr ss:[ebp-C]
```

那么自己和自己比，任你怎么比都一样，因为是同一个文件。

用代码改可以是：`mov dword ptr ds:[486918],8BF4458B`。

当然到这里我们的程序已经完美暴破。但却不想改变文件大小，于是做个内存补丁。

### 3. 打内存补丁

因为文件是加壳的，代码被压缩，在哪里补丁是个问题。因为要修改 `486B51` 处代码，所以在此下内存写入断，`f9` 中断在：

```asm
00508179     F3:A5          rep movs dword ptr es:[edi],dwor>  //断在这里
0050817B     8BC8           mov ecx,eax
0050817D     83E1 03        and ecx,3
00508180     F3:A4          rep movs byte ptr es:[edi],byte >
00508182     5E             pop esi
00508183     68 00800000    push 8000
00508188     6A 00          push 0
0050818A     FFB5 F43F4400  push dword ptr ss:[ebp+443FF4]
00508190     FF95 00404400  call dword ptr ss:[ebp+444000]  //这里 kernel32.VirtualFree
00508196     83C6 08        add esi,8
00508199     833E 00        cmp dword ptr ds:[esi],0        //这里比较是否解压完毕
0050819C   ^ 0F85 26FFFFFF  jnz cdloc_a.005080C8           //跳回继续解压
```

```asm
005081A2     68 00800000    push 8000                        //f2，f9 断下，解压完毕，在此修改正合适
005081A7     6A 00          push 0
005081A9     FFB5 F83F4400  push dword ptr ss:[ebp+443FF8]
005081AF     FF95 00404400  call dword ptr ss:[ebp+444000]
```

从这里往下拖，在后面区段的间隙里找一处空间，添上修改代码。这里我找的是 `0050A560`：

```asm
0050A560     66:C705 516B48>mov word ptr ds:[486B51],0CEB
0050A569     C705 18694800 >mov dword ptr ds:[486918],8BF4458B
0050A573     68 00800000    push 8000
0050A578     68 A7815000    push cdlock.005081A7
0050A57D     C3             retn
```

填上以上代码。

二进制数据：
`66 C7 05 51 6B 48 00 EB 0C C7 05 18 69 48 00 8B 45 F4 8B 68 00 80 00 00 68 A7 81 50 00 C3`

将 `005081A2` 处的 `push 8000` 一句改为 `jmp 0050A560`（跳向我们添加的代码处）。

将所有修改复制到可执行文件另存一下，发现不能运行，于是再载入调试。跟踪发现 `005081A2 push 8000` 在程序运行前也是压缩的，即使修改为 `jmp 0050A560`，那么壳代码解压完后也会把 `jmp 0050A560` 重新覆盖为 `push 8000`。

那么只能加一层 `smc` 了，重新下内存写入断点在 `005081A2`，`f9`，中断在：

```asm
00508533     81B40E 5EF28CD>xor dword ptr ds:[esi+ecx+D18CF25E],8C7B780F     //这里循环解压
0050853E     0FBFEE         movsx ebp,si
00508541     81F7 69C4D54D  xor edi,4DD5C469
00508547     42             inc edx
00508548     E9 1C000000    jmp cdloc_a.00508569
00508569     66:8BD9        mov bx,cx
0050856C     81EE 04000000  sub esi,4
00508572     B3 D6          mov bl,0D6
00508574     0FBFCA         movsx ecx,dx
00508577     81FA 779AF635  cmp edx,35F69A77                         //这里比较是否解压完毕
0050857D   ^ 0F85 A0FFFFFF  jnz cdloc_a.00508523                     //跳回继续解压
```

```asm
00508583     66:8BDF        mov bx,di                     //解压完执行这里，当然在此修改了
00508586     0FBFC9         movsx ecx,cx
```



`00508589 0FBFDE movsx ebx,si`

`0050858C ^ E9 AAFEFFFF jmp cdloc__.0050843B`

将`00508583`处改为`jmp cdlock.0050A530`，`0050A530`自然是我们找的第二处添加代码处

如下：

`00508583 E9 A81F0000 jmp cdlock.0050A530`

`00508588 90 nop`

添加如下代码：

`0050A530 C705 A2815000 >mov dword ptr ds:[5081A2],23B9E9 ; //意思是将 005081A2 push 8000 一句改为 jmp 0050A560`

`0050A53A 66:8BDF mov bx,di //因为占用了原来代码位置，所以在这里将被占用的补齐`

`0050A53D 0FBFC9 movsx ecx,cx`

`0050A540 68 89855000 push cdlock.00508589`

`0050A545 C3 retn //返回到 00508589`

二进制代码如下：

```
C7 05 A2 81 50 00 E9 B9 23 00 66 8B DF 0F BF C9 68 89 85 50 00 C3
```

最后复制所有修改到可执行文件，另存为。。。。

到此全部完成。

`2007-2-1`

`adios`

感谢阿铁大哥的支持

`4dd466d0e1e0cc123afbe504da2462ca`
