

# 对抗 GetTickCount 的一种方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-632.htm`](https://www.pediy.com/kssd/pediy07/pediy7-632.htm)

对抗 GetTickCount 的一种方法  
  
  
很多壳喜欢用检测代码运行时间来判断是否被调试。常用的有 rdtsc 及 GetTickCount。  
调试时一一跳过非常麻烦。rdtsc 可以通过设置 CR4 中的 Time Stamp Disable 位来解  
决。下面尝试把 GetTickCount 一并解决。  
  
简单的办法是直接 Hook 这个 api，返回自己设定的值。但 Hook 容易被检测。有的壳直  
接模仿 GetTickCount 代码，Hook 就不管用了。  
  
  
下面是 WinXP SP1 下 GetTickCount 代码:  
  
77E5A29B >  BA 0000FE7F   mov edx,7FFE0000  
77E5A2A0    8B02          mov eax,dword ptr ds:\[edx\]  
77E5A2A2    F762 04       mul dword ptr ds:\[edx+4\]  
77E5A2A5    0FACD0 18     shrd eax,edx,18  
77E5A2A9    C3            retn  
  
可以看到,直接用 7FFE0000 处的 2 个 dword 计算出结果。<Windows2000 Native API>对这  
里的数据结构说明为:  
  
GetTickCount reads from the KUSER\_SHARED\_DATA page.This page is mapped read-only   
into the user mode range of the virtual address and read-write in the kernel range.  
The system clock tick updates the system tick count, which is stored in this page   
directly.  
  
Reading the tick count from this page is faster than calling ZwGetTickCount.  
The KUSER\_SHARED\_DATA structure is defined in the Windows 2000 versions of ntddk.h.  
  
即这个地址对应的物理页以只读方式映射到用户空间(7FFE0000),以读写方式映射到内核  
空间(FFDF0000)。两个线性地址共享同一物理页。  
  
用 WinDbg 可显示其详细结构:  
  
lkd> dt \_KUSER\_SHARED\_DATA -r  
   +0x000 TickCountLow     : Uint4B  
   +0x004 TickCountMultiplier : Uint4B  
   +0x008 InterruptTime    : \_KSYSTEM\_TIME  
      +0x000 LowPart          : Uint4B  
      +0x004 High1Time        : Int4B  
      +0x008 High2Time        : Int4B  
   +0x014 SystemTime       : \_KSYSTEM\_TIME  
      +0x000 LowPart          : Uint4B  
      +0x004 High1Time        : Int4B  
      +0x008 High2Time        : Int4B  
   +0x020 TimeZoneBias     : \_KSYSTEM\_TIME  
      +0x000 LowPart          : Uint4B  
      +0x004 High1Time        : Int4B  
      +0x008 High2Time        : Int4B  
   +0x02c ImageNumberLow   : Uint2B  
   +0x02e ImageNumberHigh  : Uint2B  
   +0x030 NtSystemRoot     : \[260\] Uint2B  
   +0x238 MaxStackTraceDepth : Uint4B  
   +0x23c CryptoExponent   : Uint4B  
   +0x240 TimeZoneId       : Uint4B  
   +0x244 Reserved2        : \[8\] Uint4B  
   +0x264 NtProductType    :   
      NtProductWinNt = 1  
      NtProductLanManNt = 2  
      NtProductServer = 3  
   +0x268 ProductTypeIsValid : UChar  
   +0x26c NtMajorVersion   : Uint4B  
   +0x270 NtMinorVersion   : Uint4B  
   +0x274 ProcessorFeatures : \[64\] UChar  
   +0x2b4 Reserved1        : Uint4B  
   +0x2b8 Reserved3        : Uint4B  
   +0x2bc TimeSlip         : Uint4B  
   +0x2c0 AlternativeArchitecture :   
      StandardDesign = 0  
      NEC98x86 = 1  
      EndAlternatives = 2  
   +0x2c8 SystemExpirationDate : \_LARGE\_INTEGER  
      +0x000 LowPart          : Uint4B  
      +0x004 HighPart         : Int4B  
      +0x000 u                : \_\_unnamed  
         +0x000 LowPart          : Uint4B  
         +0x004 HighPart         : Int4B  
      +0x000 QuadPart         : Int8B  
   +0x2d0 SuiteMask        : Uint4B  
   +0x2d4 KdDebuggerEnabled : UChar  
   +0x2d8 ActiveConsoleId  : Uint4B  
   +0x2dc DismountCount    : Uint4B  
   +0x2e0 ComPlusPackage   : Uint4B  
   +0x2e4 LastSystemRITEventTickCount : Uint4B  
   +0x2e8 NumberOfPhysicalPages : Uint4B  
   +0x2ec SafeBootMode     : UChar  
   +0x2f0 TraceLogging     : Uint4B  
   +0x2f8 Fill0            : Uint8B  
   +0x300 SystemCall       : \[4\] Uint8B  
  
  
