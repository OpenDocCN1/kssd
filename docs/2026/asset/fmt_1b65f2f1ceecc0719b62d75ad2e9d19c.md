
# [原创]计算机软件水平考试测试系统 2005 V5.0 网络工程师版

软件大小： 19465 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 考试系统

应用平台： Win9x/NT/2000/XP

界面预览： 无

加入时间： 2005-02-27 10:04:28

下载次数： 27337

下载地址：afeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4C8P5h3y4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0V1I4z5e0q4Q4x3X3g2Z5N6r3#2D9

加壳方式：PE-Armor V0.46-> Hying \*

这个Hying旧版的，估计版本应该是最老的

OD载入,略去所有异常：

入口：

```
0073F000 >  E8 AA000000     call NE_St.0073F0AF
0073F005    2D F0330000     sub eax,33F0
0073F00A    0000            add byte ptr ds:[eax],al
0073F00C    0000            add byte ptr ds:[eax],al
0073F00E    0000            add byte ptr ds:[eax],al
0073F010    003D F033002D   add byte ptr ds:[2D0033F0],bh
0073F016    F0:3300         lock xor eax,dword ptr ds:[eax] ; 不允许锁定前缀
0073F019    0000            add byte ptr ds:[eax],al
0073F01B    0000            add byte ptr ds:[eax],al
0073F01D    0000            add byte ptr ds:[eax],al
```

Alt+M 打开内存镜象，如下:

```
----------------------------------------------------------------------
内存镜像
地址??? 大小??? Owner???区段??? 包含??????? 类型? 访问???初始访问?
00400000? 00001000? MASM32??????? PE header????? Imag? R???? RWE
00401000? 00003000? MASM32??????? code????????Imag? R???? RWE
00404000? 00001000? MASM32??????? resources????? Imag? R???? RWE
00405000? 00001000? MASM32??????? SFX,data,imports??Imag? R???? RWE
```

在code区段下内存写入断点，F9运行，中断在如下:

```
0073F105    A4              movs byte ptr es:[edi],byte ptr ds>
0073F106    B3 02           mov bl,2
0073F108    E8 6D000000     call NE_St.0073F17A
0073F10D  ^ 73 F6           jnb short NE_St.0073F105
0073F10F    33C9            xor ecx,ecx
0073F111    E8 64000000     call NE_St.0073F17A
0073F116    73 1C           jnb short NE_St.0073F134
```

下命令`bp GetProcAddress`, F9运行，中断后取消断点，

Alt+F9返回在如下:

```
0037302B    8907            mov dword ptr ds:[edi],eax  ; ntdll.RtlDeleteCriticalSection //中断在这里
0037302D    5A              pop edx
0037302E    0FB642 FF       movzx eax,byte ptr ds:[edx-1]
00373032    03D0            add edx,eax
00373034    42              inc edx
00373035    83C7 04         add edi,4
00373038    59              pop ecx
00373039  ^ E2 CA           loopd short 00373005
0037303B  ^ EB 93           jmp short 00372FD0//循环初始化IAT
0037303D    8B85 BC020000   mov eax,dword ptr ss:[ebp+2BC]
00373043    83F8 01         cmp eax,1
00373046    75 27           jnz short 0037306F
00373048    8BBD C4020000   mov edi,dword ptr ss:[ebp+2C4]
0037304E    03FD            add edi,ebp
00373050    8DB5 4D020000   lea esi,dword ptr ss:[ebp+24D]
00373056    8B07            mov eax,dword ptr ds:[edi]
00373058    0BC0            or eax,eax
0037305A    75 02           jnz short 0037305E
0037305C    EB 11           jmp short 0037306F
0037305E    25 FFFFFF7F     and eax,7FFFFFFF
00373063    8BDE            mov ebx,esi///////这里开始打补丁，改为JMP 00373185
00373065    2BD8            sub ebx,eax
00373067    8958 FC         mov dword ptr ds:[eax-4],ebx
0037306A    83C7 08         add edi,8
0037306D  ^ EB E7           jmp short 00373056
0037306F    64:FF35 3000000>push dword ptr fs:[30]
00373076    58              pop eax
00373077    85C0            test eax,eax
00373079    78 0F           js short 0037308A
0037307B    8B40 0C         mov eax,dword ptr ds:[eax+C]
0037307E    8B40 0C         mov eax,dword ptr ds:[eax+C]
00373081    C740 20 0010000>mov dword ptr ds:[eax+20],1000
00373088    EB 1C           jmp short 003730A6
0037308A    6A 00           push 0
0037308C    FF95 A8020000   call dword ptr ss:[ebp+2A8]
00373092    85D2            test edx,edx
00373094    79 10           jns short 003730A6
00373096    837A 08 FF      cmp dword ptr ds:[edx+8],-1
0037309A    75 0A           jnz short 003730A6
0037309C    8B52 04         mov edx,dword ptr ds:[edx+4]
0037309F    C742 50 0010000>mov dword ptr ds:[edx+50],1000
003730A6    89AD 58020000   mov dword ptr ss:[ebp+258],ebp
003730AC    8B85 C8020000   mov eax,dword ptr ss:[ebp+2C8]
003730B2    0385 B4020000   add eax,dword ptr ss:[ebp+2B4]
003730B8    FFE0            jmp eax?//这里跳往OEP          ; NE_St.00665AB4
```

