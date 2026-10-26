
# New essay

about AsProtect: http://www.woodmann.com/tsehp_asprotect10.htm (空)

|  |  |  |

| --- | --- | --- |

|  | Manually unpacking Asprotect version 1.0  The encrypted import table |  |

|  | by  Tsehp |  |

|  |  |  |

|  |  |  |

|  |  |  |

---

### Manually unpacking Asprotect version 2000

The encrypted import table

Written by Tsehp

|  |

| --- |

|  |

```
Asprotect is a well know target, this packer was studied a lot and often used as a target practice for newbies.

If you don't know anything about packing, improve your knowledge on the past essays, then come back, I'll try to

explain you their last tricks.

I use to find this target while looking for the sandman's newbie msgboard.

Thanks to a reverser called SV that explained very well the protection on his threads, his work made me save

a lot of time.
```

```
Important : This essay can bring you a working app with your actual system, but the import table is not

rebuilt, so if you update your OS and the dll's addresses changes, it will not work anymore and you'll have

to do the entire process again. I'm actually working on another essay to finalize the reconstruction of the

import table.
```

|  |

| --- |

|  |

Softice 4.05

Icedump

Procdump

Ida 4.04

An hex editor

Essay made on win 2000, expect to change (later in this essay) some import addresses

if you're working on win 9x.

|  |

| --- |

|  |

<http://www.tamos.com/>

This is Commview 2.1, another portsniffer.

|  |

| --- |

|  |

```
1) The anti-softice routines
```

```
Before lauching this app, you have to hide your debugger.

This prog uses two typical anti softice tricks:
```

```
-Is the driver loaded ? : bpx createfileA do "d esp-4"
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
-The boundschecker trick.

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
```

```
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
4e328c : jmp [cd4950] and cd4950 containing 77db858e (a valid import address)
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

*culprit* loop that builds the import table, just do a bpmb at the start.
```

```
Here is what you find :
```
