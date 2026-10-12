
# CD-check

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-61.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-61.htm)

**CD-check** 【习题】

1. **习题一** `chap6-1-6-01` 15K ( 需要 `cw3220.dll` ) 易
2. **习题二** `chap6-1-6-02` 5K 易
3. **习题三** `chap6-1-6-03` 6K ( 需要 `cw3220.dll` ) 中
4. **习题四** `chap6-1-6-04` 4K 中

## 1、习题一 答案

用 `W32DASM` 装载程序，反汇编，在串式数据参考查找 `"This program isn't being run from "`，如下：

```
* Reference To: KERNEL32.GetDriveTypeA, Ord:0000h

:00401087 E884040000              Call 00401510
:0040108C 8BC8                    mov ecx, eax
:0040108E BBAE3E0F00              mov ebx, 000F3EAE
:00401093 90                      nop
:00401094 40                      inc eax
:00401095 42                      inc edx
:00401096 47                      inc edi
:00401097 90                      nop
:00401098 48                      dec eax
:00401099 4A                      dec edx
:0040109A 4F                      dec edi
:0040109B 90                      nop
:0040109C 3BD9                    cmp ebx, ecx
:0040109E 7525                    jne 004010C5<----如没 CD 则跳走出错
:004010A0 6800100000              push 00001000
* Possible StringData Ref from Data Obj ->"Well Done"

:004010A5 6898204000              push 00402098
* Possible StringData Ref from Data Obj ->"This program is being run from "
                                  ->"a CD"

:004010AA 6874204000              push 00402074
:004010AF 6A00                    push 00000000
* Reference To: USER32.MessageBoxA, Ord:0000h

:004010B1 E86C040000              Call 00401522
* Possible StringData Ref from Data Obj ->"Correct CD In Drive"

:004010B6 68A2204000              push 004020A2
* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:0065, ""

:004010BB 6A65                    push 00000065
:004010BD 56                      push esi
* Reference To: USER32.SetDlgItemTextA, Ord:0000h

:004010BE E859040000              Call 0040151C
:004010C3 EB23                    jmp 004010E8
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040109E(C)
|

:004010C5 6800100000              push 00001000
* Possible StringData Ref from Data Obj ->"Bad Luck"

:004010CA 68F3204000              push 004020F3
* Possible StringData Ref from Data Obj ->"This program isn't being run from "
                                  ->"a CD, Please insert the CD"

:004010CF 68B6204000              push 004020B6
:004010D4 6A00                    push 00000000
* Reference To: USER32.MessageBoxA, Ord:0000h

:004010D6 E847040000              Call 00401522
* Possible StringData Ref from Data Obj ->"Invalid CD In Drive"

:004010DB 68FC204000              push 004020FC
```

因此，如下改动：

```
0040109E 7525                    jne 004010C5 to
0040109E 7425                    je  004010C5
```

或

```
0040109E 90                      nop
0040109F 90                      nop
```

## 2、习题二 答案

这程序利用了 `GetDriveTypeA` 检测光驱，还用 `CreateFileA` 找开光盘文件，如两者都 OK，则成功。

首先用 `W32DASM` 装载程序，用串式参考查找 `"You lost"`，来到：

```
* Reference To: KERNEL32.GetDriveTypeA, Ord:0104h

:00401349 FF1504204000            Call dword ptr [00402004]
:0040134F 83F803                  cmp eax, 00000003................如 EAX 是 3 则是硬盘
:00401352 743E                    je 00401392 -----------------------改成两个 NOP，
:00401354 8D45E8                  lea eax, dword ptr [ebp-18]
* Possible StringData Ref from Data Obj ->"CD_CHECK.DAT"

:00401357 6858304000              push 00403058
:0040135C 50                      push eax
:0040135D 8D45E0                  lea eax, dword ptr [ebp-20]
:00401360 50                      push eax
* Reference To: MFC42.Ordinal:039C, Ord:039Ch

:00401361 E822030000              Call 00401688
:00401366 8B00                    mov eax, dword ptr [eax]
:00401368 53                      push ebx
:00401369 53                      push ebx
:0040136A 53                      push ebx
:0040136B 53                      push ebx
:0040136C 6A01                    push 00000001
:0040136E 6800000080              push 80000000
:00401373 50                      push eax
* Reference To: KERNEL32.CreateFileA, Ord:0034h

:00401374 FF1500204000            Call dword ptr [00402000]
:0040137A 83F8FF                  cmp eax, FFFFFFFF................eax=-1 打开文件失败
:0040137D 8D4DE0                  lea ecx, dword ptr [ebp-20]
:00401380 0F9445F3                sete byte ptr [ebp-0D]
* Reference To: MFC42.Ordinal:0320, Ord:0320h

:00401384 E811030000              Call 0040169A
:00401389 385DF3                  cmp byte ptr [ebp-0D], bl
:0040138C 0F84F3000000            je 00401485 ......如打开文件成功则跳转，改成：jmp 00401485
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401352(C)
|

:00401392 FF45EC                  inc [ebp-14]
:00401395 83C704                  add edi, 00000004
:00401398 837DEC07                cmp dword ptr [ebp-14], 00000007
:0040139C 759F                    jne 0040133D
:0040139E 53                      push ebx
* Possible StringData Ref from Data Obj ->"Try again"

:0040139F 684C304000              push 0040304C
* Possible StringData Ref from Data Obj ->"You lost"

:004013A4 6840304000              push 00403040
```

