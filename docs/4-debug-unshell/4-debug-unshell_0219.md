
# Armadillo-手动脱变种的壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6331.htm`](https://www.pediy.com/kssd/pediy06/pediy6331.htm)

**Armadillo-手动脱变种的壳**///   
\*\*\*\* 原著 `crUsAdEr` \*\*翻译 `FTBirthday` \*\*\*\*

这篇教程的目标是讨论 `Armadillo` 2.61 的保护机制以及如何手动地脱去 `armadillo` 的保护层! 希望以此能够展示出,被越来越依赖于工具的 `cracker` 们,遗忘的手动脱壳技术的威力.

需要的工具:

*   `IDA` 4.15
*   `Soft Ice on Win2k/SP3`
*   `LordPE Deluxe`

脱壳对象: `Get Right` 5.0 beta 1

下载连接: `GetRight5.0beta`

## 序言

`Armadillo` 2.61 只是附加了一些新特性,使得逆向工程变得更有趣.`Armadillo` 调试由它自身保护的软件, 这一点使得我们这些 `crackers` 在跟踪调试目标时,遇到了很大的困难.但是好的一方面是 `Armadillo` 自身的代码并没有被以任何方式加上花指令或者加密,所以我们可以用 `IDA` 反汇编保护层,以供研究之用.

i) 这篇 tutorial 中的所有代码片断都是从 `IDA` 反汇编得到的,包括 `IAT` redirection 部分,其他部分也可以在相同的地址在 `IDA` 反汇编代码中被找到,只要你获得的是相同版本的 `Getright.exe`.

ii) 贯穿这篇 essay,我以 `ebp+someName` 的形式使用变量名,以使得读者在 `softice` 中跟踪调试时易于理解,你需要键入实际值,例如`"ebp+FFFFFAE0"`

iii) `Armadillo` 保护的程序开启了两个进程,保护壳调试被保护的目标,所以我有必要说明,调试器为`"server"` 被调试软件为`"Client"`

iv) 同样要注意的是 `IAT` redirection 在 `WIN9x` 平台上是不同于 `winNT/2k/XP` 的,所以这篇 essay 仅仅讨论 `IAT` redirection 在 `win2k` 平台,当然你可以在 `win9x` 平台上以类似的方法找到 `IAT` redirection routine!

(最后,请在 `Fravia` board 上阅读关于 `Armadillo` 保护的理论和方法,还要确信你已经牢固的理解了 `PE` 文件格式,因为那是重建 `PE` 映象的基础!)

## 第一步. 打败 copy-mem II

好了, 我希望你已经知道什么是 `copy-mem` II 和它能做什么. 标准的流程, 下 `bpx SetProcessWorkingSetSize` 去找 `OEP`

一旦 `softice` 中断了,检查 `softice` 是否在 `Get Right` 的领空,如果不在则按 `F5` 直到 `softice` 在 `Get Right` 的领空中断.然后按 `F12` 辅之以 `F10` 跟踪过去一些指令,直到你看到一个`"call edi"`指令,停在那儿,然后写下 `edi` 的值.那就是 `OEP`.移除所有断点,然后`"bpx WriteProcessMemory"`.按 `F5`,一旦 `softice` 中断按 `F12` 两次你将会在通常所谓的 `Armadillo` 的 decrypting routine, 这个你在先前的 `Armadillo` 版本中已经见过的.

```assembly
005E03FF 6A 00           push 0                      ; 标志(flag)表明 Decrypt 如果是 0
; 如果 flag 是 1 则 encrypt
005E0401 8B 4D 0C       mov ecx, [ebp+key_address]
005E0404 51             push ecx
005E0405 8B 55 08       mov edx, [ebp+block_no]
005E0408 52             push edx
005E0409 E8 76 01 00 00  call Decrypt_Encrypt        ;<--这里对异常页解码 call Decrypt_Encrypt
005E040E 83 C4 0C       add esp, 0Ch                 ;==YOU SHOULD LAND HERE!!!!
```

试着在这些指令间循环以迫使它 decrypt 所有的 code section 然后你将会得到一堆垃圾,因为现在每 `1000h` bytes 的 memory block 都有一个不同的 decrypt key.所以我们将必须去找出 decrypt key 的产生 routine.

再继续跟踪你将会看到:

