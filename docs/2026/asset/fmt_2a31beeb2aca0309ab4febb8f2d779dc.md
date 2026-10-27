
```asm
00401128  |.  F2:AE         repne   scas byte ptr es:[edi]

0040112A  |.  F7D1          not     ecx

0040112C  |.  49            dec     ecx

0040112D  |.  85C9          test    ecx, ecx

0040112F  |.  7E 1F         jle     short 00401150

00401131  |.  53            push    ebx                              ;  edx=0xFFFFFFFF

00401132  |>  8BC2          /mov     eax, edx

00401134  |.  33DB          |xor     ebx, ebx

00401136  |.  8A1E          |mov     bl, byte ptr [esi]              ;  注册名给ebx

00401138  |.  25 FF000000   |and     eax, 0FF                        ;  eax%=256;

0040113D  |.  33C3          |xor     eax, ebx                        ;  eax^=ebx

0040113F  |.  C1EA 08       |shr     edx, 8                          ;  edx/=256

00401142  |.  8B0485 208540>|mov     eax, dword ptr [eax\*4+408520]   ;  eax=a[eax]

00401149  |.  33D0          |xor     edx, eax                        ;  edx^=eax

0040114B  |.  46            |inc     esi                             ;这个运算主要是求EDX..关注哦

0040114C  |.  49            |dec     ecx

0040114D  |.^ 75 E3         \jnz     short 00401132

0040114F  |.  5B            pop     ebx

00401150  |>  33C9          xor     ecx, ecx

00401152  |.  8D4424 08     lea     eax, dword ptr [esp+8]

00401156  |.  894C24 09     mov     dword ptr [esp+9], ecx

0040115A  |.  6A 10         push    10

0040115C  |.  F7D2          not     edx
```

-----------------------------------------------

这是上面的算法分析,哈哈

```c
d=0xFFFFFFFF;

     for(i=0;i<ss.length();i++)

         {

          b=int(ss[i]);

          a=d;

          a%=256;

          a^=b;

          a=A[a];

          d/=256;

          d^=a;

         }

     d^=0xFFFFFFFF;
```

---------------------------

-----------------------------------

下面就是把得到的值字符串话,并且把小写的字母,改成大写..代码忽略..

------------------------------------------

```asm
0040115E  |.  894C24 11     mov     dword ptr [esp+11], ecx

00401162  |.  50            push    eax

00401163  |.  52            push    edx

00401164  |.  C64424 14 00  mov     byte ptr [esp+14], 0

00401169  |.  884C24 1D     mov     byte ptr [esp+1D], cl

0040116D  |.  E8 24380000   call    00404996                         ;  就是变成字符串,字母小写

00401172  |.  8D7C24 14     lea     edi, dword ptr [esp+14]

00401176  |.  83C9 FF       or      ecx, FFFFFFFF

00401179  |.  33C0          xor     eax, eax

0040117B  |.  83C4 0C       add     esp, 0C

0040117E  |.  33F6          xor     esi, esi

00401180  |.  F2:AE         repne   scas byte ptr es:[edi]           ;  计算大小

00401182  |.  F7D1          not     ecx

00401184  |.  49            dec     ecx

00401185  |.  74 25         je      short 004011AC

00401187  |>  0FBE4C34 08   /movsx   ecx, byte ptr [esp+esi+8]

0040118C  |.  51            |push    ecx

0040118D  |.  E8 4E020000   |call    004013E0                        ;  就是改下大小写,

00401192  |.  884434 0C     |mov     byte ptr [esp+esi+C], al

00401196  |.  83C4 04       |add     esp, 4

00401199  |.  8D7C24 08     |lea     edi, dword ptr [esp+8]

0040119D  |.  83C9 FF       |or      ecx, FFFFFFFF

004011A0  |.  33C0          |xor     eax, eax

004011A2  |.  46            |inc     esi

004011A3  |.  F2:AE         |repne   scas byte ptr es:[edi]

004011A5  |.  F7D1          |not     ecx

004011A7  |.  49            |dec     ecx

004011A8  |.  3BF1          |cmp     esi, ecx

004011AA  |.^ 72 DB         \jb      short 00401187

004011AC  |>  8B4424 1C     mov     eax, dword ptr [esp+1C]

004011B0  |.  8D5424 08     lea     edx, dword ptr [esp+8]

004011B4  |.  52            push    edx                              ; /String2

004011B5  |.  50            push    eax                              ; |String1

004011B6  |.  FF15 00504000 call    dword ptr [<&KERNEL32.lstrcpyA>] ; \lstrcpyA

004011BC  |.  5F            pop     edi

004011BD  |.  5E            pop     esi

004011BE  |.  83C4 0C       add     esp, 0C

004011C1  \.  C3            retn
```

这是我的代码..大小写懒得改变了..

```cpp
#include <iostream>

#include <string>

using namespace std;

int main(int argc,char\*argv[])

{

    unsigned int d,A[1600],b,a;

        char s[300];

        string ss;

        cin>>s;

        ss=s;

        for(int i=0;i<0x100;i++)

        {

           d=i;

           for(int j=0;j<8;j++)

            if(d%2) {

                     d/=2;

                     d^=0xEDB88320;

                        }

            else{

                      d/=2;

                        }

         A[i]=d;

         }

     d=0xFFFFFFFF;

     for(i=0;i<ss.length();i++)

         {

          b=int(ss[i]);

          a=d;

          a%=256;

          a^=b;

          a=A[a];

          d/=256;

          d^=a;

         }

     d^=0xFFFFFFFF;

         cout<<"输入时字母要大写"<<endl;

         cout<<hex<<d<<endl;

         return 0;

}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [CRC32CrackMe.rar](javascript:void(0);)

  （12.43kb，16次下载）
