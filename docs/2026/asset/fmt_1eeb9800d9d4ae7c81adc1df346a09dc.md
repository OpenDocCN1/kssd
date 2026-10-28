
此关键函数`00021D2A`处调用`CALL 00022090`是通过调用`RtlTimeToSecondsSince1970`、`kernel32.GetSystemTimeAsFileTime(pFiletime)`对文件属性进行修改，实现隐藏；函数`00021DEC`处的调用`CALL 00021AE0`是获取批处理进程所用的批处理命令：`ASCII ":l`

`if not exist "C:\Users\h\Desktop\C14BC5~1" goto edel` `/Q /F "C:\Users\h\Desktop\C14BC5~1"` `goto l:edel` `/Q /F "C:\Users\h\AppData\Roaming\4336348.bat""`；之后，在`00021E7E`处调用`CALL DWORD PTR DS:[2303C]`即`BOOL kernel32.CreateProcessW`启动`CMD`进程，函数参数为：

```
|CPU Stack |ApplicationName = NULL ;
|CommandLine = ""C:\Windows\system32\cmd.exe" /c "C:\Users\h\AppData\Roaming\4336348.bat"" ;
|pProcessSecurity = NULL ;
|pThreadSecurity = NULL ;
|InheritHandles = FALSE ;
|CreationFlags = CREATE\_NO\_WINDOW;
|pEnvironment = NULL ;
|CurrentDirectory = NULL;
|pStartupInfo = 0017F6E4 -> STARTUPINFOW {Size=68., Reserved1=NULL, Desktop=NULL, Title=NULL, X=0, Y=0, Width=0, Height=0, XCountChars=0, YCountChars=0, FillAttribute=0, Flags=STARTF\_USESHOWWINDOW, ShowWindow=SW\_HIDE, Reserved2=0, Reserved3=NULL, hStdInpu ;
\pProcessInformation = 0017F6D4 -> PROCESS\_INFORMATION {hProcess=00650064, hThread=0074006E, ProcessID=660069 (6684777.), ThreadID=7F}
```

至此，木马`Vadchdhv`完成了第一步；接下来会启动一个循环进行系统进程遍历搜索寻找`“explorer.exe”`进程，跟踪如下：

```
CPU Disasm
地址 十六进制转储 命令 注释
0002190F |. 8BF7 MOV ESI,EDI
00021911 |? 74 09 JE SHORT 0002191C
00021913 |? 83C6 02 ADD ESI,2
00021915 |. 0266 83 ADD AH,BYTE PTR DS:[ESI-7D]
00021918 |? 3E:0075 F7 ADD BYTE PTR DS:[EBP-9],DH
0002191C |. 68 24390200 PUSH 00023924
00021921 |? 68 34390200 PUSH 00023934
00021926 |? FF15 70300200 CALL DWORD PTR DS:[23070] ；UNICODE "csrss.exe"
0002192C |? 50 PUSH EAX
0002192D |. FF15 6C300200 CALL DWORD PTR DS:[2306C] ；KERNELBASE.GetProcAddress
00021933 |? 85C0 TEST EAX,EAX
00021935 |? 75 0E JNE SHORT 00021945
00021937 |? 33C9 XOR ECX,ECX
00021939 |? 3D 44285E09 CMP EAX,95E2844
0002193E |? 0F94C1 SETE CL
00021941 |? 5E POP ESI
00021942 |? 8BC1 MOV EAX,ECX
00021944 |? C3 RETN
00021945 |? 2BF7 SUB ESI,EDI ；UNICODE "csrss.exe"
00021947 |? D1FE SAR ESI,1
00021949 |.^ 03F6 ADD ESI,ESI
0002194B |? 56 PUSH ESI
0002194C |? 57 PUSH EDI ；UNICODE "wininit.exe"; UNICODE "services.exe" ; UNICODE "lsass.exe"
0002194D |? 6A 00 PUSH 0
0002194F |? FFD0 CALL EAX ；ntdll.RtlComputeCrc32
00021951 |? 33C9 XOR ECX,ECX
00021953 |? 3D 44285E09 CMP EAX,95E2844
00021958 |> 0F94C1 SETE CL
0002195B |? 5E POP ESI
0002195C |? 8BC1 MOV EAX,ECX
0002195E |? C3 RETN
```

