

# Synchromagic v3.5 build 557

> 原文：[`www.pediy.com/kssd/pediy06/pediy6179.htm`](https://www.pediy.com/kssd/pediy06/pediy6179.htm)

我这个只能说是脱壳流程，没有什么技术含量：P  
工具:LPE-DLX,OllyDbg 1.09d,Import REConstructor v1.6 FINAL,隐藏 OllyDbg 的插件 IsDebug.dll  
目标:Synchromagic v3.5 build 557 

  
这几天在研究用 OD 脱 ASPR 新版本的壳,在此我要感谢 Newbie,jingulong,VolX,lipton,等人对我的帮助,才使得我能写出这篇文章.  

  
我们首先用 OllyDbg 打开 Synchromagic 的主程序.弹出的窗口,点 No.选择 Plugins->IsDebugPresent->Hide,然后按 F9,接着中止在异常处,然后我们按 Shift+F9 一直到如下指令,(注意:不是每台电脑前面的地址都一样的.以指令内容为标准)  
00DB3D5F   3100             XOR DWORD PTR DS:\[EAX\],EAX  
00DB3D61   64:8F05 00000000 POP DWORD PTR FS:\[0\]  
00DB3D68   58               POP EAX  
00DB3D69   833D BC7EDB00 00 CMP DWORD PTR DS:\[DB7EBC\],0   
00DB3D70   74 14            JE SHORT 00DB3D86  
00DB3D72   6A 0C            PUSH 0C  
00DB3D74   B9 BC7EDB00      MOV ECX,0DB7EBC  
00DB3D79   8D45 F8          LEA EAX,DWORD PTR SS:\[EBP-8\]  
00DB3D7C   BA 04000000      MOV EDX,4  
00DB3D81   E8 8ED2FFFF      CALL 00DB1014  
00DB3D86   FF75 FC          PUSH DWORD PTR SS:\[EBP-4\]  
00DB3D89   FF75 F8          PUSH DWORD PTR SS:\[EBP-8\]  
00DB3D8C   8B45 F4          MOV EAX,DWORD PTR SS:\[EBP-C\]  
00DB3D8F   8338 00          CMP DWORD PTR DS:\[EAX\],0  
00DB3D92   74 02            JE SHORT 00DB3D96  
00DB3D94   FF30             PUSH DWORD PTR DS:\[EAX\]  
00DB3D96   FF75 F0          PUSH DWORD PTR SS:\[EBP-10\]  
00DB3D99   FF75 EC          PUSH DWORD PTR SS:\[EBP-14\]  
00DB3D9C   C3               RETN  

我们先在 00DB3D69   833D BC7EDB00 00 CMP DWORD PTR DS:\[DB7EBC\],0 处按 F2 设断.然后再按一次 Shift+F9 中断在该地址,然后去掉断点,接着一直按 F8 单步执行到 00DB3D9C   C3               RETN,此时我们就可以 DUMP 文件了,运行 LORDPE,在进程列表里面选中 Synchromagic 主程序的进程,点击右键选择 full dump,另存为 dumped.exe 文件.接着我们要修补代码.  

我们来看看下面的这些指令  
00467757 >/$ 55 PUSH EBP <此处是 OEP  
00467758 |. 8BEC MOV EBP,ESP  
0046775A |. 6A FF PUSH -1  
0046775C |. 68 A8384A00 PUSH dumped\_.004A38A8  
00467761 |. 68 6CC54600 PUSH dumped\_.0046C56C   
00467766 |. 64:A1 00000000 MOV EAX,DWORD PTR FS:\[0\]  
0046776C |. 50 PUSH EAX  
0046776D |. 64:8925 000000>MOV DWORD PTR FS:\[0\],ESP  
00467774 |. 83EC 58 SUB ESP,58  
00467777 |. 53 PUSH EBX  
00467778 |. 56 PUSH ESI  
00467779 |. 57 PUSH EDI  
0046777A |. 8965 E8 MOV DWORD PTR SS:\[EBP-18\],ESP  
0046777D |. FF15 94934900 CALL DWORD PTR DS:\[<&kernel32.GetVersion>; <-在此处伪造 OEP 00467783 |. 33D2 XOR EDX,EDX  

我们从 67757 开始修复.我们用 LORDPE 打开刚刚 DUMPE 出来的文件,点击 FLC,选中 RVA:填入 67757,然后点 Do,再点 Hex Edit,这时我们看到有一段空白的部分,把以下代码填入后再保存退出即可.  
55 8B EC 6A FF 68 A8 38 4A 00 68 6C C5 46 00 64 A1 00 00 00 00 50 64 89 25 00  
00 00 00 83 EC 58 53 56 57 89 65 E8  

或者有一种方便的方法,用 OllyDbg 打开 DUMP 出来的文件,在 46777A 处双击,填入 push ebp,在 46777B 处填入 mov ebp,esp,然后再保存即可.接下来我们来修复 IAT,先运行未脱壳的 Synchromagic 的主程序,然后再运行 Import REConstructor v1.6 FINAL,选择 Synchromagic 的主程序的进程 OEP 处填入 06777A,再点 IAT AutoSearch,再点 Get  Imports,再点 Show invalid,这时有很多无效的指针,我们在无效的指针处点击右键,选择 Trace Level1 (Dasam)这样大部分的都修复了.接着再点 Show invalid,然后再点击右键选择 Plugin Tracers->ASProtect 1.22 然后就全部的修复了.但是注意,插件修复有一个地方修复错误了.就是 rva:000992D0 mod:kernel32.dll ord:0131 name:GetCurrentProcessId 修复错误了.根据分析,正确的应该是 name=GetCurrentProcess,接着我们再点 Fix Dump,选择刚刚修补代码好的文件,再点 Open,这样 IAT 就修复好了.现在我们来运行主程序看看是不是已经可以运行了呢?我们再点一下 About,结果又出错了.我们关闭主程序,用 OD 打开他,然后点 F9 运行,然后我们再点 About,中断到以下地址:  
004012ED   . F2:AE          REPNE SCAS BYTE PTR ES:\[EDI\]  
我们分析他的 edi,他的 edi 错了，那是壳中的地址，这个值来源与前面不远的地方,我们来看 edi 取值的语句是  
我们往上拉,可以看到如下数据.点击这里  
004012E2  MOV EDI,DWORD PTR DS:\[4B7100\]   
然后在底下出现的 DS:\[004B7100\]=00DA3951 处点击右键,选择 Follow address in Dump.接着我们看 Hex Dump 的那个窗口,地址 004B71000 的数据是 51 39 DA.我们选中这 3 个机器码,按 Ctrl+e 弹出个窗口在 Hex+00 处填入 D01455,因为我们要选一个空闲的地方让他指向,我选择的是 5514DO 所以填入 DO1455 接着再点 OK,然后我们在 Hex dump 窗口处点击右键,选择 Copy to executable file.弹出个窗口,再关闭他,他提示我们保存文件,我们保存这个文件为一个新的文件.然后再运行这个文件,点击 ABOUT,是不是可以显示拉?不过现在显示的是注册给未注册,这样看起来不舒服,我们关闭他,再用 OllyDbg 载入修复 ABOUT 后的文件,然后在 Hex dump 窗口处,拉动滚动条到地址 5514D0 处我选中 5514D0 的 5 个 00,就是 10 个 0,然后按 Ctrl+e,在 ASCII 处填入 Feisu.然后点击 OK,然后我们在 Hex dump 窗口处点击右键,选择 Copy to executable file.弹出个窗口,再关闭他,他提示我们保存文件,我们保存这个文件为一个新的文件.再运行修改后的文件点击 ABOUT 是不是显示为注册给 Feisu 了?你在 5514D0 处也可以填入别的文件,这样子 ABOUT 就显示注册给你填入的值.到此,这个脱壳文章全部写完了.  

飞速\[BCG\]\[IPB\]\[DEF\]\[DCM\]\[YCG\]  

  
 4dd466d0e1e0cc123afbe504da2462ca