这样改之后，成功 `crack`。

## 3、习题三 答案

`reversing a lame cd-check, pay attention boy! by R!SC -- risc@notme.com`

`starting from the top, cd-checks normally use kernel32!getdrivetypea to find out what sort of`

`drive they are looking at, you simply push a pointer to a drive letter, then after getdrivetypea`

`eax=03 for hard disk or eax=05 for a cd-rom.`

`so load crackcd.exe, enter softice, and type in 'bpx getdrivetypea'. Click on the CheckCD`

`button, and bingo! we have located the code to check the CD.. now pay close attention to the`

`comments in dead listing...`

```
015F:004011B3  68EB234000         PUSH    004023EB    ; pointer to 'kernel32.dll',0
015F:004011B8  E861070000         CALL    KERNEL32!LoadLibraryA
015F:004011BD  8BF0               MOV     ESI,EAX     ; save address of kernel32 in esi
015F:004011BF  8D85E8FBFFFF       LEA     EAX,[EBP+FFFFFBE8]
015F:004011C5  50                 PUSH    EAX         
015F:004011C6  68F8234000         PUSH    004023F8    ; pointer to getdrivetypea (wide char...)
015F:004011CB  E8C3020000         CALL    00401493    ; convert it to normal, pasting it into the free space
015F:004011D0  83C408             ADD     ESP,08
015F:004011D3  8D95E8FBFFFF       LEA     EDX,[EBP+FFFFFBE8]
015F:004011D9  52                 PUSH    EDX         
015F:004011DA  56                 PUSH    ESI         
015F:004011DB  E820070000         CALL    KERNEL32!GetProcAddress ; returns the address of 'getdrivetypea' in eax
015F:004011E0  6A00               PUSH    00          
```



### 015F:004011E2 FFD0 CALL EAX
`CALL EAX` ; call `getdrivetypea` (where softice will break)

### 015F:004011E4 8BF8 MOV EDI,EAX
`MOV EDI,EAX` ; `eax` will == 3 if the program is run from harddisk

### 015F:004011E6 56 PUSH ESI
`PUSH ESI`

### 015F:004011E7 E820070000 CALL KERNEL32!FreeLibrary
`CALL KERNEL32!FreeLibrary` ; free the kernel :) he didnt do it..

### 015F:004011EC 682C010000 PUSH 0000012C
`PUSH 0000012C` ; size of buffer to store windows directory

### 015F:004011F1 8D85BCFAFFFF LEA EAX,[EBP+FFFFFABC]
`LEA EAX,[EBP+FFFFFABC]`

### 015F:004011F7 50 PUSH EAX
`PUSH EAX` ; pointer to buffer

### 015F:004011F8 E8EB060000 CALL KERNEL32!GetWindowsDirectoryA
`CALL KERNEL32!GetWindowsDirectoryA`

### 015F:004011FD 6880000000 PUSH 00000080
`PUSH 00000080` ; size of buffer for modulefilename

### 015F:00401202 8D95E8FBFFFF LEA EDX,[EBP+FFFFFBE8]
`LEA EDX,[EBP+FFFFFBE8]`

### 015F:00401208 52 PUSH EDX
`PUSH EDX` ; pointer to buffer

### 015F:00401209 FF35B0254000 PUSH DWORD PTR [004025B0]
`PUSH DWORD PTR [004025B0]` ; `00400000`, this module...

### 015F:0040120F E8DA060000 CALL KERNEL32!GetModuleFileNameA
`CALL KERNEL32!GetModuleFileNameA`

### 015F:00401214 8A8DBCFAFFFF MOV CL,[EBP+FFFFFABC]
`MOV CL,[EBP+FFFFFABC]` ; `[c:\windows]` ;`WindowsDirectoryA`

### 015F:0040121A 3A8DE8FBFFFF CMP CL,[EBP+FFFFFBE8]
`CMP CL,[EBP+FFFFFBE8]` ; `[c:\checkcd1\checkcd.exe]` ;`ModuleFileNameA`

