
```
0058ED0A  |.  B3 01          MOV     BL,1

0058ED0C  |.  E9 D4010000    JMP     dumped\_.0058EEE5

0058ED11  |>  8B45 E8        MOV     EAX,DWORD PTR SS:[EBP-18]

0058ED14  |.  E8 9F61E7FF    CALL    dumped\_.00404EB8

0058ED19  |.  3BF0           CMP     ESI,EAX

0058ED1B  |.  7E 0A          JLE     SHORT dumped\_.0058ED27

0058ED1D  |.  8B45 E8        MOV     EAX,DWORD PTR SS:[EBP-18]

0058ED20  |.  E8 9361E7FF    CALL    dumped\_.00404EB8

0058ED25  |.  8BF0           MOV     ESI,EAX

0058ED27  |>  8BC6           MOV     EAX,ESI

0058ED29  |.  83F8 01        CMP     EAX,1

0058ED2C  |.  7C 21          JL      SHORT dumped\_.0058ED4F

0058ED2E  |>  84DB           /TEST    BL,BL

0058ED30  |.  74 10          |JE      SHORT dumped\_.0058ED42

0058ED32  |.  8B55 0C        |MOV     EDX,DWORD PTR SS:[EBP+C]

0058ED35  |.  8A5402 FF      |MOV     DL,BYTE PTR DS:[EDX+EAX-1]

0058ED39  |.  8B4D E8        |MOV     ECX,DWORD PTR SS:[EBP-18]

0058ED3C  |.  3A5401 FF      |CMP     DL,BYTE PTR DS:[ECX+EAX-1]

0058ED40  |.  74 04          |JE      SHORT dumped\_.0058ED46

0058ED42  |>  33D2           |XOR     EDX,EDX

0058ED44  |.  EB 02          |JMP     SHORT dumped\_.0058ED48

0058ED46  |>  B2 01          |MOV     DL,1

0058ED48  |>  8BDA           |MOV     EBX,EDX

0058ED4A  |.  48             |DEC     EAX

0058ED4B  |.  85C0           |TEST    EAX,EAX

0058ED4D  |.^ 75 DF          \JNZ     SHORT dumped\_.0058ED2E

0058ED4F  |>  8D45 E4        LEA     EAX,DWORD PTR SS:[EBP-1C]

0058ED52  |.  50             PUSH    EAX

0058ED53  |.  8D45 D4        LEA     EAX,DWORD PTR SS:[EBP-2C]

0058ED56  |.  8B4D F8        MOV     ECX,DWORD PTR SS:[EBP-8]

0058ED59  |.  8B55 F4        MOV     EDX,DWORD PTR SS:[EBP-C]

0058ED5C  |.  E8 A361E7FF    CALL    dumped\_.00404F04                          ;  SOFTAA和用户名合并(SOFTAAfcrjzmd)

0058ED61  |.  8B4D D4        MOV     ECX,DWORD PTR SS:[EBP-2C]

0058ED64  |.  8B45 FC        MOV     EAX,DWORD PTR SS:[EBP-4]

0058ED67  |.  8B40 58        MOV     EAX,DWORD PTR DS:[EAX+58]

0058ED6A  |.  BA 38000000    MOV     EDX,38

0058ED6F  |.  E8 D4FAFFFF    CALL    dumped\_.0058E848

0058ED74  |.  8B45 0C        MOV     EAX,DWORD PTR SS:[EBP+C]

0058ED77  |.  E8 3C61E7FF    CALL    dumped\_.00404EB8

0058ED7C  |.  8BF0           MOV     ESI,EAX

0058ED7E  |.  8B45 E4        MOV     EAX,DWORD PTR SS:[EBP-1C]

0058ED81  |.  E8 3261E7FF    CALL    dumped\_.00404EB8

0058ED86  |.  3BF0           CMP     ESI,EAX

0058ED88  |.  7E 0A          JLE     SHORT dumped\_.0058ED94

0058ED8A  |.  8B45 E4        MOV     EAX,DWORD PTR SS:[EBP-1C]

0058ED8D  |.  E8 2661E7FF    CALL    dumped\_.00404EB8

0058ED92  |.  8BF0           MOV     ESI,EAX

0058ED94  |>  8B45 E4        MOV     EAX,DWORD PTR SS:[EBP-1C]

0058ED97  |.  8B55 0C        MOV     EDX,DWORD PTR SS:[EBP+C]

0058ED9A  |.  E8 6562E7FF    CALL    dumped\_.00405004

0058ED9F  |.  75 07          JNZ     SHORT dumped\_.0058EDA8                    ;  ？？？

0058EDA1  |.  B3 01          MOV     BL,1

0058EDA3  |.  E9 3D010000    JMP     dumped\_.0058EEE5

0058EDA8  |>  8BC6           MOV     EAX,ESI

0058EDAA  |.  83F8 01        CMP     EAX,1

0058EDAD  |.  7C 21          JL      SHORT dumped\_.0058EDD0

0058EDAF  |>  84DB           /TEST    BL,BL

0058EDB1  |.  74 10          |JE      SHORT dumped\_.0058EDC3

0058EDB3  |.  8B55 0C        |MOV     EDX,DWORD PTR SS:[EBP+C]

0058EDB6  |.  8A5402 FF      |MOV     DL,BYTE PTR DS:[EDX+EAX-1]

0058EDBA  |.  8B4D E4        |MOV     ECX,DWORD PTR SS:[EBP-1C]

0058EDBD  |.  3A5401 FF      |CMP     DL,BYTE PTR DS:[ECX+EAX-1]

0058EDC1  |.  74 04          |JE      SHORT dumped\_.0058EDC7

0058EDC3  |>  33D2           |XOR     EDX,EDX

0058EDC5  |.  EB 02          |JMP     SHORT dumped\_.0058EDC9

0058EDC7  |>  B2 01          |MOV     DL,1

0058EDC9  |>  8BDA           |MOV     EBX,EDX

0058EDCB  |.  48             |DEC     EAX

0058EDCC  |.  85C0           |TEST    EAX,EAX

0058EDCE  |.^ 75 DF          \JNZ     SHORT dumped\_.0058EDAF

0058EDD0  |>  B3 01          MOV     BL,1

0058EDD2  |.  8D45 F0        LEA     EAX,DWORD PTR SS:[EBP-10]

0058EDD5  |.  50             PUSH    EAX

0058EDD6  |.  8D45 D0        LEA     EAX,DWORD PTR SS:[EBP-30]

0058EDD9  |.  8B4D F8        MOV     ECX,DWORD PTR SS:[EBP-8]

0058EDDC  |.  8B55 F4        MOV     EDX,DWORD PTR SS:[EBP-C]

0058EDDF  |.  E8 2061E7FF    CALL    dumped\_.00404F04                          ;  SOFTAA和用户名合并(SOFTAAfcrjzmd)

0058EDE4  |.  8B4D D0        MOV     ECX,DWORD PTR SS:[EBP-30]

0058EDE7  |.  8B45 FC        MOV     EAX,DWORD PTR SS:[EBP-4]

0058EDEA  |.  8B40 58        MOV     EAX,DWORD PTR DS:[EAX+58]

0058EDED  |.  BA C8010000    MOV     EDX,1C8                                   ;  EDX=1C8

0058EDF2  |.  E8 51FAFFFF    CALL    dumped\_.0058E848                          ;  ◆关键算法CALL跟进!!◆

0058EDF7  |.  8D45 EC        LEA     EAX,DWORD PTR SS:[EBP-14]

0058EDFA  |.  BA 34EF5800    MOV     EDX,dumped\_.0058EF34                      ;  EDX=th4gi8fsu0

0058EDFF  |.  E8 8C5EE7FF    CALL    dumped\_.00404C90                          ;  取假码

0058EE04  |.  8B45 0C        MOV     EAX,DWORD PTR SS:[EBP+C]

0058EE07  |.  E8 AC60E7FF    CALL    dumped\_.00404EB8                          ;  取假码长度!!

0058EE0C  |.  8BF0           MOV     ESI,EAX

0058EE0E  |.  8B45 F0        MOV     EAX,DWORD PTR SS:[EBP-10]                 ;  ┥真码出现!!!145122-19050768-18314466

0058EE11  |.  E8 A260E7FF    CALL    dumped\_.00404EB8                          ;  取真码长度!

0058EE16  |.  3BF0           CMP     ESI,EAX                                   ;  比较真假码字符长度是否相等!!不相等则注册失败!

0058EE18  |.  74 07          JE      SHORT dumped\_.0058EE21

0058EE1A  |.  33DB           XOR     EBX,EBX                                   ;  EBX清0

0058EE1C  |.  E9 C4000000    JMP     dumped\_.0058EEE5                          ;  无条件跳走失败!!

0058EE21  |>  8BC6           MOV     EAX,ESI

0058EE23  |.  83E8 04        SUB     EAX,4

0058EE26  |.  83F8 08        CMP     EAX,8

0058EE29  |.  7C 22          JL      SHORT dumped\_.0058EE4D

0058EE2B  |>  84DB           /TEST    BL,BL

0058EE2D  |.  74 10          |JE      SHORT dumped\_.0058EE3F

0058EE2F  |.  8B55 0C        |MOV     EDX,DWORD PTR SS:[EBP+C]

0058EE32  |.  8A5402 FF      |MOV     DL,BYTE PTR DS:[EDX+EAX-1]

0058EE36  |.  8B4D F0        |MOV     ECX,DWORD PTR SS:[EBP-10]

0058EE39  |.  3A5401 FF      |CMP     DL,BYTE PTR DS:[ECX+EAX-1]

0058EE3D  |.  74 04          |JE      SHORT dumped\_.0058EE43

0058EE3F  |>  33D2           |XOR     EDX,EDX

0058EE41  |.  EB 02          |JMP     SHORT dumped\_.0058EE45

0058EE43  |>  B2 01          |MOV     DL,1

0058EE45  |>  8BDA           |MOV     EBX,EDX

0058EE47  |.  48             |DEC     EAX

0058EE48  |.  83F8 07        |CMP     EAX,7

0058EE4B  |.^ 75 DE          \JNZ     SHORT dumped\_.0058EE2B

0058EE4D  |>  B8 01000000    MOV     EAX,1

0058EE52  |>  84DB           /TEST    BL,BL

0058EE54  |.  74 0B          |JE      SHORT dumped\_.0058EE61

0058EE56  |.  8B55 EC        |MOV     EDX,DWORD PTR SS:[EBP-14]

0058EE59  |.  8A5402 FF      |MOV     DL,BYTE PTR DS:[EDX+EAX-1]

0058EE5D  |.  3AD2           |CMP     DL,DL

0058EE5F  |.  74 04          |JE      SHORT dumped\_.0058EE65

0058EE61  |>  33D2           |XOR     EDX,EDX

0058EE63  |.  EB 02          |JMP     SHORT dumped\_.0058EE67

0058EE65  |>  B2 01          |MOV     DL,1

0058EE67  |>  8BDA           |MOV     EBX,EDX

0058EE69  |.  40             |INC     EAX

0058EE6A  |.  83F8 09        |CMP     EAX,9

0058EE6D  |.^ 75 E3          \JNZ     SHORT dumped\_.0058EE52

0058EE6F  |.  8BC6           MOV     EAX,ESI

0058EE71  |.  83F8 08        CMP     EAX,8

0058EE74  |.  7C 22          JL      SHORT dumped\_.0058EE98

0058EE76  |>  84DB           /TEST    BL,BL

0058EE78  |.  74 10          |JE      SHORT dumped\_.0058EE8A

0058EE7A  |.  8B55 0C        |MOV     EDX,DWORD PTR SS:[EBP+C]

0058EE7D  |.  8A5402 FF      |MOV     DL,BYTE PTR DS:[EDX+EAX-1]
```
