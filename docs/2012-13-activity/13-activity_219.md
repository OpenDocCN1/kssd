
# 看雪论坛读书月第一题答案

> 原文：[`www.pediy.com/kssd/pediy10/68352.html`](https://www.pediy.com/kssd/pediy10/68352.html)

分析报告、源码、注册机都已经上传

## 一、通过关键字“非法用户”找到这里

这段代码主要检查用户名是否合法，用户名合法条件为：

1. 用户名由字母 b～y 组成；
2. 用户名为 6 个字符；
3. 由于判断不严谨，可以前 6 个字符满足要求 1，第七个字符不满足要求 1，就可以跳过这里的“非法用户”判断。

```asm
.text:00401D82                  mov     [ebp+i], 0  初始化变量，用于记录用户名长度
.text:00401D89

.text:00401D89 loc_401D89:

.text:00401D89                  mov     ecx, [ebp+UserName]
.text:00401D8C                  add     ecx, [ebp+i]
.text:00401D8F                  movsx   edx, byte ptr [ecx]
.text:00401D92                  test    edx, edx
.text:00401D94                  jz      short loc_401DBD    检查用户名是否结束
.text:00401D96                  mov     eax, [ebp+UserName]
.text:00401D99                  add     eax, [ebp+i]
.text:00401D9C                  movsx   ecx, byte ptr [eax]
.text:00401D9F                  cmp     ecx, 61h
.text:00401DA2                  jle     short loc_401DBD     检查用户名是否小于等于 a
.text:00401DA4                  mov     edx, [ebp+UserName]
.text:00401DA7                  add     edx, [ebp+i]
.text:00401DAA                  movsx   eax, byte ptr [edx]
.text:00401DAD                  cmp     eax, 7Ah
.text:00401DB0                  jge     short loc_401DBD      检查用户名是否大于等于 z
.text:00401DB2                  mov     ecx, [ebp+i]
.text:00401DB5                  add     ecx, 1
.text:00401DB8                  mov     [ebp+i], ecx
.text:00401DBB                  jmp     short loc_401D89  循环执行
.text:00401DBD ; ---------------------------------------------------------------------------

.text:00401DBD

.text:00401DBD loc_401DBD:

.text:00401DBD                  cmp     [ebp+i], 6    检查用户名长度是否为 6
.text:00401DC1                  jz      short loc_401DD6  若等于 6，跳过“非法用户”
.text:00401DC3                  push    0
.text:00401DC5                  push    0
.text:00401DC7                  push    offset aIZ      ; "非法用户!"
.text:00401DCC                  mov     ecx, [ebp+var_4]
.text:00401DCF                  call    MessageBoxA
.text:00401DD6 loc_401DD6:

.text:00401DD6                  mov     edx, [ebp+UserName]
.text:00401DD9                  push    edx             ; Source
.text:00401DDA                  push    offset UserName ; Dest
.text:00401DDF                  call    strcpy    ;复制用户名
.text:00401DE4                  add     esp, 8
.text:00401DE7                  mov     eax, [ebp+Password]
.text:00401DEA                  push    eax             ; Source
.text:00401DEB                  push    offset Password ; Dest
.text:00401DF0                  call    strcpy    ;复制序列号
.text:00401DF5                  add     esp, 8
.text:00401DF8                  mov     [ebp+_ESP], esp
.text:00401DFB                  mov     ecx, [ebp+Password]
.text:00401DFE                  push    ecx
.text:00401DFF                  mov     edx, [ebp+UserName]
.text:00401E02                  push    edx
.text:00401E03                  mov     eax, [ebp+_ESP]
.text:00401E06                  push    eax
.text:00401E07                  call    sub_401BB0  ;将用户名、序列号、ESP 压栈用于验证帐号及序列号
.text:00401E0C                  add     esp, 0Ch
```

## 二、通过压入的 ESP 找到父函数的返回地址，并将其覆盖为第三次验证序列号的函数地址

```asm
.text:00401BD2                  mov     eax, [ebp+FATHER_ESP]
.text:00401BD5                  sub     eax, 10h
.text:00401BD8                  mov     [ebp+pRetAddr], eax
.text:00401BDB                  mov     ecx, [ebp+pRetAddr]
.text:00401BDE                  mov     dword ptr [ecx], offset Msg_Info
```

下面开始为一个循环体用于第二次验证用户名以及序列号，若验证失败，则将返回地址改成弹出错误窗口的函数地址：

转换为 C 语言函数如下：

```c
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

```asm
.text:00401BE4                  mov     [ebp+i_UserName], 0
.text:00401BEB                  mov     [ebp+i_Password], 0
.text:00401BF2 ; hua_code
.text:00401BFE

.text:00401BFE loc_401BFE:

.text:00401BFE                  cmp     [ebp+i_UserName], 6    检查变量
.text:00401C02                  jge     short loc_401C71    大于等于 6 时跳出循环
.text:00401C04 ; hua_code
.text:00401C10                  mov     edx, [ebp+UserName]
.text:00401C13                  add     edx, [ebp+i_UserName]
.text:00401C16                  movsx   eax, byte ptr [edx]
.text:00401C19                  mov     ecx, [ebp+Password]
.text:00401C1C                  add     ecx, [ebp+i_Password]
.text:00401C1F                  movsx   edx, byte ptr [ecx]
.text:00401C22                  add     edx, 1Bh
.text:00401C25             cmp     eax, edx  ; UserName[i] == Password[2*i] + 0x1B ?
.text:00401C27                  jz      short loc_401C2E  ; 相同时跳过报错
.text:00401C29                  call    Msg_Error
.text:00401C2E ; hua_code
.text:00401C3A                  mov     eax, [ebp+UserName]
.text:00401C3D                  add     eax, [ebp+i_UserName]
.text:00401C40                  movsx   ecx, byte ptr [eax]
.text:00401C43                  mov     edx, [ebp+Password]
.text:00401C46                  add     edx, [ebp+i_Password]
.text:00401C49                  movsx   eax, byte ptr [edx+1]
.text:00401C4D                  add     eax, 20h
.text:00401C50            cmp     ecx, eax     ; UserName[i] <= Password[2*i + 1] + 0x20 ?
.text:00401C52                  jle     short loc_401C5D  ; 小于等于时跳过报错
.text:00401C54                  mov     ecx, [ebp+pRetAddr]
.text:00401C57                  mov     dword ptr [ecx], offset Msg_Error
.text:00401C5D

.text:00401C5D loc_401C5D:

.text:00401C5D                  mov     edx, [ebp+i_Password]  ; 密码左移 2 位
.text:00401C60                  add     edx, 2
.text:00401C63                  mov     [ebp+i_Password], edx
.text:00401C66                  mov     eax, [ebp+i_UserName]  ; 用户名左移 1 位
.text:00401C69                  add     eax, 1
.text:00401C6C                  mov     [ebp+i_UserName], eax
.text:00401C6F                  jmp     short loc_401BFE
.text:00401C71 loc_401C71:

.text:00401C71                  mov     [ebp+var_4], 0
.text:00401C78 ; hua_code
```

三、这里人为造成一个异常，跳到异常处理函数中，其 `eax` 是指向 `_S_FUNCINFO` 结构体的指针，该结构体如下：

```asm
_S_FUNCINFO STRUCT
MagicNumber  DWORD    ?    magic number
MaxState    DWORD    ?    max state in the function
PUnwindMap  DWORD    ?    unwind function list
NTryBlocks   DWORD    ?    how many try blocks
PTryBlockMap DWORD    ?    try block list
_S_FUNCINFO ENDS
```

而我们在 `0x004036F8` 中看到的数据如下：

```
004036F8  19930520
004036FC  00000002
00403700  00403718  cm.00403718
00403704  00000001
00403708  00403728  cm.00403728
```

关键在于 `PTryBlockMap` 这个变量，即异常处理数组指针，也是一个结构体，其结构如下：

```asm
S_TRYBLOCKMAPENTRY STRUCT
TryLow     DWORD    ?    begin state
TryHi      DWORD    ?    end state
CatchHi    DWORD    ?    catch end state
NCatches   DWORD    ?    how many catches
PHandlerArray  DWORD    ?    cathe block entry list
S_TRYBLOCKMAPENTRY ENDS
```

而我们在 `0x403728` 地址中看到的该结构体的数据为：

```
00403728  00000000
0040372C  00000000
00403730  00000001
00403734  00000001
00403738  00403740  cm.00403740
```

关键在于 `PHandlerArray` 这个变量，这个指针指向的结构体中即是我们所要的信息，结构体如下：

```asm
_S_HANDLERTYPE STRUCT
Adjectives      DWORD  ?  properites
PType        DWORD  ?  type_info pointer
DispCatchObj    DWORD  ?  offset from ebp
AddressOfHandler  DWORD  ?  handler address
_S_HANDLERTYPE ENDS
```

而我们在 `0x403740` 得到的数据如下：

```
00403740  00000000
00403744  00000000
00403748  00000000
0040374C  00401C93  cm.00401C93
```

在这里可以看出 `401C93` 地址为异常处理函数的地址，可以在网上找到相关资料：



## 四、该函数用于反调试（穷举当前进程 ID，得到 explorer 的 ID，检查本进程的父进程 ID 是否与 explorer.exe 的 ID 相同，若不同则关闭本程序），过程比较简单，就不贴代码了：

```
.text:00401C93 loc_401C93:
.text:00401C93 call    anti_debug
```

## 五、这里通过将返回值赋值给 EAX，使其跳转到 401C9E：

```
.text:00401C98 mov    eax, offset loc_401C9E
.text:00401C9D retn
```

## 六、将异常处理链的地址改成父函数的异常链，由于返回地址已经被修改，下面的 retn 返回到第三个处理用户名与序列号的函数：

```
.text:00401C9E loc_401C9E:
.text:00401C9E mov    [ebp+var_4], 0FFFFFFFFh
.text:00401CA5 mov    ecx, [ebp+var_C]
.text:00401CA8 mov    large fs:0, ecx
.text:00401CAF pop    edi
.text:00401CB0 pop    esi
.text:00401CB1 pop    ebx
.text:00401CB2 mov    esp, ebp
.text:00401CB4 pop    ebp
.text:00401CB5 retn
```

## 七、这里为一个循环体，通过用户名和一组顺序大写字符串 A，算出一组字符串 B 用于检查序列号是否正确，下面用字符串 A、B 表示：

### 初始化部分代码：

```
.text:00401812 mov    eax, [ebp+var0_ABCDE__WXY]
.text:00401815 mov    [ebp+var1_ABCDE__WXY], eax
.text:00401818 mov    [ebp+i], 0
.text:0040181F
```

检查变量是否为 `18h`，即 `24`，若等于，则跳出循环：

```
.text:0040181F loc_40181F:
.text:0040181F cmp    [ebp+i], 18h
.text:00401823 jz     loc_4018BB
.text:00401829 ; hua_code
```

检查用户名的首尾字符是否等于字符串 A 当前的首字母的小写，若相等，数组左移一位：

```
.text:00401835 movsx  ecx, UserName
.text:0040183C mov    edx, [ebp+var0_ABCDE__WXY]
.text:0040183F movsx  eax, byte ptr [edx]
.text:00401842 add    eax, 20h
.text:00401845 cmp    ecx, eax
.text:00401847 jz     short loc_40186A
.text:00401849 push   offset UserName ; Str
.text:0040184E call   strlen
.text:00401853 add    esp, 4
.text:00401856 movsx  ecx, byte_40416B[eax]
.text:0040185D mov    edx, [ebp+var0_ABCDE__WXY]
.text:00401860 movsx  eax, byte ptr [edx]
.text:00401863 add    eax, 20h
.text:00401866 cmp    ecx, eax
.text:00401868 jnz    short loc_401873
.text:0040186A
.text:0040186A loc_40186A:
.text:0040186A mov    ecx, [ebp+var0_ABCDE__WXY]
.text:0040186D add    ecx, 1
.text:00401870 mov    [ebp+var0_ABCDE__WXY], ecx
```

将当前字符串 A 的首字母放入字符串 B 中：

```
.text:00401873 loc_401873:
.text:00401873 mov    edx, [ebp+i]
.text:00401876 mov    eax, [ebp+var0_ABCDE__WXY]
.text:00401879 mov    cl, [eax]
.text:0040187B mov    [ebp+edx+var_CrackStr], cl
```

变量自增一，字符串 A 左移 2 位：

```
.text:0040187F mov    edx, [ebp+i]
.text:00401882 add    edx, 1
.text:00401885 mov    [ebp+i], edx
.text:00401888 mov    eax, [ebp+var0_ABCDE__WXY]
.text:0040188B add    eax, 2
.text:0040188E mov    [ebp+var0_ABCDE__WXY], eax
.text:00401891 ; hua_code
```

左移后检查字符串 A 是否已经移动到末尾，若未移动到末尾，则继续循环：

```
.text:0040189D mov    ecx, [ebp+var0_ABCDE__WXY]
.text:004018A0 movsx  edx, byte ptr [ecx]
.text:004018A3 test   edx, edx
.text:004018A5 jnz    short loc_4018B6
```

若已经移动到了末尾，则检查是否已经组成了 24 个字符长度的字符串 B，若已经组成，则退出循环，否则将字符串 B 的地址自加一后赋值给字符串 A，继续循环：

```
.text:004018A7 cmp    [ebp+i], 18h
.text:004018AB jge    short loc_4018B6
.text:004018AD mov    eax, [ebp+var1_ABCDE__WXY]
.text:004018B0 add    eax, 1
.text:004018B3 mov    [ebp+var0_ABCDE__WXY], eax
.text:004018B6
.text:004018B6 loc_4018B6:
.text:004018B6 jmp    loc_40181F
```

## 八、这一步注意修改 MessageBox 的句柄，即密码的第十一位一定要为 `"U"`，否则就会失败：

```
.text:004018BB push    0               ; uType
.text:004018BD push    offset aA       ; "错了!"
.text:004018C2 push    offset asc_403554 ; "继续努力!"
.text:004018C7 movsx   ecx, byte_4040F6
.text:004018CE sub     ecx, 55h       0x55 == "U"
.text:004018D1 neg     ecx
.text:004018D3 sbb     ecx, ecx
.text:004018D5 inc     ecx
.text:004018D6 push    ecx             ; hWnd
.text:004018D7 call    ds:MessageBoxA
```

## 九、这里是一个双层循环，也是最后的验证：

### 初始化变量：

```
.text:004018FB mov    [ebp+i], 0
.text:00401902 mov    [ebp+var_30], 5
.text:00401909 mov    [ebp+j], 0
```

`var_30` 作为基值，乘 4 减 4 后赋值给字符串 B 的偏移：

```
.text:00401910 loc_401910:
.text:00401910 cmp    [ebp+i], 0Ch
.text:00401914 jz     loc_4019D5
.text:0040191A mov    eax, [ebp+var_30]
.text:0040191D lea    ecx, ds:0FFFFFFFCh[eax*4] ; eax * 4 - 4
.text:00401924 mov    [ebp+Str_Offset], ecx
```

这里为里面的循环，允许字符串 B 偏移为首地址的 4 个字符中任意一个为当前的序列号值：

```
.text:00401927 loc_401927:
.text:00401927 mov    edx, [ebp+i]
.text:0040192A movsx  eax, Password[edx]
.text:00401931 mov    ecx, [ebp+Str_Offset]
.text:00401934 movsx  edx, [ebp+ecx+var_CrackStr]
.text:00401939 mov    ecx, [ebp+Str_Offset]
.text:0040193C add    ecx, 1
.text:0040193F mov    [ebp+Str_Offset], ecx
.text:00401942 cmp    eax, edx
.text:00401944 jz     short loc_401959
.text:00401946 mov    edx, [ebp+j]
.text:00401949 add    edx, 1
.text:0040194C mov    [ebp+j], edx
.text:0040194F cmp    [ebp+j], 4
.text:00401953 jle    short loc_401957
.text:00401955 jmp    short loc_401959
.text:00401957 loc_401957:
.text:00401957 jmp    short loc_401927
.text:00401959 loc_401959:
.text:00401959 cmp    [ebp+j], 5      ; 检查是不是超过 5 次
.text:0040195D jnz    short loc_40199E    ; 超过 5 次不对即报错
.text:00401999 call   Msg_Error
```

序列号的序号自加 2：

```
.text:0040199E loc_40199E:
.text:0040199E mov    eax, [ebp+i]
.text:004019A1 add    eax, 2
.text:004019A4 mov    [ebp+i], eax
```

`; var_30` 基值自减 1，减到 0 时初始化为 6：

```
.text:004019A7 mov    ecx, [ebp+var_30]
.text:004019AA sub    ecx, 1
.text:004019AD mov    [ebp+var_30], ecx
.text:004019B0 ; hua_code
.text:004019BC cmp    [ebp+var_30], 0
.text:004019C0 jnz    short loc_4019C9
.text:004019C2 mov    [ebp+var_30], 6
.text:004019C9 loc_4019C9:
.text:004019C9 mov    [ebp+j], 0
.text:004019D0 jmp    loc_401910    这是外循环的回跳处
.text:004019D5 ; ---------------------------------------------------------------------------
```




```
.text:004019D5 loc_4019D5:                             ; CODE XREF: Msg_Info+124 j
.text:004019D5          call    Msg_Ok      最后报成功～～～～！
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `keygen.rar` |