GetTickCount 使用的是:  
  
+0x000 TickCountLow     : Uint4B  
+0x004 TickCountMultiplier : Uint4B  
  
kernel32 中不少 api 都使用了这里的数据。用 IDA 可以找出不少:  
  
GetSystemTime  
GetSystemTimeAsFileTime  
GetLocalTime  
......  
  
注意不只是与时间或日期相关的 api,还有些别的如 Beep。  
  
  
用 OD 加载 notepad,可以看到 7FFE0000 处数据是变化的(每单击 1 次)。  
是谁在写入数据? 用 SoftIce 可找到:  
  
ntoskrnl!KeUpdateSystemTime  
  
.text:0043EFFE     public \_\_stdcall KeUpdateSystemTime()  
.text:0043EFFE \_\_stdcall KeUpdateSystemTime() proc near  
.text:0043EFFE     mov     ecx, 0FFDF0000h  
.text:0043F003     mov     edi, \[ecx+8\]  
.text:0043F006     mov     esi, \[ecx+0Ch\]  
.text:0043F009     add     edi, eax  
.text:0043F00B     adc     esi, 0  
.text:0043F00E     mov     \[ecx+10h\], esi  
.text:0043F011     mov     \[ecx+8\], edi  
.text:0043F014     mov     \[ecx+0Ch\], esi  
.text:0043F017     sub     \_KiTickOffset, eax  
.text:0043F01D     mov     eax, \_KeTickCount.LowPart  
.text:0043F022     mov     ebx, eax  
.text:0043F024     jg      short loc\_43F08B  
.text:0043F026     mov     ebx, 0FFDF0000h  
.text:0043F02B     mov     ecx, \[ebx+14h\]  
.text:0043F02E     mov     edx, \[ebx+18h\]  
.text:0043F031     add     ecx, \_KeTimeAdjustment  
.text:0043F037     adc     edx, 0  
.text:0043F03A     mov     \[ebx+1Ch\], edx  
.text:0043F03D     mov     \[ebx+14h\], ecx  
.text:0043F040     mov     \[ebx+18h\], edx  
.text:0043F043     mov     ebx, eax  
.text:0043F045     mov     ecx, eax  
.text:0043F047     mov     edx, \_KeTickCount.High1Time  
.text:0043F04D     add     ecx, 1  
.text:0043F050     adc     edx, 0  
.text:0043F053     mov     \_KeTickCount.High2Time, edx  
.text:0043F059     mov     \_KeTickCount.LowPart, ecx  
.text:0043F05F     mov     \_KeTickCount.High1Time, edx  
.text:0043F065     mov     ds:0FFDF0000h, ecx  
.text:0043F06B     and     eax, 0FFh  
.text:0043F070     lea     ecx, \_KiTimerTableListHead\[eax\*8\]  
……  
  
可以用驱动直接 patch 这里的代码,使其不再更新数据,或者插入一些别的代码,  
减慢更新速度等。但这里的修改是全局性的,简单地 NOP 掉更新数据的 opcode,  
会导致一些程序不能正常运行。  
  
由于 KUSER\_SHARED\_DATA 的特殊性，可以换个办法。windows 操作系统使用分页  
机制管理内存,每个进程拥有自己的页目录和页表。可以替换掉被调试进程内  
地址 7FFE0000 对应页的 pte,使其指向我们分配的 buffer,这样可以向 buffer 内  
写入数据影响被调试进程,而不会产生全局性的后果。  
  
  
大致的实现如下:  
  
