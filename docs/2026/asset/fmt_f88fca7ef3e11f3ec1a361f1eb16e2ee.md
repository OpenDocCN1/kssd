
# 一款木马的分析[原创]

一款木马的分析

by nbw

4 NE365

为了配合kanxue老大倡议的多发文，加上近来有些乏，找个别的东西分析一下耍耍。随便从网上下了个木马生成器，是个所谓的下载者，就是Downloader。

名字叫什么 木马基地八一极限下载者，木马么，都喜欢搞得比较牛的样子，其实都比较龌龊，技术和人品一样龌龊。

附件是木马生成器，建议先运行一下看看，生成器无毒。

我下面分析的是这个生成器产生的木马，配置信息就用默认的。

## 木马第一次运行

### 1.读取配置信息

```asm
//第一个函数,Delphi编译器产生的用来初始化异常处理之类，比较经典。
0040464B  |.  E8 88F6FFFF   call    00403CD8
0040440E  |.  E8 D9F9FFFF   call    <jmp.&kernel32.GetSystemDirector>; \GetSystemDirectoryA
//这里打开自身文件
004041A9  |.  E8 FEFBFFFF   call    <jmp.&kernel32.CreateFileA>       ; \CreateFileA
//读取文件最后面4个字节
004041B5  |.  6A 02         push    2                                 ; /Origin = FILE\_END
004041B7  |.  6A 00         push    0                                 ; |pOffsetHi = NULL
004041B9  |.  6A FC         push    -4                                ; |OffsetLo = FFFFFFFC (-4.)
004041BB  |.  53            push    ebx                               ; |hFile
004041BC  |.  E8 6BFCFFFF   call    <jmp.&kernel32.SetFilePointer>    ; \SetFilePointer
004041C1  |.  6A 00         push    0                                 ; /pOverlapped = NULL
004041C3  |.  8D45 F8       lea     eax, [ebp-8]                      ; |
004041C6  |.  50            push    eax                               ; |pBytesRead
004041C7  |.  6A 04         push    4                                 ; |BytesToRead = 4
004041C9  |.  8D45 F4       lea     eax, [ebp-C]                      ; |
004041CC  |.  50            push    eax                               ; |Buffer
004041CD  |.  53            push    ebx                               ; |hFile
004041CE  |.  E8 49FCFFFF   call    <jmp.&kernel32.ReadFile>          ; \ReadFile
//将结果异或以下,象征性加过密
004041D3  |.  8175 F4 697A6>xor     dword ptr [ebp-C], 4D617A69
//根据该结果重新定位文件指针地址
004041EB  |.  8B45 F4       mov     eax, [ebp-C]                      ; |
004041EE  |.  F7D8          neg     eax                               ; |
004041F0  |.  50            push    eax                               ; |OffsetLo
004041F1  |.  53            push    ebx                               ; |hFile
004041F2  |.  E8 35FCFFFF   call    <jmp.&kernel32.SetFilePointer>    ; \SetFilePointer
//读取开始设置的配置信息
00404212  |.  50            push    eax                               ; |Buffer
00404213  |.  53            push    ebx                               ; |hFile
00404214  |.  E8 03FCFFFF   call    <jmp.&kernel32.ReadFile>          ; \ReadFile
0040421A  |.  E8 85FBFFFF   call    <jmp.&kernel32.CloseHandle>       ; \CloseHandle
```

### 2.创建DLL文件

```asm
//试图删除文件"C:\Program Files\Common Files\Microsoft Shared\MSINFO\InfoMs.dlt"
004046D9  |.  E8 D6F6FFFF   call    <jmp.&kernel32.DeleteFileA>       ; \DeleteFileA
如果删除不了,则设置该文件为下次开机后由系统删除。
//查找资源
004040A6  |.  53            push    ebx                               ; /ResourceType
004040A7  |.  56            push    esi                               ; |ResourceName
004040A8  |.  A1 50664000   mov     eax, [406650]                     ; |
004040AD  |.  50            push    eax                               ; |hModule => 00400000 (Hack169)
004040AE  |.  E8 09FDFFFF   call    <jmp.&kernel32.FindResourceA>     ; \FindResourceA
004040BD  |.  57            push    edi                               ; /hResource
004040BE  |.  A1 50664000   mov     eax, [406650]                     ; |
004040C3  |.  50            push    eax                               ; |hModule => 00400000 (Hack169)
004040C4  |.  E8 3BFDFFFF   call    <jmp.&kernel32.LoadResource>      ; \LoadResource
004040D3  |.  56            push    esi                               ; /hResource
004040D4  |.  E8 33FDFFFF   call    <jmp.&kernel32.LockResource>      ; \LockResource
//创建文件："C:\Program Files\Common Files\Microsoft Shared\MSINFO\InfoMs.dlt"
004040F5  |.  E8 B2FCFFFF   call    <jmp.&kernel32.CreateFileA>       ; \CreateFileA
00404108  |.  E8 27FDFFFF   call    <jmp.&kernel32.SizeofResource>    ; \SizeofResource
//将加载的资源写入该文件
0040411B  |.  E8 1CFDFFFF   call    <jmp.&kernel32.WriteFile>         ; \WriteFile
//将上面读取得配置信息写入该文件
00404139  |.  50            push    eax                               ; |Buffer
0040413A  |.  53            push    ebx                               ; |hFile
0040413B  |.  E8 FCFCFFFF   call    <jmp.&kernel32.WriteFile>         ; \WriteFile
00404140  |.  53            push    ebx                               ; /hFile
00404141  |.  E8 DEFCFFFF   call    <jmp.&kernel32.SetEndOfFile>      ; \SetEndOfFile
00404146  |.  53            push    ebx                               ; /hObject
00404147  |.  E8 58FCFFFF   call    <jmp.&kernel32.CloseHandle>       ; \CloseHandle
//释放资源
00404154  |.  E8 73FCFFFF   call    <jmp.&kernel32.FreeResource>      ; \FreeResource
```

