
# HiHand 2005 Build 0430

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-214.htm`](https://www.pediy.com/kssd/pediy07/pediy7-214.htm)

## HiHand 结构设计工具箱 2005 Build 0430 破解教程

软件大小：1134 KB

软件下载地址：http://www.skycn.com/soft/16018.html

### 简介

『HiHand 结构设计工具箱』是依照现行建筑结构设计规范编制的一款结构设计软件。软件主要面向结构工程师，旨在解决建筑结构设计中的一些手算问题。

`ASP2.12` 壳用 `AspackDie141` 脱掉。

### 破解目的

为了学习解密编程。

### 破解过程

学习了头疼的浮点算法。

注册加密在 `UIClass.dll` 中，用了 `ASP2.12` 壳，用 `AspackDie141` 脱掉。VCMFC 编写。

在 `RegQueryValueExA` 下断：

```text
\====================================================================================================

1000AB8E   |.  50                 push eax
1000AB8F   |.  8D4C24 18          lea ecx,dword ptr ss:[esp+18]
1000AB93   |.  C68424 5C040000 0> mov byte ptr ss:[esp+45C],6
1000AB9B   |.  E8 38CB0000        call <jmp.&MFC42.#858_CString::operator=>
1000ABA0   |.  8D4C24 18          lea ecx,dword ptr ss:[esp+18]
1000ABA4   |.  889C24 58040000    mov byte ptr ss:[esp+458],bl
1000ABAB   |.  E8 62CA0000        call <jmp.&MFC42.#800_CString::~CString>
1000ABB0   |.  68 02000080        push 80000002
1000ABB5   |.  8D4C24 28          lea ecx,dword ptr ss:[esp+28]
1000ABB9   |.  E8 221C0000        call UIClass.CRegistry::CRegistry
1000ABBE   |.  68 E0300210        push UIClass.100230E0                   ;  ASCII "Software\\HiHand"
1000ABC3   |.  8D4C24 28          lea ecx,dword ptr ss:[esp+28]
1000ABC7   |.  C68424 5C040000 0> mov byte ptr ss:[esp+45C],7
1000ABCF   |.  E8 2C1D0000        call UIClass.CRegistry::Open
1000ABD4   |.  8D5424 0C          lea edx,dword ptr ss:[esp+C]
1000ABD8   |.  8D4C24 24          lea ecx,dword ptr ss:[esp+24]
1000ABDC   |.  52                 push edx                                 ; /Arg2
1000ABDD   |.  68 D8300210        push UIClass.100230D8                  ; |Arg1 = 100230D8 ASCII "UName"
1000ABE2   |.  E8 B91E0000        call UIClass.CRegistry::Read           ; \\CRegistry::Read
1000ABE7   |.  8D4424 10          lea eax,dword ptr ss:[esp+10]
1000ABEB   |.  8D4C24 24          lea ecx,dword ptr ss:[esp+24]
1000ABEF   |.  50                 push eax                                 ; /Arg2
1000ABF0   |.  68 D0300210        push UIClass.100230D0                  ; |Arg1 = 100230D0 ASCII "RegNo"
1000ABF5   |.  E8 A61E0000        call UIClass.CRegistry::Read           ; \\CRegistry::Read
1000ABFA   |.  8D4C24 24          lea ecx,dword ptr ss:[esp+24]
1000ABFE   |.  8BF8               mov edi,eax
1000AC00   |.  E8 3B1D0000        call UIClass.CRegistry::Close
1000AC05   |.  85FF               test edi,edi
1000AC07   |.  0F84 13010000      je UIClass.1000AD20
1000AC0D   |.  8D4C24 14          lea ecx,dword ptr ss:[esp+14]
1000AC11   |.  8D5424 0C          lea edx,dword ptr ss:[esp+C]
1000AC15   |.  51                 push ecx
1000AC16   |.  8D4424 1C          lea eax,dword ptr ss:[esp+1C]
1000AC1A   |.  52                 push edx
1000AC1B   |.  50                 push eax
1000AC1C   |.  E8 A5CD0000        call <jmp.&MFC42.#922_operator+>
1000AC21   |.  50                 push eax
1000AC22   |.  8D4C24 24          lea ecx,dword ptr ss:[esp+24]
1000AC26   |.  C68424 5C040000 0> mov byte ptr ss:[esp+45C],8
1000AC2E   |.  E8 A5CA0000        call <jmp.&MFC42.#858_CString::operator=>
1000AC33   |.  8D4C24 18          lea ecx,dword ptr ss:[esp+18]
1000AC37   |.  C68424 58040000 0> mov byte ptr ss:[esp+458],7
1000AC3F   |.  E8 CEC90000        call <jmp.&MFC42.#800_CString::~CString>
1000AC44   |.  8D4C24 20          lea ecx,dword ptr ss:[esp+20]
1000AC48   |.  8D5424 18          lea edx,dword ptr ss:[esp+18]
1000AC4C   |.  51                 push ecx                                 ; /Arg2
1000AC4D   |.  52                 push edx                                 ; |Arg1
1000AC4E   |.  8BCE               mov ecx,esi                              ; |
1000AC50   |.  E8 6B010000        call UIClass.CCheckReg::HowAreYou        ; \\CCheckReg::HowAreYou 这个函数名有意思吧  f7

\===================================================================================================================

