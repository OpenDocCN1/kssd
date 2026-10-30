
# [看雪读书月]看雪论坛读书月第一题答案

分析报告、源码、注册机都已经上传

## 一、通过关键字“非法用户”找到这里：

这段代码主要检查用户名是否合法，用户名合法条件为：

1. 用户名由字母`b`～`y`组成；
2. 用户名为6个字符；
3. 由于判断不严谨，可以前6个字符满足要求1，第七个字符不满足要求1，就可以跳过这里的“非法用户”判断。

```
.text:00401D82                 mov     [ebp+i], 0        初始化变量，用于记录用户名长度
.text:00401D89

.text:00401D89 loc_401D89:
.text:00401D89                 mov     ecx, [ebp+UserName]
.text:00401D8C                 add     ecx, [ebp+i]
.text:00401D8F                 movsx   edx, byte ptr [ecx]
.text:00401D92                 test    edx, edx
.text:00401D94                 jz      short loc_401DBD                检查用户名是否结束
.text:00401D96                 mov     eax, [ebp+UserName]
.text:00401D99                 add     eax, [ebp+i]
.text:00401D9C                 movsx   ecx, byte ptr [eax]
.text:00401D9F                 cmp     ecx, 61h
.text:00401DA2                 jle     short loc_401DBD           检查用户名是否小于等于a
.text:00401DA4                 mov     edx, [ebp+UserName]
.text:00401DA7                 add     edx, [ebp+i]
.text:00401DAA                 movsx   eax, byte ptr [edx]
.text:00401DAD                 cmp     eax, 7Ah
.text:00401DB0                 jge     short loc_401DBD            检查用户名是否大于等于z
.text:00401DB2                 mov     ecx, [ebp+i]
.text:00401DB5                 add     ecx, 1
.text:00401DB8                 mov     [ebp+i], ecx
.text:00401DBB                 jmp     short loc_401D89        循环执行
.text:00401DBD ; ---------------------------------------------------------------------------

.text:00401DBD
.text:00401DBD loc_401DBD:
.text:00401DBD                 cmp     [ebp+i], 6                检查用户名长度是否为6
.text:00401DC1                 jz      short loc_401DD6        若等于6，跳过“非法用户”
.text:00401DC3                 push    0
.text:00401DC5                 push    0
.text:00401DC7                 push    offset aIZ      ; "非法用户!"
.text:00401DCC                 mov     ecx, [ebp+var_4]
.text:00401DCF                 call    MessageBoxA
.text:00401DD6 loc_401DD6:
.text:00401DD6                 mov     edx, [ebp+UserName]
.text:00401D99                 push    edx             ; Source
.text:00401DDA                 push    offset UserName ; Dest
.text:00401DDF                 call    strcpy                ;复制用户名
.text:00401DE4                 add     esp, 8
.text:00401DE7                 mov     eax, [ebp+Password]
.text:00401DEA                 push    eax             ; Source
.text:00401DEB                 push    offset Password ; Dest
.text:00401DF0                 call    strcpy                ;复制序列号
.text:00401DF5                 add     esp, 8
.text:00401DF8                 mov     [ebp+_ESP], esp
.text:00401DFB                 mov     ecx, [ebp+Password]
.text:00401DFE                 push    ecx
.text:00401DFF                 mov     edx, [ebp+UserName]
.text:00401E02                 push    edx
.text:00401E03                 mov     eax, [ebp+_ESP]
.text:00401E06                 push    eax
.text:00401E07                 call    sub_401BB0        ;将用户名、序列号、ESP压栈用于验证帐号及序列号
.text:00401E0C                 add     esp, 0Ch
```

## 二、通过压入的`ESP`找到父函数的返回地址，并将其覆盖为第三次验证序列号的函数地址：

```
.text:00401BD2                 mov     eax, [ebp+FATHER_ESP]
.text:00401BD5                 sub     eax, 10h
.text:00401BD8                 mov     [ebp+pRetAddr], eax
.text:00401BDB                 mov     ecx, [ebp+pRetAddr]
.text:00401BDE                 mov     dword ptr [ecx], offset Msg_Info
```

下面开始为一个循环体用于第二次验证用户名以及序列号，若验证失败，则将返回地址改成弹出错误窗口的函数地址：

转换为`C`语言函数如下：

```
        for (int i = 0; i < 6; i++)
        {
                //当该条件不成立时，直接弹出错误并退出
                if (UserName[i] != Password[2*i] + 0x1B)
                {
                        call Msg_Error;
                }
                //当该条件不成立时，将返回地址覆盖为报错函数，结果一样
                if (UserName[i] <= Password[2*i + 1] + 0x20)
                {
                        pRetAddr = Msg_Error;
                }
        }
```

