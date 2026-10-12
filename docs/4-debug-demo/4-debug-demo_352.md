
# Windows Commander 4.52

> 原文：[`www.pediy.com/kssd/pediy03/forum724.htm`](https://www.pediy.com/kssd/pediy03/forum724.htm)

## 注册你的 Windows Commander 4.52

`wincmd4.52`发布有一段时间了，我一直忙的要命，没空去搞（实际是陪朋友们去打`sc`，如果不打的话，即时战略就会变成即时真人快打，-\_-所以~~~~~明白~~~？嘿嘿我的`sc`水平倒是大有长进哦^^），今天花了几个小时搞定它;-)。

现在进入正题，如何注册你的`wincmd452`？先进行准备工作：`wincmd`是`aspack`压过的，拿`trw2000 ver1.23`脱壳吧。用`trw`载入`wincmd32.exe`，下`PNEWSEC`，晕~~~~~没反应~~~~~，看来还是偷不得懒呀^Q^。用冲击波找到`wincmd`的入口点`543530`，用`trw`载入`wincmd32.exe`，下`g 543530`，下`suspend`，用`PROCDUMP`找到`wincmd`进程，右击之，选`dump(full)`即可,这样脱壳不完全，能改主程序就行，将就用了^^。好了，看看`wincmd`脱壳后还能怎么折腾:)。大致分析了一下，它的注册机制同`wincmd4.0-4.51`一样，根本没有变化，hehe....，这下好办了;-)。

### 准备工作完成，开始正式 crack

一共需要改`8`处。`wincmd`是`keyfile`保护的那种软件，没有`keyfile`一样改。先用`trw2000`载入`wincmd32.exe`，点`load`,停下来后下`bpx readfile`，按`f5`执行，被拦到，下`bd *`，按`f12`直到`wincmd`的领空。当走到`4d6b1c call 4d8cd0`时停下来，记下这个地址，按`f5`，接着退出`trw2000`，换乘`softice`。我为什么换成`softice`，原因很简单，调试的话用`s-ice`是最舒服的了，脱壳的话用`trw`或天意是最方便的，所以`choose s-ice`没商量。由于不是完全脱壳，不能用`s-ice`的`loader`装载，所以~~~按`ctrl+d`进入`softice`下`bpx hmemcpy`,按`f5`返回，执行`wincmd32.exe`,被拦到，按数次`f12`，进入`wincmd`的领空，下`bpx 4d6b1c`,按`f5`又被拦到，按`f8`接着按`f10`来到如1.所示的程序:

**1.**

```
4d8db1 e86aa7f2ff    call 403520
4d8db6 83b80c00000000 cmp dword ptr [eax+0c],00
4d8dbd 7521 ->9090    jnz 4d8de0 ->nop
4d8dbf 8d85e2feffff  lea eax,[ebp-011e]
```

`wincmd`是靠比较内存判断是否注册的，所用的寄存器是`ebx`,需要保持为`1`,走到`4d8de0`,跳下去的话,会清掉`ebx`,所以不能让他跳。按`f10`继续，接着他会比较你的`keyfile`是否为`128`字节，若是的话，提示这是旧版，让你更新。不管他，按`f10`走到如2.所示的程序：

**2.**

```
4d8fd6 b966000000 mov eax,00000066
4d8fdb babca42000 mov edx,0020a4bc
4d8fe0 e84ffbffff call 4d8b34
4d8fe5 33c9       xor ecx,ecx
4d8fe7 b201       mov dl,01
```

`4d8b34`这个`call`有问题，跟进去如A.所示程序，把它改为B.所示的程序。其中`b5e7c9f1c4a7b9ed20`或`7a6f6d6269657973`是我注册的名字，你可以改成别的名字，其余的部分按B.所示的改。

`004cde88->004cdf00`

