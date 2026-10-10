

# PicMaster V2.5

> 原文：[`www.pediy.com/kssd/pediy04/kanxue181.htm`](https://www.pediy.com/kssd/pediy04/kanxue181.htm)

Key File 破解之 PicMaster V2.5   
  
作者：PaulYoung　（ 属于 China Cracking Group ）   
软件：PicMaster V2.5   
简介：可以帮你快速轻松地编辑图形文件。功能相当广泛，包括简单的扫描、使用滤镜、把图形文件放到大型海报目录之中、抓取影音画面、网络摄影机、幻灯片播放、3D 浮雕画面。photoshop 的滤镜都可以用在这个软件之上，你也可以记录音效指令到图片之中，让你的幻灯片播放更有真实的多媒体效果。一个看图管理工具让你可以用许多方式管理你的图形文件，网络摄影机画面也可以上传。   
下载：http://www.graphics-tools.com/   
工具：REGMonitor,W32DASM V9.00,TRW 2000,HIEW，还有……东方快车（唉，这位兄台，你可别倒……因为我不会英文，没它看不懂呀？:) ）   
日期：2001.10.26   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
　　最近玩破解实在出不了什么好作品，水平太低了，没办法，只好拿这个 PicMaster V2.5 的爆破来充充数，望各位不要见笑。   
  
    请先看软件的说明书：   
As a registered User you have the following advantages:   
  - No Shareware reminders any more   
  - No disturbing while playing the slideshow   
  - No length limit for recording sound comments   
  - No length limit for number of images in the catalog   
  - No "unregistered version" text on your printed paper...   
How does registration work?   
  You only need a serial number, which you have to enter at the beginning so   
  that your program becomes the FULL-Version.   
  
    看不懂？象我一样用东方快车吧，嘻……嘻……反正是功能限制多多。   
  
　　好了，开始步入正题。运行时，会出现一个提示输入注册码的对话框，可惜我跟了好久，跟不出来……只好另想办法。哇，你可以？告诉我！！！   
　　用 RegMonitor 跟踪，可以发现它启动时读取一个"picmaster.key"的文件，再用 W32DASM 反编译，可以找到以下的东东。呵……注意了，用 W32DASM 反汇编出的字符串是 Standard 的，这个软件是 English、Standard 双语版，可在"OPTION"处设置语言，但用 W32DASM 反出的字符串肯定是 Standard 。还有，如果"PicMaster.exe"改了名，菜单会变成 Standard 的，所以不要将它改名，除非你会看 Standard 。   
  
　　先建一个"picmaster.key"文件，放在安装目录处。再运行，可以发现，它不再提示你输入注册码，但提示"The serial number is invalid"，接着显示一个"Shareware Info"，并开始 10 秒倒计时，我烦……   
  
　　★第一步，去掉提示输入注册码的 NAG 及"The serial number is invalid"等提示。   
  
　　用 TRW 2000 设断、跟踪……Let's go!My dear Cracker!   
  
  
  
\* Possible StringData Ref from Code Obj ->"picmaster.key"   
                                  |   
:005B8586 BA14875B00              mov edx, 005B8714   
:005B858B E820BCE4FF              call 004041B0      //在此设断   
:005B8590 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:005B8593 E89C1BE5FF              call 0040A134      //安装目录是否有"picmaster.key"   
:005B8598 84C0                    test al, al   
:005B859A 755C                    jne 005B85F8      //有则跳   
:005B859C 803D90395C0000          cmp byte ptr \[005C3990\], 00   
:005B85A3 7453                    je 005B85F8   
:005B85A5 8BCE                    mov ecx, esi   
:005B85A7 B201                    mov dl, 01   
  
\* Possible StringData Ref from Code Obj ->"ㄕC"   
                                  |   