```assembly
0044E421 A1 88 9A 45 00      mov eax, ds:block_count               ;<==注意!!! ds:block_count
0044E426 83 C0 01           add eax, 1                              ; 已经 decrypted memory block 数量加 1
0044E429 A3 88 9A 45 00      mov ds:block_count, eax               ; in process counter

\--------SNIP-----------

0044E470                    mov edx, ds:block_count               ; compare number if decrypted blocks
0044E476 3B 15 70 66 45 00  cmp edx, ds:max_number_of_decrypted_block ; with maximum no. allowed
0044E47C 0F 8E FA 00 00 00  jle ok                              ; <=则继续解密块
```

正如你能猜到的,`Armadillo` 储存了已经 decrypted 的 memory block 数量,以确保在任何时刻,并非所有的 blocks 都被同时 decrypted 到内存中.如果你`"bpm max_number_of_decrypted_block"`,你将会看到 `Armadillo` 通过把 text code section size 除以 `1000h` (number of memory blocks)然后再除以 2,来获得此值.这里你有两种方法, 给跳转打补丁以确保它始终跳转或者我们可以改变 value at `ds:max_number_of_decrypted_block` 为 `1000h`, 某些很大的值!!!!

做完这些后,按 `F12` 我们将在 `Armadillo` `"server"` 的调试循环(Debug loop)中了.

```assembly
0044D697                     LOOP:                   ; Start looping here to force
0044D697 8B 8D 2C FA FF FF    mov ecx, [ebp+mem_block]     ; decrypt the whole code section
0044D69D 3B 0D 84 9A 45 00    cmp ecx, ds:text_section_size
0044D6A3 0F 8D C7 00 00 00    jge continue_1               ;change to "jmp eip", infinite loop whenfinished
0044D6A9 8B 95 9C FA FF FF    mov edx, [ebp+trap_flag_not_set_flag]
0044D6AF 81 E2 FF 00 00 00    and edx, 0FFh               ;armadillo set trap flag at the beginning of(可以 decrypt 时 edx 被置 0 了)
0044D6B5 85 D2                test edx, edx               ;its debug loop to check if it is being traced(edx=edx and 0FFh=000h)
0044D6B7 0F 84 A9 00 00 00    jz continue_ok               ; and it is checking for the flag now!
0044D6BD 6A 00                push 0                    ; instruct to decrypt code
0044D6BF 8B B5 2C FA FF       mov esi, [ebp+mem_block]
0044D6C5 C1 E6 04             shl esi, 4
0044D6C8 8B 85 2C FA FF FF    mov eax, [ebp+mem_block]

\----------SNIP--------------- ; Decrypt key Generation

0044D73F 83 E7 0F             and edi, 0Fh                ; codes removed but u can refer to IDA
0044D742 03 F7               add esi, edi                ; disassembly
0044D744 8B 15 74 9A 45 00    mov edx, ds:key_address_table
0044D74A 8D 04 B2             lea eax, [edx+esi*4]
0044D74D 50                   push eax
0044D74E 8B 8D 2C FA FF FF    mov ecx, [ebp+mem_block]
0044D754 51                   push ecx
0044D755 E8 86 0B 00 00       call Decrypt_codes             ; push crypt_flag
0044D755                                      ; push key_add
0044D755                                      ; push mem_address
0044D75A 83 C4 0C             add esp, 0Ch                  ;==YOU SHOULD LAND HERE!!!!
0044D75D 25 FF 00 00 00       and eax, 0FFh
0044D762 85 C0                test eax, eax                 ;here patch to increment the mem block
0044D762                                      ;value to force decrypting the next block
0044D764 74 0A                jz short bad_jump             ;then jump to LOOP 0067B0E8
```

现在我们知道 `Armadillo` 如何为每个 memory block generates keys,然后把它传递 `Decrypt_codes` routine. 同样的, 我们可以理解为什么当我们单步跟踪调试`"server-debugger"`code 时 `Armadillo` 会悄无声息的退出了.这里,我们应该使得包含 key generation routine 的大 loop 去强迫 `armadillo` decrypt 整个 code section. 在地址 `44D75D` 处,do 下命令 `a eip`:

```assembly
inc dword ptr [ebp+mem_block]
jmp LOOP:44D697 ;跳回循环头
```

