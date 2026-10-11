
### 0040367E

```
0040367E  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 00403685

```
00403685  |.  C64424 78 03  MOV BYTE PTR SS:\[ESP+78\],3
```

### 0040368A

```
0040368A  |.  E8 780C0A00   CALL Capture.004A4307
```

### 0040368F

```
0040368F  |.  8BF0          MOV ESI,EAX
```

### 00403691

```
00403691  |.  6A 01         PUSH 1
```

### 00403693

```
00403693  |.  8D5424 5C     LEA EDX,DWORD PTR SS:\[ESP+5C\]
```

### 00403697

```
00403697  |.  6A 1F         PUSH 1F                           ;  取试炼码的第 1F+1 位字

符
```

### 00403699

```
00403699  |.  52            PUSH EDX
```

### 0040369A

```
0040369A  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004036A1

```
004036A1  |.  C64424 78 04  MOV BYTE PTR SS:\[ESP+78\],4
```

### 004036A6

```
004036A6  |.  E8 5C0C0A00   CALL Capture.004A4307
```

### 004036AB

```
004036AB  |.  8BF8          MOV EDI,EAX
```

### 004036AD

```
004036AD  |.  6A 01         PUSH 1
```

### 004036AF

```
004036AF  |.  8D4424 58     LEA EAX,DWORD PTR SS:\[ESP+58\]
```

### 004036B3

```
004036B3  |.  6A 06         PUSH 6                            ;  取试炼码的第 6+1 位字

符，后面依次取第某位的试炼码
```

### 004036B5

```
004036B5  |.  50            PUSH EAX
```

### 004036B6

```
004036B6  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004036BD

```
004036BD  |.  C64424 78 05  MOV BYTE PTR SS:\[ESP+78\],5
```

### 004036C2

```
004036C2  |.  E8 400C0A00   CALL Capture.004A4307
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 0040373B

```
0040373B  |.  894424 18     MOV DWORD PTR SS:\[ESP+18\],EAX
```

### 0040373F

```
0040373F  |.  6A 01         PUSH 1
```

### 00403741

```
00403741  |.  8D5424 44     LEA EDX,DWORD PTR SS:\[ESP+44\]
```

### 00403745

```
00403745  |.  6A 00         PUSH 0                            ;  取试炼码的第 0+1 位字

符
```

### 00403747

```
00403747  |.  52            PUSH EDX
```

### 00403748

```
00403748  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 0040374F

```
0040374F  |.  C64424 78 0A  MOV BYTE PTR SS:\[ESP+78\],0A
```

### 00403754

```
00403754  |.  E8 AE0B0A00   CALL Capture.004A4307
```

### 00403759

```
00403759  |.  8B4C24 18     MOV ECX,DWORD PTR SS:\[ESP+18\]
```

### 0040375D

```
0040375D  |.  C64424 6C 0B  MOV BYTE PTR SS:\[ESP+6C\],0B
```

### 00403762

```
00403762  |.  51            PUSH ECX
```

### 00403763

```
00403763  |.  50            PUSH EAX
```

### 00403764

```
00403764  |.  8D5424 44     LEA EDX,DWORD PTR SS:\[ESP+44\]
```

### 00403768

```
00403768  |.  52            PUSH EDX
```

### 00403769

```
00403769  |.  E8 47920A00   CALL Capture.004AC9B5
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 0040389C

```
0040389C  |.  E8 E58E0A00   CALL Capture.004AC786
```

### 004038A1

```
004038A1  |.  8D4C24 58     LEA ECX,DWORD PTR SS:\[ESP+58\]
```

### 004038A5

```
004038A5  |.  C64424 6C 04  MOV BYTE PTR SS:\[ESP+6C\],4
```

### 004038AA

```
004038AA  |.  E8 D78E0A00   CALL Capture.004AC786
```

### 004038AF

```
004038AF  |.  8D4C24 5C     LEA ECX,DWORD PTR SS:\[ESP+5C\]
```

### 004038B3

```
004038B3  |.  C64424 6C 03  MOV BYTE PTR SS:\[ESP+6C\],3
```

### 004038B8

```
004038B8  |.  E8 C98E0A00   CALL Capture.004AC786                    ;  这段程序将上面取出的

字符反过来组成字符串 `str1`
```

### 004038BD

```
004038BD  |.  6A 01         PUSH 1                            ;  下面程序段功能与上面

一样，最终形成字符串 `str2`
```

### 004038BF

```
004038BF  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]
```

### 004038C3

```
004038C3  |.  6A 03         PUSH 3
```

