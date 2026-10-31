
# 这是我今天破出来的第一个软件，呵呵，平安夜的早晨很冷的 (9千字)

SWF BROWSER 2。93（今天运气好二十多分钟就找到号了，虽然对高手这不算什么，呵呵，这可是我一周来的最快的一个了，好高兴）

开始时间：12/22 14:49

查找注册提示句

```
:004A99C1 E806FEFFFF              call 004A97CC
```
```
:004A99C6 84C0
  test al, al
```
```
:004A99C8 0F8492000000            je 004A9A60
```
跳到the serials number is invalid 偏移`0A8DC8h`

```
:004A99CE 6A00
  push 00000000
```
```
:004A99D0 668B0DA49A4A00          mov cx, word ptr
[004A9AA4]
```
```
:004A99D7 B202
  mov dl, 02
```
* Possible StringData Ref from Code Obj ->"Thank you for registering SWF "
->"Browser!"

再找到入口处：

```
:0043238F 89430C
mov dword ptr [ebx+0C], eax
```

先改改试试

```
:004A99C8 0F8492000000            je 004A9A60
```

还跟注册表有关，在成功注册下面：

* Possible StringData Ref from Code Obj ->"Software\Grooveware Multimedia\SWF
"
->"Browser\Registration"

* Possible StringData Ref from Code Obj ->"Name"

* Possible StringData Ref from Code Obj ->"Serial"

输入：“xuebuhui ”“68941367”后再启动时，出错！打开注册表看看，写入的资料没错，由错误类型看，应该是读取注册资料时可能由于字节什么的不符合，而出现的不可预料的错误！

重装吧，再看看对注册码有什么要求！

继续时间：12/24 3：31

跟入判断语句上面的call调用：

* Referenced by a CALL at Addresses:
|:004A99C1 , :004AC8B3 , :004AC9EF , :004ACCB3 , :004ACFD9
|:004AE003

