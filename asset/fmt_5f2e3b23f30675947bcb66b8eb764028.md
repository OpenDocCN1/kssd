
# BusinessSkinForm 安装破解笔记

> 原文：[`www.pediy.com/kssd/pediy11/116622.html`](https://www.pediy.com/kssd/pediy11/116622.html)

备忘笔记，别无他意。。。

在使用 BCB6.0 找到这个控件，在官网上下载来看看。。。
觉得还可以不过安装有点问题要
在组件那里安装后要再将相关的文件拷到相关目录里。。。

* `*.hpp` ==> bcb 的 include
* `BSFCB6.lib` ==> `D:\Program Files\Borland\CBuilder6\Lib`
* `BSFCB6.bpl` ==> `D:\Program Files\Borland\CBuilder6\Projects\Bpl`

具体请网上查找。。。不详细说明。

安装成功

运行实例。跳出 `This application use unregistered BusinessSkinForm VCL!!!` 的提示。。

## 一。网上查找控件应该是破解 bpi 或是 dcu 文件

破解过程中发现
先对 `BSFCB6.bpi` 进行分析发面 其实是一个 dll 文件用 OD 载入，并发现上述标示

```
004302C6   .  E8 856A1C00   CALL  MainDemo.005F6D50
004302CB   .  85C0          TEST  EAX,EAX
004302CD   .  75 18         JNZ   SHORT MainDemo.004302E7   ;重要部位
004302CF   .  6A 10         PUSH  10
004302D1   .  B9 48034300   MOV   ECX,MainDemo.00430348                  ;  sorry
004302D6   .  BA 50034300   MOV   EDX,MainDemo.00430350                  ;  this application use unregistered businessskinform vcl!!!
004302DB   .  A1 84866600   MOV   EAX,DWORD PTR [668684]
004302E0   .  8B00          MOV   EAX,DWORD PTR [EAX]
004302E2   .  E8 79621F00   CALL  MainDemo.00626560
004302E7   >  33C0          XOR   EAX,EAX
004302E9   .  5A            POP   EDX
004302EA   .  59            POP   ECX
004302EB   .  59            POP   ECX
004302EC   .  64:8910       MOV   DWORD PTR FS:[EAX],EDX
004302EF   .  68 09034300   PUSH  MainDemo.00430309
004302F4   >  8D45 F0       LEA   EAX,DWORD PTR [EBP-10]
004302F7   .  BA 03000000   MOV   EDX,3
004302FC   .  E8 7F641C00   CALL  MainDemo.005F6780
00430301   .  C3            RETN
00430302   .  E9 995E1C00   JMP   MainDemo.005F61A0
00430307   .  ^ EB EB       JMP   SHORT MainDemo.004302F4
00430309   .  8A45 FF       MOV   AL,BYTE PTR [EBP-1]
0043030C   .  8BE5          MOV   ESP,EBP
0043030E   .  5D            POP   EBP
0043030F   .  C3            RETN
00430310   .  FFFFFFFF      DD    FFFFFFFF
00430314   .  07000000      DD    00000007
00430318   .  62 64 73 2E 6>ASCII "bds.exe",0
00430320   .  FFFFFFFF      DD    FFFFFFFF
00430324   .  0C000000      DD    0000000C
00430328   .  64 65 6C 70 6>ASCII "delphi32.exe",0
00430335      00            DB    00
00430336      00            DB    00
00430337      00            DB    00
00430338   .  FFFFFFFF      DD    FFFFFFFF
0043033C   .  07000000      DD    00000007
00430340   .  62 63 62 2E 6>ASCII "bcb.exe",0
00430348   .  53 6F 72 72 7>ASCII "Sorry",0
0043034E      00            DB    00
0043034F      00            DB    00
00430350   .  54 68 69 73 2>ASCII "This application"
00430360   .  20 75 73 65 2>ASCII " use unregistere"
00430370   .  64 20 42 75 7>ASCII "d BusinessSkinFo"
00430380   .  72 6D 20 56 4>ASCII "rm VCL!!!",0
```

看红色的部位。发现其实这二个地方，挨得很近。。。。

关键码是 `85 C0 75 18 6A 10`

别的那些 call 数除了前面一个字节是准的外，别的要重定位。。。
所以确定的方法是查找 `85 c0 75 18 6a 10` (B9)
然后后面不远可以找到 `This application use unregistered BusinessSkinForm VCL!!!`
我不算其具体距离。不过可以肯定这个距离是固定的。。。有兴趣可以计算。

改好后以为破解成功，运行发现不行，提示还是出现！！！！。。。
找 `dcu` 也没有找到。。。。
破解暂时进行迷局之中。。。

## 二。

在相关的 `HPP` 文件也没有找到相关数据。
唯一的方法笨也比较实用的方法。
下载一个 `filemon` 软件，监控文件操作。。。因为我 `bcb` 及控件全在 D 盘。所在过滤中只留下 D 盘且只点掉 `OPEN` `write` 二个选项的。（控件文件使用过程不会修改）

开始临控（瑞星杀毒要关掉监控）==》重建工程==》 暂停监控。。。
一页一页的翻找发现建立过程中只读取 `*.hpp` `*.lib` 文件。。
其中属于 `bcb` 的文件排除，重点排查和控件有关的文件
`BSFCB6.lib`
`bs\*.hpp`

查看几个相关的 `hpp` 文件(用 `editplus` 搜索 `unreg` 找不到相应的字串）
排除了 `hpp` 中有注册提示的可能。

重点放在了 `BSFCB6.lib` 文件上。。。
不清这个是什么结构的文件，而且 `borland` 与 `MS` 可能不同。。。。
用 `winhex` 打开查找。真的找到了。。。相关字串。。。
原来在这。。。向上查看。。。发现 `85 C0 75 18 6A 10`
太高兴了。原来在这。。。。
修改吧。。。`75` 改成 `EB`
存盘。。。。
重构程序运行，没有提示。。。宣告破解成功。。
(第一次改成 `BE` 结果出现错误。。。后来发现再改才成功。。。)

## 再补一个 suipack 的破解过程

官网下载。。。安装。
也是在 `lib` 里破解。
`SUIPackC6.Lib`
`OD` 打一个新建立的程序
查找 `suipack trial edition`（标题名）字串找到相应的位置。。

```
00437DF0 > $  55            PUSH  EBP
00437DF1   .  8BEC          MOV   EBP,ESP
00437DF3   .  832D 1CF24300>SUB   DWORD PTR [$115678332],1
00437DFA   .  73 73         JNB   SHORT Project1.00437E6F              ;  估计这里是判断是否程序第一次运行
00437DFC   .  A1 243F4400   MOV   EAX,DWORD PTR [<&rtl60.@Classes@Curr>
00437E01   .  8338 FF       CMP   DWORD PTR [EAX],-1;是否注册
00437E04   .  75 69         jnz   SHORT Project1.00437E6F ;jmp 过去就可以成功了。
00437E06   .  33C9          XOR   ECX,ECX
00437E08   .  B2 01         MOV   DL,1
00437E0A   .  A1 9CB24000   MOV   EAX,DWORD PTR [40B29C]
00437E0F   .  E8 7443FDFF   CALL  Project1.@Suidlg@TsuiMessageDialog@>
00437E14   .  A3 20F24300   MOV   DWORD PTR [@Suiresdef@Msg],EAX
00437E19   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E1E   .  83C0 68       ADD   EAX,68
00437E21   .  BA 7C7E4300   MOV   EDX,Project1.00437E7C              ;  您使用的是 suipack 的试用版本!\n\n\n\n 如果您喜欢它，清注册。\n\n\n\n 注册后可以得到免费升级服务。\n\n\n\n 请访问东日软件中文网站 http://www.sunisoft.cn/suipack/\n\support@sunisoft.cn
00437E26   .  E8 E9540000   CALL  <JMP.&rtl60.@System@@LStrAsg$qqrpvp>
00437E2B   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E30   .  C640 48 01    MOV   BYTE PTR [EAX+48],1
00437E34   .  B2 01         MOV   DL,1
00437E36   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E3B   .  E8 AC42FDFF   CALL  Project1.@Suidlg@TsuiDialog@SetUISt>
00437E40   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E45   .  C640 64 03    MOV   BYTE PTR [EAX+64],3
00437E49   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E4E   .  83C0 34       ADD   EAX,34
00437E51   .  BA 2C7F4300   MOV   EDX,Project1.00437F2C              ;  suipack trial edition
00437E56   .  E8 B9540000   CALL  <JMP.&rtl60.@System@@LStrAsg$qqrpvp>
00437E5B   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E60   .  8B10          MOV   EDX,DWORD PTR [EAX]
00437E62   .  FF52 30       CALL  NEAR DWORD PTR [EDX+30]
00437E65   .  A1 20F24300   MOV   EAX,DWORD PTR [@Suiresdef@Msg]
00437E6A   .  E8 35550000   CALL  <JMP.&rtl60.@System@TObject@Free$qq>
00437E6F   >  5D            POP   EBP
00437E70   .  C3            RETN
00437E04   .  75 69         jnz   SHORT Project1.00437E6F ;jmp 过去就可以成功了。
00437E51   .  BA 2C7F4300   MOV   EDX,Project1.00437F2C              ;  suipack trial edition
```

二个关键点：

地址一：`00437E04` 跳转地址。
地址二：`Project1.00437F2C` ; `suipack trial edition` 字串所在位址。

偏移值=地址二-地址一=`0x0128`
打找 `lib` 文件。查找 `suipack trial edition` 得 新地址二 `1897760`
这一值减去 `0x0128`（16 进制 C 语言表达法）
得 `1897464`
查看这一位置刚好是 `75 69`
修改 `75`==》`EB` 存盘，重建，成功。。。

## 总结一下吧

对于 `delphi` 和 `bcb` 的控件其 `lib` 文件和生成的程序中关于注册的代码是类似的，是类似不是相同。。。。
而且不可能相同，因为涉及 `winapi` 及相关 `VCL` 操作或是语言内置函数都要重定位。。。
而短跳，如 `push eax` 操作为寄存器的，则代码是相同。
立即数相关的一般不同。
这二次都遇到有个 `jnz` 可以改成 `jmp` 跳过的。。。如果没有这个语句的就要自己构造了。。。

如何构造，要注意函数调用压入参数前跳过如以下

```
push eax
push edx
push ecx
call xxxxx
```

假设有三个参数。在压第一个参数地方就要 `jmp` 不能简单的 `nop` 掉 `call`
