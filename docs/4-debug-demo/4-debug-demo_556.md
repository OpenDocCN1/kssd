
# 脱双层壳去自校验破解《国家药品 XXXX 2.18 专业版》一条龙

> 原文：`www.pediy.com/kssd/pediy07/pediy7-349.htm`

**【破文标题】**：脱双层壳去自校验破解《国家药品审评中心受理品种搜索专家 2.18 专业版》一条龙

**【破文作者】**：`KuNgBiM[DFCG]`

**【作者邮箱】**：`gb_1227@163.com`

**【软件名称】**：国家药品审评中心受理品种搜索专家 2.18 专业版

**【保护方式】**：序列号 + 功能限制 + 自校验 + 重启验证

**【加密保护】**：`EXEStealth 2.75a`、`ASPack 2.12`

**【编译语言】**：`Microsoft Visual C++ 6.0`

---

## 分析过程

1. 运行程序，东看看西看看，查找程序有那些功能限制和对我们有用的信息。

2. 用 `PEiD` 查壳，`EXEStealth 2.75a` -> `WebtoolMaster`，是个加密壳，再深一步分析（察看区段情况）：

   | No | Name | VSize | VOffset | RSize | ROffset | Charact. |
   | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
   | 01 | `.text` | `00056000` | `00001000` | `00021200` | `00000600` | `C0000040` |
   | 02 | `.rdata` | `00011000` | `00057000` | `00004C00` | `00021800` | `C0000040` |
   | 03 | `.data` | `0000E000` | `00068000` | `00002A00` | `00026400` | `C0000040` |
   | 04 | `.rsrc` | `00006000` | `00076000` | `00006000` | `00028E00` | `C0000040` |
   | 05 | `.aspack` | `00002000` | `0007C000` | `00001400` | `0002EE00` | `C0000040` |
   | 06 | `.adata` | `00001000` | `0007E000` | `00000000` | `00030200` | `C0000040` |
   | 07 | `ExeS` | `00002000` | `0007F000` | `00000DF2` | `00030200` | `E00000E0` |

   光从区段名来看，初略估计该软件加壳不只一个，至少加有 `EXEStealth` 和 `Aspack` 壳，如果估计没错的话，我想作者可能是先用 `Aspack` 压缩程序大小，然后用 `EXEStealth` 加密吧~呵呵~~

3. 用 `Ollydbg` 载入，跟踪分析破解。

---

## 脱壳过程

`Ollydbg` 载入主程序：

老规矩：设置 `Ollydbg` 忽略所有的异常选项，用 `IsDebugPresent 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
0047F060  > /EB 58             jmp short drugdir.0047F0BA             ; 载入程序后停在这里，F7 让它跳
0047F062   |53                 push ebx
0047F063   |68 61726577        push 77657261
0047F068   |61                 popad
0047F069   |72 65              jb short drugdir.0047F0D0
........
```

```
0047F0BA    90                 nop                                     ; 跳到这里，继续 F7 单步运行 2 次
0047F0BB    60                 pushad
0047F0BC    90                 nop                                     ; 单步运行到这里，注意观察寄存器变化
0047F0BD    E8 00000000        call drugdir.0047F0C2
0047F0C2    5D                 pop ebp
0047F0C3    81ED F7274000      sub ebp,drugdir.004027F7
0047F0C9    B9 15000000        mov ecx,15
0047F0CE    83C1 04            add ecx,4
0047F0D1    83C1 01            add ecx,1
0047F0D4    EB 05              jmp short drugdir.0047F0DB
0047F0D6  - EB FE              jmp short drugdir.0047F0D6
........
```

**寄存器状态**：

```
EAX 00000000
ECX 0012FFB0
EDX 7FFE0304
EBX 7FFDF000
ESP 0012FFA4       // esp=0012ffa4
EBP 0012FFF0
ESI 77F57D70 ntdll.77F57D70
EDI 77F944A8 ntdll.77F944A8
EIP 0047F0BC drugdir.0047F0BC
```

根据 ESP 定律规则，现在在命令栏中下 `hr 0012ffa4` 命令，回车，`F9` 运行：