:005B85A9 A1B4595B00              mov eax, dword ptr \[005B59B4\]   
:005B85AE E8C968E9FF              call 0044EE7C   
:005B85B3 8B15E43D5C00            mov edx, dword ptr \[005C3DE4\]   
:005B85B9 8902                    mov dword ptr \[edx\], eax   
:005B85BB A1E43D5C00              mov eax, dword ptr \[005C3DE4\]   
:005B85C0 8B00                    mov eax, dword ptr \[eax\]   
:005B85C2 8B10                    mov edx, dword ptr \[eax\]   
:005B85C4 FF92D8000000            call dword ptr \[edx+000000D8\]  //没有"picmaster.key"，则提示输入注册码   
:005B85CA 8D55F8                  lea edx, dword ptr \[ebp\-08\]   
:005B85CD A1E43D5C00              mov eax, dword ptr \[005C3DE4\]   
:005B85D2 8B00                    mov eax, dword ptr \[eax\]   
:005B85D4 8B80DC020000            mov eax, dword ptr \[eax+000002DC\]   
:005B85DA E8B5E1E7FF              call 00436794   
:005B85DF 8B55F8                  mov edx, dword ptr \[ebp\-08\]   
:005B85E2 B898A55C00              mov eax, 005CA598   
:005B85E7 E890B9E4FF              call 00403F7C   
:005B85EC A1E43D5C00              mov eax, dword ptr \[005C3DE4\]   
:005B85F1 8B00                    mov eax, dword ptr \[eax\]   
:005B85F3 E8ACABE4FF              call 004031A4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:005B859A(C), :005B85A3(C)   
|   
:005B85F8 8D45F4                  lea eax, dword ptr \[ebp\-0C\]   
:005B85FB E838050000              call 005B8B38   
:005B8600 8D45F4                  lea eax, dword ptr \[ebp\-0C\]   
  
\* Possible StringData Ref from Code Obj ->"picmaster.key"   
                                  |   
:005B8603 BA14875B00              mov edx, 005B8714   
:005B8608 E8A3BBE4FF              call 004041B0   
:005B860D 8B45F4                  mov eax, dword ptr \[ebp\-0C\]   
:005B8610 E81F1BE5FF              call 0040A134        //再次验证是否有"picmaster.key"   
:005B8615 84C0                    test al, al   
:005B8617 0F84BB000000            je 005B86D8   
:005B861D B8C4A55C00              mov eax, 005CA5C4   
:005B8622 E801B9E4FF              call 00403F28   
:005B8627 A1C4A55C00              mov eax, dword ptr \[005CA5C4\]   
  
\* Possible StringData Ref from Code Obj ->"7437-4578-4868-3487"   
                                  |   
:005B862C BA2C875B00              mov edx, 005B872C   
:005B8631 E882BCE4FF              call 004042B8   
:005B8636 7514                    jne 005B864C   
:005B8638 B8C4A55C00              mov eax, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"Vielen Dank f 黵 die Registrierung"   
                                  |   
:005B863D BA48875B00              mov edx, 005B8748   
:005B8642 E835B9E4FF              call 00403F7C   
:005B8647 BB03000000              mov ebx, 00000003   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B8636(C)   
|   
:005B864C 83FB04                  cmp ebx, 00000004   
:005B864F 750F                    jne 005B8660   
:005B8651 B8C4A55C00              mov eax, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"Thank you for registration"   
                                  |   
:005B8656 BA74875B00              mov edx, 005B8774   
:005B865B E81CB9E4FF              call 00403F7C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B864F(C)   
|   
:005B8660 833DC4A55C0000          cmp dword ptr \[005CA5C4\], 00000000   
:005B8667 750F                    jne 005B8678      //一定要跳，不跳则提示"The serial number is invalid"，改为  EB0F ①   
:005B8669 B8C4A55C00              mov eax, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"Die Seriennummer ist ung 黮 tig"   
                                  |   
:005B866E BA98875B00              mov edx, 005B8798   
:005B8673 E804B9E4FF              call 00403F7C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B8667(C)   
|   
:005B8678 83FB05                  cmp ebx, 00000005   
:005B867B 750F                    jne 005B868C      //一定要跳，不跳则提示"The serial number is invalid"，改为  EB0F　②   
:005B867D B8C4A55C00              mov eax, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"The serial number is invalid"   
                                  |   
:005B8682 BAC0875B00              mov edx, 005B87C0   
:005B8687 E8F0B8E4FF              call 00403F7C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B867B(C)   
|   
:005B868C 83FB06                  cmp ebx, 00000006   
:005B868F 7411                    je 005B86A2      //一定要跳，不跳则提示"The serial number is invalid"  ，改为 EB11　③   
:005B8691 BAC4A55C00              mov edx, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"蹹\\"   
                                  |   
:005B8696 B8145E5B00              mov eax, 005B5E14   
:005B869B E86CDFE4FF              call 0040660C   
:005B86A0 EB0F                    jmp 005B86B1   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B868F(C)   
|   
:005B86A2 BAC4A55C00              mov edx, 005CA5C4   
  
\* Possible StringData Ref from Code Obj ->"蹹\\"   
                                  |   
