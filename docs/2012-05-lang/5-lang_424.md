
# 音乐海盗 V1.32 的注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-465.htm`](https://www.pediy.com/kssd/pediy07/pediy7-465.htm)

## 破文作者信息

【破文作者】 rdsnow\[BCG\]\[PYG\]\[D.4s\]

【作者主页】 [`rdsnow.ys168.com`](http://rdsnow.ys168.com)

【E-mail】 rdsnow@163.com

【作者 QQ】 83757177

【文章题目】 音乐海盗 V1.32 的注册

【软件名称】 音乐海盗 1.32

【下载地址】 [`www.enowsoft.com/asp\_7i24\_co...reg\_yyhd132.exe`](http://www.enowsoft.com/asp_7i24_com/enowsoft/reg_yyhd/reg_yyhd132.exe)

----------------------------------------------------------------------------------------------

## 加密与破解信息

【加密方式】 序列号

【破解工具】 `ODbyDYK` v1.10\[05.09\]

【软件限制】 功能限制

【破解平台】 Microsoft Windows XP SP2

## 文章简介

呵呵，这个版本是一个注册用户发给我的下载地址，只有这个地址下载到的才是完整版，其他的都是试用版的。

Microsoft Visual Basic 5.0 / 6.0，其他没有什么说的，原因是最近被壳搞得头大，来个简单的，全当灌水吧!高手勿进。

----------------------------------------------------------------------------------------------

## 破解过程

程序启动时会弹出一个输入对话框，让你输入注册码

于是输入假码，命令行下断：`bp rtcInputBox`

运行程序，中断，并返回到程序领空，来到这里：

```
007C0E5F    52               PUSH EDX  
007C0E60    FF15 88104000    CALL DWORD PTR DS:[<&MSVBVM60.#596>]            ; MSVBVM60.rtcInputBox  
007C0E66    8BD0             MOV EDX,EAX  
007C0E68    8D4D D4          LEA ECX,DWORD PTR SS:[EBP-2C]  
007C0E6B    FFD3             CALL EBX  
007C0E6D    8D85 4CFFFFFF    LEA EAX,DWORD PTR SS:[EBP-B4]  
007C0E73    50               PUSH EAX  
```
………………

省略中间一大段次要代码

```
007C0EDB    52               PUSH EDX  
007C0EDC    8D45 9C          LEA EAX,DWORD PTR SS:[EBP-64]  
007C0EDF    50               PUSH EAX  
007C0EE0    FFD3             CALL EBX  
007C0EE2    8D4D AC          LEA ECX,DWORD PTR SS:[EBP-54]  
007C0EE5    51               PUSH ECX                                     ; 假注册码  
007C0EE6    8D55 9C          LEA EDX,DWORD PTR SS:[EBP-64]  
007C0EE9    52               PUSH EDX                                     ; 真注册码  
007C0EEA    FF15 D4104000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarTstEq>] ; MSVBVM60.__vbaVarTstEq，真假注册码比较  
007C0EF0    66:8BD8          MOV BX,AX  
007C0EF3    8D45 9C          LEA EAX,DWORD PTR SS:[EBP-64]  
007C0EF6    50               PUSH EAX  
007C0EF7    8D4D AC          LEA ECX,DWORD PTR SS:[EBP-54]  
007C0EFA    51               PUSH ECX  
007C0EFB    6A 02            PUSH 2  
007C0EFD    FFD7             CALL EDI                                     ; MSVBVM60.__vbaFreeVarList  
007C0EFF    83C4 0C          ADD ESP,0C  
007C0F02    66:85DB          TEST BX,BX  
007C0F05    0F84 B0010000    JE 音乐海盗.007C10BB                        ; 关键跳转  
007C0F0B    8B55 E0          MOV EDX,DWORD PTR SS:[EBP-20]  
007C0F0E    52               PUSH EDX  
007C0F0F    56               PUSH ESI  
007C0F10    6A FF            PUSH -1  
007C0F12    6A 02            PUSH 2  
007C0F14    FF15 80114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaFileOpen>] ; MSVBVM60.__vbaFileOpen  
007C0F1A    8B45 D4          MOV EAX,DWORD PTR SS:[EBP-2C]  
007C0F1D    50               PUSH EAX  
007C0F1E    56               PUSH ESI  
007C0F1F    68 64F74000      PUSH 音乐海盗.0040F764  
007C0F24    FF15 30114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaPrintFile>] ; MSVBVM60.__vbaPrintFile  
007C0F2A    83C4 0C          ADD ESP,0C  
007C0F2D    56               PUSH ESI  
007C0F2E    FF15 C0104000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaFileClose>] ; MSVBVM60.__vbaFileClose  
007C0F34    BA 08104100      MOV EDX,音乐海盗.00411008  
007C0F39    8D4D CC          LEA ECX,DWORD PTR SS:[EBP-34]  
007C0F3C    FF15 9C114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCopy>]  ; MSVBVM60.__vbaStrCopy  
007C0F42    B8 04000280      MOV EAX,80020004  
```
……………………

```
007C0F7C    51               PUSH ECX  
007C0F7D    6A 40            PUSH 40  
007C0F7F    8D95 3CFFFFFF    LEA EDX,DWORD PTR SS:[EBP-C4]  
007C0F85    52               PUSH EDX                                     ; 注册成功的对话框  
007C0F86    FF15 7C104000    CALL DWORD PTR DS:[<&MSVBVM60.#595>]         ; MSVBVM60.rtcMsgBox  
007C0F8C    8D45 8C          LEA EAX,DWORD PTR SS:[EBP-74]  
```

其实直接在`__vbaVarTstEq`上下断，就可以看到注册码了。

不过瘾，那就继续分析程序的注册码是怎么形成的：

程序在不同的电脑上有不同的注册码，可能跟硬盘序列号有关，于是命令行下断：`bp GetVolumeInformationA`

中断后，来到这里：

```
007BFD0B   .  50             PUSH EAX  
007BFD0C   .  E8 BBF2C4FF    CALL 音乐海盗.0040EFCC                     ;  读取程序所在分区的硬盘序列号  
```
偶的硬盘序号：`80D252D7`

下面的注释中的数据是偶机器上得到的：

```
007BFD11   .  FF15 60104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaSetSystemErr>;  MSVBVM60.__vbaSetSystemError  
007BFD17   .  8B55 D4        MOV EDX,DWORD PTR SS:[EBP-2C]  
007BFD1A   .  52             PUSH EDX  
007BFD1B   .  8D45 DC        LEA EAX,DWORD PTR SS:[EBP-24]  
007BFD1E   .  50             PUSH EAX  
007BFD1F   .  FF15 2C114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrToUnicode>;  MSVBVM60.__vbaStrToUnicode  
007BFD25   .  8D4D D4        LEA ECX,DWORD PTR SS:[EBP-2C]  
007BFD28   .  8B35 14124000  MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>] ;  MSVBVM60.__vbaFreeStr  
007BFD2E   .  FFD6           CALL ESI                                   ;  <&MSVBVM60.__vbaFreeStr>  
007BFD30   .  6A 06          PUSH 6             ;  堆栈中压入 6  
007BFD32   .  8B4D E0        MOV ECX,DWORD PTR SS:[EBP-20]  
007BFD35   .  51             PUSH ECX             ;  硬盘序号转换成 10 进制字符串得到"-2133699881"  
007BFD36   .  FF15 18104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrI4>]     ;  MSVBVM60.__vbaStrI4  
007BFD3C   .  8BD0           MOV EDX,EAX  
007BFD3E   .  8D4D D4        LEA ECX,DWORD PTR SS:[EBP-2C]  
007BFD41   .  8B3D F8114000  MOV EDI,DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>] ;  MSVBVM60.__vbaStrMove  
007BFD47   .  FFD7           CALL EDI                                   ;  <&MSVBVM60.__vbaStrMove>  
007BFD49   .  50             PUSH EAX           ;  上面堆栈中压入 6，这里取硬盘序号的前 6 位"-21336"  
007BFD4A   .  FF15 E4114000  CALL DWORD PTR DS:[<&MSVBVM60.#616>]         ;  MSVBVM60.rtcLeftCharBstr  
007BFD50   .  8BD0           MOV EDX,EAX  
007BFD52   .  8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]  
007BFD55   .  FFD7           CALL ESI                                   ;  <&MSVBVM60.__vbaStrMove>  
007BFD57   .  8D4D D4        LEA ECX,DWORD PTR SS:[EBP-2C]  
007BFD5A   .  FFD6           CALL ESI                                   ;  <&MSVBVM60.__vbaFreeStr>  
007BFD5C   .  FF15 78104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaExitProc>] ;  MSVBVM60.__vbaExitProc  
007BFD62   .  68 A7FD7B00    PUSH 音乐海盗.007BFDA7  
007BFD67   .  EB 34          JMP SHORT 音乐海盗.007BFD9D  
007BFD69   .  BA 14FF4000    MOV EDX,音乐海盗.0040FF14  
007BFD6E   .  8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]  
007BFD71   .  FF15 9C114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCopy>]  ;  MSVBVM60.__vbaStrCopy  
007BFD77   .  FF15 78104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaExitProc>] ;  MSVBVM60.__vbaExitProc  
007BFD7D   .  68 A7FD7B00    PUSH 音乐海盗.007BFDA7  
007BFD82   .  EB 19          JMP SHORT 音乐海盗.007BFD9D  
007BFD84   .  F645 F4 04     TEST BYTE PTR SS:[EBP-C],4  
007BFD88   .  74 09          JE SHORT 音乐海盗.007BFD93  
007BFD8A   .  8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]  
007BFD8D   .  FF15 14124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr  
007BFD93   >  8D4D D4        LEA ECX,DWORD PTR SS:[EBP-2C]  
007BFD96   .  FF15 14124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr  
007BFD9C   .  C3             RETN  
007BFD9D   >  8D4D DC        LEA ECX,DWORD PTR SS:[EBP-24]  
007BFDA0   .  FF15 14124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr  
007BFDA6   .  C3             RETN  
```

这段代码是：

取程序所在分区的序列号：`80D2-52D7`

转为 10 进制字符串得到：`"-2133699881"`

再取前 6 位得到：`"-21336"`

跟出 `RETN` 继续对`"-21336"`处理：

```
007C08F0   >  6A 01         PUSH 1             ;  取出"-21336"的第一位"-"  
007C08F2   .  8B45 D4       MOV EAX,DWORD PTR SS:[EBP-2C]  
```



# 汇编代码分析

```
007C08F5   .  50            PUSH EAX  
007C08F6   .  8B3D E4114000 MOV EDI,DWORD PTR DS:[<&MSVBVM60.#616>]         ;  MSVBVM60.rtcLeftCharBstr  
007C08FC   .  FFD7          CALL EDI                                         ;  <&MSVBVM60.#616>  
007C08FE   .  8BD0          MOV EDX,EAX  
007C0900   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  
007C0903   .  FFD3          CALL EBX  
007C0905   .  50            PUSH EAX  
007C0906   .  68 84FA4000   PUSH 音乐海盗.0040FA84  
007C090B   .  FFD6          CALL ESI                          ;  判断第一位是不是"-"  
007C090D   .  8BF0          MOV ESI,EAX  
007C090F   .  F7DE          NEG ESI  
007C0911   .  1BF6          SBB ESI,ESI  
007C0913   .  46            INC ESI  
007C0914   .  F7DE          NEG ESI  
007C0916   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  
007C0919   .  FF15 14124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]     ;  MSVBVM60.__vbaFreeStr  
007C091F   .  66:85F6       TEST SI,SI  
007C0922   .  74 2F         JE SHORT 音乐海盗.007C0953                 ;  不是"-"就跳走  
007C0924   .  68 BC0F4100   PUSH 音乐海盗.00410FBC  
007C0929   .  6A 05         PUSH 5          ;  取后 5 位"21336"  
007C092B   .  8B4D D4       MOV ECX,DWORD PTR SS:[EBP-2C]  
007C092E   .  51            PUSH ECX  
007C092F   .  FF15 F0114000 CALL DWORD PTR DS:[<&MSVBVM60.#618>]           ;  MSVBVM60.rtcRightCharBstr  
007C0935   .  8BD0          MOV EDX,EAX  
007C0937   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  
007C093A   .  FFD3          CALL EBX  
007C093C   .  50            PUSH EAX               ;  "1"+后 5 位，得到"121336"  
007C093D   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]    ;  MSVBVM60.__vbaStrCat  
007C0943   .  8BD0          MOV EDX,EAX  
007C0945   .  8D4D D4       LEA ECX,DWORD PTR SS:[EBP-2C]  
007C0948   .  FFD3          CALL EBX  
007C094A   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  
007C094D   .  FF15 14124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]   ;  MSVBVM60.__vbaFreeStr  
```

这里就是如果硬盘序号是负数，那么就用`"1"`替换掉`"-"`号。

我电脑上得到的是`"-21336"`，前面有`"-"`用`"1"`换掉，得到`"121336"`。

如果前面没有`"-"`则没有这个操作。

```
007C0953   >  6A 03         PUSH 3          ; 取"121336"的后三位"336"  
007C0955   .  8B55 D4       MOV EDX,DWORD PTR SS:[EBP-2C]  
007C0958   .  52            PUSH EDX  
007C0959   .  FF15 F0114000 CALL DWORD PTR DS:[<&MSVBVM60.#618>]           ;  MSVBVM60.rtcRightCharBstr  
007C095F   .  8BD0          MOV EDX,EAX  
007C0961   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]  
007C0964   .  FFD3          CALL EBX  
007C0966   .  50            PUSH EAX  
007C0967   .  6A 03         PUSH 3          ;  取"121336"的前三位"121"  
007C0969   .  8B45 D4       MOV EAX,DWORD PTR SS:[EBP-2C]  
007C096C   .  50            PUSH EAX  
007C096D   .  FFD7          CALL EDI  
007C096F   .  8BD0          MOV EDX,EAX  
007C0971   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]  
007C0974   .  FFD3          CALL EBX  
007C0976   .  50            PUSH EAX               ;  "336"+"121"="336121"  
007C0977   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]    ;  MSVBVM60.__vbaStrCat  
007C097D   .  8BD0          MOV EDX,EAX  
007C097F   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]  
007C0982   .  FFD3          CALL EBX  
007C0984   .  8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]  
007C0987   .  51            PUSH ECX  
007C0988   .  8D55 C8       LEA EDX,DWORD PTR SS:[EBP-38]  
007C098B   .  52            PUSH EDX  
007C098C   .  6A 02         PUSH 2  
007C098E   .  FF15 A8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStrList>] ;  MSVBVM60.__vbaFreeStrList  
007C0994   .  83C4 0C       ADD ESP,0C  
007C0997   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]  
007C099A   .  50            PUSH EAX               ;  字符串－>整数,"336121"得到 0x520F9  
007C099B   .  FF15 A0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Str>]      ;  MSVBVM60.__vbaI4Str  
007C09A1   .  69C0 FE010000 IMUL EAX,EAX,1FE                 ;  0x520F9 × 0x1FE = 0xA37B00E  
007C09A7   .  0F80 90080000 JO 音乐海盗.007C123D  
007C09AD   .  50            PUSH EAX               ;  再转为 10 进制字符串得到"171421710"  
007C09AE   .  FF15 18104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrI4>]      ;  MSVBVM60.__vbaStrI4  
```

上面拿`"121336"`的前三位和后三位交换得到`"336121"`。

`336121 × 510 = 171421710`

得到的`"171421710"`就是我机器上的注册码了。

---

## 破解心得

个人觉得 `VB` 程序的流程中不断地在调用 `VB` 的函数库，只要熟悉一些常用的函数库，那么它的注册流程基本上就暴露出来的。

- 中断地址：`7C09B4`
- 中断次数：`1`
- 第一字节：`8B`
- 指令长度：`2`
- 注册码保存为：内存方式
- 寄存器：`EAX`
- 记得选上：宽字符串

算法比较简单，算法注册机懒得写了。

---

## 声明

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享 :)

**【版权声明】** 本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

---

文章写于 `2005-10-18 18:28:50`

`4dd466d0e1e0cc123afbe504da2462ca`
