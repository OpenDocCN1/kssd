
```
:004143AB E8F0020100              call 004246A0

:004143B0 80257892470000          and byte ptr [00479278],

00

:004143B7 59
  pop ecx
:004143B8 59
  pop ecx
:004143B9 BE78924700              mov esi,
00479278
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004143D3(C)
|
:004143BE 6A0F
  push 0000000F
:004143C0 E8CA020000              call 0041468F
:004143C5 59
  pop ecx
:004143C6 50
  push eax
:004143C7 56
  push esi
:004143C8 57
  push edi
:004143C9 56
  push esi
:004143CA E8F4120100              call 004256C3
:004143CF 83C410
add esp, 00000010
:004143D2 4B
  dec ebx
:004143D3 75E9
  jne 004143BE
:004143D5 A078924700              mov al,
byte ptr [00479278]
:004143DA 33D2
  xor edx, edx
:004143DC 84C0
  test al, al
:004143DE 741B
  je 004143FB
:004143E0 8BCE
  mov ecx, esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004143F9(C)
|
:004143E2 3C20
  cmp al, 20
:004143E4 740D
  je 004143F3
:004143E6 0FBE01
movsx eax, byte ptr [ecx]
:004143E9 03C2
  add eax, edx
:004143EB BFFF000000              mov edi,
000000FF
:004143F0 99
  cdq
:004143F1 F7FF
  idiv edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004143E4(C)
|
:004143F3 8A4101
mov al, byte ptr [ecx+01]
:004143F6 41
  inc ecx
:004143F7 84C0
  test al, al
:004143F9 75E7
  jne 004143E2
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004143DE(C)
|
:004143FB 8BC2
  mov eax, edx
:004143FD 6A63
  push 00000063
:004143FF 99
  cdq
:00414400 59
  pop ecx
:00414401 F7F9
  idiv ecx
:00414403 52
  push edx
:00414404 56
  push esi
* Possible Reference to Dialog:
            |
:00414405 68581F4700              push 00471F58
:0041440A 56
  push esi
:0041440B E8B3120100              call 004256C3
:00414410 6A28
  push 00000028
:00414412 E834020000              call 0041464B
:00414417 83C414
add esp, 00000014
:0041441A 33F6
  xor esi, esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00414431(C)
|
:0041441C 6A0F
  push 0000000F
:0041441E E86C020000              call 0041468F
:00414423 3086B0914700            xor byte ptr
[esi+004791B0], al
:00414429 46
  inc esi
:0041442A 81FEC8000000            cmp esi, 000000C8
:00414430 59
  pop ecx
:00414431 72E9
  jb 0041441C
:00414433 E857FDFFFF              call 0041418F
:00414438 8BF0
  mov esi, eax
:0041443A 8D45A0
lea eax, dword ptr [ebp-60]
:0041443D 50
  push eax
:0041443E FF750C
push [ebp+0C]
:00414441 E8E9F8FEFF              call 00403D2F
:00414446 59
  pop ecx
:00414447 59
  pop ecx
:00414448 EB02
  jmp 0041444C
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00414255(C), :004142B2(C), :00414389(C)
|
:0041444A 33F6
  xor esi, esi  <---esi清为0了,注意有三个地方会跳到这里来
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00414448(U)
|
:0041444C 8BC6
  mov eax, esi  <---如果跳到上面一行,这时eax也为0了,下面就跳出这个Call了,还记得外面那个test
eax,eax吗所以无论如何都不能让程序跳到41444A这一行
:0041444E 5F
  pop edi
:0041444F 5E
  pop esi
:00414450 5B
  pop ebx
:00414451 C9
  leave
:00414452 C3
  ret
```

后记:

这次破解我不想写太多东西,因为我并没有将它的算法搞清楚,它的很多`call`都在拿我的假资料做运算,不过都是些乘法除法之类的,请高手们看看吧.将它破解后我上到`http://astalavista.box.sk/`去查找它的注册机,但找到的都是注册了的`Gerces23.dru`文件,我想它应该不是那么简单的,不然怎么会连个正确的注册码都找不到呢~~~~~~~~~~呵呵

      \_/\_/\_/

    \_/          \_/\_/\_/  \_/\_/\_/

\_/\_/

    \_/\_/    \_/    \_/  \_/    \_/

  \_/

        \_/  \_/    \_/  \_/

\_/    \_/

\_/\_/\_/      \_/\_/\_/  \_/    \_/    \_/

        Sam.com

    2:55 2001-7-20
