
# PEid 插件——Generic OEP Finder 原理分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-491.htm`](https://www.pediy.com/kssd/pediy07/pediy7-491.htm)

PEid 的这个小插件用了很久了，一直觉得功能不错，准确率也挺高的，自己在写壳的过程中也曾尝试避开其检测，但一直没有成功，于是抽了些时间看看它的实现原理，这才恍然大悟。

下面是这个插件的一级输出函数：

```
10001870 ; Exported entry   1. DoMyJob

10001870

10001870 ; ************************************************* SUBROUTINE *************************************************

10001870

10001870

10001870           public DoMyJob
10001870 DoMyJob   proc near
10001870
10001870 hWnd      = dword ptr  4
10001870 arg_4     = dword ptr  8
10001870 arg_8     = dword ptr  0Ch

10001870           mov   eax, [esp+arg_8]
10001874           push  ebx
10001875           push  esi
10001876           cmp   eax, 50456944h
1000187B           push  edi
1000187C           jz    short loc_10001889
1000187E           cmp   eax, 5852445Ah
10001883           jnz   loc_10001A81
10001889
10001889 loc_10001889:                       ; ...
10001889           mov   ebx, [esp+0Ch+arg_4]
1000188D           or    ecx, 0FFFFFFFFh
10001890           mov   edi, ebx
10001892           xor   eax, eax
10001894           repne scasb
10001896           not   ecx
10001898           dec   ecx
10001899           cmp   ecx, 1
1000189C           jnb   short loc_100018BE
1000189E           mov   eax, [esp+0Ch+hWnd]
100018A2           push  40000h                 ; uType
100018A7           push  offset szError          ; lpCaption
100018AC           push  offset szNoFileSpecifie ; lpText
100018B1           push  eax                   ; hWnd
100018B2           call  ds:MessageBoxA
100018B8           pop   edi
100018B9           pop   esi
100018BA           xor   eax, eax
100018BC           pop   ebx
100018BD           retn
100018BE ; ----------------------------------------------------------------------------
100018BE
100018BE loc_100018BE:                       ; ...
100018BE           push  0                   ; hTemplateFile
100018C0           push  80h                 ; dwFlagsAndAttributes
100018C5           push  3                   ; dwCreationDisposition
100018C7           push  0                   ; lpSecurityAttributes
100018C9           push  1                   ; dwShareMode
100018CB           push  80000000h             ; dwDesiredAccess
100018D0           push  ebx                   ; lpFileName
100018D1           call  ds:CreateFileA
100018D7           cmp   eax, 0FFFFFFFFh
100018DA           mov   ds:hObject, eax
100018DF           jnz   short loc_10001908
100018E1           push  eax                   ; hObject
100018E2           call  ds:CloseHandle
100018E8           mov   ecx, [esp+0Ch+hWnd]
100018EC           push  40000h                 ; uType
100018F1           push  offset szError          ; lpCaption
100018F6           push  offset szCouldNotOpenTh ; lpText
100018FB           push  ecx                   ; hWnd
100018FC           call  ds:MessageBoxA
10001902           pop   edi
10001903           pop   esi
10001904           xor   eax, eax
10001906           pop   ebx
10001907           retn
10001908 ; ----------------------------------------------------------------------------
10001908
10001908 loc_10001908:                       ; ...
10001908           push  0                   ; lpName
1000190A           push  0                   ; dwMaximumSizeLow
1000190C           push  0                   ; dwMaximumSizeHigh
1000190E           push  2                   ; flProtect
10001910           push  0                   ; lpFileMappingAttributes
10001912           push  eax                   ; hFile
10001913           call  ds:CreateFileMappingA
10001919           push  0                   ; dwNumberOfBytesToMap
1000191B           push  0                   ; dwFileOffsetLow
1000191D           mov   edi, eax
1000191F           push  0                   ; dwFileOffsetHigh
10001921           push  4                   ; dwDesiredAccess
10001923           push  edi                   ; hFileMappingObject
10001924           call  ds:MapViewOfFile
1000192A           mov   esi, eax
1000192C           test  esi, esi
1000192E           jnz   short MapViewCreated
10001930           mov   edx, ds:hObject
10001936           mov   esi, ds:CloseHandle
1000193C           push  edx                   ; hObject
1000193D           call  esi ; CloseHandle
1000193F           push  edi                   ; hObject
10001940           call  esi ; CloseHandle
10001942           mov   eax, [esp+0Ch+hWnd]
10001946           push  40000h                 ; uType
1000194B           push  offset szError          ; lpCaption
10001950           push  offset szMappingError_ ; lpText
10001955           push  eax                   ; hWnd
10001956           call  ds:MessageBoxA
1000195C           pop   edi
1000195D           pop   esi
1000195E           xor   eax, eax
10001960           pop   ebx
10001961           retn
10001962 ; ----------------------------------------------------------------------------
10001962
10001962 MapViewCreated:                      ; ...
10001962           mov   ds:lpFileHeader, esi
10001968           cmp   word ptr [esi], 5A4Dh   ; 是否为可执行文件
1000196D           jz    short IsExeFile
1000196F           mov   ecx, ds:hObject
10001975           mov   esi, ds:CloseHandle
1000197B           push  ecx                   ; hObject
1000197C           call  esi ; CloseHandle
1000197E           push  edi                   ; hObject
1000197F           call  esi ; CloseHandle
10001981           mov   edx, [esp+0Ch+hWnd]
10001985           push  40000h                 ; uType
1000198A           push  offset szError          ; lpCaption
1000198F           push  offset szNotADosExecuta ; lpText
10001994           push  edx                   ; hWnd
10001995           call  ds:MessageBoxA
1000199B           pop   edi
1000199C           pop   esi
1000199D           xor   eax, eax
1000199F           pop   ebx
100019A0           retn
100019A1 ; ----------------------------------------------------------------------------
100019A1
100019A1 IsExeFile:                          ; ...
100019A1           mov   eax, [esi+3Ch]
100019A4           push  4                   ; ucb
100019A6           add   eax, esi
100019A8           push  eax                   ; lp
100019A9           mov   ds:lpPEHeader, eax
100019AE           call  ds:IsBadReadPtr
100019B4           test  eax, eax
100019B6           jnz   ReadMemError
100019BC           mov   eax, ds:lpPEHeader
100019C1           cmp   dword ptr [eax], 4550h  ; 是否为 PE 文件
100019C7           jnz   ReadMemError
100019CD           lea   ecx, [eax+0F8h]
100019D3           push  ebp
100019D4           mov   ebp, [esp+10h+hWnd]
100019D8           mov   ds:lpScnNameInfo, ecx
100019DE           mov   ecx, [eax+50h]         ; SizeOfImage
100019E1           mov   edx, [eax+34h]         ; ImageBase
100019E4           push  ebx
100019E5           push  ecx
100019E6           push  ebp
100019E7           mov   ds:BaseAddress, edx
100019ED           call  GetOEP     ---------->这个函数是关键

{
  10001610 ; ************************************************ SUBROUTINE ************************************************
  10001610
  10001610
  10001610 GetOEP    proc near                    ; ...
  10001610
```



