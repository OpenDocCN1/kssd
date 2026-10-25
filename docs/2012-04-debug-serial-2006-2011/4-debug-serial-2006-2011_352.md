
# 旺旺聊天记录解码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-60.htm`](https://www.pediy.com/kssd/pediy09/pediy09-60.htm)

> 文章标题:旺旺聊天记录解码

> 软件名称:淘宝旺旺

> 使用工具:OD

> 作者声明:只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

> 软件下载地址:`www.taobao.com`

> 旺旺所有文件(这里声明文件)的解码都只用到下面这个函数，我将它还原成高级语言了。有错误之处请高手指教。

```asm
005C1070  /.  55            push    ebp
005C1071  |.  8BEC          mov     ebp, esp
005C1073  |.  83EC 14       sub     esp, 14
005C1076  |.  53            push    ebx
005C1077  |.  56            push    esi
005C1078  |.  8B45 10       mov     eax, dword ptr [ebp+10]            ; [ebp+10]指向缓冲区
005C107B  |.  8945 F4       mov     dword ptr [ebp-C], eax             ; 将[ebp+10]保存在临时变量[ebp-C]，即[ebp-C]指向缓冲区
005C107E  |.  8B4D 14       mov     ecx, dword ptr [ebp+14]
005C1081  |.  894D F8       mov     dword ptr [ebp-8], ecx             ; 将[ebp+14]保存在临时变量[ebp-8]
005C1084  |.  C745 FC 00000>mov     dword ptr [ebp-4], 0               ; 初始化计数器为 0
005C108B  |.  EB 09         jmp     short 005C1096
005C108D  |>  8B55 FC       /mov    edx, dword ptr [ebp-4]             ; [ebp-4]计数器
005C1090  |.  83C2 01       |add    edx, 1                             ; 计数器加 1
005C1093  |.  8955 FC       |mov    dword ptr [ebp-4], edx             ; 将新值保存入变量
005C1096  |>  8B45 FC       mov     eax, dword ptr [ebp-4]
005C1099  |.  3B45 18       |cmp    eax, dword ptr [ebp+18]            ; [ebp+18]是数据块大小，按字节加密
005C109C  |.  7D 7D         |jge    short 005C111B
005C109E  |.  8D4D F0       |lea    ecx, dword ptr [ebp-10]
005C10A1  |.  894D EC       |mov    dword ptr [ebp-14], ecx            ; 局部[ebp-14]指针变量，指向局部变量[ebp-10]
005C10A4  |.  8B45 FC       |mov    eax, dword ptr [ebp-4]             ; 计数器载入 eax
005C10A7  |.  99            |cdq                                      ; 符号扩展
005C10A8  |.  8B4D 1C       |mov    ecx, dword ptr [ebp+1C]            ; B9800 常量,该常量是数据块起始位置在文件中的偏移
005C10AB  |.  03C8          |add    ecx, eax                           ; B9800(密钥)加上计数器
005C10AD  |.  8B45 20       |mov    eax, dword ptr [ebp+20]
005C10B0  |.  13C2          |adc    eax, edx
005C10B2  |.  8B55 EC       |mov    edx, dword ptr [ebp-14]            ; 指针运算
005C10B5  |.  890A          |mov    dword ptr [edx], ecx               ; ecx 保存入[ebp-10]
005C10B7  |.  8B45 F4       |mov    eax, dword ptr [ebp-C]             ; 缓冲区数地址载入 eax
005C10BA  |.  0345 FC       |add    eax, dword ptr [ebp-4]             ; 指向偏移
005C10BD  |.  33C9          |xor    ecx, ecx                           ; ecx 清 0
005C10BF  |.  8A08          |mov    cl, byte ptr [eax]                 ; 取出一个字节，以下进行解密运算
005C10C1  |.  8B55 F0       |mov    edx, dword ptr [ebp-10]            ; [ebp-10]取的每个字节
005C10C4  |.  81E2 FF000000 |and    edx, 0FF
005C10CA  |.  33C0          |xor    eax, eax
005C10CC  |.  8A82 ECD56F00 |mov    al, byte ptr [edx+6FD5EC]
005C10D2  |.  8BF0          |mov    esi, eax                           ; 暂时存入 esi
005C10D4  |.  8B55 F1       |mov    edx, dword ptr [ebp-F]             ; [ebp-10]，取它的每个字节作为 g_buf 的下标，进行运算。
005C10D7  |.  81E2 FF000000 |and    edx, 0FF
005C10DD  |.  33DB          |xor    ebx, ebx
005C10DF  |.  8A9A ECD56F00 |mov    bl, byte ptr [edx+6FD5EC]
005C10E5  |.  8B55 F2       |mov    edx, dword ptr [ebp-E]             ; [ebp-10]取的每个字节
005C10E8  |.  81E2 FF000000 |and    edx, 0FF
005C10EE  |.  33C0          |xor    eax, eax
005C10F0  |.  8A82 ECD56F00 |mov    al, byte ptr [edx+6FD5EC]
005C10F6  |.  03D8          |add    ebx, eax
005C10F8  |.  8B45 F3       |mov    eax, dword ptr [ebp-D]             ; [ebp-10]取的每个字节
005C10FB  |.  25 FF000000   |and    eax, 0FF
005C1100  |.  33D2          |xor    edx, edx
005C1102  |.  8A90 ECD56F00 |mov    dl, byte ptr [eax+6FD5EC]         ; 6FD5EC 是个数组，大小为 255，我从内存中直接 dump 下来了，名为 g_buf
005C1108  |.  2BDA          |sub    ebx, edx
005C110A  |.  33F3          |xor    esi, ebx
005C110C  |.  33CE          |xor    ecx, esi
005C110E  |.  8B45 F8       |mov    eax, dword ptr [ebp-8]
005C1111  |.  0345 FC       |add    eax, dword ptr [ebp-4]
005C1114  |.  8808          |mov    byte ptr [eax], cl
005C1116  |.^ E9 72FFFFFF   \jmp    005C108D
005C111B  |>  5E            pop     esi
005C111C  |.  5B            pop     ebx
005C111D  |.  8BE5          mov     esp, ebp
005C111F  |.  5D            pop     ebp
005C1120  \.  C3            retn
```

