
# 计算机资料大全 3.0 Overlay 反脱壳去 NAG

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-503.htm`](https://www.pediy.com/kssd/pediy07/pediy7-503.htm)

【脱文标题】 计算机资料大全 3.0 Overlay 反脱壳分析+Ctrl+F8 智取 NAG  
【破文作者】 `weiyi75[Dfcg]`  
【作者邮箱】 `weiyi75@sohu.com`  
【作者主页】 `Dfcg` 官方大本营  
【使用工具】 `Peid`, `UpxShell`, `olldbg`, `winhex`  
【破解平台】 `WinNT/2000/XP`  
【软件名称】 计算机资料大全 3.0  
【下载页面】 [`downloads.daqing.net/`](http://downloads.daqing.net/)  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

------------------------------------------------------------------------

## 【破解分析】

首先 `Peid` 查壳，为 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo [Overlay]`，注意到`[Overlay]`，然后用 `Upxshell` 轻松脱壳，正常运行。

幸运的是它是个普通 `Upx` 壳，我们假设它是一个变形 `Upx` 壳。

```
004A1430 >   60             PUSHAD    //Upx 外壳入口
004A1431     BE 00804600      MOV ESI,《计算机.00468000
004A1436     8DBE 0090F9FF   LEA EDI,DWORD PTR DS:[ESI+FFF99000]
004A143C     C787 10270800 1>MOV DWORD PTR DS:[EDI+82710],F82C9510
004A1446     57             PUSH EDI
004A1447     83CD FF         OR EBP,FFFFFFFF
004A144A     EB 0E          JMP SHORT 《计算机.004A145A
004A144C     90             NOP
.........................................................
```

直接 `Ctrl+F` 向下找 `Popad`

