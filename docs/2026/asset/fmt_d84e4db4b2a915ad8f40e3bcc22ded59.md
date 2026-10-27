
### 汇编代码片段

```
0043564C   .  FF15 70724400 call    [<&msvbvm50.\_\_vbaStrI4>]  ;  msvbvm50.\_\_vbaStrI4  51048E的10进制5309582

00435652   .  8BD0          mov     edx, eax

00435654   .  8D4D E0       lea     ecx, [ebp-20]

00435657   .  FFD6          call    esi

00435659   .  8B15 58304400 mov     edx, [443058]                  ；注册码放入Edx

0043565F   .  6A 01         push    1

00435661   .  52            push    edx                           ；Edx入栈

00435662   .  33FF          xor     edi, edi

00435664   .  68 2C954000   push    0040952C                      ;  UNICODE "CJHBYS" 入栈

00435669   .  57            push    edi

0043566A   .  FF15 DC734400 call    [<&msvbvm50.\_\_vbaInStr>] ;  msvbvm50.\_\_vbaInStr 检验是否有"CJHBYS"

00435670   .  83F8 01       cmp     eax, 1                        ；"CJHBYS"出现在第几位开始（19）和1比较

00435673   .  7D 12         jge     short 00435687   ；大于等于1（及注册码中含"CJHBYS"）则跳如下一步，生成In.bat见下面

00435675   .  8D45 D0       lea     eax, [ebp-30]

00435678   .  50            push    eax

00435679   .  E8 92F7FFFF   call    00434E10

0043567E   .  8D4D D0       lea     ecx, [ebp-30]

00435681   .  FF15 78724400 call    [<&msvbvm50.\_\_vbaFreeVar>]    ;  msvbvm50.\_\_vbaFreeVar

00435687   >  8D4D A0       lea     ecx, [ebp-60]

0043568A   .  8D55 D0       lea     edx, [ebp-30]

0043568D   .  51            push    ecx

0043568E   .  52            push    edx

0043568F   .  C745 A8 58304>mov     dword ptr [ebp-58], 00443058

00435696   .  C745 A0 08400>mov     dword ptr [ebp-60], 4008

0043569D   .  E8 9EBF0000   call    00441640

004356A2   .  8B45 E0       mov     eax, [ebp-20]         ；将5309582放入Eax

004356A5   .  8D4D D0       lea     ecx, [ebp-30]

004356A8   .  6A 01         push    1

004356AA   .  8D55 90       lea     edx, [ebp-70]

004356AD   .  8945 98       mov     [ebp-68], eax

004356B0   .  51            push    ecx

004356B1   .  52            push    edx

004356B2   .  8D45 C0       lea     eax, [ebp-40]

004356B5   .  57            push    edi

004356B6   .  50            push    eax

004356B7   .  C745 90 08000>mov     dword ptr [ebp-70], 8

004356BE   .  C745 88 01000>mov     dword ptr [ebp-78], 1

004356C5   .  C745 80 02800>mov     dword ptr [ebp-80], 8002

004356CC   .  FF15 AC734400 call    [<&msvbvm50.\_\_vbaInStrVar>;msvbvm50.\_\_vbaInStrVar注册码换算后检验是否含5309582
```

## 7）推注册码：

`22222222222854720884\_754026158`和`5309582`连接

`222222222228547208847540261585309582`

`x x x x xFUNKSFAUUKSNKAFPDNUNIAXNUF`

`xxxxxFUNKSFAUUKSNKAFPDNUNIAXNUF` 共31位为奇数,所以要头一个尾一个的排下序

`xxxUKFUKNAPNNANFUXIUDFKSUASNFxx` 最后再把要检验的`CJHBYS`插入

`xxxUKFUKNAPNNANFCJHBYSUXIUDFKSUASNFxx`

把注册码拷贝到注册表 `SOFTWARE\softjas\data\Nam`下(程序把注册码放在这个地方),`SmartCheck`运行程序,初始化时校验注册码通过,退出程序,`Unload`时校验注册码也通过,这回就不生成`In.bat`文件了,所以程序也就不被删除了.

## 8）退出程序时代码分析

；以下为程序退出时生成新文件来删除程序 注: 我把<超级函数表达式运算器v20.ece>改为了<1234.exe>方便调试

（因为调试时的中文文件名有时会显示出乱码，所以改了）

```
0043507A   > \8B55 C8       mov     edx, [ebp-38]              ;文件名放倒Edx 本事例为"1234"

0043507D   .  52            push    edx                        ;入堆栈

0043507E   .  68 A4944000   push    004094A4                   ;

00435083   .  FFD7          call    edi                        ;字符串连接后1234.exe放入Eax

00435085   .  8BD0          mov     edx, eax                   ;edx 1234.exe

00435087   .  8D4D CC       lea     ecx, [ebp-34]              ;
```

