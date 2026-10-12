
# Setup2Go 1.97

> 原文：[`www.pediy.com/kssd/pediy05/pediy50663.htm`](https://www.pediy.com/kssd/pediy05/pediy50663.htm)

## Setup2Go 1.97 破解手记--算法分析

**作者**：`newlaos[CCG][DFCG]`

**软件名称**：`Setup2Go` 1.97(安装制作)

**整理日期**：2003.4.23

**最新版本**：1.97 (04.15)

**文件大小**：955KB

**软件授权**：共享软件

**使用平台**：`Win9x`/`Me`/`NT`/`2000`/`XP`

**发布公司**：[`www.dev4pc.com/products.html`](http://www.dev4pc.com/products.html)

**软件简介**：`Setup2Go` 是一个很不错的安装程序制作工具，易于使用且交互性强，它不需要使用者具备多少编程知识和编程经验就可在极短的时间内轻松完成制作，该软件还支持当前所有的 32 位 Windows 操作系统的程序，包括 `Windows` 95、98、`ME`、`NT4`、`2000`、`XP` 等。软件还自带工程向导帮助你快速生成安装项目，像建立快捷方式、写入注册表、文件类型关联、定制对话框及屏幕样式、使用外部工具、修改 `INI` 文件、添加安装密码、测试运行等等这些功能它都具备，并且你还可以利用 `Setup2Go` 制作出支持多国语言的安装程序，便于你向外国人出售自己的软件产品。

**加密方式**：注册码+`VC6.0`

**功能限制**：功能限制

**PJ 工具**：`TRW20001.23` 注册版，`W32Dasm8.93` 黄金版，`FI2.5`，`Ollydbg` `V1.09b` 中文版

**PJ 日期**：2003-04-17

**作者 `newlaos` 申明**：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

### 分析过程

1、用 `FI2.5` 查壳，发现是 `VC6.0` 程序(其实加了一个压缩壳)

2、用 `W32Dasm` 黄金修正版本进行静态反汇编，什么也找不到

3、只好动态跟踪调试。请出国宝 `TRW2000`，输入姓名 `newlaos`,假码 `78787878`，下断点 `BPX` `hmemcpy(万能断点)`。点确定被断下来，用命令 `pmodule`，发现却来到这里 `00415F55`，而在 `W32Dasm` 黄金修正版本进行静态反汇编时，没有这个代码段，说明原程序被压缩过了。

4、最好用 `Ollydbg` `V1.09b` 中文版来调试，有人问为什么？原因就是这样我才好为大家写破文呀，在 `TRW2000` 动态跟踪里，汇编代码也不可能手抄呀。在 `00415F55` 处，按 `F2` 下断点：

```
.......
.......

00415F4F   FF15 C4124000    CALL DWORD PTR DS:[<&USER32.SendMessageA>
00415F55   8BFB             MOV EDI,EBX       <===程序来到这里，EAX=7(注册名的长度)，EBX=newlaos
00415F57   83C9 FF          OR ECX,FFFFFFFF
00415F5A   33C0             XOR EAX,EAX
00415F5C   8D95 F4FDFFFF    LEA EDX,DWORD PTR SS:[EBP-20C]
00415F62   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
00415F64   F7D1             NOT ECX
00415F66   2BF9             SUB EDI,ECX
00415F68   53               PUSH EBX
00415F69   8BC1             MOV EAX,ECX
00415F6B   8BF7             MOV ESI,EDI
00415F6D   8BFA             MOV EDI,EDX
00415F6F   68 00020000      PUSH 200
00415F74   C1E9 02          SHR ECX,2
00415F77   F3:A5            REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
00415F79   8BC8             MOV ECX,EAX
00415F7B   8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00415F7E   6A 0D            PUSH 0D
00415F80   83E1 03          AND ECX,3
00415F83   FF70 38          PUSH DWORD PTR DS:[EAX+38]
00415F86   F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:>
00415F88   FF15 C4124000    CALL DWORD PTR DS:[<&USER32.SendMessageA>
00415F8E   8BFB             MOV EDI,EBX        <===EAX=8(假码的长度)，EBX=78787878
00415F90   83C9 FF          OR ECX,FFFFFFFF
00415F93   33C0             XOR EAX,EAX
00415F95   8D95 F8FEFFFF    LEA EDX,DWORD PTR SS:[EBP-108]
00415F9B   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
00415F9D   F7D1             NOT ECX
00415F9F   2BF9             SUB EDI,ECX
00415FA1   8BC1             MOV EAX,ECX
00415FA3   8BF7             MOV ESI,EDI
00415FA5   8BFA             MOV EDI,EDX
00415FA7   C1E9 02          SHR ECX,2
00415FAA   F3:A5            REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
00415FAC   8BC8             MOV ECX,EAX
00415FAE   8D85 F8FEFFFF    LEA EAX,DWORD PTR SS:[EBP-108]
00415FB4   50               PUSH EAX
00415FB5   83E1 03          AND ECX,3
00415FB8   8D85 F4FDFFFF    LEA EAX,DWORD PTR SS:[EBP-20C]
00415FBE   F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:>
00415FC0   50               PUSH EAX
00415FC1   E8 DA3A0200      CALL SETUP2GO.00439AA0
00415FC6   8BC8             MOV ECX,EAX
00415FC8   E8 5AE5FFFF      CALL SETUP2GO.00414527 <===这里是关键的 CALL，F8 跟进
00415FCD   5F               POP EDI
00415FCE   5E               POP ESI
00415FCF   84C0             TEST AL,AL  <===要想注册成功，则这里 AL 不能为 0
00415FD1   5B               POP EBX
00415FD2  ^F84 5DFFFFFF     JE SETUP2GO.00415F35 <===这里是关键的跳转，这里不能跳
00415FD8   68 0D080000      PUSH 80D
00415FDD   EB 18            JMP SHORT SETUP2GO.00415FF7
00415FDF   68 10040000      PUSH 410
00415FE4   6A 01            PUSH 1
00415FE6   E8 7F3B0200      CALL SETUP2GO.00439B6A
00415FEB   59               POP ECX
00415FEC   59               POP ECX
00415FED  ^E9 43FFFFFF      JMP SETUP2GO.00415F35
00415FF2   68 01080000      PUSH 801
00415FF7   FF75 08          PUSH DWORD PTR SS:[EBP+8]
00415FFA   FF15 8C134000    CALL DWORD PTR DS:[<&USER32.EndDialog>] <===结束对话框函数，胜利的标志
00416000   B0 01            MOV AL,1
00416002   C9               LEAVE
00416003   C2 1000          RETN 10
```

----------00415FC8 `CALL` `00414527` 这里是关键的 `CALL`，`F8` 跟进-------------------------

```
00414527   55               PUSH EBP
00414528   8BEC             MOV EBP,ESP
0041452A   51               PUSH ECX
0041452B   51               PUSH ECX
0041452C   53               PUSH EBX      <===EBX=78787878
0041452D   56               PUSH ESI
0041452E   57               PUSH EDI
0041452F   8B7D 08          MOV EDI,DWORD PTR SS:[EBP+8] <===EDI=newlaos
00414532   FF75 0C          PUSH DWORD PTR SS:[EBP+C]
00414535   8BD9             MOV EBX,ECX
00414537   57               PUSH EDI
00414538   E8 97FFFFFF      CALL SETUP2GO.004144D4    <===又是关键的 CALL，F8 跟进
0041453D   84C0             TEST AL,AL                 <===AL 不能为 0
0041453F   0F84 C0000000    JE SETUP2GO.00414605       <===这里一就 OVER 了
00414545   33F6             XOR ESI,ESI
00414547   8D45 FC          LEA EAX,DWORD PTR SS:[EBP-4]
0041454A   56               PUSH ESI
0041454B   50               PUSH EAX
0041454C   56               PUSH ESI
0041454D   68 3F000F00      PUSH 0F003F
00414552   56               PUSH ESI
00414553   68 2CB14000      PUSH SETUP2GO.0040B12C           ; ASCII "Setup2GO"
00414558   56               PUSH ESI
00414559   68 B0B14000      PUSH SETUP2GO.0040B1B0           ; ASCII "software\\SDS Software\\Setup2GO"

.......

.......此处略去一段正确的注册信息保存代码

00414601   B0 01            MOV AL,1    <===关键的标志位赋值，必须经过
00414603   EB 02            JMP SHORT SETUP2GO.00414607
00414605   32C0             XOR AL,AL   <===标志位清 0，就 OVER 了
00414607   5F               POP EDI
00414608   5E               POP ESI
00414609   5B               POP EBX
0041460A   C9               LEAVE
0041460B   C2 0800          RETN 8
```

-----------00414538 `CALL` `004144D4` 又是关键的 `CALL`，`F8` 跟进--------------------

```
004144D4   55               PUSH EBP
004144D5   8BEC             MOV EBP,ESP
004144D7   51               PUSH ECX
004144D8   33D2             XOR EDX,EDX
004144DA   57               PUSH EDI
004144DB   3955 08          CMP DWORD PTR SS:[EBP+8],EDX  <===看是否输入的用户名
004144DE   74 40            JE SHORT SETUP2GO.00414520    <===这里不能跳
004144E0   3955 0C          CMP DWORD PTR SS:[EBP+C],EDX  <===看是否输入的注册码
004144E3   74 3B            JE SHORT SETUP2GO.00414520    <===这里不能跳
004144E5   8B7D 08          MOV EDI,DWORD PTR SS:[EBP+8]  <===EDI=newlaos
004144E8   83C9 FF          OR ECX,FFFFFFFF
004144EB   33C0             XOR EAX,EAX
004144ED   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
004144EF   F7D1             NOT ECX
004144F1   49               DEC ECX
004144F2   74 2C            JE SHORT SETUP2GO.00414520
004144F4   8B7D 0C          MOV EDI,DWORD PTR SS:[EBP+C]
004144F7   83C9 FF          OR ECX,FFFFFFFF
004144FA   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
004144FC   F7D1             NOT ECX
004144FE   49               DEC ECX
004144FF   83F9 0A          CMP ECX,0A                   <===注册码的长度，必须为 10 位
00414502   75 1C            JNZ SHORT SETUP2GO.00414520  <===如果不是，一跳就 OVER 了
00414504   FF75 0C          PUSH DWORD PTR SS:[EBP+C]    <===重新输入假码 7878787878，来到这里
00414507   8D45 FC          LEA EAX,DWORD PTR SS:[EBP-4]
0041450A   8955 FC          MOV DWORD PTR SS:[EBP-4],EDX
0041450D   50               PUSH EAX
0041450E   68 74B44000      PUSH SETUP2GO.0040B474           ; ASCII "pasha and andrey"
```
