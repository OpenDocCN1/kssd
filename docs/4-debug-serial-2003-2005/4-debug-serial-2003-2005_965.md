
# 超简单算法--长坤立体大师 V3.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-92.htm`](https://www.pediy.com/kssd/pediy07/pediy7-92.htm)

**【软件信息】**

- **大小**：`314` `KB`
- **语言**：简体中文
- **类别**：国产软件 / 共享版 / 图像处理
- **平台**：`Win9x`/`NT`/`2000`/`XP`
- **时间**：`2005-02-20 11:40:16`
- **下载**：`http://www2.skycn.com/soft/22105.html`

**【软件介绍】**

a) 可将普通照片快速加工成立体照片;

b) 可以即时观看立体效果，直接打印或存盘输出;

c) 使用简单，功能强大，立体效果好。

不需要立体模板，不需要立体相机不用专业技术，会上网者半小时可学会。

**【作者声明】**

初学破解，仅作学习交流之用，失误之处敬请大侠赐教!

**【破解工具】**

`Ollydbg1.10`、`WDasmv10.0`

这个软件的算法极其简单，但由于涉及到一些浮点指令，还是贴出来，当作是对浮点指令的一个温习吧! 如果你熟悉浮点指令就不要再往下看了!

用 `PEid` 检测无壳，为 `Dephi` 编写! 首先运行程序，输入密码：`138651` (电脑编号为自动生成为 `68295`)，点注册出错! 用 `WDasm` 反汇编查找字串参考下断，记下断点，`OD` 载入来到如下代码:

```
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

..........
004958ED  |>LEA EDX,DWORD PTR SS:\[EBP\-1D4\]   ;  我们在此下断
004958F3  |>MOV EAX,DWORD PTR DS:\[EBX+2F0\]
004958F9  |>CALL photo3d.00441CB0
004958FE  |>MOV EAX,DWORD PTR SS:\[EBP\-1D4\]   ;  到这里 EAX="68295",我的电脑编号
00495904  |>CALL photo3d.00409078               ;  这个 Call 将我的电脑编号转换成 Hex 值
00495909  |>MOV DWORD PTR SS:\[EBP\-1D8\],EAX   ;  这里可看到 EAX=10AC7(68295 的 Hex 形式)
0049590F  |>FILD DWORD PTR SS:\[EBP\-1D8\]      ;  浮点指令,把 10AC7 的整数形式(即 68295)装入到 st0 中
00495915  |>FDIV DWORD PTR DS:\[495A4C\]   ;DS:\[495A4C\]中为定值 1000000,68295/1000000->st0=0.068295
0049591B  |>CALL photo3d.004029B8               ;  浮点堆栈操作
00495920  |>FMUL DWORD PTR DS:\[495A50\]       ;  DS:\[495A50\]中为 10000,st0*10000->st0=682.95
00495926  |>FADD DWORD PTR DS:\[495A54\]       ;  DS:\[495A4C\]中为固定值 1980,1980+st0=2662.95->st0
0049592C  |>CALL photo3d.004029E8              ; 这个 Call 实现 Dec->Hex,EAX 中返回 0A66(2662 的 Hex 值)
00495931  |>CMP EDX,DWORD PTR SS:\[ESP+4\]     ;  这个比较不清楚干什么的
00495935  |>JNZ SHORT photo3d.0049593A
00495937  |>CMP EAX,DWORD PTR SS:\[ESP\]       ;  真假码比较,EAX 中为真码的 Hex 值
0049593A  |>POP EDX
0049593B  |>POP EAX
0049593C  |>JNZ photo3d.00495A01             ;  关键跳转
..........

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
```

是不是很简单，让我们再看看电脑编号是怎样产生的，反汇编时看 Import 函数时我们注意到有 `GetVolumeInformationA` 函数，有门，用这个函数下断，`OD` 载入，中断，先不要理他，`F9` 过，出现启动画面后马上又中断，OK! 我们断下来，看到以下代码:

```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

0049D32A  .>MOV EBX,EAX
0049D32C  .>PUSH 0                          ; /pFileSystemNameSize = NULL
0049D32E  .>PUSH 0                          ; |pFileSystemNameBuffer = NULL
0049D330  .>LEA EAX,DWORD PTR SS:\[ESP+C\]      ; |
0049D334  .>PUSH EAX                      ; |pFileSystemFlags
0049D335  .>LEA EAX,DWORD PTR SS:\[ESP+C\]      ; |
0049D339  .>PUSH EAX                      ; |pMaxFilenameLength
0049D33A  .>PUSH EBX                      ; |pVolumeSerialNumber
0049D33B  .>PUSH 0                          ; |MaxVolumeNameSize = 0
0049D33D  .>PUSH 0                          ; |VolumeNameBuffer = NULL
0049D33F  .>PUSH photo3d.0049D350           ; |RootPathName = "c:\\"
0049D344  .>CALL <JMP.&kernel32.GetVolumeInformat>; \GetVolumeInformationA
0049D349  .>MOV EAX,DWORD PTR DS:\[EBX\]        ;  EAX=14531ADA,我的硬盘序列号
0049D34B  .>POP ECX
0049D34C  .>POP EDX
0049D34D  .>POP EBX
0049D34E  \.>RETN
0049D34F   >DB 00
0049D350  .>ASCII "c:\\",0
0049D354  .>MOV DWORD PTR DS:\[4A0F00\],32
```

