
# 梦幻邮件制作 v1.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50458.htm`](https://www.pediy.com/kssd/pediy05/pediy50458.htm)

简单算法——梦幻邮件制作 v1.2 ！

软件大小： 1.36M

适用平台: Win9x, WinNT, Win2000

下载页面： [`www.softreg.com.cn/shareware\_view.asp?id=/EFD448C3-92B3-48C4-9B32-CD553CFEB53F/`](http://www.softreg.com.cn/shareware_view.asp?id=/EFD448C3-92B3-48C4-9B32-CD553CFEB53F/)

【软件简介】：梦幻邮件制作用于制作无附件方式的样式邮件，帮助用户进行独特风格的电子邮件制作。梦幻邮件制作可以插入文本框、图像、线条、表格等对象，并进行 MIME 编码，生成不需要任何附件即可在对方计算机上正常显示的电子邮件。用户可以使用本软件进行电子私人信函、商业信函、明信片、贺卡、产品宣传单的制作。梦幻邮件制作内含的模板功能使电子邮件的制作更加方便且富有情趣。

梦幻邮件制作内嵌了一个非常实用的编辑器进行电子邮件的制作，并将这些电子邮件编码导入成可以使用 `OutLook`，`Foxmail` 发送的电子邮件格式。制作好的电子邮件可以在 `Outlook`、`Foxmail`、`Eudora` 等具有解码能力的电子邮件收发软件中显示。

梦幻邮件制作同时提供了网页编译功能，可以将编辑好的电子邮件导出成网页文件。

【软件限制】：30 次试用

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`FI2.5`、`AspackDie`、`W32Dasm8.93` 黄金版

【过 程】：

用户名：`fly`

试炼码：`13572468901234567`

`MMMaker.exe` 是 `ASPACK` 2.11 壳，用 `AspackDie` 脱之。`504K` -> `1.52M`。

反汇编。关键提示都在，省点事。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004BFB88(C)

|

:004BFBFA 8B45FC                 mov eax, dword ptr [ebp-04]
:004BFBFD 8B80E0020000           mov eax, dword ptr [eax+000002E0]
:004BFC03 E83839F7FF             call 00433540
:004BFC08 8D45F8                 lea eax, dword ptr [ebp-08]
:004BFC0B E844F6FEFF             call 004AF254
                                  ==== >关键 CALL！F8 进入！

:004BFC10 84C0                   test al, al
:004BFC12 750F                   jne 004BFC23
                                  ==== >不跳则 OVER！

* Possible StringData Ref from Code Obj -> "无效的注册码！"
                                  ==== >BAD BOY！

:004BFC14 B8C8FD4B00             mov eax, 004BFDC8
:004BFC19 E82295F9FF             call 00459140
:004BFC1E E919010000             jmp 004BFD3C
```

`F8` 进入关键 `CALL`：`4BFC0B call 004AF254`

```
* Referenced by a CALL at Addresses:

|:004BFC0B   , :004C1D71   , :004CEBBA   

|

:004AF254 55                     push ebp
:004AF255 8BEC                   mov ebp, esp
:004AF257 83C4CC                 add esp, FFFFFFCC
:004AF25A 53                     push ebx
:004AF25B 56                     push esi
:004AF25C 57                     push edi
:004AF25D 33D2                   xor edx, edx
:004AF25F 8955CC                 mov dword ptr [ebp-34], edx
:004AF262 8955D0                 mov dword ptr [ebp-30], edx
:004AF265 8955E0                 mov dword ptr [ebp-20], edx
:004AF268 8955F0                 mov dword ptr [ebp-10], edx
:004AF26B 8955F8                 mov dword ptr [ebp-08], edx
:004AF26E 8955F4                 mov dword ptr [ebp-0C], edx
:004AF271 8BF0                   mov esi, eax
:004AF273 33C0                   xor eax, eax
:004AF275 55                     push ebp
:004AF276 68B6F44A00             push 004AF4B6
:004AF27B 64FF30                 push dword ptr fs:[eax]
:004AF27E 648920                 mov dword ptr fs:[eax], esp
:004AF281 C645FF00               mov [ebp-01], 00
:004AF285 33FF                   xor edi, edi
:004AF287 33C0                   xor eax, eax
:004AF289 55                     push ebp
:004AF28A 686CF44A00             push 004AF46C
:004AF28F 64FF30                 push dword ptr fs:[eax]
:004AF292 648920                 mov dword ptr fs:[eax], esp
:004AF295 8D45EC                 lea eax, dword ptr [ebp-14]
:004AF298 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF29A 8A5209                 mov dl, byte ptr [edx+09]
                                  ==== >取试炼码的第 10 位！DL=0

:004AF29D 885001                 mov byte ptr [eax+01], dl
:004AF2A0 C60001                 mov byte ptr [eax], 01
:004AF2A3 8D55EC                 lea edx, dword ptr [ebp-14]
:004AF2A6 8D45E8                 lea eax, dword ptr [ebp-18]
:004AF2A9 E88A37F5FF             call 00402A38
:004AF2AE 8D45E4                 lea eax, dword ptr [ebp-1C]
:004AF2B1 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF2B3 8A5206                 mov dl, byte ptr [edx+06]
                                  ==== >取试炼码的第 7 位！DL=6

:004AF2B6 885001                 mov byte ptr [eax+01], dl
:004AF2B9 C60001                 mov byte ptr [eax], 01
:004AF2BC 8D55E4                 lea edx, dword ptr [ebp-1C]
:004AF2BF 8D45E8                 lea eax, dword ptr [ebp-18]
:004AF2C2 B102                   mov cl, 02
:004AF2C4 E83F37F5FF             call 00402A08
:004AF2C9 8D55E8                 lea edx, dword ptr [ebp-18]
:004AF2CC 8D45F0                 lea eax, dword ptr [ebp-10]
:004AF2CF E8584BF5FF             call 00403E2C
:004AF2D4 8B45F0                 mov eax, dword ptr [ebp-10]
:004AF2D7 E8C09DF5FF             call 0040909C
:004AF2DC 8BD8                   mov ebx, eax
                                  ==== >把上面所取的 2 个数连接起来：06
                                  并取其 HEX 值 06 入 EBX，以备下面相乘！

:004AF2DE 8D45EC                 lea eax, dword ptr [ebp-14]
:004AF2E1 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF2E3 8A520D                 mov dl, byte ptr [edx+0D]
                                  ==== >取试炼码的第 14 位！DL=4

:004AF2E6 885001                 mov byte ptr [eax+01], dl
:004AF2E9 C60001                 mov byte ptr [eax], 01
:004AF2EC 8D55EC                 lea edx, dword ptr [ebp-14]
:004AF2EF 8D45E8                 lea eax, dword ptr [ebp-18]
:004AF2F2 E84137F5FF             call 00402A38
:004AF2F7 8D45E4                 lea eax, dword ptr [ebp-1C]
:004AF2FA 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF2FC 8A5207                 mov dl, byte ptr [edx+07]
                                  ==== >取试炼码的第 8 位！DL=8

:004AF2FF 885001                 mov byte ptr [eax+01], dl
:004AF302 C60001                 mov byte ptr [eax], 01
:004AF305 8D55E4                 lea edx, dword ptr [ebp-1C]
:004AF308 8D45E8                 lea eax, dword ptr [ebp-18]
:004AF30B B102                   mov cl, 02
:004AF30D E8F636F5FF             call 00402A08
:004AF312 8D55E8                 lea edx, dword ptr [ebp-18]
:004AF315 8D45DC                 lea eax, dword ptr [ebp-24]
:004AF318 E81B37F5FF             call 00402A38
:004AF31D 8D45E4                 lea eax, dword ptr [ebp-1C]
:004AF320 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF322 8A5210                 mov dl, byte ptr [edx+10]
                                  ==== >取试炼码的第 17 位！7

:004AF325 885001                 mov byte ptr [eax+01], dl
:004AF328 C60001                 mov byte ptr [eax], 01
:004AF32B 8D55E4                 lea edx, dword ptr [ebp-1C]
:004AF32E 8D45DC                 lea eax, dword ptr [ebp-24]
:004AF331 B103                   mov cl, 03
:004AF333 E8D036F5FF             call 00402A08
:004AF338 8D55DC                 lea edx, dword ptr [ebp-24]
:004AF33B 8D45D4                 lea eax, dword ptr [ebp-2C]
:004AF33E E8F536F5FF             call 00402A38
:004AF343 8D45E4                 lea eax, dword ptr [ebp-1C]
:004AF346 8B16                   mov edx, dword ptr [esi]
                                  ==== >EDX=13572468901234567

:004AF348 8A5203                 mov dl, byte ptr [edx+03]
                                  ==== >取试炼码的第 4 位！7
```



这是一个汇编代码分析片段，主要涉及试炼码验证逻辑。

## 代码段 1：取试炼码位

```
:004AF34B 885001                  mov byte ptr [eax+01], dl  

:004AF34E C60001                  mov byte ptr [eax], 01  

:004AF351 8D55E4                  lea edx, dword ptr [ebp-1C]  

:004AF354 8D45D4                  lea eax, dword ptr [ebp-2C]  

:004AF357 B104                    mov cl, 04  

:004AF359 E8AA36F5FF              call 00402A08  

:004AF35E 8D55D4                  lea edx, dword ptr [ebp-2C]  

                                 ====>EDX=4877   
```

呵呵，上面所取的试炼码的第 `14` 位、`8` 位、`17` 位、`4` 位连接起来=`4877`

## 代码段 2：关键 CALL 与数字检查

```
:004AF361 8D45E0                  lea eax, dword ptr [ebp-20]  

:004AF364 E8C34AF5FF              call 00403E2C  

:004AF369 8B45E0                  mov eax, dword ptr [ebp-20]  

                                 ====>EAX=4877

:004AF36C E82B9DF5FF              call 0040909C  

                                 ====>关键 CALL  

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  

F8 进入 4AF36C  call 0040909C

:0040909C 53                      push ebx  

:0040909D 56                      push esi  

:0040909E 83C4F4                  add esp, FFFFFFF4  

:004090A1 8BD8                    mov ebx, eax  

:004090A3 8BD4                    mov edx, esp  

:004090A5 8BC3                    mov eax, ebx  

:004090A7 E8EC9AFFFF              call 00402B98  

                                 ====>检测上面取的值是否是数字？  

                                 并把 4877 转换成 HEX 值：130D

:004090AC 8BF0                    mov esi, eax  

:004090AE 833C2400                cmp dword ptr [esp], 00000000  

:004090B2 7419                    je 004090CD  

                                 ====>不跳则 OVER！

:004090B4 895C2404                mov dword ptr [esp+04], ebx  

:004090B8 C64424080B              mov [esp+08], 0B  

:004090BD 8D542404                lea edx, dword ptr [esp+04]  

:004090C1 A134234D00              mov eax, dword ptr [004D2334]  

:004090C6 33C9                    xor ecx, ecx  

:004090C8 E8BFFAFFFF              call 00408B8C  

                                 ====>BAD BOY！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004090B2(C)  

|  

:004090CD 8BC6                    mov eax, esi  

:004090CF 83C40C                  add esp, 0000000C  

:004090D2 5E                      pop esi  

:004090D3 5B                      pop ebx  

:004090D4 C3                      ret  

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
```

## 代码段 3：计算与位数补齐

```
:004AF371 0FAFD8                  imul ebx, eax  

                                 ====>06*130D=0000724E  

             即：上面 4AF2DC 处与 4AF369 处所得出的值相乘！

:004AF374 8BC3                    mov eax, ebx  

:004AF376 8D55F8                  lea edx, dword ptr [ebp-08]  

:004AF379 E87E9CF5FF              call 00408FFC  

:004AF37E 8B45F8                  mov eax, dword ptr [ebp-08]  

                                 ====>724E（H）=29262（D） 入 EAX

:004AF381 E8024BF5FF              call 00403E88  

                                 ====>求 29262 的位数

:004AF386 8BD8                    mov ebx, eax  

                                 ====>EBX=5  

:004AF388 83FB05                  cmp ebx, 00000005  

:004AF38B 7F13                    jg 004AF3A0  

                                 ====>如果上面所得的数大于 5 位，则此处跳去比较。  

否则不跳，到下面的 4AF395 处补 F。若还不够 6 位，则继续补数直至 6 位！我上面得出 5 位，因此只补 F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004AF39E(C)  

|  

:004AF38D 8D45F8                  lea eax, dword ptr [ebp-08]  

:004AF390 BAD0F44A00              mov edx, 004AF4D0  

:004AF395 E8F64AF5FF              call 00403E90  

                                 ====>在 29262 后接上 F =29262F

:004AF39A 43                      inc ebx  

:004AF39B 83FB06                  cmp ebx, 00000006  

:004AF39E 75ED                    jne 004AF38D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004AF38B(C)  

|  
```

## 代码段 4：逐位比较验证

```
:004AF3A0 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3A2 8A400C                  mov al, byte ptr [eax+0C]  

                                 ====>取试炼码第 13 位 3 入 AL

:004AF3A5 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3A8 3A4205                  cmp al, byte ptr [edx+05]  

                                 ====>第 13 位 3 与 29262F 第 6 位 F 比较！

:004AF3AB 7540                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3AD 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3AF 8A00                    mov al, byte ptr [eax]  

                                 ====>取试炼码第 1 位 1 入 AL

:004AF3B1 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3B4 3A4203                  cmp al, byte ptr [edx+03]  

                                 ====>第 1 位 1 与 29262F 第 4 位 6 比较！

:004AF3B7 7534                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3B9 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3BB 8A400B                  mov al, byte ptr [eax+0B]  

                                 ====>取试炼码第 12 位 2 入 AL

:004AF3BE 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3C1 3A4202                  cmp al, byte ptr [edx+02]  

                                 ====>第 12 位 2 与 29262F 第 3 位 2 比较！

:004AF3C4 7527                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3C6 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3C8 8A400F                  mov al, byte ptr [eax+0F]  

                                 ====>取试炼码第 16 位 6 入 AL

:004AF3CB 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3CE 3A4201                  cmp al, byte ptr [edx+01]  

                                 ====>第 16 位 6 与 29262F 第 2 位 9 比较！

:004AF3D1 751A                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3D3 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3D5 8A4001                  mov al, byte ptr [eax+01]  

                                 ====>取试炼码第 2 位 3 入 AL

:004AF3D8 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3DB 3A4204                  cmp al, byte ptr [edx+04]  

                                 ====>第 2 位 3 与 29262F 第 5 位 2 比较！

:004AF3DE 750D                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3E0 8B06                    mov eax, dword ptr [esi]  

                                 ====>EAX=13572468901234567

:004AF3E2 8A4008                  mov al, byte ptr [eax+08]  

                                 ====>取试炼码第 9 位 9 入 AL

:004AF3E5 8B55F8                  mov edx, dword ptr [ebp-08]  

                                 ====>29262F 入 EDX

:004AF3E8 3A02                    cmp al, byte ptr [edx]  

                                 ====>第 9 位 9 与 29262F 第 1 位 2 比较！

:004AF3EA 7501                    jne 004AF3ED  

                                 ====>跳则 OVER！

:004AF3EC 47                      inc edi  

                                 ====>EDI 增 1=1

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:004AF3AB(C), :004AF3B7(C), :004AF3C4(C), :004AF3D1(C), :004AF3DE(C)  

|:004AF3EA(C)  

|  

:004AF3ED 8D45D0                  lea eax, dword ptr [ebp-30]  

:004AF3F0 8B16                    mov edx, dword ptr [esi]  

                                 ====>EDX=13572468901234567
```



# 汇编代码分析

```
:004AF3F2 8A5202                  mov dl, byte ptr [edx+02]
                                  ==== > 取试炼码的第 3 位！DL=5

:004AF3F5 E8B649F5FF              call 00403DB0

:004AF3FA 8B45D0                  mov eax, dword ptr [ebp-30]

:004AF3FD E89A9CF5FF              call 0040909C

:004AF402 8BD8                    mov ebx, eax
                                  ==== > 5 入 EBX

:004AF404 8D45CC                  lea eax, dword ptr [ebp-34]

:004AF407 8B16                    mov edx, dword ptr [esi]
                                  ==== > EDX=13572468901234567

:004AF409 8A520E                  mov dl, byte ptr [edx+0E]
                                  ==== > 取试炼码的第 15 位！DL=5

:004AF40C E89F49F5FF              call 00403DB0

:004AF411 8B45CC                  mov eax, dword ptr [ebp-34]

:004AF414 E8839CF5FF              call 0040909C

:004AF419 0FAFD8                  imul ebx, eax
                                  ==== > EBX=5*5=19（H）
                                  即：上面所取试炼码的第 3 位与试炼码的第 15 位相乘！

:004AF41C 8BC3                    mov eax, ebx

:004AF41E 8D55F4                  lea edx, dword ptr [ebp-0C]

:004AF421 E8D69BF5FF              call 00408FFC

:004AF426 8B45F4                  mov eax, dword ptr [ebp-0C]
                                  ==== > EAX=25 即：19（H）的十进制值

:004AF429 E85A4AF5FF              call 00403E88

:004AF42E 8BD8                    mov ebx, eax

:004AF430 83FB01                  cmp ebx, 00000001

:004AF433 7F13                    jg 004AF448

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF446(C)
|

:004AF435 8D45F4                  lea eax, dword ptr [ebp-0C]

:004AF438 BAD0F44A00              mov edx, 004AF4D0

:004AF43D E84E4AF5FF              call 00403E90

:004AF442 43                      inc ebx

:004AF443 83FB02                  cmp ebx, 00000002

:004AF446 75ED                    jne 004AF435

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF433(C)
|

:004AF448 8B06                    mov eax, dword ptr [esi]
                                  ==== > EAX=13572468901234567

:004AF44A 8A4004                  mov al, byte ptr [eax+04]
                                  ==== > 取试炼码第 5 位 2 入 AL

:004AF44D 8B55F4                  mov edx, dword ptr [ebp-0C]
                                  ==== > EDX=25

:004AF450 3A02                    cmp al, byte ptr [edx]
                                  ==== > 第 5 位 2 与 25 的第一位 2 比较！

:004AF452 750E                    jne 004AF462
                                  ==== > 跳则 OVER！

:004AF454 8B06                    mov eax, dword ptr [esi]
                                  ==== > EAX=13572468901234567

:004AF456 8A400A                  mov al, byte ptr [eax+0A]
                                  ==== > 取试炼码第 11 位 1 入 AL

:004AF459 8B55F4                  mov edx, dword ptr [ebp-0C]

:004AF45C 3A4201                  cmp al, byte ptr [edx+01]
                                  ==== > 第 11 位 1 与 25 的第二位 5 比较！

:004AF45F 7501                    jne 004AF462
                                  ==== > 跳则 OVER！

:004AF461 47                      inc edi
                                  ==== > EDI 增 1=2

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004AF452(C), :004AF45F(C)
|

:004AF462 33C0                    xor eax, eax

:004AF464 5A                      pop edx

:004AF465 59                      pop ecx

:004AF466 59                      pop ecx

:004AF467 648910                  mov dword ptr fs:[eax], edx

:004AF46A EB11                    jmp 004AF47D

:004AF46C E9F73EF5FF              jmp 00403368

:004AF471 E84E42F5FF              call 004036C4

:004AF476 EB0E                    jmp 004AF486

:004AF478 E84742F5FF              call 004036C4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF46A(U)
|

:004AF47D 83FF02                  cmp edi, 00000002
                                  ==== > 比较 EDI 是否=2？

:004AF480 7C04                    jl 004AF486
                                  ==== > 跳则 OVER！

:004AF482 C645FF01                mov [ebp-01], 01
                                  ==== > 1 入 [ebp-01]  注册标志位置 1！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004AF476(U), :004AF480(C)
|

:004AF486 33C0                    xor eax, eax

:004AF488 5A                      pop edx

:004AF489 59                      pop ecx

:004AF48A 59                      pop ecx

:004AF48B 648910                  mov dword ptr fs:[eax], edx

:004AF48E 68BDF44A00              push 004AF4BD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF4BB(U)
|

:004AF493 8D45CC                  lea eax, dword ptr [ebp-34]

:004AF496 BA02000000              mov edx, 00000002

:004AF49B E88C47F5FF              call 00403C2C

:004AF4A0 8D45E0                  lea eax, dword ptr [ebp-20]

:004AF4A3 E86047F5FF              call 00403C08

:004AF4A8 8D45F0                  lea eax, dword ptr [ebp-10]

:004AF4AB BA03000000              mov edx, 00000003

:004AF4B0 E87747F5FF              call 00403C2C

:004AF4B5 C3                      ret

:004AF4B6 E96141F5FF              jmp 0040361C

:004AF4BB EBD6                    jmp 004AF493

:004AF4BD 8A45FF                  mov al, byte ptr [ebp-01]
                                  ==== > [ebp-01] 入 AL  注册标志位！

:004AF4C0 5F                      pop edi

:004AF4C1 5E                      pop esi

:004AF4C2 5B                      pop ebx

:004AF4C3 8BE5                    mov esp, ebp

:004AF4C5 5D                      pop ebp

:004AF4C6 C3                      ret
```

## 算法总结

1. 注册码至少 `17` 位！多于 `17` 位则只取前 `17` 位运算。试炼码：`13572468901234567`
2. 试炼码的第 `10` 位 `0`、第 `7` 位 `6`，连接起来=`06`，取其 `16` 进制值=`06`
3. 取试炼码的第 `14` 位 `4`、第 `8` 位 `8`、第 `17` 位 `7`、第 `4` 位 `7`，连接起来=`4877`，取其 `16` 进制值=`130D`
4. 上面两次所得结果相乘：`06*130D=724E`   转换成 `10` 进制值=`29262`
5. `29262` 少于 `5` 位，因此其后再接上 `F` =`29262F`  （若上面所得多于 `5` 位则不接）
6. 试炼码第 `13` 位 `3` 与 `29262F` 第 `6` 位 `F` 比较！所以：注册码第 `13` 位=`F`
7. 试炼码第 `1` 位 `1` 与 `29262F` 第 `4` 位 `6` 比较！所以：注册码第 `1` 位=`6`
8. 试炼码第 `12` 位 `2` 与 `29262F` 第 `3` 位 `2` 比较！所以：注册码第 `12` 位=`2`
9. 试炼码第 `16` 位 `6` 与 `29262F` 第 `2` 位 `9` 比较！所以：注册码第 `16` 位=`9`
10. 试炼码第 `2` 位 `3` 与 `29262F` 第 `5` 位 `2` 比较！所以：注册码第 `2` 位=`2`
11. 试炼码第 `9` 位 `9` 与 `29262F` 第 `1` 位 `2` 比较！所以：注册码第 `9` 位=`2`
12. 取试炼码的第 `3` 位 `5` 与第 `15` 位 `5` 相乘=`19`（`H`）=`25`（`D`）
13. 试炼码第 `5` 位 `2` 与 `25` 的第一位 `2` 比较！所以：注册码第 `5` 位=`2`
14. 试炼码第 `11` 位 `1` 与 `25` 的第二位 `5` 比较！所以：注册码第 `11` 位=`5`

综上所述：我所输入的试炼码所对应的注册码=`625724682052F4597`

呵呵，注册码与用户名无关！

## 注册信息保存

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\[www.sangao.com\MagicMailMaker\]]
"UserName"="fly"
"KeyCode"="625724682052F4597"
```

## 整理

- 用户名：`fly` （随意）
- 试炼码：`625724682052F4597`  或者：`085726481055F45978`    很多的……

```
                 Cracked By 巢水工作坊——fly【OCN】
                              23:36 03-3-3  4dd466d0e1e0cc123afbe504da2462ca
```