```
  10001610 hProcess  = dword ptr -54h  
  10001610 StartupInfo= _STARTUPINFOA ptr -44h  
  10001610 hWnd      = dword ptr  4  
  10001610 dwBytes   = dword ptr  8  
  10001610 NumberOfBytesRead= dword ptr  0Ch  
  10001610  

  10001610           sub   esp, 54h  
  10001613           push  esi  
  10001614           push  edi  
  10001615           mov   ecx, 11h  
  1000161A           xor   eax, eax  
  1000161C           lea   edi, [esp+5Ch+StartupInfo]  
  10001620           lea   edx, [esp+5Ch+StartupInfo]  
  10001624           rep stosd  
  10001626           mov   [esp+5Ch+hProcess], eax  
  1000162A           lea   ecx, [esp+5Ch+hProcess]  
  1000162E           mov   [esp+0Ch], eax  
  10001632           push  ecx                   ; lpProcessInformation  
  10001633           mov   [esp+14h], eax  
  10001637           push  edx                   ; lpStartupInfo  
  10001638           push  eax                   ; lpCurrentDirectory  
  10001639           push  eax                   ; lpEnvironment  
  1000163A           push  20h                   ; dwCreationFlags  
  1000163C           push  eax                   ; bInheritHandles  
  1000163D           push  eax                   ; lpThreadAttributes  
  1000163E           push  eax                   ; lpProcessAttributes  
  1000163F           mov   [esp+34h], eax  
  10001643           push  eax                   ; lpCommandLine  
  10001644           mov   eax, [esp+80h+NumberOfBytesRead]  
  1000164B           mov   [esp+80h+StartupInfo.cb], 44h  
  10001653           push  eax                   ; lpApplicationName  
  10001654           call  ds:CreateProcessA       ; 启动待检测进程  

            注意这里并没有设调试标志，GenOEP 是用普通方式启动进程的，  

            所以调试器检测型的反跟踪对 PEid 无效，但是父进程检测还是有效的  

  1000165A           test  eax, eax  
  1000165C           jnz   short ProcessCreated  
  1000165E           mov   ecx, [esp+5Ch+hWnd]  
  10001662           push  40000h                ; uType  
  10001667           push  offset szError        ; lpCaption  
  1000166C           push  offset szErrorWhileCrea ; lpText  
  10001671           push  ecx                   ; hWnd  
  10001672           call  ds:MessageBoxA  
  10001678  

  10001678 ExitGetOEP:                    ; ...  
  10001678           pop   edi  
  10001679           xor   eax, eax  
  1000167B           pop   esi  
  1000167C           add   esp, 54h  
  1000167F           retn  
  10001680 ; ----------------------------------------------------------------------------  
  10001680  

  10001680 ProcessCreated:                 ; ...  
  10001680           mov   edx, [esp+5Ch+hProcess]  
  10001684           push  0FFFFFFFFh        ; dwMilliseconds  
  10001686           push  edx             ; hProcess  
  10001687           call  ds:WaitForInputIdle ; 当待检测程序进入空闲  
  10001687                                ; 状态时再进行内存扫描  
  1000168D           mov   eax, [esp+0Ch]  
  10001691           push  eax             ; hThread  
  10001692           call  ds:SuspendThread ; 挂起待检测程序，避免  
  10001692                                ; 扫描过程中代码被修改  
  10001698           mov   edi, [esp+5Ch+dwBytes] ; SizeOfImage  
  1000169C           push  edi             ; dwBytes  
  1000169D           push  0               ; uFlags  
  1000169F           call  ds:GlobalAlloc   ；分配内存准备读取文件映象  
  100016A5           mov   edx, ds:BaseAddress  
  100016AB           lea   ecx, [esp+5Ch+NumberOfBytesRead]  
  100016AF           push  ecx             ; lpNumberOfBytesRead  
  100016B0           push  edi             ; nSize  
  100016B1           mov   ds:hMem, eax  
  100016B6           push  eax             ; lpBuffer  
  100016B7           mov   eax, [esp+68h+hProcess]  
  100016BB           push  edx             ; lpBaseAddress  
  100016BC           push  eax             ; hProcess  
  100016BD           call  ds:ReadProcessMemory    ; 读取整个映象到内存  
  100016C3           test  eax, eax  
  100016C5           jnz   short ErrorMemAccess  
  100016C7           mov   ecx, [esp+5Ch+hWnd]  
  100016CB           push  40000h                ; uType  
  100016D0           push  offset szError        ; lpCaption  
  100016D5           push  offset szErrorWhileRead ; lpText  
  100016DA           push  ecx                   ; hWnd  
  100016DB           call  ds:MessageBoxA  
  100016E1           pop   edi  
  100016E2           xor   eax, eax  
  100016E4           pop   esi  
  100016E5           add   esp, 54h  
  100016E8           retn  
  100016E9 ; ----------------------------------------------------------------------------  
  100016E9  

  100016E9 ErrorMemAccess:                 ; ...  
  100016E9           mov   edx, [esp+5Ch+hProcess]  
  100016ED           push  0               ; uExitCode  
  100016EF           push  edx             ; hProcess  
  100016F0           call  ds:TerminateProcess ; 可以结束被调试进程了  
  100016F6           push  1  
  100016F8           push  28h           ; 特征码长度  
  100016FA           push  offset FeatureCode_1  
  100016FF           push  edi             ; 映象大小  
  10001700           call  ScanForFeatureCode  
  10001705           mov   esi, eax  
  10001707           add   esp, 10h  
  1000170A           test  esi, esi  
  1000170C           jnz   FeatureCodeFound  
  10001712           push  9  
  10001714           push  24h  
  10001716           push  offset FeatureCode_2  
  1000171B           push  edi  
  1000171C           call  ScanForFeatureCode  
  10001721           mov   esi, eax  
  10001723           add   esp, 10h  
  10001726           test  esi, esi  
  10001728           jnz   FeatureCodeFound  
  1000172E           push  8  
  10001730           push  20h  
  10001732           push  offset FeatureCode_3  
  10001737           push  edi  
  10001738           call  ScanForFeatureCode  
  1000173D           mov   esi, eax  
  1000173F           add   esp, 10h  
  10001742           test  esi, esi  
  10001744           jnz   FeatureCodeFound  
  1000174A           push  4  
  1000174C           push  20h  
  1000174E           push  offset FeatureCode_4  
  10001753           push  edi  
  10001754           call  ScanForFeatureCode  
  10001759           mov   esi, eax  
  1000175B           add   esp, 10h  
  1000175E           test  esi, esi  
  10001760           jnz   FeatureCodeFound  
  10001766           push  5  
  10001768           push  28h  
  1000176A           push  offset FeatureCode_5  
  1000176F           push  edi  
  10001770           call  ScanForFeatureCode  
  10001775           mov   esi, eax  
  10001777           add   esp, 10h  
  1000177A           test  esi, esi  
  1000177C           jnz   FeatureCodeFound  
  10001782           push  5  
  10001784           push  20h  
  10001786           push  offset FeatureCode_6  
  1000178B           push  edi  
  1000178C           call  ScanForFeatureCode  
  10001791           mov   esi, eax  
  10001793           add   esp, 10h  
  10001796           test  esi, esi  
  10001798           jnz   FeatureCodeFound  
  1000179E           push  3  
  100017A0           push  1Bh  
  100017A2           push  offset FeatureCode_7  
  100017A7           push  edi  
  100017A8           call  ScanForFeatureCode  
  100017AD           mov   esi, eax  
  100017AF           add   esp, 10h  
  100017B2           test  esi, esi  
  100017B4           jnz   FeatureCodeFound  
  100017BA           push  0Ah  
  100017BC           push  13h  
  100017BE           push  offset FeatureCode_8  
  100017C3           push  edi  
```