然后我们 edit 在`"ebp+mem_block"`处的值为 0 (告诉 `Armadillo` 从第一个 memory block 开始 decrypting), 并且设置 end point at `44D6A3` 通过改变那儿的指针为`"jge 46D6A3"`.这儿 `Armadillo` 在检查 mem block 是否超过了 code section,我们简单的设了一个无限循环在那儿.让程序运行, 等一会儿以后启动 `LordPE` 去 make a full image dump. 如果 `LordPE` 报告`"Can't grab memory"`那么你可能做错了什么,`Armadillo` 没有完全 decrypted the code section. 完成这一步, 我们进入下一阶段 rebuild IAT.

## 第二步.Rebuild Import data

用 `LordPE` 打开 packed file 然后研究 section header, 你将会发现 import data 应该在 section `.idata`! 为什么?因为名字 name 已经告诉了... 你可以再确认一下,用 `Hex Workshop` 打开 file 然后去那个 section offset,你将会看到 scattering(分散的) words like `"kernel32.dll"` and `"user32.dll"` 等等... Import data 应该就在那儿!

我们的任务是找到 `API` redirection routine, 研究它然后 based on this routine 去 rebuild our Import.

为了找到 `IAT` redirection routine,使用`"addr"`command of `softice` to 进入被保护(client) program 的领空, 然后

```
bpm IAT_address W (一旦中断,disable 这个断点)
bpx GetProcAddress (一旦中断,disable 这个断点)
F12 两次,我们现在应该在这儿.
```

```assembly
10006622 39 3D E8 C7 01 10          cmp dll_handle_table, edi         ; check if pointer is 0 because edi = 0 here
10006628 B8 E8 C7 01 10             mov eax, offset dll_handle_table  ; here is the Table listing
1000662D 74 0C                      jz   short no_special_IAT         ; special DLL and their handles
1000662F                            ; the APIs in these DLL will be redirected
1000662F  loop:   
1000662F 3B 48 08                  cmp ecx, [eax+8]                  ; is dll handle same as handle listed
10006632 74 1B                     jz   short special_dll
10006634 83 C0 0C                  add eax, 0Ch                      ; next dll on the table
10006637 39 38                     cmp [eax], edi
10006639 75 F4                     jnz short loop
1000663B

1000663B                            no_special_IAT:                   ; Get API address normally, no redirection!!!
1000663B                            push dword ptr [ebp+0Ch]          ; lpProcName
```



```
1000663E FF 75 08                     push [ebp+dll_handle]              ; int   
10006641 E8 41 00 00 00               call GetProcAddr   
10006646 59                           pop ecx                              ;==WE SHOULD LAND HERE!!!!   
10006647 59                           pop ecx   

----------SNIP---------;   

1000664F; ---------------------------------------------------------------------------   

1000664F                      special_dll:   

1000664F 8B 40 04                 mov eax, [eax+4]                   ; get Special IAT table   
10006652 3B C7                     cmp eax, edi                       ; if eax = 0   
10006654 74 E5                     jz  short no_special_IAT   
10006656 39 78 08                  cmp [eax+8], edi   
10006659 8B F0                     mov esi, eax                       ; esi points to start of special API   
1000665B 74 DE                     jz  short no_special_IAT           ; name table?these APIs will be redirected   

1000665D                      API_name_check_loop:   

1000665D 66 3B DF                 cmp bx, di   
10006660 74 06                     jz   short loc_10006668   
10006662 66 3B 5E 04              cmp bx, [esi+4]   
10006666 EB 0E                     jmp short loc_10006676   

10006668                      loc_10006668:   

10006668 FF 36                     push dword ptr [esi]               ; The API name in special API table   
1000666A FF 75 0C                  push dword ptr [ebp+0Ch]           ; our API name   
1000666D E8 8E 2B 01 00            call StringCompare   
10006672 59                         pop ecx   
10006673 59                         pop ecx   
10006674 85 C0                      test eax, eax                      ; if API name is found on the table   
10006676                                                             ; jump to Redirect_API_now   

10006676                      loc_10006676:   

10006676 74 0A                     jz  short Redirect_API_now   
10006678 83 C6 10                  add esi, 10h                       ; if not, update pointer on table to next API   
1000667B 39 7E 08                  cmp [esi+8], edi                     ; Is there anymore API on the table?   
1000667E 75 DD                     jnz short API_name_check_loop        ; if Yes then loop back to check this API   
10006680 EB B9                     jmp short no_special_IAT           ; else go ahead and find API address normally   

10006682; ---------------------------------------------------------------------------   

10006682                      Redirect_API_now:   

10006682 8B 46 08                 mov eax, [esi+8]                     ; eax = redirected API address   
10006685 EB C1                     jmp short end_routine 00A4A496     load_next_dl_import:   
```

