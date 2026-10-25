
# 打造处理 OutputDebugString("%s")的 OD

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-511.htm`](https://www.pediy.com/kssd/pediy07/pediy7-511.htm)

打造能处理 `OutputDebugString("%s%s%s")` 的 OD

最近好多壳利用 `OutputDebugString` 函数来对付 `OD`, 当然处理方法也比较简单, 更有大侠出了修改版 `OD`.

不过我这个人比较喜欢寻根究底, 想看看 `OD` 的毛病出在哪里.

## 1. 基础知识 from MSDN

下面是调试器和被调试进程之间的关系, 调试器用 `CreateProcess` 创建被调试进程.

被调试进程运行中会遇到 9 种需要向调试器汇报的事件, 调试器处理好后, 被调试进程才能继续.

平时我们最关心 `Exception`, 今天我们的主角是 `DebugString`.

`EXCEPTION_DEBUG_EVENT` equ `1`
`CREATE_THREAD_DEBUG_EVENT` equ `2`
`CREATE_PROCESS_DEBUG_EVENT` equ `3`
`EXIT_THREAD_DEBUG_EVENT` equ `4`
`EXIT_PROCESS_DEBUG_EVENT` equ `5`
`LOAD_DLL_DEBUG_EVENT` equ `6`
`UNLOAD_DLL_DEBUG_EVENT` equ `7`
`OUTPUT_DEBUG_STRING_EVENT` equ `8`
`RIP_EVENT` equ `9`

```
while(WaitForDebugEvent(&DBEvent, INFINITE))
{
    if (DBEvent.dwDebugEventCode==EXCEPTION_DEBUG_EVENT)
    {
      // 子进程遇到一个异常
    }
                else if (DBEvent.dwDebugEventCode==OUTPUT_DEBUG_STRING_EVENT)
    {
                        // 子进程执行了 OutputDebugString 函数
    }
                ....    // 共有 9 种情形
                ContinueDebugEvent(DBEvent.dwProcessId, DBEvent.dwThreadId, dwContinueStatus);
}
```

```
typedef struct _DEBUG_EVENT {  // de
    DWORD dwDebugEventCode;
    DWORD dwProcessId;
    DWORD dwThreadId;
    union {
        EXCEPTION_DEBUG_INFO        Exception;
        CREATE_THREAD_DEBUG_INFO    CreateThread;
        CREATE_PROCESS_DEBUG_INFO   CreateProcessInfo;
        EXIT_THREAD_DEBUG_INFO      ExitThread;
        EXIT_PROCESS_DEBUG_INFO     ExitProcess;
        LOAD_DLL_DEBUG_INFO         LoadDll;
        UNLOAD_DLL_DEBUG_INFO       UnloadDll;
        OUTPUT_DEBUG_STRING_INFO    DebugString;
        RIP_INFO                    RipInfo;
    } u;
} DEBUG_EVENT;
```

```
typedef struct _OUTPUT_DEBUG_STRING_INFO {  // odsi
    LPSTR lpDebugStringData;
    WORD  fUnicode;
    WORD  nDebugStringLength;
} OUTPUT_DEBUG_STRING_INFO;
```

`int vsprintf( char *buffer, const char *format, va_list argptr );` 这个是错误的根源

## 2. 测试用的程序

```
//=========================================== TEST1.EXE ===============================
.386
.model flat, stdcall
option casemap:none
include c:\masm32\include\kernel32.inc
include c:\masm32\include\user32.inc
includelib c:\masm32\lib\kernel32.lib
includelib c:\masm32\lib\user32.lib
include c:\masm32\include\windows.inc
.const
szOutput                db "%s",0
szMessage               db "The program is fine", 0
szTitle                 db "Test",0
szFormat                db "%s",0
.data?
szBuffer                db 20 dup (?)
;------------ CODE ----------
.code
main:
  invoke wsprintf, offset szBuffer, offset szFormat, offset szMessage
  invoke OutputDebugString, offset szOutput
  invoke MessageBox, 0, offset szBuffer, offset szTitle, MB_OK
  invoke ExitProcess,0
end main
//===================================== End ==============================================
```

用 `OD` 调试 `test1.exe`, 执行完 `OutputDebugString`, `OD` 左下角出现了 `"Debug string: "`

```
//================================= Test2.EXE =============================================
  其余不变, 只变一句
  szOutput           db "%s%s",0
