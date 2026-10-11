
# Beyond Compare 1.9f

> 原文：[`www.pediy.com/kssd/pediy05/pediy50108.htm`](https://www.pediy.com/kssd/pediy05/pediy50108.htm)

目标: `Beyond Compare 1.9f`

下载: `www.scootersoftware.com`

简介: 功能强大的文件比较工具, 正是我要找的

原因: 论坛上有人介绍, 下载后发现要注册

程序出了 2.0 beta 版, 看了一下, 难度较大, `blowfish` 大哥说 1.9 版比较好用, 先用着先吧. 此次破解比较简单, 程序初运行出现提示框, 选输入注册码按键, 填入注册信息, 下 `bpx hmemcpy`, 查找后用 `bpm`, 调试过程中发现程序用了 `WritePrivateProfileStringA` 的函数, 原来程序先将注册信息写入 `INI` 文件, 再读出做运算, 错误的话要求你重新输入.

在 `windows` 目录下发现 `beyond32.ini` 文件, 打开后找到以下注册信息:

```
[Registration]
Version=1.9f
InstallCode=000091FD
Evaluation=0006A1EE
Name=Sam Von
Organization=CCG
Registration #=67676767 <---假注册码
```

既然程序已将注册信息写入 `INI` 文件, 于是用 `bpx getprivateprofilestringa` 来跟踪, 在 `4AA479` 处程序已将以上信息读入内存, 经过一段代码, 程序用以上的版本号 `1.9f` 算出 `InstallCode` 和 `Evaluation`, 可能是安装时间和使用天数, 这些不用理它, 我们主要是找到算注册码的地方, 下面是算法部分:

==Part I===========================================

之前的代码你会发现程序将我的注册名和组织名与字符串 `"Beyond Compare 1"` 连在一起变成 `"Beyond Compare 1Sam VonCCG"`, 然后变成全大写. 下面的代码将字串的每个字符做变形然后保存在原处, 其中有 3 个分支, 字符和数字分开处理, 如果是空格就去掉.

```
0167:004A8CAF  MOV      EAX,[EBP-04]  <---指向大写的字串
0167:004A8CB2  MOV      BL,[EAX+ESI-01]<---第一位"B"
0167:004A8CB6  MOV      EAX,EBX
0167:004A8CB8  ADD      AL,BF
0167:004A8CBA  SUB      AL,1A
0167:004A8CBC  JNC      004A8CDA       <---判断字符 or 数字 or 空格
0167:004A8CBE  LEA      EAX,[EBP-04]
0167:004A8CC1  CALL     004040D0       <---不用进入
0167:004A8CC6  MOV      EDX,[EBP-04]
0167:004A8CC9  XOR      EDX,EDX
0167:004A8CCB  MOV      DL,BL
0167:004A8CCD  SUB      EDX,BYTE +41
0167:004A8CD0  ADD      EDX,BYTE +0A
0167:004A8CD3  MOV      [EAX+ESI-01],DL<---字符的处理
0167:004A8CD7  INC      ESI
0167:004A8CD8  JMP      SHORT 004A8D0D
0167:004A8CDA  MOV      EAX,[EBP-04]
0167:004A8CDD  MOV      EAX,EBX
0167:004A8CDF  ADD      AL,D0
0167:004A8CE1  SUB      AL,0A
0167:004A8CE3  JNC      004A8CFE       <---判断数字 or 空格
0167:004A8CE5  LEA      EAX,[EBP-04]
0167:004A8CE8  CALL     004040D0       <---不用进入
0167:004A8CED  MOV      EDX,[EBP-04]
0167:004A8CF0  XOR      EDX,EDX
0167:004A8CF2  MOV      DL,BL
0167:004A8CF4  SUB      EDX,BYTE +30
0167:004A8CF7  MOV      [EAX+ESI-01],DL<---数字的处理
0167:004A8CFB  INC      ESI
0167:004A8CFC  JMP      SHORT 004A8D0D
0167:004A8CFE  LEA      EAX,[EBP-04]
0167:004A8D01  MOV      ECX,01
0167:004A8D06  MOV      EDX,ESI
0167:004A8D08  CALL     00404148       <---去掉空格
0167:004A8D0D  MOV      EAX,[EBP-04]
0167:004A8D10  CALL     00403F00       <---只是取原字串长度
0167:004A8D15  CMP      ESI,EAX
0167:004A8D17  JNG      004A8CAF
```

最后我们得到数据: `0B 0E 22 18 17 0D 0C 18-16 19 0A 1B 0E 01 1C 0A 16 1F 18 17 0C 0C 10` (已去掉空格), 于是 `bpm` 这个地址, 找到以下代码:

==Part II========================================

```
0167:004A180C  PUSH     EBX
0167:004A180D  PUSH     ESI
0167:004A180E  PUSH     EDI
0167:004A180F  MOV      EBX,EDX
0167:004A1811  XOR      EDX,EDX
0167:004A1813  TEST     CX,CX
0167:004A1816  JNA      004A1841
0167:004A1818  MOVZX    ESI,DX
0167:004A181B  MOVZX    ESI,BYTE [EBX+ESI]  <---数据第一位 0B
0167:004A181F  MOV      EDI,EAX  <---eax=0
0167:004A1821  SHR      EDI,18
0167:004A1824  XOR      ESI,EDI
0167:004A1826  AND      ESI,FF    <---保证 esi 为 000000XX 的形式
0167:004A182C  MOV      ESI,[ESI*4+004B8A4C]<---查表得到一个数
0167:004A1833  SHL      EAX,08
0167:004A1836  XOR      ESI,EAX
0167:004A1838  MOV      EAX,ESI
0167:004A183A  INC      EDX
0167:004A183B  DEC      ECX
0167:004A183C  TEST     CX,CX     <---循环完后 eax=B8368083
0167:004A183F  JA       004A1818
0167:004A1841  POP      EDI
0167:004A1842  POP      ESI
0167:004A1843  POP      EBX
0167:004A1844  RET
```

看到程序的算法很简单, 猜想 `eax` 的值就是注册码, 退出后将 `eax` 转换成 10 进制, 输入后提示错误, 直接将它按字符形式输入, 注册成功:-)

