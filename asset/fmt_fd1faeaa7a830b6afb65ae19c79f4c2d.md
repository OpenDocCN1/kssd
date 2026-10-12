
### UltraEdit-32 注册码验证算法分析

`.text:0041890C j ... sub_4185B8+32`

`.text:0041890C xor eax, eax`

`.text:0041890E`

`.text:0041890E loc_41890E: ; CODE XREF: sub_4185B8+364`

```
.text:0041890E        pop    edi
.text:0041890F        pop    ebx
.text:00418910        leave
.text:00418911        retn
```

别以为这样找到的注册码就能顺利过关。当你输入注册码后，重新启动后发现注册成功。关闭后再启动，`UltraEdit-32` 又会弹出注册窗口，再看看 `UltraEdit-32` 目录，`Uedit32.reg` 文件被删除了。这说明还有其他的地方检查注册码。于是下 `bpx deletefilea`，再注册一次，找到：

```
016F:004486E1 MOV EAX,[005933F4]
016F:004486E6 PUSH DWORD [005933D8]
016F:004486EC MOV [EBP-1C],EAX
016F:004486EF CALL 004FF590
016F:004486F4 POP ECX
016F:004486F5 CMP EAX,BYTE +0F
016F:004486F8 POP ECX
016F:004486F9 JC NEAR 004487F6
016F:004486FF CMP [0059DD3C],EDI
016F:00448705 JNZ NEAR 004487F6
016F:0044870B CMP [0059DD40],EDI
016F:00448711 JZ NEAR 004487F6
016F:00448717 MOVZX EDI,BYTE [EBP-1C]
016F:00448717 MOVZX EDI,BYTE [EBP-1C]
016F:0044871B MOV EAX,EDI ; eax=byte prt [005933f4]
016F:0044871D PUSH BYTE +09
016F:0044871F CDQ
016F:00448720 POP EBX
016F:00448721 MOV ECX,[005933D8]
016F:00448727 IDIV EBX
016F:00448729 MOVSX EAX,BYTE [ECX+0E] //取注册码的第 15 位
016F:0044872D ADD EDX,BYTE +41 ; EDX=(eax mod 9h)+41h
016F:00448730 CMP EAX,EDX ; EDX 与注册码的第 15 位进行比较
016F:00448732 JNZ 0044874D ; 若不等,Game Over
016F:00448734 MOV EAX,EDI
016F:00448736 PUSH BYTE +07
016F:00448738 CDQ
016F:00448739 POP EDI
016F:0044873A MOV CL,[ECX+16] //取出注册码的第 23 位
016F:0044873D IDIV EDI
016F:0044873F MOVSX EAX,CL
016F:00448742 ADD EDX,BYTE +41 ; EDX=(eax mod 7h)+41h
016F:00448745 CMP EAX,EDX ; EDX 与注册码的第 23 位进行比较
016F:00448747 JZ NEAR 004487F3 ; 若不等,Game Over
016F:0044874D AND DWORD [0059DD40],BYTE +00
016F:00448754 CALL 00533F9D
016F:00448759 MOV EAX,[EAX+04]
016F:0044875C LEA ECX,[EBP-10]
016F:0044875F PUSH DWORD [EAX+8C]
016F:00448765 CALL 00519DBA
016F:0044876A MOV EAX,[EBP-10]
016F:0044876D LEA ECX,[EBP-10]
016F:00448770 MOV BYTE [EBP-04],01
016F:00448774 MOV EAX,[EAX-08]
016F:00448777 ADD EAX,BYTE -03
016F:0044877A PUSH EAX
016F:0044877B LEA EAX,[EBP-24]
016F:0044877E PUSH EAX
016F:0044877F CALL 00511C19
016F:00448784 PUSH DWORD 00587018
016F:00448789 PUSH EAX
016F:0044878A LEA EAX,[EBP-1C]
016F:0044878D MOV BYTE [EBP-04],02
016F:00448791 PUSH EAX
016F:00448792 CALL 00519FE1
016F:00448797 PUSH EAX
016F:00448798 LEA ECX,[EBP-10]
016F:0044879B MOV BYTE [EBP-04],03
016F:0044879F CALL 00519E85
016F:004487A4 LEA ECX,[EBP-1C]
016F:004487A7 MOV BYTE [EBP-04],02
016F:004487AB CALL 00519D4C
016F:004487B0 LEA ECX,[EBP-24]
016F:004487B3 MOV BYTE [EBP-04],01
016F:004487B7 CALL 00519D4C
016F:004487BC PUSH DWORD 80
016F:004487C1 PUSH DWORD [EBP-10]
016F:004487C4 CALL KERNEL32!SetFileAttributesA
016F:004487CA PUSH DWORD [EBP-10]
016F:004487CD CALL KERNEL32!DeleteFileA
```

还记得在计算注册码时有个值没用到吗？对了，就是 `[005933F4]`。它在这里用来检验注册码的有效性。将 `[005933F4]` 中的低 8 位（二进制）分别与 `09h` 进行异或运算后加上 `41h`，再与我们输入的注册码的第 15 位和第 23 位进行比较，有一位不对，就删掉 `Uedit32.reg`。真够狡猾的。

