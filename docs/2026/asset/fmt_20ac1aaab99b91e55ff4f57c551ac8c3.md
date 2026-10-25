
# TextAloudMp3

> 标题：TextAloudMp3  
> 发信人：ShenGe  
> 时间：2003/08/01 00:08am

## 详细信息

- **软件大小**：4525KB
- **软件语言**：英文
- **软件类别**：国外软件/共享版/音频工具
- **运行环境**：Win9x/Me/NT/2000/XP
- **加入时间**：2003-7-30 17:30:44
- **下载地址**：http://www.getafile.com/cgi-bin/merlot/get/nextup/TxAl1452.exe
- **软件评级**：☆☆☆☆

## 软件介绍

TextAloud MP3 是一个相当新颖的应用程序，它可以转换任何应用程序中的文字成为声音或 MP3 文件（很可惜的，目前并不支持中文字的发声）。平常是常驻在 Windows 系统列中，随时等待使用者利用它来读取 Email、Web page 或其它文件中的文字。TextAloud MP3 是利用监视剪贴簿的方式来抓取所要发声的文字（所以使用者可以直接利用复制文字的方式来启动 TextAloud MP3），而使用者可以决定是要马上聆听，还是转存成 MP3、WAV 文件，等到有空闲时再利用喜欢的 MP3 播放程序来收听。除了 TextAloud MP3 内附的声音之外，使用者还可以到作者网页去下载其它的声音，让使用者可以随自己的喜好来变换发声的人喔！

## 破解工具

- `TRW1.22娃娃修改版`
- `OllyDbgV1.09`
- `WdasmV10.0`
- `Guw`

## 作者声明

初学破解，仅作学习交流之用，失误之处敬请大侠赐教。

## 简要过程

试验码：2234567865148455

好久没来论坛了，发觉鲜有破文，大概大家都太忙了，抽空找了两个简单的软件，把过程放上来顶顶人气，高手莫见笑！

用 `Pe-scan` 检测是 `aspack2.12` 的壳，用 `pe-scan` 的脱壳功能脱壳成功，但是不能运行！请求 `GUW` 搞定，621K-->1578K。照例是用 `TRW` 找断点（`bpx hmemcpy`），`OD` 载入分析！

..........（略）

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497BDE(C)
|
:00497C23 8D55FC                  lea edx, dword ptr [ebp-04]
:00497C26 8B835C030000            mov eax, dword ptr [ebx+0000035C]
:00497C2C E82BF5F9FF              call 0043715C
:00497C31 689999F13F              push 3FF19999
:00497C36 689A999999              push 9999999A
:00497C3B 8B55FC                  mov edx, dword ptr [ebp-04]

           <===edx="2234567865148455",假码!

:00497C3E 8B837C030000            mov eax, dword ptr [ebx+0000037C]
:00497C44 E803E6FFFF              call 0049624C

           <===关键的Call,追算法就跟进吧!

:00497C49 3C01                    cmp al, 01
:00497C4B 7543                    jne 00497C90

\* Possible StringData Ref from Code Obj ->"Thank you for your purchase."
           |
:00497C4D B80C7D4900              mov eax, 00497D0C
:00497C52 E8714AFCFF              call 0045C6C8

           <===是不是很有成就感!

:00497C57 8B837C030000            mov eax, dword ptr [ebx+0000037C]
:00497C5D 8B10                    mov edx, dword ptr [eax]
:00497C5F FF12                    call dword ptr [edx]
:00497C61 84C0                    test al, al
:00497C63 7409                    je 00497C6E
:00497C65 B201                    mov dl, 01
:00497C67 8BC3                    mov eax, ebx
:00497C69 E8F2F3FFFF              call 00497060

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497C63(C)
|
:00497C6E 8B45FC                  mov eax, dword ptr [ebp-04]

\* Possible StringData Ref from Code Obj ->"1234123412341234"
           |
:00497C71 BA347D4900              mov edx, 00497D34
:00497C76 E815C6F6FF              call 00404290
:00497C7B 751D                    jne 00497C9A
:00497C7D C7833402000001000000    mov dword ptr [ebx+00000234], 00000001
:00497C87 8BC3                    mov eax, ebx
:00497C89 E802AAFBFF              call 00452690
:00497C8E EB0A                    jmp 00497C9A

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497C4B(C)
|
\* Possible StringData Ref from Code Obj ->"Invalid Registration Code."
           |