```asm
100017C4             call  ScanForFeatureCode
100017C9             mov   esi, eax
100017CB             add   esp, 10h
100017CE             test  esi, esi
100017D0             jnz   short FeatureCodeFound
100017D2             push  7
100017D4             push  1Eh
100017D6             push  offset FeatureCode_9
100017DB             push  edi
100017DC             call  ScanForFeatureCode
100017E1             mov   esi, eax
100017E3             add   esp, 10h
100017E6             test  esi, esi
100017E8             jnz   short FeatureCodeFound
100017EA             push  2
100017EC             push  1Ch
100017EE             push  offset FeatureCode_10
100017F3             push  edi
100017F4             call  ScanForFeatureCode
100017F9             mov   esi, eax
100017FB             add   esp, 10h
100017FE             test  esi, esi
10001800             jnz   short FeatureCodeFound
10001802             push  6
10001804             push  5
10001806             push  offset FeatureCode_11
1000180B             push  edi
1000180C             call  ScanForFeatureCode
10001811             mov   esi, eax
10001813             add   esp, 10h
10001816             test  esi, esi
10001818             jnz   short FeatureCodeFound
1000181A             push  0Bh
1000181C             push  8
1000181E             push  offset FeatureCode_12
10001823             push  edi
10001824             call  ScanForFeatureCode
10001829             mov   esi, eax
1000182B             add   esp, 10h
1000182E             test  esi, esi
10001830             jnz   short FeatureCodeFound
10001832             push  9
10001834             push  8
10001836             push  offset FeatureCode_13
1000183B             push  edi
1000183C             call  ScanForFeatureCode
10001841             add   esp, 10h
10001844             mov   esi, eax
10001846
10001846 FeatureCodeFound:                     ; ...
10001846             mov   eax, ds:hMem
1000184B             push  eax                 ; hMem
1000184C             call  ds:GlobalFree
10001852             test  esi, esi
10001854             jz    ExitGetOEP
1000185A             mov   ecx, ds:BaseAddress
10001860             pop   edi
10001861             lea   eax, [ecx+esi]
10001864             pop   esi
10001865             add   esp, 54h
10001868             retn
10001868 GetOEP    endp
10001868

10001868 ; ----------------------------------------------------------------------------
}

100019F2             add   esp, 0Ch
100019F5             mov   ebx, eax
100019F7             push  esi                 ; lpBaseAddress
100019F8             call  ds:UnmapViewOfFile
100019FE             mov   eax, ds:hObject
10001A03             mov   esi, ds:CloseHandle
10001A09             push  eax                 ; hObject
10001A0A             call  esi ; CloseHandle
10001A0C             push  edi                 ; hObject
10001A0D             call  esi ; CloseHandle
10001A0F             test  ebx, ebx
10001A11             mov   eax, offset szFoundOep08lx ; "Found OEP: %08lX"
10001A16             jnz   short OEPFound
10001A18             mov   eax, offset szNoOepFound ; "No OEP found"

            你也有找不到的时候。。。

            看看怎样才能欺骗它:P
10001A1D

10001A1D OEPFound:                             ; ...
10001A1D             push  ebx
10001A1E             push  eax                 ; LPCSTR
10001A1F             push  offset Text         ; LPSTR
10001A24             call  ds:wsprintfA
10001A2A             mov   eax, [esp+1Ch+arg_8]
10001A2E             add   esp, 0Ch
10001A31             cmp   eax, 50456944h
10001A36             jnz   short loc_10001A4E
10001A38             push  40000h              ; uType
10001A3D             push  offset aGenoep      ; lpCaption
10001A42             push  offset Text         ; lpText
10001A47             push  ebp                 ; hWnd
10001A48             call  ds:MessageBoxA
10001A4E

10001A4E loc_10001A4E:                          ; ...
10001A4E             pop   ebp
10001A4F             pop   edi
10001A50             mov   eax, ebx
10001A52             pop   esi
10001A53             pop   ebx
10001A54             retn
10001A55 ; ----------------------------------------------------------------------------
10001A55

10001A55 ReadMemError:                         ; ...
10001A55             mov   ecx, ds:hObject
10001A5B             mov   esi, ds:CloseHandle
10001A61             push  ecx                 ; hObject
10001A62             call  esi ; CloseHandle
10001A64             push  edi                 ; hObject
10001A65             call  esi ; CloseHandle
10001A67             mov   edx, [esp+0Ch+hWnd]
10001A6B             push  40000h              ; uType
10001A70             push  offset szError      ; lpCaption
10001A75             push  offset szDosExecutable_ ; lpText
10001A7A             push  edx                 ; hWnd
10001A7B             call  ds:MessageBoxA
10001A81

10001A81 loc_10001A81:                          ; ...
10001A81             pop   edi
10001A82             pop   esi
10001A83             xor   eax, eax
10001A85             pop   ebx
10001A86             retn
10001A86 DoMyJob   endp
10001A86

10001A86 ; ----------------------------------------------------------------------------
```