### 004038C5

```
004038C5  |.  51            PUSH ECX
```

### 004038C6

```
004038C6  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004038CD

```
004038CD  |.  E8 350A0A00   CALL Capture.004A4307
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 00403999

```
00403999  |.  8B4C24 5C     MOV ECX,DWORD PTR SS:\[ESP+5C\]
```

### 0040399D

```
0040399D  |.  8D5424 38     LEA EDX,DWORD PTR SS:\[ESP+38\]
```

### 004039A1

```
004039A1  |.  51            PUSH ECX
```

### 004039A2

```
004039A2  |.  50            PUSH EAX
```

### 004039A3

```
004039A3  |.  52            PUSH EDX
```

### 004039A4

```
004039A4  |.  C64424 78 1A  MOV BYTE PTR SS:\[ESP+78\],1A
```

### 004039A9

```
004039A9  |.  E8 07900A00   CALL Capture.004AC9B5
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 00403AD3

```
00403AD3  |.  8D4C24 20     LEA ECX,DWORD PTR SS:\[ESP+20\]
```

### 00403AD7

```
00403AD7  |.  C64424 6C 14  MOV BYTE PTR SS:\[ESP+6C\],14
```

### 00403ADC

```
00403ADC  |.  E8 A58C0A00   CALL Capture.004AC786
```

### 00403AE1

```
00403AE1  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]
```

### 00403AE5

```
00403AE5  |.  C64424 6C 13  MOV BYTE PTR SS:\[ESP+6C\],13
```

### 00403AEA

```
00403AEA  |.  E8 978C0A00   CALL Capture.004AC786
```

### 00403AEF

```
00403AEF  |.  8D4C24 18     LEA ECX,DWORD PTR SS:\[ESP+18\]
```

### 00403AF3

```
00403AF3  |.  C64424 6C 03  MOV BYTE PTR SS:\[ESP+6C\],3
```

### 00403AF8

```
00403AF8  |.  E8 898C0A00   CALL Capture.004AC786
```

### 00403AFD

```
00403AFD  |.  51            PUSH ECX
```

### 00403AFE

```
00403AFE  |.  8D5424 14     LEA EDX,DWORD PTR SS:\[ESP+14\]
```

### 00403B02

```
00403B02  |.  8BCC          MOV ECX,ESP
```

### 00403B04

```
00403B04  |.  896424 60     MOV DWORD PTR SS:\[ESP+60\],ESP
```

### 00403B08

```
00403B08  |.  52            PUSH EDX
```

### 00403B09

```
00403B09  |.  E8 ED890A00   CALL Capture.004AC4FB
```

### 00403B0E

```
00403B0E  |.  51            PUSH ECX
```

### 00403B0F

```
00403B0F  |.  8D4424 1C     LEA EAX,DWORD PTR SS:\[ESP+1C\]
```

### 00403B13

```
00403B13  |.  8BCC          MOV ECX,ESP
```

### 00403B15

```
00403B15  |.  896424 60     MOV DWORD PTR SS:\[ESP+60\],ESP
```

### 00403B19

```
00403B19  |.  50            PUSH EAX
```

### 00403B1A

```
00403B1A  |.  C64424 78 22  MOV BYTE PTR SS:\[ESP+78\],22
```

### 00403B1F

```
00403B1F  |.  E8 D7890A00   CALL Capture.004AC4FB                    ;  注意观察堆栈区，有两

个字符串
```

### 00403B24

```
00403B24  |.  8B4C24 68     MOV ECX,DWORD PTR SS:\[ESP+68\]          ;  |`str1="afhlxgFj"`

`str2="bmnEuCzd"`
```

### 00403B28

```
00403B28  |.  C64424 74 03  MOV BYTE PTR SS:\[ESP+74\],3           ;  |
```

### 00403B2D

```
00403B2D  |.  E8 8E000000   CALL Capture.00403BC0                    ;  \\第二个关键 CALL，继