1000ADC0 U>/$  6A FF               push -1
1000ADC2   |.  68 0F8A0110        push UIClass.10018A0F                  ;  SE handler installation
1000ADC7   |.  64:A1 00000000     mov eax,dword ptr fs:[0]
1000ADCD   |.  50                 push eax
1000ADCE   |.  64:8925 00000000   mov dword ptr fs:[0],esp
1000ADD5   |.  83EC 24            sub esp,24
1000ADD8   |.  53                 push ebx
1000ADD9   |.  33C0               xor eax,eax
1000ADDB   |.  55                 push ebp
1000ADDC   |.  56                 push esi
1000ADDD   |.  33ED               xor ebp,ebp
1000ADDF   |.  894424 25          mov dword ptr ss:[esp+25],eax
1000ADE3   |.  57                 push edi
1000ADE4   |.  8D4C24 10          lea ecx,dword ptr ss:[esp+10]
1000ADE8   |.  896C24 1C          mov dword ptr ss:[esp+1C],ebp
1000ADEC   |.  C64424 28 00       mov byte ptr ss:[esp+28],0
1000ADF1   |.  894424 2D          mov dword ptr ss:[esp+2D],eax
1000ADF5   |.  E8 30C80000        call <jmp.&MFC42.#540_CString::CString>
1000ADFA   |.  8B4C24 48          mov ecx,dword ptr ss:[esp+48]
1000ADFE   |.  33C0               xor eax,eax
1000AE00   |.  33D2               xor edx,edx
1000AE02   |.  BE 01000000        mov esi,1
1000AE07   |.  8B19               mov ebx,dword ptr ds:[ecx]
1000AE09   |.  83C9 FF            or ecx,FFFFFFFF
1000AE0C   |.  8BFB               mov edi,ebx
1000AE0E   |.  897424 3C          mov dword ptr ss:[esp+3C],esi
1000AE12   |.  F2:AE              repne scas byte ptr es:[edi]
1000AE14   |.  F7D1               not ecx
1000AE16   |.  49                 dec ecx                                 ;  ecx=len
1000AE17   |.  896C24 14          mov dword ptr ss:[esp+14],ebp
1000AE1B   |.  895C24 18          mov dword ptr ss:[esp+18],ebx
1000AE1F   |.  8BF9               mov edi,ecx
1000AE21   |.  0F84 A5010000      je UIClass.1000AFCC
1000AE27   |.  3BFE               cmp edi,esi
1000AE29   |.  897424 48          mov dword ptr ss:[esp+48],esi
1000AE2D   |.  0F8C AD000000      jl UIClass.1000AEE0
1000AE33   |.  DD05 00A80110      fld qword ptr ds:[1001A800]
1000AE39   |.  EB 04              jmp short UIClass.1000AE3F
1000AE3B   |>  8B5C24 18          /mov ebx,dword ptr ss:[esp+18]            ;  ebx=name
1000AE3F   |>  8A5C33 FF          mov bl,byte ptr ds:[ebx+esi-1]            ;  bl=name[i]
1000AE43   |.  DB4424 48          |fild dword ptr ss:[esp+48]
1000AE47   |.  0FBED3             |movsx edx,bl                             ;  edx=bl
1000AE4A   |.  DD5C24 20          |fstp qword ptr ss:[esp+20]
1000AE4E   |.  895424 48          |mov dword ptr ss:[esp+48],edx
1000AE52   |.  DB4424 48          |fild dword ptr ss:[esp+48]               ;  st0=name[i]
1000AE56   |.  0FBEC3             |movsx eax,bl                             ;  eax=bl
1000AE59   |.  D9FA               |fsqrt                                     ;  st0=sqrt(bl) 用户名求平方根
1000AE5B   |.  0FAFC6             |imul eax,esi                              ;  eax*=esi
1000AE5E   |.  DC4C24 20          |fmul qword ptr ss:[esp+20]               ;  st0*(i+1)
1000AE62   |.  DC05 F8A70110      |fadd qword ptr ds:[1001A7F8]             ;  st0++
1000AE68   |.  0FAFC6             |imul eax,esi                              ;  eax*=esi
1000AE6B   |.  894424 48          |mov dword ptr ss:[esp+48],eax            ;  [esp+48]=eax
1000AE6F   |.  DB4424 48          |fild dword ptr ss:[esp+48]               ;  st1=[esp+48]
```



```
1000AE73   |.  DEC9                |fmulp st(1),st                          ; sum=st0*=st1

1000AE75   |.  D8C1                |fadd st,st(1)

1000AE77   |.  E8 94CD0000        |call <jmp.&MSVCRT._ftol>                 ; 结果取整后保存到 EAX

1000AE7C   |.  99                 |cdq

1000AE7D   |.  DDD8               |fstp st

1000AE7F   |.  B9 A0860100        |mov ecx,186A0                            ; ecx=186A0

1000AE84   |.  F7F9               |idiv ecx                                 ; eax/ecx

1000AE86   |.  895424 14          |mov dword ptr ss:[esp+14],edx             ; [esp+14]=eax%ecx  这里保存累计计算重要数

1000AE8A   |.  0FBED3             |movsx edx,bl                             ; edx=bl (name[i])

1000AE8D   |.  895424 48          |mov dword ptr ss:[esp+48],edx

1000AE91   |.  DB4424 48          |fild dword ptr ss:[esp+48]                 ; st0=edx

1000AE95   |.  DD05 10AE0110      |fld qword ptr ds:[1001AE10]

1000AE9B   |.  E8 6ACE0000        |call <jmp.&MSVCRT._CIpow>                 ; st0*st0

1000AEA0   |.  DC4C24 20          |fmul qword ptr ss:[esp+20]

1000AEA4   |.  E8 67CD0000       |call <jmp.&MSVCRT._ftol>                 ; 结果取整后保存到 EAX

