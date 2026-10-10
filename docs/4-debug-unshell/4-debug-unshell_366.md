

# EldoS Thumbnailer

> 原文：[`www.pediy.com/kssd/pediy02/forum199.htm`](https://www.pediy.com/kssd/pediy02/forum199.htm)

 这是一个用来查看图标和图像的 30 天试用软件，程序每次启动时首先取得注册表中的 HKEY\_CURRENT\_USER\\Software\\EldoS\\Thumbnailer\\AppData 这一项的值，然后判断是否已过试用期限。如果把注册表中的该项键值删除的话，则程序会重新计算试用时间。但是我比较烦它每次启动时都弹出一个对话框，所以想请各位高手予以破解。   
另外，需要告诉各位的是，该程序如果检测到 SoftICE 存在则不运行。   
下载 URL：http://dengdun.163.net/eltninst.exe   
程序大小为：797KB  

  

 对于懂怎么脱 Asprotect 壳的朋友就看看【前言】【后记】就行了。   
  
脱壳 ------ EldoS Thumbnailer v1.0.0.45   
  
前言：人说："瞎猫碰上死老鼠"，俺居然也有撞上这大运的时候。呵呵！   
  
分析：(以下对可执行文件简称为 EXE，动态链接库简称为 DLL)   
用 FileInfo 分析可知 EXE 没有加壳，而 DLL 用 Asprotect+Aspack2000 加壳。对怎么脱 DLL 的壳还是头一次碰上。先多分析分析，用 ShowDep 看看 EXE 中对 DLL 的引入情况。没有发现对 ElThumbs.dll 的引入！看看 DLL 中的输出函数的情况，没有输出函数！yi！！！怎么发现 Trace 为可用的！！！！一般只有 EXE 才只用的呀！奇怪！先试试再说，单击 Start Tracing 按钮，居然运行了！（当然倒地）。到此一切谜团已解，把 ElThumbs.dll 改名为 WhoAmI.EXE 直接运行成功！ElThumbs.EXE 不过是个......。以后不过是用对 Asprotect 加壳程序的脱壳方法而已。对此有深入分析的文章太多了。我只是简要介绍一下对此脱壳的一些步骤。   
  
目标文件：    WhoAmI.EXE (由 ElThumbs.dll 换名而来)   
加壳方式：    Asprotect+Aspack2000   
所用工具：    SoftICE v4.05,FrogsICE v0.43,ICEDump 6.0.16,ProcDump v1.62   
检测工具：    FileInfo,ShowDep   
作者：        ljttt   
日期：        2000-07-29   
  
1、用 ProcDump 分析基本信息   
Entry Point:    000D7001   
Size of image:    000E9000   
Image Base:    00450000   
  
.idata Section   
Virtual Offset:    000B4000   
Virtual Size:    00003000   
  
因此 .idata Section 内存的地址偏移为 450000+B4000=504000   
  
2、运行 FrogsICE，用 SoftICE 的 Loader 加载 WhoAmI.exe。   
  
3、设断点，使中断后显示.idata Section 信息   
bpx loadlibraya do "dd 450000+B4000"   
  
4、按 F5，中断！此时还没有到时候，按 F5 继续，又中断。这回.idata   
Section 显示的数据是"干净"的，保存到文件。下指令   
/dump 504000 3000 c:\\temp\\dump.bin   
  
5、按 F12 回到程序领空，继续跟踪，直到如下（大家跟踪时的地址可能不一样哦，看代码吧）   
(提示：从跟踪 LoadLibrarya 到跟踪到此，并不远，经过三两个 ret 就到了，具体多少，自己把握)   
015F:00440B19  59                  POP      ECX   
015F:00440B1A  648910              MOV      FS:\[EAX\],EDX   
015F:00440B1D  EB13                JMP      00440B32   
015F:00440B1F  E9C820FFFF          JMP      00432BEC   
015F:00440B24  66B80E00            MOV      AX,000E   
015F:00440B28  E81FF4FFFF          CALL      0043FF4C   
015F:00440B2D  E82622FFFF          CALL      00432D58   
015F:00440B32  8B4508              MOV      EAX,\[EBP+08\]   
015F:00440B35  8B10                MOV      EDX,\[EAX\]   
015F:00440B37  8B4508              MOV      EAX,\[EBP+08\]   
015F:00440B3A  035018              ADD      EDX,\[EAX+18\]   
015F:00440B3D  8B4508              MOV      EAX,\[EBP+08\]   
015F:00440B40  8B401C              MOV      EAX,\[EAX+1C\]   
015F:00440B43  E880F9FFFF          CALL      004404C8    <--以前的按 F10 没关系,这里要按 F8 进入   
015F:00440B48  5F                  POP      EDI   
015F:00440B49  5E                  POP      ESI   
015F:00440B4A  5B                  POP      EBX   
015F:00440B4B  59                  POP      ECX   
015F:00440B4C  59                  POP      ECX   
015F:00440B4D  5D                  POP      EBP   
015F:00440B4E  C20400              RET      0004   
015F:00440B51  8D4000              LEA      EAX,\[EAX+00\]   
015F:00440B54  E82B38FFFF          CALL      KERNEL32!GetCommandLineA   
015F:00440B59  A3F8664400          MOV      \[004466F8\],EAX   
015F:00440B5E  C3                  RET   
      
6、按 F8 进入 Call 004404C8 内部继续跟踪，就到了终点站了。欢呼！   
015F:004404C8  89C4                MOV      ESP,EAX                            
015F:004404CA  89D0                MOV      EAX,EDX                              
015F:004404CC  8B1D34564400        MOV      EBX,\[00445634\]                      
015F:004404D2  89041C              MOV      \[EBX+ESP\],EAX                        
015F:004404D5  61                  POPAD                                          
015F:004404D6  50                  PUSH      EAX    <--此时 EAX=5009B8，所以 Entry Point=5009B8-450000=B09B8   
015F:004404D7  C3                  RET    <--这里就要跳转到真正的 Entry Point 了   
  
7、下指令   
/dump 450000 E9000 c:\\temp\\dump.exe   
  
8、修补工作开始，用 ProcDump 改各 Section 的 Offset 为 RVA 大小，PSize 为 VSize 大小。改 Entry Point 为 B09B8，改 Directory 中的 Import Table 的 RVA 为 B4000，大小为 3000。   
  
9、最后别忘了把 dump.exe 中偏移 B4000 开始的 3000 字节用 dump.bin 替换掉。运行 OK！   
  
后记：   
注册码，俺没有，问题有两个。（哈哈，俺的老毛病也犯了，问大家几个问题？）   
1、据说"Interactive Disassembler v4.04"是目前最好的反汇编工具"，不知国内有哪个地方下载？   
2、该工具和 W32Dasm 相比，块头不一般哦，20M，自然有其过人之处，请问它强在哪里？有什么新功能吗？（和 W32Dasm 相比）   4dd466d0e1e0cc123afbe504da2462ca