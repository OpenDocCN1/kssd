
# Easy CD Ripper 1.41

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-19.htm`](https://www.pediy.com/kssd/pediy08/pediy8-19.htm)

题目：`Easy CD Ripper` 的注册码（菜鸟级）

软件名称：`Easy CD Ripper 1.41`

文件大小：`3949K`

使用平台：`Win9x/NT/2000/XP`

最新版本：`1.41`

软件简介：

`Easy CD Ripper` 可将 CD 音乐及 `WAV` 直接转换成 `MP3`、`WMA`、`VQF` 格式的工具，采用全新快速的 `MP3` 编码(`LAME v3.89`)、支持 `CDDB`、且使用上相当简单快速。在注册之前，只可以解压缩 20 首歌曲，而且每个文件只可以解码出 70% 来。

破解目的：找出注册码

工具：`TRW`，`PEID`

引子：这几天碰到几个软件是用 `Asprotect` 压缩的，才网上找到的工具无法正确解压缩，用 `Casper` 也无法解压缩，不知什么缘故？是不是解压出来的文件的 `Import` 表破坏了啊？现在正在学习如何手工脱壳，偶尔找出几个小软件练手吧，也排解一下烦恼。这不找了一个 `Easy CD Ripper`（CD 音乐分割器 2002 电脑报合订本），用 `PEID` 一看是用 `Petite2.x` 加壳了，用 `prodump` 没有成功去壳，运行出现错误。我只好用动态跟踪来解决了。运行软件，出现注册窗口，输入你的大名和假注册码。比如 `wanggang`，`78787878`。`Ctrl+N` 调出 `TRW`，下 `bpx hmemcpy` 断点，`F5` 退出 `TRW`，然后点击 `OK` 按钮，被拦住，`bd*` 禁止断点，`F12` 按 7 次来到主程序空间。如下代码处，按 `F10` 继续跟踪：

```
0167:00428972 894304           MOV      [EBX+04],EAX
0167:00428975 8BD3             MOV      EDX,EBX
0167:00428977 8BC6             MOV      EAX,ESI
0167:00428979 E872B10000       CALL     00433AF0
0167:0042897E 5E               POP      ESI          //我们停在这里，F10 继续跟踪下去
0167:0042897F 5B               POP      EBX
0167:00428980 C3               RET
```

==================================================

```
0167:00430E4B 8955F0          MOV      [EBP-10],EDX
0167:00430E4E 894DF4          MOV      [EBP-0C],ECX
0167:00430E51 8B5508          MOV      EDX,[EBP+08]
0167:00430E54 8955F8          MOV      [EBP-08],EDX
0167:00430E57 33D2           XOR      EDX,EDX
0167:00430E59 8955FC          MOV      [EBP-04],EDX
0167:00430E5C 85C0            TEST     EAX,EAX
0167:00430E5E 740B            JZ       00430E6B
0167:00430E60 8D55F0          LEA      EDX,[EBP-10]
0167:00430E63 8BD8            MOV      EBX,EAX
0167:00430E65 8B432C          MOV      EAX,[EBX+2C]  //注册码长度送 EAX
0167:00430E68 FF5328          CALL     NEAR [EBX+28] //把用户名拷贝到另一个地方去。
0167:00430E6B 8B45FC          MOV      EAX,[EBP-04]
0167:00430E6E 5B               POP      EBX
0167:00430E6F 8BE5            MOV      ESP,EBP
0167:00430E71 5D               POP      EBP
0167:00430E72 C20400          RET      04
```

==================================================

下面这段把用户名复制到另一个地方去。

```
0167:00484AF9 8B45F8          MOV      EAX,[EBP-08]
0167:00484AFC 8A00            MOV      AL,[EAX]
0167:00484AFE 8803            MOV      [EBX],AL
0167:00484B00 8B45F8          MOV      EAX,[EBP-08]
0167:00484B03 8A4001          MOV      AL,[EAX+01]
。。。省略多行
0167:00484B2A 884305          MOV      [EBX+05],AL
0167:00484B2D 8B45F8          MOV      EAX,[EBP-08]
0167:00484B30 8A4006          MOV      AL,[EAX+06]
0167:00484B33 884306          MOV      [EBX+06],AL
0167:00484B36 8B45F8          MOV      EAX,[EBP-08]
0167:00484B39 8A4007          MOV      AL,[EAX+07]
0167:00484B3C 884307          MOV      [EBX+07],AL
0167:00484B3F 8D55F4          LEA      EDX,[EBP-0C]
0167:00484B42 8B45FC          MOV      EAX,[EBP-04]
0167:00484B45 8B80F0020000    MOV      EAX,[EAX+02F0]
0167:00484B4B E850B3FAFF      CALL     0042FEA0
0167:00484B50 8B45F4          MOV      EAX,[EBP-0C]
0167:00484B53 E8C03CF8FF      CALL     00408818      //此 CALL 是关键函数。
```

==================================================

`F8` 跟入 `484B53` 处的 `CALL`，代码如下：

```
0167:0040881D 8BD8            MOV      EBX,EAX
0167:0040881F 8BD4            MOV      EDX,ESP
0167:00408821 8BC3            MOV      EAX,EBX
0167:00408823 E810A3FFFF      CALL     00402B38      //此 CALL 测试注册码第一位是否是 20h,2Dh,2Bh,
第二位是否是 78h,58h。
0167:00408828 8BF0            MOV      ESI,EAX
0167:0040882A 833C2400       CMP      DWORD [ESP],BYTE +00
0167:0040882E 7419            JZ       00408849
```

==================================================

`F8` 跟入 `408823` 处的 `CALL`，代码如下：

```
0167:00402B38 53               PUSH     EBX
0167:00402B39 56               PUSH     ESI
0167:00402B3A 57               PUSH     EDI
0167:00402B3B 89C6             MOV      ESI,EAX
0167:00402B3D 50               PUSH     EAX
0167:00402B3E 85C0             TEST     EAX,EAX
0167:00402B40 7473             JZ       00402BB5
0167:00402B42 31C0             XOR      EAX,EAX
0167:00402B44 31DB             XOR      EBX,EBX
0167:00402B46 BFCCCCCC0C       MOV      EDI,0CCCCCCC   //这个值在后面用来比较。

