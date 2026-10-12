
# 用 C++编写注册机的一点技巧

> 原文：[`www.pediy.com/kssd/pediy06/pediy6850.htm`](https://www.pediy.com/kssd/pediy06/pediy6850.htm)

下面是加密算法；

```
0040105D    895424 14                 mov dword ptr ss:[esp+14],edx
00401061    894424 20                 mov dword ptr ss:[esp+20],eax
00401065    897C24 1C                 mov dword ptr ss:[esp+1C],edi
00401069    8DB426 00000000           lea esi,dword ptr ds:[esi]
00401070    8B6C24 38                 mov ebp,dword ptr ss:[esp+38]
00401074    8B4424 14                 mov eax,dword ptr ss:[esp+14]                 ; 三次循环，十二位注册码，注册码加密之后再进行比较
00401078    8D6C05 00                 lea ebp,dword ptr ss:[ebp+eax]
0040107C    0FBE75 00                 movsx esi,byte ptr ss:[ebp]                   ; 第一位
00401080    56                        push esi
00401081    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
00401085    E8 16010000               call crackme_.004011A0
0040108A    884424 10                 mov byte ptr ss:[esp+10],al
0040108E    0FBE75 01                 movsx esi,byte ptr ss:[ebp+1]                 ; 第二位
00401092    56                        push esi
00401093    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
00401097    E8 04010000               call crackme_.004011A0
0040109C    884424 11                 mov byte ptr ss:[esp+11],al
004010A0    0FBE75 02                 movsx esi,byte ptr ss:[ebp+2]                 ; 注册码第三位
004010A4    56                        push esi
004010A5    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
004010A9    E8 F2000000               call crackme_.004011A0
004010AE    884424 12                 mov byte ptr ss:[esp+12],al
004010B2    0FBE6D 03                 movsx ebp,byte ptr ss:[ebp+3]                 ; 第四位
004010B6    55                        push ebp
004010B7    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
004010BB    E8 E0000000               call crackme_.004011A0
004010C0    0FBEC0                    movsx eax,al
004010C3    894424 18                 mov dword ptr ss:[esp+18],eax
004010C7    0FBE4424 11               movsx eax,byte ptr ss:[esp+11]               ; 二位
004010CC    0FB64C24 10               movzx ecx,byte ptr ss:[esp+10]               ; 一位
004010D1    03C9                      add ecx,ecx
004010D3    03C9                      add ecx,ecx
004010D5    8BD0                      mov edx,eax
004010D7    C1FA 04                   sar edx,4                                 ; 二位右移位
004010DA    0BCA                      or ecx,edx
004010DC    0FB7D3                    movzx edx,bx
004010DF    8B6C24 3C                 mov ebp,dword ptr ss:[esp+3C]
004010E3    880C2A                    mov byte ptr ds:[edx+ebp],cl              ; 移位这后的数－－－－
004010E6    0FBE4C24 12               movsx ecx,byte ptr ss:[esp+12]             ; 三位
004010EB    C1E0 04                   shl eax,4                                 ; 二位
004010EE    8BD1                      mov edx,ecx
004010F0    C1FA 02                   sar edx,2
004010F3    0BC2                      or eax,edx
004010F5    8D53 01                   lea edx,dword ptr ds:[ebx+1]
004010F8    0FB7D2                    movzx edx,dx
004010FB    88042A                    mov byte ptr ds:[edx+ebp],al              ; －－－－－
004010FE    C1E1 06                   shl ecx,6
00401101    0B4C24 18                 or ecx,dword ptr ss:[esp+18]              ; 四位
00401105    8D7B 02                   lea edi,dword ptr ds:[ebx+2]
00401108    0FB7C7                    movzx eax,di
0040110B    880C28                    mov byte ptr ds:[eax+ebp],cl              ; －－－－－
0040110E    8B4424 14                 mov eax,dword ptr ss:[esp+14]
00401112    83C0 04                   add eax,4
00401115    0FB7C0                    movzx eax,ax
00401118    894424 14                 mov dword ptr ss:[esp+14],eax
0040111C    83C3 03                   add ebx,3
0040111F    0FB77424 40               movzx esi,word ptr ss:[esp+40]
00401124    3BC6                      cmp eax,esi
00401126   0F8C 44FFFFFF             jl crackme_.00401070
0040112C    8B5424 18                 mov edx,dword ptr ss:[esp+18]
00401130    8B4424 20                 mov eax,dword ptr ss:[esp+20]             ; Stack ss:[0012F858]=00000002
00401134    8B4C24 38                 mov ecx,dword ptr ss:[esp+38]
00401138    0FB77424 40               movzx esi,word ptr ss:[esp+40]
0040113D    885424 13                 mov byte ptr ss:[esp+13],dl
\------------------------
004011A0    0FBE4424 04               movsx eax,byte ptr ss:[esp+4]
004011A5    83F8 41                   cmp eax,41
004011A8    7C 0E                     jl short crackme_.004011B8
004011AA    83F8 5A                   cmp eax,5A
004011AD    7F 09                     jg short crackme_.004011B8
004011AF    8D50 BF                   lea edx,dword ptr ds:[eax\-41]
004011B2    0FB6C2                    movzx eax,dl
004011B5    C2 0400                   retn 4
004011B8    83F8 61                   cmp eax,61
004011BB    7C 0E                     jl short crackme_.004011CB
004011BD    83F8 7A                   cmp eax,7A
004011C0    7F 09                     jg short crackme_.004011CB
004011C2    8D50 B9                   lea edx,dword ptr ds:[eax\-47]
004011C5    0FB6C2                    movzx eax,dl
004011C8    C2 0400                   retn 4
004011CB    83F8 30                   cmp eax,30
004011CE    7C 0E                     jl short crackme_.004011DE
004011D0    83F8 39                   cmp eax,39
004011D3    7F 09                     jg short crackme_.004011DE
004011D5    8D50 04                   lea edx,dword ptr ds:[eax+4]
004011D8    0FB6C2                    movzx eax,dl
004011DB    C2 0400                   retn 4
004011DE    83F8 2B                   cmp eax,2B
004011E1    74 0A                     je short crackme_.004011ED
004011E3    83F8 2F                   cmp eax,2F
004011E6    74 0D                     je short crackme_.004011F5
004011E8    33C0                      xor eax,eax
004011EA    C2 0400                   retn 4
004011ED    B8 3E000000               mov eax,3E
004011F2    C2 0400                   retn 4
004011F5    B8 3F000000               mov eax,3F
004011FA    C2 0400                   retn 4
004011FD    8D76 00                   lea esi,dword ptr ds:[esi]
00401200    8B5424 04                 mov edx,dword ptr ss:[esp+4]
00401204    8915 EC714100             mov dword ptr ds:[4171EC],edx
0040120A    B8 65000000               mov eax,65
0040120F    50                        push eax
00401210    68 30124000               push crackme_.00401230
00401215    6A 00                     push 0
00401217    50                        push eax
00401218    52                        push edx
00401219    FF15 18314100             call dword ptr ds:[413118]
0040121F    33C0                      xor eax,eax
00401221    C2 1000                   retn 10
00401224    8DB6 00000000             lea esi,dword ptr ds:[esi]
\----------------------------------->下面是 c++程序
```

```cpp
#include<iostream.h>

#include<fstream.h>

int change(char b)

{

     if(b >= 0x41 && b <= 0x5a)  //0-->19
       b=b-0x41;

     else if(b >= 0x61 && b <= 0x7a)//1a--->33
       b=b-0x47;

     else if(b >= 0x30 && b <= 0x39)//34--->43       3e  3f
       b=b+0x4;

     else if(b==0x2b )
      b=0x3e;

     else if(b==0x2f)
      b=0x3f;

     else
       b=0;

     return b;

}

void main()

{

  char str[12];

   int i,j;

   int change(char b);

   int mima[9];

   int mi[9]={0xa8,0xff,0xbc,0x7c,0xe7,0x89,0x94,0x98,0x00};

   int eax,ecx,edx;//就是这里了，

  for(i=0; i<12; i++)

  {

    cout<<"str["<<i<<"\]=";

    cin>>str[i];

  }

  for (i=0, j=0; i<12; i+=4, j+=3)

  {

     str[i]=change(str[i]);

     str[i+1]=change(str[i+1]);

     str[i+2]=change(str[i+2]);

     str[i+3]=change(str[i+3]);

     cout<<"    "<<str[i]<<endl;

     cout<<"    "<<str[i+1]<<endl;
```



```
      cout<<"    "<<str[i+2]<<endl;  

      cout<<"    "<<str[i+3]<<endl;  

        eax=str[i+1];  

      ecx=str[i];  

      ecx*=4;  

      edx=eax;  

      edx=edx>>4;  

      ecx=ecx|edx;  

      ecx=ecx&0xff;  

       mima[j]=ecx;  

      cout<<mima[j]<<endl;  

      ecx=str[i+2];  

      eax=eax<<4;  

      edx=ecx;  

      edx=edx>>2;  

      eax=eax|edx;  

      eax=eax&0xff;//相当于 mov eax,al  

      mima[j+1]=eax;  

      cout<<mima[j+1]<<endl;  

      ecx=ecx<<6;  

      ecx=ecx|str[i+3];  

      ecx=ecx&0xff;  

      mima[j+2]=ecx;  

      cout<<mima[j+2]<<endl;      

   }  

  ofstream fd("try.txt");  

  for(j=0; j<9; j++)  

  {  

    mima[j]=mima[j]^mi[j];  

    fd.setf(ios::hex);  

    fd<<mima[j]<<",";  

  }  

}  
```

进行这种计算，我通常用 `unsigned long`（或是 `DWORD`，固定是 32 位）

虽然 `int` 在这里也是 32 位，但总觉得 `dword` 比较安全

我記得 `int` 的寬度是以系統 `address bus` 的寬度來決定，不規定是 32 位元。 `C++` 我不肯定，如果是 `C` 就是這樣

对比一下,看来偶的编程风格就是烂啊....

/*               __Design By Hume[AfO]__

             (C)CopyRight SyT0z Corporation.

________________________________________________________________________________*/

```
#include <windows.h>  

#include <stdio.h>  

#include <tchar.h>  

#include <iostream>  

using namespace std;  

void print_sub_key(unsigned int key,unsigned int rbase)  

{  

    unsigned char bf[64];  

    unsigned char *p;  

    key+=rbase;  

    sprintf((char *)bf,"%lX",key);  

    p=bf;  

    for(int j=0;j<strlen((char*)bf);j++,p++) {  

        if(*p==0x38) *p+=0xA;  

        else if(*p==0x42) *p+=0xF6;  

    }  

    printf("%s\n",bf);  

}  

void __cdecl main(int argc,char *argv[])  

{  

    char name[1024];   

    unsigned char *p;  

    unsigned int tkey,tmgc;  

    cout<<"Winxxx version xxxx kegen by Hume"<<endl;  

    cout<<"your name:";  

    cin.getline(name,1024);  

    cout<<"your usable keys are:"<<endl;  

    strupr(name);  

    p=(unsigned char *)name;  

    tkey= 0x47694C;  

    for(int i=0;i<strlen(name);i++,p++) {  

        if(i%14==0) tmgc=0x27;  

        tkey+=((unsigned char)(*p))*tmgc;  

        if((i+3)%14==0)  tmgc*=7;  

        else tmgc*=3;  

    }  

    if(tkey==0xB8DCDD26) {  

        cout<<"sorry this name can't generate keys!"<<endl;  

    }  

    print_sub_key(tkey,0);  

    print_sub_key(tkey,0x14051948);  

    print_sub_key(tkey,0x17061954);  

    print_sub_key(tkey,0x10051981);  

    print_sub_key(tkey,0x4011995);    

    print_sub_key(tkey,0x2061997);  

}
```

在 `vc6` 里对于有符号整数移位和无符号整数移位是不一样的.

编个程序如下:

```
int main(int argc, char* argv[])  

{  

  long a = 55;  

  unsigned long b =0xaa;  

  a = a << 2;  

  b = b << 2;  

  a = a >> 2;  

  b = b >> 2;  

  return 0;  

}
```

再看看反汇编后的代码:

```
12:       a = a << 2;  

00401036   mov         eax,dword ptr [ebp-4]  

00401039   shl         eax,2  

0040103C   mov         dword ptr [ebp-4],eax  

13:       b = b << 2;  

0040103F   mov         ecx,dword ptr [ebp-8]  

00401042   shl         ecx,2  

00401045   mov         dword ptr [ebp-8],ecx  

14:  

15:       a = a >> 2;  

00401048   mov         edx,dword ptr [ebp-4]  

0040104B   sar         edx,2  

0040104E   mov         dword ptr [ebp-4],edx  

16:       b = b >> 2;  

00401051   mov         eax,dword ptr [ebp-8]  

00401054   shr         eax,2  

00401057   mov         dword ptr [ebp-8],eax
```

可以看出,左移时都是 `shl`,而右移时有符号数是 `sar`,而无符号数是 `shr`.

`4dd466d0e1e0cc123afbe504da2462ca`