### 015F:00401220 7518 JNZ 0040123A
`JNZ 0040123A` ; check the drive letters...jump if not equal

yah, all the above code does is get the windows path/directory, get its own path/directory,

compare the drive letters, so it cant be run from the same drive that windows is installed on.

it also get's the drivetype of the current drive, and stores this in `EDI`...

### 015F:00401220 7518 JNZ 0040123A
`JNZ 0040123A` ; has to be taken, so change it to a `JMP`

```
eb18
```

on with the reversing...

### 015F:00401222 6A00 PUSH 00
`PUSH 00`

### 015F:00401224 6A00 PUSH 00
`PUSH 00`

### 015F:00401226 8D45B0 LEA EAX,[EBP-50]
`LEA EAX,[EBP-50]`

### 015F:00401229 50 PUSH EAX
`PUSH EAX` ; pointer to wide char 'HeHe! Try again'

### 015F:0040122A 53 PUSH EBX
`PUSH EBX`

### 015F:0040122B E81C020000 CALL 0040144C
`CALL 0040144C` ; convert wide char, display messagebox

### 015F:00401230 83C410 ADD ESP,10
`ADD ESP,10`

### 015F:00401233 33C0 XOR EAX,EAX
`XOR EAX,EAX`

### 015F:00401235 E909020000 JMP 00401443
`JMP 00401443` ; jump to exit (failed cd-check)

### 015F:0040123A 83EF05 SUB EDI,05
`SUB EDI,05` ; where we end up if we take the first good-check jump

### 015F:0040123D 0F8596010000 JNZ 004013D9
`JNZ 004013D9` ; 05=cd-rom, 05-05=0, so `edi` must be 0 to carry on.

### 015F:00401243 8D9574FCFFFF LEA EDX,[EBP-038C]
`LEA EDX,[EBP-038C]` ; total number of clusters

### 015F:00401249 52 PUSH EDX
`PUSH EDX` ; pointers

### 015F:0040124A 8D8D78FCFFFF LEA ECX,[EBP-0388]
`LEA ECX,[EBP-0388]` ; how many free clusters

### 015F:00401250 51 PUSH ECX
`PUSH ECX` ; to

### 015F:00401251 8D857CFCFFFF LEA EAX,[EBP-0384]
`LEA EAX,[EBP-0384]` ; bytes per sector

### 015F:00401257 50 PUSH EAX
`PUSH EAX` ; various

### 015F:00401258 8D9580FCFFFF LEA EDX,[EBP-0380]
`LEA EDX,[EBP-0380]` ; sectors per cluster

### 015F:0040125E 52 PUSH EDX
`PUSH EDX` ; buffers

### 015F:0040125F 6A00 PUSH 00
`PUSH 00` ; `pRootPathName`, null = current drive

### 015F:00401261 E8BE060000 CALL KERNEL32!GetDiskFreeSpaceA
`CALL KERNEL32!GetDiskFreeSpaceA`

### 015F:00401266 83BD78FCFFFF00 CMP DWORD PTR [EBP-0388],00
`CMP DWORD PTR [EBP-0388],00` ; compare the amount of free clusters with '0'

### 015F:0040126D 7418 JZ 00401287
`JZ 00401287` ; again, if it was run from CD, the freespace is always '0'

okay, still simple enough, subtracts 5 from the returned value for this drive, if its not equal,

i.e. not zero, jumps to the 'hehe try again' messagebox, if it passed this part, it checks for

free space, and there should be none on a CD, so ...

### 015F:0040123D 0F8596010000 JNZ 004013D9
`JNZ 004013D9` ; this jump has to be killed to carry on with the check

```
0f8500000000
```

### 015F:0040126D 7418 JZ 00401287
`JZ 00401287` ; this jump has to be taken to carry on...

```
eb18
```

on with the reversing...

### 015F:0040126F 6A00 PUSH 00
`PUSH 00`

### 015F:00401271 6A00 PUSH 00
`PUSH 00`

### 015F:00401273 8D4DB0 LEA ECX,[EBP-50]
`LEA ECX,[EBP-50]` ; de-ja-vu

### 015F:00401276 51 PUSH ECX
`PUSH ECX` ; pointer to wide char 'HeHe! Try again'

### 015F:00401277 53 PUSH EBX
`PUSH EBX`

### 015F:00401278 E8CF010000 CALL 0040144C
`CALL 0040144C` ; our message box friend again

### 015F:0040127D 83C410 ADD ESP,10
`ADD ESP,10`

### 015F:00401280 33C0 XOR EAX,EAX
`XOR EAX,EAX`

