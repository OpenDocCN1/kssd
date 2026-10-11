
# Armadillo 1.83b

> 原文：[`www.pediy.com/kssd/pediy02/forum138.htm`](https://www.pediy.com/kssd/pediy02/forum138.htm)

> 这个玩意儿自己并未加壳，但是它会在运行时在 `windows` 的 `temp` 目录下生成一个临时文件，这个文件实际上是个已还原的 `DLL`，然后它就用 `LoadLibraryA( )` 加载这个 `DLL`，用这个 `DLL` 中的函数来检查日期和注册码。

> 先要跳过它对 `SoftICE` 的检测：

> 1、用 `bpint 3` 设断点，看见它用 `SoftICE` 的 `BoundsChecker` 后门。这个是在 `EXE` 中进行的，可以直接 patch，即把 `EBP` 的值改掉。

```
015F:0040582E  MOV      DWORD PTR [EBP-0168],4675446D
015F:00405838  MOV      DWORD PTR [EBP-0148],04360C26
015F:00405842  MOV      DWORD PTR [EBP-0164],00000000
015F:0040584C  MOV      DWORD PTR [EBP-04],00000000
015F:00405853  MOV      [EBP-0164],EBP
015F:00405859  MOV      EAX,[EBP-0148]
015F:0040585F  XOR      EAX,[EBP-0168]
015F:00405865  MOV      EBP,EAX       //改掉 EBP 的值即可
015F:00405867  MOV      EAX,00000004
015F:0040586C  INT      3
```

> 2、再用 `CreateFileA( )` 设断点，这下是在解出来的 `DLL` 中检测 `SoftICE`，无法直接修改 `DLL` 文件。可以有两种策略，一是用 `process patcher`，另外一种是等它在内存中解压出这个 `DLL` 后但是还未写入这个 `DLL` 之前将该 `DLL` 在内存中打补丁。

```
015F:10005BA3  PUSH      EBX
015F:10005BA4  PUSH      00000080
015F:10005BA9  PUSH      03
015F:10005BAB  PUSH      EBX
015F:10005BAC  PUSH      03
015F:10005BAE  PUSH      EBX
015F:10005BAF  PUSH      EAX
015F:10005BB0  CALL      10005C64
015F:10005BB5  POP       ECX
015F:10005BB6  PUSH      EAX
015F:10005BB7  CALL      [KERNEL32!CreateFileA]  //打开 softICE
015F:10005BBD  CMP       EAX,-01
015F:10005BC0  JZ        10005BCB
015F:10005BC2  PUSH      EAX
015F:10005BC3  CALL      [KERNEL32!CloseHandle]
015F:10005BC9  JMP       10005BD6
015F:10005BCB  CALL      [KERNEL32!GetLastError]
015F:10005BD1  CMP       EAX,02
015F:10005BD4  JZ        10005BDA
015F:10005BD6  MOV       BYTE PTR [EBP-01],01  //Bad  flag
015F:10005BDA  MOV       EAX,[ESI+04]
015F:10005BDD  ADD       ESI,04
015F:10005BE0  CMP       EAX,EBX
015F:10005BE2  JNZ       10005BA3              //循环 3 次
015F:10005BE4  PUSH      1000D390
015F:10005BE9  CALL      10005C64
015F:10005BEE  POP       ECX
015F:10005BEF  PUSH      EAX
015F:10005BF0  CALL      [KERNEL32!LoadLibraryA] //kernel32.dll
015F:10005BF6  MOV       ESI,EAX
015F:10005BF8  CMP       ESI,EBX
015F:10005BFA  JZ        10005C24
015F:10005BFC  PUSH      1000D37C
015F:10005C01  CALL      10005C64
015F:10005C06  POP       ECX
015F:10005C07  PUSH      EAX
015F:10005C08  PUSH      ESI
015F:10005C09  CALL      [KERNEL32!GetProcAddress]
015F:10005C0F  CMP       EAX,EBX
015F:10005C11  JZ        10005C1D
015F:10005C13  CALL      EAX            //调用 IsDebugerPresent( )
015F:10005C15  TEST      EAX,EAX
015F:10005C17  JZ        10005C1D
015F:10005C19  MOV       BYTE PTR [EBP-01],01    //bad flag
015F:10005C1D  PUSH      ESI
015F:10005C1E  CALL      [KERNEL32!FreeLibrary]
015F:10005C24  CMP       [EBP-01],BL
015F:10005C27  JZ        10005C5D                //一定要跳
015F:10005C29  PUSH      00010010
015F:10005C2E  PUSH      EBX
015F:10005C2F  PUSH      EBX
015F:10005C30  PUSH      EBX
015F:10005C31  PUSH      EBX
015F:10005C32  PUSH      EBX
015F:10005C33  PUSH      EBX
015F:10005C34  PUSH      EBX
015F:10005C35  PUSH      EBX
015F:10005C36  PUSH      EBX
015F:10005C37  PUSH      0F
015F:10005C39  CALL      100042B7      //导致非法操作的指令
015F:10005C3E  ADD       ESP,28
015F:10005C41  PUSH      EAX
015F:10005C42  PUSH      EBX
015F:10005C43  PUSH      EBX
015F:10005C44  PUSH      EBX
015F:10005C45  PUSH      EBX
015F:10005C46  PUSH      EBX
015F:10005C47  PUSH      EBX
015F:10005C48  PUSH      EBX
015F:10005C49  PUSH      EBX
015F:10005C4A  PUSH      EBX
015F:10005C4B  PUSH      03
015F:10005C4D  CALL      100042B7
015F:10005C52  ADD       ESP,28
015F:10005C55  PUSH      EAX
015F:10005C56  PUSH      EBX
015F:10005C57  CALL      [USER32!MessageBoxA]
015F:10005C5D  MOV       AL,[EBP-01]       //一定要返回 0 才行
015F:10005C60  POP       ESI
015F:10005C61  POP       EBX
015F:10005C62  LEAVE
015F:10005C63  RET
```

> 至于注册码，不懂点密码学是绝对搞不定的！

> 这个软件每次生成的那个 `DLL` 的名字是不同的。只要在 `temp` 目录下得到这个 `DLL` 之后，将这个 `DLL` 取个固定的名字，比如 `protect.dll`，并将这个 `DLL` 中防 `SICE`、检查日期和注册码的代码打补丁，然后把这个 `DLL` 放到它的安装目录下。再修改它的 `EXE` 主程序，不让它以后每次运行时再生成这个 `DLL`（即跳过它生成这个 `DLL` 的那些代码，其实让它生成也没关系，因为我们不调用 `temp` 目录下的这个玩意儿），而是每次都直接 `LoadLibraryA("protect.dll",...)`，即每次加载打过补丁的 `dll`。这样也可以搞定它。

> 解压出来的那个 `DLL` 负责检查注册码，如果注册码正确则用 `SetEnvironmentVariablesA( )` 将注册信息填入到环境块中，而 `EXE` 则用 `GetEnvironmentVariablesA( )` 将注册信息读出来做简单判断（主要是判长度），因此很好 patch。主要的三个环境变量是 `USERNAME`、`USERKEY`、`PAIDFOR`。

# How To Un-dillo A Program Protected With Armadillo 1.8X or prior - The Silicon Realms Toolworks

------------------------------------------------------

## Tools Needed

- `WinHex 9.0` or other memory dumper
- `W32DASM 8.7` or `8.9` (for `Armadillo 1.8 - 1.8x` protection)
- Debugger Not Needed and must be disabled!

------------------------------------------------------

Whether or not you have an executable with the most recent version of `Armadillo` or not really doesn't matter. There is a slightly more involved procedure for un-dilloing the recent `1.80 - 1.8x` versions but as you'll see it's really very simple.

The target application we are going to work on is going to be `Stay On Pro 3.2`. If you don't already have this file it should be available at the site `www.stayonpro.com`. This is a popular utility designed to keep you connected to your internet service if you happen to idle for a while doing nothing and is protected with `Armadillo`.

If you wish to read more about the internal `armadillo` workings read on.

If you wish to skip to how to dump out `StayOnPro3.2` click HERE.

If you wish to skip to how to dump out `Armadillo 1.8x` or an `Armadillo 1.8 - 1.8x` protected program click HERE.

To begin I will explain the basic protection found in most `Armadillo` versions. No real knowledge of it's internal workings is necessary to crack the program but might help in other similar protection methods.

Starting an `armadillo` protected program will result in instantly creating a file either in the local directory where the program was started or in one of the temporary directories on your hard drive. This file is named `ArmXXXX.tmp`. This file is actually the `Armadillo Access (security) DLL`. This file may be copied while the program is running and then disassembled. In newer versions this file is hidden in the `local user\local settings\temp` directory.

Once the `ArmXXXX.tmp` is created with `CreateFileA` it is placed on the hard drive and loaded with the main program's `LoadLibraryA` function. All functions in `Armadillo` seem to be loaded on the spot in memory to help prevent direct text access.

Now, the main program calls the un-packing and un-encrypting function. In this particular version this function is named `GetProgramInfo`. `GetProgramInfo` also checks for the `SoftIce` hardware debugger, and any `Windows` debugger. It does this with a few functional steps:

\---------------------------------------------------------------------

1. Calls `CreateFileA` 3 times with the file name parameters `"\\.\\SICE"`, `"\\.\\NTICE"`, and `"\\.\\SIWDEBUG"` (which are internally encrypted). If they do exist or `GetLastError` doesn't return `2`, a flag is set. This flag will be referred to as the `DEBUG` flag.

2. A call to `IsDebuggerPresent` is made to test if a `windows` debugger is present. `SoftIce` and other hardware debuggers do not test true to this function. If a `Windows` debugger is found the `DEBUG` flag is set to true



3. Depending on the version, a dialog may be shown here to warn any user who has a debugger in operation to turn it off and restart. This dialog displays under certain versions, if the `DEBUG` flag is true.

4. If you now see a, "This program uses a secret key", at this point and it normally doesn't ask for a key, delete the `Software\\The Silicon Realms` registry keys and start over. This happens if you screw up the steps and it catches a debugger or the program can't extract the main program's code right.

5. `INT 3`. This is the final protecting method. A fake breakpoint set in the program to stop a debugger in it's tracks. But, `SoftIce`, doesn't respond to `INT 3` unless you set `I3HERE ON` you say? Wrong, this is what baffled me for a while. `SoftIce` doesn't want to respond, but it does due to the fact that `EBP` is set to, "BCHK", not as a pointer, but actual text. When `SoftIce` is active it has a debug exception called bounds checker. With, "BCHK", loaded into `EBP` the breakpoint instruction (`INT 3`) calls the exception handler for `SoftIce`'s bounds checker. `SoftIce` doesn't expect anything coming in and the stack isn't prepared or something similar because a page fault will occur after a `return`. To break this final protection you must set a breakpoint for `IsDebuggerPresent` not because you need to change the return value of that function, just because it's a good stopping point. Pressing `F12` in `SoftIce` to return from that function and `F12` twice again will get you back into the original program code and out of the security (armadillo) dll. Now `F10` (trace over) a bunch of times and you'll get to the `INT 3` instruction. You want to stop right after `EBP` is initialized. Type `R EBP\=4444444` in `SoftIce`. Now it's tricked and pressing `CTRL + D` to exit Ice will result in the target program running. So how does this really work? If there is no debugger present, Windows or `SoftIce`, the `INT 3` causes an unhandled exception handler to trip. This unhandled exception handler is right after the `JMP` instruction. If a debugger exists program flow will continue after the `INT 3`. A bunch of code to crash the program intentionally is after this `INT 3` instruction. If not it jumps over to the exception block and flow continues as if there were no debugger present.

\--------------------------------------------------------

Now you know the insides. Surprisingly, you don't need any of the above information. You can un-arm any Armadillo application in a few easy steps. It isn't necessary to figure out what version Armadillo is protection the program either. This you will know if you need to do extra `1.80 - 1.8x` steps. You do however, need to know that armadillo is protecting the program. The above tips will let you know if this is the latter. Some other useful information is the use of `GetPrivateProfileString`, and it's brothers and sisters by protected applications to tell if they are for instance, "PAIDFOR". Dis-assemble the security DLL, Armadillo `1.8x`, its protected programs and you can pick other variables out as well. Whoops, that makes it really hard to patch a un-dilloed application doesn't it? To think, some programmers are paying $89.00 for this! The only people is it working for is Silicon Realms Toolworks employees as they can easily use any application that uses it for free. Well, now, so can us all.

\--------------------------

1. Without any debuggers present, including `SoftIce`, start the protected application. The file you need is the program's name you are running followed by a number and "ProgName.tmpX". So, if you are starting "StayOnPro.Exe" you'll see in the local directory, "StayOnPro.tmp0". Copy this file by selecting it and pressing `CTRL+C` and `CTRL+V`.

2. Rename this copied file ("Copy of StayOnPro.tmp0") to read, "StayOnPro(undilloed).exe".

3. Start `WinHex 9.0` with the crack for it previously applied.

4. Press `ALT+F9` to start the process memory selection.

5. Select `StayOnPro.tmp0` and select it's Primary Memory.

6. Go to address `00400000H`. This address should always be the starting point of the windows exe image. You'll always know you are at the correct position if you see "MZ" at the address.

7. Now you are going to select a large block of memory. Click next to the `00400000H` address in the cell that shows `4D` press and hold the shift key down while pressing page down (this works the quickest). Select the block all the way to address `4323D6`. The same works for any other program. Ignore the first block of 0's after the program's header. This normally starts at around `400600H-401000H`. The code starts at normally `401000`. You want to select all the code till the block of zeros which separates a file section. In this case the second block of zeros starts at `4323D7` so we stop at `4323D6`. Sometimes the code segment, the `401000H` block starts at `0600H` in the file. Armadillo gives the code section away by putting "X"s where the code segment starts. You can always check this first to make sure.

8. Press `CTRL + C` to copy the selected block into the clipboard.

9. Choose `File/Open` and select the `StayOnPro(undilloed).exe` file to open it up.

10. Leaving the cursor right where it is on the `4D` block, choose menu `Edit/Clipboard data/Write` and press the okay button in the dialog telling you to write at offset `0`.

11. Save the file.

12. Stay On Pro is now almost completely un-armed. You may now easily disassemble and patch the file.

13. Did you need a debugger? Nope. Say thank you GrandPa. ;-)

