
```text
CODE:007B2850                                         ; 5. 从user3中，通过一定的运算，组合出最终编码过后的用户名，
CODE:007B2850                                         ; 新用户名长度为0CH
CODE:007B2850                                         ;
CODE:007B2850                                         ;   userLen := Length(user3);
CODE:007B2850                                         ;   result := '';
CODE:007B2850                                         ;   for i := 1 to $c do
CODE:007B2850                                         ;   begin
CODE:007B2850                                         ;     AIndex := i * userLen div $C;
CODE:007B2850                                         ;     result := result + UpperCase(user3[AIndex]);
CODE:007B2850                                         ;   end;
CODE:007B2850                                         ;
CODE:007B2855                 mov     eax, [ebp+var_248]
CODE:007B285B                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个dword值
CODE:007B285B                                         ;
CODE:007B285B                                         ; 算法很简单，直接列出来吧:
CODE:007B285B                                         ; Encode          proc near
CODE:007B285B                                         ;      push    ebx
CODE:007B285B                                         ;      push    esi
CODE:007B285B                                         ;      xor     ecx, ecx
CODE:007B285B                                         ;      mov     ebx, edx
CODE:007B285B                                         ;      dec     ebx
CODE:007B285B                                         ;      test    ebx, ebx
CODE:007B285B                                         ;      jl      short loc_7B2088
CODE:007B285B                                         ;      inc     ebx
CODE:007B285B                                         ;
CODE:007B285B                                         ; loc_7B2064:
CODE:007B285B                                         ;      shl     ecx, 4
CODE:007B285B                                         ;      xor     edx, edx
CODE:007B285B                                         ;      mov     dl, [eax]
CODE:007B285B                                         ;      add     ecx, edx
CODE:007B285B                                         ;      mov     edx, ecx
CODE:007B285B                                         ;      and     edx, 0F0000000h
CODE:007B285B                                         ;      test    edx, edx
CODE:007B285B                                         ;      jz      short loc_7B2080
CODE:007B285B                                         ;      mov     esi, edx
CODE:007B285B                                         ;      shr     esi, 18h
CODE:007B285B                                         ;      xor     ecx, esi
CODE:007B285B                                         ;
CODE:007B285B                                         ; loc_7B2080:
CODE:007B285B                                         ;      not     edx
CODE:007B285B                                         ;      and     ecx, edx
CODE:007B285B                                         ;      inc     eax
CODE:007B285B                                         ;      dec     ebx
CODE:007B285B                                         ;      jnz     short loc_7B2064
CODE:007B285B                                         ;
CODE:007B285B                                         ; loc_7B2088:
CODE:007B285B                                         ;      mov     eax, ecx
CODE:007B285B                                         ;      pop     esi
CODE:007B285B                                         ;      pop     ebx
CODE:007B285B                                         ;      retn
CODE:007B285B                                         ; Encode          endp
CODE:007B285B                                         ;
CODE:007B285B                                         ;
CODE:007B285B                                         ;
CODE:007B285B                                         ; 函数原型: function ComputeValue(username: string): DWORD;
CODE:007B285B                                         ;
CODE:007B2860                 cmp     ebx, eax        ; 如果计算得到的用户名的值等于解码后的RegCodeNumber的后4个字节
CODE:007B2860                                         ; 那么注册成功
CODE:007B2862                 jz      short loc_7B2868
CODE:007B2864
CODE:007B2864 loc_7B2864:                             ; CODE XREF: CheckRegCode+34Ej
CODE:007B2864                 xor     ebx, ebx
CODE:007B2866                 jmp     short @Exit     ; default
CODE:007B2868 ; -------------------------------------------------------------------------------?
CODE:007B2868
CODE:007B2868 loc_7B2868:                             ; CODE XREF: CheckRegCode+38Ej
CODE:007B2868                 mov     bl, 1
CODE:007B286A
CODE:007B286A @Exit:                                  ; CODE XREF: CheckRegCode+7Fj
CODE:007B286A                                         ; CheckRegCode+FFj ...
CODE:007B286A                 xor     eax, eax        ; default
CODE:007B286C                 pop     edx
CODE:007B286D                 pop     ecx
CODE:007B286E                 pop     ecx
CODE:007B286F                 mov     fs:[eax], edx
CODE:007B2872                 push    offset loc_7B288F
CODE:007B2877
CODE:007B2877 loc_7B2877:                             ; CODE XREF: CODE:007B288Dj
CODE:007B2877                 lea     eax, [ebp+var_24C]
CODE:007B287D                 mov     edx, 11h
CODE:007B2882                 call    System::__linkproc__ LStrArrayClr(void)
CODE:007B2887                 retn
CODE:007B2887 CheckRegCode    endp ; sp = -20h
```

至此，注册算法我们已经分析完毕, 这个软件的注册算法中对用户名的运算比较复杂点，其他都一般，注册算法用简单的伪码表示为：

```text
if Left(decrypt(regcode), 2) = 5B9C then
   if ComputeValue(EnCodeUser(user)) = Right(decrypt(regcode), 4) then
      注册成功
   else
      注册失败
else
  注册失败
```

注册码的解密`Key`为：

```text
Key  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h
```

注册码必须为16个字符

---

## 算法注册机

现在可以写出注册机了：

```text
const
  a09 = '1234567890';
  aAZBig = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  aAZSmall = 'abcdefghijklmnopqrstuvwxyz';

function f2(newuser: string): DWORD;

var
  i: integer;
  l: integer;
  c: byte;
  r, k, n: DWORD;

begin
  l := Length(newuser);
  r := 0;
  k := 0;
  for i := 1 to l do
  begin
    r := r shl 4;
    c := Ord(newuser[i]);
    r := r + c;
    k := r and $F0000000;
    if k <> 0 then
    begin
      n := k shr $18;
      r := r xor n;
    end;
    r := r and (not k);
  end;
  result := r;
end;

function ComputUsernameValue(Username: string): string;

var
  buf, buf1, buf2, buf3: string;
  userLen: DWORD;
  i: integer;
  d: integer;
  a, b, c: integer;
  k: integer;
  w: smallint;
  s, s1, s2, s3: string;
  AIndex: integer;
  aUser: string;

begin
  buf1 := ReverseString(a09);
  buf2 := Copy(ReverseString(aAZSmall), 1, $c);
  buf3 := Copy(aAZBig, 8, $F);
  buf := buf2 + buf3 + buf1;
  aUser := Username;
  userLen := Length(aUser);
  if userLen < $17 then
  begin
    k := 0;
    s1 := '';
    for i := 1 to $17 do
    begin
      d := i;
      d := d * userLen;
      a := d div $17;
      if a = 0 then
      begin
        a := 1;
      end;
      if a = k then
      begin
        b := i;
        c := $25 - b;
        s1 := s1 + UpperCase(buf[c]);
      end
      else
      begin
        s := aUser[a];
        s := UpperCase(s);
        s1 := s1 + s;
        k := a;
      end;
    end;
  end
  else if userLen = $17 then
  begin
    for i := 1 to $17 do
    begin
      s1 := s1 + UpperCase(aUser[i]);
    end;
  end
  else
  begin
    for i := 1 to $17 do
    begin
      AIndex := i * userLen div $17;
      s1 := s1 + UpperCase(aUser[AIndex]);
    end;
  end;
  s2 := '';
  for i := 1 to $17 do
  begin
    s2 := s2 + IntToStr(Ord(s1[i]));
  end;
  i := 1;
```
