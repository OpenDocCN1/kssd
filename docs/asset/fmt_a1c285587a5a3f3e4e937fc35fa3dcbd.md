
```assembly
00000350: 3B 45 CC         cmp         eax,dword ptr [ebp-34h]  
00000353: 75 02            jne         00000357  
00000355: EB 02            jmp         00000359  
00000357: EB DE            jmp         00000337  
00000359: 83 7D D0 00      cmp         dword ptr [ebp-30h],0  
0000035D: 75 75            jne         000003D4  
0000035F: 83 3D 00 00 00 00 cmp         dword ptr [_nValidPages],0Fh  
        0F  
00000366: 7E 09            jle         00000371  
00000368: C7 45 A4 0F 00 00 mov         dword ptr [ebp-5Ch],0Fh  
        00  
0000036F: EB 09            jmp         0000037A  
00000371: 8B 0D 00 00 00 00 mov         ecx,dword ptr [_nValidPages]  
00000377: 89 4D A4         mov         dword ptr [ebp-5Ch],ecx  
0000037A: 8B 55 A4         mov         edx,dword ptr [ebp-5Ch]  
0000037D: 89 55 D0         mov         dword ptr [ebp-30h],edx  
00000380: C7 45 D8 00 00 00 mov         dword ptr [ebp-28h],0  
        00  
00000387: EB 09            jmp         00000392  
00000389: 8B 45 D8         mov         eax,dword ptr [ebp-28h]  
0000038C: 83 C0 01         add         eax,1  
0000038F: 89 45 D8         mov         dword ptr [ebp-28h],eax  
00000392: 8B 4D D8         mov         ecx,dword ptr [ebp-28h]  
00000395: 3B 4D D0         cmp         ecx,dword ptr [ebp-30h]  
00000398: 7F 22            jg          000003BC  
0000039A: 8B 55 D8         mov         edx,dword ptr [ebp-28h]  
0000039D: 8B 04 95 00 00 00 mov         eax,dword ptr [edx*4]  
        00  
000003A4: 89 45 E8         mov         dword ptr [ebp-18h],eax  
000003A7: 8B 4D D8         mov         ecx,dword ptr [ebp-28h]  
000003AA: 8B 55 CC         mov         edx,dword ptr [ebp-34h]  
000003AD: 89 14 8D 00 00 00 mov         dword ptr [ecx*4],edx  
        00  
000003B4: 8B 45 E8         mov         eax,dword ptr [ebp-18h]  
000003B7: 89 45 CC         mov         dword ptr [ebp-34h],eax  
000003BA: EB CD            jmp         00000389  
000003BC: 83 3D 00 00 00 00 cmp         dword ptr [_nValidPages],10h  
        10  
000003C3: 7D 0F            jge         000003D4  
000003C5: 8B 0D 00 00 00 00 mov         ecx,dword ptr [_nValidPages]  
000003CB: 83 C1 01         add         ecx,1  
000003CE: 89 0D 00 00 00 00 mov         dword ptr [_nValidPages],ecx  
000003D4: 6A 00            push        0  
000003D6: 68 00 00 00 00   push        offset _lModifying  
000003DB: FF 15 00 00 00 00 call        dword ptr [__imp__InterlockedExchange@8]  
000003E1: B8 01 00 00 00   mov         eax,1  
000003E6: 8B E5            mov         esp,ebp  
000003E8: 5D               pop         ebp  
000003E9: C3               ret  
```

`Summary`

| .bss | .debug$S | .debug$T | .text |
| :--- | :--- | :--- | :--- |
| `4C` | `6B5` | `11A4` | `3EA` |

下面是 `DUMPBIN` 输出的 `eh3valid.obj` 的反汇编结果：

```text
Microsoft (R) COFF/PE Dumper Version 7.10.3077
Copyright (C) Microsoft Corporation. All rights reserved.


Dump of file eh3valid.obj

File Type: COFF OBJECT
```

`__ValidateEH3RN`:

```assembly
  00000000: 55               push        ebp  
  00000001: 8B EC            mov         ebp,esp  
  00000003: 83 EC 5C         sub         esp,5Ch  
  00000006: 8B 45 08         mov         eax,dword ptr [ebp+8]  
  00000009: 8B 48 08         mov         ecx,dword ptr [eax+8]  
  0000000C: 89 4D E0         mov         dword ptr [ebp-20h],ecx  
  0000000F: 8B 55 E0         mov         edx,dword ptr [ebp-20h]  
  00000012: 83 E2 03         and         edx,3  
  00000015: 74 07            je          0000001E  
  00000017: 33 C0            xor         eax,eax  
  00000019: E9 C6 03 00 00   jmp         000003E4  
  0000001E: 64 A1 18 00 00 00 mov         eax,dword ptr fs:[00000018h]  
  00000024: 89 45 DC         mov         dword ptr [ebp-24h],eax  
  00000027: 8B 45 DC         mov         eax,dword ptr [ebp-24h]  
  0000002A: 8B 4D E0         mov         ecx,dword ptr [ebp-20h]  
  0000002D: 3B 48 08         cmp         ecx,dword ptr [eax+8]  
  00000030: 72 12            jb          00000044  
  00000032: 8B 55 DC         mov         edx,dword ptr [ebp-24h]  
  00000035: 8B 45 E0         mov         eax,dword ptr [ebp-20h]  
  00000038: 3B 42 04         cmp         eax,dword ptr [edx+4]  
  0000003B: 73 07            jae         00000044  
  0000003D: 33 C0            xor         eax,eax  
  0000003F: E9 A0 03 00 00   jmp         000003E4  
  00000044: 8B 4D 08         mov         ecx,dword ptr [ebp+8]  
  00000047: 83 79 0C FF      cmp         dword ptr [ecx+0Ch],0FFFFFFFFh  
  0000004B: 75 0A            jne         00000057  
  0000004D: B8 01 00 00 00   mov         eax,1  
  00000052: E9 8D 03 00 00   jmp         000003E4  
  00000057: C7 45 E4 00 00 00 mov         dword ptr [ebp-1Ch],0  
          00  
  0000005E: C7 45 F8 00 00 00 mov         dword ptr [ebp-8],0  
          00  
  00000065: EB 09            jmp         00000070  
  00000067: 8B 55 F8         mov         edx,dword ptr [ebp-8]  
  0000006A: 83 C2 01         add         edx,1  
  0000006D: 89 55 F8         mov         dword ptr [ebp-8],edx  
  00000070: 8B 45 08         mov         eax,dword ptr [ebp+8]  
  00000073: 8B 4D F8         mov         ecx,dword ptr [ebp-8]  
  00000076: 3B 48 0C         cmp         ecx,dword ptr [eax+0Ch]  
  00000079: 77 3F            ja          000000BA  
  0000007B: 8B 55 F8         mov         edx,dword ptr [ebp-8]  
  0000007E: 6B D2 0C         imul        edx,edx,0Ch  
  00000081: 8B 45 E0         mov         eax,dword ptr [ebp-20h]  
  00000084: 8B 0C 10         mov         ecx,dword ptr [eax+edx]  
  00000087: 89 4D A8         mov         dword ptr [ebp-58h],ecx  
  0000008A: 83 7D A8 FF      cmp         dword ptr [ebp-58h],0FFFFFFFFh  
  0000008E: 74 0F            je          0000009F  
  00000090: 8B 55 A8         mov         edx,dword ptr [ebp-58h]  
  00000093: 3B 55 F8         cmp         edx,dword ptr [ebp-8]  
  00000096: 72 07            jb          0000009F  
  00000098: 33 C0            xor         eax,eax  
  0000009A: E9 45 03 00 00   jmp         000003E4  
  0000009F: 8B 45 F8         mov         eax,dword ptr [ebp-8]  
  000000A2: 6B C0 0C         imul        eax,eax,0Ch  
  000000A5: 8B 4D E0         mov         ecx,dword ptr [ebp-20h]  
  000000A8: 83 7C 01 04 00   cmp         dword ptr [ecx+eax+4],0  
  000000AD: 74 09            je          000000B8  
  000000AF: 8B 55 E4         mov         edx,dword ptr [ebp-1Ch]  
  000000B2: 83 C2 01         add         edx,1  
  000000B5: 89 55 E4         mov         dword ptr [ebp-1Ch],edx  
  000000B8: EB AD            jmp         00000067  
  000000BA: 83 7D E4 00      cmp         dword ptr [ebp-1Ch],0  
  000000BE: 74 20            je          000000E0  
  000000C0: 8B 45 08         mov         eax,dword ptr [ebp+8]  
  000000C3: 8B 4D DC         mov         ecx,dword ptr [ebp-24h]  
  000000C6: 8B 50 F8         mov         edx,dword ptr [eax-8]  
  000000C9: 3B 51 08         cmp         edx,dword ptr [ecx+8]  
  000000CC: 72 0B            jb          000000D9  
  000000CE: 8B 45 08         mov         eax,dword ptr [ebp+8]  
  000000D1: 8B 48 F8         mov         ecx,dword ptr [eax-8]  
```