\----------------------------------

Below is how to dump out Armadillo `1.80 - 1.8x` itself and Armadillo `1.8 - 1.8x` applications.

\-------------------------------------------------------

1. Without any debuggers present, including `SoftIce`, start the protected application. The file you need is the program's name you are running followed by a number and "ProgName.tmpX". So, if you are starting "Armadillo.Exe" you'll see in the local directory, "Armadillo.tmp0". Copy this file by selecting it and pressing `CTRL+C` and `CTRL+V`.

2. Rename this copied file ("Copy of Armadillo.tmp0") to read, "Armadillo1.8x(undilloed).exe".

3. Start `WinHex 9.0` with the crack for it previously applied.

4. Press `ALT+F9` to start the process memory selection.

5. Select `Armadillo.tmp0` and select it's Primary Memory.

6. Go to address `00400000H`. This address should always be the starting point of the windows exe image. You'll always know you are at the correct position if you see "MZ" at the address.

7. Now you are going to select a large block of memory. Click next to the `00400000H` address in the cell that shows `4D` press and hold the shift key down while pressing page down (this works the quickest). Select the block all the way to and including address `41894B`.

8. Press `CTRL + C` to copy the selected block into the clipboard.

9. Choose `File/Open` and select the `Armadillo1.8x(undilloed).exe` file to open it up.

