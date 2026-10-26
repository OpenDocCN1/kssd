
```
.rdata:0057AE94                 dd 43F7A4h

.rdata:0057AE98                 dd 19h

.rdata:0057AE9C                 dd 0

.rdata:0057AEA0                 dd 0

.rdata:0057AEA4                 dd 0

.rdata:0057AEA8                 dd 4

.rdata:0057AEAC                 dd 43F7ADh

.rdata:0057AEB0                 dd 111h

.rdata:0057AEB4                 dd 0

.rdata:0057AEB8                 dd 623h

.rdata:0057AEBC                 dd 623h

.rdata:0057AEC0                 dd 0Ch

.rdata:0057AEC4                 dd 43F867h

.rdata:0057AEC8                 dd 0

.rdata:0057AECC                 dd 0

.rdata:0057AED0                 dd 0

.rdata:0057AED4                 dd 0

.rdata:0057AED8                 dd 0

.rdata:0057AEDC                 dd 0

;=================================================================================
```

好了，现在找到了注册对话框中的ok按钮的处理函数，看一下这个函数：

```
.text:0043F6B7 sub_43F6B7      proc near

.text:0043F6B7                 mov     eax, offset unknown_libname_1327

.text:0043F6BC                 call    __EH_prolog

.text:0043F6C1                 push    ecx

.text:0043F6C2                 push    esi

.text:0043F6C3                 mov     esi, ecx

.text:0043F6C5                 push    1

.text:0043F6C7                 call    CWnd::UpdateData(int)

.text:0043F6CC                 mov     ecx, [esi+1A0h]

.text:0043F6D2                 lea     eax, [esi+188h]

.text:0043F6D8                 push    eax

.text:0043F6D9                 call    CString::operator=(CString const &)

.text:0043F6DE                 mov     ecx, [esi+1A0h]

.text:0043F6E4                 lea     eax, [esi+18Ch]

.text:0043F6EA                 push    eax

.text:0043F6EB                 add     ecx, 4

.text:0043F6EE                 call    CString::operator=(CString const &)

.text:0043F6F3                 mov     ecx, [esi+1A0h]

.text:0043F6F9                 lea     eax, [esi+190h]

.text:0043F6FF                 push    eax

.text:0043F700                 add     ecx, 8

.text:0043F703                 call    CString::operator=(CString const &)

.text:0043F708                 mov     ecx, [esi+1A0h]

.text:0043F70E                 lea     eax, [esi+194h]

.text:0043F714                 push    eax

.text:0043F715                 add     ecx, 0Ch

.text:0043F718                 call    CString::operator=(CString const &)

.text:0043F71D                 mov     ecx, [esi+1A0h]

.text:0043F723                 lea     eax, [esi+198h]

.text:0043F729                 push    eax

.text:0043F72A                 add     ecx, 10h

.text:0043F72D                 call    CString::operator=(CString const &)

.text:0043F732                 lea     eax, [esi+19Ch]

.text:0043F738                 push    0

.text:0043F73A                 push    eax

.text:0043F73B                 mov     ecx, offset unk_5ECE48

.text:0043F740                 call    sub_46CB82             //这里是关键函数，下面着重分析

.text:0043F745                 test    eax, eax

.text:0043F747                 jz      short loc_43F750       //返回0就失败了

.text:0043F749                 mov     ecx, esi

.text:0043F74B                 call    CDialog::OnOK(void)

.text:0043F750

.text:0043F750 loc_43F750:

.text:0043F750                 mov     eax, dword_5D537C      //到这里就是注册失败提示了

.text:0043F755                 mov     [ebp-10h], eax

.text:0043F758                 and     dword ptr [ebp-4], 0

.text:0043F75C                 push    0Dh             ; Invalid License Key. Please try again or contact Wisdom Software

.text:0043F75E                 lea     ecx, [ebp-10h]

.text:0043F761                 call    CString::LoadStringA(uint)

.text:0043F766                 push    1037h

.text:0043F76B                 mov     ecx, esi

.text:0043F76D                 call    CWnd::GetDlgItem(int)

.text:0043F772                 test    eax, eax

.text:0043F774                 jz      short loc_43F780

.text:0043F776                 push    dword ptr [ebp-10h]

.text:0043F779                 mov     ecx, eax

.text:0043F77B                 call    CWnd::SetWindowTextA(char const *)

.text:0043F780

.text:0043F780 loc_43F780:

.text:0043F780                 lea     ecx, [esi+14Ch]

.text:0043F786                 call    CWnd::SetFocus(void)

.text:0043F78B                 or      dword ptr [ebp-4], 0FFFFFFFFh

.text:0043F78F                 lea     ecx, [ebp-10h]

.text:0043F792                 call    CString::~CString(void)

.text:0043F797                 mov     ecx, [ebp-0Ch]

.text:0043F79A                 pop     esi

.text:0043F79B                 mov     large fs:0, ecx

.text:0043F7A2                 leave

.text:0043F7A3                 retn

.text:0043F7A3 sub_43F6B7      endp ; sp =  4

;=================================================================================
```

