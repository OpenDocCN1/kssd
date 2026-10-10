

# LanExplorer 3.5

> 原文：[`www.pediy.com/kssd/pediy02/forum248.htm`](https://www.pediy.com/kssd/pediy02/forum248.htm)

 LanExplorer 3.5   
http://www.intellimax.com   
  
一个很不错的网络抓包工具。它有 15 天时间限制，15 天到期之后可以输入一个密码，然后就可以再用 15 天或 50 次，密码好找，但没什么用处，   
  
因为始终会过期。安装的时候可以输入一个序列号，这个序列号会放在注册表中一个名叫 SerialNumber 的键中，不过序列号似乎没什么用处（   
  
只是推测，未经证实）。我的目的是去除其 15 天的限制。其目录下有两个可执行程序：probe.exe，inprobe.exe。probe.exe 就是普通的 EXE，   
  
用 W32Dasm 反汇编 inprobe.exe 之后能发现如下的程序段读取 SerialNumber，不过根据程序流程分析，它读了注册码之后一定会判断时间，因此   
  
推测注册码可能没什么用。   
  
\* Possible StringData Ref from Data Obj ->"SOFTWARE\\Intellimax\\LanExplorer\\CurrentVersion"   
                                  |   
:0044E79F 68980A4B00              push 004B0A98   
:0044E7A4 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]   
:0044E7A7 E8FEC70200              Call 0047AFAA   
:0044E7AC 8B45AC                  mov eax, dword ptr \[ebp\-54\]   
:0044E7AF C7400C02000080          mov \[eax+0C\], 80000002   
:0044E7B6 683F000F00              push 000F003F   
:0044E7BB 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]   
:0044E7BE E87D30FBFF              call 00401840   
:0044E7C3 50                      push eax   
:0044E7C4 8B4DAC                  mov ecx, dword ptr \[ebp\-54\]   
:0044E7C7 8B11                    mov edx, dword ptr \[ecx\]   
:0044E7C9 8B4DAC                  mov ecx, dword ptr \[ebp\-54\]   
:0044E7CC FF5274                  call \[edx+74\]   
:0044E7CF B819000200              mov eax, 00020019   
:0044E7D4 85C0                    test eax, eax   
:0044E7D6 7428                    je 0044E800   
:0044E7D8 8B8D24FCFFFF            mov ecx, dword ptr \[ebp+FFFFFC24\]   
:0044E7DE 81C1DC020000            add ecx, 000002DC   
:0044E7E4 51                      push ecx   
  
\* Possible StringData Ref from Data Obj ->"SerialNumber"   
                                  |   
:0044E7E5 68C80A4B00              push 004B0AC8   
:0044E7EA 8B55AC                  mov edx, dword ptr \[ebp\-54\]   
:0044E7ED 8B02                    mov eax, dword ptr \[edx\]   
:0044E7EF 8B4DAC                  mov ecx, dword ptr \[ebp\-54\]   
:0044E7F2 FF505C                  call \[eax+5C\]   
  
首先修改机器时间使其过期，然后再运行它，则显示出一个对话框，要求输入 password 以便延长试用期限。在 W32Dasm 中查找“expired”，共   
  
找到两处，如下：   
  
第一处：这里它试图删文件，但是已加载到内存中的 DLL 对应的磁盘文件无法删除。   
  
:0044E918 B960944B00              mov ecx, 004B9460   
:0044E91D E81E2FFBFF              call 00401840   
:0044E922 3BF0                    cmp esi, eax   
:0044E924 7C18                    jl 0044E93E    //此处 NOP 掉   
:0044E926 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]   
:0044E929 E8122FFBFF              call 00401840   
:0044E92E 8BF0                    mov esi, eax   
:0044E930 B95C944B00              mov ecx, 004B945C   
:0044E935 E8062FFBFF              call 00401840   
:0044E93A 3BF0                    cmp esi, eax   
:0044E93C 7E23                    jle 0044E961 //此处改为 JMP   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044E924(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"indis.dll"   
                                  |   
:0044E93E 68D80A4B00              push 004B0AD8   
  
\* Reference To: MSVCRT.remove, Ord:02A8h   
                                  |   
:0044E943 FF15345D4800            Call dword ptr \[00485D34\]  //试图删文件   
:0044E949 83C404                  add esp, 00000004   
  
