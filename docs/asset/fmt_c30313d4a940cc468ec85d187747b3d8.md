
# EbookMaker 1.21

> 原文：[`www.pediy.com/kssd/pediy05/pediy50086.htm`](https://www.pediy.com/kssd/pediy05/pediy50086.htm)

`EbookMaker` `1.21`

`OCG`

下面是关键的地方。

```
\==============OCG=================================OCG=================================OCG

\* Possible StringData Ref from Data Obj ->"1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"

                  |
:004016A5 BEE5E54B00                mov esi, 004BE5E5
:004016AA F3                        repz
:004016AB A5                        movsd
:004016AC C745D40A000000            mov [ebp-2C], 0000000A
:004016B3 8B45D4                    mov eax, dword ptr [ebp-2C]
:004016B6 40                        inc eax
:004016B7 50                        push eax
:004016B8 E84BD00900                call 0049E708
:004016BD 59                        pop ecx
:004016BE 8945D0                    mov dword ptr [ebp-30], eax
:004016C1 33DB                      xor ebx, ebx
:004016C3 8B45D0                    mov eax, dword ptr [ebp-30]
:004016C6 8BF8                      mov edi, eax
:004016C8 3B5DD4                    cmp ebx, dword ptr [ebp-2C]
:004016CB 7D61                      jge 0040172E

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040172C(C)

|
:004016CD 837D0C00                  cmp dword ptr [ebp+0C], 00000000
:004016D1 7408                      je 004016DB
:004016D3 8B450C                    mov eax, dword ptr [ebp+0C]         ; This is your inputed name
:004016D6 8B50FC                    mov edx, dword ptr [eax-04]         ; lenth your name
:004016D9 EB02                      jmp 004016DD

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004016D1(C)

|
:004016DB 33D2                      xor edx, edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004016D9(U)

|
:004016DD 8BC3                      mov eax, ebx                        ; start clac real sn!!(ebx is the times of loop)
:004016DF 8BCA                      mov ecx, edx
:004016E1 99                        cdq
:004016E2 F7F9                      idiv ecx
:004016E4 8BF2                      mov esi, edx
:004016E6 46                        inc esi
:004016E7 56                        push esi
:004016E8 8D450C                    lea eax, dword ptr [ebp+0C]
:004016EB 50                        push eax
:004016EC E8CF790A00                call 004A90C0
:004016F1 83C408                    add esp, 00000008
:004016F4 8D450C                    lea eax, dword ptr [ebp+0C]
:004016F7 E8887D0A00                call 004A9484
:004016FC 03750C                    add esi, dword ptr [ebp+0C]
:004016FF 8BD3                      mov edx, ebx
:00401701 4E                        dec esi
:00401702 81E20F000080              and edx, 8000000F
:00401708 0FBE06                    movsx eax, byte ptr [esi]           ;name
:0040170B 7905                      jns 00401712
:0040170D 4A                        dec edx
:0040170E 83CAF0                    or edx, FFFFFFF0
:00401711 42                        inc edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040170B(C)

|
:00401712 0FBE4C15BC                movsx ecx, byte ptr [ebp+edx-44]    ;table2("EBOOKMAKER")
:00401717 03C1                      add eax, ecx
:00401719 B924000000                mov ecx, 00000024
:0040171E 99                        cdq
:0040171F F7F9                      idiv ecx
:00401721 8A441598                  mov al, byte ptr [ebp+edx-68]       ;table1
:00401725 8807                      mov byte ptr [edi], al
:00401727 43                        inc ebx
:00401728 47                        inc edi
:00401729 3B5DD4                    cmp ebx, dword ptr [ebp-2C]         ; the end??if yes then end this loop
:0040172C 7C9F                      jl 004016CD

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004016CB(C)

|
:0040172E 8B45D0                    mov eax, dword ptr [ebp-30]         ;可以看见 sn!!
:00401731 8B55D4                    mov edx, dword ptr [ebp-2C]

this is my keygen

\==============OCG=================================OCG=================================OCG
```

`this` `is` `my` `keygen`

```
\==============OCG=================================OCG=================================OCG

#include <iostream.h>
#include <math.h>
#include <string.h>

int main( )
{
    char name[99],f[10],sn[0xa];
    int i=0,l=0;

    static unsigned char table2[]={"EBOOKMAKER"},table1[]={"1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"};

    cout<<endl<<"               EBook Maker keygen by [OCG]"<<endl;
    cout<<endl<<"Please input your name:";
    cin>>name;

    cout<<"Your registration code:";
    l=strlen(name);
    for (i=0;i<0xA;i++)
        sn[i]=table1[(table2[i]+name[i%l])%0x24];

    cout<<sn;
    return 0;
}

\==============OCG=================================OCG=================================OCG

keygen end!

bye:)
```

`main()`

```
{
    char n[30],s[10];
    int i;

    char k1[]="EBOOKMAKER";
    char k2[]="1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    clrscr();
    printf("EBookMaker v1.21 keygen by XF[BCG]\n\n");
    printf("Enter your name: ");
    scanf("%s",n);

    for (i=0;i<10;i++)
        s[i]=k2[(k1[i]+n[i%strlen(n)])%0x24];

    s[10]=0;
    printf("Your RegCode is: %s\n",s);
    getch();
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
