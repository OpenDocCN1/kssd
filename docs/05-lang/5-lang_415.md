
# 水晶排课系统

> 原文：`www.pediy.com/kssd/pediy08/pediy8-781.htm` (链接)

【破文标题】菜鸟破解之四---水晶排课系统软件注册算法分析

【破文作者】紫色缘\[TFW\]\[PCG\]

【作者邮箱】Cn\_Fish@126.com

【作者主页】www.cniso.org

【破解工具】OD/红色警戒~~（没精神的时候就打打它，嘿嘿~~）

【破解平台】WinXP SP2

【软件大小】9MB

【原版下载】不提供~

【保护方式】无

【软件简介】本人觉得这款软件真的很不错

-------------------------------------------------------------------------

【破解过程】

未查壳直接`OD`载入，有提示压入壳加密，询问是否加以分析，点否后，进入如下代码处：

```
005EC710  >  60                pushad
005EC711   BE 00A05400       mov     esi, 0054A000
005EC716   8DBE 0070EBFF     lea     edi, [esi+FFEB7000]
005EC71C   57                push    edi
005EC71D   83CD FF          or      ebp, FFFFFFFF
005EC720   EB 10            jmp     short 005EC732
005EC722   90                nop
```

-------------------------------------------------------------------------

从压缩的特征看，不是 `UPX` 就是 `asp` 了，直接`CTRL+F`输入`popad`，到达如下代码处~

```
005EC869   /74 07           je      short 005EC872
005EC86B   |8903            mov     [ebx], eax
005EC86D   |83C3 04         add     ebx, 4
005EC870  ^|EB D8           jmp     short 005EC84A
005EC872   \\FF96 843A1F00  call    [esi+1F3A84]
005EC878   61                popad                       ; 此处直接 F2 后，F9 断在此，然后取消断点后，F8 下
005EC879  - E9 AE70E1FF     jmp     0040392C               ; F7 跟进
```

---------------跟进 5EC879 后，代码如下----------------------------------------------------------------------

```
0040392C   68 403D4100      push    00413D40
00403931   E8 F0FFFFFF      call    00403926               ; jmp 到 MSVBVM60.ThunRTMain
00403936   0000            add     [eax], al
00403938   0000            add     [eax], al
0040393A   0000            add     [eax], al
0040393C   3000            xor     [eax], al
0040393E   0000            add     [eax], al
00403940   40               inc     eax
00403941   0000            add     [eax], al
00403943   0000            add     [eax], al
00403945   0000            add     [eax], al
00403947   0070 B5         add     [eax-4B], dh
0040394A   0315 D9E0954F   add     edx, [4F95E0D9]
00403950   B3 2F            mov     bl, 2F                     ;   原来此软件是 VB 编写的~
```

---------------------------------------------------------------------------

-------------------------找到关键点后直接跟踪到此，代码如下-------------------------------------------------

