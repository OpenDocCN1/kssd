

# CuteFTP V4.2.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue066.htm`](https://www.pediy.com/kssd/pediy04/kanxue066.htm)

CuteFTP 最新版 V4.2.4 在线注册的破解   
  
破解者：moonlite\[BCG\]\[FCG\]   
目标： CuteFTP 最新版 V4.2.4   
应用平台：Win9X/ME/WinNT/2K   
下载：http://www.globalscape.com/   
大小：1694k   
软件用途： 当然是最 cool 的 FTP 客户端软件了，不用再多说了吧。   
                      
工具：TRW1.22，W32dasm, filemon, regmon, Winhex   
保护： 每次启动都弹出注册窗，提示上网注册; 30 天试用期；动态 CRC 校验。   
  
【前言】： xy2000\[BCG\]老兄推荐的软件，就拿它练练手吧. 我很喜欢这个软件的原因有三：   
      ㈠. 没加壳；㈡. 没有反调试；   
      ㈢. CRC 的出错信息中体现了对 crack 们的尊重：请看   
  
┼————————————————————————————————   
CuteFTP consistency check failed. This means that you are probably using a corrupted version. This   
may caused by a virus. Please, do a virus scan, reinstall CuteFTP and try to start it again.   
                          ——————————————————————————————————┼   
  
它不象有些软件，你一调试，它就说 "Hmm...Debug yourself".   
  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　===>好，开始工作吧!◆   
  
  
★(第一部分)找注册码   
  
1)启动 cutftp32.exe,提示在线注册 nag 窗口弹出。分析它的注册信息一定存放在注册表中，或有 keyfile 保护。   
  
2)分别启动 filemon 和 regmon 分析：   
  
发现以下可疑点→   
  
AUTONAME.DAT, COMMANDS.DAT-------->调用到的文件   
  
  
QueryValueEx    HKLM\\Software\\GlobalSCAPE Inc.\\CuteFTP\\Key2                    NOTFOUND           
  
QueryValueEx    HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\ProductId    SUCCESS    "80123-026-6304672-53376"       
CloseKey    HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion                    SUCCESS           
OpenKey    HKCR\\Rl                                                            NOTFOUND    ※※※※       
OpenKey    HKLM\\Software\\GlobalSCAPE Inc.\\CuteFTP    SUCCESS    hKey: 0xC2A0E050       
QueryValueEx    HKLM\\Software\\GlobalSCAPE Inc.\\CuteFTP\\Key1                    NOTFOUND       
  
  
3)我尝试了建 Key1 和 Key2 两个键值，没有发现效果。就在 HKCR\\下建了个 Rl\\1, 随便输入字符串  "23232323232323".   
  
　\[HKEY\_LOCAL\_MACHINE\\Software\\GlobalSCAPE Inc.\\CuteFTP\]下建"RegUserName"="moonLite\[BCG\]"   
  
  
4)再次运行 cutftp32.exe，在线注册窗口弹出。唤出 TRW，点击按钮 "Contiue Trial" 并 Ctrl+D 激活 TRW。程序来到--->   
  
\* Reference To: USER32.GetMessageA, Ord:012Ah   
                                  |   
:004DD7E4 FF1594D75100            Call dword ptr \[0051D794\]   
:004DD7EA 85C0                    test eax, eax<-------------------------光标在这！   
:004DD7EC 7426                    je 004DD814   
:004DD7EE 817E346A030000          cmp dword ptr \[esi+34\], 0000036A   
:004DD7F5 741A                    je 004DD811   
:004DD7F7 8B06                    mov eax, dword ptr \[esi\]   
:004DD7F9 57                      push edi   
:004DD7FA 8BCE                    mov ecx, esi   
:004DD7FC FF5058                  call \[eax+58\]   
:004DD7FF 85C0                    test eax, ea   
:004DD801 750E                    jne 004DD811   
:004DD803 57                      push edi   
  
  
开始按 F12+F10， 记录下来可疑的跳转：4D8249,43B873.   
  
  
:0043B849 68F4235500              push 005523F4   
:0043B84E 8BCB                    mov ecx, ebx   
:0043B850 E82CB70B00              call 004F6F81   
:0043B855 8983DC000000            mov dword ptr \[ebx+000000DC\], eax   
:0043B85B B801000000              mov eax, 00000001   
:0043B860 898344060000            mov dword ptr \[ebx+00000644\], eax   
:0043B866 898380060000            mov dword ptr \[ebx+00000680\], eax   
:0043B86C E8FF4F0500              call 00490870------------------------->进入   
:0043B871 85C0                    test eax, eax\-------------------------|这里，让 eax=1 可以跳过 nag！   
:0043B873 753D                    jne 0043B8B2   
:0043B875 33F6                    xor esi, esi   
:0043B877 8BCB                    mov ecx, ebx   
:0043B879 56                      push esi   
  