1000AEA9   |.  DB4424 14          |fild dword ptr ss:[esp+14]

1000AEAD   |.  8BCE               |mov ecx,esi                              ; ecx=i+1

1000AEAF   |.  0FAFCD             |imul ecx,ebp                             ; ecx*ebp

1000AEB2   |.  D9C0               |fld st

1000AEB4   |.  D9FA               |fsqrt                                      ; sqrt(sum)

1000AEB6   |.  03C1               |add eax,ecx                              ; eax+ecx

1000AEB8   |.  B9 A0860100        |mov ecx,186A0

1000AEBD   |.  99                 |cdq

1000AEBE   |.  F7F9               |idiv ecx                                 ; eax/ecx

1000AEC0   |.  8BEA               |mov ebp,edx                              ; ebp=eax%ecx

1000AEC2   |.  E8 49CD0000       |call <jmp.&MSVCRT._ftol>                 ; eax=sqrt(sum)

1000AEC7   |.  03C5               |add eax,ebp                              ; eax+=ebp

1000AEC9   |.  B9 A0860100        |mov ecx,186A0

1000AECE   |.  99                 |cdq

1000AECF   |.  F7F9               |idiv ecx                                 ; eax/186a0

1000AED1   |.  46                 |inc esi

1000AED2   |.  3BF7               |cmp esi,edi

1000AED4   |.  897424 48          |mov dword ptr ss:[esp+48],esi

1000AED8   |.^ 0F8E 5DFFFFFF      \\jle UIClass.1000AE3B

1000AEDE   |.  DDD8               fstp st

1000AEE0   |>  33C0               xor eax,eax                                ; eax=0

1000AEE2   |>  8BC8               /mov ecx,eax                               ; ecx=eax

1000AEE4   |.  8B7424 14          mov esi,dword ptr ss:[esp+14]              ; esi=[esp+14]=sum

1000AEE8   |.  0FAFC8            imul ecx,eax                                ; ecx=eax

1000AEEB   |.  0FAFC8            imul ecx,eax                                ; ecx=eax

1000AEEE   |.  8D4C31 1F         lea ecx,dword ptr ds:[ecx+esi+1F]          ; ecx=ecx+esi+if

1000AEF2   |.  81E1 7F000080     and ecx,8000007F                            ; ecx&8000007f

1000AEF8   |.  79 05             jns short UIClass.1000AEFF

1000AEFA   |.  49                 dec ecx

1000AEFB   |.  83C9 80            or ecx,FFFFFF80

1000AEFE   |.  41                 inc ecx                                     ; ecx++

1000AEFF   |>  884C04 28          mov byte ptr ss:[esp+eax+28],cl             ; zhucema[i]=cl

1000AF03   |.  40                 inc eax                                     ; eax++

1000AF04   |.  83F8 03            cmp eax,3

1000AF07   |.^ 7C D9             \\jl short UIClass.1000AEE2

1000AF09   |.  B8 03000000        mov eax,3                                   ; eax=3

1000AF0E   |>  8BC8               /mov ecx,eax                                ; ecx=eax

1000AF10   |.  0FAFC8            imul ecx,eax                                ; ecx*=eax

1000AF13   |.  0FAFC8            imul ecx,eax                                ; ecx*=eax

1000AF16   |.  8D4C29 1F         lea ecx,dword ptr ds:[ecx+ebp+1F]

1000AF1A   |.  81E1 7F000080     and ecx,8000007F

1000AF20   |.  79 05             jns short UIClass.1000AF27

1000AF22   |.  49                 dec ecx

1000AF23   |.  83C9 80            or ecx,FFFFFF80

1000AF26   |.  41                 inc ecx

1000AF27   |>  884C04 28          mov byte ptr ss:[esp+eax+28],cl

1000AF2B   |.  40                 inc eax

1000AF2C   |.  83F8 06            cmp eax,6

1000AF2F   |.^ 7C DD             \\jl short UIClass.1000AF0E

1000AF31   |.  B8 03000000        mov eax,3

1000AF36   |>  8BC8               /mov ecx,eax

1000AF38   |.  0FAFC8            imul ecx,eax

1000AF3B   |.  0FAFC8            imul ecx,eax

1000AF3E   |.  8D4C11 1F         lea ecx,dword ptr ds:[ecx+edx+1F]

1000AF42   |.  81E1 7F000080     and ecx,8000007F

1000AF48   |.  79 05             jns short UIClass.1000AF4F

1000AF4A   |.  49                 dec ecx

1000AF4B   |.  83C9 80            or ecx,FFFFFF80

1000AF4E   |.  41                 inc ecx

1000AF4F   |>  884C04 28          mov byte ptr ss:[esp+eax+28],cl

1000AF53   |.  40                 inc eax

1000AF54   |.  83F8 08            cmp eax,8

1000AF57   |.^ 7C DD             \\jl short UIClass.1000AF36

1000AF59   |.  33D2              xor edx,edx                                ; edx=0

1000AF5B   |.  C64424 30 00      mov byte ptr ss:[esp+30],0

1000AF60   |.  33C9              xor ecx,ecx

1000AF62   |>  8A4414 28         /mov al,byte ptr ss:[esp+edx+28]

1000AF66   |.  3C 30             cmp al,30

1000AF68   |.  7C 04             jl short UIClass.1000AF6E                   ; 小于 30 吗

1000AF6A   |.  3C 39             cmp al,39

1000AF6C   |.  7E 29             jle short UIClass.1000AF97                  ; 小于等于 39 吗

1000AF6E   |>  3C 41             cmp al,41                                   ; 小于 41 吗