### 015F:00401282 E9BC010000 JMP 00401443
`JMP 00401443` ; jump to exit (failed cd-check)

### 015F:00401287 6880000000 PUSH 00000080
`PUSH 00000080`

### 015F:0040128C 8D95BCF9FFFF LEA EDX,[EBP+FFFFF9BC]
`LEA EDX,[EBP+FFFFF9BC]`

### 015F:00401292 52 PUSH EDX
`PUSH EDX`

### 015F:00401293 8D8D68FCFFFF LEA ECX,[EBP-0398]
`LEA ECX,[EBP-0398]`

### 015F:00401299 51 PUSH ECX
`PUSH ECX`

### 015F:0040129A 8D856CFCFFFF LEA EAX,[EBP-0394]
`LEA EAX,[EBP-0394]`

### 015F:004012A0 50 PUSH EAX
`PUSH EAX`

### 015F:004012A1 8D9570FCFFFF LEA EDX,[EBP-0390]
`LEA EDX,[EBP-0390]`

### 015F:004012A7 52 PUSH EDX
`PUSH EDX`

### 015F:004012A8 6880000000 PUSH 00000080
`PUSH 00000080`

### 015F:004012AD 8D8D3CFAFFFF LEA ECX,[EBP+FFFFFA3C]
`LEA ECX,[EBP+FFFFFA3C]`

### 015F:004012B3 51 PUSH ECX
`PUSH ECX`

### 015F:004012B4 6A00 PUSH 00
`PUSH 00`

### 015F:004012B6 E839060000 CALL KERNEL32!GetVolumeInformationA
`CALL KERNEL32!GetVolumeInformationA`

### 015F:004012BB 81BD70FCFFFF21787573 CMP DWORD PTR [EBP-0390],73757821
`CMP DWORD PTR [EBP-0390],73757821` ; compare read label with 'sux!'

### 015F:004012C5 0F85F9000000 JNZ 004013C4
`JNZ 004013C4` ; jump if not equal to failed check...

### 015F:004012CB F68568FCFFFF10 TEST BYTE PTR [EBP-0398],10
`TEST BYTE PTR [EBP-0398],10` ; i dont know, but it has to be equal :)

### 015F:004012D2 7418 JZ 004012EC
`JZ 004012EC` ; jump passed horrid message if above test is true

### 015F:004012D4 6A00 PUSH 00
`PUSH 00`

### 015F:004012D6 6A00 PUSH 00
`PUSH 00`

### 015F:004012D8 8D45B0 LEA EAX,[EBP-50]
`LEA EAX,[EBP-50]` ;de-ja-vu again

### 015F:004012DB 50 PUSH EAX
`PUSH EAX` ; see the pattern, `ebp-50`, call `40144c`?

### 015F:004012DC 53 PUSH EBX
`PUSH EBX`

### 015F:004012DD E86A010000 CALL 0040144C
`CALL 0040144C`

### 015F:004012E2 83C410 ADD ESP,10
`ADD ESP,10`

### 015F:004012E5 33C0 XOR EAX,EAX
`XOR EAX,EAX`

### 015F:004012E7 E957010000 JMP 00401443
`JMP 00401443`

### 015F:004012EC F68569FCFFFF80 TEST BYTE PTR [EBP-0397],80
`TEST BYTE PTR [EBP-0397],80` ; i dont know, but it has to be equal :)

### 015F:004012F3 7418 JZ 0040130D
`JZ 0040130D` ; jump passed horrid message if above test is true

### 015F:004012F5 6A00 PUSH 00
`PUSH 00`

### 015F:004012F7 6A00 PUSH 00
`PUSH 00`

### 015F:004012F9 8D55B0 LEA EDX,[EBP-50]
`LEA EDX,[EBP-50]` ; we know this is the bad cracker bit

### 015F:004012FC 52 PUSH EDX
`PUSH EDX` ; so we take the above jump...

### 015F:004012FD 53 PUSH EBX
`PUSH EBX`

### 015F:004012FE E849010000 CALL 0040144C
`CALL 0040144C`

### 015F:00401303 83C410 ADD ESP,10
`ADD ESP,10`

### 015F:00401306 33C0 XOR EAX,EAX
`XOR EAX,EAX`

### 015F:00401308 E936010000 JMP 00401443
`JMP 00401443`

right, i have been a very naughty boy, and admit that two parts of this code i dont understand,

but from the bad-cracker code being duped everywhere, i know to take the jumps.. basicaly, this

is a label check, chacks the disk label against 'sux!', and if they are the same, does these

other two tests, which have to pass to skip the 'hehe! try again' message..



015F:004012C5 0F85F9000000 JNZ 004013C4 ; label check, we want to skip this jump...
0f8500000000

