

# Yoda's Crypter v1.0 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue443.htm`](https://www.pediy.com/kssd/pediy04/kanxue443.htm)

Manually UnPacking of Yoda's Crypter v1.0   
  
Author:        Yoda   
Tools:        SoftICE v4.05   
        Hex-Editor   
        ProcDump v1.6.2   
  
\--->    Some Tips...   
Let me tell you a few things that are important when you want to Unpack something :)   
  
1.    Find the OEP (Original Entry Point, the starting location of the Original Program).   
2.    Dump the File (Doh!).   
3.    Check if the Import Table is corrupted (And then try to fix it offcourse ;).   
4.    Remove the Encrypter Code (Not necessary, but fun and in some cases it decreases the   
    size of the File ;).   
  
\---->    Some words...   
I just want to say that i always break at the Entry Point of the Encrypted Program with Int3.   
Why?   
Simple, because then i can see exactly what is going on :)   
If you want to break on an API, check the Info page of this Packer/Encrypter and then look for   
what's the best API to break on.   
  
\--->    Special Note...   
The Target File (Yc\_v10\_Notepad.exe) has been Encrypted with all the options enabled.   
Maximum Protection ;)   
  
\--->    Let's MUP... (Manually UnPack)   
We're going to take a few steps ;)   
  
1.    Getting the OEP of the Encrypted program together with some info.   
2.    Breaking on the Entry Point with Int 03 and then TAL (Trace And Learn) ;)   
3.    When we're TALling hehe, we're going to fool the Decryption Routine a little bit ;)   
4.    Dumping the file.   
5.    Fixing the Dumped Executable (Together with removing some unwanted Code :).   
  
                  \*\*\* Step 1 \*\*\*   
  
Ok, put the Target File (Yc\_v10\_Notepad.exe) on your Desktop (Or where ever you prefer ;).   
Then open ProcDump and click on "PE Editor" select the Target File and press "Open".   
You'll see the following under "Header Infos":   
  
Entry Point:    0000D060   
Size of Image:    0000E000   
Image Base:    00400000   
  
There's the Entry Point but remember this is now the Virtual Offset and we need the Raw Offset   
in order to put our Int 03 :)   
So click on "Sections" and look for something suspicious.   
Notice the Section "yC"? (Yoda's Crypter :)   
As you can see in this Section the Virtual Offset and the Raw Offset are the same = 0000D000   
And the Entry Point = 0000D060   
So nothing changes this time because the Virtual and Raw Offset are the same.   
The real Entry Point (For the Encrypted Executable) = 0000D060 :)   
  
You can close ProcDump now and open the Target File in your Hex-Editor.   
Now go to the Offset "0000D060" and there you can change this:   
  
60    pushad        (Remember this instruction to change it back later)   
  
into:   
  
CC    Int 03   
  
"Pushad" a well known beginning of Packers/Encrypters ;)   
  
                  \*\*\* Step 2 \*\*\*   
  
Now get into SoftICE (Ctrl+D) and type "bpint 3" (To break on Interrupt 03 "CC").   
Followed by "enter" and then out of SoftICE (Ctrl+D).   
Now click on the Target File and SoftICE should popup.   
We're at the beginning of the Decryption Routine now, but we can't Trace yet because of the "CC".   
So we need to change that back to "60".   
So type/press this:   
  
A EIP        (To edit the instruction at the current location)   
pushad (enter)    (to replace the Int 03 "CC" with pushad "60")   
ESC        (To get out of the "a" Command)   
  
Now we can TAL (Trace And Learn) ;)   
  
                  \*\*\* Step 3 \*\*\*   
  
