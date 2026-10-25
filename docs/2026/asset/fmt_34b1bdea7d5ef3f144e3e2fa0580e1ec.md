
我逆向了部分及其中的3个子程序，如下：

```c
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
    for(i = 21; i <= 26; i ++)  //取key的21到26位，这里必须为0
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
    for(i = 13; i <= 16; i ++)
    {
        if(sa2.sp[i] != 0)
        {
            x |= (1 <<(i-13));
        }
    }
    retValue[4] = x;
    x = 0;
    for(i = 17; i <= 20; i ++)
    {
        if(sa2.sp[i] != 0)
        {
            x |= (1 <<(i-17));
        }
    }
    if((x < 0) || (x >= 4)) x = 0;
    retValue[5] = x;
    retValue[6] = sLic[x];
    x = 0;
    for(i = 27; i <= 31; i ++)
    {
        if(sa2.sp[i] != 0)
        {
            x |= (1 <<(i-27));
        }
    }
    retValue[1] = x;
    x = 0;
    for(i = 32; i <= 45; i ++)
    {
        if(sa2.sp[i] != 0)
        {
            x |= (1 <<(i-32));
        }
    }
    __int64 y = Int32x32To64(x,0x15180);
    y += 0x301DDF00;
    *(__int64*)(&retvalue[8]) = y;
    if(sa2.sp != NULL)delete [] sa2.sp;
    sa2.sp = NULL;
    sa2.len = 0;
    if(sa1.sp != NULL)delete [] sa1.sp;
    sa1.sp = NULL;
    sa1.len = 0;
}
```

下面是其中3个子程序的逆向：

```
=======================================================================================

.text:006F37C0 fun1_6F37C0     proc near
.text:006F37C0

.text:006F37C0 var_10          = dword ptr -10h
.text:006F37C0 var_8           = dword ptr -8
.text:006F37C0 var_4           = dword ptr -4
.text:006F37C0 arg_0           = dword ptr  4
.text:006F37C0 arg_4           = dword ptr  8
.text:006F37C0

.text:006F37C0                 push    0FFFFFFFFh
.text:006F37C2                 push    offset loc_7F7069
.text:006F37C7                 mov     eax, large fs:0
.text:006F37CD                 push    eax
.text:006F37CE                 mov     large fs:0, esp
.text:006F37D5                 push    ecx
.text:006F37D6                 mov     eax, [esp+10h+arg_0] ; 注册码
.text:006F37DA                 push    ebx
.text:006F37DB                 push    ebp
.text:006F37DC                 push    esi
.text:006F37DD                 push    edi
.text:006F37DE                 push    eax
.text:006F37DF                 lea     ecx, [esp+24h+var_10] ; 保存到这里
.text:006F37E3                 call    ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(char const *)
.text:006F37E9                 mov     esi, [esp+20h+arg_4] ; 动态数组
.text:006F37ED                 mov     eax, [esi+4]    ; 取数组长度
.text:006F37F0                 xor     ebp, ebp
.text:006F37F2                 cmp     eax, ebp
.text:006F37F4                 mov     [esp+20h+var_4], ebp
.text:006F37F8                 jz      short loc_6F3806
.text:006F37FA                 push    eax
.text:006F37FB                 call    operator delete[](void *) ; 数组已经分配则先释放
.text:006F3800                 add     esp, 4
.text:006F3803                 mov     [esi+4], ebp
.text:006F3806

.text:006F3806 loc_6F3806:
.text:006F3806                 push    0FFFFFFFFh
.text:006F3808                 lea     ecx, [esp+24h+var_10] ; 注册码
.text:006F380C                 mov     [esi+0Ch], ebp
.text:006F380F                 mov     [esi+8], ebp
.text:006F3812                 call    ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:006F3818                 lea     ecx, [eax+eax*4] ; 长度乘以5
.text:006F381B                 push    ecx
.text:006F381C                 mov     ecx, esi
.text:006F381E                 call    new_6F3410      ; 分配空间
.text:006F3823                 lea     ecx, [esp+20h+var_10] ; 注册码
.text:006F3827                 call    ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:006F382D                 test    eax, eax        ; 取注册码长度
.text:006F382F                 jle     short loc_6F389C
.text:006F3831                 mov     [esp+20h+arg_4], ebp ; 计数器清零,这里使用的一个参数空间作为变量
.text:006F3835

.text:006F3835 loc_6F3835:
.text:006F3835                 push    ebp             ; 取一个字符
.text:006F3836                 lea     ecx, [esp+24h+var_10]
.text:006F383A                 call    ds:ATL::CSimpleStringT<char,1>::operator[](int)
.text:006F3840                 or      edi, 0FFFFFFFFh
.text:006F3843                 xor     ecx, ecx
.text:006F3845

.text:006F3845 loc_6F3845:
.text:006F3845                 cmp     al, byte ptr ds:Table1_865058[ecx]
.text:006F384B                 jz      short loc_6F3855
.text:006F384D                 inc     ecx
```
