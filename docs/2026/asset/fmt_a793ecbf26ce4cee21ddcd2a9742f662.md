
The `OF`, `SF`, `ZF`, `AF`, `CF`, and `PF` flags are set according to the result.

把`00000286` `AND` `00000815`使用二进制表示：

```
0000 0000 0000 0000 0000 0010 1000 0110
AND        0000 0000 0000 0000 0000 1000 0001 0101
```

我们现在就可以看到，VM要保留的是 `OF` `AF` `PF` `CF` 位。那么，`SF`和`ZF`位为什么不在这里保留呢？我们要想到，由于这里并不是A-B的最后结果，`SF` 和 `ZF`位要等到最后的运算完成才能知道。在标志位A中，`PF`位为1，`PF`位被保留。

第一个`AND`数`00000815`与第二个`AND`数`FFFFF7EA`之间是有内在联系的。`00000815+FFFFF7EA=FFFFFFFF`，也就是说，这两个这两个`AND`操作时可以把所有的标志位都保留下来的，不会出现遗漏。而把它分开的话，是由于变换了减法的运算方式不进行保留对应的保留。

最后的`NAND(A,A)`：

```
NOT A                ；第一个操作数
NOT A                ；第二个操作数
AND A,A                ；最终标志位B 00000246是来自这里
```

关于`AND`逻辑运算，Intel的资料显示：

`AND`—Logical AND

Operation

`DEST`        `DEST AND SRC`;

Flags Affected

The `OF` and `CF` flags are cleared; the `SF`, `ZF`, and `PF` flags are set according to the

result. The state of the `AF` flag is undefined.

把`00000246` `AND` `FFFFF7EA`使用二进制表示：

```
0000 0000 0000 0000 0000 0010 0**1**00 0110
AND        1111 1111 1111 1111 1111 0111 1**1**10 1010
```

VM要把除了上面`00000815`保留了的 `OF AF PF CF` 以外的标志位都保留了下来。在标志位B中，`IF ZF PF` 和第二位是Intel的保留位默认为1 这4个标志位为1，所以`IF ZF PF`被保留。

两个标志位相加后，最终合并成为两个操作数`SUB`指令后的标志位`00000246`

下面我们来看第三阶段：

构建跳转结构：

```
0013F9A0   0004
0013F9A4   0013F9A8  .
0013F9A8   4DBE49D5  IM
0013F9AC   4DBE4AD8  JM
```

把两个跳转地址`4DBE49D5`与`4DBE4AD8`压入堆栈。`0013F9A8`成为跳转地址指针，指向第一个跳转地址。如果`0013F9A8`指针+4，它就会指向第二个指针。最后还有1个`0004`，它并不是用于给指针+4的操作数，它要参与到巧妙判断`ZF`位的运算中。

接下来，VM用`NAND`执行一次`AND`操作，操作数是：标志位`00000246`与`00000040` （在`NAND`操作中，VM不意外的隐藏了一次`NAND(B,B)`操作，直接传递`00000040`的相反数`FFFFFFBF`）

```
0013F998   FFBF
0013F99C   FDB9FFFF
0013F9A0       FFFF  .
0013F9A0   0004                                ；为了清晰变现，把它分开显示
0013F9A4   0013F9A8  .
0013F9A8   4DBE49D5  IM
0013F9AC   4DBE4AD8  JM
```

`VM_NANDdw`

```
0013F99C   0040                                ；运算结果为00000040
0013F9A0   00040000  ...
0013F9A4   0013F9A8  .
0013F9A8   4DBE49D5  IM
0013F9AC   4DBE4AD8  JM
```

`NAND`操作是：

```
NOT(A)
NOT(B)
AND(A,B)
```

所以，在这条伪指令的内部，`00000246 AND 0000040`

把`00000246 AND 00000040`使用二进制表示：

```
0000 0000 0000 0000 0000 0010 0100 0110
AND        0000 0000 0000 0000 0000 0000 0100 0000
```