015F:004012D2 7418 JZ 004012EC ; dont know, but it needs taking (JMP)
eb18

015F:004012F3 7418 JZ 0040130D ; still dont know, but take it (JMP)
eb18

on with the reversing...

015F:0040130D 6814244000 PUSH 00402414 ; some bullshit 'stack overflow'
015F:00401312 8D953CFAFFFF LEA EDX,[EBP+FFFFFA3C] ;[ebp+fffffa3c] is from getvolumeinformationa routine
015F:00401318 52 PUSH EDX ; some more bullshit 'OVERFLOW'
015F:00401319 E888050000 CALL 004018A6 ; dont care :(
015F:0040131E 83C408 ADD ESP,08
015F:00401321 85C0 TEST EAX,EAX ; fuckit, change this to xor eax,eax
015F:00401323 0F8586000000 JNZ 004013AF ; so this jump wont be taken cause at 4013af, is the bad check code
015F:00401329 681A244000 PUSH 0040241A ; pointer to a new label
015F:0040132E 6A00 PUSH 00 ; what drive to change
015F:00401330 E8DD050000 CALL KERNEL32!SetVolumeLabelA
015F:00401335 48 DEC EAX ; eax = 1 if function succeed
015F:00401336 7526 JNZ 0040135E ; you cant change a CD's label, so eax should be 0 or -1
; but after the dec eax, if it failed, this jump would be taken..
015F:00401338 8D8D3CFAFFFF LEA ECX,[EBP+FFFFFA3C] ; waste
015F:0040133E 51 PUSH ECX ; of
015F:0040133F 6A00 PUSH 00 ; time
015F:00401341 E8CC050000 CALL KERNEL32!SetVolumeLabelA
015F:00401346 6A00 PUSH 00
015F:00401348 6A00 PUSH 00
015F:0040134A 8D45B0 LEA EAX,[EBP-50] ; bad cracker routine
015F:0040134D 50 PUSH EAX
015F:0040134E 53 PUSH EBX
015F:0040134F E8F8000000 CALL 0040144C ; again :)
015F:00401354 83C410 ADD ESP,10
015F:00401357 33C0 XOR EAX,EAX
015F:00401359 E9E5000000 JMP 00401443
015F:0040135E 6A00 PUSH 00
015F:00401360 8D55A8 LEA EDX,[EBP-58]
015F:00401363 52 PUSH EDX ;pointer to a filename
015F:00401364 E891050000 CALL KERNEL32!_lcreat ;try to create a file
015F:00401369 83F8FF CMP EAX,-01 ; would fail, if it tried to create a file on a CD
015F:0040136C 7428 JZ 00401396 ; as they are read-only, so force this jump...
015F:0040136E 50 PUSH EAX
015F:0040136F E892050000 CALL KERNEL32!_lclose ; whoops, we succeeded, so close the newly created file-handle
015F:00401374 8D4DA8 LEA ECX,[EBP-58]
015F:00401377 51 PUSH ECX
015F:00401378 E835050000 CALL 004018B2
015F:0040137D 59 POP ECX
015F:0040137E 6A00 PUSH 00
015F:00401380 6A00 PUSH 00
015F:00401382 8D45B0 LEA EAX,[EBP-50] ;DEJAVU again :0
015F:00401385 50 PUSH EAX
015F:00401386 53 PUSH EBX
015F:00401387 E8C0000000 CALL 0040144C ; messagebox
015F:0040138C 83C410 ADD ESP,10
015F:0040138F 33C0 XOR EAX,EAX
015F:00401391 E9AD000000 JMP 00401443
015F:00401396 6A00 PUSH 00 ; if we couldnt create the file, we end up here
015F:00401398 6823244000 PUSH 00402423 ; pointer to 'y.e.p.'
015F:0040139D 8D55D4 LEA EDX,[EBP-2C]
015F:004013A0 52 PUSH EDX ; pointer to 'y.a.y.!. .y.o.u. .c.r.a.c.k.e.d. .i.t.!'
015F:004013A1 53 PUSH EBX
015F:004013A2 E8A5000000 CALL 0040144C ; messagebox...
015F:004013A7 83C410 ADD ESP,10
015F:004013AA E992000000 JMP 00401441

hmm, the bit at the start, stack overflow, dont understand what or why, but kill this jump

015F:00401323 0F8586000000 JNZ 004013AF
0f8500000000

you cant change the label of a CD, so this jump needs forcing

015F:00401336 7526 JNZ 0040135E
eb26

last but not least, you cant create a file on a CD, so this jump needs forcing aswell..

015F:0040136C 7428 JZ 00401396
eb28

wayhey, then we finally reach the good boy message box!! yippee!!!

after studying the code, and realising what has been done, it just does some simple checks,

