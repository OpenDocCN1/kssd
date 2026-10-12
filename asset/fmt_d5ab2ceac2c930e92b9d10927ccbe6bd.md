
```
0046BF90  |.  837D E4 04    |cmp     dword ptr [ebp-1C], 4           ;  while(loc7 < 4);  
0046BF94  |.^ 0F85 D7FEFFFF \jnz     0046BE71  
0046BF9A  |.  8B45 F8       mov     eax, [ebp-8]  
0046BF9D  |.  8B55 E8       mov     edx, [ebp-18]                     ;  loc_6  
0046BFA0  |.  8910          mov     [eax], edx  
0046BFA2  |.  8B45 F8       mov     eax, [ebp-8]  
0046BFA5  |.  8B55 EC       mov     edx, [ebp-14]                     ;  loc_5  
0046BFA8  |.  8950 04       mov     [eax+4], edx  
0046BFAB  |.  8BE5          mov     esp, ebp  
0046BFAD  |.  5D            pop     ebp  
0046BFAE  \.  C3            retn  
```

老天，我看到这堆运算的第一个反应是几乎吐血，虽然注册码的值最后是由 `loc_5` 和 `loc_6` 两个变量写入的，但它的运算过程中用到了 `loc_7`（循环控制变量）到 `loc_11` 中所有的值，没有办法精简掉一些指令！如果仅仅是指令多些也还罢了，更恼人的是读入 `loc_9` 到 `loc_11` 的值的时候用了两级寄存器间接寻址，要知道，指针值随便修改一点，所指向的内容就可能相差十万八千里，何况还是两级指针！这要从何跟起？到这里已经用了我一整天时间，真有点想就此放弃，索性先去睡觉了。

第二天回来，再按头天的方法跟注册码，发现注册码变了。看来软件中确实有个 keygen 过程，而不仅仅是常值注册码那么简单！同时经过一夜的整理思路，对于那个寄存器二级寻址的问题好歹有了点头绪。首先找到一级间址所在的内存地址：

```
loc_9  = [546e24+[546de4+loc_7*0C]*4]
loc_10 = [546e24+[546de4+loc_7*0C+4]*4]
loc_11 = [546e24+[546de4+loc_7*0C+8]*4]
```

由于循环控制变量 `loc_7` 从 0 到 3，一级间址就是 `[546de4]` 到 `[546de4+2C]`，在数据窗口中找到这片内存：

```
00546DE4  03 00 00 00 02 00 00 00 00 00 00 00 01 00 00 00  ............  
00546DF4  00 00 00 00 02 00 00 00 02 00 00 00 01 00 00 00  ............  
00546E04  03 00 00 00 00 00 00 00 03 00 00 00 01 00 00 00  ............  
```

设置内存写入断点，没有发现程序往这片内存中写入数据，也就意味着这片内存的值是一些常数！再一看每个 `dword` 的值都不超过 3，代入到二级间址的表达式中，也就意味着 `loc_9` 到 `loc_11` 所读取的内容是在从 `[546e24]` 起始的 4 个 `dword` 中！同样在数据窗口中找到这片内存：

```
00546E24  CC 20 0C 71 C5 18 27 FF 20 7F 86 F3 DC 15 A0 ED  ?.q?'...?
```

也没有发现程序往其中写入数据！尝试把这 4 个 `dword` 的数值修改，跟出的注册码便无效了。这么说，这串数值可能是某种密钥。再看 `0046BE60` 和 `0046BE69` 两句，说明了这整个过程就是用这个密钥对注册码存放的内存进行变换。回到过程 `0046C6F4` 中看变换前的数据，`dword ptr [0012F9D8]` 的内容很明白是 `0000A4CB`，而 `dword ptr [0012F9DC]` 由两次过程调用 `call 0046C690` 给其赋值，过程的参数分别是 `qword ptr [0012F998]` 和 `qword ptr [0012F990]`。这两个 `qword` 是父过程传递进来的参数，回到父过程 `0050DFBC` 中，得知它们原来的地址是 `qword ptr ss:[0012F9C0]` 和 `qword ptr ss:[0012F9B8]`。而 `0050E09A` 一句明显地表明 `qword ptr [0012F9B8]` 所表示的浮点数是 `qword ptr [0012F9C0]` 加上 `10.00000`（注意，`[0050E230]` 仍属于代码段，其中的内容不可能被改写，无疑是常量。）所以关键就在 `qword ptr [0012F9C0]` 的内容是从什么地方来的。任何事物的产生总要有个源头，总不至于凭空变出一个注册码来吧！说了那么多，还是先把变量之间的依赖关系小结一下：

