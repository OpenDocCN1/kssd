
```
.text:006F3E09                    mov     edi, 1  

.text:006F3E0E                    shl     edi, cl  

.text:006F3E10                    or      eax, edi  

.text:006F3E12  

.text:006F3E12 loc_6F3E12:                    

.text:006F3E12                    inc     edx  

.text:006F3E13                    cmp     edx, 45        ; 32-45  

.text:006F3E16                    jle     short loc_6F3DF0 ; Cnbragon 说这个是试用版天数限制:)  

.text:006F3E18                    push    0  

.text:006F3E1A                    cdq  

.text:006F3E1B                    push    15180h  

.text:006F3E20                    push    edx           ; 0  

.text:006F3E21                    push    eax  

.text:006F3E22                    call    __allmul  

.text:006F3E27                    mov     esi, [esp+5Ch+var_30_pStr]  

.text:006F3E2B                    add     eax, 301DDF00h  

.text:006F3E30                    adc     edx, 0  

.text:006F3E33                    mov     [ebp+20h], eax  

.text:006F3E36                    mov     [ebp+24h], edx  

.text:006F3E39                    mov     byte ptr [esp+5Ch+arg_0], 1  

.text:006F3E3E                    xor     edi, edi  

.text:006F3E40  

.text:006F3E40 loc_6F3E40:                     

.text:006F3E40                    cmp     esi, edi  

.text:006F3E42                    mov     byte ptr [esp+5Ch+var_4], 6  

.text:006F3E47                    jz      short loc_6F3E52  

.text:006F3E49                    push    esi  

.text:006F3E4A                    call    operator delete\  

.text:006F3E4F                    add     esp, 4  

.text:006F3E52  

.text:006F3E52 loc_6F3E52:                    

.text:006F3E52                    lea     ecx, [esp+5Ch+var_48_md5s]  

.text:006F3E56                    call    ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::~CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(void)  

.text:006F3E5C                    mov     eax, [esp+5Ch+var_1C_pStr]  

.text:006F3E60                    cmp     eax, edi  

.text:006F3E62                    mov     byte ptr [esp+5Ch+var_4], 2  

.text:006F3E67                    jz      short loc_6F3E72  

.text:006F3E69                    push    eax  

.text:006F3E6A                    call    operator delete\  

.text:006F3E6F                    add     esp, 4  

.text:006F3E72  

.text:006F3E72 loc_6F3E72:                    

.text:006F3E72                    lea     ecx, [esp+5Ch+var_4C_r21]  

.text:006F3E76                    call    ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::~CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(void)  

.text:006F3E7C                    lea     ecx, [esp+5Ch+var_3C_r4]  

.text:006F3E80                    call    ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::~CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(void)  

.text:006F3E86  

.text:006F3E86 loc_6F3E86:                   

.text:006F3E86                    lea     ecx, [esp+5Ch+var_44_RegCode]  

.text:006F3E8A                    call    ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::~CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(void)  

.text:006F3E90                    mov     ecx, [esp+50h]  

.text:006F3E94                    mov     al, byte ptr [esp+5Ch+arg_0]  

.text:006F3E98                    pop     edi  

.text:006F3E99                    pop     esi  

.text:006F3E9A                    pop     ebp  

.text:006F3E9B                    pop     ebx  

.text:006F3E9C                    mov     large fs:0, ecx  

.text:006F3EA3                    add     esp, 4Ch  

.text:006F3EA6                    ret     4  

.text:006F3EA9  

.text:006F3EA9 failed_6F3EA9:               

.text:006F3EA9                    call    AfxThrowInvalidArgException(void)  

.text:006F3EAF v_6F38C0        endp  
```

从上面也可以看出，注册码长度必须为 25。

最后解析出来的 60 个位就是 `key` 的内容了。

我逆向了部分及其中的 3 个子程序，如下：

