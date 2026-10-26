

```asm
CODE:0054C880 __account_key_2 proc near               ; CODE XREF: sub_54DE3C+44p
CODE:0054C880                                         ; _TFrmRegPass_Button1Click+3Ap
CODE:0054C880

CODE:0054C880 var_18          = dword ptr -18h
CODE:0054C880 var_14          = dword ptr -14h
CODE:0054C880 pnAry           = byte ptr -10h

CODE:0054C880
CODE:0054C880                 push    ebp
CODE:0054C881                 mov     ebp, esp
CODE:0054C883                 add     esp, 0FFFFFFE8h
CODE:0054C886                 push    ebx
CODE:0054C887                 push    esi
CODE:0054C888                 push    edi
CODE:0054C889                 xor     ecx, ecx
CODE:0054C88B                 mov     [ebp+var_14], ecx
CODE:0054C88E                 mov     [ebp+var_18], ecx
CODE:0054C891                 mov     esi, eax
CODE:0054C893                 lea     edi, [ebp+pnAry]
CODE:0054C896                 movsd                   ; 数组拷贝
CODE:0054C897                 movsd                   ; dword ptr:[esi] -> dword ptr:[edi]
CODE:0054C898                 movsd
CODE:0054C899                 movsd
CODE:0054C89A                 mov     edi, edx
CODE:0054C89C                 xor     eax, eax
CODE:0054C89E                 push    ebp
CODE:0054C89F                 push    offset loc_54C91B ; int
CODE:0054C8A4                 push    dword ptr fs:[eax] ; int
CODE:0054C8A7                 mov     fs:[eax], esp
CODE:0054C8AA                 mov     eax, edi
CODE:0054C8AC                 call    __LStrClr
CODE:0054C8B1                 mov     bl, 10h
CODE:0054C8B3                 lea     esi, [ebp+pnAry] ;

CODE:0054C8B3                                         ; do
CODE:0054C8B3                                         ; {
CODE:0054C8B3                                         ; }while(--16)

CODE:0054C8B6

CODE:0054C8B6 loc_54C8B6:                             ; CODE XREF: __account_key_2+7Ej
CODE:0054C8B6                 push    dword ptr [edi] ; int to string; int数组转成字符串
CODE:0054C8B8                 lea     eax, [ebp+var_14]
CODE:0054C8BB                 xor     edx, edx
CODE:0054C8BD                 mov     dl, [esi]
CODE:0054C8BF                 shr     edx, 4
CODE:0054C8C2                 and     edx, 0Fh
CODE:0054C8C5                 mov     dl, ds:byte_557F54[edx]
CODE:0054C8CB                 call    __itoa          ; BDS 2005-2006 and Delphi6-7 Visual Component Library
CODE:0054C8D0                 push    [ebp+var_14]    ; int
CODE:0054C8D3                 lea     eax, [ebp+var_18]
CODE:0054C8D6                 mov     dl, [esi]
CODE:0054C8D8                 and     dl, 0Fh
CODE:0054C8DB                 and     edx, 0FFh
CODE:0054C8E1                 mov     dl, ds:byte_557F54[edx]
CODE:0054C8E7                 call    __itoa          ; BDS 2005-2006 and Delphi6-7 Visual Component Library
CODE:0054C8EC                 push    [ebp+var_18]
CODE:0054C8EF                 mov     eax, edi
CODE:0054C8F1                 mov     edx, 3
CODE:0054C8F6                 call    __strcat     ; 字符串连接
CODE:0054C8FB                 inc     esi
CODE:0054C8FC                 dec     bl
CODE:0054C8FE                 jnz     short loc_54C8B6
CODE:0054C900                 xor     eax, eax
CODE:0054C902                 pop     edx
CODE:0054C903                 pop     ecx
CODE:0054C904                 pop     ecx
CODE:0054C905                 mov     fs:[eax], edx
CODE:0054C908                 push    offset loc_54C922
CODE:0054C90D

CODE:0054C90D loc_54C90D:                             ; CODE XREF: __account_key_2+A0j
CODE:0054C90D                 lea     eax, [ebp+var_18]
CODE:0054C910                 mov     edx, 2
CODE:0054C915                 call    __LStrArrayClr
CODE:0054C91A                 retn

CODE:0054C91B ; ---------------------------------------------------------------------------
CODE:0054C91B

CODE:0054C91B loc_54C91B:                             ; DATA XREF: __account_key_2+1Fo
CODE:0054C91B                 jmp     unknown_libname_75 ; BDS 2005-2006 and Delphi6-7 Visual Component Library
CODE:0054C920 ; ---------------------------------------------------------------------------
CODE:0054C920                 jmp     short loc_54C90D
CODE:0054C922 ; ---------------------------------------------------------------------------
CODE:0054C922

CODE:0054C922 loc_54C922:                             ; CODE XREF: __account_key_2+9Aj
CODE:0054C922                                         ; DATA XREF: __account_key_2+88o
CODE:0054C922                 pop     edi
CODE:0054C923                 pop     esi
CODE:0054C924                 pop     ebx
CODE:0054C925                 mov     esp, ebp
CODE:0054C927                 pop     ebp
CODE:0054C928                 retn
CODE:0054C928 __account_key_2 endp
```

第二次只是把第一次算 key 得到的一个 4 个 `int` 的数组转成字符串，所以我自己写了个功能一样的逆向代码：

```cpp
void Account_Key_2(string& sKey, char* pAry)
{
    for (int n = 0; n < 16; n++)
    {
        char buffer[5] = {0};
        // itoa(unsigned char(pAry[n]), buffer, 16);
        sprintf_s(buffer, "%02x", unsigned char(pAry[n]));
        sKey += buffer;
    }
}
```

第三次传入第二次返回的字符串，传出的是最后的 key。

