
# AutoRun Design Specialty 9.1.1.21 算法分析(变形 MD5)

> 原文：[`www.pediy.com/kssd/pediy10/81783.html`](https://www.pediy.com/kssd/pediy10/81783.html)

【破文标题】`AutoRun Design Specialty 9.1.1.21` 算法分析(变形 MD5)

【破文作者】`tianxj`

【作者邮箱】`tianxj_2007@126.com`

【作者主页】`WwW.ChiNaPYG.CoM`

【破解工具】`PEiD`,`OD`

【破解平台】`Windows XP`

【软件名称】`AutoRun Design Specialty 9.1.1.21`

【软件类别】国外软件/光碟工具

【软件授权】共享版

【软件语言】英文

【运行环境】`Win9x/Me/NT/2000/XP/2003`

【软件大小】`6.98MB`

【原版下载】`http://www.alleysoft.com/autorundesignspecialty/ARDSSetup90.exe`

【保护方式】注册码

【软件简介】一款专业的所见即所得的光盘自启动程序制作工具。软件支持多种启动屏幕画面，可插入图片、声音、`HTML` 和 `RTF` 文档。内建了媒体播放器和 `RTF` 阅读器，可在程序启动时播放 `AVI` 视频和 `RTF` 文档。软件内置了许多模板，可根据需要调用和修改。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

**一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息**

"Please input `Name`,`Serial`,`Code` again."

**二、用** `PEiD` **对** `AutoRunDS.exe` **查壳，为** `UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`

**三、带壳调试，运行** `OD` **，打开** `AutoRunDS.exe` **，下** `bp MessageBoxA` **，来到关键之处**

```
00571BA0    55              PUSH EBP
00571BA1    8BEC            MOV EBP,ESP
00571BA3    B9 06000000     MOV ECX,6
00571BA8    6A 00           PUSH 0
00571BAA    6A 00           PUSH 0
00571BAC    49              DEC ECX
00571BAD  ^ 75 F9           JNZ SHORT AutoRunD.00571BA8
00571BAF    53              PUSH EBX
00571BB0    56              PUSH ESI
00571BB1    57              PUSH EDI
00571BB2    8BD8            MOV EBX,EAX
00571BB4    33C0            XOR EAX,EAX
00571BB6    55              PUSH EBP
00571BB7    68 7B1D5700     PUSH AutoRunD.00571D7B
00571BBC    64:FF30         PUSH DWORD PTR FS:[EAX]
00571BBF    64:8920         MOV DWORD PTR FS:[EAX],ESP
00571BC2    8D55 FC         LEA EDX,DWORD PTR SS:[EBP-4]
00571BC5    8B83 1C030000   MOV EAX,DWORD PTR DS:[EBX+31C]
00571BCB    E8 C8D9F0FF     CALL AutoRunD.0047F598
00571BD0    8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]
00571BD3    8B83 20030000   MOV EAX,DWORD PTR DS:[EBX+320]
00571BD9    E8 BAD9F0FF     CALL AutoRunD.0047F598
00571BDE    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
00571BE1    8B83 18030000   MOV EAX,DWORD PTR DS:[EBX+318]
00571BE7    E8 ACD9F0FF     CALL AutoRunD.0047F598
00571BEC    8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]
00571BEF    50              PUSH EAX
00571BF0    B9 03000000     MOV ECX,3
00571BF5    BA 01000000     MOV EDX,1
00571BFA    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]             ; //Serial
00571BFD    E8 4E01EDFF     CALL AutoRunD.00441D50
00571C02    8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
00571C05    BA 941D5700     MOV EDX,AutoRunD.00571D94               ; ASCII "002"
00571C0A    E8 6130E9FF     CALL AutoRunD.00404C70
00571C0F    0F85 26010000   JNZ AutoRunD.00571D3B                   ; //Serial 的 1-3 位不是"002"则挂
00571C15    8D45 D4         LEA EAX,DWORD PTR SS:[EBP-2C]
00571C18    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]           ; //Name
00571C1B    8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]           ; //Serial
00571C1E    E8 4D2FE9FF     CALL AutoRunD.00404B70                 ; //Serial 和 Name 相连
00571C23    8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]          ; //相连字符串
00571C26    8D55 D8         LEA EDX,DWORD PTR SS:[EBP-28]
00571C29    E8 0AF4FFFF     CALL AutoRunD.00571038                 ; //作变形 MD5 运算
00571C2E    8D45 D8         LEA EAX,DWORD PTR SS:[EBP-28]
00571C31    8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]
00571C34    E8 67F2FFFF     CALL AutoRunD.00570EA0
00571C39    8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]          ; //变形 MD5 小写
00571C3C    8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]
00571C3F    E8 2071E9FF     CALL AutoRunD.00408D64                 ; //小写转大写
00571C44    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]          ; //变形 MD5 大写,即真码
00571C47    8B55 F4         MOV EDX,DWORD PTR SS:[EBP-C]           ; //假码
00571C4A    E8 2130E9FF     CALL AutoRunD.00404C70                 ; //比较 CALL
00571C4F    74 1D           JE SHORT AutoRunD.00571C6E
00571C51    6A 40           PUSH 40
00571C53    B9 981D5700     MOV ECX,AutoRunD.00571D98              ; ASCII "Information"
00571C58    BA A41D5700     MOV EDX,AutoRunD.00571DA4              ; ASCII "Please input Name,Serial,Code again."
00571C5D    A1 90495F00     MOV EAX,DWORD PTR DS:[5F4990]
00571C62    8B00            MOV EAX,DWORD PTR DS:[EAX]
00571C64    E8 D7EFF2FF     CALL AutoRunD.004A0C40
00571C69    E9 E5000000     JMP AutoRunD.00571D53
00571C6E    8D45 D0         LEA EAX,DWORD PTR SS:[EBP-30]
00571C71    E8 C65B0600     CALL AutoRunD.005D783C
00571C76    8D45 D0         LEA EAX,DWORD PTR SS:[EBP-30]
00571C79    BA D41D5700     MOV EDX,AutoRunD.00571DD4              ; ASCII "\AutoRunDS.ini"
00571C7E    E8 A92EE9FF     CALL AutoRunD.00404B2C
00571C83    8B4D D0         MOV ECX,DWORD PTR SS:[EBP-30]
00571C86    B2 01           MOV DL,1
00571C88    A1 B0674400     MOV EAX,DWORD PTR DS:[4467B0]
00571C8D    E8 CE4BEDFF     CALL AutoRunD.00446860
00571C92    8BF0            MOV ESI,EAX
00571C94    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00571C97    50              PUSH EAX
00571C98    B9 EC1D5700     MOV ECX,AutoRunD.00571DEC              ; ASCII "Name"
00571C9D    BA FC1D5700     MOV EDX,AutoRunD.00571DFC              ; ASCII "REGISTRATION"
00571CA2    8BC6            MOV EAX,ESI
00571CA4    8B38            MOV EDI,DWORD PTR DS:[EAX]
00571CA6    FF57 04         CALL DWORD PTR DS:[EDI+4]
00571CA9    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
00571CAC    50              PUSH EAX
00571CAD    B9 141E5700     MOV ECX,AutoRunD.00571E14              ; ASCII "Serial"
00571CB2    BA FC1D5700     MOV EDX,AutoRunD.00571DFC              ; ASCII "REGISTRATION"
00571CB7    8BC6            MOV EAX,ESI
00571CB9    8B38            MOV EDI,DWORD PTR DS:[EAX]
00571CBB    FF57 04         CALL DWORD PTR DS:[EDI+4]
00571CBE    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
00571CC1    50              PUSH EAX
00571CC2    B9 241E5700     MOV ECX,AutoRunD.00571E24              ; ASCII "Code"
00571CC7    BA FC1D5700     MOV EDX,AutoRunD.00571DFC              ; ASCII "REGISTRATION"
00571CCC    8BC6            MOV EAX,ESI
00571CCE    8B30            MOV ESI,DWORD PTR DS:[EAX]
00571CD0    FF56 04         CALL DWORD PTR DS:[ESI+4]
00571CD3    A1 6C495F00     MOV EAX,DWORD PTR DS:[5F496C]
00571CD8    C600 01         MOV BYTE PTR DS:[EAX],1
00571CDB    A1 444C5F00     MOV EAX,DWORD PTR DS:[5F4C44]
00571CE0    8B00            MOV EAX,DWORD PTR DS:[EAX]
00571CE2    8B80 2C030000   MOV EAX,DWORD PTR DS:[EAX+32C]
00571CE8    8B40 68         MOV EAX,DWORD PTR DS:[EAX+68]
00571CEB    BA 080000FF     MOV EDX,FF000008
00571CF0    E8 739CEBFF     CALL AutoRunD.0042B968
00571CF5    A1 444C5F00     MOV EAX,DWORD PTR DS:[5F4C44]
00571CFA    8B00            MOV EAX,DWORD PTR DS:[EAX]
00571CFC    8B80 2C030000   MOV EAX,DWORD PTR DS:[EAX+32C]
00571D02    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]
00571D05    E8 BED8F0FF     CALL AutoRunD.0047F5C8
00571D0A    A1 54465F00     MOV EAX,DWORD PTR DS:[5F4654]
00571D0F    8B00            MOV EAX,DWORD PTR DS:[EAX]
```