```
char * Table1[] = {"CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8"};  

int Table2[30] = {0   ,    6, 0x0B, 0x0C, 0x11, 0x16, 0x19, 0x1C, 0x1D, 0x1E,  

                  0x1F, 0x26, 0x27, 0x28, 0x29, 0x2F, 0x30, 0x31, 0x32, 0x37,  

                  0x3C, 0x41, 0x44, 0x45, 0x46, 0x47, 0x4D, 0x4E, 0x54, 0x59};  

CString sLic[] = {"Eval",  

                  "Single",  

                  "Multi",  

                  "Academic"};  

struct strarray{  

    int len;  

    unsigned char * sp;  

};  

int funv1(int retValue[],CString RegCode)  

{  

    CString      r21,r4,md5s;  

    char         c1;  

    int          len,i,j,k,l,n,m;  

    int          i6,i11;  

    unsigned int x;  

    RegCode.TrimLeft();  

    RegCode.TrimRight();  

    RegCode.Remove('-');  

    RegCode.Remove(' ');  

    RegCode.MakeUpper();  

    if(RegCode.GetLength() != 25)return 0;  

    r4 = RegCode.Left(4);  

    r21 = RegCode.Right(21);  

    c1 = r21[5];  

    i6 = -1;  

    for(i = 0; i < 32; i ++)  

    {  

        if(c1 == Table1[i])  

        {  

            i6 = i;  

            break;  

        }  

    }  

    r21.Delete(5,1);  

    c1 = r21[10];  

    i11 = -1;  

    for(i = 0; i < 32; i ++)  

    {  

        if(c1 == Table1[i])  

        {  

            i11 = i;  

            break;  

        }  

    }  

    r21.Delete(10,1);  

    strarray sa1;  

    sa1.len = 0;  

    sa1.sp = NULL;  

    fun1(r21,&sa1);    

    i11 += 32;  

    if(i11 == -1)i11 = sa1.len - 1;  

    i = (i11 + 1) - i6 % (i11 + 1);  

    fun2(&sa1,i,0,i11);  

    r21 = fun3(&sa1);  

    c1 = r21[10];  

    i11 = -1;  

    for(i = 0; i < 32; i ++)  

    {  

        if(c1 == Table1[i])  

        {  

            i11 = i;  

            break;  

        }  

    }  

    r21.Delete(10,1);  

    fun1(r21,&sa1);  

    fun2(&sa1,i11,0,-1);  

    r21 = fun3(&sa1);  

    md5s = md5(r21);  

    md5s = md5s.Left(4);  

    for(i = 0; i < 4; i ++)  

    {  

        if(md5s[i] == '0')  

            md5s.SetAt(i,'R');  

        esle if(md5s[i] == '1')  

            md5s.SetAt(i,'M');  

    }  

    md5s.MakeUpper();  

    if(md5s.Compare(r4) != 0)return 0;  

    strarray sa2;  

    sa2.len = 0;  

    sa2.sp = NULL;  

    sa2.sp = new unsigned char[60];  

    sa2.len = 60;  

    memset(sa2.sp,0,60);  

    m = 0;  

    l = 0;  

    if(r21.GetLength() > 0)  

    {  

        for(j = 0; j < r21.GetLength(); j ++)  

        {  

            i6 = -1;  

            c1 = r21[j];  

            for(i = 0; i < 32; i ++)  

            {  

                if(c1 == Table1[i])  

                {  

                    i6 = i;  

                    break;  

                }  

            }  

            k = 0;  

            n = 0;            

            for(k = 0; k < 5; k ++)  

            {  

                for(n = 0; n < 30; n ++)  

                    if((k+l) == Table2[n])  

                        goto next1;  

                sa2.sp[m] = (((1 << k) & i6) != 0) ? 1 : 0;  

                m ++;  

next1:            

            }  

            l += 5;     

        }  

    }  

    x = 0;  

    for(i = 21; i <= 26; i ++)  //取 key 的 21 到 26 位，这里必须为 0  

    {  

        if(sa2.sp[i] != 0)  

        {  

            x |= (1 << (i - 21));  

        }  

    }  

    retValue[0] = x;  

    if(x != 0)return 0;  

    x = 0;  

    for(i = 0; i <= 4; i ++)  

    {  

        if(sa2.sp[i] != 0)  

        {  

            x |= (1 << i);  

        }  

    }  

    retValue[7] = x;  

    x = 0;  

    for(i = 5; i <= 8; i ++)  

    {  

        if(sa2.sp[i] != 0)  

        {  

            x |= (1 <<(i-5));  

        }  

    }  

    retValue[2] = x;  

    x = 0;  

    for(i = 9; i <= 12; i ++)  

    {  

        if(sa2.sp[i] != 0)  

        {  

            x |= (1 <<(i-9));  

        }  

    }  

    retValue[3] = x;         

    x = 0;
```
