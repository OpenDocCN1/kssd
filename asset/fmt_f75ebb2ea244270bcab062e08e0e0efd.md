
`0x90000247(0x02E3F91C) ---> 0x01018E7D` `LocalAlloc`  
`0x90000247(0x02E3F91C) ---> 0x01018E7D` `LocalAlloc`  
`0x90000247(0x02E3F91C) ---> 0x01018E7D` `LocalAlloc`  
`0x90000051(0x02E3F91C) ---> 0x01030B80` `CreateFileMappingW`  
`0x90000241(0x02E3F91C) ---> 0x01030B29` `LoadLibraryA`  
`0x90000241(0x02E3F91C) ---> 0x01030B29` `LoadLibraryA`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000241(0x02E3F91C) ---> 0x01030B29` `LoadLibraryA`  
`0x90000052(0x02E3F91C) ---> 0x01030B80` `CreateFileW`  
`0x90000052(0x02E3F91C) ---> 0x01030B80` `CreateFileW`  
`0x90000197(0x02E3F91C) ---> 0x01030B29` `GetProcAddress`  
`0x90000241(0x02E3F91C) ---> 0x01030B29` `LoadLibraryA`  
`0x90000051(0x02E3F91C) ---> 0x01030B80` `CreateFileMappingW`  
`0x90000051(0x02E3F91C) ---> 0x01030B80` `CreateFileMappingW`  
`0x90000197(0x02E3F91C) ---> 0x01030B29` `GetProcAddress`  
`0x90000038(0x02E3F91C) ---> 0x01030B80` `CompareStringW`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x900001D1(0x02E3F91C) ---> 0x01018FDC` `GetTickCount`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x900000D0(0x02E3F91C) ---> 0x01030B29` `FindFirstFileA`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000592(0x02E3F91C) ---> 0x01030B29` `MessageBoxA` `-->@ payload`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000371(0x02E3F91C) ---> 0x01030B29` `VirtualProtect`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x90000031(0x02E3F91C) ---> 0x01030B80` `CloseHandle`  
`0x9000006C(0x02E3F91C) ---> 0x010315AC` `CreateThread`  
`0x90000175(0x02E3F91C) ---> 0x01006AF7` `GetModuleHandleA`  
`... //转向宿主程序调用`

这样配合前面提到过的被感染程序 `IID` 方面的信息，就可以检测出当前程序是否被感染 `Win32/Leon` 了。

### 5.3. 静态代码分析

由于 `Win32/Leon` 的 `payload` 仅是个 `msgbox`，如果在动态环境中在执行 `GetModuleHandleA` 前还不能完全确认其感染，此时动态可转调静态分析，静态分析开头的部分情况如下：

```
0x0     : 0x1006b63 call[0] - > __p__fmode
0x1     : 0x1006b71 call[0] - > __p__commode
0x2     : 0x1006b8b call[1] - > sub_1006d10
0x4     : 0x1006b9d call[0] - > __setusermatherr
0x5     : 0x1006ba4 call[1] - > sub_1006cfe
0x6     : 0x1006d08 call[1] - > _controlfp
0x8     : 0x1006bb3 call[0] - > _initterm
0x9     : 0x1006bd6 call[0] - > __getmainargs
0xa     : 0x1006be9 call[0] - > _initterm
0xb     : 0x1006c28 call[0] - > GetStartupInfoA
0xc     : 0x1006c4f call[1] - > sub_1002801
0xd     : 0x1002809 call[1] - > GetCommandLineW
0xe     : 0x1002818 call[1] - > GetSystemMetrics
0xf     : 0x100281f call[1] - > GetProcAddress
0x10    : 0x1002838 call[2] - > sub_1001b4a
0x11    : 0x1001b75 call[2] - > CharNextW
0x13    : 0x1002844 call[2] - > sub_10041ca
0x14    : 0x10041d5 call[2] - > RegisterWindowMessageW
0x15    : 0x10041fe call[2] - > GetDC
0x16    : 0x100420f call[3] - > sub_1003d57
0x17    : 0x1003d5b call[4] - > sub_1003cf2
0x18    : 0x1003d15 call[4] - > LoadStringW
0x19    : 0x1003d2d call[4] - > LocalFree
0x1a    : 0x1003d40 call[4] - > LocalAlloc
0x1b    : 0x1003d5b call[4] - > sub_1003cf2
0x1c    : 0x1003d70 call[4] - > LocalAlloc
0x1d    : 0x1003d81 call[4] - > LocalSize
0x1e    : 0x1003d9c call[4] - > LoadStringW
0x1f    : 0x1003dbb call[4] - > lstrcpynW
0x20    : 0x1003e04 call[4] - > MessageBoxW
0x21    : 0x1003e3f call[5] - > sub_1003c15
0x22    : 0x1003c28 call[5] - > CharUpperW
0x24    : 0x1003e5d call[4] - > GetSystemMenu
0x25    : 0x1003e73 call[4] - > LoadAcceleratorsW
...
```

可以看到，这是一个标准 `VC++` 编写的程序开头，同时出现了在病毒程序中一般不会出现的，窗口类,字体，图形类 `API`，可以肯定 `GetModuleHandleA` 前开头的代码可能是宿主程序

被加壳后的代码，或是被病毒感染后的结果。

总结一下检测的依据：

1. 被感染程序出现同名 `advapi.dll` 的 `IID`，同时导入了 `cryptapi`。再通过壳特征库，排除壳的可能。
2. 代码段扫描，发现 `crypt API` 固定序列，`advapi.dll` 导入函数不是完全被调用。
3. 动态代码分析下，出现 `crypt API` 固定序列，在调用 `CryptDecrypt` 解密结束后，存在大的跨段跳转。
4. 动态代码分析下，出现查找当前目录文件，依据条件感染文件，执行 `payload`，此时动态分析可以确定病毒，停止分析。
5. 跳向宿主程序，静态分析出现病毒少见的 `API` 调用（窗口类,字体，图形类 `API`）情况，该项依据被感染的宿主程序而定，是补充条件。

这样再配合被感染程序自身的文件特性，就可综合的得出被 `Win32/Leon` 感染的结论了。

## [0x06]. 其他

`Win32/Leon` 使用的隐形解密技术是对 `AVs` 有效的一个反击，而且它的规则化引擎也非常有趣，但对反病毒研究人员来说，强大的动态代码分析能力，仍然是对抗这类复杂病毒的最有效武器。

**附参考文献:**

[1] kaze. 《Stealth api-based decryptor》
