
```asm
0042B046    8B75 0C         MOV ESI,DWORD PTR SS:[EBP+C]     ; pAPIName

0042B049    8B7D E8         MOV EDI,DWORD PTR SS:[EBP-18]    ; var6

0042B04C    8B3C8F          MOV EDI,DWORD PTR DS:[EDI+ECX*4]

0042B04F    037D 08         ADD EDI,DWORD PTR SS:[EBP+8]                  ; RVA->VA

0042B052    8A043B          MOV AL,BYTE PTR DS:[EBX+EDI]

0042B055    3A0433          CMP AL,BYTE PTR DS:[EBX+ESI]

0042B058    75 0A           JNZ SHORT Keygen.0042B064                     ; 字符不等, 下一个 API

0042B05A    43              INC EBX

0042B05B    B0 00           MOV AL,0                                      ; NULL

0042B05D    3A0433          CMP AL,BYTE PTR DS:[EBX+ESI]

0042B060  ^ 75 F0           JNZ SHORT Keygen.0042B052

0042B062    74 08           JE SHORT Keygen.0042B06C                      ; 一个 API 找到了

0042B064    33DB            XOR EBX,EBX

0042B066    41              INC ECX

0042B067    3B4D F0         CMP ECX,DWORD PTR SS:[EBP-10]    ; var4

0042B06A  ^ 75 DD           JNZ SHORT Keygen.0042B049

0042B06C    8B7D F4         MOV EDI,DWORD PTR SS:[EBP-C]     ; var3

0042B06F    8B048F          MOV EAX,DWORD PTR DS:[EDI+ECX*4]

0042B072    0345 08         ADD EAX,DWORD PTR SS:[EBP+8]     ; API address 结果

0042B075    5F              POP EDI

0042B076    5E              POP ESI

0042B077    59              POP ECX

0042B078    5B              POP EBX

0042B079    C9              LEAVE

0042B07A    C2 0800         RETN 8
```

```asm
00435B8F  47 65 74 4D 6F 64 75 6C 65 48 61 6E 64 6C 65 41  GetModuleHandleA

00435B9F  00 00 00 00 00 4C 6F 61 64 4C 69 62 72 61 72 79  .....LoadLibrary

00435BAF  41 00 00 00 00 00 47 65 74 50 72 6F 63 41 64 64  A.....GetProcAdd

00435BBF  72 65 73 73 00 00 00 00 00 47 6C 6F 62 61 6C 41  ress.....GlobalA

00435BCF  6C 6C 6F 63 00 00 00 00 00 47 6C 6F 62 61 6C 46  lloc.....GlobalF

00435BDF  72 65 65 00 00 00 00 00 55 6E 68 61 6E 64 6C 65  ree.....Unhandle

00435BEF  64 45 78 63 65 70 74 69 6F 6E 46 69 6C 74 65 72  dExceptionFilter

00435BFF  00 00 00 00 00 47 65 74 4D 6F 64 75 6C 65 46 69  .....GetModuleFi

00435C0F  6C 65 4E 61 6D 65 41 00 00 00 00 00 45 78 69 74  leNameA.....Exit

00435C1F  50 72 6F 63 65 73 73 00 00 00 00 00 43 72 65 61  Process.....Crea

00435C2F  74 65 50 72 6F 63 65 73 73 00 00 00 00 00 57 61  teProcess.....Wa

00435C3F  69 74 46 6F 72 44 65 62 75 67 45 76 65 6E 74 00  itForDebugEvent.

00435C4F  00 00 00 00 43 6F 6E 74 69 6E 75 65 44 65 62 75  ....ContinueDebu

00435C5F  67 45 76 65 6E 74 00 00 00 00 00 47 65 74 43 6F  gEvent.....GetCo

00435C6F  6D 6D 61 6E 64 4C 69 6E 65 00 00 00 00 00 6C 73  mmandLine.....ls

00435C7F  74 72 6C 65 6E 00 00 00 00 00 6C 73 74 63 70 79  trlen.....lstrcp

00435C8F  79 00 00 00 00 00 4D 65 73 73 61 67 65 42 6F 78  y.....MessageBox

00435C9F  41 00 00 00 00 00 52 74 6C 5A 65 72 6F 4D 65 6D  A.....RtlZeroMem

00435CAF  6F 72 79 00 00 00 00 00 43 72 65 61 74 65 54 6F  ory.....CreateTo

00435CBF  6F 6C 68 65 6C 70 33 32 53 6E 61 70 73 68 6F 74  olhelp32Snapshot

00435CCF  00 00 00 00 00 50 72 6F 63 65 73 73 33 32 46 69  .....Process32Fi

00435CDF  72 73 74 00 00 00 00 00 50 72 6F 63 65 73 73 33  rst.....Process3

00435CEF  32 4E 65 78 74 00 00 00 00 00 47 65 74 43 75 72  2Next.....GetCur

00435CFF  72 65 6E 74 50 72 6F 63 65 73 73 49 64 00 00 00  rentProcessId...

00435D0F  00 00 4F 70 65 6E 50 72 6F 63 65 73 73 00 00 00  ..OpenProcess...

00435D1F  00 00 43 6C 6F 73 65 48 61 6E 64 6C 65 00 00 00  ..CloseHandle...

00435D2F  00 00 52 65 61 64 50 72 6F 63 65 73 73 4D 65 6D  ..ReadProcessMem

00435D3F  6F 72 79 00 00 00 00 00 47 65 74 54 68 72 65 61  ory.....GetThrea

00435D4F  64 43 6F 6E 74 65 78 74 00 00 00 00 00 53 65 74  dContext.....Set

00435D5F  54 68 72 65 61 64 43 6F 6E 74 65 78 74 00 00 00  ThreadContext...

00435D6F  00 00 57 72 69 74 65 50 72 6F 63 65 73 73 4D 65  ..WriteProcessMe

00435D7F  6D 6F 72 79 00 00 00 00 00 46 6C 75 73 68 49 6E  mory.....FlushIn

00435D8F  73 74 72 75 63 74 69 6F 6E 43 61 63 68 65 00 00  structionCache..
```

