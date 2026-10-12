
# MSN 密码监控器

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-3.htm`](https://www.pediy.com/kssd/pediy08/pediy8-3.htm)

## 软件信息

软件下载：`http://www.ngnsss.com/main/single_down.asp?id=4`

- **名称**：`MSN` 密码监控器
- **作者**：`ngnsss`
- **语言**：中文

## 简介

本程序为绿色软件，无需安装。使用方法：点击 `MSNPwd.exe` 启动程序。程序启动后，点击“开始监控”，监控程序在后台运行。然后所有的登陆过的 `MSN` 密码都记录在 `c:\msnpass.txt` 文件里。以后监控程序每次开机会自动运行。然后，你已经可以退出 `MSNPwd` 这个配置程序了。`c:\msnpass.txt` 属性为隐藏。注册版本后台运行，没有界面。未注册版本每 3 分钟弹出注册对话框，如果觉得烦，你或者注册，或者使用配置程序结束监控。

想知道你的小孩整天在网上做什么吗，想知道你的情人整天在和谁聊天吗？注册本软件，得到他(她)的密码，你就可以看看它的聊天记录，还有可以看看他(她)到底整天和谁聊天了。

## 逆向分析

### 关键代码段 1

```asm
004025A1     .   C74424 14 000000>mov dword ptr ss:[esp+14],0
004025A9     .   51             push ecx
004025AA     .   8D4C24 08      lea ecx,dword ptr ss:[esp+8]
004025AE     .   E8 82C30000    call MSNPwd.0040E935                     ;  注册码
004025B3     .   51             push ecx
004025B4     .   8D5424 08      lea edx,dword ptr ss:[esp+8]            ;  注册码
004025B8     .   8BCC           mov ecx,esp
004025BA     .   896424 0C      mov dword ptr ss:[esp+C],esp
004025BE     .   52             push edx
004025BF     .   E8 3DC00000    call MSNPwd.0040E601
004025C4     .   E8 77FEFFFF    call MSNPwd.00402440                    ;  关键
004025C9     .   83C4 04        add esp,4
004025CC     .   85C0           test eax,eax
004025CE     .   6A 00          push 0
004025D0     .   68 BCA14100    push MSNPwd.0041A1BC                   ;  ngnsss
004025D5     .   74 15          je short MSNPwd.004025EC
004025D7     .   68 C4A14100    push MSNPwd.0041A1C4                   ;  注册成功
004025DC     .   8BCE           mov ecx,esi
004025DE     .   E8 ADA80000    call MSNPwd.0040CE90
004025E3     .   8BCE           mov ecx,esi
004025E5     .   E8 A2D40000    call MSNPwd.0040FA8C
\======================================================================================
```

### 关键代码段 2 (Key Function `MSNPwd.00402440`)

