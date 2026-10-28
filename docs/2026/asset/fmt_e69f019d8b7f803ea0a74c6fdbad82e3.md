
```
  this program has been deformed,and is a anti anti virus check the pragram.first the with PEID check shell。shows "Borland Delphi 3.0 (???) *",use with OD loaded verify。like the Delphi program,which use some functions of Delphi,or may not be is the Delphi program,first look all to anti anti virus check the process.

  there are three decryption code segment, cycle the decryption.the first piece of code decryption code,save to the allocation of the heap,the heap code is responsibe for filling code 3 of the code of the data,such as PE input table,and copy the decryption code parahraph 3 is called the code segment UPX3.04 packers,form 400000 the following you can see the label.summary.
```

### 总结

**one:**  
`the first piece of code decryption code to the allocated heap space.`

**two:**  
`heap of code responsibe for filling in paragraph 3 of UPX shell segment data.such as the of input table.`

**three:**  
`UPX shell segment decryption code.`

```
some code is very clecn,very precise.you can determine that this program is carefully constructed.
```

### 步骤跟踪

`step-by-step tracking,look assembly code flow,the first paragraph decryption code is not given ,because it is meaningless code,and very long.Characteristics of the first piece code of the call stack code.`

在地址 `0041FB17` 使用断点, `F7` 跟随代码信息到 `heap`.

```
0041FB06  |.  81C6 4C1F0000 ADD ESI,1F4C
0041FB0C  |.  56            PUSH ESI
0041FB0D  |.  E8 71E9FFFF   CALL DHL-Inte.0041E483
0041FB12  |.  83C4 40       ADD ESP,40
0041FB15  |.  6A 2C         PUSH 2C
0041FB17  |.  FF15 B4134200 CALL DWORD PTR DS:[4213B4]    ; call heap filled with code,this address is filled.
0041FB1D  |.  59            POP ECX
0041FB1E  |.  FF15 28004200 CALL DWORD PTR DS:[<&KERNEL32.GetLastError>] ; [GetLastError
0041FB24  |.  5F            POP EDI
0041FB25  |.  5E            POP ESI
0041FB26  |.  33C0          XOR EAX,EAX
0041FB28  |.  5B            POP EBX
0041FB29  |.  C9            LEAVE
```

当前代码是 `heap` 代码, 第一个 `heap` 代码和 `PEB` 和 `PE` 结构相关, 看汇编代码.

```
00940048    55              PUSH EBP                                     ; PE filled PE
00940049    89E5            MOV EBP,ESP
0094004B    83EC 1C         SUB ESP,1C
0094004E    64:A1 30000000  MOV EAX,DWORD PTR FS:[30]                    ; get PEB structed address
00940054    53              PUSH EBX
00940055    8B58 0C         MOV EBX,DWORD PTR DS:[EAX+C]                 ; get PEB_LDR_DATA structed address
00940058    8B40 08         MOV EAX,DWORD PTR DS:[EAX+8]                 ; get ImageBaseAddress,both loaded base address
0094005B    56              PUSH ESI
0094005C    83C3 0C         ADD EBX,0C                                   ; point to the address of the PEB_LDR_DATA member three the address of,get the list of loaded module
0094005F    57              PUSH EDI
00940060    8B3B            MOV EDI,DWORD PTR DS:[EBX]                   ; LDR_MODULE first data a member of the data,point the address of the next module, the typical sigle linked list.
00940062    8945 FC         MOV DWORD PTR SS:[EBP-4],EAX                 ; save their own base address,in this case is 0x400000.
00940065   /EB 79           JMP SHORT 009400E0                           ; an jump,the comparisons are for their own address ,by tracking LDR_MODULE is doubly linked list.
00940067   |0FB757 2C       MOVZX EDX,WORD PTR DS:[EDI+2C]               ; get LDR_MODULE->HashTableEntry
0094006B   |89D0            MOV EAX,EDX
0094006D   |31F6            XOR ESI,ESI
0094006F   |D1E8            SHR EAX,1                                    ; logical shift right one bit 
00940071   |74 1E           JE SHORT 00940091
00940073   |8B4F 30         MOV ECX,DWORD PTR DS:[EDI+30]                ; get LDR_MODULE->BaseDllName
00940076   |8945 F8         MOV DWORD PTR SS:[EBP-8],EAX
00940079   |0FB701          MOVZX EAX,WORD PTR DS:[ECX]                  ; get the to get names of 
0094007C   |83F8 61         CMP EAX,61                                   ; whether the cimparison is less than the capital 'A'
0094007F   |72 03           JB SHORT 00940084
00940081   |83C0 E0         ADD EAX,-20                                  ; greater the or epqual to capital 'A' to lowercase
00940084   |C1CE 0D         ROR ESI,0D                                   ; rotate right 0xD calculation HASH
00940087   |01C6            ADD ESI,EAX                                  ; adding 
00940089   |83C1 02         ADD ECX,2                                    ; pointer next char
0094008C   |FF4D F8         DEC DWORD PTR SS:[EBP-8]                     ; name length sub 1
0094008F  ^|75 E8           JNZ SHORT 00940079                           ; loop calculation the name of HASH
00940091   |83FA 18         CMP EDX,18                                   ; whether the cimparison is LDR_MODULE->HashTableEntry for 0x18
00940094   |75 48           JNZ SHORT 009400DE
00940096   |81FE 17CA2B6E   CMP ESI,6E2BCA17                             ; comparison to obtain the name of HASH whether a specific HASH
0094009C   |75 40           JNZ SHORT 009400DE
0094009E   |8B77 18         MOV ESI,DWORD PTR DS:[EDI+18]                ; get LDR_MODULE->BaseAddress
009400A1   |68 76468B8A     PUSH 8A8B4676
009400A6   |E8 3B010000     CALL 009401E6                                ; by HASH search function address 
009400AB   |8B77 18         MOV ESI,DWORD PTR DS:[EDI+18]                ; get LDR_MODULE->BaseAddress
009400AE   |68 7AEECA1A     PUSH 1ACAEE7A
009400B3   |8945 EC         MOV DWORD PTR SS:[EBP-14],EAX                ; save the search for the address,the shell used LoadLibraryA
009400B6   |E8 2B010000     CALL 009401E6                                ; by HASH search function address
009400BB   |8B77 18         MOV ESI,DWORD PTR DS:[EDI+18]                ; get LDR_MODULE->BaseAddress
009400BE   |68 E3B70318     PUSH 1803B7E3
009400C3   |8945 F4         MOV DWORD PTR SS:[EBP-C],EAX                 ; shell code use of GetProcAddress
009400C6   |E8 1B010000     CALL 009401E6
009400CB   |8B77 18         MOV ESI,DWORD PTR DS:[EDI+18]                ; get LDR_MODULE->BaseAddress
009400CE   |68 4ECCDF12     PUSH 12DFCC4E
009400D3   |8945 F0         MOV DWORD PTR SS:[EBP-10],EAX                ; shell code use of VirtualProtect
009400D6   |E8 0B010000     CALL 009401E6
009400DB   |8945 E8         MOV DWORD PTR SS:[EBP-18],EAX                ; ExitProcess
009400DE   |8B3F            MOV EDI,DWORD PTR DS:[EDI]                   ; LDR_MODULE->InLoadOrderModuleList
009400E0   \39DF            CMP EDI,EBX                                  ; comparisons are led by the address,which is its own address,the doubly linked linked list.
009400E2  ^ 75 83           JNZ SHORT 00940067                           ; no traversal afyer continue
```

