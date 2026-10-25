
# 推箱子 V2.5

> 原文：[`www.pediy.com/kssd/pediy06/pediy6847.htm`](https://www.pediy.com/kssd/pediy06/pediy6847.htm)

学习了 `Win32Asm` 汇编，终觉心痒痒，拿几个程序开刀玩玩，这只是个人心得，写得不好请不要笑本生。

此前破了几个程序，计有玄奥四柱推命 `V3.2`、紫微斗数 `V3.5`、起名向导 `V5.3`、`ACCESS` 密码克星、`Filter Solutions Ver 10.0` 滤波器设计、`Smith V2.01` 史密斯圆图工具，此次又拿蒋刚的推箱子 `V2.5` 开刀（主要看不惯“禁止破解，否则后果自负！”字样），先前看过 `CrackerABC[BCG]` 的推箱子 `V1.9` 的破解，终觉不完美，现写出我的破解方法供参考。

此程序有自校验，采用暴破法还需破掉自校验代码。为此我采用完美破解法，即不修改原程序。

用 `IDA` 反汇编后分析，找出读文件的地方，发现读 `cargador.cfg` 文件的地方比较可疑（这里用 `Ollydbg` 反汇编代替）。

```
0040162A  .  B8 38244000      MOV EAX,cargador.00402438            ;  ASCII "cargador.cfg"
0040162F  .  E8 D4A20500      CALL cargador.0045B908
00401634  .  84C0             TEST AL,AL
00401636  .  0F84 75010000    JE cargador.004017B1                 ; 如果 AL=0 则文件不存在
```

好，在磁盘上程序运行目录建立一 `cargador.cfg` 长度为 10 个字节的二进制文件(全部填充 `FF`)，再运行程序，提示文件已经 `end of file`。

用 `Ollydbg` 跟踪，到

```
0040165F  .  C745 FC 010000>  MOV DWORD PTR SS:[EBP-4],1          ; 存放读取字节初值
00401666  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00401669  |> 8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
00401670  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFile
00401675  .  E8 7A380600     CALL cargador.00464EF4              ; 读文件
0040167A  .  E8 290D0600     CALL cargador.004623A8              ; 检查异常
0040167F  .  FF45 FC        INC DWORD PTR SS:[EBP-4]             ; 下一个字节
00401682  .  817D FC 010200> CMP DWORD PTR SS:[EBP-4],201        ; 读取 201H-1 个字节
00401689  .  75 DB           JNZ SHORT cargador.00401666
0040168B  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
00401692  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00401695  .  8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
0040169C  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
004016A1  .  E8 4E380600     CALL cargador.00464EF4
004016A6  .  E8 FD0C0600     CALL cargador.004623A8
004016AB  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
004016AE  .  837D FC 11      CMP DWORD PTR SS:[EBP-4],11         ; 读取 11H-1 个字节
004016B2  .  75 DE           JNZ SHORT cargador.00401692
004016B4  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
004016BB  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004016BE  .  8D90 E3754A00   LEA EDX,DWORD PTR DS:[EAX+4A75E3]
004016C4  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
004016C9  .  E8 26380600     CALL cargador.00464EF4             ; 注意这个循环有两次读文件！
004016CE  .  E8 D50C0600     CALL cargador.004623A8
004016D3  .  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004016D6  .  8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
004016DD  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
004016E2  .  E8 0D380600     CALL cargador.00464EF4
004016E7  .  E8 BC0C0600     CALL cargador.004623A8
004016EC  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
004016EF  .  837D FC 05      CMP DWORD PTR SS:[EBP-4],5          ; 读取 2*(5-1)个字节
004016F3  .  75 C6           JNZ SHORT cargador.004016BB
004016F5  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
004016FC  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004016FF  .  8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
00401706  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
0040170B  .  E8 E4370600     CALL cargador.00464EF4
00401710  .  E8 930C0600     CALL cargador.004623A8
00401715  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
00401718  .  837D FC 09      CMP DWORD PTR SS:[EBP-4],9          ; 读取 9-1 个字节
0040171C  .  75 DE           JNZ SHORT cargador.004016FC
0040171E  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
00401725  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00401728  .  8D90 E7754A00   LEA EDX,DWORD PTR DS:[EAX+4A75E7]
0040172E  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
00401733  .  E8 BC370600     CALL cargador.00464EF4
00401738  .  E8 6B0C0600     CALL cargador.004623A8
0040173D  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
00401740  .  837D FC 07      CMP DWORD PTR SS:[EBP-4],7          ; 读取 7-1 个字节
00401744  .  75 DF           JNZ SHORT cargador.00401725
00401746  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
0040174D  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00401750  .  8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
00401757  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
0040175C  .  E8 93370600     CALL cargador.00464EF4
00401761  .  E8 420C0600     CALL cargador.004623A8
00401766  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
00401769  .  837D FC 0B      CMP DWORD PTR SS:[EBP-4],0B         ; 读取 0BH-1 个字节
0040176D  .  75 DE           JNZ SHORT cargador.0040174D
0040176F  .  C745 FC 010000> MOV DWORD PTR SS:[EBP-4],1
00401776  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00401779  .  8D9405 C4FCFFF> LEA EDX,DWORD PTR SS:[EBP+EAX-33C]
00401780  .  B8 88744A00     MOV EAX,OFFSET cargador.@Regist@SaveFil>
00401785  .  E8 6A370600     CALL cargador.00464EF4
0040178A  .  E8 190C0600     CALL cargador.004623A8
0040178F  .  FF45 FC        INC DWORD PTR SS:[EBP-4]
00401792  .  817D FC 010200> CMP DWORD PTR SS:[EBP-4],201        ; 读取 201H-1 个字节
00401799  .  75 DB           JNZ SHORT cargador.00401776
```

.
.
.

`(201H-1)+(11H-1)+(2*(5-1))+(9-1)+(7-1)+(0B-1)+(201-1)=430H=1072D`

好，将 `cargador.cfg` 文件数增加到 1072 个字节(全部填 `FF`)，再试，`OK`。

继续往下走可看见

```
00401EEA  >  C605 84744A00 >  MOV BYTE PTR DS:[@Regist@Registered],0
```

搜索字符`@Regist@Registered`来到

```
00402087  .  C685 CCFEFFFF >  MOV BYTE PTR SS:[EBP-134],0
0040208E  .  C605 84744A00 >  MOV BYTE PTR DS:[@Regist@Registered],1 ; 先置为成功
00402095  .  A1 80744A00     MOV EAX,DWORD PTR DS:[@Regist@Reg_Form]
0040209A  .  8B80 F4020000   MOV EAX,DWORD PTR DS:[EAX+2F4]
004020A0  .  C740 0C 020000>  MOV DWORD PTR DS:[EAX+C],2
004020A7  .  C745 FC 010000>  MOV DWORD PTR SS:[EBP-4],1
004020AE  |> 8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004020B1  .  8A8405 C5FEFFF>  MOV AL,BYTE PTR SS:[EBP+EAX-13B]
004020B8  .  8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]
004020BB  .  3A8415 2AFFFFF>  CMP AL,BYTE PTR SS:[EBP+EDX-D6] ; 与 0012ADB7 处的内容比较
004020C2  .  74 23          JE SHORT cargador.004020E7      ; 不等就完蛋
                                ; (@Regist@Registered=0)
004020C4  .  C605 84744A00 >  MOV BYTE PTR DS:[@Regist@Registered],0
004020CB  .  C605 EE754A00 >  MOV BYTE PTR DS:[@Regist@ReadReg],0
004020D2  .  A1 80744A00     MOV EAX,DWORD PTR DS:[@Regist@Reg_Form]
004020D7  .  8B80 F4020000   MOV EAX,DWORD PTR DS:[EAX+2F4]
004020DD  .  33D2            XOR EDX,EDX
004020DF  .  8950 0C         MOV DWORD PTR DS:[EAX+C],EDX
004020E2  .  E9 1D030000     JMP cargador.00402404
004020E7  |> FF45 FC        INC DWORD PTR SS:[EBP-4]
004020EA  .  837D FC 07      CMP DWORD PTR SS:[EBP-4],7     ; 比较 6 次(从 1 开始)
004020EE  .  75 BE           JNZ SHORT cargador.004020AE
```

查看 `0012AD52` 处的内存情况

```
0012AD52  46 4E 56 60 6A 77 00 56 3F 3D 3B 39 37 35 00 00  FNV`jw.V?=;975..
0012AD62  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012AD72  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012AD82  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012AD92  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012ADA2  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012ADB2  00 00 00 00 06 9B 9B 9B 9B 9B 9B 00 00 00 00 00  ....FNV`jw.....
```

记下 `0012AD52` 处的连续 6 个字节（这里为 `46 4E 56 60 6A 77`）。

注意看到 `0012ADB7` 处的连续 6 个字节为 `9B`，就是我们的 `FF-64H`。

经过实验改变 `cargador.cfg` 文件的 `00000220H` 处的连续 6 个字节可以看到 `0012ADB7` 处的内容发生了相应的变化。

（可先将头一半字节改为 `AA` 试试，看是否变为 `46H`，否则改后一半，只要经过几次实验即可判断出来！）

好了，到这里就结束了。最后将这 6 个字节(`46 4E 56 60 6A 77`)加 `64H`（这里即为 `AA B2 BA C4 CE DB`）

将这计算出的 6 个数据写入 `cargador.cfg` 文件中的 `00000220H` 处的连续 6 个字节保存即完美破解！

程序下方显示为“谢谢您使用这个游戏，祝你早日通关！”

有兴趣的可写出内存注册机。

`4dd466d0e1e0cc123afbe504da2462ca`
