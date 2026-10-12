
# WinZIP.Self.Extractor.v2.2.Final.W9xNT2K.Incl.Keymaker.Source.Code-CORE

> 原文：[`www.pediy.com/kssd/pediy03/forum574.htm`](https://www.pediy.com/kssd/pediy03/forum574.htm)

```c
//
// WinZIP Self-Extractor v2.2 Final Keymaker
//
// Coded by [eGIS!/CORE].
//

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

typedef unsigned char BYTE;
typedef unsigned int WORD;
typedef unsigned long DWORD;

void main(void)
{
    char name[50], *p = name;
    WORD k1 = 0, k2 = 0, k, i;

    printf("\n ._____/\\_____/\\_____/\\_____._\n"
           " /   ___\\_   ___   \\____  \\   ___/\n"
           "/   \\      \\   \\_\\   / / /__   __\\__\n"
           "\\_________/_____\\__\\__  \\______/\n"
           "+-ds-----------+ C.O.R.E..2.K +- \\_/ -------+\n\n"
           "WinZIP Self-Extractor v2.2 Final Keymaker (c) by Egis/CORE.\n\nEnter your name: ");
    gets(name);
    strlwr(name);

    i = 1;
    while(*p) {
        if(!isalpha(*p)) {
            p++;
            continue;
        }
        k1 += (i++) * (WORD) *(p++);
    }

    p = name;
    while(*p) {
        if(!isalpha(*p)) {
            p++;
            continue;
        }
        k = *p;
        k <<= 8;
        for(i = 0; i < 8; i++) {
            if((k ^ k2) & 0x8000) {
                k2 <<= 1;
                k2 &= 0xffff;
                k2 ^= 0x1021;
            } else {
                k2 <<= 1;
                k2 &= 0xffff;
            }
            k <<= 1;
            k &= 0xffff;
        }
        p++;
    }

    sprintf(name, "%03u", k1);
    sprintf(name + 3, "%03u", k2 + 0x9dd);
    *(name + 6) = '\0';
    printf("\nYour registration code: %s\n", name);
}
```

4dd466d0e1e0cc123afbe504da2462ca