```
0047F839    50                 push eax                                  ; 这里断下，继续 F7 单步运行
0047F83A    33C0               xor eax,eax
0047F83C    64:FF30            push dword ptr fs:[eax]
0047F83F    64:8920            mov dword ptr fs:[eax],esp
0047F842    EB 01              jmp short drugdir.0047F845                ; 运行到这里，继续 F7 一次就会跳到解压代码的地方
0047F844    8700               xchg dword ptr ds:[eax],eax
0047F846    0000               add byte ptr ds:[eax],al
0047F848    0000               add byte ptr ds:[eax],al
0047F84A    0000               add byte ptr ds:[eax],al
........
```

```
0047F845    0000               add byte ptr ds:[eax],al                   ; 这里，代码就开始解压了，继续 F9 一次，让代码解压
0047F847    0000               add byte ptr ds:[eax],al
0047F849    0000               add byte ptr ds:[eax],al
0047F84B    0000               add byte ptr ds:[eax],al
0047F84D    0000               add byte ptr ds:[eax],al
0047F84F    0000               add byte ptr ds:[eax],al
0047F851    0000               add byte ptr ds:[eax],al
0047F853    0000               add byte ptr ds:[eax],al
........
```

```
0047C002    E8 03000000        call drugdir.0047C00A                       ; 代码到这里就基本上解密完毕了，准备解压，继续 F9 一次
0047C007  - E9 EB045D45        jmp 45A4C4F7
0047C00C    55                 push ebp
0047C00D    C3                 retn
0047C00E    E8 01000000        call drugdir.0047C014
0047C013    EB 5D              jmp short drugdir.0047C072
0047C015    BB EDFFFFFF        mov ebx,-13
........
```

```
0047C3B0   /75 08             jnz short drugdir.0047C3BA                 ; 解密解压全部完成，准备返回程序入口，F7 一次
0047C3B2   |B8 01000000       mov eax,1
0047C3B7   |C2 0C00           retn 0C
0047C3BA   \68 3D134300       push drugdir.0043133D                       ; 这里 0043133D 就是程序的 OEP，F7 继续
0047C3BF    C3                 retn                                      ; 飞向光明之颠~~ F7 继续一次
........
```

```
0043133D    55                 push ebp                                  ; 在这儿用 LordPE 纠正 ImageSize 后完全 Dump 这个进程
0043133E    8BEC               mov ebp,esp
00431340    6A FF              push -1
00431342    68 88B54500        push drugdir.0045B588
00431347    68 DC724300        push drugdir.004372DC
0043134C    64:A1 00000000     mov eax,dword ptr fs:[0]
00431352    50                 push eax
00431353    64:8925 00000000   mov dword ptr fs:[0],esp
0043135A    83EC 58            sub esp,58
0043135D    53                 push ebx
0043135E    56                 push esi
0043135F    57                 push edi
00431360    8965 E8            mov dword ptr ss:[ebp-18],esp
00431363    FF15 70724500      call dword ptr ds:[457270]               ; kernel32.GetVersion
00431369    33D2               xor edx,edx
0043136B    8AD4               mov dl,ah
0043136D    8915 943A4700      mov dword ptr ds:[473A94],edx
00431373    8BC8               mov ecx,eax
........
```

运行 `ImportREC 1.6`，选择这个进程，把 `OEP` 改为 `0003133D`，点 `IT AutoSearch`，cut 一个无效指针，其余函数全部有效。`FixDump`！

再用 `PEiD` 插件 `Rebuild PE` 优化一下，程序大小变为 `486 KB`，`Microsoft Visual C++ 6.0` 编译。

关闭 `Ollydbg`，试运行，窗口一闪而过，靠~~~~程序有自校验，没办法，去掉烦人的自校验！！！GO~~

---

## 去自校验过程

重新打开 `Ollydbg`，载入刚刚我们脱壳修复优化后的 `dumped_.exe` 文件（这里我采用的是默认脱壳文件名）。

根据以往的经验，软件校验无非就是采用校验文件名和大小作为依据，那么我们大胆猜想到肯定使用了下面这条语句：

