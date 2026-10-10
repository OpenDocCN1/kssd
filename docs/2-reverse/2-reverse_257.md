

# ICEEXT0.67 指令中英文说明【翻译】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-284.htm`](https://www.pediy.com/kssd/pediy07/pediy7-284.htm)

!BC -  清除内存区块中所有的断点  
!BL -  列出内存区块中所有的断点  
!BPR - 中断在内存范围中  
!CP - 设定当前代码页(866 or 1251)   
!DUMP - dump 内存到磁盘   
!DUMPSCREEN - dumpSoftICE 屏显成 RAW 格式到磁盘  
!INTSAVE - 存储中断句柄  
!INTREST - 恢复中断句柄  
!INTXCHG - 置换中断句柄  
!INTSHOW - 显示内部中断句柄表  
!EB -内存补丁 patch byte in memory (restores PTE after patching)   
!FONT - 设定屏幕字体   
!FPOS        - 显示文件指针  
!LASTBRANCH - 显示最后的 MSR 分支信息  
!LOADFILE - 打开硬盘文件载入到内存镜像  
!PROTECT - 开关 SoftICE 自我保护功能  
!HELP - 显示帮助   
!SUSPEND - 挂起当前执行的线程(仅 ring-3 线程)   
!RESUME - 唤醒被挂起的线程   
!TETRIS - 俄罗斯方块  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
!BC          - clear break point on range                                         
!BL          - list break points on range                                         
!BPR         - break point on range                                               
!CP          - set current codepage                                               
!DUMP        - dump memory to disk                                                
!DUMPSCREEN  - dump SoftICE screen to disk                                        
!EB          - edit byte in memory                                                
!FONT        - set screen font                                                    
!FPOS        - display file pointer                                               
!INTSAVE     - save interrupt handler                                             
!INTREST     - restore interrupt handler                                          
!INTXCHG     - exchange interrupt handler                                         
!INTSHOW     - show internal interrupt handlers table                             
!HELP        - help on specified function                                         
!LASTBRANCH  - show MSR last branch information                                   
!LOADFILE    - load disk file to memory                                           
!PROTECT     - turn protection on/off                                             
!SUSPEND     - suspend thread                                                     
!RESUME      - resume thread                                                      
!TETRIS      - tetris game      
另外  
 如果不知道具体的使用方式可以使用!help 命令查询  
其中还增加了!if ,!idt  
关于在 ICEEXT 的重新编译需要修改  
make.bat 和 source 文件  
set SAVEDDIR=%CD%  
  
call %BASEDIR%\\bin\\setenv.bat %BASEDIR% fre wxp  
cd /d %SAVEDDIR%  
  
set SAVEDDIR=  
  
del .\\objfre\_wxp\_x86\\i386\\\*.res  
  
build  
rem prefast build  
rem prefast list  
//  
TARGETNAME=IceExt  
TARGETPATH=obj  
TARGETTYPE=MINIPORT  
  
AXP64\_FLAGS=/QA21164  
  
INCLUDES=..\\inc;..\\..\\inc  
  
MSC\_WARNING\_LEVEL=-W4 -WX  
MSC\_OPTIMIZATION =-0xs  
  
LINKER\_FLAGS=/DEF:IceExt.def /MERGE:.rdata=.data  
  
CBSTRING=;  
  
DDKLIBPATH=$(BASEDIR)\\LIB\\WXP\\i386  
/\*W2K 修改为 WXP  
# \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
# remove ntoskrnlnt4.lib if you don't have it  
# I'm using old library for NT4 compability  
# \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
TARGETLIBS=$(DDKLIBPATH)\\ntoskrnl.lib $(DDKLIBPATH)\\hal.lib ntice.lib .\\libmad\\msvc++\\Release\\libmad.lib  
/\*ntoskrnlnt4.lib 修改为 ntoskrnl.lib  
SOURCES=ac97.cpp           \\  
        calloc.cpp         \\  
        bpr.cpp            \\  
        fileio.cpp         \\  
        seh.cpp            \\  
        pci.cpp            \\  
        taskswch.cpp       \\  
        util.cpp           \\  
        softice.cpp        \\  
        Search.cpp         \\  
        screen.cpp         \\  
        siwvid.cpp         \\  
        modules.cpp        \\  
        main.cpp           \\  
        multicpu.cpp       \\  
        mp3.cpp            \\  
        pgfault.cpp        \\  
        process.cpp        \\  
        protect.cpp        \\  
        pageinprogress.cpp \\  
        tracer.cpp         \\  
        activatehook.cpp   \\  
        ntoskrnl.cpp       \\  
        cp1251.c           \\  
        font08x08ru.c      \\  
        font08x14ru.c      \\  
        font08x16ru.c      \\  
        font10x20ru.c      \\  
        cmd\_cp.cpp         \\  
        cmd\_pte.cpp        \\  
        cmd\_bpr.cpp        \\  
        cmd\_dump.cpp       \\  
        cmd\_dumpscreen.cpp \\  
        cmd\_eb.cpp         \\  
        cmd\_help.cpp       \\  
        cmd\_font.cpp       \\  
        cmd\_fpos.cpp       \\  
        cmd\_irql.cpp       \\  
        cmd\_idt.cpp        \\  
        cmd\_ifx.cpp        \\  
        cmd\_int.cpp        \\  
        cmd\_lastbranch.cpp \\  
        cmd\_loadfile.cpp   \\  
        cmd\_protect.cpp    \\  
        cmd\_sdt.cpp        \\  
        cmd\_suspend.cpp    \\  
        cmd\_test.cpp       \\  
        cmd\_tetris.cpp     \\  
        cmd\_unhook.cpp     \\  
        iceext.rc            
  
DLLDEF=IceExt.def 

现在的问题不是 ICEEXT 怎么用的问题，是挂不上！  
我的系统 XP SP2  安 STUDIO 3。2 之后安装了 0。67  
在启动 ICE，再启动 ICEEXT 后，执行命令，总说当前扩展  
命令未找到。前几天看到有兄弟问这个事了，但好象  
没有得以答案。所以我也只好问问了！  
  
比如执行命令 ！PROTECT 就出上面的提示。 

1、确保修改了,NTICE 在注册表中的 heap,stack 的键值为 0x00100000  
2、取确保 ICEEXT 驱动被注册，如果没有注册请先注册  
3、XP 操作系统 SP2 是否正常，据说 ICEEXT 和某些杀毒软件有冲突，建议使用 antivirs,nv9 等杀毒软件。  
ICEEXT 是共享内存访问的，对于某些实时监控内存读写的杀毒软件会导致保护性错误，建议停掉杀毒软件后调用 

 4dd466d0e1e0cc123afbe504da2462ca