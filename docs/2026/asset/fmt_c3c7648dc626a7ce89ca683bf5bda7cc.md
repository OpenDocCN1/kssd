
# 【原创】手把手系列之二―Axman 3.12rGB的注册算法详解及注册机

【破解作者】 jackily
【作者邮箱】 jackily\_zhang@msn.com；jackily\_zhang@yahoo.com.cn
【作者主页】 http://estudy.ys168.com
【使用工具】 `ollydbg` 1.10
【破解平台】 Win9x/NT/2000/XP
【软件名称】 `axman` 3.12rGB
【下载地址】 http://www.pc123.cn/SoftView/SoftView\_5625.html
【软件简介】 一个非常不错的文件分割工具，该工具的主要作用是可以将大文件切成几个小文件便于拷贝到软盘或作为电子邮件附件发送。对于分割的文件，程序提供自动合并功能，同时该工具也提供非常简单的操作界面和操作方式以方便用户使用。未注册版本有30天的使用限制，并在开始程序时有nag窗口。
【破解声明】 本破解纯属以学习为目的，偶得一点心得，愿与大家分享：） xiA Qin 曾破解过3.10版（具体参见看雪精华II），但只是给了一个注册码而已，并没有给出算法和注册机，3.12版程序稍有变化，我们的目的是要找到算法并写出注册机。

## 【破解内容】

安装完毕后主文件为`axman.exe`，用`peid`检查无壳。用`ollydbg`载入，运行，出现提示注册窗口，点击注册，要求输入`name`、`company`和注册码。这里我们分别输入jackily，http://estudy.ys168.com和123456。点确定，弹出“Invalid RegisterInformation”。好了，重新载入，点右键―“搜索”―“字符参考”，找到刚才的ASCII码，地址为`00406933`，向上看，`00406923`处有一`call`，为典型的注册关键调用。在此处下断，跟进。代码分析如下：

```asm
004068DA    .  8D45 F0           lea eax,dword ptr ss:[ebp-10]
004068DD    .  8D8E C8020000     lea ecx,dword ptr ds:[esi+2C8]
004068E3    .  50                push eax
004068E4    .  C645 FC 02        mov byte ptr ss:[ebp-4],2
004068E8    .  E8 B5B00000       call <jmp.&MFC42.#3874>          ;将"jackily"放入地址00674660
004068ED    .  8D45 E8           lea eax,dword ptr ss:[ebp-18]
004068F0    .  8D8E 08030000     lea ecx,dword ptr ds:[esi+308]
004068F6    .  50                push eax
004068F7    .  E8 A6B00000       call <jmp.&MFC42.#3874>          ; 将"http://estudy.ys168.com"放入地址006747a0
004068FC    .  8D45 EC           lea eax,dword ptr ss:[ebp-14]
004068FF    .  8D8E 88020000     lea ecx,dword ptr ds:[esi+288]
00406905    .  50                push eax
00406906    .  E8 97B00000       call <jmp.&MFC42.#3874>          ; 将ASCII "123456")放入地址00674840
0040690B    .  8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
0040690E    .  E8 25FBFFFF       call AXMAN.00406438
00406913    .  FF75 EC           push dword ptr ss:[ebp-14]       ; /Arg3 = 00674840 ASCII "123456"
00406916    .  8D4D E4           lea ecx,dword ptr ss:[ebp-1C]    ; |
00406919    .  C645 FC 03        mov byte ptr ss:[ebp-4],3        ; |
0040691D    .  FF75 E8           push dword ptr ss:[ebp-18]       ; |Arg2 = 006747A0 ASCII "http://estudy.ys168.com"
00406920    .  FF75 F0           push dword ptr ss:[ebp-10]       ; |Arg1 = 00674660 ASCII "jackily"
00406923    .  E8 3CFBFFFF       call AXMAN.00406464                  ；典型的注册模式，有点意思，跟进
00406928    .  84C0              test al,al                                  ; 检验注册标志，这里成功应是非0
0040692A    .  75 18             jnz short AXMAN.00406944            ; 成功则到00406944
0040692C    .  6A 30             push 30
0040692E    .  68 10AC4100       push AXMAN.0041AC10              ;  ASCII "AxMan - Error 400"
00406933    .  68 ECAB4100       push AXMAN.0041ABEC              ;  ASCII "Invalid Registration Information"
00406938    .  8BCE              mov ecx,esi
0040693A    .  E8 21B00000       call <jmp.&MFC42.#4224>
0040693F    .  E9 8E000000       jmp AXMAN.004069D2
00406944    >  8D85 54FEFFFF     lea eax,dword ptr ss:[ebp-1AC]
0040694A    .  68 BCAB4100       push AXMAN.0041ABBC              ; /src = "Thank you for registering your copy of AxMan
"
0040694F    .  50                push eax                         ; |dest
00406950    .  E8 CBB50000       call <jmp.&MSVCRT.strcpy>        ; \strcpy
00406955    .  8D85 54FEFFFF     lea eax,dword ptr ss:[ebp-1AC]
0040695B    .  68 68AB4100       push AXMAN.0041AB68              ; /src = "You will need to stop and restart AxMan to remove  the banner ads from displaying."
00406960    .  50                push eax                         ; |dest
00406961    .  E8 C0B50000       call <jmp.&MSVCRT.strcat>        ; \strcat
00406966    .  83C4 10           add esp,10
00406969    .  8D85 54FEFFFF     lea eax,dword ptr ss:[ebp-1AC]
0040696F    .  8BCE              mov ecx,esi
00406971    .  6A 30             push 30
00406973    .  68 D8A64100       push AXMAN.0041A6D8              ;  ASCII "AxMan"
00406978    .  50                push eax
00406979    .  E8 E2AF0000       call <jmp.&MFC42.#4224>          ; 解除程序限制，以下省略
                .
                .
                .
```

