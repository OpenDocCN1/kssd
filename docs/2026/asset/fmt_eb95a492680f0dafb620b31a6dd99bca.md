
```asm
.text:0060C963 mov eax, [eax]
.text:0060C965 mov dl, 1
.text:0060C967 call sub_6017F0
.text:0060C96C jmp short loc_60C9A8
```

游戏启动入口找到了，发现了一个替换原程序dll文件的代码，就在游戏进程启动之前：

```asm
push 80h ; dwFileAttributes
lea edx, [ebp+var_428]
mov eax, ds:dword_6755F4
mov eax, [eax+3E0h] ; this
call sub_495714
lea eax, [ebp+var_428]
mov edx, offset aOpengl32Dll ; "opengl32.dll"
call sub_407B68
mov eax, [ebp+var_428]
call sub_407954
push eax ; lpFileName
call SetFileAttributesW
lea edx, [ebp+var_42C]
mov eax, ds:dword_6755F4
mov eax, [eax+3E0h] ; this
call sub_495714
lea eax, [ebp+var_42C]
mov edx, offset aOpengl32Dll ; "opengl32.dll"
call sub_407B68
mov eax, [ebp+var_42C]
call sub_4213C4
lea edx, [ebp+var_430]
mov eax, ds:dword_6755F4
mov eax, [eax+3E0h] ; this
call sub_495714
lea eax, [ebp+var_430]
mov edx, offset aOpengl322Dll ; "opengl322.dll"
call sub_407B68
mov ecx, [ebp+var_430]
mov edx, offset aOpengl32 ; "opengl32"
mov eax, offset off_60CCB4
call sub_6076D0
lea edx, [ebp+var_434]
mov eax, ds:dword_6755F4
mov eax, [eax+3E0h] ; this
call sub_495714
mov edx, [ebp+var_434]
lea eax, [ebp+var_10]
mov ecx, offset aOpengl322Dll ; "opengl322.dll"
call sub_407BC0
lea edx, [ebp+var_438]
mov eax, ds:dword_6755F4
mov eax, [eax+3E0h] ; this
call sub_495714
mov edx, [ebp+var_438]
lea eax, [ebp+var_14]
mov ecx, offset aOpengl32Dll ; "opengl32.dll"
call sub_407BC0
mov eax, [ebp+var_14]
call sub_407954
push eax ; lpNewFileName
mov eax, [ebp+var_10]
call sub_407954
push eax ; lpExistingFileName
call MoveFileW
```

（用登陆器`opengl322.dll`替换掉原来的，即使读不懂代码，单步跟踪也是可以发现的。我曾经尝试过IDA分析替换后的`opengl32`文件，水平有限没看懂具体功能，估计是用作玩家个性化功能的实现，这个文件主要是玩家服装、枪和道具的功能，游戏中VIP玩家样式是不同的，实现功能可能是在这里）

```asm
cmp eax, 1
sbb eax, eax
inc eax
jmp short loc_60C5C1
```

### 4. 关键的问题：登陆器是如何监护游戏进程，进行截图反作弊的

这是Windows平台进程通信机制的问题，正如前文所述，对于我这技术小白来讲只能查资料，一步步排除，这个过程很繁琐；但，只要明白了几种通信机制的原理，大体可以判断出来用什么实现的。

首先肯定的是，登陆器不能一直打扰游戏进程，不能进行大量的数据通信和处理，因为这会降低游戏进程的效率，这种方式是不可取的，也完全没有必要。所以，一定要通过一种最简单、效率最高的方式进行进程间协作。

其次，进程间主要进行的是图片的操作和处理，这里涉及文件的操作、图片的操作，因此可以从这两个方向进行代码定位。但是，经过跟踪排查，这两个方向均是错误的。

当定位文件操作时，发现Dephi的代码并无明显的文件操作函数，因为我对Dephi不熟悉，跟踪`SaveToStream`函数时，发现并非截图相关代码。这里边走了很多弯路。

而且，Dephi里边有个巧妙的应用，就是窗口函数的跟踪，消息的处理机制虽然底层依然是基于Windows消息驱动机制，但是Dephi做了优化和处理，也就是通过跟踪代码理清消息究竟到了哪个窗口过程，是很不容易的。后来，查了资料，有大佬指出，Dephi的这种机制其实是可以进行代码保护的，就是把窗口函数私有化，即使DDDark也是不容易找到入口的，因此，当初的确小瞧了Dephi和这款软件没有加壳了，不加壳不等于没有反跟踪保护的。

