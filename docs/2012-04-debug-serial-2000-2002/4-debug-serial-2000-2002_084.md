
# Noterpro V1.1

> 原文：[`www.pediy.com/kssd/pediy02/forum271.htm`](https://www.pediy.com/kssd/pediy02/forum271.htm)

那位好心人帮我看下这个程序怎么破，看了好多破解教程，可是破解的第一个软件仍然是失败，真的好伤心，是不是我太笨了？它的下载地址是：

`http://www.newhua.com.cn/down/notepro.zip`

Noterpro V1.1

程式猎人

简介：这个软件是一个小型的编辑软件，功能比 `note` 强大。

追踪：`name: dahuilang`

`RN: 01234567`

在写这个教程前，我看到了看雪关于这个软件的整个计算过程，在他的文章中讲解的非常清楚，如何得到的注册码大家想必一看就知道了。但是看雪没有写出来如何能够追踪到计算的地方，我想对于初学者来说追踪到这里是非常关键的，而大家没有追踪出来的原因大多数也就在此，所以我就将它从开始处直到追踪到计算过程处写出来，供大家学习。

下 `bpx hmemcpy` 后你将被拦下。跳跃到主程序中，如下：

```
0040b1f7  call 0049a768
          lea edx [ebp-08]  <-出来
          xor eax eax
          ……
          ……
0040b214  call 0049a768
          lea edx [ebp-08]
          mov edx [edx]
          mov eax [004e9c80]
          pop eax
          call 00406bb0
          push eax      ****
          ……
          ……
0040b245  call 004dccc
          pop ecx
          test cl cl
          jz 0040b38b
```

现在我们就追踪到这里，你将首先从上面标记的地方出来，因为程序要读取两次，所以它使用了 `call 0049a768` 两次的，程序读取完后下面的那个 `call` 就是关键的地方，我在这里向大家解释一下为什么说 `call 00406bb0` 就是重要的地方。

因为在下面 `0040b245` 处开始，如果程序在这里跳跃的话，将出现错误提示，所以说这个跳跃是一个相当关键的地方，那么 `ecx` 的值决定这个程序能够注册成功与否了。程序调用 `ecx` 是使用出栈的方法得到 `ecx` 的，那么就要看在那里进栈了，观察程序发现只有`****`处进入堆栈，所以现在就知道 `call 00406bb0` 是一个重要的函数了。

进入 `call 00406bb0` 中，如下：

```
00406e6b  mov edx [ebp-08]
          call notepro!@stratil@xordecord$qqrx17
          lea edx [ebp-2c]
          lea eax [ebp-14]
          call 004dcd9c
          push eax      ****
          ……
          ……
00406e92  call 004dcccc
          pop ecx
          test cl cl
          jz 00406e98
          mov byte [ebp-51] 01  ****
00406e98  dec dword [ebp-34]
          ……
          ……
00406eae  mov al [ebp-51] ***
          mov edx 02
          push eax  **
          leax eax [ebp-08]
          ……
          ……
00406ecd  call 004dcccc
          pop eax  *
          mov edx [ebp-50]
```

现在开始介绍程序在这个 `call` 中如何得到的 `eax` 值的，因为出这个 `call` 后我们最关心的就是 `eax` 值，那么就要注意这个值的变化。我先从后面看，这个 `eax` 就是程序返回的值，它从这里得到的。

`pop eax  *`

那么在上面就有进入堆栈的地方

`push eax  **`

它又是 `mov al [ebp-51]` 从这得到的，再向上查找程序在这里使用 `mov byte [ebp-51] 01`，这个是决定你注册成功的关键，那么上面正好有一个 `jz`，这就应当找到了关键地方了。但是在这里你不要进入

```
00406e92  call 004dcccc
```

中，因为它不是最后的运行 `call`，为什么？还是使用前面讲到的地方，因为程序使用的是 `pop ecx` 来传值的，所以上面才是关键的地方。

`call 004dcd9c`
`push eax`

这里者关键的地方，那么如果你进入这里 `call` 中你会发现这里只是一个比较过程，它的两个值一个是明码，另一个不是，其中名码就是通过你的 `name` 计算得到的，而不是明码的就是通过你的注册码计算得到的。现在的关键是找到如何通过你的注册码计算得到这个比较码的。那么在上面茫茫的代码中如何找呢？

这里也有一个技巧，因为如果你进入 `call 00406bb0` 这里后，你会发现程序前面的计算入口值都是 `name`，也就是前面的计算都真对 `name` 的，而在这里你可以看到它的入口值是你输入的 `RN`

```
00406e6b  mov edx [ebp-08]  <-RN    ******
          call notepro!@stratil@xordecord$qqrx17
          lea edx [ebp-2c]
          lea eax [ebp-14]
```

那么下面的那个 `call` 就是运行的关键了，进入后你就会发现它同看雪介绍的一样，你手中一定要有一个 `XOR` 表，如果没有话，自己做一个就行了。下面就是看雪破解的计算过程。

程序名是：`notepro 1.04`

下载：`http://www.newhua.com.cn/down/notepro.zip`

我们看看这个程序的算法，这程序先跟据姓名如：`toye`，计算出一中间数 `m`，这里 `m=4091327126`，然后将这 `m` 各位依次转换 `ASCII` 码，再依次与内部固定字串：`67gjhab480klvn176` 异或运算，结果就是以两位十六进制表示，这就是注册码。

如：设我们输入的姓名格式：`a1 a2 a3 ...a(n)`

`m` 分成三段，依次根据姓名计算。设 `m=m1+m2+m3`

`m1=a1+a2+a3+--+a(n-1)+61` （全部数字是以十进制表示计算的）

`m2=（a1+a(n))*3D` （全部数字以十六进制计算，结果转换成十进制）

`m3=3D+3D+n` （全部数字以十六进制计算，结果转换成十进制）

在这里：`toye`

`m=m1 m2 m3= 409 1327 126 =4091327126`

`m=4091327126`

展开：`34 30 39 31 33 32 37 31 32 36` （将 `m` 转换成 `ASCII` 码）

`XOR：36 37 67 6A 68 61 62 34 38 30` （固定字串 `67gjhab480klvn176` 以 `ASCII` 表示）

——————————————————

`02 07 5e 5b 5b 53 51 03 09 02 5d`

因此：注册码就是：`02075e5b5b53510309025d`

至此，注册机应不难写了。

下面简单将破解过程的代码例出：

用 `SOFTICE` 来破，用 `WF` 打开浮点开关。

①计算 `m1` 部分

```
ST0  111.                           ST4  empty
ST1  empty                           ST5  empty
ST2  empty                           ST6  empty
ST3  empty                           ST7  empty
-----------------------------------------------------------------------------

0167:00406898  MOV       ESI,EBX
0167:0040689A  PUSH      ESI
0167:0040689B  LEA       EAX,[EBP-04]
0167:0040689E  PUSH      EAX
0167:0040689F  CALL      004DBEDC
0167:004068A4  ADD       ESP,08
0167:004068A7  LEA       EAX,[EBP-04]
0167:004068AA  CALL      004DC178
0167:004068AF  ADD       ESI,[EBP-04]
0167:004068B2  DEC       ESI
0167:004068B3  MOVSX     EDX,BYTE PTR [ESI]  //依次将姓名各个字符传给 edx
0167:004068B6  AND       EDX,7F
0167:004068B9  MOV       [EBP-70],EDX
0167:004068BC  FILD      DWORD PTR [EBP-70]    //将这个字符放进浮点寄存器 ST0
0167:004068BF  FADD      REAL8 PTR [EBP-5C]    //ebp-5c 的初值为 0 ，ST0 与 ebp-5c 的值相加
0167:004068C2  FSTP      REAL8 PTR [EBP-5C]    //将结果取出，放进 ebp-5c
0167:004068C5  INC       EBX
0167:004068C6  CMP       DWORD PTR [EBP-04],00
0167:004068CA  JZ        004068D4
0167:004068CC  MOV       ECX,[EBP-04]
0167:004068CF  MOV       EAX,[ECX-04]
0167:004068D2  JMP       004068D6
0167:004068D4  XOR       EAX,EAX
0167:004068D6  CMP       EBX,EAX    //依次循环 n-1 次，只有姓名的最后一字符没参加运算。
0167:004068D8  JL        00406898       (JUMP )
0167:004068DA  FILD      DWORD PTR [EDI+000005C8]  //将定值 61（十进制）放进浮点寄存器 ST1
0167:004068E0  FADD      REAL8 PTR [EBP-5C]    //将 61 加上前 n-1 位的和。成为 m1
0167:004068E3  FSTP      REAL8 PTR [EBP-5C]
0167:004068E6  PUSH      01
0167:004068E8  LEA       EDX,[EBP-04]
0167:004068EB  PUSH      EDX
```

②`m2` 部分计算

```
ST0  13237.                         ST4  empty
ST1  empty                           ST5  empty
ST2  empty                           ST6  empty
ST3  empty                           ST7  empty
————————————————————————————————

0167:00406929  ADD       EBX,[EBP-04]
0167:0040692C  DEC       EBX
0167:0040692D  MOVSX     EDX,BYTE PTR [EBX]
0167:00406930  AND       EDX,7F
0167:00406933  POP       ECX
0167:00406934  ADD       ECX,EDX     //ecx=是姓名第一字符，edx=姓名的第后一字符
0167:00406936  IMUL      ECX,[EDI+000005C8] //将上面的结果乘以十六进制数：3D.  成为 m2
```



### 汇编代码片段 ③ m3 的计算

```
0167:0040693D MOV [EBP-70],ECX
0167:00406940 FILD DWORD PTR [EBP-70]
0167:00406943 FSTP REAL8 PTR [EBP-64]
0167:00406946 CMP DWORD PTR [EBP-04],00
0167:0040694A JZ 00406954
0167:0040694C MOV EAX,[EBP-04]
0167:0040694F MOV EDX,[EAX-04]
0167:00406952 JMP 00406956

0167:0040695C ADD EAX,EAX //EAX 的值是定值：3D（十六进制）
0167:0040695E ADD EDX,EAX //EDX 值是姓名的位数 n 计算的结果就是 m3
0167:00406960 MOV [EBP-70],EDX
0167:00406963 FILD DWORD PTR [EBP-70]
0167:00406966 FSTP REAL8 PTR [EBP-6C]
0167:00406969 MOV WORD PTR [EBP-40],0038
0167:0040696F FLD REAL8 PTR [EBP-6C]
0167:00406972 CALL 004D5094
0167:00406977 PUSH EDX
```

### 汇编代码片段 ④

```
167:00406A4E XOR ECX,ECX
0167:00406A50 MOV [EBP-2C],ECX
0167:00406A53 LEA ECX,[EBP-2C]
0167:00406A56 INC DWORD PTR [EBP-34]
0167:00406A59 MOV EAX,[EDI+000005DC]
0167:00406A5F MOV EDX,[EBP-08]
0167:00406A62 CALL 004354D8 //将输入的注册码与定值 67gjhab480klvn176 异或运算，以结果 K 表示
0167:00406A67 LEA EDX,[EBP-2C]
0167:00406A6A LEA EAX,[EBP-14]
0167:00406A6D CALL 004DC0F0 // 将结果 k 与 M 比较
0167:00406A72 PUSH EAX //如相等 EAX =1 ,将 EAX 入栈
0167:00406A73 DEC DWORD PTR [EBP-34]
0167:00406A76 LEA EAX,[EBP-2C]
0167:00406A79 MOV EDX,00000002
0167:00406A7E CALL 004DC020
0167:00406A83 POP ECX //cl 出栈，即将 EAX 的值传到 cl
0167:00406A84 TEST CL,CL // 如相等 cl =1
0167:00406A86 JZ 00406A8C
0167:00406A88 MOV BYTE PTR [EBP-51],01
0167:00406A8C DEC DWORD PTR [EBP-34]
```

### 初学者快速获得 Notepro v1.1 注册码

1. 启动 `trw2000` 和 `notepro v1.1`
2. 填入 `Name` 和 `Code` 作为注册资料
   我用 `name`: `wind`
   `code`: `7878787878`
3. `Ctrl+N`, 下 `bpx hmemcpy`
4. 返回 `Notepad`, 按 `OK`!程序被中断!
5. `pmodule`, 来到 `Notepro`
6. `F10` 开始追踪

...来到如下程序段

```
015F:0040B219 LEA EDX,[EBP-04]
015F:0040B21C MOV EDX,[EDX]
015F:0040B21E MOV EAX,[004E9C80] ----->用 d edx 可以看到 Name
015F:0040B223 POP ECX ----->d ecx 可以看到我们输入的注册码！
```

故判断下面那个 `call` 是计算或转换部分!

```
015F:0040B224 CALL 00406BB0 ---->进入这个 call，
```

这个 `call` 就是计算注册码的部分

进入 `40b224` 那个 `call` 后，用 `f10` 跟踪一段时间后，来到：

```
015F:00406E0F CALL 004DCCCC
015F:00406E14 DEC DWORD [EBP-34] ----->用 d edx 看，可以在内存区看到
```

通过名字计算得出的数值！

这里用 `ncode` 表示

```
015F:00406E17 LEA EAX,[EBP-24]
```

然后来到如下处：

```
015F:00406E65 MOV EAX,[EDI+0610]
015F:00406E6B MOV EDX,[EBP-08] ----->用 d eax 可以看到一串字符！这串字符是固定的！
015F:00406E6E CALL `NOTEPRO!@Strutils@XorDecode$qqrx17System@AnsiStringt1`
```

通过 `xor` 就可以得到自己的注册码了！

我的 `ncode` 是：`39513359126`

固定字串为：`67gjhab480klvn176`

转换成 `ascii`: `33 39 35 31 33 33 35 39 31 32 36`

固定字串转换成 `ascii`：`36 37 67 6A 68 61 62 34 38 30 6B`

进行异或操作后得到： `05 0e 52 5b 5b 52 57 0d 09 02 5d`

所以我的注册码就是：

`Name`: `wind`

`Code`: `050e525b5b52570d09025d`

谢谢看雪大虾的文章！ `4dd466d0e1e0cc123afbe504da2462ca`
