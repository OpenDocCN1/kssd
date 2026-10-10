

# IceExt 分析文章系列 2. IceExt Loader 的实现原理

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-822.htm`](https://www.pediy.com/kssd/pediy08/pediy8-822.htm)

运行方法  
      
0.70  
iceext -s  
iceext -e  
  
0.67, 0.66, 0.65  
net start iceext  
net stop iceext  
  
；从 0.70 开始，IceExt 由自己的装载器而不再是系统服务管理器负责载入。  
  
  
　设计思路  
  
IceExt.sys 的映像保存在 IceExt.exe 的资源组 BINARY 的第一项中。  
  
IceExt.exe 将资源取出，保存为文件，文件名为 IceExt.sys，保存路径为系统驱动程序文件夹。  
  
然后利用 NtLoadDriver 将其作为设备驱动程序加载。  
  
  
  
  
  
    运行 IceExt 需要满足的条件  
  
\[CheckSoftICEConfig() in IceExt.c v0.70\]  
  
I.  
HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\NTice 下有键值，  
名为 KDExtensions，类型为 REG\_SZ。  
  
II.  
HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\NTice 下有键值，  
名为 KDHeapSize，类型为 REG\_DWORD，值大于或等于 0x00008000。  
  
HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\NTice 下有键值，  
名为 KDStackSize，类型为 REG\_DWORD，值大于或等于 0x00008000。 

 4dd466d0e1e0cc123afbe504da2462ca