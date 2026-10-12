
00457E2E 8B15 C85B4500 mov edx,dword ptr ds:[455BC8] ; YUCE.00455C14

00457E34 E8 2FE2FEFF call YUCE.00446068

00457E39 A1 048F4500 mov eax,dword ptr ds:[458F04]

00457E3E 8B00 mov eax,dword ptr ds:[eax]

00457E40 E8 A3E2FEFF call YUCE.004460E8

00457E45 E8 F6B8FAFF call YUCE.00403740

00457E4A 0000 add byte ptr ds:[eax],al

00457E4C FFFF ??? ; Unknown command

00457E4E FFFF ??? ; Unknown command

00457E50 0C 00 or al,0

00457E52 0000 add byte ptr ds:[eax],al

00457E54 C9 leave

00457E55 FA cli

00457E56 C3 retn

55 8B EC 83 C4 F4 B8 C8 7C 45 00 E8 44 E4 FA FF A1 04 8F 45 00 8B 00 BA 54 7E 45 00 E8 83 DE FE

FF 8B 0D 68 8D 45 00 A1 04 8F 45 00 8B 00 8B 15 C8 5B 45 00 E8 2F E2 FE FF A1 04 8F 45 00 8B 00

E8 A3 E2 FE FF E8 F6 B8 FA FF 00 00 FF FF FF FF 0C 00 00 00 C9 FA C3

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Borland Delphi 6.0 - 7.0

CODE

DATA

BSS

`.idata`

`.tls`

`.rdata`

`.reloc`

`.rsrc`

0047845C `pymf> 55 push ebp`

0047845D `8BEC mov ebp,esp`

0047845F `83C4 E8 add esp,-18`

00478462 `33C0 xor eax,eax`

00478464 `8945 EC mov dword ptr ss:[ebp-14],eax`

00478467 `8945 E8 mov dword ptr ss:[ebp-18],eax`

0047846A `B8 6C824700 mov eax,pymf.0047826C`

0047846F `E8 94E3F8FF call pymf.00406808`

00478474 `33C0 xor eax,eax`

00478476 `55 push ebp`

00478477 `68 21854700 push pymf.00478521`

0047847C `64:FF30 push dword ptr fs:[eax]`

0047847F `64:8920 mov dword ptr fs:[eax],esp`

00478482 `E8 0DD5FFFF call pymf.00475994`

00478487 `84C0 test al,al`

00478489 `75 18 jnz short pymf.004784A3`

0047848B `6A 24 push 24`

0047848D `68 30854700 push pymf.00478530 ; ASCII "Error"`

00478492 `68 38854700 push pymf.00478538 ; ASCII "YMF7x4 driver is not found. Start the program anyway?"`

00478497 `6A 00 push 0`

00478499 `E8 C2ECF8FF call <jmp.&user32.MessageBoxA>`

0047849E `83F8 06 cmp eax,6`

004784A1 `75 63 jnz short pymf.00478506`

004784A3 `8D55 E8 lea edx,dword ptr ss:[ebp-18]`

004784A6 `B8 01000000 mov eax,1`

004784AB `E8 BCA5F8FF call pymf.00402A6C`

004784B0 `8B45 E8 mov eax,dword ptr ss:[ebp-18]`

004784B3 `8D55 EC lea edx,dword ptr ss:[ebp-14]`

004784B6 `E8 3900F9FF call pymf.004084F4`

004784BB `8B45 EC mov eax,dword ptr ss:[ebp-14]`

004784BE `BA 78854700 mov edx,pymf.00478578 ; ASCII "-clean"`

004784C3 `E8 5CC3F8FF call pymf.00404824`

004784C8 `75 07 jnz short pymf.004784D1`

004784CA `E8 B1C7FFFF call pymf.00474C80`

004784CF `EB 35 jmp short pymf.00478506`

004784D1 `A1 E8B24700 mov eax,dword ptr ds:[47B2E8]`

004784D6 `8B00 mov eax,dword ptr ds:[eax]`