到OEP后跟进任何一个CALL，可以看到跳转表被加密了，不过解码再简单不过了

随便在此内存段末尾找片空白片，我找的是00373185这里，所以在00341EAA代码改成了JMP 00373185)

```
00373185    8BF0            mov esi,eax                        ; NE_St.004014BA
00373187    83C6 FA         add esi,-6
0037318A    66:C706 FF25    mov word ptr ds:[esi],25FF
0037318F    8B47 04         mov eax,dword ptr ds:[edi+4]
00373192    8946 02         mov dword ptr ds:[esi+2],eax
00373195  ^ E9 D0FEFFFF     jmp 0037306A
```

补丁完后F4到003730B8这行就到OEP，DUMP、修复之后就可以运行了。

用Peid 检测为Borland Delphi 6.0 - 7.0

## 2.解除自校验

BP CreateFileA,F9,请看堆栈：

开始检测调试器了

```
0012FDB0   0065C1F3   /CALL 到 CreateFileA 来自 1\_.0065C1EE
0012FDB4   0065C204   |FileName = "\\.\SICE"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING
0012FDB0   0065C3CB   /CALL 到 CreateFileA 来自 1\_.0065C3C6
0012FDB4   0065C3DC   |FileName = "\\.\TRWDEBUG"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING
0012FDB0   0065C40B   /CALL 到 CreateFileA 来自 1\_.0065C406
0012FDB4   0065C41C   |FileName = "\\.\TRW2000"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING
```

没了，看来只检测2个最出名的：）

```
0011FC30   00403384   /CALL 到 CreateFileA 来自 1\_.0040337F
0011FC34   0012FCB4   |FileName = "C:\Flyhua\网络工程师\1\_.exe"
0011FC38   80000000   |Access = GENERIC_READ
0011FC3C   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0011FC40   00000000   |pSecurity = NULL
```

返回

```
0065F60A    33D2            xor edx,edx
0065F60C    52              push edx
0065F60D    50              push eax
0065F60E    8D55 F0         lea edx,dword ptr ss:[ebp-10]
0065F611    B8 08000000     mov eax,8
0065F616    E8 89A4DAFF     call 1\_.00409AA4
0065F61B    8B45 F0         mov eax,dword ptr ss:[ebp-10]      ; 19F76797
0065F61E    8D55 F4         lea edx,dword ptr ss:[ebp-C]
0065F621    E8 86FEFFFF     call 1\_.0065F4AC
0065F626    8B45 F4         mov eax,dword ptr ss:[ebp-C]       ; 57856
0065F629    50              push eax
0065F62A    8D45 E8         lea eax,dword ptr ss:[ebp-18]
0065F62D    E8 F2FEFFFF     call 1\_.0065F524
0065F632    8B45 E8         mov eax,dword ptr ss:[ebp-18]      ; CA719B6F
0065F635    8D55 EC         lea edx,dword ptr ss:[ebp-14]
0065F638    E8 6FFEFFFF     call 1\_.0065F4AC
0065F63D    8B55 EC         mov edx,dword ptr ss:[ebp-14]      ; 61824
0065F640    58              pop eax                            ; 57856
0065F641    E8 5656DAFF     call 1\_.00404C9C
0065F646    74 14           je short 1\_.0065F65C               //改为JMP 0065F65C
```

这里并非是自校验比较的关键地，而是注册成功与否的关键比较地，试想一下如果改为注册成功就能使软件运行，何乐而不为呢，

我才不管你是不是还在自身比较呢：）

## 3。寻找算法

输入注册信息：

申请码：NEBS-4088-C747-9238-8222

认证码：11111111-22222222

        33333333-44444444

提示重启验证注册码。注册信息保存在目录下`\DATA\friend.ini`中。

```
-------------------------------------
[Options]
aa=11111111222222223333333344444444
ab=F26C49BAD73D8B3F1E9BA973B646FFDF
--------------------------------------
```

用OLLYDB加载运行。搜索提示字符串`DATA\friend.ini`的找到两处，分别下断点

```
0065F54D    8B45 FC         mov eax,dword ptr ss:[ebp-4]        ;ASCII "F26C49BAD73D8B3F1E9BA973B646FFDF"
0065F550    8A40 03         mov al,byte ptr ds:[eax+3]         // "C"
0065F553    8845 F4         mov byte ptr ss:[ebp-C],al
0065F556    8B45 FC         mov eax,dword ptr ss:[ebp-4]
0065F559    8A40 07         mov al,byte ptr ds:[eax+7]         //"A"
0065F55C    8845 F5         mov byte ptr ss:[ebp-B],al
0065F55F    8B45 FC         mov eax,dword ptr ss:[ebp-4]
```
