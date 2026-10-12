
# 万能五笔

> 原文：[`www.pediy.com/kssd/pediy03/forum1126.htm`](https://www.pediy.com/kssd/pediy03/forum1126.htm)

万能五笔的破解:

下载：`http://www.tt98.net.cn/wnwb.htm`

输入注册信息并下断点后来这里:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CB8D(C)
  |
:0048CB74 8B15346D4900              mov edx, dword ptr [00496D34]
:0048CB7A 0FBFC1                    movsx eax, cx
:0048CB7D 8A1402                    mov dl, byte ptr [edx+eax]
:0048CB80 80FA3F                    cmp dl, 3F
:0048CB83 7406                      je 0048CB8B
:0048CB85 3854041C                  cmp byte ptr [esp+eax+1C], dl //在这里比较注册码前四个字符是不是"tt98"
:0048CB89 7504                      jne 0048CB8F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CBEE(C)
  |
:0048CBCF A1BC6C4900                mov eax, dword ptr [00496CBC]
:0048CBD4 0FBFFE                    movsx edi, si
:0048CBD7 8A0438                    mov al, byte ptr [eax+edi]
:0048CBDA 3C3F                      cmp al, 3F
:0048CBDC 740B                      je 0048CBE9
:0048CBDE 0FBFD9                    movsx ebx, cx
:0048CBE1 03DF                      add ebx, edi
:0048CBE3 38441C1C                  cmp byte ptr [esp+ebx+1C], al //在这里比较注册码后四个字符是不是"0615"
:0048CBE7 7507                      jne 0048CBF0
:0048CC76 668B3D2E6D4900            mov di, word ptr [00496D2E]
:0048CC7D 8B15B06C4900              mov edx, dword ptr [00496CB0] //6364
:0048CC83 66C1EF08                  shr di, 08
:0048CC87 668B0D2E6D4900            mov cx, word ptr [00496D2E]
:0048CC8E 6681E1FF00                and cx, 00FF
:0048CC93 E8F8FAFFFF                call 0048C790 //关键计算被调用２次
:0048CC98 03F0                      add esi, eax
:0048CC9A 6685FF                    test di, di
:0048CC9D 750A                      jne 0048CCA9
:0048CC9F 8B15B46C4900              mov edx, dword ptr [00496CB4]
:0048CCA5 8BCF                      mov ecx, edi
:0048CCA7 EB0B                      jmp 0048CCB4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CC9D(C)
  |
:0048CCA9 668BCF                    mov cx, di
:0048CCAC 8B15B46C4900              mov edx, dword ptr [00496CB4]
:0048CCB2 6641                      inc cx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CCA7(U)
  |
:0048CCB4 E8D7FAFFFF                call 0048C790 //再计算一次
:0048CCB9 8BC8                      mov ecx, eax
:0048CCBB 85C9                      test ecx, ecx
:0048CCBD 7507                      jne 0048CCC6
:0048CCBF BDFBFFFFFF                mov ebp, FFFFFFFB
:0048CCC4 EB36                      jmp 0048CCFC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CCBD(C)
  |
:0048CCC6 8BC6                      mov eax, esi
:0048CCC8 99                        cdq
:0048CCC9 F7F9                      idiv ecx
:0048CCCB 8BEA                      mov ebp, edx
:0048CCCD EB2D                     jmp 0048CCFC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048CC74(C)
  |
:0048CCCF 66833D286D490002          cmp word ptr [00496D28], 0002
:0048CCD7 7523                      jne 0048CCFC
:0048CCD9 668B152E6D4900            mov dx, word ptr [00496D2E]
:0048CCE0 A1B46C4900                mov eax, dword ptr [00496CB4]
:0048CCE5 50                        push eax
:0048CCE6 8B0DB06C4900              mov ecx, dword ptr [00496CB0]
:0048CCEC 51                        push ecx
:0048CCED 8B0DC46A4900              mov ecx, dword ptr [00496AC4]
:0048CCF3 E828FBFFFF                call 0048C820
:0048CCF8 8BE8                      mov ebp, eax //? eax 正确的数字
:0048CCFA 2BEE                      sub ebp, esi //? esi 假数字

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0048CCC4(U), :0048CCCD(U), :0048CCD7(C)
  |
:0048CCFC 85ED                      test ebp, ebp
:0048CCFE 7429                      je 0048CD29 //if jump good boy:>
```

核心计算一共被调用２次，看看

```
:0048C790 53                        push ebx
:0048C791 56                        push esi
:0048C792 668BD9                    mov bx, cx
:0048C795 57                        push edi
:0048C796 55                        push ebp
:0048C797 8BF2                      mov esi, edx
:0048C799 85F6                      test esi, esi
:0048C79B 7475                      je 0048C812
:0048C79D 803E00                    cmp byte ptr [esi], 00
:0048C7A0 7470                      je 0048C812
:0048C7A2 8BFE                      mov edi, esi
:0048C7A4 B9FFFFFFFF                mov ecx, FFFFFFFF
:0048C7A9 2BC0                      sub eax, eax
:0048C7AB F2                       repnz
:0048C7AC AE                       scasb
:0048C7AD F7D1                     not ecx
:0048C7AF 49                       dec ecx //strlen(name)
:0048C7B0 6685DB                   test bx, bx
:0048C7B3 7444                     je 0048C7F9
:0048C7B5 6683FB01                 cmp bx, 0001
:0048C7B9 743E                     je 0048C7F9
:0048C7BB 0FB7FB                   movzx edi, bx
:0048C7BE 8BC7                     mov eax, edi //eax=0x63
:0048C7C0 99                       cdq
:0048C7C1 F7F9                     idiv ecx
:0048C7C3 0FBE0416                 movsx eax, byte ptr [esi+edx]
:0048C7C7 0FAFC2                   imul eax, edx
:0048C7CA 0FAFC7                   imul eax, edi
:0048C7CD 03C1                     add eax, ecx
:0048C7CF 33D2                     xor edx, edx
:0048C7D1 85C9                     test ecx, ecx
:0048C7D3 7E19                     jle 0048C7EE
:0048C7D5 8BD9                     mov ebx, ecx
:0048C7D7 2BDF                     sub ebx, edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048C7EC(C)
  |
:0048C7D9 0FBE3C16                 movsx edi, byte ptr [esi+edx]
:0048C7DD 8BEB                     mov ebp, ebx
:0048C7DF 2BEA                     sub ebp, edx
:0048C7E1 42                       inc edx
:0048C7E2 83C56F                    add ebp, 0000006F
:0048C7E5 0FAFFD                   imul edi, ebp
:0048C7E8 03C7                     add eax, edi
:0048C7EA 3BCA                     cmp ecx, edx
:0048C7EC 7FEB                     jg 0048C7D9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0048C7D3(C)
  |
:0048C7EE 85C0                     test eax, eax
:0048C7F0 7D25                     jge 0048C817
:0048C7F2 F7D8                     neg eax
:0048C7F4 5D                       pop ebp
:0048C7F5 5F                       pop edi
:0048C7F6 5E                       pop esi
:0048C7F7 5B                       pop ebx
:0048C7F8 C3                       ret
```

注册机如下：

```c
#include<stdio.h>

main()
{
    char name[60];
    long n, EAX, ESI, EDI, i;

    printf("your name:");
    gets(name);

    n = 0X63 % strlen(name);
    EAX = name[n] * n * 0X63 + strlen(name);
    i = 0;

    do {
        EDI = (strlen(name) - 0X63 + 0X6f - i) * name[i++];
        EAX = EAX + EDI;
    } while (i < strlen(name));

    if (EAX < 0) EAX = ~EAX;
    ESI = EAX;

    n = 0X65 % strlen(name);
    EAX = name[n] * n * 0X65 + strlen(name);
    i = 0;

    do {
        EDI = (strlen(name) - 0X65 + 0X6f - i) * name[i++];
        EAX = EAX + EDI;
    } while (i < strlen(name));

    if (EAX < 0) EAX = ~EAX;

    EAX = ESI + EAX + 0X13311;

    printf("Regcode is:%ld\n", EAX);
    printf("press any key to exit!\n");
    getch();
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