```
.text:00401BE4                 mov     [ebp+i_UserName], 0
.text:00401BEB                 mov     [ebp+i_Password], 0
.text:00401BF2 ; hua_code
.text:00401BFE

.text:00401BFE loc_401BFE:
.text:00401BFE                 cmp     [ebp+i_UserName], 6                检查变量
.text:00401C02                 jge     short loc_401C71                大于等于6时跳出循环
.text:00401C04 ; hua_code
.text:00401C10                 mov     edx, [ebp+UserName]
.text:00401C13                 add     edx, [ebp+i_UserName]
.text:00401C16                 movsx   eax, byte ptr [edx]
.text:00401C19                 mov     ecx, [ebp+Password]
.text:00401C1C                 add     ecx, [ebp+i_Password]
.text:00401C1F                 movsx   edx, byte ptr [ecx]
.text:00401C22                 add     edx, 1Bh
.text:00401C25             cmp     eax, edx        ; UserName[i] == Password[2*i] + 0x1B ?
.text:00401C27                 jz      short loc_401C2E  ; 相同时跳过报错
.text:00401C29                 call    Msg_Error
.text:00401C2E ; hua_code
.text:00401C3A                 mov     eax, [ebp+UserName]
.text:00401C3D                 add     eax, [ebp+i_UserName]
.text:00401C40                 movsx   ecx, byte ptr [eax]
.text:00401C43                 mov     edx, [ebp+Password]
.text:00401C46                 add     edx, [ebp+i_Password]
.text:00401C49                 movsx   eax, byte ptr [edx+1]
.text:00401C4D                 add     eax, 20h
.text:00401C50            cmp     ecx, eax           ; UserName[i] <= Password[2*i + 1] + 0x20 ?
.text:00401C52                 jle     short loc_401C5D        ; 小于等于时跳过报错
.text:00401C54                 mov     ecx, [ebp+pRetAddr]
.text:00401C57                 mov     dword ptr [ecx], offset Msg_Error
.text:00401C5D

.text:00401C5D loc_401C5D:
.text:00401C5D                 mov     edx, [ebp+i_Password]        ; 密码左移2位
.text:00401C60                 add     edx, 2
.text:00401C63                 mov     [ebp+i_Password], edx
.text:00401C66                 mov     eax, [ebp+i_UserName]        ; 用户名左移1位
.text:00401C69                 add     eax, 1
.text:00401C6C                 mov     [ebp+i_UserName], eax
.text:00401C6F                 jmp     short loc_401BFE
.text:00401C71 loc_401C71:
.text:00401C71                 mov     [ebp+var_4], 0
.text:00401C78 ; hua_code
```

## 三、这里人为造成一个异常，跳到异常处理函数中，其`eax`是指向`_S_FUNCINFO`结构体的指针，该结构体如下：

```
_S_FUNCINFO STRUCT
MagicNumber        DWORD                ?                magic number
MaxState                DWORD                ?                max state in the function
PUnwindMap        DWORD                ?                unwind function list
NTryBlocks                DWORD                ?                how many try blocks
PTryBlockMap        DWORD                ?                try block list
_S_FUNCINFO ENDS
```

而我们在`0x004036F8`中看到的数据如下：

```
004036F8  19930520
004036FC  00000002
00403700  00403718  cm.00403718
00403704  00000001
00403708  00403728  cm.00403728
```

关键在于`PTryBlockMap`这个变量，即异常处理数组指针，也是一个结构体，其结构如下：

```
S_TRYBLOCKMAPENTRY STRUCT
TryLow                 DWORD                ?                begin state
TryHi                        DWORD                ?                end state
CatchHi                DWORD                ?                catch end state
NCatches                DWORD                ?                how many catches
PHandlerArray        DWORD                ?                cathe block entry list
S_TRYBLOCKMAPENTRY ENDS
```

而我们在`0x403728`地址中看到的该结构体的数据为：

```
00403728  00000000
0040372C  00000000
00403730  00000001
00403734  00000001
00403738  00403740  cm.00403740
```

关键在于`PHandlerArray`这个变量，这个指针指向的结构体中即是我们所要的信息，结构体如下：

```
_S_HANDLERTYPE STRUCT
Adjectives                        DWORD        ?        properites
PType                                DWORD        ?        type_info pointer
DispCatchObj                DWORD        ?        offset from ebp
```
