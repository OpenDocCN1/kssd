
00407C6D  |.  2B05 98824000  sub     eax, dword ptr [408298]  

00407C73  |.  A3 9C824000   mov     dword ptr [40829C], eax  

两个浮点数进行相加  

      tReal := 12.2;  

      bReal := tReal + 0.5;  

      tReal := tReal + bReal;  

00407C68  |.  C705 9C974000>mov     dword ptr [40979C], 66666666  

00407C72  |.  C705 A0974000>mov     dword ptr [4097A0], 40286666  

00407C7C  |.  DD05 9C974000 fld     qword ptr [40979C]  

00407C82  |.  D805 B07C4000 fadd    dword ptr [407CB0]  

00407C88  |.  DD1D A4974000 fstp    qword ptr [4097A4]  

00407C8E  |.  9B            wait  

00407C8F  |.  DD05 9C974000 fld     qword ptr [40979C]  

00407C95  |.  DC05 A4974000 fadd    qword ptr [4097A4]  

00407C9B  |.  DD1D 9C974000 fstp    qword ptr [40979C]  

00407CA1  |.  9B            wait  

两个浮点进行相除  

00407C68  |.  DB05 94824000 fild    dword ptr [408294]  

00407C6E  |.  DB05 98824000 fild    dword ptr [408298]  

00407C74  |.  DEF9          fdivp   st(1), st  

00407C76  |.  DD1D 9C974000 fstp    qword ptr [40979C]  

00407C7C  |.  9B            wait  

两个浮点进行整除 `tReal := Size div pSize;` 取整。  

00407C69  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C6E  |.  99            cdq  

00407C6F  |.  F73D 98824000 idiv    dword ptr [408298]  

00407C75  |.  890424        mov     dword ptr [esp], eax //eax 存储商  

00407C78  |.  DB0424        fild    dword ptr [esp]         //装入整数  

00407C7B  |.  DD1D 9C974000 fstp    qword ptr [40979C]  

00407C81  |.  9B            wait  

两个浮点进行取余  

00407C69  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C6E  |.  99            cdq  

00407C6F  |.  F73D 98824000 idiv    dword ptr [408298]  

00407C75  |.  891424        mov     dword ptr [esp], edx // edx 存储余数  

00407C78  |.  DB0424        fild    dword ptr [esp]  

00407C7B  |.  DD1D 9C974000 fstp    qword ptr [40979C]  

00407C81  |.  9B            wait  

对一个整数进行取反  

00407C68  |.  F71D 94824000 neg     dword ptr [408294]   

对两个 `Boolean` 进行逻辑与 `and`  

00407C68  |.  803D 94824000>cmp     byte ptr [408294], 0  

00407C6F  |.  74 09          je      short 00407C7A  

00407C71  |.  803D 98824000>cmp     byte ptr [408298], 0  

00407C78  |.  75 04          jnz     short 00407C7E  

00407C7A  |>  33C0          xor     eax, eax  

00407C7C  |.  EB 02          jmp     short 00407C80  

00407C7E  |>  B0 01         mov     al, 1  

00407C80  |>  A2 94824000   mov     byte ptr [408294], al  

为两 `Boolean` 赋真值和假值  

00407C68  |.  C605 94824000>mov     byte ptr [408294], 1 , 真  

00407C6F  |.  C605 98824000>mov     byte ptr [408298], 0 , 假  

进行逻辑与后再存储在 `Boolean` 变量中  

00407C68  |.  C605 94824000>mov     byte ptr [408294], 1  

00407C6F  |.  C605 98824000>mov     byte ptr [408298], 0  

00407C76  |.  803D 94824000>cmp     byte ptr [408294], 0  

00407C7D  |.  74 09          je      short 00407C88  

00407C7F  |.  803D 98824000>cmp     byte ptr [408298], 0  

00407C86  |.  75 04          jnz     short 00407C8C  

00407C88  |>  33C0          xor     eax, eax  

00407C8A  |.  EB 02          jmp     short 00407C8E  

00407C8C  |>  B0 01         mov     al, 1  

00407C8E  |>  A2 9C974000   mov     byte ptr [40979C], al  

进行逻辑或  

00407C68  |.  C605 94824000>mov     byte ptr [408294], 1  

00407C6F  |.  C605 98824000>mov     byte ptr [408298], 0  

00407C76  |.  803D 94824000>cmp     byte ptr [408294], 0  

00407C7D  |.  75 0D          jnz     short 00407C8C  

00407C7F  |.  803D 98824000>cmp     byte ptr [408298], 0  

00407C86  |.  75 04          jnz     short 00407C8C  

00407C88  |.  33C0          xor     eax, eax  

00407C8A  |.  EB 02          jmp     short 00407C8E  