`cmp eax,30FF2` （`30FF2` 的十进制就是原文件大小 `200,690` 字节）

现脱壳文件为 `497,664` 字节，也就是说现在代码应该替换为：`cmp eax,79800`

所以用 `Ctrl+S` 搜索命令 `cmp eax,30FF2`：

```
00404E03    E8 EA280400        call dumped_1.004476F2
00404E08    3D F20F0300        cmp eax,30FF2                             ; 第一处
00404E0D    74 07              je short dumped_1.00404E16

0040931D    E8 D0E30300        call dumped_1.004476F2
00409322    3D F20F0300        cmp eax,30FF2                             ; 第二处
00409327    74 07              je short dumped_1.00409330

00409815    E8 D8DE0300        call dumped_1.004476F2
0040981A    3D F20F0300        cmp eax,30FF2                             ; 第三处
0040981F    74 07              je short dumped_1.00409828

0040A213    E8 DAD40300        call dumped_1.004476F2
0040A218    3D F20F0300        cmp eax,30FF2                             ; 第四处
```



### 0040A21D - 004173ED 汇编代码片段

```
0040A21D    74 07                 je short dumped_1.0040A226

0040B413    E8 DAC20300          call dumped_1.004476F2

0040B418    3D F20F0300          cmp eax,30FF2                    ; 第五处

0040B41D    74 07                 je short dumped_1.0040B426

0040FE94    E8 59780300          call dumped_1.004476F2

0040FE99    3D F20F0300          cmp eax,30FF2                    ; 第六处

0040FE9E    74 07                 je short dumped_1.0040FEA7

00410EA5    E8 48680300          call dumped_1.004476F2

00410EAA    3D F20F0300          cmp eax,30FF2                    ; 第七处

00410EAF    74 07                 je short dumped_1.00410EB8

00412423    E8 CA520300          call dumped_1.004476F2

00412428    3D F20F0300          cmp eax,30FF2                    ; 第八处

0041242D    0F84 A6000000        je dumped_1.004124D9

00413E35    E8 B8380300          call dumped_1.004476F2

00413E3A    3D F20F0300          cmp eax,30FF2                    ; 第九处

00413E3F    74 07                 je short dumped_1.00413E48

0041587F    E8 6E1E0300          call dumped_1.004476F2

00415884    3D F20F0300          cmp eax,30FF2                    ; 第十处

00415889    74 07                 je short dumped_1.00415892

004173E8    E8 05030300          call dumped_1.004476F2

004173ED    3D F20F0300          cmp eax,30FF2                    ; 第十一处

004173F2    74 07                 je short dumped_1.004173FB
```

### 总结去自校验修改点

```
00404E08    3D F20F0300          cmp eax,30FF2
00409322    3D F20F0300          cmp eax,30FF2
0040981A    3D F20F0300          cmp eax,30FF2
0040A218    3D F20F0300          cmp eax,30FF2
0040FE99    3D F20F0300          cmp eax,30FF2
00410EAA    3D F20F0300          cmp eax,30FF2
00412428    3D F20F0300          cmp eax,30FF2
00413E3A    3D F20F0300          cmp eax,30FF2
00415884    3D F20F0300          cmp eax,30FF2
004173ED    3D F20F0300          cmp eax,30FF2
```

以上的汇编代码`cmp eax,30FF2`全部替换为`cmp eax,79800`保存即可！

好了，修改以上的代码后保存文件为`dumped_1.exe`！OK，正常运行！校验解除咯~~~~

### 破解过程

我们知道了该程序是 VC6.0 写的，而且在做准备工作的时候我们得到了一些的重要提示信息作为破解入手点：

再次打开`Ollydbg`，载入我们脱壳除校验文件`dumped_1.exe`，右键使用`Ultra String Reference`插件的`Find ASCII`功能项，查找我们需要的相关信息：

