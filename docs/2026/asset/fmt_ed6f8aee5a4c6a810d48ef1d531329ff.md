
# Windows内核-系统调用（三）

## 三、系统调用

### 3.1.系统调用

**OpenProcess和ReadProcessMemory从3环进0环的过程**

- `kernel32.OpenProcess`
- `KernelBase.OpenProcess`
- `Ntdll.NtOpenProcess`
- `7FFE0300`（获取`KiFastCallEntry`）
- `sysenter`

```
kernel32.OpenProcess(什么都不干，直接jmp到KernelBase) --> KernelBase.OpenProcess -->Ntdll.NtOpenProcess (导出的函数是ZwOpenProcess) -->7FFE0300(调用KiFastCallEntry) -->sysenter-->进入内核

kernel32.ReadProcessMemory --> KernelBase.ReadProcessMemory -->Ntdll.NTReadProcessMemory -->7FFE0300 -->sysenter
```

**`ZwOpenProcess`**

![](https://bbs.kanxue.com/upload/attach/202208/939298_V8GVS4BUDWZWS3X.jpg)

所有进程在R3和R0共享同一个结构`_KUSER_SHARED_DATA`

- R3地址为：`7FFE0000`
- R0地址为：`FFFE0000`

```
//R3
kd> dt _KUSER_SHARED_DATA 7FFE0000
   +0x300 SystemCall       : 0x772b70b0

//R0
kd> dt _KUSER_SHARED_DATA FFDF0000
   +0x300 SystemCall       : 0x772b70b0
        
kd> u 0x772b70b0
ntdll!KiFastSystemCall:
772b70b0 8bd4            mov     edx,esp
772b70b2 0f34            sysenter
ntdll!KiFastSystemCallRet:
772b70b4 c3              ret
772b70b5 8da42400000000  lea     esp,[esp]
772b70bc 8d642400        lea     esp,[esp]
ntdll!KiIntSystemCall:
772b70c0 8d542408        lea     edx,[esp+8]
772b70c4 cd2e            int     2Eh
772b70c6 c3              ret
```

**`KiFastSystemCall`**

![](https://bbs.kanxue.com/upload/attach/202208/939298_9ARMK249QBK3ZF6.jpg)

****API实现OpenProcess功能****

![](https://bbs.kanxue.com/upload/attach/202208/939298_DYC96E5QDQFNW4P.jpg)

**main.cpp**

```
#include <Windows.h>
#include <winternl.h>

typedef struct _CLIENT_ID
{
	HANDLE UniqueProcess;
	HANDLE UniqueThread;
}CLIENT_ID, *PCLIENT_ID; 

//ZwOpenProcess函数原型
typedef NTSTATUS (WINAPI *ZwOpenProcessProc)
(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, 
	POBJECT_ATTRIBUTES ObjectAttributes, PCLIENT_ID ClientId);

int main()
{
	HMODULE hModule = LoadLibraryA("ntdll.dll");
	PUCHAR temp = (PUCHAR)GetProcAddress(hModule, "ZwOpenProcess");
	ULONG size = 0;
	for (int i = 0; i < 100; i++)
	{
		if (temp[i] == 0xc2)    
		{
			size = i+ 2;	//ZwOpenProcess函数结尾
			break;
		}
	}

	ZwOpenProcessProc func =(ZwOpenProcessProc)VirtualAlloc(NULL, 0x1000, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(func, temp, size);

	HANDLE outProcess = NULL;
	CLIENT_ID client = {0};
	client.UniqueProcess = (HANDLE)5776;    //要打开的进程pid
	OBJECT_ATTRIBUTES obattr = {0};
	obattr.Length = sizeof(OBJECT_ATTRIBUTES);
	NTSTATUS status = func(&outProcess, PROCESS_ALL_ACCESS, &obattr, &client);

    return 0;
}
```

### 3.2.系统调用进内核

**1.在 Ring3 的代码调用了 sysenter 指令之后，CPU 会做出如下的操作：**

```
1． 将 SYSENTER_CS_MSR 的值装载到 cs 寄存器    174
2． 将 SYSENTER_EIP_MSR 的值装载到 eip 寄存器  176
3． 将 SYSENTER_CS_MSR 的值加 8（Ring0 的堆栈段描述符）装载到 ss 寄存器。
4． 将 SYSENTER_ESP_MSR 的值装载到 esp 寄存器 175
5． 将特权级切换到 Ring0
6． 如果 EFLAGS 寄存器的 VM 标志被置位，则清除该标志
7． 开始执行指定的 Ring0 代码
```

**2.查看msr寄存器**

```
kd> rdmsr 174
msr[174] = 00000000`00000008		//CS

kd> rdmsr 176
msr[176] = 00000000`83e7a760		//EIP

kd> u 83e7a760
nt!KiFastCallEntry:
83e7a760 b923000000      mov     ecx,23h
83e7a765 6a30            push    30h
83e7a767 0fa1            pop     fs
83e7a769 8ed9            mov     ds,cx
83e7a76b 8ec1            mov     es,cx
83e7a76d 648b0d40000000  mov     ecx,dword ptr fs:[40h]
83e7a774 8b6104          mov     esp,dword ptr [ecx+4]
83e7a777 6a23            push    23h

kd> rdmsr 175
msr[175] = 00000000`80790000		//ESP

//读写MSR的方式，只能在0环，MSR寄存器属于特权指令
//1.读  eax：存低32位   edx：存高32位	
    mov ecx,0x175
    rdmsr
    
//2,写   
    mov eax,0x8402a9dc
    mov edx,0
    mov ecx,176h
    wrmsr
```

**3.`_KTRAP_FRAME`**

`dt _KTRAP_FRAME`：内核的上下文

```
ntdll!_KTRAP_FRAME
   +0x000 DbgEbp           : Uint4B
   +0x004 DbgEip           : Uint4B
   +0x008 DbgArgMark       : Uint4B
   +0x00c DbgArgPointer    : Uint4B
   +0x010 TempSegCs        : Uint2B
   +0x012 Logging          : UChar
   +0x013 Reserved         : UChar
   +0x014 TempEsp          : Uint4B
   +0x018 Dr0              : Uint4B
   +0x01c Dr1              : Uint4B
   +0x020 Dr2              : Uint4B
   +0x024 Dr3              : Uint4B
   +0x028 Dr6              : Uint4B
   +0x02c Dr7              : Uint4B
   +0x030 SegGs            : Uint4B
   +0x034 SegEs            : Uint4B
   +0x038 SegDs            : Uint4B
   +0x03c Edx              : Uint4B
   +0x040 Ecx              : Uint4B
   +0x044 Eax              : Uint4B
   +0x048 PreviousPreviousMode : Uint4B
   +0x04c ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD
   +0x050 SegFs            : Uint4B
   +0x054 Edi              : Uint4B
   +0x058 Esi              : Uint4B
   +0x05c Ebx              : Uint4B
   +0x060 Ebp              : Uint4B
   +0x064 ErrCode          : Uint4B
   +0x068 Eip              : Uint4B
   +0x06c SegCs            : Uint4B
   +0x070 EFlags           : Uint4B
   +0x074 HardwareEsp      : Uint4B
   +0x078 HardwareSegSs    : Uint4B
   +0x07c V86Es            : Uint4B
   +0x080 V86Ds            : Uint4B
   +0x084 V86Fs            : Uint4B
   +0x088 V86Gs            : Uint4B
```

`dt _KPCR` 每个cpu逻辑核的状态

```
kd> dt _KPCR
ntdll!_KPCR
   +0x000 NtTib            : _NT_TIB
   +0x000 Used_ExceptionList : Ptr32 _EXCEPTION_REGISTRATION_RECORD
   +0x004 Used_StackBase   : Ptr32 Void
   +0x008 Spare2           : Ptr32 Void
   +0x00c TssCopy          : Ptr32 Void
   +0x010 ContextSwitches  : Uint4B
   +0x014 SetMemberCopy    : Uint4B
   +0x018 Used_Self        : Ptr32 Void
   +0x01c SelfPcr          : Ptr32 _KPCR
   +0x020 Prcb             : Ptr32 _KPRCB
   +0x024 Irql             : UChar
   +0x028 IRR              : Uint4B
   +0x02c IrrActive        : Uint4B
   +0x030 IDR              : Uint4B
   +0x034 KdVersionBlock   : Ptr32 Void
   +0x038 IDT              : Ptr32 _KIDTENTRY
   +0x03c GDT              : Ptr32 _KGDTENTRY
   +0x040 TSS              : Ptr32 _KTSS           //fs:40的位置为 TSS
   +0x044 MajorVersion     : Uint2B
   +0x046 MinorVersion     : Uint2B
   +0x048 SetMember        : Uint4B
   +0x04c StallScaleFactor : Uint4B
   +0x050 SpareUnused      : UChar
   +0x051 Number           : UChar
   +0x052 Spare0           : UChar
   +0x053 SecondLevelCacheAssociativity : UChar
   +0x054 VdmAlert         : Uint4B
   +0x058 KernelReserved   : [14] Uint4B
   +0x090 SecondLevelCacheSize : Uint4B
   +0x094 HalReserved      : [16] Uint4B
   +0x0d4 InterruptMode    : Uint4B
   +0x0d8 Spare1           : UChar
   +0x0dc KernelReserved2  : [17] Uint4B
   +0x120 PrcbData         : _KPRCB
```

**5.`KUSER_SHARED_DATA`**

```
kd> dt _KUSER_SHARED_DATA
ntdll!_KUSER_SHARED_DATA
   +0x304 SystemCallReturn : Uint4B
```

**6.`KUSER_SHARED_DATA FFDF0000`**

304位置是 SystemCallReturn

```
kd> dt _KUSER_SHARED_DATA FFDF0000
ntdll!_KUSER_SHARED_DATA
   +0x304 SystemCallReturn : 0x76de70b4
```

**7.ntdll.dll --> ZwOpenProcess**

![](https://bbs.kanxue.com/upload/attach/202208/939298_H62VFEBZTRZA4NV.jpg)

****8.ntdll.dll --> KiFastSystemCall****

![](https://bbs.kanxue.com/upload/attach/202208/939298_ABK4YKXET34QWE2.jpg)

******9.ntoskrnl.exe -->kiFastCallEntry******

![](https://bbs.kanxue.com/upload/attach/202208/939298_6T7JNF7ZKKXTKMH.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_MJTTS5ZZNWXP3F2.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_87X5Q6BAQS852T5.jpg)

**10.SSDT表**

`KeServiceDescriptorTable`

- `ServiceTable`：指向函数地址表，`KeServiceDescriptorTable+服务号*4 = 函数地址`
- `Count`：系统服务表被调用的次数
- `ServiceLimit`：函数数量
- `ArgmentTable`：函数参数表，每个参数4个字节.获取字节数后除以4，得到函数参数个数
