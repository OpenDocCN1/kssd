
# 金山毒霸 2002

> 原文：[`www.pediy.com/kssd/pediy05/pediy50082.htm`](https://www.pediy.com/kssd/pediy05/pediy50082.htm)

```
/////////////////////////////////////////////////////////////////////   
//   
//  目标软件：金山毒霸 2002   
//   
//  软件版本：标准版   
//   
//  官方网站：http://www.iduba.net/   
//   
//  软件授权：商业软件   
//   
//  操作系统：Win95/98/ME、WinNT/2000   
//   
//  软件简介：国产知名杀毒软件...   
//   
//  软件保护：安装序列号保护   
//   
/////////////////////////////////////////////////////////////////////   
//   
//  使用工具：InstallShield Decompiler v1.00 beta 7   
//             (用于反汇编 SETUP.INX 脚本文件)   
//             TRW2000 v1.22 娃娃修改版   
//             (主要用于调试分析)   
//             Our Brain...:-)   
//   
/////////////////////////////////////////////////////////////////////   
//   
//  关于本文：本文主要目的在于教学，研究调用外界 DLL 进行序列号校验的分   
//             析方法...请勿将此教程用于商业目的。   
//   
//                   Always Your Best Friend: FiNALSErAPH   
//   
//  水平有限，难免疏漏...   
//   
//  Any Question?   
//  Mail To: FiNALSErAPH@yahoo.com.cn   
//   
//                                   2002-04-17   
//   
/////////////////////////////////////////////////////////////////////   
//   
//  已知序列号 101200-010000-886681-583893   
//   
/////////////////////////////////////////////////////////////////////   
//   
//  序列号校验程序入口   
//   
/////////////////////////////////////////////////////////////////////   
// : Call Referenced(1):   
// :  00006DAE,   
```

```
function NUMBER function_26()
    STRING s0;
    STRING s1;
    STRING s2;
    STRING s6;
    STRING s7;
    STRING s8;
    STRING s9;
    STRING s10;
    STRING s11;
    NUMBER n0;
    NUMBER n8;
    NUMBER n10;
    NUMBER n11;

begin

/* 0000BCFC: 0006 */    g_str14 = "";
/* 0000BD06: 0006 */    g_str15 = "";
/* 0000BD10: 0006 */    s0 = "";
/* 0000BD1A: 0006 */    s1 = "";

// : Jump Referenced(3):
// :  0000BDDB, 0000C054, 0000C164,

label_148:

/* 0000BD26: 0021 */    function_425(s0, s1, g_str14, g_str15, g_str16);
/* 0000BD3B: 0006 */    n0 = LAST_RESULT;
/* 0000BD45: 000D */    n10 = n0 == 12;
/* 0000BD54: 0004 */    if(! n10) goto label_149;                          // normal if
/* 0000BD60: 0027 */    // -- Start Return Code -- //
/* 0000BD64: 0023 */    return n0;

// : Jump Referenced(1):
// :  0000BD54,

label_149:

/* 0000BD6D: 0021 */    function_479(g_str16);
/* 0000BD76: 0006 */    n8 = LAST_RESULT;
/* 0000BD80: 000E */    n10 = n8 != 27;
                            //判断输入序列号的长度是否为 27, 不是出错
/* 0000BD8F: 0004 */    if(! n10) goto label_150;                          // normal if
/* 0000BD9B: 0021 */    function_288("IDS_Script_Message_ERROR_Serial");
/* 0000BDC3: 0006 */    s11 = LAST_RESULT;
/* 0000BDCD: 0021 */    function_318(s11, -65534);
/* 0000BDDB: 0005 */    goto label_148;

// : Jump Referenced(1):
// :  0000BD8F,

label_150:

/* 0000BDE6: 001E */    n10 = g_str16[0];
/* 0000BDF5: 000E */    n10 = n10 != 49;
/* 0000BE04: 001E */    n11 = g_str16[1];
/* 0000BE13: 000E */    n11 = n11 != 48;
/* 0000BE22: 0019 */    n10 = n10 && n11;
                            //[1] [2]
/* 0000BE2F: 001E */    n11 = g_str16[2];
/* 0000BE3E: 000E */    n11 = n11 != 49;
/* 0000BE4D: 0018 */    n10 = n10 || n11;
                            //[3]
/* 0000BE5A: 001E */    n11 = g_str16[3];
/* 0000BE69: 000E */    n11 = n11 != 50;
/* 0000BE78: 0018 */    n10 = n10 || n11;
                            //[4]
/* 0000BE85: 001E */    n11 = g_str16[4];
/* 0000BE94: 000E */    n11 = n11 != 48;
/* 0000BEA3: 0018 */    n10 = n10 || n11;
                            //[5]
/* 0000BEB0: 001E */    n11 = g_str16[5];
/* 0000BEBF: 000E */    n11 = n11 != 48;
/* 0000BECE: 0018 */    n10 = n10 || n11;
                            //[6]
/* 0000BEDB: 001E */    n11 = g_str16[6];
/* 0000BEEA: 000E */    n11 = n11 != 45;
/* 0000BEF9: 0018 */    n10 = n10 || n11;
                            //[7]
/* 0000BF06: 001E */    n11 = g_str16[7];
/* 0000BF15: 000E */    n11 = n11 != 48;
/* 0000BF24: 0018 */    n10 = n10 || n11;
                            //[8]
/* 0000BF31: 001E */    n11 = g_str16[8];
/* 0000BF40: 000E */    n11 = n11 != 49;
/* 0000BF4F: 0018 */    n10 = n10 || n11;
                            //[9]
/* 0000BF5C: 001E */    n11 = g_str16[9];
/* 0000BF6B: 000E */    n11 = n11 != 48;
/* 0000BF7A: 0018 */    n10 = n10 || n11;
                            //[10]
/* 0000BF87: 001E */    n11 = g_str16[10];
/* 0000BF96: 000E */    n11 = n11 != 48;
/* 0000BFA5: 0018 */    n10 = n10 || n11;
                            //[11]
/* 0000BFB2: 001E */    n11 = g_str16[11];
/* 0000BFC1: 000E */    n11 = n11 != 48;
/* 0000BFD0: 0018 */    n10 = n10 || n11;
                            //[12]
/* 0000BFDD: 001E */    n11 = g_str16[12];
/* 0000BFEC: 000E */    n11 = n11 != 48;
/* 0000BFFB: 0018 */    n10 = n10 || n11;
                            //序列号的前 13 位满足以下条件:
                            //((第 1 位!='1') && (第 2 位!='0')) || (第 3 位!='1') \
                            //(第 4 位!='2') || (第 5 位!='0') || (第 6 位!='0') \
                            //(第 7 位!='-') || (第 8 位!='0') || (第 9 位!='1') \
                            //(第 10 位!='0') || (第 11 位!='0') || (第 12 位!='0') \
                            //(第 13 位!='0')为假
                            //由此推出序列号格式:
                            //??1200-010000-??????-??????
                            //第 1 位='1'或第 2 位='0'
/* 0000C008: 0004 */    if(! n10) goto label_151;                          // normal if
/* 0000C014: 0021 */    function_288("IDS_Script_Message_ERROR_Serial");
/* 0000C03C: 0006 */    s11 = LAST_RESULT;
/* 0000C046: 0021 */    function_318(s11, -65534);
/* 0000C054: 0005 */    goto label_148;

// : Jump Referenced(1):
// :  0000C008,

label_151:

/* 0000C05F: 0029 */    StrSub(s2, g_str16, 0, 27);
/* 0000C073: 0006 */    s2 = g_str16;
                            //这里我有点纳闷? 反编译结果正确么
                            //反正不影响分析(我忽略这一段)
/* 0000C07D: 0021 */    function_288("IDS_Script_Message_ERROR_Serial");
/* 0000C0A5: 0006 */    s1 = LAST_RESULT;
/* 0000C0AF: 0021 */    function_503();
                            //这个函数的功能是获得安装临时文件夹的路径
/* 0000C0B5: 0006 */    s11 = LAST_RESULT;
/* 0000C0BF: 0014 */    s10 = s11 ^ "INSTSW.dll";
/* 0000C0D6: 0039 */    UseDLL(s10);
/* 0000C0DD: 0006 */    n8 = LAST_RESULT;
/* 0000C0E7: 0020 */    INSTSW.UninstallHook123_QDZ(0, g_str16, g_str16);
/* 0000C0F8: 0006 */    n8 = LAST_RESULT;
/* 0000C102: 003A */    UnUseDLL(s10);
/* 0000C109: 000E */    n10 = n8 != 0;
                            //这里是调用 INSTW.dll 的函数继续校验序列号
                            //要求返回值为 0
                            //既然进入 DLL，我想使用调试工具就可以了
                            //关键问题：如何下断点
                            //我采用 GetMoudelHandleA
                            //找到 10001190 -> 01FD1190(可能因机器而异)
                            //本来考虑过使用 INT3 法的，但未获成功。TRW2000
                            //没有中断，估计是 SEH 的后果。程序事先发现错误
                            //直接报错了
/* 0000C118: 0004 */    if(! n10) goto label_152;                          // normal if
/* 0000C124: 0021 */    function_288("IDS_Script_Message_ERROR_Serial");
/* 0000C14C: 0006 */    s11 = LAST_RESULT;
/* 0000C156: 0021 */    function_318(s11, -65534);
/* 0000C164: 0005 */    goto label_148;

// : Jump Referenced(1):
```



```
// : 0000C118,

label_152:

/* 0000C16F: 0021 */    function_341(-2147483646);

/* 0000C17A: 0006 */    s6 = "SOFTWARE\\Kingsoft\\AntiVirus";

/* 0000C19F: 0006 */    s9 = "";

/* 0000C1A9: 0021 */    function_337(s6, s9);

/* 0000C1B5: 0006 */    s6 = "SOFTWARE\\Kingsoft\\AntiVirus";

/* 0000C1DA: 0006 */    s7 = "";

/* 0000C1E4: 0006 */    s8 = "";

/* 0000C1EE: 0021 */    function_343(s6, s7, 1, s8, -1);

/* 0000C207: 0021 */    function_341(-2147483646);

/* 0000C212: 0006 */    s6 = "SOFTWARE\\Kingsoft\\AntiVirus\\V3.0";

/* 0000C23C: 0006 */    s9 = "";

/* 0000C246: 0021 */    function_337(s6, s9);

/* 0000C252: 0006 */    s6 = "SOFTWARE\\Kingsoft\\AntiVirus\\V3.0";

/* 0000C27C: 0006 */    s7 = "Serial Number";

/* 0000C293: 0006 */    s8 = g_str16;

/* 0000C29D: 0021 */    function_343(s6, s7, 1, s8, -1);

/* 0000C2B6: 0021 */    function_341(-2147483647);

/* 0000C2C1: 0006 */    s6 = "SOFTWARE\\Kingsoft\\AntiVirus\\V3.0";

/* 0000C2EB: 0006 */    s7 = "Serial Number";

/* 0000C302: 0006 */    s8 = g_str16;

/* 0000C30C: 0021 */    function_343(s6, s7, 1, s8, -1);

/* 0000C325: 0027 */    // -- Start Return Code -- //

/* 0000C329: 0023 */    return n0;

end;

/////////////////////////////////////////////////////////////////////

//

//  INSTSW.dll 内程序分析

//

/////////////////////////////////////////////////////////////////////

//

//  主过程

//

/////////////////////////////////////////////////////////////////////

.text:10001190

.text:10001190

.text:10001190                 public UninstallHook123_QDZ

.text:10001190 UninstallHook123_QDZ proc near

.text:10001190

.text:10001190 arg_4             = dword ptr  8

.text:10001190

.text:10001190                 mov     eax, [esp+arg_4]

.text:10001194                 push    esi

.text:10001195                 push    eax

.text:10001196                 xor     esi, esi

.text:10001198                 call    sub_10001960

                            //关键调用，只要返回值不为 0 就 OK！

                            //也许可以爆破！(但小心主执行程序校验)

                            //事实确实如此，从.NET 版开始，就有主程

                            //序额外校验。结果就是安装了还不一定能

                            //用！

.text:1000119D                 add     esp, 4

.text:100011A0                 test    eax, eax

.text:100011A2                 mov     eax, 1

.text:100011A7                 jz      short loc_100011AB

.text:100011A9                 mov     eax, esi

.text:100011AB

.text:100011AB loc_100011AB:

.text:100011AB                 pop     esi

.text:100011AC                 retn    0Ch

.text:100011AC UninstallHook123_QDZ endp

/////////////////////////////////////////////////////////////////////

//

//  sub_10001960

//

/////////////////////////////////////////////////////////////////////

.text:10001960 sub_10001960     proc near

.text:10001960

.text:10001960 var_154         = dword ptr -154h

.text:10001960 var_12C         = dword ptr -12Ch

.text:10001960 arg_0           = dword ptr  4

.text:10001960

.text:10001960                 sub     esp, 154h

.text:10001966                 mov     ecx, [esp+154h+arg_0]

.text:1000196D                 push    ebx

.text:1000196E                 push    esi

.text:1000196F                 lea     eax, [esp+15Ch+var_154]

.text:10001973                 push    edi

.text:10001974                 push    eax

.text:10001975                 push    ecx

.text:10001976                 xor     ebx, ebx

.text:10001978                 mov     [esp+168h+var_12C], 12Ch

.text:10001980                 mov     [esp+168h+var_154], 28h

.text:10001988                 call    sub_100015C0

.text:1000198D                 sub     esp, 20h

.text:10001990                 mov     ecx, 0Ah

.text:10001995                 lea     esi, [esp+188h+var_154]

.text:10001999                 mov     edi, esp

.text:1000199B                 lea     edx, [esp+188h+var_12C]

.text:1000199F                 repe movsd

.text:100019A1                 push    edx

.text:100019A2                 call    sub_10001550

.text:100019A7                 lea     eax, [esp+18Ch+var_154]

.text:100019AB                 lea     ecx, [esp+18Ch+var_12C]

.text:100019AF                 push    eax

.text:100019B0                 push    ecx

.text:100019B1                 call    sub_100016E0

                            //返回值必须不为 0

                            //关键调用

                            //为统一结果，采用下面的序列号分析

                            //101200-010000-123456-654321

.text:100019B6                 add     esp, 34h

.text:100019B9                 test    eax, eax

.text:100019BB                 mov     eax, 1

.text:100019C0                 jnz     short loc_100019C4

.text:100019C2                 mov     eax, ebx

.text:100019C4

.text:100019C4 loc_100019C4:

.text:100019C4                 pop     edi

.text:100019C5                 pop     esi

.text:100019C6                 pop     ebx

.text:100019C7                 add     esp, 154h

.text:100019CD                 retn

.text:100019CD sub_10001960     endp

/////////////////////////////////////////////////////////////////////

//

//  算法校验核心程序片段

//

//  注意：有一些中间调用我没有注释具体过程，虽然很重要，但分析并不困难

//        特别注意循环条件及入口参数到输出的动态变化即可。

//

//        因为 KanXue 的论坛已经不欢迎国产软件的破解相关，所以本文仅从技

//        术角度理清了校验流程。不做进一步具体分析... 如果你具体调试了

//        自然会有所收获

//

/////////////////////////////////////////////////////////////////////

.text:100016E0 sub_100016E0     proc near

.text:100016E0

.text:100016E0 var_1EC         = byte ptr -1ECh

.text:100016E0 var_1D8         = byte ptr -1D8h

.text:100016E0 var_1D2         = dword ptr -1D2h

.text:100016E0 var_1CE         = dword ptr -1CEh

.text:100016E0 var_1CA         = dword ptr -1CAh

.text:100016E0 var_1C6         = byte ptr -1C6h

.text:100016E0 var_1AC         = byte ptr -1ACh

.text:100016E0 var_1A4         = byte ptr -1A4h

.text:100016E0 var_16C         = dword ptr -16Ch

.text:100016E0 var_168         = dword ptr -168h

.text:100016E0 var_164         = dword ptr -164h

.text:100016E0 var_160         = byte ptr -160h

.text:100016E0 var_12C         = byte ptr -12Ch

.text:100016E0 var_128         = byte ptr -128h

.text:100016E0 var_D8          = dword ptr -0D8h

.text:100016E0 var_D4          = dword ptr -0D4h

.text:100016E0 var_D0          = dword ptr -0D0h

.text:100016E0 var_CC          = byte ptr -0CCh

.text:100016E0 var_30          = byte ptr -30h

.text:100016E0 arg_0           = dword ptr  4

.text:100016E0 arg_4           = dword ptr  8

.text:100016E0

.text:100016E0                 sub     esp, 1ECh

.text:100016E6                 mov     ecx, 4Bh

.text:100016EB                 push    ebx

.text:100016EC                 mov     ebx, [esp+1F0h+arg_0]

.text:100016F3                 push    ebp

.text:100016F4                 push    esi

.text:100016F5                 push    edi

.text:100016F6                 mov     esi, ebx

.text:100016F8                 lea     edi, [esp+1FCh+var_12C]

.text:100016FF                 lea     eax, [ebx+0FCh]

.text:10001705                 repe movsd

.text:10001707                 push    0Ch

.text:10001709                 lea     ecx, [esp+200h+var_30]

.text:10001710                 push    eax

.text:10001711                 push    ecx

.text:10001712                 call    sub_100019D0

.text:10001717                 add     ebx, 60h

.text:1000171A                 push    27h
```



```
.text:1000171C                lea     edx, [esp+20Ch+var_CC]

.text:10001723                push    ebx

.text:10001724                push    edx

.text:10001725                call    sub_100019D0

.text:1000172A                mov     ebp, [esp+214h+arg_4]

.text:10001731                add     esp, 18h

.text:10001734                lea     ecx, [esp+1FCh+var_1AC]

.text:10001738                mov     eax, [ebp+20h]

.text:1000173B                push    eax

.text:1000173C                push    offset a08d

.text:10001741                push    ecx

.text:10001742                call    _sprintf
                                //'12345665'

.text:10001747                mov     edx, [ebp+24h]

.text:1000174A                lea     ebx, [ebp+24h]

.text:1000174D                add     esp, 0Ch

.text:10001750                lea     eax, [esp+1FCh+var_1A4]

.text:10001754                push    edx

.text:10001755                push    offset a04d

.text:1000175A                push    eax

.text:1000175B                call    _sprintf
                                //'4321'

                                //'123456654321'

.text:10001760                lea     ecx, [esp+208h+var_30]

.text:10001767                push    0Ch

.text:10001769                push    ecx

.text:1000176A                call    sub_10001A60

.text:1000176F                add     esp, 14h

.text:10001772                test    eax, eax

.text:10001774                jz      loc_1000194C

.text:1000177A                lea     edx, [esp+1FCh+var_30]

.text:10001781                push    0Ch

.text:10001783                lea     eax, [esp+200h+var_1AC]

.text:10001787                push    edx

.text:10001788                push    eax

.text:10001789                call    sub_10001A00
                                //'12345 66543 21'

                                //

                                //'66543 12345 21'

.text:1000178E                lea     ecx, [esp+208h+var_1A4]

.text:10001792                push    ecx

.text:10001793                call    _atoi
                                //'4521' -> 0x11A9

.text:10001798                lea     edx, [esp+20Ch+var_1AC]

.text:1000179C                mov     [ebx], eax

.text:1000179E                push    edx

.text:1000179F                mov     [esp+210h+var_1A4], 0

.text:100017A4                call    _atoi
                                //'66543123' -> 0x03F75E13

.text:100017A9                lea     ecx, [esp+210h+var_1AC]

.text:100017AD                push    1Ah

.text:100017AF                push    ecx

.text:100017B0                push    eax

.text:100017B1                mov     [ebp+20h], eax

.text:100017B4                call    sub_10001AB0
                                //0x03F75E13

                                //

                                //00 01 00 01  01 01 00 01  01 01 01 01  01 01 01 01

                                //01 00 00 00  00 01 00 00  01 01

.text:100017B9                mov     eax, [ebx]

.text:100017BB                lea     edx, [esp+21Ch+var_16C]

.text:100017C2                push    0Dh

.text:100017C4                push    edx

.text:100017C5                push    eax

.text:100017C6                call    sub_10001AB0
                                //0x11A9

                                //

                                //01 00 00 00  01 01 00 01  00 01 00 00

.text:100017CB                mov     ecx, 6

.text:100017D0                lea     esi, [esp+228h+var_1AC]

.text:100017D4                lea     edi, [esp+228h+var_1EC]

.text:100017D8                mov     edx, [esp+228h+var_168]

.text:100017DF                repe movsd

.text:100017E1                mov     eax, [esp+228h+var_164]

.text:100017E8                mov     ecx, [esp+228h+var_16C]

.text:100017EF                movsw

.text:100017F1                mov     [esp+228h+var_1CE], edx

.text:100017F5                mov     edx, [esp+228h+arg_0]

.text:100017FC                mov     [esp+228h+var_1CA], eax

.text:10001800                add     edx, 4

.text:10001803                mov     [esp+228h+var_1D2], ecx

.text:10001807                mov     cl, [esp+228h+var_160]

.text:1000180E                push    14h

.text:10001810                lea     eax, [esp+22Ch+var_1EC]

.text:10001814                push    edx

.text:10001815                push    eax

.text:10001816                mov     [esp+234h+var_1C6], cl

.text:1000181A                call    sub_10001A00

.text:1000181F                mov     ecx, 6

.text:10001824                lea     esi, [esp+234h+var_1EC]

.text:10001828                lea     edi, [esp+234h+var_1AC]

.text:1000182F                mov     edx, [esp+234h+var_1CE]

.text:10001833                repe movsd

.text:10001835                mov     ecx, [esp+234h+var_1D2]

.text:10001839                mov     eax, [esp+234h+var_1CA]

.text:1000183D                movsw

.text:1000183F                mov     [esp+234h+var_168], edx

.text:10001846                lea     esi, [ebp+20h]

.text:10001849                mov     [esp+234h+var_16C], ecx

.text:10001850                mov     cl, [esp+234h+var_1C6]

.text:10001854                push    1Ah

.text:10001856                lea     edx, [esp+238h+var_1AC]

.text:1000185D                push    esi

.text:1000185E                push    edx

.text:1000185F                mov     [esp+240h+var_164], eax

.text:10001866                mov     [esp+240h+var_160], cl

.text:1000186D                call    sub_10001AE0
                                //00 01 00 01  01 01 00 01  01 01 01 01  01 01 01 01

                                //01 00 00 00  00 01 00 00  01 01

                                //

                                //0x0177FE13

.text:10001872                add     esp, 44h

.text:10001875                lea     eax, [esp+1FCh+var_16C]

.text:1000187C                push    0Dh

.text:1000187E                push    ebx

.text:1000187F                push    eax

.text:10001880                call    sub_10001AE0
                                //01 00 00 00  01 01 00 01  00 01 00 00  01

                                //

                                //0x02B046BF

.text:10001885                mov     ecx, [esp+208h+var_D4] ; 0x02B046BF

.text:1000188C                mov     eax, [esi]     ; 0x0177FE13

.text:1000188E                mov     edx, [esp+208h+var_D0]

.text:10001895                mov     edi, [ebx]     ; 0x11A9

.text:10001897                add     esp, 0Ch

.text:1000189A                xor     eax, ecx
                                //0x0177FE13 ^ 0x02B046BF = 0x03C7B8AC

                                //0x03C7B8AC -> 63420588

.text:1000189C                xor     edi, edx
                                //0x11A9 ^ 0x425 = 0x158C(参与最后的校验)

.text:1000189E                mov     [esi], eax

.text:100018A0                mov     esi, eax

.text:100018A2                mov     [ebx], edi

.text:100018A4                xor     edx, edx

.text:100018A6                mov     ecx, 64h

.text:100018AB                div     ecx

.text:100018AD                push    0

.text:100018AF                mov     eax, 51EB851Fh

.text:100018B4                mov     ecx, [ebp+14h]

.text:100018B7                push    edx

.text:100018B8                mul     esi

.text:100018BA                mov     eax, [ebp+18h]

.text:100018BD                shr     edx, 5
```



```assembly
.text:100018C0                 push    edx
.text:100018C1                 mov     edx, [ebp+1Ch]
.text:100018C4                 push    edx
.text:100018C5                 mov     edx, [ebp+10h]
.text:100018C8                 push    eax
.text:100018C9                 mov     eax, [ebp+0Ch]
.text:100018CC                 push    ecx
.text:100018CD                 mov     ecx, [ebp+8]
.text:100018D0                 push    edx
.text:100018D1                 mov     edx, [ebp+4]
.text:100018D4                 push    eax
.text:100018D5                 push    ecx
.text:100018D6                 push    edx
.text:100018D7                 lea     eax, [esp+224h+var_1EC]
.text:100018DB                 push    offset a02d02d02d01d01
.text:100018E0                 push    eax
.text:100018E1                 call    _sprintf
; '1012 0001 0000 6342 0588'
.text:100018E6                 lea     ecx, [esp+22Ch+var_128]
.text:100018ED                 push    14h
.text:100018EF                 push    ecx
.text:100018F0                 mov     [esp+234h+var_1D8], 0
.text:100018F5                 call    sub_10001A60
; '1012 0001 0000 6342 0588'
.text:100018FA                 add     esp, 38h
.text:100018FD                 test    eax, eax
.text:100018FF                 jz      short loc_1000194C
.text:10001901                 lea     edx, [esp+1FCh+var_128]
.text:10001908                 push    14h
; '1012 0001 0000 6342 0588'
;
; '0063 4001 0010 1202 0588'
.text:1000190A                 lea     eax, [esp+200h+var_1EC]
.text:1000190E                 push    edx
.text:1000190F                 push    eax
.text:10001910                 call    sub_10001A00
.text:10001915                 call    sub_10001B30
.text:1000191A                 mov     edx, [esp+208h+var_D8]
.text:10001921                 lea     ecx, [esp+208h+var_1EC]
.text:10001925                 push    14h
.text:10001927                 push    ecx
.text:10001928                 push    edx
.text:10001929                 call    sub_10001B70
; 这是一个查表(256)转换运算
; 0x3419720A(常量)
; '0063 4001 0010 1202 0588'
;
; 0x317E3EF7
.text:1000192E                 mov     ecx, [ebx]
; 0x158C
.text:10001930                 and     eax, 1FFFh
; 0x317E3EF7 & 0x1FFF = 0x1EF7
; 得到参与最后校验的另一个值
.text:10001935                 add     esp, 18h
.text:10001938                 cmp     eax, ecx
; 相等则表明序列号正确
.text:1000193A                 jnz     short loc_1000194C
.text:1000193C                 pop     edi
.text:1000193D                 pop     esi
.text:1000193E                 pop     ebp
.text:1000193F                 mov     eax, 1
.text:10001944                 pop     ebx
.text:10001945                 add     esp, 1ECh
.text:1000194B                 retn
```

///

// 注册机的思路

// 注意：其中逆算法未实际验证，仅根据校验模型认为是可逆的。

// 国产优秀软件(至少是大众接受的软件)。注册机是不会公开的。

```text
63420588 -> 0x03C7B8AC
 |
'1012 0001 0000 6342 0588'
 |
 |->sub_10001A00 的逆算法(?)
 |
'0063 4001 0010 1202 0588' + 0x3419720A
 |
 |(本应是正确的校验值，这里为对照阅读方便用错误的值进行说明)
 |
0x158C ^ 0x425
 |
0x11A9 ->45 21
 |
 |->sub_10001AB0
 |
01 00 00 00  01 01 00 01  00 01 00 00
 |
 |->sub_10001AE0
 |
0x02B046BF ^ 0x03C7B8AC
 |
0x0177FE13
 |
 |->sub_10001AE0 的逆算法(?)
 |
00 01 00 01  01 01 00 01  01 01 01 01  01 01 01 01
01 00 00 00  00 01 00 00  01 01
 |
 |->sub_10001AB0 的逆算法(?)
 |
0x03F75E13
 |
66543 123 + 45 21
 |
66543 12345 21
 |
 |->sub_10001A00 的逆算法(?)
 |
12345 66543 21
```

///

// 谨以此文献给茁壮成长的 OCG(Opening Crack Group - 暂定名)...

// Personal Greeting To: [AFO] [CCG] [BCG] Members & All My Friends

// 特别希望夜月看看，提些意见！因为上次谈到过调用 DLL 进行序列号校验的

// 问题。关于 PhotoShop 5.0 的序列号分析，因反编译不成功，暂时放下。

// 水平有限，想不出动态跟踪这种解释运行程序的有效方法。

// 另：请问 InstallShield Decompiler 的作者是如何做到解释脚本的???太牛

// 了

// FiNALSErAPH | OCG

// 2002-04-16 AM2:00

///

// 后记

///

今天不爽，一个人值通宵夜班！单位内部网络因病毒瘫痪！某部门的热心人拿来了“瑞星杀毒 2002 增强版”和“金山毒霸 2002”。向我宣称这是正版的！(我觉得好笑，正版？他拿着到处装算盗版吧！违反用户许可协议的！)然后向我炫他的密钥盘和序列号！KAO，瑞星？加密完全是放水！金山毒霸 2002 的序列号还有点意思！趁着无聊的值班时间，分析着玩！(也因为没有看到此版注册机的缘故) 再说也有段时间没写文章了...

原来写过金山毒霸.NET 的注册机，这次重新审视，发现校验算法上专业了许多。呵呵，显然是精心设计的。向辛勤的工作人员致敬...

金山确是很有潜力的软件公司。至少市场、宣传、软件工程方面很不错！软件本身？中等但界面一流！能在如此短的时间内占据相当的市场份额，实属不易...

我最希望看到新版的 `WPS` 上市(有“学生版”很好...)。

`4dd466d0e1e0cc123afbe504da2462ca`
