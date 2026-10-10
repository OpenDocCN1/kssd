

# SoundEdit Pro

> 原文：[`www.pediy.com/kssd/pediy06/pediy6346.htm`](https://www.pediy.com/kssd/pediy06/pediy6346.htm)

FTBirthday 的 Armadillo 全程教程--第一章．标准加壳方式  
第一章．标准加壳方式  
Armadillo 标准加壳方式如果用了 key 和试用期保护。过期了可以删掉   
\[HKEY\_CLASSES\_ROOT\\CLSID\\{EC9E9EEB-7275-13D1-B2E4-0060975B8649}\]项   
及 HKLM software 下 License 项就不过期了。  
  
\[实例解析一\] Armadillo V3\_01 标准加壳方式的脱壳--SoundEdit Pro  
程序名:SoundEdit Pro V1.3.634    
※软件简介※:SoundEdit Pro 可以打开 WAV、MPC、MP+、MP2、MP3、VOX、RAW、OGG、WMA、CDA 等扩展名的音乐文件，进行声波编辑、剪裁、混音等工作；还有完整的录音功能，可以从 CD、麦克风、立体混音装置、讯号线来源、影片等来源录音。    
软件语言： 英文    
软件类别： 国外软件/共享版/音频处理    
运行环境： Win9x/Me/NT/2000/XP    
界面预览： 无    
软件大小： 10273KB    
软件更新： 2003-7-8    
下载页面： http://www.pcdog.com/soft/942.htm    
  
※参考文章※:Armadillo 标准加壳的程序的脱壳和引入表修复方案 作者：jwh51    
             脱 Armadillo 3.01 壳 作者：yesky1    
             Armadillo V3.40 标准加壳方式的脱壳——Win98 的 Notepad 作者：fly    
  
※软件限制※:SoundEdit 是一个声音编辑软件,采用了 ARMADILLO 的 KEY 加密,启动要你输入 KEY,否则要等几秒才能让你启动,还有 30 天试用期,所以,如果脱了壳,上述现象都将没有了.现在开始脱壳历程.!!      
  
※破解工具※:DriverSuite2.7 中的 softice4.2.7、FI 3.01   
  
※调试平台※:Windows XP   
  
Armadillo 是当今猛壳之一,加壳方式有两种,一种是标准方式(Standard Protections Only)加壳,    
另一种是 CopyMem-II+Debug-Blocker 的加壳,其标准加壳方式相对来说则容易的多.用 COPYMEMII 的一般可用 DILLODUMP 脱,而用标准加壳用 DILLODUMP 是脱不了的。    
  
※开始吧※     
FI 3.01 侦测知其用 Armadilolo v3.01 加壳,并且运行是没有两个进程,所以应该是标准加壳.    
  
\[一\],DUMP 出程序并修复:     
首先一点是 Armadillo 是通过类似于调试的方法来运行程序的。      
新版本 Armadillo 脱壳的一个难点是它的代码段是分页解密的，我估计它的做法是将代码段     
锁定，即将其属性设为不可读、写、执行，当被执行到时会产生一个页面保护错误，而调     
试程序就会捕获这个错误，将对应的页面解锁并解密，然后返回执行.    
  
第一步,打开 softice4.2.7   
第二步,开启 SoundEdit Pro 直到出现注册框,那个注册框上的 OK 键会延迟生效。   
第三步,ctrl+D 调出 softice   
  
附:如果系统时间在安装 SoundEdit Pro 后修改过,程序会检测到,并警告修改系统时间而退出程序.   
如下方法可挫败它的这个保护。   
bpx getlocaltime   
g 回来点击 ok 按键,中断两次后,先别忙 g 退出.   
经过测试内存地址 00CB4677 处的 test al,al 是关键,所以 bpx 00CB4677 再 g.   
((((    
00CB4672   E8 08060000      CALL 00CB4C7F    
00CB4677   84C0             TEST AL,AL      ====>走到这里将 AL 改为 00000000 即可     
00CB4679   74 1C            JE SHORT 00CB4697    
00CB467B   E8 56050000      CALL 00CB4BD6   ====>这个 call 警告修改系统时间   
))))   
中断后,R EAX 00000000 修改寄存器 EAX 的值为 0,然后 BC \*清除所有断点.   
先别忙退出,继续 BPX VirtualProtect.   
至此击败它了.   
  
如果没有改过时间,则跳过上面一步,直接在按 OK 前 BPX VirtualProtect.   
  
然后再次中断,这时你要注意开始记下中断的次数了.我在第 21 次时按 F10 单步返回时得知   
程序 CALL to VirtualProtect from MSVBVM60.7342F400    
说明程序,已经运行了.(VB 是先运行 MSVBVM60.DLL 的,在其中也用调用 VirtualProtect)      
    
重新来一遍当在 20 次出现中断在 VirtualProtect 时停下,BC \*清除所有断点,然后按 F10 单步   
返回到主程序.   
004DA1E0    83C404        ADD    ESP, BYTE+04      
004DA1E3    8945FC        MOV    \[EBP\-4\], EAX      
004DA1E6    837DB800      CMP    DWORD \[EBP\-4C\], BYTE+00      
004DA1EA    740A          JZ     004DA1F6      
004DA1EC    8B45B8        MOV    EAX, \[EBP\-4C\]      
004DA1EF    50            PUSH   EDX      
004DA1F0    FF15E8615000  CALL   USER32.DestroyWindow      
004DA1F6    8B45FC        MOV    EAX,\[EBP\-4\]      
004DA1F9    8BE5          MOV    ESP, EBP      
004DA1FB    5D            POP    EBP      
004DA1FC    C3            RETN      
再耐心地用 F10 单步走,因为有一堆的 JMP,JNZ,JO 等,用不了多久的就可来到这:        
004DB6B0    7511          JNZ    004DB6C3      
004DB6B2    6878BC65000   PUSH   0050BC78      
004DB6B7    FF150C975000  CALL   NEAR \[0050BC9C\]    ,这个 CALL 用 F8 跟进      
004DB6B9    83C404        ADD    ESP, BYTE+04       
在 004DB6B7 这个 CALL 用 F8 跟进,然后继续 F10,几次后就来到这:      
00CBCEA9    334830        XOR       ECX,\[EAX+30\]   
00CBCEAC    334820        XOR       ECX,\[EAX+20\]   
00CBCEAF    2BF9          SUB       EDI,ECX   
00CBCEB1    FFD7          CALL      EDI    =====>EDI\=004059B4   
看到最后这个 CALL EDI 吗,004059B4 就是程序的 OEP 了,我们 F8 跟进:        
004059AE    FF25EC114000  JMP    NEAR\[4011EC\]    
004059B4    68787D4000    PUSH   00407D7C   =====>这是 OEP 了     
004059B9    E8F0FFFFFF    CALL   MSVBVM60!ThunRTMain     
004059BE    0000          ADD    \[EAX\], AL    
典型的 VB 代码.   
  
dump 取内存中己脱壳的文件   
00CBCEB1    FFD7          CALL      EDI   
现在这一行,键入以下命令:    
a eip (然后按回车)   
jmp eip (然后按回车)    
按下 F5 最好在 OEP 处挂起进程再 dump，因为这时程序的初始化变量还没有被修改。大家可对比一下，当程序运行后 dump 出的文件和在 OEP 处 dump 出的有何不同。  
这样将改变 0137:00CBCEB1 行的代码. 你会注意到在键入"jmp eip"并按下回车后,00CBCEB1 的指令现在是一个 jmp.这将有效地使程序"暂停"（有点类似 TRW2000 的 suspend 命令）. 按下 F5 使你回到 window XP。 运行 LordPE,在进程列表中选择"SoundEdit.exe"然后点击鼠标右键选中"Dump (Full)",给脱壳的程序起名 dump 存盘.   
  
\[二\],修复引入表   
一般步骤如下：   
（1）找被脱壳的入口点（OEP）；   
（2）完全 Dump 目标文件；   
（3）运行 Import REConstructor 和需要脱壳的应用程序；   
（4）在 Import REConstructor 下拉列表框中选择应用程序进程；    
（5）在左下角填上应用程序的真正入口点偏移（OEP）；   
（6）按"IAT AutoSearch"按钮，让其自动检测 IAT 位置， 出现"Found address which may be in the Original IAT.Try 'Get Import'"对话框，这表示输入的 OEP 发挥作用了。   
（7）按"Get Import"按钮，让其分析 IAT 结构得到基本信息；   
（8）如发现某个 DLL 显示"valid :NO" ，按"Show Invalids"按钮将分析所有的无效信息，在 Imported Function Found 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"，再按"Show Invalids"按钮。如果成功，可以看到所有的 DLL 都为"valid:YES"字样；    
（9）再次刷新"Show Invalids"按钮查看结果，如仍有无效的地址，继续手动用右键的 Level 2 或 3 修复；   
（10）如还是出错，可以利用"Invalidate function(s)"、"Delete thunk(s)"、编辑 Import 表（双击函数）等功能手动修复。   
（11）开始修复已脱壳的程序。选择 Add new section (缺省是选上的) 来为 Dump 出来的文件加一个 Section(虽然文件比较大，但避免了许多不必要的麻烦) 。   
（12）按"Fix Dump"按钮，并选择刚在（2）步 Dump 出来的文件，在此不必要备份。如修复的文件名是"Dump.exe"，它将创建一个"Dump\_.exe"，此外 OEP 也被修正。   
（13）生成的文件可以跨平台运行。   
    
1.先用 importREC 试试,输入我们找到的正确的 OEP:004059B4,注意输入 004059B4-00400000=000059B4   
有很多指针无效,如果 CUT 程序退出时会出错,所以要修复它.通过它我们可看到 IAT 的     
rva 为 1000   
  
运行 SoundEdit,先等提示注册框出来，再下断 bpx GetModuleHandleA，再按 OK，   
断下 9 次左右，每次断下都 F12 看看回到哪儿了,直到到了如下:   
001B:00CA59F2  FF15A850CC00        CALL      \[KERNEL32!GetModuleHandleA\]   
001B:00CA59F8  394508              CMP       \[EBP+08\],EAX   
001B:00CA59FB  7507                JNZ       00CA5A04   
001B:00CA59FD  B9E083CC00          MOV       ECX,00CC83E0   
001B:00CA5A02  EB52                JMP       00CA5A56   
001B:00CA5A04  393DD889CC00        CMP       \[00CC89D8\],EDI   
001B:00CA5A0A  B9D889CC00          MOV       ECX,00CC89D8   
001B:00CA5A0F  0F8491000000        JZ        00CA5AA6  ======>magic jump!   
001B:00CA5A15  8B3558E8CC00        MOV       ESI,\[00CCE858\]   
001B:00CA5A1B  A1D028CD00          MOV       EAX,\[00CD28D0\]   
001B:00CA5A20  F6410801            TEST      BYTE PTR \[ECX+08\],01   
001B:00CA5A24  740E                JZ        00CA5A34   
001B:00CA5A26  8B507C              MOV       EDX,\[EAX+7C\]   
001B:00CA5A29  335030              XOR       EDX,\[EAX+30\]   
001B:00CA5A2C  335028              XOR       EDX,\[EAX+28\]   
001B:00CA5A2F  F6C280              TEST      DL,80   
001B:00CA5A32  7513                JNZ       00CA5A47   
001B:00CA5A34  8B5074              MOV       EDX,\[EAX+74\]   
001B:00CA5A37  335064              XOR       EDX,\[EAX+64\]   
001B:00CA5A3A  335060              XOR       EDX,\[EAX+60\]   
001B:00CA5A3D  335030              XOR       EDX,\[EAX+30\]   
001B:00CA5A40  3316                XOR       EDX,\[ESI\]   
001B:00CA5A42  395508              CMP       \[EBP+08\],EDX   
001B:00CA5A45  740C                JZ        00CA5A53   
001B:00CA5A47  83C10C              ADD       ECX,0C   
001B:00CA5A4A  83C604              ADD       ESI,04   
001B:00CA5A4D  3939                CMP       \[ECX\],EDI   
001B:00CA5A4F  75CF                JNZ       00CA5A20   
这里面有几个跳转,其中最后一个 00CBA3EB 是往上跳的.我们看一个就可发现:      
00CBA364 JZ  00CBA3F0,这个跳正好可跳出这个循环.只要改这个跳,就不会破坏引入表了.      
再来,出现注册框后 OK,程序再断下来,把 00CA5A0F 的 JZ  00CA5AA6 改成 JMP 00CA5AA6,   
清除所有断点,按 F5 运行,程序运行后打开 importREC,选中程序,入口填   
入 59b4,-->IAT 自动搜索-->获得输入信息",如果还有无效的大可放心的 cut,因为它们本来就不是有用指针了.然后修复脱壳后的文件.OK,大功告成.      
脱壳后的程序,运行正常,烦人的注册框和时间限制都没了,圆满成功.    
  
总结:曾经我用 olly 也试过,但好像我的 olly 有问题，老是断不下来，   
今次换上老将 softice,终于成功了,好激动啊!   
感谢 jwh51 yesky1 fly  4dd466d0e1e0cc123afbe504da2462ca