
# 关于 MindManager 6 的分析笔记

> 原文：[`www.pediy.com/kssd/pediy11/109482.html`](https://www.pediy.com/kssd/pediy11/109482.html)

`MindManager 6`

`HomePage`：[`www.mindjet.com/`](http://www.mindjet.com/)

很久没调试程序了，要做一个视图顺便下了该程序顺手调了一下，有兴趣的可以先自行调试下，软件不难，这里我给出我的两种修改方案，学习软件破解要看淡结果，重视过程，重视方法，重视从不同思路去探寻突破口。破解技术本是一种思维的养成，乐趣在于过程，而非漫天的发布以满足一己的虚荣。

一款制作视图的工具，顺便调试了一下，官方已经升级到 8 了，我分析的是 `6.0.643` 的版本。后续版本有兴趣可以自行分析一下，应该是大同小异的。

本文分析的版本下载地址：[`download.mindjet.com/MM60-E-643.exe`](http://download.mindjet.com/MM60-E-643.exe)

最新版下载:

`DownLoad`：[`download.mindjet.com/MM80-E-211.exe`](http://download.mindjet.com/MM80-E-211.exe)

`DownLoad`：[`download.mindjet.com/MM80-E-235.exe`](http://download.mindjet.com/MM80-E-235.exe)

软件重启验证，`KEY` 保存在注册表中。可以下读取注册表的断点来分析，也可以分析软件的 `ASCII` 来定位：

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Mindjet\MindManager\6\Registration]

"BrandName"="Nisy"
"BrandCompany"="ChinaPYG"
"LicenseKey"="Nisy/ChinaPYG"
```

## 修改方法一

```
00625450  /$  56            PUSH ESI
00625451  |.  8BF1          MOV ESI,ECX
00625453  |.  8B8E 64010000 MOV ECX,DWORD PTR DS:[ESI+164]
00625459  |.  E8 52E8E0FF   CALL MindMana.00433CB0
0062545E  |.  8B8E 64010000 MOV ECX,DWORD PTR DS:[ESI+164]
00625464    E8 F70DDEFF   CALL MindMana.00406260                 ;  2222222
00625469  |.  8B0D D80FAC00 MOV ECX,DWORD PTR DS:[AC0FD8]
0062546F  |.  8A91 FE010000 MOV DL,BYTE PTR DS:[ECX+1FE]
00625475  |.  84D2          TEST DL,DL
00625477  |.  5E            POP ESI
00625478    75 08          JNZ SHORT MindMana.00625482
0062547A  |.  84C0          TEST AL,AL
0062547C    75 04          JNZ SHORT MindMana.00625482
0062547E    B0 01          MOV AL,1
00625480  |.  EB 02          JMP SHORT MindMana.00625484
00625482  |>  32C0          XOR AL,AL
00625484  |>  50            PUSH EAX
00625485  |.  E8 9612DEFF   CALL MindMana.00406720
0062548A  |.  8B0D D80FAC00 MOV ECX,DWORD PTR DS:[AC0FD8]
00625490  |.  8A81 FE010000 MOV AL,BYTE PTR DS:[ECX+1FE]
00625496  |.  84C0          TEST AL,AL
00625498  |.  74 07          JE SHORT MindMana.006254A1
0062549A  |.  6A 00          PUSH 0
0062549C  |.  E8 DF10DEFF   CALL MindMana.00406580
006254A1  |>  B0 01          MOV AL,1
006254A3  \.  C3            RETN
```

修改为：

```
00625450   $  56            PUSH ESI
00625451   .  8BF1          MOV ESI,ECX
00625453   .  8B8E 64010000 MOV ECX,DWORD PTR DS:[ESI+164]
00625459   .  E8 52E8E0FF   CALL MindMana.00433CB0
0062545E   .  8B8E 64010000 MOV ECX,DWORD PTR DS:[ESI+164]
00625464   .  C641 04 01    MOV BYTE PTR DS:[ECX+4],1             ;  //
00625468   .  90            NOP
00625469   .  8B0D D80FAC00 MOV ECX,DWORD PTR DS:[AC0FD8]
0062546F   .  8A91 FE010000 MOV DL,BYTE PTR DS:[ECX+1FE]
00625475   .  84D2          TEST DL,DL
00625477   .  5E            POP ESI
00625478   .  32C0          XOR AL,AL                              ;  //
0062547A   .  84C0          TEST AL,AL
0062547C   .  EB 04         JMP SHORT MindMana.00625482
0062547E   .  B0 01         MOV AL,1
00625480   .  EB 02         JMP SHORT MindMana.00625484
00625482   >  32C0          XOR AL,AL
00625484   >  50            PUSH EAX
00625485   .  E8 9612DEFF   CALL MindMana.00406720
0062548A   .  8B0D D80FAC00 MOV ECX,DWORD PTR DS:[AC0FD8]
00625490   .  8A81 FE010000 MOV AL,BYTE PTR DS:[ECX+1FE]
00625496   .  84C0          TEST AL,AL
00625498   .  74 07         JE SHORT MindMana.006254A1
0062549A   .  6A 00         PUSH 0
0062549C   .  E8 DF10DEFF   CALL MindMana.00406580
006254A1   >  B0 01         MOV AL,1
006254A3   .  C3            RETN
```

## 修改方法二（这里用的是模拟验证 `Call` 的方法来实现的）

```
0062D7BB   . /0F84 D8040000    JE MindMana.0062DC99
0062D7C1   . |8BCE            MOV ECX,ESI
0062D7C3   . |E8 887CFFFF      CALL MindMana.00625450
0062D7C8   . |84C0            TEST AL,AL
0062D7CA   . |0F84 C9040000    JE MindMana.0062DC99
```

修改为：

```
0062D7BB   .  8BCE          MOV ECX,ESI
0062D7BD   .  8B89 64010000 MOV ECX,DWORD PTR DS:[ECX+164]
0062D7C3   .  C641 04 01    MOV BYTE PTR DS:[ECX+4],1
0062D7C7   .  90            NOP
```
