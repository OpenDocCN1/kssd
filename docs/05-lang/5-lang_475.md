
# 科建课件录制浏览器

> 原文：`www.pediy.com/kssd/pediy08/pediy8-483.htm`

一个网友让看一个东东。给偶之后发现是科建浏览器。这东东是 `InstallShield` 安装需要输入注册号。之后安装完毕后，又需要加密狗。

偶知道科建系列的东东早就分析的 N 次捏，这里捏偶随便写点东东，只做技术研究。其他东东活动捏盖不负责！

`sid.exe` 反编译其安装 `setup.inx`，代码如下：

```
///////////////////////////////////////////////////////////////////////////////////

// global variables

NUMBER global_number0, global_number1, global_number2, global_number3, global_number4, global_number5, global_number6, global_number7, global_number8, global_number9, global_number10, global_number11, global_number12, global_number13, global_number14, global_number15, global_number16, global_number17, global_number18, global_number19, global_number20, global_number21, global_number22, global_number23, global_number24, global_number25, global_number26, global_number27, global_number28, global_number29, global_number30, global_number31, global_number32, global_number33, global_number34, global_number35, global_number36, global_number37, global_number38, global_number39, global_number40, global_number41, global_number42, global_number43, global_number44, global_number45, global_number46, global_number47, global_number48, global_number49, global_number50, global_number51, global_number52, global_number53, global_number54, global_number55, global_number56, global_number57, global_number58, global_number59, global_number60, global_number61, global_number62, global_number63, global_number64, global_number65, global_number66, global_number67, global_number68, global_number69;

STRING global_string0, global_string1, global_string2, global_string3, global_string4, global_string5, global_string6, global_string7, global_string8, global_string9, global_string10, global_string11, global_string12, global_string13, global_string14, global_string15, global_string16, global_string17, global_string18, global_string19, global_string20, global_string21, global_string22, global_string23, global_string24, global_string25, global_string26, global_string27, global_string28, global_string29, global_string30, global_string31, global_string32, global_string33, global_string34, global_string35, global_string36, global_string37, global_string38, global_string39, global_string40, global_string41, global_string42, global_string43, global_string44, global_string45, global_string46, global_string47, global_string48, global_string49, global_string50, global_string51, global_string52, global_string53, global_string54, global_string55, global_string56, global_string57, global_string58, global_string59, global_string60, global_string61, global_string62, global_string63, global_string64, global_string65, global_string66, global_string67, global_string68, global_string69, global_string70, global_string71, global_string72;

OBJECT global_object3, global_object4, global_object5, global_object6, global_object7, global_object8, global_object9, global_object10, global_object11, global_object12, global_object13, global_object14;

///////////////////////////////////////////////////////////////////////////////////

// code start

@00004DEC:0012   label_4dec:

@00004DEE:0022   function NUMBER function_0()

@00004DEE         NUMBER local_number1, local_number2, local_number3, local_number4, local_number5, local_number6, local_number7, local_number8, local_number9, local_number10, local_number11;

@00004DEE         STRING local_string1, local_string2, local_string3, local_string4, local_string5, local_string6, local_string7, local_string8, local_string9, local_string10, local_string11, local_string13, local_string14, local_string15, local_string16, local_string17, local_string18, local_string19;

@00004DEE

@00004DEE        begin

@00004DF7:0006      global_number20 = 0;

@00004E03:0006      local_number2 = 301;

@00004E0F:0021      function_434();--------

@00004E15:0006      local_string18 = LASTRESULT;

@00004E1F:0021      function_242("COMPANY_NAME");------>取公司名例程函数

@00004E34:0006      local_string19 = LASTRESULT;---------------------------->返回公司名并赋给串变量 local_string19

@00004E3E:0014      local_string18 = (local_string18 ^ local_string19);------->串 local_string18 连接串 local_string19

@00004E4B:0021      function_242("PRODUCT_NAME");------>取公司名例程函数

@00004E60:0006      local_string19 = LASTRESULT;

@00004E6A:0014      global_string8 = (local_string18 ^ local_string19);

@00004E77:0006      local_string9 = global_string8;

@00004E81:0006      local_string5 = "";

@00004E8B:0006      local_string6 = "";

@00004E95:0006      local_string7 = "";

@00004E9F:0006      global_string13 = "";

@00004EA9:0006      global_string14 = "";

@00004EB3:0006      global_string15 = "";

@00004EBD:0007   label_4ebd:

@00004EBF:0006      local_string1 = "";

@00004EC9:0006      local_string2 = "";

@00004ED3:0021      SdWelcome(local_string1, local_string2);----------------->欢迎闪屏窗体弹出

@00004EDF:0006      local_number1 = LASTRESULT;

@00004EE9:000D      local_number9 = (local_number1 = 12);

@00004EF8:0004      if(local_number9) then // ref index: 1

@00004F04:0005        goto label_4ebd;---->回滚上一步骤

@00004F0D:000B      endif;

@00004F0D:000B   label_4f0d:

@00004F0F:0021      function_439();

@00004F15:0006      local_string18 = LASTRESULT;

@00004F1F:0014      local_string3 = (local_string18 ^ "license.txt");

@00004F37:0006      local_string1 = "";

@00004F41:0006      local_string2 = "";

@00004F4B:0006      local_string4 = "";

@00004F55:0021      SdLicense(local_string1, local_string2, local_string4, local_string3);----->许可证 SdLicense 窗口

@00004F67:0006      local_number1 = LASTRESULT;

@00004F71:000D      local_number9 = (local_number1 = 12);

@00004F80:0004      if(local_number9) then // ref index: 1

@00004F8C:0005        goto label_4ebd;---->回滚上一步骤 SdWelcome

@00004F95:0007      endif;

@00004F95:0007   label_4f95:

@00004F97:0006      local_string2 = "";

@00004FA1:0006      local_string1 = "";

@00004FAB:0021      SdRegisterUserEx(local_string1, local_string2, global_string13, global_string14, global_string15);------>注册验证窗体 SdRegisterUserEx

@00004FC0:0006      local_number1 = LASTRESULT;

@00004FCA:000D      local_number9 = (local_number1 = 12);

@00004FD9:0004      if(local_number9) then // ref index: 1

@00004FE5:0005        goto label_4f0d;------------------>回滚上一步骤 SdLicense 窗体

@00004FEE:000E      endif;

@00004FEE:000E   label_4fee:

@00004FF0:0021      StrToUpper/StrToLower(global_string15, global_string15);------->注册编保存在 global_string15,转换大小写

@00004FFC:0021      StrLength(global_string15);------->求注册码长度,返回结果保存在:LASTRESULT

@00005005:0006      local_number1 = LASTRESULT;

@0000500F:000E      local_number9 = (local_number1 != 29);------>其是否等于 29 位

@0000501E:000E      local_number10 = (local_number1 != 23);----->其长度是否等于 23 位

@0000502D:0019      local_number9 = (local_number9 = local_number10);------>长度是否等于 29 位或 23 位.

@0000503A:0004      if(local_number9) then // ref index: 1----->不等于就错误

@00005046:0021        function_242("REGISTER_STRING_ERROR");

@00005064:0006        local_string18 = LASTRESULT;

@0000506E:0021        SetDialogTitle(2, local_string18);----->设置对话框标题

@0000507C:0021        function_242("REGISTER_STRING_AGAIN");

@0000509A:0006        local_string18 = LASTRESULT;

@000050A4:0021        MessageBox(local_string18, -65534);------->提示错误对话框

@000050B2:0005        goto label_4f95;----------->回滚到 SdLicense 窗体

@000050BB:0013      endif;

@000050BB:0013   label_50bb:
```



