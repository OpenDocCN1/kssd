

# ShellCode 编写之 hash 式函数调用及相关

> 原文：[`www.pediy.com/kssd/pediy10/58393.html`](https://www.pediy.com/kssd/pediy10/58393.html)

前段时间的 Exploit Me 挑战赛，使我深刻的感到 ShellCode 编写完全不同与病毒，对于栈空间得按字节来省着用，ShellCode 中 hash 式函数调用势在必行。另外，在 unicode 环境下的 shell 处理，也得过关才行。参照《The Shellcoder's Handbook》 中关于“百叶窗”法，及相关网文资料，自己也写了拆分 ShellCode 字符串模块，在此一并放出![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
   CreateWinAPINameHash 功能是获取指定模块中函数名称串的 HASH 值,并返回 Hash 的 Key 值，要计算实际中函数名称串的 HASH 值只需要调用次过程就可以；ShellCodeToStdByteChar 功能是对于已经写好的 ShellCode，将其中的非纯数字和字母字节进行拆分，使之转换为字母字节。另外，GetBaseAddressOfKernel32ByTEB 与 GetKernel32BaseAddressBySearch 是两种不同方法获取 Kernel32.dll 的基址的子过程，以及 GetStringSize 用于获取字符串大小的子程序，是顺便送给大家公用子程序，省的自己写![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   

代码:

```
.586
.model  flat, stdcall
option  casemap:none

include windows.inc
include kernel32.inc
include user32.inc                                                  
includelib kernel32.lib
includelib user32.lib

    .const
szLoadLibraryA    db  'LoadLibraryA',0
szGetProcAddressA  db  'GetProcAddressA',0
szGetProcAddresAs  db  'GetProcAddresAs',0
szFileName    db  ".\\ShellCode.log",0
szkernel32Dll    db  "kernel32.Dll",0

    .data
ppKernel32Win32ApiName  dd  szLoadLibraryA,szGetProcAddressA,szGetProcAddresAs  

    .data?
pKernel32Win32ApiHash  dd  lengthof ppKernel32Win32ApiName dup  (?)

    .code
start:
  call  GetBaseAddressOfKernel32ByTEB

  push  -1
  push  -1
  push  lengthof ppKernel32Win32ApiName
  push  offset pKernel32Win32ApiHash
  push  offset ppKernel32Win32ApiName
  push  eax
  call  CreateWinAPINameHash
  xor  eax, eax
  ret
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
;如何编写 Unicode 形式的 ShellCode 及过 WideCharToMultiByte
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;--------------------------------------CreateWinAPINameHash---------------------------------------------
;函数功能：  获取指定模块中函数名称串的 HASH 值,并返回 Hash 的 Key 值
;入口参数：  hModule    :  被 Hash 函数所在模块的句柄或基址
;       lppWinApiName  :  函数名称串地址表首地址
;       lpHashCode    :  接收函数名称串被 Hash 后的缓冲区地址
;       dwNumberString  :  函数名称串的个数
;            dwHashKey          :  可以预设 Hash 的 Key 值的范围的上限(-1 则为系统寻找合适 KEY 值)\[0--31\]
;            dwFilterFlags      :  设置 Hash 值过滤标志:
;                                  0--不含零字节;1--不含连续的零字节;-1--不过滤
;出口参数：  成功则 EAX 返回正确 Hash 的 Key 值,否则为返回值小于零.           
;-------------------------------------------------------------------------------------------------------
CreateWinAPINameHash  proc  hModule:dword,lppWinApiName:dword,lpHashCode:dword,\\
                                dwNumberString:dword,dwHashKey:dword,dwFilterFlags:dword
local  ret\_value:dword  
  pushad
  mov  ebx, hModule
  mov  eax, ebx
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 从 PE 文件头的数据目录获取导出表地址
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  add  eax, dword ptr \[ebx + IMAGE\_DOS\_HEADER.e\_lfanew\]
  assume  eax : ptr IMAGE\_NT\_HEADERS
  mov  eax, \[eax\].OptionalHeader.DataDirectory.VirtualAddress
  add  eax, ebx
  assume  eax : ptr IMAGE\_EXPORT\_DIRECTORY
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 计算函数名的 hash 值
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  mov  ecx, dword ptr \[eax\].NumberOfNames  ;检查是否搜索完毕
  mov  eax, dword ptr \[eax\].AddressOfNames  ;取出函数名字符串地址表指针(RVA)
  add  eax, ebx        ;eax 真实指向函数名字符串地址表首地址
  push  ecx
  push  eax
  shl  ecx, 2
  push  ecx
  push  LMEM\_ZEROINIT
  call  LocalAlloc        ;申请临时内存
  pop  esi
  pop  ecx
  test  eax, eax
  je  exit\_CreateWinAPINameHash
  mov  edx, eax        ;检测执行结果
  
  mov  eax, dwHashKey
  mov  edi, -1
  cmp  eax, edi
  jne  @f
  mov  al, 32
@@:
  push  eax
  jmp  @f
next\_CreateWinAPINameHash:
  push  edi
@@:
  push  ecx
  push  edx
  push  esi
  push  ebx
@@:
  call  HashWinAPIString
  cmp  eax, 0
  jl  @f
  cmp  eax, edi
  je  @f
  xchg  eax, edi
  cmp  edi, eax
  jl  next\_CreateWinAPINameHash
  push  edi
  push  dwNumberString
  push  lpHashCode
  push  lppWinApiName
  push  dwFilterFlags
  jmp  @b
@@:
  push  eax
  push  edx
  call  LocalFree      ;释放临时内存
  pop  eax
exit\_CreateWinAPINameHash:
  mov  ret\_value, eax
  popad
  mov  eax, ret\_value
  ret
CreateWinAPINameHash  endp
;--------------------------------------HashWinAPIString-------------------------------------------------
;函数功能：  将函数名称串进行 HASH 计算
;入口参数：  lpBaseIED    :  被 Hash 函数所在模块的 IMAGE\_EXPORT\_DIRECTORY 指针
;       lpStringTable  :  函数名称串地址表首地址
;       lpHashCode    :  接收函数名称串被 Hash 后的缓冲区地址
;       dwNumberString  :  函数名称串的个数
;            dwHashKey          :  可以预设 Hash 的 Key 值(-1 则为系统寻找合适 KEY 值)\[0--31\]
;出口参数：  成功则 EAX 返回正确 Hash 的 Key 值,否则为-1.             
;-------------------------------------------------------------------------------------------------------
HashWinAPIString  proc  lpBaseIED:dword,lpStringTable:DWORD,lpHashCode:dword,\\
                                dwNumberString:dword,dwHashKey:dword
local  hash\_key:byte,string\_table\_base:dword
  pushad
  mov  eax, dwHashKey
  inc  eax
  test  eax, eax
  jne  @f
  mov  al, 32
@@:
  dec  eax
  mov  hash\_key, al
  mov  ebx, lpBaseIED
hash\_key\_HashWinAPIString:
  mov  edi, lpHashCode
  mov  ecx, dwNumberString
  push  ebx
  push  ecx
  push  edi          ; call CheckSameDwordByte
  dec  ebx
  je  @f
  inc  ebx
  jge  @f
  inc  ebx
@@:
  push  lpStringTable
  pop  string\_table\_base 
hash\_next\_HashWinAPIString:
  push  ecx
  xor  eax, eax
  cdq
  mov  esi, string\_table\_base
  mov  esi, dword ptr \[esi\]      ;取出函数名字符串地址表第一个单元内容(函数名字符串 RVA)
  add  esi, ebx        ;esi 真实指向函数名字符串
;----------------------hash--------------------------
@@:
  lodsb
  mov  cl, hash\_key
  ror  edx, cl
  add  edx, eax
  test  al, al
  jne  @b
;----------------------------------------------------
  mov  eax, edx
  stosd
  add  string\_table\_base, 4
  pop  ecx
  loop  hash\_next\_HashWinAPIString

  call  CheckSameDwordByte
  test  eax, eax
  je  @f
  dec  hash\_key
  jge  hash\_key\_HashWinAPIString
@@:
  popad
  movzx  eax, hash\_key
  ret
HashWinAPIString  endp
;--------------------------------------CheckSameDwordByte-----------------------------------------------
;函数功能：  检验 HASH 值的合法性
;入口参数：  lpBuffer    :  需要检验的数据缓冲区首地址
;       dwLengthOfType  :  被检数据的长度(以字位单位计算 DWORD)
;       dwFlags    :  设定检验标志: 0,则检验是否非法 ASCII 码 0;
;                                                1,则检验是否非法 Unicode 码 0;
;                                                -1,则不检验是否非法;
;                                                否则检验是否含重复字.
;出口参数：  成功则 EAX 返回 0, 否则为 EAX≠0              
;-------------------------------------------------------------------------------------------------------
CheckSameDwordByte  proc  lpBuffer:dword, dwLengthOfType:dword, dwFlags:dword
  push  ecx
  push  edi
  push  esi
  mov  esi, lpBuffer
  mov  ecx, dwLengthOfType
  cld
  xor  eax, eax
  push  eax
  dec  eax
  cmp  dwFlags, eax
  pop  eax
  je  ret\_CheckSameDwordByte
  mov  edi, esi
  cmp  dwFlags, eax
  je  ASCII\_CheckSameDwordByte
  cmp  dwFlags, 1
  je  Unicode\_CheckSameDwordByte
  dec  ecx
@@:
  push  ecx
  lodsd
  mov  edi, esi
  repne  scasd
  test  ecx, ecx
  jne  @f
  pop  ecx
  loop  @b
  push  ecx
@@:
  pop  ecx
  jmp  exit\_CheckSameDwordByte
ASCII\_CheckSameDwordByte:
  shl  ecx, 2
  repne  scasb
  jmp  exit\_CheckSameDwordByte
Unicode\_CheckSameDwordByte:
  lodsd
  ror  eax, 8
  test  ax, ax
  je  exit\_CheckSameDwordByte
  loop  Unicode\_CheckSameDwordByte
exit\_CheckSameDwordByte:
  mov  eax, ecx
ret\_CheckSameDwordByte:
  pop  esi
  pop  edi
  pop  ecx
  ret
CheckSameDwordByte  endp
;----------------------------------ShellCodeToStdByteChar------------------------------
;函数功能：  拆分 ShellCode 字符串,使之转换为纯数字和字母字符
;入口参数：  lpStdByteCharStr  :  需要处理的 ShellCode 字符串地址
;       lpShellCodeCharStr :  转换后的接收缓冲区
;出口参数：  EAX 返回转换后的字符串长度(BYTE)                  
;--------------------------------------------------------------------------------------
ShellCodeToStdByteChar  proc  lpStdByteCharStr:dword,lpShellCodeCharStr:dword
  push  ebx
  push  ecx
  push  esi
  push  edi

  mov  esi, lpStdByteCharStr
  mov  edi, lpShellCodeCharStr
  push  edi
  xor  eax, eax
  mov  ecx, eax
  dec  ecx
  cld
  repnz  scasb
  not  ecx
  dec  ecx        ; 获取待处理串长度

  mov  ebx, esp
  cld
next\_ShellCodeToStdByteChar:
  lodsb
  cmp  al, 30h
  jl  must\_ShellCodeToStdByteChar
  cmp  al, 7ah
  jg  must\_ShellCodeToStdByteChar
  cmp  al, 61h
  jge  @f
  cmp  al, 5ah
  jg  must\_ShellCodeToStdByteChar
  cmp  al, 41h
  jge  @f
  cmp  al, 3ah
  jge  must\_ShellCodeToStdByteChar
@@:
  stosb          ; 对数字,字母不需要拆分
  jmp  continue\_ShellCodeToStdByteChar
must\_ShellCodeToStdByteChar:
  push  eax
@@:
  and  al, 1111b      ; 拆分: 将一个字节的高低四位分别扩展到
  add  al, 41h        ; 两个字节中,加 41h 后存储
  stosb
  pop  eax
  shr  al, 4
  cmp  ebx, esp
  je  @b
  push  eax
continue\_ShellCodeToStdByteChar:
  loop  next\_ShellCodeToStdByteChar
  xor  eax, eax
  lodsb          ; 添加串结束符
  pop  eax
  xchg  eax, edi
  sub  eax, edi      ; 统计转换后串长度

  pop  edi
  pop  esi
  pop  ecx
  pop  ebx
  ret
ShellCodeToStdByteChar  endp
;--------------------------------GetBaseAddressOfKernel32ByTEB-----------------------------------
;函数功能：  根据 TEB 结构定位 Kernel32.dll 的基址
;入口参数：  无
;出口参数：  成功则返回 Kernel32.dll 的基址在 EAX 中             
;------------------------------------------------------------------------------------------------
GetBaseAddressOfKernel32ByTEB  proc
  assume  fs:nothing
  xor  eax, eax
  mov  eax, dword ptr fs:\[eax+30h\]
  mov  eax, dword ptr \[eax+0ch\]
  mov  eax, dword ptr \[eax+1ch\]
  mov  eax, dword ptr \[eax\]
  mov  eax, dword ptr \[eax+8\]
  ret
GetBaseAddressOfKernel32ByTEB  endp
;-----------------------------------GetKernel32BaseAddressBySearch--------------------------------------
;函数功能：  在内存中扫描 Kernel32.dll 的基址
;入口参数：  lpKernel32SpaceOfAny  :  Kernel32.dll 领空内的任一地址
;出口参数：  成功则返回 Kernel32.dll 的基址在 EAX 中,否则为 EAX=0                  
;-------------------------------------------------------------------------------------------------------
GetKernel32BaseAddressBySearch    proc  lpKernel32SpaceOfAny : DWORD    
  mov  eax,lpKernel32SpaceOfAny      
  xor  edx,edx
@@:
  dec  eax               ;暴力查找 Kernel32.dll 的基地址
  mov  dx,word ptr \[eax+IMAGE\_DOS\_HEADER.e\_lfanew\]
  test  dx,0f000h
  jnz  @b
  cmp  eax,\[eax+edx+IMAGE\_NT\_HEADERS.OptionalHeader.ImageBase\]
  jnz  @b
  ret  4  
GetKernel32BaseAddressBySearch   endp
;--------------------------------------------GetStringSize---------------------------------------------
;函数功能：  用于获取字符串大小的子程序
;入口参数：  lpOutTexts    :  需要检测字符串的地址  
;出口参数：  eax 字符串大小
;------------------------------------------------------------------------------------------------------
GetStringSize  proc  lpStringBuffer:dword
  push  ecx
  push  edi
  mov  edi, lpStringBuffer
  xor  eax, eax
  mov  ecx, eax
  dec  ecx
  cld
  repnz  scasb
  not  ecx
  mov  eax, ecx
  dec  eax
  pop  edi
  pop  ecx
  ret
GetStringSize  endp
end  start

```

        以上个功能模块的使用注释中写的很明白，如同 MSDN，在此就不重复说明了。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || HashCall v 2.20.rar  
 |