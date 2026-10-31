
```
.text:0000000000401F7B ; catch(char const*) // owned by 401EEA
.text:0000000000401F7B                 endbr64
.text:0000000000401F7F                 cmp     rdx, 1
.text:0000000000401F83                 jz      short loc_401F8D
.text:0000000000401F85                 mov     rdi, rax        ; struct _Unwind_Exception *
.text:0000000000401F88                 call    __Unwind_Resume
.text:0000000000401F8D ; ---------------------------------------------------------------------------
.text:0000000000401F8D
.text:0000000000401F8D loc_401F8D:                             ; CODE XREF: main+D0↑j
.text:0000000000401F8D                 mov     rdi, rax        ; void *
.text:0000000000401F90                 call    ___cxa_begin_catch
.text:0000000000401F95                 mov     [rbp+var_28], rax
.text:0000000000401F99                 lea     rax, aYouCatchTheErr ; "you catch the error "
.text:0000000000401FA0                 mov     rsi, rax
.text:0000000000401FA3                 lea     rax, _ZSt4cout  ; std::cout
.text:0000000000401FAA                 mov     rdi, rax
.text:0000000000401FAD ; try {
.text:0000000000401FAD                 call    __ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc ; std::operator<<<std::char_traits<char>>(std::ostream &,char const*)
.text:0000000000401FB2                 mov     rdx, rax
.text:0000000000401FB5                 mov     rax, [rbp+var_28]
.text:0000000000401FB9                 mov     rsi, rax
.text:0000000000401FBC                 mov     rdi, rdx
.text:0000000000401FBF                 call    __ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc ; std::operator<<<std::char_traits<char>>(std::ostream &,char const*)
.text:0000000000401FC4                 mov     rdx, cs:_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6__ptr
.text:0000000000401FCB                 mov     rsi, rdx
.text:0000000000401FCE                 mov     rdi, rax
.text:0000000000401FD1                 call    __ZNSolsEPFRSoS_E ; std::ostream::operator<<(std::ostream & (*)(std::ostream &))
.text:0000000000401FD6                 lea     rax, aHereIsAGiftFor ; "here is a gift for you!"
.text:0000000000401FDD                 mov     rsi, rax
.text:0000000000401FE0                 lea     rax, _ZSt4cout  ; std::cout
.text:0000000000401FE7                 mov     rdi, rax
.text:0000000000401FEA                 call    __ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc ; std::operator<<<std::char_traits<char>>(std::ostream &,char const*)
.text:0000000000401FEF                 mov     rdx, cs:_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6__ptr
.text:0000000000401FF6                 mov     rsi, rdx
.text:0000000000401FF9                 mov     rdi, rax
.text:0000000000401FFC                 call    __ZNSolsEPFRSoS_E ; std::ostream::operator<<(std::ostream & (*)(std::ostream &))
.text:0000000000402001                 mov     rax, cs:setbuf_ptr
.text:0000000000402008                 mov     rsi, rax
.text:000000000040200B                 lea     rax, format     ; "setbufaddr:%p\n"
.text:0000000000402012                 mov     rdi, rax        ; format
.text:0000000000402015                 mov     eax, 0
.text:000000000040201A                 call    _printf
.text:000000000040201F                 lea     rax, [rbp+var_2C]
.text:0000000000402023                 mov     rsi, rax
.text:0000000000402026                 lea     rax, aStackaddrP ; "stackaddr:%p\n"
.text:000000000040202D                 mov     rdi, rax        ; format
.text:0000000000402030                 mov     eax, 0
.text:0000000000402035                 call    _printf
.text:0000000000402035 ;   } // starts at 401FAD
.text:000000000040203A                 call    ___cxa_end_catch
.text:000000000040203F                 jmp     loc_401ED4
.text:0000000000402044 ; ---------------------------------------------------------------------------
.text:0000000000402044 ;   cleanup() // owned by 401FAD
```

其中`visit`用于创造堆块,编辑堆块,但是`create`把输入大小与`malloc`分为两个单独的步骤

且`size`会检查整形溢出,`edit`不会检查下标