找到`explorer.exe`后会进行进程注入，关于进程注入和后续部分，改天再补充。

### 目的：
分析此木马运行机制及内部实现原理，剖析此木马会带来哪些危害。

### 调式环境及工具：
`VMware+Windows7`（推荐）、`DIE`、`IDA`、`OD`

### 1. 脱壳拿shellcode，为拿木马主体样本做准备

`IDA`打开原始样本后，定位在`sub_402FBA`模块处，注意`OD`动态调试每次基址会有变化，相对地址是不变的，此模块核心功能就是在内存释放`Shellcode`的木马`Vadchdhv`主体；如下`0040304C`处调用`VirtualAlloc`申请内存，之后进行`Shellcode`第一次存放。

```
|  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `.text:``0040301F`                 `add     edi, dword_405D2C`  `.text:``00403025`                 `push    flProtect ;flProtect`  `.text:``0040302B`                 `push    flAllocationType ; flAllocationType`  `.text:``00403031`                 `lea     eax, [ebp``+``var_1C]`  `.text:``00403034`                 `push    edi`  `.text:``00403035`                 `pop     dword ptr [eax]`  `.text:``00403037`                 `mov     esi, [ebp``+``var_1C]`  `.text:``0040303A`                 `mov     off_405B1C, esi`  `.text:``00403040`                 `push    off_405B1C      ; dwSize`  `.text:``00403046`                 `push    dword ptr asc_405598``+``4` `; lpAddress`  `.text:``0040304C`                 `call    ds:VirtualAlloc` |
```

完成之后，退回到`sub_4043A9`模块处，调用`sub_401000`会再次进行`Shellcode`的内存释放。

```
|  |
| --- | --- |
| 1 | `text:``00404436`                 `call    sub_401000` |
```

完成之后，退回到`start`代码处，跟踪到此其实**我们最关心的是壳是从哪里跳转进入内存`Shellcode`主体的，这才是我们拿`Shellcode`的关键，否则木马是逮不到的；在以上的`Shellcode`释放主体的过程可能有些繁琐和枯燥，但要注意进行多次跟踪调试一下，注意观察寄存器和栈数据的变化，捕捉关键信息，对于尤其是分析暴露信息较少的木马来说也是一个不错的小技巧；**释放`Shellcode`是通过循环完成的，在退出循环后下断，进行`OD`单步跟踪`F7`;

```
|  |
| --- | --- |
| 1  2  3  4  5 | `.text:``00402F9A`                 `mov     esi, [ebp``+``var_20]`  `.text:``00402F9D`                 `mov     off_40521C, esi`  `.text:``00402FA3`                 `push    off_40521C`  `.text:``00402FA9`                 `mov     dword_405090, edx`  `.text:``00402FAF`                 `call    dword_405090` |
```

注意上述**代码中的`00402FAF`调用`call dword_405090`，这个就是从壳进入`shellcode`内存主体**了，`F7`跟入。接下来，才算是接近了木马`Vadchdhv`主体代码，但还不全是，因为`Vadchdhv`不是一次性释放进内存运行的，而是先要从`Shellcode`释放到更高权限的内存，进行`PE`文件的组装和修正，然后才木马的核心主体代码。

### 2. 脱ShellCode,拿木马Vadchdhv内存进程主体

`OD`跟入后，注意在`00260261`（地址会有变化）处调用`002601EE`取内核函数字符串，然后返回到`0026026C`，在`00260270`处调用`CALL 0026020A`进行内核函数`SHA`值比对，然后调用`00260318`取得下一个函数名，然后回到`00260261`处进行下一次循环；**此循环是完成`ntdll`内核函数的定位，获取木马所用内核函数地址，并进行木马主体地址修正。**

**接下来，木马`Vadchdhv`会调用内核函数`ntdll.NtProtectVirtualMemory`、`ntdll.NtAllocateVirtualMemory`等，完成提权并再次申请内存，准备进行进程主体释放、拼接。**