```
005D0960   .  55               PUSH EBP                                ;  在此下断，F8 继续跟
005D0961   .  8BEC             MOV EBP,ESP
005D0963   .  83EC 08          SUB ESP,8
005D0966   .  68 06354000      PUSH <JMP.&MSVBVM60.__vbaExceptHandler>         ;  SE 处理程序安装
005D096B   .  64:A1 0000000>   MOV EAX,DWORD PTR FS:[0]
005D0971   .  50               PUSH EAX
005D0972   .  64:8925 00000>   MOV DWORD PTR FS:[0],ESP
.............................................省略 N 行代码~~...................................................
005D0B78   .  50               PUSH EAX
005D0B79   .  53               PUSH EBX
005D0B7A   .  8B13             MOV EDX,DWORD PTR DS:[EBX]
005D0B7C   .  FF92 A0000000    CALL DWORD PTR DS:[EDX+A0]                   ;  读取机器码
005D0B82   .  3BC7             CMP EAX,EDI                                 ;  2 者作为比较
005D0B84   .  DBE2             FCLEX
005D0B86   .  7D 12            JGE SHORT unpack.005D0B9A                  ;  大于或等于则跳
005D0B88   .  68 A0000000      PUSH 0A0
005D0B8D   .  68 10594200      PUSH unpack.00425910
005D0B92   .  53               PUSH EBX
005D0B93   .  50               PUSH EAX
005D0B94   .  FF15 7C104000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaHresultCheckObj>>;  MSVBVM60.__vbaHresultCheckObj
005D0B9A   >  8B45 DC           MOV EAX,DWORD PTR SS:[EBP-24]                   ;  机器码送入 EAX
005D0B9D   .  8D8D 40FFFFFF    LEA ECX,DWORD PTR SS:[EBP-C0]
005D0BA3   .  8D95 30FFFFFF    LEA EDX,DWORD PTR SS:[EBP-D0]
005D0BA9   .  BB 08000000      MOV EBX,8                                 ;  EBX=8
005D0BAE   .  51               PUSH ECX
005D0BAF   .  52               PUSH EDX
005D0BB0   .  897D DC          MOV DWORD PTR SS:[EBP-24],EDI
005D0BB3   .  8985 48FFFFFF    MOV DWORD PTR SS:[EBP-B8],EAX             ;  EAX（机器码）送入[EBP-B8]
005D0BB9   .  899D 40FFFFFF    MOV DWORD PTR SS:[EBP-C0],EBX             ;  EBX（8）送入[EBP-C0]
005D0BBF   .  FF15 0C114000    CALL DWORD PTR DS:[<&MSVBVM60.rtcUpperCaseVar>]      ;  MSVBVM60.rtcUpperCaseVar
005D0BC5   .  8B06             MOV EAX,DWORD PTR DS:[ESI]
005D0BC7   .  56               PUSH ESI
.............................................省略 N 行代码~~...................................................
005D10B9   .  52               PUSH EDX
005D10BA   .  50               PUSH EAX
005D10BB   .  C785 D8FBFFFF>   MOV DWORD PTR SS:[EBP-428],unpack.0042847C
005D10C5   .  FFD3             CALL EBX                                 ;  <&MSVBVM60.__vbaVarCat>
005D10C7   .  8D4D BC          LEA ECX,DWORD PTR SS:[EBP-44]
005D10CA   .  50               PUSH EAX
005D10CB   .  51               PUSH ECX
005D10CC   .  FF15 C8114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]      ;  开始取注册编号第 3 位，尾部加“0”
005D10D2   .  50               PUSH EAX                                 ;  EAX="-0"入栈
005D10D3   .  FF15 20104000    CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]    ;  转换为 16 进制
005D10D9   .  50               PUSH EAX                                 ;  EAX=2D（16 进制）
005D10DA   .  FF15 8C114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]         ;  转为 10 进制
005D10E0   .  8B35 7C124000    MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]     ;  MSVBVM60.__vbaStrMove
005D10E6   .  8BD0             MOV EDX,EAX                                 ;  EAX=45（转为 10 进制）送入 EDX
005D10E8   .  8D4D B0          LEA ECX,DWORD PTR SS:[EBP-50]
005D10EB   .  FFD6             CALL ESI                                   ;  <&MSVBVM60.__vbaStrMove>
005D10ED   .  50               PUSH EAX                                 ;  EAX 入栈
005D10EE   .  8D95 20FEFFFF    LEA EDX,DWORD PTR SS:[EBP-1E0]
005D10F4   .  8D85 50FCFFFF    LEA EAX,DWORD PTR SS:[EBP-3B0]
005D10FA   .  52               PUSH EDX
005D10FB   .  8D8D 10FEFFFF    LEA ECX,DWORD PTR SS:[EBP-1F0]
005D1101   .  50               PUSH EAX
005D1102   .  51               PUSH ECX
005D1103   .  FFD3             CALL EBX
005D1105   .  8D55 B4          LEA EDX,DWORD PTR SS:[EBP-4C]
005D1108   .  50               PUSH EAX
005D1109   .  52               PUSH EDX
005D110A   .  FF15 C8114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]      ;  取注册编号第 4 位，尾部加“0”
005D1110   .  50               PUSH EAX                                 ;  EAX=WO 入栈
005D1111   .  FF15 20104000    CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]    ;  转换为 16 进制
005D1117   .  50               PUSH EAX                                 ;  EAX=57（16 进制）
005D1118   .  FF15 8C114000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]         ;  转为 10 进制
005D111E   .  8BD0             MOV EDX,EAX                                 ;  EAX=87（转为 10 进制）送入 EDX
005D1120   .  8D4D AC          LEA ECX,DWORD PTR SS:[EBP-54]
005D1123   .  FFD6             CALL ESI
005D1125   .  50               PUSH EAX                                 ;  EAX 入栈
005D1126   .  FF15 68104000    CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]         ;  2 个 EAX 值连接
005D112C   .  8BD0             MOV EDX,EAX                                 ;  EAX="4587"送入 EDX
005D112E   .  8D4D A8          LEA ECX,DWORD PTR SS:[EBP-58]
005D1131   .  FFD6             CALL ESI
005D1133   .  50               PUSH EAX                                 ;  EAX 最后值入栈
```