下面付上解码程序的 C++源码

```cpp
#include<iostream.h>

#include<windows.h>

//解码函数，上面的解码函数有 7 个参数，其中指向缓冲区的两个，到现在还不明白为什么(所以被我减少到 6 个参数)，下面注释保留两个字的，在反汇编过程中发现没有用到过。

void DeCode(const int m, /*保留*/, const int n, /*保留*/ char *buf,
            int bufSize, int fOffset, const int l = 0 /*保留*/);

unsigned char g_buf[256] = {
(char)0xE9, (char)0xF4, (char)0x8B, (char)0x9E, (char)0xAA,
(char)0x50, (char)0x2F, (char)0x3E, (char)0xC2, (char)0xDB,
(char)0x8C, (char)0xB8, (char)0x08, (char)0xC7, (char)0x52,
(char)0x75, (char)0x59, (char)0x0A, (char)0x42, (char)0xCB,
(char)0x94, (char)0x34, (char)0x21, (char)0x03, (char)0x53,
(char)0x06, (char)0x43, (char)0x76, (char)0x15, (char)0x5C,
(char)0xE3, (char)0x2A, (char)0xFB, (char)0x6B, (char)0x4A,
(char)0x48, (char)0x1C, (char)0x9C, (char)0x17, (char)0x98,
(char)0x05, (char)0x11, (char)0x40, (char)0x86, (char)0x46,
(char)0xF1, (char)0xCC, (char)0x9A, (char)0xA9, (char)0x1F,
(char)0x2E, (char)0x57, (char)0xD6, (char)0x8E, (char)0xDA,
(char)0xB6, (char)0x64, (char)0xCA, (char)0x67, (char)0x31,
(char)0x2C, (char)0x3D, (char)0x5E, (char)0x85, (char)0x5F,
(char)0x04, (char)0xA0, (char)0xC3, (char)0x81, (char)0xD7,
(char)0xF5, (char)0xD3, (char)0x39, (char)0xED, (char)0x24,
(char)0xD1, (char)0x54, (char)0x36, (char)0x8F, (char)0x35,
(char)0xAE, (char)0x07, (char)0x4F, (char)0xC9, (char)0x71,
(char)0x37, (char)0xA5, (char)0xBF, (char)0x51, (char)0x61,
(char)0x8A, (char)0x69, (char)0x77, (char)0x6A, (char)0x0E,
(char)0x2B, (char)0x4D, (char)0x3B, (char)0xFE, (char)0xA2,
(char)0xEF, (char)0x8D, (char)0xB4, (char)0x5D, (char)0x90,
(char)0x1E, (char)0xFC, (char)0x26, (char)0x14, (char)0xF9,
(char)0x1A, (char)0xD5, (char)0x58, (char)0x12, (char)0x76,
(char)0x0D, (char)0xE4, (char)0xEB, (char)0x88, (char)0x2D,
(char)0x9D, (char)0xEE, (char)0x78, (char)0x97, (char)0xF7,
(char)0x32, (char)0x7D, (char)0x19, (char)0xC5, (char)0xAD,
(char)0x10, (char)0x7B, (char)0xF3, (char)0x1D, (char)0xE8,
(char)0x55, (char)0x3F, (char)0x09, (char)0x01, (char)0xAC,
(char)0x7A, (char)0x3A, (char)0x5B, (char)0x56, (char)0x13,
(char)0xC1, (char)0x6E, (char)0x0C, (char)0x4B, (char)0x7F,
(char)0x60, (char)0xD8, (char)0xD4, (char)0xA7, (char)0x68,
(char)0xDF, (char)0xCE, (char)0x95, (char)0x3C, (char)0x99,
(char)0x74, (char)0x25, (char)0x72, (char)0xF8, (char)0x93,
(char)0x00, (char)0x28, (char)0xBD, (char)0xC4, (char)0xC6,
(char)0xFD, (char)0xE0, (char)0xF2, (char)0x23, (char)0x27,
(char)0xB9, (char)0x29, (char)0xBA, (char)0xE5, (char)0xBE,
(char)0xCD, (char)0x5A, (char)0x89, (char)0x70, (char)0xD0,
(char)0xEA, (char)0xF0, (char)0xFF, (char)0xB7, (char)0xAB,
(char)0x47, (char)0xF6, (char)0x6D, (char)0xB1, (char)0x80,
(char)0xFA, (char)0xC0, (char)0xA3, (char)0x82, (char)0x73,
(char)0xAF, (char)0xEC, (char)0x33, (char)0x20, (char)0x63,
(char)0x9F, (char)0xE1, (char)0x9B, (char)0x4C, (char)0xD2,
(char)0xE2, (char)0x42, (char)0x4E, (char)0x16, (char)0x84,
(char)0x65, (char)0x7E, (char)0x6F, (char)0xB2, (char)0x45,
(char)0xB0, (char)0x83, (char)0xBC, (char)0x7C, (char)0xA4,
};
```



