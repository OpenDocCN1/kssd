

# Asprotect by Tsehp

> 原文：[`www.pediy.com/kssd/pediy02/forum162.htm`](https://www.pediy.com/kssd/pediy02/forum162.htm)

|  ||  Choose a TITLE and a subtitle, choose well! Manually unpacking Asprotect version 1.0  
The encrypted import table ||  Choose a PROJECT GIF, leave this if unsure    
 |
| --- | --- | --- |
|   CHOOSE A DATE (will probably be changed) June 2000  || by   CHOOSE A HANDLE , i.e. your pseudo (wont be changed)  Tsehp ||  <a href="hcu98\_3.htm"><IMG SRC="hcu1.gif" ALT="+cracker" ALIGN=BOTTOM WIDTH=114 HEIGHT=43 BORDER=0 VSPACE=0 HSPACE=0></a>   |
|  || Courtesy of Reverser's page of reverse engineering  ||  Your truly+ will edit only if really necessary  slightly edited   
by reverser+  | this is for the data.....fra\_00xx....yymmdd....handle..beg+int...not ass...  
|  | Leonard Coehn's old song, because we are poets, not only crackers  | **There is a crack, a crack in everything That's how the light gets in**  Leonard Coehn's old song, because we are poets, not only crackers   ||  |
| Rating  ||   CHOOSE A RATING (may be changed)  ( )**Beginner** (x)**Intermediate** (x)**Advanced** ( )**Expert**  ||  |

 END HEAD   
 CORPUS  CHOOSE A COMMENT (may be changed) 

* * *

  Repeat your TITLE Manually unpacking Asprotect version 2000  
  Repeat your SUBTITLE  The encrypted import table  
  REPEAT YOUR CHOSEN HANDLE HERE Written by Tsehp  
  
 INTRO STARTS HERE  

| Introduction   |
| --- |

```

Asprotect is a well know target, this packer was studied a lot and often used as a target practice for newbies.
If you don't know anything about packing, improve your knowledge on the past essays, then come back, I'll try to
explain you their last tricks.
 PASTE HERE YOUR INTRODUCTION I use to find this target while looking for the sandman's newbie msgboard.
Thanks to a reverser called SV that explained very well the protection on his threads, his work made me save
a lot of time.
 TOOLS STARTS HERE 
```

```
Important : This essay can bring you a working app with your actual system, but the import table is not
rebuilt, so if you update your OS and the dll's addresses changes, it will not work anymore and you'll have
to do the entire process again. I'm actually working on another essay to finalize the reconstruction of the
import table.
```

| Tools required   |
| --- |

Softice 4.05  
Icedump  
Procdump  
Ida 4.04  
An hex editor  PASTE HERE YOUR REQUIRED TOOLS   
  
Essay made on win 2000, expect to change (later in this essay) some import addresses if you're working on win 9x.  
 TARGET URL STARTS HERE 

| Target's URL/FTP   |
| --- |

[  DON'T FORGET TO PASTE HERE THE URL/FTP OF YOUR TARGET(S) http://www.tamos.com/](http://www.tamos.com/)  
This is Commview 2.1, another portsniffer.   
  
 PROGRAM HISTORY STARTS HERE  REAL ESSAY STARTS HERE 

| Essay  |
| --- |

 PASTE HERE THE TEXT OF YOUR ESSAY THIS IS OF COURSE THE MOST IMPORTANT PART PLEASE CHECK THE MARGINS WHEN YOU ARE FINISHED! SHOULD NOT BLAST OPERA'S MARGINS OUT! HAVE A LOOK INSIDE YOUR OWN BROWSER WHEN YOU FINISH! 

 

```
1) The anti-softice routines
```

```
Before lauching this app, you have to hide your debugger.
This prog uses two typical anti softice tricks:
```

```
\-Is the driver loaded ? : bpx createfileA do "d esp-4"
```

```
Commview tries to see if three different versions of softice are loaded. You will hit three times :
SIWDEBUG
SICE
NTICE
```

```
Well it depends on the system you're on, press F12 at each time, the createfilea must return 0xFFFFFFFF
If not, press F12 again and set your eip to the address pointed by the second jnz below, you will pass all the
tests.
```

```
\-The boundschecker trick.
Sice uses int3 for its breakpoints, but when special values are set to register, especially ebp containing
'BCHK' 0x4243484b, then if an int3 is encountered, the normal interrupt handler just returns after this
Int3 inside your target, so the code just after is executed if sice is present.
To avoid this, press F10 several times after setting your eip, go slowly through the code until you see the
int3 in sice. When you're on it, change your ebp to something else than 0x4243484b, then the program will run
normally until a last check detected by bpx createfilea, just as above.
```

```
For the final check, just invert the jz after the createfileA, all the tests are now defeated.
```

```
Note : I used to munually do that, because frogsice is not actually implemented on win NT, the actual copy on
nt called Ntall is not fully working, so I couldn't use a loaded device to avoid those manipulations.
```

```
2) Locating the Entry point.
```

```
If you want to dump your packed program, you must stop the target here, so all the sections would be loaded.
I have to admit that I directly used the technique that Eternal Bliss found on the newbie's forum. Here it is, 
untouched :
```

```
Steps:
1) bpx on getprocaddress
2) Once you break, look at the .idata section by "dd 4E3000". This is found by doing "map32 cv"
3) bpm on the 1st byte (just to break there) and disable getprocaddress breakpoint
4) F5 and let the program run
5) Upon breaking, disable all breakpoints
6) F12 4 times and you will land at a Xor EAX, EAX
7) Start tracing with F10
8) Everytime you come across a CALL, check that a few instructions below are JMP
9) If it is, you can trace over it
10) If it is a ret, you have to trace into that CALL with F8
11) Then step over the 2 subsequent CALLs
12) You will see the OEP moved into EAX somewhere there
13) Then you will come across a POPAD
14) The JMP stuff appears again like the start of the unpacking code.
15) Trace into the next (3rd) Call and F8 all the way till you hit a ret
16) Continue with it and you will land in on the OEP.

Just apply this and you will find it, 0x4de384 on my system. Just remember that you will find almost all the
time a POPAD instruction just before the final jump.
```

```
Just as an exercise, you can try to dump the exe, using icedump and procdump to rebuild the pe (this will not
be explained here, if you don't know how to do this, just look at the past packer/unpacker essays).
```

```
You have to manually hex copy the .aspr original section to your dumped file to continue the essay, this
manipulation will just replicate the partial import table to your dumped exe, just like the packed commview.
```

```
But this will not work, just try to load your dumped exe inside ida and you will notice that the import table
is damaged, some virtual addresses in other sections are also pointed to wrong places.
```

```
The main protection of Asprotect is just here : Some imports are fine, some other are pointing to a table (located
in mem at 0x00CDxxxx) normally containing some decrypted import's addresses, and this table originally contains
also some routines to decrypt other addresses when commview needs them, even if this prog is unpacked.
```

```
Well we have here a big problem :
You can manually fix the import table if you convert all the tables addresses to the .idata at the right place.
```

```
So if inside you dumped app, inside .idata you have
```

```
4e328c : jmp \[cd4950\] and cd4950 containing 77db858e (a valid import address)
```

```
you have to convert the jump at 4e328c to directly point at 77db858e.
```

```
You will spend a lot of time, then it will not work. Because inside the cdxxxx table, some addresses points
to code like this :
cd4950 push cd495a
       call 182f68 (for example)
```

```
Whats happening ? When commview does call 4e328c (import table) the routine located at cd4950 just decrypts
the right import adress AT RUNTIME ! You can also try to dump the cdxxxx table, the decrypting code will
still be here and definitely not working. This feature is the real improvement made since the last Asprotect 
versions.
```

```
To summarize, four different things happens to the import table : 4 different cases in my following prog
1- api's address is copied directly to .idata
2- api's address is relocated to the Cxxxxx table
3- decypting routine is copied to the Cxxxxx table
4- an indirect call to GetprocAddress (in Cxxxxx table) is copied to .idata
```

```
3) Preparing to dump
```

```
You are still on the int3 instruction, just do a S 0 L FFFFFFFF  AC 08 C0 74 E4 and you will localize the
\*culprit\* loop that builds the import table, just do a bpmb at the start.
```

```
Here is what you find :
```

```
001B:0018A51E  AC                  LODSB
001B:0018A51F  08C0                OR        AL,AL
001B:0018A521  74E4                JZ        0018A507
001B:0018A523  4E                  DEC       ESI
001B:0018A524  56                  PUSH      ESI
001B:0018A525  53                  PUSH      EBX
001B:0018A526  80F802              CMP       AL,02
001B:0018A529  7407                JZ        0018A532
001B:0018A52B  0FB64E01            MOVZX     ECX,BYTE PTR \[ESI+01\]
001B:0018A52F  41                  INC       ECX
001B:0018A530  EB05                JMP       0018A537
001B:0018A532  B904000000          MOV       ECX,00000004
001B:0018A537  41                  INC       ECX
001B:0018A538  01CE                ADD       ESI,ECX
001B:0018A53A  E8B5FDFFFF          CALL      0018A2F4 <- redirect this point note the address 0x18a2f4.
001B:0018A53F  AB                  STOSD
001B:0018A540  EBDC                JMP       0018A51E <- note this address too, it's the loop back jump.

All we have to do is to find a free mem location filled with 0 so we can write a small code to modify this
routine. The address 0x190000 will be used as an example.
```

```
First patch 18a53a jmp 190000
```

```
Then write the following code in mem with softice :
```

```
190000 call 18a2f4 ; we use the legal call
190005 cmp ecx,40000000 ; ecx contains an api address ?
19000b jle 19001c ; if not
19000d add ecx,eax ; if yes convert the address\*\*\*case 2
19000f add ecx,5 ; to the absolute api address for a normal import table
190012 mov dword ptr \[edi\],ecx ; put it in import table, pointed by edi
190014 add edi,4 ; updates edi to the next import
190017 jmp 18a51e ; back to normal
19001c cmp ecx,0 ; is it case 4 ?
19001f jz 19002e
190021 cmp eax,40000000 ; eax contains an api address ?
190026 jle 19003c ; if not go to 19003c
190028 stosd ; we're in case 1, direct copy of valid api address in import table
190029 jmp 18a51e ; back to normal
19002e push dword ptr \[eax+1\] ; we're in case 3, pushes the encrypted api address
190031 call 16df80 ;IMPORTANT 
you have to locate this Asprotect address by doing a S 0 L FFFFFFFF 55 8b ec 81 c4 f8 fe ff ff 53 56
so we found as example 16df80, you MUST modify the code at offset 16df80+95 with three nops (90), if you don't
this call will generate an error 13 in Asprotect (the call doesn't return properly)
this call is used to decypher the encrypted api address, normally decrypted at run time
```

```
190036 stosd ; the api's address is then decrypted, copy it to import table
190037 jmp 18a51e ; back to normal, don't forget that this is the loop back jump
19003c mov eax,KERNEL32!GetProcAddress ; use softice to have your system's getprocaddress. this is case 4
190041 stosd ; copy to import table
190042 jmp 18a51e ; back to normal
```

```
When you are finished typing this, put a bpmb to the prog's starting point (I found 0x4de384 )
then launch, if you typed correctly, the import table is now normal and you can dump it using icedump :
```

```
PAGEIN D 400000 12d000 c:\\temp\\cv\_dump.exe
```

```
Then use procdump to rebuild the pe.
```

```
Very important : Use those procump options when rebuilding the pe :
Structure : all enabled
Import : don't rebuild import
Leave other options untouched.

You have to manually copy the .aspr section from cv.exe (original) to cv\_dumped.exe, using your favorite
hexeditor. As an example I found inside original raw offset : 5aa00 length 14800
dumped raw offset : ff400 length 14800, this manipulation will copy back the original import table.
```

```
Modify the entry point to 000de384 inside procdump, so your dumped exe will start at 4de384.
Copy your dumped exe to the commview directory.
```

```
Now if you launch it, it will direct exit, because there is a crc check, we have to patch this code inside
the dumped cv\_dumped.exe :
004DE3B9                 call    sub\_4524A0
004DE3BE                 mov     eax, \[ebp+var\_10\]
004DE3C1                 call    sub\_465118
004DE3C6                 cmp     eax, 6F200h
004DE3CB                 jz      short loc\_4DE3D2 <- patch to jmp 4de3d2
004DE3CD                 call    sub\_403BB4

```

```
Don't forget one thing, there is still an anti softice left. Reactivate the bpx createfilea do "d esp->4";
press F12 when it hits then invert the jz just after.
```

```
Launch again, it's now working and you're ready to continue the reversing of this crippled, time trial exe.
```

```
4) What's left to crack ?
```

```
Now the real work is finished, we can patch whatever we want inside. I will easily give you the patches,
just because this section was not the main part of my essay, this will surely save you some work.
To finish its agony, we dwelve with :
```

```
\-Anti softice routine : typical one, just after the well known createfileA :

0046573E                 call    sub\_406D0C (createfileA with softice driver as an argument)
00465743 
00465743 crack5:
00465743                 cmp     eax, 0FFFFFFFFh is it loaded ?
00465746                 jz      short loc\_465750 <- patch this to jump
00465748                 push    eax
00465749                 call    sub\_406CEC
0046574E                 mov     bl, 1

\-The time limit :You usually find those after a bpx to getlocaltime, and trace after.
```

```

004D8D5B                 call    sub\_44EB74 ; this call contains a call to getlocaltime
004D8D60 
004D8D60 loc\_4D8D60:                             ; CODE XREF: sub\_4D899C+386j
004D8D60                 mov     eax, ds:dword\_4E051C
004D8D65                 cmp     byte ptr \[eax\], 0 ;is the trial finished ?
004D8D68 
004D8D68                 jnz     short loc\_4D8DB8 <- force this to jump
004D8D6A                 mov     eax, ds:dword\_4E0378

```

```
\-half of the packets displayed : an easy one with ida, just find the reference to string :
```

```
0048B5A4                 push    offset loc\_48BB1C
0048B5A9                 push    dword ptr fs:\[eax\]
0048B5AC                 mov     fs:\[eax\], esp
0048B5AF                 cmp     byte ptr \[ebx+5Fh\], 0
0048B5B3 
0048B5B3                 nop ; nop those so it never jumps to 48b5c2, you guessed why
0048B5B4                 nop
0048B5B5                 lea     edx, \[ebp-14h\]
0048B5B8                 mov     eax, \[ebx+22h\]
0048B5BB                 call    sub\_48A7C0
0048B5C0                 jmp     short loc\_48B5CF
0048B5C2 ; ---------------------------------------------------------------------------
0048B5C2                 lea     eax, \[ebp-14h\]
0048B5C5                 mov     edx, offset aDataThisEvalua ; "DATA:THIS EVALUATION VERSION DISPLAYS O"...
0048B5CA                 call    sub\_403E08

you have two more places to do the same, find them and correct the \*bug\* .
```

```
\-the annoying nag when you leave commview: I found it after a bpx to createwindowsexa, you have to press F12
a lot of times to find this routine :

00459718 loc\_459718:                             ; CODE XREF: sub\_45969C+6Cj
00459718                                         ; sub\_45969C+70j
00459718                 mov     eax, \[ebp+var\_C\]
0045971B                 mov     edx, \[eax\]
0045971D                 nop <- you have to nop this call to avoid the nag
0045971E                 nop
0045971F                 nop
00459720                 nop
00459721                 nop
00459722                 nop
00459723                 mov     \[ebp-8\], eax
00459726                 xor     eax, eax
00459728                 pop     edx
00459729                 pop     ecx
0045972A                 pop     ecx
0045972B                 mov     fs:\[eax\], edx
0045972E                 push    offset loc\_459743
00459733 
00459733 loc\_459733:                             ; CODE XREF: CODE:00459741j
00459733                 mov     eax, \[ebp+var\_C\]
00459736                 call    sub\_403024
0045973B                 retn

Now you can enjoy your new portsniffer, auto rebuilded by itself, just like it always should be.
```

  
  
 FINAL NOTES STARTS HERE 

| Final Notes   |
| --- |

```
I need your feedback for this essay, so please write your comments after trying it at the msgboard :
http://www.insidetheweb.com/mbs.cgi/mb155985
```

```
You can join all the reverser's forums at sandman's , where I had this idea :
http://www.insidetheweb.com/mbs.cgi/mb628842
```

```
Or on the fravia msgboard, for intermediate and advanced ones :
http://www.insidetheweb.com/mbs.cgi/mb155985
```

```
Thanks to Eternal Bliss ,r!sc and SV for their preliminary work on that interesting target.
```

```
Tsehp

```

| Ob Duh  |
| --- |

*I wont even bother explaining you that you should BUY this target program if you intend to use it for a longer period than the allowed one. Should you want to STEAL this software instead, you don't need to crack its protection scheme at all: you'll find it on most Warez sites, complete and already regged, farewell, don't come back.*  WAY OUT STARTS HERE 

* * *

*You are deep inside reverser's page of reverse engineering, choose your way out:  
  
* 

> 

 找到 stosd 的方法：原作者的实在是太乱。   
以 arpr 1.0 build 45 为例，   
1，bpx 45c001   
2, bpmd cs:41841c （在第二次中断时，有可能每个人的都不相同）   
  
输入的代码基本上相同，只在第 4 种情况判断时要改正，我反正是   
直接 stosd, 现在还没有发现问题。   4dd466d0e1e0cc123afbe504da2462ca