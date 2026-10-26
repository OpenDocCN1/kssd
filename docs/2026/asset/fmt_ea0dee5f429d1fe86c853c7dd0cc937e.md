
`CODE:007B27C2`                                        ; `DATA:009D8730 Key1`  dd `95AF0D1Dh`, `0D66C365Eh`, `46F457DAh`, `0C214D7D5h`

`CODE:007B27C2`                                         ;

### 函数原型
`CODE:007B27C2`                                         ; function `CheckRegCode2bytes(Key: TKey128; RegCodeNumber: array of byte): boolean`;

```asm
CODE:007B27C7                 test    al, al
CODE:007B27C9                 jz      short loc_7B280B
CODE:007B27CB                 lea     edx, [ebp+RegCodeNumber]
CODE:007B27CE                 mov     eax, offset Key1
CODE:007B27D3                 call    GetRegcodeNumberLast4Bytes ; 用Lockbox Quick cipher 解码RegCodeNumber
```
`CODE:007B27D3`                                         ; 并检查解码后的`RegCodeNumber`的前2个字节
`CODE:007B27D3`                                         ; 是否为`5B9C`
`CODE:007B27D3`                                         ; 如果为`5B9C`则返回解码后的`RegCodeNumber`的后4个字节
`CODE:007B27D3`                                         ; 否则返回0
`CODE:007B27D3`                                         ;
`CODE:007B27D3`                                         ; `Key`为:
`CODE:007B27D3`                                         ; `DATA:009D8730 Key1`  dd `95AF0D1Dh`, `0D66C365Eh`, `46F457DAh`, `0C214D7D5h`
`CODE:007B27D3`                                         ;
### 函数原型
`CODE:007B27D3`                                         ; function `GetRegcodeNumberLast4Bytes(Key: TKey128; RegCodeNumber: array of byte): DWORD`;

```asm
CODE:007B27D8                 mov     ebx, eax
CODE:007B27DA                 lea     eax, [ebp+var_244]
CODE:007B27E0                 lea     edx, [ebp+User]
CODE:007B27E6                 call    PChar           ; Borland Visual Component Library & Packages
CODE:007B27EB                 mov     eax, [ebp+var_244]
CODE:007B27F1                 lea     edx, [ebp+var_240]
CODE:007B27F7                 call    EnCodeUser      ; 编码用户名
```
`CODE:007B27F7`                                         ; 分下面5个步骤
`CODE:007B27F7`                                         ; 1. 构建码表 `buf`
`CODE:007B27F7`                                         ; 把上面运算得到的3个字符串连接起来
`CODE:007B27F7`                                         ; `0012F61C`    `01B05BF0`  ASCII "zyxwvutsrqpo"
`CODE:007B27F7`                                         ; `0012F614`    `01B05C0C`  ASCII "HIJKLMNOPQRSTUV"
`CODE:007B27F7`                                         ; `0012F610`    `015B083C`  ASCII "0987654321"
`CODE:007B27F7`                                         ;
`CODE:007B27F7`                                         ; 2.如果用户名长度等于`17H`，则把用户转为大写
`CODE:007B27F7`                                         ; 如果用户名长度大于`17H`，则通过一定的规则取其输入
`CODE:007B27F7`                                         ; 的用户名，组合成`17H`长度，全部字母转为大写
`CODE:007B27F7`                                         ; 如果用户名长度不足`17H`，则通过一个补码表把用户名补足
`CODE:007B27F7`                                         ; `17H`长度，全部字母转为大写
`CODE:007B27F7`                                         ;
`CODE:007B27F7`                                         ; 转换后的用户名存入`user1`
`CODE:007B27F7`                                         ;
`CODE:007B27F7`                                         ; 3. 把`user1`的用户名，逐个字母的Ascii码转成`Str`，并存入
`CODE:007B27F7`                                         ; `user2`:
```pascal
for i := 1 to Length(user1) do
begin
  user2 := user2 + IntToStr(Ord(user1[i]));
end;
```
`CODE:007B27F7`                                         ;
`CODE:007B27F7`                                         ; 4. 以`user2`中5个字母为一组，计算索引`AIndex`
`CODE:007B27F7`                                         ;    从`buf[AIndex]`选出字母，组成一个新的中间用户名`user3`:
```pascal
while i <= Length(user2) do
begin
  s := copy(user2, i, 5);
  w := StrToInt(s);
  s1 := '';
  if w > 0 then
  repeat
    a := w div $25;
    AIndex := w mod $25;
    s1 := buf[AIndex+1] + s1;
    w := a;
  until a = 0;
  user3 := user3 + s1;
  i := i + 5;
end;
```
`CODE:007B27F7`                                         ;
`CODE:007B27F7`                                         ; 5. 从`user3`中，通过一定的运算，组合出最终编码过后的用户名，
`CODE:007B27F7`                                         ; 新用户名长度为`0CH`
```pascal
userLen := Length(user3);
result := '';
for i := 1 to $c do
begin
  AIndex := i * userLen div $C;
  result := result + UpperCase(user3[AIndex]);
end;
```

```asm
CODE:007B27FC                 mov     eax, [ebp+var_240]
CODE:007B2802                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个dword值
```
`CODE:007B2802`                                         ;
`CODE:007B2802`                                         ; 算法很简单，直接列出来吧:
```asm
Encode          proc near
     push    ebx
     push    esi
     xor     ecx, ecx
     mov     ebx, edx
     dec     ebx
     test    ebx, ebx
     jl      short loc_7B2088
     inc     ebx

loc_7B2064:
     shl     ecx, 4
     xor     edx, edx
     mov     dl, [eax]
     add     ecx, edx
     mov     edx, ecx
     and     edx, 0F0000000h
     test    edx, edx
     jz      short loc_7B2080
     mov     esi, edx
     shr     esi, 18h
     xor     ecx, esi

loc_7B2080:
```