004784D8 `E8 A75BFEFF call pymf.0045E084`

004784DD `8B0D 00B14700 mov ecx,dword ptr ds:[47B100] ; pymf.0047CDC0`

004784E3 `A1 E8B24700 mov eax,dword ptr ds:[47B2E8]`

004784E8 `8B00 mov eax,dword ptr ds:[eax]`

004784EA `8B15 F85B4700 mov edx,dword ptr ds:[475BF8] ; pymf.00475C44`

004784F0 `E8 A75BFEFF call pymf.0045E09C`

004784F5 `A1 E8B24700 mov eax,dword ptr ds:[47B2E8]`

004784FA `8B00 mov eax,dword ptr ds:[eax]`

004784FC `E8 1B5CFEFF call pymf.0045E11C`

00478501 `E8 5ED6FFFF call pymf.00475B64`

00478506 `33C0 xor eax,eax`

00478508 `5A pop edx ; kernel32.7C816FD7`

00478509 `59 pop ecx ; kernel32.7C816FD7`

0047850A `59 pop ecx ; kernel32.7C816FD7`

0047850B `64:8910 mov dword ptr fs:[eax],edx ; ntdll.KiFastSystemCallRet`

0047850E `68 28854700 push pymf.00478528`

00478513 `8D45 E8 lea eax,dword ptr ss:[ebp-18]`

00478516 `BA 02000000 mov edx,2`

0047851B `E8 2CBFF8FF call pymf.0040444C`

00478520 `C3 retn`

55 8B EC 83 C4 E8 33 C0 89 45 EC 89 45 E8 B8 6C 82 47 00 E8 94 E3 F8 FF 33 C0 55 68 21 85 47 00

64 FF 30 64 89 20 E8 0D D5 FF FF 84 C0 75 18 6A 24 68 30 85 47 00 68 38 85 47 00 6A 00 E8 C2 EC

F8 FF 83 F8 06 75 63 8D 55 E8 B8 01 00 00 00 E8 BC A5 F8 FF 8B 45 E8 8D 55 EC E8 39 00 F9 FF 8B

45 EC BA 78 85 47 00 E8 5C C3 F8 FF 75 07 E8 B1 C7 FF FF EB 35 A1 E8 B2 47 00 8B 00 E8 A7 5B FE

FF 8B 0D 00 B1 47 00 A1 E8 B2 47 00 8B 00 8B 15 F8 5B 47 00 E8 A7 5B FE FF A1 E8 B2 47 00 8B 00

E8 1B 5C FE FF E8 5E D6 FF FF 33 C0 5A 59 59 64 89 10 68 28 85 47 00 8D 45 E8 BA 02 00 00 00 E8

2C BF F8 FF C3

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

MASM32 / TASM32

`.text`

`.rdata`

`.data`

`.rsrc`

00401000 `RVA.>/$ 6A 00 push 0 ; /pModule = NULL`

00401002 `|. E8 830A0000 call <jmp.&KERNEL32.GetModuleHandleA> ; \GetModuleHandleA`

00401007 `|. A3 07304000 mov dword ptr ds:[403007],eax`

0040100C `|. 6A 00 push 0 ; /lParam = NULL`

0040100E `|. 68 6C104000 push RVA.0040106C ; |DlgProc = RVA.0040106C`

00401013 `|. 6A 00 push 0 ; /hOwner = NULL`

00401015 `|. 68 00304000 push RVA.00403000 ; |pTemplate = "DIALOG"`

0040101A `|. FF35 07304000 push dword ptr ds:[403007] ; |hInst = NULL`

00401020 `|. E8 F9090000 call <jmp.&USER32.DialogBoxParamA> ; \DialogBoxParamA`

00401025 `|. 50 push eax ; /ExitCode = 0`

00401026 `\. E8 4D0A0000 call <jmp.&KERNEL32.ExitProcess> ; \ExitProcess`

0040102B `/. 55 push ebp`