00407C8C  |>  B0 01         mov     al, 1  

00407C8E  |>  A2 9C974000   mov     byte ptr [40979C], al  

进行逻辑异或  

00407C68  |.  C605 94824000>mov     byte ptr [408294], 1  

00407C6F  |.  C605 98824000>mov     byte ptr [408298], 0  

00407C76  |.  A0 94824000   mov     al, byte ptr [408294]  只对是真的进行处理,另外逻辑运算都进行转换到 `Byte` - 8bit  

00407C7B  |.  3205 98824000 xor     al, byte ptr [408298]  

00407C81  |.  A2 9C974000   mov     byte ptr [40979C], al  

对整数则为位运算，这容易让人混淆。`Size := not Size;`  

00407C68  |.  F715 94824000 not     dword ptr [408294]  

对整数进行位运算与  

00407C68  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C6D  |.  2305 98824000 and     eax, dword ptr [408298]  

00407C73  |.  A3 9C824000   mov     dword ptr [40829C], eax  

00407C78  |.  89C0          mov     eax, eax  

对整数进行位运算或  

00407C68  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C6D  |.  0B05 98824000 or      eax, dword ptr [408298]  //整数的比 `Boolean` 的容易看，  

                   //使用与或非 异或指令很明显。  

00407C73  |.  A3 9C824000   mov     dword ptr [40829C], eax  

对整数进行位运算异或  

00407C68  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C6D  |.  3305 98824000 xor     eax, dword ptr [408298]  

00407C73  |.  A3 9C824000   mov     dword ptr [40829C], eax  

对整数进行位运算左移位 `shl`  

00407C68  |.  8B0D 98824000 mov     ecx, dword ptr [408298]  

00407C6E  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C73  |.  D3E0          shl     eax, cl  

00407C75  |.  A3 9C824000   mov     dword ptr [40829C], eax  

对整数进行位运算右移位 `shr`  

00407C68  |.  8B0D 98824000 mov     ecx, dword ptr [408298]  

00407C6E  |.  A1 94824000   mov     eax, dword ptr [408294]  

00407C73  |.  D3E8          shr     eax, cl  

00407C75  |.  A3 9C824000   mov     dword ptr [40829C], eax  

进行条件判断 `=` 时  

[源]  

if `Size` = 100 then  
        `Writeln`('Hello World');  
004080F4  |.  833D 98924000>cmp     dword ptr [409298], 64  
004080FB  |.  75 19          jnz     short 00408116  
004080FD  |.  A1 08934000   mov     eax, dword ptr [409308]  
00408102  |.  BA 2C814000   mov     edx, 0040812C                   ;  ASCII "Hello World"  

关系运算符 `<>` 时  

if `Size` <> 100 then  
      begin  
        `Writeln`('Hello World');  
      end;  
004080F4  |.  833D 98924000>cmp     dword ptr [409298], 64  
004080FB  |.  74 19          je      short 00408116        (ZF = 1)时  
004080FD  |.  A1 08934000   mov     eax, dword ptr [409308]  
00408102  |.  BA 2C814000   mov     edx, 0040812C                   ;  ASCII "Hello World"  
00408107  |.  E8 48BDFFFF   call    00403E54  
0040810C  |.  E8 07ABFFFF   call    00402C18  
00408111  |.  E8 16A5FFFF   call    0040262C  

关系运算 `<` 时  

if `Size` < 100 then  
      begin  
        `Writeln`('Hello World');  
      end;  
004080F4  |.  833D 98924000>cmp     dword ptr [409298], 64  
004080FB  |.  7D 19          jge     short 00408116  
004080FD  |.  A1 08934000   mov     eax, dword ptr [409308]  
00408102  |.  BA 2C814000   mov     edx, 0040812C                   ;  ASCII "Hello World"  
00408107  |.  E8 48BDFFFF   call    00403E54  
0040810C  |.  E8 07ABFFFF   call    00402C18  
00408111  |.  E8 16A5FFFF   call    0040262C  

关系运算 `>` 时  

if `Size` > 100 then  
      begin  
        `Writeln`('Hello World');  
      end;  

004080F4  |.  833D 98924000>cmp     dword ptr [409298], 64  
004080FB  |.  7E 19          jle     short 00408116  
004080FD  |.  A1 08934000   mov     eax, dword ptr [409308]  
00408102  |.  BA 2C814000   mov     edx, 0040812C                   ;  ASCII "Hello World"  
00408107  |.  E8 48BDFFFF   call    00403E54  
0040810C  |.  E8 07ABFFFF   call    00402C18  
00408111  |.  E8 16A5FFFF   call    0040262C  

关系运算 `<=` 时  

004080F4  |.  833D 98924000>cmp     dword ptr [409298], 64
