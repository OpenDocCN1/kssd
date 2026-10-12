
## 汇编代码分析

```
:004E5989 8D45DC                 lea eax, dword ptr [ebp-24]  
:004E598C 50                     push eax  
:004E598D 8D55D8                 lea edx, dword ptr [ebp-28]  
:004E5990 8B83FC020000           mov eax, dword ptr [ebx+000002FC]  
:004E5996 E81D2AF6FF             call 004483B8  
:004E599B 8B55D8                 mov edx, dword ptr [ebp-28]  
                                ====>103B2C1E 入 EDX  申请码
* Possible StringData Ref from Code Obj ->"EZWord"  
                                |  
:004E599E B9085D4E00             mov ecx, 004E5D08  
                                ===> EZWord 入 ECX
:004E59A3 33C0                   xor eax, eax  
:004E59A5 E82EF9FFFF             call 004E52D8  
                                ===>运算第二部分！进入！
:004E59AA 8B55DC                 mov edx, dword ptr [ebp-24]  
                                ===>结果：e4e8a4fe 入 EDX
:004E59AD 58                     pop eax  
                                ===>EAX=2468 试炼码的后部分
:004E59AE E8DDEEF1FF             call 00404890  
                                ===>比较第二部分！进入！
:004E59B3 0F854A020000           jne 004E5C03  
                                ===>跳则 OVER！
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
```

1. 进入运算部分！`4E595E` `call 004E52D8`
2. 进入运算部分！`4E59A5` `call 004E52D8`

* Referenced by a CALL at Addresses:

```
:004E595E   , :004E59A5   , :004E59F6   , :004E5A14   , :004E5A46   
:004E5A64   , :004E5A96   , :004E5AB4   , :004E5AE6   , :004E5B04   
:004E8DD8   , :004E8E11   , :004E8E4A   , :004E8E83   
```

```
:004E52D8 55                     push ebp  
:004E52D9 8BEC                   mov ebp, esp  
:004E52DB 83C4EC                 add esp, FFFFFFEC  
:004E52DE 53                     push ebx  
:004E52DF 33DB                   xor ebx, ebx  
:004E52E1 895DEC                 mov dword ptr [ebp-14], ebx  
:004E52E4 895DF0                 mov dword ptr [ebp-10], ebx  
:004E52E7 894DF4                 mov dword ptr [ebp-0C], ecx  
:004E52EA 8955F8                 mov dword ptr [ebp-08], edx  
:004E52ED 8945FC                 mov dword ptr [ebp-04], eax  
:004E52F0 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E52F3 E83CF6F1FF             call 00404934  
:004E52F8 8B45F8                 mov eax, dword ptr [ebp-08]  
:004E52FB E834F6F1FF             call 00404934  
:004E5300 8B45F4                 mov eax, dword ptr [ebp-0C]  
:004E5303 E82CF6F1FF             call 00404934  
:004E5308 8B450C                 mov eax, dword ptr [ebp+0C]  
:004E530B E824F6F1FF             call 00404934  
:004E5310 33C0                   xor eax, eax  
:004E5312 55                     push ebp  
:004E5313 687E534E00             push 004E537E  
:004E5318 64FF30                 push dword ptr fs:[eax]  
:004E531B 648920                 mov dword ptr fs:[eax], esp  
:004E531E FF75FC                 push [ebp-04]  
:004E5321 FF75F8                 push [ebp-08]  
:004E5324 FF75F4                 push [ebp-0C]  
:004E5327 FF750C                 push [ebp+0C]  
:004E532A 8B450C                 mov eax, dword ptr [ebp+0C]  
:004E532D 50                     push eax  
:004E532E 8D45EC                 lea eax, dword ptr [ebp-14]  
:004E5331 50                     push eax  
:004E5332 8B4DF4                 mov ecx, dword ptr [ebp-0C]  
:004E5335 8B55F8                 mov edx, dword ptr [ebp-08]  
:004E5338 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E533B E840FDFFFF             call 004E5080
:004E5340 FF75EC                 push [ebp-14]  
    运算第一部分！               ===>U@pg%,p`       程序自给  
    运算第二部分！               ===>npnczwFj       程序自给