\* Possible Reference to Dialog: DialogID\_00AC, CONTROL\_ID:00FF, ""   
                                  |   
:0044E94C 6AFF                    push FFFFFFFF   
:0044E94E 6A10                    push 00000010   
  
        "This trial version of LanExplorer has expired. Please contac"   
                                  |   
:0044E950 688EF60000              push 0000F68E   
  
\* Reference To: MFC42.Ordinal:04AF, Ord:04AFh   
                                  |   
:0044E955 E804C70200              Call 0047B05E   
  
第二处：从下面的代码中可以看出它把时间在 INI 文件中放了一份。   
  
\* Possible StringData Ref from Data Obj ->"IntactValue"   
                                  |   
:0041592B 68C4E84A00              push 004AE8C4   
  
\* Possible StringData Ref from Data Obj ->"Intact"   
                                  |   
:00415930 68D0E84A00              push 004AE8D0   
  
\* Reference To: KERNEL32.GetPrivateProfileIntA, Ord:0134h   
                                  |   
:00415935 FF1588534800            Call dword ptr \[00485388\]   
:0041593B 898584FEFFFF            mov dword ptr \[ebp+FFFFFE84\], eax   
:00415941 8D8DDCFEFFFF            lea ecx, dword ptr \[ebp+FFFFFEDC\]   
:00415947 E8F4BEFEFF              call 00401840   
:0041594C 50                      push eax   
  
\* Reference To: MSVCRT.remove, Ord:02A8h   
                                  |   
