
```
; #########################################################################



    ; --------------------------------------
    ; This procedure was written by Iczelion
    ; --------------------------------------



      .386
      .model flat, stdcall  ; 32 bit memory model
      option casemap :none  ; case sensitive



      include \MASM32\INCLUDE\kernel32.inc



    .code



; #########################################################################



a2dw proc uses ecx edi edx esi String:DWORD



      ;----------------------------------------
      ; Convert decimal string into dword value
      ; return value in eax
      ;----------------------------------------



      xor ecx, ecx
      mov edi, String
      invoke lstrlen, String    ;在前面清空ecx是没用的，WinApi可能改变ecx、edx的值



      .while eax != 0
        xor edx, edx
        mov dl, byte ptr [edi]
        sub dl, "0" ; subtrack each digit with "0" to convert it to hex value
        mov esi, eax
        dec esi
        push eax
        mov eax, edx    ;ascii对应的byte
        push ebx
        mov ebx, 10
          .while esi > 0    ;num*10^esi
            mul ebx    ;结果到eax（低位）,edx（高位）中
            dec esi
          .endw
        pop ebx
        add ecx, eax        ;ecx储存结果
        pop eax
        inc edi
        dec eax
      .endw



        mov eax, ecx
        ret



a2dw endp



; #########################################################################



end

```

以前听罗老师说不建议用`masm32`自带的宏、`lib`，可能有各种奇葩错误，我还不大相信，直到我看到了这个函数。

这个函数看着工工整整，算法的可行性也在上一篇分析过了，但其实完全无法工作。

为什么呢？请看第24、26行，

为了使`ecx`储存结果，函数在调用`winAPI`之前将`ecx`清零，

但这么做其实是徒劳的，`WinAPI`执行后，`ebx`、`edi`、`esi`和`ebp`的值总是不会被改变的，但`ecx`和`edx`的值有极大几率被改变。（同时这也是win对api调用的callback的要求）

这就导致`ecx`有了一个未知的初始值，导致累加发生在一个未知值的基础上，最终得到了没有任何意义的返回值。

# `atodw.asm`

```
; #########################################################################



  ; ---------------------------------------------------------------
  ;      本程序最初由 Tim Roberts 编写
  ;
  ;     Alexander Yackubtchik 优化了部分代码
  ; ---------------------------------------------------------------



    .486
    .model flat, stdcall  ; 32 bit memory model
    option casemap :none  ; case sensitive
    .code



; #########################################################################



atodw proc String:DWORD



  ; ----------------------------------------
  ; 十进制转dword
  ; eax储存返回值
  ; ----------------------------------------



    push esi
    push edi



    xor eax, eax
    mov esi, [String]
    xor ecx, ecx
    xor edx, edx
    mov al, [esi]
    inc esi
    cmp al, 2D    ;检测负号
    jne proceed    ;不是负号就跳转
    mov al, byte ptr [esi]
    not edx    ;FFFFFFFF
    inc esi
    jmp proceed



  @@: 
    sub al, 30h    ;ascii->byte
    lea ecx, dword ptr [ecx+4*ecx]    ;ecx*=5
    lea ecx, dword ptr [eax+2*ecx]    ;ecx=ecx*2+eax
    mov al, byte ptr [esi]
    inc esi



  proceed:
    or al, al
    jne @B    ;非0（没处理完）上跳
    lea eax, dword ptr [edx+ecx]
    xor eax, edx



    pop edi
    pop esi



    ret



atodw endp



; #########################################################################



end

```

这个函数支持有符号数，好评。

## 补码

补码是当今广泛使用的有符号数编码规则，

负数表示为其绝对值表示的正数按位取反再+1（或-1后按位取反），

补码的优点是用无符号数规则进行有符号数加减计算，结果仍满足补码规则，且符合数学运算规定。

## 代码亮点：灵活的使用`xor`和`lea`

这个函数对有符号数的处理可谓颇为巧妙，

若字符串的第一个符号不是“-”，函数会将其按照正数处理，令`edx`为0，

在执行到49行时，`ecx`中就储存了正确的结果，而由于`edx`为0，不会改变`ecx`的值，就相当于执行了`eax=ecx+0`，`eax`中储存正确的结果。

由于`0 xor any=any`，下一行的`xor`不会改变`eax`的值。

若字符串的第一个符号是“-”，`edx`将会=`0xFFFFFFFF`，

这样，49行会使`eax-=1`，

由于`edx`的所有二进制位均为1，所以`any xor edx=not any`，50行实际上将`eax`进行了按位取反操作，

这样，`eax`正好满足补码规则，被顺利的转换为负数。

39~45行《字符串转双字函数分析(1)》已经做了详细讨论，不再赘述。

# `a2dw_ex.asm`

```
; ?



    .486                      ; force 32 bit code
    .model flat, stdcall      ; memory model & calling convention
    option casemap :none      ; case sensitive



    EXTERNDEF decade :DWORD



    .code



; ?



align 4    ;使指令从4整数倍的内存地址开始以加快速度。



atodw_ex proc lpstring:DWORD



    mov [ebp+12], edi    ;备份edi



  ; ----------------------------------------------------
  ; 用展开的strlen函数测试12个byte中是否有0。
  ; ----------------------------------------------------
    mov eax,lpstring                    
    lea edx,[eax+3]                     ; pointer+3 used in the end



    mov edi,[eax]                       ; read first 4 bytes
    add eax,4                           ; increment pointer
    lea ecx,[edi-01010101h]             ; subtract 1 from each byte
    not edi                             ; invert all bytes
    and ecx,edi                         ; and these two
    and ecx,80808080h
    jnz @F



    mov edi,[eax]                       ; read next 4 bytes
    add eax,4                           ; increment pointer
    lea ecx,[edi-01010101h]             ; subtract 1 from each byte
    not edi                             ; invert all bytes
    and ecx,edi                         ; and these two
    and ecx,80808080h
    jnz @F



    mov edi,[eax]                       ; read last 4 bytes
    add eax,4                           ; increment pointer
    lea ecx,[edi-01010101h]             ; subtract 1 from each byte
    not edi                             ; invert all bytes
    and ecx,edi                         ; and these two
    and ecx,80808080h



  @@:
    test ecx,00008080h                  ; test first two bytes
    jnz @F
    shr ecx,16                          ; not in the first 2 bytes
    add eax,2
  @@:
    shl cl,1                            ; use carry flag to avoid branch
    sbb eax,edx                         ; compute length
  ; -------------------------------------------



    mov edi, eax                        ; EDI放字符串长度
    mov edx, lpstring
    sub edx, 1    ;使得edx+edi刚好可以指向最后一个字符



    xor eax, eax



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+40]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+80]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+120]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+160]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+200]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+240]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+280]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+320]
    sub edi, 1
    jz atout



    movzx ecx, BYTE PTR [edx+edi]
    add eax, [ecx*4+decade-192+360]



  atout:



    mov edi, [ebp+12]



    ret



atodw_ex endp



; ?



end

```

`masm32`的`lib`帮助文件中居然说这个函数没有返回值（黑人问号脸），其实是有的（要不咱执行这个函数图个什么），在`eax`中。

### `dectbl.asm`
