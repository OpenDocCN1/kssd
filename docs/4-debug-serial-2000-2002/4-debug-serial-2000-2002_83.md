
# NMI's Java Code Viewer 4.8.2

> 原文：[`www.pediy.com/kssd/pediy01/forum084.htm`](https://www.pediy.com/kssd/pediy01/forum084.htm)

首先加载`snjrt11.dll`的符号表，这个`DLL`从名字上看似为`Sun`的`Java Run Time`支撑库。

用`bpx MultiByteToWideChar`设断点，因为`Java`程序和`VB`程序一样，也是用`WideChar`形式的字符串。

之后再用`BPR`断点监视`WideChar`形式的注册码，发现它把你输入的注册码转换成一个整数，

再用`BPR`或`BPM`断点监视这个整数，就到了生成注册码的程序段。

一个可用的注册码为：

Name: `Me`

Country: `China`

Code: `1605226644`

下面的程序段生成并比较注册码。可见，写一个注册机也并非不可能。

```
0167:0040FD2C  SUB     ESP,18
0167:0040FD2F  PUSH    EBX
0167:0040FD30  PUSH    EBP
0167:0040FD31  PUSH    ESI
0167:0040FD32  PUSH    EDI
0167:0040FD33  MOV     ECX,[ESP+3C]
0167:0040FD37  MOV     EBP,EAX
0167:0040FD39  MOV     [ESP+10],EAX
0167:0040FD3D  MOV     ESI,[ESP+34]
0167:0040FD41  MOV     EDI,[ESP+38]
0167:0040FD45  MOV     EBX,[ECX]
0167:0040FD47  CMP     DWORD PTR [ESP+2C],00
0167:0040FD4C  JNZ     0040FD76
0167:0040FD4E  MOV     EDX,[SNJRT11!$Classjava_lang_String]
0167:0040FD54  MOV     ECX,[EBX+04]
0167:0040FD57  PUSH    EDX
0167:0040FD58  PUSH    ECX
0167:0040FD59  MOV     EAX,[00427F00]
0167:0040FD5E  CMP     EAX,00000000
0167:0040FD63  MOV     EDX,[EBX+04]
0167:0040FD66  MOV     ECX,[EDX+04]
0167:0040FD69  CALL    [ECX+44]
0167:0040FD6C  PUSH    EAX
0167:0040FD6D  CALL    SNJRT11!__j_checkcast@8
0167:0040FD72  MOV     [ESP+2C],EAX
0167:0040FD76  TEST    EBP,EBP
0167:0040FD78  JNZ     0040FDA0
0167:0040FD7A  MOV     EDX,[SNJRT11!$Classjava_lang_String]
0167:0040FD80  MOV     ECX,[EBX+04]
0167:0040FD83  PUSH    EDX
0167:0040FD84  PUSH    ECX
0167:0040FD85  MOV     EAX,[00427F44]
0167:0040FD8A  CMP     EAX,00000000
0167:0040FD8F  MOV     EDX,[EBX+04]
0167:0040FD92  MOV     ECX,[EDX+04]
0167:0040FD95  CALL    [ECX+44]
0167:0040FD98  PUSH    EAX
0167:0040FD99  CALL    SNJRT11!__j_checkcast@8
0167:0040FD9E  MOV     EBP,EAX
0167:0040FDA0  MOV     EAX,[ESP+30]
0167:0040FDA4  TEST    EAX,EAX
0167:0040FDA6  JNZ     0040FDB2
0167:0040FDA8  MOV     DWORD PTR [ESP+14],00000000
0167:0040FDB0  JMP     0040FDC1
0167:0040FDB2  MOV     EAX,[ESP+30]
0167:0040FDB6  MOV     EDX,[EAX]
0167:0040FDB8  CALL    SNJRT11!$java_lang_Integer_intValue__
0167:0040FDBD  MOV     [ESP+14],EAX
0167:0040FDC1  MOV     [ESP+10],EBP
0167:0040FDC5  MOV     ECX,[SNJRT11!$Classjava_lang_StringBuffer]
0167:0040FDCB  PUSH    ECX
0167:0040FDCC  CALL    SNJRT11!__j_new@4
0167:0040FDD1  CALL    SNJRT11!$java_lang_StringBuffer__4init_5__
0167:0040FDD6  MOV     [ESP+18],EAX
0167:0040FDDA  PUSH    EAX
0167:0040FDDB  MOV     EAX,EDI
0167:0040FDDD  CALL    SNJRT11!$java_lang_StringBuffer_append__Ljava_lang_Stri
0167:0040FDE2  MOV     EDX,[ESP+18]
0167:0040FDE6  MOV     EAX,ESI
0167:0040FDE8  PUSH    EDX
0167:0040FDE9  CALL    SNJRT11!$java_lang_StringBuffer_append__Ljava_lang_Stri
0167:0040FDEE  CALL    SNJRT11!$java_lang_StringBuffer_toString__
0167:0040FDF3  MOV     [ESP+1C],EAX
0167:0040FDF7  MOV     EAX,EDI
0167:0040FDF9  MOV     ECX,[EDI]
0167:0040FDFB  CALL    SNJRT11!$java_lang_String_hashCode__
0167:0040FE00  MOV     [ESP+20],EAX
0167:0040FE04  MOV     EAX,ESI
0167:0040FE06  MOV     EDX,[ESI]
0167:0040FE08  CALL    SNJRT11!$java_lang_String_hashCode__
0167:0040FE0D  MOV     [ESP+24],EAX
0167:0040FE11  MOV     EAX,[ESP+1C]
0167:0040FE15  MOV     ECX,[EAX]
0167:0040FE17  CALL    SNJRT11!$java_lang_String_hashCode__
0167:0040FE1C  MOV     EBP,[ESP+20]
0167:0040FE20  IMUL    EBP,[ESP+24]
0167:0040FE25  MOV     EDX,[ESP+1C]
0167:0040FE29  ADD     EBP,EAX
0167:0040FE2B  MOV     ECX,[EDX]
0167:0040FE2D  SUB     EBP,[ECX+08]
0167:0040FE30  JNS     0040FE34
0167:0040FE32  NEG     EBP
0167:0040FE34  CMP     [ESP+14],EBP                       //这里比较注册码！！！！
0167:0040FE38  JNZ     0040FF1F
```

```
//Note: only English characters allowed.
//       so you can't use your Chinese name :-(
#include <stdio.h>
#include <string.h>

long java_lang_String_hashCode(char *str);

void main(void)
{
    char Name[128];
    char Country[128];
    long NameCode, CountryCode, AllCode, Code;
    printf("KeyGen for NMI's Java Code Viewer 4.8.2.\n");
    printf("Input your name: ");
    gets(Name);
    printf("Input your country: ");
    gets(Country);
    NameCode    = java_lang_String_hashCode(Name);
    CountryCode = java_lang_String_hashCode(Country);
    AllCode     = java_lang_String_hashCode(strcat(Name, Country));
    Code = NameCode * CountryCode + AllCode;
    Code -= strlen(Name);
    if (Code < 0)
    {
        Code = -Code;
    }
    printf("You code: %lu\n", Code);
}

long java_lang_String_hashCode(char *str)
{
    int  k, len, edi,step;
    long result;
    if (str == NULL)
    {
        return 0;
    }
    result = 0;
    len = strlen(str);
    if (len < 0x10)
    {
        for (k = 0; k < len; k++)
        {
            result *= 37;
            result += (long)str[k] & 0xFF;
        }
    }
    else
    {
        k = 0;
        step = (len >> 3);
        edi = len;
        do
        {
            edi -= step;
            result *= 0x27;
            result += (long)str[k];
            k += step;
        } while (edi > 0);
    }
    return result;
}
```
