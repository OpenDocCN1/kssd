
```
:004962CF 64FF30               push dword ptr fs:[eax]   
:004962D2 648920               mov dword ptr fs:[eax], esp   
:004962D5 8D55FC               lea edx, dword ptr [ebp-04]   
:004962D8 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
:004962DE E8F96EF9FF           call 0042D1DC   
:004962E3 8B45FC               mov eax, dword ptr [ebp-04]   
:004962E6 50                   push eax   
:004962E7 8D55F8               lea edx, dword ptr [ebp-08]   
:004962EA 8B83D0020000         mov eax, dword ptr [ebx+000002D0]   
:004962F0 8B8024020000         mov eax, dword ptr [eax+00000224]   
:004962F6 2DD1080000           sub eax, 000008D1   
:004962FB E80829F7FF           call 00408C08   
:00496300 8B55F8               mov edx, dword ptr [ebp-08]   
:00496303 58                   pop eax   
:00496304 E893DEF6FF           call 0040419C   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00496290(C)   
|   
:00496309 757E                 jne 00496389============》在这里必须跳过   
* Possible StringData Ref from Code Obj ->"注册成功!"   
                          |   
:0049630B B864654900           mov eax, 00496564   
:00496310 E8B397FBFF           call 0044FAC8   
:00496315 E8CAFEFFFF           call 004961E4   
:0049631A A180EE4B00           mov eax, dword ptr [004BEE80]   
:0049631F 8B80D4020000         mov eax, dword ptr [eax+000002D4]   
* Possible StringData Ref from Code Obj ->"已经注册!"   
                          |   
:00496325 BA78654900           mov edx, 00496578   
:0049632A E8DD6EF9FF           call 0042D20C   
:0049632F 33D2                 xor edx, edx   
:00496331 8B83E4020000         mov eax, dword ptr [ebx+000002E4]   
:00496337 E8B86DF9FF           call 0042D0F4   
:0049633C 33D2                 xor edx, edx   
:0049633E 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
:00496344 E8AB6DF9FF           call 0042D0F4   
:00496349 33D2                 xor edx, edx   
:0049634B 8B83D8020000         mov eax, dword ptr [ebx+000002D8]   
:00496351 E89E6DF9FF           call 0042D0F4   
* Possible StringData Ref from Code Obj ->"已注册"   
                          |   
:00496356 BA8C654900           mov edx, 0049658C   
:0049635B 8B83E0020000         mov eax, dword ptr [ebx+000002E0]   
:00496361 E8A66EF9FF           call 0042D20C   
:00496366 A14CDF4B00           mov eax, dword ptr [004BDF4C]   
:0049636B 8B00                 mov eax, dword ptr [eax]   
:0049636D 8B805C030000         mov eax, dword ptr [eax+0000035C]   
* Possible StringData Ref from Code Obj ->"      "   
                          |   
:00496373 BA9C654900           mov edx, 0049659C   
:00496378 E88F6EF9FF           call 0042D20C   
:0049637D B263                 mov dl, 63   
:0049637F A188EE4B00           mov eax, dword ptr [004BEE88]   
:00496384 E83327FFFF           call 00488ABC============》在这里调用格式化代码   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00496309(C)   
|   
:00496389 8D55F4               lea edx, dword ptr [ebp-0C]   
:0049638C 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
:00496392 E8456EF9FF           call 0042D1DC   
:00496397 8B45F4               mov eax, dword ptr [ebp-0C]   
:0049639A 50                   push eax   
:0049639B 8D55F0               lea edx, dword ptr [ebp-10]   
:0049639E 8B83D0020000         mov eax, dword ptr [ebx+000002D0]   
:004963A4 8B8024020000         mov eax, dword ptr [eax+00000224]   
:004963AA 2DE9040000           sub eax, 000004E9   
:004963AF E85428F7FF           call 00408C08   
:004963B4 8B55F0               mov edx, dword ptr [ebp-10]   
:004963B7 58                   pop eax   
:004963B8 E8DFDDF6FF           call 0040419C   
:004963BD 757E                 jne 0049643D============》在这里必须跳过   
* Possible StringData Ref from Code Obj ->"注册成功!"   
                          |   
:004963BF B864654900           mov eax, 00496564   
:004963C4 E8FF96FBFF           call 0044FAC8   
:004963C9 E816FEFFFF           call 004961E4   
:004963CE A180EE4B00           mov eax, dword ptr [004BEE80]   
:004963D3 8B80D4020000         mov eax, dword ptr [eax+000002D4]   
* Possible StringData Ref from Code Obj ->"已经注册!"   
                          |   
:004963D9 BA78654900           mov edx, 00496578   
:004963DE E8296EF9FF           call 0042D20C   
:004963E3 33D2                 xor edx, edx   
:004963E5 8B83E4020000         mov eax, dword ptr [ebx+000002E4]   
:004963EB E8046DF9FF           call 0042D0F4   
:004963F0 33D2                 xor edx, edx   
:004963F2 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
:004963F8 E8F76CF9FF           call 0042D0F4   
:004963FD 33D2                 xor edx, edx   
:004963FF 8B83D8020000         mov eax, dword ptr [ebx+000002D8]   
:00496405 E8EA6CF9FF           call 0042D0F4   
* Possible StringData Ref from Code Obj ->"已注册"   
                          |   
:0049640A BA8C654900           mov edx, 0049658C   
:0049640F 8B83E0020000         mov eax, dword ptr [ebx+000002E0]   
:00496415 E8F26DF9FF           call 0042D20C   
:0049641A A14CDF4B00           mov eax, dword ptr [004BDF4C]   
:0049641F 8B00                 mov eax, dword ptr [eax]   
:00496421 8B805C030000         mov eax, dword ptr [eax+0000035C]   
* Possible StringData Ref from Code Obj ->"      "   
                          |   
:00496427 BA9C654900           mov edx, 0049659C   
:0049642C E8DB6DF9FF           call 0042D20C   
:00496431 B263                 mov dl, 63   
:00496433 A188EE4B00           mov eax, dword ptr [004BEE88]   
:00496438 E87F26FFFF           call 00488ABC============》在这里调用格式化代码   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004963BD(C)   
|   
:0049643D 8D55EC               lea edx, dword ptr [ebp-14]   
:00496440 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
:00496446 E8916DF9FF           call 0042D1DC   
:0049644B 8B45EC               mov eax, dword ptr [ebp-14]   
:0049644E 50                   push eax   
:0049644F 8D55E8               lea edx, dword ptr [ebp-18]   
:00496452 8B83D0020000         mov eax, dword ptr [ebx+000002D0]   
:00496458 8B8024020000         mov eax, dword ptr [eax+00000224]   
:0049645E 056D010000           add eax, 0000016D   
:00496463 E8A027F7FF           call 00408C08   
:00496468 8B55E8               mov edx, dword ptr [ebp-18]   
:0049646B 58                   pop eax   
:0049646C E82BDDF6FF           call 0040419C   
:00496471 7574                 jne 004964E7   
* Possible StringData Ref from Code Obj ->"注册成功!"   
                          |   
:00496473 B864654900           mov eax, 00496564   
:00496478 E84B96FBFF           call 0044FAC8   
:0049647D E862FDFFFF           call 004961E4   
:00496482 A180EE4B00           mov eax, dword ptr [004BEE80]   
:00496487 8B80D4020000         mov eax, dword ptr [eax+000002D4]   
* Possible StringData Ref from Code Obj ->"已经注册!"   
                          |   
:0049648D BA78654900           mov edx, 00496578   
:00496492 E8756DF9FF           call 0042D20C   
:00496497 33D2                 xor edx, edx   
:00496499 8B83E4020000         mov eax, dword ptr [ebx+000002E4]   
:0049649F E8506CF9FF           call 0042D0F4   
:004964A4 33D2                 xor edx, edx   
:004964A6 8B83DC020000         mov eax, dword ptr [ebx+000002DC]   
```
