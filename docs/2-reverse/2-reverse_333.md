

# 蓝屏的调试艺术

> 原文：[`www.pediy.com/kssd/pediy11/113285.html`](https://www.pediy.com/kssd/pediy11/113285.html)

原文已发表黑客防线  
蓝屏的调试艺术  
1、前言  
  当前，恶意程序和杀毒软件对系统控制权的争夺是愈演愈烈，各大杀毒厂商都是挖空心思在系统底层做文章。众多网友也是不甘示弱，都投入到系统底层开发之中，但是不少人是比葫芦画瓢，直接拿别人代码来用，包括我刚开始做的时候也是这样。但是由于种种原因，别人运行正常的代码到你电脑上它就是不能正常运行了，蓝屏了。遇到这种情况，可能就傻眼了，盯着代码不知所措，问高手也不知道什么时候高手心情高兴给解释了，或者高手直接来句“根据 dump 文件双机调试就解决了“。但是我们是菜鸟，我们没有高手所修的内功的，我们不知道怎么做“根据 dump 文件双机调试就解决了”，我们也双机调试了，但是我们没解决。其实很多时候，高手所谓的简单也是在拥有了几年的开发经验之后才觉得简单。最近看黑防有些读者写驱动遇到问题，不知所措，有感于我的学习经历，“授人以鱼不如授人以渔”，因此作此文，一来解惑二来共勉。  
2、Windbg 调试环境搭建  
2.1、Loacl Kernel Debug（本地调试）：有时候我们只是看看内核的某些数据结构，直接利用本地调试就行。  
环境配置：  
1、  打开 windbg 的 kernel debug，选择第四个 Local 点确定  
2、  设置本地调试的符号路径，打开 windbg 的 symbol file path 选项  
srv\*C:\\LocalSymbols\*http://msdl.microsoft.com/download/symbols 然后点 reload。Windbg 从微软符号服务器下载跟你操作系统相对应的符号文件，因此需要联网操作。  
符号文件跟操作系统正确对应的话，就出现 Connected to Windows XP 2600 x86 compatible target, ptr64 FALSE  
lkd> .reload  
Unable to read head of debugger data list  
Connected to Windows XP 2600 x86 compatible target, ptr64 FALSE  
Loading Kernel Symbols  
............................................................................................................................  
Loading User Symbols  
Loading unloaded module list  
  
之后，我们就可以开始本地调试了。比如看 EPROCESS 的结构，直接本地 dt \_eprocess 就出来了。比如反汇编函数 ZwCreateProcessEx  
lkd> uf ZwCreateProcessex  
ntdll!NtCreateProcessEx:  
7c92d140 b830000000      mov     eax,30h  
7c92d145 ba0003fe7f      mov     edx,offset SharedUserData!SystemCallStub (7ffe0300)  
7c92d14a ff12            call    dword ptr \[edx\]  
7c92d14c c22400          ret     24h  
2.2、双机调试：主要用于单步调试程序，动态调试，定位驱动程序出现蓝屏的原因或者逆向动态分析。  
环境配置：  
1、设置虚拟机（以 Vmware 汉化版、Windows  XP 系统为例）  
  打开 Vmware 中安装的 XP 虚拟机，右键点击“设置”，出现虚拟机硬件设置项  
![](img/44f05c6a7bcdc72e3091a48120aa2b69.jpg)   
点击下面的“添加”，下一步选择“串口，下一步选择“输出到重命名管道”  
![](img/231036169e2e7ae7c916c36c0a64674e.jpg)  
![](img/649e1994ff338489c8afe10accf7ee3e.jpg)  
按照上边设置，点击完成就 OK。这样双机调试的时候，虚拟机就作为服务器让 Windbg 来调试了。  
2、Vmware 中 XP 设置  
编辑 C 盘根目录文件 boot.int（隐藏只读文件，编辑的时候去掉右键去掉只读属性）  
\[boot loader\]  
timeout=5  
default=multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS  
\[operating systems\]  
multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Microsoft Windows XP Professional" /noexecute=optin /fastdetect   
将最后一行复制，加上新的启动参数就完成的虚拟机的设置了。  
multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Microsoft Windows XP Professional Debug" /fastdetect /debugport=com1 /baudrate=115200  
  
3、Windbg 参数设置  
  我一般建立 2 个 Windbg 快捷方式，分别对应 Local 调试和双机调试。  
双机调试 Windbg 参数设置如下：  
右键点击双机调试对应的快捷方式，点“属性”，在“目标”一栏设置下面就 OK  
"C:\\Program Files\\Debugging Tools for Windows\\windbg.exe"   
\-b -k com:port=\\\\.\\pipe\\com\_1,baud=115200,pipe  
之后双机调试的时候选择双机调试的快捷方式，Local 调试用另一个，这样设置后就方便多了。  
  
4、dump 文件设置  
  蓝屏发生后，依据 dump 文件查找程序错误。  
右键“我的电脑”，选择系统属性，点击“高级”，对“启动和故障恢复”进行设置  
将事件写入系统日志打钩，选择小内存存储就可以了，填上保存 dump 的目录。按照我下图设置就 OK 了，这样当蓝屏发生的时候 windows 就会自动把蓝屏记录的信息保存到这个文件夹里了。  
![](img/8a0cff3910cbcfa2a27416322d8be571.jpg)  
5、设置虚拟机符号链接  
  启动虚拟机之后，启动双机调试 windbg 快捷方式，选择 Debug 模式的操作系统，连接上之后就可以设置虚拟机对应的符号链接了  
srv\*C:\\VmSymbols\*http://msdl.microsoft.com/download/symbols 然后点 reload。Windbg 从微软符号服务器下载跟你操作系统相对应的符号文件，因此需要联网操作。  
  这一部分，原本并不想这么详细的介绍，网上也有详细资料。但是许多资料都是偏于一方面，导致刚开始环境都配置错误，双机调试更别提了，因此我在这这里加以详细介绍，我也几多想删去这一部分，考虑到双击调试环境的重要性及诸多问题，最终还是保留下来了。  
  
  
  
  
3、  Windbg 双机源码级调试常用命令  
源码级双机调试常用命令分为 3 类：  
1、  下断点  
2、  跟踪代码:  
3、  查看数据  
基本用法见表一，详细用法介绍，请参考 windbg 帮助文档。  
Windbg 源码级双机调试常用命令  
命令  用法  描述  
bp  bp 驱动名!DriverEntry  下函数中断  
bl   bl  查看所下断点  
bc   bc 断点号  取消断点  
F9  选中某行，按下 F9 下断，再按一下取消断点，相当于 OD 中的 F2，源码随机下断  
F10  单步跳过，遇到 CALL 函数调用跳过函数，相当于 OD 中的 F8  
F8 或者 F11  单步进入，遇到 CALL 函数调用进入函数，相当于 OD 中的 F7  
F7  运行到执行行，相当于 OD 中的 F4  
Shift + F11  跳出当前函数  
dd   dd 地址 或 dd 变量名  查看内存地址数据  
dt   dt  结构名 地址  查看结构中的数据  
  
  
  
4、源码双机调试详细步骤（环境搭建成功的前提下，环境没搭建成功，一切白搭）  
1、成功连接上虚拟机  
2、设置符号路径  
srv\*C:\\VmSymbols\*http://msdl.microsoft.com/download/symbols;G:\\SAE\\20100107\\objchk\_wnet\_x86\\i386  
虚拟机符号+我们驱动符号  
3、设置源代码路径  
G:\\SAE\\20100107  
4、下函数断点      bp sae!DriverEntry  
5、加载驱动         
6、中断在所下函数入口，然后跟踪代码  
7、查看一些变量的数据，是否正确，确定程序问题  
5、基于源码的双机蓝屏调试  
常见蓝屏问题：  
1、内存问题：溢出，指针指向错误地址、地址无效等  
2、参数无效：参数指向错误，无效等  
3、系统问题：线程、DPC、中断级等  
解决蓝屏步骤：  
1、依据 dump 文件，结合符号文件定位出错所在模块及所在函数  
2、依据 dump 分析的结果，进行代码注释、审查，试图找出问题  
3、若无法确定问题，采用双机调试，准确定位出错所在代码  
4、依据定位到的错误进行修改（可以利用搜索引擎查找相关信息，然后再进行修改）  
下面就分别介绍 2 个蓝屏实例进行介绍：  
1、内存越界  
代码如下：  
//=======函数声明=============================================  
NTSTATUS DriverEntry(PDRIVER\_OBJECT pDrvObject, PUNICODE\_STRING pRegString);  
VOID DriverUnload(PDRIVER\_OBJECT pDrvObject);  
//创建一个系统线程  
BOOLEAN CreateLogThread(PKSTART\_ROUTINE StartRoutine,ULONG Flags,PVOID Thread);  
//写数据  
VOID WriteProcessLog(PVOID context);  
  
NTSTATUS PsLookupThreadByThreadId(  
                  IN HANDLE ThreadId,      
                  OUT PETHREAD \*Thread);  
  
ULONG pFlags=0;  
PVOID pThread=NULL;  
//==========================================  
NTSTATUS DriverEntry(PDRIVER\_OBJECT pDrvObject, PUNICODE\_STRING pRegString)  
{  
  NTSTATUS status = STATUS\_SUCCESS;  
  KdPrint(("\[1\] DriverEntry: %S\\n",pRegString->Buffer));  
  pDrvObject->DriverUnload = DriverUnload;  
  //创建线程  
  if(CreateLogThread(WriteProcessLog,pFlags,pThread))       //线程  
  {  
    KdPrint(("CreateThread Success\\n"));  
  }  
  
  return STATUS\_SUCCESS;  
}  
  
  
VOID DriverUnload(PDRIVER\_OBJECT pDrvObject)  
{    
  if(pThread!=NULL)  
  {  
    pFlags = 0;  
    KeWaitForSingleObject(pThread,Executive,KernelMode,0,0);  
  }  
  KdPrint(("\[1\] Unloaded\\n"));  
}  
//  
//写数据  
VOID WriteProcessLog(PVOID context)  
{  
  do  
  {  
    KdPrint(("\[WriteProcessLog\] OK\\n"));  
  
  }while(pFlags);  
  //退出循环，线程结束  
  PsTerminateSystemThread(STATUS\_SUCCESS);  
}  
  
//创建一个系统线程  
BOOLEAN CreateLogThread(PKSTART\_ROUTINE StartRoutine,PULONG Flags,PVOID Thread)  
{  
  NTSTATUS status;  
  CLIENT\_ID cid;  
  HANDLE ThreadHandle = NULL;  
  status = PsCreateSystemThread(  
                  &ThreadHandle,  
                  0L,  
                  NULL,  
                  NULL,  
                  &cid,  
                  StartRoutine,  
                  NULL);  
  if(!NT\_SUCCESS(status))  
  {  
    KdPrint(("\[CreateLogThread\]PsCreateSystemThread error\\n"));  
    KdPrint(("\[CreateLogThread\] NTSTATUS error:0x%x\\n",status));  
    return 0;  
  }  
  status = PsLookupThreadByThreadId(cid.UniqueThread, (PETHREAD \*)Thread);  
  if(!NT\_SUCCESS(status))  
  {  
    KdPrint(("\[CreateLogThread\]PsCreateSystemThread error\\n"));  
    KdPrint(("\[CreateLogThread\] NTSTATUS error:0x%x\\n",status));  
    ZwClose(ThreadHandle);  
    return 0;  
  }  
  ZwClose(ThreadHandle);  
  Flags = 1;  
  return 1;  
}  
蓝屏分析过程  
1、1.sys 驱动加载的时候，蓝屏发生，因此可以初步猜测蓝屏发生在 DriverEntry 中  
2、Windbg 打开蓝屏对应的 dump 文件，配置正确符号文件，点分析获取详细信息  
3、dump 文件反馈的信息，我们重点关注  
（1）蓝屏错误号  
（2）蓝屏发生的模块及所在函数  
（3）蓝屏发生在哪一行代码（这个有时候不准确的）  
蓝屏错误号：SYSTEM\_THREAD\_EXCEPTION\_NOT\_HANDLED\_M (1000007e)  
蓝屏发生的模块：IMAGE\_NAME:  1.sys  
                SYMBOL\_NAME:  1!CreateLogThread+61  
蓝屏发生的代码：  
FOLLOWUP\_IP:   
1!CreateLogThread+61 \[d:\\\\\\2010.2\\code\\1\\1.c @ 76\]  
f9ca6271 8945f4          mov     dword ptr \[ebp-0Ch\],eax  
  
FAULTING\_SOURCE\_CODE:    
    72:     KdPrint(("\[CreateLogThread\]PsCreateSystemThread error\\n"));  
    73:     KdPrint(("\[CreateLogThread\] NTSTATUS error:0x%x\\n",status));  
    74:     return 0;  
    75:   }  