至此整个算法分析完了。这是我加入 `CCG` 后的第一篇文章，让各位高手见笑了。

### 注册信息示例

```
User Name: wolverine[CCG]
Authorization: V7H7L-M2O6S-J1F3H-I8D9B
```

`wolverine[CCG]`
`2003-5-17`

### 附录

1. **数据表**

```
586c38
{
    00586C28, 00586B30 ;
    0058CA38, 0058D288 ;=>我称这个为目录
}
```

**码表**

```
00586C28 {
    D8,00,FE,00,
    8E,00,66,00,
    2F,00,DE,00,
    43,00,A5,00
}

00586B30 {
    D3,00,1F,00,
    78,00,6F,00,
    38,00,8F,00,
    65,00,5A,00
}

0058CA38 {
    00,10,00,F1,
    00,A5,00,2F,
    00,1F,00,CB,
    00,98,00,F3
}

0058D288 {
    00,EF,00,E0,
    00,73,00,78,
    00,73,00,34,
    00,77,00,33
}
```

2. **注册机代码**

（至于注册机……）

```cpp
//-----------kg.cpp--------------------
#include "windows.h"
#include "Kg.h"

#pragma comment(linker,"/OPT:NOWIN98")
#pragma comment(linker,"/ENTRY:WinMain")
#pragma comment(linker, "/SECTION:Mini,")
#pragma comment(linker, "/MERGE:.data=Mini")
#pragma comment(linker, "/MERGE:.text=Mini")
#pragma comment(linker, "/MERGE:.rdata=Mini")

HWND hWnd;
HINSTANCE hInst;
unsigned int iSum = 0, iStrLen = 0;
int iCheckNum = 0;
unsigned char szName[20]; //名字的最大长度
unsigned char szSN[0x60];
unsigned char M[0x40];

unsigned char T[4][0x10] = {
    // t1[0x10] =
    {
        0xD8,0x00,0xFE,0x00,
        0x8E,0x00,0x66,0x00,
        0x2F,0x00,0xDE,0x00,
        0x43,0x00,0xA5,0x00
    },
    // t2[0x10] =
    {
        0xD3,0x00,0x1F,0x00,
        0x78,0x00,0x6F,0x00,
        0x38,0x00,0x8F,0x00,
        0x65,0x00,0x5A,0x00
    },
    // t3[0x10] =
    {
        0x00,0x10,0x00,0xF1,
        0x00,0xA5,0x00,0x2F,
        0x00,0x1F,0x00,0xCB,
        0x00,0x98,0x00,0xF3
    },
    // t4[0x10] =
    {
        0x00,0xEF,0x00,0xE0,
        0x00,0x73,0x00,0x78,
        0x00,0x73,0x00,0x34,
        0x00,0x77,0x00,0x33
    }
};

void FillM()
{
    int i;
    for (i = 0; i < 0x3d; i++) M[i] = (i - 1) * 3;
}

void CvtName() //1.转换名字应该是 strlen-2 次循环。
{
    int i, l, c = 2, t;
    l = iStrLen;
    strcpy((char *)M, (const char *)szName);
    M[l] = 0;
    for (i = 0; i < l - 2; i++) {
        t = M[i] * M[i + 2] - M[i + 1] - c;
        c += 2;
        iSum = iSum + t + 3;
    }
}

void CvtSN()
{
    unsigned char i;
    i = '0';
    szSN[1] = i;
    szSN[0xa] = i;
    szSN[0xc] = i;
    i = iCheckNum & 0xff;
    szSN[0xe] = (i % 9) + 'A';
    szSN[0x16] = (i % 7) + 'A';
    szSN[0x17] = '\0';
}

void KeyGen()
{
    unsigned i, j, t, q, p, s, high, low;
    iSum = 0;
    iCheckNum = 0;
    FillM();
    CvtName();
    s = iSum % 0x100;
    for (i = 0, j = 0; i < 0x3c; i++) {
        high = i % 4;
        low = (i % 0x20) >> 1;
        q = T[high][low];
        high = ((i + s) ^ q) % 0x3b;
        t = M[high];
        high = ((t + i) % 0x12) % 4;
        low = ((t + i) % 0x12 /*%0x20*/) >> 1;
        p = T[high][low]; //mod 12,then mod 20???
        M[i] = ((t ^ p) + t) ^ (M[i + 1] << 1);
        szSN[j++] = ((i % 5) & 1) ? M[i] % 10 + '0' : M[i] % 26 + 'A';
        if (i % 5 == 4) szSN[j++] = '-';
        if (i) {
            if ((i < iStrLen) && (j < iStrLen)) iCheckNum += szName[i - 1] * szName[j];
        }
    }
    CvtSN();
}

LRESULT CALLBACK MainProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
        case WM_COMMAND:
            switch (wParam)
            {
```
