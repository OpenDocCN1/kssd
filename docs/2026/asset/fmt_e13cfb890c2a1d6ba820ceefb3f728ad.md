
# [原创]Nidesoft DVD to MP4 Converter 的注册分析

【课    题】: Nidesoft DVD to MP4 Converter 的注册分析

【作    者】: HailDuz

【软件名称】: Nidesoft DVD to MP4 Converter V5.X.XX

【软件大小】: 724992

【下载地址】: 658K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6A6k6r3g2K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3

【保护方式】: 注册码验证

【使用工具】: OD，UltraEdit32

【操作平台】: WinXP

【软件介绍】: 好像是一个DVD转MP4的工具。

【作者声明】: 纯技术问题，只是想试试自己的耐心，无其他目的。失误之处敬请诸位赐教!

--------------------------------------------------------------------------------

## 【过    程】

Nidesoft DVD to MP4 Converter是一个媒体转换的工具，需要注册码完成注册，保护程度一般，有一个爆破点可以省去注册码的回溯。本文主要分析其注册过程。

检查其主程序`DVDRipper.exe`，无壳，VC6编译的。

OD载入，运行后标题显示未注册，提示输入注册信息：邮件地址，注册码。

任意输入些信息确认，提示注册码错误"Register code not correct,please check it!"，查找字符串看到 "Registration Successful"信息，来到`00416533`这里提示注册成功。

```
00416505  |.  51            PUSH ECX                                 ; /Arg2
00416506  |.  52            PUSH EDX                                 ; |Arg1
00416507  |.  8D8C24 4C0700>LEA ECX,DWORD PTR SS:[ESP+74C]           ; |
0041650E  |.  E8 7DD6FEFF   CALL DVDRippe.00403B90                   ; 关键Call 函数调用
00416513  |.  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]
00416517  |.  5F            POP EDI
00416518  |.  85C0          TEST EAX,EAX                             ；比较EAX是否小于等于0
0041651A  |.  5D            POP EBP
0041651B  |.  0F86 A7000000 JBE DVDRippe.004165C8                    ;  跳到注册失败1
00416521  |.  3D 9F860100   CMP EAX,1869F                           ; 比较EAX是否小于99999
00416526  |.  0F83 9C000000 JNB DVDRippe.004165C8                    ;  跳到注册失败2
0041652C  |.  6A 40         PUSH 40
0041652E  |.  68 E8E64400   PUSH DVDRippe.0044E6E8                   ;  ASCII "Nidesoft"
00416533  |.  68 D0E64400   PUSH DVDRippe.0044E6D0                   ;  ASCII "Registration Successful"
00416538  |.  8BCE          MOV ECX,ESI
0041653A  |.  E8 6D180200   CALL <JMP.&MFC42.#4224\_?MessageBoxA@CWnd>
```

在`00416518`处更改`EAX`值在0~99999的值，提示注册成功，主程序标题未注册信息消失。重启程序依然需要注册，又重启验证。检查程序后发现在这里：读该程序`Skins`目录的`ifo.bmp`的信息。（注册成功后追加到文件尾部，好像可以多次累加，不知设计人想要它长到多大，没试。）

