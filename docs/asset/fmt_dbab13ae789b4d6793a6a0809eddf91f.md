
"Hey! You only got `6` characters in the table, but the strings can be longer (and one of them in fact is)!" Calm down, That's because something is happening after `6` bytes has been processed. Now the first byte in our "result string" (`34` in our case) will also be included in the scheme. So, `34` will be `xor'ed` to every `7'th` char in the other strings, and in our case the "username" string will be the only one effected, as the others are too small. The result is written in the first position in the "result string", thus removing the old value. After `6` more bytes, it does the same again, thus keeping the size of the result string to `6` bytes. When `20h` (= `32` dec) characters has been processed the routine is done.

Something has to be happening with these `6` bytes (why write that routine otherwise), so lets put a `bpr` on them. You will break in a string comparing routine. So...this string is compared to another string! Very interesting indeed... I wonder what happends if they matches??? let's see! Change one of the strings so it matches the other and let the program run...

**`YEEEEES! Program registered!`**

So, now all we have to do is to make the strings match...just one BIG problem...One would have thought that the string that our "pack, username, ID" string was compared with should be constant...but this is not true! It changes everytime, so there is not a chance to make these strings match eachother! First I thought this was some anti-cracking feature, but that is not true either. I changed the program so it would display a little `messagebox` with the first bytes of the string, and I ran it without `softice` (for the first time in months, `softice` wasnt loaded on my machine :)). No luck, it still changed! One alternative still remains...`patching`!

**`Step 4 - Cracking it!`** First of all I'd like to say that I dont really like `patches` because one never knows what comes with a patch. It could be a nasty `checksum`, or even worse - a delayed `checksum`. That's why I always like to find/calculate a valid `serial` when cracking `name/serial` protections, but when you're out of ideas, I guess `patching` can be justified. But even if we are going to patch, let's patch as little as we have too. We dont have to patch the first `check` (remember? one byte is compared to `zero`...) as we can pretty easily find a `serial` that justifies that. As most crackers are lazy by nature (well, atleast I am :)), lets just write a quick and dirty `bruteforcer`. You can find my source and the executable for it [here](http://fravia.org/99solu/bb.zip). Run it, and we'll get the `serial` `125000`. If our `bruteforcer` does what it's suppose to do we would now get by the first `check` by using this `serial`, so let's test it! Go back to the `register` window, and enter the same information as before, but change the `serial` to `125000`. Wow! It worked! Of course we didnt get the "thanks for registering" message, but we didnt expect that either, because the last `check` is still there. For reasons that I told you about earlier we have to patch that (or spend the next couple of weeks trying to figure out a way how to crack it without patching, but as I said - we're all lazy).

Of course the next question will be *`Where shall we patch?`* Before we can answer that we have to make sure we understand what we want to do. In this case we want two strings to match eachother (or atleast making the program think they do). If we investigate the `comparision` routine we see that the routine compares the number of bytes specified in `edi`. In our case `edi=6` and therefor `6` bytes are compared. If we look at the code right after the comparison routine we will also see that `dl` is set to `1` if the strings matches. If they dont `dl` is set to `zero`. Now we can make a small list of possible `patches`:

```
1)   Action: Set edi to zero so that zero bytes will be compared.
Consequence: The strings wont be compared.
2)   Action: Overwrite the two strings with the same 6 bytes.
Consequence: The strings will match.
3)   Action: Fake the return value so dl will be set to one either way.
Consequence: The program thinks that the strings matches.
4)   Action: Fake a conditional jump
Consequence: The program thinks that the strings matches.
```

There are even more possibilites but these are more than enough. We can choose any of these and the program would, in theory, think our entered `serial` is valid.

Let's try for example to set `edi` to `zero`. First of all lets find where the program sets `edi` to `6`:

```
:00459C43 8B7508                  mov esi, dword ptr [ebp+08]
:00459C46 807E2400                cmp byte ptr [esi+24], 00
:00459C4A 7415                    je 00459C61
:00459C4C BF06000000              mov edi, 00000006
```

Can we change this into "`mov edi,0`" instead? Yes we can but the program would crash later due to a couple of "`IDIV`" instructions that relies on `edi` not beeing zero. So where can we change `edi` to `zero` then? Look at this:

```
:00459CFF 33C9                    xor ecx, ecx
:00459D01 3B4DEC                  cmp ecx, dword ptr [ebp-14]
:00459D04 7D4F                    jge 00459D55
...
```

This is the begining of the code that creates the `6` bytes long string from our entered Name, ID and Pack. This code is completely useless if we change `edi` to `zero` (because `zero` bytes will be compared) so we can saftly patch here. Let's change the code so it looks like this instead:

```
:00459CFF 33FF                    xor edi,edi	;set edi to zero
:00459D01 EB52                    jmp 00459D55	;Jump over the rest
						;of the function
```

Oh yeah! This works very nicely! Patch it using your favourite `hex-editor` and run it.

*`Integrity check:`

`Program seems to be altered from his original contents`

Of course...a `checksum` function as well. How can we crack this then?

In fact, the author has already done half the work for us. Notice how the message is presented - through a `messagebox`! Very good news for us crackers, we just have to put a `bpx` on `MessageBoxA` now in order to land right in the interesting code. So, do just that and trace upward and you'll see this:

```
:00442CC2 E870DBFFFF              call 00440837
:00442CC7 3D22334455              cmp eax, 55443322	;Whoah! What a nice hardcoded value!
:00442CCC 7472                    je 00442D40		;Take this jump if you want to jump
							;over the messagebox!
:00442CCE 6810200100              push 00012010
:00442CD3 FF35843B4800            push dword ptr [00483B84]
:00442CD9 83C4FC                  add esp, FFFFFFFC
:00442CDC 66C70424BE00            mov word ptr [esp], 00BE
:00442CE2 E8C16FFFFF              call 00439CA8
:00442CE7 50                      push eax
:00442CE8 6A00                    push 00000000
:00442CEA E80AEF0300		  call USER32!MessageBoxA
```

You can see directly that something is fuzzy about the "`cmp eax, 55443322`" line. That looks VERY suspicious if you ask me (why not say "Hey Cracker! I'm here!!!" instead). Now we could quickly crack this just by changing the "`je 00442D40`" instruction to "`jmp 00442D40`" but we wont do that. The program might save the value returned in `eax` and use it later on for more `checksum-checking` (this is infact true for this program) so instead we want the above `call` to return `55443322`. So, let's trace the `call`! You'll soon find this piece of code:

```
:00440DAE E8B504FCFF              call 00401268
:00440DB3 83C40C                  add esp, 0000000C
:00440DB6 85C0                    test eax, eax
:00440DB8 752E                    jne 00440DE8
:00440DBA A1803B4800              mov eax, dword ptr [00483B80]
:00440DBF 8B10                    mov edx, dword ptr [eax]
:00440DC1 8B0D243B4800            mov ecx, dword ptr [00483B24]
:00440DC7 8911                    mov dword ptr [ecx], edx
:00440DC9 B822334455              mov eax, 55443322	;We want this!
:00440DCE 50                      push eax
```
