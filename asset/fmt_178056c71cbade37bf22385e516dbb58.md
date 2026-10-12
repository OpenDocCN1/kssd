
# 问题答案

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-122.htm`](https://www.pediy.com/kssd/pediy09/pediy09-122.htm)

为什么每个进程都有四个不同的 ID(XP SP2)? 请高手指教!

```
0496   0x0000009C  \??\C:\WINDOWS\system32\csrss.exe  
0497   0x0000009C  \??\C:\WINDOWS\system32\csrss.exe  
0498   0x0000009C  \??\C:\WINDOWS\system32\csrss.exe  
0499   0x0000009C  \??\C:\WINDOWS\system32\csrss.exe  

0520   0x0000009C  \??\C:\WINDOWS\system32\winlogon.exe  
0521   0x0000009C  \??\C:\WINDOWS\system32\winlogon.exe  
0522   0x0000009C  \??\C:\WINDOWS\system32\winlogon.exe  
0523   0x0000009C  \??\C:\WINDOWS\system32\winlogon.exe  

0564   0x0000009C  C:\WINDOWS\system32\services.exe  
0565   0x0000009C  C:\WINDOWS\system32\services.exe  
0566   0x0000009C  C:\WINDOWS\system32\services.exe  
0567   0x0000009C  C:\WINDOWS\system32\services.exe  

0576   0x0000009C  C:\WINDOWS\system32\lsass.exe  
0577   0x0000009C  C:\WINDOWS\system32\lsass.exe  
0578   0x0000009C  C:\WINDOWS\system32\lsass.exe  
0579   0x0000009C  C:\WINDOWS\system32\lsass.exe  

.......  

......  
```

请注意 上面显示的是同一个进程有四个不同的 ID 并不是一个 `EXE` 打开四次显示的

根据上面的原理 有一个神奇的关闭 `RootkitUnhooker` 的方法 就是将 `RootkitUnhooker` 的进程 `ID+1`、`ID+2`、`ID+3` 然后就能轻易地将其关闭.因为 `RootkitUnhooker` `HOOK` 了 `NtOpenProcess` 直接用原来的进程 `ID` 是无法将它的进程打开的

> `NtOpenProcess->PsLookupProcessByProcessId->ExMapHandleToPointer->ExpLookupHandleTableEntry`

```c
ExpLookupHandleTableEntry 
(
   IN PHANDLE_TABLE HandleTable, 
   IN EXHANDLE Handle 
   ) 
{ 
   ULONG i,j,k,l;     


   l = (Handle.Index >> 24) & 255; 
   i = (Handle.Index >> 16) & 255; 
   j = (Handle.Index >> 8)  & 255; 
   k = (Handle.Index)        & 255; 
 

     return &(HandleTable->Table[i][j][k]); 
} 
```

> 其中 `EXHANDLE` 结构如下:

```c
typedef struct _EXHANDLE { 
   union { 
       struct {        
           ULONG TagBits : 2; 
           ULONG Index : 30; 
       }; 
       HANDLE GenericHandleOverlay; 
   }; 
} EXHANDLE, *PEXHANDLE; 
```

> 而 `EXHANDLE.GenericHandleOverlay` = 传过来的进程 `ID`，由上可知

> `EXHANDLE.Index` = `(GenericHandleOverlay/4)` & `0x3FFFFFFF`;

> 可见 `Index` 是 `ID` 整除 4 的结果，所以就有“四个 `ID`”了。。。

 4dd466d0e1e0cc123afbe504da2462ca
