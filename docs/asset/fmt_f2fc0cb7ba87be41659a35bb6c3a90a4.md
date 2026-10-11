
```
0058CE23  |>  8D4D E4      /lea     ecx, [ebp-1C]  
0058CE26  |.  8BC3         |mov     eax, ebx  
0058CE28  |.  C1E0 02      |shl     eax, 2  
0058CE2B  |.  33D2         |xor     edx, edx  
0058CE2D  |.  E8 7ECFE7FF  |call    00409DB0  
0058CE32  |.  8B55 E4      |mov     edx, [ebp-1C]  
0058CE35  |.  8D45 F8      |lea     eax, [ebp-8]  
0058CE38  |.  E8 2F7DE7FF  |call    00404B6C  
0058CE3D  |.  43           |inc     ebx  
0058CE3E  |.  83FB 04      |cmp     ebx, 4  
0058CE41  |.^ 75 E0        \\jnz    short 0058CE23  
0058CE43  |>  8B45 F4      mov     eax, [ebp-C]                  ;  第 5～8 位 (ASCII "3593")  
0058CE46  |.  E8 197DE7FF  call    00404B64  
0058CE4B  |.  83F8 04      cmp     eax, 4  
0058CE4E  |.  7D 2F        jge     short 0058CE7F  
0058CE50  |.  8B45 F4      mov     eax, [ebp-C]  
0058CE53  |.  E8 0C7DE7FF  call    00404B64  
0058CE58  |.  8BD8         mov     ebx, eax  
0058CE5A  |.  83FB 03      cmp     ebx, 3  
0058CE5D  |.  7F 20        jg      short 0058CE7F  
0058CE5F  |>  8D4D E0      /lea    ecx, [ebp-20]  
0058CE62  |.  8BC3         |mov     eax, ebx  
0058CE64  |.  C1E0 02      |shl     eax, 2  
0058CE67  |.  33D2         |xor     edx, edx  
0058CE69  |.  E8 42CFE7FF  |call    00409DB0  
0058CE6E  |.  8B55 E0      |mov     edx, [ebp-20]  
0058CE71  |.  8D45 F4      |lea     eax, [ebp-C]  
0058CE74  |.  E8 F37CE7FF  |call    00404B6C  
0058CE79  |.  43           |inc     ebx  
0058CE7A  |.  83FB 04      |cmp     ebx, 4  
0058CE7D  |.^ 75 E0        \\jnz    short 0058CE5F  
0058CE7F  |>  8D45 F0      lea     eax, [ebp-10]  
0058CE82  |.  BA 0CCF5800  mov     edx, 0058CF0C                 ;  常量 (ASCII "MPW456ei878")  
0058CE87  |.  E8 B07AE7FF  call    0040493C  
0058CE8C  |.  8D45 DC      lea     eax, [ebp-24]  
0058CE8F  |.  50           push    eax  
0058CE90  |.  B9 04000000  mov     ecx, 4                        ;  长度 4  
0058CE95  |.  BA 01000000  mov     edx, 1                        ;  从第一位开始  
0058CE9A  |.  8B45 F0      mov     eax, [ebp-10]  
0058CE9D  |.  E8 227FE7FF  call    00404DC4                      ;  取常量的前 4 位  
0058CEA2  |.  FF75 DC      push    dword ptr [ebp-24]            ;  (ASCII "MPW4")  
0058CEA5  |.  68 20CF5800  push    0058CF20                      ;  -  
0058CEAA  |.  FF75 F8      push    dword ptr [ebp-8]             ;  (ASCII "1453")  
0058CEAD  |.  8D45 D8      lea     eax, [ebp-28]  
0058CEB0  |.  50           push    eax  
0058CEB1  |.  B9 05000000  mov     ecx, 5                        ;  长度 5  
0058CEB6  |.  BA 05000000  mov     edx, 5                        ;  从第五位开始  
0058CEBB  |.  8B45 F0      mov     eax, [ebp-10]  
0058CEBE  |.  E8 017FE7FF  call    00404DC4  
0058CEC3  |.  FF75 D8      push    dword ptr [ebp-28]            ;  第 5～9 位 (ASCII "56ei8")  
0058CEC6  |.  68 20CF5800  push    0058CF20                      ;  -  
0058CECB  |.  FF75 F4      push    dword ptr [ebp-C]             ;  (ASCII "3593")  
0058CECE  |.  8BC7         mov     eax, edi  
0058CED0  |.  BA 06000000  mov     edx, 6  
0058CED5  |.  E8 4A7DE7FF  call    00404C24  
0058CEDA  |.  33C0         xor     eax, eax  
0058CEDC  |.  5A           pop     edx  
0058CEDD  |.  59           pop     ecx  
0058CEDE  |.  59           pop     ecx  
0058CEDF  |.  64:8910      mov     fs:[eax], edx  
0058CEE2  |.  68 FCCE5800  push    0058CEFC  
0058CEE7  |>  8D45 D8      lea     eax, [ebp-28]  
0058CEEA  |.  BA 0A000000  mov     edx, 0A  
0058CEEF  |.  E8 D479E7FF  call    004048C8  
0058CEF4  \\.  C3          retn  
0058CEF5   .^ E9 CE72E7FF  jmp     004041C8  
0058CEFA   .^ EB EB        jmp     short 0058CEE7  
0058CEFC   .  5F          pop     edi  
0058CEFD   .  5E          pop     esi  
0058CEFE   .  5B          pop     ebx  
0058CEFF   .  8BE5        mov     esp, ebp  
0058CF01   .  5D          pop     ebp  
0058CF02   .  C3          retn  
```

------------------------------------------------------------------------

【破解总结】明码比较的，算法也简单，还是分析下。

1. 令机器码为`A`；
2. 将字符串`A`转为16进制，令其为`B`；
3. 将`B`翻转后为`C`；
4. 取`C`的前4位为`C1`、第5～8位为`C2`；
5. 取常量`D`（即`MPW456ei878`）的前4位为`D1`、第5～9位为`D2`；
6. 注册码`K`=`D1-C1+D2-C2`（这里的`-`为分隔符，`+`为连接符），如我这里的是`MPW4-145356ei8-3593`
7. 注册码`K8`也可是这样的，`K8=K-SQL`（这里的`-`为分隔符），如`MPW4-145356ei8-3593-SQL`

注册信息保存处：

`[HKEY_LOCAL_MACHINE\\SOFTWARE\\vt\\mpw]`

`"Date"=hex:00,00,00,00,e0,23,e3,40`

`"Name"="PF2B27K2119S5A"`

`"Pass"="MPW4-145356ei8-3593-SQL"`

------------------------------------------------------------------------

【版权声明】本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
