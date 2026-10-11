
`0049F67C  74 1A               JE SHORT 0049F698                       ; 如果全部 DLL 加载完则跳去结束处`

`0049F67E  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]`

`0049F681  8B5F 10             MOV EBX,DWORD PTR DS:[EDI+10]`

`0049F684  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]            ; 获取 DLL 的名称`

`0049F687  8BF3                MOV ESI,EBX`

`0049F689  56                  PUSH ESI                                ; /push FileName`

`0049F68A  8B5D 0C             MOV EBX,DWORD PTR SS:[EBP+C]           ; |`

`0049F68D  FF93 D1BA4000       CALL DWORD PTR DS:[EBX+40BAD1]         ; \\LoadLibraryA`

`0049F693  83C7 14             ADD EDI,14`

`0049F696 ^ EB DE              JMP SHORT 0049F676                     ; 跳去取下一个 dll 名`

`0049F698  61                  POPAD`

`0049F699  C9                  LEAVE`

`0049F69A  C2 0800             RETN 8`

……

`004A7780  CC                  INT3                                    ; 再下来这里就异常了,记得上面我们保存下来的代码吗?`

是了，把从 `4A7781` 处到 `4A9A04` 处的代码还原正确的回去。

`004A7781  33C9                XOR ECX,ECX`

`004A7783  8B9D FEC14000      MOV EBX,DWORD PTR SS:[EBP+40C1FE]       ; 还原 API 的地址[4AA1FE]=004AAA00`

`004A7789  EB 15               JMP SHORT 004A77A0`

`004A778B  FF348B              PUSH DWORD PTR DS:[EBX+ECX*4]          ; 471cec`

`004A778E  FFB5 F6BD4000       PUSH DWORD PTR SS:[EBP+40BDF6]         ; relo =9e000`

`004A7794  FFB5 DEBD4000       PUSH DWORD PTR SS:[EBP+40BDDE]         ; pe header =400000`

`004A779A  E8 DE78FFFF         CALL 0049F07D                         ; Fill Api Function`

`0049F07D  55                  PUSH EBP`

`0049F07E  8BEC                MOV EBP,ESP`

`0049F080  83C4 EC             ADD ESP,-14`

`0049F083  60                  PUSHAD`

`0049F084  8B45 08             MOV EAX,DWORD PTR SS:[EBP+8]`

`0049F087  8B50 3C             MOV EDX,DWORD PTR DS:[EAX+3C]`

`0049F08A  03C2                ADD EAX,EDX`

`0049F08C  8B90 80000000       MOV EDX,DWORD PTR DS:[EAX+80]`

`0049F092  0355 08             ADD EDX,DWORD PTR SS:[EBP+8]`

`0049F095  8955 FC             MOV DWORD PTR SS:[EBP-4],EDX`

`0049F098  8BFA                MOV EDI,EDX`

`0049F09A  8B5F 10             MOV EBX,DWORD PTR DS:[EDI+10]`

`0049F09D  83FB 00             CMP EBX,0`

`0049F0A0  0F84 09010000       JE 0049F1AF                           ; 如果输入表处理完则啵`

`0049F0A6  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]`

`0049F0A9  895D F0             MOV DWORD PTR SS:[EBP-10],EBX`

`0049F0AC  FF75 F0             PUSH DWORD PTR SS:[EBP-10]            ; push DLL name`

`0049F0AF  8B5D 0C             MOV EBX,DWORD PTR SS:[EBP+C]`

`0049F0B2  FF93 C0BA4000       CALL DWORD PTR DS:[EBX+40BAC0]        ; GetModuleHandleA 获取 dll 的句柄`

`0049F0B8  8BD0                MOV EDX,EAX`

`0049F0BA  8B5F 0C             MOV EBX,DWORD PTR DS:[EDI+C]`

`0049F0BD  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]`

`0049F0C0  895D F8             MOV DWORD PTR SS:[EBP-8],EBX`

`0049F0C3  8B1F                MOV EBX,DWORD PTR DS:[EDI]            ; OfirstThunk`

`0049F0C5  83FB 00             CMP EBX,0`

`0049F0C8  0F84 8B000000       JE 0049F159                           ; 如果 OriginalFirstThunk 为空则跳`

`0049F0CE  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]`

`0049F0D1  895D F4             MOV DWORD PTR SS:[EBP-C],EBX`

`0049F0D4  33C9                XOR ECX,ECX`

`0049F0D6  8B75 F4             MOV ESI,DWORD PTR SS:[EBP-C]`

`0049F0D9  8B048E              MOV EAX,DWORD PTR DS:[ESI+ECX*4]`

`0049F0DC  83F8 00             CMP EAX,0`

`0049F0DF  0F84 C2000000       JE 0049F1A7                           ; 比较当前 DLL 的 API 函数是否已经处理完了`

`0049F0E5  25 00000080         AND EAX,80000000`

`0049F0EA  83F8 00             CMP EAX,0`