"Yoda's Crypter v1.0" has some Protections and now we're going to remove them ;)   
So from this point (If everything's alright you should still be on the Entry Point :) you   
should see something like this:   
  
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
pushad   
call 0040D066            - Get the "Delta" Offset (As they call it ;)   
pop ebp   
sub ebp, 00401AE7   
call 0040D113            - Get the Base Addresses of "LoadLibraryA" and "GetProcAddress" out of the Import Table in the Decryption Routine.   
call 0040D148            - Get the Base Addresses of "VirtualProtect" , "GetModuleFileNameA" , "CreateFileA" , "GlobalAlloc" , "GlobalFree" , "ReadFile" , "GetFileSize" and "CloseHandle".   
call 0040D201            - Here it uses "VirtualProtect" to protect some space after the Decryption Routine.   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Ok, you can simply skip this piece, because it doesn't do much more then getting the Base   
Addresses of several API's ;)   
But offcourse you can Trace in it to see exactly what's going on.   
Ok, after this stuff you'll see this:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
test \[ebp+00401F28\], 00000008    - This is to check if you've enabled the CRC option.   
jz 0040D093            - If not skip the CRC check.   
call 0040D35F            - In here it opens the File and calculates a CRC (Gets a Unique Value :).   
mov \[ebp+00401F2C\], eax     - Save the Value.   
mov eax, \[ebp+00401F14\]        - Get the Image Base.   
mov ebx, 00000001   
call 0040D3F6            - This is the "Simple" Decryption Algo ;)   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Ok, first it checks if the CRC option has been enabled, if so it calculates an unique value out   
of the file else it skips this process (In our case it'll calculate an unique value because   
i enabled all options ;).   
When your on the "jz 0040D093" type "r fl z" to skip this process :) (or you can look in the call   
for learning purposes ;).   
Then in the next call it does the actual Decryption of the Sections.   
You can skip this, or Trace in it if you want :)   
It's a simple XOR process ;) (It Decrypts the Sections with "xor 79" and skips several Sections, for example ".edata" ;)   
Ok, after the Decryption call you'll see this:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
test \[ebp+00401F28\], 00000001    - This is to check if you've enabled the SoftICE Detection option.   
jz 0040D0BB            - If not skip this process.   
call 0040D1E9            - Here it does the SoftICE Detection (it uses the "BCHK" method).   
cmp eax, 00000001        - SoftICE Detected?   
jnz 0040D0BB            - If EAX isn't 00000001 it jumps and continue the Decryption Routine.   
popad                - Else it pops all Registers back.   
ret                - And quits.   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
First it checks if the SoftICE Detection option is enabled.   
If not it skips this process else it goes into the call and executes the "BCHK" method :)   
When your on the "jz 0040D0BB" type "r fl z" to skip this process :)   
Then it Compares EAX with 00000001, if it holds 00000001 then it means SoftICE was detected   
and the Program just quits (popad, ret).   
Else it continues at 0040D0BB.   
  
The BCHK Method (BoundsChecker).   
  
mov ebp, 04243484Bh    ; "BCHK"   
mov ax, 04h   
int 3        
cmp al,4   
jnz sice   
  
