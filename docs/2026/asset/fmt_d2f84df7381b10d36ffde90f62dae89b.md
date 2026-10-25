
# Konvertor 3.03的注册码算法模块的分析

（7千字）

## 软件简介

Konvertor 是一个图片、音乐、文字及影音格式文件的转换程序，它总共可以转换135种文件格式，包含25种音乐格式、4种影音格式、105种图片格式及1种文字格式。

## 破解工具

`SOFTICE`，`W32DASM`

## 分析

在软件注册窗口输入以下信息：

```
First Name: alpha
Last Name: ckck
Registration Code: 98765432
```

用`BPX GETDLGITEMTEXTA`下断，拦截后用`F11`跳出函数，继续跟踪，来到：

```asm
\* Possible Reference to String Resource ID=00020: "&Cancel"
    |
:0044BFA5 B914000000  mov ecx, 00000014
:0044BFAA 33C0        xor eax, eax
:0044BFAC 8DBC24BC000000  lea edi, dword ptr [esp+000000BC]　<<-------输入的注册码的首地址->EDI
\* Possible Reference to String Resource ID=00128: "Choose the page type, the image position and its size. (posi"
    |
:0044BFB3 6880000000  push 00000080
:0044BFB8 F3          repz
:0044BFB9 AB          stosd
:0044BFBA 8D8C24C0000000  lea ecx, dword ptr [esp+000000C0]
:0044BFC1 51          push ecx
:0044BFC2 687D050000  push 0000057D
:0044BFC7 55          push ebp
:0044BFC8 FFD6        call esi
:0044BFCA 85C0        test eax, eax
:0044BFCC 7538        jne 0044C006          <<---------跳
    。
    。来到：
    。
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:0044BFCC(C)
 |
:0044C006 8D542414    lea edx, dword ptr [esp+14]　　　<<--------输入的名字的首地址->EDX
:0044C00A 8D442468    lea eax, dword ptr [esp+68]　　　<<--------输入的姓的首地址->EAX
:0044C00E 52          push edx
\* Possible StringData Ref from Data Obj ->"Konvertor"
    |
:0044C00F 68B8524700  push 004752B8　　　　　　　　<<---------字符串“Konvertor”入栈
:0044C014 50          push eax
:0044C015 8D8C24B8060000  lea ecx, dword ptr [esp+000006B8]
\* Possible StringData Ref from Data Obj ->"%s%s%s"
    |
:0044C01C 683CE14700  push 0047E13C
:0044C021 51          push ecx
:0044C022 FFD3        call ebx　　　　　　　<<--------此函数将输入的姓名和字符串“Konvertor”以“姓＋Konvertor＋名”的形式连接起来组成新的字符串，在此例中即：ckckKonvertoralpha
:0044C024 8DBC24C0060000  lea edi, dword ptr [esp+000006C0]　<<------新的字符串的首地址->EDI
:0044C02B 83C9FF      or ecx, FFFFFFFF
:0044C02E 33C0        xor eax, eax
:0044C030 83C414      add esp, 00000014
:0044C033 F2          repnz
:0044C034 AE          scasb
:0044C035 F7D1        not ecx
:0044C037 2BF9        sub edi, ecx
:0044C039 8D942494020000  lea edx, dword ptr [esp+00000294]
:0044C040 8BC1        mov eax, ecx
:0044C042 8BF7        mov esi, edi
:0044C044 8BFA        mov edi, edx
:0044C046 C1E902      shr ecx, 02
:0044C049 F3          repz
:0044C04A A5          movsd
:0044C04B 8BC8        mov ecx, eax
:0044C04D 8D842494020000  lea eax, dword ptr [esp+00000294]  <<-----新的字符串的首地址->EAX
:0044C054 83E103      and ecx, 00000003
:0044C057 F3          repz
:0044C058 A4          movsb
:0044C059 8A8C2494020000  mov cl, byte ptr [esp+00000294]  <<-----新字串的第一个字符->CL
:0044C060 84C9        test cl, cl　　<<------测试上一步的操作是否成功
:0044C062 741F        je 0044C083
<<-----不成功则跳走
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:0044C081(C)
 |
:0044C064 80385F      cmp byte ptr [eax], 5F　<<-----比较字符是否为“＿”
:0044C067 7503        jne 0044C06C　　　　　<<------不同就跳
:0044C069 C60020      mov byte ptr [eax], 20
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:0044C067(C)
 |
:0044C06C 0FBE08      movsx ecx, byte ptr [eax]　<<------新串的第一个字符->ECX
:0044C06F 334C2410    xor ecx, dword ptr [esp+10]
<<------ECX XOR FFFFFFFF->ECX
:0044C073 81F1CE9A5713  xor ecx, 13579ACE
:0044C079 40          inc eax
:0044C07A 894C2410    mov dword ptr [esp+10], ecx
<<------保存ECX
:0044C07E 803800      cmp byte ptr [eax], 00　　　　<<-----比较新串的下一位是否为0
:0044C081 75E1        jne 0044C064
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:0044C062(C)
 |
:0044C083 8D9424BC000000  lea edx, dword ptr [esp+000000BC]　<<----输入的注册码的首地址->EDX
:0044C08A 52          push edx
:0044C08B E8AE8B0100  call 00464C3E　　　<<-----将输入的字符串形式的注册码转为十进制形式->EAX
:0044C090 8B4C2414    mov ecx, dword ptr [esp+14]  <<------新串的运算结果->ECX
:0044C094 83C404      add esp, 00000004
:0044C097 81F1F0BD6824  xor ecx, 2468BDF0
:0044C09D 3BC1        cmp eax, ecx
:0044C09F 742E        je 0044C0CF
如果不同则来到：
\* Possible StringData Ref from Data Obj ->"Enregistrement"　　<<-----注册失败对话框的标题
    |
:0044C0A3 6838244800  push 00482438
\* Possible StringData Ref from Data Obj ->"Le code est incorrect\n Vous devez "
  ->"entrer vos nom et prnom puis "
  ->"le code qui vous a t?dlivr?
    |
:0044C0AD 55          push ebp
\* Reference To: USER32.MessageBoxA, Ord:01C3h
    |
:0044C0AE FF150C164700  Call dword ptr [0047160C]
```