@000050BD:0029 StrSub(local\_string14, global\_string15, 5, 1);---->从注册码第 5 位后取出 1 位给 local\_string14

@000050D1:0029 StrSub(local\_string15, global\_string15, 11, 1);---->从注册码第 11 位后取出 1 位给 local\_string15

@000050E5:0029 StrSub(local\_string16, global\_string15, 17, 1);---->从注册码第 17 位后取出 1 位给 local\_string16

@000050F9:0029 StrSub(local\_string17, global\_string15, 23, 1);---->从注册码第 23 位后取出 1 位给 local\_string17

@0000510D:000D local\_number9 = (local\_number1 = 23);------->注册码长度是否等于 23 位

@0000511C:0004 if(local\_number9) then // ref index: 2----->不等就跳

@00005128:000E local\_number9 = (local\_string14 != "-");---->local\_string14 是否等字符"-"

@00005136:000E local\_number10 = (local\_string15 != "-");---->local\_string15 是否等字符"-"

@00005144:0018 local\_number9 = (local\_number9 || local\_number10);-->结果相或

@00005151:000E local\_number10 = (local\_string16 != "-");--->local\_string16 是否等字符"-"

@0000515F:0018 local\_number9 = (local\_number9 || local\_number10);--->结果相或

@0000516C:0004 if(local\_number9) then // ref index: 1

