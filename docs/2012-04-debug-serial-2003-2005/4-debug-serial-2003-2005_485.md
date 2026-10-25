
# 书香门第 V1.23 Build 1638

> 原文：[`www.pediy.com/kssd/pediy05/pediy50487.htm`](https://www.pediy.com/kssd/pediy05/pediy50487.htm)

## 基本信息

软件大小： 665 KB  
软件语言： 简体中文  
软件类别： 国产软件 / 共享版 / 电子阅读  
应用平台： `Win9x/NT/2000/XP`  
界面预览：  
加入时间： 2003-03-13 08:28:43  
下载次数： 7419  
推荐等级：  
联系人： `gentlebreeze@vip.163.com`  
开发商： [`www.gentle-breeze.com`](http://www.gentle-breeze.com)

## 软件介绍

《书香门第》是一款适合于真正读书迷的电子小说、文本阅读软件，它外表并不花哨，但对于长时间、大量阅读的读书迷，却最舒适、体贴、细致，因为它具有十二个鲜明特点：

1. 多达 27 种各种质感的窗口背景、页面背景可供选择，总共超过 700 种背景组合，为读书迷提供最高舒适度和最大程度的视力保护。
2. 强大、智能化的自动排版功能，并可以随意设定字体大小、颜色、行距、标题行。
3. 极其高速的排版速度：目前主流机器上排版速度超过一万页/秒，所以通常你根本无法感觉到排版过程。
4. 高速的 `DirectDraw` 图形引擎，翻页寻迹流畅自如。
5. 附带的 `html` 转换、合成工具能够迅速依次将一批 `html` 文件转换且合并为一个大的文本文件，方便阅读。
6. 宽广的平台适用性：从 `486/win95` 到最新 `P4/XP` 都能从容应对。
7. 体贴的左手键操作，使你从此摆脱长期右手操作鼠标、键盘带来的疲劳。
8. 与页面字数成正比的自动翻页间隔，自然优于呆板的固定翻页间隔。
9. 可以选择使用窗口模式（寻迹方便）或全屏幕模式（阅读效果更好）。
10. 全书遍历/测试功能，保护你的计算机，节约能源。
11. 搜索功能方便读者在书中查找。
12. 具有强于 `word` 和 `IE` 的汉字乱码纠错功能。

## 作者声明

本人是个初学者，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

本人感谢软件作者,请见谅了.

## 破解工具

`0llydbg v1.09cn` `W32Dasm8.93 黄金版`

---

## 过程

### 1. 使用 W32Dasm8.93 黄金版反汇编

查找关键提示。

```
0040373C E81A6C0100           call 0041A35B
:00403741 83F818             cmp eax, 00000018
                             =====>注册码是否 24 位

:00403744 0F85C0000000       jne 0040380A
:0040374A A1280E4500         mov eax, dword ptr [00450E28]
:0040374F 83F804             cmp eax, 00000004
                             ====>E-mail 不能少于 4 位

:00403752 0F8CB2000000       jl 0040380A
:00403758 8364241000         and dword ptr [esp+10], 00000000
:0040375D EB05               jmp 00403764

* Reference To: USER32.SendMessageA, Ord:0214h
                          |
:004037B0 FF15F4444200       Call dword ptr [004244F4]
:004037B6 80BEC000000000     cmp byte ptr [esi+000000C0], 00
:004037BD 742F               je 004037EE
* Possible StringData Ref from Data Obj ->"  祝贺你       <=== 关键词,在上面中断
                                      ->" 你已经成为了《书香门第》注册用户！"
:004037BF 6898C64200         push 0042C698
```

### 2. 使用 0llydbg v1.09cn 跟踪

填 `e_mail`: `fxyang@163.com`  
试验码: `789456123012345678901234`

```
00403712  PUSH    EDI
00403713  MOV     EDI, zbook.00452268            ;  ASCII "fxyang@163.com"
00403718  LEA     EBP, DWORD PTR DS:[ESI+13C]
0040371E  PUSH    20
00403720  PUSH    EDI                             ;  EDI=00452268 ,ASCII "fxyang@163.com"
00403721  MOV     ECX, EBP
00403723  CALL    zbook.0041A35B
00403728  LEA     EBX, DWORD PTR DS:[ESI+100]
0040372E  PUSH    20
00403730  PUSH    zbook.00451A48                  ;  ASCII "789456123012345678901234"
00403735  MOV     ECX, EBX
00403737  MOV     DWORD PTR DS:[450E28], EAX
0040373C  CALL    zbook.0041A35B
00403741  CMP     EAX, 18
00403744  JNZ     zbook.0040380A
0040374A  MOV     EAX, DWORD PTR DS:[450E28]
0040374F  CMP     EAX, 4
00403752  JL      zbook.0040380A
00403758  AND     DWORD PTR SS:[ESP+10], 0
0040375D  JMP     SHORT zbook.00403764
0040375F  MOV     EAX, DWORD PTR DS:[450E28]
00403764  PUSH    EAX                             ;  EAX=E
00403765  IMUL    EAX, DWORD PTR SS:[ESP+14]      ;  EAX=EAX*SS:[12F454]=E*0=0||EAX*SS:[12F454]=E*01=E
0040376A  ADD     EAX, zbook.00451A68             ;  ASCII "fxyang@163.comfxyang@163.com.......
0040376F  PUSH    EDI                             ;  EDI=00452268 ,ASCII "fxyang@163.com"
00403770  PUSH    EAX                             ;  EAX=00451A76
00403771  CALL    zbook.004083E0                  ;  把 e_mail 传送到 451A68 开始的地址中一共 40h 组
00403776  ESP, 0C
00403779  INC     DWORD PTR SS:[ESP+10]           ;  SS:[12F454]=0 ++
0040377D  CMP     DWORD PTR SS:[ESP+10], 40       ;  SS:[12F454]=1 ++
00403782  JL      SHORT zbook.0040375F            ;  把 e_mail 传送到 451A68 开始的地址中一共 40h 组连接起来
00403784  MOV     EDI, zbook.00451248             ;  ASCII "789456123012345678901234"
00403789  PUSH    20
0040378B  PUSH    zbook.00451A48                  ;  ASCII "789456123012345678901234"
00403790  PUSH    EDI
00403791  CALL    zbook.004083E0
00403796  ADD     EDI, 20
00403799  ADD     ESP, 0C
0040379C  CMP     EDI, zbook.00451A48             ;  ASCII "789456123012345678901234"
004037A2  JL      SHORT zbook.00403789            ;  把试验码传送到 451248-451A48 开始的内存中
004037A4  XOR     EDI, EDI
004037A6  PUSH    EDI                             ; /lParam => 0
004037A7  PUSH    EDI                             ; |wParam => 0
004037A8  PUSH    1501                            ; |Message = MSG(1501)
004037AD  PUSH    DWORD PTR DS:[ESI+1C]           ; ||hWnd = B025C
004037B0  CALL    DWORD PTR DS:[<&USER32.SendMessageA>] ; \这个 api 是转移到计算注册码地址的 call
004037B6  CMP     BYTE PTR DS:[ESI+C0], 0         ;  DS:[ESI+C0]=DS:[12FAF4]注册标记
004037BD  JE      SHORT zbook.004037EE            ;  这里跳到错误的提示框
004037BF  PUSH    zbook.0042C698                  ;  到这里就成功
004037C4  MOV     DWORD PTR DS:[452A68], EDI
004037CA  CALL    zbook.00405666
```

`//因为上面的跳转是通过 API 的方式,所以太难找到核心`

`=============================================================================`

`CALL`

`DWORD PTR DS:[<&USER32.SendMessageA>] ; \这个 api 是转移到计算注册码地址的 call`

`|`

`|`

`77DF1D07  CALL DWORD PTR SS:[EBP+8]`

`|`

`CALL DWORD PTR SS:[EBP+8]`

`|`

### 3. 核心代码分析

```
00403860  MOV     EBP, ESP
00403862  SUB     ESP, 100
00403868  PUSH    ESI
00403869  PUSH    20
0040386B  CALL    zbook.004088FB
00403870  PUSH    40
00403872  MOV     ESI, zbook.00451148
00403877  CDQ
00403878  POP     ECX                           ;  ECX=40
00403879  IDIV    ECX                           ;  EAX=13CA  IDIV ECX=40  ==>EAX=4F   EDX=A
0040387B  SHL     EDX, 5                        ;  EDX=A SHL 5=140
0040387E  ADD     EDX, zbook.00451248           ;  EDX=EDX+451248( ASCII "789456123012345678901234")=
00403884  PUSH    EDX                           ;  EDX=zbook.00451388,ASCII "789456123012345678901234")
00403885  PUSH    ESI                           ;  //经过上面的漫长的转折终于来到试验码的地址
00403886  CALL    zbook.004083E0                ;  00451148 <==ASCII "789456123012345678901234")
0040388B  PUSH    0FF
00403890  LEA     EAX, [LOCAL.64]
00403896  PUSH    0
00403898  PUSH    EAX
00403899  CALL    zbook.004087A0
0040389E  ADD     ESP, 18
004038A1  PUSH    DWORD PTR DS:[450E28] ;  DS:[450E28]=0E (e_mail 的长度)
004038A7  CALL    zbook.004088FB                ;  这个 CALL 是取 EAX=69F9 (常数)
004038AC  PUSH    40
004038AE  CDQ
004038AF  POP     ECX
004038B0  IDIV    ECX                           ;  EAX=69F9  IDIV ECX=40  ==>EAX=1A7   EDX=39
004038B2  LEA     EAX, [LOCAL.64]
004038B8  IMUL    EDX, DWORD PTR DS:[450E28] ;  EDX=EDX*DS:[450E28]=39*0E=31E
004038BF  ADD     EDX, zbook.00451A68           ;  ASCII "fxyang@163.comfxyang@163.com
004038C5  PUSH    EDX                             ;  zbook.00451D86
004038C6  PUSH    EAX                             ;  //用上面的方法取计算的参数从 451D86 处做基数
004038C7  CALL    zbook.004083E0
004038CC  PUSH    0C
004038CE  PUSH    ESI                             ;  ESI=00451148 ,ASCII "789456123012345678901234")
004038CF  PUSH    zbook.004510C8                  ;  ASCII "骧溻滗怃纩祧"
004038D4  CALL    zbook.004083E0                  ;  取试验码的前 12 位==>004510C8
004038D9  PUSH    0C
004038DB  PUSH    zbook.00451154
004038E0  PUSH    zbook.00451048
004038E5  CALL    zbook.004083E0                  ;  取试验码的后 12 位==>00451048
004038EA  MOV     EAX, DWORD PTR DS:[450E28];  EAX=E
004038EF  AND     BYTE PTR DS:[451054], 0
004038F6  AND     BYTE PTR DS:[4510D4], 0
004038FD  ADD     ESP, 24
00403900  XOR     EDX, EDX
00403902  TEST    EAX, EAX
00403904  JLE     SHORT zbook.00403934
00403906  PUSH    EBX
00403907  PUSH    EDI
```



### 汇编代码段

```
00403908  LEA      EBX, DWORD PTR DS:[EAX-1]     ; EBX=E-1=D
0040390B /LEA      EDI, DWORD PTR SS:[EBP+EDX-100] ; EDI<==0012F21C,(ASCII "fxyang@163.com")
00403912 |MOV      CL, BYTE PTR DS:[EDI]         ; CL=DS:[12F21C]=66 ('f')
00403914 |CMP      CL, 40
00403917 |JE       SHORT zbook.0040391E
00403919 |CMP      CL, 2E
0040391C |JNZ      SHORT zbook.0040392D
0040391E |CMP      EDX, EBX
00403920 |JGE      SHORT zbook.0040392B
00403922 |MOV      ECX, EBX
00403924 |LEA      ESI, DWORD PTR DS:[EDI+1]
00403927 |SUB      ECX, EDX
00403929 |REP      MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]
0040392B |DEC      EAX
0040392C |DEC      EBX
0040392D |INC      EDX
0040392E |CMP      EDX, EAX
00403930 \JL       SHORT zbook.0040390B          ; 这一段循环把 email 中的@和.取出
00403932  POP      EDI                             //只取 email 的前 12 位
00403933  POP      EBX
00403934  CMP      EAX, 0C
00403937  POP      ESI
00403938  JGE      SHORT zbook.00403952
0040393A  PUSH     0C
0040393C  POP      ECX
0040393D  SUB      ECX, EAX
0040393F  LEA      EAX, DWORD PTR SS:[EBP+EAX-100]
00403946  PUSH     ECX
00403947  PUSH     30
00403949  PUSH     EAX
0040394A  CALL     zbook.004087A0
0040394F  ADD      ESP, 0C
00403952  AND      BYTE PTR SS:[EBP-F4], 0
00403959  CALL     zbook.004039A5                ; 关键的计算部分
0040395E  PUSH     0C
00403960  LEA      EAX, [LOCAL.64]               ; EAX<==0012F21C,(ASCII "fxyang163com")
00403966  PUSH     zbook.004511C8                ; 004511C8<== E6 E4 E4 E2 E6 EC F7 F5 F1 FD F5 FF
0040396B  PUSH     EAX                            ; EAX<==0012F21C,(ASCII "fxyang163com")
0040396C  CALL     zbook.00408E10                ; 应该是比较部分
00403971  ADD      ESP, 0C
00403974  TEST     EAX, EAX                       ; 关键的比较
00403976  JNZ      SHORT zbook.00403993           ; 关键的跳转
00403978  AND      BYTE PTR DS:[450C23], AL
0040397E  AND      BYTE PTR DS:[450C22], AL
00403984  AND      BYTE PTR DS:[450C21], AL
0040398A  AND      BYTE PTR DS:[450C20], AL
00403990  INC      EAX
00403991  LEAVE
00403992  RETN
00403993  MOV      BYTE PTR DS:[450C22], 78       ; 标记
0040399A >MOV      BYTE PTR DS:[450C23], 79
004039A1  XOR      EAX, EAX
004039A3  LEAVE
```

---

### 调用 `zbook.004039A5`

```
CALL     zbook.004039A5      ; 关键的计算部分
|
004039A5   PUSH     0C
004039A7   CALL     zbook.004039AE
004039AC   POP      ECX
004039AD   RETN
```

---

### 调用 `zbook.004039AE`

```
CALL     zbook.004039AE      终于来到计算的地方,头发炸了
|
|
004039AE  PUSH     ESI
004039AF  MOV      ESI, DWORD PTR SS:[ESP+8]     ; ESI=0C (email 去掉@和.后的长度)
004039B3  XOR      ECX, ECX
004039B5  PUSH     EDI
004039B6  TEST     ESI, ESI
004039B8  JLE      SHORT zbook.004039DA           //第一次的关键计算
004039BA  /LEA     EAX, DWORD PTR DS:[ECX+ESI]    ; EAX=ECX+ESI=0 ++ C=C++
004039BD  |PUSH    9
004039BF  |CDQ
004039C0  |POP     EDI                             ; EDI=9
004039C1  |IDIV   EDI                               ; EAX=C IDIV EDI=9 ==>EAX=1  EDX=3
004039C3  |MOV     AL, 8
004039C5  |SUB     AL, DL                           ; AL=8-3=5
004039C7  |MOV     BYTE PTR DS:[ECX*2+451148], DL   ; 00451148=03
004039CE  |MOV     BYTE PTR DS:[ECX*2+451149], AL   ; AL=05
004039D5  |INC     ECX
004039D6  |CMP     ECX, ESI                         ; 计算的次数=12
004039D8  \JL      SHORT zbook.004039BA
004039DA  AND      BYTE PTR DS:[ESI*2+451148], 0
004039E2  PUSH     EBX
004039E3  PUSH     EBP
004039E4  LEA      EDI, DWORD PTR DS:[ESI+451148]
004039EA  PUSH     ESI                               ; ESI=0C
004039EB  MOV      EBP, zbook.00450F48
004039F0  PUSH     EDI                               ;   zbook.00451154
004039F1  PUSH     EBP
004039F2  CALL     zbook.004083E0

; 把后 12 位==>00450F48 (00 08 01 07 02 06 03 05 04 04 05 03)
004039F7  MOV      EBX, zbook.00451148
004039FC  PUSH     ESI
004039FD  PUSH     EBX
004039FE  PUSH     EDI
004039FF  CALL     zbook.004083E0

; 把前 12 位==>00451145 (03 05 04 04 05 03 06 02 07 01 08 00)
00403A04  PUSH     ESI
00403A05  PUSH     EBP
00403A06  PUSH     EBX
00403A07  CALL     zbook.004083E0

; 把前 12 位倒置后==>00451148 (00 08 01 07 02 06 03 05 04 04 05 03)
00403A0C  ADD      ESP, 24
00403A0F  XOR      EBX, EBX
00403A11  TEST     ESI, ESI
```

---

### 第一次计算的总结

1.  计算的参数是一个固定值 `C`
2.  这段计算是 `C MOD 9` 的值设 `M` 然后用 `8-M` 得到值设 `N`，再把 `C++` 继续
3.  取得到的数组的后 12 位与前 12 位值倒置后的数组连接成 24 位的数组
4.  得到一组值==> `00451148`

    `N=00 08 01 07 02 06 03 05 04 04 05 03 03 05 04 04 05 03 06 02 07 01 08 00`

5.  就是说上面的数组是固定的.

```
004039DA  AND      BYTE PTR DS:[ESI*2+451148], 0
004039E2  PUSH     EBX
004039E3  PUSH     EBP
004039E4  LEA      EDI, DWORD PTR DS:[ESI+451148]
004039EA  PUSH     ESI                           ;  ESI=0C
004039EB  MOV      EBP, zbook.00450F48
004039F0  PUSH     EDI                           ;   zbook.00451154
004039F1  PUSH     EBP
004039F2  CALL     zbook.004083E0               ;  把后 12 位==>00450F48 (00 08 01 07 02 06 03 05 04 04 05 03)
004039F7  MOV      EBX, zbook.00451148
004039FC  PUSH     ESI
004039FD  PUSH     EBX
004039FE  PUSH     EDI
004039FF  CALL     zbook.004083E0               ;  把前 12 位==>00451145 (03 05 04 04 05 03 06 02 07 01 08 00)
00403A04  PUSH     ESI
00403A05  PUSH     EBP
00403A06  PUSH     EBX
00403A07  CALL     zbook.004083E0               ;  把前 12 位倒置后==>00451148 (00 08 01 07 02 06 03 05 04 04 05 03)
00403A0C  ADD      ESP, 24
00403A0F  XOR      EBX, EBX
00403A11  TEST     ESI, ESI
00403A13  JLE      SHORT zbook.00403A66         //第二次的关键计算:
00403A15  /MOV     AL, BL
00403A17 |MOV      CL, BYTE PTR DS:[EBX+4510C8] ;  CL <==DS:[EBX+4510C8]=37 ('7') 试验码的前 12 位"789456123012"
00403A1D |INC      AL
00403A1F |MOV      DL, BYTE PTR DS:[EBX+451048] ;  DL <==DS:[EBX+451048]=33 ('3') 试验码的后 12 位"345678901234"
00403A25 |IMUL     BL                              ;  BL=0
00403A27 |LEA      EDI, DWORD PTR DS:[EBX+EBX]
00403A2A |INC      AL
00403A2C |AND      AL, 0F
00403A2E |SUB      CL, BYTE PTR DS:[EDI+451148]    ;  CL=CL-DS:[451148]=37-00=37||=38-01(第一次计算的第三位)=37
00403A34 |SUB      DL, BYTE PTR DS:[EDI+451149]    ;  DL=DL-DS:[451149]=33-08=2B
00403A3A |MOV      BYTE PTR DS:[450F40], AL
00403A3F |SUB      CL, 41                          ;  CL=CL-41=37-41=F6
00403A42 |SUB      DL, 41                          ;  DL=DL-41=2B-41=EA
00403A45 |XOR      CL, AL                          ;  CL=CL XOR AL=F6 XOR 01=F7
00403A47 |XOR      DL, AL                          ;  DL=DL XOR AL=EA XOR 01=EB
00403A49 |MOV      BYTE PTR DS:[EBX+4510C8], CL    ;  //得到新值 DS:[4510C8]<==( F7 F5 F1 FD F5 FF  E6 E4 E4 E2 E6 EC )
00403A4F |MOV      BYTE PTR DS:[EBX+451048], DL    ;  //得到新值 DS:[451048]<==( EB EF E9 FD F7 FB F8 E2 E4 E4 FE F6 )
00403A55 |INC      EBX                             ;  EBX++
00403A56 |MOV      BYTE PTR DS:[EDI+450F48], CL    ;  CL =F7
00403A5C |CMP      EBX, ESI                        ;  计算的次数=C (12D)
00403A5E |MOV      BYTE PTR DS:[EDI+450F49], DL    ;  //把试验码的前 12 位
00403A64 \JL      SHORT zbook.00403A15

00403A66  PUSH     ESI                              ;  ESI=C
00403A67  PUSH     EBP                              ;  上面的新值地址
00403A68  PUSH     zbook.00451048                  ;  上面的新值的第二部分地址
00403A6D  CALL     zbook.004083E0                  ;  第一部分的值==>00451048
00403A72  LEA      EDI, DWORD PTR DS:[ESI+450F48]  ;  EDI=00450F54,后 12 位地址
00403A78  PUSH     ESI
```



## 汇编代码段 1

```asm
00403A79  MOV     EBX, zbook.004510C8          ;  ASCII "骧溻滗怃纩祧"
00403A7E  PUSH    EDI
00403A7F  PUSH    EBX                           ;  第一部分的值==>004510C8
00403A80  CALL    zbook.004083E0                ;  后 12 组==>004510C8
00403A85  PUSH    ESI
00403A86  PUSH    EBX
00403A87  PUSH    EBP
00403A88  CALL    zbook.004083E0                ;  连接后的后 12 位==>00450F48
00403A8D  PUSH    ESI
00403A8E  PUSH    zbook.00451048                ;  连接后的后 12 位==>00450F48
00403A93  PUSH    EDI
00403A94  CALL    zbook.004083E0                ;  连接后的前 12 位==>00450F54
00403A99  ADD     ESP, 30
00403A9C  XOR     EAX, EAX
00403A9E  TEST    ESI, ESI
00403AA0  POP     EBP
00403AA1  POP     EBX
00403AA2  JLE     SHORT zbook.00403AC0
00403AA4 /MOV      CL, BYTE PTR DS:[EAX*2+450F49] ;  CL<==DS:[450F49]=F8
00403AAB |SHL      CL, 4                         ;  CL=CL SHL 4=F8 SHL 4 =80
00403AAE |OR       CL, BYTE PTR DS:[EAX*2+450F48] ;  CL= CL OR DS:[450F48]=80 OR E6=E6
00403AB5 |INC      EAX                           ;  EAX++
00403AB6 |CMP      EAX, ESI                      ;  ESI=C
00403AB8 |MOV      BYTE PTR DS:[EAX+4511C7], CL ;  CL==>DS:[4511C8]=E6
00403ABE \JL       SHORT zbook.00403AA4          //循环取值
00403AC0  AND     BYTE PTR DS:[ESI+4511C8], 0
;  004511C8<== E6 E4 E4 E2 E6 EC F7 F5 F1 FD F5 FF
00403AC7  POP     EDI
00403AC8  POP     ESI
00403AC9  RETN
```

## 汇编代码段 2

```asm
CALL    zbook.00408E10              ;  应该是比较部分

00408E10  MOV     EAX, DWORD PTR SS:[ESP+C]
00408E14  TEST    EAX, EAX          ;  EAX=C
00408E16  JE      SHORT zbook.00408E5C
00408E18  MOV     EDX, DWORD PTR SS:[ESP+4]

;  EDX<==0012F21C,(ASCII "fxyang163com")

00408E1C  PUSH    ESI
00408E1D  PUSH    EDI
00408E1E  MOV     ESI, EDX

;  ESI<==0012F21C,(ASCII "fxyang163com")

00408E20  MOV     EDI, DWORD PTR SS:[ESP+10]

;  EDI<==004511C8== E6 E4 E4 E2 E6 EC F7 F5 F1 FD F5 FF

00408E24  OR      EDX, EDI
00408E26  AND     EDX, 3
00408E29  JE      SHORT zbook.00408E5D
00408E2B  TEST    EAX, 1
00408E30  JE      SHORT zbook.00408E3D
00408E32  MOV     CL, BYTE PTR DS:[ESI]
00408E34  CMP     CL, BYTE PTR DS:[EDI]
00408E36  JNZ     SHORT zbook.00408E8A
00408E38  INC     ESI
00408E39  INC     EDI
00408E3A  DEC     EAX
00408E3B  JE      SHORT zbook.00408E5A
00408E3D /MOV      CL, BYTE PTR DS:[ESI]      ; 逐位比较,相同就成功(可能吗?)
00408E3F |MOV      DL, BYTE PTR DS:[EDI]
00408E41 |CMP      CL, DL
00408E43 |JNZ      SHORT zbook.00408E8A
00408E45 |MOV      CL, BYTE PTR DS:[ESI+1]
00408E48 |MOV      DL, BYTE PTR DS:[EDI+1]
00408E4B |CMP      CL, DL
00408E4D |JNZ      SHORT zbook.00408E8A
00408E4F |ADD      EDI, 2
00408E52 |ADD      ESI, 2
00408E55 |SUB      EAX, 2
00408E58 \JNZ      SHORT zbook.00408E3D
00408E5A  POP     EDI
00408E5B  POP     ESI
00408E5C  RETN
00408E5D  MOV     ECX, EAX        ; EAX=C
00408E5F  AND     EAX, 3
00408E62  SHR     ECX, 2
00408E65  JE      SHORT zbook.00408E92      ;  上面不成立就到这里
00408E67  REPE    CMPS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI]

;  比较 fxya 和"E6E4E4E2"-前四位相同

00408E69  JE      SHORT zbook.00408E92

;  关键的跳转,相同就注册成功.完美暴破点

00408E6B  MOV     ECX, DWORD PTR DS:[ESI-4]   ;  ECX<==DS:[12F21C]=61797866
00408E6E  MOV     EDX, DWORD PTR DS:[EDI-4]   ;  EDX<==DS:[4511C8]=E2E4E4E6
00408E71  CMP     CL, DL
00408E73  JNZ     SHORT zbook.00408E85
00408E75  CMP     CH, DH
00408E77  JNZ     SHORT zbook.00408E85
00408E79  SHR     ECX, 10
00408E7C  SHR     EDX, 10
00408E7F  CMP     CL, DL
00408E81  JNZ     SHORT zbook.00408E85
00408E83  CMP     CH, DH
00408E85  MOV     EAX, 0
00408E8A  SBB     EAX, EAX
00408E8C  POP     EDI
00408E8D  SBB     EAX, -1
00408E90  POP     ESI
00408E91  RETN
```

## 第二次的关键计算的总结

1. 先把试验码 `789456123012345678901234` 分成二组每组 12 位
   1. 第一组: 前 12 位 `789456123012` 设为 `M0`
   2. 第二组: 后 12 位 `345678901234` 设为 `N0`
2. 第一组的 `hex` 值-第一次计算值的奇数位的值设为 `K0` ;第二组的 `hex` 值-第一次计算值的偶数位的值设为 `L0`
3. 把 `(K0-41)XOR{次数*(次数-1)+1}` 的个位设为 `P`
   `(L0-41)XOR{次数*(次数-1)+1}` 的个位设为 `Q` 然后把这个值按 `P Q P1 Q1 ..P12 Q12` 放在以 `00450F48` 开始的地址中
   `{次数*(次数-1)+1}` 的个位这个值是一组固定值设位 `H:1 3 7 D 5 F B 9 9 B F 5 D 7 1`
   `//得到新值 DS:[450F48]<==( F7 EB F5 EF F1 E9 FD FD F5 F7 FF FB E6 F8 E4 E2  E4 E4 E2 E4 E6 FE EC F6 )`
4. 把后 12 位与前 12 位对换 `==>E6 F8 E4 E2 E4 E4 E2 E4 E6 FE EC F6 F7 EB F5 EF F1 E9 FD FD F5 F7 FF`
5. 然后取奇数位得到一个 12 位的新数组 `==>E6 E4 E4 E2 E6 EC F7 F5 F1 FD F5 FF`
   实际上就是把 `P` 组的前六位与后六位对换就是需要的数组
6. 取得到的数组的后 12 位与输入 `e_mail` 的前 12 位比较,相同就注册成功.
7. 如果不成功,就取数组的前四位和 `e_mail` 的前四位比较,相同就注册成功

**特别说明:** 上面的注册吗我没有办法可逆,可能我很菜.请大侠们帮忙了.

<div align="right">
`fxyang`<br>
`2003.3.15`  `4dd466d0e1e0cc123afbe504da2462ca`
</div>
