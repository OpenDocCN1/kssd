
# [旧帖][原创]用汇编构造`sprintf`函数0.00雪花

使用汇编构造`sprintf`函数。

我没有去逆向`sprintf`函数，主要是过程太多了，并且只要知道运算结果就能大概推算出过程了。

以下构造并不完整，但能实现`sprintf`函数大多数功能，比如字符串中的`%%`解析为`%`，`%x`解析为十六进制单字节数，`%X`解析为十六进制双字数，`%d`解析为十进制数，其他的出于时间关系，我就没写了。

另外，为了方便调用，我将过程写为`__stdcall`调用方式，也算是`__stdcall`的`sprintf`吧。。。

## 使用前宏定义

在使用前应该加上两个宏：

```asm
put macro
    mov     byte ptr [ebp+ecx+08h], al
endm

get macro
    mov     al, byte ptr [ebp+ebx+0ch]
endm
```

`put`就是向新指针写入一字节，`get`就是向老字符串读取一字节

目前暂时只能处理ANSI格式字符串，不过对于不熟悉Unicode的童鞋们也够了

废话不多说，上源码：

## 源码

```asm
;void sprintf(LPCTSTR lpFormatStr, LPCSTR lpStr, ...)

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

sprintf:
    push    ebp
    mov     ebp, esp
    ;无局部变量
    xor     ebx, ebx                              ;格式化前指针
    xor     ecx, ecx                              ;格式化后指针
    mov     edx, 0ch                              ;参数指针
@@1:                                              ;主循环
    inc     ecx
    get
    cmp     al, 0
    jz      @@end
    .if     al == '%'
        inc     ebx
        add     edx, 04h
        get
        .if     al == 'x'
            mov     al, '0'
            put
            inc     ecx
            mov     al, 'x'
            put
            xor     eax, eax
            mov     al, byte ptr [ebp+edx]
            ror     eax, 4
            .if     al > 9
                add     al, 37h
            .else
                add     al, 30h
            .endif
            inc     ecx
            put
            rol     eax, 4
            .if     al > 9
                add     al, 37h
            .else
                add     al, 30h
            .endif
            inc     ecx
            put
        .elseif al == 'X'
            mov     al, '0'
            put
            inc     ecx
            mov     al, 'x'
            put
            mov     eax, dword ptr [ebp+edx]
            push    edx
            mov     edx, 8
            .while  edx > 0
                push    eax
                dec     edx
                lea     edx, [edx*4]
                ror     eax, edx
                and     al, 0fh
                .if     al > 9
                    add     al, 37h
                .else
                    add     al, 30h
                .endif
                inc     ecx
                put
                pop     eax
            .endw
            pop     edx
        .elseif al == 'd'
            mov     eax, dword ptr [ebp+edx]
            push    ebx
            push    edx
            xor     ebx, ebx
            .while  eax > 0
                mov     edx, 0ah
                div     edx
                push    edx
                inc     ebx
            .endw
            .while ebx > 0
                pop     eax
                inc     ecx
                put
                dec     ebx
            .endw
            pop     edx
            pop     ebx
        .elseif al == '%'
            sub     edx, 04h
            put
        .endif
    .else
        put
    .endif
    inc     ebx
    jmp     @@1
@@end:                                              ;循环结束
    inc     ecx
    xor     al, al
    put
    leave
    sub     edx, 04h
    sub     esp, edx
    ret
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