@00005178:0021 function\_242("REGISTER\_STRING\_ERROR");

@00005196:0006 local\_string18 = LASTRESULT;

@000051A0:0021 SetDialogTitle(2, local\_string18);----->设置对话框标题

@000051AE:0021 function\_242("REGISTER\_STRING\_AGAIN");

@000051CC:0006 local\_string18 = LASTRESULT;

@000051D6:0021 MessageBox(local\_string18, -65534);------->提示错误对话框

@000051E4:0005 goto label\_4f95;----------->回滚到 SdLicense 窗体

@000051ED:0001 endif;

@000051ED:0001 label\_51ed:

@000051EF:0005 goto label\_546e;------>如果其长度等于 23 位,就跳转到 label\_546e

@000051F8:0012 endif;

@000051F8:0012 label\_51f8:

@000051FA:000D local\_number9 = (local\_number1 = 29);------->注册码长度是否等于 29 位

@00005209:0004 if(local\_number9) then // ref index: 5

@00005215:000E local\_number9 = (local\_string14 != "-");---->local\_string14 是否等字符"-"

@00005223:000E local\_number10 = (local\_string15 != "-");---->local\_string15 是否等字符"-"

@00005231:0018 local\_number9 = (local\_number9 || local\_number10);-->结果相或

@0000523E:000E local\_number10 = (local\_string16 != "-");--->local\_string16 是否等字符"-"

@0000524C:0018 local\_number9 = (local\_number9 || local\_number10);--->结果相或

@00005259:000E local\_number10 = (local\_string17 != "-");--->local\_string17 是否等字符"-"

@00005267:0018 local\_number9 = (local\_number9 || local\_number10);--->结果相或

@00005274:0004 if(local\_number9) then // ref index: 1

@00005280:0021 function\_242("REGISTER\_STRING\_ERROR");

@0000529E:0006 local\_string18 = LASTRESULT;

@000052A8:0021 SetDialogTitle(2, local\_string18);----->设置错误对话框标题

@000052B6:0021 function\_242("REGISTER\_STRING\_AGAIN");

@000052D4:0006 local\_string18 = LASTRESULT;

@000052DE:0021 MessageBox(local\_string18, -65534);------->提示错误对话框

@000052EC:0005 goto label\_4f95;------->回滚到 SdLicense 窗体

@000052F5:0005 goto label\_546e;

@000052FE:0001 endif;

@000052FE:0001 label\_52fe:

@00005300:0006 local\_number8 = 24;----->整数变量 local\_number8 赋于 24,它是循环变量计算器

@0000530C:0010 label\_530c:

@0000530E:000B local\_number9 = (local\_number8 <= 28);--->循环比较注册码字符是否在 A-z,0-9 之间

@0000531D:0004 if(local\_number9) then // ref index: 3

@00005329:0029 StrSub(local\_string13, global\_string15, local\_number8, 1);---->从注册码第 24 位后取出 1 位给 local\_string13

@0000533B:002B StrCompare(local\_string13, "A");----->与"A"比较

@00005346:0006 local\_number6 = LASTRESULT;---->结果给 local\_number6

@00005350:002B StrCompare(local\_string13, "0");----->与"0"比较

@0000535B:0006 local\_number7 = LASTRESULT;---->结果给 local\_number7

@00005365:000C local\_number9 = (local\_number6 >= 0);

@00005374:000B local\_number10 = (local\_number6 <= 25);

@00005383:0019 local\_number9 = (local\_number9 = local\_number10);上两句意思是:local\_string13 是否在 asii 码:A-z 之间

@00005390:000C local\_number10 = (local\_number7 >= 0);

@0000539F:000B local\_number11 = (local\_number7 <= 9);

