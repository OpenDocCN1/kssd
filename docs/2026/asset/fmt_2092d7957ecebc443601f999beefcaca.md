
```
  Mov Eax, IMAGE_SECTION_HEADER.VirtualAddress[Ebx + Esi]

  Add Eax, ImageBase[Edx]

  Mov Esi, Eax

  Mov Eax, IMAGE_IMPORT_DESCRIPTOR.FirstThunk[Esi]

  Add Eax, ImageBase[Edx]

  Mov ImportTableStart[Edx], Eax

  Mov Ecx, IMAGE_IMPORT_DESCRIPTOR.Name1[Esi]

  Add Ecx, ImageBase[Edx]

  Sub Ecx, 4

  Sub Ecx, Eax

  Shr Ecx, 2

  Mov ImportTabLen[Edx], Ecx

L2:

  Add Esi, SizeOf(IMAGE_IMPORT_DESCRIPTOR)

  Mov Eax, IMAGE_IMPORT_DESCRIPTOR.Name1[Esi]

  Add Eax, ImageBase[Edx]

  Cmp Dword Ptr [Eax], "resu"

  Jne L2

  Cmp Dword Ptr [Eax + 4], "d.23"

  Jne L2

  Cmp Dword Ptr [Eax + 6], "lld."

  Jne L2

  Mov Ecx, IMAGE_IMPORT_DESCRIPTOR.FirstThunk[Esi]

  Mov Esi, Eax

  Mov x1, 0

L3:

  Inc x1

  Invoke SearchChar, Esi

  Add Esi, Eax

  Invoke SearchZero, Esi

  Add Esi, Eax

  Cmp Dword Ptr [Esi], "sseM"

  jne L3

  Cmp Dword Ptr [Esi + 4], "Bega"

  jne L3

  Dec x1

  Mov Eax, 4

  Push Edx

  Mul x1

  Pop Edx

  Add Ecx, Eax

  Add Ecx, ImageBase[Edx]

  Mov pMessageBoxA[Edx], Ecx

  Mov Edi, FakeAddrOfEntryPoint[Edx]

PILoop1:

  Inc Edi

  Cmp Edi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Edi]

  Cmp Eax, "ipxE"   ; Expired!

  Jne PILoop1

  Mov Eax, [Edi + 4]

  Cmp Eax, "!der"

  Jne PILoop1

  Mov Eax, Edi

  Sub Eax, 3000 * 10

  Mov CodeReplaceCodeTable[Edx], Eax

  Sub Eax, 3000 * 4

  Mov CodeReplaceOffset[Edx], Eax

  Mov Edi, FakeAddrOfEntryPoint[Edx]

PILoop2:

  Inc Edi

  Cmp Edi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Edi]

  Cmp Eax, "MteG"       ;GetMachineID

  Jne PILoop2

  Mov Eax, [Edi + 4]

  Cmp Eax, "ihca"

  Jne PILoop2

  Mov Eax, [Edi + 8]

  Cmp Eax, "DIen"

  Jne PILoop2

  Mov Eax, Edi

  Sub Edi, 0A0H

  Sub Edi, 256 * 4

  Sub Edi, 256 * 4

  Mov CryptImportTable[Edx], Edi

  Sub Edi, 1000

  Mov APIPosition[Edx], Edi

  Sub Edi, 1000 * 4

  Mov ProcOffAddr[Edx], Edi

  Mov Edi, FakeAddrOfEntryPoint[Edx]

PILoop3:

  Inc Edi

  Cmp Edi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Edi]

  Cmp Eax, "_pca"

  Jne PILoop3

  Mov Eax, [Edi + 3]

  Cmp Eax, "ipa_"

  Jne PILoop3

  Add Edi, 7

  Mov ADPAPIStart[Edx], Edi

  Add Edi, acp_api_Len

  Mov ADPAPIEnd[Edx], Edi

  Mov Edi, FakeAddrOfEntryPoint[Edx]

  Add Edi, TempCodeTableOffset

  Mov TempCodeTable[Edx], Edi

  Mov Esi, FakeAddrOfEntryPoint[Edx]

PILoop4:

  Inc Esi

  Cmp Esi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Esi]   ; Randimize

  Cmp Eax, "dnaR"

  Jne PILoop4

  Mov Eax, [Esi + 4]

  Cmp Eax, "zimi"

  Jne PILoop4

  Add Esi, 0AH

  Lea Edi, AddressOfEntryPoint[Edx]

  Mov Ecx, 11

  Rep MovsD

  Mov Esi, FakeAddrOfEntryPoint[Edx]

PILoop5:

  Inc Esi

  Cmp Esi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Esi]   ; 替_换_代_码_zcf

  Cmp Eax, 0BB5FE6CCH

  Jne PILoop5

  Mov Eax, [Esi + 4]

  Cmp Eax, 0FAB45FBBH

  Jne PILoop5

  Mov Eax, [Esi + 8]

  Cmp Eax, 05FEBC25FH

  Jne PILoop5

  Mov Al, [Esi - 1]

  Mov DecodeKey[Edx], Al

  Mov Esi, FakeAddrOfEntryPoint[Edx]

PILoop6:

  Inc Esi

  Cmp Esi, SizeOfImage[Edx]

  Ja Error

  Mov Eax, [Esi]   ; RETRIVAPIZCF

  Cmp Eax, "RTER"

  Jne PILoop6

  Mov Eax, [Esi + 4]

  Cmp Eax, "PAVI"

  Jne PILoop6

  Mov Eax, [Esi + 8]

  Cmp Eax, "FCZI"

  Jne PILoop6

  Add Esi, 0EH

PILoop7:

  LodsD

  Test Eax, 40000000H

  Jnz PILoop7

  Sub Esi, 4

  Mov Eax, pMessageBoxA[Edx]

  Mov [Esi], Eax

  Pop Edi

  Pop Esi

  Pop Edx

  Pop Ecx

  Pop Ebx

  Ret

Error:

  Int 3

ProgInit EndP

Align 4

SearchZero Proc Source:DWORD

  Push Ecx

  Push Edi

  Mov Ecx, 0FFFFFFFFH

  Mov Edi, Source

  Xor Al, Al

  Repe Scasb

  Not Ecx

  Dec Ecx

  Mov Eax, Ecx

  Pop Edi

  Pop Ecx

  Ret

SearchZero EndP

Align 4

SearchChar Proc Source:DWORD

  Push Ecx

  Push Edi

  Mov Ecx, 0FFFFFFFFH

  Mov Edi, Source

  Xor Al, Al

  Repne Scasb

  Not Ecx

  Dec Ecx

  Mov Eax, Ecx

  Pop Edi

  Pop Ecx

  Ret

SearchChar EndP

Align 4

FillZero Proc Source:DWORD, Dest:DWORD

  Push Eax

  Push Ecx

  Push Edi

  Mov Eax, Source

  Add Eax, 5

  Mov Edi, Eax

  Mov Ecx, Dest

  Sub Ecx, Eax

  Xor Eax, Eax

  Rep Stosb

  Pop Edi

  Pop Ecx

  Pop Eax

  Ret

FillZero EndP

Align 4

BulidJmp Proc Source:DWORD, Dest:DWORD

  Push Eax

  Push Esi

  Mov Esi, Source

  Mov Byte Ptr [Esi], JmpCode

  Mov Eax, Dest

  Sub Eax, Source

  Sub Eax, 5

  Mov [Esi + 1], Eax

  Pop Esi

  Pop Eax

  Ret

BulidJmp EndP

Align 4

RemoveNop Proc Source:DWORD

  Push Ecx

  Push Edi

  Mov Edi, Source

RNLoop:

  Dec Edi

  Mov Al, [Edi]

  Cmp Al, 90H

  Jz RNLoop

  Inc Edi

  Mov Eax, Edi

  Pop Edi

  Pop Ecx

  Ret

RemoveNop EndP

Align 4

SearchEnd Proc Source:DWORD, Mark:DWORD

  Push Ebx

  Push Edi

  Mov Ebx, Mark

  Mov Edi, Source

SELoop:

  Inc Edi

  Mov Eax, [Edi]

  Cmp Eax, Ebx

  Jnz SELoop

  Mov Eax, Edi

  Pop Edi

  Pop Ebx

  Ret

SearchEnd EndP

;=============================================

RemoveCodeReplace Proc

S1:

  Invoke  GetCurrAddr

  Invoke  GetProgAddr

  Invoke  DecodeCryptCode

  Mov Esi, CodeReplaceCodeTable[Edx]

  Lodsb

  Cmp Al, 0

  Je EndProc

  Invoke MoveBadCode45

  Cmp Eax, 1

  Je S1

  Invoke MoveBadCode33

  Cmp Eax, 1

  Je S1

  Invoke MoveBadCode03

  Cmp Eax, 1

  Je S1

EndProc:

  Ret

RemoveCodeReplace Endp

RepairOEPCode Proc

  Push Ebx

  Push Ecx

  Push Esi

  Push Edi

  Call GetCurrAddr

  Mov Edi, AddressOfEntryPoint[Edx]

  Lea Esi, OEPStolenCode[Edx]

  Mov Ecx, 33

  Rep Movsb

  Mov Esi, AddressOfEntryPoint[Edx]

  Mov Eax, SaveEbp[Edx]

  Sub Eax, SaveEsp[Edx]

  Sub Eax, 9

  Not Eax

  Mov Byte Ptr [Esi + 5], Al

  Mov Eax, InitProgTable[Edx]

  Mov [Esi + 9], Eax

  Mov Eax, Sysinit_InitExe[Edx]

  Mov Ebx, Esi

  Add Ebx, 12H

  Sub Eax, Ebx

  Mov [Esi + 0EH], Eax

  Mov Eax, TApplication_instance[Edx]

  Mov Edi, BaseOfData[Edx]

  Mov Ecx, 0FFFFFFFFH

  Repne ScasD

  Sub Edi, 4

  Mov [Esi + 14H], Edi

  Mov Eax, aGoodName[Edx]

  Mov [Esi + 19H], Eax

; Mov Ebx, ImageBase[Edx]
; Mov Esi, IMAGE_DOS_HEADER.e_lfanew[Ebx]
; Mov Eax, AddressOfEntryPoint[Edx]
; Sub Eax, Ebx
; Mov IMAGE_NT_HEADERS.OptionalHeader.AddressOfEntryPoint[Ebx + Esi], Eax

  Pop Edi

  Pop Esi

  Pop Ecx

  Pop Ebx

  Ret

RepairOEPCode EndP

Align 4

RepairCall68Bxxx Proc

  Call GetCurrAddr

L0:

  Mov Edi, ProgStartAddr[Edx]

  Mov Ecx, CodeLen[Edx]

L1:

  Mov Al, 0E8H

  Repne Scasb

  Jecxz RepairEnd

  Mov Eax, [Edi]

  Add Eax, Edi

  Add Eax, 4

  Cmp Eax, ADPAPIStart[Edx]

  Jb L1

  Cmp Eax, ADPAPIEnd[Edx]

  Ja L1

  Cmp Word Ptr [Eax + 1], 25FFH

  Jne L1

  Mov ProgStartAddr[Edx], Edi

  Mov CodeLen[Edx], Ecx

  Mov Eax, [Eax + 3]

  Mov Eax, [Eax]

  Mov Edi, ImportTableStart[Edx]

  Mov Ecx, ImportTabLen[Edx]

  Repne Scasd

  Jecxz L2

  Mov Eax, Edi

  Sub Eax, 4

L1_1:

  Mov TmpCall.OAddr[Edx], Eax

  Call SearchJmpImport

  Or Eax, Eax

  Jz IsWrong

  Mov Eax, TmpCall.SAddr[Edx]

  Mov Edi, ProgStartAddr[Edx]

  Sub Eax, Edi

  Sub Eax, 4

  Stosd

  Sub DWord Ptr CodeLen[Edx], 4

  Jmp L0

RepairEnd:

  Return 1

L2:

  Mov Edi, ProgStartAddr[Edx]

  Sub Eax, Edi

  Sub Eax, 4

  Stosd

  Sub DWord Ptr CodeLen[Edx], 4

  Jmp L0

IsWrong:

  Return 0

RepairCall68Bxxx EndP

Align 4

RepairCall6886F3 Proc

  Call GetCurrAddr

  Mov Ebp, Edx

  Mov Esi, ProcOffAddr[Ebp]

  Xor Edx, Edx  ;Count

L0:

  Lodsd

  Add Eax, 400000H

  Mov Edi, Eax

  Mov Eax, APIPosition[Ebp]

  Movzx Ebx, Byte Ptr [Eax + Edx]

  Mov Eax, CryptImportTable[Ebp]

  Mov Eax, [Eax + Ebx * 4]

  Or Eax, Eax

  Jz LEnd

  Test Eax, 40000000H

  Jnz L1

  Mov Ebx, Eax

  Mov Eax, [Ebx + 1]

  Xor Eax, [Ebx + 8]

L1:

  Push Edi

  Mov Edi, ImportTableStart[Ebp]

  Mov Ecx, ImportTabLen[Ebp]

  Repne Scasd

  Jecxz L3

  Mov Eax, Edi

  Sub Eax, 4

  Pop Edi

  Mov TmpCall.OAddr[Ebp], Eax

  Call SearchJmpImport

  Or Eax, Eax

  Jz IsWrong

  Mov Eax, TmpCall.SAddr[Ebp]

  Sub Eax, Edi

  Mov [Edi - 4], Eax

L2:

  Inc Edx

  Cmp Edx, 1000

  Jnz Short L0

LEnd:

  Ret

L3:

  Pop Edi

  Sub Eax, Edi

  Mov [Edi - 4], Eax

  Jmp L2
```