:0041594D FF15345D4800            Call dword ptr \[00485D34\]   
:00415953 83C404                  add esp, 00000004   
:00415956 8B95D4FEFFFF            mov edx, dword ptr \[ebp+FFFFFED4\]   
:0041595C 899574FEFFFF            mov dword ptr \[ebp+FFFFFE74\], edx   
:00415962 8B8574FEFFFF            mov eax, dword ptr \[ebp+FFFFFE74\]   
:00415968 83E801                  sub eax, 00000001   
:0041596B 898574FEFFFF            mov dword ptr \[ebp+FFFFFE74\], eax   
:00415971 83BD74FEFFFF05          cmp dword ptr \[ebp+FFFFFE74\], 00000005   
:00415978 0F87C2000000            ja 00415A40   
:0041597E 8B8D74FEFFFF            mov ecx, dword ptr \[ebp+FFFFFE74\]   
:00415984 FF248DA35A4100          jmp dword ptr \[4\*ecx+00415AA3\]  //跳转表   
:0041598B 8B95D0FEFFFF            mov edx, dword ptr \[ebp+FFFFFED0\]   
:00415991 52                      push edx   
:00415992 8B857CFEFFFF            mov eax, dword ptr \[ebp+FFFFFE7C\]   
:00415998 50                      push eax   
:00415999 E852BF0100              call 004318F0   
:0041599E 83C408                  add esp, 00000008   
:004159A1 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
:004159A7 E994000000              jmp 00415A40   
:004159AC 8B8DD0FEFFFF            mov ecx, dword ptr \[ebp+FFFFFED0\]   
:004159B2 51                      push ecx   
:004159B3 8B957CFEFFFF            mov edx, dword ptr \[ebp+FFFFFE7C\]   
:004159B9 52                      push edx   
:004159BA E88DBF0100              call 0043194C   
:004159BF 83C408                  add esp, 00000008   
:004159C2 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
:004159C8 EB76                    jmp 00415A40   
:004159CA 8B85D0FEFFFF            mov eax, dword ptr \[ebp+FFFFFED0\]   
:004159D0 50                      push eax   
:004159D1 8B8D7CFEFFFF            mov ecx, dword ptr \[ebp+FFFFFE7C\]   
:004159D7 51                      push ecx   
:004159D8 E8C8BF0100              call 004319A5   
:004159DD 83C408                  add esp, 00000008   
:004159E0 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
:004159E6 EB58                    jmp 00415A40   
:004159E8 8B95D0FEFFFF            mov edx, dword ptr \[ebp+FFFFFED0\]   
:004159EE 52                      push edx   
:004159EF 8B857CFEFFFF            mov eax, dword ptr \[ebp+FFFFFE7C\]   
:004159F5 50                      push eax   
:004159F6 E806C00100              call 00431A01   
:004159FB 83C408                  add esp, 00000008   
:004159FE 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
:00415A04 EB3A                    jmp 00415A40   
:00415A06 8B8DD0FEFFFF            mov ecx, dword ptr \[ebp+FFFFFED0\]   
:00415A0C 51                      push ecx   
:00415A0D 8B957CFEFFFF            mov edx, dword ptr \[ebp+FFFFFE7C\]   
:00415A13 52                      push edx   
:00415A14 E844C00100              call 00431A5D   
:00415A19 83C408                  add esp, 00000008   
:00415A1C 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
:00415A22 EB1C                    jmp 00415A40   
:00415A24 8B85D0FEFFFF            mov eax, dword ptr \[ebp+FFFFFED0\]   
:00415A2A 50                      push eax   
:00415A2B 8B8D7CFEFFFF            mov ecx, dword ptr \[ebp+FFFFFE7C\]   
:00415A31 51                      push ecx   
:00415A32 E885C00100              call 00431ABC   
:00415A37 83C408                  add esp, 00000008   
:00415A3A 898580FEFFFF            mov dword ptr \[ebp+FFFFFE80\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00415978(C), :004159A7(U), :004159C8(U), :004159E6(U), :00415A04(U)   
|:00415A22(U)   
|   
:00415A40 8B9584FEFFFF            mov edx, dword ptr \[ebp+FFFFFE84\]   
:00415A46 3B9580FEFFFF            cmp edx, dword ptr \[ebp+FFFFFE80\]   
:00415A4C 7419                    je 00415A67   
:00415A4E 83BD84FEFFFF00          cmp dword ptr \[ebp+FFFFFE84\], 00000000   
:00415A55 740E                    je 00415A65  //改为 JMP   
  
\* Possible Reference to Dialog: DialogID\_00AC, CONTROL\_ID:00FF, ""   
                                  |   
:00415A57 6AFF                    push FFFFFFFF   
:00415A59 6A10                    push 00000010   
  
"This trial version of LanExplorer has expired.Please contact"   
                                  |   
:00415A5B 688EF60000              push 0000F68E   
  
\* Reference To: MFC42.Ordinal:04AF, Ord:04AFh   
                                  |   
:00415A60 E8F9550600              Call 0047B05E   
  
将上述几处修改之后，再运行 probe.exe，发现还是会弹出对话框要求输密码。用 bpx GetLocalTime 或 bpx DialogBoxIndirectParamA 设断点，   
  
中断之后发现处在 inprobe.exe 的代码段中，而 inprobe.exe 是真正的 EXE（用 IDA 可以正确地反汇编），并非 DLL，所以它肯定是用   
  
CreateProcessA、WinExec（过时的，实际是调用 CreateProcessA）、ShellExecute 之类的函数启动 inprobe.exe 的。用这几个函数作断点，发   
  
现在下面的地方它启动 inprobe.exe，并把时间作为命令行参数传给它，而后用 WaitForSingleObject 阻塞自己，等待 inprobe.exe 退出，再取其   
  
退出值进行判断。   
  
\* Referenced by a CALL at Address:   
|:0044EABD    
|   
:004157C4 55                      push ebp   
:004157C5 8BEC                    mov ebp, esp   
  
\* Possible Reference to Dialog: DialogID\_00AC, CONTROL\_ID:00FF, ""   
                                  |   
:004157C7 6AFF                    push FFFFFFFF   
:004157C9 688CF24700              push 0047F28C   
:004157CE 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:004157D4 50                      push eax   
:004157D5 64892500000000          mov dword ptr fs:\[00000000\], esp   
:004157DC 81EC80010000            sub esp, 00000180   
:004157E2 8B4508                  mov eax, dword ptr \[ebp+08\]   
:004157E5 50                      push eax   
:004157E6 8D8DDCFEFFFF            lea ecx, dword ptr \[ebp+FFFFFEDC\]   
  
\* Reference To: MFC42.Ordinal:0219, Ord:0219h   
                                  |   
:004157EC E8B1590600              Call 0047B1A2   
:004157F1 C745FC00000000          mov \[ebp\-04\], 00000000   
  
\* Reference To: MSVCRT.rand, Ord:02A6h   
                                  |   
:004157F8 FF152C5D4800            Call dword ptr \[00485D2C\]   
:004157FE 99                      cdq   
:004157FF B906000000              mov ecx, 00000006   
:00415804 F7F9                    idiv ecx   
:00415806 83C201                  add edx, 00000001   
:00415809 8995D4FEFFFF            mov dword ptr \[ebp+FFFFFED4\], edx   
  
\* Reference To: MSVCRT.rand, Ord:02A6h   
                                  |   
:0041580F FF152C5D4800            Call dword ptr \[00485D2C\]   
:00415815 89857CFEFFFF            mov dword ptr \[ebp+FFFFFE7C\], eax   
  
\* Reference To: MSVCRT.rand, Ord:02A6h   
                                  |   
:0041581B FF152C5D4800            Call dword ptr \[00485D2C\]   
:00415821 8985D0FEFFFF            mov dword ptr \[ebp+FFFFFED0\], eax   
:00415827 C78584FEFFFF00000000    mov dword ptr \[ebp+FFFFFE84\], 00000000   
:00415831 C78580FEFFFF00000000    mov dword ptr \[ebp+FFFFFE80\], 00000000   
:0041583B C785D8FEFFFF00000000    mov dword ptr \[ebp+FFFFFED8\], 00000000   
  
\* Possible StringData Ref from Data Obj ->"intact.ini"   
                                  |   
:00415845 689CE84A00              push 004AE89C   
:0041584A 8D8DDCFEFFFF            lea ecx, dword ptr \[ebp+FFFFFEDC\]   
  
\* Reference To: MFC42.Ordinal:03AD, Ord:03ADh   
                                  |   
:00415850 E835590600              Call 0047B18A   
:00415855 8B95D0FEFFFF            mov edx, dword ptr \[ebp+FFFFFED0\]   
:0041585B 52                      push edx   
:0041585C 8B857CFEFFFF            mov eax, dword ptr \[ebp+FFFFFE7C\]   
:00415862 50                      push eax   
:00415863 8B8DD4FEFFFF            mov ecx, dword ptr \[ebp+FFFFFED4\]   
:00415869 51                      push ecx   
  
\* Possible StringData Ref from Data Obj ->"InProbe.exe -f%d -s%d -e%d"   
                                  |   
:0041586A 68A8E84A00              push 004AE8A8   
:0041586F 8D95E0FEFFFF            lea edx, dword ptr \[ebp+FFFFFEE0\]   
:00415875 52                      push edx   
  
\* Reference To: MSVCRT.sprintf, Ord:02B2h   
                                  |   
:00415876 FF15705D4800            Call dword ptr \[00485D70\]   
:0041587C 83C414                  add esp, 00000014   
:0041587F 6A44                    push 00000044   
:00415881 6A00                    push 00000000   
:00415883 8D858CFEFFFF            lea eax, dword ptr \[ebp+FFFFFE8C\]   
:00415889 50                      push eax   
  
\* Reference To: MSVCRT.memset, Ord:0299h   
                                  |   
:0041588A E87B630600              Call 0047BC0A   
:0041588F 83C40C                  add esp, 0000000C   
:00415892 C7858CFEFFFF44000000    mov dword ptr \[ebp+FFFFFE8C\], 00000044   
:0041589C C785B8FEFFFF01000000    mov dword ptr \[ebp+FFFFFEB8\], 00000001   
:004158A6 66C785BCFEFFFF0500      mov word ptr \[ebp+FFFFFEBC\], 0005   
:004158AF 8D4DE4                  lea ecx, dword ptr \[ebp\-1C\]   
:004158B2 51                      push ecx   
:004158B3 8D958CFEFFFF            lea edx, dword ptr \[ebp+FFFFFE8C\]   
:004158B9 52                      push edx   
:004158BA 6A00                    push 00000000   
:004158BC 6A00                    push 00000000   
  
\* Possible Reference to Dialog: DialogID\_0080   
                                  |   
:004158BE 6880000000              push 00000080   
:004158C3 6A00                    push 00000000   
:004158C5 6A00                    push 00000000   
:004158C7 6A00                    push 00000000   
:004158C9 8D85E0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEE0\]   
:004158CF 50                      push eax   
:004158D0 6A00                    push 00000000   
  