then compares the return codes against what they should be, we dont want it re-labeling our

hard disk drive to 'overflow', or creating a file called 'my.dog', basically, we can skip the

whole routine, and just end up at the goodboy message box..so bpx at the first instruction

015F:004011B3 68EB234000 PUSH 004023EB ; pointer to 'kernel32.dll',0

and re-assemble it to jump to the good-boy message box..

```
a 4011b3 <ret> 
jmp 401396 <ret> 
<esc> 
x <ret> 
015F:004011B3 E9DE010000 JMP 00401396 ; skip whole of check, go straight to jail 
; do not pass go, do not collect 200.. 
:) 
```

then you have a almost cracked checkcd.exe... just gotta patch it, but i cant be bothered, so 

i used my process patcher to create a loader for it..(availble from http://csir.xxx.xxx :) 

no plugz.. :) 

happy reversing / cracking / whatever.. 

R!SC 6/6/99  

### 4、习题四 答案

How to crack R!SC's Play The Game CD-Check Crackme by Killer_3K [DSi/Shock]

Tools: Sice & a mempatcher (i use R!SC's process patcher ;p)

hey there, in this tut i'll teach u how to crack risc's PTG (play the game)

CD-Check crackme. This Crackme is pretty nice, it detects sice (via int68),

sorta hidden crc-check, packed, fake conditional jumps that leads to crash

and more interesting stuff ;p

btw, don't bather unpacking it (it's packed w/ upx), as the readme says that

ur not allowed to unpack in order to patch, it doesn't really matter anywayz,

cause the way he fucked around w/ it about 96% of the code u'll get after dasm

will be garbadge :P

ok lets get started :)

fireup the crackme.. Doh, we get a msgbox saying "Kill Softice Mr. Cracker"

ok lets get rid of it :) since the first time i got that crackme i didn't

know how it detected it, so i'll tell u how i figured out how to kill the sice

w/o knowing it uses int68 :)

1) bpx on `GetModuleHandleA` and run the crackme, sice pops, but we see Explorer 

in the down-right corner, we don't want Explorer now do we :) Press F5 again 

till u see 'Play the' in the down-left corner, ok, press F11 and start tracing :) 

u should see this: 

```
0177:00401143 68F0104000 PUSH 004010F0
0177:00401148 50 PUSH EAX
0177:00401149 E818060000 CALL KERNEL32!GetProcAddress
0177:0040114E A3B2204000 MOV [004020B2],EAX
0177:00401153 33C0 XOR EAX,EAX
0177:00401155 7533 JNZ 0040118A
0177:00401157 3BF6 CMP ESI,ESI
0177:00401159 68E7104000 PUSH 004010E7
0177:0040115E E80F060000 CALL KERNEL32!GetModuleHandleA
0177:00401163 68FE104000 PUSH 004010FE
```



### 1. Initial Trace: Calling `GetProcAddress`

`0177:00401168  50`  PUSH  EAX  
`0177:00401169  3BF6`  CMP  ESI,ESI  
`0177:0040116B  E8F6050000`  CALL  `KERNEL32!GetProcAddress`  
`0177:00401170  A3B2204000`  MOV  `[004020B2]`,EAX  

...

ok, lets trace abit till we pass

`0177:00401168  50`  PUSH  EAX  
`0177:00401169  3BF6`  CMP  ESI,ESI  
`0177:0040116B  E8F6050000`  CALL  `KERNEL32!GetProcAddress`  
`0177:00401170  A3B2204000`  MOV  `[KERNEL32!AddAtomW]`,EAX  
`0177:00401175  C70530204000433A2F00`  MOV  DWORD PTR `[00402030]`,002F3A43  
`0177:0040117F  688A114000`  PUSH  0040118A  
`0177:00401184  FF2507214000`  JMP  `[00402107]`  <<---

ok, lets trace and pass the jmp

u should now see this:

`0177:00401442  33D2`  XOR  EDX,EDX  
`0177:00401444  3BF6`  CMP  ESI,ESI  
`0177:00401446  7401`  JZ  00401449  
`0177:00401448  Bd686C1440`  MOV  EBP,40146C68  

. and some junk code after it

the jz is gonna jump, let it jump, or else the proggi will crash :]

after the jz is taken, the code changed abit, and will change abit after

couple of lines u trace.. u should now see this (maybe it will change abit

during tracing :)):

`0177:00401449  686C144000`  PUSH  0040146C <-- will change to Add `[edx]`, BH after we traced it  
`0177:0040144E  3AC0`  CMP  AL,AL <-- will change to INVALID after we traced it  
`0177:00401450  7401`  JZ  00401453  

.

ok this jz must be taken as well, or proggi will crash :)