:00497C90 B8507D4900              mov eax, 00497D50
:00497C95 E82E4AFCFF              call 0045C6C8

           <===:(!真是讨人厌!

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00497BF6(C), :00497C11(C), :00497C21(U), :00497C7B(C), :00497C8E(U)
|
:00497C9A 33C0                    xor eax, eax
:00497C9C 5A                      pop edx
:00497C9D 59                      pop ecx
:00497C9E 59                      pop ecx
:00497C9F 648910                  mov dword ptr fs:[eax], edx
:00497CA2 68C77C4900              push 00497CC7

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497CC5(U)
|
:00497CA7 8D45EC                  lea eax, dword ptr [ebp-14]
:00497CAA E835C2F6FF              call 00403EE4
:00497CAF 8D45F0                  lea eax, dword ptr [ebp-10]
:00497CB2 E82DC2F6FF              call 00403EE4
:00497CB7 8D45FC                  lea eax, dword ptr [ebp-04]
:00497CBA E825C2F6FF              call 00403EE4
:00497CBF C3                      ret
```

---

### ①跟进上面的那个关键的 Call，来到如下代码：

```
:0049624C 55                      push ebp
:0049624D 8BEC                    mov ebp, esp

-------------------------------------------------------

:0049624F B910000000              mov ecx, 00000010

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00496259(C)
|
:00496254 6A00                    push 00000000
:00496256 6A00                    push 00000000
:00496258 49                      dec ecx
:00496259 75F9                    jne 00496254

-------------------------------------------------------

清16个堆栈单元0

:0049625B 51                      push ecx
:0049625C 53                      push ebx
:0049625D 56                      push esi
:0049625E 57                      push edi
:0049625F 8955FC                  mov dword ptr [ebp-04], edx
:00496262 8BF0                    mov esi, eax
:00496264 8B45FC                  mov eax, dword ptr [ebp-04]
:00496267 E8C8E0F6FF              call 00404334
:0049626C 33C0                    xor eax, eax
:0049626E 55                      push ebp
:0049626F 687D644900              push 0049647D
:00496274 64FF30                  push dword ptr fs:[eax]
:00496277 648920                  mov dword ptr fs:[eax], esp
:0049627A 8BC6                    mov eax, esi
:0049627C E82FF5FFFF              call 004957B0
:00496281 33DB                    xor ebx, ebx
:00496283 8D55E4                  lea edx, dword ptr [ebp-1C]
:00496286 8B45FC                  mov eax, dword ptr [ebp-04]
:00496289 E85E33F7FF              call 004095EC
:0049628E 8B55E4                  mov edx, dword ptr [ebp-1C]
:00496291 8D45FC                  lea eax, dword ptr [ebp-04]
:00496294 E8E3DCF6FF              call 00403F7C
:00496299 8B45FC                  mov eax, dword ptr [ebp-04]

           <===eax="2234567865148455"

\* Possible StringData Ref from Code Obj ->"2234123412341234"
           |
:0049629C BA98644900              mov edx, 00496498

           <===edx="1234123412341234"
           猜一下,注册码应该是16位

:004962A1 E8EADFF6FF              call 00404290
:004962A6 7510                    jne 004962B8
:004962A8 B201                    mov dl, 01
:004962AA 8BC6                    mov eax, esi
:004962AC E82BFBFFFF              call 00495DDC
:004962B1 8BD8                    mov ebx, eax
:004962B3 E961010000              jmp 00496419

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004962A6(C)
|
:004962B8 8B55FC                  mov edx, dword ptr [ebp-04]

           <===edx="2234567865148455"

:004962BB 8BC6                    mov eax, esi
:004962BD E8CAFCFFFF              call 00495F8C

           <====此Call当然也要跟进的!②

:004962C2 84C0                    test al, al
:004962C4 0F844F010000            je 00496419

           <===关键跳转!why?因为00496419距现在的代码
           位置004962C4有很大一段嘛!接跟着后面的大概
           是注册表操作了

:004962CA 8B55FC                  mov edx, dword ptr [ebp-04]
:004962CD 8BC6                    mov eax, esi
:004962CF E880F3FFFF              call 00495654
:004962D4 3C01                    cmp al, 01
:004962D6 0F843D010000            je 00496419
:004962DC B301                    mov bl, 01
:004962DE E8194FF7FF              call 0040B1FC
:004962E3 DD5DF0                  fstp qword ptr [ebp-10]
:004962E6 9B                      wait
:004962E7 FF75F4                  push [ebp-0C]
:004962EA FF75F0                  push [ebp-10]
:004962ED 8D45E8                  lea eax, dword ptr [ebp-18]
:004962F0 E81F4AF7FF              call 0040AD14
:004962F5 33C9                    xor ecx, ecx
:004962F7 B201                    mov dl, 01

\* Possible StringData Ref from Code Obj ->"?A"
           |
:004962F9 A184D74700              mov eax, dword ptr [0047D784]
:004962FE E8D975FEFF              call 0047D8DC
:00496303 8BF8                    mov edi, eax
:00496305 C6472D00                mov [edi+2D], 00
:00496309 8D4728                  lea eax, dword ptr [edi+28]
:0049630C 8B4E14                  mov ecx, dword ptr [esi+14]

\* Possible StringData Ref from Code Obj ->"Software\Microsof\"
           |
:0049630F BAB4644900              mov edx, 004964B4
:00496314 E8B3DEF6FF              call 004041CC
:00496319 8D45D4                  lea eax, dword ptr [ebp-2C]
:0049631C 8B55FC                  mov edx, dword ptr [ebp-04]
:0049631F E844F2F6FF              call 00405568
:00496324 8D4DD4                  lea ecx, dword ptr [ebp-2C]

\* Possible StringData Ref from Code Obj ->"SerNum"
           |
:00496327 BAD0644900              mov edx, 004964D0
:0049632C 8BC7
```
