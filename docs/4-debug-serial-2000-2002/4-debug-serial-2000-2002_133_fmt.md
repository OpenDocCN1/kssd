
# WinImp 1.11

> 原文：[www.pediy.com/kssd/pediy02/forum140.htm](https://www.pediy.com/kssd/pediy02/forum140.htm)

`http://newhua.xingtai.net/down/winimp111-32.exe`

## WinImp 1.11 注册码破解

`WINIMP` 是一个压缩软件，同等条件下其压缩率远高于 `WINZIP`，是我的新爱。

因为在最新的番外地 3.6 注册码查询系统中 `winimp` 只有 0.99 版的，不能用，所以就自己破了。

在 `HELP` 中可以输入注册码。

首先随便输入一个号码，得到告示“the keys do not match the names...”，再用 `W32DASM` 反汇编，从 `STRING DATA REFERENCE` 中找到：

```
:004260B5 81FA00000001   cmp edx, 01000000
:004260BB 7216           jb 004260D3
:004260BD 3D00000001     cmp eax, 01000000
:004260C2 720F           jb 004260D3
:004260C4 89D0           mov eax, edx
:004260C6 8B55F8         mov edx, dword ptr [ebp-08]
:004260C9 E8D9010000     call 004262A7
:004260CE 3B45FC         cmp eax, dword ptr [ebp-04]
:004260D1 7418           je 004260EB

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004260BB(C), :004260C2(C)

:004260D3 6A30           push 00000030
* Possible StringData Ref from Data Obj ->"WinImp"
:004260D5 6830D24400     push 0044D230
* Possible StringData Ref from Data Obj ->"The keys do not match the name. "
                                  ->"Please check your registration "
                                  ->"details and try again."
:004260DA 6828CC4400     push 0044CC28
:004260DF 56             push esi
* Reference To: USER32.MessageBoxA, Ord:0048h
:004260E0 2EFF150CD84300 Call dword ptr cs:[0043D80C]
:004260E7 31C0           xor eax, eax
:004260E9 EB68           jmp 00426153
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004260D1(C)
:004260EB BB603C4500     mov ebx, 00453C60
```

向上看，

1. `:004260B5   cmp edx, 01000000` `<--edx 为 key1，必须大于等于 01000000`
   `:004260BB   jb 004260D3`

2. `:004260BD   cmp eax, 01000000` `<--eax 为 key2，也必须大于等于 01000000`
   `:004260C2   jb 004260D3`

3. `:004260CE   cmp eax, dword ptr [ebp-04]` `<--相等就注册成功`
   `:004260D1   je 004260EB`

所以要追入 `:004260C9   call 004262A7`。

遗憾的是，其中的计算很烦，不能搞懂，但是我发现在 `:004260CE   cmp eax, dword ptr [ebp-04]`，只要用户名，`key1` 固定，`key2` 值的变化只是影响到 `eax` 的变化，且各个数位之间是对应的，所以可以用断点 `bpx 004260CE`，通过改变 `key2` 值来猜。

4. **结果**
   - `name`: `xixiaolou [CCG]`
   - `key1`: `10000000`
   - `key2`: `3e64a67e`

没有“Thank you”之类的东西。

### 注册机

```c
#include <stdio.h>

unsigned char Table[256] =
{
0x00,0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08,0x09,0x0A,0x0B,0x0C,0x0D,0x0E,0x0F,
0x10,0x11,0x12,0x13,0x14,0x15,0x16,0x17,0x18,0x19,0x1A,0x1B,0x1C,0x1D,0x1E,0x1F,
0x20,0x21,0x22,0x23,0x24,0x25,0x26,0x27,0x28,0x29,0x2A,0x2B,0x2C,0x2D,0x2E,0x2F,
0x30,0x31,0x32,0x33,0x34,0x35,0x36,0x37,0x38,0x39,0x3A,0x3B,0x3C,0x3D,0x3E,0x3F,
0x40,0x61,0x62,0x63,0x64,0x65,0x66,0x67,0x68,0x69,0x6A,0x6B,0x6C,0x6D,0x6E,0x6F,
0x70,0x71,0x72,0x73,0x74,0x75,0x76,0x77,0x78,0x79,0x7A,0x5B,0x5C,0x5D,0x5E,0x5F,
0x60,0x61,0x62,0x63,0x64,0x65,0x66,0x67,0x68,0x69,0x6A,0x6B,0x6C,0x6D,0x6E,0x6F,
0x70,0x71,0x72,0x73,0x74,0x75,0x76,0x77,0x78,0x79,0x7A,0x7B,0x7C,0x7D,0x7E,0x7F,
0x80,0x81,0x82,0x83,0x84,0x85,0x86,0x87,0x88,0x89,0x8A,0x8B,0x8C,0x8D,0x8E,0x8F,
0x90,0x91,0x92,0x93,0x94,0x95,0x96,0x97,0x98,0x99,0x9A,0x9B,0x9C,0x9D,0x9E,0x9F,
0xA0,0xA1,0xA2,0xA3,0xA4,0xA5,0xA6,0xA7,0xA8,0xA9,0xAA,0xAB,0xAC,0xAD,0xAE,0xAF,
0xB0,0xB1,0xB2,0xB3,0xB4,0xB5,0xB6,0xB7,0xB8,0xB9,0xBA,0xBB,0xBC,0xBD,0xBE,0xBF,
0xC0,0xC1,0xC2,0xC3,0xC4,0xC5,0xC6,0xC7,0xC8,0xC9,0xCA,0xCB,0xCC,0xCD,0xCE,0xCF,
0xD0,0xD1,0xD2,0xD3,0xD4,0xD5,0xD6,0xD7,0xD8,0xD9,0xDA,0xDB,0xDC,0xDD,0xDE,0xDF,
0xE0,0xE1,0xE2,0xE3,0xE4,0xE5,0xE6,0xE7,0xE8,0xE9,0xEA,0xEB,0xEC,0xED,0xEE,0xEF,
0xF0,0xF1,0xF2,0xF3,0xF4,0xF5,0xF6,0xF7,0xF8,0xF9,0xFA,0xFB,0xFC,0xFD,0xFE,0xFF
};

unsigned long key1, key2;

char sum[4];

void main(void)
{
    char Name[64];
    int k, index;
    printf("Input your name: ");
    gets(Name);
    do
    {
        printf("Input HEX key1(must no less than 0x01000000): ");
        scanf("%lX", &key1);
    } while(key1 < 0x01000000);
    for(k = 0; k < 4; k++)
    {
        sum[k] = 0;
    }
    k = 0;
    index = 0;
    while(Name[k])
    {
        if ((Name[k] == 0x20) || (Name[k] == 0x2E))
        {
            k++;
            continue;
        }
        index &= 3;
        sum[index] +=  Table[Name[k] & 0xFF];
        k++;
        index++;
    }
    _asm
    {
        pushad
        MOV       ECX, [key1]
        MOV       ebp, 0x00000002
        MOV       EDI,[key1]
        JMP       _00426259
    _00426250:
        SHL       ebp, 1
        CMP       ebp, 0
        JZ        _0042629F
    _00426259:
        MOV       ESI,0xF527789F
        MOV       EAX,EDI
        MOV       EDX,EDI
        MUL       EDX
        CMP       ESI,EDX
        JA        _00426273
        MOV       EBX,EAX
        MOV       EAX,EDX
        XOR       EDX,EDX
        DIV       ESI
        MOV       EAX,EBX
    _00426273:
        DIV       ESI
        MOV       ESI, 0xE8B8D413
        MOV       EAX,EDX
        MOV       EDI,EDX
        TEST      ebp,ESI
        JZ        _00426250
        MOV       ESI, 0xF527789F
        MOV       EDX,ECX
        MUL       EDX
        CMP       ESI,EDX
        JA        _00426299
        MOV       EBX,EAX
        MOV       EAX,EDX
        XOR       EDX,EDX
        DIV       ESI
        MOV       EAX,EBX
    _00426299:
        DIV       ESI
        MOV       ECX,EDX
        JMP       _00426250
    _0042629F:
        mov       eax, dword ptr [sum]
        xor       eax, ecx
        mov       [key2], eax
        popad
    }
    printf("Your key2 is: %lX\n", key2);
}
```

`Blowfish` 的表格才真的是大

```
015F:00430CB7  CALL       CS:[KERNEL32!GetModuleFileNameA]
015F:00430CBE  MOV        EDX,80000000
015F:00430CC3  LEA        EAX,[EBP-0108]
015F:00430CC9  CALL       004355B1        //CreateFileA( )
015F:00430CCE  MOV        ECX,EAX
015F:00430CD0  CMP        EAX,-01
015F:00430CD3  JNZ        00430CDF
015F:00430CD5  MOV        EAX,FFFFFFFB
015F:00430CDA  JMP        00430DD3
015F:00430CDF  MOV        EAX,[0044D214]
015F:00430CE4  CALL       00436690
015F:00430CE9  MOV        [00453840],EAX
015F:00430CEE  TEST       EAX,EAX
015F:00430CF0  JNZ        00430CFC
015F:00430CF2  MOV        EAX,FFFFFFFE
015F:00430CF7  JMP        00430DD3
015F:00430CFC  MOV        EBX,[0044D214]
015F:00430D02  MOV        EDX,EAX
015F:00430D04  MOV        EAX,ECX
015F:00430D06  CALL       00435633               //ReadFile( )
015F:00430D0B  CMP        EAX,[0044D214]      //检查文件长度
015F:00430D11  JZ         00430D1D
015F:00430D13  MOV        EAX,FFFFFFFA
015F:00430D18  JMP        00430DD3
015F:00430D1D  MOV        EAX,ECX
015F:00430D1F  CALL       004355FE
015F:00430D24  MOV        EAX,[00453840]
015F:00430D29  ADD        EAX,[0044D218]
015F:00430D2F  MOV        DWORD PTR [EAX],00000000
015F:00430D35  MOV        EBX,FFFFFFFF
015F:00430D3A  MOV        EDX,[0044D214]
015F:00430D40  MOV        EAX,[00453840]
015F:00430D45  CALL       0042B3A0          //calculate CRC
015F:00430D4A  CMP        EAX,[0044D21C]    //compare  CRC
015F:00430D50  JZ         00430D6E          //只需把这里改成 jmp 即可
015F:00430D52  PUSH       30
015F:00430D54  PUSH       0044D230
015F:00430D59  PUSH       0044AEC0
015F:00430D5E  PUSH       00
015F:00430D60  CALL       CS:[USER32!MessageBoxA]
015F:00430D67  MOV        EAX,FFFFFFFF
015F:00430D6C  JMP        00430DD3
```



### 汇编代码片段

```
015F:00430D6E  MOV     EAX,[00453840]   
015F:00430D73  XOR     ECX,ECX   
015F:00430D75  MOV     ESI,EAX   
015F:00430D77  ADD     EAX,[0044D218]   
015F:00430D7D  ADD     ESI,00405504   
015F:00430D83  SUB     EAX,00000108   
015F:00430D88  SUB     ESI,00400A80   
015F:00430D8E  MOV     [EBP\-04],EAX   
015F:00430D91  MOV     EAX,FFFFFFFF   
015F:00430D96  JMP     00430DB8
```

### 断点设置与修改

其它常用的对付 `CRC` 的断点还有`CreateFileA`，`\_lopen`，`OpenFile( )`，`ReadFile( )`，`\_hread( )`等等。总之只要是跟文件（或者内存映射文件 API，因为有的程序用内存映射文件来检查 `CRC`）操作有关的 `API`(`Win32` 或者 `Win16`）都可以拿来做断点。

将

```
cmp eax, [0044d21c]   
jz  00430d6e   
（机器码：3b051cd24400741c)   
```

改为

```
mov  [0044d21c], eax   
nop   
jmp 00430d6e   
 （机器码：a31cd2440090eb1c）   
```

检查 `CRC` 的只此一处，上述改法是没问题的。可能别的地方改错了。

汉化完毕之后最好能恢复上述原来的代码，并在文件中填入新的 `CRC` 值（旧 `CRC` 值是`0x33438027`，保存在该文件的偏移`0x4C01C`处)，这样可以保留该软件检测病毒的功能。
4dd466d0e1e0cc123afbe504da2462ca