该插件把 Idle 以后的 PE 文件映象读取至一块动态分配的区域，然后扫描各种语言的入口特征码。

由此看来，有如下几种方法可以使 `Generic OEP Finder` 失效：

1. `Stolen Code`（也是最彻底的方法），等有空些个程序演示一下；
2. 双进程方式（当然包括多进程）；
3. 父进程检测（但是把 `PEid` 改名为 `Cmd` 就可以避开了）；
4. `Hook` 启动 API，在 `WaitForInputIdle` 之前清除 OEP 代码，比如 `GetVersion`/`GetCommandLine` 等；
5. 在 `Loader` 中设置文件头不可访问，不过这很可能在升级版本中解决，只要 `VirtualProtectEx` 就可以了；
6. 在 `Loader` 中清除 PE 标志（不知是否对程序运行有影响）；
7. 有待各位看官补充:P

最后来看看特征码都是什么：

```asm
10001068

10001068 FeatureCode_2:                       ; ...
10001068             push  0
1000106A             push  0
1000106F             call  $+5
10001074             mov   edi, 0
10001079             mov   eax, edi
1000107B             call  $+5
10001080             mov   [ebp+0], esp
10001083             mov   esi, esp
10001085             mov   [esi], edi
10001087             push  esi
10001087 ; ----------------------------------------------------------------------------
10001088             dw 15FFh
1000108A             dw 0
1000108C ; ----------------------------------------------------------------------------
1000108C

1000108C FeatureCode_13:                      ; ...
1000108C             push  70h
1000108E             push  0
1000108E ; ----------------------------------------------------------------------------
10001093             db 0E8h
10001094 ; ----------------------------------------------------------------------------
10001094

10001094 FeatureCode_1:                       ; ...
10001094             push  ebp
10001095             mov   ebp, esp
10001097             push  0FFFFFFFFh
10001099             push  0
1000109E             push  0
100010A3             mov   eax, large fs:0
100010A9             push  eax
100010AA             mov   large fs:0, esp
100010B1             add   dword ptr [eax], 0
100010B4             push  ebx
100010B5             push  esi
100010B6             push  edi
100010B7             mov   [ebp-18h], esp
100010B7 ; ----------------------------------------------------------------------------
100010BA             db 2 dup(0)
100010BC ; ----------------------------------------------------------------------------
100010BC

100010BC FeatureCode_3:                       ; ...
100010BC             mov   eax, large fs:0
100010C2             push  ebp
100010C3             mov   ebp, esp
```



