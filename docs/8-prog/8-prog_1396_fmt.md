
# inline IopCreateFile 保护文件

> 原文：[`www.pediy.com/kssd/pediy11/114719.html`](https://www.pediy.com/kssd/pediy11/114719.html)

这个思路来源于 2008 年 360 竞赛的文件保护题，要解决得问题是 `IopCreateFile` 并未导出，其函数的参数也比 `IoCreateFile` 多几个，由于我逆了其给得驱动文件，所以代码也就仿照其写的，尽量做到最大还原，下面给出完整得源代码，自己写 `make` 文件通过 `DDK` 编译一下即可：

```c
#include <ntddk.h>  
#include <windef.h>  
#include <string.h>  

ULONG OldFuncAddr;  
ULONG Pdrange;  
ULONG g_uCr0=0;  

void WPOFF();  
void WPON();  
int MyHookProc();  
void MyUnhookProc();  
ULONG GetFunctionAddr(IN PCWSTR FunctionName);  
void DriverUnload(PDRIVER_OBJECT pDriverObject);  

NTSTATUS (*Iocreatefile)(PHANDLE FileHandle,  
                          ACCESS_MASK DesiredAccess,  
                          POBJECT_ATTRIBUTES ObjectAttributes,  
                          PIO_STATUS_BLOCK IoStatusBlock,  
                          PLARGE_INTEGER AllocationSize,  
                          ULONG FileAttributes,  
                          ULONG ShareAccess,  
                          ULONG Disposition,  
                          ULONG CreateOptions,  
                          PVOID EaBuffer OPTIONAL,  
                          ULONG EaLength,  
                          CREATE_FILE_TYPE CreateFileType,  
                          PVOID InternalParameters,  
                          ULONG Options,  
                          int Esi01,  
                          int Esi02);  

void DriverUnload(PDRIVER_OBJECT pDriverObject)  
{  
    DbgPrint("Unload Called \\r\\n");  
}  

//自己函数用以替代系统 IoCreateFile 的函数  
NTSTATUS MYIoCreateFile(PHANDLE FileHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, PIO_STATUS_BLOCK IoStatusBlock, PLARGE_INTEGER AllocationSize, ULONG FileAttributes, ULONG ShareAccess, ULONG Disposition, ULONG CreateOptions, PVOID EaBuffer, ULONG EaLength, CREATE_FILE_TYPE CreateFileType, PVOID ExtraCreateParameters,  
ULONG Options,int Esi01, int Esi02)  
{          
NTSTATUS status, OringinalReturn;  
ANSI_STRING     FullName;  
POBJECT_ATTRIBUTES objectattributes;  
status   =     (Iocreatefile)(FileHandle,  
                            DesiredAccess,  
                            ObjectAttributes,  
                            IoStatusBlock,  
                            AllocationSize,  
                            FileAttributes,  
                            ShareAccess,  
                            Disposition,  
                            CreateOptions,  
                            EaBuffer,  
                            EaLength,  
                            CreateFileType,  
                            ExtraCreateParameters,  
                            Options,  
                            Esi01,  
                            Esi02);  
OringinalReturn=status;  
if (status>=0)  
{  
   RtlUnicodeStringToAnsiString(&FullName,ObjectAttributes->ObjectName,TRUE);  
   RtlUpperString(&FullName,&FullName);  
   if (strstr(FullName.Buffer,"360GAME.TXT"))  
   {  
     DbgPrint("FIle Fullname=%Z\\n",&FullName);  
    *(DWORD*)FileHandle=-1;  
     status=STATUS_ACCESS_DENIED;  
   }else  
   {  
    status=OringinalReturn;  
   }  
}  
return status;  
}  

NTSTATUS DriverEntry( IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath )  
{  
DriverObject->DriverUnload=DriverUnload;  
OldFuncAddr=(ULONG)GetFunctionAddr(L"IoCreateFile");  
DbgPrint("IoCreateFile Address=0x%8.1x\\n",OldFuncAddr);  
MyHookProc();  
return STATUS_SUCCESS;  
}  

ULONG GetFunctionAddr(IN PCWSTR FunctionName)  
{  
UNICODE_STRING HookFunctionName;  
RtlInitUnicodeString(&HookFunctionName,FunctionName);  
return (ULONG)MmGetSystemRoutineAddress(&HookFunctionName);  
}  

//Hook IoCreateFile 过程  
int MyHookProc()  
{  
KSPIN_LOCK   spinlock;  
KIRQL    oldirql;  
ULONG fiAddress,fiContent,HookAddress;  
ULONG NewOffset;  
int result;  
BOOL NotFindHook=FALSE;  
Pdrange=OldFuncAddr + 0x100;  
fiAddress=OldFuncAddr;  
if (OldFuncAddr<;Pdrange)  
{  
    while (1)  
    {  
  if (*(DWORD*)fiAddress == 0xE80875FF)  
  {  
  //后面 4 个字节的内容  
   fiContent=*(DWORD*)(fiAddress+4);         
   //找到 Hook 函数的地址  
   HookAddress=fiAddress+fiContent +8;  
     DbgPrint("address=0x%x ,Content =0x%x,Hook address=0x%x\\n",fiAddress,*(DWORD*)fiAddress,HookAddress);  
   //如果地址合法  
   if (MmIsAddressValid((PVOID)(HookAddress)))  
   break;  
   if (fiAddress >= Pdrange)  
   {  
    NotFindHook=TRUE;  
    goto continue11;  
   }  
  }  
  fiAddress++;  
    }  
    _asm{  
     mov eax,HookAddress  
     mov Iocreatefile,eax  
    }  
}  

continue11:  
         if (!NotFindHook)  
       {  
        NewOffset=(ULONG)MYIoCreateFile - (ULONG)fiAddress - 8;  
      //进行 inline Hook,加入自旋锁，防止多核访问  
   KeInitializeSpinLock(&spinlock);  
     KeAcquireSpinLock(&spinlock,&oldirql);  
     WPOFF();  
     *(DWORD*)(fiAddress +4)=NewOffset;  
     WPON();  
     KeReleaseSpinLock(&spinlock, oldirql); 
     result=1;  
    }  
   else  
   {  
    result=0;  
    }  
return result;  
}  

void WPOFF()  
{  
    _asm  
    {  
        push eax;  
        mov eax, cr0;  
        mov g_uCr0, eax;  
        and eax, 0x0FFFEFFFF; // CR0 16 BIT = 0  
        mov cr0, eax;  
        pop eax;  
        cli  
    };  
}  

void WPON()  
{  
    _asm  
    {  
        sti  
        push eax;  
        mov eax, g_uCr0; //恢原有 CR0 性  
     mov cr0, eax;  
        pop eax;  
    };  
}
```
