

# 答复

> 原文：[`www.pediy.com/kssd/pediy03/forum957.htm`](https://www.pediy.com/kssd/pediy03/forum957.htm)

超级解霸 2000 试用版的破解  
  
    平常也不怎么看 VCD，但买了电脑报 2000 年合订本，配套光盘里有超级解霸  
2000 试用版，就随手安装了，顺便练练手。  
    工具：WinIce4.05、W32DASM、Hiew6.4 和一只笔、纸若干。  
    试用版在使用 30 次后会弹出过期报错窗口，运行次数记录在 WINDOWS 文件夹  
下的 STHSVCD.INI 配置文件里，BERUN=运行次数。可以想象，每次启动 STHSVCD，  
它都先要读取 BERUN 项，和 30 比较，小于的话，则该项加 1，写入 STHSVCD.INI 并  
转入主程序执行，否则报错。那么可以下断 GetPrivateProfileStringA 或者下断  
WritePrivateProfileStringA，考虑到 STHSVCD 启动时要设置多项参数，可能多  
次调用 GetPrivateProfileStringA 可能多次被调用，不易追踪，最好利用  
WritePrivateProfileStringA 设断。  
    bpx   writeprivateprofilestringa,  
运行 STHSVCD.EXE，果然...  
按 F12  
  
    00414767   push 00426724   ->"STHVCD.INI"  
    0041476C   push eax  
    0041476D   push 00428C9C   ->"BERUN"  
    00414772   push 00426710   ->"SETTING"  
    00414777   call dword ptr \[004A0670\]  调用 WritePrivateProfileStringA  
    0041477D   cmp ebx, 0000001E  <-中断返回此处，1E 即 30  
    00414780   jle 004147A4    不超过 30 次，则跳转，否则  
    00414782   call 0040FAF0   弹出过期报错窗口  
向上看  
  
    00414732    push 00426724   ->"STHVCD.INI"  
    00414737    push 00000001  
    00414739    push 00428C9C   ->"BERUN"  
    0041473E    push 00426710   ->"SETTING"  
    00414743    Call dword ptr \[004A06F8\]  调用 GetPrivateProfileIntA  
    00414749    lea ebx,\[eax+1\]   将已使用次数加 1，送入 EBX  
    0041474C    lea eax, dword ptr \[esp+78\]  
  
Crack 方案有两个：  
1、jle 004147A4 改为 jmp 004147A4；  
2、取消 EBX 加 1 的操作，lea ebx,\[eax+1\]改为三个 NOP。  
   具体修改过程就不用我再一一罗嗦了吧。  
   慢着，别以为万事大吉，还有一个陷阱呢！STHSVCD 还在 HKEY\_LOCAL\_MACHINE\\  
Software\\Microsoft\\Windows\\CurrentVersion\\Setup 处设置键值 RunTime，保存剩  
余的运行次数，每次运行 STHSVCD，RunTime 减 1，当 RunTime 为 0 时，报错。根据已  
有的破解成果表明，将此值改为 FFFFFFFF 后，再运行 STHSVCD 该值将保持不变。确  
实这里挺难追，用 DialogBoxParamA、RegQueryValueExA 等均未成功。最后只有请  
出“依天宝剑”W32DASM，虽然反汇编时报错，但不睬它，继续，发现和读取  
注册表有关的也就一个 RegQueryValueExA，查找 RegQueryValueExA 字符串，在程序  
中发现一处调用。  
  
    0040FBF9    call dword ptr \[004A0590\]  调用 RegQueryValueExA  
    0040FBFF    test eax, eax   
    0040FC01    je 0040FC28  返回值为 0 则跳转，出错  
  
试着把 je 0040FC28 屏蔽(改为两个 NOP，或者 INC EAX，DEC EAX)，OK！无论怎么  
运行 STHSVCD，RunTime 将保持 0x63 即 99 不变，永远也不会过期了。  
    用同样的手段修改音频解霸。  
    好了，今天就说到这儿，还是让我们举起双手拥护正版吧！  
  
南阳 wxd   2001-1-5  
E-mail:wxd125@163.net  
Oicq:21331694  
   
  
  
  
  
                              再谈《超级解霸 2000 试用版的破解》  
  
\*\*\* 这是对《超级解霸 2000 试用版的破解》的修正版，介绍的较详细，很适合新手入门提高观摩 \*\*\*  
  
    前面我写了《超级解霸 2000 试用版的破解》，觉得已经成功了，但后来在解“虚拟光驱 2000”时，把  
日期向后调了一个月，无意中运行超级解霸，讨厌的过期报错窗口又跳了出来，我靠！还有一个陷阱！！  
    于是又用 W32asm 详细分析了原代码，果然是三道关卡，嘿嘿，国产软件确也费尽心机。运行试用版，  
先后要通过“BERUN”30 次运行次数的检查、30 天运行期限的检查和注册表 64 次运行次数的检查。你可以  
用“串数据参考”查找串 "The Time is over !Please Get one Gold Version !"找到过期报错窗口代码，  
如下：  
突破口  
\* Referenced by a CALL at Addresses:  
|:00414782   , :004147E4   , :0041480F     
|  
:0040FAF0 81ECD4000000            sub esp, 000000D4  
:0040FAF6 8D442454                lea eax, dword ptr \[esp+54\]  
:0040FAFA 53                      push ebx  
:0040FAFB 56                      push esi  
:0040FAFC 8B0D9C874200            mov ecx, dword ptr \[0042879C\]  
  
\* Possible Reference to Dialog: DialogID\_0080   
                                  |  
:0040FB02 6880000000              push 00000080  
  
\* Reference To: USER32.LoadStringA, Ord:01A9h  
                                  |  
:0040FB07 8B35D8084A00            mov esi, dword ptr \[004A08D8\]  
:0040FB0D C744240CD2040000        mov \[esp+0C\], 000004D2  
:0040FB15 50                      push eax  
  
\* Possible Reference to String Resource ID=50061: "The Time is over !  
Please Get one Gold   
Version !"  
                                  |  
:0040FB16 688DC30000              push 0000C38D  
:0040FB1B 51                      push ecx  
:0040FB1C FFD6                    call esi  
突破口  
  
    可以看到有三处 00414782、004147E4、0041480F 调用了这段代码，那么一切都昭然若解了。且看我沿  
着程序流程娓娓道来：  
  
突破防线  
:004146E8 8D442478                lea eax, dword ptr \[esp+78\] ；指定返回项字串的缓冲区  
  
\* Possible StringData Ref from Data Obj ->"STHVCD.INI" ；初始化文件的名字  
                                  |  
:004146EC 6824674200              push 00426724  
  
\* Possible Reference to Dialog: DialogID\_0080   
                                  |  
:004146F1 6880000000              push 00000080 ；缓冲区的最大字符数量  
:004146F6 50                      push eax ；指定返回项字串的缓冲区  
:004146F7 68D0664200              push 004266D0 ；指定的项没有找到时返回的默认值，此处为 NULL  
  
\* Possible StringData Ref from Data Obj ->"SOURCEPATH"  
                                  |  
:004146FC 68A48C4200              push 00428CA4 ；欲获取的项名  
  
\* Possible StringData Ref from Data Obj ->"INSTALL" ；欲在其中查找项的小节名称  
                                  |  
:00414701 68D0694200              push 004269D0  
  
\* Reference To: KERNEL32.GetPrivateProfileStringA, Ord:019Fh  
                                  |  
:00414706 FF1564064A00            Call dword ptr \[004A0664\]  
:0041470C 85C0                    test eax, eax  
:0041470E 7422                    je 00414732 ；复制到缓冲区的字节数量为 0，即 STHVCD.INI 无该项  
:00414710 0FBE442478              movsx eax, byte ptr \[esp+78\] ；复制缓冲区的第一个字节到 EAX  
:00414715 83F861                  cmp eax, 00000061 ；和“A”比较  
:00414718 7C05                    jl 0041471F  
:0041471A 83F87A                  cmp eax, 0000007A ；和“Z”比较  
:0041471D 7E0D                    jle 0041472C  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00414718(C)                                                ；猜测：如果从光盘安装解霸 2000，  
|                                                            ；SOURCEPATH 的第一个字符被设置为  
:0041471F 8A44247B                mov al, byte ptr \[esp+7B\]  ；小写字母，并且如果第四个字符为  
:00414723 84C0                    test al, al                ；NULL(即根目录)时，如 e: \\，则跳  
:00414725 B801000000              mov eax, 00000001          ；过 BERUN 值的检查，到日期检测处。  
:0041472A 7402                    je 0041472E                ；如果从光盘安装，第一个字符被设  
                                                             ；置为大写，需检查 BERUN 值  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041471D(C)  
|  
:0041472C 33C0                    xor eax, eax  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041472A(C)  
|  
:0041472E 85C0                    test eax, eax  
:00414730 7572                    jne 004147A4 ；跳到到日期检测处 1.1  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041470E(C)  
|  
  
\* Possible StringData Ref from Data Obj ->"STHVCD.INI"  
                                  |  
:00414732 6824674200              push 00426724  
:00414737 6A01                    push 00000001  
  
\* Possible StringData Ref from Data Obj ->"BERUN"  
                                  |  
:00414739 689C8C4200              push 00428C9C  
  
\* Possible StringData Ref from Data Obj ->"SETTING"  
                                  |  
:0041473E 6810674200              push 00426710  
  
\* Reference To: KERNEL32.GetPrivateProfileIntA, Ord:0199h  
                                  |  
:00414743 FF15F8064A00            Call dword ptr \[004A06F8\]  
:00414749 8D5801                  lea ebx, dword ptr \[eax+01\] ；将已使用次数加 1，送入 EBX  
:0041474C 8D442478                lea eax, dword ptr \[esp+78\]  
:00414750 53                      push ebx  
  
\* Possible StringData Ref from Data Obj ->"%d"  
                                  |  
:00414751 68F8664200              push 004266F8  
:00414756 50                      push eax  
  
\* Reference To: USER32.wsprintfA, Ord:029Fh  
                                  |  
:00414757 FF15E4084A00            Call dword ptr \[004A08E4\]  
:0041475D 8D842484000000          lea eax, dword ptr \[esp+00000084\]  
:00414764 83C40C                  add esp, 0000000C  
  
\* Possible StringData Ref from Data Obj ->"STHVCD.INI"  
                                  |  
:00414767 6824674200              push 00426724  
:0041476C 50                      push eax  
  
\* Possible StringData Ref from Data Obj ->"BERUN"  
                                  |  
:0041476D 689C8C4200              push 00428C9C  
  
\* Possible StringData Ref from Data Obj ->"SETTING"  
                                  |  
:00414772 6810674200              push 00426710  
  
\* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:033Bh  
                                  |  
:00414777 FF1570064A00            Call dword ptr \[004A0670\]  
:0041477D 83FB1E                  cmp ebx, 0000001E  
:00414780 7E22                    jle 004147A4 ；不超过 30 次，则跳转到日期检测处 1.2  
:00414782 E869B3FFFF              call 0040FAF0 ；否则弹出过期报错窗口  
:00414787 A19C874200              mov eax, dword ptr \[0042879C\]  
:0041478C 8B0D98874200            mov ecx, dword ptr \[00428798\]  
:00414792 3BC1                    cmp eax, ecx  
:00414794 7407                    je 0041479D  
:00414796 50                      push eax  
  
\* Reference To: KERNEL32.FreeLibrary, Ord:0133h  
                                  |  
:00414797 FF15F0064A00            Call dword ptr \[004A06F0\]  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414794(C)  
|  
:0041479D 33C0                    xor eax, eax  
:0041479F E972070000              jmp 00414F16  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00414730(C), :00414780(C)  
|  
:004147A4 A160874200              mov eax, dword ptr \[00428760\] ；猜测：如果是正版安装，则变  
:004147A9 85C0                    test eax, eax                 ；量\[00428760\]=0，跳转到注册  
:004147AB 7459                    je 00414806 2.1        
:004147AD 8D442468                lea eax, dword ptr \[esp+68\]   ；表检查处；否则\[00428760\]存  
:004147B1 50                      push eax                      ；放有效期限信息  
  
\* Reference To: KERNEL32.GetSystemTime, Ord:01C6h  
                                  |  
:004147B2 FF15AC064A00            Call dword ptr \[004A06AC\]  
:004147B8 8B4C2468                mov ecx, dword ptr \[esp+68\] ；当前日期，以下代码为日期格式  
:004147BC 33C0                    xor eax, eax  
:004147BE 668B44246A              mov ax, word ptr \[esp+6A\]   
:004147C3 81E1FFFF0000            and ecx, 0000FFFF  
:004147C9 C1E104                  shl ecx, 04  
:004147CC 8B1560874200            mov edx, dword ptr \[00428760\]  
:004147D2 0BC8                    or ecx, eax  
:004147D4 33C0                    xor eax, eax  
:004147D6 C1E108                  shl ecx, 08  
:004147D9 668B44246E              mov ax, word ptr \[esp+6E\]  
:004147DE 0BC8                    or ecx, eax  
:004147E0 3BCA                    cmp ecx, edx ；和有效期限比较，  
:004147E2 7622                    jbe 00414806 ；未过期，跳到注册表 RunTime 键值的检查处 2.2  
:004147E4 E807B3FFFF              call 0040FAF0 ；弹出过期报错窗口  
:004147E9 A19C874200              mov eax, dword ptr \[0042879C\]  
:004147EE 8B0D98874200            mov ecx, dword ptr \[00428798\]  
:004147F4 3BC1                    cmp eax, ecx  
:004147F6 7407                    je 004147FF  
:004147F8 50                      push eax  
  
\* Reference To: KERNEL32.FreeLibrary, Ord:0133h  
                                  |  
:004147F9 FF15F0064A00            Call dword ptr \[004A06F0\]  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004147F6(C)  
|  
:004147FF 33C0                    xor eax, eax  
:00414801 E910070000              jmp 00414F16  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:004147AB(C), :004147E2(C)  
|  
:00414806 E875B3FFFF              call 0040FB80 ；返回 KEY\_LOCAL\_MACHINE\\Software\\Microsoft  
:0041480B 85C0                    test eax, eax ；\\Windows\\CurrentVersion\\Setup 键值 RunTime  
:0041480D 7522                    jne 00414831 ；RunTime 不为 0，祝贺你能看 VCD 了！！3  
:0041480F E8DCB2FFFF              call 0040FAF0 ；弹出过期报错窗口  
:00414814 8B0D9C874200            mov ecx, dword ptr \[0042879C\]  
:0041481A A198874200              mov eax, dword ptr \[00428798\]  
:0041481F 3BC8                    cmp ecx, eax  
:00414821 7407                    je 0041482A  
:00414823 51                      push ecx  
  
\* Reference To: KERNEL32.FreeLibrary, Ord:0133h  
                                  |  
:00414824 FF15F0064A00            Call dword ptr \[004A06F0\]  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00414821(C)  
|  
:0041482A 33C0                    xor eax, eax  
:0041482C E9E5060000              jmp 00414F16  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041480D(C)  
|  
:00414831 33DB                    xor ebx, ebx ；精彩世界由此进入......  
突破防线  
  
对注册表的相关操作  
\* Referenced by a CALL at Address:  
|:00414806     
|  
:0040FB80 83EC10                  sub esp, 00000010  
:0040FB83 8D442404                lea eax, dword ptr \[esp+04\]  
:0040FB87 53                      push ebx  
:0040FB88 56                      push esi  
:0040FB89 50                      push eax  
:0040FB8A 681F000200              push 0002001F  
:0040FB8F 6A00                    push 00000000  
  
\* Possible StringData Ref from Data Obj ->"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setu"  
                                        ->"p"  
                                  |  
:0040FB91 689C7F4200              push 00427F9C  
:0040FB96 6802000080              push 80000002  
  
\* Reference To: ADVAPI32.RegOpenKeyExA, Ord:00EFh  
                                  |  
:0040FB9B FF1598054A00            Call dword ptr \[004A0598\]  
:0040FBA1 85C0                    test eax, eax  
:0040FBA3 7424                    je 0040FBC9 ；成功打开键，则跳，否则创建键  
:0040FBA5 8D44240C                lea eax, dword ptr \[esp+0C\]  
:0040FBA9 50                      push eax  
  
\* Possible StringData Ref from Data Obj ->"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setu"  
                                        ->"p"  
                                  |  
:0040FBAA 689C7F4200              push 00427F9C  
:0040FBAF 6802000080              push 80000002  
  
\* Reference To: ADVAPI32.RegCreateKeyA, Ord:00DBh  
                                  |  
:0040FBB4 FF1594054A00            Call dword ptr \[004A0594\]  
:0040FBBA 85C0                    test eax, eax  
:0040FBBC 740B                    je 0040FBC9 ；成功创建键，则跳，否则 EAX=FFFFFFFF，返回  
:0040FBBE B8FFFFFFFF              mov eax, FFFFFFFF  
:0040FBC3 5E                      pop esi  
:0040FBC4 5B                      pop ebx  
:0040FBC5 83C410                  add esp, 00000010  
:0040FBC8 C3                      ret  
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0040FBA3(C), :0040FBBC(C)  
|  
:0040FBC9 BE04000000              mov esi, 00000004  
:0040FBCE 8D442410                lea eax, dword ptr \[esp+10\]  
:0040FBD2 8D4C2408                lea ecx, dword ptr \[esp+08\]  
:0040FBD6 50                      push eax  
:0040FBD7 8D542418                lea edx, dword ptr \[esp+18\]  
:0040FBDB 51                      push ecx  
:0040FBDC 8B442414                mov eax, dword ptr \[esp+14\]  
:0040FBE0 52                      push edx  
:0040FBE1 8974241C                mov dword ptr \[esp+1C\], esi  
:0040FBE5 6A00                    push 00000000  
:0040FBE7 89742424                mov dword ptr \[esp+24\], esi  
:0040FBEB 68947F4200              push 00427F94  
:0040FBF0 C744241C00000000        mov \[esp+1C\], 00000000  
:0040FBF8 50                      push eax  
  
\* Reference To: ADVAPI32.RegQueryValueExA, Ord:00F7h  
                                  |  
:0040FBF9 FF1590054A00            Call dword ptr \[004A0590\]  
:0040FBFF 85C0                    test eax, eax ；在这里我曾失误过，改为无条件跳转  
:0040FC01 7425                    je 0040FC28 ；成功读取键值，则跳，否则设置键值 RunTime=64  
:0040FC03 8D442408                lea eax, dword ptr \[esp+08\]  
:0040FC07 56                      push esi  
:0040FC08 8B4C2410                mov ecx, dword ptr \[esp+10\]  
:0040FC0C 50                      push eax  
:0040FC0D C744241064000000        mov \[esp+10\], 00000064  
:0040FC15 56                      push esi  
:0040FC16 6A00                    push 00000000  
:0040FC18 68947F4200              push 00427F94  
  
\* Reference To: ADVAPI32.RegSetValueExA, Ord:0103h  
                                  |  
:0040FC1D 8B359C054A00            mov esi, dword ptr \[004A059C\]  
:0040FC23 51                      push ecx  
:0040FC24 FFD6                    call esi  
:0040FC26 EB06                    jmp 0040FC2E  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040FC01(C)  
|  
  
\* Reference To: ADVAPI32.RegSetValueExA, Ord:0103h  
                                  |  
:0040FC28 8B359C054A00            mov esi, dword ptr \[004A059C\]  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040FC26(U)  
|  
:0040FC2E 8B442408                mov eax, dword ptr \[esp+08\] ；RunTime 键值->eax  
:0040FC32 85C0                    test eax, eax   
:0040FC34 7421                    je 0040FC57 ；为 0 则跳，表示已到期了  
:0040FC36 83F8FF                  cmp eax, FFFFFFFF ；为 FFFFFFFF 则跳，为什么 RunTime=FFFFFFFF  
:0040FC39 741C                    je 0040FC57       ；永不过期，明白了吧！！  
:0040FC3B 48                      dec eax ；RunTime 键值减 1  
:0040FC3C 6A04                    push 00000004  
:0040FC3E 8D4C240C                lea ecx, dword ptr \[esp+0C\]  
:0040FC42 8B542410                mov edx, dword ptr \[esp+10\]  
:0040FC46 8944240C                mov dword ptr \[esp+0C\], eax  
:0040FC4A 51                      push ecx  
:0040FC4B 6A04                    push 00000004  
:0040FC4D 6A00                    push 00000000  
:0040FC4F 68947F4200              push 00427F94  
:0040FC54 52                      push edx  
:0040FC55 FFD6                    call esi  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0040FC34(C), :0040FC39(C)  
|  
:0040FC57 8B44240C                mov eax, dword ptr \[esp+0C\]  
:0040FC5B 50                      push eax  
  
\* Reference To: ADVAPI32.RegCloseKey, Ord:00D8h  
                                  |  
:0040FC5C FF15A0054A00            Call dword ptr \[004A05A0\]  
:0040FC62 8B442408                mov eax, dword ptr \[esp+08\] ；RunTime 键值->eax  
:0040FC66 5E                      pop esi  
:0040FC67 5B                      pop ebx  
:0040FC68 83C410                  add esp, 00000010  
:0040FC6B C3                      ret  
对注册表的相关操作  
  
    诸位，明白了么？什么，还没有？！#^$~&那就再好好看看看雪的教程吧！怎样打补丁就不用我多说了，  
在 1.1 或 1.2、2.1 或 2.2、3 处改为无条件跳转就 Ok。  
  
  
\--### 严正声名 ###--  
    以上内容只能用于经验交流领域，严禁商业用途，请维护正版利益！  
    版权所有，请保障文章的完整性！  
  
woLONGwxd   2001-1-20  
E-mail:wxdny@263.net