*省略多行*

0167:00402B69 80FB58          CMP      BL,58
0167:00402B6C 745C            JZ       00402BCA
0167:00402B6E 80FB30          CMP      BL,30
0167:00402B71 7513            JNZ      00402B86       //注册码不为 0 则跳走
0167:00402B73 8A1E            MOV      BL,[ESI]
0167:00402B75 46              INC      ESI
0167:00402B76 80FB78          CMP      BL,78
0167:00402B79 744F            JZ       00402BCA
0167:00402B7B 80FB58          CMP      BL,58
0167:00402B7E 744A            JZ       00402BCA
0167:00402B80 84DB            TEST     BL,BL
0167:00402B82 7420            JZ       00402BA4
0167:00402B84 EB04            JMP      SHORT 00402B8A
0167:00402B86 84DB            TEST     BL,BL
0167:00402B88 7434            JZ       00402BBE
0167:00402B8A 80EB30          SUB      BL,30          //注册码减 30h 变为 16 进制数
0167:00402B8D 80FB09          CMP      BL,09          //与 9 比较
0167:00402B90 772C            JA       00402BBE       //大于 9 则错。
0167:00402B92 39F8            CMP      EAX,EDI        //与 EDI 里面的值比较
0167:00402B94 7728            JA       00402BBE       //如果大于则错
0167:00402B96 8D0480          LEA      EAX,[EAX+EAX*4] //EAX=5*EAX
0167:00402B99 01C0            ADD      EAX,EAX        //EAX=2*EAX （这 2 句相当于 EAX 乘 10 倍）
0167:00402B9B 01D8            ADD      EAX,EBX        //把各位注册码累加起来
0167:00402B9D 8A1E            MOV      BL,[ESI]       //继续取下位注册码
0167:00402B9F 46              INC      ESI
```

==================================================

```
0167:00484B58 8BD8            MOV      EBX,EAX        //EAX 保存的累加和送 EBX
0167:00484B5A 33C0            XOR      EAX,EAX
0167:00484B5C 5A               POP      EDX
0167:00484B5D 59               POP      ECX
0167:00484B5E 59               POP      ECX
0167:00484B5F 648910          MOV      [FS:EAX],EDX
0167:00484B62 EB0C            JMP      SHORT 00484B70
0167:00484B64 E943E7F7FF      JMP      004032AC
0167:00484B69 33DB            XOR      EBX,EBX
0167:00484B6B E898EAF7FF      CALL     00403608
0167:00484B70 6A00            PUSH     BYTE +00
0167:00484B72 6A00            PUSH     BYTE +00
0167:00484B74 6893810000      PUSH     DWORD 8193
0167:00484B79 8B45FC          MOV      EAX,[EBP-04]
0167:00484B7C E8F713FBFF      CALL     00435F78
0167:00484B81 50               PUSH     EAX
0167:00484B82 E84D26F8FF      CALL     `USER32!SendMessageA`
0167:00484B87 A1002E4900      MOV      EAX,[00492E00]
0167:00484B8C 33D2            XOR      EDX,EDX
0167:00484B8E 52               PUSH     EDX
0167:00484B8F 50               PUSH     EAX
0167:00484B90 8BC3            MOV      EAX,EBX        //累加和送 EAX
0167:00484B92 99               CDQ                     //扩充到 EDX
0167:00484B93 3B542404        CMP      EDX,[ESP+04]
0167:00484B97 7503            JNZ      00484B9C        //此处不能跳
0167:00484B99 3B0424          CMP      EAX,[ESP]       //真注册码值与假注册码累加和比较，D ESP
见 16 进制的真码 1004Ah
0167:00484B9C 5A               POP      EDX
0167:00484B9D 58               POP      EAX
0167:00484B9E 0F850A010000     JNZ      NEAR 00484CAE    //此处跳走则 OVER，关键。
0167:00484BA4 813D002E4900E803+CMP      DWORD [00492E00],03E8
0167:00484BAE 0F86FA000000     JNA      NEAR 00484CAE    //此处跳走也没戏。
0167:00484BB4 6A10            PUSH     BYTE +10
0167:00484BB6 8D45F0          LEA      EAX,[EBP-10]
0167:00484BB9 50               PUSH     EAX
```



#### 0167:00484BBA 8B1594194900     MOV      EDX,\[00491994\]  

\*  

\*省略多行，向注册表内写入好多内容。  

\*  

#### 0167:00484C7E 50              PUSH     EAX  

#### 0167:00484C7F 68D84D4800      PUSH     DWORD 00484DD8  

#### 0167:00484C84 6A00           PUSH     BYTE +00  

#### 0167:00484C86 E835A2FCFF     CALL     `SHELL32!ShellExecuteA`  

#### 0167:00484C8B A100194900      MOV      EAX,\[00491900\]  

#### 0167:00484C90 C70003000000   MOV      DWORD \[EAX\],03  

#### 0167:00484C96 6A00           PUSH     BYTE +00  

#### 0167:00484C98 6A00           PUSH     BYTE +00  

#### 0167:00484C9A 6A10           PUSH     BYTE +10  

#### 0167:00484C9C A1081C4900      MOV      EAX,\[00491C08\]  

#### 0167:00484CA1 8B00           MOV      EAX,\[EAX\]  

#### 0167:00484CA3 8B4024         MOV      EAX,\[EAX+24\]  

#### 0167:00484CA6 50             PUSH     EAX  

#### 0167:00484CA7 E8C024F8FF     CALL     `USER32!PostMessageA`   //此处显示注册成功提示框  

#### 0167:00484CAC EB60           JMP      SHORT 00484D0E            //此处通往光明！  

#### 0167:00484CAE 6A30           PUSH     BYTE +30  

#### 0167:00484CB0 8D45E4         LEA      EAX,\[EBP-1C\]  

\*  

\*省略多行，为 game over 准备材料  

\*  

#### 0167:00484D01 A1081C4900      MOV      EAX,\[00491C08\]  

#### 0167:00484D06 8B00           MOV      EAX,\[EAX\]  

#### 0167:00484D08 59             POP      ECX  

#### 0167:00484D09 E82691FCFF     CALL     0044DE34                 //此处显示失败提示框！  

\==================================================  

### 后记：  

这个小程序我用了很短时间就找到注册码了，输入后显示成功注册。可是就是因为注册成功了，我也后悔了，因为  

我没有把代码保存下来，无论我怎么卸载重新安装，这个软件都告诉你已经注册过的了，我 FAINT 了。我用 `REGSNAP`  

看了一下，也没有什么线索，不知这个软件在哪里记下了暗号了。真是滑稽！我上面的代码是通过操作一个功能，  

出现对话框后，我在 `TRW` 下消息断点才进入程序的，然后 U 出来的。`SOFTICE` 的 u 命令不如 `TRW` 的，`SOFTICE` 的这个 U 命  

令无法保存代码。  

看来现在到了该学习如何脱壳的时候了，真是水到自然成。现在的软件都很邪门的，都采取了各种保护措施，不会  

脱壳可是没的混了。：）希望各位大侠多写点这方面的文章供菜鸟学习。真是如饥似渴啊！！  

感谢您花费宝贵时间阅读，让你见笑了！  

### 总结：  

`Username: wanggang`  

`Registercode: 65610` （注册码是 `1004A` 的十进制数。）  

`QduWg`  

`qduwg@163.com`  

`2006 年 1 月 8 日完成` 

`4dd466d0e1e0cc123afbe504da2462ca`