为了写注册机, 我们必须得到程序数据表的内容, 从 `4A182C` 处可以判断表长应为 `3F8` 位左右, `dump` 下来, 写出以下注册机.

==KeyGen===========================================

请将下面源程序保存为 `.rek` 文件, 用注册机编写器编译, 注册机编写器 1.73 测试通过!

```asm
.const

CharUpperBuffA PROTO :DWORD,:DWORD

.data

szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:ljyljx@163.com",0
szErrMess  db "输入的序列号不正确！",0
DATA       dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch
           dd 000000000h,05BD4B01Bh,0B3687D81h,0E8BCCD9Ah
           dd 066D0FB02h,03D044B19h,0D5B88683h,08E6C3698h
           dd 0CDA1F604h,09675461Fh,07EC98B85h,0251D3B9Eh
           dd 0AB710D06h,0F0A5BD1Dh,018197087h,043CDC09Ch

szDATA   db "BEYONDCOMPARE1"
         db 50 dup(0)
szXor16 db "%lX",0
szOut   db 20 dup(0)

.code

lea edi,szDATA+14

mov eax,0

mov ecx,50

rep stosb

invoke lstrcat,addr szDATA,addr hInput1
```



### 汇编代码段

```asm
invoke lstrcat, addr szDATA, addr hInput2

invoke lstrlen, addr szDATA

push eax

invoke CharUpperBuffA, addr szDATA, eax  ;<--此段连接字串并转为大写

mov eax, 20h
lea edi, szDATA
pop ecx

PRO1:
repnz scasb
jnz PRO2

mov esi, edi
push edi
dec edi
push ecx
rep movsb
mov [edi], byte ptr 0

pop ecx
pop edi
jmp PRO1       ;<--此段去掉空格

PRO2:
invoke lstrlen, addr szDATA
mov edi, eax
mov esi, 1

PRO5:
lea eax, szDATA
mov bl, [eax+esi-1]
mov ecx, ebx
add cl, 0BFh
sub cl, 1Ah
jnc PRO3

xor edx, edx
mov dl, bl
sub edx, byte ptr 41h
add edx, byte ptr 0Ah
mov [eax+esi-1], dl
inc esi
jmp PRO4

PRO3:
xor edx, edx
mov dl, bl
sub edx, byte ptr 30h
mov [eax+esi-1], dl
inc esi
jmp PRO4

PRO4:
cmp esi, edi
jng PRO5       ;<--以上是 Part I 的算法

mov ecx, edi
mov ebx, eax
xor eax, eax
xor edx, edx

PRO6:
movzx esi, dx
movzx esi, byte ptr [ebx+esi]

mov edi, eax
shr edi, 18h

xor esi, edi
and esi, 0FFh

mov esi, [esi*4+DATA]

shl eax, 08
xor esi, eax
mov eax, esi
inc edx
dec ecx
test cx, cx
ja PRO6       ;<--Part II 算法

invoke wsprintf, addr szOut, addr szXor16, eax

lea eax, szOut

-=End=-

  _/_/_/
  \_/       _/_/_/  _/_/_/  _/_/
  \_/   \_/   \_/ \_/   \_/   \_/
      \_/ \_/   \_/ \_/   \_/   \_/
_/_/_/   _/_/_/ \_/   \_/   \_/

                                   Sam.com
                                 6:19 2002-4-28
```

