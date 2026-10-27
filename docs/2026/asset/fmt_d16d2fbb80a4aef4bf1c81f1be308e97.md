
```
00405CAA     C0E1 05            shl cl,5
00405CAD     E9 210B0000        jmp CV\_TES~3.004067D3

// eax 不变
// edx 不变
// ecx->40
//--------------------------------------
//004067D3
004067D3     80C1 F4            add cl,0F4
004067D6     00C8               add al,cl
004067D8     59                 pop ecx
004067D9     5A                 pop edx
004067DA   ^ E9 B8FFFFFF        jmp CV\_TES~3.00406797

// 堆栈已经被恢复
// eax -> 0x9A
// edx -> 变为最初值 0x7C92E514
// ecx -> 变为最初值 0x00000001
//--------------------------------------
// 修改ebx值
//00406797
00406797     80C3 4C            add bl,4C
0040679A     80C3 BE            add bl,0BE
0040679D     00C3               add bl,al
0040679F     80EB BE            sub bl,0BE
004067A2   ^ E9 E8FFFFFF        jmp CV\_TES~3.0040678F

// eax -> 0x9A
// ebx -> 00407000
//--------------------------------------
// 0040678F
0040678F     51                 push ecx
00406790     B5 CC              mov ch,0CC
00406792   ^ E9 F2DEFFFF        jmp CV\_TES~3.00404689

// eax -> 0x9A
// ebx -> 00407000
// ecx -> 0000CC01
//--------------------------------------
00404689
00404689     D0ED               shr ch,1
0040468B     80E5 95            and ch,95
0040468E     80F5 48            xor ch,48
00404691     28EB               sub bl,ch
00404693     59                 pop ecx      ; 恢复了ecx为0x00000001
00404694     0FB6C0             movzx eax,al ; eax -> 0x0000009A
00404697   ^\FF2487             jmp dword ptr ds:[edi+eax\*4]  ; 调向了真正的解析例程中

// ebx -> 0x004070b4
```

`dword ptr ds:[edi+eax\*4]` 相当于 `ds:[00403468]=00403FFE (CV\_TES~3.00403FFE)`

`403468`是`vm_dispatch`的数据范围，里面存放的时候`00403FFE`解析例程

到此时我们已经知道，这一路跟踪下来的代码仅为了计算一个`eax`的值，用来定位解析例程的索引。

但`al`从`0x3F`如何变成`0x9A` 的？只要查看这一路跟下了起变化的过程即可。

`al = vm_data[0 ~ n] + 0x81 + 0x90 + bl - 0x90 - 0x81 ^ 0xD2  - 0x34 - 0xF1  + 0x34`

`al = vm_data[0 ~ n] + bl ^ 0xD2 - 0xF1`

`al = (0x3f + 0x1a ) ^ 0xD2 - 0xF1`

`al = 0x9A`

所以固定的计算就是

`index = (al + bl ) ^ 0xD2 - 0xF1 ;`

同时更新`ebx`的值，

`bl = bl + al`

还是用这个程序加壳生成另外一个时，会发现和前面过程中执行的指令一摸一样，仅是利用大量的`jmp`打乱原有的指令而已,也就是说指令的运算及执行条数是固定的，然后用`jmp`来打乱。只有设置不同的加密等级时才变得不一样。

```
x0:                                x0:
  xxx1                               xxx1
  xxx2                               xxx2
  xxx3            混乱为           jmp x1
jmp x1         --------------->
x1:                                x1:
  xxx4                               xxx3
  xxx5                             jmp x2
  xxx6
jmp x2                             x2:
...                                  xxx4
                                     xxx5
                                     xxx6
                                     jmp x3
                                     ...
```