@000053AE:0019 local\_number10 = (local\_number10 = local\_number11);上两句意思是:local\_string13 是否在数字:0-9 之间

@000053BB:0018 local\_number9 = (local\_number9 || local\_number10);

@000053C8:0004 if(local\_number9) then // ref index: 1-------->符合条件就继续往下跳

@000053D4:0005 goto label\_5454;------->转到 label\_5454

@000053DD:0007 endif;

@000053DD:0007 label\_53dd:

@000053DF:0021 function\_242("REGISTER\_STRING\_ERROR");

@000053FD:0006 local\_string18 = LASTRESULT;

@00005407:0021 SetDialogTitle(2, local\_string18);----->设置错误对话框标题

@00005415:0021 function\_242("REGISTER\_STRING\_AGAIN");

@00005433:0006 local\_string18 = LASTRESULT;

@0000543D:0021 MessageBox(local\_string18, -65534);------->提示错误对话框

@0000544B:0005 goto label\_4f95;

@00005454:0002 label\_5454:

@00005456:0007 local\_number8 = (local\_number8 + 1);----->整数变量 local\_number8 赋于 24,它是循环变量计算器

@00005465:0005 goto label\_530c;------->上跳 label\_530c 循环比较注册码位是否是在 0-9，A-Z

@0000546E:0001 endif;

@0000546E:0001 endif;

@0000546E:0001 label\_546e:

@00005470:0006 local\_number8 = 0;--->循环记数器清 0

@0000547C:0016 label\_547c:

@0000547E:000B local\_number9 = (local\_number8 <= 22);---循环 23 次.从 0-22 次.

@0000548D:0004 if(local\_number9) then // ref index: 3

@00005499:000E local\_number9 = (local\_number8 != 5);--->跳过第 5 次

@000054A8:000E local\_number10 = (local\_number8 != 11);--->跳过第 11 次

@000054B7:0019 local\_number9 = (local\_number9 = local\_number10);

@000054C4:000E local\_number10 = (local\_number8 != 17);--->跳过第 17 次

@000054D3:0019 local\_number9 = (local\_number9 = local\_number10);

@000054E0:0004 if(local\_number9) then // ref index: 2

@000054EC:0029 StrSub(local\_string13, global\_string15, local\_number8, 1);---->分别依次提取注册码 0-22 各个位

@000054FE:002B StrCompare(local\_string13, "A");----->与"A"比较

@00005509:0006 local\_number6 = LASTRESULT;

@00005513:002B StrCompare(local\_string13, "0");----->与"0"比较

@0000551E:0006 local\_number7 = LASTRESULT;

@00005528:000C local\_number9 = (local\_number6 >= 0);

@00005537:000B local\_number10 = (local\_number6 <= 25);上两句意思是:注册码各个位是否在 asii 码:A-z 之间

@00005546:0019 local\_number9 = (local\_number9 = local\_number10);

@00005553:000C local\_number10 = (local\_number7 >= 0);

@00005562:000B local\_number11 = (local\_number7 <= 9);两句意思是:注册码各个位是否在 asii 码:0-9 之间

@00005571:0019 local\_number10 = (local\_number10 = local\_number11);---->比较这两中情况是否对

@0000557E:0018 local\_number9 = (local\_number9 || local\_number10);--->相或

@0000558B:0004 if(local\_number9) then // ref index: 1---->如果对就继续下一循环

@00005597:0005 goto label\_5617;---------------------------->下跳

@000055A0:0007 endif;

@000055A0:0007 label\_55a0:-

@000055A2:0021 function\_242("REGISTER\_STRING\_ERROR");

@000055C0:0006 local\_string18 = LASTRESULT;

@000055CA:0021 SetDialogTitle(2, local\_string18);----->设置错误对话框标题

@000055D8:0021 function\_242("REGISTER\_STRING\_AGAIN");

@000055F6:0006 local\_string18 = LASTRESULT;



## 代码段分析