:005B86A7 B81C5E5B00              mov eax, 005B5E1C   
:005B86AC E85BDFE4FF              call 0040660C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B86A0(U)   
|   
:005B86B1 A1C4A55C00              mov eax, dword ptr \[005CA5C4\]   
:005B86B6 E8795AEAFF              call 0045E134 //提示"Registered to %s %s"，NOP 掉它　④   
:005B86BB 83FB4D                  cmp ebx, 0000004D   
:005B86BE 750D                    jne 005B86CD   
  
.   
.   
.   
  
:0059EC32 E8A99B0100              call 005B87E0        //弹出 NAG   
:0059EC37 8B8658060000            mov eax, dword ptr \[esi+00000658\]   
:0059EC3D 8B808C000000            mov eax, dword ptr \[eax+0000008C\]   
  
\* Possible StringData Ref from Code Obj ->"NoWelcome"  （不欢迎我 :~( ）   
  
  
　　这样修改上面 4 处之后，运行软件，不再提示输入注册码了，但运行时那个大大而丑陋的 NAG 还有向你奸笑，还有 10 秒倒计时（总使我想起世纪交替之际的倒计时 :) ），非常非常的讨厌。进入后标题栏还显示为"PicMaster V2.5 - Shareware Version"，功能限制当然还没有去除。   
  
　　★第二步，瞄准目标，一矢中的。   
  
　　用 W32DASM 的“字符串数据参考”，找到"PicMaster V2.5 - Shareware Version"   
  
  
:005B8A1D 803D70395C0000          cmp byte ptr \[005C3970\], 00    //经典吧，一看就知道\[005C3970\]这个变量存放的就是注册标志了，   
:005B8A24 754F                    jne 005B8A75   
  
.   
.   
.   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:005B8A24(C)　　　　　　　　　　　　　　　　　　　　　　　　　　　　　//找这个地址   
|   
:005B8A75 A1383F5C00              mov eax, dword ptr \[005C3F38\]   
:005B8A7A 8B00                    mov eax, dword ptr \[eax\]   
  
\* Possible StringData Ref from Code Obj ->"PicMaster V2.5 - Shareware Version"   
                                  |   
:005B8A7C BA148B5B00              mov edx, 005B8B14   
:005B8A81 E83EDDE7FF              call 004367C4   
  
  
　　用 TRW 2000 设断：bpx 005B8A1D ，中断后，下 E 5C3970 1, F5 退出，我倒……NAG 还在奸笑，倒计时还在继续，标题栏还是"PicMaster V2.5 - Shareware Version"。气昏……再来，这次下 E 5C3970 0, F5 退出，再看，嘻……这回天下太平了，NAG 消失，标题栏看到是 "Registered to PicMaster V2.5"。   
    明白了吗？这就是说，\[005C3970\]这个变量的值为 0 是注册版，非 0 就不是了注册版了。   
　　我这人比较懒，看上面的说明书可以知道，它的功能限制可不少，一个一个地去除，太麻烦了，更何况我根本不会用，去得不完全，会被人骂的，嘻……我要一步到位，找到它在哪里把 \[005C3970\] 赋值为 1 的。   
　　清除所有断点后，退出软件。在 TRW 2000 下断：BPM 5C3970 W ，再运行软件，中断到……   
  
:005B9764 A198A55C00              mov eax, dword ptr \[005CA598\]   
:005B9769 E84A64F9FF              call 0054FBB8   
:005B976E 3401                    xor al, 01　　　　//异或运算，结果 AL=1  ⑤   
:005B9770 A270395C00              mov byte ptr \[005C3970\], al    //把 AL，也就是 1 赋值给\[005C3970\]这个变量，就是未注册版了。   
:005B9775 33C0                    xor eax, eax      //你在这里呢   
  
　　知道怎么改了吧……呵……就是把   
  
:005B976E 3401                    xor al, 01  改为 xor eax,eax（机器码为 33C0） 或 xor al,al（机器码为 32C0），那么，AL 的值为 0，也就是 \[005C3970\]的值为 0 ，也就是变成注册版了！也就是你成功了！也就是……&\*^$%#@@\_)+)\*$##……   
    （真罗嗦，弟兄们，操家伙，扁他……）   
  
　　哇……我跑！！！！！！   
  
\--- THE END --- 

Comparing files PICMAS~1.EXE   
001B799A: E9 75   
001B799B: 1C 5C   
001B799C: 01 80   
001B799D: 00 3D   
001B799E: 00 90   
001B8B6E: 33 34   
001B8B6F: C0 01  4dd466d0e1e0cc123afbe504da2462ca