```
// 继续看处理例程，还是那套路子，不过这次计算的al是用来获得vm\_context中要操作的一个寄存器的索引标志
// 简化后 al = al - bl - 0x2b - 0x38

00403FFE     AC                 lods byte ptr ds:[esi] ；取vm\_data 下一个数据
00403FFF     2C 36              sub al,36
00404001     E9 F81A0000        jmp CV\_TES~3.00405AFE

00405AFE     28D8               sub al,bl
00405B00     52                 push edx
00405B01     66:53              push bx
00405B03   ^ E9 71F9FFFF        jmp CV\_TES~3.00405479

00405479     B7 AB              mov bh,0AB
0040547B     88FE               mov dh,bh
0040547D     66:5B              pop bx
0040547F     C0EE 06            shr dh,6
00405482     FEC6               inc dh
00405484     80F6 35            xor dh,35
00405487     00F0               add al,dh
00405489     E9 0B190000        jmp CV\_TES~3.00406D99

00406D99     5A                 pop edx
00406D9A   ^ E9 9CD0FFFF        jmp CV\_TES~3.00403E3B

00403E3B     51                 push ecx
00403E3C     B5 C2              mov ch,0C2
00403E3E     F6DD               neg ch
00403E40     FECD               dec ch
00403E42     F6D5               not ch
00403E44     E9 8C0E0000        jmp CV\_TES~3.00404CD5

00404CD5     80E5 6A            and ch,6A
00404CD8   ^ E9 47F0FFFF        jmp CV\_TES~3.00403D24

00403D24     F6D5               not ch
00403D26     80C5 6E            add ch,6E
00403D29     28E8               sub al,ch
00403D2B     8B0C24             mov ecx,dword ptr ss:[esp]
00403D2E     83C4 04            add esp,4
00403D31     53                 push ebx
00403D32     B7 A7              mov bh,0A7
00403D34     50                 push eax
00403D35     B4 B3              mov ah,0B3
00403D37     80C4 26            add ah,26
00403D3A     80EC 91            sub ah,91
00403D3D     E9 3F1F0000        jmp CV\_TES~3.00405C81

00405C81     80C7 D1            add bh,0D1
00405C84     28E7               sub bh,ah
00405C86   ^ E9 27E4FFFF        jmp CV\_TES~3.004040B2

004040B2     80EF D1            sub bh,0D1
004040B5     58                 pop eax
004040B6     F6D7               not bh
004040B8     E9 6D2E0000        jmp CV\_TES~3.00406F2A

00406F2A     C0EF 05            shr bh,5
00406F2D     F6DF               neg bh
00406F2F     C0EF 05            shr bh,5
00406F32     80C7 31            add bh,31
00406F35     28F8               sub al,bh
00406F37     5B                 pop ebx
00406F38     51                 push ecx
00406F39     B1 F7              mov cl,0F7
00406F3B   ^ E9 26D9FFFF        jmp CV\_TES~3.00404866

00404866     D0E1               shl cl,1
00404868   ^ E9 5BFAFFFF        jmp CV\_TES~3.004042C8

004042C8     80C1 4F            add cl,4F
004042CB     80E1 E7            and cl,0E7
004042CE     E9 40140000        jmp CV\_TES~3.00405713

00405713     80E1 BD            and cl,0BD
00405716     80E9 2D            sub cl,2D
00405719     00CB               add bl,cl
0040571B     59                 pop ecx
0040571C     80EB 68            sub bl,68
0040571F   ^ E9 F2E2FFFF        jmp CV\_TES~3.00403A16

00403A16     00C3               add bl,al
00403A18     E9 56000000        jmp CV\_TES~3.00403A73

00403A73     80C3 68            add bl,68
00403A76     80EB F8            sub bl,0F8
00403A79     0FB6C0             movzx eax,al
// eax == 7
00403A7C     8D0487             lea eax,dword ptr ds:[edi+eax\*4]
// 上面执行完eax == 0040321C,是vm\_context 结构中的偏移1c的位置，经过分析知道是eflag标志
00403A7F     68 7E320000        push 327E  ; 压入一个临时数字，后面要改写这个数字
00403A84     E9 270D0000        jmp CV\_TES~3.004047B0

004047B0     890424             mov dword ptr ss:[esp],eax ; 修改这个数字
// 这相当于把虚拟机的eflag标志的地址进栈
// ebx - > 004070BB
// esi - > 0040701C -- 此时指向vm\_data+3处
004047B3   ^ E9 9EEDFFFF        jmp CV\_TES~3.00403556    ;再次跳回vm起始位置
```

相当于执行：`push eflag`

至此我们已经知道刚才的那个`00403FFE`例程是`cv`虚拟机压入`eflag`标志进栈操作,但该例程并不是就单纯完成`push eflag` 因为`eflag`的偏移是根据`vm_data`中的数据计算而来的，所以这个例程应该是`push imm32` 功能，`imm32`是根据传入的参数来确定的。

继续看下面的例程，`al = 0xe1` ,计算后得到的例程地址为`5D`，`[edi + eax \* 4] = 0x403f8f`

分析可知`0x403f8f`的解析例程目的是执行`pop edx`，执行过程如下，`\*`表示垃圾指令

```
00403F8F     8B1424             mov edx,dword ptr ss:[esp] ; CV\_TES~3.0040321C
00403F92     55                 push ebp      \*
00406EC7     89E5               mov ebp,esp   \*
00406EC9     81C5 04000000      add ebp,4     \*
00406C51     51                 push ecx      \*
00406C52     B9 04000000        mov ecx,4     \*
00406BEA     01CD               add ebp,ecx   \*
00406BEC     59                 pop ecx       \*
00406BED     872C24             xchg dword ptr ss:[esp],ebp \*
00406BF0     5C                 pop esp       \*
```

执行；`pop edx`

大家可以在`00404697  jmp dword ptr ds:[edi+eax\*4]`这个地方大家可以下断点，因为每一个`cv`的dispatch执行完毕后，都要调向这里去执行下一个dispatch

```
// 继续获得解析例程地址为ds:[00403324] = 0x00405F70

00405F70     8F02               pop dword ptr ds:[edx]

00405F72   ^ E9 DFD5FFFF        jmp CV\_TES~3.00403556  --- 跳回vm\_start

// 也就是0x00405F70例程
```

执行 ：`pop [edx]`

```
// 下一条解析指令 ds:[00403468]= 0x00403FFE ，这是已知的，上面分析过

执行 ：push vm\_off\_04

// 下一条解析指令 ds:[00403374]=00403F8F ，这是已知的，上面分析过

执行 : pop edx

// 下一条解析指令 ds:[00403324]=00405F70 ，这是已知的，上面分析过

执行： pop [edx]
```

此时堆栈弹出来的值原`EDI`的值，所以可以推断出`vm_off_04`是`cv`虚拟机的存放`edi`的地方

```
//--------------------------------------

// 下一条解析指令 ds:[00403468]=00403FFE ,这是已知的，上面分析过

执行：push vm\_off\_14
```