```
........
004271E2   push drugdir.00455D70                    肛 2F
0042721A   push drugdir.0046D1A4                    药搜
0042722D   push drugdir.0046D19C                    高手                       <-- 这是什么意思？？
0042730C   push drugdir.0046D198                    %c
00427319   push drugdir.0046D194                    %d
004274EB   push drugdir.0046D24C                    注册成功                   ★重要提示信息★
004274F0   push drugdir.0046D214                    注册成功！搜索专家的所有功能已对你开放！感谢你的支持！   ★重要提示信息①★
004274FC   push drugdir.0046D208                    drugreg.ini                 ★  可疑文件  ★
0042750E   push drugdir.00468618                    w
00427522   push drugdir.004682B8                    \\n\\n
00427543   push drugdir.004682B8                    \\n\\n
004275A2   push drugdir.0046D1FC                    注册失败                   ★重要提示信息★
004275A7   push drugdir.0046D1D8                    注册失败！请认真核对你的注册码！             ★重要提示信息②★
004275B7   push drugdir.0046D1FC                    注册失败                   ★重要提示信息★
004275BC   push drugdir.0046D1AC                    暂停验证：你连续尝试 3 次注册码验证均未成功！       ★重要提示信息③★
00427615   push drugdir.0046D25C                    C:\\
0042762B   push drugdir.0046D258                    %ld
004276B7   mov dword ptr ds:[esi+7C],drugdir.004593  ⑷D
004276EE   mov dword ptr ds:[esi+1A0],drugdir.00459  ㄈD
00427708   mov dword ptr ds:[esi+1DC],drugdir.00459  ⑷D
........
```

在“★重要提示信息★①、②、③”处分别双击，然后在可疑的地址`F2`下断：

```
00427450    6A FF                push -1                          ; 我下断在此，F9 运行，填写注册相关信息！ ^_^
00427452    68 A85D4500          push drugdir.0045DA8
00427457    64:A1 00000000       mov eax,dword ptr fs:[0]
0042745D    50                   push eax                         ; eax=0012DB1C
0042745E    64:8925 00000000     mov dword ptr fs:[0],esp
00427465    83EC 10              sub esp,10                       ; esp=0012D9F0
00427468    A1 1CD64600          mov eax,dword ptr ds:[46D61C]
0042746D    53                   push ebx
0042746E    55                   push ebp                         ; ebp=0012DA08
0042746F    56                   push esi
00427470    57                   push edi                         ; edi=0012EA6C
00427471    8BF1                 mov esi,ecx                      ; ecx=0012EA6C，esi=00458908
00427473    894424 10            mov dword ptr ss:[esp+10],eax    ; eax=0046D630，堆栈 ss:[0012D9E0]=00000111
00427477    6A 01                push 1
00427479    C74424 2C 00000000   mov dword ptr ss:[esp+2C],0
00427481    E8 F1D80100          call drugdir.0044D77             ; 取用户名
00427486    51                   push ecx                         ; ecx=0012D9F0
00427487    8D96 00020000        lea edx,dword ptr ds:[esi+200]   ; 从用户名第 2 位开始取字符，edx=009746A9, (ASCII "uNgBiM")
0042748D    8BCC                 mov ecx,esp                      ; esp=0012D9CC，ecx=0012D9F0
0042748F    896424 20            mov dword ptr ss:[esp+20],esp
00427493    52                   push edx                         ; edx=0012EC6C
00427494    E8 4FEA0100          call drugdir.00445EE8            ; 取机器码
00427499    51                   push ecx                         ; ecx=009740AC
0042749A    8DBE 04020000        lea edi,dword ptr ds:[esi+204]   ; edi=0012EA6C
004274A0    8BCC                 mov ecx,esp                      ; esp=0012D9C8，ecx=009740AC
004274A2    896424 20            mov dword ptr ss:[esp+20],esp    ; esp=0012D9C8，堆栈 ss:[0012D9E8]=00090408
004274A6    57                   push edi                         ; edi=0012EC70
004274A7    C64424 34 01          mov byte ptr ss:[esp+34],1
004274AC    E8 37EA0100          call drugdir.00445EE8            ; 取注册码
004274B1    51                   push ecx                         ; ecx=00973FBC
004274B2    8DAE 08020000        lea ebp,dword ptr ds:[esi+208]   ; 堆栈地址=0012EC74，ebp=0012DA08
004274B8    8BCC                 mov ecx,esp                      ; esp=0012D9C4，ecx=00973FBC
004274BA    896424 20            mov dword ptr ss:[esp+20],esp    ; esp=0012D9C4，堆栈 ss:[0012D9E4]=00000001
004274BE    55                   push ebp                         ; ebp=0012EC74
004274BF    C64424 38 02          mov byte ptr ss:[esp+38],2
004274C4    E8 1FEA0100          call drugdir.00445EE8            ; 把注册信息数据依次存放起来，准备下一步计算
004274C9    8BCE                 mov ecx,esi                      ; esi=0012EA6C，ecx=0097469C
004274CB    C64424 34 00          mov byte ptr ss:[esp+34],0
004274D0    E8 0BFDFFFF          call drugdir.004271E0            ; ★算法 CALL★ F7 跟进！
004274D5    85C0                 test eax,eax
004274D7    8B46 5C              mov eax,dword ptr ds:[esi+5C]
004274DA    0F84 BB000000        je drugdir.0042759B              ; 注册验证失败则跳！
004274E0    83F8 03              cmp eax,3                        ; 比较是否连续 3 次注册验证失败
```



