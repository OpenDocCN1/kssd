

# 我也发修改磁盘中的 PE 文件，增加一个 API 函数！

> 原文：[`www.pediy.com/kssd/pediy11/104709.html`](https://www.pediy.com/kssd/pediy11/104709.html)

;好像是 11 月左右写的，那时正在学习罗大侠的汇编教程，学到 PE 那里。  
  
对了添加新节，可以先计算然后再添加。这样麻烦点。而是直接把映象加大到一个固定值。  
  

代码:

```
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;添加一个新节，把导入表复制到新节中，更改导入表 rva,oep 到新加的 api 函数中然后跳回去
;by onepc/153785587
;添加新节参照玩命的代码，添加 api 函数参照 lordpe 手动添加一个 api 然后用 winhex 打开观看变化的内容
;然后根据它的变化写出来的。
;想到那里就写那里，写完之后就不想再去分类写子程序了。给初学的参考下。因我也是初学的。有注释
;还有就是修改之后启动慢了好多。map 时可以映小的+
;感谢 924792768\[QQ\]
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
.386
.model flat,stdcall
option casemap:none
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
include windows.inc
include user32.inc
include kernel32.inc
includelib user32.lib
includelib kernel32.lib
include comdlg32.inc
includelib comdlg32.lib
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_OpenFile proto ;打开文件并映射文件到内存
\_FileAlign proto dwVirSize : DWORD, dwAlign : DWORD
\_RvaToOffset proto \_lpNtHeader:DWORD,\_RvaAddr:DWORD
\_CheckPeInfo proto \_lpMem:DWORD ;检查是否是 PE 文件
\_SetSection proto  \_lpNtHeader:DWORD,\_RvaAddr:DWORD
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*


dwSectionSize equ 100h

.data
szCreateFileError db '打开文件失败',0
szCreateFileMappingError db '创建文件映射失败',0
szMapViewOfFileError db '映射文件到内存失败',0
szError db 'ERROR',0
szInvalPe db '无效 PE 格式',0

szNewSectionName db 'onepc',0

szWz db 'http://www.baidu.com/index.php?tn=onepc\_pg',0
szShell32 db 'Shell32.dll',0
szShellExecuteA db 'ShellExecuteA',0

szFmat db '%08x',0

szFilter  db  'Text Files(\*.exe)',0,'\*.exe',0,'Dll Files(\*.dll)',0,'\*.dll',0,0
.data?
hInstance dd ?

szBuffer db MAX\_PATH dup (?)

.code



;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_OpenFile proc
      local @hFile:DWORD,@hFileMap:DWORD
      local @lpMem:DWORD
      local @szFileNameBuf\[MAX\_PATH\]:BYTE
      local @stFile:OPENFILENAME
    pushad
          invoke RtlZeroMemory,addr @szFileNameBuf,sizeof @szFileNameBuf
          invoke RtlZeroMemory,addr @stFile,sizeof @stFile 
          mov @stFile.lStructSize,sizeof @stFile
          ;push hMain
          ;pop @stFile.hwndOwner
          ;mov @stFile.hwndOwner,hMin
          mov @stFile.lpstrFilter,offset szFilter
          lea eax,@szFileNameBuf
          mov @stFile.lpstrFile,eax
          mov @stFile.nMaxFile,MAX\_PATH
          mov @stFile.Flags,OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST
          invoke GetOpenFileName,addr @stFile
          .if !eax      ;getopenfilename 打开成功的话返回非 0 值
             popad 
             ret
          .endif
          
      invoke CreateFile,addr @szFileNameBuf,GENERIC\_READ+GENERIC\_WRITE,\\
                                        FILE\_SHARE\_READ+FILE\_SHARE\_WRITE,\\
                                        NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL
     .if eax==INVALID\_HANDLE\_VALUE
          jmp CreateFileError
     .endif
     mov @hFile,eax
     invoke GetFileSize,@hFile,NULL
     .if !eax ;等于 0 失败
         jmp CreateFileError
     .endif
     add eax,2000h
     xchg eax,ecx
     invoke CreateFileMapping,@hFile,NULL,PAGE\_READWRITE,0,ecx,NULL ; 这里若是都设 0 表示是以原文件的尺寸映射到内存中
     .if !eax
         jmp CreateFileMapError
     .endif
     mov @hFileMap,eax
     invoke MapViewOfFile,@hFileMap,FILE\_MAP\_READ+FILE\_MAP\_WRITE+FILE\_MAP\_COPY,NULL,NULL,NULL
     .if !eax
         jmp MapViewOfFileError
     .endif
         mov @lpMem,eax
         
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*         
     invoke \_CheckPeInfo,@lpMem  ;检查 PEINFO
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
             
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*     
      ExitOpenFile:
                 invoke UnmapViewOfFile, @lpMem
                 invoke CloseHandle, @hFileMap
                 invoke CloseHandle, @hFile  
           popad  
           ret
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*           
      CreateFileError:
              lea eax,szCreateFileError
              jmp ShowError
      CreateFileMapError:
              lea eax,szCreateFileMappingError
              jmp ShowError
      MapViewOfFileError:
              lea eax,szMapViewOfFileError
              jmp ShowError
      ShowError: 
              invoke MessageBox,NULL,eax,addr szError,MB\_OK
              jmp ExitOpenFile
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_OpenFile endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*



\_CheckPeInfo proc \_lpMem:DWORD ;起始地址
        local @lpNtHeader:DWORD ;NTHEADER
        local @dwSectionNum:DWORD ;原来节数目
        local @dwSectionAlign:DWORD ;节对齐值
        local @dwFileAlign:DWORD ;文件对齐值
        local @dwStartSection:DWORD ;原始节的起始位置
        local @dwNewStartSection:DWORD ;新节超始位置
        local @dwNewVaddr:DWORD ;新节的内存偏移值
        local @dwNewFaddr:DWORD ;新节的文件偏移值
        local @dwShell32:DWORD ;Shell32.dll 名所在的文件的文件偏移 含 lpMem
        local @dwShellExecuteA:DWORD ;指向 shellexecuteA 的地址
        local @dwDllNameAddr:DWORD ;
        local @dwShellExecuteAddr:DWORD
        local @dwNewImprotable:DWORD ;新的导入表位置
        local @dwSaveOep:DWORD ;旧入口点
        local @dwNewOep:DWORD ;新入口点
        local @dwImageBase:DWORD ;基址
        
     mov esi,\_lpMem
     assume esi:ptr IMAGE\_DOS\_HEADER
     .if \[esi\].e\_magic!='ZM'
         jmp PeError
     .endif
     add esi,\[esi\].e\_lfanew ;指向 NTHEADER
     mov @lpNtHeader,esi
     assume esi:ptr IMAGE\_NT\_HEADERS
     .if WORD PTR \[esi\].Signature!='EP'
         jmp PeError
     .endif
    movzx ecx,\[esi\].FileHeader.NumberOfSections
    mov @dwSectionNum,ecx
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;   
    push \[esi\].OptionalHeader.ImageBase  ;基址
    pop @dwImageBase
    push \[esi\].OptionalHeader.AddressOfEntryPoint ;oep
    pop @dwSaveOep
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;; 

    inc \[esi\].FileHeader.NumberOfSections ;节加 1

    push \[esi\].OptionalHeader.SectionAlignment
    pop @dwSectionAlign ;内存对齐值
    push \[esi\].OptionalHeader.FileAlignment 
    pop @dwFileAlign ;文件对齐值
;    mov ebx,\[esi\].OptionalHeader.SizeOfImage
;    add ebx,@dwSectionAlign
;    mov \[esi\].OptionalHeader.SizeOfImage,ebx
    
    
    add esi,sizeof IMAGE\_NT\_HEADERS ;esi 指向节表
    assume esi:PTR IMAGE\_SECTION\_HEADER
    mov @dwStartSection,esi ;保存第一个节表值
    mov ebx,sizeof IMAGE\_SECTION\_HEADER ;一个节表大小
    mov eax,@dwSectionNum ;有 N 个节表
    mul ebx ;eax 的值就是 N 个节表的大小
    add esi,eax ;esi 指向是要新加的节表的起始位置
    mov @dwNewStartSection,esi ;新节表起始位置
    lea edi,\[esi\].Name1
    lea esi,szNewSectionName
       SetSectionValue:
            lodsb ;把 esi 指向的字节一个一个装入 al
            test al,al
            jz ExitSetSectionName
            stosb ;从 al 里一个一个地传到 edi 中
        jmp SetSectionValue

ExitSetSectionName:
    mov esi,@dwNewStartSection
    push 0E00000E0h    ;设置节块属性为可读可写可执行   C0000040
    pop \[esi\].Characteristics
    push dwSectionSize    ;设置内存中节块大小
    pop \[esi\].Misc.VirtualSize
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

    
    invoke \_FileAlign,dwSectionSize,@dwFileAlign ;返回的是文件对齐后的值
    mov \[esi\].SizeOfRawData,eax  ;文件对齐值      
;    invoke \_FileAlign,dwSectionSize,@dwSectionAlign ;实际大小除于节对齐，那么返回的是节对齐之后的值
;    mov ecx,eax ;节对齐之后的值保存在 ecx 中
    
    sub @dwNewStartSection,sizeof IMAGE\_SECTION\_HEADER ;上一节的偏移
    mov eax,@dwNewStartSection
    assume eax:PTR IMAGE\_SECTION\_HEADER
    
    mov ecx,\[eax\].Misc.VirtualSize ;上节的实际大小
    push eax
    invoke \_FileAlign,ecx,@dwSectionAlign ;对齐之后的大小
    mov ecx,eax
    pop eax
    
    add ecx,\[eax\].VirtualAddress  ;上面 ecx 是节对齐后的大小 加上 上一节的内存偏移，那么 ecx 现在的值就是新节的内存偏移地址
     
    mov \[esi\].VirtualAddress,ecx ;新节内存偏移
    mov @dwNewVaddr,ecx ;保存新节内存偏移
    
    mov ecx,\[eax\].PointerToRawData ;上一节的文件偏移
    mov ebx,\[eax\].SizeOfRawData ;上一节的文件对齐的大小
    add ecx,ebx
    mov \[esi\].PointerToRawData,ecx ;新节文件偏移
    mov @dwNewFaddr,ecx ;保存新节的文件偏移
    push ecx ;新了文件偏移
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
    mov ecx,\[esi\].Misc.VirtualSize ;新节的没对齐的大小
    add ecx,\[esi\].VirtualAddress ;新节内存偏移 
    invoke \_FileAlign,ecx,@dwSectionAlign  ;返回对齐之后的值，就是整个镜像的大小
    mov ecx,eax
    mov eax,@lpNtHeader
    assume eax:PTR IMAGE\_NT\_HEADERS
    mov \[eax\].OptionalHeader.SizeOfImage,ecx   ;镜像大小
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
    pop ecx;新了文件偏移
    mov edi,ecx  ;;;;;;;
;    invoke wsprintf,addr szBuffer,addr szFmat,edi
;    invoke MessageBox,NULL,addr szBuffer,addr szBuffer,0
    add edi,\_lpMem ;在内存中的新节的文件偏移的位置
;    invoke wsprintf,addr szBuffer,addr szFmat,edi
;    invoke MessageBox,NULL,addr szBuffer,addr szBuffer,0
push edi   
    mov ecx,\[esi\].SizeOfRawData ;文件对齐后的大小给 ecx , 目的是要把它填零
    xor eax, eax
    cld
    rep stosb
    ;s:stosb
    ;loop s
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;添加字符串
invoke lstrlen,addr szWz ;不包含终止符的长度
;invoke wsprintf,addr szBuffer,addr szFmat,eax
;invoke MessageBox,NULL,addr szBuffer,addr szBuffer,0
pop edi ;这个是新节的文件偏移起如位置
mov ecx,eax ;长度
lea esi,szWz
cld
rep movsb
;edi 已加上了 szWz 的长度的位置了。
;invoke wsprintf,addr szBuffer,addr szFmat,edi
;invoke MessageBox,NULL,addr szBuffer,addr szBuffer,0
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;复制导入表结构到此




;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
add edi,6 ;其实已算出它多长
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
mov eax,edi
sub eax,\_lpMem ;
sub eax,@dwNewFaddr ;对节头的偏移量
mov @dwDllNameAddr,eax ;对节头的偏移量

mov eax,@dwNewVaddr ;新节内存偏移+偏移量
add eax,@dwDllNameAddr
xchg eax,@dwDllNameAddr
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*




mov @dwShell32,edi  ;指向 shell32.dll 的起始位置
invoke lstrlen,addr szShell32
mov ecx,eax ;长度
lea esi,szShell32
cld
rep movsb
inc edi ;加一个 0 字节  shell32.dll 结尾 edi 的文件偏移是加上镜像地址的
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
mov eax,edi
sub eax,\_lpMem ;shellexecutea 文件偏移量
sub eax,@dwNewFaddr ;对节头的偏移量
mov @dwShellExecuteAddr,eax ;对节头的偏移量
;mov @dwShellExecuteA,edi ;指向 ShellExecuteA 的起始位置
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*


add edi,2 ;这二个值是 Hint，填零

invoke lstrlen,addr szShellExecuteA
mov ecx,eax ;长度
lea esi,szShellExecuteA
cld
rep movsb
;add edi,5 ;指向下一行 准备放导入表
inc edi ;加一个 0 shellexecutea 以 0 结尾空字符


mov eax,@dwNewVaddr ;新节内存偏移+偏移量=指向 shellexecute 的前二位 即 hitn
add eax,@dwShellExecuteAddr ;比如内存偏移是 4000 + 6 那么 rva 就是 4006 了
mov DWORD PTR \[edi\],eax

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
mov eax,edi ;edi 现在是指向指向 shellexeute 偏移的偏移地址
sub eax,\_lpMem ;文件偏移量
sub eax,@dwNewFaddr ;对节头的偏移量
mov @dwShellExecuteAddr,eax ;对节头的偏移量 注:这里只是用同一个变量。
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

add edi,8
;;;;;;;;;;;;;;;;; 注意这里 
;;还有前面的变量偏移加上四就是到导入表的偏移变量
add eax,8
mov @dwNewImprotable,eax ;保存偏移量


      mov edx,@lpNtHeader
      assume edx:ptr IMAGE\_NT\_HEADERS
      mov ebx,\[edx\].OptionalHeader.DataDirectory\[sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress ;导入表 RVA
      invoke \_SetSection,@lpNtHeader,ebx
      invoke \_RvaToOffset,@lpNtHeader,ebx ;把 RVA 转为文件偏移，返回的 eax 的值就是文件偏移 
      push eax ;导入表文件偏移
      add eax,\_lpMem   ;eax 是导入表的文件偏移，所以加上\_lpMem，那么现在 eax 就是指向导入表
      
;;;;;;;;;;;;;;;;;      
       mov esi,eax ;源
;;;;;;;;;;;;;;;;; 注意这里
       
       mov edx,eax
       assume edx:ptr IMAGE\_IMPORT\_DESCRIPTOR ;IMAGE\_IMPORT\_DESCRIPTOR 最后一个是全 0 的结构
       ;\*\*\*
          .while \[edx\].OriginalFirstThunk || \[edx\].TimeDateStamp || \[edx\].ForwarderChain || \[edx\].FirstThunk || \[edx\].Name1
               add edx,sizeof IMAGE\_IMPORT\_DESCRIPTOR
          .endw ;全 0 时退出
;       add edx,20 ;全 0 的结构 一个导入表的结构是 20 字节 不要加
       sub edx,\_lpMem ;导入表结尾的文件偏移
       pop eax ;导入表文件偏移
       sub edx,eax ;edx 导入表大小

mov ecx,edx ;长度
cld
rep movsb
;复制之前的导入表到新节中完成

;edi 现在的位置就可以写入新的导入表结构
  mov edx,edi
assume edx:ptr IMAGE\_IMPORT\_DESCRIPTOR
mov eax,@dwNewVaddr ;新节内存偏移+偏移量
add eax,@dwShellExecuteAddr ;这里指向-指向 shellexecuteaddr 的 rva
;;;;;;;
mov @dwShellExecuteAddr,eax ;存回去，下面用
;;;;;;;
  mov \[edx\].OriginalFirstThunk,eax ;设成相同
  mov \[edx\].TimeDateStamp,0 
  mov \[edx\].ForwarderChain,0
  mov ecx,@dwDllNameAddr
  mov \[edx\].Name1,ecx  ;指向 dll 文件的 Rva 偏移地址。这里的是新节的 RVA 
  mov \[edx\].FirstThunk,eax ;设成相同
;这里设置新增导入表完成


;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
;更改 oep
add edi,50  ;20\*2=40 +10 预多 
mov edx,edi  ;用 edx 保存文件里的起始位置 
sub edi,\_lpMem
sub edi,@dwNewFaddr ;对节头的偏移量
add edi,@dwNewVaddr ;加上新节的 RVA 那就是入口点了
mov @dwNewOep,edi
mov WORD PTR \[edx\],016Ah
add edx,2
mov WORD PTR \[edx\],006Ah
add edx,2
mov WORD PTR \[edx\],006Ah
add edx,2
mov BYTE PTR \[edx\],68h
add edx,1
mov ecx,@dwNewVaddr;网址在新节 0 偏移处
add ecx,@dwImageBase ;rva+基址 就是装载到内存中的实际地址
mov DWORD PTR \[edx\],ecx 
add edx,4
mov WORD PTR \[edx\],006Ah
add edx,2
mov WORD PTR \[edx\],006Ah
add edx,2
mov BYTE PTR \[edx\],0E8h
add edx,1
mov DWORD PTR \[edx\],00000005h ;因为下边还有一条 jmp 指令,预长点 ;call
add edx,4
mov BYTE PTR \[edx\],0E9h    ;jmp oldoep
add edx,1
;=============================================================
mov ecx,@dwSaveOep ;旧的 oep 
add ecx,@dwImageBase ;ecx 装入内存后的地址  要跳到的目的地址
mov eax,edx
add eax,4 ;这里是已加上 jmp 指令的长度了
sub eax,\_lpMem
sub eax,@dwNewFaddr ;对新节的偏移值
add eax,@dwNewVaddr ;rva 值
add eax,@dwImageBase ;装入内存中的地址  本身 jmp 地址+指令长度
sub ecx,eax
;=============================================================
mov DWORD PTR \[edx\],ecx
add edx,4
;=============================================================
;0000000A5 处
mov WORD PTR \[edx\],25FFh ;这个 jmp 是跳到 dll 里去的。后面跟的地址是导入表中 API 函数的地址
add edx,2
mov eax,@dwShellExecuteAddr ;指向指向 API 的 rva 值
add eax,@dwImageBase ;装入内存中的地址
mov DWORD PTR \[edx\],eax





;jmp 计算：目的地址-(本身 jmp 的起始地址+本身指令长度)=jmp 后面的数据

;call xxxxxxx 变成机器码的话这个是在文件里存放的。这个表标的是
;比如当前的 call 代码的起始地址是 00401041    汇编这个 CALL 0040104A
;那么机器码就是：目标地址(0040104A)-起始地址+5 字节(call 指令长度)=00000004
;就变成了 call E8 04000000 这个机器码了






;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
mov esi,@lpNtHeader
assume esi:ptr IMAGE\_NT\_HEADERS
;;;;;;更改 oep
mov ecx,@dwNewOep
mov \[esi\].OptionalHeader.AddressOfEntryPoint,ecx

mov eax,@dwNewVaddr ;新节内存偏移+偏移量
add eax,@dwNewImprotable
mov \[esi\].OptionalHeader.DataDirectory\[sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress,eax   ;设置新的导入表 rva 值
;再把 IAT 清零
mov \[esi\].OptionalHeader.DataDirectory\[12\*(sizeof IMAGE\_DATA\_DIRECTORY)\].VirtualAddress,0
mov \[esi\].OptionalHeader.DataDirectory\[12\*(sizeof IMAGE\_DATA\_DIRECTORY)\].isize,0










;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 
ExitPeInfo:
    assume edx:nothing
    assume eax:nothing
    assume esi:nothing
  ret

PeError:
   lea eax,szInvalPe
   invoke MessageBox,NULL,eax,addr szError,MB\_OK
   jmp ExitPeInfo    

\_CheckPeInfo endp


;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_RvaToOffset proc uses edi ebx \_lpNtHeader:DWORD,\_RvaAddr:DWORD
             local @dwSetionNum
       mov edi,\_lpNtHeader ;\*\*\*\*\*\*\*\*\*\*\*
       assume edi:ptr IMAGE\_NT\_HEADERS
       movzx eax,\[edi\].FileHeader.NumberOfSections
       mov @dwSetionNum,eax
       add edi,sizeof IMAGE\_NT\_HEADERS
       assume edi:ptr IMAGE\_SECTION\_HEADER
       mov edx,\_RvaAddr
          .while @dwSetionNum>0
                     mov eax,\[edi\].VirtualAddress
                     add eax,\[edi\].SizeOfRawData
                 .if (edx>=\[edi\].VirtualAddress) && (edx<eax)
                     mov ebx,\[edi\].VirtualAddress
                     sub edx,ebx
                     mov eax,\[edi\].PointerToRawData ;节头的文件偏移
                     add eax,edx   ;节头的文件偏移+对节头的偏移 即已把 RVA 转换成了文件偏移

                     jmp @F
                 .endif    
                dec @dwSetionNum
                add edi,sizeof IMAGE\_SECTION\_HEADER ;指向下一个节头
          .endw  
       @@:assume edi:nothing 
     ret
\_RvaToOffset endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;设置可写可读
\_SetSection proc uses edi edx eax ebx \_lpNtHeader:DWORD,\_RvaAddr:DWORD
             local @dwSetionNum
       mov edi,\_lpNtHeader ;\*\*\*\*\*\*\*\*\*\*\*
       assume edi:ptr IMAGE\_NT\_HEADERS
       movzx eax,\[edi\].FileHeader.NumberOfSections
       mov @dwSetionNum,eax
       add edi,sizeof IMAGE\_NT\_HEADERS
       assume edi:ptr IMAGE\_SECTION\_HEADER
       mov edx,\_RvaAddr
          .while @dwSetionNum>0
                     mov eax,\[edi\].VirtualAddress
                     add eax,\[edi\].SizeOfRawData
                 .if (edx>=\[edi\].VirtualAddress) && (edx<eax)

                    push 0C0000040h
                    pop \[edi\].Characteristics

                     jmp @F
                 .endif    
                dec @dwSetionNum
                add edi,sizeof IMAGE\_SECTION\_HEADER ;指向下一个节头
          .endw  
       @@:assume edi:nothing 
     ret
\_SetSection endp
















\_FileAlign proc uses ecx edx dwVirSize : DWORD, dwAlign : DWORD     
    mov ecx, dwAlign  ;对齐值
    mov eax, dwVirSize ;写入的实值大小
    xor edx, edx
    div ecx ;用没有对齐的实际值除以对齐值，若余数(放在 edx 中)等于 0，能整除说明它的大小刚好就用 商\*对齐值 =这个就是大小
    cmp edx, 0
    jz AlreadyAligned
    inc eax    ;若余数不等于 0,说明(商+1) \*对齐值就可以放值对齐后的大小
AlreadyAligned:
    mul ecx      ;对齐后的值放在 eax 中
    ret
\_FileAlign endp 
;例：没对齐的大小是 100   对齐值是 200 那么 100/200 edx 存的余数不为 0 则 要商(eax+1)\*对齐值就可以存放这个 100 的大小
;即是(0+1) \*200=200 即经对齐后的大小是 200

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;添加 API 函数
;用 lordpe 添加了导入表结构看了下，里面的导入表大小无相关
;IAT 那里可以清零
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;\_AddImportTableAPI proc
;   
;
;
;\_AddImportTableAPI endp



;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
start:
   invoke GetModuleHandle,NULL
   mov hInstance,eax
   invoke \_OpenFile
   invoke ExitProcess,NULL
end start
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```