```
|  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `002605DE`    `FF55 D0         CALL DWORD PTR SS:[EBP``-``30``]             ; ntdll.NtProtectVirtualMemory`  `002605E1`    `85C0`            `TEST EAX,EAX`  `002605E3`    `0F85` `87010000`   `JNE` `00260770`  `002605E9`    `6A` `40`           `PUSH` `40`  `002605EB`    `68` `00300000`     `PUSH` `3000`  `002605F0`    `FF75` `0C`         `PUSH DWORD PTR SS:[EBP``+``0C``]`  `002605F3`    `6A` `00`           `PUSH` `0`  `002605F5`    `C745 DC` `0000000` `MOV DWORD PTR SS:[EBP``-``24``],``0`  `002605FC`    `8D45` `DC         LEA EAX,[EBP``-``24``]`  `002605FF`    `50`              `PUSH EAX`  `00260600`    `6A` `FF           PUSH` `-``1`  `00260602`    `FF55 D4         CALL DWORD PTR SS:[EBP``-``2C``]             ; ntdll.NtAllocateVirtualMemory` |
```

调用`ntdll.NtAllocateVirtualMemory`的栈信息：

```
|  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `CPU Stack`  `地址        值          ASCII 注释`  `0017FE88`  `/``FFFFFFFF  ÿÿÿÿ  ; |Arg1` `=` `-``1`  `0017FE8C`  `|``0017FED8`  `Øþ   ; |Arg2` `=` `17FED8`  `0017FE90`  `|``00000000`        `; |Arg3` `=` `0`  `0017FE94`  `|``00025C20`   `\   ; |Arg4` `=` `c14bc530f959cb8dedb0f51527173a4.``25C20`  `0017FE98`  `|``00003000`   `0`    `; |Arg5` `=` `3000`  `0017FE9C`  `|``00000040`  `@     ; \Arg6` `=` `40` |
```

接下来在新申请内存处，首先写入`PE`格式的文件头：

```
|  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `CPU Disasm`  `地址        十六进制转储          命令                 注释`  `00260602`    `FF55 D4         CALL DWORD PTR SS:[EBP``-``2C``]             ; ntdll.NtAllocateVirtualMemory`  `00260605`    `85C0`            `TEST EAX,EAX`  `00260607`    `0F85` `63010000`   `JNE` `00260770`  `0026060D`    `8B75` `DC         MOV ESI,DWORD PTR SS:[EBP``-``24``]`  `00260610`    `33C0`            `XOR EAX,EAX`  `00260612`    `8B4D` `E0         MOV ECX,DWORD PTR SS:[EBP``-``20``]`  `00260615`    `8B7D` `FC         MOV EDI,DWORD PTR SS:[EBP``-``4``]`  `00260618`    `F2:AA           REPNE STOS BYTE PTR ES:[EDI]           ; Undocumented instruction` `or` `encoding   :复制到EDI：``0002004`    `A    ； ECX ：``0001AFB6``; 包含字符串：“this program cannot be run` `in` `DOS mode.&&”` |
```

接下来，木马`Vadchdhv`从`ShellCode`中解码释放并拼接，

```
|  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `CPU Disasm`  `地址        十六进制转储            命令                                     注释`  `002606A7`    `8BF3`            `MOV ESI,EBX`  `002606A9`    `8B7D` `FC         MOV EDI,DWORD PTR SS:[EBP``-``4``]`  `002606AC`    `0372` `14`         `ADD ESI,DWORD PTR DS:[EDX``+``14``]`  `002606AF`    `037A` `0C`         `ADD EDI,DWORD PTR DS:[EDX``+``0C``]`  `002606B2`    `51`              `PUSH ECX`  `002606B3`    `8B4A` `10`         `MOV ECX,DWORD PTR DS:[EDX``+``10``]`  `002606B6`    `85C9`            `TEST ECX,ECX`  `002606B8`    `75` `07`           `JNE SHORT` `002606C1`  `002606BA`    `C742` `14` `0000000` `MOV DWORD PTR DS:[EDX``+``14``],``0`  `002606C1`    `3B4A` `08`         `CMP` `ECX,DWORD PTR DS:[EDX``+``8``]`  `002606C4`    `76` `03`           `JBE SHORT` `002606C9`  `002606C6`    `8B4A` `08`         `MOV ECX,DWORD PTR DS:[EDX``+``8``]`  `002606C9`    `F2:A4           REPNE MOVS BYTE PTR ES:[EDI],BYTE PTR  ; Undocumented instruction` `or` `encoding ：解码释放：``00021000` |