004274E3 0F8D CC000000 jge drugdir.004275B5 ; 如果大于等于 3 次则停止注册！★调试的时候先把这里 nop 掉！★

004274E9 6A 40 push 40

004274EB 68 4CD24600 push drugdir.0046D24C

004274F0 68 14D24600 push drugdir.0046D214

004274F5 8BCE mov ecx,esi

004274F7 E8 1FD10100 call drugdir.0044461B ; 注册成功后信息写入 ini 文件

004274FC 68 08D24600 push drugdir.0046D208 ; ASCII "drugreg.ini"

00427501 8D4C24 14 lea ecx,dword ptr ss:\[esp+14\]

00427505 E8 F2ED0100 call drugdir.004462FC

0042750A 8B4424 10 mov eax,dword ptr ss:\[esp+10\]

0042750E 68 18864600 push drugdir.00468618

00427513 50 push eax

00427514 E8 E0970000 call drugdir.00430CF9

00427519 8BD8 mov ebx,eax

0042751B 83C4 08 add esp,8

0042751E 85DB test ebx,ebx

00427520 74 67 je short drugdir.00427589

00427522 68 B8824600 push drugdir.004682B8 ; ASCII "" ★这里是读取用户名的地址★ 想留名的就在这里弄吧！

00427527 8D4C24 20 lea ecx,dword ptr ss:\[esp+20\]

0042752B 55 push ebp

0042752C 51 push ecx

0042752D E8 D6EE0100 call drugdir.00446408

00427532 57 push edi

00427533 8D5424 1C lea edx,dword ptr ss:\[esp+1C\]

00427537 50 push eax

00427538 52 push edx

00427539 C64424 34 03 mov byte ptr ss:\[esp+34\],3

0042753E E8 5FEE0100 call drugdir.004463A2

00427543 68 B8824600 push drugdir.004682B8 ; ASCII "" ★这里是读取注册码的地址★ 想留名的就在这里弄吧！

00427548 50 push eax

00427549 8D4424 1C lea eax,dword ptr ss:\[esp+1C\]

0042754D C64424 30 04 mov byte ptr ss:\[esp+30\],4

00427552 50 push eax

........

\================================= 跟进 004274D0 E8 0BFDFFFF call drugdir.004271E0 ============================

004271E0 6A FF push -1

004271E2 68 705D4500 push dumped\_1.00455D70

004271E7 64:A1 00000000 mov eax,dword ptr fs:\[0\]

004271ED 50 push eax

004271EE 64:8925 00000000 mov dword ptr fs:\[0\],esp

004271F5 83EC 70 sub esp,70

004271F8 53 push ebx

004271F9 55 push ebp

004271FA 56 push esi

004271FB 57 push edi

004271FC 33ED xor ebp,ebp

004271FE 89AC24 88000000 mov dword ptr ss:\[esp+88\],ebp