定位File相关函数，如`fopen`、`fwrite`、`fread`等，用WinDbg跟踪拦截均失败，说明显然两个进程之间没有通过文件进行协作的；准确的说，跟踪登陆器进程的文件操作代码，并没有拦截到游戏进程的截图和文件操作；这说明，截图和保存均是在游戏进程中实现的。这个问题就更复杂了，难道是登陆器能够控制游戏进程进行文件操作？这其中的代价有点高吧？

我当初首先想到的是hook，以为登陆器可以hook游戏进程进行截图，但这也是不对的，因为如果是这样的话，游戏进程需要在截图完成后通知登陆器进程进行上传操作，这又是进程间通信，而且hook游戏进程难度较大（并不是说软件作者技术无法实现），而且会带了很多问题。

当时，进行到这里，的确是把我难住了。如果文件函数定位失败，那就换个思路。

然后拦截`BitBlt`函数，发现截图是在一处调用`BitBlt`之前已经完成了，在这里倒是发现了上传服务器的代码：

```asm
0059AA15 call dword ptr [ecx+38h]
.text:0059AA18 push 40CC0020h ; DWORD
.text:0059AA1D push 0 ; int
.text:0059AA1F push 0 ; int
.text:0059AA21 mov eax, [ebp+hDC]
.text:0059AA24 push eax ; HDC
.text:0059AA25 mov eax, off_66F110
.text:0059AA2A mov eax, [eax]
.text:0059AA2C call sub_543ABC
.text:0059AA31 push eax ; int
.text:0059AA32 mov eax, off_66F110
.text:0059AA37 mov eax, [eax]
.text:0059AA39 call sub_543AC4
.text:0059AA3E push eax ; int
.text:0059AA3F push 0 ; int
.text:0059AA41 push 0 ; int
.text:0059AA43 mov eax, [ebp+var_28] ; this
.text:0059AA46 call @Graphics@TBitmap@GetCanvas$qqrv ; Graphics::TBitmap::GetCanvas(void)
.text:0059AA4B call sub_46ECE4
.text:0059AA50 push eax ; HDC
.text:0059AA51 call BitBlt （读取截图）
.text:0059AA56 test eax, eax
.text:0059AA58 jnz short loc_59AA69
.text:0059AA5A call @System@@TryFinallyExit$qqrv ; System::__linkproc__ TryFinallyExit(void)
.text:0059AA5F call @System@@TryFinallyExit$qqrv ; System::__linkproc__ TryFinallyExit(void)
.text:0059AA64 jmp loc_59ACFF
……
0059ABBC mov eax, esi
.text:0059ABBE mov ebx, [eax]
.text:0059ABC0 call dword ptr [ebx+90h]
.text:0059ABC6 lea eax, [ebp+var_10]
.text:0059ABC9 mov edx, offset aYyyyMmDdHhMmSs_0 ; "yyyy-MM-dd hh:mm:ss"
.text:0059ABCE call sub_4070CC
.text:0059ABD3 call @Sysutils@Now$qqrv ; Sysutils::Now(void)
.text:0059ABD8 fstp [ebp+var_38]
.text:0059ABDB wait
.text:0059ABDC push dword ptr [ebp+var_38+4]
.text:0059ABDF push dword ptr [ebp+var_38]
.text:0059ABE2 lea ecx, [ebp+var_14]
.text:0059ABE5 mov edx, off_66ED4C
.text:0059ABEB mov eax, [ebp+var_10]
.text:0059ABEE call @Sysutils@FormatDateTime$qqrx17System@AnsiString16System@TDateTimerx24Sysutils@TFormatSettings ; Sysutils::FormatDateTime(System::AnsiString,System::TDateTime,Sysutils::TFormatSettings &)（打时间码，上传服务器的每张截图都有时间码）
.text:0059ABF3 lea eax, [ebp+var_70]
.text:0059ABF6 push eax
.text:0059ABF7 mov eax, [ebp+var_14]
.text:0059ABFA mov [ebp+var_64], eax
.text:0059ABFD mov [ebp+var_60], 11h
.text:0059AC01 lea edx, [ebp+var_64]
.text:0059AC04 xor ecx, ecx
.text:0059AC06 mov eax, offset dword_59AE00
.text:0059AC0B call sub_421D6C
.text:0059AC10 mov eax, [ebp+var_70]
.text:0059AC13 push eax
.text:0059AC14 mov ecx, 0AAh
.text:0059AC19 mov edx, 1Eh
.text:0059AC1E mov eax, esi
.text:0059AC20 mov ebx, [eax]
.text:0059AC22 call dword ptr [ebx+90h]
.text:0059AC28 mov dl, 1
.text:0059AC2A mov eax, ds:off_586AC8 ; this
.text:0059AC2F call @Jpeg@TJPEGImage@$bctr$qqrv ; Jpeg::TJPEGImage::TJPEGImage(void)
.text:0059AC34 mov [ebp+var_30], eax
.text:0059AC37 xor eax, eax
.text:0059AC39 push ebp
.text:0059AC3A push offset loc_59AC8F
.text:0059AC3F push dword ptr fs:[eax]
.text:0059AC42 mov fs:[eax], esp
.text:0059AC45 mov edx, [ebp+var_2C]
.text:0059AC48 mov eax, [ebp+var_30]
.text:0059AC4B mov ecx, [eax]
.text:0059AC4D call dword ptr [ecx+8]
.text:0059AC50 mov eax, [ebp+var_30]
.text:0059AC53 mov byte ptr [eax+44h], 3Ch
.text:0059AC57 mov eax, [ebp+var_30] ; this
.text:0059AC5A call @Jpeg@TJPEGImage@Compress$qqrv ; Jpeg::TJPEGImage::Compress(void) （压缩图片）
.text:0059AC5F mov dl, 1
.text:0059AC61 mov eax, ds:off_43E42C
.text:0059AC66 call sub_405028
.text:0059AC6B mov [ebp+var_18], eax
.text:0059AC6E mov edx, [ebp+var_18]
.text:0059AC71 mov eax, [ebp+var_30]
.text:0059AC74 mov ecx, [eax]
.text:0059AC76 call dword ptr [ecx+60h]
.text:0059AC79 xor eax, eax
.text:0059AC7B pop edx
.text:0059AC7C pop ecx
.text:0059AC7D pop ecx
.text:0059AC7E mov fs:[eax], edx
.text:0059AC81 push offset loc_59AC96
```

