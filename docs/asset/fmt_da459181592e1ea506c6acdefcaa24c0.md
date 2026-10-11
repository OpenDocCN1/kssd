
`006892D6  ^ E2 EC loopd short XPStyle.006892C4`

我们从调试 OD 知道，IAT 在地址 `727000` 附近，在数据窗口 `CTRL+G:727000`

向下回滚，发现 IAT 开始：

`0072722C  00000000`

`00727230  7C93188A  ntdll.RtlDeleteCriticalSection`

`00727234  7C9210ED  ntdll.RtlLeaveCriticalSection`

`00727238  7C921005  ntdll.RtlEnterCriticalSection`

`0072723C  7C809FA1  kernel32.InitializeCriticalSection`

`00727240  7C809B14  kernel32.VirtualFree`

`00727244  7C809A81  kernel32.VirtualAlloc`

`00727248  7C80995D  kernel32.LocalFree`

。。。。。。。。。。。。

`00727B7C  77455011  SHELL32.RestartDialog`

`00727B80  01347550`

`00727B84  73ACFB78  avifil32.AVIStreamGetFrameClose`

`00727B88  73AC5E5F  avifil32.AVIStreamRelease`

`00727B8C  73AC5E5F  avifil32.AVIStreamRelease`

`00727B90  73AC589F  avifil32.AVIFileExit`

`00727B94  0134748C`

选择 `00727230~00727B90` 段二进制复制。

`----------------------------------------------------------------------------------------------------------`

回到调试 OD，选择 IAT 开始 `00727340~00727BA4` IAT 结束部分。

`0072733C  01347451`

`00727340  77DAEBE7  ADVAPI32.RegSetValueExA`

`00727344  77DA7883  ADVAPI32.RegQueryValueExA`

`00727348  77DCC1B5  ADVAPI32.RegQueryInfoKeyA`

`0072734C  77DA761B  ADVAPI32.RegOpenKeyExA`

`00727350  77DBB908  ADVAPI32.RegFlushKey`

`00727354  77DCC8C1  ADVAPI32.RegEnumKeyExA`

`00727358  77DAEDE5  ADVAPI32.RegDeleteValueA`

`0072735C  77DCC123  ADVAPI32.RegDeleteKeyA`

`00727360  0134AD7E`

。。。。。。。。。。。。。

`00727B90  73AC589F  avifil32.AVIFileExit`

`00727B94  0134748C`

`00727B98  73B423CA  MSVFW32.DrawDibRealize`

`00727B9C  73B441C7  MSVFW32.DrawDibOpen`

`00727BA0  73B4191A  MSVFW32.DrawDibDraw`

`00727BA4  73B49AAD  MSVFW32.DrawDibClose`

`00727BA4-00727340=864`

二进制粘贴，反汇编窗口已经补上，在数据窗口中有很多红色的改变项。

这样，参考 OD 的任务已经完成，可以关闭了。

`00406F4A    C3               retn`

`00406F4B    90               nop`

`00406F4C  - FF25 38737200    jmp dword ptr ds:[727338]    ; kernel32.GetModuleHandleA`

`00406F52    8BC0             mov eax,eax`

`00406F54  - FF25 34737200    jmp dword ptr ds:[727334]    ; kernel32.LocalAlloc`

`00406F5A    8BC0             mov eax,eax`

`00406F5C  - FF25 30737200    jmp dword ptr ds:[727330]    ; kernel32.TlsGetValue`

`00406F62    8BC0             mov eax,eax`

`00406F64  - FF25 2C737200    jmp dword ptr ds:[72732C]    ; kernel32.TlsSetValue`

`00406F6A    8BC0             mov eax,eax`

`00406F6C    50               push eax`

`00406F6D    6A 40            push 40`

`00406F6F    E8 E0FFFFFF      call XPStyle.00406F54        ; jmp to kernel32.LocalAlloc`

`---------------------------------------------------------------------------------------------------------`

现在可以修复 `Strategic Code Splicing+Nanomites Processing`。

`4dd466d0e1e0cc123afbe504da2462ca`
