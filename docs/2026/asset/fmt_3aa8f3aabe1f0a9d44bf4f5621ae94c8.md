
```
:004071D5 E8717B0100              call 0041ED4B
:004071DA C7442428FC814200        mov [esp+28], 004281FC
:004071E2 C684246404000006        mov byte ptr [esp+00000464], 06
:004071EA EB6B                    jmp 00407257

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004071AE(C)
|
:004071EC 6A10                    push 00000010
:004071EE 8D4C2420                lea ecx, dword ptr [esp+20]
:004071F2 E8DB7A0100              call 0041ECD2
:004071F7 8D4C243C                lea ecx, dword ptr [esp+3C]
:004071FB 6A08                    push 00000008
:004071FD 51                      push ecx
:004071FE 8D4C2420                lea ecx, dword ptr [esp+20]
:00407202 E8807A0100              call 0041EC87
:00407207 6A00                    push 00000000
:00407209 6A28                    push 00000028
:0040720B 8D4C2420                lea ecx, dword ptr [esp+20]
:0040720F E8BE7A0100              call 0041ECD2
:00407214 8D542448                lea edx, dword ptr [esp+48]
:00407218 6A10                    push 00000010
:0040721A 52                      push edx
:0040721B 8D4C2420                lea ecx, dword ptr [esp+20]
:0040721F E8637A0100              call 0041EC87
:00407224 6A00                    push 00000000
\* Possible StringData Ref from Data Obj ->"恭喜"
            |
:00407226 6868E54200              push 0042E568
\* Possible StringData Ref from Data Obj ->"注册成功,请重新启动本程序"
            |
:0040722B 684CE54200              push 0042E54C
:00407230 8BCE                    mov ecx, esi
:00407232 E8604A0100              call 0041BC97
:00407237 8D4C2418                lea ecx, dword ptr [esp+18]
:0040723B E80B7B0100              call 0041ED4B
:00407240 8BCE                    mov ecx, esi
:00407242 E8362E0100              call 0041A07D
:00407247 C7442428FC814200        mov [esp+28], 004281FC
:0040724F C684246404000007        mov byte ptr [esp+00000464], 07
```

跟一阵子之后，我觉得这好像是生成注册码的核心。主要是根据不同的名字（比如`12121212`）分别取每个字符跟`dl`、`bl`进行一些简单的运算，并把结果保存在`al`里，并根据不同的`al`值（应该是字符的ASCII码）的范围再进行一些运算。就可得到一个16位的注册码。

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004073C8(C)
|
:00407351 8B07                    mov eax, dword ptr [edi]  <-----在这设断注意一下al、bl、dl、ecx、[esp+ecx+14]之间的关系，大家可以可清楚的看到注册码的生成。
:00407353 8A540C14                mov dl, byte ptr [esp+ecx+14]
:00407357 8A1C28                  mov bl, byte ptr [eax+ebp]
:0040735A 8D3428                  lea esi, dword ptr [eax+ebp]
:0040735D 8AC2                    mov al, dl
:0040735F 02C3                    add al, bl
:00407361 3C41                    cmp al, 41
:00407363 88440C24                mov byte ptr [esp+ecx+24], al      生成第ecx+1位注册码（因为ecx初始值为0），不符合范围再进行调整。
        ^^^ ECX 的值是指现在取得是第几位字符
:00407367 7D06                    jge 0040736F
:00407369 8AC2                    mov al, dl
:0040736B 0441                    add al, 41
:0040736D EB18                    jmp 00407387
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407367(C)
|
:0040736F 3C5A                    cmp al, 5A
:00407371 7E0A                    jle 0040737D
:00407373 3C61                    cmp al, 61
:00407375 7D06                    jge 0040737D
:00407377 B05A                    mov al, 5A
:00407379 2AC2                    sub al, dl
:0040737B EB0A                    jmp 00407387
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00407371(C), :00407375(C)
|
:0040737D 3C7A                    cmp al, 7A
:0040737F 7E0A                    jle 0040738B
:00407381 8A440C1C                mov al, byte ptr [esp+ecx+1C]
:00407385 0461                    add al, 61
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040736D(U), :0040737B(U)
|
:00407387 88440C24                mov byte ptr [esp+ecx+24], al      <-----生成第ecx+1位注册码
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040737F(C)
|
:0040738B 8A06                    mov al, byte ptr [esi]
:0040738D 8A5C0C1C                mov bl, byte ptr [esp+ecx+1C]
:00407391 2AC3                    sub al, bl
:00407393 3C41                    cmp al, 41
:00407395 88440C2C                mov byte ptr [esp+ecx+2C], al      <-----生成第ecx+9位注册码，不符合范围再进行调整。
:00407399 7D0A                    jge 004073A5
:0040739B B05A                    mov al, 5A
:0040739D 2AC2                    sub al, dl
:0040739F 88440C2C                mov byte ptr [esp+ecx+2C], al      <-----生成第ecx+9位注册码
:004073A3 EB1C                    jmp 004073C1
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407399(C)
|
:004073A5 3C5A                    cmp al, 5A
:004073A7 7E09                    jle 004073B2
:004073A9 3C61                    cmp al, 61
:004073AB 7D05                    jge 004073B2
:004073AD 80C261                  add dl, 61
:004073B0 EB0B                    jmp 004073BD
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004073A7(C), :004073AB(C)
|
:004073B2 807C0C247A              cmp byte ptr [esp+ecx+24], 7A
:004073B7 7E08                    jle 004073C1
:004073B9 B27A                    mov dl, 7A
:004073BB 2AD3                    sub dl, bl
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004073B0(U)
|
:004073BD 88540C2C                mov byte ptr [esp+ecx+2C], dl
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004073A3(U), :004073B7(C)
|
:004073C1 41                      inc ecx            <-------ecx=ecx+1
:004073C2 83C704                  add edi, 00000004  <-------改变edi的值用于下次计算
:004073C5 83F908                  cmp ecx, 00000008  <-------供取八次（由此可知，注册名用八位以上的字符毫无意义，只取前八位。）
:004073C8 7C87                    jl 00407351        <-------循环
```

整理一下：

```
Name: Crack007(Crack007[BCG]也可，不过没什么实际意义)
Code: DosHAedFWgpBZcdU
```

似乎是万事大吉了，但，但最初的`edi`等值是怎么来的，我一直没看出来。于是对写注册机而言以上变得毫无意义。大家帮帮忙指点一下吧。
