

# 使用 CloseHandle 欺骗和对抗调试器

> 原文：[`www.pediy.com/kssd/pediy12/127358.html`](https://www.pediy.com/kssd/pediy12/127358.html)

我最近调试程序才发觉，CloseHandle 这个函数的一个怪异的行为。我突然想到可以利用这个行为来欺骗或者对抗调试器。  
    这个怪异的行为是这样的，调用 CloseHandle 释放一个无效句柄，如果进程在调试器之外，那么函数返回 FALSE，而 GetLastError 得到 ERROR\_INVALID\_HANDLE；但是如果进程在调试器内，那么系统将抛出异常 C0000008H。  
  
**一、首先说如何得到一个无效句柄呢？**  
  
方法一：一个句柄释放多次次  
    这个方法看似没问题，但实际有很大的隐患。因为新创建的内核对象是总是寻找句柄表的空白记录，因此有可能存在已经释放过的句柄又被利用，而这时再次释放会很不安全。  
  
方法二：硬编码一个足够大的句柄值  
    用 procexp.exe 看一下就会发现进程里的句柄值很少大于 1000h，因此赌一下也是可以的  
  
方法三：释放一个不可关闭的句柄  
   这个我认为是最安全的一种方法，首先自己随便调用一个 CreateXXX 得到一个句柄 h1，再调用 SetHandleInformation 将句柄 h1 设为不可关闭的，最后调用 CloseHandle 释放它就行了  
  
**二、再说如何利用这个行为**  
  
方法一：利用 SEH 欺骗  
     这样利用 CloseHandle 根据调试器抛出异常，就可以通过 SEH 使得程序内外有不同的行为  
  
方法二：隐式调用 CloseHandle 引发调试异常  
    关于方法范围比较广泛，大家八仙过海各显神通吧  
  
偶然发现而已，文章发的比较匆忙，现在只是发现 FindClose 有相同的现象，我估计很多带 Close 字段都有这种现象,也不知道其它 API 是否有相似的现象，希望大家不吝指教。 

from ob\\Obclose.c  
if (PreviousMode != KernelMode) {  
  
                if ((NtGlobalFlag & \*\*\*\_ENABLE\_CLOSE\_EXCEPTIONS) ||  
                    (CurrentProcess->DebugPort != NULL) ||  
                    (ObjectTable->DebugInfo != NULL)) {  
  
                    if (!KeIsAttachedProcess()) {  
                        return KeRaiseUserException (STATUS\_INVALID\_HANDLE);  
                    } else {  
                        return STATUS\_INVALID\_HANDLE;  
                    }  
  
                } 

code from fengyue.sys  

代码:

```

NTSTATUS 
Hooked\_NtClose(  IN HANDLE Handle)
{
  NTSTATUS status;
  PVOID  pObject = NULL;

  if(!bInBlackListByEProcess(PsGetCurrentProcess()))  //不是黑名单
    return ((pfnNtClose)fiNtClose.uRealAddr)(Handle);

  if(NT\_SUCCESS(ObReferenceObjectByHandle(Handle, GENERIC\_READ, 0, KernelMode, &pObject, NULL)))
  {
    ObDereferenceObject(pObject);
    status = ((pfnNtClose)fiNtClose.uRealAddr)(Handle);
  }
  else
  {
    status = STATUS\_INVALID\_HANDLE;
  }
  return status;
}

```