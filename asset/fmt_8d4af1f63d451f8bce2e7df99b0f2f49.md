
```
0065F18E    E8 1D5CDAFF     call 1_.00404DB0

0065F193    FF75 DC        push dword ptr ss:[ebp-24]        ;ASCII "9238"

0065F196    68 0CF26500    push 1_.0065F20C

0065F19B    8D45 D8        lea eax,dword ptr ss:[ebp-28]

0065F19E    50             push eax

0065F19F    B9 04000000    mov ecx,4

0065F1A4    BA 0D000000    mov edx,0D

0065F1A9    8B45 FC        mov eax,dword ptr ss:[ebp-4]

0065F1AC    E8 FF5BDAFF    call 1_.00404DB0

0065F1B1    FF75 D8        push dword ptr ss:[ebp-28]        ;ASCII "8222"

0065F1B4    8BC3           mov eax,ebx

....

0066007E    8B55 FC        mov edx,dword ptr ss:[ebp-4]       ;ASCII "NEBS-4088-C747-9238-8222"组合成申请码的格式

00660081    8B83 14030000  mov eax,dword ptr ds:[ebx+314]

00660087    E8 481EE1FF    call 1_.00471ED4

0066008C    E8 1BF7FFFF    call 1_.0065F7AC

00660091    8BD0           mov edx,eax

00660093    8B83 FC020000  mov eax,dword ptr ds:[ebx+2FC]

00660099    8B08           mov ecx,dword ptr ds:[eax]

0066009B    FF51 64        call dword ptr ds:[ecx+64]

0066009E    33C0           xor eax,eax

006600A0    5A             pop edx

006600A1    59             pop ecx

006600A2    59             pop ecx
```

算法就是上面的大概呢，太累了，不想找下去了，哪位请继续一下![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

很久没发帖了，心痒痒，所以![](img/710b5589904d0d2bf58a67748fca40e1.jpg)![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