- 注册码 依赖于 `qword ptr ds:[0012F9D8]`
- `qword ptr ds:[0012F9D8]` 经过程 `0046BE4C` 变换，变换前：
  - `dword ptr ds:[0012F9D8] = 0000A4CB`
  - `word ptr ds:[0012F9DC]` 依赖于 `qword ptr ss:[0012F998]`（经过程 `0046C690` 变换）
  - `word ptr ds:[0012F9DE]` 依赖于 `qword ptr ss:[0012F990]`（经过程 `0046C690` 变换）
- `qword ptr ss:[0012F998] = qword ptr ss:[0012F9C0]`
- `qword ptr ss:[0012F990] = qword ptr ss:[0012F9B8]`
- `qword ptr ss:[0012F9B8] = qword ptr ss:[0012F9C0] fadd 10.00000`

接着往上看，`qword ptr [0012F9C0]` 的值是从浮点堆栈中拉出来的，而在 `0050E08E` 这个 `call` 之前，浮点堆栈还是空的。这说明 `qword ptr [0012F9C0]` 的值就是由这个 `call` 生成的。这个 `call` 我已经跟进去过，具体过程就不写了。其大致操作是先调用 `GetLocalTime` 取得本地机时间，然后计算出自从公元 1901 年元月 1 日 0 时 0 分 0 秒以来到这个时间为止所经历的天数（不足一天的部分以浮点小数表示）返回到浮点堆栈中。

到此为止，这个注册码生成的流程基本如下：

```
qword_1 = (double)公元 1901 年元月 1 日 0 时 0 分 0 秒以来到本地机时间为止所经历的天数
qword_2 = qword_1 + 10.00000
dword ptr qword_3 = 0000A4CB
word ptr qword_3+4 = sub_0046C690(qword_1)
word ptr qword_3+6 = sub_0046C690(qword_2)
sub_00468A4C(qword_3)
返回 qword_3——注册码
```

然而我对 `0046C690` 这个过程的处理是怎么回事还是不了解，兹列代码如下：

```
0046C690  /$  55            push    ebp  
0046C691  |.  8BEC          mov     ebp, esp  
0046C693  |.  51            push    ecx  
0046C694  |.  DD45 08       fld     qword ptr [ebp+8]  
0046C697  |.  E8 E464F9FF   call    00402B80  
0046C69C  |.  83FA 00       cmp     edx, 0  
0046C69F  |.  75 03         jnz     short 0046C6A4  
0046C6A1  |.  83F8 00       cmp     eax, 0  
0046C6A4  |>  74 29         je      short 0046C6CF                     ;  EDX:EAX != 0  
0046C6A6  |.  DD45 08       fld     qword ptr [ebp+8]  
0046C6A9  |.  E8 D264F9FF   call    00402B80  
0046C6AE  |.  52            push    edx  
0046C6AF  |.  50            push    eax  
0046C6B0  |.  A1 B06D5400   mov     eax, [546DB0]                     ;  ds:[00546DB0]=000088F9  
0046C6B5  |.  99            cdq  
0046C6B6  |.  290424        sub     [esp], eax  
0046C6B9  |.  195424 04     sbb     [esp+4], edx  
0046C6BD  |.  58            pop     eax  
0046C6BE  |.  5A            pop     edx  
0046C6BF  |.  83FA 00       cmp     edx, 0  
0046C6C2  |.  75 09         jnz     short 0046C6CD  
0046C6C4  |.  3D FFFF0000   cmp     eax, 0FFFF  
0046C6C9  |.  76 0C         jbe     short 0046C6D7  
0046C6CB  |.  EB 02         jmp     short 0046C6CF                     ;  花指令  
0046C6CD  |>  7E 08         jle     short 0046C6D7  
0046C6CF  |>  66:C745 FE 0>mov      word ptr [ebp-2], 0  
0046C6D5  |.  EB 13         jmp     short 0046C6EA  
0046C6D7  |>  DD45 08       fld     qword ptr [ebp+8]  
0046C6DA  |.  E8 A164F9FF   call    00402B80  
0046C6DF  |.  66:2B05 B06D5>sub      ax, [546DB0]  
0046C6E6  |.  66:8945 FE    mov     [ebp-2], ax  
0046C6EA  |>  66:8B45 FE    mov     ax, [ebp-2]  
0046C6EE  |.  59            pop     ecx  
0046C6EF  |.  5D            pop     ebp  
0046C6F0  \.  C2 0800       retn    8  
```