那个 `dll_handle_Table` 看起来象这样的:

```
1001C7E8 18 C8 01 10     dd offset strKernel32_dll        ; "kernel32.dll"   
1001C7EC A8 C4 01 10     dd offset kernel32_API_table     

1001C7F0 00 00 00 00     kernel32_handle dd 0             

1001C7F4 0C C8 01 10     dd offset strUser32_dll          ; "user32.dll"   
1001C7F8 C8 C7 01 10     dd offset User32_API_table       
1001C7FC 00 00 00 00     User32_handle dd 0                
```

那个 `kernel32_API_table` 看起来象这样的:

```
1001C4A8 1C CB 01 10     dd offset strExitprocess         ; "ExitProcess"   
1001C4AC 00 00 00 00     dd 0   
1001C4B0 77 6C 00 10     dd offset Fake_ExitProcess        ; if string compare is same, this   
1001C4B4 00 00 00 00     dd ExitProcess_Address            ; Fake_ExitProcess routine is used!!!   

1001C4B8 08 CB 01 10     dd offset strTerminateprocess     ; "TerminateProcess"   
1001C4BC 00 00 00 00     dd 0   
1001C4C0 CA 6C 00 10     dd offset Fake_TerminateProcess   
1001C4C4 00 00 00 00     dd TerminateProcess_Address   

1001C4C8 FC CA 01 10     dd offset strExitthread            ; "ExitThread"   
1001C4CC 00 00 00 00     dd 0   
1001C4D0 00 6D 00 10     dd offset Fake_ExitThread   
1001C4D4 00 00 00 00     dd ExitThread_Address   
```

etc...

那么 `User32_API_table` 当然很类似!

在这一点上,如果你很懒,你可以简单地对 `dll_handle_table` 检测部分的代码打补丁,以跳过它.然后我们的 IAT 就永远不会被 redirected 了,我们可以简单地启动 Imprec 或者 Revirgin 选择 running process 去 re-build IAT... 但是我们是在手动地做一切,不是吗:)所以让我们继续 trace.如果你按 `F12` one more time,你将会在 Import loading loop 之中!

```
10010DED FF B5 70 FD FF FF        push [ebp+API_name]   
10010DF3 FF B5 94 FD FF FF        push [ebp+dll_handle]   
10010DF9 E8 FA 57 FF FF           call Get_API_address   
10010DFE 89 85 78 FD FF FF        mov [ebp+API_add], eax  <== WE SHOULD LAND HERE!!!!   
10010E04 83 BD 78 FD FF FF 00     cmp [ebp+API_add], 0   
10010E0B 75 38                     jnz short API_add_found   

<--------------SNIP--------------->   
10010E45                      API_add_found :   

10010E45 8B 85 80 FD FF FF        mov eax, [ebp+IAT_add]   
10010E4B 8B 8D 78 FD FF FF        mov ecx, [ebp+API_add]   
10010E51 89 08                     mov [eax], ecx <==HERE is where first thunk is updated!!!   
10010E53 8B 85 80 FD FF FF        mov eax, [ebp+IAT_add]   
10010E59 83 C0 04                 add eax, 4   
10010E5C 89 85 80                 FD FF FF mov [ebp+IAT_add], eax   
```

我的注释已经说过,设一个 breakpoint 在 `Call Get_API_Address` 然后 trace around the loop 去找它的 hang. 然后你将会看到`[ebp+API_name]`存储了 pointer to Import ASCIIs 紧接着这个 pointer,你将会看到一长窜 API 和 DLL 名"WOW"这就是我们要找的 Import ASCII :)吗,转储整个 Table 然后你会看到 dump 看起来如下:

```
WINMM.dll00   
9C511D00   
01000000   
PlaySoundA 0000KE   
RNEL32.dll00   
744B1D00   
9F000000   
ExitProcess 00T   
ErminateProcess 00 ExitThread 00 HeapReAlloc   
```

