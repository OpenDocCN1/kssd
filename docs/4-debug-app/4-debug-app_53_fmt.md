
# 数字公司卫士插件检测中一个文件解密过程

> 原文：[`www.pediy.com/kssd/pediy11/124993.html`](https://www.pediy.com/kssd/pediy11/124993.html)

菜鸟第一次发帖，欢迎大家拍砖。

能力有限，分析得不好，而且走了很多弯路![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

根据 360 运行插件检测的过程中，有各自检测某些插件的过程，故猜想在检测过程中，插件检测是在调用特征库中的数据，所以接下来应该是找到调用的哪个 `dat` 文件，该特征库文件通过某段解密代码解密之后得到特征库的真正数据，然后进行检测。如果需要调用某个数据文件，必然用到 `OpenFile` 函数，所以用 IDA 载入该动态链接库，查看输入表，找到是否有 `OpenFile` 函数（确实调用过）。接下来用 OD 附加 360 主程序，然后 F9 运行，进入到 360 主界面。Ctrl+G，输入 `OpenFile`，Enter。在主界面中点击插件检测，程序断在了 `OpenFile` 函数的入口处，从寄存器中看到读取的文件为 `LibAntiSpywa.dat` 该数据文件，故这个数据文件就是存特征库的文件。然后 Alt+F9 返回程序领空，程序领空正好是在 `AntiAdwa.dll` 这个动态链接库中。至此，说明之前猜测的插件检测的插件是该动态链接库是正确的，也说明之前猜测的调用某个特征库文件也是正确的，而且找到了该文件。上述已经到 `OpenFile()` 函数处了，那么对于文件的读取函数应该不远，再断一个 `ReadFile()`，回到程序领空。再跟踪几步，到以下代码处

```
03CB5369  /$  55            push    ebp

03CB536A  |.  8BEC          mov     ebp, esp

03CB536C  |.  83EC 10       sub     esp, 10

03CB536F  |.  57            push    edi

03CB5370  |.  33FF          xor     edi, edi

03CB5372  |.  57            push    edi           ; /hTemplateFile => NULL

03CB5373  |.  57            push    edi             ; |Attributes => 0

03CB5374  |.  6A 03         push    3               ; |Mode = OPEN_EXISTING

03CB5376  |.  57            push    edi             ; |pSecurity => NULL

03CB5377  |.  6A 01         push    1       ; |ShareMode = FILE_SHARE_READ

03CB5379  |.  68 00000080   push    80000000       ; |Access = GENERIC_READ

03CB537E  |.  FF75 08       push    dword ptr [ebp+8]             ; |FileName

03CB5381  |.  894D F8       mov     dword ptr [ebp-8], ecx       ; |

03CB5384  |.  897D F0       mov     dword ptr [ebp-10], edi      ; |

03CB5387  |.  FF15 5092D403 call    dword ptr [<&KERNEL32.CreateFile>; \CreateFileA

03CB538D  |.  83F8 FF       cmp     eax, -1

03CB5390  |.  8945 F4       mov     dword ptr [ebp-C], eax

03CB5393  |.  75 07         jnz     short 03CB539C

03CB5395  |.  33C0          xor     eax, eax

03CB5397  |.  E9 C7000000   jmp     03CB5463

03CB539C  |>  53            push    ebx

03CB539D  |.  56            push    esi

03CB539E  |.  57            push    edi             ; /pFileSizeHigh

03CB539F  |.  50            push    eax             ; |hFile

03CB53A0  |.  FF15 5492D403 call    dword ptr [<&KERNEL32.GetFileSiz>; \GetFileSize

03CB53A6  |.  8BD8          mov     ebx, eax

03CB53A8  |.  8D43 01       lea     eax, dword ptr [ebx+1]

03CB53AB  |.  50            push    eax             ; /size

03CB53AC  |.  FF15 4093D403 call    dword ptr [<&MSVCRT.malloc>]     ; \malloc

03CB53B2  |.  8BF0          mov     esi, eax

03CB53B4  |.  59            pop     ecx

03CB53B5  |.  3BF7          cmp     esi, edi

03CB53B7  |.  0F84 98000000 je      03CB5455

03CB53BD  |.  8D45 FC       lea     eax, dword ptr [ebp-4]

03CB53C0  |.  57            push    edi             ; /pOverlapped

03CB53C1  |.  50            push    eax         ; |pBytesRead

03CB53C2  |.  53            push    ebx         ; |BytesToRead

03CB53C3  |.  56            push    esi                 ; |Buffer

03CB53C4  |.  897D FC       mov     dword ptr [ebp-4], edi       ; |

03CB53C7  |.  FF75 F4       push    dword ptr [ebp-C]            ; |hFile

03CB53CA  |.  FF15 5892D403 call    dword ptr [<&KERNEL32.ReadFile>] ; \ReadFile 函数，真正的读取文件

03CB53D0  |.  85C0          test    eax, eax

03CB53D2  |.  74 79         je      short 03CB544D

03CB53D4  |.  8B46 30       mov     eax, dword ptr [esi+30]    ;dat 文件偏移 30 处

03CB53D7  |.  69C0 98010000 imul    eax, eax, 198

03CB53DD  |.  83C0 34       add     eax, 34            ; dat 文件大小，总字节数

03CB53E0  |.  3945 FC       cmp     dword ptr [ebp-4], eax

03CB53E3  |.  75 68         jnz     short 03CB544D

03CB53E5  |.  8B45 0C       mov     eax, dword ptr [ebp+C]

03CB53E8  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]

03CB53EB  |.  C745 F0 01000>mov     dword ptr [ebp-10], 1

03CB53F2  |.  897D 08       mov     dword ptr [ebp+8], edi

03CB53F5  |.  897C81 18     mov     dword ptr [ecx+eax*4+18], edi

03CB53F9  |.  397E 30       cmp     dword ptr [esi+30], edi

03CB53FC  |.  76 46         jbe     short 03CB5444

03CB53FE  |.  8D9E C0000000 lea     ebx, dword ptr [esi+C0]

03CB5404  |.  BF 80000000   mov     edi, 80          ; edi=80 每次读取 80 个字节

03CB5409  |>  57            /push    edi             ; 80

03CB540A  |.  8D43 80       |lea     eax, dword ptr [ebx-80]        ;每次将 80 字节读入之后 eax 到该 80 字节的开头处

03CB540D  |.  FF76 0C       |push    dword ptr [esi+C]           ;  3

03CB5410  |.  50            |push    eax             ;字符串

03CB5411  |.  E8 29F7FFFF   |call    03CB4B3F           ;  解密函数

03CB5416  |.  57            |push    edi             ;  设置该组下一 80 字节

03CB5417  |.  FF76 0C       |push    dword ptr [esi+C]           ;  3

03CB541A  |.  53            |push    ebx             ;字符串

03CB541B  |.  E8 1FF7FFFF   |call    03CB4B3F        ;第二组解密

03CB5420  |.  57            |push    edi             ; 80

03CB5421  |.  8D83 80000000 |lea     eax, dword ptr [ebx+80]        ;  ebx 之后 80 位

03CB5427  |.  FF76 0C       |push    dword ptr [esi+C]

03CB542A  |.  50            |push    eax

03CB542B  |.  E8 0FF7FFFF   |call    03CB4B3F          ;第三组解密

03CB5430  |.  83C4 24       |add     esp, 24

03CB5433  |.  FF45 08       |inc     dword ptr [ebp+8]       ;上一组解密完成，组计数器自加

03CB5436  |.  8B45 08       |mov     eax, dword ptr [ebp+8]       ;

03CB5439  |.  81C3 98010000 |add     ebx, 198         ; 设置下一组需解密的 198 位

03CB543F  |.  3B46 30       |cmp     eax, dword ptr [esi+30]   ;组计数器与文件组大小比较

03CB5442  |.^ 72 C5         \jb      short 03CB5409      ;小于文件大小则继续
.
```

## 解密函数

```
03CB4B3F  /$  55            push    ebp

03CB4B40  |.  8BEC          mov     ebp, esp

03CB4B42  |.  837D 0C 01    cmp     dword ptr [ebp+C], 1

03CB4B46  |.  75 09         jnz     short 03CB4B51

03CB4B48  |.  FF75 08       push    dword ptr [ebp+8]

03CB4B4B  |.  E8 40FFFFFF   call    03CB4A90           ;  若计数器为 1 调用该函数

03CB4B50  |.  59            pop     ecx

03CB4B51  |>  837D 0C 03    cmp     dword ptr [ebp+C], 3           ;  计数器位为 3 调用这段

03CB4B55  |.  75 0D         jnz     short 03CB4B64

03CB4B57  |.  FF75 10       push    dword ptr [ebp+10]            ;  80

03CB4B5A  |.  FF75 08       push    dword ptr [ebp+8]         ;  80 位字符串

03CB4B5D  |.  E8 40FFFFFF   call    03CB4AA2

03CB4B62  |.  59            pop     ecx

03CB4B63  |.  59            pop     ecx

03CB4B64  |>  5D            pop     ebp

03CB4B65  \\.  C3            retn
```

## 跳转调用真正的解密函数

```
03CB4AA2  /$  B8 98A2D303   mov     eax, 03D3A298

03CB4AA7  |.  E8 54410800   call    03D38C00

03CB4AAC  |.  83EC 10       sub     esp, 10

03CB4AAF  |.  53            push    ebx

03CB4AB0  |.  8B5D 08       mov     ebx, dword ptr [ebp+8]       ;  ebx 为前 80 字符串

03CB4AB3  |.  57            push    edi

03CB4AB4  |.  33FF          xor     edi, edi             ;  edi=0

03CB4AB6  |.  3BDF          cmp     ebx, edi

03CB4AB8  |.  74 77         je      short 03CB4B31

03CB4ABA  |.  397D 0C       cmp     dword ptr [ebp+C], edi       ;  ebp+c 为 80

03CB4ABD  |.  74 72         je      short 03CB4B31

03CB4ABF  |.  8A45 0F       mov     al, byte ptr [ebp+F]

03CB4AC2  |.  56            push    esi

03CB4AC3  |.  57            push    edi

03CB4AC4  |.  8D4D E4       lea     ecx, dword ptr [ebp-1C]

03CB4AC7  |.  8845 E4       mov     byte ptr [ebp-1C], al

03CB4ACA  |.  E8 B0610000   call    03CBAC7F

03CB4ACF  |.  BE 5059D603   mov     esi, 03D65950
```



`"A56667511DFF47dbA8146AD6EBF00A45"`

```
03CB4AD4   |.   56             push    esi                      

 ; /s => "A56667511DFF47dbA8146AD6EBF00A45"  

03CB4AD5   |.   E8 A6410800    call    <jmp.&MSVCRT.strlen>      ; \strlen  

03CB4ADA   |.   59             pop     ecx  

03CB4ADB   |.   50             push    eax                   ;  字符串长度 20  

03CB4ADC   |.   56             push    esi                     ;  字符串  

03CB4ADD   |.   8D4D E4        lea     ecx, dword ptr [ebp-1C]       

03CB4AE0   |.   E8 EC620000    call    03CBADD1              ;将上述字符串拷贝  

03CB4AE5   |.   33F6           xor     esi, esi  

03CB4AE7   |.   397D 0C        cmp     dword ptr [ebp+C], edi  

03CB4AEA   |.   897D FC        mov     dword ptr [ebp-4], edi  

03CB4AED   |.   76 33          jbe     short 03CB4B22  

03CB4AEF   |>   8BC6           /mov    eax, esi              ;字符串  

03CB4AF1   |.   33D2           |xor    edx, edx  

03CB4AF3   |.   F775 EC        |div    dword ptr [ebp-14]     ;计数器模 20  

03CB4AF6   |.   8BFA           |mov    edi, edx               ;  

03CB4AF8   |.   397D EC        |cmp    dword ptr [ebp-14], edi      ;20 与计数器比较  

03CB4AFB   |.   72 15          |jb     short 03CB4B12         :小于则跳转  

03CB4AFD   |.   837D E8 00     |cmp    dword ptr [ebp-18], 0      ;明码字符串 A56667511DFF47dbA8146AD6EBF00A45  

03CB4B01   |.   74 0F          |je     short 03CB4B12       ;等于则跳转  

03CB4B03   |.   8D4D E4        |lea    ecx, dword ptr [ebp-1C]    

03CB4B06   |.   E8 48610000    |call   03CBAC53              ;校验函数  

03CB4B0B   |.   8B45 E8        |mov    eax, dword ptr [ebp-18]    ;明码字符串  

03CB4B0E   |.   03C7           |add    eax, edi              ;上字符串的第 n 个字符   n=edi  

03CB4B10   |.   EB 05          |jmp    short 03CB4B17       ;跳转至下下一句  

03CB4B12   |>   B8 BC96D403    |mov    eax, 03D496BC  

03CB4B17   |>   8A00           |mov    al, byte ptr [eax]    ;当前的字符，也就是第 n 个字符  

03CB4B19   |.   30041E         |xor    byte ptr [esi+ebx], al      ;用上字符与需解密的字符进行异或操作  

03CB4B1C   |.   46             |inc    esi                   ;计数器自加  

03CB4B1D   |.   3B75 0C        |cmp    esi, dword ptr [ebp+C]       ;计数器与 80 比较  

03CB4B20   |.^ 72 CD          \jb     short 03CB4AEF      ;小于 80 跳转继续解密  
```

………………………..

如果该组解密完成之后，释放内存和重置寄存器等，开始下一组的解密

……………………….