`0049F0ED  74 32               JE SHORT 0049F121                     ; 判断是否为序号方式获取，如果是认名字则啵`

`0049F0EF  8B048E              MOV EAX,DWORD PTR DS:[ESI+ECX*4]`

`0049F0F2  25 FFFFFF7F         AND EAX,7FFFFFFF`

`0049F0F7  8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]`

`0049F0FA  8D1C8B              LEA EBX,DWORD PTR DS:[EBX+ECX*4]`

`0049F0FD  3B5D 10             CMP EBX,DWORD PTR SS:[EBP+10]`

`0049F100  75 1C               JNZ SHORT 0049F11E`

`0049F102  51                  PUSH ECX`

`0049F103  52                  PUSH EDX`

`0049F104  50                  PUSH EAX`

`0049F105  52                  PUSH EDX`

`0049F106  8B5D 0C             MOV EBX,DWORD PTR SS:[EBP+C]`

`0049F109  FF93 E4BA4000       CALL DWORD PTR DS:[EBX+40BAE4]`

`0049F10F  5A                  POP EDX`

`0049F110  59                  POP ECX`

`0049F111  8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]`

`0049F114  8D1C8B              LEA EBX,DWORD PTR DS:[EBX+ECX*4]`

`0049F117  8903                MOV DWORD PTR DS:[EBX],EAX`

`0049F119  E9 91000000         JMP 0049F1AF`

`0049F11E  41                  INC ECX`

`0049F11F ^ EB B5             JMP SHORT 0049F0D6`

`0049F121  8B75 F4             MOV ESI,DWORD PTR SS:[EBP-C]          ; OriginalFirstThunk`

`0049F124  8B048E              MOV EAX,DWORD PTR DS:[ESI+ECX*4]`

`0049F127  83F8 00             CMP EAX,0`

`0049F12A  74 7B               JE SHORT 0049F1A7`

`0049F12C  0345 08             ADD EAX,DWORD PTR SS:[EBP+8]`

`0049F12F  83C0 02             ADD EAX,2`

`0049F132  8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]`

`0049F135  8D1C8B              LEA EBX,DWORD PTR DS:[EBX+ECX*4]`

`0049F138  3B5D 10             CMP EBX,DWORD PTR SS:[EBP+10]`

`0049F13B  75 19               JNZ SHORT 0049F156`

`0049F13D  51                  PUSH ECX`

`0049F13E  52                  PUSH EDX`

`0049F13F  50                  PUSH EAX                              ; /ProcNameOrOrdinal`

`0049F140  52                  PUSH EDX                              ; |hModule`

`0049F141  8B5D 0C             MOV EBX,DWORD PTR SS:[EBP+C]          ; |`

`0049F144  FF93 E4BA4000       CALL DWORD PTR DS:[EBX+40BAE4]        ; \\GetProcAddress`

`0049F14A  5A                  POP EDX`

`0049F14B  59                  POP ECX`

`0049F14C  8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]`

`0049F14F  8D1C8B              LEA EBX,DWORD PTR DS:[EBX+ECX*4]`

`0049F152  8903                MOV DWORD PTR DS:[EBX],EAX`

`0049F154  EB 59               JMP SHORT 0049F1AF`

`0049F156  41                  INC ECX`

`0049F157 ^ EB C8             JMP SHORT 0049F121`

`0049F159  33C9                XOR ECX,ECX`

`0049F15B  8B5F 24             MOV EBX,DWORD PTR DS:[EDI+24]`

`0049F15E  035D 08             ADD EBX,DWORD PTR SS:[EBP+8]`

`0049F161  895D EC             MOV DWORD PTR SS:[EBP-14],EBX`

`0049F164  8B75 F0             MOV ESI,DWORD PTR SS:[EBP-10]`

`0049F167  803E 00             CMP BYTE PTR DS:[ESI],0`

`0049F16A  74 03               JE SHORT 0049F16F`

`0049F16C  46                  INC ESI`

`0049F16D ^ EB F8             JMP SHORT 0049F167`

`0049F16F  46                  INC ESI`

`0049F170  3B75 EC             CMP ESI,DWORD PTR SS:[EBP-14]`

`0049F173  74 32               JE SHORT 0049F1A7`

`0049F175  803E 00             CMP BYTE PTR DS:[ESI],0`

`0049F178 ^ 74 F5             JE SHORT 0049F16F`

`0049F17A  837E 01 00          CMP DWORD PTR DS:[ESI+1],0`

`0049F17E  74 2F               JE SHORT 0049F1AF`

`0049F180  8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]`

`0049F183  8D1C8B              LEA EBX,DWORD PTR DS:[EBX+ECX*4]`

`0049F186  3B5D 10             CMP EBX,DWORD PTR SS:[EBP+10]`

`0049F189  75 19               JNZ SHORT 0049F1A4`

`0049F18B  51                  PUSH ECX`

`0049F18C  52                  PUSH EDX`

`0049F18D  56                  PUSH ESI`

`0049F18E  52                  PUSH EDX`