```
005D1134   .   FF15 10124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Str>]            ; 10 进制的值连接后转换为 16 进制
005D113A   .   8BD0          MOV EDX,EAX                                             ; 然后 EAX=“11EB”送入 EDX
005D113C   .   8D85 10FFFFFF LEA EAX,DWORD PTR SS:[EBP-F0]
005D1142   .   50            PUSH EAX
005D1143   .   8D8D B0FCFFFF LEA ECX,DWORD PTR SS:[EBP-350]
005D1149   .   8D85 00FFFFFF LEA EAX,DWORD PTR SS:[EBP-100]
005D114F   .   51            PUSH ECX
005D1150   .   50            PUSH EAX
005D1151   .   8995 48FBFFFF MOV DWORD PTR SS:[EBP-4B8],EDX                          ; EDX 送入[EBP-4B8]
005D1157   .   FFD3         CALL EBX
005D1159   .   50            PUSH EAX
005D115A   .   8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]
005D115D   .   51            PUSH ECX
005D115E   .   FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]          ; 取注册编号第 1 位，尾部加“0”
005D1164   .   50            PUSH EAX                                                ; EAX=“W0”入栈
005D1165   .   FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]        ; 转换为 16 进制
005D116B   .   50            PUSH EAX                                                ; EAX=57（16 进制）
005D116C   .   FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]            ; 转为 10 进制
005D1172   .   8BD0          MOV EDX,EAX                                             ; EAX=87（转为 10 进制）送入 EDX
005D1174   .   8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]
005D1177   .   FFD6         CALL ESI
005D1179   .   50            PUSH EAX                                                ; EAX 入栈
005D117A   .   8D95 C0FEFFFF LEA EDX,DWORD PTR SS:[EBP-140]
005D1180   .   8D85 90FCFFFF LEA EAX,DWORD PTR SS:[EBP-370]
005D1186   .   52            PUSH EDX
005D1187   .   8D8D B0FEFFFF LEA ECX,DWORD PTR SS:[EBP-150]
005D118D   .   50            PUSH EAX
005D118E   .   51            PUSH ECX
005D118F   .   FFD3         CALL EBX
005D1191   .   8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]
005D1194   .   50            PUSH EAX
005D1195   .   52            PUSH EDX
005D1196   .   FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]          ; 取注册编号第 2 位，尾部加“0”
005D119C   .   50            PUSH EAX                                                ; EAX=“D0”入栈
005D119D   .   FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]        ; 转换为 16 进制
005D11A3   .   50            PUSH EAX                                                ; EAX=44（16 进制）
005D11A4   .   FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]            ; 转为 10 进制
005D11AA   .   8BD0          MOV EDX,EAX                                             ; EAX=68（转为 10 进制）送入 EDX
005D11AC   .   8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]
005D11AF   .   FFD6         CALL ESI
005D11B1   .   50            PUSH EAX                                                ; EAX 入栈
005D11B2   .   FF15 68104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]            ; 2 个 EAX 值连接
005D11B8   .   8BD0          MOV EDX,EAX                                             ; EAX=“8768”送入 EDX （即转换为 16 进制为 “2240”）
005D11BA   .   8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]
005D11BD   .   FFD6         CALL ESI
005D11BF   .   50            PUSH EAX                                                ; EAX 的值入栈
005D11C0   .   FF15 10124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Str>]             ; MSVBVM60.__vbaI4Str
005D11C6   .   8B9D 48FBFFFF MOV EBX,DWORD PTR SS:[EBP-4B8]                         ; [EBP-4B8]送入 EBX
005D11CC   .   8D8D 30FCFFFF LEA ECX,DWORD PTR SS:[EBP-3D0]                         ; 把[EBP-3D0]装入 ECX 的有效地址
005D11D2   .   03D8          ADD EBX,EAX                                             ; EBX=EBX+EAX，即 EBX=11EB+2240=342B
005D11D4   .   8D85 D0FDFFFF LEA EAX,DWORD PTR SS:[EBP-230]
005D11DA   .   50            PUSH EAX
005D11DB   .   8D95 C0FDFFFF LEA EDX,DWORD PTR SS:[EBP-240]
005D11E1   .   51            PUSH ECX
005D11E2   .   52            PUSH EDX
005D11E3   .   0F80 E50B0000 JO unpack.005D1DCE
005D11E9   .   FF15 D0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCat>]            ; MSVBVM60.__vbaVarCat
005D11EF   .   50            PUSH EAX
005D11F0   .   8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]
005D11F3   .   50            PUSH EAX
005D11F4   .   FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]          ; 取注册编号第 5 位，尾部加“0”
005D11FA   .   50            PUSH EAX                                                ; EAX=“C0”入栈
005D11FB   .   FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]        ; 转换为 16 进制
005D1201   .   50            PUSH EAX                                                ; EAX=43（16 进制）
005D1202   .   FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]            ; 转为 10 进制
005D1208   .   8BD0          MOV EDX,EAX                                             ; EAX=67（转为 10 进制）送入 EDX
005D120A   .   8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]
005D120D   .   FFD6         CALL ESI
005D120F   .   8D8D 80FDFFFF LEA ECX,DWORD PTR SS:[EBP-280]
005D1215   .   50            PUSH EAX                                                ; EAX 入栈
005D1216   .   8D95 10FCFFFF LEA EDX,DWORD PTR SS:[EBP-3F0]
005D121C   .   51            PUSH ECX
005D121D   .   8D85 70FDFFFF LEA EAX,DWORD PTR SS:[EBP-290]
005D1223   .   52            PUSH EDX
005D1224   .   50            PUSH EAX
005D1225   .   FF15 D0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCat>]            ; MSVBVM60.__vbaVarCat
005D122B   .   8D4D 98       LEA ECX,DWORD PTR SS:[EBP-68]
005D122E   .   50            PUSH EAX
005D122F   .   51            PUSH ECX
005D1230   .   FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]          ; 取注册编号第 6 位，尾部加“0”
005D1236   .   50            PUSH EAX                                                ; EAX=“A0”入栈
005D1237   .   FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]        ; 转换为 16 进制
005D123D   .   50            PUSH EAX                                                ; EAX=41 入栈
005D123E   .   FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]            ; 转为 10 进制
005D1244   .   8BD0          MOV EDX,EAX                                             ; EAX=“65”送入 EDX
005D1246   .   8D4D 90       LEA ECX,DWORD PTR SS:[EBP-70]
005D1249   .   FFD6         CALL ESI
005D124B   .   50            PUSH EAX                                                ; EAX 入栈
005D124C   .   FF15 68104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]            ; 2 个 EAX 值连接
005D1252   .   8BD0          MOV EDX,EAX                                             ; EAX=“6765”送入 EDX（即转换为 16 进制为 “1AD6”）
005D1254   .   8D4D 8C       LEA ECX,DWORD PTR SS:[EBP-74]
005D1257   .   FFD6         CALL ESI
005D1259   .   50            PUSH EAX                                                ; EAX 入栈
005D125A   .   FF15 10124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Str>]             ; MSVBVM60.__vbaI4Str
005D1260   .   03D8          ADD EBX,EAX                                             ; EBX=EBX+EAX，即 EBX=342B+1AD6=4E98
005D1262   .   8D95 30FDFFFF LEA EDX,DWORD PTR SS:[EBP-2D0]
005D1268   .   8D85 F0FBFFFF LEA EAX,DWORD PTR SS:[EBP-410]
005D126E   .   52            PUSH EDX
005D126F   .   8D8D 20FDFFFF LEA ECX,DWORD PTR SS:[EBP-2E0]
005D1275   .   50            PUSH EAX
005D1276   .   51            PUSH ECX
005D1277   .   0F80 510B0000 JO unpack.005D1DCE
005D127D   .   FF15 D0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCat>]            ; MSVBVM60.__vbaVarCat
005D1283   .   8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]
005D1286   .   50            PUSH EAX
005D1287   .   52            PUSH EDX
005D1288   .   FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]          ; 取注册编号第 7 位，尾部加“0”
```