00427205 A1 1CD64600 mov eax,dword ptr ds:\[46D61C\]

0042720A 894424 10 mov dword ptr ss:\[esp+10\],eax

0042720E 8D8C24 98000000 lea ecx,dword ptr ss:\[esp+98\]

00427215 8D5424 18 lea edx,dword ptr ss:\[esp+18\]

00427219 51 push ecx

0042721A 68 A4D14600 push dumped\_1.0046D1A4

0042721F 52 push edx

00427220 C68424 94000000 03 mov byte ptr ss:\[esp+94\],3

00427228 E8 4FF20100 call dumped\_1.0044647C ; ★机器码运算 CALL★

0042722D 68 9CD14600 push dumped\_1.0046D19C

00427232 50 push eax

00427233 8D4424 1C lea eax,dword ptr ss:\[esp+1C\]

00427237 B3 04 mov bl,4

00427239 50 push eax

0042723A 889C24 94000000 mov byte ptr ss:\[esp+94\],bl

00427241 E8 C2F10100 call dumped\_1.00446408

00427246 50 push eax

00427247 8D8C24 94000000 lea ecx,dword ptr ss:\[esp+94\]

0042724E C68424 8C000000 05 mov byte ptr ss:\[esp+8C\],5

00427256 E8 30F30100 call dumped\_1.0044658B ; ★用户名运算 CALL★

0042725B 8D4C24 14 lea ecx,dword ptr ss:\[esp+14\]

0042725F 889C24 88000000 mov byte ptr ss:\[esp+88\],bl

00427266 E8 08EF0100 call dumped\_1.00446173

0042726B 8D4C24 18 lea ecx,dword ptr ss:\[esp+18\]

0042726F C68424 88000000 03 mov byte ptr ss:\[esp+88\],3

00427277 E8 F7EE0100 call dumped\_1.00446173

0042727C 8BBC24 90000000 mov edi,dword ptr ss:\[esp+90\]

00427283 83C9 FF or ecx,FFFFFFFF ; ecx=7FFDE000

00427286 33C0 xor eax,eax ; eax 清零

00427288 8D5424 1C lea edx,dword ptr ss:\[esp+1C\] ; 堆栈地址=0012D950，edx=004733B0

0042728C F2:AE repne scas byte ptr es:\[edi\] ; ★★★注册码就从这里开始计算了★★★

; ecx=FFFFFFFF (十进制 4294967295.)

0042728E F7D1 not ecx ; ecx 取反，ecx=FFFFFFE4

00427290 2BF9 sub edi,ecx ; ecx=0000001B，edi=00974763

00427292 8BC1 mov eax,ecx ; ecx=0000001B，eax=00000000

00427294 8BF7 mov esi,edi ; edi=00974748，esi=0012EA6C

00427296 8BFA mov edi,edx ; edx=0012D950，edi=00974748

00427298 C1E9 02 shr ecx,2 ; ecx=0000001B

0042729B F3:A5 rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\] ; ecx=00000006 (十进制 6.)

; ds:\[esi\]=\[00974748\]=674E754B

; es:\[edi\]=stack \[0012D950\]=000A054E

0042729D 8BC8 mov ecx,eax ; eax=0000001B

0042729F 33C0 xor eax,eax

004272A1 83E1 03 and ecx,3 ; ecx=0000001B

004272A4 F3:A4 rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\] ; ecx=00000003 (十进制 3.)

; ds:\[esi\]=\[00974760\]=CA

; es:\[edi\]=stack \[0012D968\]=5B ('\[')

004272A6 8D7C24 1C lea edi,dword ptr ss:\[esp+1C\]

004272AA 83C9 FF or ecx,FFFFFFFF ; ecx=00000000

004272AD 33F6 xor esi,esi ; esi=00974763

004272AF F2:AE repne scas byte ptr es:\[edi\] ; ecx=FFFFFFFF (十进制 4294967295.)

004272B1 F7D1 not ecx ; ecx 取反，ecx=FFFFFFE4

004272B3 49 dec ecx ; ecx=0000001B

004272B4 0F84 CD000000 je dumped\_1.00427387 ; ★★注册验证、重启验证爆破点★★

