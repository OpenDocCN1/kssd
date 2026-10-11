
# 拱猪大战 1.8

> 原文：[`www.pediy.com/kssd/pediy05/pediy50508.htm`](https://www.pediy.com/kssd/pediy05/pediy50508.htm)

## 1. 基本信息

**拱猪大战 1.8 破解手记--算法分析**

*   **作者**：newlaos[DFCG]
*   **软件名称**：拱猪大战 1.8 (棋牌游戏)
*   **整理日期**：2003.3.14
*   **文件大小**：1107KB
*   **软件授权**：共享软件
*   **使用平台**：Win9x/Me/NT/2000/XP
*   **发布公司**：[`www.joygames.com/`](http://www.joygames.com/)

**软件简介**：支持对家结盟模式，支持现在流行的拱双猪，允许亮牌，支持"除羊全亮"，支持几乎所有的拱猪规则；可以随时察看亮牌和已经得到的分数；具有积分榜和英雄榜，更加你的成就感；自由的设计软件皮肤；软件界面友好，易用性强。

## 2. 加密分析与工具

*   **加密方式**：注册码
*   **功能限制**：次数限制
*   **PJ 工具**：TRW20001.23 注册版、PE-SCAN3.31、W32Dasm8.93 黄金版、FI2.5
*   **PJ 日期**：2003-03-22

**作者 newlaos 申明**：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 3. 分析过程

### 3.1 脱壳

1.  先用 `FI2.5` 看一下主程序“拱猪大战.exe”，加了个 `ASPACK2.12` 的壳，先脱了它。这种壳很容易脱，用 `PE-scan3.31` 很快搞定。脱壳生成文件 `UNPACK.EXE`。

### 3.2 静态分析

2.  用 `W32Dasm8.93` 黄金版对 `UNPACK.EXE` 进行静态反汇编，再用串式数据参考，找到"注册码输入错误,请重新检查后输入!"(很经典的句子)，双击来到下面代码段。这样就找到注册码的计算部分。

### 3.3 动态跟踪

3.  再用 `TRW20001.23` 注册版进行动态跟踪，下断 `BPX 00425ACE` (通常在注册成功与否的前面一些下断，这样，才能找到关键部分)，先输入假码 `78787878`。

```
:00425ACE 66C747101400              mov [edi+10], 0014
:00425AD4 33C9                      xor ecx, ecx
:00425AD6 894DF4                    mov dword ptr [ebp-0C], ecx
:00425AD9 8D55F4                    lea edx, dword ptr [ebp-0C]
:00425ADC FF471C                    inc [edi+1C]
:00425ADF 8B830C030000              mov eax, dword ptr [ebx+0000030C]
:00425AE5 E8D6EB0300                call 004646C0
:00425AEA 8D4DF4                    lea ecx, dword ptr [ebp-0C]
:00425AED 33D2                      xor edx, edx
:00425AEF 8B01                      mov eax, dword ptr [ecx]
:00425AF1 50                        push eax
:00425AF2 8955F0                    mov dword ptr [ebp-10], edx
:00425AF5 FF471C                    inc [edi+1C]
:00425AF8 8D55F0                    lea edx, dword ptr [ebp-10]
:00425AFB 8B8310030000              mov eax, dword ptr [ebx+00000310]
:00425B01 E8BAEB0300                call 004646C0
:00425B06 8D4DF0                    lea ecx, dword ptr [ebp-10]
:00425B09 8B01                      mov eax, dword ptr [ecx]  <===EAX=78787878
:00425B0B 50                        push eax                   <===这里压进去的还是 78787878
:00425B0C E81F39FFFF                call 00419430              <===最后推断，这里就是算法 CALL 了，F8 跟进
:00425B11 83C408                    add esp, 00000008
:00425B14 BA02000000                mov edx, 00000002
:00425B19 50                        push eax                   <===这里 EAX 的值就决定了，不能为 0
:00425B1A 8D45F0                    lea eax, dword ptr [ebp-10]
:00425B1D FF4F1C                    dec [edi+1C]
:00425B20 E88B6B0900                call 004BC6B0
:00425B25 FF4F1C                    dec [edi+1C]
:00425B28 8D45F4                    lea eax, dword ptr [ebp-0C]
:00425B2B BA02000000                mov edx, 00000002
:00425B30 E87B6B0900                call 004BC6B0
:00425B35 59                        pop ecx                    <===说明，最终判断是由堆栈最上面的值决定!向上看
:00425B36 84C9                      test cl, cl              <===如果，ECX 的低位是 0，则下面就跳转了，也就 OVER 了
:00425B38 0F84EF010000              je 00425D2D              <===关键跳转
:00425B3E B201                      mov dl, 01
:00425B40 A1808F4900                mov eax, dword ptr [00498F80]
:00425B45 E836350700                call 00499080
:00425B4A 8BF0                      mov esi, eax
:00425B4C BA02000080                mov edx, 80000002
:00425B51 8BC6                      mov eax, esi
:00425B53 E8CC690900                call 004BC524
:00425B58 66C747102000              mov [edi+10], 0020
```

中间这段代码的主要功能，就是在验证注册码正确后，把注册信息写入注册表，与关键算法无关，所以省略。

```
* Possible StringData Ref from Data Obj ->"信息"
                          |
:00425CCC 6800E24F00      push 004FE200

* Possible StringData Ref from Data Obj ->"注册成功!注册版本在重新启动程序后生效!"
                          |
:00425CD1 68D9E14F00      push 004FE1D9
:00425CD6 8BC3            mov eax, ebx
:00425CD8 E873500400      call 0046AD50
:00425CDD 50              push eax
:00425CDE E8FD1B0D00      call 004F78E0
:00425CE3 BAEBFFFFFF      mov edx, FFFFFFEB
:00425CE8 8B83F8020000    mov eax, dword ptr [ebx+000002F8]
:00425CEE E8F16C0900      call 004BC9E4
:00425CF3 8BD0            mov edx, eax
:00425CF5 8B83F8020000    mov eax, dword ptr [ebx+000002F8]
:00425CFB FFD2            call edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00425BBC(C)
|
:00425CFD 8BC6            mov eax, esi
:00425CFF E8EC330700      call 004990F0
:00425D04 8BDE            mov ebx, esi
:00425D06 895DCC          mov dword ptr [ebp-34], ebx
:00425D09 85DB            test ebx, ebx
:00425D0B 7439            je 00425D46
:00425D0D 8B03            mov eax, dword ptr [ebx]
:00425D0F 8945D0          mov dword ptr [ebp-30], eax
:00425D12 66C747106800    mov [edi+10], 0068
:00425D18 BA03000000      mov edx, 00000003
:00425D1D 8B45CC          mov eax, dword ptr [ebp-34]
:00425D20 8B08            mov ecx, dword ptr [eax]
:00425D22 FF51FC          call [ecx-04]
:00425D25 66C747105C00    mov [edi+10], 005C
:00425D2B EB19            jmp 00425D46

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00425B38(C)                          <===由于上一行是无条件跳转，所以，关键跳转在 00425B38
|
:00425D2D 6A40            push 00000040

* Possible StringData Ref from Data Obj ->"信息"
                          |
:00425D2F 6826E24F00      push 004FE226

* Possible StringData Ref from Data Obj ->"注册码输入错误,请重新检查后输入!"
                          |
:00425D34 6805E24F00      push 004FE205
:00425D39 8BC3            mov eax, ebx
:00425D3B E810500400      call 0046AD50
:00425D40 50              push eax
:00425D41 E89A1B0D00      call 004F78E0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00425D0B(C), :00425D2B(U)
|
:00425D46 8B17            mov edx, dword ptr [edi]
:00425D48 64891500000000  mov dword ptr fs:[00000000], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00425AC9(U), :00425C0D(U)
|
:00425D4F 5F              pop edi
:00425D50 5E              pop esi
:00425D51 5B              pop ebx
:00425D52 8BE5            mov esp, ebp
:00425D54 5D              pop ebp
:00425D55 C3              ret
```

### 3.4 算法分析

------ `00425B0C call 00419430` 算法 CALL，F8 跟进到下列代码段 ------

```
:00419430 55              push ebp
:00419431 8BEC            mov ebp, esp
:00419433 81C4BCFEFFFF    add esp, FFFFFEBC
:00419439 53              push ebx
:0041943A 56              push esi
:0041943B 57              push edi
:0041943C 8D7DCC          lea edi, dword ptr [ebp-34]
:0041943F B830AC4F00      mov eax, 004FAC30
:00419444 E88B750900      call 004B09D4
:00419449 C7471C02000000  mov [edi+1C], 00000002
:00419450 8D550C          lea edx, dword ptr [ebp+0C]
:00419453 8D450C          lea eax, dword ptr [ebp+0C]
:00419456 E835310A00      call 004BC590
:0041945B FF471C          inc [edi+1C]
:0041945E 8D5508          lea edx, dword ptr [ebp+08]
:00419461 66C747100800    mov [edi+10], 0008
:00419467 8D4508          lea eax, dword ptr [ebp+08]
:0041946A E821310A00      call 004BC590
:0041946F FF471C          inc [edi+1C]
:00419472 57              push edi
:00419473 8DBDBCFEFFFF    lea edi, dword ptr [ebp+FFFFFEBC]

* Possible StringData Ref from Data Obj ->""
                          |
```



```
:00419479 BEF4814F00              mov esi, 004F81F4

:0041947E B90C000000              mov ecx, 0000000C

:00419483 F3                     repz

:00419484 A5                     movsd

:00419485 837D0800               cmp dword ptr [ebp+08], 00000000

:00419489 5F                     pop edi

:0041948A 7408                   je 00419494

:0041948C 8B4508                 mov eax, dword ptr [ebp+08]  <===EAX=78787878

:0041948F 8B50FC                 mov edx, dword ptr [eax-04]  <===EDX=8(为输入的注册码长度)

:00419492 EB02                   jmp 00419496                 <===从这里跳走

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:0041948A(C)

:00419494 33D2                   xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419492(U)

:00419496 83FA0B                 cmp edx, 0000000B

                           <===跳到这里，可以看出，规定注册注册码必须等于 0xB(也就是 11 位)，将输入的假码改为 78787878787，重新来

:00419499 7432                   je 004194CD       <===当注册码长度为 11 位时，这里要跳走

:0041949B 33C0                   xor eax, eax

:0041949D BA02000000             mov edx, 00000002

:004194A2 50                     push eax

:004194A3 8D4508                 lea eax, dword ptr [ebp+08]

:004194A6 FF4F1C                  dec [edi+1C]

:004194A9 E802320A00             call 004BC6B0

:004194AE FF4F1C                  dec [edi+1C]

:004194B1 8D450C                 lea eax, dword ptr [ebp+0C]

:004194B4 BA02000000             mov edx, 00000002

:004194B9 E8F2310A00             call 004BC6B0

:004194BE 58                     pop eax

:004194BF 8B17                   mov edx, dword ptr [edi]

:004194C1 64891500000000         mov dword ptr fs:[00000000], edx

:004194C8 E9B0020000             jmp 0041977D          <===如果，输入的假码长度不为 11，则从这里跳到 OVER

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419499(C)

:004194CD BE01000000             mov esi, 00000001     <===从上面跳到这里

:004194D2 8D8548FFFFFF           lea eax, dword ptr [ebp+FFFFFF48]

:004194D8 8945C8                 mov dword ptr [ebp-38], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004195A3(C)

      <===从这一行开始到 004195A3 是一个循环结构，主要的功能是检测输入的 11 位注册码，是不是数字

:004194DB 66C747101400           mov [edi+10], 0014

:004194E1 33D2                   xor edx, edx

:004194E3 8D4DFC                 lea ecx, dword ptr [ebp-04]

:004194E6 8955FC                 mov dword ptr [ebp-04], edx

:004194E9 51                     push ecx

:004194EA FF471C                 inc [edi+1C]

:004194ED B901000000             mov ecx, 00000001

:004194F2 8BD6                   mov edx, esi

:004194F4 8D4508                 lea eax, dword ptr [ebp+08]

:004194F7 E82C330A00             call 004BC828

:004194FC 837DFC00               cmp dword ptr [ebp-04], 00000000

:00419500 7405                   je 00419507

:00419502 8B45FC                 mov eax, dword ptr [ebp-04]

:00419505 EB05                   jmp 0041950C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419500(C)

:00419507 B888874F00             mov eax, 004F8788

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419505(U)

:0041950C 8A18                   mov bl, byte ptr [eax]

:0041950E FF4F1C                 dec [edi+1C]

:00419511 8D45FC                 lea eax, dword ptr [ebp-04]

:00419514 BA02000000             mov edx, 00000002

:00419519 E892310A00             call 004BC6B0

:0041951E 0FBEC3                 movsx eax, bl  <===这里就是将依次取出的注册码的 ASC 码值的(16 进制表示形式)，放入 EAX

:00419521 83F830                 cmp eax, 00000030   --

:00419524 7C05                   jl 0041952B          |

:00419526 83F839                 cmp eax, 00000039     |===>很典型的一段代码，功能是看输入的注册码每位是不是数字

:00419529 7E32                   jle 0041955D        -- ===>如果是数字，这里小跳

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419524(C)

:0041952B 33C0                   xor eax, eax

        <===一但有一位是注册码不是数字，就来到这里，随着 00419558 的无条件跳转，也就是要 OVER 了

:0041952D BA02000000             mov edx, 00000002

:00419532 50                     push eax

:00419533 8D4508                 lea eax, dword ptr [ebp+08]

:00419536 FF4F1C                 dec [edi+1C]

:00419539 E872310A00             call 004BC6B0

:0041953E FF4F1C                 dec [edi+1C]

:00419541 8D450C                 lea eax, dword ptr [ebp+0C]

:00419544 BA02000000             mov edx, 00000002

:00419549 E862310A00             call 004BC6B0

:0041954E 58                     pop eax

:0041954F 8B17                   mov edx, dword ptr [edi]

:00419551 64891500000000         mov dword ptr fs:[00000000], edx

:00419558 E920020000             jmp 0041977D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00419529(C)

:0041955D 66C747102000           mov [edi+10], 0020  <===每次循环(共 11 次了)，只要注册都是数字，就会从上面小跳到这里

:00419563 33C9                   xor ecx, ecx

:00419565 8D45F8                 lea eax, dword ptr [ebp-08]

:00419568 894DF8                 mov dword ptr [ebp-08], ecx

:0041956B 50                     push eax

:0041956C FF471C                 inc [edi+1C]

:0041956F 8D4508                 lea eax, dword ptr [ebp+08]

:00419572 B901000000             mov ecx, 00000001

:00419577 8BD6                   mov edx, esi

:00419579 E8AA320A00             call 004BC828

:0041957E 8D45F8                 lea eax, dword ptr [ebp-08]

:00419581 E886330A00             call 004BC90C

:00419586 8B55C8                 mov edx, dword ptr [ebp-38]

:00419589 8902                   mov dword ptr [edx], eax

:0041958B FF4F1C                 dec [edi+1C]

:0041958E 8D45F8                 lea eax, dword ptr [ebp-08]

:00419591 BA02000000             mov edx, 00000002

:00419596 E815310A00             call 004BC6B0

:0041959B 46                     inc esi

:0041959C 8345C804               add dword ptr [ebp-38], 00000004

:004195A0 83FE0B                 cmp esi, 0000000B      <===ESI 实际上是个计数器

:004195A3 0F8E32FFFFFF           jle 004194DB          <===循环 11 次后，这里就不再跳转了!

:004195A9 BE01000000             mov esi, 00000001

      <===呵呵，这个计数器又被初始化了。也就是又要循环了，几次呢看下面 0041967C 行，原来是 9 次。

:004195AE 8D85F0FEFFFF           lea eax, dword ptr [ebp+FFFFFEF0]

:004195B4 8945C4                 mov dword ptr [ebp-3C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:0041967F(C)

     <===从这一行开始到 0041967F 是一个循环结构，循环 9 次，正好是取机器码的 9 次了。我的机器码是 642652146

:004195B7 66C747102C00           mov [edi+10], 002C

:004195BD 33D2                   xor edx, edx

:004195BF 8D4DF4                 lea ecx, dword ptr [ebp-0C]

:004195C2 8955F4                 mov dword ptr [ebp-0C], edx

:004195C5 51                     push ecx

:004195C6 FF471C                 inc [edi+1C]

:004195C9 B901000000             mov ecx, 00000001

:004195CE 8BD6                   mov edx, esi

:004195D0 8D450C                 lea eax, dword ptr [ebp+0C] <===EAX 里放的是一个地址指针，指向机器码

:004195D3 E850320A00             call 004BC828              <===这里当然是将机器码处理一下了。

:004195D8 837DF400               cmp dword ptr [ebp-0C], 00000000

:004195DC 7405                   je 004195E3

:004195DE 8B45F4                 mov eax, dword ptr [ebp-0C]

:004195E1 EB05                   jmp 004195E8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004195DC(C)

:004195E3 B889874F00             mov eax, 004F8789

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004195E1(U)
```



:004195E8 8A18  mov bl, byte ptr [eax]  
:004195EA FF4F1C  dec [edi+1C]  
:004195ED 8D45F4  lea eax, dword ptr [ebp-0C]  
:004195F0 BA02000000  mov edx, 00000002  
:004195F5 E8B6300A00  call 004BC6B0  
:004195FA 0FBEC3  movsx eax, bl  
:004195FD 83F830  cmp eax, 00000030  ----很典型的一小段代码(功能同上)  
:00419600 7C05  jl 00419607  ----  
:00419602 83F839  cmp eax, 00000039  ----  
:00419605 7E32  jle 00419639  ----这里小跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00419600(C)  
|

:00419607 33C0  xor eax, eax  
:00419609 BA02000000  mov edx, 00000002  
:0041960E 50  push eax  
:0041960F 8D4508  lea eax, dword ptr [ebp+08]  
:00419612 FF4F1C  dec [edi+1C]  
:00419615 E896300A00  call 004BC6B0  
:0041961A FF4F1C  dec [edi+1C]  
:0041961D 8D450C  lea eax, dword ptr [ebp+0C]  
:00419620 BA02000000  mov edx, 00000002  
:00419625 E886300A00  call 004BC6B0  
:0041962A 58  pop eax  
:0041962B 8B17  mov edx, dword ptr [edi]  
:0041962D 64891500000000  mov dword ptr fs:[00000000], edx  
:00419634 E944010000  jmp 0041977D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00419605(C)  
|

:00419639 66C747103800  mov [edi+10], 0038  <===小跳来到这里  
:0041963F 33C9  xor ecx, ecx  
:00419641 8D45F0  lea eax, dword ptr [ebp-10]  
:00419644 894DF0  mov dword ptr [ebp-10], ecx  
:00419647 50  push eax  
:00419648 FF471C  inc [edi+1C]  
:0041964B 8D450C  lea eax, dword ptr [ebp+0C]  
:0041964E B901000000  mov ecx, 00000001  
:00419653 8BD6  mov edx, esi  
:00419655 E8CE310A00  call 004BC828  
:0041965A 8D45F0  lea eax, dword ptr [ebp-10]  
:0041965D E8AA320A00  call 004BC90C  
:00419662 8B55C4  mov edx, dword ptr [ebp-3C]  
:00419665 8902  mov dword ptr [edx], eax  
:00419667 FF4F1C  dec [edi+1C]  
:0041966A 8D45F0  lea eax, dword ptr [ebp-10]  
:0041966D BA02000000  mov edx, 00000002  
:00419672 E839300A00  call 004BC6B0  
:00419677 46  inc esi  
:00419678 8345C404  add dword ptr [ebp-3C], 00000004  
:0041967C 83FE09  cmp esi, 00000009  
:0041967F 0F8E32FFFFFF  jle 004195B7  <===循环 9 次后，这里就不再跳转了!  
:00419685 33F6  xor esi, esi  <===ESI 被清 0 了。又要循环了。  
:00419687 8D95BCFEFFFF  lea edx, dword ptr [ebp+FFFFFEBC]  
:0041968D 8D8548FFFFFF  lea eax, dword ptr [ebp+FFFFFF48]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196A8(C)  
|  <===这一行到 004196A5，又构成一个小循环

:00419693 8B1A  mov ebx, dword ptr [edx]  
   <===这里出现 11 次的数字是固定的(4，A，6，8，9，2，1，3，5，7，B )，估计是机器码的变形(后来发现没有用)  
:00419695 8B08  mov ecx, dword ptr [eax]  
   <===这里依次就是我们输入的注册码的了(7，8，7，8，7，8，7，8，7，8，7)  
:00419697 83C004  add eax, 00000004  
:0041969A 83C204  add edx, 00000004  
:0041969D 46  inc esi  
:0041969E 898C9D14FFFFFF  mov dword ptr [ebp+4*ebx-000000EC], ecx  
:004196A5 83FE0A  cmp esi, 0000000A  <===由于初始值是 0，所以这里也要循环 11 次，  
:004196A8 7EE9  jle 00419693  <===循环 11 次后，不再跳转了。  
:004196AA BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:004196AF 8D55A0  lea edx, dword ptr [ebp-60]  
:004196B2 8D8518FFFFFF  lea eax, dword ptr [ebp+FFFFFF18]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196C6(C)  <===这一行到 004196C6，又构成一个小循环  
|

:004196B8 8B08  mov ecx, dword ptr [eax]

*******注：这里的 ECX 的取值如下 1、取输入的注册码第 7 位(7)  
                       2、取输入的注册码第 6 位(8)  
                       3、取输入的注册码第 8 位(8)  
                       4、取输入的注册码第 1 位(7)  
                       5、取输入的注册码第 9 位(7)  
                       6、取输入的注册码第 3 位(7)  
                       7、取输入的注册码第 10 位(8)  
                       8、取输入的注册码第 4 位(8)  
                       9、取输入的注册码第 5 位(7)  
           *****注：这里可以推断出输入的注册码第 2 位和第 11 位，可以为任意数字!  
             由于是定位取值，所在用假码 78787878787 将很定位，所以将假码改为 12345678901，重新来。  
             对应的取值变码是 768193045，记住这个对应关系后面用得到!

:004196BA 890A  mov dword ptr [edx], ecx  
   <===取得这些值依次放在 EDX 步进为 4 的位置上，ECX 的初始地址 8AF250 正好与最后比较的地址相同，故要仔细观察其变化  
:004196BC 46  inc esi  
:004196BD 83C204  add edx, 00000004  
:004196C0 83C004  add eax, 00000004  
:004196C3 83FE09  cmp esi, 00000009  <===初始值是 1，所以要循环 9 次。  
:004196C6 7EF0  jle 004196B8  <===循环 9 次后，不再跳转了。  
:004196C8 BB04000000  mov ebx, 00000004  
:004196CD BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:004196D2 3BDE  cmp ebx, esi  <===不相等  
:004196D4 7C2B  jl 00419701  <===不会跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196FF(C)  
            <===这一行到 004196FF，又构成一个小循环，

:004196D6 8D8574FFFFFF  lea eax, dword ptr [ebp+FFFFFF74]  
:004196DC 50  push eax  
:004196DD 8D559C  lea edx, dword ptr [ebp-64]  
:004196E0 52  push edx  
:004196E1 E8FAFCFFFF  call 004193E0  
            <===第一个 CALL 会使我们所关心的值(以 8AF250 为起始地址的步进为 4 的位置)有变化，F8 进去看得究竟吧  
:004196E6 83C408  add esp, 00000008  
:004196E9 8D4D9C  lea ecx, dword ptr [ebp-64]  
:004196EC 51  push ecx  
:004196ED 8D8574FFFFFF  lea eax, dword ptr [ebp+FFFFFF74]  
:004196F3 50  push eax  
:004196F4 E8E7FCFFFF  call 004193E0  
            <===第二个同样的 CALL 使我们所关心的值有变化  
:004196F9 83C408  add esp, 00000008  
:004196FC 46  inc esi  <===esi 为计数器，每次加 1  
:004196FD 3BDE  cmp ebx, esi  <===EBX=4  
:004196FF 7DD5  jge 004196D6  
            <===这里循环 4 次，但由于有了两个同样的 CALL，所以实际上 768193045(假码的变形)已经被处理了 8 次，如何处理的呢？我们再研究一下 call 004193E0(功能在下面有说)，  
             *****  将 768193045 处理后为：524472195  
             将 524472195 处理后为：502225745  
             将 502225745 处理后为：055757895  
             将 055757895 处理后为：550523445  
             将 550523445 处理后为：969148595  
             将 969148595 处理后为：815477145  
             将 815477145 处理后为：083225295  
             将 083225295 处理后为：826757845  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196D4(C)  
|

:00419701 BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:00419706 8D95F0FEFFFF  lea edx, dword ptr [ebp+FFFFFEF0]  
:0041970C 8D45A0  lea eax, dword ptr [ebp-60]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041974E(C)  
|  <===这一行到 0041974E，又构成一个小循环，将注册码处理成 826757845 后，与机器码依次比较，只要一次错了，就 OVER



# 汇编代码分析

## 1. 主循环逻辑

```
:0041970F 8B08            mov ecx, dword ptr [eax]  <===EAX 的初始地址 8AF250
:00419711 3B0A            cmp ecx, dword ptr [edx]  <===ECX 的初始地址 8AF1A0
:00419713 742F            je 00419744     <===这里必须跳走，不然到 00419742 一行无条件跳转，就 OVER 了
:00419715 33C0            xor eax, eax
:00419717 BA02000000      mov edx, 00000002
:0041971C 50              push eax
:0041971D 8D4508          lea eax, dword ptr [ebp+08]
:00419720 FF4F1C          dec [edi+1C]
:00419723 E8882F0A00      call 004BC6B0
:00419728 FF4F1C          dec [edi+1C]
:0041972B 8D450C          lea eax, dword ptr [ebp+0C]
:0041972E BA02000000      mov edx, 00000002
:00419733 E8782F0A00      call 004BC6B0
:00419738 58              pop eax
:00419739 8B17            mov edx, dword ptr [edi]
:0041973B 64891500000000 mov dword ptr fs:[00000000], edx
:00419742 EB39            jmp 0041977D
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00419713(C)`

```
:00419744 46              inc esi
:00419745 83C204          add edx, 00000004
:00419748 83C004          add eax, 00000004
:0041974B 83FE09          cmp esi, 00000009
:0041974E 7EBF            jle 0041970F         <===循环 9 次后，不再跳转了
:00419750 B001            mov al, 01
:00419752 BA02000000      mov edx, 00000002
:00419757 50              push eax
:00419758 8D4508          lea eax, dword ptr [ebp+08]
:0041975B FF4F1C          dec [edi+1C]
:0041975E E84D2F0A00      call 004BC6B0
:00419763 FF4F1C          dec [edi+1C]
:00419766 8D450C          lea eax, dword ptr [ebp+0C]
:00419769 BA02000000      mov edx, 00000002
:0041976E E83D2F0A00      call 004BC6B0
:00419773 58              pop eax         <===这里要求是 EAX 出来的不能为 0
:00419774 8B17            mov edx, dword ptr [edi]
:00419776 64891500000000 mov dword ptr fs:[00000000], edx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004194C8(U), :00419558(U), :00419634(U), :00419742(U)`

```
:0041977D 5F              pop edi
:0041977E 5E              pop esi
:0041977F 5B              pop ebx
:00419780 8BE5            mov esp, ebp
:00419782 5D              pop ebp
:00419783 C3              ret
```

## 2. 数字变形子程序 `004193E0`

\--------------F8 进入的数字变形 call 004193E0-------------------------------------------------

\* Referenced by a CALL at Addresses:

`:004196E1, :004196F4`

```
:004193E0 55              push ebp
:004193E1 8BEC            mov ebp, esp
:004193E3 53              push ebx
:004193E4 8B450C          mov eax, dword ptr [ebp+0C]
:004193E7 8B5508          mov edx, dword ptr [ebp+08]
:004193EA 8B4A24          mov ecx, dword ptr [edx+24]
:004193ED 894824          mov dword ptr [eax+24], ecx
:004193F0 8B4A20          mov ecx, dword ptr [edx+20]
:004193F3 3B4A24          cmp ecx, dword ptr [edx+24]   <===第 8 位与第 9 位相比
:004193F6 7D04            jge 004193FC                  <===相等就跳走
:004193F8 8342200A        add dword ptr [edx+20], 0000000A <===如果不相等，这里第 8 位就加上 A(防止第 8 位比第 9 小)
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004193F6(C)`

```
:004193FC 8B4A20          mov ecx, dword ptr [edx+20]
:004193FF 2B4A24          sub ecx, dword ptr [edx+24]
                          <===如果第 8 位与第 9 位相等，则 ECX=0；如果不等，则 ECX=第 8 位+第 9 位
:00419402 894820          mov dword ptr [eax+20], ecx     <===将 ECX 的值
:00419405 B907000000      mov ecx, 00000007     <===ECX 为计数器，初始值为 7
:0041940A 83C020          add eax, 00000020
:0041940D 83C21C          add edx, 0000001C
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041942A(C)`  <===这一行到 0041942A，又构成一个小循环

```
:00419410 8B1A            mov ebx, dword ptr [edx]
:00419412 3B18            cmp ebx, dword ptr [eax]
:00419414 7D03            jge 00419419
:00419416 83020A          add dword ptr [edx], 0000000A
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00419414(C)`

```
:00419419 8B1A            mov ebx, dword ptr [edx]
:0041941B 2B18            sub ebx, dword ptr [eax]
:0041941D 8958FC          mov dword ptr [eax-04], ebx
:00419420 49              dec ecx       <===计数器减去 1
:00419421 83C0FC          add eax, FFFFFFFC
:00419424 83C2FC          add edx, FFFFFFFC
:00419427 83F901          cmp ecx, 00000001
:0041942A 7DE4            jge 00419410     <===共循环 7 次
:0041942C 5B              pop ebx
:0041942D 5D              pop ebp
:0041942E C3              ret
```

*** 此小段功能是(为了说明问题，这里假设在新值尾部加个 0，这个 0 在得出新值后去掉)：将得到的 9 位数字，从尾部开始依次取值，例如取到第 N 位值，减去新值的第 N+1 的值(如果不够减，则第 N 位值就加上 10)，得到的数就是新值的第 N 位值!

*** 768193045 经过这段处理后，就应该是 524472195 (详细数字处理变化情况，在上面有列表)

\-----------------------------------------------------------------------------------

## 3. 算法总结

1.  将注册码 `12345678901`，按特定取位得到 `768193045`。
2.  将 `768193045` 经过 8 次的处理：
    ```
    768193045
    524472195
    502225745
    055757895
    550523445
    969148595
    815477145
    083225295
    826757845  <===将最后这个值与机器码比较
    ```
3.  反推的话就是：假设在旧值尾部加个 0，新值第 N 位=旧第 N 位值+旧第 N+1 位值(如果有进位，就只取个位数)，机器码做如此处理 8 次，再按当初取码位置反归位，就得到了注册码。
4.  我的机器码是：`64265214`，注册码就是：`4?82648431?` (`?`为任意数字)。

## 4. 注册机的制作

我的 C 语言没学完，就不在这现丑了，哪位 C 高手请帮忙写一段代码给我，谢谢!(TC 就可以了，其它的看不懂:)

## 5. 注册信息存放

注册信息存放在注册表(删除整个键值就成未注册版本)：

```ini
[HKEY_LOCAL_MACHINE\Software\Hearts\Register]

"user"="newlaos"
"pass"="48826484318" 4dd466d0e1e0cc123afbe504da2462ca
```