```
005D128E   .  50             PUSH EAX                                        ;  EAX="M0"入栈  

005D128F   .  FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]    ;  转换为 16 进制  

005D1295   .  50             PUSH EAX                                        ;  EAX="4D"入栈  

005D1296   .  FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]         ;  转换为 10 进制  

005D129C   .  8BD0           MOV EDX,EAX                                     ;  EAX="77"送入 EDX  

005D129E   .  8D8D 78FFFFFF LEA ECX,DWORD PTR SS:[EBP-88]

005D12A4   .  FFD6           CALL ESI

005D12A6   .  50             PUSH EAX                                        ;  EAX 入栈  

005D12A7   .  8D85 E0FCFFFF LEA EAX,DWORD PTR SS:[EBP-320]  

005D12AD   .  8D8D D0FBFFFF LEA ECX,DWORD PTR SS:[EBP-430]  

005D12B3   .  50             PUSH EAX

005D12B4   .  8D95 D0FCFFFF LEA EDX,DWORD PTR SS:[EBP-330]  

005D12BA   .  51             PUSH ECX  

005D12BB   .  52             PUSH EDX  

005D12BC   .  FF15 D0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCat>]         ;  MSVBVM60.__vbaVarCat  

005D12C2   .  50             PUSH EAX  

005D12C3   .  8D85 7CFFFFFF LEA EAX,DWORD PTR SS:[EBP-84]  

005D12C9   .  50             PUSH EAX  

005D12CA   .  FF15 C8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]       ;  取注册编号第 8 位，尾部加"0"  

005D12D0   .  50             PUSH EAX                                        ;  EAX="90"  

005D12D1   .  FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.rtcByteValueBstr>]    ;  转换 16 进制  

005D12D7   .  50             PUSH EAX                                        ;  EAX="39"  

005D12D8   .  FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrUI1>]         ;  转换 10 进制  

005D12DE   .  8BD0           MOV EDX,EAX                                     ;  EAX="57"送入 EDX  

005D12E0   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:[EBP-8C]  

005D12E6   .  FFD6           CALL ESI

005D12E8   .  50             PUSH EAX                                        ;  EAX 入栈  

005D12E9   .  FF15 68104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]         ;  2 个 EAX 值连接  

005D12EF   .  8BD0           MOV EDX,EAX                                     ;  EAX="7757"送入 EDX  

005D12F1   .  8D8D 70FFFFFF LEA ECX,DWORD PTR SS:[EBP-90]  

005D12F7   .  FFD6           CALL ESI

005D12F9   .  50             PUSH EAX                                        ;  EAX 入栈  

005D12FA   .  FF15 10124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Str>]          ;  MSVBVM60.__vbaI4Str  

005D1300   .  03D8           ADD EBX,EAX                                     ;  EBX=EBX+EAX ，即 EBX=4E98+1E4D=6CE5  

005D1302   .  8D95 C0FBFFFF LEA EDX,DWORD PTR SS:[EBP-440]  

005D1308   .  0F80 C00A0000 JO unpack.005D1DCE  

005D130E   .  8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20]  

005D1311   .  899D C8FBFFFF MOV DWORD PTR SS:[EBP-438],EBX                   ;  EBX 送入[EBP-438]  

005D1317   .  C785 C0FBFFFF>MOV DWORD PTR SS:[EBP-440],3

005D1321   .  FF15 1C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarMove>]        ;  MSVBVM60.__vbaVarMove  

005D1327   .  8D8D 70FFFFFF LEA ECX,DWORD PTR SS:[EBP-90]  

005D132D   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:[EBP-8C]  

.............................................省略 N 行代码~~...................................................  

005D1663   .  50             PUSH EAX  

005D1664   .  51             PUSH ECX  

005D1665   .  FF15 8C124000 call    [<&MSVBVM60.rtcRightCharVar>]               ;  MSVBVM60.rtcRightCharVar  

005D166B   .  8D95 40FFFFFF lea     edx, [ebp-C0]                               ;  把[EBP-C0]的内容装入有效地址 EDX  

005D1671   .  52             push    edx                                       ;  EDX 入栈  

005D1672   .  FF15 4C104000 call    [<&MSVBVM60.__vbaStrErrVarCopy>]            ;  MSVBVM60.__vbaStrErrVarCopy  

005D1678   .  8BD0           mov     edx, eax                                  ;  EAX（7877）送入 EDX  

005D167A   .  8D4D D8        lea     ecx, [ebp-28]  

005D167D   .  FFD6           call    esi

005D167F   .  50             push    eax                                       ;  EAX 入栈  

005D1680   .  FF15 10124000 call    [<&MSVBVM60.__vbaI4Str>]                    ;  转换 16 进制  

005D1686   .  8985 C8FCFFFF mov     [ebp-338], eax                              ;  EAX（1EC5）送入[EBP-338]  

005D168C   .  8D45 E0        lea     eax, [ebp-20]  

005D168F   .  8D8D C0FCFFFF lea     ecx, [ebp-340]  

005D1695   .  50             push    eax  

005D1696   .  8D95 30FFFFFF lea     edx, [ebp-D0]  

005D169C   .  51             push    ecx  

005D169D   .  52             push    edx  

005D169E   .  C785 C0FCFFFF>mov     dword ptr [ebp-340], 3

005D16A8   .  FF15 44124000 call    [<&MSVBVM60.__vbaVarAdd>]                   ;  MSVBVM60.__vbaVarAdd  

005D16AE   .  50             push    eax  

005D16AF   .  8D85 20FFFFFF lea     eax, [ebp-E0]  

005D16B5   .  50             push    eax  

005D16B6   .  FF15 04124000 call    [<&MSVBVM60.rtcHexVarFromVar>]              ;  MSVBVM60.rtcHexVarFromVar  

005D16BC   .  8D8D 10FFFFFF lea     ecx, [ebp-F0]  

005D16C2   .  8D55 E0        lea     edx, [ebp-20]  

005D16C5   .  51             push    ecx  

005D16C6   .  6A 03          push    3  

005D16C8   .  8D85 00FFFFFF lea     eax, [ebp-100]  

005D16CE   .  52             push    edx  

005D16CF   .  50             push    eax  

005D16D0   .  C785 18FFFFFF>mov     dword ptr [ebp-E8], 4

005D16DA   .  C785 10FFFFFF>mov     dword ptr [ebp-F0], 2

005D16E4   .  FF15 F8104000 call    [<&MSVBVM60.rtcMidCharVar>]                 ;  MSVBVM60.rtcMidCharVar  

005D16EA   .  8D8D 00FFFFFF lea     ecx, [ebp-100]  

005D16F0   .  8D95 F0FEFFFF lea     edx, [ebp-110]  

005D16F6   .  51             push    ecx  

005D16F7   .  52             push    edx  

005D16F8   .  FF15 04124000 call    [<&MSVBVM60.rtcHexVarFromVar>]              ;  MSVBVM60.rtcHexVarFromVar  

005D16FE   .  8B45 DC        mov     eax, [ebp-24]  

005D1701   .  8D8D 20FFFFFF lea     ecx, [ebp-E0]  

005D1707   .  50             push    eax  

005D1708   .  51             push    ecx  

005D1709   .  FF15 4C104000 call    [<&MSVBVM60.__vbaStrErrVarCopy>]            ;  MSVBVM60.__vbaStrErrVarCopy  

005D170F   .  8BD0           mov     edx, eax                                  ;  EAX（8BAA）送入 EDX  

005D1711   .  8D4D D4        lea     ecx, [ebp-2C]  

005D1714   .  FFD6           call    esi

005D1716   .  8D95 F0FEFFFF lea     edx, [ebp-110]  

005D171C   .  50             push    eax                                       ;  EAX 入栈  

005D171D   .  52             push    edx  

005D171E   .  FF15 4C104000 call    [<&MSVBVM60.__vbaStrErrVarCopy>]            ;  MSVBVM60.__vbaStrErrVarCopy  

005D1724   .  8BD0           mov     edx, eax                                  ;  EAX（36D）送入 EDX  

005D1726   .  8D4D D0        lea     ecx, [ebp-30]  

005D1729   .  FFD6           call    esi

005D172B   .  50             push    eax                                       ;  EAX 入栈  

005D172C   .  FF15 68104000 call    [<&MSVBVM60.__vbaStrCat>]                   ;  2 者连接  

005D1732   .  8BD0           mov     edx, eax                                  ;  EAX 送入 EDX（此处可做内存注册机）  

005D1734   .  8D4D CC        lea     ecx, [ebp-34]  

005D1737   .  FFD6           call    esi

005D1739   .  50             push    eax                                       ;  EAX 入栈  

005D173A   .  FF15 14114000 call    [<&MSVBVM60.__vbaStrCmp>]                   ;  MSVBVM60.__vbaStrCmp  

005D1740   .  8BF0           mov     esi, eax

005D1742   .  8D45 CC        lea     eax, [ebp-34]  

005D1745   .  8D4D DC        lea     ecx, [ebp-24]  

005D1748   .  50             push    eax  

005D1749   .  F7DE           neg     esi

005D174B   .  8D55 D0        lea     edx, [ebp-30]  

005D174E   .  51             push    ecx  
```