004272BA 8D7C24 1C lea edi,dword ptr ss:\[esp+1C\]

004272BE 83C9 FF or ecx,FFFFFFFF ; ecx=0000001A

004272C1 33C0 xor eax,eax

004272C3 0FBE5434 1C movsx edx,byte ptr ss:\[esp+esi+1C\] ; 堆栈 ss:\[0012D950\]=4B ('K')，edx=0012D950

004272C8 F2:AE repne scas byte ptr es:\[edi\] ; ecx=FFFFFFFF (十进制 4294967295.)

004272CA F7D1 not ecx ; ecx 取反，ecx=FFFFFFE4

004272CC 49 dec ecx ; ecx=0000001B

004272CD 8BC1 mov eax,ecx ; ecx=0000001A



```
004272CF    8D0CD2          lea ecx,dword ptr ds:[edx+edx*8]       ; ecx=edx*8+edx=2A3   (注意：edx=4B ('K'))
004272D2    8D0CC9          lea ecx,dword ptr ds:[ecx+ecx*8]       ; ecx=ecx*8+ecx=17BB
004272D5    8D0C4A          lea ecx,dword ptr ds:[edx+ecx*2]       ; ecx=ecx*2+edx=2FC1
004272D8    8D0C8A          lea ecx,dword ptr ds:[edx+ecx*4]       ; ecx=ecx*4+edx=BF4F
004272DB    8D0C4A          lea ecx,dword ptr ds:[edx+ecx*2]       ; ecx=ecx*2+edx=17EE9
004272DE    2BCE            sub ecx,esi                             ; ecx=ecx-esi=17EE9
004272E0    03C1            add eax,ecx                             ; ecx=eax+ecx=1A+17EE9=17F03
004272E2    8D0C52          lea ecx,dword ptr ds:[edx+edx*2]       ; ecx=edx*2+edx=E1    (注意：edx=4B ('K'))
004272E5    8D1489          lea edx,dword ptr ds:[ecx+ecx*4]       ; ecx=ecx*4+ecx=465
004272E8    B9 5B000000     mov ecx,5B                              ; ecx=E1
004272ED    33C2            xor eax,edx                             ; edx=465，eax=17F03
004272EF    33D2            xor edx,edx                             ; edx=465
004272F1    F7F1            div ecx                                 ; ecx=5B
004272F3    83FA 30         cmp edx,30                              ; edx=1D
004272F6    7C 05           jl short dumped_1.004272FD
004272F8    83FA 39         cmp edx,39
004272FB    7E 0A           jle short dumped_1.00427307
004272FD    83FA 41         cmp edx,41                              ; edx=1D
00427300    7C 12           jl short dumped_1.00427314
00427302    83FA 5A         cmp edx,5A
00427305    7F 0D           jg short dumped_1.00427314
00427307    52              push edx
00427308    8D5424 14       lea edx,dword ptr ss:[esp+14]
0042730C    68 98D14600     push dumped_1.0046D198                  ; ASCII "%c"
00427311    52              push edx
00427312    EB 0B           jmp short dumped_1.0042731F
00427314    52              push edx                                ; edx=1D
00427315    8D4424 14       lea eax,dword ptr ss:[esp+14]           ; eax=42B
00427319    68 94D14600     push dumped_1.0046D194                  ; ASCII "%d"
0042731E    50              push eax                                ; eax=0012D944
0042731F    E8 EA970100     call dumped_1.00440B0E
00427324    8B4C24 1C       mov ecx,dword ptr ss:[esp+1C]           ; ecx=00974798, (ASCII "29")
00427328    83C4 0C         add esp,0C
0042732B    8D5424 18       lea edx,dword ptr ss:[esp+18]           ; esp=0012D928
0042732F    8B41 F8         mov eax,dword ptr ds:[ecx-8]           ; edx=00974799
00427332    8D8C24 94000000 lea ecx,dword ptr ss:[esp+94]
00427339    50              push eax                                ; eax=00000002
0042733A    55              push ebp
0042733B    52              push edx                                ; edx=0012D94C
0042733C    E8 BD920100     call dumped_1.004405FE
00427341    8B00            mov eax,dword ptr ds:[eax]
00427343    50              push eax                                ; eax=00974068, (ASCII "98")
00427344    8B4424 14       mov eax,dword ptr ss:[esp+14]           ; 堆栈 ss:[0012D944]=00974798, (ASCII "29")
                                                                ; eax=00974068, (ASCII "98")
00427348    50              push eax                                ; eax=00974798, (ASCII "29")
00427349    E8 F1950000     call dumped_1.0043093F
0042734E    83C4 08         add esp,8                               ; esp=0012D92C
00427351    8D4C24 18       lea ecx,dword ptr ss:[esp+18]           ; ecx=00000019
00427355    85C0            test eax,eax                            ; eax=FFFFFFFF
00427357    0F95C3          setne bl                                ; 条件为真 TRUE，bl=04
0042735A    E8 14EE0100     call dumped_1.00446173
0042735F    84DB            test bl,bl                              ; bl=01
00427361    0F85 8D000000   jnz dumped_1.004273F4
00427367    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0042736B    8D7C24 1C       lea edi,dword ptr ss:[esp+1C]
0042736F    8B41 F8         mov eax,dword ptr ds:[ecx-8]
00427372    83C9 FF         or ecx,FFFFFFFF
00427375    03E8            add ebp,eax
00427377    33C0            xor eax,eax
00427379    46              inc esi
0042737A    F2:AE           repne scas byte ptr es:[edi]
0042737C    F7D1            not ecx
0042737E    49              dec ecx
0042737F    3BF1            cmp esi,ecx
00427381    0F82 33FFFFFF   jb dumped_1.004272BA
00427387    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
0042738B    C68424 88000000 02 mov byte ptr ss:[esp+88],2
00427393    E8 DBED0100     call dumped_1.00446173
00427398    8D8C24 90000000 lea ecx,dword ptr ss:[esp+90]
0042739F    C68424 88000000 01 mov byte ptr ss:[esp+88],1
004273A7    E8 C7ED0100     call dumped_1.00446173
004273AC    8D8C24 94000000 lea ecx,dword ptr ss:[esp+94]
004273B3    C68424 88000000 00 mov byte ptr ss:[esp+88],0
004273BB    E8 B3ED0100     call dumped_1.00446173
004273C0    8D8C24 98000000 lea ecx,dword ptr ss:[esp+98]
004273C7    C78424 88000000 FFFFF mov dword ptr ss:[esp+88],-1
004273D2    E8 9CED0100     call dumped_1.00446173
004273D7    B8 01000000     mov eax,1
004273DC    8B8C24 80000000 mov ecx,dword ptr ss:[esp+80]
004273E3    5F              pop edi
004273E4    5E              pop esi
004273E5    5D              pop ebp
004273E6    5B              pop ebx
004273E7    64:890D 00000000 mov dword ptr fs:[0],ecx
004273EE    83C4 7C         add esp,7C
004273F1    C2 0C00         retn 0C                                 ; 返回程序
```

### 完美注册验证爆破点

原文：
```
004272B4    0F84 CD000000     je dumped_1.00427387                ; je 改 jnz
```

修改后：
```
004272B4    0F85 CD000000     jnz dumped_1.00427387
```

### 破解总结

本文适合中等 Cracker 练手，难点主要是在解除程序校验部分，软件调用十一次自校验，往往不注意就 Over 了，本文主要是采用了暴力破解，不过还是属于比较完美的爆破，呵呵~~至于算法部分，运算太多了，我懒得总结了，上面我已经写得比较清楚了。

有兴趣的朋友可以详细看看！

--------------------------------------------------------------------------------------------
版权所有（C）2005 `KuNgBiM[DFCG]` &nbsp;&nbsp;&nbsp;&nbsp; `Copyright (C) 2005 KuNgBiM[DFCG]`

--------------------------------------------------------------------------------------------

`Cracked By KuNgBiM[DFCG]`
2005-07-23
12:09:18 PM

`4dd466d0e1e0cc123afbe504da2462ca`
