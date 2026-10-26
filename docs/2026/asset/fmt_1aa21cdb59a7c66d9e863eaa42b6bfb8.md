
# 反汇编代码

```
004C1B4A  |.  8B55 F8       |MOV EDX,DWORD PTR SS:[EBP-0x8]
004C1B4D  |.  8A1402        |MOV DL,BYTE PTR DS:[EDX+EAX]            ;  nameBuf[len-i]
004C1B50  |.  8D45 E8       |LEA EAX,DWORD PTR SS:[EBP-0x18]
004C1B53  |.  E8 A029F4FF   |CALL <mp4.System.LStrFromChar>
004C1B58  |.  8B55 E8       |MOV EDX,DWORD PTR SS:[EBP-0x18]
004C1B5B  |.  8D45 F4       |LEA EAX,DWORD PTR SS:[EBP-0xC]
004C1B5E  |.  E8 912AF4FF   |CALL <mp4.System.LStrCat>               ;  保存16进制用户名倒序
004C1B63  |.  43            |INC EBX                                 ;  ++i
004C1B64  |.  4E            |DEC ESI
004C1B65  |.^ 75 D9         \JNZ SHORT mp4.004C1B40
004C1B67  |>  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-0x8]           ;  16进制转换的缓冲区地址
004C1B6A  |.  50            PUSH EAX
004C1B6B  |.  B9 04000000   MOV ECX,0x4
004C1B70  |.  BA 01000000   MOV EDX,0x1
004C1B75  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-0xC]
004C1B78  |.  E8 CF2CF4FF   CALL <mp4.System.LStrCopy>               ;  复制倒序缓冲区前4个数据 Buf1
004C1B7D  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-0xC]
004C1B80  |.  50            PUSH EAX
004C1B81  |.  B9 04000000   MOV ECX,0x4
004C1B86  |.  BA 05000000   MOV EDX,0x5
004C1B8B  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-0xC]
004C1B8E  |.  E8 B92CF4FF   CALL <mp4.System.LStrCopy>               ;  复制倒序缓冲区第4个位置后面的5个数据 Buf2
004C1B93  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-0x8]
004C1B96  |.  E8 512AF4FF   CALL <mp4.System.LStrLen>
004C1B9B  |.  83F8 04       CMP EAX,0x4                              ;  Buf1 缓冲区长度是否大于等于4
004C1B9E  |.  7D 2F         JGE SHORT mp4.004C1BCF
004C1BA0  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-0x8]
004C1BA3  |.  E8 442AF4FF   CALL <mp4.System.LStrLen>                ;  计算Buf1长度
004C1BA8  |.  8BD8          MOV EBX,EAX
004C1BAA  |.  83FB 03       CMP EBX,0x3                              ;  是否小于3?
004C1BAD  |.  7F 20         JG SHORT mp4.004C1BCF
004C1BAF  |>  8D4D E4       /LEA ECX,DWORD PTR SS:[EBP-0x1C]
004C1BB2  |.  8BC3          |MOV EAX,EBX
004C1BB4  |.  C1E0 02       |SHL EAX,0x2                             ;  长度左移2位
004C1BB7  |.  33D2          |XOR EDX,EDX
004C1BB9  |.  E8 9272F4FF   |CALL <mp4.IntToHex>                     ;  转换成16进制
004C1BBE  |.  8B55 E4       |MOV EDX,DWORD PTR SS:[EBP-0x1C]
004C1BC1  |.  8D45 F8       |LEA EAX,DWORD PTR SS:[EBP-0x8]
004C1BC4  |.  E8 2B2AF4FF   |CALL <mp4.System.LStrCat>
004C1BC9  |.  43            |INC EBX                                 ;  增加长度变量
004C1BCA  |.  83FB 04       |CMP EBX,0x4
004C1BCD  |.^ 75 E0         \JNZ SHORT mp4.004C1BAF                  ;  是否大于等于4
004C1BCF  |>  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-0xC]
004C1BD2  |.  E8 152AF4FF   CALL <mp4.System.LStrLen>
004C1BD7  |.  83F8 04       CMP EAX,0x4                              ;  Buf2 缓冲区长度是否大于等于4
004C1BDA  |.  7D 2F         JGE SHORT mp4.004C1C0B
004C1BDC  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-0xC]
004C1BDF  |.  E8 082AF4FF   CALL <mp4.System.LStrLen>                ;  计算 Buf2 缓冲区长度
004C1BE4  |.  8BD8          MOV EBX,EAX
004C1BE6  |.  83FB 03       CMP EBX,0x3                              ;  Buf2缓冲区是否大于3?
004C1BE9  |.  7F 20         JG SHORT mp4.004C1C0B
004C1BEB  |>  8D4D E0       /LEA ECX,DWORD PTR SS:[EBP-0x20]
004C1BEE  |.  8BC3          |MOV EAX,EBX
004C1BF0  |.  C1E0 02       |SHL EAX,0x2                             ;  Buf2长度左移2位
004C1BF3  |.  33D2          |XOR EDX,EDX
004C1BF5  |.  E8 5672F4FF   |CALL <mp4.IntToHex>
004C1BFA  |.  8B55 E0       |MOV EDX,DWORD PTR SS:[EBP-0x20]
004C1BFD  |.  8D45 F4       |LEA EAX,DWORD PTR SS:[EBP-0xC]
004C1C00  |.  E8 EF29F4FF   |CALL <mp4.System.LStrCat>
004C1C05  |.  43            |INC EBX                                 ;  增加长度变量
004C1C06  |.  83FB 04       |CMP EBX,0x4
004C1C09  |.^ 75 E0         \JNZ SHORT mp4.004C1BEB
004C1C0B  |>  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-0x10]
004C1C0E  |.  BA 981C4C00   MOV EDX,mp4.004C1C98                     ;  ASCII "mp46687i686c"
004C1C13  |.  E8 9027F4FF   CALL <mp4.System.LStrLAsg>               ;  复制字符串
004C1C18  |.  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-0x24]
004C1C1B  |.  50            PUSH EAX
004C1C1C  |.  B9 04000000   MOV ECX,0x4
004C1C21  |.  BA 01000000   MOV EDX,0x1
004C1C26  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-0x10]
004C1C29  |.  E8 1E2CF4FF   CALL <mp4.System.LStrCopy>               ;  保存字符串的前4个数据
004C1C2E  |.  FF75 DC       PUSH DWORD PTR SS:[EBP-0x24]
004C1C31  |.  68 B01C4C00   PUSH mp4.004C1CB0                        ;  "-"
004C1C36  |.  FF75 F8       PUSH DWORD PTR SS:[EBP-0x8]              ;  Buf1
004C1C39  |.  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-0x28]
004C1C3C  |.  50            PUSH EAX
004C1C3D  |.  B9 05000000   MOV ECX,0x5
004C1C42  |.  BA 05000000   MOV EDX,0x5
004C1C47  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-0x10]
004C1C4A  |.  E8 FD2BF4FF   CALL <mp4.System.LStrCopy>               ;  复制字符串"687i6"到缓冲区
004C1C4F  |.  FF75 D8       PUSH DWORD PTR SS:[EBP-0x28]
004C1C52  |.  68 B01C4C00   PUSH mp4.004C1CB0
004C1C57  |.  FF75 F4       PUSH DWORD PTR SS:[EBP-0xC]              ;  Buf2
004C1C5A  |.  8BC7          MOV EAX,EDI
004C1C5C  |.  BA 06000000   MOV EDX,0x6
004C1C61  |.  E8 462AF4FF   CALL <mp4.System.LStrCatN>               ;  复制正确的注册码到缓冲区中
004C1C66  |.  33C0          XOR EAX,EAX
004C1C68  |.  5A            POP EDX
004C1C69  |.  59            POP ECX
004C1C6A  |.  59            POP ECX
004C1C6B  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004C1C6E  |.  68 881C4C00   PUSH mp4.004C1C88
004C1C73  |>  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-0x28]
004C1C76  |.  BA 0A000000   MOV EDX,0xA
004C1C7B  |.  E8 B426F4FF   CALL <mp4.System.LStrArrayClr>
004C1C80  \.  C3            RETN
```

