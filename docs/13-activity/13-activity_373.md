
# 第三题分析

> 原文：[`www.pediy.com/kssd/pediy11/123371.html`](https://www.pediy.com/kssd/pediy11/123371.html)

很郁闷。。。
这是我处女 KEYGEN，原本想的挺简单，谁知道弄了我一个晚上，伤心吖。。。 评委组有同情分没？![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

代码如下：
哈希算法，注册算法，都没什么问题，就是逆回去的时候发现 根本对应不到用户名与注册码。这个程序是把 `ABCDEFGHIJKMNPQRSTVWXYZ1234567890` 32 个字符的位置对应到一章表（256 个字节）上 然后取注册码的字符然后用这个字符做索引在这个表里查询，由于有效就有 34 个，(算上了 `0xFF` + `=`号)， 后面有个计算的路径，不过最多的组合就是 34 * 34 种算子的组合。 里面各有两个运算， 单个字节是 256，这样有很大的几率对应不上。
也不多说了。。我把代码贴这里，希望 KEYGEN 高手可以帮我解答一下。。。 向你们学习。
代码:



```
#include <stdio.h>
#include <windows.h>

#define \_\_MAX\_USERNAME\_SIZE\_\_                    35
#define \_\_MAX\_SERIAL\_NUMBER\_\_                    35

#define \_\_USERNAME\_SERIAL\_NUMBER\_SIZE\_\_            0x5C//92        
BYTE g\_UserNameKeyTable\[\_\_USERNAME\_SERIAL\_NUMBER\_SIZE\_\_\] = {0};
DWORD g\_UserNameSerialNumber\[5\] = {0};
BYTE g\_SerialNumber\[32\] = {0};
BYTE g\_KeyTable\[256\] = {0};
CONST PCHAR g\_ValidCharTable = "ABCDEFGHJKMNPQRSTVWXYZ1234567890";
CONST PCHAR g\_AnsiTable = "ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890";

DWORD Bswap(DWORD dwDword) {
    DWORD r = 0;
    \_\_asm {
        mov eax, dwDword
        bswap eax
        mov r, eax
    }
    return r;
}

BYTE Shl(BYTE b, BYTE n) {
    BYTE r = 0;
    \_\_asm {
        mov al, b
        mov cl, n
        shl al, cl
        mov r, al
    }
    return r;
}

BYTE Shr(BYTE b, BYTE n) {
    BYTE r = 0;
    \_\_asm {
        mov al, b
        mov cl, n
        shr al, cl
        mov r, al
    }
    return r;
}

#define \_\_TABLE\_1\_\_                    0
#define \_\_TABLE\_2\_\_                    64
DWORD dwSwitch = 0;
VOID HashIt(PUCHAR pKeyArray, PUCHAR pUserNameKeyString) {
    DWORD dwKeyX = 0;
    DWORD dwKeyY = 0;
    DWORD dwEbp = 0;//模仿的 EBP
    BYTE Table\[64 + 256\] = {0};//第一张表 64 字节,第二张表 256 字节

    // 生成第一章表
    // 第一次循环将 pUserNameKeyString 指向的内存中的 64 个字节每
    // 4 个字节做反转存入表 1
    \_\_asm {
        cld
        xor ecx, ecx
        mov esi, pUserNameKeyString
        lea edi, Table
\_make\_table\_1:
        lodsd
        bswap eax
        stosd
        add ecx, 1
        cmp ecx, 0x10
        jl \_make\_table\_1
    }

    // 生成第二章表
    \_\_asm {
        mov edx, 0x40
        lea eax, \[Table + 0x08\]
\_make\_table\_2:
        mov ecx, dword ptr \[eax + 0x2C\]
        xor ecx, dword ptr \[eax + 0x18\]
        add eax, 4
        xor ecx, dword ptr \[eax - 0x0C\]
        xor ecx, dword ptr \[eax - 0x04\]
        rol ecx, 1
        sub edx, 1
        mov dword ptr \[eax + 0x34\], ecx
        jnz \_make\_table\_2
    }

    // 这里开始计算两个重要的 KEY
    \_\_asm {
        mov eax, pKeyArray
        mov edx, dword ptr \[eax + 0x04\]
        //push ebx
        mov ebx, dword ptr \[eax + 0x0C\]
        //push ebp
        //push esi
        mov esi, dword ptr \[eax + 0x08\]
        //push edi
        mov edi, dword ptr \[eax\]
        mov eax, dword ptr \[eax + 0x10\]
        mov dwKeyX, eax
        mov dwKeyY, edi
        xor eax, eax
        mov edi, edi
\_fuck\_loop:
        rol edi, 5
        mov dwEbp, esi
        and dwEbp, edx
        mov ecx, edx
        not ecx
        and ecx, ebx
        xor ecx, dwEbp
        add edi, ecx
        add edi, dword ptr \[Table + eax \* 4\]
        mov ecx, dwKeyX
        lea ecx, dword ptr \[edi + ecx + 0x5A827999\]
        mov edi, dwKeyY
        mov dwKeyX, ebx
        ror edx, 2
        mov dwKeyY, ecx
        rol ecx, 5
        mov ebx, edi
        not ebx
        and ebx, esi
        mov dwEbp, edx
        and dwEbp, edi
        xor ebx, dwEbp
        add ecx, ebx
        {
            // add ecx, dword ptr \[esp + eax \* 4 + 0x1C\]
            mov dwSwitch, edi
            lea edi, \[Table + eax \* 4\]
            add edi, 0x04
            add ecx, dword ptr \[edi\]
            mov edi, dwSwitch
        }
        mov ebx, dwKeyX
        lea ecx, dword ptr \[ecx + ebx + 0x5A827999\]
        ror edi, 2
        mov dwKeyX, esi
        mov ebx, edx
        mov edx, dwKeyY
        mov dwKeyY, ecx
        rol ecx, 5
        mov dwEbp, edi
        and dwEbp, edx
        mov esi, edx
        not esi
        and esi, ebx
        xor esi, dwEbp
        add ecx, esi
        {
            // add ecx, dword ptr \[esp + eax \* 4 + 0x20\]
            mov dwSwitch, edi
            lea edi, \[Table + eax \* 4\]
            add edi, 0x08
            add ecx, dword ptr \[edi\]
            mov edi, dwSwitch
        }
        mov esi, dwKeyX
        lea ecx, dword ptr \[ecx + esi + 0x5A827999\]
        mov esi, dwKeyY
        ror edx, 2
        mov dwKeyX, ebx
        mov dwKeyY, ecx
        rol ecx, 5
        mov ebx, esi
        not ebx
        and ebx, edi
        mov dwEbp, edx
        and dwEbp, esi
        xor ebx, dwEbp
        add ecx, ebx
        {
            // add ecx, dword ptr \[esp + eax \* 4 + 0x24\]
            mov dwSwitch, edi
            lea edi, \[Table + eax \* 4\]
            add edi, 0x0C
            add ecx, dword ptr \[edi\]
            mov edi, dwSwitch
        }
        mov ebx, dwKeyX
        lea ecx, dword ptr \[ecx + ebx + 0x5A827999\]
        ror esi, 2
        mov dwKeyX, edi
        mov ebx, esi
        mov esi, dwKeyY
        mov edi, esi
        not edi
        and edi, edx
        mov dwKeyY, ecx
        rol ecx, 5
        mov dwEbp, ebx
        and dwEbp, esi
        xor edi, dwEbp
        add ecx, edi
        {
            // add ecx, dword ptr \[esp + eax \* 4 + 0x28\]
            mov dwSwitch, edi
            lea edi, \[Table + eax \* 4\]
            add edi, 0x10
            add ecx, dword ptr \[edi\]
            mov edi, dwSwitch
        }
        mov edi, dwKeyX
        lea ecx, dword ptr \[ecx + edi + 0x5A827999\]
        mov dwEbp, edx
        mov edx, dwKeyY
        mov edi, ecx
        add eax, 5 //指针计数增加
        ror esi, 2
        {
            // mov dword ptr \[esp + 0x10\], ebp
            mov dwSwitch, eax
            mov eax, dwEbp
            mov dwKeyX, eax
            mov eax, dwSwitch
        }
        mov dwKeyY, edi
        cmp eax, 0x14
        jl \_fuck\_loop
    }

    // 继续计算 KEY
    \_\_asm {
\_fuck\_loop2:
        mov dwEbp, ebx
        xor dwEbp, esi
        xor dwEbp, edx
        rol ecx, 5
        add ecx, dwEbp
        add ecx, dword ptr \[Table + eax \* 4\]
        {
            // mov ebp, dwKeyX
            mov dwSwitch, eax
            mov eax, dwKeyX
            mov dwEbp, eax
            mov eax, dwSwitch
        }
        {
            // lea ecx, dword ptr \[ecx + ebp + 0x6ED9EBA1\]
            mov dwSwitch, eax
            mov eax, dwEbp
            lea ecx, dword ptr \[ecx + eax + 0x6ED9EBA1\]
            mov eax, dwSwitch
        }
        ror edx, 2
        mov dwEbp, ebx
        mov ebx, esi
        add eax, 1    // 移动指针
        cmp eax, 0x28
        mov esi, edx
        mov edx, edi
        {
            // mov dwKeyX, ebp
            mov dwSwitch, eax
            mov eax, dwEbp
            mov dwKeyX, eax
            mov eax, dwSwitch
        }
        mov edi, ecx
        jl \_fuck\_loop2
    }

    // 单独设置一下 KeyY
    \_\_asm {
        mov dwKeyY, ecx
    }

    // 继续
    \_\_asm {
\_fuck\_loop3:
        mov edi, esi
        xor edi, edx
        and edi, ebx
        mov dwEbp, esi
        and dwEbp, edx
        rol ecx, 5
        add ecx, dwKeyX
        xor edi, dwEbp
        add edi, dword ptr \[Table + eax \* 4\]
        ror edx, 2
        mov dwEbp, ebx
        mov ebx, esi
        add eax, 1
        cmp eax, 0x3C
        lea ecx, dword ptr \[edi + ecx + 0x8F1BBCDC\]
        mov esi, edx
        mov edx, dwKeyY
        {
            // mov dwKeyX, ebp
            mov dwSwitch, eax
            mov eax, dwEbp
            mov dwKeyX, eax
            mov eax, dwSwitch
        }
        mov dwKeyY, ecx
        jl \_fuck\_loop3
    }

    // 还有
    \_\_asm {
\_fuck\_loop4:
        mov edi, ebx
        xor edi, esi
        rol ecx, 5
        xor edi, edx
        add edi, dword ptr \[Table + eax \* 4\]
        add ecx, dwEbp
        ror edx, 2
        mov dwEbp, ebx
        mov ebx, esi
        add eax, 1
        cmp eax, 0x50
        lea ecx, \[edi + ecx + 0xCA62C1D6\]
        mov esi, edx
        mov edx, dwKeyY
        mov dwKeyY, ecx
        jl \_fuck\_loop4
    }

    // 输出结果
    \_\_asm {
        mov eax, pKeyArray
        mov edi, dword ptr \[eax\]
        add edi, ecx
        mov ecx, dword ptr \[eax + 0x04\]
        add ecx, edx
        mov dword ptr \[eax + 0x04\], ecx
        mov ecx, dword ptr \[eax + 0x08\]
        add ecx, esi
        mov dword ptr \[eax + 0x08\], ecx
        mov ecx, dword ptr \[eax + 0x0C\]
        add ecx, ebx
        mov dword ptr \[eax\], edi
        mov dword ptr \[eax + 0x0C\], ecx
        mov ecx, dword ptr \[eax + 0x10\]
        add ecx, dwEbp
        mov dword ptr \[eax + 0x10\], ecx
    }
    return;
}

VOID ShowSign() {
    DWORD dwSign = 0;
    int i = 0;

    do {
        dwSign += 0xFFFFFFFD;
        dwSign &= 7;
        printf("%d ", dwSign);
        if ((i != 0) && (i % 0x10 == 0))
            printf("\\r\\n");
        i++;
    } while (i < 0x20);
}

// 注册算法
VOID SerialNumberGen(PCHAR pSerialNumberString) {
    DWORD dwSign = 0;
    int i = 0, j = 0;
    DWORD n = 0;
    BYTE ch = 0, ch2 = 0;
    
    do {
        // 读取一个字符
        ch = pSerialNumberString\[i\];
        ch = g\_KeyTable\[ch\];

        if (dwSign <= 3) {
            dwSign += 0xFFFFFFFD;
            dwSign &= 7;
            if (dwSign == 0) {
                g\_SerialNumber\[j\] |= ch;
                j++;
            } else {
                n = 8 - dwSign;
                ch = Shl(ch, (BYTE)n);
                g\_SerialNumber\[j\] |= ch;
            }
        } else {
            dwSign += 0xFFFFFFFD;
            dwSign &= 7;
            ch2 = ch;
            n = (BYTE)dwSign;
            ch2 = Shr(ch2, (BYTE)n);
            g\_SerialNumber\[j\] |= ch2;
            j++;
            // 设置下一个字节
            n = 8 - dwSign;
            ch = Shl(ch, (BYTE)n);
            g\_SerialNumber\[j\] |= ch;
        }
        i++;
    } while (i < 0x20);
}


/\*
 \* 最多 34 \* 34 种组合的值,不能满足所有用户名的需求
 \*/
// 逆向注册算法
// 流程处理到 5, 2 ,7, 4, 1, 6, 3, 0 其余做同样的循环
CONST BYTE g\_ReverseSign\[33\] = {//由 ShowSign 得出
    "\\x05\\x02\\x07\\x04\\x01\\x06\\x03\\x00\\x05\\x02\\x07\\x04\\x01\\x06\\x03\\x00"
    "\\x05\\x02\\x07\\x04\\x01\\x06\\x03\\x00\\x05\\x02\\x07\\x04\\x01\\x06\\x03\\x00"
};
VOID ReverseSerialNumberGen(PUCHAR pSerialNumber, PCHAR pSerialNumberString) {
    CHAR ch = 0, ch2 = 0, x = 0, y = 0, z = 0, result = 0;
    BYTE \_ch = 0, \_ch2 = 0, \_c = 0;
    int i = 0, j = 0;
    int n = 0, m = 0;

    for (n = 0; n < 20; n+=5) {
        // 0, 5, 2-
        result = pSerialNumber\[n\];
        for (i = 0; i < 36; i++) {
            ch = g\_AnsiTable\[i\];
            \_ch = g\_KeyTable\[ch\];
            \_ch = Shl(\_ch, 8 - 5);
            x |= \_ch;
            y = x;
            for (j = 0; j < 36; j++) {
                ch2 = g\_AnsiTable\[j\];
                \_ch2 = g\_KeyTable\[ch2\];
                \_c = \_ch2;
                \_ch2 = Shr(\_ch2, 2);
                x |= \_ch2;
                if (x == result) {
                    pSerialNumberString\[m++\] = ch;
                    pSerialNumberString\[m++\] = ch2;
                    goto \_byte\_2;
                }
                x = y;
            }/\* end for \*/
            x = 0;
        }

\_byte\_2:
        // -2, 7, 4-
        x = 0;
        result = pSerialNumber\[n+1\];
        \_c = Shl(\_c, 8 - 2);
        x |= \_c;
        z = x;

        for (i = 0; i < 36; i++) {
            ch = g\_AnsiTable\[i\];
            \_ch = g\_KeyTable\[ch\];
            \_ch = Shl(\_ch, 8 - 7);
            x |= \_ch;
            y = x;
            for (j = 0; j < 36; j++) {
                ch2 = g\_AnsiTable\[j\];
                \_ch2 = g\_KeyTable\[ch2\];
                \_c = \_ch2;
                \_ch2 = Shr(\_ch2, 4);
                x |= \_ch2;
                if (x == result) {
                    pSerialNumberString\[m++\] = ch;
                    pSerialNumberString\[m++\] = ch2;
                    goto \_byte\_3;
                }
                x = y;
            }/\* end for \*/
            x = z;
        }

\_byte\_3:
        // -4, 1-
        x = 0;
        result = pSerialNumber\[n+2\];
        \_c = Shl(\_c, 8 - 4);
        x |= \_c;
        z = x;

        for (i = 0; i < 36; i++) {
            ch = g\_AnsiTable\[i\];
            \_ch = g\_KeyTable\[ch\];
            \_c = \_ch;
            \_ch = Shr(\_ch, 1);
            x |= \_ch;
            if (x == result) {
                pSerialNumberString\[m++\] = ch;
                goto \_byte\_4;
            }
            x = z;
        }

\_byte\_4:
        // -1, 6, 3-
        x = 0;
        result = pSerialNumber\[n+3\];
        \_c = Shl(\_c, 8 - 1);
        x |= \_c;
        z = x;

        for (i = 0; i < 36; i++) {
            ch = g\_AnsiTable\[i\];
            \_ch = g\_KeyTable\[ch\];
            \_ch = Shl(\_ch, 8 - 6);
            x |= \_ch;
            y = x;
            for (j = 0; j < 36; j++) {
                ch2 = g\_AnsiTable\[j\];
                \_ch2 = g\_KeyTable\[ch2\];
                \_c = \_ch2;
                \_ch2 = Shr(\_ch2, 3);
                x |= \_ch2;
                if (x == result) {
                    pSerialNumberString\[m++\] = ch;
                    pSerialNumberString\[m++\] = ch2;
                    goto \_byte\_5;
                }
                x = y;
            }
            x = z;
        }

\_byte\_5:
        // -3, 0
        x = 0;
        result = pSerialNumber\[n+4\];
        \_c = Shl(\_c, 8 - 3);
        x |= \_c;
        z = x;

        for (i = 0; i < 36; i++) {
            ch = g\_AnsiTable\[ch\];
            \_ch = g\_KeyTable\[i\];
            x |= \_ch;
            if (x == result) {
                pSerialNumberString\[m++\] = ch;
                goto \_next\_turn;
            }
            x = z;
        }
\_next\_turn:
        ;
    }/\* end for \*/
}

// 生成注册字符表
VOID MakeValidCharTable() {
    int i = 0;
    CHAR ch = '=';
    memset(g\_KeyTable, 0xFF, 256);
    do {
        ch = g\_ValidCharTable\[i\];
        g\_KeyTable\[ch\] = i;
        i++;
    } while (i < 0x20);
    g\_KeyTable\[0x3D\] = 0x20;//i
}

/\*
 \* 感觉最后逆注册算法上算子组合的有很大可能碰不到用户名生成的 HASH 值
 \* 最后只有 34 种可能 32 个有效字符 + 0xFF + 0x20,但是可能有些路径上
 \* 只有 34 种单一的选择,至少我的 KEYGEN 能力到这里了。
 \* 哈希算法正确,注册算法正确,生表算法正确,最后的逆运算应该也没错,
 \* 不知道神人们是怎么弄出来的，多多向你们学习。
 \*/
int main() {
    DWORD VolumeSerialNumber = 0;
    DWORD dwMagicLen = 0;
    DWORD dwTmp = 0;
    CHAR UserName\[\_\_MAX\_USERNAME\_SIZE\_\_ + 1\] = {0};
    CHAR SerialNumber\[\_\_MAX\_SERIAL\_NUMBER\_\_ + 1\] = {0};
    CHAR Command\[64\] = {0};
    int Len = 0;
    PUCHAR pKeyArray = NULL;
    PUCHAR pUserNameKeyString = NULL;
    int i = 0;

    //ShowSign();

    printf("输入用户名(最长 35 个字节):");
    fgets(Command, 64, stdin);
    Len = strlen(Command);
    Len--;
    Command\[Len\] = '\\0';
    strcpy(UserName, Command);

    // 获取 C 盘序列号
    GetVolumeInformationA("C:\\\\", \\
                          0, \\
                          0, \\
                          &VolumeSerialNumber, \\
                          0, \\
                          0, \\
                          0, \\
                          0);

    // 通过长度获取值
    dwMagicLen = (Len + 11) \* 8;
    dwMagicLen = Bswap(dwMagicLen);

    // 开始设置表
    memset(g\_UserNameKeyTable, 0, \_\_USERNAME\_SERIAL\_NUMBER\_SIZE\_\_);
    memcpy(g\_UserNameKeyTable + 0x00, "\\x00\\x20\\x00\\x00\\x00\\x00\\x00\\x00", 8);
    memcpy(g\_UserNameKeyTable + 0x08, "\\xCA\\xB1\\xCA\\xB1", 4);
    memcpy(g\_UserNameKeyTable + 0x0C, "\\xBF\\xCC\\xBF\\xCC", 4);
    memcpy(g\_UserNameKeyTable + 0x10, "\\xBE\\xD6\\xB2\\xBF", 4);
    memcpy(g\_UserNameKeyTable + 0x14, "\\xB5\\xD8\\xC7\\xF8", 4);
    memcpy(g\_UserNameKeyTable + 0x18, "\\xCD\\xBC\\xC7\\xEE", 4);
    memcpy(g\_UserNameKeyTable + 0x1C, UserName, Len);
    memcpy(g\_UserNameKeyTable + 0x1C + Len, &VolumeSerialNumber, 4);
    memcpy(g\_UserNameKeyTable + 0x1C + Len + 4, "Tencent", 7);
    memcpy(g\_UserNameKeyTable + 0x1C + Len + 4 + 7, "\\x80\\x00\\x00\\x00", 4);
    memcpy(g\_UserNameKeyTable + 0x58, &dwMagicLen, 4);

    // 将这个数据传人哈希算法
    pKeyArray = (g\_UserNameKeyTable + 0x08);
    pUserNameKeyString = (g\_UserNameKeyTable + 0x1C);
    HashIt(pKeyArray, pUserNameKeyString);
    // 将得到的哈希值每个都经过一次翻转, 总共 20 个字节
    for (i = 0; i < 5; i++) {
        dwTmp = \*((PDWORD)pKeyArray + i);
        g\_UserNameSerialNumber\[i\] = Bswap(dwTmp);
    }
    
    // 通过用户名的哈希值算出注册码
    MakeValidCharTable();//生成有效字符表
    ReverseSerialNumberGen((PUCHAR)g\_UserNameSerialNumber, SerialNumber);
    printf("Serial Number = %C%C%C%C%C%C%C%C-%C%C%C%C%C%C%C%C-%C%C%C%C%C%C%C%C-%C%C%C%C%C%C%C%C\\r\\n", \\
            SerialNumber\[0\], SerialNumber\[1\], SerialNumber\[2\], SerialNumber\[3\], \\
            SerialNumber\[4\], SerialNumber\[5\], SerialNumber\[6\], SerialNumber\[7\], \\
            SerialNumber\[8\], SerialNumber\[9\], SerialNumber\[10\], SerialNumber\[11\], \\
            SerialNumber\[12\], SerialNumber\[13\], SerialNumber\[14\], SerialNumber\[15\], \\
            SerialNumber\[16\], SerialNumber\[17\], SerialNumber\[18\], SerialNumber\[19\], \\
            SerialNumber\[20\], SerialNumber\[21\], SerialNumber\[22\], SerialNumber\[23\], \\
            SerialNumber\[24\], SerialNumber\[25\], SerialNumber\[26\], SerialNumber\[27\], \\
            SerialNumber\[28\], SerialNumber\[29\], SerialNumber\[30\], SerialNumber\[31\]);

    //SerialNumberGen("FFFFFFFF-FFFFFFFF-FFFFFFFF-FFFFFFFF");

    fgetc(stdin);
    // 打印最终结果
    return 0;
}

```



其实不需要汇编代码的

`SHA1` 直接用 `openssl` 库就行了

```c
#include <openssl/sha.h>

#pragma comment (lib,"libeay32.lib")

#define SWAPDWORD(x) (((x)<<24) | (((x)<<8)&0x00FF0000) | (((x)>>8) & 0x0000FF00) | ((x)>>24))

void GetHash(const char* pszUserName, BYTE* pHash)
{
  DWORD dwVolumeSerialNumber = 0;
  GetVolumeInformation("c:\\\\", NULL, 0, &dwVolumeSerialNumber, NULL, NULL, NULL, 0);
  int nLen = strlen(pszUserName);
  BYTE* pData = new BYTE[nLen + 16];
  memcpy(pData, pszUserName, nLen);
  memcpy(pData+nLen, &dwVolumeSerialNumber, 4);
  memcpy(pData+nLen+4, "Tencent", 7);
  SHA_CTX c;
  unsigned long h[] = {
      0xB1CAB1CA,
      0xCCBFCCBF,
      0xBFB2D6BE,
      0xF8C7D8B5,
      0xEEC7BCCD};
  unsigned char byte_40CD50[0x80] = {0};
  byte_40CD50[0] = byte_40CD50[0x48] = 0x80;
  byte_40CD50[0x40] = 0x3d;
  SHA1_Init(&c);
  c.h0 = h[0];
  c.h1 = h[1];
  c.h2 = h[2];
  c.h3 = h[3];
  c.h4 = h[4];
  SHA1_Update(&c, pData, nLen+4+7);
  delete[] pData;
  pData = NULL;
  DWORD xlen[2];
  xlen[0] = SWAPDWORD(c.Nh);
  xlen[1] = SWAPDWORD(c.Nl);
  int n1 = (c.Nl >> 3) & 0x3F;
  int n2 = 56;
  if ( (unsigned int)n1 >= 0x38 )
    n2 = 120;
  SHA1_Update(&c, byte_40CD50, n2-n1);
  SHA1_Update(&c, xlen, 8);
  DWORD* hash = (DWORD*)pHash;
  hash[0] = SWAPDWORD(c.h0);
  hash[1] = SWAPDWORD(c.h1);
  hash[2] = SWAPDWORD(c.h2);
  hash[3] = SWAPDWORD(c.h3);
  hash[4] = SWAPDWORD(c.h4);
}

void HashToData(BYTE* pHash, BYTE* pData)
{
  int i;
  int j;
  BYTE bits[20*8];
  for(i=0; i<20; i++)
  {
    for(j=0; j<8; j++)
    {
      bits[i*8+7-j] = (pHash[i]>>j)&1;
    }
  }
  for(i=0; i<32; i++)
  {
    for(j=0; j<5; j++)
    {
      pData[i] |= bits[i*5+j] << (4-j);
    }
  }
}

void DataToCode(BYTE* pData, char* pszCode)
{
  char* pstr = "ABCDEFGHJKMNPQRSTVWXYZ1234567890";
  int i;
  for(i=0; i<32; i++)
  {
    pszCode[i] = pstr[pData[i]];
  }
}

void Keygen(const char* pszUserName, char* pszLicenseCode)
{
  BYTE cbHash[20];
  BYTE cbData[32];
  GetHash(pszUserName, cbHash);
  memset(cbData, 0, sizeof(cbData));
  HashToData(cbHash, cbData);
  char szCode[64] = {0};
  DataToCode(cbData, szCode);
  pszLicenseCode[8] = pszLicenseCode[0x11] = pszLicenseCode[0x1a] = '-';
  memcpy(pszLicenseCode, szCode, 8);
  memcpy(pszLicenseCode+9, szCode+8, 8);
  memcpy(pszLicenseCode+0x12, szCode+16, 8);
  memcpy(pszLicenseCode+0x1b, szCode+24, 8);
}

void CKeygenDlg::OnGen()
{
  CString sUserName;
  GetDlgItemText(IDC_EDIT_USERNAME, sUserName);
  if(sUserName.IsEmpty())
  {
    SetDlgItemText(IDC_EDIT_LIC_CODE, NULL);
    GetDlgItem(IDC_EDIT_USERNAME)->SetFocus();
    return;
  }
  char szLicenseCode[128];
  memset(szLicenseCode, 0, sizeof(szLicenseCode));
  Keygen(sUserName, szLicenseCode);
  SetDlgItemText(IDC_EDIT_LIC_CODE, szLicenseCode);
}
```
