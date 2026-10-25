
# Comic Book Manager 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/71586.html`](https://www.pediy.com/kssd/pediy10/71586.html)

## 基本信息

- **【文章标题】**: Comic Book Manager 算法分析
- **【文章作者】**: `qifeon`
- **【软件名称】**: Comic Book Manager 1.12
- **【下载地址】**: [`http://www.onlinedown.net/soft/18946.htm`](http://www.onlinedown.net/soft/18946.htm)
- **【保护方式】**: 注册码
- **【使用工具】**: `od`, `peid`
- **【操作平台】**: `winxp sp2`
- **【软件介绍】**: Comic Book Manager 是一个漫画图书管理软件！
- **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 详细过程

### 一、分析过程

1. 使用 `PEID` 查壳，显示无壳，编程语言为 Borland Delphi 6.0 - 7.0；
2. 运行软件，输入试炼码 `"qifeon,123456"`，有错误提示：`"the registration code that you provided does not\nmatch the name entered."`；
3. 使用 `OD` 载入，通过字符串插件查找 `"the registration code that you provided does not\nmatch the name entered."`。

   **Ultra String Reference, 条目 5679**

   ```
   Address=006A68A6
   Disassembly=mov     eax, 006A6914
   Text String=the registration code that you provided does not\nmatch the name entered
   ```

   双击返回后向上找到段首：

   ```asm
   006A67C0  /.  55             push    ebp                              ;  段首
   006A67C1  |.  8BEC           mov     ebp, esp
   006A67C3  |.  81C4 ECFEFFFF add     esp, -114
   006A67C9  |.  53             push    ebx
   006A67CA  |.  33C9           xor     ecx, ecx
   006A67CC  |.  898D ECFEFFFF mov     dword ptr [ebp-114], ecx
   006A67D2  |.  898D F4FEFFFF mov     dword ptr [ebp-10C], ecx
   006A67D8  |.  898D F0FEFFFF mov     dword ptr [ebp-110], ecx
   006A67DE  |.  894D FC        mov     dword ptr [ebp-4], ecx
   006A67E1  |.  894D F8        mov     dword ptr [ebp-8], ecx
   006A67E4  |.  8BD8           mov     ebx, eax
   006A67E6  |.  33C0           xor     eax, eax
   006A67E8  |.  55             push    ebp
   006A67E9  |.  68 F9686A00    push    006A68F9
   006A67EE  |.  64:FF30        push    dword ptr fs:[eax]
   006A67F1  |.  64:8920        mov     dword ptr fs:[eax], esp
   006A67F4  |.  8D55 F8        lea     edx, dword ptr [ebp-8]
   006A67F7  |.  8B83 FC020000  mov     eax, dword ptr [ebx+2FC]
   006A67FD  |.  E8 96CCFFFF    call    006A3498
   006A6802  |.  8B45 F8        mov     eax, dword ptr [ebp-8]            ;  用户名
   006A6805  |.  8D55 FC        lea     edx, dword ptr [ebp-4]
   006A6808  |.  E8 D73AD6FF    call    0040A2E4
   006A680D  |.  8B55 FC        mov     edx, dword ptr [ebp-4]
   006A6810  |.  A1 04A16B00    mov     eax, dword ptr [6BA104]
   006A6815  |.  E8 32ECD5FF    call    0040544C
   006A681A  |.  8D95 F0FEFFFF  lea     edx, dword ptr [ebp-110]
   006A6820  |.  8B83 00030000  mov     eax, dword ptr [ebx+300]
   006A6826  |.  E8 6DCCFFFF    call    006A3498
   006A682B  |.  8B85 F0FEFFFF  mov     eax, dword ptr [ebp-110]          ;  试炼码
   006A6831  |.  8D95 F4FEFFFF  lea     edx, dword ptr [ebp-10C]
   006A6837  |.  E8 A83AD6FF    call    0040A2E4
   006A683C  |.  8B95 F4FEFFFF  mov     edx, dword ptr [ebp-10C]
   006A6842  |.  8D85 F8FEFFFF  lea     eax, dword ptr [ebp-108]
   006A6848  |.  B9 FF000000    mov     ecx, 0FF
   006A684D  |.  E8 42EED5FF    call    00405694
   006A6852  |.  8D95 F8FEFFFF  lea     edx, dword ptr [ebp-108]
   006A6858  |.  A1 809E6B00    mov     eax, dword ptr [6B9E80]
   006A685D  |.  B1 1E          mov     cl, 1E
   006A685F  |.  E8 50CED5FF    call    004036B4
   006A6864  |.  6A 6B          push    6B                                  ;  参与计算常数 6Bh 入栈
   006A6866  |.  8D85 ECFEFFFF  lea     eax, dword ptr [ebp-114]
   006A686C  |.  8B15 809E6B00  mov     edx, dword ptr [6B9E80]            ;  cbm.006BE070
   006A6872  |.  E8 E5EDD5FF    call    0040565C
   006A6877  |.  8B8D ECFEFFFF  mov     ecx, dword ptr [ebp-114]
   006A687D  |.  8B15 04A16B00  mov     edx, dword ptr [6BA104]            ;  cbm.006BE06C
   006A6883  |.  8B12           mov     edx, dword ptr [edx]
   006A6885  |.  A1 88A16B00    mov     eax, dword ptr [6BA188]
   006A688A  |.  8B00           mov     eax, dword ptr [eax]
   006A688C  |.  8B80 58030000  mov     eax, dword ptr [eax+358]
   006A6892  |.  E8 1977ECFF    call    0056DFB0                            ;  关键 CALL
   006A6897  |.  84C0           test    al, al
   006A6899  |.  75 29          jnz     short 006A68C4                      ;  关键跳转
   006A689B  |.  6A 00          push    0
   006A689D  |.  66:8B0D 08696>mov     cx, word ptr [6A6908]
   006A68A4  |.  B2 01          mov     dl, 1
   006A68A6  |.  B8 14696A00    mov     eax, 006A6914                       ;  the registration code that you provided does not\nmatch the name entered.
   006A68AB  |.  E8 B868D9FF    call    0043D168
   006A68B0  |.  48             dec     eax                                   返回处
   006A68B1  |.  75 1B          jnz     short 006A68CE
   006A68B3  |.  A1 58E06B00    mov     eax, dword ptr [6BE058]
   006A68B8  |.  C780 4C020000>mov     dword ptr [eax+24C], 1
   006A68C2  |.  EB 0A          jmp     short 006A68CE
   006A68C4  |>  C783 4C020000>mov     dword ptr [ebx+24C], 1
   006A68CE  |>  33C0           xor     eax, eax
   006A68D0  |.  5A             pop     edx
   006A68D1  |.  59             pop     ecx
   006A68D2  |.  59             pop     ecx
   006A68D3  |.  64:8910        mov     dword ptr fs:[eax], edx
   006A68D6  |.  68 00696A00    push    006A6900
   006A68DB  |.  8D85 ECFEFFFF  lea     eax, dword ptr [ebp-114]
   006A68E1  |.  BA 03000000    mov     edx, 3
   006A68E6  |.  E8 31EBD5FF    call    0040541C
   006A68EB  |.  8D45 F8        lea     eax, dword ptr [ebp-8]
   006A68EE  |.  BA 02000000    mov     edx, 2
   006A68F3  |.  E8 24EBD5FF    call    0040541C
   006A68F8  |\   C3             ret
   ```

   段首下断，重载，`F9` 运行，输入试炼码 `"qifeon,123456"`，断下后单步，在 `006A6892` 处进入 `call 0056DFB0`：

   ```asm
   0056DFB0  /$  55             push    ebp
   0056DFB1  |.  8BEC           mov     ebp, esp
   0056DFB3  |.  83C4 F4        add     esp, -0C
   0056DFB6  |.  53             push    ebx
   0056DFB7  |.  56             push    esi
   0056DFB8  |.  57             push    edi
   0056DFB9  |.  33DB           xor     ebx, ebx
   0056DFBB  |.  895D F4        mov     dword ptr [ebp-C], ebx
   0056DFBE  |.  894D F8        mov     dword ptr [ebp-8], ecx            ;  试炼码
   0056DFC1  |.  8955 FC        mov     dword ptr [ebp-4], edx            ;  用户名
   0056DFC4  |.  8BF8           mov     edi, eax
   0056DFC6  |.  8B75 08        mov     esi, dword ptr [ebp+8]            ;  常数 6Bh 传入 esi
   0056DFC9  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
   0056DFCC  |.  E8 D778E9FF    call    004058A8
   0056DFD1  |.  8B45 F8        mov     eax, dword ptr [ebp-8]
   0056DFD4  |.  E8 CF78E9FF    call    004058A8
   0056DFD9  |.  33C0           xor     eax, eax
   0056DFDB  |.  55             push    ebp
   0056DFDC  |.  68 2FE05600    push    0056E02F
   0056DFE1  |.  64:FF30        push    dword ptr fs:[eax]
   0056DFE4  |.  64:8920        mov     dword ptr fs:[eax], esp
   0056DFE7  |.  33DB           xor     ebx, ebx
   0056DFE9  |.  837D FC 00     cmp     dword ptr [ebp-4], 0              ;  用户名是否为空？
   ```



**0056DFED** 处跳转流程：

```
0056DFED  |.  74 25          je      short 0056E014  
0056DFEF  |.  85F6          test    esi, esi                      ; esi 值是否为 0？  
0056DFF1  |.  74 21         je      short 0056E014  
0056DFF3  |.  8D45 F4       lea     eax, dword ptr [ebp-C]  
0056DFF6  |.  50            push    eax  
0056DFF7  |.  8BCE         mov     ecx, esi                      ; ecx=esi=6Bh  
0056DFF9  |.  8B55 FC      mov     edx, dword ptr [ebp-4]         ; 用户名  
0056DFFC  |.  8BC7        mov     eax, edi  
0056DFFE  |.  E8 8DFEFFFF  call    0056DE90                      ; 算法 call  
0056E003  |.  8B45 F4      mov     eax, dword ptr [ebp-C]         ; 真正注册码  
0056E006  |.  8B55 F8      mov     edx, dword ptr [ebp-8]         ; 试炼码  
0056E009  |.  E8 92FBFFFF  call    0056DBA0                      ; 真假注册码比较  
0056E00E  |.  84C0         test    al, al  
0056E010  |.  74 02        je      short 0056E014                 ; 关键跳转  
0056E012  |.  B3 01        mov     bl, 1  
0056E014  |>  33C0        xor     eax, eax  
0056E016  |.  5A            pop     edx                      ; (initial cpu selection)  
0056E017  |.  59            pop     ecx  
0056E018  |.  59            pop     ecx  
0056E019  |.  64:8910      mov     dword ptr fs:[eax], edx  
0056E01C  |.  68 36E05600  push    0056E036  
0056E021  |>  8D45 F4      lea     eax, dword ptr [ebp-C]  
0056E024  |.  BA 03000000  mov     edx, 3  
0056E029  |.  E8 EE73E9FF  call    0040541C  
0056E02E  \\.  C3            retn  
```

在 **0056DFFE** 处进入 `call 0056DE90`。

**0056DE90** 函数实现：

```
0056DE90  /$  55            push    ebp  
0056DE91  |.  8BEC        mov     ebp, esp  
0056DE93  |.  6A 00       push    0  
0056DE95  |.  6A 00       push    0  
0056DE97  |.  6A 00       push    0  
0056DE99  |.  6A 00       push    0  
0056DE9B  |.  6A 00       push    0  
0056DE9D  |.  53            push    ebx  
0056DE9E  |.  56            push    esi  
0056DE9F  |.  57            push    edi  
0056DEA0  |.  8BF1        mov     esi, ecx  
0056DEA2  |.  8955 FC      mov     dword ptr [ebp-4], edx  
0056DEA5  |.  8B7D 08      mov     edi, dword ptr [ebp+8]  
0056DEA8  |.  8B45 FC      mov     eax, dword ptr [ebp-4]  
0056DEAB  |.  E8 F879E9FF  call    004058A8  
0056DEB0  |.  33C0        xor     eax, eax  
0056DEB2  |.  55            push    ebp  
0056DEB3  |.  68 93DF5600  push    0056DF93  
0056DEB8  |.  64:FF30      push    dword ptr fs:[eax]  
0056DEBB  |.  64:8920      mov     dword ptr fs:[eax], esp  
0056DEBE  |.  837D FC 00   cmp     dword ptr [ebp-4], 0           ; 用户名是否为空？  
0056DEC2  |.  74 04        je      short 0056DEC8  
0056DEC4  |.  85F6        test    esi, esi                      ; ESI 值是否为 0？  
0056DEC6  |.  75 0C        jnz     short 0056DED4  
0056DEC8  |>  8BC7        mov     eax, edi  
0056DECA  |.  E8 2975E9FF  call    004053F8  
0056DECF  |.  E9 A4000000  jmp     0056DF78  
0056DED4  |>  8D45 F8      lea     eax, dword ptr [ebp-8]  
0056DED7  |.  E8 1C75E9FF  call    004053F8  
0056DEDC  |.  8B45 FC      mov     eax, dword ptr [ebp-4]  
0056DEDF  |.  E8 D477E9FF  call    004056B8  
0056DEE4  |.  8BD8        mov     ebx, eax                      ; 用户名长度 len  
0056DEE6  |.  0FAFDE      imul    ebx, esi                      ; ebx 与 esi 相乘，乘积保存在 ebx  
0056DEE9  |.  8B45 FC      mov     eax, dword ptr [ebp-4]         ; 用户名  
0056DEEC  |.  0FB600      movzx   eax, byte ptr [eax]           ; 用户名首位扩展送入 eax  
0056DEEF  |.  69C0 842F0100 imul    eax, eax, 12F84             ; eax=eax * 12F84h  
0056DEF5  |.  03D8        add     ebx, eax                      ; ebx=ebx+eax  
0056DEF7  |.  8D55 F4      lea     edx, dword ptr [ebp-C]  
0056DEFA  |.  8BC3        mov     eax, ebx                      ; eax=ebx  
0056DEFC  |.  E8 7FC9E9FF  call    0040A880                      ; eax 值 16 进制转化为 10 进制字符串，设为 regcode1  
0056DF01  |.  8B55 F4      mov     edx, dword ptr [ebp-C]  
0056DF04  |.  8D45 F8      lea     eax, dword ptr [ebp-8]         ; regcode1  
0056DF07  |.  B9 ACDF5600  mov     ecx, 0056DFAC                 ; -  
0056DF0C  |.  E8 F377E9FF  call    00405704  
0056DF11  |.  8B45 FC      mov     eax, dword ptr [ebp-4]         ; 用户名  
0056DF14  |.  0FB600      movzx   eax, byte ptr [eax]           ; 用户名首位扩展送入 eax  
0056DF17  |.  F7EE        imul    esi                           ; eax=eax*esi  
0056DF19  |.  69D8 C8010000 imul    ebx, eax, 1C8               ; ebx=eax * 1C8h  
0056DF1F  |.  FF75 F8      push    dword ptr [ebp-8]             ; regcode1 入栈  
0056DF22  |.  8D55 F0      lea     edx, dword ptr [ebp-10]  
0056DF25  |.  8BC3        mov     eax, ebx                      ; eax=ebx  
0056DF27  |.  E8 54C9E9FF  call    0040A880                      ; eax 值 16 进制转化为 10 进制字符串，设为 regcode2  
0056DF2C  |.  FF75 F0      push    dword ptr [ebp-10]           ; regcode2 入栈  
0056DF2F  |.  68 ACDF5600  push    0056DFAC                      ; -  
0056DF34  |.  8D45 F8      lea     eax, dword ptr [ebp-8]  
0056DF37  |.  BA 03000000  mov     edx, 3  
0056DF3C  |.  E8 3778E9FF  call    00405778  
0056DF41  |.  8B45 FC      mov     eax, dword ptr [ebp-4]  
0056DF44  |.  E8 6F77E9FF  call    004056B8  
0056DF49  |.  8B55 FC      mov     edx, dword ptr [ebp-4]         ; eax=len,用户名地址传入 edx  
0056DF4C  |.  0FB612      movzx   edx, byte ptr [edx]           ; 用户名首位扩展送入 edx  
0056DF4F  |.  F7EA        imul    edx                           ; eax=eax * edx  
0056DF51  |.  69D8 2E160000 imul    ebx, eax, 162E             ; ebx=eax * 162Eh  
0056DF57  |.  03DE        add     ebx, esi                      ; ebx=ebx+ esi  
0056DF59  |.  8D55 EC      lea     edx, dword ptr [ebp-14]  
0056DF5C  |.  8BC3        mov     eax, ebx                      ; eax=ebx  
0056DF5E  |.  E8 1DC9E9FF  call    0040A880                      ; eax 值 16 进制转化为 10 进制字符串，设为 regcode3  
0056DF63  |.  8B55 EC      mov     edx, dword ptr [ebp-14]      ; regcode3 地址送入 edx  
0056DF66  |.  8D45 F8      lea     eax, dword ptr [ebp-8]  
0056DF69  |.  E8 5277E9FF  call    004056C0                      ; 三个字符串 regcode1,regcode2,regcode3 用"-"相连构成注册码  
0056DF6E  |.  8BC7        mov     eax, edi  
0056DF70  |.  8B55 F8      mov     edx, dword ptr [ebp-8]  
0056DF73  |.  E8 D474E9FF  call    0040544C  
0056DF78  |>  33C0        xor     eax, eax  
0056DF7A  |.  5A            pop     edx  
0056DF7B  |.  59            pop     ecx  
0056DF7C  |.  59            pop     ecx  
0056DF7D  |.  64:8910      mov     dword ptr fs:[eax], edx  
0056DF80  |.  68 9ADF5600  push    0056DF9A  
0056DF85  |>  8D45 EC      lea     eax, dword ptr [ebp-14]  
0056DF88  |.  BA 05000000  mov     edx, 5  
0056DF8D  |.  E8 8A74E9FF  call    0040541C  
0056DF92  \\.  C3            retn  
```



## 二、算法总结

注册码由三段用 `-` 相连组成：

1. 用户名长度 `len` 乘以常数 `0x6B` 然后加上 用户名首位 ASCII 值乘以常数 `0x12F84` 的值；
2. 上面计算值转化为 10 进制字符串即为注册码第一段；
3. 用户名首位 ASCII 值乘以常数 `0x6B`，再乘以常数 `00x1C8`；
4. 上面计算值转化为 10 进制字符串即为注册码第二段；
5. 用户名长度 `len` 乘以 用户名首位 ASCII 值，再乘以常数 `0x162`，三者相乘后加上常数 `0x6B`；
6. 上面计算值转化为 10 进制字符串即为注册码第三段；

## 三、C 语言注册机源代码

```c
#include "stdio.h"
#include "string.h"

void main()
{
    int con=0x2D;    /* 连字符'-' */
    int reg1,reg2,reg3,len;
    char name[20];
    char regcode1[20];
    char regcode2[20];
    char regcode3[20];
    
    printf("%s","请输入用户名：");
    scanf("%s",name);
    
    len=strlen(name);
    
    reg1=len*0x6B+name[0]*0x12F84;
    reg2=name[0]*0x6B*0x1C8;
    reg3=name[0]*len*0x162E+0x6B;
    
    itoa(reg1, regcode1, 10);
    itoa(reg2, regcode2, 10);
    itoa(reg3, regcode3, 10);
    
    printf("%s","注册码是：");
    printf("%s",regcode1);
    printf("%c",con);
    printf("%s",regcode2);
    printf("%c",con);
    printf("%s\n",regcode3);
    
    system("PAUSE");
}
```