研究这张表,我们将会看到它由一个 DLL 开始,然后 RVA of first thunk corresponding with 那个 DLL, 然后从那个 DLL Import 的个数和从那个 DLL 所有 Imports ASCII 列表. 例如,`WIMM.dll` first thunk 从 `001D519C` 开始, (记住字节是反转的)那儿有一个从这个 DLL 的 import form 它是 `PlaySoundA`. `Kernel32.dll` first thunk 从 `1D4B74` 开始,那儿有 `9Fh` Imports from `kernel32.dll`...等等

更进一步,如果当你在指令 `10010E51` 处通过`"d eax"`来看 first thunk,你将会注意到 First Thunk 没有被 Armadillo touched !那意味着我们知道了 original location of ASCII entries of API, 但是看一下 those location 我们只看到了很多 0:(...所以 Import ASCII 被删除了,但是嘿,至少我们知道了 each entry of Import ASCII 被存储在何处.

所以我们现在要做我们必须得做的?我们需要 rebuild Import Table,换句话说就是 copy Import ASCII to location where they should be. Time for some patching on the fly! 设一个断点`"bpm 10010DF3 x"` 然后重启程序. 一旦中断.

```
a eip   
mov ecx, [ebp+IAT_add]; get the RVA of Import ASCII   
add ecx, 400002h ; calculate VA of Import ASCII by adding image base and 2 is size of HINT   
push ecx   
call lstrcpy   
jmp API_add_found   
```

啊哈,我希望你理解了那些补丁代码的作用.下一个任务是阻止 Armadillo 破坏我们的 First Thunk, 简单的 NOP `10010E51` 处的指令.最后,按 `F12` 让 Armadillo 为你 rebuilt IAT... 一旦 softice 再次中断 dump 整个.idata segment.

当然你将会注意到 the Import Directory 丢失了,所以你的任务就是 rebuild this directory. 我们应该把它当作留给读者的练习吗.如果你很想知道 Import by ordinal 因为我们不能复制 Import ASCII 象 import by name 一样,别担心! Import by ordinal is handled 独立地被 Armadillo (如果你在 softice 窗口中翻页代码一至两页,你将看到 all `Get_API_address` 都在那儿被 called as well to handle ordinal cases. 然而,正如我说过的,我们的 First Thunk 是完整的所以 Ordinals 已经被存储在 First Thunk 中了只要我们 NOP 了 `10010E51` 处的代码, 我们的 Import by ordinals are preserved!

请注意:Another more generic approach would be coding a small utility that reads the Import ASCII dump and insert Import ASCII into our exe dump file and rebuild the Import Directory.这将会在当前任何 Armadillo 保护目标上都有用!!!

### 3.新的 int3 trick

现在启动 exe and BOOM, 你将会从 Windows 得到一个消息说 int3 occurs 但是没有被 handled. 那是最新的 trick, Armadillo 用来阻止被转储. 是时候重新进入 softice 和 IDA 去跟随了,仅仅去看 Debug loop "server" 你将会看到 int3 handling routine.

```
0044DAAC 8B 85 38 FA FF FF     mov eax, [ebp+lpDebugEvent]   
```



### 1. INT3 异常检测与处理

`0044DAB2 81 78 0C 03 00 00 80`   `cmp dword ptr [eax+0Ch], 80000003h ; INT 3 exception code`   

`0044DAB9 0F 85 96 03 00 00`      `jnz loc_44DE55`   

`0044DABF`                        `INT3_occur:`   

`0044DABF 33 C9`                  `xor ecx, ecx`   

`0044DAC1 8A 0D 6E 9A 45 00`     `mov cl, ds:int3_feature_installed`   

从这儿开始我们的追求去 `kill int3`. 从这个 `point` 开始 `tracing`, 你将会看到 `"server"` 从 `"client"` `memory space` 读了 a `block of data` 然后将其分成四块 `tables` 然后将这 `tables` `decrypt` 到 4 块独立的虚拟内存空间中去. 然后 `"server"` 使用 `GetThreadContext` 去检测 `"client"` 中 `eip` 的 `location`. 这个 `eip` is one byte after the `location int3 exception occurs! Trace on` and `we'll see` that `armadillo` performs `binary search` on `Table_1` to find the `position of eip`. 一旦 `eip location in Table_1 found`, 我们到达这里:   

`005DFDD5 8D 95 28 F7 FF FF`     `lea edx, [ebp+Context]`   

`005DFDDB 52`                     `push edx`   

`005DFDDC A1 58 BA 5E 00`        `mov eax, ds:table_2`   

`005DFDE1 03 85 20 F7 FF FF`     `add eax, [ebp+relative_location] ; location in table_1 where eip is found`   

`005DFDE7 8A 08`                 `mov cl, [eax]`   

`005DFDE9 51`                    `push ecx`   

`005DFDEA E8 24 12 00 00`       `call Get_Jump_Type? ; determine which kind of Jump it is`   

`005DFDEF 83 C4 08`            `add esp, 8`   

`005DFDF2 25 FF 00 00 00`       `and eax, 0FFh`   

`005DFDF7 85 C0`               `test eax, eax`   

`005DFDF9 74 1C`               `jz short it_is_NOT_a_jump ; if al = 1 then it is a jump`   

`005DFDFB 8B 95 20 F7 FF FF`   `mov edx, [ebp+relative_location]; if al = 0 then it is not a jump`   

`005DFE01 A1 48 BA 5E 00`       `mov eax, ds:Table_4`   

`005DFE06 8B 8D E0 F7 FF FF`   `mov ecx, [ebp+Context.Eip]; add eip with corresponding value in Table_3`   

`005DFE0C 03 0C 90`            `add ecx, [eax+edx*4]? ;?relative location = location of eip`   

`005DFE0F 89 8D E0 F7 FF FF`   `mov [ebp+Context.Eip], ecx ;?in Table 1`   

`005DFE15 EB 1E`               `jmp short Update_eip`   

`005DFE17`                      `--------------------------------------------------------------`   

`005DFE17`                      `it_is_NOT_a_jump:`   

`005DFE17 8B 15 5C BA 5E 00`   `mov edx, ds:Table_3`   

`005DFE1D 03 95 20 F7 FF FF`   `add edx, [ebp+relative_location] ; set edx to corresponding value in Table_4`   

`005DFE23 33 C0`               `xor eax, eax`   

`005DFE25 8A 02`               `mov al, [edx] ; get distance eip to be moved by`   

`005DFE27 8B 8D E0 F7 FF FF`   `mov ecx, [ebp+Context.Eip]`   

`005DFE2D 03 C8`               `add ecx, eax`   

`005DFE2F 89 8D E0 F7 FF FF`   `mov [ebp+Context.Eip], ecx; update eip`   

`005DFE35`                      `Update_eip:`   

`005DFE35 8D 95 28 F7 FF FF`   `lea edx, [ebp+Context]`   

`005DFE3B 52`                  `push edx; lpContext`   

`005DFE3C 8B 85 F4 F9 FF FF`   `mov eax, [ebp+hThread]`   

`005DFE42 50`                  `push eax; hThread`   

`005DFE43 FF 15 80 80 5E 00`    `call ds:SetThreadContext; SetThreadContext:`   

### 2. Armadillo 跳转修复机制分析

基本上, `Armadillo` 已经在 `original exe` 中通过 `int3` 来 `replaced` 一些跳转(条件跳转和无条件跳转) 然后产生 4 `tables` 存储了那些跳转的信息包括位置,距离. 所有的都在 `packing time` 完成. 正如你能看到的, 当 `int3` 在 `"client"` 进程中发生时, `"server"` 会检查 `int3` 发生处的 `eip` 值, 向上看 `Table_1`, 一旦被发现, `"server"` 会在 `table_2` 中检查相符合的值, 那指的是在这个 `eip` 处是何种跳转. 然后 `"server"` 会检查 `eflags register` 在 `"Client"` 领空(在 `call Get_Jump_type` 中 `routine`) 去看是否 `"client"` 应该 `jump` 或不跳. 如果 `jump` 然后 `"Client"` 的 `eip` 会被移入 `Table_4` 中相应(`jump distance`)的值, 如果不跳然后 `"Client"` 的 `eip` 会被移入 `table_3` 中相应(`length of jump instruction`)的值. 看一下 `Get_Jump_Type` `routine`.

#### 2.1 跳转类型示例

`005E1046`                       `jump type 9:`   

`005E1046 B0 01 mov al, 1`      `;al always 1 => always jump no matter what`   

`005E1048 E9 AF 01 00 00`       `jmp end ;==>> JMP`   

`005E104D`                       `jump type E:`   

`005E104D 8B 55 0C`            `mov edx, [ebp+context]`   

`005E1050 8B 82 C0 00 00 00`   `mov eax, [edx+0C0h] ; get eflags register`   

`005E1056 83 E0 41`            `and eax, 41h; check Zero and Carry flag`   

`005E1059 F7 D8`               `neg eax ; eax is negative if either flag set`   

`005E105B 1B C0`               `sbb eax, eax; eax = -1 if either flag is set`   

                               `; eax = 0 if neither flag is set`   

`005E105D 40`                  `inc eax ; al = 1 when neither flag set`   

`005E105E E9 99 01 00 00`       `jmp loc_5E11FC? ; ==> jump when CF=0 and ZF=0 => JA`   

`005E108C`                       `jump type 4 ; CODE XREF: Get_Jump_Type+2Cj`   

`005E108C 8B 55 0C`            `mov edx, [ebp+context]`   

`005E108F 8B 82 C0 00 00 00`   `mov eax, [edx+0C0h] ; get eflags register`   

`005E1095 25 80 00 00 00`      `and eax, 80h; check Sign Flag`   

`005E109A F7 D8`               `neg eax`   

`005E109C 1B C0`               `sbb eax, eax`   

`005E109E 40`                  `inc eax ; eax = 0 is S flag set, else eax = 1`   

`005E109F 8B 4D 0C`            `mov ecx, [ebp+context]`   

`005E10A2 8B 91 C0 00 00 00`   `mov edx, [ecx+0C0h] ; get eflags`   

`005E10A8 81 E2 00 08 00 00`   `and edx, 800h ; check Overflow flag`   

`005E10AE F7 DA`               `neg edx`   

`005E10B0 1B D2`               `sbb edx, edx`   

`005E10B2 42`                  `inc edx ; edx = 0 if O flag set, else edx = 1`   

`005E10B3 33 C9`               `xor ecx, ecx`   

`005E10B5 3B C2`               `cmp eax, edx; compare eax with edx`   

`005E10B7 0F 95 C1`            `setnz cl; al = 1 if not equal`   

`005E10BA 8A C1`               `mov al, cl`   

`005E10BC E9 3B 01 00 00`       `jmp end ; jump if SF != OF==>> JL`   

好的, 总共那儿有 `11h` 种 `jump`. 最后, `fix` 那些 `int3` 就是我们的任务. 所以我编了一小段代码去 `scan through Table_1`, 检测每一个 `virtual address entry in table_1` 在我们转储的文件中, 如果 `opcode` 是 `"CC"` 那它就 `out int3`, 检测 `Table_2` 中相应的值以决定它是那种跳转(`JNZ` or `JZ` or `JMP` 等等). 然后它检测 `table_3` 去看 `how long the jump instruction should be` 然后写入指定的正确的 `jump opcodes` 到我们的 `exe dump` 文件中. 最后, 它检测 `Table_4` 中相应的值, 那个值是 `distance how far the jump is` 然后再次 `update` 在 `exe` 中的值. 因此, 运行被保护的程序, `dump the 4 tables into 4 separate files`, (`确信 that Table_1 size is same as table_4 size and is 4 times the table_2 and 3 size. Why?`). 我已经免费附加了这个小程序的 `masm` 源代码, 以供你做参考.

### 4. 最后的, 决定性的 Touch

在 `fixing int3` 之后, 转储的文件仍旧拒绝运行, 以很吵的 `beeping` 退出. 这个使我一时很受挫很迷惑... `tracing` doesn't seem to get me anywhere, 我又重新检查了一遍我的 `int3 patching routine`, 和 `IAT`... 所有的东西看起来都很正常. 我已经想尽办法, 突然间我想到, 嘿, 那些 `Beep sound` 并未在我运行 `original exe` 时出现, `voila`, `bpx MessageBeep` 然后按 `F12` 我们到了这儿...

`00438E03 68 C4 0E 5C 00`         `push offset strDellatsnisyad ; "DELLATSNISYAD" <==WHAT A STRING!!!`   

`00438E08 8D 4D E4`              `lea ecx, [ebp-1Ch]`   

`00438E0B 89 46 6C`              `mov [esi+6Ch], eax`   

`00438E0E E8 F4 42 10 00`        `call 4CString@@QAEABV0@PBD@Z ; CString::operator=(char const \*)`   

`00438E13 8D 4D E4`              `lea ecx, [ebp-1Ch]`   

`00438E16 E8 91 46 10 00`        `call MakeReverse@CString@@QAEXXZ ; The name says it all :>`   

`00438E1B 8B 7D E4`              `mov edi, [ebp-1Ch]`   

`00438E1E B8 FF 00 00 00`        `mov eax, 0FFh`   

`00438E23 50`                    `push eax ; nSize`   

`00438E24 50`                    `push eax`   

`00438E25 8D 4D E8`              `lea ecx, [ebp-18h]`   

`00438E28 E8 84 45 10 00`        `call GetBuffer@CString@@QAEPADH@Z ; CString::GetBuffer(int)`   

`00438E2D 50`                    `push eax ; lpBuffer`   

`00438E2E 57`                    `push edi ; lpName`   

`00438E2F FF 15 8C 4C 5D 00`     `call ds:GetEnvironmentVariableA ; GetEnvironmentVariableA:`   

`00438E35 85 C0`                `test eax, eax ; check how many days installed :)`   

`00438E37 6A FF`                `push 0FFFFFFFFh`   

`00438E39 8D 4D E8`              `lea ecx, [ebp-18h]`   

`00438E3C 75 36`                `jnz short GOOD_JUMP ; if variable not found, BEEP n exit`   

`00438E3E E8 BD 45 10 00`        `call ReleaseBuffer@CString@@QAEXH@Z ; CString::ReleaseBuffer(int)`   

`00438E43 6A 01`                `push 1 ; uType`



`00438E45 FF 15 64 50 5D 00`          call `ds:MessageBeep` ; `MessageBeep:`   

`00438E4B 53`                            push `ebx` ;<== LAND HERE   

所以我们的任务就是去为 `program` 的 conditional jump 打补丁, 就像试图去恢复一个 `download` 你将会收到另一个 `beep` :)... 哈 太好,我们只需要用 `IDA`, click on `GetEnvironmentalVarableA` and 检查所有参考, 什么地方 called 去检测 `Armadillo` 全局变量像 `DAYSINSTALLED`,等等.注意那些检测后面通常紧跟着一个 `MessageBeep` 你会很快找到那些 check.   

这是 patches 的概要   

`438E3C?75?<== EB DAYSINSTALLED`   

`4720EE?75?<== EB DAYSINSTALLED`   

`4E6B7F?75?<== EB? ALTUSERNAME? `   

现在 `Get Right` 就像一个婴儿一样运行:)... 虽然我发现下载很慢.我个人不推荐你去使用这个下载引擎:),所以一旦你 crack 它,请立即卸载它!   