```
:004A97CC 55
  push ebp            ebp入栈
:004A97CD 8BEC
  mov ebp, esp            ebp=esp
:004A97CF 6A00
  push 00000000
:004A97D1 6A00
  push 00000000
:004A97D3 6A00
  push 00000000
:004A97D5 6A00
  push 00000000
:004A97D7 6A00
  push 00000000
:004A97D9 6A00
  push 00000000
:004A97DB 6A00
  push 00000000
:004A97DD 53
  push ebx            ebx入栈
:004A97DE 56
  push esi            esi入栈
:004A97DF 57
  push edi
:004A97E0 894DF8
mov dword ptr [ebp-08], ecx
:004A97E3 8955FC
mov dword ptr [ebp-04], edx
:004A97E6 8B45FC
mov eax, dword ptr [ebp-04]        用户名到eax
:004A97E9 E81AA8F5FF              call 00404008
  什么用处？
:004A97EE 8B45F8
mov eax, dword ptr [ebp-08]        假sn到eax
:004A97F1 E812A8F5FF              call 00404008
  ？
:004A97F6 33C0
  xor eax, eax            eax清0
:004A97F8 55
  push ebp
:004A97F9 68F5984A00              push 004A98F5
:004A97FE 64FF30
push dword ptr fs:[eax]
:004A9801 648920
mov dword ptr fs:[eax], esp
:004A9804 33C0
  xor eax, eax            eax清0
:004A9806 55
  push ebp
:004A9807 68C6984A00              push 004A98C6
:004A980C 64FF30
push dword ptr fs:[eax]
:004A980F 648920
mov dword ptr fs:[eax], esp
:004A9812 33C9
  xor ecx, ecx            ecx清0
:004A9814 B201
  mov dl, 01            dl=01
* Possible StringData Ref from Code Obj ->"0A"
            |
:004A9816 A120874A00              mov eax,
dword ptr [004A8720]
:004A981B E84CFCFFFF              call 004A946C
:004A9820 8BD8
  mov ebx, eax
:004A9822 33D2
  xor edx, edx        edx清0
:004A9824 8BC3
  mov eax, ebx
:004A9826 E879F4FFFF              call 004A8CA4
:004A982B 8D45F4
lea eax, dword ptr [ebp-0C]
* Possible StringData Ref from Code Obj ->"1232hfbsdjdh2834121"
            |
:004A982E BA10994A00              mov edx,
004A9910
:004A9833 E834A4F5FF              call 00403C6C
:004A9838 8B55F4
mov edx, dword ptr [ebp-0C]
:004A983B 8BC3
  mov eax, ebx
:004A983D E8B6F1FFFF              call 004A89F8
:004A9842 8D4DF0
lea ecx, dword ptr [ebp-10]
:004A9845 8B55FC
mov edx, dword ptr [ebp-04]
:004A9848 8BC3
  mov eax, ebx
:004A984A E8F5F2FFFF              call 004A8B44
* Possible StringData Ref from Code Obj ->"ewrwk214134g7df2"
            |
:004A984F BA2C994A00              mov edx,
004A992C
:004A9854 8BC3
  mov eax, ebx
:004A9856 E89DF1FFFF              call 004A89F8
:004A985B 8D4DEC
lea ecx, dword ptr [ebp-14]
:004A985E 8B55F0
mov edx, dword ptr [ebp-10]
:004A9861 8BC3
  mov eax, ebx
:004A9863 E8DCF2FFFF              call 004A8B44
:004A9868 C745E8EFFFFFFF          mov [ebp-18], FFFFFFEF
:004A986F 8B45EC
mov eax, dword ptr [ebp-14]
:004A9872 E8DDA5F5FF              call 00403E54
:004A9877 85C0
  test eax, eax
:004A9879 7E1A
  jle 004A9895
:004A987B 8B45EC
mov eax, dword ptr [ebp-14]
:004A987E E8D1A5F5FF              call 00403E54
:004A9883 50
  push eax
:004A9884 8D45EC
lea eax, dword ptr [ebp-14]
:004A9887 E898A7F5FF              call 00404024
:004A988C 8D4DE8
lea ecx, dword ptr [ebp-18]
:004A988F 5A
  pop edx
:004A9890 E883FCFFFF              call 004A9518
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A9879(C)
|
:004A9895 8B45E8
mov eax, dword ptr [ebp-18]
:004A9898 33D2
  xor edx, edx
:004A989A 52
  push edx
:004A989B 50
  push eax
:004A989C 8D55E4
lea edx, dword ptr [ebp-1C]
:004A989F B820000000              mov eax,
00000020
:004A98A4 E89FF8F5FF              call 00409148
:004A98A9 8B55E4
mov edx, dword ptr [ebp-1C]
:004A98AC 8B45F8
mov eax, dword ptr [ebp-08]
:004A98AF E8B0A6F5FF              call 00403F64
          真假注册码的比较
:004A98B4 7504
  jne 004A98BA
:004A98B6 B301
  mov bl, 01
:004A98B8 EB02
  jmp 004A98BC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A98B4(C)
|
:004A98BA 33DB
  xor ebx, ebx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A98B8(U)
|
:004A98BC 33C0
  xor eax, eax
:004A98BE 5A
  pop edx
:004A98BF 59
  pop ecx
:004A98C0 59
  pop ecx
:004A98C1 648910
mov dword ptr fs:[eax], edx
:004A98C4 EB0C
  jmp 004A98D2
:004A98C6 E9ED9AF5FF              jmp 004033B8
:004A98CB 33DB
  xor ebx, ebx
:004A98CD E8429EF5FF              call 00403714
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A98C4(U)
|
:004A98D2 33C0
  xor eax, eax
:004A98D4 5A
  pop edx
:004A98D5 59
  pop ecx
:004A98D6 59
  pop ecx
:004A98D7 648910
mov dword ptr fs:[eax], edx
:004A98DA 68FC984A00              push 004A98FC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A98FA(U)
|
:004A98DF 8D45E4
lea eax, dword ptr [ebp-1C]
:004A98E2 E8EDA2F5FF              call 00403BD4
:004A98E7 8D45EC
lea eax, dword ptr [ebp-14]
:004A98EA BA05000000              mov edx,
00000005
:004A98EF E804A3F5FF              call 00403BF8
:004A98F4 C3
  ret
:004A98F5 E9729DF5FF              jmp 0040366C
:004A98FA EBE3
  jmp 004A98DF
:004A98FC 8BC3
  mov eax, ebx
:004A98FE 5F
  pop edi
:004A98FF 5E
  pop esi
:004A9900 5B
  pop ebx
:004A9901 8BE5
  mov esp, ebp
:004A9903 5D
  pop ebp
:004A9904 C3
  ret
```

-----------------------------------------------------------------------

* Referenced by a CALL at Address:
|:004A981B
|
```
:004A946C 53
  push ebx
:004A946D 56
  push esi
:004A946E 84D2
  test dl, dl        dl为0（实际上是1）
:004A9470 7408
  je 004A947A        就跳
:004A9472 83C4F0
add esp, FFFFFFF0        esp=esp-14
:004A9475 E8D29DF5FF              call 0040324C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A9470(C)
|
:004A947A 8BDA
  mov ebx, edx
:004A947C 8BF0
  mov esi, eax
:004A947E 33D2
  xor edx, edx
:004A9480 8BC6
  mov eax, esi
:004A9482 E8C1ECF6FF              call 00418148
:004A9487 B201
  mov dl, 01
* Possible StringData Ref from Code Obj ->"    TBlowCorelJ"
            |
:004A9489 A1C8864A00              mov eax,
dword ptr [004A86C8]
:004A948E E8499AF5FF              call 00402EDC
:004A9493 894624
mov dword ptr [esi+24], eax
:004A9496 8BC6
  mov eax, esi
:004A9498 84DB
  test bl, bl
:004A949A 740F
  je 004A94AB
:004A949C E8039EF5FF              call 004032A4
:004A94A1 648F0500000000          pop dword ptr fs:[00000000]
:004A94A8 83C40C
add esp, 0000000C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A949A(C)
|
:004A94AB 8BC6
  mov eax, esi
:004A94AD 5E
  pop esi
:004A94AE 5B
  pop ebx
:004A94AF C3
  ret
```

------------------------------------------------------------

用`trw`在`004A98AF`中设断，然后`d eax`和`d edx`可以看到自己的真假注册码

这里是Username:XueBuhui  假注册码：68941367  真注册码：81BDC2D3