唯一的检测`ZF`位，如果`ZF`位为1，那么结果将是`00000040`，否则是`00000000`。由于`ZF`位刚好是在byte的4的位置，把它和前面的跳转地址指针相加，`0013F9A8+0`则是不变，指向第一个地址，+4就指向第二个地址，所以刚好可以让`AND`后的结果与指针`0013F9A8`进行1次加法运算，如果`ZF`位是1,`0013F9A8+4`将指向`4DBE4AD8`完成判断跳转。由于`ZF`位的前面还有1个byte的数据0，就是`00000040`中最后的1个byte0，如果直接和`0013F9A8`相加，就变成+40，所以要先进行1次4个bit的右移，`00000040`变为`00000004`,这样才正确。

`[LEFT]ZF位为1,AND 00000040后：                                ZF位为0，AND 00000040后：`

```
00000040                                                00000000
SHR(4)        00000004                                        SHR(4)        00000000
ADD        0013F9A8                                        ADD        0012F9A8
结果        0013F9AC                                        结果        0013F9A8
0013F9A4   0013F9AC  .                                        0013F9A4   0013F9A8  .
0013F9A8   4DBE49D5  IM                                        0013F9A8   4DBE49D5  IM
0013F9AC   4DBE4AD8  JM                                        0013F9AC   4DBE4AD8  JM
[/LEFT]`

`ZF`位的不同带来跳转地址的不同，把相应的跳转地址解密后，使用`VM_JMP`给VM的指令指针`ESI`赋值，全程结束。

进行`ZF`位比较的话，只需要比较最后的标志位B就可以了，而且可以进行直接的比较，不需要这样截取+拼接，那么为什么VMP还需要在整个过程中截取了所有的标志位呢？我想，可以这么来考虑，在VMP中标志位的截取+拼接在代码中是属于一个模块，不管VMP要检测哪个标志位，它都是先调用这个模块然后再进行标志位检测。虽然在单纯的`ZF`位检测中，有了很多的不必要的操作，但是它增加了通用性，只要调用了这个模块，VMP在后面可以接上任意标志位的检测。

## 3.NOTEPAD全程跟踪

在这一章里，我们将全过程浏览`NOTEPAD.EXE`文件。本章显示的代码，外壳的花指令将全部跳过，VM的伪指令则部份跳过。

### 3.1.TLS

#### 3.1.1.到达Dispatch部份

`VMProtect2.04`加壳程序是从`TLS`开始运行的，我们首先点击`OD`的`options`菜单，修改`Startup and exit`选项，让`OD`中断在`TLS callback`里。载入`NOTOPAD.EXE`后，程序停在这里：

```
004253CD   $  68 9AA597B7   PUSH B797A59A                            ; TLS callback function
```

当前的寄存器值：

```
EAX 004253CD NOTEPAD.004253CD
ECX 00000020
EDX 000359F4
EBX 00000000
ESP 0013F9B0
EBP 0013F9CC
ESI 0013F9C0
EDI 00400000 NOTEPAD.
```

进入`VM`之前，`VMP`要保存当前的各个寄存器值，`VM`堆栈要分配，同时给伪指令指针寄存器`ESI`赋值等等，初始化结束后，进入`Dispatch`部份`VM`开始运行。`F7`单步下去：