\* Possible StringData Ref from Data Obj ->"TSUninstaller"   
                                  |   
:0043B87A 68DC465500              push 005546DC   
  
\* Possible StringData Ref from Data Obj ->"CtFPRgsraeoe"   
                                  |   
:0043B87F 68F4235500              push 005523F4   
:0043B884 E85B890A00              call 004E41E4   
:0043B889 89B380060000            mov dword ptr \[ebx+00000680\], esi   
:0043B88F 6A01                    push 00000001   
:0043B891 8BCB                    mov ecx, ebx   
:0043B893 89B388060000            mov dword ptr \[ebx+00000688\], esi   
:0043B899 E812130000              call 0043CBB0   
:0043B89E 8BCB                    mov ecx, ebx   
:0043B8A0 E87B0A0000              call 0043C320-------------------------|在线注册窗口   
:0043B8A5 85C0                    test eax, eax   
:0043B8A7 751E                    jne 0043B8C7   
:0043B8A9 56                      push esi   
  
可见，0043B86C 的 CALL 有问题，得进去看看！   
  
  
5)   
  
\* Referenced by a CALL at Addresses:   
|:004013FA  , :004300A8  , :004346DB  , :0043B86C  , :0044045B    
|:004459D9  , :004476A3  , :00457F8F  , :0047D15E  , :0047D8FE    
|:0048B82F  , :0048C470  , :00491F79  , :004ACB68    
|   
:00490870 64A100000000            mov eax, dword ptr fs:\[00000000\]   
  
  
\* Possible Reference to String Resource ID=00255: "No entry for the current site found. Do you wish to create o"   
                                  |   
:00490876 6AFF                    push FFFFFFFF   
:00490878 68D34F5100              push 00514FD3   
:0049087D 50                      push eax   
:0049087E B81C180000              mov eax, 0000181C   
:00490883 64892500000000          mov dword ptr fs:\[00000000\], esp   
:0049088A E801130300              call 004C1B90   
:0049088F 53                      push ebx   
:00490890 8D8424680C0000          lea eax, dword ptr \[esp+00000C68\]   
:00490897 56                      push esi   
:00490898 50                      push eax   
:00490899 E882F9FFFF              call 00490220   
:0049089E 83C404                  add esp, 00000004   
:004908A1 85C0                    test eax, eax   
:004908A3 7517                    jne 004908BC   
:004908A5 5E                      pop esi   
:004908A6 5B                      pop ebx   
:004908A7 8B8C241C180000          mov ecx, dword ptr \[esp+0000181C\]   
:004908AE 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:004908B5 81C428180000            add esp, 00001828   
:004908BB C3                      ret   
  
  
\-->不断按 F10，会来到：   
  