有了上面的代码,自己写了个注册机.附C代码

# C 注册机代码

```c
#include <iostream>
#include <cstdlib>
#include <string>
using namespace std;

const char* Key = "mp46687i686c";
#define MAX_LENGTH                512

// 注册码不足位数时候则填充
void FillValue( char* pBuf, int nBufSize )
{
        int len;
        char szTemp[MAX_LENGTH];
        while ( (len = strlen(pBuf)) < 4 )
        {
                sprintf_s( szTemp, MAX_LENGTH, "%X", len <<= 2 );
                strcat_s( pBuf, nBufSize, szTemp );
        }
}

int main(void)
{
        string strName;
        cout << "用户名: ";
        cin >> strName;
        // 将用户名转换成16进制
        char szTemp[MAX_LENGTH];
        string strHex;
        string::iterator it = strName.begin();
        while ( it != strName.end() )
        {
                sprintf_s( szTemp, MAX_LENGTH, "%X", *it++ );
                strHex += szTemp;
        }
        // 倒序
        string strReverse;
        for ( int n = strHex.length() - 1; n >= 0; n-- )
        {
                strReverse += strHex.at(n);
        }
        // 缓冲区的前4个数据存放到Buf1中
        char Buf1[5];
        strncpy_s( Buf1, 5, strReverse.c_str(), 4 );
        // 拷贝后4个数据到缓冲区中
        char Buf2[5];
        strncpy_s( Buf2, 5, strReverse.c_str() + 4, strReverse.length() > 4 ? 4 : 0 );
        // 计算Buf1长度,不够则左移长度填充
        if ( strlen(Buf1) < 4 )
        {
                FillValue( Buf1, 5 );
        }
        // 计算Buf2长度,不够则左移长度填充
        if ( strlen(Buf2) < 4 )
        {
                FillValue( Buf2, 5 );
        }
        // 拷贝KEY前4位数据
        string strCode;
        strncpy_s( szTemp, MAX_LENGTH, Key, 4 );
        szTemp[4] = 0x00;
        strCode += szTemp;
        strCode += "-";
        strCode += Buf1;
        strncpy_s( szTemp, MAX_LENGTH, Key + 4, 5);
        szTemp[5] = 0x00;
        strCode += szTemp;
```
