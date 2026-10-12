
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