:004908E7 83C40C                  add esp, 0000000C   
:004908EA 85C0                    test eax, eax   
:004908EC 5F                      pop edi   
:004908ED 0F857A020000            jne 00490B6D   
:004908F3 8A84249C040000          mov al, byte ptr \[esp+0000049C\]---------------|从“23232323232323”取一个字符   
:004908FA 84C0                    test al, al   
:004908FC 0F84C1020000            je 00490BC3   
:00490902 8D8C249C040000          lea ecx, dword ptr \[esp+0000049C\]---------------|ecx 指向“23232323232323”字符串   
:00490909 8D542418                lea edx, dword ptr \[esp+18\]   
:0049090D 51                      push ecx   
:0049090E 52                      push edx   
:0049090F C7442420FFFFFF7F        mov \[esp+20\], 7FFFFFFF   
:00490917 E824690200              call 004B7240--------------->注意到紧跟的判断，得追进去   
:0049091C 83C408                  add esp, 00000008   
:0049091F 6685C0                  test ax, ax\---------------|ax 不为 0，就能成功了！   
:00490922 7519                    jne 0049093D---------------|不跳转则失败！   
:00490924 5E                      pop esi   
:00490925 33C0                    xor eax, eax\---------------|eax 为注册标志   
:00490927 5B                      pop ebx   
:00490928 8B8C241C180000          mov ecx, dword ptr \[esp+0000181C\]   
:0049092F 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:00490936 81C428180000            add esp, 00001828   
:0049093C C3                      ret   
  
\--------------------   
  
\* Referenced by a CALL at Addresses:   
|:00490917  , :00490BA2  , :004915A6    
|   
:004B7240 83EC20                  sub esp, 00000020--------------------------------------------|   
:004B7243 83C9FF                  or ecx, FFFFFFFF                                            |   
:004B7246 33C0                    xor eax, eax                                                |   
:004B7248 56                      push esi                                                    |   
:004B7249 8B74242C                mov esi, dword ptr \[esp+2C\]/指向从“23232323232323”字符串  |计算字符串长度   
:004B724D 57                      push edi                                                    |   
:004B724E 8BFE                    mov edi, esi                                                |   
:004B7250 F2                      repnz                                                        |   
:004B7251 AE                      scasb                                                        |   
:004B7252 F7D1                    not ecx                                                      |   
:004B7254 49                      dec ecx -----------------------------------------------------|   
:004B7255 83F90E                  cmp ecx, 0000000E--------------------|长度不是 14 位，就不带玩了！   
:004B7258 7573                    jne 004B72CD-------------------------|不要在此跳啊！   
:004B725A 56                      push esi   
:004B725B E863E10000              call 004C53C3   
  
............   
  
接着走到   
:004B728C C644242800              mov \[esp+28\], 00   
:004B7291 E86A20FEFF              call 00499300   
:004B7296 8D442438                lea eax, dword ptr \[esp+38\]-------------------|下 d eax 看看   
  
\* Possible Reference to String Resource ID=00014: "Paste Url"   
                                  |   
:004B729A 6A0E                    push 0000000E   
:004B729C 8D4C242C                lea ecx, dword ptr \[esp+2C\]-------------------|下 d ecx 可以看到精彩部分啊！   
  
  
\============================================================================   
  
0030:0071DAE4 41 32 32 32 32 32 32 32-32 32 32 32 32 32 00 C2 A2222222222222.?   
0030:0071DAF4 32 33 32 33 32 33 32 33-32 33 32 33 32 33 00 00 23232323232323..   
  
\============================================================================   
  
  
:004B72A0 50                      push eax   
:004B72A1 51                      push ecx   
:004B72A2 E879C90000              call 004C3C20-------------------|关键的比较部分！（不想列出了,否则篇幅太长了）   
:004B72A7 83C42C                  add esp, 0000002C   
:004B72AA 85C0                    test eax, eax\-------------------|eax\=0 就对了！eax=1,则失败   
:004B72AC 7510                    jne 004B72BE--------------------|eax\=1,则做失败跳转   
:004B72AE 8B54242C                mov edx, dword ptr \[esp+2C\]   
:004B72B2 660DFFFF                or ax, FFFF   
:004B72B6 893A                    mov dword ptr \[edx\], edi   
:004B72B8 5F                      pop edi   
:004B72B9 5E                      pop esi   
:004B72BA 83C420                  add esp, 00000020   
:004B72BD C3                      ret   
  
▲试着将\[HKEY\_CLASSES\_ROOT\\Rl\]\\1 的键值改为"A2222222222222",重新运行程序--哇!  nag 窗口没有了!!但是在 about 窗口中是   
Licensed to: UNVERIFIED - moonLite \[BCG\], 难道还要上网验证吗？   
  
  
5)果然，上网后，启动程序后，自动与它的服务器连接并验证，返回 “moonLite\[BCG\] & A2222222222222” not accepted....真厉害!   
  ——>看来只有爆破了。   
  