100010C5 `push 0FFFFFFFFh`

100010C7 `push 0`

100010CC `push 0`

100010D1 `push eax`

100010D2 `mov large fs:0, esp`

100010D9 `sub esp, 0`

100010DC `sub edx, [esi+57h]`

100010DC ; ----------------------------------------------------------------------------

100010DF `db 0 ;`

100010E0 ; ----------------------------------------------------------------------------

100010E0

100010E0 `FeatureCode_11:` ; ...

100010E0 `push ebp`

100010E1 `mov ebp, esp`

100010E3 `add esp, 0`

100010E3 ; ----------------------------------------------------------------------------

100010E6 `db 0 ;`

100010E7 `db 0 ;`

100010E8 ; ----------------------------------------------------------------------------

100010E8

100010E8 `FeatureCode_4:` ; ...

100010E8 `mov eax, large fs:0`

100010EE `push ebp`

100010EF `mov ebp, esp`

100010F1 `push 0FFFFFFFFh`

100010F3 `push 0`

100010F8 `push 40109Ah`

100010FD `push eax`

100010FE `mov large fs:0, esp`

10001105 `add dword ptr [eax], 0`

10001108

10001108 `FeatureCode_5:` ; ...

10001108 `mov eax, large ds:0`

1000110D `shl eax, 2`