`PE` 结构相关操作, 依次将 `heap` 代码复制到内存中, 然后获取 `API` 地址以填充 `UPX` shell 代码; 它们应该参考 `UPX` shell 填充过程, 然后稍作变形. 具体可以分析 `UPX` 的 shell, 不全部给出汇编代码, 一部分汇编代码

```
009400E4    68 3B020000     PUSH 23B
009400E9    E8 83010000     CALL 00940271                                       ; relocatble address,navigate to the PE address,the PE address stored in the heap
009400EE    8B7D FC         MOV EDI,DWORD PTR SS:[EBP-4]                        ; get 
009400F1    89C6            MOV ESI,EAX                                         ; get PE header offser,here is related and PE structed operations,and recommend an information.PE structure field offset reference
009400F3    8B5E 3C         MOV EBX,DWORD PTR DS:[ESI+3C]                       ; DOSHEADER->e_lfanew
009400F6    8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]
009400F9    50              PUSH EAX
009400FA    6A 40           PUSH 40
009400FC    01F3            ADD EBX,ESI                                         ; get PE header
009400FE    FF73 50         PUSH DWORD PTR DS:[EBX+50]     ; get segment address offset ,OptHeader>SizeOfHeaders+PE header=first segment offset
00940101    8975 F8         MOV DWORD PTR SS:[EBP-8],ESI
00940104    57              PUSH EDI
00940105    FF55 F0         CALL DWORD PTR SS:[EBP-10]                          ;called VirtualProtect modify memory attribute
00940108    85C0            TEST EAX,EAX
0094010A    0F84 D2000000   JE 009401E2                                         ; judgment wherher to overwrite the success,or else lead to INT3
00940110    8B4B 50         MOV ECX,DWORD PTR DS:[EBX+50]                       ; first segment offset
00940113    30C0            XOR AL,AL
00940115    F3:AA           REP STOS BYTE PTR ES:[EDI]                          ; not use,save shell code to heap space
00940117    8B7D FC         MOV EDI,DWORD PTR SS:[EBP-4]                        ; get thrie own base
0094011A    8B4B 54         MOV ECX,DWORD PTR DS:[EBX+54]                       ; segment virtual address
0094011D    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]        ; in heap code TO segment memory piece
0094011F    0FB743 14       MOVZX EAX,WORD PTR DS:[EBX+14]                      ; Characteristics
00940123    31C9            XOR ECX,ECX
00940125    31D2            XOR EDX,EDX
00940127    8D4418 28       LEA EAX,DWORD PTR DS:[EAX+EBX+28]
0094012B    66:3B4B 06      CMP CX,WORD PTR DS:[EBX+6]                          ; segment number.NumberOfSections
0094012F    73 25           JNB SHORT 00940156
```

`call UPX shell code key.`

```
009401D4    8B4D F0         MOV ECX,DWORD PTR SS:[EBP-10]                       ; DHL-Inte.00400080
009401D7    8B41 28         MOV EAX,DWORD PTR DS:[ECX+28]
009401DA    01D8            ADD EAX,EBX
009401DC    FFD0            CALL EAX                           ;call UPX shell code.UPX shell code into the program code after decompression
```

这里是 `UPX` shell 代码, 在此处下断点直接到程序入口点. 代码特征如下

```
00417DE3   .  8D4424 80     LEA EAX,DWORD PTR SS:[ESP-80]
00417DE7   >  6A 00         PUSH 0
00417DE9   .  39C4          CMP ESP,EAX
00417DEB   .^ 75 FA         JNZ SHORT DHL-Inte.00417DE7
00417DED   .  83EC 80       SUB ESP,-80
00417DF0   .^ E9 CEB0FFFF   JMP DHL-Inte.00412EC3           ;under the main program entry point.F2 off
```
