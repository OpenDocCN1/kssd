
# 脱Armadillo

3.75D 加壳的DLL.

脱Armadillo 3.75D 加壳的DLL.

DLL的名称`VA_X.DLL`（VASSIST_X 1246的加密的主要DLL）

使用工具：`ollydbg`、`010Editor`、`LordPE`、`PEID`。

目的：仅仅是为了兴趣。

声明：我是菜鸟.水平很次,很多时候可能用了很多笨的办法,如有发现,还望指出,让我有所提高:D

本文作者：AERROR

## 壳的执行流程/原理

入口点处是一个标准的由VC生成的DLL的入口代码。壳在执行时，首先检测`IsDubberPresent`，没有的话，分配一段内存，将脱壳的保护代码填充进去；为`.TEXT CODE`两段分配内存；

然后去执行这段动态代码。那里将会把真正的代码解压到`.TEXT CODE`...等段中，把`OEP`放进一个全局变量`[1EFB66C4]`中，并且处理`IAT`，处理重定位表，检测SOFTICE，文件有效性的检测，解压原程序中保护数据段到一个申请的内存中，调整跳转地址等等。

上面的完成后，就呼叫的`OEP`。

过程参考了FLY版主的文章。但`BP GetModuleHandleA+5`这样下断点的情况和FLY版主所说的情况很不一样。因为我之前在不脱壳的情况下（因为当时我不知道它是由Armadillo加壳的）用动态补丁的方法破解过它较早的版本，并把壳检测SOFTICE的代码和文件有效性效验的代码破解了，因此，比较熟悉壳的代码，我就一步一步地跟踪找出了OEP。但因此我就没有用`BP GetModuleHandleA+5`这样找出那个处理重定位表的代码。我用了另外的笨办法把它修复了（也不知是不是正确的，详细的做法请看下面的修复的过程）。

同时，这个DLL不同于FLY版主那个测试的DLL，它有一段是由壳动态分配的，并且脱壳了之后，还需要调用的代码。

（我真的是菜，虽然我把它搞定了，但我还是想问问，这个是不是COPYMEM II？望知道的大侠一定要答我，谢谢:D）

## 一.脱壳和修复

我自己写一个的加载DLL的工具，在加载这下断点，在`LdrLoadDll`下断点。

断下后，查找命令`call dword ptr ss:[ebp+8]`，于是到了下面：

