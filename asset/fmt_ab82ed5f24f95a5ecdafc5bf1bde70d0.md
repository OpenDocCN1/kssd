
# Ulead MediaStudio Pro 7.0 多国语言版

> 原文：[`www.pediy.com/kssd/pediy06/pediy6158.htm`](https://www.pediy.com/kssd/pediy06/pediy6158.htm)

友立独立制片人 7 安装程序暴破

大家应该知道这东东吧~~~ 好有名的影视编辑工具

所以我才想破解这东东~~~ 只是学习而已没有其它目的大家一起学习

用 `isd` 反编译 `setup.inx`

\*    InstallShiled Decompiler v1.00 beta 7

\*    by NEKOSUKI

\* Script Version  : InstallShiled 6.00

\* Decompiled Time :

\* Source Filename : setup.inx

\* Source FileTime :

\* Debug Symbol    :

\* CRC Check       : invalid

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

/\*    Table Strings

\*/

/\*    Strings

                             /-014-

/ 0000CC8F,0000CD75,0001089E,00019C88,0001A05D,0001A0E8,0001AFD3,00022B6D,000236C

9,0002C5ED,0003278F,000327FD,000328BE,0003DA94,

                             /-022-

/ 0000CED0,0000CED0,0000CF3A,0000CF3A,0000CF5B,0000CFBA,0000CFBA,0000CFDB,0000D03

B,0000D03B,0000D05C,0000D0C9,0000D0C9,0000D0EA,0000D148,0000D148,0000D169,0000D1C

9,0000D1C9,0000D2AD,0000D324,0000D39C,

       ACCESS = READ-ONLY\r\n                                     /-008-

/ 0002BAB6,0002BBE5,0002BD14,0002BE42,0002BF77,0002C0AB,0002C273,0002C3A6,

       ID = 1\r\n                                                 /-002-

/ 0002BA9B,0002C258,

       ID = 2\r\n                                                 /-002-

/ 0002BBCA,0002C38B,

       ID = 3\r\n                                                 /-001-

/ 0002BCF9,

       ID = 4\r\n                                                 /-001-

/ 0002BE27,

       ID = 5\r\n                                                 /-001-

/ 0002BF5C,

       ID = 6\r\n                                                 /-001-

/ 0002C090,

       NAME = \"Description\"\r\n                                  /-001-

/ 0002C362,

       NAME = \"Installation\"\r\n                                  /-001-

/ 0002C066,

       NAME = \"Locale\"\r\n                                          /-001-

/ 0002BE03,

中间删掉一些代码，因为太多了呀！

```
prototype NUMBER IfxFinalConstruct();

prototype NUMBER IfxOnInitInstall();

prototype NUMBER IfxOnShowWizardPages(NUMBER);

prototype NUMBER function_731();

prototype NUMBER IfxOnTransferring();

prototype NUMBER IfxOnDisk1Installing();

prototype NUMBER IfxOnDisk1Installed();

prototype NUMBER IfxFinalRelease();

prototype NUMBER IfxonExitInstall();

prototype NUMBER IfxOnFileReadOnly(STRING);

prototype NUMBER IfxOnRemovingSharedFile(STRING);

prototype NUMBER IfxOnMD5Error(STRING);

prototype NUMBER IfxOnFileError(STRING, NUMBER);

prototype NUMBER IfxOnInternetError(NUMBER, STRING, NUMBER);

prototype NUMBER IfxOnFileLocked(STRING);

prototype NUMBER IfxOnNextDisk(STRING, NUMBER, BYREF STRING);

prototype NUMBER IfxRunAfterReboot();

prototype NUMBER IfxInitProperties();

prototype NUMBER InitProperties();

prototype NUMBER IfxReadProperties(NUMBER);

prototype NUMBER ReadProperties(NUMBER);

prototype NUMBER IfxWriteProperties(NUMBER);

prototype NUMBER WriteProperties(NUMBER);

prototype NUMBER ISRT.__ISRTGetPropertyBag(NUMBER, BYREF NUMBER);

prototype NUMBER ISRT.__ISRTReleasePropertyBag(NUMBER);

// : Call Referenced(1):

// :  0001A863,

function BOOL function_0(s0, s1)

   STRING s3;

   NUMBER n0;

begin

/* 00007317: 0022 */    // -- Start Function Code -- //

/* 00007320: 0014 */    s3 = s0 ^ "U32cfg.dll";

/* 00007337: 0021 */    function_369(3, s3);

/* 00007345: 0006 */    n0 = LAST_RESULT;

/* 0000734F: 000D */    n0 = n0 == 1;

/* 0000735E: 0004 */    if(! n0) goto label_1;                        // normal if

/* 0000736A: 0005 */    goto label_4;

/* 00007373: 0005 */    goto label_3;

// : Jump Referenced(1):

// :  0000735E,

label_1:

/* 0000737E: 0014 */    s3 = s0 ^ "U32base.dll";

/* 00007396: 0021 */    function_369(3, s3);

/* 000073A4: 0006 */    n0 = LAST_RESULT;

/* 000073AE: 000D */    n0 = n0 == 1;

/* 000073BD: 0004 */    if(! n0) goto label_2;                        // normal if

/* 000073C9: 0005 */    goto label_4;

/* 000073D2: 0005 */    goto label_3;

// : Jump Referenced(1):

// :  000073BD,

label_2:

/* 000073DD: 0014 */    s3 = s0 ^ "U32file.dll";

/* 000073F5: 0021 */    function_369(3, s3);

/* 00007403: 0006 */    n0 = LAST_RESULT;

/* 0000740D: 000D */    n0 = n0 == 1;

/* 0000741C: 0004 */    if(! n0) goto label_3;                        // normal if

/* 00007428: 0005 */    goto label_4;

// : Jump Referenced(3):

// :  00007373, 000073D2, 0000741C,

label_3:

/* 00007433: 0027 */    // -- Start Return Code -- //

/* 00007437: 0023 */    return 0;

// : Jump Referenced(3):

// :  0000736A, 000073C9, 00007428,

label_4:

/* 00007442: 0021 */    function_427(s1, -65534);

/* 00007450: 0027 */    // -- Start Return Code -- //

/* 00007454: 0023 */    return 1;

/* 0000745D: 0026 */    // -- Create Local Variables -- //

end;

// : Call Referenced(1):

// :  0000B3AC,

function NUMBER function_1(s0, s1)

   STRING s2;

   STRING s3;

   NUMBER n0;

begin

/* 0000746B: 0022 */    // -- Start Function Code -- //

/* 00007474: 0006 */    s2 = "";

/* 0000747E: 0021 */    function_596(s0);

/* 00007487: 0021 */    function_591(s0, s3);

/* 00007493: 000D */    n0 = s3 == s0;

/* 000074A0: 0004 */    if(! n0) goto label_6;                        // normal if

/* 000074AC: 0007 */    s0 = s0 + "\\";

// : Jump Referenced(2):

// :  000074A0, 0000752A,

label_6:

/* 000074BC: 0001 */    // -- While/Switch -- //

/* 000074C0: 0021 */    function_322(s0, s1, s2);

/* 000074CF: 0006 */    n0 = LAST_RESULT;

/* 000074D9: 000D */    n0 = n0 == 0;

/* 000074E8: 0004 */    if(! n0) goto label_7;                        // normal if

/* 000074F4: 0014 */    s2 = s0 ^ s2;

/* 00007501: 0021 */    function_315(s2, 1, 0, "");

/* 00007517: 0021 */    function_321(s2);

/* 00007520: 0006 */    s2 = "";

/* 0000752A: 0005 */    goto label_6;

// : Jump Referenced(1):

// :  000074E8,

label_7:

/* 00007535: 0026 */    // -- Create Local Variables -- //

end;
```

一直到下面关键代码--------------------------------------------------

看我看半个小时才看到这里,暴破的关键地方看下面代码

```
// : Jump Referenced(1):

// :  0001A4CB,

label_485:

/* 0001A51B: 0021 */    function_57(s0, s1, g_str25, g_str26, g_str27);

/* 0001A530: 0006 */    n0 = LAST_RESULT;

/* 0001A53A: 000D */    n9 = n0 == 12;

/* 0001A549: 0004 */    if(! n9) goto label_486;                      // normal if

/* 0001A555: 0005 */    goto label_481;

// : Jump Referenced(1):

// :  0001A549,

label_486:

/* 0001A560: 0014 */    s31 = g_str10 ^ "Info.ini";

/* 0001A575: 0021 */    function_369(3, s31);

/* 0001A583: 0006 */    n9 = LAST_RESULT;

/* 0001A58D: 0004 */    if(! n9) goto label_490;                      // normal if

/* 0001A599: 000D */    n9 = g_number37 == 1;

/* 0001A5A8: 000D */    n10 = g_number38 == 1;

/* 0001A5B7: 0019 */    n9 = n9 && n10;

/* 0001A5C4: 0004 */    if(! n9) goto label_488;                      // normal if

/* 0001A5D0: 000E */    n9 = g_str27 != g_str28;

/* 0001A5DD: 0004 */    if(! n9) goto label_487;                      // normal if
```
