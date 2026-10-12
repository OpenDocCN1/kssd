
Now, this isn't very hard either. As the program is already patched, the call at `440DAE` will return `eax <> 0` and thus the jump will be taken. Btw, that call is our good old "comparison" call. As you see, it's used quite a lot in the program, so it was a good idea not to patch there! Now just NOP away the jump. `55443322` will be stored in `eax` and the checksum routine thinks that everything is alright and `we have a fully functional registered copy of BrainsBreaker!`

### Step 5 - Making a nice crack!

You can skip this part if you want to. This step is not really necessary, but if we were to release a crack for this program, we had to make a nice looking patcher. The problem is that we don't know how to bring up the "enter registration info" window, and thus the user wouldn't have anywhere to enter his name. We can solve this by making our own little "registration window" that asks for a username in our patcher. If you look where `BrainsBreaker` saves the registration information, you will see that all of the reg-info is saved in the `BBRK.INI` file. It will save both plaintext and encrypted values. It will look something like this:

```
[PackFull]            ;What pack we're using
0=Cruehead            ;Username plain text
1=123                 ;ID plain text
2=7E15AB3D1516D6      ;Serial encrypted
3=2100                ;Not sure. I believe it's version number
4=192BEC6D323CF86C    ;Username encrypted
5=6B6BAA              ;ID encrypted
```

Of course I didn't know all of this before I fully examined the program. I started out by putting a breakpoint on `GetPrivateProfileStringA` and when the encrypted serial line was read I put a bpr on that so I would see what was happening to that string. I won't describe every step on how I did it because it's fairly easy. Instead I'll try to describe how it works:

Every two letters in `7E15AB3D1516D6` is in fact one byte. So first of all the string is transformed so it looks like this: `7E 15 AB 3D 15 16 D6`. 7 bytes that is. Now this string is XOR'ed against a four byte long string that was `5A 59 99 08`. This string caught my curiosity because it didn't look like a hardcoded value and neither could I find it in the `.EXE` file, but I continued nevertheless, not worrying about it. The string was XOR'ed like this:

| String from INI file: | `7E` | `15` | `AB` | `3D` | `15` | `16` | `D6` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Four byte string: | `5A` | `59` | `99` | `08` | `5A` | `59` | `99` |
| Result after XOR: | `24` | `4C` | `32` | `35` | `4F` | `4F` | `4F` |

You see? The result is `$L25OOO`. Our entered serial string (somewhat transformed). The same procedure is done with the two other encrypted strings. If the decrypted username matches the username in plain text and if the decrypted ID matches the ID in plain text, then the program executes the registration check with the decrypted serial. The program will gladly accept it because we have patched the last check!

Ok, I sat down one night and coded this crack. It worked great on my computer, but when I tried it on another computer, it simply didn't work! Back to the drawing board again. It turned out that I was right when thinking that the 4 bytes XOR string was strange. After some more examination, I found out that this value was created from a special ID which is stored in the registry. The ID is created the first time `brainsbreaker` is installed and there isn't much point in reversing the algorithm for creating a `BrainsBreaker` ID because it uses the `TlsGetValue` function as source. The complete ID is about 50 bytes long, but only 8 of these bytes are used to create the XOR string (if you're really interested, byte number 5, 10, 8, 13, 17, 19, 22, 38 of the ID is used to make the XOR string). That's why my crack didn't work on the other machine - when I installed `BrainsBreaker` for the first time a different ID was created and thus also a different XOR string. So I sat down another night and re-coded the crack, this time for Windows as we need to access the registry. This time the crack worked like a charm on both machines! Oh joy oh joy!

### Thoughts about BrainsBreaker

This is not important for the crack, but it's nonetheless very interesting. I'm not sure about this but it looks like every puzzle is a stand-alone part of the program. Every puzzle stores a number of things including the name of the author of the puzzle, if the main program has been altered, whether this puzzle is free/demo/registered and a number of other things. My guess is that these values (or flags) are stored directly after the main program checks if it's registered or not. This way every puzzle can check if the main program has been cracked. But we don't have to care about all of this because the crack I presented above takes care of everything :). None of these flags are stored directly in the `.exe` file, instead they are stored in encrypted state. At startup these puzzles are decrypted using different key sources, but the most common seems to be `(c)J. Trujillo T.` I managed to get this info by stumbling across a `xor [edi], al` function when the program was loading the startup screen with all the puzzles. That of course caught my interest and I started to examine it a bit more closely.

A small hint: It's a good idea to search the deadlisting for `MessageBoxA` functions as these can give you a lot of useful information sometimes. Fake a flag or something to make the program show the messagebox. For example, I found out that every puzzle stores if the main program has been altered or not (if it has, a value of `12345` is stored in each of the puzzles) using this method.

But there are some things that remain a mystery for me still. I haven't been able to figure out how to bring up the registration-window yet, and you can find a few very interesting functions if you look at the dead-listing:

```
Addr:00487ABC Ord:  14 (000Eh) Name: __DebuggerHookData
Addr:004035BC Ord:  15 (000Fh) Name: __lockDebuggerData(void)
Addr:004035E4 Ord:  16 (0010h) Name: __unlockDebuggerData(void)
```

Antidebugging functions probably, but I haven't seen any marks of them. Pretty strange...

### Conclusion - BrainsBreaker:

A somewhat complicated protection scheme, but nothing that we haven't seen before. The hidden registration window is a good idea, but it's far too easy to find it using a dead-listing approach.

-->-->-->

破 `BrainsBreaker`3.0[07b]!

`http://www.brainsbreaker.com/download.htm`

非常感谢 1212 提供的 `Cruehead` 的优秀教程。

如有疑问，请参看此教程!

`http://www.instinct.org/fravia/99solu/cruecom.htm`

===========================================================

#### 1、Key 运算检测，最后保证 `[eax]` 为 0，可 BPM 跟踪。

```
0167:00426D93 8BC1            MOV     EAX,ECX
0167:00426D95 99              CDQ
0167:00426D96 F7FF            IDIV    EDI
0167:00426D98 8D041A          LEA     EAX,[EDX+EBX]
0167:00426D9B 8A1431          MOV     DL,[ECX+ESI]
0167:00426D9E 3010            XOR     [EAX],DL    ******
0167:00426DA0 41              INC     ECX
0167:00426DA1 3B4DF8          CMP     ECX,[EBP-08]
0167:00426DA4 7CED            JL      00426D93---->循环 9 次
```

改为：

```
0167:00426D93 8BC1            MOV     EAX,ECX
0167:00426D95 99              CDQ
0167:00426D96 F7FF            IDIV    EDI
0167:00426D98 8D041A          LEA     EAX,[EDX+EBX]
0167:00426D9B 8A10            MOV     DL,[EAX]    ******
0167:00426D9D 90              NOP
0167:00426D9E 3010            XOR     [EAX],DL    ******we shall have a final value of zero
0167:00426DA0 41              INC     ECX
0167:00426DA1 3B4DF8          CMP     ECX,[EBP-08]
0167:00426DA4 7CED            JL      00426D93
```

===========================================================

#### 2、Pack、Name、Your ID 运算检测，对比处可 BPM 跟踪。

```
0167:00427587 33C9            XOR     ECX,ECX
0167:00427589 8BC1            MOV     EAX,ECX
```