Ok, to get around this you can simply NOP (90) the "int 3" or type "r fl z" at the instruction   
"jnz sice" :)   
Ok, next thing is at Offset 0040D0BB, and you'll see this:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
call 0040D269            - This is the "Import Table F\*ck Up" ;)   
cmp eax, 00000001        - Succesfully finished?   
jz 0040D0C7            - If so, jump and continue.   
popad                - Else pop all Registers.   
ret                - And quit.   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Now this is an Important call to remove ;)   
Because in there it tries to replace the entry's in the Import Table with the Base Addresses.   
And we don't want that now do we? ;)   
So Trace into the call with F8 and you'll see this:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
mov eax, \[ebp+00401F2C\]        - Get the unique Value (In our case now it's 00000000 because we skipped the CRC option ;)   
or eax, eax            - Check if EAX is 00000000.   
jz 0040D280            - For us it's equal so we jump, else continue.   
cmp eax, \[ebp+00401FC3\]        - Compare the computed Value with the original CRC.   
jz 0040D280            - If equal everything's ok and we jump, else continue.   
jmp 0040D34A            - Jumps to the end and returns to the "cmp eax, 00000001" and we fail at that Compare, and so the program quits ;)   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Ok, this is the place where it Compares the computed Value with the original.   
We skipped the CRC option and so we jump at "jz 0040D280" (The first jz).   
Let's see what's at 0040D280:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
mov esi, \[ebp+00401F1C\]        - Here it gets the Virtual Offset,   
add esi, \[ebp+00401F14\]        - of the Import Table.   
jmp 0040D33D   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Here it puts the Virtual Offset to the Import Table in ESI.   
Remember this Virtual Offset, because after we Dumped the file we need to fix the RVA ;) (The Virtual Offset to the Import Table = 00406000).   
And then jumps to 0040D33D, let's see what's there:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
cmp \[esi+10\], 00000000        - Check in the Import Table if there's any .DLL at \[esi+10\] (this is where the RVA to the .DLL is stored).   
jnz 0040D291            - If there's something that points to a .DLL jump to the process that gets the Base Addresses, else continue.   
xor eax, eax            - EAX == 00000000.   
inc eax                - EAX +1.   
ret                - Return to the main Code.   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
It checks here to see if it has done all the .DLL's that are stored in the Import Table, by checking the RVA to the .DLL name :)   
So to bypass this you simply type "r fl z" at the instruction "jnz 0040D291" :)   
And it just continues without changing the Import Table ;)   
Now when we get back to the main Code with the "ret", EAX will be compared to 00000001 and we'll jump to "0040D0C7" (Because of the instructions "xor eax, eax" , "inc eax").   
So we'll continue at "0040D0C7" let's see what's there:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
test \[ebp+00401F28\], 00000002    - This is to check if you've enabled the Erase PE Header option.   
jz 0040D0D8            - If not, we jump and skip this process else continue.   
call 0040D22A            - in here it Erases the entire PE Header :)   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Here it checks if the Erase PE Header option is enabled ;) (in this case it is).   
So it won't jump now, but we'll change that ;)   
If your on the "jz 0040D0D8" type "r fl z" and we skip this process.   
Offcourse again you may Trace into the call ;)   
You'll see that it'll take the SizeOfHeaders and the ImageBase, and Erases everything (by putting 00 into it ;).   
So we jump to "0040D0D8" let's see what's there:   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
test \[ebp+00401F28\], 00000004    - This is to check if you've enabled the Delete Import Information option.   
jz 0040D0F5            - If not, it jumps and skips this process, else continue.   
mov eax, \[ebp+00401F1C\]        - Here it gets the Virtual Offset,   
add eax, \[ebp+00401F14\]        - Of the Import Table.   
call 0040D243            - And here it removes the Import Table structure ;)   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
It checks here if you've enabled the Delete Import Information option, if not we jump and skip   
the process else, it'll erase everything ;)   
So when your on the "jz 0040D0F5" type "r fl z" to skip this process.   
In the call it counts what the length of the Import Table is (Not including the ILT's "Import Lookup Table's" and the IAT's "Import Address Table's").   
And then simply Erases it with 00 :)   
Let's see what's at Offset "0040D0F5":   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
mov eax, \[ebp+00401F18\]        - Puts the OEP without Image Base in EAX.   
add eax, \[ebp+00401F14\]        - Add the Image Base to the OEP.   
mov ebx, 00401B8D        - This Offset will,   
add ebx, ebp            - point to the instruction,   
inc ebx                - "mov eax, FFFFFFFF".   
mov \[ebx\], eax            - Moves the OEP in EBX.   
popad                - Pop all Registers back.   
mov eax, FFFFFFFF        - Move the OEP in EAX.   
jmp eax                - Jump to the OEP :)   
\--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
And this is the end of our Trip ;)   
Here it gets the OEP and puts it at the instruction "mov eax, FFFFFFFF", then pops all Registers   
back, and then moves the OEP in EAX and jumps to it ;)   
In this "Notepad" case the OEP is "004010CC".   
  
                  \*\*\* Step 4 \*\*\*   
  
