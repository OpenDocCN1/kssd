
# 代码分析

## 汇编代码片段

```assembly
.text:004010DC            mov     [ebp+var_1], 1  
.text:004010E0            push    esi                 ; 注册名  
.text:004010E1            push    edi                 ; 0  
.text:004010E2            push    eax                 ; eax 为第一次分配返回地址  
.text:004010E3            call    sub_40156A  
.text:004010E3  
.text:004010E8            add     esp, 18h  
.text:004010EB            cmp     [ebp+var_1], 0  
.text:004010EF            push    edi  
.text:004010F0            jnz     short GootOK  
.text:004010F0  
.text:004010F2            push    offset s_BadBoy     ; "Bad boy"  
.text:004010F7            push    offset s_KeepOnTrying ; "Keep on trying!"  
.text:004010FC            jmp     short UseMessageBoxA  
.text:004010FC  
.text:004010FE ; ---------------------------------------------------------------------------  
.text:004010FE  
.text:004010FE GootOK:                    ; CODE XREF: sub_401000+F0j  
.text:004010FE            push    offset s_GoodBoy    ; "Good boy"  
.text:00401103            push    offset s_YouDidIt   ; "You did it!"  
.text:00401108            jmp     short UseMessageBoxA  
.text:00401108  
.text:0040110A ; ---------------------------------------------------------------------------  
.text:0040110A  
.text:0040110A Wrongserial:               ; CODE XREF: sub_401000:loc_40106Dj  
.text:0040110A                     ; sub_401000+76j  
.text:0040110A            push    10h  
.text:0040110C            push    offset Caption      ; "ERROR"  
.text:00401111            push    offset s_WrongSerial ; "Wrong serial!"  
.text:00401116            jmp     short UseMessageBoxA  
.text:00401116  
.text:00401118 ; ---------------------------------------------------------------------------  
.text:00401118  
.text:00401118 NameError:                 ; CODE XREF: sub_401000+58j  
.text:00401118            push    10h                 ; uType  
.text:0040111A            push    offset Caption      ; "ERROR"  
.text:0040111F            push    offset Text         ; "Use some other name!"  
.text:0040111F  
.text:00401124  
.text:00401124 UseMessageBoxA:            ; CODE XREF: sub_401000+FCj  
.text:00401124                     ; sub_401000+108j  
.text:00401124                     ; sub_401000+116j  
.text:00401124            push    hDlg                ; hWnd  
.text:0040112A            call    ds:MessageBoxA  
.text:0040112A  
.text:00401130            pop     edi  
.text:00401131            pop     esi  
.text:00401132            pop     ebx  
.text:00401133            leave  
.text:00401134            retn  
.text:00401134  
.text:00401134 sub_401000      endp  
.text:00401134  
```

## 二、VM 初始化

这个 VM 是"非标准"的虚拟机，初始化时它所做的主要是把 VM 指令指针保存在一个分配好的内存地址中，为以下调用作准备。