〓 待续 〓 

〓 CuteFTP 最新版 V4.2.4 在线注册的破解-下篇〓   
  
★(第二部分)爆破   
  
  
6) 在 HKCR\\下再建个 Rl\\3 (为什么呢，用 regmon 查查便知), 随便输入字符串  "123456789".在 4B72AA 下断点，bpx 4B72AA,运行   
cutftp32.exe，中断后接着走到：   
  
:00490C29 FF15B4D25100            Call dword ptr \[0051D2B4\]   
:00490C2F 50                      push eax   
  
\* Reference To: KERNEL32.LockResource, Ord:01D5h   
                                  |   
:00490C30 FF15B8D25100            Call dword ptr \[0051D2B8\]   
:00490C36 8D8C2484080000          lea ecx, dword ptr \[esp+00000884\]--------------->ecx 指向字符串“123456789”   
:00490C3D 50                      push eax   
:00490C3E 51                      push ecx   
:00490C3F 8D94245C100000          lea edx, dword ptr \[esp+0000105C\]   
:00490C46 56                      push esi   
:00490C47 52                      push edx   
:00490C48 E8C32E0100              call 004A3B10---------------|重要的 CALL，进入-->   
:00490C4D 83C410                  add esp, 00000010   
:00490C50 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
:00490C54 8BF0                    mov esi, eax\------------------------------------┐   
:00490C56 C784242C180000FFFFFFFF  mov dword ptr \[esp+0000182C\], FFFFFFFF          │   
:00490C61 E88AB90400              call 004DC5F0                                  │eax=esi\=1 就对了！   
:00490C66 8B8C2424180000          mov ecx, dword ptr \[esp+00001824\]              │   
:00490C6D 8BC6                    mov eax, esi\------------------------------------┘   
:00490C6F 5E                      pop esi   
:00490C70 5B                      pop ebx   
:00490C71 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:00490C78 81C428180000            add esp, 00001828   
:00490C7E C3                      ret   
  
  
  
\* Referenced by a CALL at Address:   
|:00490C48    
|   
  
\* Possible Reference to String Resource ID=00255: "No entry for the current site found. Do you wish to create o"   
                                  |   
:004A3B10 6AFF                    push FFFFFFFF   
:004A3B12 682D6A5100              push 00516A2D   
:004A3B17 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:004A3B1D 50                      push eax   
:004A3B1E 64892500000000          mov dword ptr fs:\[00000000\], esp   
:004A3B25 81ECB0000000            sub esp, 000000B0   
:004A3B2B 56                      push esi   
:004A3B2C 57                      push edi   
:004A3B2D 6A00                    push 00000000   
:004A3B2F 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
:004A3B33 E8982A0100              call 004B65D0   
:004A3B38 6A00                    push 00000000   
  
...............   
  
\* Possible Reference to String Resource ID=00008: "Toolbar changes will take effect after CuteFTP is restarted"   
                                  |   
:004A3C1A B908000000              mov ecx, 00000008—————————————>置循环次数   
:004A3C1F 8D7C2430                lea edi, dword ptr \[esp+30\]————————>指向一个不变的十六进制串 (值得研究)   
:004A3C23 8D742470                lea esi, dword ptr \[esp+70\]————————>指向的十六进制串与输入字符串  "123456789"相关   
:004A3C27 33C0                    xor eax, eax   
:004A3C29 F3                      repz   
:004A3C2A A7                      cmpsd\-------------------|DWORD 值循环比较！   
:004A3C2B 5F                      pop edi   
:004A3C2C 5E                      pop esi   
:004A3C2D 745E                    je 004A3C8D-------------------|在此一定得跳呵，这可是最后的机会了--->修改①★   
:004A3C2F 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:004A3C33 C68424B800000005        mov byte ptr \[esp+000000B8\], 05   
:004A3C3B E8402A0100              call 004B6680   
  
7) 用 Winhex 把上面的第一处改为 745E-->EB5E.重新运行程序，CRC 报错 “CuteFTP consistency check failed...”   
好！ 打开我们的利器 W32dasm, 反汇编 cutftp32.exe 后，search 文本“check failed", 来到-->   
  
  
\* Possible Reference to Dialog: DialogID\_0181   
                                  |   