这个软件的注册码算法很简单，它就是把输入的姓名与字符串“Konvertor”连起来进行运算，第一次跟踪的时候，我在`00448C08B`的`CALL`中转了好一阵子才发现它不过是将用户输入的注册码转为十进制形式：）

## 附：注册机源程序

```cpp
#include<iostream.h>
#include<stdlib.h>
#include<string.h>

void main()
{
    unsigned long int tempString;
    int temp=-1,LengthOfString;
    char buff1[80],buff2[80],YourFirstName[80],YourLastName[1000],SoftName[10]="Konvertor";

    cout<<"\nKeyGen for Konvertor 3.03 (Made by alpha)\n\n";
    cout<<"Please input your first name:";
    while(cin.getline(buff1,80))
    {
        strcpy(YourFirstName,buff1);
        break;
    }
    cout<<"Please input your last name:";
    while(cin.getline(buff2,80))
    {
        strcpy(YourLastName,buff2);
        break;
    }
    strcat(YourLastName,SoftName);
    strcat(YourLastName,YourFirstName);
    LengthOfString=strlen(YourLastName);
    for(int i=0;i<LengthOfString;i++)
    {
        tempString=YourLastName[i];
        tempString^=temp;
        tempString^=324508366;
        temp=tempString;
    }
    tempString^=610844144;
    cout<<"\nYour Registration Code is:"<<tempString<<endl;
    cout<<"Press Enter key to terminate...";
    if(cin.get())
        exit(0);
}
```

源程序在`VC++6.0`下编译通过

`CrAcKeD BY alphakk(alpha)`
