
### 汇编代码与注释分析

#### 1. 用户名计算逻辑
```
CODE:007B27F7                              ; AIndex := i * userLen div $C;  

CODE:007B27F7                              ; result := result + UpperCase(user3[AIndex]);  

CODE:007B27F7                              ;   end;  

CODE:007B27F7                              ;  
```

#### 2. ComputeValue 调用
```
CODE:007B27FC       mov     eax, [ebp+var_240]  

CODE:007B2802       call    ComputeValue    ; 通过一定的算法计算用户名的值，得到一个 dword 值  
```

#### 3. Encode 算法实现
算法很简单，直接列出来吧:

```
CODE:007B2802                              ; Encode         proc near  

CODE:007B2802                              ;      push    ebx  

CODE:007B2802                              ;      push    esi  

CODE:007B2802                              ;      xor     ecx, ecx  

CODE:007B2802                              ;      mov     ebx, edx  

CODE:007B2802                              ;      dec     ebx  

CODE:007B2802                              ;      test    ebx, ebx  

CODE:007B2802                              ;      jl      short loc_7B2088  

CODE:007B2802                              ;      inc     ebx  

CODE:007B2802                              ;  

CODE:007B2802                              ; loc_7B2064:  

CODE:007B2802                              ;      shl     ecx, 4  

CODE:007B2802                              ;      xor     edx, edx  

CODE:007B2802                              ;      mov     dl, [eax]  

CODE:007B2802                              ;      add     ecx, edx  

CODE:007B2802                              ;      mov     edx, ecx  

CODE:007B2802                              ;      and     edx, 0F0000000h  

CODE:007B2802                              ;      test    edx, edx  

CODE:007B2802                              ;      jz      short loc_7B2080  

CODE:007B2802                              ;      mov     esi, edx  

CODE:007B2802                              ;      shr     esi, 18h  

CODE:007B2802                              ;      xor     ecx, esi  

CODE:007B2802                              ;  

CODE:007B2802                              ; loc_7B2080:  

CODE:007B2802                              ;      not     edx  

CODE:007B2802                              ;      and     ecx, edx  

CODE:007B2802                              ;      inc     eax  

CODE:007B2802                              ;      dec     ebx  

CODE:007B2802                              ;      jnz     short loc_7B2064  

CODE:007B2802                              ;  

CODE:007B2802                              ; loc_7B2088:  

CODE:007B2802                              ;      mov     eax, ecx  

CODE:007B2802                              ;      pop     esi  

CODE:007B2802                              ;      pop     ebx  

CODE:007B2802                              ;      retn  

CODE:007B2802                              ; Encode         endp  
```

**函数原型**: `function ComputeValue(username: string): DWORD;`

#### 4. 注册码验证逻辑
```
CODE:007B2807       cmp     ebx, eax        ; 如果计算得到的用户名的值等于解码后的 RegCodeNumber 的后 4 个字节  

CODE:007B2802                              ; 那么注册成功  

CODE:007B2809       jz      short loc_7B280F  

CODE:007B280B  

CODE:007B280B loc_7B280B:                   ; CODE XREF: CheckRegCode+2F5j  

CODE:007B280B       xor     ebx, ebx  

CODE:007B280D       jmp     short @Exit     ; default  

CODE:007B280F ; -------------------------------------------------------------------------------?  

CODE:007B280F  

CODE:007B280F loc_7B280F:                   ; CODE XREF: CheckRegCode+335j  

CODE:007B280F       mov     bl, 1  

CODE:007B2811       jmp     short @Exit     ; default  

CODE:007B2813 ; -------------------------------------------------------------------------------?  

CODE:007B2813  

CODE:007B2813 loc_7B2813:                   ; CODE XREF: CheckRegCode+85j  

CODE:007B2813                              ; DATA XREF: CheckRegCode:off_7B2560o  

CODE:007B2813       lea     edx, [ebp+RegCodeNumber] ; case 0x7  

CODE:007B2816       mov     eax, offset Key1  

CODE:007B281B       call    CheckRegCode2bytes ; 用 Lockbox Quick cipher 解码 RegCodeNumber  

CODE:007B281B                              ; 并检查解码后的 RegCodeNumber 的前 2 个字节  

CODE:007B281B                              ; 是否为 5B9C  

CODE:007B281B                              ; 如果为 5B9C 则返回 true  

CODE:007B281B                              ; 否则返回 false  

CODE:007B281B                              ;  

CODE:007B281B                              ; Key 为:  

CODE:007B281B                              ; DATA:009D8730 Key1  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h  

CODE:007B281B                              ;  

CODE:007B281B                              ; 函数原型:  

CODE:007B281B                              ; function CheckRegCode2bytes(Key: TKey128; RegCodeNumber: array of byte): boolean;  

CODE:007B2820       test    al, al  

CODE:007B2822       jz      short loc_7B2864  

CODE:007B2824       lea     edx, [ebp+RegCodeNumber]  

CODE:007B2827       mov     eax, offset Key1  

CODE:007B282C       call    GetRegcodeNumberLast4Bytes ; 用 Lockbox Quick cipher 解码 RegCodeNumber  

CODE:007B282C                              ; 并检查解码后的 RegCodeNumber 的前 2 个字节  

CODE:007B282C                              ; 是否为 5B9C  

CODE:007B282C                              ; 如果为 5B9C 则返回解码后的 RegCodeNumber 的后 4 个字节  

CODE:007B282C                              ; 否则返回 0  

CODE:007B282C                              ;  

CODE:007B282C                              ; Key 为:  

CODE:007B282C                              ; DATA:009D8730 Key1  dd 95AF0D1Dh, 0D66C365Eh, 46F457DAh, 0C214D7D5h  

CODE:007B282C                              ;  

CODE:007B282C                              ; 函数原型:  

CODE:007B282C                              ; function GetRegcodeNumberLast4Bytes(Key: TKey128; RegCodeNumber: array of byte): DWORD;  

CODE:007B2831       mov     ebx, eax  

CODE:007B2833       lea     eax, [ebp+var_24C]  

CODE:007B2839       lea     edx, [ebp+User]  

CODE:007B283F       call    PChar         ; Borland Visual Component Library & Packages  

CODE:007B2844       mov     eax, [ebp+var_24C]  

CODE:007B284A       lea     edx, [ebp+var_248]  

CODE:007B2850       call    EnCodeUser    ; 编码用户名  

CODE:007B2850                              ; 分下面 5 个步骤  

CODE:007B2850                              ; 1. 构建码表 buf  

CODE:007B2850                              ; 把上面运算得到的 3 个字符串连接起来  

CODE:007B2850                              ; 0012F61C    01B05BF0  ASCII "zyxwvutsrqpo"  

CODE:007B2850                              ; 0012F614    01B05C0C  ASCII "HIJKLMNOPQRSTUV"  
```