10. Leaving the cursor right where it is on the `4D` block, choose menu `Edit/Clipboard data/Write` and press the okay button in the dialog telling you to write at offset `0`.

11. Save the file.

12. Armadillo `1.8x` or the program it's protecting is almost un-armed.

13. Unfortunately, starting the program this time results in an exception. What happened? The program's header is modified to go to a different starting code location. What do we do to find a code location? Follow on...

14. In this case we'll need `W32DASM` so fire it up.

15. Disassemble the file `Armadillo1.8x(undilloed).exe`. Neat huh?



16. Now the trick is to find the starting location of the program. Experience tells me almost all windows entry points have the functions `GetCommandLineA`, `GetStartupInfoA`, `GetModuleHandleA`, `GetVersion`, and `ExitProcess`. For this particular program do a search for `GetVersion`. Ignore the first and second find since that is only a reference to the import table. The third find isn't a correct match either, this is `GetVersionEx` which is not what we are looking for. The fourth find will be the correct location

17. Now look around. In this function do we have some or all of the above functions. Yes we do, this is most likely the windows entry point

18. The next step would be to trace backwards (move up) until a `NOP` or `RET` is found. In this case the `RET` is at `412D3D`. That makes the entry point: `412D40`. Close down `W32DASM`.

19. So let's plug this in and give it a try. Start `WinHex` up and load in the `Armadillo1.8x(unarmed).exe` file

20. Location `E0` starts the program header. At location `108H` is the entry point called by windows when the program starts

21. At `108H` we are going to write the 32-bit number of the entry point but we have to do a little math first.

22. The entry point is `412D40`. Subtract the virtual memory offset to get the correct value. This one loads in at `400000H`. So `412D40`-`400000H`=`12D40`. Make this a 32-bit number for ease and now it's `00012D40`.

23. Large numbers (words, doublewords) in windows are normally written backwards for this reason the number written will be `40 2D 01 00`. Write `40H` @ file offset `108H`, `2D` @ `109H`, `01H` @ `10AH` and `00H` @ `10BH` and save the file.

24. `Armadillo 1.8x` or it's protected program is now completely un-armed

25. Did you need a debugger? Nope. Say thank you GrandPa. ;-) `4dd466d0e1e0cc123afbe504da2462ca`