```
005D174F   .  8D45 D4       lea     eax, [ebp-2C]  
005D1752   .  52           push    edx  
005D1753   .  1BF6         sbb     esi, esi  
005D1755   .  8D4D D8      lea     ecx, [ebp-28]  
005D1758   .  50           push    eax  
005D1759   .  46           inc     esi  
005D175A   .  51           push    ecx  
005D175B   .  6A 05        push    5  
005D175D   .  F7DE         neg     esi  
005D175F   .  FF15 18124000 call   [<&MSVBVM60.__vbaFreeStrList>]              ;  MSVBVM60.__vbaFreeStrList  

005D1765   .  83C4 18      add     esp, 18  
005D1768   .  8D8D 6CFFFFFF lea    ecx, [ebp-94]  
005D176E   .  FF15 BC124000 call   [<&MSVBVM60.__vbaFreeObj>]                ;  MSVBVM60.__vbaFreeObj  
005D1774   .  8D95 F0FEFFFF lea    edx, [ebp-110]  
005D177A   .  8D85 F0FEFFFF lea    eax, [ebp-110]  
005D1780   .  52           push    edx  
005D1781   .  8D8D 00FFFFFF lea    ecx, [ebp-100]  
005D1787   .  50           push    eax  
005D1788   .  8D95 10FFFFFF lea    edx, [ebp-F0]  
005D178E   .  51           push    ecx  
005D178F   .  8D85 20FFFFFF lea    eax, [ebp-E0]  
005D1795   .  52           push    edx  
005D1796   .  8D8D 20FFFFFF lea    ecx, [ebp-E0]  
005D179C   .  50           push    eax  
005D179D   .  8D95 30FFFFFF lea    edx, [ebp-D0]  
005D17A3   .  51           push    ecx  
005D17A4   .  8D85 40FFFFFF lea    eax, [ebp-C0]  
005D17AA   .  52           push    edx  
005D17AB   .  8D8D 40FFFFFF lea    ecx, [ebp-C0]  
005D17B1   .  50           push    eax  
005D17B2   .  51           push    ecx  
005D17B3   .  6A 09        push    9  
005D17B5   .  FF15 3C104000 call   [<&MSVBVM60.__vbaFreeVarList>]           ;  MSVBVM60.__vbaFreeVarList  
005D17BB   .  83C4 28      add     esp, 28  
005D17BE   .  66:3BF7      cmp     si, di  
005D17C1   .  0F84 4F030000 je     005D1B16                                   ;  爆破处 NOP~  
```