// Note 2

// `42B1B7` , SEHhandler

```asm
0042B1B7    55              PUSH EBP

0042B1B8    8BEC            MOV EBP,ESP

0042B1BA    56              PUSH ESI

0042B1BB    57              PUSH EDI

0042B1BC    53              PUSH EBX

0042B1BD    8B75 08         MOV ESI,DWORD PTR SS:[EBP+8]                   ; pEXCEPTION_RECORD

0042B1C0    8B7D 10         MOV EDI,DWORD PTR SS:[EBP+10]                  ; pContext

0042B1C3    813E 050000C0   CMP DWORD PTR DS:[ESI],C0000005                ; ExceptionCode 读写内存冲突

0042B1C9    75 28           JNZ SHORT Keygen.0042B1F3

0042B1CB    60              PUSHAD

0042B1CC    E8 00000000     CALL Keygen.0042B1D1

0042B1D1    5D              POP EBP

0042B1D2    81ED D1114000   SUB EBP,Keygen.004011D1                        ; 2A000

0042B1D8    8D9D AF624000   LEA EBX,DWORD PTR SS:[EBP+4062AF]              ; 4302AF

0042B1DE    C787 B0000000 0>MOV DWORD PTR DS:[EDI+B0],0                    ; regEAX

0042B1E8    899F B8000000   MOV DWORD PTR DS:[EDI+B8],EBX                  ; regEIP=4302AF

0042B1EE    61              POPAD

0042B1EF    33C0            XOR EAX,EAX                                    ; 已处理

0042B1F1    EB 05           JMP SHORT Keygen.0042B1F8

0042B1F3    B8 01000000     MOV EAX,1                                      ; 异常未处理

0042B1F8    5B              POP EBX

0042B1F9    5F              POP EDI

0042B1FA    5E              POP ESI

0042B1FB    C9              LEAVE

0042B1FC    C3              RETN
```

[附件：unpacked.zip](https://bbs.kanxue.com/bbs.pediy.com/upload/file/2005/1/unpacked.zip_462.zip)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