### 由`00406923`跟进，一级调用

```asm
00406464   /$  55                push ebp
00406465   |.  8BEC              mov ebp,esp
00406467   |.  83EC 18           sub esp,18
0040646A   |.  8D45 E8           lea eax,dword ptr ss:[ebp-18]
0040646D   |.  50                push eax
0040646E   |.  FF75 0C           push dword ptr ss:[ebp+C]       ; (ASCII "http://estudy.ys168.com")
00406471   |.  FF75 08           push dword ptr ss:[ebp+8]       ; (ASCII "jackily")
00406474   |.  E8 18000000       call AXMAN.00406491             ；关键call，数据处理，为计算做准备，跟进
00406479   |.  FF75 10           push dword ptr ss:[ebp+10]           ; /s2 = "123456" 我们输入的假码
0040647C   |.  8D45 E8           lea eax,dword ptr ss:[ebp-18]        ; |
0040647F   |.  50                push eax                             ; |s1 = "156-936-773" ,真实的注册码，但不要忘记我们的目                                                                      ；的是找算法，不应满足在找到注册码的水平上
00406480   |.  E8 51BA0000       call <jmp.&MSVCRT.strcmp>            ; \strcmp 比较
00406485   |.  F7D8              neg eax
00406487   |.  59                pop ecx
00406488   |.  1AC0              sbb al,al
0040648A   |.  59                pop ecx
0040648B   |.  FEC0              inc al
0040648D   |.  C9                leave
0040648E   \.  C2 0C00           retn 0C
```

### 由`00406474`跟进，以下为数据处理，为计算做准备，把`name`和`company`字符串小写，再将`"My Love"`(注意大小写和空格）`strcat`在一起，也就是连接在一起。

```asm
00406491   /$  B8 C42B4100       mov eax,AXMAN.00412BC4
00406496   |.  E8 15BA0000       call AXMAN.00411EB0
0040649B   |.  81EC F8000000     sub esp,0F8
004064A1   |.  53                push ebx
004064A2   |.  56                push esi
004064A3   |.  57                push edi
004064A4   |.  8BF9              mov edi,ecx
004064A6   |.  FF75 08           push dword ptr ss:[ebp+8]            ; /s = "jackily"
004064A9   |.  E8 22BA0000       call <jmp.&MSVCRT.strlen>            ; \strlen  取字符串长度
004064AE   |.  8B35 10464100     mov esi,dword ptr ds:[<&MSVCRT.mallo>;  MSVCRT.malloc
004064B4   |.  40                inc eax
004064B5   |.  50                push eax                             ; /size
004064B6   |.  FFD6              call esi                             ; \malloc
004064B8   |.  59                pop ecx
004064B9   |.  8BD8              mov ebx,eax
004064BB   |.  59                pop ecx
004064BC   |.  53                push ebx
004064BD   |.  8BCF              mov ecx,edi
004064BF   |.  FF75 08           push dword ptr ss:[ebp+8]
004064C2   |.  E8 63020000       call AXMAN.0040672A
004064C7   |.  FF75 0C           push dword ptr ss:[ebp+C]            ; /s = "http://estudy.ys168.com"
004064CA   |.  E8 01BA0000       call <jmp.&MSVCRT.strlen>            ; \strlen 取字符串长度
            .
            .                                                         ；004064CF至0040651A非关键部分省略
            .
0040651F   |.  FF75 EC           push dword ptr ss:[ebp-14]           ; /src = "jackily"
00406522   |.  8D85 FCFEFFFF     lea eax,dword ptr ss:[ebp-104]       ; |(ASCII "http://estudy.ys168.com")
00406528   |.  50                push eax                             ; |dest = 0065EB98
00406529   |.  E8 F2B90000       call <jmp.&MSVCRT.strcpy>            ; \strcpy
0040652E   |.  FF75 F0           push dword ptr ss:[ebp-10]           ; /src = "http://estudy.ys168.com"
00406531   |.  8D85 FCFEFFFF     lea eax,dword ptr ss:[ebp-104]       ; | (ASCII "jackily")
00406537   |.  50                push eax                             ; |dest "jackily"
00406538   |.  E8 E9B90000       call <jmp.&MSVCRT.strcat>            ; \strcat
```

