

# ExeCryptor 2.2.X 脱壳

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-638.htm`](https://www.pediy.com/kssd/pediy08/pediy8-638.htm)

【文章标题】: 脱 ExeCryptor2.2.50  
【文章作者】: wynney  
【软件名称】: UnPackMe ExeCryptor2.2.50.j  
【软件大小】: 481K  
【下载地址】: 自己搜索下载  
【加壳方式】: ExeCryptor2.2.50  
【保护方式】: All Protection Enabled + 100% Virtualization  
【编写语言】: VC++  
【操作平台】: XP SP2  
【软件介绍】: Compressed Resources/Code/Data/Maximum ComPression  
【作者声明】: 闲得无聊。凑热闹~^\_^  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、序言  
  昨天找大家测试了一翻，初步结果是部分输入表没有修复的文件只能在我的同平台上运行，修复了输入表的找朋友把 SP1、  
  SP2、2000、2003 上都测试了个遍，没发现问题。至于如果在你那有问题，还望能够提供点实质性的帮助信息给我:)  
  至于为什么放个部分输入表没修复的上来，是为了证实我的某些想法:)  
  
  关于 ExeCryptor 的反 OD 能力的确是厉害，至于有多厉害，前人已有述说，我就不在这里累述。当然，也有不少破解之道，  
  有兴趣的倒可以去好好拜读下 loveboom 的“穿透 eXeCryptor 2.2x 保护体系(年终篇)”，的确经典，我也不多说。我这  
  里介绍另外一种法子。使用 NtGlobalFlag(OD 插件)可以让它停在 EP 处，另外需要 HideOD，IsDebug，Hidedbg 插件。  
   

引用:    
  我在配置 OD 的时候发现 NtGlobalFlag 与某些 OD 有冲突，导致 OD 根本就没法正常工作，不知道大家那里情况如何。为此，  
  我专门配置了一个脱 ExeCryptor，大家可以多找几个 OD 试下。  
  

  
  
  二、代码解压，找 OEP  
    1、打开 OD  
    2、忽略所有异常  
    3、插件--NtGlobalFlag 选下 Set TempBreak on Tls Callback，SET LDR\_SHOW\_SNAPS  
    4、隐藏 OD  
    5、打开需要脱壳的文件  
  如果 OD 插件配置得好，隐藏得好，此时，一载入 OD 呈程序运行状态，不一会而就停在了 Tls Callback 处  
  

引用:    
  00526918    E8 EBFEFFFF     CALL UnPackMe.00526808                   ;EP  
  0052691D    05 E55E0000     ADD EAX,5EE5  
  00526922    FFE0            JMP EAX  
  00526924    E8 04000000     CALL UnPackMe.0052692D                   ; TLS CallBack  
  00526929    FFFF            ???                                      ; Unknown command  
  0052692B    FFFF            ???                                      ; Unknown command  
  0052692D    5E              POP ESI  
  0052692E    C3              RETN  
  0052692F    0000            ADD BYTE PTR DS:\[EAX\],AL  
  00526931    0000            ADD BYTE PTR DS:\[EAX\],AL  
  

  
  
  Alt+B,会发现有 2 个断点  

引用:    
  Address    Module     Active                     Disassembly                           Comment  
  0052690C   UnPackMe   One-shot                   CALL UnPackMe.00526808  
  7C92EBAC   ntdll      When STRING \[\[\[esp+4\]+18h  PUSH EBP  
  

  
  都删掉吧  
  
  接下来，Alt+M，在 00401000 段 F2，Shift+F9 中断下来  

引用:    
  00519E01    AA              STOS BYTE PTR ES:\[EDI\]                  ；中断在此  
  00519E02  ^ EB E9           JMP SHORT UnPackMe.00519DED  
  00519E04    E8 FC000000     CALL UnPackMe.00519F05  
  00519E09    0F82 97000000   JB UnPackMe.00519EA6  
  

  
  Ctr+F：CMP ECX,2  

引用:    
  00519E91   /EB 5E           JMP SHORT UnPackMe.00519EF1  
  00519E93   |83F9 02         CMP ECX,2                                ；找到这里，F2  
  00519E96   |74 66           JE SHORT UnPackMe.00519EFE  
  00519E98   |41              INC ECX  
  

  
  这里呢，当 ECX=2 的时候解码。Shift+F9 运行，中断 10 次(注意观察 ECX 的值变化，第 10 次 ECX=2，第 11 次程序运行)  
  OK，取消断点(可以在此处 Dump 了，但是，为了后面的 Stolen Code 还是到后面 Dump 吧)  
  
  Alt+M，在 00401000 段 F2，Shift+F9，如果一切顺利会中断下来  

引用:    
  004271AE    90              NOP  
  004271AF    90              NOP  
  004271B0  - E9 6CD10500     JMP UnPackMe.00484321  
  004271B5    58              POP EAX  
  004271B6    C1C0 18         ROL EAX,18  
  004271B9  - E9 424E0400     JMP UnPackMe.0046C000  
  004271BE    70 12           JO SHORT UnPackMe.004271D2  
  004271C0    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O command  
  004271C1    90              NOP  
  004271C2    321F            XOR BL,BYTE PTR DS:\[EDI\]  
  004271C4    ED              IN EAX,DX                                ; I/O command  
  004271C5    50              PUSH EAX                                 ; 中断在这里  
  004271C6    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  004271CD    83C4 A8         ADD ESP,-58  
  004271D0    53              PUSH EBX  
  004271D1    56              PUSH ESI  
  004271D2    57              PUSH EDI  
  004271D3    8965 E8         MOV DWORD PTR SS:\[EBP-18\],ESP  
  004271D6    FF15 DC0A4600   CALL DWORD PTR DS:\[460ADC\]               ; UnPackMe.00475EF8  
  004271DC    33D2            XOR EDX,EDX  
  004271DE    8AD4            MOV DL,AH  
  004271E0    8915 34E64500   MOV DWORD PTR DS:\[45E634\],EDX  
  004271E6    8BC8            MOV ECX,EAX  
  004271E8    81E1 FF000000   AND ECX,0FF  
  004271EE    890D 30E64500   MOV DWORD PTR DS:\[45E630\],ECX  
  004271F4    C1E1 08         SHL ECX,8  
  004271F7    03CA            ADD ECX,EDX  
  004271F9    890D 2CE64500   MOV DWORD PTR DS:\[45E62C\],ECX  
  004271FF    C1E8 10         SHR EAX,10  
  

  
  
  三、找 Stolen Code  
  
  大致一看，就知道是 VC++的程序了  
  现在需要还原 Stolen Code 了  
  注意看堆栈  

引用:    
  0012FFB4   004292C8  UnPackMe.004292C8       ;push 004292C8  
  0012FFB8   00450E60  UnPackMe.00450E60       ;push 00450E60  
  0012FFBC   FFFFFFFF                          ;push -1(FFFFFFFF)  
  0012FFC0   0012FFF0   
  

  
  再结合 VC 入口特征(封装)即可得到 Stolen Code  
  

引用:    
  004271B0 U>  55                push ebp  
  004271B1     8BEC              mov ebp,esp  
  004271B3     6A FF             push -1  
  004271B5     68 600E4500       push UnPackED.00450E60  
  004271BA     68 C8924200       push UnPackED.004292C8  
  004271BF     64:A1 00000000    mov eax,dword ptr fs:\[0\]  
    
  55 8B EC 6A FF 68 60 0E 45 00 68 C8 92 42 00 64 A1 00 00 00 00  
  

  
  另外还有种方法，可以 Enter 进  
  004271B0  - E9 6CD10500     JMP UnPackMe.00484321  
  004271B9  - E9 424E0400     JMP UnPackMe.0046C000  
  再根据 VC 入口特征找到 Stolen Code  
  
  好了，恢复 Stolen Code 之后，在 OEP=004271B0 处右键新建 EIP，至此可以 Dump 了！  
  
  四、修复 IAT  
  
  小插曲：我发现用 OllyDump.dll 插件，以 Method1，勾选 Rebulid Import 之后的脱壳文件竟然可以在本机同样平台上运行，  
  在其他平台上就跑不起来了，这个已经得到各位帮忙测试的朋友的证实。  
  关于 IAT 的修复，前面有大侠介绍脚本、补代码等修复方法，只是本人比较懒了。  
  可以使用以下懒方法修复  
  
    单独运行加壳程序，OEP 既然知道了，IAT Start&IAT End 就很容易知道了，填入 OEP RVA Size，直接点获取输入表  
  法 1、等级 3 修复  
  法 2、使用 ImportREC 的 ExeCryptor 插件  
  
  
  注意：我找了几个 ExeCryptor2.2.50 加壳的程序(包括 loveboom 的“穿透 eXeCryptor 2.2x 保护体系(年终篇)”中的例子)，  
  使用这 2 个方法屡试不爽。但是，值得注意的是用等级 3 修复会有个别 kernel32 范围内的指针无法修复，或者是修复不准确，  
  根据经验判断此指针应该(不是一定)是 GetModuleHandleA。  
  
  五、纠正 Tls  
  
  获取了全部 IAT 之后，Fix Dump!  
  运行程序，既然出现了两个同样的窗口，偶尔几次运行会出现提示文件冲突！  
  这就是 Tls Callback 的原因了  
  
  使用 LoadPE 修正以下 3 个值为：  
  Base Of Code=1000  
  Tls Directory RVA=00000000  
  Tls Directory SIZE=00000000  
  
  再次运行程序发现一切正常  
  
  Alt+M  
  
  双击 0046B000 区段，发现  
  

引用:    
  0046B000  CC B0 06 00 00 00 00 00 FF FF FF FF 3C B0 06 00  贪.....����<?.  
  0046B010  B4 B0 06 00 08 B1 06 00 00 00 00 00 FF FF FF FF  窗.?.....����  
  0046B020  E8 B0 06 00 04 B1 06 00 00 00 00 00 00 00 00 00  璋.?.........  
  0046B030  00 00 00 00 00 00 00 00 00 00 00 00 6B 65 72 6E  ............kern  
  0046B040  65 6C 33 32 2E 64 6C 6C 00 00 00 00 00 00 47 65  el32.dll......Ge  
  0046B050  74 4D 6F 64 75 6C 65 48 61 6E 64 6C 65 41 00 00  tModuleHandleA..  
  0046B060  00 00 4C 6F 61 64 4C 69 62 72 61 72 79 41 00 00  ..LoadLibraryA..  
  0046B070  00 00 47 65 74 50 72 6F 63 41 64 64 72 65 73 73  ..GetProcAddress  
  0046B080  00 00 00 00 00 00 45 78 69 74 50 72 6F 63 65 73  ......ExitProces  
  0046B090  73 00 00 00 00 00 56 69 72 74 75 61 6C 41 6C 6C  s.....VirtualAll  
  0046B0A0  6F 63 00 00 00 00 56 69 72 74 75 61 6C 46 72 65  oc....VirtualFre  
  0046B0B0  65 00 00 00 29 B5 80 7C C4 2F 88 7C 28 AC 80 7C  e...)祤|?坾(瑎|  
  0046B0C0  A2 CA 81 7C 81 9A 80 7C 14 9B 80 7C 4C B0 06 00  ⑹亅仛€|泙|L?.  
  0046B0D0  60 B0 06 00 70 B0 06 00 84 B0 06 00 94 B0 06 00  \`?.p?.劙.敯.  
  0046B0E0  A4 B0 06 00 00 00 00 00 75 73 65 72 33 32 2E 64  ぐ.....user32.d  
  0046B0F0  6C 6C 00 00 00 00 4D 65 73 73 61 67 65 42 6F 78  ll....MessageBox  
  0046B100  41 00 00 00 0B 05 D5 77 F4 B0 06 00 00 00 00 00  A... 誻舭.....  
  

  
  这些函数都有了，不需要我们手工添加了，手工！  
  
  使用此法，可以很快搞定 loveboom 的“穿透 eXeCryptor 2.2x 保护体系(年终篇)”中的例子，  
  关于哪个例子，南蛮妈妈已经写有文章，只是 OEP 这个地方纠正得不太妥当，他是在程序中找空闲写 Stolen Code 的  
  完全可以遵照 VB 入口情况在原 OEP 处修复！  
  
  
\--------------------------------------------------------------------------------  
【经验总结】  
  1、大家也可以使用 loveboom 的“穿透 eXeCryptor 2.2x 保护体系(年终篇)”中介绍的方法来防止反 OD  
  2、脱这个完全是稀里糊涂的 oοО  
  3、再此请教下关于 DarkBull 在他的文章之中提到的修复已初始数据的原理  
  4、以上需要的插件什么的。。大家自行搜索下载吧  
  5、感觉这一个 Stolen Code 不强，大家自行练习其他软件的时候，自行解决遇到的问题  
  6、此发适合 2.2.50 至于其他版本在下不知，2.2.50 以下的版本还要简单  
  7、文中有何错误望大家指出，谢谢  
  8、经过测试此法基本适用于 2.2.60，当然具体问题得具体分析了  
  9、Thanks All  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 08 月 09 日 11:04:52  
  

 4dd466d0e1e0cc123afbe504da2462ca