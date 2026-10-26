
`CODE:007B2739`                                         ; 5. 从`user3`中，通过一定的运算，组合出最终编码过后的用户名，

`CODE:007B2739`                                         ; 新用户名长度为0CH

`CODE:007B2739`                                         ;

`CODE:007B2739`                                         ;   `userLen` := Length(`user3`);

`CODE:007B2739`                                         ;   `result` := '';

`CODE:007B2739`                                         ;   for `i` := 1 to `$c` do

`CODE:007B2739`                                         ;   begin

`CODE:007B2739`                                         ;     `AIndex` := `i` \* `userLen` div `$C`;

`CODE:007B2739`                                         ;     `result` := `result` + `UpperCase`(`user3`[`AIndex`]);

`CODE:007B2739`                                         ;   end;

`CODE:007B2739`                                         ;

```
CODE:007B273E                 mov     eax, [ebp+var_230]
CODE:007B2744                 call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个dword值
```

`CODE:007B2744`                                         ; 算法很简单，直接列出来吧:

### `Encode` 过程

```
CODE:007B2744                                         ; Encode          proc near
CODE:007B2744                                         ;      push    ebx
CODE:007B2744                                         ;      push    esi
CODE:007B2744                                         ;      xor     ecx, ecx
CODE:007B2744                                         ;      mov     ebx, edx
CODE:007B2744                                         ;      dec     ebx
CODE:007B2744                                         ;      test    ebx, ebx
CODE:007B2744                                         ;      jl      short loc_7B2088
CODE:007B2744                                         ;      inc     ebx
CODE:007B2744                                         ;
CODE:007B2744                                         ; loc_7B2064:
CODE:007B2744                                         ;      shl     ecx, 4
CODE:007B2744                                         ;      xor     edx, edx
CODE:007B2744                                         ;      mov     dl, [eax]
CODE:007B2744                                         ;      add     ecx, edx
CODE:007B2744                                         ;      mov     edx, ecx
CODE:007B2744                                         ;      and     edx, 0F0000000h
CODE:007B2744                                         ;      test    edx, edx
CODE:007B2744                                         ;      jz      short loc_7B2080
CODE:007B2744                                         ;      mov     esi, edx
CODE:007B2744                                         ;      shr     esi, 18h
CODE:007B2744                                         ;      xor     ecx, esi
CODE:007B2744                                         ;
CODE:007B2744                                         ; loc_7B2080:
CODE:007B2744                                         ;      not     edx
CODE:007B2744                                         ;      and     ecx, edx
CODE:007B2744                                         ;      inc     eax
CODE:007B2744                                         ;      dec     ebx
CODE:007B2744                                         ;      jnz     short loc_7B2064
CODE:007B2744                                         ;
CODE:007B2744                                         ; loc_7B2088:
CODE:007B2744                                         ;      mov     eax, ecx
CODE:007B2744                                         ;      pop     esi
CODE:007B2744                                         ;      pop     ebx
CODE:007B2744                                         ;      retn
CODE:007B2744                                         ; Encode          endp
```

`CODE:007B2744`                                         ;

函数原型: `function ComputeValue(username: string): DWORD`;

```
CODE:007B2749                 cmp     ebx, eax        ; 如果计算得到的用户名的值等于解码后的RegCodeNumber的后4个字节
CODE:007B2749                                         ; 那么注册成功
CODE:007B274B                 jz      short loc_7B2754
CODE:007B274D
CODE:007B274D loc_7B274D:                             ; CODE XREF: CheckRegCode+237j
CODE:007B274D                 xor     ebx, ebx
CODE:007B274F                 jmp     @Exit           ; default
CODE:007B2754 ; -------------------------------------------------------------------------------?
CODE:007B2754
CODE:007B2754 loc_7B2754:                             ; CODE XREF: CheckRegCode+277j
CODE:007B2754                 mov     bl, 1
CODE:007B2756                 jmp     @Exit           ; default
CODE:007B275B ; -------------------------------------------------------------------------------?
CODE:007B275B
CODE:007B275B loc_7B275B:                             ; CODE XREF: CheckRegCode+85j
CODE:007B275B                                         ; DATA XREF: CheckRegCode:off_7B2560o
CODE:007B275B                 lea     edx, [ebp+RegCodeNumber] ; case 0x5
CODE:007B275E                 mov     eax, offset Key1
CODE:007B2763                 call    CheckRegCode2bytes ; 用Lockbox Quick cipher 解码RegCodeNumber
CODE:007B2763                                         ; 并检查解码后的RegCodeNumber的前2个字节
CODE:007B2763                                         ; 是否为5B9C
CODE:007B2763                                         ; 如果为5B9C则返回true
CODE:007B2763                                         ; 否则返回false
CODE:007B2763                                         ;
CODE:007B2763                                         ; Key为:
CODE:007B2763                                         ; DATA:009D8730 Key1  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h
CODE:007B2763                                         ;
CODE:007B2763                                         ; 函数原型:
CODE:007B2763                                         ; function CheckRegCode2bytes(Key: TKey128; RegCodeNumber: array of byte): boolean;
CODE:007B2768                 test    al, al
CODE:007B276A                 jz      short loc_7B27AC
CODE:007B276C                 lea     edx, [ebp+RegCodeNumber]
CODE:007B276F                 mov     eax, offset Key1
CODE:007B2774                 call    GetRegcodeNumberLast4Bytes ; 取出解码后的RegCode的后4个字节
CODE:007B2779                 mov     ebx, eax
CODE:007B277B                 lea     eax, [ebp+var_23C]
CODE:007B2781                 lea     edx, [ebp+User]
CODE:007B2787                 call    PChar           ; Borland Visual Component Library & Packages
CODE:007B278C                 mov     eax, [ebp+var_23C]
CODE:007B2792                 lea     edx, [ebp+var_238]
CODE:007B2798                 call    EnCodeUser      ; 编码用户名
```

`CODE:007B2798`                                         ; 分下面5个步骤

`CODE:007B2798`                                         ; 1. 构建码表 buf

`CODE:007B2798`                                         ; 把上面运算得到的3个字符串连接起来

`CODE:007B2798`                                         ; 0012F61C    01B05BF0  ASCII "zyxwvutsrqpo"

`CODE:007B2798`                                         ; 0012F614    01B05C0C  ASCII "HIJKLMNOPQRSTUV"

`CODE:007B2798`                                         ; 0012F610    015B083C  ASCII "0987654321"

`CODE:007B2798`                                         ;

`CODE:007B2798`                                         ; 2.如果用户名长度等于17H，则把用户转为大写

`CODE:007B2798`                                         ; 如果用户名长度大于17H，则通过一定的规则取其输入

`CODE:007B2798`                                         ; 的用户名，组合成17H长度，全部字母转为大写

`CODE:007B2798`                                         ; 如果用户名长度不足17H，则通过一个补码表把用户名补足

`CODE:007B2798`                                         ; 17H长度，全部字母转为大写
