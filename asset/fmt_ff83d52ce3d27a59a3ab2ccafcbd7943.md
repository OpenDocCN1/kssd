
```
CODE:007B2850                                                                                                  ; 0012F610    015B083C  ASCII "0987654321"

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ; 2.如果用户名长度等于 17H，则把用户转为大写

CODE:007B2850                                                                                                  ; 如果用户名长度大于 17H，则通过一定的规则取其输入

CODE:007B2850                                                                                                  ; 的用户名，组合成 17H 长度，全部字母转为大写

CODE:007B2850                                                                                                  ; 如果用户名长度不足 17H，则通过一个补码表把用户名补足

CODE:007B2850                                                                                                  ; 17H 长度，全部字母转为大写

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ; 转换后的用户名存入 user1

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ; 3. 把 user1 的用户名，逐个字母的 Ascii 码转成 Str，并存入

CODE:007B2850                                                                                                  ; user2:

CODE:007B2850                                                                                                  ;   for i := 1 to Length(user1) do

CODE:007B2850                                                                                                  ;   begin

CODE:007B2850                                                                                                  ;     user2 := user2 + IntToStr(Ord(user1[i]));

CODE:007B2850                                                                                                  ;   end;

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ; 4. 以 user2 中 5 个字母为一组，计算索引 AIndex

CODE:007B2850                                                                                                  ;    从 buf[AIndex]选出字母，组成一个新的中间用户名 user3:

CODE:007B2850                                                                                                  ;   while i <= Length(user2) do

CODE:007B2850                                                                                                  ;   begin

CODE:007B2850                                                                                                  ;     s := copy(user2, i, 5);

CODE:007B2850                                                                                                  ;     w := StrToInt(s);

CODE:007B2850                                                                                                  ;     s1 := '';

CODE:007B2850                                                                                                  ;     if w > 0 then

CODE:007B2850                                                                                                  ;     repeat

CODE:007B2850                                                                                                  ;       a := w div $25;

CODE:007B2850                                                                                                  ;       AIndex := w mod $25;

CODE:007B2850                                                                                                  ;       s1 := buf[AIndex+1] + s1;

CODE:007B2850                                                                                                  ;       w := a;

CODE:007B2850                                                                                                  ;     until a = 0;

CODE:007B2850                                                                                                  ;     user3 := user3 + s1;

CODE:007B2850                                                                                                  ;     i := i + 5;

CODE:007B2850                                                                                                  ;   end;

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ; 5. 从 user3 中，通过一定的运算，组合出最终编码过后的用户名，

CODE:007B2850                                                                                                  ; 新用户名长度为 0CH

CODE:007B2850                                                                                                  ;

CODE:007B2850                                                                                                  ;   userLen := Length(user3);

CODE:007B2850                                                                                                  ;   result := '';

CODE:007B2850                                                                                                  ;   for i := 1 to $c do

CODE:007B2850                                                                                                  ;   begin

CODE:007B2850                                                                                                  ;     AIndex := i * userLen div $C;

CODE:007B2850                                                                                                  ;     result := result + UpperCase(user3[AIndex]);

CODE:007B2850                                                                                                  ;   end;

CODE:007B2850                                                                                                  ;

CODE:007B2855                 mov     eax, [ebp+var_248]

CODE:007B285B                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个 dword 值

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ; 算法很简单，直接列出来吧:

CODE:007B285B                                                                                                  ; Encode          proc near

CODE:007B285B                                                                                                  ;     push    ebx

CODE:007B285B                                                                                                  ;     push    esi

CODE:007B285B                                                                                                  ;     xor     ecx, ecx

CODE:007B285B                                                                                                  ;     mov     ebx, edx

CODE:007B285B                                                                                                  ;     dec     ebx

CODE:007B285B                                                                                                  ;     test    ebx, ebx

CODE:007B285B                                                                                                  ;     jl      short loc_7B2088

CODE:007B285B                                                                                                  ;     inc     ebx

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ; loc_7B2064:

CODE:007B285B                                                                                                  ;     shl     ecx, 4

CODE:007B285B                                                                                                  ;     xor     edx, edx

CODE:007B285B                                                                                                  ;     mov     dl, [eax]

CODE:007B285B                                                                                                  ;     add     ecx, edx

CODE:007B285B                                                                                                  ;     mov     edx, ecx

CODE:007B285B                                                                                                  ;     and     edx, 0F0000000h

CODE:007B285B                                                                                                  ;     test    edx, edx

CODE:007B285B                                                                                                  ;     jz      short loc_7B2080

CODE:007B285B                                                                                                  ;     mov     esi, edx

CODE:007B285B                                                                                                  ;     shr     esi, 18h

CODE:007B285B                                                                                                  ;     xor     ecx, esi

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ; loc_7B2080:

CODE:007B285B                                                                                                  ;     not     edx

CODE:007B285B                                                                                                  ;     and     ecx, edx

CODE:007B285B                                                                                                  ;     inc     eax

CODE:007B285B                                                                                                  ;     dec     ebx

CODE:007B285B                                                                                                  ;     jnz     short loc_7B2064

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ; loc_7B2088:

CODE:007B285B                                                                                                  ;     mov     eax, ecx

CODE:007B285B                                                                                                  ;     pop     esi

CODE:007B285B                                                                                                  ;     pop     ebx

CODE:007B285B                                                                                                  ;     retn

CODE:007B285B                                                                                                  ; Encode          endp

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ;

CODE:007B285B                                                                                                  ; 函数原型: function ComputeValue(username: string): DWORD;

CODE:007B285B                                                                                                  ;

CODE:007B2860                 cmp     ebx, eax        ; 如果计算得到的用户名的值等于解码后的 RegCodeNumber 的后 4 个字节

CODE:007B2860                                                                                                  ; 那么注册成功

CODE:007B2862                 jz      short loc_7B2868

CODE:007B2864

CODE:007B2864 loc_7B2864:                                                    ; CODE XREF: CheckRegCode+34Ej

CODE:007B2864                 xor     ebx, ebx

CODE:007B2866                 jmp     short @Exit     ; default

CODE:007B2868 ; -----------------------------------------------------------------------------?

CODE:007B2868

CODE:007B2868 loc_7B2868:                                                    ; CODE XREF: CheckRegCode+38Ej

CODE:007B2868                 mov     bl, 1

CODE:007B286A

CODE:007B286A @Exit:                                                        ; CODE XREF: CheckRegCode+7Fj

CODE:007B286A                                                                                                  ; CheckRegCode+FFj ...
```