```
0043BD02   .  C74424 40 0A4 MOV DWORD PTR SS:[ESP+40],2EF6420A       ; |\*
00429088  |> \C74424 44 19C MOV DWORD PTR SS:[ARG.17],C456C619       ; \*
；压入VM的两个定量。
0043DCD2  |.  893424        MOV DWORD PTR SS:[ESP],ESI               ; \*
0043CF0D   .  57            PUSH EDI                                 ; \*
0043CF17   .  891424        MOV DWORD PTR SS:[ESP],EDX               ; \*
0043CF1D   .  50            PUSH EAX                                 ; \*
0043E17A  |.  896C24 04     MOV DWORD PTR SS:[ARG.1],EBP             ; \*
0043D741  |> /871C24        XCHG DWORD PTR SS:[ESP],EBX              ; \*
0043D746  |.  894C24 20     MOV DWORD PTR SS:[ESP+20],ECX            ; \*
；7个寄存器保存完毕
0043D750  |.  875424 40     XCHG DWORD PTR SS:[ESP+40],EDX           ; |Arg17, \*
0043E62E  /$  9C            PUSHFD                                   ; \*
0043E62F  |.  8F4424 40     POP DWORD PTR SS:[ESP+40]                ; \*
0043E636  |.  FF35 89D24300 PUSH DWORD PTR DS:[43D289]               ; \*
0043E63C  |.  8F4424 3C     POP DWORD PTR SS:[ESP+3C]                ; \*
0043E646  |.  C74424 38 000 MOV DWORD PTR SS:[ESP+38],0              ; \*
；多保存1个寄存器，由于ESP是动态的，这个位置相当于是ESP寄存器的位置；EFLAGS；内存地址[43D289]；常量0；一共保存13个数据
        0013F97C   00000000  ....  ;常量0                        20
        0013F980   00000000  ....  ;[43D289]                        24        8121D2F0相加
        0013F984   00000246  F..  ;EFLAGS                        0C
        0013F988   000359F4  Y.   ;EDX(给ESP寄存器的位置）        00
        0013F98C   00000020   ...  ;ECX                                08
        0013F990   00000000  ....  ;EBX                                1C
        0013F994   0013F9CC  .    ;EBP                                28
        0013F998   004253CD  SB.   ;EAX                                10
        0013F99C   000359F4  Y.   ;EDX                                2C
        0013F9A0   00400000  ..@.  ;EDI                                30
        0013F9A4   0013F9C0  .    ;ESI                                38
        0013F9A8   C456C619  V    ;常量B                        3C
        0013F9AC   2EF6420A  .B.   ;常量A                        18
        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A
0043E65B  |.  8B7424 68     MOV ESI,DWORD PTR SS:[ESP+68]            ; \*
；这里读取的是常量A=2EF6420A，以下是解密2EF6420A的过程：
0043E665  |.  0FCE          BSWAP ESI                                ; \*
0043E66E  |.  4E            DEC ESI                                  ; \*
0043E67D  |.  81F6 63A1000A XOR ESI,0A00A163                         ; \*
；结束，ESI=0042574E，这里就是VM的指令序列
0043E68C  |.  8D6C24 34     LEA EBP,[ESP+34]
0043E692  |.  81EC 8C000000 SUB ESP,8C                               ; \*
0043E69C  |.  89E7          MOV EDI,ESP                              ; \*
；第一条指令EBP取得的地址是0013F97C的位置，也就是13个保存数据的结束地址，要在这个的基础上分配VM堆栈
；保存的13个dword=34byte，ESP在执行减法前，位于0013F9FC的34个byte位置，减法分配8Cbyte；34+34+8C=F4byte=61dword
；VM堆栈的详细分析请查阅1.2.VM堆栈
0043E6A9  |.  89F3          MOV EBX,ESI                              ; \*
；EBX是VM解密数据的辅运算寄存器，初始化为VM的指令序列地址0042574E
0043E6B8  |.  0375 00       ADD ESI,DWORD PTR SS:[EBP]               ; \*
；ESI的地址还要和常量0相加，相加会改变VM指令序列的地址，决定VM第一次运行后要执行的指令
；常量0可以看作是VM内部的一个修正量，在NOTEPAD里它是0
```

到这里，一切初始化完成，程序已经到达`Dispatch`（调遣）部份。整个过程概括为：保存数据、分配`VM`堆栈、给`ESI`赋值。下一条指令就是所有伪指令执行完成后的返回地址`0043E6BB`。

```
0043E6BB  |> >66:0FA5FA     SHLD DX,DI,CL                            ; Dispatch                花指令，没有什么意义
```

#### 3.1.2.Dispatch部份解析

下面是第1条伪指令的获取过程：

```
0043E6BF  |.  8A46 FF       MOV AL,BYTE PTR DS:[ESI-1]               ; \*
；开始读取伪指令序列号，以下是解密伪指令序列号C0的过程：
0043E6C4  |.  30D8          XOR AL,BL
0043E6CE  |.  F6D0          NOT AL                                   ; \*
0043E6D6  |.  FEC8          DEC AL                                   ; \*
0043E6DA  |.  C0C8 07       ROR AL,7                                 ; \*
；结束，AL=E0，它将用于在DispatchTable（调遣表）中定位出伪指令地址。
0043E6E1  |.  83EE 01       SUB ESI,1                                ; \*
```
