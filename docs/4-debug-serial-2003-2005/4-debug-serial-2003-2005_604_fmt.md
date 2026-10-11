
# bmp2txt v1.22

> 原文：[`www.pediy.com/kssd/pediy06/pediy6982.htm`](https://www.pediy.com/kssd/pediy06/pediy6982.htm)

## 破解目标

`bmp2txt v1.22`

## 软件简介

一个 ASC 编辑的软件,把 bmp 图片转换成形似的 ASC 字符图案

## 破解作者

xuanqing[BCG]

## 破解工具

ollydbg1.09d(en),ollydump,ImportREC1.42

## 破解目的

本不为破解而破解,只因为技术而破解.对作者表示歉意.

## 破解环境

WINDOWS XP (cn OEM with sp1)

## 相关关键字

telock0.98 脱壳,浮点比较指令,注册机 C++代码

## 破解说明

小菜刚刚(蒙混)加入 BCG,刚刚开始学习手动脱壳,第一次接触浮点指令

这是第一次以[BCG]后缀发表破文,错误和不足难免百出,还请大虾指出.

## 破解过程

### 1. 脱壳

peid 侦测为 TeLock0.98,按照二哥提供的教程中方法原封不动的搞定脱壳

(其实后来看这个算法很简单,完全可以不脱搞定的-\_-)

先试着载入,数异常的次数,直到运行(15 次既正常启动)

再次载入,点 14 次,然后在[esp+4]上下断

shift+f9 运行,被断下

之后小心的用 f7 和 f4 即可到达 oep:4044a4

ollydump 之

用 ImportREC1.42 修复,level3 搜索后,剩下少数几个(我这里是 4 个)无效指针,cut 掉

fixdump

检验文件已经运行正常

(其实,这个脱壳过程按照 fly 大虾的笔记也可以完成,但我还没有看太明白, 所以还是用 ImportREC1.42 搞定了)

### 2. 算法分析

这个软件的算法部分有一些浮点指令,

通过查阅论坛上网友提供的 intel80x86Opcodes.pdf

可以详细的知道每条指令的具体含义.

而这个算法又相对简单,是在破解中学习浮点的好例子

通过下 bp `MessageBoxA` 和一定的向回追溯,可以找到注册码的合法性检验部分

注释如下:

```
00403790   .  51            PUSH ECX
00403791   .  56            PUSH ESI
00403792   .  57            PUSH EDI
00403793   .  8BF1          MOV ESI,ECX
00403795   .  6A 01         PUSH 1
00403797   .  E8 140C0000   CALL <JMP.&mfc42.#6334_?UpdateData@CWnd@@QA>
0040379C   .  8B86 A0000000 MOV EAX,DWORD PTR DS:[ESI+A0]
004037A2   .  8DBE A0000000 LEA EDI,DWORD PTR DS:[ESI+A0]
004037A8   .  8B48 F8       MOV ECX,DWORD PTR DS:[EAX-8]
004037AB   .  85C9          TEST ECX,ECX
004037AD   .  74 5F         JE SHORT Bmp2TxtO.0040380E
004037AF   .  6A 2D         PUSH 2D                           ;  '-'
004037B1   .  8BCF          MOV ECX,EDI
004037B3   .  E8 6A0C0000   CALL <JMP.&mfc42.#2763_?Find@CString@@QBEHD>;  在 key 串中查找'-'
004037B8   .  83F8 04       CMP EAX,4                         ;  是否在第五个字节是?
004037BB   .  75 43         JNZ SHORT Bmp2TxtO.00403800        ;  不是跳出非法注册
004037BD   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]
004037C1   .  68 44734000   PUSH Bmp2TxtO.00407344              ;  ASCII "LI3SHU1CENG2BA4F-"
004037C6   .  51            PUSH ECX
004037C7   .  8BCF          MOV ECX,EDI
004037C9   .  E8 600C0000   CALL <JMP.&mfc42.#6223_?SpanIncluding@CStri>
004037CE   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]
004037D2   .  E8 F9090000   CALL <JMP.&mfc42.#800_??1CString@@QAE@XZ>
004037D7   .  8B3F          MOV EDI,DWORD PTR DS:[EDI]
004037D9   .  837F F8 09    CMP DWORD PTR DS:[EDI-8],9           ;  只是查长度??一定有别的验证地方
004037DD   .  75 21         JNZ SHORT Bmp2TxtO.00403800
004037DF   .  57            PUSH EDI                            ; /Arg1
004037E0   .  8BCE          MOV ECX,ESI                         ; |
004037E2   .  E8 C9000000   CALL Bmp2TxtO.004038B0              ; Bmp2TxtO.004038B0
004037E7   .  6A 00         PUSH 0
004037E9   .  6A 40         PUSH 40
004037EB   .  68 7C734000   PUSH Bmp2TxtO.0040737C
004037F0   .  E8 770A0000   CALL <JMP.&mfc42.#1200_?AfxMessageBox@@YGHP>
004037F5   .  8BCE          MOV ECX,ESI
004037F7   .  E8 C0080000   CALL <JMP.&mfc42.#4853_?OnOK@CDialog@@MAEXX>
004037FC   .  5F            POP EDI
004037FD   .  5E            POP ESI
004037FE   .  59            POP ECX
004037FF   .  C3            RETN
00403800   >  6A 00         PUSH 0
00403802   .  6A 10         PUSH 10
00403804   .  68 6C734000   PUSH Bmp2TxtO.0040736C
00403809   .  E8 5E0A0000   CALL <JMP.&mfc42.#1200_?AfxMessageBox@@YGHP>
0040380E   >  8D4E 60       LEA ECX,DWORD PTR DS:[ESI+60]
00403811   .  E8 DC0B0000   CALL <JMP.&mfc42.#5981_?SetFocus@CWnd@@QAEP>
00403816   .  5F            POP EDI
00403817   .  5E            POP ESI
00403818   .  59            POP ECX
00403819   .  C3            RETN
```

经简单分析可以知道这个软件会在别的地方严格验证注册码的合法性

大概搜索了一下,即可知道注册码被存在了注册表里面

下断 bp `RegQuaryExA`,重启程序即可找到关键比较代码.

注释如下:

```
00403250  /$  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00403256  |.  6A FF         PUSH -1
00403258  |.  68 B04B4000   PUSH Bmp2TxtO.00404BB0
0040325D  |.  50            PUSH EAX
0040325E  |.  64:8925 00000>MOV DWORD PTR FS:[0],ESP
00403265  |.  83EC 14       SUB ESP,14
00403268  |.  53            PUSH EBX
00403269  |.  55            PUSH EBP
0040326A  |.  56            PUSH ESI
0040326B  |.  8B7424 30     MOV ESI,DWORD PTR SS:[ESP+30]
0040326F  |.  33DB          XOR EBX,EBX
00403271  |.  8BE9          MOV EBP,ECX
00403273  |.  3BF3          CMP ESI,EBX
00403275  |.  75 16         JNZ SHORT Bmp2TxtO.0040328D          ;  是否有 Reg 保存的注册码
00403277  |.  5E            POP ESI
00403278  |.  5D            POP EBP
00403279  |.  33C0          XOR EAX,EAX
0040327B  |.  5B            POP EBX
0040327C  |.  8B4C24 14     MOV ECX,DWORD PTR SS:[ESP+14]
00403280  |.  64:890D 00000>MOV DWORD PTR FS:[0],ECX
00403287  |.  83C4 20       ADD ESP,20
0040328A  |.  C2 0400       RETN 4
0040328D  |>  68 48714000   PUSH Bmp2TxtO.00407148              ;  ASCII "LI3SHU1CENG2BA4F-"
00403292  |.  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]
00403296  |.  E8 650F0000   CALL <JMP.&mfc42.#537_??0CString@@QAE@PBD@Z>
0040329B  |.  56            PUSH ESI
0040329C  |.  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]
004032A0  |.  895C24 2C     MOV DWORD PTR SS:[ESP+2C],EBX
004032A4  |.  E8 570F0000   CALL <JMP.&mfc42.#537_??0CString@@QAE@PBD@Z>
004032A9  |.  8D4424 14     LEA EAX,DWORD PTR SS:[ESP+14]
004032AD  |.  68 48714000   PUSH Bmp2TxtO.00407148              ;  ASCII "LI3SHU1CENG2BA4F-"
004032B2  |.  50            PUSH EAX
004032B3  |.  8D4C24 38     LEA ECX,DWORD PTR SS:[ESP+38]
004032B7  |.  C64424 30 01  MOV BYTE PTR SS:[ESP+30],1
004032BC  |.  E8 6D110000   CALL <JMP.&mfc42.#6223_?SpanIncluding@CStri>
004032C1  |.  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]
004032C5  |.  E8 060F0000   CALL <JMP.&mfc42.#800_??1CString@@QAE@XZ>
004032CA  |.  6A 2D         PUSH 2D
004032CC  |.  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]
004032D0  |.  E8 53110000   CALL <JMP.&mfc42.#6874_?Remove@CString@@QAE>
004032D5  |.  8B4C24 30     MOV ECX,DWORD PTR SS:[ESP+30]
004032D9  |.  8B41 F8       MOV EAX,DWORD PTR DS:[ECX-8]
004032DC  |.  83F8 08       CMP EAX,8
004032DF  |.  74 34         JE SHORT Bmp2TxtO.00403315
004032E1  |.  8D4C24 30     LEA ECX,DWORD PTR SS:[ESP+30]
004032E5  |.  885C24 28     MOV BYTE PTR SS:[ESP+28],BL
004032E9  |.  E8 E20E0000   CALL <JMP.&mfc42.#800_??1CString@@QAE@XZ>
004032EE  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]
004032F2  |.  C74424 28 FFF>MOV DWORD PTR SS:[ESP+28],-1
004032FA  |.  E8 D10E0000   CALL <JMP.&mfc42.#800_??1CString@@QAE@XZ>
004032FF  |.  5E            POP ESI
00403300  |.  5D            POP EBP
00403301  |.  33C0          XOR EAX,EAX
00403303  |.  5B            POP EBX
00403304  |.  8B4C24 14     MOV ECX,DWORD PTR SS:[ESP+14]
00403308  |.  64:890D 00000>MOV DWORD PTR FS:[0],ECX
0040330F  |.  83C4 20       ADD ESP,20
00403312  |.  C2 0400       RETN 4
00403315  |>  57            PUSH EDI
00403316  |.  33FF          XOR EDI,EDI
00403318  |.  33F6          XOR ESI,ESI
0040331A  |>  8B5424 34     /MOV EDX,DWORD PTR SS:[ESP+34]        ;  去除了'-'的 key 串
0040331E  |.  8A0416        |MOV AL,BYTE PTR DS:[ESI+EDX]        ;  循环次字母
00403321  |.  884424 1C     |MOV BYTE PTR SS:[ESP+1C],AL         ;  入栈
00403325  |.  8B4C24 1C     |MOV ECX,DWORD PTR SS:[ESP+1C]       ;  0 扩展存 ECX
```



```
00403329  |.  51            |PUSH ECX   

0040332A  |.  8D4C24 14     |LEA ECX,DWORD PTR SS:\[ESP+14\]   

0040332E  |.  E8 EF100000   |CALL <JMP.&mfc42.#2763\_?Find@CString@@QBEH>;  查找在上面长度 10 的固定字串中的位置   

00403333  |.  C1E7 04       |SHL EDI,4                       ;  腾地方   

00403336  |.  03F8          |ADD EDI,EAX                     ;  该字母在串中前面还有几个字母,这个数值放到 EDI 的最低字节   

00403338  |.  46            |INC ESI                         ;  循环变量兼指针   

00403339  |.  83FE 08       |CMP ESI,8   

0040333C  |.^ 7C DC         JL SHORT Bmp2TxtO.0040331A   

0040333E  |.  8D5424 14     LEA EDX,DWORD PTR SS:\[ESP+14\]           ;  指向前 4 字节   

00403342  |.  8BCD          MOV ECX,EBP   

00403344  |.  52            PUSH EDX                        ; /Arg1   

00403345  |.  895C24 18     MOV DWORD PTR SS:\[ESP+18\],EBX           ; |   

00403349  |.  E8 82000000   CALL Bmp2TxtO.004033D0                  ; 取 C:盘的 volumeSerialNumber   

004033D0  /$  83EC 08       SUB ESP,8   

004033D3  |.  56            PUSH ESI   

004033D4  |.  8B7424 10     MOV ESI,DWORD PTR SS:\[ESP+10\]   

004033D8  |.  85F6          TEST ESI,ESI   

004033DA  |.  75 09         JNZ SHORT Bmp2TxtO.004033E5   

004033DC  |.  33C0          XOR EAX,EAX   

004033DE  |.  5E            POP ESI   

004033DF  |.  83C4 08       ADD ESP,8   

004033E2  |.  C2 0400       RETN 4   

004033E5  |>  6A 00         PUSH 0                        ; /pFileSystemNameSize = Null   

004033E7  |.  8D4424 08     LEA EAX,DWORD PTR SS:\[ESP+8\]        ; |   

004033EB  |.  6A 00         PUSH 0                        ; |pFileSystemNameBuffer = Null   

004033ED  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]       ; |   

004033F1  |.  50            PUSH EAX                        ; |pFileSystemFlags = 0012F5E4   

004033F2  |.  8D5424 1C     LEA EDX,DWORD PTR SS:\[ESP+1C\]       ; |   

004033F6  |.  51            PUSH ECX                        ; |pMaxFilenameLength = 0012F5E8   

004033F7  |.  52            PUSH EDX                        ; |pVolumeSerialNumber = 0012F5F0   

004033F8  |.  6A 00         PUSH 0                        ; |MaxVolumeNameSize = 0   

004033FA  |.  6A 00         PUSH 0                        ; |VolumeNameBuffer = Null   

004033FC  |.  68 1C734000   PUSH Bmp2TxtO.0040731C              ; |RootPathName = "c:\\
```

4dd466d0e1e0cc123afbe504da2462ca