### C++ 代码

```cpp
#include <windows.h>
#include <tchar.h>
#include "resource.h"

// Return value must be of BOOL type
BOOL CALLBACK DlgProc(HWND, UINT, WPARAM, LPARAM);
BOOL CALLBACK AboutDlgProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI _tWinMain(HINSTANCE hInstance, HINSTANCE, LPTSTR, int) {
    DialogBoxParam(hInstance, MAKEINTRESOURCE(IDD_DIALOG1), NULL, DlgProc, (LPARAM)hInstance);
    return 0;
}

BOOL CALLBACK AboutDlgProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch (msg) {
        case WM_COMMAND:
            if (LOWORD(wParam) == IDCANCEL) {
                EndDialog(hWnd, 0);
                return TRUE;
            }
    }
    return FALSE;
}

void CalcKey(HWND);

BOOL CALLBACK DlgProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    HICON hIcon;
    HMENU hSysMenu;
    switch (msg) {
        case WM_INITDIALOG:
            hIcon = LoadIcon((HINSTANCE)lParam, MAKEINTRESOURCE(MAINICON));
            SendMessage(hWnd, WM_SETICON, TRUE, (LPARAM)hIcon);
            SendMessage(hWnd, WM_SETICON, FALSE, (LPARAM)hIcon);
            SendMessage(GetDlgItem(hWnd, IDC_NAME), EM_LIMITTEXT, 120, 0L);
            SendMessage(GetDlgItem(hWnd, IDC_ORGANIZATION), EM_LIMITTEXT, 120, 0L);
            hSysMenu = GetSystemMenu(hWnd, FALSE);
            if (hSysMenu != NULL) {
                AppendMenu(hSysMenu, MF_SEPARATOR, NULL, NULL);
                // IDM_ABOUT must be in the system command range.
                // IDM_ABOUT &  0xFFF0) == IDM_ABOUTBOX
                //  IDM_ABOUT < 0xF000
                AppendMenu(hSysMenu, MF_STRING, IDM_ABOUT, TEXT("关于..."));
            }
            // tell the default procedure to set  focus to the first non-disabled
            //  control that have a tab-stop.
            return TRUE;
        case WM_COMMAND:
            if (LOWORD(wParam) == IDCANCEL) {
                EndDialog(hWnd, 0);
                return TRUE;
            } else if (LOWORD(wParam) == IDOK) {
                CalcKey(hWnd);
                return TRUE;
            }
            break;
        case WM_SYSCOMMAND:
            if (LOWORD(wParam) == IDM_ABOUT) {
                DialogBoxParam(GetModuleHandle(NULL), MAKEINTRESOURCE(IDD_ABOUT), NULL, AboutDlgProc, 0);
                return TRUE;
            }
    }
    return FALSE;
}

void CalcKey(HWND hWnd) {
    TCHAR buf[260] = "Beyond Compare 1";
    TCHAR name[120], organization[120], key[9];
    unsigned char data[1024]; //生成的数据!!!
    int *p, esi, ecx, eax, edx;
    for (esi = 0, p = (int*)&data[0]; esi < 256; esi++) {
        for (ecx = 0, eax = esi << 24; ecx < 8; ecx++) {
            edx = 0;
            if ((eax & 0x8000000) != 0)
                edx = 0x4c11db7;
            eax += eax;
            edx ^= eax;
            eax = edx;
        }
        *p++ = eax;
    }
    GetDlgItemText(hWnd, IDC_NAME, name, sizeof(name));
    GetDlgItemText(hWnd, IDC_ORGANIZATION, organization, sizeof(organization));
    _tcsncat(buf, name, _tcslen(name));
    _tcsncat(buf, organization, _tcslen(organization));
    _tcsupr(buf);
    int len = _tcslen(buf);
    for (int i = 0; i < len; i++) {
        if ((unsigned char)(buf[i] - 'A') < 26)
            buf[i] = buf[i] - 'A' + 10;
        else if ((unsigned char)(buf[i] - '0') < 10)
            buf[i] = buf[i] - '0';
        else
            _tcsncpy(&buf[i], &buf[i+1], len - i - 1), len--, i--;
    }
    for (eax = edx = 0, ecx = len, p = (int*)data; ecx > 0; ecx--, edx++) {
        esi = buf[edx];
        esi ^= eax >> 24;
        esi &= 0xff;
        esi = p[esi];
        eax <<= 8;
        esi ^= eax;
        eax = esi;
    }
    wsprintf(key, "%08x", eax);
    SetDlgItemText(hWnd, IDC_KEY, key);
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