取得我的硬盘序列号，`RETN` 后返回后来到这里：

```
00495B56  |>MOV DWORD PTR SS:\[EBP\-C\],EAX
00495B59  |>XOR EAX,EAX
00495B5B  |>MOV DWORD PTR SS:\[EBP\-8\],EAX
00495B5E  |>FILD QWORD PTR SS:\[EBP\-C\]        ;  不用再解释了吧,340990682(Dec 值)-->st0
00495B61  |>FDIV DWORD PTR DS:\[495D0C\]  ;DS:\[495D0C\]中为定值 10000,st0=340990682/10000=34099.0682
00495B67  |>CALL photo3d.004029B8               ;  这个 Call 我们也跟进去看看吧!

-----------------------------------------------------------------------------------

   004029B8  />FLD ST                          ;  st0-->st1?,寄存器区可看到   st0=st1=34099.0682
   004029BA  |>SUB ESP,4
   004029BD  |>FSTCW WORD PTR SS:\[ESP\]       ;  将 FPU 的控制字保存到 SS:\[ESP\]
   004029C0  |>FSTCW WORD PTR SS:\[ESP+2\]
   004029C4  |>WAIT
   004029C5  |>OR WORD PTR SS:\[ESP+2\],0F00   ;  不知作什么用?
   004029CC  |>FLDCW WORD PTR SS:\[ESP+2\]     ;  从 SS:\[ESP+2\]装入 FPU 的控制字
   004029D0  |>FRNDINT                      ;  st0 取整,st0=34099
   004029D2  |>WAIT
   004029D3  |>FLDCW WORD PTR SS:\[ESP\]
   004029D6  |>ADD ESP,4
   004029D9  |>FSUBP ST(1),ST                ;st0=st1-st0=34099.0682-34099=0.0682,然后执行一次出栈
   004029DB  \>RETN

-------------------------------------------------------------------------------------

00495B6C  |>FMUL DWORD PTR DS:\[495D10\]       ; DS:\[495D10\]中为定值 1000000,st0=st0*1000000=68200
00495B72  |>CALL photo3d.004029E8              ; 同前,这个 Call 实现 Dec->Hex,EAX 中返回为 10A68
00495B77  |>PUSH EDX
00495B78  |>PUSH EAX
00495B79  |>MOV EAX,DWORD PTR DS:\[4A0EE8\]    ; DS:\[4A0EE8\]中为 5F,定值
00495B7E  |>CDQ
00495B7F  |>ADD EAX,DWORD PTR SS:\[ESP\]       ; SS:\[ESP\]中为上面算得的值,EAX=10A68+5F=10AC7
00495B82  |>ADC EDX,DWORD PTR SS:\[ESP+4\]
00495B86  |>ADD ESP,8
00495B89  |>PUSH EDX                         ; /Arg2
00495B8A  |>PUSH EAX                         ; |EAX=10AC7
00495B8B  |>LEA EAX,DWORD PTR SS:\[EBP\-4\]     ; |作 Buffer 用,Call 后返回值就在这里面
00495B8E  |>CALL photo3d.00409044             ; \\同理,这个 Call 将 Hex-->Dec
00495B93  |>MOV EDX,DWORD PTR SS:\[EBP\-4\]     ;  SS:\[EBP-4\]中为"68295",我的电脑编号哦!
00495B96  |>MOV EAX,DWORD PTR DS:\[EBX+2F0\]

.........
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

**【总结】**

软件的算法太简单，由我们的硬盘序列号 `a` (`Dec` 进制值)-->`a/10000=b`-->取 `b` 的小数部分 `c`-->`c*1000000=d`(`Hex` 进制值)-->`d+5F=e`(`Dec` 进制值)，然后由生成的电脑编号 `e`-->`e/100+1980=f` 再取整就是正确的注册码了!

有一点没弄明白，用 `GetWindowTextA` 不能下断，不清楚软件是怎么取得我们输入的假码的，懒得再跟了! 另外 `Dephi` 中取得字符串都有哪些方法啊，有知道的兄弟贡献出来啊!

欢迎 e-mail 到 `sharpair@163.com` 交流!

`4dd466d0e1e0cc123afbe504da2462ca`