:004E5343 8D45F0                 lea eax, dword ptr [ebp-10]  
:004E5346 BA05000000             mov edx, 00000005  
:004E534B E8BCF4F1FF             call 0040480C  
    运算第一部分！     ===>此 CALL 把用户名、申请码以及上面的 U@pg%,p`连接起来  
    运算第二部分！     ===>此 CALL 把申请码以及上面的 npnczwFj 连接起来
:004E5350 8B5508                 mov edx, dword ptr [ebp+08]  
:004E5353 8B45F0                 mov eax, dword ptr [ebp-10]  
    运算第一部分！     ===>fly103B2C1EEZWord1U@pg%,p`   入 EAX  
    运算第二部分！     ===>103B2C1EEZWord1npnczwFj       入 EAX       
:004E5356 E831000000             call 004E538C  
                                ===>运算了！进入！
:004E535B 33C0                   xor eax, eax  
:004E535D 5A                     pop edx  
:004E535E 59                     pop ecx  
:004E535F 59                     pop ecx  
:004E5360 648910                 mov dword ptr fs:[eax], edx  
:004E5363 6885534E00             push 004E5385
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:004E5383(U)  
:004E5368 8D45EC                 lea eax, dword ptr [ebp-14]  
:004E536B BA05000000             mov edx, 00000005  
:004E5370 E833F1F1FF             call 004044A8  
:004E5375 8D450C                 lea eax, dword ptr [ebp+0C]  
:004E5378 E807F1F1FF             call 00404484  
:004E537D C3                     ret
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
```

进入运算 CALL：`4E5356` `call 004E538C`

* Referenced by a CALL at Address:
`:004E5356`

```
:004E538C 55                     push ebp  
:004E538D 8BEC                   mov ebp, esp  
:004E538F 83C4F4                 add esp, FFFFFFF4  
:004E5392 53                     push ebx  
:004E5393 56                     push esi  
:004E5394 33C9                   xor ecx, ecx  
:004E5396 894DF4                 mov dword ptr [ebp-0C], ecx  
:004E5399 8955F8                 mov dword ptr [ebp-08], edx  
:004E539C 8945FC                 mov dword ptr [ebp-04], eax  
:004E539F 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E53A2 E88DF5F1FF             call 00404934  
:004E53A7 33C0                   xor eax, eax  
:004E53A9 55                     push ebp  
:004E53AA 682B544E00             push 004E542B  
:004E53AF 64FF30                 push dword ptr fs:[eax]  
:004E53B2 648920                 mov dword ptr fs:[eax], esp  
:004E53B5 33DB                   xor ebx, ebx  
:004E53B7 8B45FC                 mov eax, dword ptr [ebp-04]  
    运算第一部分！       ===>fly103B2C1EEZWord1U@pg%,p` 入 EAX  
    运算第二部分！       ===>103B2C1EEZWord1npnczwFj    入 EAX
:004E53BA E88DF3F1FF             call 0040474C  
                         ===>求上面字符串的长度  
:004E53BF 85C0                   test eax, eax  
    运算第一部分！       ===>EAX=1A （即：26 位）  
    运算第二部分！       ===>EAX=17 （即：23 位）
:004E53C1 7E2C                   jle 004E53EF
:004E53C3 BE01000000             mov esi, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:004E53ED(C)  
:004E53C8 8B55FC                 mov edx, dword ptr [ebp-04]  
      运算第一部分！    ===>fly103B2C1EEZWord1U@pg%,p` 入 EDX  
      运算第二部分！    ===>103B2C1EEZWord1npnczwFj    入 EDX
:004E53CB 8A5432FF               mov dl, byte ptr [edx+esi-01]  
    运算第一部分！  ===>逐位从 fly103B2C1EEZWord1U@pg%,p`中取字符的 HEX 值！  
                         ===>1、DL=66  
                ……  …… 省略（共 26 次） …… ……
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    运算第二部分！      ===>逐位从 103B2C1EEZWord1npnczwFj 中取字符的 HEX 值！  
                         ===>1、DL=31  
                ……  …… 省略（共 23 次） …… ……
:004E53CF 32D3                   xor dl, bl  
    运算第一部分！      ===>与 BL 进行异或！ 第一次 BL=0  
                         ===>1、DL=66 XOR 0=66  
                ……  …… 省略（共 26 次） …… ……
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    运算第二部分！      ===>与 BL 进行异或！ 第一次 BL=0  
                         ===>1、DL=31 XOR 0=31  
                ……  …… 省略（共 23 次） …… ……
:004E53D1 81E2FF000000           and edx, 000000FF  
:004E53D7 8B1495A4BC4E00         mov edx, dword ptr [4*edx+004EBCA4]  
```