\* Reference To: KERNEL32.CreateProcessA, Ord:0044h   
                                  |   
:004158D2 FF15B8534800            Call dword ptr \[004853B8\]   
  
\* Possible Reference to Dialog: DialogID\_07D0   
                                  |   
:004158D8 68D0070000              push 000007D0   
:004158DD 8B4DE4                  mov ecx, dword ptr \[ebp\-1C\]   
:004158E0 51                      push ecx   
  
\* Reference To: KERNEL32.WaitForSingleObject, Ord:02CEh   
                                  |   
:004158E1 FF15A0534800            Call dword ptr \[004853A0\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00415911(U)   
|   
:004158E7 8D9588FEFFFF            lea edx, dword ptr \[ebp+FFFFFE88\]   
:004158ED 52                      push edx   
:004158EE 8B45E4                  mov eax, dword ptr \[ebp\-1C\]   
:004158F1 50                      push eax   
  
\* Reference To: KERNEL32.GetExitCodeProcess, Ord:010Bh   
                                  |   
:004158F2 FF15B4534800            Call dword ptr \[004853B4\]   
:004158F8 85C0                    test eax, eax   
:004158FA 7417                    je 00415913   
:004158FC 81BD88FEFFFF03010000    cmp dword ptr \[ebp+FFFFFE88\], 00000103   
:00415906 7507                    jne 0041590F   
:00415908 E8490C0400              call 00456556   
:0041590D EB02                    jmp 00415911   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00415906(C)   
|   
:0041590F EB02                    jmp 00415913   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0041590D(U)   
|   
:00415911 EBD4                    jmp 004158E7   
  
注意到上面这个函数是在 probe.exe 的 CS:0044EABD 处进行调用的，如下。很显然，只要把 CS:0044EABC 处的指令改为 JMP 0044EAD0 就可以将   
  
inprobe.exe 整个 bypass 掉，也就可以把 inprobe.exe 删了。   
  
:0044EABC 50                      push eax   
:0044EABD E8026DFCFF              call 004157C4  //加载 inprobe.exe   
:0044EAC2 83C404                  add esp, 00000004   
:0044EAC5 85C0                    test eax, eax   
:0044EAC7 7507                    jne 0044EAD0  //jump if good guy   
:0044EAC9 33C0                    xor eax, eax  //bad guy, exit   
:0044EACB E916160000              jmp 004500E6   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044EAC7(C)   
|   
:0044EAD0 8B8D24FCFFFF            mov ecx, dword ptr \[ebp+FFFFFC24\]  

> 很显然，只要把 CS:0044EABC 处的指令改为 JMP 0044EAD0 就可以将   
>   
> inprobe.exe 整个 bypass 掉，也就可以把 inprobe.exe 删了。   
>   
> :0044EABC 50                      push eax   
> :0044EABD E8026DFCFF              call 004157C4  //加载 inprobe.exe   
> :0044EAC2 83C404                  add esp, 00000004   
> :0044EAC5 85C0                    test eax, eax   
> :0044EAC7 7507                    jne 0044EAD0  //jump if good guy   
> :0044EAC9 33C0                    xor eax, eax  //bad guy, exit   
> :0044EACB E916160000              jmp 004500E6   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:0044EAC7(C)   
> |   
> :0044EAD0 8B8D24FCFFFF            mov ecx, dword ptr \[ebp+FFFFFC24\]   
>   
> 这个程序启动的时候还有一个 splash screen，比较讨厌。用 bpx CreateWindowExA 设断点，会发现 OT60as.dll 中的一个函数负责显示 splash   
>   
> screen，在 probe.exe 中的如下地方被调用。跳过下面的这个 call 即可，不会造成资源泄漏，修改的时候注意保持堆栈平衡即可。   
>   
> :0044EA62 6A00                    push 00000000   
> :0044EA64 6800000090              push 90000000   
> :0044EA69 6A00                    push 00000000   
> :0044EA6B 8B45C4                  mov eax, dword ptr \[ebp\-3C\]   
> :0044EA6E 8B10                    mov edx, dword ptr \[eax\]   
> :0044EA70 8B4DC4                  mov ecx, dword ptr \[ebp\-3C\]   
> :0044EA73 FF92C0000000            call dword ptr \[edx+000000C0\]   
>   
> dr0, 2000/08/17 

* * *

 4dd466d0e1e0cc123afbe504da2462ca