续跟进
```

### 00403B32

```
00403B32  |.  84C0          TEST AL,AL
```

### 00403B34

```
00403B34  |.  C64424 6C 02  MOV BYTE PTR SS:\[ESP+6C\],2
```

### 00403B39

```
00403B39  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]
```

### 00403B3D

```
00403B3D  |.  74 36         JE SHORT Capture.00403B75
```

### 3、

### 00403BC0

```
00403BC0  /$  6A FF         PUSH -1                           ;  这次来到了这里
```

### 00403BC2

```
00403BC2  |.  68 E84F4D00   PUSH Capture.004D4FE8                    ;  SE handler installation
```

### 00403BC7

```
00403BC7  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]
```

### 00403BCD

```
00403BCD  |.  50            PUSH EAX
```

### 00403BCE

```
00403BCE  |.  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP
```

### 00403BD5

```
00403BD5  |.  83EC 0C       SUB ESP,0C
```

### 00403BD8

```
00403BD8  |.  53            PUSH EBX
```

### 00403BD9

```
00403BD9  |.  55            PUSH EBP
```

### 00403BDA

```
00403BDA  |.  56            PUSH ESI
```

### 00403BDB

```
00403BDB  |.  57            PUSH EDI
```

### 00403BDC

```
00403BDC  |.  8B4424 2C     MOV EAX,DWORD PTR SS:\[ESP+2C\]          ;  `eax="afhlxgFj"`
```

### 00403BE0

```
00403BE0  |.  C74424 24 010>MOV DWORD PTR SS:\[ESP+24\],1
```

### 00403BE8

```
00403BE8  |.  8B40 F8       MOV EAX,DWORD PTR DS:\[EAX-8\]
```

### 00403BEB

```
00403BEB  |.  83F8 08       CMP EAX,8                             ;  看看 `str1` 是否 8 位
```

### 00403BEE

```
00403BEE  |.  0F85 E8000000 JNZ Capture.00403CDC
```

### 00403BF4

```
00403BF4  |.  8B4C24 30     MOV ECX,DWORD PTR SS:\[ESP+30\]          ;  `ecx="bmnEuCzd"`
```

### 00403BF8

```
00403BF8  |.  8379 F8 08    CMP DWORD PTR DS:\[ECX-8\],8            ;  看看 `str2` 是否 8 位
```

### 00403BFC

```
00403BFC  |.  0F85 DA000000 JNZ Capture.00403CDC
```

### 00403C02

```
00403C02  |.  68 4C945000   PUSH Capture.0050944C                    ;  push

`str3="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"`
```

### 00403C07

```
00403C07  |.  8D4C24 14     LEA ECX,DWORD PTR SS:\[ESP+14\]
```

### 00403C0B

```
00403C0B  |.  E8 E48B0A00   CALL Capture.004AC7F4
```

### 00403C10

```
00403C10  |.  B3 02         MOV BL,2
```

### 00403C12

```
00403C12  |.  33F6          XOR ESI,ESI
```

### 00403C14

```
00403C14  |.  885C24 24     MOV BYTE PTR SS:\[ESP+24\],BL
```

### 00403C18

```
00403C18  |>  6A 01         /PUSH 1
```

### 00403C1A

```
00403C1A  |.  8D5424 18     |LEA EDX,DWORD PTR SS:\[ESP+18\]
```

### 00403C1E

```
00403C1E  |.  56            |PUSH ESI
```

### 00403C1F

```
00403C1F  |.  52            |PUSH EDX
```

### 00403C20

```
00403C20  |.  8D4C24 38     |LEA ECX,DWORD PTR SS:\[ESP+38\]
```

### 00403C24

```
00403C24  |.  E8 DE060A00   |CALL Capture.004A4307
```

### 00403C29

```
00403C29  |.  8B00          |MOV EAX,DWORD PTR DS:\[EAX\]
```

### 00403C2B

```
00403C2B  |.  8D4C24 10     |LEA ECX,DWORD PTR SS:\[ESP+10\]
```

### 00403C2F

```
00403C2F  |.  50            |PUSH EAX
```

### 00403C30

```
00403C30  |.  C64424 28 03  |MOV BYTE PTR SS:\[ESP+28\],3
```

### 00403C35

```
00403C35  |.  E8 A4080A00   |CALL Capture.004A44DE
```

### 00403C3A

```
00403C3A  |.  8D4C24 14     |LEA ECX,DWORD PTR SS:\[ESP+14\]
```

### 00403C3E

```
00403C3E  |.  8BF8          |MOV EDI,EAX
```

### 00403C40

```
00403C40  |.  885C24 24     |MOV BYTE PTR SS:\[ESP+24\],BL
```

### 00403C44

```
00403C44  |.  E8 3D8B0A00   |CALL Capture.004AC786
```

### 00403C49

```
00403C49  |.  6A 01         |PUSH 1
```

### 00403C4B

```
00403C4B  |.  8D4424 1C     |LEA EAX,DWORD PTR SS:\[ESP+1C\]
```

### 00403C4F

```
00403C4F  |.  56            |PUSH ESI
```