| A. | B. |
| :--- | :--- |
| `017f:004d8b34 55 push ebp` | `017f:004d8b34 0517000000 add eax,00000017` |
| `017f:004d8b35 8bec mov ebp,esp` | `017f:004d8b39 c60000 mov byte ptr [eax],00` |
| `017f:004d8b37 83c4f8 add esp,-08` | `017f:004d8b3c 0503000000 add eax,00000003` |
| `017f:004d8b3a 53 push ebx` | `017f:004d8b43 c600b5 mov byte ptr [eax],b5` |
| `017f:004d8b3b 56 push esi` | `017f:004d8b46 40 inc eax` |
| `017f:004d8b3c 57 push edi` | `017f:004d8b47 c600e7 mov byte ptr [eax],e7` |
| `017f:004d8b3d 8bf9 mov edi,ecx` | `017f:004d8b4a 40 inc eax` |
| `017f:004d8b3f 8945fc mov [ebp-04],eax` | `017f:004d8b4b c600c9 mov byte ptr [eax],c9` |
| `.` | `017f:004d8b4e 40 inc eax` |
| `.` | `017f:004d8b4f c600f1 mov byte ptr [eax],f1` |
| `.` | `017f:004d8b52 40 inc eax` |
| `.` | `017f:004d8b53 c600c4 mov byte ptr [eax],c4` |
| `.` | `017f:004d8b56 40 inc eax` |
| `.` | `017f:004d8b57 c600a7 mov byte ptr [eax],a7` |
| `.` | `017f:004d8b5a 40 inc eax` |
| `.` | `017f:004d8b5b c600b9 mov byte ptr [eax],b9` |
| `.` | `017f:004d8b5e 40 inc eax` |
| `.` | `017f:004d8b5f c600ed mov byte ptr [eax],ed` |
| `.` | `017f:004d8b62 40 inc eax` |
| `017f:004d8ba2 ff4df8 dec dword ptr [ebp-08]` | `017f:004d8b63 c60020 mov byte ptr [eax],20` |
| `017f:004d8ba5 75a8 jnz 004d8b4f` | `017f:004d8b66 40 inc eax` |
| `017f:004d8ba7 5f pop edi` | `.` |
| `017f:004d8ba8 5e pop esi` | `.` |
| `017f:004d8ba9 5b pop ebx` | `.` |
| `017f:004d8baa 59 pop ecx` | `.` |
| `017f:004d8bab 59 pop ecx` | `017f:004d8ba9 c60000 mov byte ptr [eax],00` |
| `017f:004d8bac 5d pop ebp` | `017f:004d8bac 90 nop` |
| `017f:004d8bad c3 ret` | `017f:004d8bad c3 ret` |

接着按`F10`来到如3.所示的程序，强制他跳过去，不跳的话会清掉`ebx`的。

**3.**

```
017f:004d903b e8e499f2ff    call 00402a24
017f:004d9040 0f84ae000000 ->e9af000000 jz 004d90f4 ->jmp 004d90f4
017f:004d9046 33db          xor ebx,ebx
017f:004d9048 33d2          xor edx,edx
```

继续跟到如4.所示的程序，按列出的方法改。

**4.**

```
017f:004d91ff 8d8558f8ffff    lea eax,[ebp+fffff858]
017f:004d9205 8bd6            mov edx,esi
017f:004d9207 e81cdaf2ff ->90 call 00406c28 ->nop
    ->33c9                  xor ecx,ecx
    ->8808                  mov bytes ptr [eax],cl
017f:004d920c bad0954d00      mov edx,004ce930
017f:004d9211 8d8558f8ffff    lea eax,[ebp+fffff858]
```

当跟到5.所示的地方，这是最后一个跳转了，强制它跳。

**5.**

```
017f:004d9456 e8c995f2ff call 00402a24
017f:004d945b 7402 ->eb02 jz 004d945f ->jmp 004d945f
017f:004d945d 33db       xor ebx,ebx
```

接下来，按几次`f10`到如6.所示的程序，按我列出的方法改。

**6.**

```
017f:00543893 a1c4f65400      mov eax,[0054f6c4]
017f:00543898 e88342edff ->eb00 call 00417b20 ->jmp 0054389a
    ->eb00                  jmp 0054389c
    ->90                    nop
017f:0054389d a1b0e65300      mov eax,[0054f6c4]
017f:005438a2 e86dadeeff      call 0041e834
```

再按几下`f10`就进入`wincmd`了，好了，用我的名字注册的。别忙还没完呢，`wincmd`还没把你折腾够呢，你晃动几下鼠标，`wincmd`就会冒出一个框框，告诉你感染`virus`一类的东西。这时按`ctrl+d`进入`softice`,下`bpx lockmytask`,按`f5`返回，点确定，被拦到，下`bd *`，按几次`f12`进入`wincmd`的地盘，往回倒到如7.所示的地方，将`JG 4EA8EC`改为`nop`，这是试出来的。

**7.**

```
4EA8A9 E87AC3F1FF CALL 406C28
4EA8AE 8B4304     MOV EAX,[EBX+04]
4EA8B1 83F808     CMP EAX,08
4EA8B4 7F36->9090 JG 4EA8EC ->NOP
4EA8B6 0F8456060000 JZ 4EAF12
```

不忙还没完呢，当你点`help`,在点`about`时，会弹出一个注册信息框，你再点确定时，`wincmd`会自动退出。所以你在点确定之前进入`softice`,下`bpx lockmytask`,按几次`f12`到如8.所示的地方，把`JZ 4F3BB7`改为`JMP 4F3BB7`才能继续使用它。

**8.**

```
4F3BA1 A198525400 MOV EAX,[545298]
4F3BA6 2DB9230000 SUB EAX,0023B9
4F3BAB 740A->EB0A JZ 4F3BB7 ->JMP 4F3BB7
```

完活:)

很麻烦吧@\_@

### 收尾工作

最后就是用`winhex`以来的东西把`wincmd`改为注册版了，机器码我都列出来了。

**说明：**`wincmd4.0--4.52`之间的所有版本都可以这么改，只是对应的地址不一样。

`cracked by zombieys`

`homepage http://zombieys.126.com`

`my oicq is 1789655`

`thanks for your supports 4dd466d0e1e0cc123afbe504da2462ca`