```
//ntdll.dll

77F830DB    FF75 14         push dword ptr ss:[ebp+14]

77F830DE    FF75 10         push dword ptr ss:[ebp+10]

77F830E1    FF75 0C         push dword ptr ss:[ebp+C]

77F830E4    FF55 08         call dword ptr ss:[ebp+8]  -->//DLL 入口点,这个地址取决于操作系统.按F7跟进去.

77F830E7    8BE6            mov esp,esi

///

//跟着我们来到这DLL的入口点.

1EF9A977 >/$  55            push ebp

1EF9A978  |.  8BEC          mov ebp,esp

1EF9A97A  |.  53            push ebx

...

1EF9A9BB  |> \33C0          xor eax,eax

1EF9A9BD  |.  EB 4E         jmp short VA\_X.1EF9AA0D

1EF9A9BF  |>  57            push edi

1EF9A9C0  |.  56            push esi

1EF9A9C1  |.  53            push ebx

1EF9A9C2  |.  E8 59DAFEFF   call VA\_X.1EF88420   <-- //DLL main,这段是标准的VC生成的DLL的开始
                                                     //的代码,然后转入的就是壳的代码段了.

1EF9A9C7  |.  83FE 01       cmp esi,1

1EF9A9CA  |.  8945 0C       mov dword ptr ss:[ebp+C],eax

///////////////////////////////////////////////////////

1EF88420   $  55            push ebp

1EF88421   .  8BEC          mov ebp,esp

1EF88423   .  6A FF         push -1

1EF88425   .  68 004DFB1E   push VA\_X.1EFB4D00

1EF8842A   .  68 C0A8F91E   push <jmp.&MSVCRT.\_except\_handler3>      ;  SE handler installation

1EF8842F   .  64:A1 0000000>mov eax,dword ptr fs:[0]

.........

1EF88509   > \E8 181B0000   call VA\_X.1EF8A026    <--// 解壳和检验,呼叫OEP请看下面.

1EF8850E   .  E9 04010000   jmp VA\_X.1EF88617

1EF88513   >  837D 0C 00    cmp dword ptr ss:[ebp+C],0

/////////////////////////////////////////////////////

1EF8A026  /$  55            push ebp

1EF8A027  |.  8BEC          mov ebp,esp

1EF8A029  |.  81EC 08010000 sub esp,108

1EF8A02F  |.  6A 00         push 0                                   ; /Value = NULL

1EF8A031  |.  68 D032FB1E   push VA\_X.1EFB32D0                       ; |VarName = "INITIALIZEDLLADDR"

1EF8A036  |.  FF15 4430FB1E call dword ptr ds:[<&KERNEL32.SetEnviron>; \SetEnvironmentVariableA

1EF8A03C  |.  833D D468FB1E>cmp dword ptr ds:[1EFB68D4],0

1EF8A043  |.  74 20         je short VA\_X.1EF8A065

1EF8A045  |.  A1 D468FB1E   mov eax,dword ptr ds:[1EFB68D4]

1EF8A04A  |.  8945 F8       mov dword ptr ss:[ebp-8],eax

1EF8A04D  |.  C705 D468FB1E>mov dword ptr ds:[1EFB68D4],0

1EF8A057  |.  FF55 F8       call dword ptr ss:[ebp-8]

1EF8A05A  |.  85C0          test eax,eax

1EF8A05C  |.  75 07         jnz short VA\_X.1EF8A065

1EF8A05E  |.  33C0          xor eax,eax

1EF8A060  |.  E9 C0000000   jmp VA\_X.1EF8A125

1EF8A065  |>  E8 EAFEFFFF   call VA\_X.1EF89F54  <---- //请看下面,解壳和检测SOFTICE和Win32debug

1EF8A06A  |.  8945 FC       mov dword ptr ss:[ebp-4],eax

1EF8A06D  |.  837D FC 01    cmp dword ptr ss:[ebp-4],1

1EF8A071  |.  75 40         jnz short VA\_X.1EF8A0B3

1EF8A073  |.  833D C466FB1E>cmp dword ptr ds:[1EFB66C4],0

1EF8A07A  |.  74 30         je short VA\_X.1EF8A0AC

1EF8A07C  |.  68 E868FB1E   push VA\_X.1EFB68E8

1EF8A081  |.  6A 01         push 1

1EF8A083  |.  8B0D 9066FB1E mov ecx,dword ptr ds:[1EFB6690]

1EF8A089  |.  51            push ecx

1EF8A08A  |.  FF15 C466FB1E call dword ptr ds:[1EFB66C4]<--------// 呼叫OEP,请看1EE4868E

1EF8A090  |.  8945 FC       mov dword ptr ss:[ebp-4],eax

1EF8A093  |.  837D FC 00    cmp dword ptr ss:[ebp-4],0

1EF8A097  |.  75 0C         jnz short VA\_X.1EF8A0A5

1EF8A099  |.  E8 5FFFFFFF   call VA\_X.1EF89FFD

1EF8A09E  |.  33C0          xor eax,eax

//////////////////////////////////////////////////////////////////////////////////////////////////

1EF89F54  /$  55            push ebp

1EF89F55  |.  8BEC          mov ebp,esp

1EF89F57  |.  81EC 10010000 sub esp,110

1EF89F5D  |.  C645 F8 00    mov byte ptr ss:[ebp-8],0

1EF89F61  |.  68 8435FB1E   push VA\_X.1EFB3584                       ; /FileName = "Kernel32"

1EF89F66  |.  FF15 6030FB1E call dword ptr ds:[<&KERNEL32.LoadLibrar>; \LoadLibraryA

1EF89F6C  |.  8945 F4       mov dword ptr ss:[ebp-C],eax

1EF89F6F  |.  837D F4 00    cmp dword ptr ss:[ebp-C],0

1EF89F73  |.  74 4C         je short VA\_X.1EF89FC1

1EF89F75  |.  68 7035FB1E   push VA\_X.1EFB3570                       ; /ProcNameOrOrdinal = "IsDebuggerPresent"

1EF89F7A  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]             ; |

1EF89F7D  |.  50            push eax                                 ; |hModule

1EF89F7E  |.  FF15 5830FB1E call dword ptr ds:[<&KERNEL32.GetProcAdd>; \GetProcAddress

1EF89F84  |.  8945 F0       mov dword ptr ss:[ebp-10],eax            ;  KERNEL32.IsDebuggerPresent

1EF89F87  |.  837D F0 00    cmp dword ptr ss:[ebp-10],0

1EF89F8B  |.  74 34         je short VA\_X.1EF89FC1

1EF89F8D  |.  FF55 F0       call dword ptr ss:[ebp-10]

1EF89F90  |.  85C0          test eax,eax

1EF89F92  |.  74 2D         je short VA\_X.1EF89FC1

1EF89F94  |.  FF15 2031FB1E call dword ptr ds:[<&KERNEL32.GetCurrent>; [GetCurrentProcessId

1EF89F9A  |.  50            push eax                                 ; /<%X>

1EF89F9B  |.  68 6835FB1E   push VA\_X.1EFB3568                       ; |format = "%X:DAF"

1EF89FA0  |.  8D8D F0FEFFFF lea ecx,dword ptr ss:[ebp-110]           ; |

1EF89FA6  |.  51            push ecx                                 ; |s

1EF89FA7  |.  FF15 5031FB1E call dword ptr ds:[<&MSVCRT.sprintf>]    ; \sprintf

1EF89FAD  |.  83C4 0C       add esp,0C

1EF89FB0  |.  8D95 F0FEFFFF lea edx,dword ptr ss:[ebp-110]

1EF89FB6  |.  52            push edx                                 ; /MutexName

1EF89FB7  |.  6A 00         push 0                                   ; |InitialOwner = FALSE

1EF89FB9  |.  6A 00         push 0                                   ; |pSecurity = NULL

1EF89FBB  |.  FF15 8830FB1E call dword ptr ds:[<&KERNEL32.CreateMute>; \CreateMutexA

1EF89FC1  |>  68 6C86F81E   push VA\_X.1EF8866C                       ; /handler = VA\_X.1EF8866C

1EF89FC6  |.  FF15 4031FB1E call dword ptr ds:[<&MSVCRT.\_set\_new\_han>; \\_set\_new\_handler

1EF89FCC  |.  83C4 04       add esp,4

1EF89FCF  |.  E8 C8E6FFFF   call VA\_X.1EF8869C        <--------------- //跟进去.看下面

1EF89FD4  |.  8945 FC       mov dword ptr ss:[ebp-4],eax

1EF89FD7  |.  6A 00         push 0                                   ; /handler = NULL

1EF89FD9  |.  FF15 4031FB1E call dword ptr ds:[<&MSVCRT.\_set\_new\_han>; \\_set\_new\_handler

1EF89FDF  |.  83C4 04       add esp,4

1EF89FE2  |.  837D FC 01    cmp dword ptr ss:[ebp-4],1

1EF89FE6  |.  75 0E         jnz short VA\_X.1EF89FF6

1EF89FE8  |.  68 A866FB1E   push VA\_X.1EFB66A8

1EF89FED  |.  FF15 CC66FB1E call dword ptr ds:[1EFB66CC]

1EF89FF3  |.  83C4 04       add esp,4

1EF89FF6  |>  8B45 FC       mov eax,dword ptr ss:[ebp-4]

1EF89FF9  |.  8BE5          mov esp,ebp

1EF89FFB  |.  5D            pop ebp

1EF89FFC  \.  C3            retn

/////////////////////////////////////////////////////////////////////////////////////

1EF8869C  /$  55            push ebp

1EF8869D  |.  8BEC          mov ebp,esp

1EF8869F  |.  83EC 5C       sub esp,5C
```