```
00403E52  |> \8B46 04       MOV EAX,DWORD PTR DS:[ESI+4]
00403E55  |.  53            PUSH EBX
00403E56  |.  8B1D A4F64300 MOV EBX,DWORD PTR DS:[<&MSVCRT.fseek>]   ;  msvcrt.fseek
00403E5C  |.  6A 02         PUSH 2                                   ; /whence = SEEK\_END
00403E5E  |.  6A 00         PUSH 0                                   ; |offset = 0
00403E60  |.  50            PUSH EAX                                 ; |stream
00403E61  |.  FFD3          CALL EBX                                 ; \fseek
00403E63  |.  8B4E 04       MOV ECX,DWORD PTR DS:[ESI+4]
00403E66  |.  51            PUSH ECX                                 ; /stream
00403E67  |.  FF15 A0F64300 CALL DWORD PTR DS:[<&MSVCRT.ftell>]      ; \ftell
00403E6D  |.  8B56 04       MOV EDX,DWORD PTR DS:[ESI+4]
00403E70  |.  2BC7          SUB EAX,EDI
00403E72  |.  6A 00         PUSH 0
00403E74  |.  50            PUSH EAX
00403E75  |.  52            PUSH EDX
00403E76  |.  FFD3          CALL EBX
00403E78  |.  8B46 04       MOV EAX,DWORD PTR DS:[ESI+4]
00403E7B  |.  8B8E 10010000 MOV ECX,DWORD PTR DS:[ESI+110]
00403E81  |.  50            PUSH EAX                                 ; /stream
00403E82  |.  57            PUSH EDI                                  ; |n
00403E83  |.  6A 01         PUSH 1                                     ; |size = 1
00403E85  |.  51            PUSH ECX                                  ; |ptr
00403E86  |.  FF15 9CF64300 CALL DWORD PTR DS:[<&MSVCRT.fread>]      ; \fread
00403E8C  |.  83C4 2C       ADD ESP,2C                                 ;  读取了Ifo.bmp最后16个数据
00403E8F  |.  33C0          XOR EAX,EAX
00403E91  |.  85FF          TEST EDI,EDI
00403E93  |.  7E 1B         JLE SHORT DVDRippe.00403EB0
00403E95  |>  8B96 10010000 /MOV EDX,DWORD PTR DS:[ESI+110]
00403E9B  |.  8A1C02        |MOV BL,BYTE PTR DS:[EDX+EAX]
00403E9E  |.  8D0C02        |LEA ECX,DWORD PTR DS:[EDX+EAX]
00403EA1  |.  8A96 18010000 |MOV DL,BYTE PTR DS:[ESI+118]
00403EA7  |.  32DA          |XOR BL,DL
00403EA9  |.  40            |INC EAX
00403EAA  |.  3BC7          |CMP EAX,EDI
00403EAC  |.  8819          |MOV BYTE PTR DS:[ECX],BL
00403EAE  |.^ 7C E5         \JL SHORT DVDRippe.00403E95
00403EB0  |>  8BB6 10010000 MOV ESI,DWORD PTR DS:[ESI+110]           ;  反算出注册码，FF与AscII异或
00403EB6  |.  8BCF          MOV ECX,EDI
00403EB8  |.  8B7C24 10     MOV EDI,DWORD PTR SS:[ESP+10]
00403EBC  |.  8BC1          MOV EAX,ECX
00403EBE  |.  C1E9 02       SHR ECX,2
00403EC1  |.  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
00403EC3  |.  8BC8          MOV ECX,EAX
00403EC5  |.  5B            POP EBX
00403EC6  |.  83E1 03       AND ECX,3
00403EC9  |.  B8 01000000   MOV EAX,1
00403ECE  |.  F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>
00403ED0  |.  5F            POP EDI
00403ED1  |.  5E            POP ESI
00403ED2  \.  C2 0800       RETN 8
```

反溯到启动验证部分：

```
0040D090  /$  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0040D096  |.  6A FF         PUSH -1
```

```
0040D0E6  |.  8D8C24 C00200>LEA ECX,DWORD PTR SS:[ESP+2C0]
0040D0ED  |.  50            PUSH EAX                                 ;  Eax=1D（29）个
0040D0EE  |.  52            PUSH EDX                                 ;  Edx地址 对应为：Nidesoft-DVD-to-MP4-Converter
0040D0EF  |.  E8 2C69FFFF   CALL DVDRippe.00403A20                   ;  生成了一堆怪异的码，后来看是计算注册码用计算码表
0040D0F4  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]
```

```
0040D1B4  |.  51            PUSH ECX                                 ; /Arg2
0040D1B5  |.  52            PUSH EDX                                 ; |Arg1
0040D1B6  |.  8D8C24 C80200>LEA ECX,DWORD PTR SS:[ESP+2C8]           ; |
0040D1BD  |.  E8 CE69FFFF   CALL DVDRippe.00403B90                   ; \判断是否注册的关键Call
0040D1C2  |.  8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]
0040D1C6      85C0          TEST EAX,EAX
0040D1C8      76 25         JBE SHORT DVDRippe.0040D1EF
0040D1CA      3D 9F860100   CMP EAX,1869F                           ；同上面所述的注册判断
0040D1CF      73 1E         JNB SHORT DVDRippe.0040D1EF
0040D1D1  |.  8D86 5CD10100 LEA EAX,DWORD PTR DS:[ESI+1D15C]
0040D1D7  |.  8D8E 4CD10100 LEA ECX,DWORD PTR DS:[ESI+1D14C]
0040D1DD  |.  50            PUSH EAX
0040D1DE  |.  C786 50D10100>MOV DWORD PTR DS:[ESI+1D150],1
0040D1E8  |.  E8 89A80200   CALL <JMP.&MFC42.#858\_??4CString@@QAEABV>
0040D1ED  |.  EB 45         JMP SHORT DVDRippe.0040D234               ；跳到注册版
0040D1EF  |>  8D8E 5CD10100 LEA ECX,DWORD PTR DS:[ESI+1D15C]
0040D1F5  |.  68 40DA4400   PUSH DVDRippe.0044DA40                   ;  ASCII "(Unregistered)"；到这里就是未注册版
0040D1FA  |.  8D5424 14     LEA EDX,DWORD PTR SS:[ESP+14]
```