1000AF70   |.  7C 04             jl short UIClass.1000AF76

1000AF72   |.  3C 5A             cmp al,5A

1000AF74   |.  7E 21             jle short UIClass.1000AF97                  ; 小于等于 5A 吗

1000AF76   |>  3C 61             cmp al,61

1000AF78   |.  7C 04             jl short UIClass.1000AF7E

1000AF7A   |.  3C 7A             cmp al,7A

1000AF7C   |.  7E 19             jle short UIClass.1000AF97

1000AF7E   |>  0FBEC0           movsx eax,al

1000AF81   |.  8D4408 1F         lea eax,dword ptr ds:[eax+ecx+1F]           ; EAX=EAX+ECX+IF

1000AF85   |.  25 7F000080      and eax,8000007F

1000AF8A   |.  79 05             jns short UIClass.1000AF91

1000AF8C   |.  48                 dec eax

1000AF8D   |.  83C8 80            or eax,FFFFFF80

1000AF90   |.  40                 inc eax

1000AF91   |>  884414 28          mov byte ptr ss:[esp+edx+28],al

1000AF95   |.^ EB CB             |jmp short UIClass.1000AF62

1000AF97   |>  3C 4F             cmp al,4F                                   ; 不等于 4F 吗 等于 4F 就换成 44

1000AF99   |.  75 05             jnz short UIClass.1000AFA0

1000AF9B   |.  C64414 28 44      mov byte ptr ss:[esp+edx+28],44

1000AFA0   |>  807C14 28 6F     cmp byte ptr ss:[esp+edx+28],6F

1000AFA5   |.  75 05             jnz short UIClass.1000AFAC                  ; 不等于 6F 吗 等于 6F 就换成 48

1000AFA7   |.  C64414 28 48      mov byte ptr ss:[esp+edx+28],48

1000AFAC   |>  83C1 07          add ecx,7                                   ; ECX+=7

1000AFAF   |.  42                 inc edx                                     ; EDX++
```



```
1000AFB0   |.   83F9 38           |cmp ecx,38                                ;  ECX<38 吗  

1000AFB3   |.^ 7C AD             \jl short UIClass.1000AF62  

1000AFB5   |.   8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

1000AFB9   |.   8D5424 10        lea edx,dword ptr ss:[esp+10]  

1000AFBD   |.   51             push ecx  

1000AFBE   |.   68 F4300210    push UIClass.100230F4                       ;  ASCII "%s"  

1000AFC3   |.   52             push edx  

1000AFC4   |.   E8 09CA0000    call <jmp.&MFC42.#2818_CString::Format>  

1000AFC9   |.   83C4 0C        add esp,0C  

1000AFCC   |>   8D4C24 10      lea ecx,dword ptr ss:[esp+10]  

1000AFD0   |.   E8 F7C90000    call <jmp.&MFC42.#4204_CString::MakeUpper>  

1000AFD5   |.   8B7424 44      mov esi,dword ptr ss:[esp+44]  
```

注册机算法：

```c
#include<iostream.h>  

#include <conio.h>  

#include <stdlib.h>  

#include<math.h>  

#include<string.h>  

void main()  

