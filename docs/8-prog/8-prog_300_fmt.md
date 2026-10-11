
# ring0 head inline hook lib

> 原文：[`www.pediy.com/kssd/pediy11/119571.html`](https://www.pediy.com/kssd/pediy11/119571.html)

没啥子技术含量，参考了海风月影的 `ring3inline` `hook` `lib` 库，主要是简化你的编程工作量，有了此库，你不需要使用一大堆的内联汇编来写那个裸函数，也不需要为了堆栈破坏问题而蓝屏，目前只支持 32 系统……

来看看到底挂钩一个函数有多简单！你可以专注的编写过滤逻辑而完全不需要理会挂钩过程

代码：

```
typedef NTSTATUS ( __stdcall *DZwCreateFile)(
  PHANDLE  FileHandle,
  ACCESS_MASK  DesiredAccess,
  POBJECT_ATTRIBUTES  ObjectAttributes,
  PIO_STATUS_BLOCK  IoStatusBlock,
  PLARGE_INTEGER  AllocationSize,
  ULONG  FileAttributes,
  ULONG  ShareAccess,
  ULONG  CreateDisposition,
  ULONG  CreateOptions,
  PVOID  EaBuffer,
  ULONG  EaLength
  );

NTSTATUS Detour_NtCreateFile(DZwCreateFile Fun, 
        PHANDLE  FileHandle,
   ACCESS_MASK  DesiredAccess,
  POBJECT_ATTRIBUTES  ObjectAttributes,
   PIO_STATUS_BLOCK  IoStatusBlock,
  PLARGE_INTEGER  AllocationSize,
   ULONG  FileAttributes,
  __in ULONG  ShareAccess,
  __in ULONG  CreateDisposition,
  __in ULONG  CreateOptions,
  __in_opt PVOID  EaBuffer,
  __in ULONG  EaLength
  )
{
  KdPrint(("NtCreateFileRoutine Called! ProcessId:%d FilePath:%ws\n",PsGetCurrentProcessId(),ObjectAttributes->ObjectName->Buffer));
  return Fun(FileHandle,DesiredAccess,ObjectAttributes,IoStatusBlock,AllocationSize,FileAttributes,ShareAccess,CreateDisposition,CreateOptions,
   EaBuffer,EaLength);
}
安装钩子只要一个函数就可完全搞定
BOOLEAN __stdcall InstallHook(PVOID FuncAddr,PVOID NewAddr,BOOLEAN IsStubPaged,HOOK_INFO* Info);
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `HookTest.rar`  

 |

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `KernelHooklib.zip`  

 |