写 1 个 dll,修改 OD 的引入表将这个 dll 映射到 OD 进程空间。Hook OD 的 WaitForDebugEvent:  
  
  
BOOL \_\_stdcall COllyDbg::NewWaitForDebugEvent(  
  LPDEBUG\_EVENT lpDebugEvent,  
  DWORD dwMilliseconds)  
{  
    
  HANDLE  hDebuggedProcess = 0;  
  HANDLE  hDebuggedThread = 0;  
  BOOL  bRet = false;  
    
    
  bRet = m\_pfnWaitForDebugEvent(lpDebugEvent,dwMilliseconds);  //call 原 api  
    
  switch(lpDebugEvent->dwDebugEventCode)   
  {  
  case EXCEPTION\_DEBUG\_EVENT:  
      
    // 第 1 个 int 3 调试事件,  
      
    if((EXCEPTION\_BREAKPOINT ==   
      lpDebugEvent->u.Exception.ExceptionRecord.ExceptionCode)   
    {  
      
      // m\_pDriver 用来与驱动通信  
  
      m\_pDriver->SetTSD();        // for rdtsc ;-)  
      m\_pDriver->ReplacePteOfKuserSharedData();  // 替换 pte  
    }  
    
    break;  
      
  case EXIT\_PROCESS\_DEBUG\_EVENT:    
      
    m\_pDriver->ClearTSD();  
    m\_pDriver->RecoverPteOfKuserSharedData();  // 恢复  
  
    break;  
      
  default:  
    break;  
  }  
    
    
  return bRet;  
}  
  
  
这是在 dll 内定义的 4KB 的 buffer,用来保存从被调试进程的 7FFE0000 读出的 1 页数据(因为这里的  
数据不少 api 要使用,必须保留)。使用单独的 section,可以保证其虚拟地址按页对齐。  
  
#pragma data\_seg("shared")    
DWORD KuserSharedData\[1024\] = {0};  
#pragma data\_seg()  
  
  
ReplacePteOfKuserSharedData 向驱动发送命令,读出 7FFE0000 处原来的 1 页数据到 KuserSharedData,  
替换 pte。  
  
BOOL CDriver::ReplacePteOfKuserSharedData()  
{  
  DWORD  dwBytesReturned = 0;  
  DWORD  dwAddr = (DWORD)&KuserSharedData;  
  
      
  if(INVALID\_HANDLE\_VALUE == m\_hDevice)  // CreateFile 驱动的返回  
  {  
    return FALSE;  
  }  
    
  // 不知道有无必要? 是想确保真正提交到物理页  
      
  for(int i = 0; i <1024; i++)    
  {  
    KuserSharedData\[i\] = i;  
  }  
        
  VirtualLock(KuserSharedData,1024 \* 4);  // 锁定  
  
    
  m\_data\[0\] = (DWORD)(&KuserSharedData);  // 用来传递参数的 array,这里只用 1 个 dword  
  
  // IOCTL\_821  
  
  return DeviceIoControl(  
    m\_hDevice,  
    IOCTL\_821,  
    &dwAddr,    // 传递 buffer 在 OllyDbg 进程空间内的 Virtual Address  
    sizeof(DWORD) \* 1,  
    (PVOID)dwAddr,    // 返回 debuggee 原来 KUSER\_SHARED\_DATA 处的 4KB 数据  
    sizeof(DWORD) \* 1024,  
    &dwBytesReturned,  
    NULL);  
    
}  
  
RecoverPteOfKuserSharedData 在被调试进程结束前调用,恢复被调试进程地址 7FFE0000 对应的原 pte。  
如果不执行,会导致蓝屏，错误为 PFN\_LIST\_CORRUPTED(页帧号链表损坏,如果不用恢复原 pte 的做法,  
应该怎样解决这个问题?我不知道:-(  
  
BOOL CDriver::RecoverPteOfKuserSharedData()  
{  
  DWORD  dwBytesReturned = 0;  
  
  VirtualUnlock(KuserSharedData,1024 \* 4);  
  
  return DeviceIoControl(  
    m\_hDevice,  
    IOCTL\_822,  
    NULL,  
    0,  
    NULL,  
    0,  
    &dwBytesReturned,  
    NULL);  
}  
  
  
  
下面是对应的驱动代码:  
  
替换被调试进程 7FFE0000 对应的 pte:  
  
void ReplacePteOfKuserSharedData(ULONG FakedDataVA,PVOID OutputBuff)  
{  
  // 参数(来自 ring3 的 DeviceIoControl):  
  // FakedDataVA: 即 OD 内 KuserSharedData 的地址  
  // OutputBuff:  指针,把被调试进程的页数据读到这里  
    
  
  ULONG        addrOfPte = 0;  
  ULONG        pte = 0;  
  ULONG        addrOfObjPte = 0;  
  
  PVOID        pEProcess = 0;  
  NTSTATUS      ret = 0;  
  
  PSLOOKUPPROCESSBYPROCESSID  PsLookupProcessByProcessId = 0;  
  KEATTACHPROCESS      KeAttachProcess = 0;  
  KEDETACHPROCESS      KeDetachProcess = 0;  
  
    
  // \_asm int 3  
  
  // 下面的 3 个函数,ntoskrnl.exe 输出了,但 ddk 不支持,要自己取  
  
  PsLookupProcessByProcessId = (PSLOOKUPPROCESSBYPROCESSID)  
      Ring0\_GetProcAddress("PsLookupProcessByProcessId");  
  
  KeAttachProcess = (KEATTACHPROCESS)  
      Ring0\_GetProcAddress("KeAttachProcess");  
  
  KeDetachProcess = (KEDETACHPROCESS)  
      Ring0\_GetProcAddress("KeDetachProcess");  
  
    
  // 此时运行在 OD 的 context 内  
  
  addrOfPte = (FakedDataVA >> 12) \* 4 + PAGE\_TABLE\_BASE;  // 计算 OD 提供的 buffer 对应的 pte;  
  pte = \*((PULONG)addrOfPte);  
  
    
  ret = PsLookupProcessByProcessId(objPid,&pEProcess);  // 获取被调试程序的 EPROCESS  
  KeAttachProcess(pEProcess);        // 等价于 SoftIce 的 addr 命令 ;-)  
  
  RtlCopyMemory(OutputBuff,(PVOID)0x7FFE0000,1024 \* 4);  // copy 原来的数据  
  
  addrOfObjPte = (0x7FFE0000 >> 12) \* 4 + PAGE\_TABLE\_BASE; // 被调试程序空间内对应 7FFE0000 的 pte 虚拟地址  
  
  savedPte = \*((PULONG)addrOfObjPte);      // 保存这个值,恢复时使用  
  
  \*((PULONG)addrOfObjPte) &= 0x00000FFF;      // 保留标记  
  \*((PULONG)addrOfObjPte) |= (pte & 0xFFFFF000);    // 修改物理页地址  
  
  KeDetachProcess();  
}  
  
  
恢复 pte:  
  
void RecoverPteOfKuserSharedData()  
{  
  PVOID        pEProcess = 0;  
  NTSTATUS      ret = 0;  
  
  PSLOOKUPPROCESSBYPROCESSID  PsLookupProcessByProcessId = 0;  
  KEATTACHPROCESS      KeAttachProcess = 0;  
  KEDETACHPROCESS      KeDetachProcess = 0;  
  
  //  
  
  PsLookupProcessByProcessId = (PSLOOKUPPROCESSBYPROCESSID)  
      Ring0\_GetProcAddress("PsLookupProcessByProcessId");  
  
  KeAttachProcess = (KEATTACHPROCESS)  
      Ring0\_GetProcAddress("KeAttachProcess");  
  
  KeDetachProcess = (KEDETACHPROCESS)  
      Ring0\_GetProcAddress("KeDetachProcess");  
  
    
  ret = PsLookupProcessByProcessId(objPid,&pEProcess);    
  KeAttachProcess(pEProcess);      
  
  \*(PULONG)((0x7FFE0000 >> 12) \* 4 + PAGE\_TABLE\_BASE) = savedPte;  
    
  KeDetachProcess();  
}  
  
  
写个程序调用 GetTickCount 测试一下,看起来不错。GetTickCount 现在总返回同一个值(进程  
开始运行时读出的值)。  
  
到这里告一段落。有的壳(如 SDP)在判断 GetTickCount 的差值时,结果为 0 也算错,所以现在的结果还不够。  
可以用 timer 或 thrad 向 KuserSharedData 写入数据，但这样做不好掌握,什么值才能恰好骗过壳代码?  
  
也可以考虑把被调试进程 7FFE0000 的 pte 的 valid 位改为无效,这样代码访问这个页时会产生  
page fault。这样就不必用 thread 或 timer,可以每次在处理页故障时赋值。问题是处理完后,  
又该在什么时候重新将其设置为无效以等待下一次页故障? 也没想清楚。  
  
对付 GetTickCount 似乎有点小题大作了,也是想学点东西。不过,在页表上做手脚绝对是威力  
强大的。 

 4dd466d0e1e0cc123afbe504da2462ca