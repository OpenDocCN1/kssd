
# GhostView 4.2 破解

> 原文：[`www.pediy.com/kssd/pediy06/pediy6559.htm`](https://www.pediy.com/kssd/pediy06/pediy6559.htm)

GhostView 4.2 破解

一直在搞我的黑白棋，很久没有破解，手都生了。![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg) 我在研究黑白棋编程时找到一堆 PS 格式的资料，于是下了这个东东来阅读，没想到还要注册。启动时有时会出现 NAG 窗口，当时忙得没顾上，现在被编程搞晕了想放一放，所以就瞄上它了。根据我的经验，这种国外的看上去比较古老的软件(尽管这是 2004 年的新版)一般都不太难破。我们来看一看：

无壳，VC7，胡乱输入用户名和前后两段序列号，记下错误信息，用 W32DASM 反一下，很容易来到下面：

```
代码:
\* Possible Reference to Dialog: DialogID\_08E8, CONTROL\_ID:08EB, ""
                          |
:0044C101 68EB080000              push 000008EB
:0044C106 8B4D08                  mov ecx, dword ptr \[ebp+08\]
:0044C109 51                     push ecx

\* Reference To: USER32.GetDlgItemInt, Ord:0112h
                          |
:0044C10A FF15705C4A00           Call dword ptr \[004A5C70\]    ;得到前面一段序列号
:0044C110 8945FC                 mov dword ptr \[ebp\-04\], eax    ;放在\[EBP\-4\]
:0044C113 6A00                   push 00000000
:0044C115 6A00                   push 00000000

\* Possible Reference to Dialog: DialogID\_08E8, CONTROL\_ID:08EC, ""
                          |
:0044C117 68EC080000             push 000008EC
:0044C11C 8B5508                 mov edx, dword ptr \[ebp+08\]
:0044C11F 52                    push edx

\* Reference To: USER32.GetDlgItemInt, Ord:0112h
                          |
:0044C120 FF15705C4A00           Call dword ptr \[004A5C70\]    ;得到后面一段序列号
:0044C126 8985F4FEFFFF           mov dword ptr \[ebp+FFFFFEF4\], eax  ;放在\[EBP+FFFFFEF4\]
:0044C12C 6800010000             push 00000100
:0044C131 8D85F8FEFFFF           lea eax, dword ptr \[ebp+FFFFFEF8\]
:0044C137 50                    push eax

\* Possible Reference to Dialog: DialogID\_08E8, CONTROL\_ID:08EA, ""
                          |
:0044C138 68EA080000             push 000008EA
:0044C13D 8B4D08                 mov ecx, dword ptr \[ebp+08\]
:0044C140 51                    push ecx

\* Reference To: USER32.GetDlgItemTextA, Ord:0113h
                          |
:0044C141 FF151C5D4A00           Call dword ptr \[004A5D1C\]    ;得到用户名
:0044C147 837DFC00               cmp dword ptr \[ebp\-04\], 00000000   
:0044C14B 7470                   je 0044C1BD        ;用户名不能为空
:0044C14D 8B55FC                 mov edx, dword ptr \[ebp\-04\]    ;前段序列号作 CALL 的参数
:0044C150 52                    push edx
:0044C151 E8B44EFBFF             call 0040100A        ;关键 CALL，跟进
:0044C156 83C404                 add esp, 00000004
:0044C159 3985F4FEFFFF           cmp dword ptr \[ebp+FFFFFEF4\], eax  ;返回值与输入的后段序列号比较
:0044C15F 755C                   jne 0044C1BD        ;不等就完蛋了

...............略

:0044C1BD 68FF000000             push 000000FF
:0044C1C2 8D85F0FDFFFF           lea eax, dword ptr \[ebp+FFFFFDF0\]
:0044C1C8 50                    push eax

\* Possible Reference to String Resource ID=00864: "Invalid Registration Name or Number"
                          |
:0044C1C9 6860030000             push 00000360
:0044C1CE E8FA58FBFF             call 00401ACD
:0044C1D3 83C40C                 add esp, 0000000C
:0044C1D6 6A30                   push 00000030

\* Possible StringData Ref from Data Obj ->"GSview"
                          |
:0044C1D8 68802E4800             push 00482E80
:0044C1DD 8D8DF0FDFFFF           lea ecx, dword ptr \[ebp+FFFFFDF0\]
:0044C1E3 51                    push ecx
:0044C1E4 8B5508                 mov edx, dword ptr \[ebp+08\]
:0044C1E7 52                    push edx

\* Reference To: USER32.MessageBoxA, Ord:01DEh
                          |
:0044C1E8 FF15505D4A00           Call dword ptr \[004A5D50\]  ;死翘翘了

进入关键的 CALL，

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040100A(U)
|
:004253A0 55                    push ebp
:004253A1 8BEC                   mov ebp, esp
:004253A3 83EC14                 sub esp, 00000014
:004253A6 C745EC08840000          mov \[ebp\-14\], 00008408  ;常数
:004253AD 8B4508                 mov eax, dword ptr \[ebp+08\]  ;传入的参数，前段序列号
:004253B0 8945F0                 mov dword ptr \[ebp\-10\], eax  ;局部变量
:004253B3 C745F800000000          mov \[ebp\-08\], 00000000  ;局部变量
:004253BA C745F400000000          mov \[ebp\-0C\], 00000000   
:004253C1 EB09                   jmp 004253CC

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00425405(U)
|
:004253C3 8B4DF4                 mov ecx, dword ptr \[ebp\-0C\]  ;\[EBP\-C\]是循环变量
:004253C6 83C101                 add ecx, 00000001
:004253C9 894DF4                 mov dword ptr \[ebp\-0C\], ecx  ;循环变量递增

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004253C1(U)
|
:004253CC 837DF420               cmp dword ptr \[ebp\-0C\], 00000020
:004253D0 7335                   jnb 00425407      ;循环 20 次
:004253D2 8B55F8                 mov edx, dword ptr \[ebp\-08\]  ;下面是核心部分
:004253D5 83E201                 and edx, 00000001
:004253D8 8955FC                 mov dword ptr \[ebp\-04\], edx
:004253DB 8B45F8                 mov eax, dword ptr \[ebp\-08\]
:004253DE D1E8                   shr eax, 1
:004253E0 8B4DF0                 mov ecx, dword ptr \[ebp\-10\]
:004253E3 83E101                 and ecx, 00000001
:004253E6 C1E10F                 shl ecx, 0F
:004253E9 03C1                   add eax, ecx
:004253EB 8945F8                 mov dword ptr \[ebp\-08\], eax
:004253EE 837DFC01               cmp dword ptr \[ebp\-04\], 00000001
:004253F2 7509                   jne 004253FD
:004253F4 8B55F8                 mov edx, dword ptr \[ebp\-08\]
:004253F7 3355EC                 xor edx, dword ptr \[ebp\-14\]
:004253FA 8955F8                 mov dword ptr \[ebp\-08\], edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004253F2(C)
|
:004253FD 8B45F0                 mov eax, dword ptr \[ebp\-10\]
:00425400 D1E8                   shr eax, 1
:00425402 8945F0                 mov dword ptr \[ebp\-10\], eax  ;上面是核心部分
:00425405 EBBC                   jmp 004253C3      ;循环结束

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004253D0(C)
|
:00425407 8B45F8                 mov eax, dword ptr \[ebp\-08\]  ;计算结果放入 EAX 传出
:0042540A 8BE5                   mov esp, ebp
:0042540C 5D                    pop ebp
:0042540D C3                    ret

```

整理一下思路，很简单了，程序根据前段的序列号经过计算得到一个值，如果与后段序列号相同就成功，用户名可以任意。那么我们写一个函数来再现这个过程就行了，也用不着费劲求逆了。那段计算乱糟糟的我就直接写在程序里了。注册机代码：(VC++6.0)

```
代码:
#include <iostream.h>

int Check(int EBP\_10)
{
  int EBP\_4=0,EBP\_8=0,EBP\_C=0;
  int EBP\_14=0x8408;
  for (int i=0;i<0x20;i++)
  {
    EBP\_4=EBP\_8 & 1;
    EBP\_8=(EBP\_8>>1)+((EBP\_10 & 1)<<0xF);
    EBP\_10>>=1;
    if (EBP\_4!=0) EBP\_8 ^= EBP\_14;
  }
  return EBP\_8;
}//基本上全按汇编码翻译过来的

void main()
{
  cout<<"The Serial Number for GhostView"<<endl;
  cout<<"One of the valid number is :";
  cout<<88888<<"-"<<Check(88888)<<endl;
  cout<<"KeyGen by RoBa  ThanQ!"<<endl;
}

```

一个可用的序列号: `88888 - 37941`

OK, Very Simple?