### 算法总结

1. 取机器码第 `3` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
2. 取机器码第 `4` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
3. 取机器码第 `1` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
4. 取机器码第 `2` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
5. 取机器码第 `5` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
6. 取机器码第 `6` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
7. 取机器码第 `7` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
8. 取机器码第 `8` 位，尾部再加上 `0`，然后转换取得的机器码的 `16` 进制数，最后再转换为 `10` 进制。
9. 然后 `(1)` 的 `10` 进制数与 `(2)` 的 `10` 进制数相加后再转换 `16` 进制，可记为 `A`。
   `(3)` 的 `10` 进制数与 `(4)` 的 `10` 进制数相加后再转换 `16` 进制，可记为 `B`。
   `(5)` 的 `10` 进制数与 `(6)` 的 `10` 进制数相加后再转换 `16` 进制，可记为 `C`。
   `(7)` 的 `10` 进制数与 `(8)` 的 `10` 进制数相加后再转换 `16` 进制，可记为 `D`。
10. 把 `(9)` 中的 `A`、`B`、`C`、`D` 全部相加后的值再加上一个固定 `16` 进制数 `"1EC5"` 后得到第一部分注册码。
11. 将第一部分注册码与固定数值 `"36D"` 连接起来，即是真的注册码。

### 破解总结

分析的算法中错误在所难免，请各位大侠不吝赐教~~~感谢你看完~

### 版权声明

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