\>   76:   status = PsLookupThreadByThreadId(cid.UniqueThread, (PETHREAD \*)Thread);  
    77:   if(!NT\_SUCCESS(status))  
    78:   {  
    79:     KdPrint(("\[CreateLogThread\]PsCreateSystemThread error\\n"));  
    80:     KdPrint(("\[CreateLogThread\] NTSTATUS error:0x%x\\n",status));  
    81:     ZwClose(ThreadHandle);  
依据 dump 反馈的信息，我们可以得到下面得猜测：1.sys 的加载导致蓝屏发生，问题是出在函数 CreateLogThread 中的这一句  
status = PsLookupThreadByThreadId(cid.UniqueThread, (PETHREAD \*)Thread);  
假如我们没有想到原因，那我们可以采用双机调试进行调试。  
1、windbg 连接上虚拟机，设置符号路径，源文件路径  
2、对 DriverEntry 下断点  bp 1!DriverEntry  
3、在虚拟机加载 1.sys 文件，系统中断在 DriverEntry 首行，单步进入 CreateLogThread 函数中  
5、先看下参数 dd pThread 发现数值为 0，因此发现问题出现在参数传递中，也就是说我们本来要出入的是 pThread 在内存中的地址，却不小心传入了 pThread 的值，也就是 0，因此导致地址无效，蓝屏发生，我们修改  
(CreateLogThread(WriteProcessLog,pFlags,pThread)为  
(CreateLogThread(WriteProcessLog,pFlags,&pThread)也就是传入 pThread 所在内存的地址而不是 pThread 的值。因此有时候 dump 文件显示的错误代码并不代表错误就一定是这里导致的，也可能是前面参数问题。至此，我们已经找出了加载驱动蓝屏发生的原因，我们修改编译后继续测试，一切正常。  
另外我提供了一道练习题，大家可以自己动手分析一下。   
2、Hook  ZwTerminateProcess 遇到的问题  
代码如下：  
//My 函数  
NTSTATUS MyZwTerminateProcess(    
                   IN HANDLE  ProcessHandle,      
                   IN NTSTATUS  ExitStatus )  
{  
  NTSTATUS status;  
  PCHAR path = (PCHAR)ExAllocatePool(NonPagedPool, 256);  
  //得到路径  
  GetTerminateProcessPath(ProcessHandle, path);  
  KdPrint(("\[MyZwTerminateProcess\]path:%s\\n",path));  
  //执行函数  
  status = OrigZwTerminateProcess(   
                                  ProcessHandle,      
                        ExitStatus);  
  return status;  
}  
//原理  
/\*Eprocess->sectionobject(0x138)->Segment(0x014)->ControlAera(0x000)->FilePointer(0x024)->(FileObject->FileName,FileObject->DeviceObject)\*/  
VOID GetProcessPath(ULONG eprocess,PCHAR ProcessImageName)  
{  
  ULONG object;  
  PFILE\_OBJECT FileObject;  
  UNICODE\_STRING FilePath;   
  UNICODE\_STRING DosName;   
  STRING AnsiString;   
  
  FileObject = NULL;   
  FilePath.Buffer = NULL;   
  FilePath.Length = 0;   
  \*ProcessImageName = 0;    
    
  if(MmIsAddressValid((PULONG)(eprocess+0x138)))//Eprocess->sectionobject(0x138)  
  {  
    object=(\*(PULONG)(eprocess+0x138));  
        //KdPrint(("\[GetProcessFileName\] sectionobject :0x%x\\n",object));  
    if(MmIsAddressValid((PULONG)((ULONG)object+0x014)))  
    {  
      object=\*(PULONG)((ULONG)object+0x014);  
      //KdPrint(("\[GetProcessFileName\] Segment :0x%x\\n",object));  
      if(MmIsAddressValid((PULONG)((ULONG)object+0x0)))  
      {  
        object=\*(PULONG)((ULONG\_PTR)object+0x0);  
        //KdPrint(("\[GetProcessFileName\] ControlAera :0x%x\\n",object));  
        if(MmIsAddressValid((PULONG)((ULONG)object+0x024)))  
        {  
          object=\*(PULONG)((ULONG)object+0x024);  
          //KdPrint(("\[GetProcessFileName\] FilePointer :0x%x\\n",object));  
        }  
        else  
          return ;  
      }  
      else  
        return ;  
    }  
    else  
      return ;  
  }  
  else  
    return ;  
    FileObject=(PFILE\_OBJECT)object;  
  
  FilePath.Buffer = ExAllocatePool(PagedPool,0x200);  
  FilePath.MaximumLength = 0x200;   
    //KdPrint(("\[GetProcessFileName\] FilePointer :%wZ\\n",&FilePointer->FileName));  
  ObReferenceObjectByPointer((PVOID)FileObject,0,NULL,KernelMode);//引用计数+1，操作对象  
    
  RtlVolumeDeviceToDosName(FileObject-> DeviceObject, &DosName);   
  RtlCopyUnicodeString(&FilePath, &DosName);   
  RtlAppendUnicodeStringToString(&FilePath, &FileObject->FileName);   
  ObDereferenceObject(FileObject);   
     
  RtlUnicodeStringToAnsiString(&AnsiString, &FilePath, TRUE);   
  if ( AnsiString.Length >= 216 )   
  {   
    memcpy(ProcessImageName, AnsiString.Buffer, 0x100u);   
    \*(ProcessImageName + 215) = 0;   
  }   
  else   
  {   
    memcpy(ProcessImageName, AnsiString.Buffer, AnsiString.Length);   
    ProcessImageName\[AnsiString.Length\] = 0;   
  }   
  RtlFreeAnsiString(&AnsiString);   
  ExFreePool(DosName.Buffer);   
  ExFreePool(FilePath.Buffer);   
}  
  
//根据 ProcessHandle 得到 EPROCESS  然后得到结束进程全路径  
VOID GetTerminateProcessPath( HANDLE ProcessHandle, char \*ProcessPath)  
{  
  NTSTATUS status;  
  PVOID ProcessObject;  
  ULONG eprocess;  
  status = ObReferenceObjectByHandle( ProcessHandle ,0,\*PsProcessType,KernelMode, &ProcessObject, NULL);  
  if(!NT\_SUCCESS(status))   //失败  
  {  
    DbgPrint("Object Error");  
    KdPrint(("\[GetTerminateProcessPath\] error status:0x%x\\n",status));  
  }  
  KdPrint(("\[GetTerminateProcessPath\] Eprocess :0x%x\\n",(ULONG)ProcessObject));  
  //Object 转换成 EPROCESS: object 低二位清零  
  eprocess = ((ULONG)ProcessObject) & 0xFFFFFFFC;  
  ObDereferenceObject(ProcessObject);  
  GetProcessPath( eprocess ,ProcessPath);  
}                                                      
蓝屏分析报告：  
1、1.sys 驱动加载后，当结束程序的蓝屏发生，因此怀疑 MyZwTerminateProcess 导致的蓝屏  
2、依据 dump 反馈的信息，我们得到：  
蓝屏代号：PAGE\_FAULT\_IN\_NONPAGED\_AREA  
蓝屏发生的模块：IMAGE\_NAME:  1.sys  
                SYMBOL\_NAME:  1!GetTerminateProcessPath+4b  
蓝屏发生在：1.c @ 201    GetProcessPath( eprocess ,ProcessPath);  
依据 dump 反馈的信息，我们可以猜测 GetProcessPath 函数导致的蓝屏发生。  
3、对 GetTerminateProcess 下断点，单步执行，当执行到  
status = ObReferenceObjectByHandle( ProcessHandle ,  
0,  
\*PsProcessType,  
KernelMode,   
&ProcessObject,   
NULL);  
时发现，这个函数执行失败，返回 Error NTSTATUS 是：0xC00000008 为什么失败：  
#define   STATUS\_INVALID\_HANDLE        ((NTSTATUS)0xC0000008L)  
无效句柄导致的函数执行失败，ProcessObject 错误导致 GetProcessPath 执行发生错误。  
4、为什么这里会有无效句柄呢，利用搜索引擎搜索，发现要结束自身进程的时候，ProcessHandle 为空，因此这里执行失败。那怎么办呢？  
5、参考 WRK 中的 ZwTerminateProcess 的代码,发现 MS 对句柄有进行修正之后，才调用的 ObReferenceObjectByHandle。  
if (ARGUMENT\_PRESENT (ProcessHandle)) //利用 ARGUMENT\_PRESENT 宏对句柄是否为空进行了判断  
{  
        ProcessHandleSpecified = TRUE;  
      
} else   
{  
        ProcessHandleSpecified = FALSE;  
        ProcessHandle = NtCurrentProcess();   //为空得到当前的进程句柄  
}  
找出了原因，我们也仿效 WRK 做法，利用宏对 ProcessHandle 进行处理。这里我需要说明一下，这段代码是来源于帮别人分析蓝屏原因得到的，里面有些地方，值得引起我们的注意。  
1、既然进行容错处理，就要进行完善的容错处理啊，这里既然对  
ObReferenceObjectByHandle 是否执行成功进行了容错处理，却缺少一句 return 跳出函数。  
2、对函数一些参数我们要进行检查，确保参数有问题时及早的中止代码执行。  
  
蓝屏原因无花八门，但是只要我们掌握规范的的内核编程，并学会利用 windbg 双机调试，我们就能够解决很大部分蓝屏，而不是处处求人。最近有人让我帮着解决蓝屏问题，经过我分析找到蓝屏原因，发现都是很小的问题，很多都是细节问题，因此觉得有必要在帮助解决问题的同时把方法也教给别人，特作此文，希望以后读者都能做到“自救”。  
6、减少蓝屏的做法  
由于内核内存空间是共享的，稍有不慎就会导致蓝屏发生，因此我们写内核模块时思路要清晰，代码尽量规范，采用一些正确的编程方法，依然能够帮助我们尽可能的减少蓝屏的几率。  
可以采用的下做法如下：  
  
1、拥有完整的函数容错处理代码，便于找问题（这个一定要重视）  
2、利用 KdPrint 不断验证代码执行后的结果是否与预期相符，及早发现异常问题，避免小问题到后面被放大化，增加分析难度  
3、内存分配与回收要异常小心，利用 try-except 处理  
4、不要随便用别人的内核代码，要自己理解后，自己写，安全才能把握，一味的复制、粘贴程序只会问题越来越多  
5、写完后，进行代码自我审查  
依据我的经验，如果读者注意了以上五点，许多蓝屏都能在初期就被扼杀住，而不至于扩大范围，影响分析。  
7、总结  
  由于蓝屏原因实在太多了，无法一一分析介绍，特地选择几个有代表性的分析一下，重在介绍方法。我也尽力把方法说清楚，但是由于这个需要真实的去实践，有些地方说的不是很明白，大家有什么问题直接发邮件问我，或者观看随文提供的录像。希望大家看完这篇文章，学会调试步骤、技巧、方法，以后遇到蓝屏，能够有勇气说：一切“蓝屏”都是纸老虎。这也是我本人想告诉大家的，只要掌握调试蓝屏的方法，蓝屏就是“纸老虎”。  
  自己动手丰衣足食，守株待兔一无所获。当我们遇到问题的时候，不妨试着自己解决下，自己靠双手花 10 个小时解决问题也比请别人花 10 分钟解决问题收获更多。最后，我用一句话来和所有从事底层开发的读者共勉：Blue Screen of Death is inevitable.But to believe yourself, it is more reliable.Never give up，everything is possible!（蓝屏是不可避免的，与其靠别人不如靠自己，坚持下去，一切皆有可能）。