```asm
@00005600:0021             MessageBox(local_string18, -65534);------->提示错误对话框

@0000560E:0005             goto label_4f95;

@00005617:0002          endif;

@00005617:0002     label_5617:

@00005619:0007          local_number8 = (local_number8 + 1);---->循环记数器加 1,直到 23 次

@00005628:0005          goto label_547c;------------------------------------->循环 向上跳转到:label_547c,从头再开始

@00005631:0008      endif;

@00005631:0008     label_5631:------------------------------------------------------>跳转到这里就成功了.

@00005633:0006      local_string1 = "";

@0000563D:0006      local_string2 = "";

@00005647:0021      SdAskDestPath(local_string1, local_string2, local_string9, 0);----->设置安装目录窗体过程:SdAskDestPath

@0000565B:0006      local_number1 = LASTRESULT;

@00005665:0006      global_string8 = local_string9;

@0000566F:000D      local_number9 = (local_number1 = 12);

@0000567E:0004      if(local_number9) then // ref index: 1

@0000568A:0005          goto label_4f95;

@00005693:0008      endif;

@00005693:0008     label_5693:

@00005695:0006      local_string1 = "";

@0000569F:0006      local_string2 = "";

@000056A9:0021      SetupType(local_string1, local_string2, "", local_number2, 0);---->设置安装类型窗体过程:SetupType

@000056C0:0006      local_number1 = LASTRESULT;

@000056CA:000D      local_number9 = (local_number1 = 12);

@000056D9:0004      if(local_number9) then // ref index: 1

@000056E5:0005          goto label_5631;

@000056EE:0005          goto label_5775;

@000056F7:000A      endif;
```

注册码很简单的形式:

23 位

`xxxxx-xxxxx-xxxxx-xxxxx`

29 位：

`xxxxx-xxxxx-xxxxx-xxxxx-xxxxx`

就可以欺骗过去.

输入 x 范围限制在 ACSII 码:`A-z`,`0-9`.

如果其中一位输入特殊字符如:`~!@#$%^&*()_+/?><,.\\] \\[` 就失败.

安装完毕之后,看看功能发现这个软件是要狗的.随便看看原来是深思的狗 3 代(狗早就被蹂躏 NK 次了).由于无狗只做简单静态分析(也可参见牛人文章)

### 应用层读狗设备名的地方:

```asm
0040C2EE   |>  \6A 00         push    0                             ; /hTemplateFile = NULL; Case 2 of switch 0040C2C7
0040C2F0   |.   6A 00        push    0                             ; |Attributes = 0
0040C2F2   |.   6A 03        push    3                             ; |Mode = OPEN_EXISTING
0040C2F4   |.   6A 00        push    0                             ; |pSecurity = NULL
0040C2F6   |.   6A 01        push    1                             ; |ShareMode = FILE_SHARE_READ
0040C2F8   |.   68 00000080  push    80000000                      ; |Access = GENERIC_READ
0040C2FD   |.   68 04524100  push    00415204                      ; |FileName = "\\\\.\\SENSE3Dev"
0040C302   |.   FF15 C0004100 call   dword ptr [<&KERNEL32.CreateFile>; \CreateFileA
0040C345   |.   6A 00        push    0                             ; /hTemplateFile = NULL
0040C347   |.   68 00000004  push    4000000                       ; |Attributes = DELETE_ON_CLOSE
0040C34C   |.   6A 01        push    1                             ; |Mode = CREATE_NEW
0040C34E   |.   6A 00        push    0                             ; |pSecurity = NULL
0040C350   |.   6A 00        push    0                             ; |ShareMode = 0
0040C352   |.   6A 00        push    0                             ; |Access = 0
0040C354   |.   51           push    ecx                           ; |FileName => "\\\\.\\SENSE3.VXD"
0040C355   |.   FF15 C0004100 call   dword ptr [<&KERNEL32.CreateFile>; \CreateFileA
0040C42C   |> /6A 00        /push    0
0040C42E   |. |6A 00        |push    0
0040C430   |. |6A 03        |push    3
0040C432   |. |6A 00        |push    0
0040C434   |. |6A 00        |push    0
0040C436   |. |68 000000C0  |push    C0000000
0040C43B   |. |68 48524100  |push    00415248                     ;  ASCII "\\\\.\\LPT1"
0040C440   |. |FFD3         |call   ebx------------->CreateFileA
```

=======================================================================================================================

### 分析驱动层 sense3.sys 创建设备对象和设备连接符号如下:

```asm
.text:00010458                   push    ebp
.text:00010459                   mov     ebp, esp
.text:0001045B                   sub     esp, 10h
.text:0001045E                   push    ebx
.text:0001045F                   push    esi
.text:00010460                   push    [ebp+SourceString] ; SourceString
.text:00010463                   mov     esi, ds:RtlInitUnicodeString
.text:00010469                   lea     eax, [ebp+DeviceName]
.text:0001046C                   push    eax               ; DestinationString
.text:0001046D                   call    esi ; RtlInitUnicodeString
.text:0001046F                   mov     ebx, [ebp+DeviceObject]
.text:00010472                   lea     eax, [ebp+DeviceName]
.text:00010475                   push    ebx               ; DeviceObject
.text:00010476                   push    0                 ; Exclusive
.text:00010478                   push    0                 ; DeviceCharacteristics
.text:0001047A                   push    [ebp+DeviceType] ; DeviceType
.text:0001047D                   push    eax               ; DeviceName------------>"\\\\Device\\\\SENSE3
.text:0001047E                   push    20h               ; DeviceExtensionSize
.text:00010480                   push    [ebp+DriverObject] ; DriverObject
.text:00010483                   call    ds:IoCreateDevice--------------------------->创建\\\\Device\\\\SENSE3 狗设备对象
.text:00010489                   test    eax, eax
.text:0001048B                   jl      short loc_10
.text:0001049A                   lea     eax, [ebp+SymbolicLinkName]
.text:0001049D                   push    offset s_DosdevicesSen ; "\\\DosDevices\\\SENSE3Dev"
.text:000104A2                   push    eax               ; DestinationString
.text:000104A3                   call    esi ; RtlInitUnicodeString
.text:000104A5                   lea     eax, [ebp+DeviceName]
.text:000104A8                   push    eax               ; DeviceName
.text:000104A9                   lea     eax, [ebp+SymbolicLinkName]
.text:000104AC                   push    eax               ; SymbolicLinkName
.text:000104AD                   call    ds:IoCreateSymbolicLink----------------创建\\\\DosDevices\\\\SENSE3Dev 连接符号给应用层
.text:000104B3                   mov     esi, eax
.text:000104B5                   pop     edi
```

========================================================================================================================

### 分析应用层与狗驱动交互的地方:

```asm
0040C407   |.   6A 00        push    0                             ; /pOverlapped = NULL
0040C409   |.   51           push    ecx                           ; |pBytesReturned
0040C40A   |.   6A 60        push    60                            ; |OutBufferSize = 60 (96.)
0040C40C   |.   57           push    edi                           ; |OutBuffer
0040C40D   |.   6A 60        push    60                            ; |InBufferSize = 60 (96.)
0040C40F   |.   57           push    edi                           ; |InBuffer
0040C410   |.   68 0464409C  push    9C406404                      ; |IoControlCode = 9C406404--------->读狗控制码
0040C415   |.   52           push    edx                           ; |hDevice
0040C416   |.   FF15 94004100 call   dword ptr [<&KERNEL32.DeviceIoCo>; \DeviceIoControl
0040C499   |.   6A 00        push    0                             ; /pOverlapped = NULL
0040C49B   |.   51           push    ecx                           ; |pBytesReturned
0040C49C   |.   6A 60        push    60                            ; |OutBufferSize = 60 (96.)
0040C49E   |.   57           push    edi                           ; |OutBuffer
0040C49F   |.   6A 60        push    60                            ; |InBufferSize = 60 (96.)
0040C4A1   |.   57           push    edi                           ; |InBuffer
0040C4A2   |.   68 0F002200  push    22000F                        ; |IoControlCode = 22000F--------->读狗控制码
```



