
# 系统优化大师 12.15

> 原文：[`www.pediy.com/kssd/pediy06/pediy6357.htm`](https://www.pediy.com/kssd/pediy06/pediy6357.htm)

注册码部分的算法我就不写了，因为它和以前的版本一样，如果想知道的朋友可以看我以前发的贴 `http://tongtian.net/pediybbs/viewtopic.php?t=1527&highlight=%D0%A1%B5%DA%B3%F5`，只是这一版加入了许多陷阱：

1. 当你输入定单号 `123456` 之后随便输入注册码就可以注册成功，难道真的成功了吗？
2. 加入"谢谢您的注册,请连网验证"等骗人的字符串。
3. 也是最重要的重起验证，重起时检测 `key0oi1l.sys`（注册后生成的文件）中的注册信息，当前两位定单号等于 `19` 或 `20` 时，并且定单号 = `7` 位才可以注册成功。

以下就是重起验证的部分。

定单号 `1234567`，注册码：`leozem[YCG]`

搜索调用字符 `key0oi1l.sys`，然后来到下面。

```
* Possible StringData Ref from Code Obj ->"syssetup"
                          |
:004F8E92 BAC0994F00        mov edx, 004F99C0
:004F8E97 8BC3            mov eax, ebx
:004F8E99 8B30            mov esi, dword ptr [eax]
:004F8E9B FF16            call dword ptr [esi]
:004F8E9D 8B8510FEFFFF    mov eax, dword ptr [ebp+FFFFFE10]-----定单号进 EAX
:004F8EA3 E8CC03F1FF      call 00409274---------追
:004F8EA8 3DBFC62D00      cmp eax, 002DC6BF-----EAX=12D687 与 002DC6BF 比
:004F8EAD 7D50            jge 004F8EFF----------大于或等于则跳到未注册
:004F8EAF 6A00            push 00000000
:004F8EB1 8D850CFEFFFF    lea eax, dword ptr [ebp+FFFFFE0C]
:004F8EB7 50             push eax
:004F8EB8 B9B4994F00      mov ecx, 004F99B4
* Possible StringData Ref from Code Obj ->"syssetup"
                          |
:004F8EBD BAC0994F00      mov edx, 004F99C0
:004F8EC2 8BC3            mov eax, ebx
:004F8EC4 8B18            mov ebx, dword ptr [eax]
:004F8EC6 FF13            call dword ptr [ebx]
:004F8EC8 8B850CFEFFFF    mov eax, dword ptr [ebp+FFFFFE0C]-----定单号进 EAX
:004F8ECE E8A103F1FF      call 00409274---------重复上面的循环
:004F8ED3 3D7F231B00      cmp eax, 001B237F-----EAX=12D687 与 001B237F 比
:004F8ED8 7E25            jle 004F8EFF----------小于或等于则跳到未注册
:004F8EDA 8B45FC          mov eax, dword ptr [ebp-04]
:004F8EDD 8B8034030000    mov eax, dword ptr [eax+00000334]
:004F8EE3 8B8008020000    mov eax, dword ptr [eax+00000208]
:004F8EE9 BA01000000      mov edx, 00000001
:004F8EEE E8C5E8F7FF      call 004777B8
* Possible StringData Ref from Code Obj ->"软件已注册"
                          |
:004F8EF3 BAD4994F00      mov edx, 004F99D4
:004F8EF8 E817E8F7FF      call 00477714
:004F8EFD EB48            jmp 004F8F47
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
 |:004F8EAD(C), :004F8ED8(C)
 |
:004F8EFF 8B45FC          mov eax, dword ptr [ebp-04]
:004F8F02 8B8034030000    mov eax, dword ptr [eax+00000334]
:004F8F08 8B8008020000    mov eax, dword ptr [eax+00000208]
:004F8F0E BA01000000      mov edx, 00000001
:004F8F13 E8A0E8F7FF      call 004777B8
* Possible StringData Ref from Code Obj ->"软件未注册"
                          |
:004F8F18 BAE8994F00      mov edx, 004F99E8
:004F8F1D E8F2E7F7FF      call 00477714
:004F8F22 EB23            jmp 004F8F47
------------------------------------------------------------------------------------------
|
:00409274 53             push ebx
:00409275 56             push esi
:00409276 83C4F4         add esp, FFFFFFF4
:00409279 8BD8           mov ebx, eax
:0040927B 8BD4           mov edx, esp
:0040927D 8BC3           mov eax, ebx
:0040927F E824A0FFFF     call 004032A8-----再追
:00409284 8BF0           mov esi, eax
:00409286 833C2400       cmp dword ptr [esp], 00000000
----------------------------------------------------------------------------------------
* Referenced by a CALL at Addresses:
 |:0040927F   , :004092B6
 |
:004032A8 53             push ebx
:004032A9 56             push esi
:004032AA 57             push edi
:004032AB 89C6           mov esi, eax-------定单号
:004032AD 50            push eax
:004032AE 85C0           test eax, eax-------定单号是否存在
:004032B0 746C           je 0040331E------不在则跳
:004032B2 31C0           xor eax, eax----清零
:004032B4 31DB           xor ebx, ebx----清零
:004032B6 BFCCCCCC0C     mov edi, 0CCCCCCC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
 |:004032C1(C)
 |
:004032BB 8A1E           mov bl, byte ptr [esi]-----定单号第一位 1 的 ASCLL 码 31 进 BL
:004032BD 46            inc esi----------计数器加 1
:004032BE 80FB20        cmp bl, 20------第一位是否等于空格
:004032C1 74F8          je 004032BB-----是则跳
:004032C3 B500          mov ch, 00
:004032C5 80FB2D        cmp bl, 2D-------是否等于-
:004032C8 7462          je 0040332C------是则跳
:004032CA 80FB2B        cmp bl, 2B-------是否等于+
:004032CD 745F          je 0040332E------是则跳
:004032CF 80FB24        cmp bl, 24-------是否等于$
:004032D2 745F          je 00403333------是则跳
:004032D4 80FB78        cmp bl, 78-------是否等于 x
:004032D7 745A          je 00403333------是则跳
:004032D9 80FB58        cmp bl, 58-------是否等于 X
:004032DC 7455          je 00403333------是则跳
:004032DE 80FB30        cmp bl, 30-------是否等于 0
:004032E1 7513          jne 004032F6-----不是则跳
:004032E3 8A1E          mov bl, byte ptr [esi]
:004032E5 46           inc esi
:004032E6 80FB78       cmp bl, 78
:004032E9 7448         je 00403333
:004032EB 80FB58       cmp bl, 58
:004032EE 7443         je 00403333
:004032F0 84DB         test bl, bl
:004032F2 7420         je 00403314
:004032F4 EB04         jmp 004032FA
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
 |:004032E1(C), :00403331(U)
 |
:004032F6 84DB          test bl, bl------定单号第一位是否存在
:004032F8 742D         je 00403327------不在则跳
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
 |:004032F4(U), :00403312(C)
 |
:004032FA 80EB30        sub bl, 30-------BL-30=BL
:004032FD 80FB09        cmp bl, 09-------与 9 比
:00403300 7725          ja 00403327------大于则跳
:00403302 39F8          cmp eax, edi-----EAX 与 EDI=0CCCCCCC 比
:00403304 7721          ja 00403327------大于则跳
:00403306 8D0480        lea eax, dword ptr [eax+4*eax]----EAX=EAX*4+EAX=EAX*5
:00403309 01C0          add eax, eax-----EAX=EAX*2
:0040330B 01D8          add eax, ebx-----EAX=EAX+EBX
:0040330D 8A1E          mov bl, byte ptr [esi]-----依次取第一位后的定单号 ASCLL 码进 BL
:0040330F 46           inc esi-------计数器加 1
:00403310 84DB         test bl, bl-------ESI 的定单号是否取完
:00403312 75E6         jne 004032FA------没有完继续循环取，取完后得出 EAX=12D687
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```

4dd466d0e1e0cc123afbe504da2462ca