定位`BitBlt`函数也是失败的。

只能再换个思路，我在想，既然登陆器要对游戏进程截图，至少在截图前首先定位游戏进程的窗口，因此是否可以进行`findwindow()`函数的定位找到代码，通过WinDbg拦截还是失败，但是这次，结合登陆器的截图机制，就是时间函数的调用，发现了一个疑似接近截图的地方。

这其中进行过`Settimer`函数的定位，发现定期截图（其实是离散时间的，控制在一定时间内）并不是通过这个函数实现的。

但是，跟踪`Gettickcount`函数就可以在截图之前进行程序的拦截，说明这一次起码能控制截图的实现了。代码在这里：

```asm
00602798 loc_602798: ; CODE XREF: .text:00602785↑j
.text:00602798 call GetTickCount_0
.text:0060279D sub eax, ebp
.text:0060279F cmp edi, eax
.text:006027A1 jnb short loc_60280D
.text:006027A3 mov eax, [ebx+0Ch]
.text:006027A6 cmp byte ptr [eax+2BEh], 0
.text:006027AD jnz short loc_6027D1
.text:006027AF mov eax, [ebx+0Ch]
.text:006027B2 mov byte ptr [eax+2Ch], 0
.text:006027B6 mov eax, [ebx+0Ch]
.text:006027B9 call sub_602214
.text:006027BE mov [esp+4], al
.text:006027C2 mov eax, [ebx+0Ch]
.text:006027C5 mov eax, [eax+34h]
.text:006027C8 mov dl, 1
.text:006027CA call unknown_libname_848
```

`006027B9` 之前断下，登陆器不会截图；跳过这个代码，下一处断下，就有截图了，说明`text:006027B9 call sub_602214`，是控制截图的代码实现的。

对于有经验的朋友，其实很容易跟进去发现的，但对于我来说第一次跟踪到这里时，有点不识庐山真面目了；因为我一直困惑，截图功能究竟是登陆器实现的还是游戏实现的，如果在这里能控制截图的话，是不是就是登陆器实现的？那对登陆器进行文件和图片的拦截是能够成功的，问题在于之前几次尝试并未成功；所以，这次真的有点懵了。

而且，截图后的图片是保存在游戏文件路径下的；可以确定的是截图功能就是游戏进程实现的；但是登陆器要对是否截图、什么时候截图、截图后读取截图（还要删除）等进行控制。

那这个过程怎么才能实现呢？我在想，既然游戏实现截图，是否自身就带有截图功能，如此来说就简单了。经过查资料，发现还真有，就是通过快捷键完全可以截图，截图保存路径和命名方式完全和登陆器是匹配的。那就简单了，其实就是通过登陆器模拟一次键盘截图操作就可以了，通过消息传递给游戏进程告诉游戏该截图了，然后等待一段时间，等游戏截图完成，再读取截图上传服务器就可以了。跟进上文的那个地址代码：

```asm
00602214 sub_602214 proc near ; CODE XREF: .text:006027B9↓p
.text:00602214 ; sub_60338C+2F↓p
.text:00602214 push ebx
.text:00602215 push esi
.text:00602216 xor ebx, ebx
```