```
0040C4A7   |.   52                 push    edx                              ; |hDevice  

0040C4A8   |.   FF15 94004100 call    dword ptr [<&KERNEL32.DeviceIoCo>; \DeviceIoControl  

0040D811   |.   8B1D 94004100 mov     ebx, dword ptr [<&KERNEL32.Devic>;  kernel32.DeviceIoControl  

0040D817   |.   55                 push    ebp  

0040D818   |.   56                 push    esi  

0040D819   |.   8B7424 14      mov     esi, dword ptr [esp+14]  

0040D81D   |.   57                 push    edi  

0040D81E   |.   8B7C24 14      mov     edi, dword ptr [esp+14]  

0040D822   |.   33ED              xor     ebp, ebp  

0040D824   |.   C606 88        mov     byte ptr [esi], 88  

0040D827   |>   8D4424 18      /lea     eax, dword ptr [esp+18]  

0040D82B   |.   6A 00          |push    0  

0040D82D   |.   50             |push    eax  

0040D82E   |.   6A 08          |push    8  

0040D830   |.   56             |push    esi  

0040D831   |.   6A 00          |push    0  

0040D833   |.   6A 00          |push    0  

0040D835   |.   68 20002200   |push    220020----------->读狗控制码  

0040D83A   |.   57             |push    edi  

0040D83B   |.   FFD3           |call    ebx  

0040D7B4   |.   6A 00          push    0                              ; /pOverlapped = NULL  

0040D7B6   |.   51             push    ecx                            ; |pBytesReturned  

0040D7B7   |.   8B4C24 58      mov     ecx, dword ptr [esp+58]        ; |  

0040D7BB   |.   8D5424 14      lea     edx, dword ptr [esp+14]        ; |  

0040D7BF   |.   66:894424 12  mov     word ptr [esp+12], ax          ; |  

0040D7C4   |.   56             push    esi                            ; |OutBufferSize  

0040D7C5   |.   52             push    edx                            ; |OutBuffer  

0040D7C6   |.   8D4424 14      lea     eax, dword ptr [esp+14]        ; |  

0040D7CA   |.   6A 08          push    8                              ; |InBufferSize = 8  

0040D7CC   |.   50             push    eax                            ; |InBuffer  

0040D7CD   |.   68 10002200   push    220010                         ; |IoControlCode = 220010----->读狗控制码  

0040D7D2   |.   51             push    ecx                            ; |hDevice  

0040D7D3   |.   C64424 24 C0  mov     byte ptr [esp+24], 0C0         ; |  

0040D7D8   |.   C64424 25 00  mov     byte ptr [esp+25], 0           ; |  

0040D7DD   |.   66:C74424 28 >mov     word ptr [esp+28], 0           ; |  

0040D7E4   |.   FF15 94004100 call    dword ptr [<&KERNEL32.DeviceIoCo>; \DeviceIoControl  

0040D768   |.   6A 00          push    0                              ; /pOverlapped = NULL  

0040D76A   |.   51             push    ecx                            ; |pBytesReturned  

0040D76B   |.   6A 00          push    0                              ; |OutBufferSize = 0  

0040D76D   |.   6A 00          push    0                              ; |OutBuffer = NULL  

0040D76F   |.   8D5424 10      lea     edx, dword ptr [esp+10]       ; |  

0040D773   |.   6A 48          push    48                             ; |InBufferSize = 48 (72.)  

0040D775   |.   52             push    edx                            ; |InBuffer  

0040D776   |.   68 0C002200   push    22000C                         ; |IoControlCode = 22000C--------->读狗控制码  

0040D77B   |.   50             push    eax                            ; |hDevice  

0040D77C   |.   FF15 94004100 call    dword ptr [<&KERNEL32.DeviceIoCo>; \DeviceIoControl  
```

===============================================================================================================================

分析驱动层 `sense3.sys` `Dispatch` `Function` 只处理了三个 `MajorFunction`: `IRP_MJ_CREATE`, `IRP_MJ_DEVICE_CONTROL`, `IRP_MJ_CLOSE` 三个例程, 这三个例程

又共享一个例程: `sub_104CA` 如下驱动层：

```
.text:0001030C

.text:0001030C                 push    ebp

.text:0001030D                 mov     ebp, esp

.text:0001030F                 sub     esp, 44h

.text:00010312                 push    ebx

.text:00010313                 push    esi

.text:00010314                 push    edi

.text:00010315                 push    9

.text:00010317                 pop     ecx

.text:00010318                 xor     eax, eax

.text:0001031A                 lea     edi, [ebp+DriverList]

.text:0001031D                 xor     esi, esi

.text:0001031F                 rep stosd

.text:00010321                 mov     dword ptr [ebp+DriverList.List.PartialResourceList.PartialDescriptors.u+4], esi

.text:00010324                 mov     esi, [ebp+DriverObject]

.text:00010327                 mov     eax, offset sub_104CA----------->驱动服务 IRP 共享一个例程:sub_104CA

.text:0001032C                 push    1

.text:0001032E                 mov     [esi+38h], eax-------------->IRP_MJ_CREATE

.text:00010331                 mov     [esi+40h], eax------------------>IRP_MJ_CLOSE

.text:00010334                 mov     [esi+70h], eax--------------------->IRP_MJ_DEVICE_CONTROL

.text:00010337                 pop     ebx

.text:00010338                 lea     eax, [ebp+DeviceObject]

.text:0001033B                 xor     edx, edx

.text:0001033D                 push    eax                 ; DeviceObject

.text:0001033E                 push    esi                 ; DriverObject

.text:0001033F                 mov     edi, 3C4h

.text:00010344                 push    9C40h               ; DeviceType
```