10001110 `mov large ds:0, eax`

10001115 `push edi`

10001116 `push ecx`

10001117 `xor eax, eax`

10001119 `mov edi, 0`

1000111E `mov ecx, 0`

10001123 `cmp ecx, edi`

10001125 `jbe short loc_1000112C`

10001127 `sub ecx, edi`

10001129 `cld`

1000112A `rep stosb`

1000112C

1000112C `loc_1000112C:` ; ...

1000112C `pop ecx`

1000112D `pop edi`

1000112D ; ----------------------------------------------------------------------------

1000112E `dw 0`

10001130 ; ----------------------------------------------------------------------------

10001130

10001130 `FeatureCode_6:` ; ...

10001130 `jmp short loc_10001142`

10001132 ; ----------------------------------------------------------------------------

10001132 `bound di, [edx]`

10001135 `inc ebx`

10001136 `sub ebp, [ebx]`

10001138 `dec eax`

10001139 `dec edi`

1000113A `dec edi`

1000113B `dec ebx`

1000113C `nop`

1000113D `jmp $+5`

10001142

10001142 `loc_10001142:` ; ...

10001142 `mov eax, large ds:0`

10001147 `shl eax, 2`

1000114A `mov large ds:0, eax`

1000114F `push edx`

10001150

10001150 `FeatureCode_8:` ; ...