```assembly
.text:00401BDC ; arg_0 与 arg_8 初始值为 40E000  
.text:00401BDC ; 使用这个函数后  
.text:00401BDC ; esi, edi, ecx 与进入是一样  
.text:00401BDC ; esi 为注册名, edi=0  
.text:00401BDC ; eax 为第一次分配返回地址  
.text:00401BDC ; [第一次分配返回地址+?]填充了大量函数偏移  
.text:00401BDC ; [第一次分配返回地址+48h]为 1000h 大小,  
.text:00401BDC ; [第一次分配返回地址+4Ch]取[40E000+20h]值  
.text:00401BDC  
.text:00401BDC ; int __stdcall sub_401BDC(int, size_t Size, int)  
.text:00401BDC sub_401BDC      proc near                 ; CODE XREF: sub_401000+CBp  
.text:00401BDC  
.text:00401BDC arg_0           = dword ptr   4  
.text:00401BDC Size            = dword ptr   8  
.text:00401BDC arg_8           = dword ptr   0Ch  
.text:00401BDC  
.text:00401BDC            push    esi  
.text:00401BDD            push    edi  
.text:00401BDE            push    24h                 ; Size  
.text:00401BE0            mov     esi, ecx            ; 分配返回地址  
.text:00401BE2            call    operator new(uint)  
.text:00401BE2  
.text:00401BE7            mov     edi, 1000h  
.text:00401BEC            push    edi                 ; Size  
.text:00401BED            mov     [esi+7Ch], eax  
.text:00401BF0            call    operator new(uint)  
.text:00401BF0  
.text:00401BF5            mov     [esi+40h], edi  
.text:00401BF8            mov     edi, [esp+10h+Size]  
.text:00401BFC            push    edi                 ; Size  
.text:00401BFD            mov     [esi+3Ch], eax  
.text:00401C00            call    operator new(uint)  
.text:00401C00  
.text:00401C05            push    edi  
.text:00401C06            push    [esp+18h+arg_0]     ; 40E000+18h  
.text:00401C0A            mov     [esi+44h], eax  
.text:00401C0D            push    eax  
.text:00401C0E            call    unknown_libname_1   ; Microsoft VisualC 2-8/net runtime  
.text:00401C0E  
.text:00401C13            mov     eax, [esp+20h+arg_8]  ; 取[40E000+20h]  
.text:00401C17            add     esp, 18h  
.text:00401C1A            mov     [esi+48h], edi      ; edi 为 1000h 大小,  
.text:00401C1A                     ; esi 为第一次分配返回地址  
.text:00401C1D            mov     [esi+4Ch], eax      ; 取[40E000+20h]  
.text:00401C20            pop     edi  
.text:00401C21            mov     dword ptr [esi+80h], offset VMEip3  
.text:00401C2B            mov     dword ptr [esi+84h], offset VMAdd  
.text:00401C35            mov     dword ptr [esi+88h], offset VMMOV  
.text:00401C3F            mov     dword ptr [esi+8Ch], offset VMMov_1  
.text:00401C49            mov     dword ptr [esi+90h], offset VMMov1  ;  
.text:00401C49                     ; [ebp+var_4]指向[[esi+20h]]  
.text:00401C49                     ; [esi+20h]加 4  
.text:00401C53            mov     dword ptr [esi+94h], offset VMAND_DOWRD  
.text:00401C5D            mov     dword ptr [esi+98h], offset VMMOV  
.text:00401C67            mov     dword ptr [esi+9Ch], offset VMImul  
.text:00401C71            mov     dword ptr [esi+0A0h], offset VMDiv  
.text:00401C7B            mov     dword ptr [esi+0A4h], offset VMMOV_BYTE  
.text:00401C85            mov     dword ptr [esi+0A8h], offset VMSub  
.text:00401C8F            mov     dword ptr [esi+0ACh], offset VMFlag  
.text:00401C99            mov     dword ptr [esi+0B0h], offset VMNextEip1  
.text:00401CA3            mov     dword ptr [esi+0B4h], offset VMEipAddress  
.text:00401CAD            mov     dword ptr [esi+0B8h], offset VMEipAddress1  
.text:00401CB7            mov     dword ptr [esi+0BCh], offset VMEipAddress_1  
.text:00401CC1            mov     dword ptr [esi+0C0h], offset VMEipAddress_0  
.text:00401CCB            mov     dword ptr [esi+0C4h], offset VMNextEip  
.text:00401CD5            mov     dword ptr [esi+0C8h], offset VMNextEip2  
.text:00401CDF            mov     dword ptr [esi+0CCh], offset VMShl  
.text:00401CE9            mov     dword ptr [esi+0D0h], offset VMShr  
.text:00401CF3            mov     dword ptr [esi+0D4h], offset VMAnd  
.text:00401CFD            mov     dword ptr [esi+0D8h], offset VMOr  
.text:00401D07            mov     dword ptr [esi+0DCh], offset VMXor  
```
