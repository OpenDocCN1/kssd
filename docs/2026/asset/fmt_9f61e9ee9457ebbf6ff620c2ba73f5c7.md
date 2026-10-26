
```
CODE:007B267B                                         ; 转换后的用户名存入user1

CODE:007B267B                                         ;

CODE:007B267B                                         ; 3. 把user1的用户名，逐个字母的Ascii码转成Str，并存入

CODE:007B267B                                         ; user2:

CODE:007B267B                                         ;   for i := 1 to Length(user1) do

CODE:007B267B                                         ;   begin

CODE:007B267B                                         ;     user2 := user2 + IntToStr(Ord(user1[i]));

CODE:007B267B                                         ;   end;

CODE:007B267B                                         ;

CODE:007B267B                                         ; 4. 以user2中5个字母为一组，计算索引AIndex

CODE:007B267B                                         ;    从buf[AIndex]选出字母，组成一个新的中间用户名user3:

CODE:007B267B                                         ;   while i <= Length(user2) do

CODE:007B267B                                         ;   begin

CODE:007B267B                                         ;     s := copy(user2, i, 5);

CODE:007B267B                                         ;     w := StrToInt(s);

CODE:007B267B                                         ;     s1 := '';

CODE:007B267B                                         ;     if w > 0 then

CODE:007B267B                                         ;     repeat

CODE:007B267B                                         ;       a := w div $25;

CODE:007B267B                                         ;       AIndex := w mod $25;

CODE:007B267B                                         ;       s1 := buf[AIndex+1] + s1;

CODE:007B267B                                         ;       w := a;

CODE:007B267B                                         ;     until a = 0;

CODE:007B267B                                         ;     user3 := user3 + s1;

CODE:007B267B                                         ;     i := i + 5;

CODE:007B267B                                         ;   end;

CODE:007B267B                                         ;

CODE:007B267B                                         ; 5. 从user3中，通过一定的运算，组合出最终编码过后的用户名，

CODE:007B267B                                         ; 新用户名长度为0CH

CODE:007B267B                                         ;

CODE:007B267B                                         ;   userLen := Length(user3);

CODE:007B267B                                         ;   result := '';

CODE:007B267B                                         ;   for i := 1 to $c do

CODE:007B267B                                         ;   begin

CODE:007B267B                                         ;     AIndex := i \* userLen div $C;

CODE:007B267B                                         ;     result := result + UpperCase(user3[AIndex]);

CODE:007B267B                                         ;   end;

CODE:007B2680                 mov     eax, [ebp+var\_220]

CODE:007B2686                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个dword值

CODE:007B2686                                         ;

CODE:007B2686                                         ; 算法很简单，直接列出来吧:

CODE:007B2686                                         ; Encode          proc near

CODE:007B2686                                         ;      push    ebx

CODE:007B2686                                         ;      push    esi

CODE:007B2686                                         ;      xor     ecx, ecx

CODE:007B2686                                         ;      mov     ebx, edx

CODE:007B2686                                         ;      dec     ebx

CODE:007B2686                                         ;      test    ebx, ebx

CODE:007B2686                                         ;      jl      short loc\_7B2088

CODE:007B2686                                         ;      inc     ebx

CODE:007B2686                                         ;

CODE:007B2686                                         ; loc\_7B2064:

CODE:007B2686                                         ;      shl     ecx, 4

CODE:007B2686                                         ;      xor     edx, edx

CODE:007B2686                                         ;      mov     dl, [eax]

CODE:007B2686                                         ;      add     ecx, edx

CODE:007B2686                                         ;      mov     edx, ecx

CODE:007B2686                                         ;      and     edx, 0F0000000h

CODE:007B2686                                         ;      test    edx, edx

CODE:007B2686                                         ;      jz      short loc\_7B2080

CODE:007B2686                                         ;      mov     esi, edx

CODE:007B2686                                         ;      shr     esi, 18h

CODE:007B2686                                         ;      xor     ecx, esi

CODE:007B2686                                         ;

CODE:007B2686                                         ; loc\_7B2080:

CODE:007B2686                                         ;      not     edx

CODE:007B2686                                         ;      and     ecx, edx

CODE:007B2686                                         ;      inc     eax

CODE:007B2686                                         ;      dec     ebx

CODE:007B2686                                         ;      jnz     short loc\_7B2064

CODE:007B2686                                         ;

CODE:007B2686                                         ; loc\_7B2088:

CODE:007B2686                                         ;      mov     eax, ecx

CODE:007B2686                                         ;      pop     esi

CODE:007B2686                                         ;      pop     ebx

CODE:007B2686                                         ;      retn

CODE:007B2686                                         ; Encode          endp

CODE:007B2686                                         ;

CODE:007B2686                                         ;

CODE:007B2686                                         ;

CODE:007B2686                                         ; 函数原型: function ComputeValue(username: string): DWORD;

CODE:007B2686                                         ;

CODE:007B268B                 cmp     ebx, eax        ; 如果计算得到的用户名的值等于解码后的RegCodeNumber的后4个字节

CODE:007B268B                                         ; 那么注册成功

CODE:007B268D                 jz      short loc\_7B2696

CODE:007B268F

CODE:007B268F loc\_7B268F:                             ; CODE XREF: CheckRegCode+179j

CODE:007B268F                 xor     ebx, ebx

CODE:007B2691                 jmp     @Exit           ; default

CODE:007B2696 ; -------------------------------------------------------------------------------?

CODE:007B2696

CODE:007B2696 loc\_7B2696:                             ; CODE XREF: CheckRegCode+1B9j

CODE:007B2696                 mov     bl, 1

CODE:007B2698                 jmp     @Exit           ; default

CODE:007B269D ; -------------------------------------------------------------------------------?

CODE:007B269D

CODE:007B269D loc\_7B269D:                             ; CODE XREF: CheckRegCode+85j

CODE:007B269D                                         ; DATA XREF: CheckRegCode:off\_7B2560o

CODE:007B269D                 lea     edx, [ebp+RegCodeNumber] ; case 0x3

CODE:007B26A0                 mov     eax, offset Key1

CODE:007B26A5                 call    CheckRegCode2bytes ; 用Lockbox Quick cipher 解码RegCodeNumber

CODE:007B26A5                                         ; 并检查解码后的RegCodeNumber的前2个字节

CODE:007B26A5                                         ; 是否为5B9C

CODE:007B26A5                                         ; 如果为5B9C则返回true

CODE:007B26A5                                         ; 否则返回false

CODE:007B26A5                                         ;

CODE:007B26A5                                         ; Key为:
```