跟进关键函数46CB82:

```
.text:0046CB82 sub_46CB82      proc near

.text:0046CB82                 mov     eax, offset unknown_libname_1421

.text:0046CB87                 call    __EH_prolog

.text:0046CB8C                 push    ecx

.text:0046CB8D                 push    ebx

.text:0046CB8E                 xor     ebx, ebx

.text:0046CB90                 push    esi

.text:0046CB91                 push    edi

.text:0046CB92                 mov     esi, ecx

.text:0046CB94                 mov     [ebp-10h], ebx

.text:0046CB97                 push    ebx

.text:0046CB98                 push    ebx

.text:0046CB99                 push    0F003Fh

.text:0046CB9E                 push    ebx

.text:0046CB9F                 push    ebx

.text:0046CBA0                 lea     ecx, [ebp-10h]

.text:0046CBA3                 push    dword_5F0CC8

.text:0046CBA9                 mov     [ebp-4], ebx

.text:0046CBAC                 push    80000001h

.text:0046CBB1                 call    sub_466121

.text:0046CBB6                 cmp     eax, ebx

.text:0046CBB8                 jl      loc_46CCFE

.text:0046CBBE                 push    dword ptr [ebp+0Ch]

.text:0046CBC1                 mov     edi, [ebp+8]

.text:0046CBC4                 push    edi

.text:0046CBC5                 call    fun1_422C8E       //检验函数1 *****

.text:0046CBCA                 pop     ecx

.text:0046CBCB                 test    eax, eax

.text:0046CBCD                 pop     ecx

.text:0046CBCE                 jz      loc_46CCFC

.text:0046CBD4

.text:0046CBD4 loc_46CBD4:

.text:0046CBD4                 mov     eax, ebx

.text:0046CBD6                 mov     ecx, ebx

.text:0046CBD8                 add     eax, [edi+4]

.text:0046CBDB                 add     ecx, [esi+120h]

.text:0046CBE1                 push    eax

.text:0046CBE2                 call    CString::operator=(CString const &)

.text:0046CBE7                 add     ebx, 4

.text:0046CBEA                 cmp     ebx, 20            //把5组注册码拷贝到[esi+120h]处

.text:0046CBED                 jl      short loc_46CBD4

.text:0046CBEF                 push    1

.text:0046CBF1                 lea     eax, [esi+11Ch]

.text:0046CBF7                 pop     ebx

.text:0046CBF8                 push    eax

.text:0046CBF9                 mov     [esi+130h], ebx

.text:0046CBFF                 call    fun2_422EF1        //检验函数2 *****

.text:0046CC04                 test    eax, eax

.text:0046CC06                 pop     ecx

.text:0046CC07                 jz      short loc_46CC0F

.text:0046CC09                 mov     [esi+134h], ebx

.text:0046CC0F

.text:0046CC0F loc_46CC0F:

.text:0046CC0F                 mov     ecx, [esi+120h]

.text:0046CC15                 lea     eax, [ebp+0Ch]

.text:0046CC18                 push    ebx             ; 1

.text:0046CC19                 push    eax

.text:0046CC1A                 add     ecx, 8

.text:0046CC1D                 call    CString::Left(int) //取第三组注册码左边一个字符

.text:0046CC22                 push    eax

.text:0046CC23                 lea     ecx, [esi+4C4h]    //保存到这里

.text:0046CC29                 mov     [ebp-4], bl
```
