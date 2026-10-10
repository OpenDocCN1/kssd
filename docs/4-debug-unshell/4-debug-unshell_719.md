

# jdpack1.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6256.htm`](https://www.pediy.com/kssd/pediy06/pediy6256.htm)

国庆之礼，特此献给广大破解爱好者   
【软件名称】:Notepad   
【大    小】:52KB   
【下载地址】:N/A   
【软件限制】:None    
【软件简介】:None    
  
破解信息   
【作    者】:loveboom   
【破解平台】:Windows 98se   
【目    标】:脱壳    
【难度系数】:★☆☆☆☆(1-2)   
【相关工具】:Ollydbg 1.09d   
  
【作者声明】: 初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！   
  
【详细过程】   
下面没有说出的地方就是可以按 f8 过去的   
0040D000 > 60               PUSHAD                                   ; 载入在此   
0040D001   E8 00000000      CALL JDPACK1\_.0040D006   
0040D006   5D               POP EBP   
0040D007   8BD5             MOV EDX,EBP   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D055   46               INC ESI   
0040D056  ^E2 EB            LOOPD SHORT JDPACK1\_.0040D043            ; F4 下一行   
0040D058   9C               PUSHFD   
0040D059   58               POP EAX   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D0B0   AA               STOS BYTE PTR ES:\[EDI\]   
0040D0B1  ^E2 F6            LOOPD SHORT JDPACK1\_.0040D0A9            ; 又一循环，F4 到下一行   
0040D0B3   61               POPAD   
0040D0B4   8BBD E1314000    MOV EDI,DWORD PTR SS:\[EBP+4031E1\]   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D0F6  ⁰F85 72FFFFFF    JNZ JDPACK1\_.0040D06E                    ; 这里往上跳不能跳下一行 F4   
0040D0FC   83BD E9314000 00 CMP DWORD PTR SS:\[EBP+4031E9\],0   
0040D103   0F84 8A000000    JE JDPACK1\_.0040D193   
0040D109   8B95 65344000    MOV EDX,DWORD PTR SS:\[EBP+403465\]   
0040D10F   8B85 E5314000    MOV EAX,DWORD PTR SS:\[EBP+4031E5\]   
0040D115   2BD0             SUB EDX,EAX   
0040D117   74 7A            JE SHORT JDPACK1\_.0040D193               ; 跳成功了   
0040D119   8BC2             MOV EAX,EDX   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D27E   75 04            JNZ SHORT JDPACK1\_.0040D284   
0040D280   61               POPAD                                    ; 这里要注意了，不要见到 popad 就 f4 到这里   
0040D281   33C0             XOR EAX,EAX   
0040D283   C3               RETN   
0040D284   8B95 65344000    MOV EDX,DWORD PTR SS:\[EBP+403465\]        ; JDPACK1\_.00400000   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D368   FFB5 F9314000    PUSH DWORD PTR SS:\[EBP+4031F9\]   
0040D36E   FF95 E0334000    CALL DWORD PTR SS:\[EBP+4033E0\]   
0040D374   EB 04            JMP SHORT JDPACK1\_.0040D37A   
0040D376   C603 00          MOV BYTE PTR DS:\[EBX\],0   
0040D379   43               INC EBX   
0040D37A   803B 00          CMP BYTE PTR DS:\[EBX\],0   
0040D37D  ⁷⁵ F7            JNZ SHORT JDPACK1\_.0040D376              ; 这里循环，f4 下一行跳过   
0040D37F   0BC0             OR EAX,EAX   
0040D381   75 4B            JNZ SHORT JDPACK1\_.0040D3CE   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略若干\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0040D3CE   8907             MOV DWORD PTR DS:\[EDI\],EAX   
0040D3D0   8385 FD314000 04 ADD DWORD PTR SS:\[EBP+4031FD\],4   
0040D3D7  ^E9 3BFFFFFF      JMP JDPACK1\_.0040D317                    ; 这里不能跳   
0040D3DC   83C6 14          ADD ESI,14   
0040D3DF   8B95 65344000    MOV EDX,DWORD PTR SS:\[EBP+403465\]   
0040D3E5  ^E9 A8FEFFFF      JMP JDPACK1\_.0040D292                    ; 这里也不能   
0040D3EA   6A 30            PUSH 30                                  ; 可以在这里按 f4   
0040D3EC   8D9D 4D324000    LEA EBX,DWORD PTR SS:\[EBP+40324D\]   
0040D3F2   53               PUSH EBX                                 ; 这里开始弹出未注册对话框了   
0040D3F3   8D9D 61324000    LEA EBX,DWORD PTR SS:\[EBP+403261\]   
0040D3F9   53               PUSH EBX   
0040D3FA   6A 00            PUSH 0   
0040D3FC   FF95 D5314000    CALL DWORD PTR SS:\[EBP+4031D5\]           ; 这里就是 Messageboxa   
0040D402   8B95 65344000    MOV EDX,DWORD PTR SS:\[EBP+403465\]        ; 可以根据这个下 bpx messageboxa 快点到入口，不过先要躲过它的 anti code   
0040D408   8B85 ED314000    MOV EAX,DWORD PTR SS:\[EBP+4031ED\]   
0040D40E   03C2             ADD EAX,EDX   
0040D410   894424 1C        MOV DWORD PTR SS:\[ESP+1C\],EAX   
0040D414   61               POPAD   
0040D415   50               PUSH EAX                                 ; 解压结束，push 程序入口   
0040D416   C3               RETN                ;过了这个地方就 dump   
注意了,这个程序不能直接用 olldump2.21 来 dump 进程,必须用别的工具来 dump,我用的是 lordpe.   
【整    理】   
这是国内少数几个加壳软件中的一款,不过壳的难度不大,虽然有 anticode 不过太过普通了.   
不可否认，我写的这些东西前已经有不少前辈写过了，我就当是一个收集者。这几篇脱壳文章没什么难度，算是给像我一样在菜鸟中苦苦挣扎的兄弟们一点帮助吧。   
  
★★★★★★★★★★★★★★★★★★★★★★★★★★   
Cracked by Loveboom   
Email:loveboomNew@hotmail.com   
QQ:88427763   
Wednesday, October 01, 2003   
 ★★★★★★★★★★★★★★★★★★★★★★★★★★   
愿天下和我一样孤独、寂寞的人能早日找到一丝归宿的感觉   
愿远在它乡的亲人永远身体健康、平安。  
 4dd466d0e1e0cc123afbe504da2462ca