{  

  char jqm[8]={0}/*  这是我的机器码 AP8UFEPM   */;  

  char name[12]={0}, zhucema[12]={0};  

  unsigned int len, tmp=0, sum=0, ebp=0, edx=0, ecx=0, tt=0;  

  double res;  

    cout<<"请输入 3 位用户名"<<endl;  

  cin>>name;  

  len=strlen(name);  

  if(len!=3)  

  {  

    cout<<"请输入 3 位用户名"<<endl;  

      exit(0);  

  }  

    cout<<"请输入 8 位机器码"<<endl;  

  cin>>jqm;  

  cout<<"name is:"<<name<<endl<<"jqm is :"<<jqm<<endl;  

  strcat(name, jqm);  

  len=strlen(name);  

  cout<<"cat name is:"<<name<<endl;  

  for(unsigned int i=0; i<len; i++)  

  {  

    tmp=name[i];  

    res=sqrt(tmp);//用户名求平方根  

    res*=(i+1);  

    res+=1;  

    tmp*=(i+1);  

    tmp*=(i+1);  

    res*=tmp;  

    tt=res;//取整保存  

    sum+=tt;  

    sum=sum%0x186A0;  

    tmp=pow(name[i],2)*(i+1);//后面利用  

    ///////////////////////////////////////////////  

    ecx=i+1;  

    ecx*=ebp;  

    tmp+=ecx;  

        res=sqrt(sum);  

        ebp=tmp%0x186A0;//后面利用  

    ///////////////////////////////////////////  

    tmp=res;//取整  

    tmp+=ebp;  

        edx=tmp%0x186A0;//后面用  

  }  

  ecx=0;  

  for(i=0; i<3; i++)  

  {  

    ecx=i;  

    tmp=sum;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  for(i=3; i<6; i++)  

  {  

    ecx=i;  

    tmp=ebp;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  for(i=3; i<8; i++)  

  {  

    ecx=i;  

    tmp=edx;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  ecx=0;  

  for(i=0; i<8; i++)  

  {  

    if(zhucema[i]<'0'||zhucema[i]>'9')//  

    {  

      if(zhucema[i]<'A'||zhucema[i]>'Z')//  

      {  

        if(zhucema[i]<'a'||zhucema[i]>'z')//  

        {  

          tmp=zhucema[i];  

            tmp=ecx+tmp+0x1f;  

          tmp=tmp&0x8000007F;  

          zhucema[i]=tmp;  

          if(zhucema[i]<'0'||zhucema[i]>'9')//  

          {  

            if(zhucema[i]<'A'||zhucema[i]>'Z')//  

            {  

              if(zhucema[i]<'a'||zhucema[i]>'z')//  

              {  

                tmp=zhucema[i];  

                tmp=ecx+tmp+0x1f;  

                tmp=tmp&0x8000007F;  

                zhucema[i]=tmp;  

              }  

            }  

          }  

        }  

      }  

      ecx+=7;  

    }  

  }  

  for(i=0; i<8; i++)  

  {  

    if(zhucema[i]==0x4f)  

      zhucema[i]=0x44;  

    else  

      if(zhucema[i]==0x6f)  

      zhucema[i]=0x48;  

  }  

  cout<<zhucema<<endl;  

}
```

\==============================================================================================  

如有得罪或疏漏之处请谅解，下次再会！

```
0041812B  |.  8B69 F8          mov ebp,dword ptr ds:[ecx-8]  

0041812E  |.  896C24 34        mov dword ptr ss:[esp+34],ebp  

00418132  |.  E8 9FF30100      call tcsearch.004374D6  

00418137  |.  8B00             mov eax,dword ptr ds:[eax]  

00418139  |.  8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

0041813D  |.  68 9CD74700      push tcsearch.0047D79C                  ; /Arg2 = 0047D79C ASCII "%Y%m%d"  

00418142  |.  51               push ecx                                ; |Arg1  

00418143  |.  8D4C24 34        lea ecx,dword ptr ss:[esp+34]           ; |  

00418147  |.  894424 34        mov dword ptr ss:[esp+34],eax          ; |  

0041814B  |.  E8 99F30100      call tcsearch.004374E9                  ; \EAX=系统日期 20050430  

00418150  |.  3BEF             cmp ebp,edi  

00418152  |.  C64424 58 03      mov byte ptr ss:[esp+58],3  

00418157  |.  0F84 34020000     je tcsearch.00418391  

0041815D  |.  BE 01000000      mov esi,1                               ;  ESI=1  

00418162  |.  3BEE             cmp ebp,esi  

00418164  |.  897424 1C        mov dword ptr ss:[esp+1C],esi  

00418168  |.  0F8C 37010000     jl tcsearch.004182A5  

0041816E  |.  8B4424 28        mov eax,dword ptr ss:[esp+28]           ;  EAX=系统日期 20050430  

00418172  |.  DD05 80664400    fld qword ptr ds:[446680]  

00418178  |.  8A58 06          mov bl,byte ptr ds:[eax+6]              ;  BL=date[6]  

0041817B  |.  8A48 04          mov cl,byte ptr ds:[eax+4]              ;  CL=date[4]  

0041817E  |.  885C24 13        mov byte ptr ss:[esp+13],bl             ;  [ESP+13]=BL=date[6]  

00418182  |.  8A58 07          mov bl,byte ptr ds:[eax+7]              ;  BL=date[7]  

00418185  |.  0FBEDB           movsx ebx,bl                            ;  ebx=bl  

00418188  |.  895C24 24        mov dword ptr ss:[esp+24],ebx           ;  [esp+24]=ebx=date[7]  

0041818C  |.  8A50 05          mov dl,byte ptr ds:[eax+5]              ;  DL=date[5]  

0041818F  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=[esp+24]=date[7]  

00418193  |.  0FBE5C24 13      movsx ebx,byte ptr ss:[esp+13]          ;  EBX=[ESP+13]=date[6]  

00418198  |.  D9FA             fsqrt                                   ;  ST1=SQRT（ST0）  

0041819A  |.  895C24 24        mov dword ptr ss:[esp+24],ebx           ;  [ESP+24]=EBX=date[6]  

0041819E  |.  8A40 03          mov al,byte ptr ds:[eax+3]              ;  AL=date[3]  

004181A1  |.  0FBEC0           movsx eax,al                            ;  eax=date[3]  

004181A4  |.  0FBEEA           movsx ebp,dl                            ;  EBP=DL=date[5]  

004181A7  |.  0FBEC9           movsx ecx,cl                            ;  ECX=CL  

004181AA  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=date[6]  

004181AE  |.  0FAFE9           imul ebp,ecx                            ;  EBP=EBP*ECX 这个数据要记住 ！  

004181B1  |.  D9FA             fsqrt                                   ;  ST0=SQRT（ST0）  

004181B3  |.  894424 24        mov dword ptr ss:[esp+24],eax           ;  [ESP+24]=EAX=date[3]  

004181B7  |.  DEC9             fmulp st(1),st                          ;  RES=ST0*ST1  

004181B9  |.  DD5C24 38        fstp qword ptr ss:[esp+38]              ;  这个数据要记住 ！  

004181BD  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=date[3]****这个重要  

004181C1  |.  D9FA             fsqrt                                   ;  SQRT（ST0）  

004181C3  |.  DCC0             fadd st,st  

004181C5  |.  DD5C24 40        fstp qword ptr ss:[esp+40]  

004181C9  |>  8B5424 64        /mov edx,dword ptr ss:[esp+64]          ;  EDX=NAME  

004181CD  |.  DB4424 1C        |fild dword ptr ss:[esp+1C]  

004181D1  |.  8A5C32 FF       |mov bl,byte ptr ds:[edx+esi-1]         ;  BL=NAME[I]  

004181D5  |.  0FBEC3          |movsx eax,bl                           ;  EAX=NAME[I]  

004181D8  |.  DD5C24 1C       |fstp qword ptr ss:[esp+1C]  

004181DC  |.  894424 24       |mov dword ptr ss:[esp+24],eax  
```



```
004181E0  |.  DB4424 24           |fild dword ptr ss:[esp+24]               ; ST0=NAME[I]  
004181E4  |.  0FBECB            |movsx ecx,bl                              ; ECX=BL  
004181E7  |.  D9FA              |fsqrt                                       ; ST0=SQRT（NAME[I]）  
004181E9  |.  0FAFCE            |imul ecx,esi                                ; ECX=ECX*ESI  
004181EC  |.  DC4C24 1C         |fmul qword ptr ss:[esp+1C]                 ; ST0=ST0*（i+1）  
004181F0  |.  DC05 78664400     |fadd qword ptr ds:[446678]                ; st0+=1  

004181F6  |.  0FAFCE            |imul ecx,esi  
004181F9  |.  0FAFCE            |imul ecx,esi  

004181FC  |.  894C24 24         |mov dword ptr ss:[esp+24],ecx  
00418200  |.  DB4424 24         |fild dword ptr ss:[esp+24]  
00418204  |.  DEC9              |fmulp st(1),st  
00418206  |.  D8C1              |fadd st,st(1)  

00418208  |.  E8 93090100       |call tcsearch.00428BA0                    ; 取整  
0041820D  |.  99                |cdq  
0041820E  |.  DDD8              |fstp st  

00418210  |.  B9 18010000       |mov ecx,118                               ; ECX/118  
00418215  |.  F7F9              |idiv ecx  
00418217  |.  895424 14         |mov dword ptr ss:[esp+14],edx             ; 余数保存在[ESP+14]这个重要  

0041821B  |.  0FBED3            |movsx edx,bl                              ; EDX=NAME[I]  
0041821E  |.  895424 24         |mov dword ptr ss:[esp+24],edx  
00418222  |.  DB4424 24         |fild dword ptr ss:[esp+24]  

00418226  |.  DD05 70664400     |fld qword ptr ds:[446670]  
0041822C  |.  E8 4F070100       |call tcsearch.00428980                    ; POW 函数 ST0=NAME[I]2 次方  
00418231  |.  DC4C24 1C         |fmul qword ptr ss:[esp+1C]                ; ST0*（i+1）  

00418235  |.  E8 66090100       |call tcsearch.00428BA0                    ; 取整  
0041823A  |.  DB4424 14         |fild dword ptr ss:[esp+14]                ; st0=edx  

0041823E  |.  8BCE              |mov ecx,esi  
00418240  |.  0FAFCF            |imul ecx,edi  
00418243  |.  D9C0              |fld st  
00418245  |.  D9FA              |fsqrt                                       ; st0=sqrt(st0)  对上面的余数求平方根  
00418247  |.  03C1              |add eax,ecx  
00418249  |.  B9 2C010000       |mov ecx,12C  

0041824E  |.  99                |cdq  
0041824F  |.  F7F9              |idiv ecx  
00418251  |.  8BFA              |mov edi,edx                               ;******这个重要  

00418253  |.  E8 48090100       |call tcsearch.00428BA0                    ; eax=对上面的余数求平方根取整  
00418258  |.  03C7              |add eax,edx                               ; EAX+=EDI  
0041825A  |.  B9 A0860100       |mov ecx,186A0  

0041825F  |.  99                |cdq  
00418260  |.  DD4424 38         |fld qword ptr ss:[esp+38]  
00418264  |.  F7F9              |idiv ecx  
00418266  |.  DC4C24 1C         |fmul qword ptr ss:[esp+1C]  
0041826A  |.  DC4424 40         |fadd qword ptr ss:[esp+40]  
0041826E  |.  8BDA              |mov ebx,edx                               ;******这个重要  
00418270  |.  8BD5              |mov edx,ebp  
00418272  |.  0FAFD6            |imul edx,esi  
00418275  |.  895424 24         |mov dword ptr ss:[esp+24],edx  
00418279  |.  DB4424 24         |fild dword ptr ss:[esp+24]  

0041827D  |.  DEC9              |fmulp st(1),st                              ; 160418=2544*63.05  
0041827F  |.  D8C1              |fadd st,st(1)                               ; 160418+93=160511  

00418281  |.  E8 1A090100       |call tcsearch.00428BA0                    ; 取整  
00418286  |.  99                |cdq  
00418287  |.  B9 18010000       |mov ecx,118  
0041828C  |.  F7F9              |idiv ecx  
0041828E  |.  8B4424 30         |mov eax,dword ptr ss:[esp+30]  

00418292  |.  46                |inc esi  
00418293  |.  3BF0              |cmp esi,eax  
00418295  |.  897424 1C         |mov dword ptr ss:[esp+1C],esi  
00418299  |.  895424 24         |mov dword ptr ss:[esp+24],edx  
0041829D  |.^ 0F8E 26FFFFFF      \\jle tcsearch.004181C9  

004182A3  |.  DDD8              |fstp st  
004182A5  |>  33C9              |xor ecx,ecx                               ; ecx=0  
004182A7  |>  8BD1              /mov edx,ecx  
004182A9  |.  8B4424 14         |mov eax,dword ptr ss:[esp+14]              ; 从上面得到的[esp+14]  

004182AD  |.  0FAFD1            |imul edx,ecx                              ; edx*=ecx  
004182B0  |.  0FAFD1            |imul edx,ecx                              ; edx*=ecx  
004182B3  |.  BE 4E000000       |mov esi,4E                                ; esi=4e  
004182B8  |.  8D4402 1F         |lea eax,dword ptr ds:[edx+eax+1F]  

004182BC  |.  99                |cdq  
004182BD  |.  F7FE              |idiv esi                                  ; eax/esi  
004182BF  |.  41                |inc ecx                                    ; ecx++  
004182C0  |.  83F9 05           |cmp ecx,5  
004182C3  |.  88540C 3F         |mov byte ptr ss:[esp+ecx+3F],dl           ; zhucema[i]=dl  
004182C7  |.^ 7C DE             \\jl short tcsearch.004182A7  

004182C9  |.  B9 05000000       |mov ecx,5                                 ; ecx=5  
004182CE  |>  8BD1              /mov edx,ecx  
004182D0  |.  BE 58000000       |mov esi,58  
004182D5  |.  0FAFD1            |imul edx,ecx  
004182D8  |.  0FAFD1            |imul edx,ecx  
004182DB  |.  8D443A 1F         |lea eax,dword ptr ds:[edx+edi+1F]          ; 从上面得到的 EDI  
004182DF  |.  99                |cdq  
004182E0  |.  F7FE              |idiv esi  
004182E2  |.  41                |inc ecx  
004182E3  |.  83F9 0A           |cmp ecx,0A  
004182E6  |.  88540C 3F         |mov byte ptr ss:[esp+ecx+3F],dl  
004182EA  |.^ 7C E2             \\jl short tcsearch.004182CE  

004182EC  |.  B9 0A000000       |mov ecx,0A                                ; ecx=10  
004182F1  |>  8BC1              /mov eax,ecx  
004182F3  |.  BE 26000000       |mov esi,26  
004182F8  |.  0FAFC1            |imul eax,ecx  
004182FB  |.  0FAFC1            |imul eax,ecx  
004182FE  |.  8D4418 1F         |lea eax,dword ptr ds:[eax+ebx+1F]          ; 从上面得到的 EBX  
00418302  |.  99                |cdq  
00418303  |.  F7FE              |idiv esi  
00418305  |.  41                |inc ecx  
00418306  |.  83F9 0F           |cmp ecx,0F  
00418309  |.  88540C 3F         |mov byte ptr ss:[esp+ecx+3F],dl  
0041830D  |.^ 7C E2             \\jl short tcsearch.004182F1  

0041830F  |.  B9 0F000000       |mov ecx,0F                                ; ecx=15  
00418314  |>  8BD1              /mov edx,ecx  
00418316  |.  8B4424 24         |mov eax,dword ptr ss:[esp+24]              ; 从上面得到的 ESP+24  
0041831A  |.  0FAFD1            |imul edx,ecx  
0041831D  |.  0FAFD1            |imul edx,ecx  
00418320  |.  BE 12000000       |mov esi,12  
00418325  |.  8D4402 1F         |lea eax,dword ptr ds:[edx+eax+1F]  
00418329  |.  99                |cdq  
0041832A  |.  F7FE              |idiv esi  
0041832C  |.  41                |inc ecx  
0041832D  |.  83F9 14           |cmp ecx,14  
00418330  |.  88540C 3F         |mov byte ptr ss:[esp+ecx+3F],dl  
00418334  |.^ 7C DE             \\jl short tcsearch.00418314  

00418336  |.  33D2              |xor edx,edx                               ; edx=0  
00418338  |.  33C9              |xor ecx,ecx                               ; ecx=0  
0041833A  |>  8A4414 40         /mov al,byte ptr ss:[esp+edx+40]  
0041833E  |.  3C 30             |cmp al,30  
00418340  |.  7C 04             |jl short tcsearch.00418346  
00418342  |.  3C 39             |cmp al,39  
00418344  |.  7E 29             |jle short tcsearch.0041836F  
00418346  |>  3C 41             |cmp al,41  
00418348  |.  7C 04             |jl short tcsearch.0041834E  
0041834A  |.  3C 5A             |cmp al,5A  
```



```
0041834C  |.  7E 21             |jle short tcsearch.0041836F  

0041834E  |>  3C 61             |cmp al,61  

00418350  |.  7C 04             |jl short tcsearch.00418356  

00418352  |.  3C 7A             |cmp al,7A  

00418354  |.  7E 19             |jle short tcsearch.0041836F  

00418356  |>  0FBEC0           |movsx eax,al  

00418359  |.  8D4408 1F         |lea eax,dword ptr ds:\[eax+ecx+1F\]  

0041835D  |.  25 7F000080       |and eax,8000007F                    ;  eax&=8000007F  

00418362  |.  79 05             |jns short tcsearch.00418369  

00418364  |.  48               |dec eax  

00418365  |.  83C8 80          |or eax,FFFFFF80  

00418368  |.  40               |inc eax  

00418369  |>  884414 40        |mov byte ptr ss:\[esp+edx+40\],al  

0041836D  |.^ EB CB            |jmp short tcsearch.0041833A  

0041836F  |>  83C1 07          |add ecx,7                           ;  ecx+=7  

00418372  |.  42               |inc edx  

00418373  |.  81F9 8C000000     |cmp ecx,8C  

00418379  |.^ 7C BF            \\jl short tcsearch.0041833A  

0041837B  |.  33F6              xor esi,esi  

0041837D  |>  8A4C34 40        /mov cl,byte ptr ss:\[esp+esi+40\]  

00418381  |.  51               |push ecx  

00418382  |.  8D4C24 1C        |lea ecx,dword ptr ss:\[esp+1C\]  

00418386  |.  E8 69090200       |call tcsearch.00438CF4  

0041838B  |.  46               |inc esi  

0041838C  |.  83FE 14          |cmp esi,14  

0041838F  |.^ 7C EC            \\jl short tcsearch.0041837D  

00418391  |>  8B7424 60         mov esi,dword ptr ss:\[esp+60\]  

00418395  |.  8D5424 18        lea edx,dword ptr ss:\[esp+18\]  

00418399  |.  52               push edx  

0041839A  |.  8BCE             mov ecx,esi  
```

调用系统时间库：`time.h`

```c
#include<time.h>//时间库 

#include<iostream.h> 

#include<string.h> 

#include<math.h> 

#include<windows.h> 

void main( ) 

{ 

  char date\[9\]={0},name\[12\]={0},zhucema\[21\]={0}; 

  unsigned int tmp,sum=0,ecx,ebp,edx,ebx,edi=0,esp14,esp24; 

  double dd1,dd2,st0,st1; 

//  gets current date 获取系统日期的算法，格式为 20050501 谁还有更好的办法呀？ 

  SYSTEMTIME st; 

    unsigned int y,m,d; 

    GetSystemTime(&st); // gets current time 

    y=st.wYear; 

    m=st.wMonth; 

    d=st.wDay; 

  if((m/10==0)&&(d/10==0)) 

    wsprintf(date,"%d0%d0%d",y,m,d); 

  else if(d/10==0) 

    wsprintf(date,"%d%d0%d",y,m,d); 

  else if(m/10==0) 

    wsprintf(date,"%d0%d%d",y,m,d); 

  else 

    wsprintf(date,"%d%d%d",y,m,d); 

    cout<<date<<endl; 

//////////////////////////日期运算 

  tmp=date\[7\]; 

  st0=sqrt(tmp); 

  ebp=date\[5\]\*date\[4\]; 

  tmp=date\[6\]; 

  st1=sqrt(tmp); 

  dd1=st0\*st1; 

  tmp=date\[3\]; 

  dd2=sqrt(tmp); 

  dd2+=dd2; 

   /////用户名运算 

  cout<<"请输入用户名： "<<endl; 

    cin>>name; 

  for(unsigned int i=0;i<strlen(name);i++) 

  { 

    tmp=name\[i\]; 

    ecx=tmp; 

    st0=sqrt(tmp); 

    ecx\*=(i+1); 

    st0+=1; 

    ecx\*=(i+1); 

    ecx\*=(i+1); 

    st1=ecx; 

    st0\*=st1; 

    tmp=st0;//取整 

    sum+=tmp; 

    sum%=0x118; 

    edx=sum; 

    esp14=edx; 

    tmp=name\[i\]; 

    st0=pow(tmp,2)\*(i+1); 

    tmp=st0; 

    ecx=i+1; 

    ecx\*=edi; 

    st1=sqrt(edx); 

    tmp+=ecx; 

    edi=tmp%0x12c; 

    tmp=st1; 

    tmp+=edi; 

    ebx=tmp%0x186A0; 

    tmp/=0x186A0; 

    edx+=ebp; 

    edx\*=(i+1); 

    dd1=dd1\*(i+1); 

    dd1=dd1+dd2; 

    dd1=edx\*dd1; 

        tmp=dd1; 

    tmp%=0x118; 

    esp24=tmp; 

  } 

//esp+24=102 这个值没找出来看的我心烦 

  //\*\*esp+14=62,esp+24=102,\*\*EBX=fc,\*\*edi=f3 

  ecx=0; 

for(i=0;i<5;i++) 

{ 

  ecx=i; 

  tmp=esp14; 

  ecx\*=i; 

  ecx\*=i; 

  ecx=ecx+tmp+0x1f; 

  ecx=ecx%0x4e; 

  zhucema\[i\]=ecx; 

} 

for(i=5;i<10;i++) 

{ 

  ecx=i; 

  tmp=edi; 

  ecx\*=i; 

  ecx\*=i; 

  ecx=ecx+tmp+0x1f; 

  ecx=ecx%0x58; 

  zhucema\[i\]=ecx; 

} 

for(i=10;i<15;i++) 

{ 

  ecx=i; 

  tmp=ebx; 

  ecx\*=i; 

  ecx\*=i; 

  ecx=ecx+tmp+0x1f; 

  ecx=ecx%0x26; 

  zhucema\[i\]=ecx; 

} 

for(i=16;i<20;i++) 

{ 

  ecx=i; 

  tmp=esp24;//0x102 

  ecx\*=i; 

  ecx\*=i; 

  ecx=ecx+tmp+0x1f; 

  ecx=ecx%0x12; 

  zhucema\[i\]=ecx; 

} 

for(i=0;i<20;i++) 

{ 

to: 

  if(zhucema\[i\]<'0'||zhucema\[i\]>'9')// 

  { 

    if(zhucema\[i\]<'A'||zhucema\[i\]>'Z')// 

    { 

      if(zhucema\[i\]<'a'||zhucema\[i\]>'z')// 

      { 

      tmp=zhucema\[i\]; 

        tmp=ecx+tmp+0x1f; 

      tmp=tmp&0x8000007F; 

      zhucema\[i\]=tmp; 

      if(zhucema\[i\]<'0'||zhucema\[i\]>'9')// 

      { 

        if(zhucema\[i\]<'A'||zhucema\[i\]>'Z')// 

        { 

          if(zhucema\[i\]<'a'||zhucema\[i\]>'z')// 

          { 

           goto to; 

          } 

        } 

      } 

      } 

    } 

  } 

      ecx+=7; 

} 

cout<<endl; 

cout<<zhucema; 

} 

 4dd466d0e1e0cc123afbe504da2462ca
```