### 5. 尾声   

这就是它, phew. 好长的一篇 tutorial!我希望你觉得它有帮助.我已经尽我所能详细解释,但是我相信我不可能解释清楚每一个细节问题或者这篇 tutorial 太长了以至于任何人读它都会觉得很枯躁. 所以你发现任何有不清楚的部分,在 `fravia board` 上发一个消息给我,我会作出适当的修正.   

针对狂热爱好者:如果你注意到 `IAT redirection routine` 是在一些隐藏的 `memory space`, 你可以试着 scrolling up 到 beginning of this memory section 然后你将会看到一个 `PE header`! Check out the image size in the header,设置一个 `bpm` 在 image 的尾部(以确保整个 image 被装载到内存) 然后重新运行 `Get Right`,一旦 `softice` 中断,检测整个 image 是否被装载到内存?转储它并重命名为 `rma.dll` 你将会得到一个 virgin `dll` 它 export functions like `"LoadProgramInfo"`, `"NukeNow"`, `"CheckNetLicense"`, `"ArmAccess"` and `"RunUserProgram"`. 希望无限了:)???   

                                                                                                                        --------------------------------------------------------   

如果离开了 `Fravia Message Board` 上人们的帮助,这篇 tutorial 不可能诞生. 所以感谢飞向 `Fravia board` 和 `mirc` 上所有的朋友.   

特别感谢 `Woody` 和 `March^drn` (get well soon :)   

最后编辑: 7 Nov 2002   

Cheers,干杯!   

 原著 `crUsAdEr` \*\*翻译 `FTBirthday`  

`4dd466d0e1e0cc123afbe504da2462ca`