:0043BC50 6881010000              push 00000181   
:0043BC55 6A00                    push 00000000   
:0043BC57 6A00                    push 00000000   
:0043BC59 8B481C                  mov ecx, dword ptr \[eax+1C\]   
:0043BC5C 51                      push ecx   
  
\* Reference To: USER32.RedrawWindow, Ord:01F1h   
                                  |   
:0043BC5D FF1544D65100            Call dword ptr \[0051D644\]   
:0043BC63 C7835006000001000000    mov dword ptr \[ebx+00000650\], 00000001   
:0043BC6D E81E510500              call 00490D90—————————————>进入-------------------------------------->|   
:0043BC72 85C0                    test eax, eax—————————————>eax≠0，即可跳过 CRC 错误!!                  | 　   
:0043BC74 756F                    jne 0043BCE5                                                                    |          
:0043BC76 8D55EC                  lea edx, dword ptr \[ebp\-14\]                                                      |   
                                                                                                                  |   
\* Possible Reference to String Resource ID=00426: "CuteFTP consistency check failed. This means that you are pr"  |   
                                  |                                                                                |   
:0043BC79 68AA010000              push 000001AA                                                                    |   
:0043BC7E 52                      push edx                                                                        |   
:0043BC7F E8CC97FEFF              call 00425450                                                                    |   
:0043BC84 83C408                  add esp, 00000008                                                                |   
:0043BC87 8B00                    mov eax, dword ptr \[eax\]                                                        |   
:0043BC89 6A00                    push 00000000                                                                    |   
                                                                                                                  |   
\* Possible Reference to String Resource ID=00016: "CuteFTP 4.0"                                                    |   
                                  |                                                                                |                                                                                                                            |   
:0043BC8B 6A10                    push 00000010                                                                    |   
:0043BC8D 50                      push eax                                                                        |   
:0043BC8E C645FC2A                mov \[ebp\-04\], 2A                                                                |   
:0043BC92 E8ECC70A00              call 004E8483—————————————> CRC 出错的 CALL!                          |   
                                                                                                                  |   
\----------------------------------------                                                                          |   
                                                                                                                  |   
\* Referenced by a CALL at Addresses:                                                                              |   
|:0043BC6D  , :004650FB                                                                                          |   
|                                                                                                                  |   
                                                                                                                  |   
\* Possible Reference to String Resource ID=00255: "No entry for the current site found. Do you wish to create o"  |   
:                                |                                                                                |   
:00490D90 6AFF                    push FFFFFFFF <------------------------------------------------------------------|          
:00490D92 6831505100              push 00515031   
:00490D97 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:00490D9D 50                      push eax   
:00490D9E 64892500000000          mov dword ptr fs:\[00000000\], esp   
:00490DA5 81EC24010000            sub esp, 00000124   
:00490DAB 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00490DAF E863CA0400              call 004DD817   
:00490DB4 8D442420                lea eax, dword ptr \[esp+20\]   
  
\* Possible Reference to String Resource ID=00260: "Rename folder"   
                                  |   
:00490DB8 6804010000              push 00000104   
:00490DBD 50                      push eax   
:00490DBE 6A00                    push 00000000   
:00490DC0 C784243801000000000000  mov dword ptr \[esp+00000138\], 00000000   
  
\* Reference To: KERNEL32.GetModuleFileNameA, Ord:0124h   
                                  |   
:00490DCB FF1540D45100            Call dword ptr \[0051D440\]   
:00490DD1 6A00                    push 00000000   
:00490DD3 8D4C2424                lea ecx, dword ptr \[esp+24\]   
:00490DD7 6A40                    push 00000040   
:00490DD9 51                      push ecx   
:00490DDA 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00490DDE E8E6CB0400              call 004DD9C9   
:00490DE3 85C0                    test eax, eax   
:00490DE5 0F858E000000            jne 00490E79—————————>程序会跳到——->|   
:00490DEB 8D542400                lea edx, dword ptr \[esp\]                    |   
                                                                              |   
