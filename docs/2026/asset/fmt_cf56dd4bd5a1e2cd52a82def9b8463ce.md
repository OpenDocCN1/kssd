
```asm
           jne 00430567      <===跳过去就OVER了
:004303B2
33C9                    xor
ecx, ecx      <===计数器ECX，初始值化为0
:004303B4 85D2
               test edx, edx
:004303B6
7E18                    jle
004303D0      <===不跳
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004303CE(C)
|
:004303B8
8A0431                  mov al, byte
ptr [ecx+esi]
:004303BB 3C41
     cmp al, 41
:004303BD 0F8CA4010000
     jl 00430567           <===跳过去就OVER了
:004303C3
3C5A                    cmp
al, 5A
:004303C5 0F8F9C010000            jg 00430567
          <===跳过去就OVER了
:004303CB 41
                 inc ecx
:004303CC
3BCA                    cmp
ecx, edx
:004303CE 7CE8
   jl 004303B8    <===构成一个小循环，主要功能依次提取注册码的字符，检测是不是位于A-Z(大写)之间，只要一个不对，就OVER!
将假码改为ABCDEFGHIJKL，再重新来
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004303B6(C)
|
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
.......
中间部分省略，主要功能是将输入的注册码分成四段，每段三个字符
.......
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
:00430492
8D4C2428                lea ecx, dword
ptr [esp+28]
:00430496 885C2434
 mov byte ptr [esp+34], bl
:0043049A E8350B0500
     call 00480FD4
:0043049F 8B7C241C
       mov edi, dword ptr [esp+1C] <===EDI=GHI(第三段)
:004304A3
8B6C2418                mov ebp, dword
ptr [esp+18] <===EBP=JKL(第四段)
:004304A7 8B742420
       mov esi, dword ptr [esp+20] <===ESI=DEF(第二段)
:004304AB
8B442424                mov eax, dword
ptr [esp+24] <===EAX=ABC(第一段)
:004304AF 8A1F
           mov bl, byte ptr [edi]   <===BL=47(是G的ASCII的16进制表示形式)
:004304B1
8A16                    mov
dl, byte ptr [esi]   <===DL=44(是D的ASCII的16进制表示形式)
:004304B3 8A08
                 mov cl, byte ptr
[eax]   <===CL=41(是A的ASCII的16进制表示形式)
:004304B5 885C2412
           mov byte ptr [esp+12], bl
:004304B9
8A5D00                  mov bl, byte
ptr [ebp+00] <===BL=4A(是J的ASCII的16进制表示形式)
:004304BC 885C2413
           mov byte ptr [esp+13], bl
:004304C0
0FBE5C2412              movsx ebx, byte ptr
[esp+12]  <===EBX=47，不变
:004304C5 0FBED2
         movsx edx, dl
<===EDX=44
:004304C8 0FBEC9
   movsx ecx, cl           <===ECX=41
:004304CB
2BD3                    sub
edx, ebx
:004304CD 03D1
               add edx, ecx
       <===EDX=EDX-EBX+ECX=3E
:004304CF 0FBE4C2413
           movsx ecx, byte ptr [esp+13] <===ECX=4A
:004304D4
41                      inc
ecx          <===ECX=4B
:004304D5 3BD1
               cmp edx, ecx
<===这里必须相等，这里EDX和ECX怎么相等，见分析
:004304D7 740B
           je 004304E4      <===从这里跳走，才能正确注册
  \*\*\* 这一小段的分析得出n4-n7+n1=n10+1(n1、n4分别对应注册码的位上的值)才能正确注册，现将假码改为ABCDEFAHICKL，重新来。
:004304D9
C644243404              mov [esp+34], 04
:004304DE
8D4C2418                lea ecx, dword
ptr [esp+18]
:004304E2 EB54
     jmp 00430538  <===跳过去就OVER了
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004304D7(C)
|
                      <===假码改为ABCDEFAHICKL后，跳到这里，为说明问题，我们将其分为四段，每段三个字符
:004304E4
8A5701                  mov dl, byte
ptr [edi+01]<===取第三段字符AHI的第二个字符(H)DL=48
:004304E7 8A5801
           mov bl, byte ptr [eax+01]<===取第一段字符ABC的第二个字符(B)BL=42
:004304EA
8A4E01                  mov cl, byte
ptr [esi+01]<===取第二段字符DEF的第二个字符(E)CL=45
:004304ED 0FBEDB
           movsx ebx, bl <===EBX=42
:004304F0
0FBED2                  movsx edx,
dl <===EDX=48
:004304F3 0FBEC9
     movsx ecx, cl <===ECX=45
:004304F6 2BD3
               sub edx, ebx
:004304F8
03D1                    add
edx, ecx  <===EDX=EDX-EBX+ECX
:004304FA 0FBE4D01
         movsx ecx, byte ptr [ebp+01] <===取第四段字符CKL的第二个字符(K)CL=4B
:004304FE
41                      inc
ecx         <===ECX=ECX+1=4C
:004304FF 3BD1
               cmp edx, ecx   <===这里必须相等，这里EDX和ECX怎么相等，见分析
:00430501
740B                    je 0043050E
   <===从这里跳走，才能正确注册
   \*\*\* 这一小段的分析得出n8-n2+n5=n11+1(n2、n5分别对应注册码的位上的值)才能正确注册，现将假码改为ABCDBFAHICGL，重新来。
:00430503
C644243404              mov [esp+34], 04
:00430508
8D4C2418                lea ecx, dword
ptr [esp+18]
:0043050C EB2A
     jmp 00430538 <===跳过去就OVER了
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00430501(C)
|
:0043050E
8A4F02                  mov cl, byte
ptr [edi+02]<===取第三段字符AHI的第三个字符(I)CL=49
:00430511 8A5602
           mov dl, byte ptr [esi+02]<===取第二段字符DBF的第三个字符(F)DL=46
:00430514
8A4002                  mov al, byte
ptr [eax+02]<===取第一段字符ABC的第三个字符(C)AL=43
:00430517 8A5D02
           mov bl, byte ptr [ebp+02]<===取第四段字符CGL的第三个字符(L)BL=4C
:0043051A
0FBED2                  movsx edx,
dl    <===EDX=46
:0043051D 0FBEC9
         movsx ecx, cl    <===ECX=49
:00430520
2BCA                    sub
ecx, edx     <===ECX=ECX-EDX=3
:00430522 C644243404
         mov [esp+34], 04
:00430527 0FBED0
               movsx edx, al    <===EDX=43
:0043052A
0FBEC3                  movsx eax,
bl    <===EAX=4C
:0043052D 03CA
         add ecx, edx     <===ECX=ECX+EDX=46
:0043052F
48                      dec
eax          <===EAX=4C-1=4B
:00430530 3BC8
                 cmp ecx, eax <===这里必须相等，这里ECX和EAX怎么相等，见分析
:00430532
8D4C2418                lea ecx, dword
ptr [esp+18]
:00430536 7452
     je 0043058A        <===这里是个关键的跳转，跳了就成功注册了。
  \*\*\* 这一小段的分析得出n9-n6+n3=n12-1(n3、n6分别对应注册码的位上的值)才能正确注册，现将假码改为ABCDBCAHICGJ，再点注册，呵呵，成功了
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004304E2(U),
:0043050C(U)
|
:00430538 E8970A0500
 call 00480FD4
:0043053D 8D4C241C
   lea ecx, dword ptr [esp+1C]
:00430541 C644243403
         mov [esp+34], 03
:00430546 E8890A0500
           call 00480FD4
:0043054B 8D4C2420
             lea ecx, dword ptr [esp+20]
:0043054F
C644243402              mov [esp+34], 02
:00430554
E87B0A0500              call 00480FD4
:00430559
8D4C2424                lea ecx, dword
ptr [esp+24]
:0043055D C644243401
 mov [esp+34], 01
:00430562 E86D0A0500
   call 00480FD4
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004303AC(C),
:004303BD(C), :004303C5(C)
|
:00430567 8D4C2414
       lea ecx, dword ptr [esp+14]
:0043056B C644243400
             mov [esp+34], 00
:00430570
E85F0A0500              call 00480FD4
:00430575
8D4C243C                lea ecx, dword
ptr [esp+3C]
:00430579 C7442434FFFFFFFF        mov [esp+34],
FFFFFFFF
:00430581 E84E0A0500              call
00480FD4
:00430586 33C0
   xor eax, eax      <===EAX被清0，呵呵，就OVER了
:00430588
EB53                    jmp
004305DD
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00430536(C)
|
:0043058A
E8450A0500              call 00480FD4
:0043058F
8D4C241C                lea ecx, dword
ptr [esp+1C]
:00430593 C644243403
 mov [esp+34], 03
:00430598 E8370A0500
   call 00480FD4
:0043059D 8D4C2420
     lea ecx, dword ptr [esp+20]
:004305A1 C644243402
           mov [esp+34], 02
:004305A6 E8290A0500
             call 00480FD4
:004305AB 8D4C2424
               lea ecx, dword ptr [esp+24]
:004305AF
C644243401              mov [esp+34], 01
:004305B4
E81B0A0500              call 00480FD4
:004305B9
8D4C2414                lea ecx, dword
ptr [esp+14]
:004305BD C644243400
 mov [esp+34], 00
:004305C2 E80D0A0500
   call 00480FD4
:004305C7 8D4C243C
     lea ecx, dword ptr [esp+3C]
:004305CB C7442434FFFFFFFF
       mov [esp+34], FFFFFFFF
:004305D3 E8FC090500
           call 00480FD4
:004305D8 B801000000
             mov eax, 00000001   <===这里是个关键的赋值，经过这里才能正确注册，向上看
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00430588(U)
|
:004305DD
8B4C242C                mov ecx, dword
ptr [esp+2C]
:004305E1 5F
       pop edi
:004305E2 5E
             pop esi
:004305E3 5D
                   pop ebp
:004305E4
5B                      pop
ebx
:004305E5 64890D00000000          mov dword ptr
fs:[00000000], ecx
:004305EC 83C428
     add esp, 00000028
:004305EF C20400
           ret 0004
-----------------------------------------------------------------------
```