其中调用的 `00402B80` 过程：

```
00402B80  /$  83EC 0C       sub     esp, 0C  
00402B83  |.  9B            wait  
00402B84  |.  D93C24        fstcw   [esp]  
00402B87  |.  9B            wait  
00402B88  |.  D92D 28605400 fldcw   [546028]                            ;  ds:[00546028]=1F32  
00402B8E  |.  DF7C24 04     fistp   qword ptr [esp+4]  
00402B92  |.  9B            wait  
00402B93  |.  D92C24        fldcw   [esp]  
00402B96  |.  59            pop     ecx  
00402B97  |.  58            pop     eax  
00402B98  |.  5A            pop     edx                                   ;  EDX:EAX=变换后的数值  
00402B99  \.  C3            retn  
```

进入这个过程的时候浮点堆栈的 `st(0)` 还是好好的那个天数的数值，中途只是把控制字改为 `1F32`，输出结果就面目全非，怎么搞的。鄙人不懂 FPU 的工作原理，有没有哪位给解释一下？

上面所总结的算法只是程序“自带”的注册算法，至于任意输入一个注册码，判定它是否合法的代码则没有被涉及。（五天当中我有两天是在研究这事，但令人失望地一无所获。）所以可能会有挂一漏万的事发生。

### (3) 注册机

这个注册算法描述起来太拖泥带水，鄙人以为不如直接抽取上面所列的汇编代码来实现一个汇编注册机。（兹从略）

### 2. 感言

(1) 不知不觉，五天过去了……

(2) 个人认为除了跟自己的工作相关的软件以外，具体软件还是少解为妙。一个软件被解多了，保护措施必然相应增强，增加了后来的人解密的难度。如果这个软件跟你工作无关，只是出于兴趣而破解，那你更是在破坏资源。最近论坛上又有不少人在解国产软件，因此说这句话表明意见。

没人回贴，也没人加精么？可能是我说得不够清楚。但是涉及 FPU 的部分我确实不懂，只有直接抽取原程序反汇编的代码。这样吧，先贴出注册机源程序的主要部分。那个计算天数的过程大家应该都懂怎么回事，这里就略了吧。好歹搞了五天啊。

```
                    .586
      .model flat
      .option casemap:none
      include windows.inc
      include kernel32.inc
      includelib kernel32.lib
  Unk_Proceed     MACRO
                  ;这段代码是从反汇编代码的 00402B80 处抽取出来的
      ;不清楚它完成什么功能
      ;在原来的代码中是一个子程序，由于短小，现把它定义为一个宏
                  sub     esp, 0Ch
                  wait
                  fstcw   [esp]
                  wait
                  fldcw   magiccw                
                  fistp   qword ptr [esp+4]
                  wait
                  fldcw   [esp]
                  pop     ecx
                  pop     eax
                  pop     edx    
  Unk_Proceed     ENDM
      .const
  CryptKey        dd      EDA015DCh, F3867F20h, 710C20CCh, FF2718C5h 
                  dd      710C20CCh, F3867F20h, F3867F20h, FF2718C5h
      dd      EDA015DCh, 710C20CCh, EDA015DCh, FF2718C5h
  magicinc        real4   10.00000
  magiccw         dw      1F32h
      .data
  Regcode         dq      ?               ;注册码的内存区
```