after it comes an interesting piece of code (which change after u trace):

`0177:00401453  64FF32`  PUSH  DWORD PTR FS:`[EDX]`  
`0177:00401456  8925A9204000`  MOV  `[004020A9]`,ESP  
`0177:0040145C  892DAD204000`  MOV  `[004020AD]`,EBP  
`0177:00401462  648922`  MOV  FS:`[EDX]`,ESP  
`0177:00401465  3ADB`  CMP  BL,BL  
`0177:00401467  7401`  JZ  0040146A  (JUMP )

hmm the jz wants to jump here too (i wonder why ;) (note the cmp bl,bl))

this time we don't have to make it jump, nop it or patch it to `7400` and the anti-sice

is gone (btw u gotta patch it, as the crackme executes that piece of code over and over..)

ok, ur prolly wondering why it doesn't detect sice now.. welp that jz leads us to the

`is_sice_there` routine..

lets take a look at that routine

after u'll take the jz u'll reach

`0177:0040146A  EB20`  JMP  0040148C

which will lead us to a VERY interesting piece of code (will keep changing during tracing):

`0177:0040148C  663BF6`  CMP  SI,SI  
`0177:0040148F  7401`  JZ  00401492 (jump)  <<  
`0177:00401492  B443`  MOV  AH,43 ; move `0x43` to AH  
`0177:00401494  CD68`  INT  68 ; int68 (no shit ;))  
`0177:00401496  5A`  POP  EDX  
`0177:00401497  3BD2`  CMP  EDX,EDX  
`0177:00401499  7401`  JZ  0040149C  (jump) <<  
`0177:0040149C  646789160000`  MOV  FS:`[0000]`,EDX  
`0177:004014A2  3BF6`  CMP  ESI,ESI  
`0177:004014A4  7401`  JZ  004014A7  (jump) <<  
`0177:004014A7  5A`  POP  EDX  
`0177:004014A8  3BD2`  CMP  EDX,EDX  
`0177:004014AA  7401`  JZ  004014AD  (jump) <<  
`0177:004014AD  663D86F3`  CMP  AX,F386 !!!  
`0177:004014B1  58`  POP  EAX  
`0177:004014B2  7402`  JZ  004014B6  (jump) <<<

now, this is the anti-ice part :) u can nop ANY jz from 40148F until 4014B2, and it will kill

the anti-ice, another way to kill it will be nopping the INT 68, anyway will fit here :)

if u'll keep tracing after 4014B2 w/o touching the jumps/int68 u'll reach this piece of code:

`0177:004014B8  7401`  JZ  004014BB  (JUMP)  
`0177:004014B6  3BF6`  CMP  ESI,ESI  
`0177:004014BB  68A5104000`  PUSH  004010A5  
`0177:004014C0  C3`  RET

which will lead u to:

`0177:004014C1  E85F020000`  CALL  00401725  
`0177:004010A5  6A00`  PUSH  00  
`0177:004010A7  6897104000`  PUSH  00401097 ; title (type `d 401097` in sice and u'll see doh! in widechar)  
`0177:004010AC  6863104000`  PUSH  00401063 ; msg (type `d 401063` in sice an u'll see Load anti-ice.. in widechar)  
`0177:004010B1  6A00`  PUSH  00  
`0177:004010B3  E8A8060000`  CALL  `USER32!MessageBoxW` ; msgbox  
`0177:004010B8  6A00`  PUSH  00  
`0177:004010BA  E8B9060000`  CALL  `KERNEL32!ExitProcess` ; exit proggi

anywayz if u'll patch like i showed u above we'll never reach that place :)

ok, to make it ez on urself, make a mempatcher to patch the anti-ice (i used R!SC's Process Patcher :))

### 2. Patching the CD Check

ok, now we can get to the real part of the protection, run ur loader (which will load&patch the exe :))

u'll see:

`No Disk inserted` (if ur cd-drive is empty) or `Wrong disk inserted` (if u got a cd in ur drive)

ok, lets kill that lovely cdcheck :)

put a `bpx` on `GetDriveTypeA` and run the proggi, u should see this:

`0177:004011A5  FF15B2204000`  CALL  `[KERNEL32!GetDriveTypeA]`  
`0177:004011AB  83F805`  CMP  EAX,05 <-- !!  
`0177:004011AE  75DA`  JNZ  0040118A

ok, this part checks to see if a drive is a cddrive (if it is, `eax=5`),

change it to `CMP EAX,03` so it will think HDD=cddrive

type `bc*` and `bpx` on `GetVolumeInformationA`, sice will break, press F11, u'll see:

`0177:004011F7  3BDB`  CMP  EBX,EBX  ; compare ebx w/ ebx heh  
`0177:004011F9  7401`  JZ  004011FC ; obviously a fake coditional jump  
`0177:004011FC  FE05B1204000`  INC  BYTE PTR `[004020B1]`  
`0177:00401202  85C0`  TEST  EAX,EAX  
`0177:00401204  0F847D000000`  JZ  00401287  (won't jump)  
`0177:0040120B  BE59204000`  MOV  ESI,00402059  
`0177:00401210  BF2F114000`  MOV  EDI,0040112F  
`0177:00401215  B907000000`  MOV  ECX,00000007  
`0177:0040121A  F3A6`  REPZ CMPSB ; compare labels (good label on edi, label from hdd on esi)  
`0177:0040121C  755D`  JNZ  0040127B ; label's rn't the same, goto badboy (will push the Wrong disk text)  
`0177:0040121E  E9B8000000`  JMP  004012DB ; this will leads to the 'goodboy' part, and then creates the dialog

### 3. Handling the Checksum

ok, we patched `CMP EAX,05` to `CMP EAX,03`, and nopped `JNZ 40127B` (or just patched it to `7500` which will make it jump to the next byte)

ok, type `bc*` and press F5.. WTF?! `Try harder!! You WILL crackme!`

checksum.. hmm, ok update the mempatcher w/ the cdcheck, and lets get ready to kill that checksum

put back that `bpx` on `GetDriveTypeA`, ok sice breaks, the `CMP EAX,05` should be patched, if not the patch it w/ sice

type `BC*` and then type `BPM 004011AB R`, this will make sice break when the crackme will try to read that addi

press F5.. Boom, sice breaks, the proggi is trying the read the part, i wonder why ;)

u should see this piece of code:

`0177:004014F9  3AC0`  CMP  AL,AL  
`0177:004014FB  7401`  JZ  004014FE ; fake conditional jump  
`0177:004014FE  03040E`  ADD  EAX,`[ECX+ESI]` <-- add the byte located in Current Location+Counter to EAX  
`0177:00401501  663BF6`  CMP  SI,SI <<--- u start here  
`0177:00401504  7401`  JZ  00401507  ; fake conditinal jump  
`0177:00401507  49`  DEC  ECX ; decrease counter



### 0177:00401508

```
0177:00401508  75EF                   JNZ      004014F9 ; are we done? if no then do the whole thing again
```

As you can see, this basically reads a byte located at `4014FE + Counter`, adds it to `eax`, and does it till `counter = 0`.

To make it a bit faster, type `bc*` and set `bpx` on `40150A`, then press `F5`.

You'll reach this very interesting piece of code:

### 0177:0040150A

```
0177:0040150A  59                    POP      ECX
0177:0040150B  5E                    POP      ESI
0177:0040150C  663BFF                CMP      DI,DI
0177:0040150F  7401                  JZ       00401512 (jump) ; fake conditional
0177:00401512  3D31B88FE0            CMP      EAX,E08FB831 ; compare result of the checksum w/ a hardcoded result (which is correct one)
0177:00401517  7543                  JNZ      0040155C ; << results rn't the same, BADBOY ;p
0177:00401519  3AD2                  CMP      DL,DL
0177:0040151B  7401                  JZ       0040151E ; fake conditonal (which will change the code below to RET)
0177:0040151E  C3                    RET      ; we're done, no harm was been done ;)
```

Weee!

Just `NOP` the `JNZ` at `401517` (or patch it to `7500`) and we're done! :)

I hope you enjoyed reading this tutorial and that you learned something new.

Here is my memory patch script (compile with R!SC's process patcher ;p):

---

```
T=60000:

f=Playth~1.exe:       ; filename
o=PTG_loader.exe:     ; loader to create
p=401467/74,01/74,00:    ; da anti-ice
p=4011AB/83,F8,05/83,F8,03:  ; make the crackme think hdd=cdrom
p=40121C/75,5D/75,00:   ; DriveVolume Check
p=401517/75,43/75,00:   ; did i fuck around w/ the cd check? sure i didn't ;)

$
```

---

Greetings to (no specific order): `R!SC`, `Acid_Burn`, `AB4DS`, `Kathras`, `Whizkid`, `Ordoc`, `Aerosmith`, `primevil`, `Senat0r`, `Cyclonex`, `Coffee`, `TORN@DO`.

Also a SPECIAL-u-r-a-piece-of-shit-lamer goes to: `Wekeke` (aka `Loko-Man`).

Thanks for all the funny logs *g*.

Welp, that's about it. If I forgot your name, don't get pissed; I sorta wrote that part in a hurry.

Cya all :)

-Killer_3K [DSi/Shock]

`4dd466d0e1e0cc123afbe504da2462ca`