`IRP_MJ_CREATE`, `IRP_MJ_DEVICE_CONTROL`, `IRP_MJ_CLOSE` 三个例程, 在设备狗对象共享一个例程 `sub_104CA` 如下：

下面的驱动偶不做分析了.

```
text:000104CA

.text:000104CA                 push    ebp

.text:000104CB                 mov     ebp, esp

.text:000104CD                 sub     esp, 54h

.text:000104D0                 mov     eax, [ebp+arg_0]

.text:000104D3                 mov     ecx, [ebp+Irp]

.text:000104D6                 push    esi

.text:000104D7                 push    edi

.text:000104D8                 mov     esi, [eax+28h]

.text:000104DB                 mov     eax, [ecx+60h]

.text:000104DE                 mov     edx, [ecx+0Ch]

.text:000104E1                 xor     edi, edi

.text:000104E3                 cmp     byte ptr [eax], 0Eh

.text:000104E6                 mov     [ebp+var_4], edx

.text:000104E9                 mov     [ebp+arg_0], edi

.text:000104EC                 jnz     loc_105BC

.text:000104EC

.text:000104F2                 cmp     dword ptr [eax+8], 60h

.text:000104F6                 jnz     loc_105D9

.text:000104F6

.text:000104FC                 cmp     dword ptr [eax+4], 60h

.text:00010500                 jnz     loc_105D9

.text:00010500

.text:00010506                 cmp     dword ptr [eax+0Ch], 9C406404h

.text:0001050D                 jnz     loc_105BC

.text:0001050D

.text:00010513                 push    ebx

.text:00010514                 mov     ebx, ds:KeInitializeEvent

.text:0001051A                 push    edi                 ; State

.text:0001051B                 lea     eax, [ebp+Object]

.text:0001051E                 push    edi                 ; Type

.text:0001051F                 push    eax                 ; Event

.text:00010520                 call    ebx ; KeInitializeEvent

.text:00010522                 lea     eax, [ebp+IoStatusBlock]

.text:00010525                 push    eax                 ; IoStatusBlock

.text:00010526                 lea     eax, [ebp+Object]

.text:00010529                 push    eax                 ; Event

.text:0001052A                 push    1                   ; InternalDeviceIoControl

.text:0001052C                 push    edi                 ; OutputBufferLength

.text:0001052D                 push    edi                 ; OutputBuffer

.text:0001052E                 push    edi                 ; InputBufferLength

.text:0001052F                 push    edi                 ; InputBuffer

.text:00010530                 push    dword ptr [esi+18h] ; DeviceObject

.text:00010533                 push    16002Ch              ; IoControlCode
```



```asm
.text:00010538          call    ds:IoBuildDeviceIoControlRequest  

.text:0001053E          mov     ecx, [esi+18h]  ; DeviceObject  

.text:00010541          mov     edx, eax        ; Irp  

.text:00010543          call    ds:IofCallDriver  

.text:00010549          cmp     eax, 103h  

.text:0001054E          jnz     short loc_1055E  

.text:0001054E  

.text:00010550          push    edi             ; Timeout  

.text:00010551          push    edi             ; Alertable  

.text:00010552          push    edi             ; WaitMode  

.text:00010553          lea     eax, [ebp+Object]  

.text:00010556          push    edi             ; WaitReason  

.text:00010557          push    eax             ; Object  

.text:00010558          call    ds:KeWaitForSingleObject  
```

简单分析完毕了。很菜的。

关键地方很容易找到。这里不描述了。大牛别踩！

`4dd466d0e1e0cc123afbe504da2462ca`
