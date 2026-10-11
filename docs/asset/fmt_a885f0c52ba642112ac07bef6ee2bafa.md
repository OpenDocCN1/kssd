
```
CODE:004EDC32                   mov     eax, [ebp+RecvText]

CODE:004EDC35                   call    @System@_16823  ; System::_16823

CODE:004EDC3A                   mov     edi, eax        ; 取字符串内某成员

CODE:004EDC3C                   sub     edi, esi        ; 计算出一个长度

CODE:004EDC3E                   jl      short loc_4EDC69

CODE:004EDC40                   inc     edi             ; 循环 取数据然后连接吧

CODE:004EDC41

CODE:004EDC41 loc_4EDC41:                           ; CODE XREF: ClientSocket1Read+D3j

CODE:004EDC41                   mov     eax, [ebp+RecvText]

CODE:004EDC44                   cmp     byte ptr [eax+esi-1], 0Dh ; 判断当前 pos 是否指向 0dh 内容的字节

CODE:004EDC49                   jz      short loc_4EDC69 ; 指向 0dh 跳转

CODE:004EDC4B                   lea     eax, [ebp+var_18]

CODE:004EDC4E                   mov     edx, [ebp+RecvText]

CODE:004EDC51                   mov     dl, [edx+esi-1]

CODE:004EDC55                   call    unknown_libname_13 ; Borland Visual Component Library & Packages

CODE:004EDC5A                   mov     edx, [ebp+var_18]

CODE:004EDC5D                   lea     eax, [ebp+var_C] ; 接到这个字符串后

CODE:004EDC60                   call    @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)

CODE:004EDC65                   inc     esi             ; 接字符串

CODE:004EDC66                   dec     edi

CODE:004EDC67                   jnz     short loc_4EDC41

CODE:004EDC69

CODE:004EDC69 loc_4EDC69:                           ; CODE XREF: ClientSocket1Read+AAj

CODE:004EDC69                                             ; ClientSocket1Read+B5j

CODE:004EDC69                   lea     edx, [ebp+struseredit2]

CODE:004EDC6C                   mov     eax, [ebx+324h]

CODE:004EDC72                   call    @TControl@GetText$qqrv ; TControl::GetText(void)

CODE:004EDC77                   mov     eax, [ebp+struseredit2] ; 从某控件取字符串(Useredit)

CODE:004EDC7A                   push    eax

CODE:004EDC7B                   lea     edx, [ebp+var_20] ; 此变量还未初始化过

CODE:004EDC7E                   mov     eax, ebx        ; self 指针

CODE:004EDC80                   call    sub_4EB218      ; 功能未知 返回服务器登陆字符串的某一部分吧

CODE:004EDC85                   mov     eax, [ebp+var_20]

CODE:004EDC88                   mov     ecx, [ebp+var_C]

CODE:004EDC8B                   pop     edx

CODE:004EDC8C                   call    sub_4E3C58      ; 外挂验证检测函数

CODE:004EDC91                   mov     eax, [ebp+CustomWinSocket]

CODE:004EDC94                   call    @Scktcomp@TCustomWinSocket@Close$qqrv ; Scktcomp::TCustomWinSocket::Close(void)

CODE:004EDC99

CODE:004EDC99 loc_4EDC99:                           ; CODE XREF: ClientSocket1Read+7Bj

CODE:004EDC99                                             ; ClientSocket1Read+91j

CODE:004EDC99                   xor     eax, eax

CODE:004EDC9B                   pop     edx

CODE:004EDC9C                   pop     ecx

CODE:004EDC9D                   pop     ecx

CODE:004EDC9E                   mov     fs:[eax], edx

CODE:004EDCA1                   push    offset loc_4EDCE0

CODE:004EDCA6

CODE:004EDCA6 loc_4EDCA6:                           ; CODE XREF: ClientSocket1Read+14Aj

CODE:004EDCA6                   lea     eax, [ebp+var_20]

CODE:004EDCA9                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCAE                   lea     eax, [ebp+struseredit2]

CODE:004EDCB1                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCB6                   lea     eax, [ebp+var_18]

CODE:004EDCB9                   mov     edx, 2

CODE:004EDCBE                   call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004EDCC3                   lea     eax, [ebp+strUseredit]

CODE:004EDCC6                   call    @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)

CODE:004EDCCB                   lea     eax, [ebp+var_C]

CODE:004EDCCE                   mov     edx, 2

CODE:004EDCD3                   call    @System@@LStrArrayClr$qqrv ; System::__linkproc__ LStrArrayClr(void)

CODE:004EDCD8                   retn

CODE:004EDCD9 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCD9

CODE:004EDCD9 loc_4EDCD9:                           ; DATA XREF: ClientSocket1Read+1Eo

CODE:004EDCD9                   jmp     @System@@HandleFinally$qqrv ; System::__linkproc__ HandleFinally(void)

CODE:004EDCDE ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCDE                   jmp     short loc_4EDCA6

CODE:004EDCE0 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CODE:004EDCE0

CODE:004EDCE0 loc_4EDCE0:                           ; DATA XREF: ClientSocket1Read+10Do

CODE:004EDCE0                   pop     edi

CODE:004EDCE1                   pop     esi

CODE:004EDCE2                   pop     ebx

CODE:004EDCE3                   mov     esp, ebp

CODE:004EDCE5                   pop     ebp

CODE:004EDCE6                   retn

CODE:004EDCE6 ClientSocket1Read endp

来到验证函数内

CODE:004E3C58 sub_4E3C58      proc near           ; CODE XREF: sub_4E26DC+4Ap

CODE:004E3C58                                             ; ClientSocket1Read+6Ep ...

CODE:004E3C58

CODE:004E3C58 var_18          = dword ptr -18h

CODE:004E3C58 var_14          = dword ptr -14h

CODE:004E3C58 var_10          = dword ptr -10h

CODE:004E3C58 var_C           = dword ptr -0Ch

CODE:004E3C58 var_8           = dword ptr -8

CODE:004E3C58 var_4           = dword ptr -4

CODE:004E3C58

CODE:004E3C58                   push    ebp

CODE:004E3C59                   mov     ebp, esp

CODE:004E3C5B                   add     esp, 0FFFFFFE8h

CODE:004E3C5E                   push    ebx

CODE:004E3C5F                   push    esi

CODE:004E3C60                   push    edi

CODE:004E3C61                   xor     ebx, ebx

CODE:004E3C63                   mov     [ebp+var_14], ebx

CODE:004E3C66                   mov     [ebp+var_18], ebx

CODE:004E3C69                   mov     [ebp+var_10], ebx

CODE:004E3C6C                   mov     [ebp+var_C], ecx

CODE:004E3C6F                   mov     [ebp+var_8], edx

CODE:004E3C72                   mov     [ebp+var_4], eax

CODE:004E3C75                   mov     eax, [ebp+var_4]

CODE:004E3C78                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C7D                   mov     eax, [ebp+var_8]

CODE:004E3C80                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C85                   mov     eax, [ebp+var_C]

CODE:004E3C88                   call    @System@@LStrAddRef$qqrv ; System::__linkproc__ LStrAddRef(void)

CODE:004E3C8D                   xor     eax, eax

CODE:004E3C8F                   push    ebp

CODE:004E3C90                   push    offset sub_4E3D73

CODE:004E3C95                   push    dword ptr fs:[eax]

CODE:004E3C98                   mov     fs:[eax], esp

CODE:004E3C9B                   push    [ebp+var_4]

CODE:004E3C9E                   push    [ebp+var_8]

CODE:004E3CA1                   lea     edx, [ebp+var_18]

CODE:004E3CA4                   mov     eax, ds:off_4F2404
```