```asm
00402440    /$   6A FF            push -1
00402442    |.   68 88324100      push MSNPwd.00413288                   ;  SE 句柄安装
00402447    |.   64:A1 00000000   mov eax,dword ptr fs:[0]
0040244D    |.   50             push eax
0040244E    |.   64:8925 00000000 mov dword ptr fs:[0],esp
00402455    |.   83EC 18        sub esp,18
00402458    |.   53             push ebx
00402459    |.   8B4C24 2C      mov ecx,dword ptr ss:[esp+2C]
0040245D    |.   33C0           xor eax,eax
0040245F    |.   894424 05      mov dword ptr ss:[esp+5],eax
00402463    |.   33DB           xor ebx,ebx
00402465    |.   66:894424 09   mov word ptr ss:[esp+9],ax
0040246A    |.   895C24 24      mov dword ptr ss:[esp+24],ebx
0040246E    |.   884424 0B      mov byte ptr ss:[esp+B],al
00402472    |.   8B41 F8        mov eax,dword ptr ds:[ecx-8]
00402475    |.   83F8 10        cmp eax,10                               ;  注册码长度是否等于 16 位
00402478    |.   885C24 04      mov byte ptr ss:[esp+4],bl
0040247C    |.   0F8C C0000000  jl MSNPwd.00402542                       ;  必须大于等于 16 位
00402482    |.   56             push esi
00402483    |.   68 04010000    push 104
00402488    |.   8D4C24 34      lea ecx,dword ptr ss:[esp+34]
0040248C    |.   E8 DFC50000    call MSNPwd.0040EA70                     ;  注册码
00402491    |.   8B10           mov edx,dword ptr ds:[eax]               ;  注册码 ASCII  分四组
00402493    |.   33F6           xor esi,esi                              ;  ESI=0
00402495    |.   895424 10      mov dword ptr ss:[esp+10],edx
00402499    |.   8B48 04        mov ecx,dword ptr ds:[eax+4]            ;  注册码 ASCII
0040249C    |.   894C24 14      mov dword ptr ss:[esp+14],ecx
004024A0    |.   8B50 08        mov edx,dword ptr ds:[eax+8]            ;  注册码 ASCII
004024A3    |.   895424 18      mov dword ptr ss:[esp+18],edx
004024A7    |.   8B40 0C        mov eax,dword ptr ds:[eax+C]            ;  注册码 ASCII
004024AA    |.   894424 1C      mov dword ptr ss:[esp+1C],eax
004024AE    |>   8A4C34 10      /mov cl,byte ptr ss:[esp+esi+10]
004024B2    |.   51             |push ecx
004024B3    |.   E8 68FFFFFF    |call MSNPwd.00402420                   ;  CODE[I]-0X30
004024B8    |.   83C4 04        |add esp,4
004024BB    |.   884434 10      |mov byte ptr ss:[esp+esi+10],al
004024BF    |.   46             |inc esi
004024C0    |.   83FE 10        |cmp esi,10
004024C3    |.^  7C E9          \jl short MSNPwd.004024AE               ;  转换成 16 进制数
004024C5    |.   33C0           xor eax,eax
004024C7    |.   8D4C24 10      lea ecx,dword ptr ss:[esp+10]           ;  注册码 16 进制数 HCODE[]
004024CB    |.   5E             pop esi
004024CC    |>   8A51 01        /mov dl,byte ptr ds:[ecx+1]            ;  DL=HCODE[I+1]
004024CF    |.   8A19           |mov bl,byte ptr ds:[ecx]               ;  BL=HCODE[I]
004024D1    |.   C0E2 04        |shl dl,4                               ;  DL SHL 4
004024D4    |.   02D3           |add dl,bl                               ;  DL =DL+BL
004024D6    |.   83C1 02        |add ecx,2                               ;  ECX+=2
004024D9    |.   885404 04      |mov byte ptr ss:[esp+eax+4],dl
004024DD    |.   40             |inc eax
004024DE    |.   83F8 08        |cmp eax,8
004024E1    |.^  7C E9          \jl short MSNPwd.004024CC               ;  SCODE[]=HCODE[I]数组每两个置换生成字符串
004024E3    |.   8A4424 07      mov al,byte ptr ss:[esp+7]              ;  AL=SCODE[3]
004024E7    |.   8A5C24 04      mov bl,byte ptr ss:[esp+4]              ;  BL=SCODE[0]
004024EB    |.   8A4C24 0B      mov cl,byte ptr ss:[esp+B]              ;  CL=SCODE[7]
004024EF    |.   8A5424 05      mov dl,byte ptr ss:[esp+5]              ;  DL=SCODE[1]
004024F3    |.   32C3           xor al,bl                                ;  AL=SCODE[3]^SCODE[0]=4D=M
004024F5    |.   8A5C24 06      mov bl,byte ptr ss:[esp+6]              ;  BL=SCODE[2]
004024F9    |.   32CA           xor cl,dl                                ;  CL=SCODE[7]^SCODE[1]=6E=n
004024FB    |.   8A5424 09      mov dl,byte ptr ss:[esp+9]              ;  DL=SCODE[5]
004024FF    |.   32D3           xor dl,bl                                ;  DL=SCODE[5]^SCODE[2]=4E=N
00402501    |.   8A5C24 08      mov bl,byte ptr ss:[esp+8]              ;  BL=SCODE[4]
00402505    |.   325C24 0A      xor bl,byte ptr ss:[esp+A]              ;  BL=SCODE[4]^SCODE[6]=19=|
00402509    |.   3C 4D          cmp al,4D                                ;  AL 必须等于 4D
0040250B    |.   75 35          jnz short MSNPwd.00402542
0040250D    |.   80F9 6E        cmp cl,6E                                ;  CL 必须等于 6E
00402510    |.   75 30          jnz short MSNPwd.00402542
00402512    |.   80FA 4E        cmp dl,4E                                ;  DL 必须等于 4E
00402515    |.   75 2B          jnz short MSNPwd.00402542
00402517    |.   80FB 19        cmp bl,19                                ;  BL 必须等于 19
0040251A    |.   75 26          jnz short MSNPwd.00402542
0040251C    |.   8D4C24 2C      lea ecx,dword ptr ss:[esp+2C]
00402520    |.   C74424 24 FFFFFF>mov dword ptr ss:[esp+24],-1
00402528    |.   E8 5FC30000    call MSNPwd.0040E88C
0040252D    |.   B8 01000000      mov eax,1
///////////////////////////////////////////////////////////////////////////////////////////////////
```

