
```
CODE:007B26A5                                         ; DATA:009D8730 Key1  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h

CODE:007B26A5                                         ;

CODE:007B26A5                                         ; 函数原型:

CODE:007B26A5                                         ; function CheckRegCode2bytes(Key: TKey128; RegCodeNumber: array of byte): boolean;

CODE:007B26AA                 test    al, al

CODE:007B26AC                 jz      short loc\_7B26EE

CODE:007B26AE                 lea     edx, [ebp+RegCodeNumber]

CODE:007B26B1                 mov     eax, offset Key1

CODE:007B26B6                 call    GetRegcodeNumberLast4Bytes ; 用Lockbox Quick cipher 解码RegCodeNumber

CODE:007B26B6                                         ; 并检查解码后的RegCodeNumber的前2个字节

CODE:007B26B6                                         ; 是否为5B9C

CODE:007B26B6                                         ; 如果为5B9C则返回解码后的RegCodeNumber的后4个字节

CODE:007B26B6                                         ; 否则返回0

CODE:007B26B6                                         ;

CODE:007B26B6                                         ; Key为:

CODE:007B26B6                                         ; DATA:009D8730 Key1  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h

CODE:007B26B6                                         ;

CODE:007B26B6                                         ; 函数原型:

CODE:007B26B6                                         ; function GetRegcodeNumberLast4Bytes(Key: TKey128; RegCodeNumber: array of byte): DWORD;

CODE:007B26BB                 mov     ebx, eax

CODE:007B26BD                 lea     eax, [ebp+var\_22C]

CODE:007B26C3                 lea     edx, [ebp+User]

CODE:007B26C9                 call    PChar           ; Borland Visual Component Library & Packages

CODE:007B26CE                 mov     eax, [ebp+var\_22C]

CODE:007B26D4                 lea     edx, [ebp+var\_228]

CODE:007B26DA                 call    EnCodeUser      ; 编码用户名

CODE:007B26DA                                         ; 分下面5个步骤

CODE:007B26DA                                         ; 1. 构建码表 buf

CODE:007B26DA                                         ; 把上面运算得到的3个字符串连接起来

CODE:007B26DA                                         ; 0012F61C    01B05BF0  ASCII "zyxwvutsrqpo"

CODE:007B26DA                                         ; 0012F614    01B05C0C  ASCII "HIJKLMNOPQRSTUV"

CODE:007B26DA                                         ; 0012F610    015B083C  ASCII "0987654321"

CODE:007B26DA                                         ;

CODE:007B26DA                                         ; 2.如果用户名长度等于17H，则把用户转为大写

CODE:007B26DA                                         ; 如果用户名长度大于17H，则通过一定的规则取其输入

CODE:007B26DA                                         ; 的用户名，组合成17H长度，全部字母转为大写

CODE:007B26DA                                         ; 如果用户名长度不足17H，则通过一个补码表把用户名补足

CODE:007B26DA                                         ; 17H长度，全部字母转为大写

CODE:007B26DA                                         ;

CODE:007B26DA                                         ; 转换后的用户名存入user1

CODE:007B26DA                                         ;

CODE:007B26DA                                         ; 3. 把user1的用户名，逐个字母的Ascii码转成Str，并存入

CODE:007B26DA                                         ; user2:

CODE:007B26DA                                         ;   for i := 1 to Length(user1) do

CODE:007B26DA                                         ;   begin

CODE:007B26DA                                         ;     user2 := user2 + IntToStr(Ord(user1[i]));

CODE:007B26DA                                         ;   end;

CODE:007B26DA                                         ;

CODE:007B26DA                                         ; 4. 以user2中5个字母为一组，计算索引AIndex

CODE:007B26DA                                         ;    从buf[AIndex]选出字母，组成一个新的中间用户名user3:

CODE:007B26DA                                         ;   while i <= Length(user2) do

CODE:007B26DA                                         ;   begin

CODE:007B26DA                                         ;     s := copy(user2, i, 5);

CODE:007B26DA                                         ;     w := StrToInt(s);

CODE:007B26DA                                         ;     s1 := '';

CODE:007B26DA                                         ;     if w > 0 then

CODE:007B26DA                                         ;     repeat

CODE:007B26DA                                         ;       a := w div $25;

CODE:007B26DA                                         ;       AIndex := w mod $25;

CODE:007B26DA                                         ;       s1 := buf[AIndex+1] + s1;

CODE:007B26DA                                         ;       w := a;

CODE:007B26DA                                         ;     until a = 0;

CODE:007B26DA                                         ;     user3 := user3 + s1;

CODE:007B26DA                                         ;     i := i + 5;

CODE:007B26DA                                         ;   end;

CODE:007B26DA                                         ;

CODE:007B26DA                                         ; 5. 从user3中，通过一定的运算，组合出最终编码过后的用户名，

CODE:007B26DA                                         ; 新用户名长度为0CH

CODE:007B26DA                                         ;

CODE:007B26DA                                         ;   userLen := Length(user3);

CODE:007B26DA                                         ;   result := '';

CODE:007B26DA                                         ;   for i := 1 to $c do

CODE:007B26DA                                         ;   begin

CODE:007B26DA                                         ;     AIndex := i \* userLen div $C;

CODE:007B26DA                                         ;     result := result + UpperCase(user3[AIndex]);

CODE:007B26DA                                         ;   end;

CODE:007B26DA                                         ;

CODE:007B26DF                 mov     eax, [ebp+var\_228]

CODE:007B26E5                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个dword值

CODE:007B26E5                                         ;

CODE:007B26E5                                         ; 算法很简单，直接列出来吧:

CODE:007B26E5                                         ; Encode          proc near

CODE:007B26E5                                         ;      push    ebx

CODE:007B26E5                                         ;      push    esi

CODE:007B26E5                                         ;      xor     ecx, ecx

CODE:007B26E5                                         ;      mov     ebx, edx

CODE:007B26E5                                         ;      dec     ebx

CODE:007B26E5                                         ;      test    ebx, ebx

CODE:007B26E5                                         ;      jl      short loc\_7B2088

CODE:007B26E5                                         ;      inc     ebx

CODE:007B26E5                                         ;

CODE:007B26E5                                         ; loc\_7B2064:

CODE:007B26E5                                         ;      shl     ecx, 4

CODE:007B26E5                                         ;      xor     edx, edx

CODE:007B26E5                                         ;      mov     dl, [eax]

CODE:007B26E5                                         ;      add     ecx, edx

CODE:007B26E5                                         ;      mov     edx, ecx

CODE:007B26E5                                         ;      and     edx, 0F0000000h

CODE:007B26E5                                         ;      test    edx, edx

CODE:007B26E5                                         ;      jz      short loc\_7B2080

CODE:007B26E5                                         ;      mov     esi, edx

CODE:007B26E5                                         ;      shr     esi, 18h

CODE:007B26E5                                         ;      xor     ecx, esi

CODE:007B26E5                                         ;

CODE:007B26E5                                         ; loc\_7B2080:
```