```
004A159A     61             POPAD   
004A159B  ^ E9 64DEFDFF      JMP 《计算机.0047F404   //直接 F4 下来。

0047F404     55             PUSH EBP  //到达 Oep,Loadpe 脱壳。
0047F405     8BEC             MOV EBP,ESP
0047F407     83C4 F0         ADD ESP,-10
0047F40A     53             PUSH EBX
0047F40B     B8 A4F14700      MOV EAX,《计算机.0047F1A4
0047F410     E8 CF76F8FF      CALL 《计算机.00406AE4
0047F415     33C0             XOR EAX,EAX
0047F417     55             PUSH EBP
0047F418     68 53F54700      PUSH 《计算机.0047F553
0047F41D     64:FF30         PUSH DWORD PTR FS:[EAX]
0047F420     64:8920         MOV DWORD PTR FS:[EAX],ESP
0047F423     E8 146CFEFF      CALL 《计算机.0046603C
0047F428     BA 143E4800      MOV EDX,《计算机.00483E14
0047F42D     B8 68F54700      MOV EAX,《计算机.0047F568                    ; ASCII "XSEI.NATC"
0047F432     E8 216EFEFF      CALL 《计算机.00466258
0047F437     833D 143E4800 0>CMP DWORD PTR DS:[483E14],0
0047F43E     74 04          JE SHORT 《计算机.0047F444
.................................................................
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `7F404`，点 `IT AutoSearch`，点“`Get Import`”，看到输入表全部有效针，`FixDump`。  

运行前看到加壳程序 `13.2M`，脱壳程序 `668KB`，肯定是无法运行的，这个是因为加壳程序尾部的数据没有复制到脱壳程序里面。

用 `Winhex` 打开加壳程序。

我们看到尾部数据从偏移量 `987` 开始。在 `987` 处点右键选块开始。

然后滚动条拉到底，到偏移量 `13889591` 为止，在 `13889591` 处点右键选块结束。

`Crtl+C` 复制选定内容。

然后打开脱壳文件，直接滚动条到尾部偏移量 `684031` 处，点右键-编辑-剪贴板数据写入，确定两次，原理搞清楚就不犹豫，保存文件，运行正常。

短短的三次试用一会就过期了，先前没有用 `RegSnap` 给这个程序照相。

## 破解 NAG

下断点

`Bp MessageBoxA`

无法中断 `NAG`，没办法

`OD` 载入程序

```
0047F404 > $ 55              PUSH EBP  //入口点。
0047F405  . 8BEC             MOV EBP,ESP
0047F407  . 83C4 F0         ADD ESP,-10
0047F40A  . 53              PUSH EBX
0047F40B  . B8 A4F14700    MOV EAX,《计算机.0047F1A4
0047F410  . E8 CF76F8FF    CALL 《计算机.00406AE4
0047F415  . 33C0             XOR EAX,EAX
0047F417  . 55              PUSH EBP
.................................................
```

这里想起风飘雪老师的加密和解密的实战功略里面对付 `NAG` 的方法。

那就是必杀 `Ctrl+F8`

简单解释一下，`Ctrl+F8` 自动步过，遇到 `Call` 中 `Call`，然后 `Nag` 调用运行。

所以遇到 `Call` 停止后就下断点，不断 `Ctrl+F2` 和 `F9` `Ctrl+F8` 三键灵活使用。

```
0045A5D8  . E8 0396FAFF    CALL 《计算机.00403BE0   //这个 Call 要调用两次。
0047F51A  . E8 9125FEFF    CALL 《计算机.00461AB0  //第一次 NAG,下断点重启再次中断,F7 跟进，Ctrl+F8 到下一个 Call NAG 出现。
00461AB0  $ 55              PUSH EBP      //跟进这里，Ctrl+F8
00461AB1  . 8BEC             MOV EBP,ESP
00461AB3  . 51              PUSH ECX
00461AB4  . 53              PUSH EBX
00461AB5  . 56              PUSH ESI
00461AB6  . 57              PUSH EDI
00461AB7  . 894D FC         MOV DWORD PTR SS:[EBP-4],ECX
00461ABA  . 8BDA             MOV EBX,EDX
00461ABC  . 8BF0             MOV ESI,EAX
00461ABE  . 8BC3             MOV EAX,EBX
00461AE1  . FF57 2C         CALL DWORD PTR DS:[EDI+2C]          //第二次 NAG,下断点重启再次中断,F7 跟进
0045A464  $ 55              PUSH EBP    //跟进这里，Ctrl+F8
0045A465  . 8BEC             MOV EBP,ESP
0045A467  . 81C4 ECFEFFFF  ADD ESP,-114
0045A46D  . 53              PUSH EBX
0045A46E  . 33DB             XOR EBX,EBX
0045A470  . 899D ECFEFFFF  MOV DWORD PTR SS:[EBP-114],EBX
0045A476  . 84D2             TEST DL,DL
0045A478  . 74 08          JE SHORT 《计算机.0045A482
0045A47A  . 83C4 F0         ADD ESP,-10
0045A47D  . E8 0697FAFF    CALL 《计算机.00403B88
0045A482  > 8BD9             MOV EBX,ECX
0045A5D8  . E8 0396FAFF    CALL 《计算机.00403BE0           //第三次 NAG,下断点重启再次中断,F7 跟进
0047BA46  . E8 2D200000    CALL 《计算机.0047DA78          //第四次 NAG,下断点重启再次中断,F7 跟进
0047DD13  . E8 F8D1FBFF    CALL 《计算机.0043AF10         //第五次 NAG,向上找到 Magic jmp
```

整个过程就是吃卷心菜，一层层拨开。

```
0047DC4B  . 3C 30          CMP AL,30    //千辛万苦找到的 Magic jmp，大家可以看 OD 的跳转线和单步跟踪最后一次 Call 的全部过程确定这是爆破点,必须跳转，刚好跳出死亡 Call。
0047DC4D  . 0F84 97010000 JE 《计算机.0047DDEA
0047DC53  . 3C 31          CMP AL,31
0047DC55  . 75 45          JNZ SHORT 《计算机.0047DC9C
0047DC57  . E8 68CAF8FF    CALL 《计算机.0040A6C4
0047DC5C  . 83C4 F8         ADD ESP,-8                          ; /
0047DC5F  . DD1C24        FSTP QWORD PTR SS:[ESP]               ; |Arg3 (8-byte)
0047DC62  . 9B             WAIT                            ; |
0047DC63  . FF75 FC         PUSH DWORD PTR SS:[EBP-4]           ; |Arg2
0047DC66  . FF75 F8         PUSH DWORD PTR SS:[EBP-8]           ; |Arg1
0047DC69  . E8 72C8FFFF    CALL 《计算机.0047A4E0                   ; \\《计算机.0047A4E0
0047DC6E  . 3B05 9C3D4800 CMP EAX,DWORD PTR DS:[483D9C]
0047DC74  . 7E 26          JLE SHORT 《计算机.0047DC9C
0047DC76  . 6A 01          PUSH 1
0047DC78  . 8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
0047DC7B  . 8B0D C03D4800 MOV ECX,DWORD PTR DS:[483DC0]
0047DC81  . 8B15 D03D4800 MOV EDX,DWORD PTR DS:[483DD0]
0047DC87  . E8 086CF8FF    CALL 《计算机.00404894
0047DC8C  . 8B55 CC        MOV EDX,DWORD PTR SS:[EBP-34]
0047DC8F  . B9 50DF4700    MOV ECX,《计算机.0047DF50                    ;  ASCII "eje"
0047DC94  . 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
0047DC97  . E8 2877FFFF    CALL 《计算机.004753C4
0047DC9C  > A1 C43D4800    MOV EAX,DWORD PTR DS:[483DC4]
0047DCA1  . 8078 14 32    CMP BYTE PTR DS:[EAX+14],32
0047DCA5  . 75 2E          JNZ SHORT 《计算机.0047DCD5
0047DCA7  . 3B35 A03D4800 CMP ESI,DWORD PTR DS:[483DA0]
0047DCAD  . 7E 26          JLE SHORT 《计算机.0047DCD5
0047DCAF  . 6A 01          PUSH 1
0047DCB1  . 8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]
0047DCB4  . 8B0D C03D4800 MOV ECX,DWORD PTR DS:[483DC0]
0047DCBA  . 8B15 D03D4800 MOV EDX,DWORD PTR DS:[483DD0]
0047DCC0  . E8 CF6BF8FF    CALL 《计算机.00404894
0047DCC5  . 8B55 C8        MOV EDX,DWORD PTR SS:[EBP-38]
0047DCC8  . B9 50DF4700    MOV ECX,《计算机.0047DF50                    ;  ASCII "eje"
0047DCCD  . 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
0047DCD0  . E8 EF76FFFF    CALL 《计算机.004753C4
0047DCD5  > 6A 00          PUSH 0
0047DCD7  . 8D45 C4        LEA EAX,DWORD PTR SS:[EBP-3C]
0047DCDA  . 8B0D C03D4800 MOV ECX,DWORD PTR DS:[483DC0]
0047DCE0  . 8B15 D03D4800 MOV EDX,DWORD PTR DS:[483DD0]
0047DCE6  . E8 A96BF8FF    CALL 《计算机.00404894
0047DCEB  . 8B55 C4        MOV EDX,DWORD PTR SS:[EBP-3C]            ; |
0047DCEE  . B9 50DF4700    MOV ECX,《计算机.0047DF50                    ; |ASCII "eje"
0047DCF3  . 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]             ; |
0047DCF6  . E8 8576FFFF    CALL 《计算机.00475380                     ; \\《计算机.00475380
0047DCFB  . 84C0           TEST AL,AL
0047DCFD  . 0F84 E7000000 JE 《计算机.0047DDEA
0047DD03  . 6A 00          PUSH 0                               ; /Arg1 = 00000000
```