```
(char)0xA8, (char)0xDE, (char)0x62, (char)0xA1, (char)0x22,

(char)0xDD, (char)0x0F, (char)0xC8, (char)0x44, (char)0x6C,

(char)0x30, (char)0xE7, (char)0xCF, (char)0x49, (char)0xBB,

(char)0xE6, (char)0x38, (char)0x66, (char)0xA6, (char)0xD9,

(char)0x87, (char)0x1B, (char)0x02, (char)0xB3, (char)0xDC,

(char)0x18, (char)0x92, (char)0x0B, (char)0xB5, (char)0x91

};

char buf[0x400];

void main()
{
    DWORD nByte;
    HANDLE hFile=::CreateFile("C:\\\\Program Files\\\\淘宝网\\\\淘宝旺旺\\\\Users\\\\system\\\\TBMsg.atb",  //该文件为旺旺的系统文件，默认密码保存在文件偏移 0x1400,大小为 400，不过只是用户名没有被加密，密码还是被加密了，还可以继续研究，嘿嘿!!
        0x80000000,1,0,3,0x80,0);
    HANDLE hFile1=::CreateFile("C:\\\\b.txt",
        GENERIC_WRITE,1,0,3,0x80,0);

    for(int i=0;i<0x2000;i+=0x400)   //0x2000 为我系统目录下这个文件的大小，大家依情况而定，旺旺都是通过 0x400 为一块数据块进行解码的，即 1024 字节。
    {
        ReadFile(hFile,buf,0x400,&nByte,0);
        cout<<nByte<<endl;
        DeCode(0,1,buf,0x400,i,0);
        WriteFile(hFile1,buf,0x400,&nByte,0);
    }
    CloseHandle(hFile);
    CloseHandle(hFile1);
}

void DeCode(const int m/*保留*/,const int n,/*保留*/char *buf,
      int bufSize,int fOffset,const int l/*保留*/)
{
    unsigned int p_count,p_deCode=fOffset;
    char *p_buf=buf;
              unsigned char *p_pdeCode=(unsigned char *)&p_deCode;
    for(p_count=0;p_count<bufSize;p_count++)
    {
        p_buf[p_count]=(g_buf[p_pdeCode[1]]+g_buf[p_pdeCode[2]]-g_buf[p_pdeCode[3]])^g_buf[p_pdeCode[0]]^p_buf[p_count];
        p_deCode+=1;
    }
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