用 OD 调试 test2.exe, 执行 OutputDebugString, OD 在 4a74cf 出现访问 `[00000260]` 异常
//================================== Test3.EXE ============================================
  其余不变, 只变一句
  szOutput           db "%s%s%s",0
用 OD 调试 test2.exe, 执行 OutputDebugString, OD 在 4a74cf 出现访问 `[00000001]` 异常

为什么 Test1 没问题, Test2, Test3 有问题? Test2 和 Test3 有区别吗? Let's go
```

## 3. 我们来看看 OD 是如何处理的

将 `Ollydbg.exe` 拷贝到 `CMD.EXE`, 运行 `CMD.exe`, 菜单 `File - Open - Ollydbg.EXE`,

这样 `CMD.exe` 是调试器, `Ollydbg.exe` 就是被调试的程序, 别搞错了.

找 `WaitForDebugEvent`, 比较容易找到处理 `DebugString` 的地方, `CMD` 下断 `431276`.

`F9` 运行 `Ollydbg.exe`, 再用 `Ollydbg.exe` 打开 `TEST3.EXE`, `F9` 运行, `CMD` 断下, `F7` 跟进

```
00439616   > \6A 00         PUSH 0                                 ; /Timeout = 0. ms
00439618   .  68 14574D00   PUSH OLLYDBG.004D5714                    ; |pDebugEvent = OLLYDBG.004D5714
0043961D   .  E8 E85B0700   CALL <JMP.&KERNEL32.WaitForDebugEvent>   ; \\WaitForDebugEvent
00439622   .  85C0          TEST EAX,EAX
00439624   .  75 44         JNZ SHORT OLLYDBG.0043966A
...
0043977D   .  51            PUSH ECX                                  ; /Arg1
0043977E   .  E8 4D54FFFF   CALL OLLYDBG.0042EBD0                     ; \\OLLYDBG.0042EBD0, F7 跟进
0042EBD0  /$  55            PUSH EBP
0042EBD1  |.  8BEC          MOV EBP,ESP
0042EBD3  |.  81C4 04F0FFFF ADD ESP,-0FFC
0042EBD9  |.  50            PUSH EAX
0042EBDA  |.  81C4 00F5FFFF ADD ESP,-0B00
0042EBE0  |.  53            PUSH EBX
0042EBE1  |.  56            PUSH ESI
0042EBE2  |.  57            PUSH EDI
0042EBE3  |.  8B35 1C574D00 MOV ESI,DWORD PTR DS:[4D571C]            ; dwThreadID
0042EBE9  |.  56            PUSH ESI
0042EBEA  |.  E8 5DF8FFFF   CALL OLLYDBG.0042E44C
0042EBEF  |.  8BF8          MOV EDI,EAX
0042EBF1  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]
0042EBF4  |.  59            POP ECX
0042EBF5  |.  8938          MOV DWORD PTR DS:[EAX],EDI
0042EBF7  |.  8B15 14574D00 MOV EDX,DWORD PTR DS:[4D5714]            ;  pDebugEvent
0042EBFD  |.  83FA 09       CMP EDX,9                               ;  Switch (cases 1..9) 被调试进程遇到的 9 种情况
0042EC00  |.  0F87 EE270000 JA OLLYDBG.004313F4
0042EC06  |.  FF2495 0DEC42>JMP DWORD PTR DS:[EDX*4+42EC0D]
0042EC0D  |.  F4134300      DD OLLYDBG.004313F4                      ;  Switch table used at 0042EC06
0042EC11  |.  35EC4200      DD OLLYDBG.0042EC35
0042EC15  |.  FF0C4300      DD OLLYDBG.00430CFF
0042EC19  |.  D70D4300      DD OLLYDBG.00430DD7
0042EC1D  |.  3F0F4300      DD OLLYDBG.00430F3F
0042EC21  |.  37104300      DD OLLYDBG.00431037
0042EC25  |.  2D114300      DD OLLYDBG.0043112D
0042EC29  |.  B7114300      DD OLLYDBG.004311B7
0042EC2D  |.  76124300      DD OLLYDBG.00431276                      ; OUTPUT_DEBUG_STRING_EVENT 在 431276 处理
0042EC31  |.  C7134300      DD OLLYDBG.004313C7
00431276  |> \\830D 74574D00>OR DWORD PTR DS:[4D5774],4               ;  Case 8 of switch 0042EBFD
0043127D  |.  68 0A674B00   PUSH OLLYDBG.004B670A                    ; /Arg2 = 004B670A ASCII "Debug string: "
00431282  |.  8D95 98FDFFFF LEA EDX,DWORD PTR SS:[EBP-268]           ; |
00431288  |.  52            PUSH EDX                                ; |Arg1 = 12F340
00431289  |.  E8 9E590700   CALL OLLYDBG.004A6C2C                    ; \\int sprintf(char *buffer,const char *format,...)
0043128E  |.  83C4 08       ADD ESP,8
00431291  |.  8945 F4       MOV DWORD PTR SS:[EBP-C],EAX             ; "Debug string: " 的长度
00431294  |.  0FB71D 26574D>MOVZX EBX,WORD PTR DS:[4D5726]           ; debug_string 的长度
0043129B  |.  85DB          TEST EBX,EBX
0043129D  |.  7D 0A         JGE SHORT OLLYDBG.004312A9
0043129F  |.  B8 01000000   MOV EAX,1
004312A4  |.  E9 7C010000   JMP OLLYDBG.00431425
004312A9  |>  66:833D 24574>CMP WORD PTR DS:[4D5724],0               ; 是否 unicode ?
004312B1  |.  74 6B         JE SHORT OLLYDBG.0043131E                ; 由于我们使用了 OutputDebugStringA,  不是 unicode
0043131E  |>  BA 00010000   MOV EDX,100                              ; 包括 "Debug string: " 最长 100h 长度
00431323  |.  2B55 F4       SUB EDX,DWORD PTR SS:[EBP-C]
00431326  |.  4A            DEC EDX
00431327  |.  3BDA          CMP EBX,EDX
00431329  |.  7E 09         JLE SHORT OLLYDBG.00431334
0043132B  |.  BB 00010000   MOV EBX,100
00431330  |.  2B5D F4       SUB EBX,DWORD PTR SS:[EBP-C]
00431333  |.  4B            DEC EBX
```



`00431334`  `|>  6A 03         PUSH 3                                   ; /Arg4 = 00000003`  
`00431336`  `|.  53            PUSH EBX                                 ; |Arg3 = 长度`  
`00431337`  `|.  A1 20574D00   MOV EAX,DWORD PTR DS:[4D5720]            ; |`  
`0043133C`  `|.  50            PUSH EAX                                 ; |Arg2 => 00402018   debug_string 在被调试进程中的地址`  
`0043133D`  `|.  8D95 98FDFFFF LEA EDX,DWORD PTR SS:[EBP-268]           ; |`  
`00431343`  `|.  0355 F4       ADD EDX,DWORD PTR SS:[EBP-C]             ; |`  
`00431346`  `|.  52            PUSH EDX                                 ; |Arg1 = 0012F34E,  Buffer, 紧接"Debug string: "`  
`00431347`  `|.  E8 C0FF0200   CALL OLLYDBG._Readmemory                  ; \\_Readmemory( 调用 ReadProcessMemory )`  
`0043134C`  `|.  83C4 10       ADD ESP,10`  
`0043134F`  `|.  3BC3          CMP EAX,EBX                              ; EAX 真正读到的字节数`  
`00431351`  `|.  74 0A         JE SHORT OLLYDBG.0043135D`  
`00431353`  `|.  B8 01000000   MOV EAX,1`  
`00431358`  `|.  E9 C8000000   JMP OLLYDBG.00431425                     ; 读到的字节数不对, 结束`  
`0043135D`  `|>  015D F4       ADD DWORD PTR SS:[EBP-C],EBX             ; "Debug string: "+ debug_string 的长度`  
`00431360`  `|>  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]`  
`00431363`  `|.  8D8415 97FDFF>LEA EAX,DWORD PTR SS:[EBP+EDX-269]       ; 指向字符串末尾`  
`0043136A`  `|.  EB 04         JMP SHORT OLLYDBG.00431370               ; 去掉未尾的 0, 0D, 0A`  
`0043136C`  `|>  FF4D F4       /DEC DWORD PTR SS:[EBP-C]`  
`0043136F`  `|.  48            |DEC EAX`  
`00431370`  `|>  8038 00        CMP BYTE PTR DS:[EAX],0`  
`00431373`  `|.^ 74 F7         |JE SHORT OLLYDBG.0043136C`  
`00431375`  `|.  33D2          |XOR EDX,EDX`  
`00431377`  `|.  8A10          |MOV DL,BYTE PTR DS:[EAX]`  
`00431379`  `|.  83FA 0D       |CMP EDX,0D`  
`0043137C`  `|.^ 74 EE         |JE SHORT OLLYDBG.0043136C`  
`0043137E`  `|.  33C9          |XOR ECX,ECX`  
`00431380`  `|.  8A08          |MOV CL,BYTE PTR DS:[EAX]`  
`00431382`  `|.  83F9 0A       |CMP ECX,0A`  
`00431385`  `|.^ 74 E5         \\JE SHORT OLLYDBG.0043136C`  
`00431387`  `|.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]              ; 修正后的长度`  
`0043138A`  `|.  8D95 98FDFFFF LEA EDX,DWORD PTR SS:[EBP-268]            ; 字符串指针`  
`00431390`  `|.  85FF          TEST EDI,EDI`  
`00431392`  `|.  C68405 98FDFF>MOV BYTE PTR SS:[EBP+EAX-268],0           ; 修正后字符串末尾补零`  
`0043139A`  `|.  52            PUSH EDX`  
`0043139B`  `|.  75 04         JNZ SHORT OLLYDBG.004313A1`  
`0043139D`  `|.  33C9          XOR ECX,ECX`  
`0043139F`  `|.  EB 03         JMP SHORT OLLYDBG.004313A4`  
`004313A1`  `|>  8B4F 2C       MOV ECX,DWORD PTR DS:[EDI+2C]`  
`004313A4`  `|>  51            PUSH ECX                                 ; |Arg1`  
`004313A5`  `|.  E8 56CDFFFF   CALL OLLYDBG.0042E100                    ; \\OLLYDBG.0042E100,  F7 跟进`  
`004313AA`  `|.  83C4 08       ADD ESP,8`

```
0042E100  /$  55            PUSH EBP
0042E101  |.  8BEC          MOV EBP,ESP
0042E103  |.  81C4 04F0FFFF ADD ESP,-0FFC
0042E109  |.  50            PUSH EAX
0042E10A  |.  81C4 FCFDFFFF ADD ESP,-204
0042E110  |.  53            PUSH EBX
0042E111  |.  56            PUSH ESI
0042E112  |.  57            PUSH EDI
0042E113  |.  8D5D 10       LEA EBX,DWORD PTR SS:[EBP+10]
0042E116  |.  837D 0C 00    CMP DWORD PTR SS:[EBP+C],0               ; 字符指针是否为 0
0042E11A  |.  74 3D         JE SHORT OLLYDBG.0042E159
0042E11C  |.  6A 00         PUSH 0                                 ; /Arg1 = 00000000
0042E11E  |.  E8 59340000   CALL OLLYDBG.0043157C                    ; \\OLLYDBG.0043157C
0042E123  |.  59            POP ECX
0042E124  |.  53            PUSH EBX                                 ; /Arg3 = 0012DA9C
0042E125  |.  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]             ; |
0042E128  |.  50            PUSH EAX                                 ; |Arg2 = 0012F340 ASCII "Debug string: %s%s%s"
0042E129  |.  8D95 FCEDFFFF LEA EDX,DWORD PTR SS:[EBP-1204]          ; |
0042E12F  |.  52            PUSH EDX                                 ; |Arg1 = 0012C888
0042E130  |.  E8 1F8B0700   CALL OLLYDBG.004A6C54                    ; \\int vsprintf( char *buffer, const char *format, va_list argptr );
```

这时如果 `F8`, OllyDbg 在 `4a74cf` 出现访问 `[00000001]` 异常, 看一下 Stack, 很容易知道原因

```
0012DA9C  |03A9066C
0012DAA0  |004B59E6  ASCII "%s - %s"
0012DAA4  |00000001
argptr=12DA9C
```

第 1 个 `%s` 就是 `03A9066C` 指向的字符串, `[03A9066C]=0`  
第 2 个 `%s` 就是 `004B59E6` 指向的字符串 `"%s - %s"` (这完全是巧合)  
第 3 个 `%s` 就是 `00000001` 指向的字符串, 哈

如果 是 `Test1.eXe` , 只有 1 个 `%s`, 运气非常好, 能走过去, `12C888` 变成 `"Debug string: "`, 以后也不会有问题了. OK

如果 是 `Test2.exe` , 只有 2 个 `%s`, 暂时运气不错, 能走过去, `12C888` 变成 `"Debug string: %s - %s"`

继续走

```
0042E21B  |.  50            PUSH EAX                                 ; /Arg3 = 0012C888 ASCII "Debug string: %s - %s"
0042E21C  |.  6A 00         PUSH 0                                 ; |Arg2 = 00000000
0042E21E  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]             ; |
0042E221  |.  52            PUSH EDX                                 ; |Arg1 = 7C59BBF3
0042E222  |.  E8 E5C30200   CALL OLLYDBG._Addtolist                  ; \\int __cdecl Addtolist(int,char,char *format,int arglist),  F7 跟进
0042E227  |.  83C4 0C       ADD ESP,0C
```

```
0045A60C >/$  55            PUSH EBP
0045A60D  |.  8BEC          MOV EBP,ESP
0045A60F  |.  81C4 F8FDFFFF ADD ESP,-208
0045A615  |.  53            PUSH EBX
0045A616  |.  56            PUSH ESI
0045A617  |.  837D 10 00    CMP DWORD PTR SS:[EBP+10],0
0045A61B  |.  0F84 0E020000 JE OLLYDBG.0045A82F
0045A621  |.  8D45 14       LEA EAX,DWORD PTR SS:[EBP+14]
0045A624  |.  50            PUSH EAX                                 ; /Arg3 = 0012C87C
0045A628  |.  52            PUSH EDX                                 ; |Arg2 = 0012C888 ASCII "Debug string: %s - %s"
0045A62F  |.  51            PUSH ECX                                 ; |Arg1 = 0012C660
0045A630  |.  E8 1FC60400   CALL OLLYDBG.004A6C54                    ; \\int vsprintf( char *buffer, const char *format, va_list argptr );
```

这时如果 `F8`, OllyDbg 在 `4a74cf` 出现访问 `[00000260]` 异常, 看一下 Stack, 很容易知道原因

```
0012C87C  |03A902EC
0012C880  |00000260
argptr=12C87C
```

第 1 个 `%s` 就是 `03A902EC` 指向的字符串, `[03A902EC]=0`  
第 2 个 `%s` 就是 `00000260` 指向的字符串,

4 结论

从上面可知 `OutputDebugString("%s%s%s")` 对付 OD 的原理, 修改 OD 方法也很简单.

`431360-431385` 这里没什么用, 可以修改成下面

```
00431360  |> \\8B55 F4            MOV EDX,DWORD PTR SS:[EBP-C]
00431363      8D85 98FDFFFF      LEA EAX,DWORD PTR SS:[EBP-268]
00431369      8038 25            CMP BYTE PTR DS:[EAX],25             ; 找 "%"
0043136C      75 03              JNZ SHORT OLLYDBG.00431371
0043136E      C600 26            MOV BYTE PTR DS:[EAX],26             ; 用 "&" 代替
00431371      40                  INC EAX
00431372      4A                  DEC EDX
00431373    ^ 75 F4              JNZ SHORT OLLYDBG.00431369
00431375      90                  NOP
00431376      90                  NOP
00431377      90                  NOP
00431378      90                  NOP
00431379      90                  NOP
0043137A      90                  NOP
0043137B      90                  NOP
0043137C      90                  NOP
0043137D      90                  NOP
0043137E      90                  NOP
0043137F      90                  NOP
00431380      90                  NOP
00431381      90                  NOP
00431382      90                  NOP
00431383      90                  NOP
00431384      90                  NOP
00431385      90                  NOP
00431386      90                  NOP
```

`4dd466d0e1e0cc123afbe504da2462ca`