\-------------                                                                  |   
                                                                              |   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:            |   
|:00490DE5(C)                                                                  |   
|                                                                              |   
:00490E79 8D442418                lea eax, dword ptr \[esp+18\]<-----------------|   
:00490E7D 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00490E81 50                      push eax   
:00490E82 51                      push ecx   
:00490E83 E858FEFFFF              call 00490CE0   
:00490E88 83C408                  add esp, 00000008   
:00490E8B 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00490E8F E850CD0400              call 004DDBE4   
:00490E94 33C0                    xor eax, eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00490EA2(C)   
|   
:00490E96 8A4C0418                mov cl, byte ptr \[esp+eax+18\]   
:00490E9A 84C9                    test cl, cl\------->cl 是验证 CRC 出错与否的标志!  !!!修改②★★    
:00490E9C 7508                    jne 00490EA6------------------------------->在这不要跳!   
:00490E9E 40                      inc eax   
:00490E9F 83F808                  cmp eax, 00000008-------------------------->循环 8 次   
:00490EA2 7CF2                    jl 00490E96   
:00490EA4 EBA5                    jmp 00490E4B------------------------------->|   
                                                                              |   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:          |   
|:00490E9C(C)                                                                |   
|                                                                            |   
:00490EA6 8D4C2408                lea ecx, dword ptr \[esp+08\]                |              
:00490EAA C784242C010000FFFFFFFF  mov dword ptr \[esp+0000012C\], FFFFFFFF      |   
:00490EB5 E852CA0400              call 004DD90C                              |   
:00490EBA 8B8C2424010000          mov ecx, dword ptr \[esp+00000124\]          |   
:00490EC1 33C0                    xor eax, eax                                |   
:00490EC3 64890D00000000          mov dword ptr fs:\[00000000\], ecx            |   
:00490ECA 81C430010000            add esp, 00000130                          |   
:00490ED0 C3                      ret                                        |   
                                                                              |   
\---------------                                                              |   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:          |   
|:00490EA4(U)                                                                |   
|                                                                            |   
:00490E4B 8D4C2408                lea ecx, dword ptr \[esp+08\]<----------------|   
:00490E4F C784242C010000FFFFFFFF  mov dword ptr \[esp+0000012C\], FFFFFFFF   
:00490E5A E8ADCA0400              call 004DD90C   
:00490E5F B801000000              mov eax, 00000001---------------------------|这是我们想要的结果!   
:00490E64 8B8C2424010000          mov ecx, dword ptr \[esp+00000124\]   
:00490E6B 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:00490E72 81C430010000            add esp, 00000130   
:00490E78 C3                      ret\---------------------------------------->CRC OK 了!返回吧。。。   
  
\-------------------   
  
打开 Winhex 将上面的第二处 84C9  test cl, cl 改为 30C9  xor cl,cl!   
重新运行程序,CRC OK!   
查看 about 窗口，UNVERIFIED 也不见了！！在线验证 OK!   
  
  
  
###########   
最后整理：   
注册文件为(粘贴并保存为 reg 注册表文件)   
  
┄┄┄┄┄┄┄┄┄┄┄Cut Here┄┄┄┄┄┄┄┄┄┄┄   
REGEDIT4   
  
\[HKEY\_CLASSES\_ROOT\\Rl\]   
"1"="A2222222222222"   
"3"="123456789"   
  
\[HKEY\_LOCAL\_MACHINE\\Software\\GlobalSCAPE Inc.\\CuteFTP\]   
"RegUserName"="moonLite\[BCG\]"   
  
┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄   
  
再打补丁两处：   
  
                        修改①　　　　　　@offset A3C2D  745E-->EB5E   
                        修改②　　　　　　@offset 90E9A  84C9-->30C9   
  
##########   
  
  
  
【后记】：虽然注册+爆破成功了，还有很多地方值得好好研究。例如：regmon 看到的 Key1 是注册码部分，   
          那么 Key2 呢？(VERIFY？)。也难怪，这是个在线注册的软件。看一下它的注册帮助就知道了。   
          愿与朋友们交流！！   
  
                  moonlite  于 2001-10-12  4dd466d0e1e0cc123afbe504da2462ca