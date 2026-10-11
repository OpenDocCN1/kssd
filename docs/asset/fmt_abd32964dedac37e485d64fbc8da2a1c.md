
:0048B7B5 56  push esi  

:0048B7B6 50  push eax

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h  

:0048B7B7 FF15A4104000  Call `dword ptr [004010A4]`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`:0048B77B(C)`, `:0048B7A9(C)`  

:0048B7BD 8B13  mov `edx`, `dword ptr [ebx]`  

:0048B7BF 53  push `ebx`  

:0048B7C0 FF9234030000  call `dword ptr [edx+00000334]`  

:0048B7C6 50  push `eax`  

:0048B7C7 8D459C  lea `eax`, `dword ptr [ebp-64]`  

:0048B7CA 50  push `eax`

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h  

:0048B7CB FF15D0104000  Call `dword ptr [004010D0]`  

:0048B7D1 8BF0  mov `esi`, `eax`  

:0048B7D3 8B0E  mov `ecx`, `dword ptr [esi]`  

:0048B7D5 8D55AC  lea `edx`, `dword ptr [ebp-54]`  

:0048B7D8 52  push `edx`  

:0048B7D9 56  push `esi`  

:0048B7DA FF5150  call `[ecx+50]`  

:0048B7DD DBE2  `fclex`  

:0048B7DF 85C0  test `eax`, `eax`  

:0048B7E1 7D0F  jge `0048B7F2`  

:0048B7E3 6A50  push `00000050`

* Possible StringData Ref from Data Obj -> `"貼?檉??"`  

:0048B7E5 6878F54000  push `0040F578`  

:0048B7EA 56  push `esi`  

:0048B7EB 50  push `eax`

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h  

:0048B7EC FF15A4104000  Call `dword ptr [004010A4]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`:0048B7E1(C)`  

:0048B7F2 8B45AC  mov `eax`, `dword ptr [ebp-54]`  
====> `EAX=121`

:0048B7F5 50  push `eax`

* Reference To: `MSVBVM60.rtcR8ValFromBstr`, Ord:0245h  

:0048B7F6 FF15F0124000  Call `dword ptr [004012F0]`  

:0048B7FC DD9D00FFFFFF  fstp `qword ptr [ebp+FFFFFF00]`  
====> `ST=121.00000000000000000`

:0048B802 8B0B  mov `ecx`, `dword ptr [ebx]`  

:0048B804 53  push `ebx`  

:0048B805 FF9130030000  call `dword ptr [ecx+00000330]`  

:0048B80B 50  push `eax`  

:0048B80C 8D5598  lea `edx`, `dword ptr [ebp-68]`  

:0048B80F 52  push `edx`

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h  

:0048B810 FF15D0104000  Call `dword ptr [004010D0]`  

:0048B816 8BF0  mov `esi`, `eax`  

:0048B818 8B06  mov `eax`, `dword ptr [esi]`  

:0048B81A 8D4DA8  lea `ecx`, `dword ptr [ebp-58]`  

:0048B81D 51  push `ecx`  

:0048B81E 56  push `esi`  

:0048B81F FF5050  call `[eax+50]`  

:0048B822 DBE2  `fclex`  

:0048B824 85C0  test `eax`, `eax`  

:0048B826 7D0F  jge `0048B837`  

:0048B828 6A50  push `00000050`

* Possible StringData Ref from Data Obj -> `"貼?檉??"`  

:0048B82A 6878F54000  push `0040F578`  

:0048B82F 56  push `esi`  

:0048B830 50  push `eax`

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h  

:0048B831 FF15A4104000  Call `dword ptr [004010A4]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`:0048B826(C)`  

:0048B837 8B55A8  mov `edx`, `dword ptr [ebp-58]`  
====> `EDX=83`

:0048B83A 52  push `edx`

* Reference To: `MSVBVM60.rtcR8ValFromBstr`, Ord:0245h  

:0048B83B FF15F0124000  Call `dword ptr [004012F0]`  

:0048B841 DD9DF8FEFFFF  fstp `qword ptr [ebp+FFFFFEF8]`  
====> `ST=83.000000000000000000`

:0048B847 8B03  mov `eax`, `dword ptr [ebx]`  

:0048B849 53  push `ebx`  

:0048B84A FF902C030000  call `dword ptr [eax+0000032C]`  

:0048B850 50  push `eax`  

:0048B851 8D4D94  lea `ecx`, `dword ptr [ebp-6C]`  

:0048B854 51  push `ecx`

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h  

:0048B855 FF15D0104000  Call `dword ptr [004010D0]`  

:0048B85B 8BF0  mov `esi`, `eax`  

:0048B85D 8B16  mov `edx`, `dword ptr [esi]`  

:0048B85F 8D45A4  lea `eax`, `dword ptr [ebp-5C]`  

:0048B862 50  push `eax`  

:0048B863 56  push `esi`  

:0048B864 FF5250  call `[edx+50]`  

:0048B867 DBE2  `fclex`  

:0048B869 85C0  test `eax`, `eax`  

:0048B86B 7D0F  jge `0048B87C`  

:0048B86D 6A50  push `00000050`

* Possible StringData Ref from Data Obj -> `"貼?檉??"`  

:0048B86F 6878F54000  push `0040F578`  

:0048B874 56  push `esi`  

:0048B875 50  push `eax`

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h  

:0048B876 FF15A4104000  Call `dword ptr [004010A4]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`:0048B86B(C)`  

:0048B87C 8B4DA4  mov `ecx`, `dword ptr [ebp-5C]`  
====> `ECX=97`

:0048B87F 51  push `ecx`

* Reference To: `MSVBVM60.rtcR8ValFromBstr`, Ord:0245h  

:0048B880 FF15F0124000  Call `dword ptr [004012F0]`  

:0048B886 DD9DF0FEFFFF  fstp `qword ptr [ebp+FFFFFEF0]`  
====> `ST=97.000000000000000000`

:0048B88C 8B13  mov `edx`, `dword ptr [ebx]`  

:0048B88E 53  push `ebx`  

:0048B88F FF9228030000  call `dword ptr [edx+00000328]`  

:0048B895 50  push `eax`  

:0048B896 8D4590  lea `eax`, `dword ptr [ebp-70]`  

:0048B899 50  push `eax`

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h  

:0048B89A FF15D0104000  Call `dword ptr [004010D0]`  

:0048B8A0 8BF0  mov `esi`, `eax`  

:0048B8A2 8B0E  mov `ecx`, `dword ptr [esi]`  

:0048B8A4 8D55A0  lea `edx`, `dword ptr [ebp-60]`  

:0048B8A7 52  push `edx`  

:0048B8A8 56  push `esi`  

:0048B8A9 FF5150  call `[ecx+50]`  

:0048B8AC DBE2  `fclex`  

:0048B8AE 85C0  test `eax`, `eax`  

:0048B8B0 7D0F  jge `0048B8C1`  

:0048B8B2 6A50  push `00000050`

* Possible StringData Ref from Data Obj -> `"貼?檉??"`  

:0048B8B4 6878F54000  push `0040F578`  

:0048B8B9 56  push `esi`  

:0048B8BA 50  push `eax`

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h  

:0048B8BB FF15A4104000  Call `dword ptr [004010A4]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`:0048B8B0(C)`  

:0048B8C1 8B45A0  mov `eax`, `dword ptr [ebp-60]`  
====> `EAX=1004365879` 程序自给？！

:0048B8C4 50  push `eax`

###############晕——下面这些浮点运算难道又是“冗余代码”？！###################