10001150 `push ebp`

10001151 `mov ebp, esp`

10001153 `sub esp, 44h`

10001156 `push esi`

10001157 `call large dword ptr ds:0`

1000115D `mov esi, eax`

1000115F `mov al, [eax]`

10001161 `cmp al, 22h`

10001161 ; ----------------------------------------------------------------------------

10001163 `db 74h`

10001164 ; ----------------------------------------------------------------------------

10001164

10001164 `FeatureCode_7:` ; ...

10001164 `push ebx`

10001165 `push ecx`

10001166 `push edx`

10001167 `push ebp`

10001168 `mov ebp, esp`

1000116A `sub esp, 8`

1000116D `mov eax, 1`

10001172 `call $+5`

10001177 `mov eax, large ds:0`

1000117C `add eax, 3`

1000117F `das`

10001180 `dec esp`

10001181 `xor edx, edx`

10001181 ; ----------------------------------------------------------------------------

10001183 `db 0`

10001184 `FeatureCode_10` `db 0FFh` ; ; ...

10001185 ; ----------------------------------------------------------------------------

10001185 `and eax, 0`

1000118A `jmp large dword ptr ds:0`

1000118A ; ----------------------------------------------------------------------------

10001190 `dd 68h, 0E800h, 0FFFFh, 0, 30h`

100011A4 ; ----------------------------------------------------------------------------

100011A4

100011A4 `FeatureCode_9:` ; ...

100011A4 `jmp large dword ptr ds:0`

100011AA ; ----------------------------------------------------------------------------

100011AA `jmp large dword ptr ds:0`

100011AA ; ----------------------------------------------------------------------------

100011B0 `dd 680000h, 0E8000000h`

100011B8 `dd 0FFFF0000h, 0, 300000h`

100011C4 ; ----------------------------------------------------------------------------

100011C4

100011C4 `FeatureCode_12:` ; ...

100011C4 `push 0`

100011C4 ; ----------------------------------------------------------------------------

100011C6 `db 0E8h`

100011C7 `db 0 ;`

100011C8 `db 0 ;`

100011C9 `db 0 ;`

100011CA `db 0 ;`

100011CB ; ----------------------------------------------------------------------------

感谢老罗的代码着色器:)

Regards,

cyclotron

05.2.9

`4dd466d0e1e0cc123afbe504da2462ca`
