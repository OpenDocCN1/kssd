

# 用 WinDbg 内核调试查找隐藏进程

> 原文：[`www.pediy.com/kssd/pediy10/75698.html`](https://www.pediy.com/kssd/pediy10/75698.html)

用 WinDbg 内核调试查找隐藏进程  
  
  
查找隐藏进程的方式有很多种，可以枚举内核进程链，可以枚举句柄，也可以暴力搜索内存等。本文介绍的方法就是暴力搜索内存法，不过不需要编码，而是直接利用 WinDbg 本机内核调试功能，查找隐藏进程。内核调试下都可以使用，不过双机调试时速度太慢，要等很久结果才会出来。  
  
首先是定位搜索范围的问题，EPROCESS 结构都是分配在 NonPagedPool 中，只需要搜索这段内存即可。  
  
lkd> dp nt!MmNonPagedPoolStart l1  
81f46104  84005000  
lkd> dp nt!MmSizeOfNonPagedPoolInBytes l1  
81f46108  03dfb000  
  
得到搜索范围，还需要用来搜索的特征码，从而定位 EPROCESS 结构。  
  
lkd> dt nt!\_OBJECT\_HEADER (@$proc - 18)  
   +0x000 PointerCount     : 60  
   +0x004 HandleCount      : 2  
   +0x004 NextToFree       : 0x00000002   
   +0x008 Type             : 0x84040d40 \_OBJECT\_TYPE  
   +0x00c NameInfoOffset   : 0 ''  
   +0x00d HandleInfoOffset : 0 ''  
   +0x00e QuotaInfoOffset  : 0 ''  
   +0x00f Flags            : 0x20 ' '  
   +0x010 ObjectCreateInfo : 0x869cab10 \_OBJECT\_CREATE\_INFORMATION  
   +0x010 QuotaBlockCharged : 0x869cab10   
   +0x014 SecurityDescriptor : 0x9e84701a   
   +0x018 Body             : \_QUAD  
  
偏移为 8 的对象类型可以做为特征码。  
  
lkd> .foreach (place { s-\[1\]b 84005000 L 03dfb000 40 0d 04 84 00 00 00 }) { !process (${place} + 10) 0 }  
PROCESS 84040910  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000  
    DirBase: 00122000  ObjectTable: 88a000c0  HandleCount: 907.  
    Image: System  
  
PROCESS 84686ae0  SessionId: 0  Cid: 0ffc    Peb: 7ffd5000  ParentCid: 02ec  
    DirBase: 7dadf760  ObjectTable: b0e967e0  HandleCount: 291.  
    Image: wmpnetwk.exe  
  
PROCESS 846fc3d0  SessionId: 1  Cid: 0458    Peb: 7ffd5000  ParentCid: 01fc  
    DirBase: 7dadf720  ObjectTable: abbb7518  HandleCount:  19.  
    Image: cmd.exe  
  
PROCESS 84704390  SessionId: 1  Cid: 0bfc    Peb: 7ffd3000  ParentCid: 0458  
    DirBase: 7dadf700  ObjectTable: a568fa90  HandleCount: 287.  
    Image: procexp.exe  
  
PROCESS 84769d90  SessionId: 1  Cid: 0bc8    Peb: 7ffdb000  ParentCid: 0458  
    DirBase: 7dadf740  ObjectTable: a8ffd418  HandleCount:  34.  
    Image: conime.exe  
  
PROCESS 8485ed90  SessionId: 1  Cid: 0f28    Peb: 7ffdc000  ParentCid: 0458  
    DirBase: 7dadf420  ObjectTable: b88b3ab0  HandleCount: 216.  
    Image: notepad.exe  
  
PROCESS 8490cb08  SessionId: 1  Cid: 0ff4    Peb: 7ffd8000  ParentCid: 01fc  
    DirBase: 7dadf380  ObjectTable: b0e6dae0  HandleCount: 113.  
    Image: wmpnscfg.exe  
  
PROCESS 84935690  SessionId: 1  Cid: 0904    Peb: 7ffd9000  ParentCid: 01fc  
    DirBase: 7dadf7a0  ObjectTable: b0f8fc38  HandleCount: 148.  
    Image: TOTALCMD.EXE  
  
PROCESS 84abcd90  SessionId: 1  Cid: 0b68    Peb: 7ffdd000  ParentCid: 01fc  
    DirBase: 7dadf7c0  ObjectTable: af18a1f0  HandleCount: 1030.  
    Image: QQ.exe  
  
PROCESS 84af6020  SessionId: 1  Cid: 0e88    Peb: 7ffda000  ParentCid: 01fc  
    DirBase: 7dadf7e0  ObjectTable: b594a128  HandleCount: 440.  
    Image: firefox.exe  
  
PROCESS 84c11ba0  SessionId: 1  Cid: 0de8    Peb: 7ffda000  ParentCid: 0458  
    DirBase: 7dadf6a0  ObjectTable: aba4ce10  HandleCount:  59.  
    Image: VistaLKD.exe  
  
......  
  
  
这样就能把所有进程结构枚举出来，不过定位哪个是隐藏进程还得和任务管理器显示的内容做比较才知道。  
  
搜索范围不是很准确，好像在 Vista 系统上显示不完整，有时间把 NonPagedPool 分配的机制再研究研究。  
  
  
\----------------------  
小喂  
http://www.DbgTech.net