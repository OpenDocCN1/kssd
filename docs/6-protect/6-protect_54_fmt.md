
# 克图鲁的呼唤

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-620.htm`](https://www.pediy.com/kssd/pediy08/pediy8-620.htm)

可以用来调用和获取 API ，为 Shellcode（或野猪力量）和壳设计。

不同的内存块开头使用 `KTULU` 宏，之后即可用 `kcall`, `kget` 宏。

注意不支持转向函数，比如 `RtlZeroMemory`，不过可以用 `kcall` 调用 `GetProcAddress` 来使用他们。

每个 `kget`/`kcall` 占用 9 字节，每次调用 `ktulu` 增加 155 字节（不过你不会经常用吧:{）

你也可以用在普通程序中，会慢一点（不过现在的 CPU...），有一定的反反汇编能力（别指望用来加密）。

优化我进行了很久，一开始 400+ 后来 208 后来 168 终于 155，希望高手可以继续进行尺寸优化（我使用了一些比较极端的方法）。这也是我贴出来的主要原因。究竟有没有极限？

### 一个例子

```asm
.386

.model flat, stdcall

option casemap :none

include  KTULU.INC

.DATA

szMsg db "Hello World!",13,10,0

.CODE

START:

KTULU

kget  KiUserExceptionDispatcher

kcall  MessageBoxA, 0, offset szMsg, offset szMsg, 0

kcall  ExitProcess, 0

end START
```

### 这是头文件

```asm
; ===================
;   THE CALL OF KTULU
; ===================

;
; Coded by forgot/iPB
;
; Size = 155 bytes !!!
;
; EXAMPLE:
;                   KTULU
;                   kget    KiUserExceptionDispatcher
;                   kcall   MessageBoxA, 0, offset szMsg, offset szMsg, 0
;                   kcall   ExitProcess, 0
;                   push    0
;                   kcall   ExitProcess
KTULU                   macro
                        local   core, hash

kcall                   macro   procedure, parameters:VARARG
                        local   reversed, param

reversed                textequ <>

                        %for    param, <parameters>
                        reversed CATSTR <param>, <!,>, reversed
                        endm

                        %for    param, <reversed>
                        push    param
                        endm

                        call    core+2
                        gethash procedure
                        endm

kget                    macro   procedure
                        call    core+2+1
                        gethash procedure
                        endm

gethash                 macro   procedure
                        hash    = 0
                        irpc    c, <procedure>
                        hash    = ((hash shl 7) and 0FFFFFFFFh) or (hash shr (32-7))
                        hash    = hash xor "&c"
                        endm
                        dd      hash
                        endm

core                    proc
                        jmp     __skip1                     ; Two short JMPs save 1 byte
                                                            ;       than a long JMP
                        test    al, 0F9h                    ; 0F9H = STC
                        xchg    esi, [esp]
                        lodsd
                        xchg    esi, [esp]
                        pushfd
                        pushad
                        xchg    ebp, eax                    ; EBP = Hash
                                                            ; EAX = Kernel32 base
                        db      64h, 0A1h                   ; MOV EAX, FS:[30H]
                        dd      30h
                        test    eax, eax
                        jns     __nt_port
                        mov     eax, [eax+34h]
                        mov     eax, [eax+7Ch+3Ch]
                        jmp     __k32_done
__nt_port:               mov     eax, [eax+0Ch]
                        mov     esi, [eax+1Ch]
                        lodsd
                        mov     eax, [eax+08h]
__k32_done:             call    __get_modules               ; EDI = Module list
                        db      'ntdll', 0                  ; Add your modules here !!!
                        db      'user32', 0
__skip1:                jmp     __skip2                     ; Hmmmmmmmmmm
__get_modules:          pop     edi
__scan_export:          xchg    ebx, eax                    ; EBX = Image base
                                                            ; BL = 0
                        mov     ecx, [ebx+3Ch]
                        mov     ecx, [ebx+ecx+78h]          ; ECX = Export table
                        add     ecx, ebx
                        xor     esi, esi                    ; ESI = Index
__check_hash:           lea     eax, [ebx+esi*4]            ; EAX = Name pointer
                        add     eax, [ecx+20h]
                        mov     eax, [eax]
                        cdq                                  ; EDX = Calculated hash
                        add     eax, ebx
;此处的 cdq 上移，我搞错顺序了，谢谢 hexer
__calc_hash:            rol     edx, 7
                        xor     dl, [eax]
                        inc     eax
                        cmp     [eax], bl
                        jne     __calc_hash
                        cmp     edx, ebp                    ; Check hash
                        jne     __next_index
                        mov     edx, [ecx+24h]              ; EBX = API address
                        add     edx, ebx
                        movzx   edx, word ptr [edx+esi*2]
                        mov     eax, [ecx+1Ch]
                        add     eax, ebx
                        add     ebx, [eax+edx*4]
                        mov     [esp+4*7], ebx              ; Update POPAD.EAX
__exit:                 popad
                        popfd
                        jc      $+2+1
                        push    eax                     ; Jump to API address if CF = 0
                        retn
__next_index:           inc     esi
                        cmp     [ecx+18h], esi
                        jge     __check_hash
__load_module:          push    edi
                        kcall   LoadLibraryA            ; AL = 0
__next_char:            scasb
                        jne     __next_char
                        test    eax, eax
                        jz      __load_module
                        jmp     __scan_export
__skip2:
core                    endp
                        endm
```

其实就是编译的时候计算 hash。

`kcall` 的真实样子是

```
call core+2

dd hash
```

`core+2` 处是 `test al, F9`,执行之后 `CF=0`，如果进入 `core+2+1` 执行 `stc`。

也就是说这两个指令重叠了。

获取 `kernel32` 用的是 `ratter` 的方法，扫 `peb`，有一个 `mov eax, [eax]`变成了 `lodsd`，省 `1byte`。

列表优化用了一个假设，调用的 `api` 都在表里，所以不设结束标志和判断。

列表推进又用了一个假设（当然是成立的），`DLL` 地址是 `1000` 对齐的，`AL=0`，于是 `scasb` 正确执行。

计算 hash 结束又用到这招，`bl=0`。

最 `yd` 得应该算那个分解 `jmp`，一个 `long jmp` `5bytes`，`2` 个 `short jmp` `4` 字节，这个很恶心了。

最后的 `jc` 也算一个普遍的技巧把