### 3.加载该DLL

```asm
00404735  |.  68 50484000   push    00404850                          ; /Title = "yundao"
0040473A  |.  68 58484000   push    00404858                          ; |Class = "ScrollBar"
0040473F  |.  E8 18F7FFFF   call    <jmp.&user32.FindWindowA>         ; \FindWindowA
//加载创建的DLL:"C:\Program Files\Common Files\Microsoft Shared\MSINFO\InfoMs.dlt"
0040474D  |.  E8 AAF6FFFF   call    <jmp.&kernel32.LoadLibraryA>      ; \LoadLibraryA
//获得该DLL导出的2个函数
0040475C  |.  68 64484000   push    00404864                          ; /ProcNameOrOrdinal = "MsgHookOff"
00404761  |.  53            push    ebx                               ; |hModule
00404762  |.  E8 75F6FFFF   call    <jmp.&kernel32.GetProcAddress>    ; \GetProcAddress
00404767  |.  A3 B0664000   mov     [4066B0], eax
0040476C  |.  68 70484000   push    00404870                          ; /ProcNameOrOrdinal = "MsgHookOn"
00404771  |.  53            push    ebx                               ; |hModule
00404772  |.  E8 65F6FFFF   call    <jmp.&kernel32.GetProcAddress>    ; \GetProcAddress
//为自己创建一个窗体
00403EBD  |.  E8 B6FFFFFF   call    <jmp.&user32.CreateWindowExA>     ; \CreateWindowExA
//调用上面的DLL导出的函数
004047B3  |.  FF15 B4664000 call    [4066B4]                          ;  InfoMs.MsgHookOn
//设置了一个钩子，这个钩子啥都没做，也比较艹蛋
00A35570    E8 BBEEFFFF     call    <jmp.&user32.SetWindowsHookExA>
//进入消息循环，不过循环2下就出来了
004047BB  |> /68 94664000   /push    00406694                         ; /pMsg = WM\_NULL
004047C0  |. |E8 8FF6FFFF   |call    <jmp.&user32.DispatchMessageA>   ; \DispatchMessageA
004047C5  |> |6A 00          push    0                                ; /MsgFilterMax = 0
004047C7  |. |6A 00         |push    0                                ; |MsgFilterMin = 0
004047C9  |. |6A 00         |push    0                                ; |hWnd = NULL
004047CB  |. |68 94664000   |push    00406694                         ; |pMsg = Hack169.00406694
004047D0  |. |E8 8FF6FFFF   |call    <jmp.&user32.GetMessageA>        ; \GetMessageA
004047D5  |. |85C0          |test    eax, eax
004047D7  |.^\75 E2         \jnz     short 004047BB
//调用DLL导出的MsgHookOff，里面就是把刚才下的钩子卸掉
004047D9  |.  FF15 B0664000 call    [4066B0]                          ;  InfoMs.MsgHookOff
```

### 4.注册表处理

```asm
//HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellExecuteHooks
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \RegCreateKeyExA
//"{F630F902-0922-45C8-B820-C8AF4C610C28}"
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \RegSetValueExA
//HKEY\_CLASSES\_ROOT\CLSID\{F630F902-0922-45C8-B820-C8AF4C610C28}
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \RegCreateKeyExA
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \RegSetValueExA
//HKEY\_CLASSES\_ROOT\CLSID\{F630F902-0922-45C8-B820-C8AF4C610C28}\InProcServer32"
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \RegCreateKeyExA
//buffer:C:\Program Files\Common Files\Microsoft Shared\MSINFO\InfoMs.dlt
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \RegSetValueExA
/* 
0012FF34   80000000  |hKey = HKEY\_CLASSES\_ROOT
```
