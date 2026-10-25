
# Windows 进程管理大师 V3.0.1 破解分析

> 原文：[`www.pediy.com/kssd/pediy03/forum757.htm`](https://www.pediy.com/kssd/pediy03/forum757.htm)

**软件信息**

*   **作者**：396
*   **下载地址**：`http://go2.163.com/~meatbird/ProcCH.zip`
*   **功能描述**：Windows 下的进程和文件管理程序，可以查看所有在内存中运行的程序，以及它们所用到的链接库或其他文件，并可以终止任意指定的进程，同时提供对指定的程序进行监控。具有同步更新几个指定文件的功能。
*   **兼容性**：`Win98`, `Win95`, `WinNT`, `Win2000`。`V3.0.1` 版兼容各语言平台。

---

## 一、注册成功补丁

使用 `w32dasm` 黄金版打开 `Proc.exe`，寻找字符串，发现“注册成功！”：

```
* Possible StringData Ref from Code Obj ->"恭喜您"
                        |
:0048D374 B934D44800             mov ecx, 0048D434
* Possible StringData Ref from Code Obj ->"注册成功!"
```

向上看：

```
:0048D331 E87238FAFF             call 00430BA8
:0048D336 8B45F4                 mov eax, dword ptr [ebp-0C]
:0048D339 8D55F8                 lea edx, dword ptr [ebp-08]
:0048D33C E86744FFFF             call 004817A8
:0048D341 8B55F8                 mov edx, dword ptr [ebp-08]
:0048D344 58                     pop eax
:0048D345 E8526BF7FF             call 00403E9C
:0048D34A 754D                   jne 0048D399 <-------------跳转则失败！（1）
* Possible StringData Ref from Code Obj ->"windows 进程管理大师"
                        |
:0048D34C BA20D44800             mov edx, 0048D420
:0048D351 8BC3                   mov eax, ebx
:0048D353 E88038FAFF             call 00430BD8
:0048D358 33D2                   xor edx, edx
:0048D35A 8B8378040000           mov eax, dword ptr [ebx+00000478]
:0048D360 8B08                   mov ecx, dword ptr [eax]
```

**修改步骤（1）**：
将 `jne` 修改为 `nop`，即 `754d` 改为 `9090`，存盘运行后注册成功。但开始时仍显示“欢迎使用《windows 进程管理大师》！您有 30 天的试用期”对话框。

---

## 二、试用期对话框补丁

回到 `w32dasm` 黄金版寻找该字符串，找到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486E3D(C)
.
.
* Possible StringData Ref from Code Obj ->"欢迎使用《windows 进程管理大师》！您有 30 天的试?
                        ->"用期?
```

在 `00486E3D` 调用了此，转到 `00486E3D`：

```
                        |
:00486DFF BA3C764800             mov edx, 0048763C
:00486E04 A174194900             mov eax, dword ptr [00491974]
:00486E09 8B30                   mov esi, dword ptr [eax]
:00486E0B FF5610                 call [esi+10]
:00486E0E A28C084900             mov byte ptr [0049088C], al
:00486E13 8A158C084900           mov dl, byte ptr [0049088C]
:00486E19 8B83B8040000           mov eax, dword ptr [ebx+000004B8]
:00486E1F 8B08                   mov ecx, dword ptr [eax]
:00486E21 FF91B8000000           call dword ptr [ecx+000000B8]
:00486E27 8D55D0                 lea edx, dword ptr [ebp-30]
:00486E2A 8B45F8                 mov eax, dword ptr [ebp-08]
:00486E2D E876A9FFFF             call 004817A8
:00486E32 8B45D0                 mov eax, dword ptr [ebp-30]
:00486E35 8B55F4                 mov edx, dword ptr [ebp-0C]
:00486E38 E85FD0F7FF             call 00403E9C
:00486E3D 0F85B8000000           jne 00486EFB 〈-------------（2）
* Possible StringData Ref from Code Obj ->"windows 进程管理大师"
```

这是标准的风飘雪说的经典比较组合：

```
move eax ?????
move edx ?????
call ?????
jne ?????
```

在此下断点可找到注册码。

**修改步骤（2）**：
将 `0f85` 修改为 `0f84`，存盘运行对话框消失，运行正常。

---

## 三、注册失败补丁

把时间往后调一年，又出现请求注册，注册不成功，出现"错误的注册信息，请重试！"。

回到 `w32dasm` 黄金版寻找该字符串，找到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048E193(C)
|
:0048E26C 6A00                   push 00000000
:0048E26E 668B0D20E34800         mov cx, word ptr [0048E320]
:0048E275 B203                   mov dl, 03
* Possible StringData Ref from Code Obj ->"错误的注册信息，请重试！"
```

在 `0048E193` 调用了此，转到 `0048E193`：

```
:0048E13D 8D4000                 lea eax, dword ptr [eax+00]
:0048E140 55                     push ebp
:0048E141 8BEC                   mov ebp, esp
:0048E143 33C9                   xor ecx, ecx
:0048E145 51                     push ecx
:0048E146 51                     push ecx
:0048E147 51                     push ecx
:0048E148 51                     push ecx
:0048E149 51                     push ecx
:0048E14A 51                     push ecx
:0048E14B 53                     push ebx
:0048E14D 56                     push esi
:0048E14E 57                     push edi
:0048E14F 8BD8                   mov ebx, eax
:0048E151 33C0                   xor eax, eax
:0048E153 55                     push ebp
:0048E154 68BEE24800             push 0048E2BE
:0048E159 64FF30                 push dword ptr fs:[eax]
:0048E15C 648920                 mov dword ptr fs:[eax], esp
:0048E15F 8D55F8                 lea edx, dword ptr [ebp-08]
:0048E162 8B83DC020000           mov eax, dword ptr [ebx+000002DC]
:0048E168 E83B2AFAFF             call 00430BA8
:0048E16D 8B45F8                 mov eax, dword ptr [ebp-08]
:0048E170 8D55FC                 lea edx, dword ptr [ebp-04]
:0048E173 E83036FFFF             call 004817A8
:0048E178 8B45FC                 mov eax, dword ptr [ebp-04]
:0048E17B 50                     push eax
:0048E17C 8D55F4                 lea edx, dword ptr [ebp-0C]
:0048E17F 8B83E0020000           mov eax, dword ptr [ebx+000002E0]
:0048E185 E81E2AFAFF             call 00430BA8
:0048E18A 8B55F4                 mov edx, dword ptr [ebp-0C]
:0048E18D 58                     pop eax
:0048E18E E8095DF7FF             call 00403E9C
:0048E193 0F85D3000000           jne 0048E26C <-------------（3）
:0048E199 8D55EC                 lea edx, dword ptr [ebp-14]
:0048E19C A1700B4900             mov eax, dword ptr [00490B70]
:0048E1A1 8B00                   mov eax, dword ptr [eax]
:0048E1A3 E83812FCFF             call 0044F3E0
:0048E1A8 8B45EC                 mov eax, dword ptr [ebp-14]
:0048E1AB 8D55F0                 lea edx, dword ptr [ebp-10]
:0048E1AE E809A7F7FF             call 004088BC
:0048E1B3 8D45F0                 lea eax, dword ptr [ebp-10]
```

**修改步骤（3）**：
将 `0f85` 修改为 `0f84`，存盘运行对话框消失，运行正常。一切 OK！收工！

---

## 四、注册算法分析（未完成）

**来源**：共享软件注册中心
`http://www.shareware.net.cn/dl.asp?id={0CB8E23A-FD70-40F2-B57F-7B50E2CD6F05}`

**规则推测**：
`注册码 = 字母 + 数字`

*   **数字部分**：
    逐个计算 `(姓名 ascii 码 × 10 + 8)`，求和后转为十进制。
    *   示例：姓名 `a`，注册码 `A1560`。计算过程：`61×10+8=618` ==> `1560`
    *   示例：姓名 `fpx[CCG]`，注册码 `CCG11776`

*   **字母部分**：
    有时不出现，规则未找出，不知哪位能帮忙完成。

**字母处理部分反汇编**：

```
016F:00481760  PUSH    EBX
016F:00481761  PUSH    ESI
016F:00481762  MOV     [EBP-01],AL   // 将字母放入
016F:00481765  MOV     BYTE [EBP-03],02
016F:00481769  MOV     BYTE [EBP-02],01  // 变量初始化
016F:0048176D  MOV     CL,[EBP-01]     // 读出字母
016F:00481770  DEC     ECX
016F:00481771  SUB     CL,02
016F:00481774  JC      004817A0
016F:00481776  INC     ECX
016F:00481777  MOV     BL,02
016F:00481779  XOR     EAX,EAX
016F:0048177B  MOV     AL,[EBP-01]
016F:0048177E  XOR     EDX,EDX
016F:00481780  MOV     DL,BL
016F:00481782  MOV     ESI,EDX
016F:00481784  XOR     EDX,EDX
016F:00481786  DIV     ESI
016F:00481788  TEST    EDX,EDX
016F:0048178A  JNZ     0048178F
016F:0048178C  INC     BYTE [EBP-03]
016F:0048178F  CMP     BYTE [EBP-03],02
016F:00481793  JNA     0048179B        // 关键跳转
016F:00481795  MOV     BYTE [EBP-02],00 // 当[ebp-02]为 0,则该字母不被选中(也就是说不显示）
016F:00481799  JMP     SHORT 004817A0
016F:0048179B  INC     EBX
016F:0048179C  DEC     CL
016F:0048179E  JNZ     00481779
016F:004817A0  MOV     AL,[EBP-02]     // return [ebp-02]
016F:004817A3  POP     ESI
```

**作者联系信息**
396
`junpo@fashida.com`
主页：`http://www.zmdinfo.net`



```
016F:004817A4  POP     EBX

016F:004817A5  POP     ECX

016F:004817A6  POP     EBP

016F:004817A7  RET
```

```c
#include "stdio.h"

void zs(a)
{
    int i;
    for(i=2;i<a;i++) if(a%i==0) break;
    if(i==a) printf("%c",a-32);
}

main()
{
    char string[81];
    long d=0;
    int i,c;
    printf("******* PROC ******\nThis keygen is maked by Pretender\nPlease input your name : ");
    gets(string);
    printf("Your name is : ");
    puts(string);
    printf("Your Register code is : ");
    for(i=0;(c=string[i])!='\0';i++)
    {
        if(c<0) c+=256;
        zs(c);
        c=c*16;
        d=c+8+d;
    }
    d+=(string[0]-string[i-1])*16;
    printf("%d\n",d);
    printf("Welcome to WWW.CRACKNOW.COM\n");
}
```

伪哥的注册机程序有点小问题,现贴出修正版(`TC2.0`)

`***`处有改动(需考虑大小写),请伪哥指正

```c
#include "stdio.h"

void zs(a)
{
    int i;
    for(i=2;a;i++) if(a%i==0) break;                    ****
    if(i==a) {
        if((i>=64)&&(i<=90)) printf("%c",a);            ****
        else printf("%c",a-32);
    }                                                    ****
}

main()
{
    char string[81];
    long d=0;
    int i,c;
    printf("******* PROC ******\nThis keygen is maked by Pretender\nPlease input your name : ");
    gets(string);
    printf("Your name is : ");
    puts(string);
    printf("Your Register code is : ");
    for(i=0;(c=string[i])!='\0';i++)
    {
        if(c<0) c+=256;
        zs(c);
        c=c*16;
        d=c+8+d;
    }
    d+=(string[0]-string[i-1])*16;
    printf("%d\n",d);
    printf("Welcome to WWW.CRACKNOW.COM\n");
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