## 总结

注册码和用户名没关系，只有逆推了。大致如下：

1. 注册码必须 16 位。
2. 注册码的 `ASCII` 值每位减 `0X30`，然后每 2 位需要置换。
3. 置换后的值必须满足以下等式。

```c
/*



# 汇编代码分析

```
004024F3      xor al,bl                ;  AL=SCODE[3]^SCODE[0]=0x4D k0=M
004024F5      mov bl,byte ptr ss:[esp+6]  ;  BL=SCODE[2]
004024F9      xor cl,dl                ;  CL=SCODE[7]^SCODE[1]=0x6E k1=n
004024FB      mov dl,byte ptr ss:[esp+9]  ;  DL=SCODE[5]
004024FF      xor dl,bl                ;  DL=SCODE[2]^SCODE[5]=0x4E k2 =N
00402501      mov bl,byte ptr ss:[esp+8]  ;  BL=SCODE[4]
00402505      xor bl,byte ptr ss:[esp+A]     ;  BL=SCODE[4]^SCODE[6]=0x19 k3
```

## C++ 逆推算法

用 VC 编了个逆推算法，不知道什么地方有误哪位大虾帮看一下吧。

```cpp
void main()
{
  int hcode[8],key[]={0x4d,0x6e,0x4e,0x19},i,j, Arr[16],tmp;
  char ccode[16]={0},scode[8]={0};

j1:
    srand(time(0));
    for (i=0; i<16; i++)
    {
        Arr[i] = rand()%9;
    }

  i=0;
  for (j=0; j<8; i=i+2,j++)
    {
    tmp=Arr[i+1]*0x10+Arr[i];
        hcode[j] = tmp;
    }

    for (i=0; i<16; i++)
    {
        printf("%4x", Arr[i]);
    }

  cout<<endl;
     for (i=0; i<8; i++)
    {
        scode[i]=hcode[i] ;
    }

   hcode[0]=0x4d^hcode[3];
   hcode[7]=0x6e^hcode[1];
   hcode[2]=0x4e^hcode[5];
   hcode[4]=0x19^hcode[6];

/*
004024F3      xor al,bl                ;  AL=SCODE[3]^SCODE[0]=0x4D k0=M
004024F5      mov bl,byte ptr ss:[esp+6]  ;  BL=SCODE[2]
004024F9      xor cl,dl                ;  CL=SCODE[7]^SCODE[1]=0x6E k1=n
004024FB      mov dl,byte ptr ss:[esp+9]  ;  DL=SCODE[5]
004024FF      xor dl,bl                ;  DL=SCODE[2]^SCODE[5]=0x4E k2 =N
00402501      mov bl,byte ptr ss:[esp+8]  ;  BL=SCODE[4]
00402505      xor bl,byte ptr ss:[esp+A]     ;  BL=SCODE[4]^SCODE[6]=0x19 k3
*/

  for (i=0; i<8; i++)
    {
        printf("%4x",  hcode[i] );
    }

  cout<<endl;

  for (i=0; i<8; i++)
  {
     hcode[i]=hcode[i]/0x10+hcode[i]%0x10*0x10;
  }

  for (i=0; i<8; i++)
  {
    printf("%4x",  hcode[i] );
  }

    cout<<endl;
   i=0,j=0;

  for (j=0; j<8; i=i+2,j++)
    {
    tmp=hcode[j]/0x10;
        ccode[i] = tmp+48;
    tmp=hcode[j]%0x10;
        ccode[i+1] = tmp+48;
    }

for (i=0; i<8; i++)
  {
    if (!(ccode[i]>='0'&&  ccode[i]<='9'))
      goto j1;
  }

  cout<<ccode<<endl;
}
```

## 补充代码示例

我来添加点外力:

```cpp
#include "stdio.h"
#include "math.h"

void main()
{
  int i, Arr[8];
    srand(time(0));
    Arr[0] = rand()%255;
    Arr[1] = rand()%255;
    Arr[2] = rand()%255;
    Arr[4] = rand()%255;
    Arr[3] = Arr[0]^0xd4;
    Arr[5] = Arr[2]^0xe4;
    Arr[6] = Arr[4]^0x91;
    Arr[7] = Arr[1]^0xe6;
    for (i=0; i<8; i++)
    {
        printf("%02X", Arr[i]);
    }
}
```

不知道为什么要穷举