We're going to use the "EBFE" trick to let the Program run in a loop so we can Dump it ;)   
So when your on "004010CC" (After the "jmp eax") type/press this:   
  
A EIP        (To edit the instruction at the current location)   
jmp eip (enter)    (This will let it jump to itself, running in a loop, Don't forget to remember the original Opcodes "558B" so we can change it back later)   
ESC        (To get out of the "a" Command)   
  
Now type "BC \*" to clear all Breakpoints (If you haven't done already ;) and press (Ctrl+D)   
to get out of SoftICE.   
Now Windows may run a little slow because the Program is still running in a loop ;)   
Open ProcDump and at the main Window click on the Entry that points to our Encrypted Program.   
In my case it's:   
  
"C:\\windows\\desktop\\Yc\_v10\_Notepad.exe"   
  
Then right click on that and select "Dump (Full)".   
Save as any file you want.   
Now right click again on that Entry and select "Kill Task" to Terminate the Process.   
Ok, we got our Dump :)   
But now we need to fix a few things ;)   
  
                  \*\*\* Step 5 \*\*\*   
  
Your still in ProcDump? ... good :)   
Click on "PE Editor" and select your Dumped Executable and click "open".   
Now we need to change the Entry Point to the original one.   
So change this:   
  
Entry Point:    0000D060    (This points to the Decryption Routine, but we don't want that anymore ;)   
  
into:   
  
Entry Point:    000010CC    (Without Image Base)   
  
Now we're going to remove the Decryption Code ;)   
In the window your looking at now (PE Structure Editor) you'll see 2 options:   
  
Only to PE Header    - Changes are made only in the PE Header   
To PE File        - Changes are made to the entire File   
  
Select the second option (To PE File) because we want to remove the Section "yC" aswell ;)   
Now click on the button "Sections" and select the Section "yC".   
Right click on it and select "Kill Section" and \*BOOM\* Section gone :P   
One more thing, because we now removed the last Section, the "Size of Image" isn't correct anymore.   
to fix this we need to count the "Virtual Offset + Virtual Size" of the last Section (Which is now ".reloc").   
So then we get:   
  
  
Virtual Offset    + Virtual Size    = Size of Image   
0000C000    + 00001000    = 0000D000   
  
  
Close this "Sections" window and change the Size of Image (Which is 0000E000) to "0000D000".   
Now click on the button "Directory" because we need to point the RVA to the good Import Table ;)   
Look at the line of the Import Table it says:   
  
        RVA        Size   
Import Table:    00006050    0000008C   
  
Change the "00006050" to "00006000" (Without Image Base) :)   
Because that's where the Import Table starts ;)   
Now click "Ok" and then click on "Ok" again and now we've fixed our Dump :P   
Close ProcDump and run the Dumped file and, OOppss ;P   
It doesn't run yet because of the "EBFE" trick at the OEP remember ;)   
So open the Dumped file in your Hex-Editor and go to the Offset "000010CC".   
Change there:   
  
  
EBFE   
  
into:   
  
558B   
  
  
Now save the file and run it, it works ;)   
"Yoda's Crypter v1.0" completely removed without too much difficulties ;)   
  
  
\--->    Final Words...   
  
This is an easy Target to Manually Unpack :)   
But the Decryption Routine goes in steps and that's nice for learning purposes ;)   
Because i skipped the "Import Table F\*ck Up" process in the Decryption Routine we didn't have   
any difficulties with the Import Table.   
Otherwise you needed to use Revirgin or something ;)   
Pretty long Tutorial but it's worth it :P   
  
  
\--->    Greetings...   
  
                "You know who you are!!!" :P   
  
  
            Don't trust the Outside, Trust the InSiDe !!!   
  
                      Cya...   
  
                    CoDe\_InSiDe   
  
Email: code.inside@home.nl  4dd466d0e1e0cc123afbe504da2462ca