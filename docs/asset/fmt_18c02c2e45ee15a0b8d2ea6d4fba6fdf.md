
```
00409757   .  895D B0       MOV DWORD PTR SS:[EBP-50],EBX   

0040975A   .  FFD7         CALL EDI   

0040975C   .  8BD0         MOV EDX,EAX------------------------------->取得的 4 位字符   

0040975E   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]------------->新地址   

00409761   .  FFD6         CALL ESI---------------------------------->保存于上面的地址   

00409763   .  8D45 CC      LEA EAX,DWORD PTR SS:[EBP-34]------------->保存的 4 位字符   

00409766   .  68 00384000  PUSH REGAVAX.00403800                    ;  UNICODE "&H"   

0040976B   .  50           PUSH EAX---------------------------------->4 位字符   

0040976C   .  E8 5F090000  CALL REGAVAX.0040A0D0--------------------->将上面的 4 位字符的顺序反转   

00409771   .  8BD0         MOV EDX,EAX------------------------------->反转后的 4 位字符   

00409773   .  8D4D C8      LEA ECX,DWORD PTR SS:[EBP-38]------------->新地址   

00409776   .  FFD6         CALL ESI---------------------------------->保存反转后的 4 位字符   

00409778   .  50           PUSH EAX---------------------------------->反转后的 4 位字符   

00409779   .  FF15 74104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat]-->将"&H"加到反转后的 4 位字符的前面   

0040977F   .  8BD0         MOV EDX,EAX-------------------------------->反转后前面加上"&H"后的 4 位字符   

00409781   .  8D4D C4      LEA ECX,DWORD PTR SS:[EBP-3C]-------------->新地址   

00409784   .  FFD6         CALL ESI----------------------------------->保存于新地址   

00409786   .  50           PUSH EAX----------------------------------->入栈   

00409787   .  FF15 B0124000 CALL DWORD PTR DS:[<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr   

0040978D   .  8B1D 58124000 MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaFp>;  MSVBVM60.__vbaFpI4   

00409793   .  FFD3         CALL EBX                                   ;  <&MSVBVM60.__vbaFpI4>   

00409795   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:[EBP-8C]   

0040979B   .  8985 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EAX   

004097A1   .  51           PUSH ECX   

004097A2   .  E8 E9080000  CALL REGAVAX.0040A090--------------------->   

004097A7   .  D99D 70FFFFFF FSTP DWORD PTR SS:[EBP-90]   

004097AD   .  D985 70FFFFFF FLD DWORD PTR SS:[EBP-90]   

004097B3   .  DC2D 48134000 FSUBR QWORD PTR DS:[401348]   

004097B9   .  DFE0         FSTSW AX   

004097BB   .  A8 0D        TEST AL,0D-------------------------------->   

004097BD   .  0F85 F2070000 JNZ REGAVAX.00409FB5   

004097C3   .  FFD3         CALL EBX   

004097C5   .  8945 EC      MOV DWORD PTR SS:[EBP-14],EAX   

004097C8   .  8D55 C4      LEA EDX,DWORD PTR SS:[EBP-3C]   

004097CB   .  8D45 C8      LEA EAX,DWORD PTR SS:[EBP-38]   

004097CE   .  52           PUSH EDX   

004097CF   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004097D2   .  50           PUSH EAX   

004097D3   .  51           PUSH ECX   

004097D4   .  6A 03        PUSH 3   

004097D6   .  FF15 0C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS];  MSVBVM60.__vbaFreeStrList   

004097DC   .  83C4 10      ADD ESP,10   

004097DF   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

004097E2   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

004097E8   .  8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]   

004097EB   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

004097F2   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

004097F9   .  52           PUSH EDX   

004097FA   .  8B45 D0      MOV EAX,DWORD PTR SS:[EBP-30]------------->掐头去尾的注册码   

004097FD   .  6A 05        PUSH 5------------------------------------>第 5 位   

004097FF   .  50           PUSH EAX---------------------------------->掐头去尾的注册码   

00409800   .  FFD7         CALL EDI---------------------------------->取相应位   

00409802   .  8BD0         MOV EDX,EAX------------------------------->取得的字符   

00409804   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

00409807   .  FFD6         CALL ESI   

00409809   .  50           PUSH EAX---------------------------(取得的字符)   

0040980A   .  68 D0374000  PUSH REGAVAX.004037D0---------------------("-")/比较的内容   

0040980F   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

00409815   .  8BD8         MOV EBX,EAX------------------------------------>为零表示相等   

00409817   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

0040981A   .  F7DB         NEG EBX   

0040981C   .  1BDB         SBB EBX,EBX   

0040981E   .  F7DB         NEG EBX   

00409820   .  F7DB         NEG EBX   

00409822   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

00409828   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

0040982B   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409831   .  66:85DB      TEST BX,BX   

00409834   .  0F85 1A070000 JNZ REGAVAX.00409F54------------------------>不为零则跳走返回   

0040983A   .  8B55 D0      MOV EDX,DWORD PTR SS:[EBP-30]   

0040983D   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]--------------->掐头去尾的注册码   

00409840   .  51           PUSH ECX------------------------------------>入栈   

00409841   .  6A 0A        PUSH 0A------------------------------------->从第十位开始   

00409843   .  52           PUSH EDX   

00409844   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

0040984B   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

00409852   .  FFD7         CALL EDI------------------------------------>取相应位   

00409854   .  8BD0         MOV EDX,EAX--------------------------------->保存   

00409856   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

00409859   .  FFD6         CALL ESI   

0040985B   .  50           PUSH EAX-------------------------------(取得的相应位)   

0040985C   .  68 D0374000  PUSH REGAVAX.004037D0---------------------------("-")/比较的内容   

00409861   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

00409867   .  8BD8         MOV EBX,EAX-------------------------------->为零表示相等   

00409869   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

0040986C   .  F7DB         NEG EBX   

0040986E   .  1BDB         SBB EBX,EBX   

00409870   .  F7DB         NEG EBX   

00409872   .  F7DB         NEG EBX   

00409874   .  FF15 AC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr   

0040987A   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]   

0040987D   .  FF15 38104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar   

00409883   .  66:85DB      TEST BX,BX   

00409886   .  0F85 C8060000 JNZ REGAVAX.00409F54------------------------>不为零则跳走返回   

0040988C   .  8B4D D0      MOV ECX,DWORD PTR SS:[EBP-30]--------------->掐头去尾的注册码   

0040988F   .  8D45 B0      LEA EAX,DWORD PTR SS:[EBP-50]   

00409892   .  50           PUSH EAX   

00409893   .  6A 0F        PUSH 0F------------------------------------->第十五位   

00409895   .  51           PUSH ECX------------------------------------>掐头去尾的注册码   

00409896   .  C745 B8 01000>MOV DWORD PTR SS:[EBP-48],1   

0040989D   .  C745 B0 02000>MOV DWORD PTR SS:[EBP-50],2   

004098A4   .  FFD7         CALL EDI------------------------------------>取相应位   

004098A6   .  8BD0         MOV EDX,EAX   

004098A8   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004098AB   .  FFD6         CALL ESI   

004098AD   .  50           PUSH EAX-------------------------(取得的相应位字符)   

004098AE   .  68 D0374000  PUSH REGAVAX.004037D0-------------------------("-")/比较的内容   

004098B3   .  FF15 10114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp   

004098B9   .  8BD8         MOV EBX,EAX--------------------------------->为零表示相等   

004098BB   .  8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   

004098BE   .  F7DB         NEG EBX   

004098C0   .  1BDB         SBB EBX,EBX   

004098C2   .  F7DB         NEG EBX   

004098C4   .  F7DB         NEG EBX   
```