```
004350A6   .  FF15 F8734400 call    [<&msvbvm50.\_\_vbaStrCopy>]       ;  msvbvm50.\_\_vbaStrCopy

004350AC   .  8B45 E4       mov     eax, [ebp-1C]                     ;in.bat放入Eax

004350AF   .  50            push    eax                                 ；入栈

004350B0   .  6A 01         push    1

004350B2   .  6A FF         push    -1

004350B4   .  6A 02         push    2

004350B6   .  FF15 D0734400 call    [<&msvbvm50.\_\_vbaFileOpen>]      ;  msvbvm50.\_\_vbaFileOpen 打开文件in.bat

004350BC   .  8B1D 8C734400 mov     ebx, [<&msvbvm50.\_\_vbaPrintFile>>;  msvbvm50.\_\_vbaPrintFile

004350C2   .  68 C8944000   push    004094C8

004350C7   .  6A 01         push    1

004350C9   .  68 D8754000   push    004075D8

004350CE   .  FFD3          call    ebx                              ;  <&msvbvm50.\_\_vbaPrintFile>

004350D0   .  8B4D CC       mov     ecx, [ebp-34]

004350D3   .  83C4 0C       add     esp, 0C

004350D6   .  68 DC944000   push    004094DC

004350DB   .  51            push    ecx

004350DC   .  FFD7          call    edi

004350DE   .  8BD0          mov     edx, eax                         ;edx=eax="del 1234.exe"

004350E0   .  8D4D C8       lea     ecx, [ebp-38]                    ;

004350E3   .  FFD6          call    esi                              ;

004350E5   .  50            push    eax

004350E6   .  6A 01         push    1

004350E8   .  68 D8754000   push    004075D8

004350ED   .  FFD3          call    ebx

004350EF   .  83C4 0C       add     esp, 0C

004350F2   .  8D4D C8       lea     ecx, [ebp-38]

004350F5   .  FF15 78744400 call    [<&msvbvm50.\_\_vbaFreeStr>]       ;  msvbvm50.\_\_vbaFreeStr

004350FB   .  8B55 CC       mov     edx, [ebp-34]                     ；1234.exe

004350FE   .  68 EC944000   push    004094EC                         ;  UNICODE "if exist "

00435103   .  52            push    edx                               ；1234.exe

00435104   .  FFD7          call    edi                             ;msvbvm50.\_\_vbaStrCat

00435106   .  8BD0          mov     edx, eax                        ;UNICODE "if exist 1234.exe"

00435108   .  8D4D C8       lea     ecx, [ebp-38]

0043510B   .  FFD6          call    esi                             ;msvbvm50.\_\_vbaStrCat

0043510D   .  50            push    eax

0043510E   .  68 04954000   push    00409504                         ;  UNICODE " GOTO START"

00435113   .  FFD7          call    edi                              ;msvbvm50.\_\_vbaStrCat

00435115   .  8BD0          mov     edx, eax        ;UNICODE "if exist 1234.exe  GOTO START"

00435117   .  8D4D C4       lea     ecx, [ebp-3C]

0043511A   .  FFD6          call    esi

0043511C   .  50            push    eax

0043511D   .  6A 01         push    1

0043511F   .  68 D8754000   push    004075D8

00435124   .  FFD3          call    ebx                              ;msvbvm50.\_\_vbaPrintFile

00435126   .  83C4 0C       add     esp, 0C

00435129   .  8D45 C4       lea     eax, [ebp-3C]

0043512C   .  8D4D C8       lea     ecx, [ebp-38]

0043512F   .  50            push    eax

00435130   .  51            push    ecx

00435131   .  6A 02         push    2

00435133   .  FF15 00744400 call    [<&msvbvm50.\_\_vbaFreeStrList>]   ;  msvbvm50.\_\_vbaFreeStrList

00435139   .  83C4 0C       add     esp, 0C

0043513C   .  68 DC944000   push    004094DC

00435141   .  68 B4944000   push    004094B4

00435146   .  FFD7          call    edi

00435148   .  8BD0          mov     edx, eax                           ;(UNICODE "del in.bat")

0043514A   .  8D4D C8       lea     ecx, [ebp-38]

0043514D   .  FFD6          call    esi

0043514F   .  50            push    eax

00435150   .  6A 01         push    1

00435152   .  68 D8754000   push    004075D8

00435157   .  FFD3          call    ebx                               ;msvbvm50.\_\_vbaPrintFile

00435159   .  83C4 0C       add     esp, 0C

0043515C   .  8D4D C8       lea     ecx, [ebp-38]

0043515F   .  FF15 78744400 call    [<&msvbvm50.\_\_vbaFreeStr>]       ;

00435165   .  6A 01         push    1

00435167   .  FF15 20734400 call    [<&msvbvm50.\_\_vbaFileClose>]     ;  msvbvm50.\_\_vbaFileClose

0043516D   .  8D55 E4       lea     edx, [ebp-1C]

00435170   .  8D45 80       lea     eax, [ebp-80]

00435173   .  6A 00         push    0

00435175   .  50            push    eax
```
