
# Flash Cam v1.78

> 原文：[`www.pediy.com/kssd/pediy05/pediy50094.htm`](https://www.pediy.com/kssd/pediy05/pediy50094.htm)

软件名称：Flash Cam v1.78

破解者：`fhmdw[BCG]`

破解工具：`trw2000` `keymake`

软件大小：1100KB

软件简介：Flash Cam 可以抓取影像为 Flash 文件，还可以设定测鼠移动路径、加入文字、外挂说明声效。

下载地址：`http://newhua.ruyi.com/down/FlashCamInstall.exe`

我在追注册码过程中共找到 4 个注册码，包括个人版的、家庭版的、专业版的以及企业版的。

```
:00512470 8B8F3C080000    mov ecx, dword ptr [edi+0000083C]
:00512476 8B9738080000    mov edx, dword ptr [edi+00000838]
:0051247C 8BC7            mov eax, edi
:0051247E E8A10A0000      call 00512F24<<---------关键 call,F8 追入
:00512483 84C0            test al, al<<---------注册标志
:00512485 7468            je 005124EF<<--------不跳则注册成功
```

====================================>>由 51247e 追入，来到这里

```
:00512F84 B920000000      mov ecx, 00000020
:00512F89 BAF19D0000      mov edx, 00009DF1
:00512F8E B81A000000      mov eax, 0000001A
:00512F93 E870FCFFFF      call 00512C08
:00512F98 8B45E8          mov eax, dword ptr [ebp-18]
:00512F9B 8B55FC          mov edx, dword ptr [ebp-04]<<--------下 d eax 得第一个注册码
:00512F9E E88D12EFFF      call 00404230<<---------真假比较
:00512FA3 7504            jne 00512FA9
:00512FA5 B301            mov bl, 01<<---------注册标志
:00512FA7 EB6D            jmp 00513016
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00512F78(C), :00512FA3(C)`

```
:00512FA9 FF75F4          push [ebp-0C]
:00512FAC FF75F0          push [ebp-10]
:00512FAF 8D45E4          lea eax, dword ptr [ebp-1C]
:00512FB2 50              push eax
:00512FB3 B921000000      mov ecx, 00000021
:00512FB8 BACB9C0000      mov edx, 00009CCB
:00512FBD B81B000000      mov eax, 0000001B
:00512FC2 E841FCFFFF      call 00512C08
:00512FC7 8B45E4          mov eax, dword ptr [ebp-1C]
:00512FCA 8B55FC          mov edx, dword ptr [ebp-04]<<--------下 d eax 得第二个注册码
:00512FCD E85E12EFFF      call 00404230<<---------真假比较
:00512FD2 750B            jne 00512FDF
:00512FD4 B301            mov bl, 01<<---------注册标志
:00512FD6 C6871608000001  mov byte ptr [edi+00000816], 01
:00512FDD EB37            jmp 00513016
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512FD2(C)`

```
:00512FDF 837DF800        cmp dword ptr [ebp-08], 00000000
:00512FE3 742F            je 00513014
:00512FE5 8B55FC          mov edx, dword ptr [ebp-04]
:00512FE8 8B45F8          mov eax, dword ptr [ebp-08]
:00512FEB E804FEFFFF      call 00512DF4<<--------第三个注册码，F8 追入
:00512FF0 8BD8            mov ebx, eax
:00512FF2 889F17080000    mov byte ptr [edi+00000817], bl
:00512FF8 84DB            test bl, bl
:00512FFA 750F            jne 0051300B
:00512FFC 8B55FC          mov edx, dword ptr [ebp-04]
:00512FFF 8B45F8          mov eax, dword ptr [ebp-08]
:00513002 E885FEFFFF      call 00512E8C<<--------第四个注册码(企业版的)，F8 追入
:00513007 8BD8            mov ebx, eax
:00513009 EB0B            jmp 00513016
```

=======================================>>由 512feb 追入，来到这里

```
:00512E44 8D4DEC          lea ecx, dword ptr [ebp-14]
:00512E47 BA47010000      mov edx, 00000147
:00512E4C 8B45FC          mov eax, dword ptr [ebp-04]
:00512E4F E888FEFFFF      call 00512CDC
:00512E54 8B45EC          mov eax, dword ptr [ebp-14]
:00512E57 8B55F8          mov edx, dword ptr [ebp-08]<<-------下 d eax 得第三个注册码
:00512E5A E8D113EFFF      call 00404230<<-------真假比较
:00512E5F 0F94C3          sete bl<<--------设标志位
```

=======================================>>由 513002 追入，来到这里

```
:00512EDC 8D4DEC          lea ecx, dword ptr [ebp-14]
:00512EDF BA86000000      mov edx, 00000086
:00512EE4 8B45FC          mov eax, dword ptr [ebp-04]
:00512EE7 E8F0FDFFFF      call 00512CDC<<----------计算注册码的 call
:00512EEC 8B45EC          mov eax, dword ptr [ebp-14]
:00512EEF 8B55F8          mov edx, dword ptr [ebp-08]<<-------下 d eax 得第四个注册码
:00512EF2 E83913EFFF      call 00404230<<-------真假比较
:00512EF7 0F94C3          sete bl<<--------设标志位
```

分析企业版注册码的算法：

======================================>>由 512ee7 追入，来到这里

```
:00512D09 BBDB070000      mov ebx, 000007DB
:00512D0E 8B45FC          mov eax, dword ptr [ebp-04]<<----大写的公司名
:00512D11 E80A14EFFF      call 00404120<<-------取用户名长度
:00512D16 8BF0            mov esi, eax
:00512D18 85F6            test esi, esi
:00512D1A 7E4A            jle 00512D66
:00512D1C BF01000000      mov edi, 00000001
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512D64(C)`

```
:00512D21 8B45FC          mov eax, dword ptr [ebp-04]
:00512D24 8A4C38FF        mov cl, byte ptr [eax+edi-01]<<-----公司名各个字符 ASC 码送 cl
:00512D28 33C0            xor eax, eax<<-------清零
:00512D2A 8AC1            mov al, cl
:00512D2C 8D570D          lea edx, dword ptr [edi+0D]
:00512D2F F7EA            imul edx<<------ax 乘以 dx，结果送 ax
:00512D31 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D33 8BC3            mov eax, ebx
:00512D35 BBFFC99A3B      mov ebx, 3B9AC9FF<<------0x3b9ac9ff 送入 bx
:00512D3A 99              cdq<<---------扩展
:00512D3B F7FB            idiv ebx<<-------ax 除以 bx，余数送 dx
:00512D3D 8BDA            mov ebx, edx
:00512D3F 8B45FC          mov eax, dword ptr [ebp-04]
:00512D42 80F145          xor cl, 45<<------cl 异或 0x45
:00512D45 33C0            xor eax, eax<<------清零
:00512D47 8AC1            mov al, cl
:00512D49 F76DF8          imul [ebp-08]
:00512D4C 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D4E 8BC3            mov eax, ebx
:00512D50 B9FFC99A3B      mov ecx, 3B9AC9FF<<------0x3b9a9c9ff 送入 cx
:00512D55 99              cdq
:00512D56 F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D58 8BDA            mov ebx, edx
:00512D5A 69C72B300600    imul eax, edi, 0006302B<<-----edi 乘以 0x6302b,结果送 ax
:00512D60 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D62 47              inc edi<<-------计数器
:00512D63 4E              dec esi
:00512D64 75BB            jne 00512D21<<-------未取完则继续取
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512D1A(C)`

```
:00512D66 8BC3            mov eax, ebx<<--------bx 入送 ax
:00512D68 B9FFE0F505      mov ecx, 05F5E0FF<<-------0x5f5e0ff 送 cx
:00512D6D 99              cdq
:00512D6E F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D70 8BDA            mov ebx, edx
:00512D72 8BC3            mov eax, ebx
:00512D74 B906000000      mov ecx, 00000006<<------6 送入 cx
:00512D79 99              cdq
:00512D7A F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D7C 83C241          add edx, 00000041<<------dx 加上 0x41,结果送 dx
:00512D7F 8855F3          mov byte ptr [ebp-0D], dl
:00512D82 895DEC          mov dword ptr [ebp-14], ebx
:00512D85 DB45EC          fild dword ptr [ebp-14]
```



## 汇编代码片段

```
:00512D88 83C4F4        add esp, FFFFFFF4   
:00512D8B DB3C24        fstp tbyte ptr [esp]   
:00512D8E 9B            wait   
========================================
```

### 总结

`name`: `BCG`

`code`: `A02-440-368`

最后用 `keymake` 做个另类注册机：

1. 按“浏览”按钮，选择 `FlashCam.exe`
2. 第一个中断地址：
    中断地址：`51247e`
    中断次数：`1`
    第一字节：`e8`
    指令长度：`5`
   第二个中断地址：
    中断地址：`512eef`
    中断次数：`1`
    第一字节：`8b`
    指令长度：`3`
3. 选“内存方式”，并勾选 `EAX`
4. 点击生成 `EXE` 文件

企业版的注册码好像是通用的，本菜鸟水平有限，希望大虾们写出它的 `keygen` 并贴上源码，好让我们初学者能学上一手。

整理于 2002.4.23 11:47

# FlashCam V1.75 的破解

`FlashCam` 可以抓取影像为 `Flash` 文件，还可以设定测鼠移动路径、加入文字、外挂说明声效。(取自海阔天空下载站的说明)

看看 `FlashCam1.75` 的帮助文件，它分未注册试用版和注册版(个人版、专业版、企业版)四种。未注册版本有 30 天的使用期限，且启动时有延时窗口，用它制作的影像文件上有广告等。注册版则没有以上限制，并且有些加强功能。

个人版、专业版、企业版功能依次加强，个人版和专业版只能在单个机器上注册使用，而企业版则可以在任意机器上注册使用。

下面我们来破解它，找出注册码来。

- **目标**：`FlashCam V1.75`
- **工具**：`TRW2000` & `DeDe`
- **作者**：`chilly200[BCG]`
- **时间**：2002.4.5

`FlashCam1.75` 是用 `ASpack2.x-modified` 加壳的，用 `Aspackdie1.3` 轻松脱掉。

由于是 `Delphi` 程序，用 `DeDe` 反编译，很容易找到注册的地方，如下：

```
018F:005114EE  PUSH    DWORD [EBP-0C]
018F:005114F1  PUSH    DWORD [EBP-10]
018F:005114F4  LEA     EAX,[EBP-18]
018F:005114F7  PUSH    EAX
018F:005114F8  MOV     ECX,20
018F:005114FD  MOV     EDX,9DF1
018F:00511502  MOV     EAX,1A
018F:00511507  CALL    0051117C      <-- 计算个人版注册码的 CALL
018F:0051150C  MOV     EAX,[EBP-18]  <-- 本机个人版正确注册码
018F:0051150F  MOV     EDX,[EBP-04]
018F:00511512  CALL    0040419C      <-- 比较个人版注册码
018F:00511517  JNZ     0051151D      <-- 不跳即注册成功，暴破点！
018F:00511519  MOV     BL,01
018F:0051151B  JMP     SHORT 0051158A
018F:0051151D  PUSH    DWORD [EBP-0C]
018F:00511520  PUSH    DWORD [EBP-10]
018F:00511523  LEA     EAX,[EBP-1C]
018F:00511526  PUSH    EAX
018F:00511527  MOV     ECX,21
018F:0051152C  MOV     EDX,9CCB
018F:00511531  MOV     EAX,1B
018F:00511536  CALL    0051117C       <-- 计算专业版注册码
018F:0051153B  MOV     EAX,[EBP-1C]    <-- 本机专业版正确注册码
018F:0051153E  MOV     EDX,[EBP-04]
018F:00511541  CALL    0040419C       <-- 比较专业版注册码
018F:00511546  JNZ     00511553       <-- 不跳即成专业版，暴破点！
018F:00511548  MOV     BL,01
018F:0051154A  MOV     BYTE [EDI+07E9],01  <-- 专业版多了这个
018F:00511551  JMP     SHORT 0051158A
018F:00511553  CMP     DWORD [EBP-08],BYTE +00  <-- 下面是企业版注册
018F:00511557  JZ      00511588                 <-- 要有公司名
018F:00511559  MOV     EDX,[EBP-04]
018F:0051155C  MOV     EAX,[EBP-08]
018F:0051155F  CALL    00511368       <-- 计算比较企业版注册码
018F:00511564  MOV     EBX,EAX
018F:00511566  MOV     [EDI+07EA],BL   <-- 企业版多了这个(BL=1 成功)
018F:0051156C  TEST    BL,BL           <-- 成为企业版的暴破点！
018F:0051156E  JNZ     0051157F
018F:00511570  MOV     EDX,[EBP-04]    <-- 下面好像又成个人版了
018F:00511573  MOV     EAX,[EBP-08]      (这个个人版与机器无关?)
018F:00511576  CALL    00511400       <-- 计算比较注册码
018F:0051157B  MOV     EBX,EAX         <-- EAX=1 个人版注册成功
018F:0051157D  JMP     SHORT 0051158A
018F:0051157F  MOV     BYTE [EDI+07E9],01
018F:00511586  JMP     SHORT 0051158A
018F:00511588  XOR     EBX,EBX
018F:0051158A  XOR     EAX,EAX
018F:0051158C  POP     EDX
018F:0051158D  POP     ECX
018F:0051158E  POP     ECX
```

程序在注册、显示 `About`、启动界面、延时窗口等地方都是调用的这段代码，因此在以上各个暴破点上修改一下就可以分别暴破成个人、专业、企业版。

或者从上面标注的地方也可以看到本机上的正确的个人版和专业版注册码，可以抄下来进行注册。但是这是和机器及安装目录有关的，换了地方就没用了。

我们还是来找企业版注册码吧，这个应该是放之四海皆准的吧。

跟进 `018F:0051155F CALL 00511368` `<-- 计算比较企业版注册码`

```
------------------------------------- Corporate Edition ($699 US)
       :
018F:005113B8  LEA     ECX,[EBP-14]
018F:005113BB  MOV     EDX,0147
018F:005113C0  MOV     EAX,[EBP-04]    <-- 公司名 BCG
018F:005113C3  CALL    00511250        <-- 计算企业版注册码的 CALL
018F:005113C8  MOV     EAX,[EBP-14]    <-- 真的注册码 D02-443-263
018F:005113CB  MOV     EDX,[EBP-08]
018F:005113CE  CALL    0040419C        <-- 比较注册码
018F:005113D3  SETZ    BL
018F:005113D6  XOR     EAX,EAX
018F:005113D8  POP     EDX
018F:005113D9  POP     ECX
018F:005113DA  POP     ECX
```

嘿嘿，现在我们就可以用下面的注册码注册成价值 $699 的企业版了！

**注册码**：`D02-443-263`

**公司名**：`BCG`

---

下面是企业版注册码的算法

```
       :
018F:00511259  XOR     EBX,EBX
018F:0051125B  MOV     [EBP-18],EBX    <-- 零
018F:0051125E  MOV     [EBP-0C],ECX    <-- 注册码的存放地址
018F:00511261  MOV     [EBP-08],EDX    <-- 常量 0x147
018F:00511264  MOV     [EBP-04],EAX    <-- 公司名
018F:00511267  MOV     EAX,[EBP-04]
018F:0051126A  CALL    00404240
018F:0051126F  XOR     EAX,EAX
018F:00511271  PUSH    EBP
018F:00511272  PUSH    DWORD 00511344
018F:00511277  PUSH    DWORD [FS:EAX]
018F:0051127A  MOV     [FS:EAX],ESP
018F:0051127D  MOV     EBX,07DB
018F:00511282  MOV     EAX,[EBP-04]
018F:00511285  CALL    0040408C        <-- 取公司名长度 len
018F:0051128A  MOV     ESI,EAX
018F:0051128C  TEST    ESI,ESI
018F:0051128E  JNG     005112DA
018F:00511290  MOV     EDI,01          <-- EDI= 1 to len
018F:00511295  MOV     EAX,[EBP-04]
018F:00511298  MOV     CL,[EAX+EDI-01]  <-- 循环取公司名的字符 CL
018F:0051129C  XOR     EAX,EAX
018F:0051129E  MOV     AL,CL
018F:005112A0  LEA     EDX,[EDI+0D]    <-- EDX=EDI+0D
018F:005112A3  IMUL    EDX
018F:005112A5  ADD     EBX,EAX          <-- EBX=EBX+EDX*CL
018F:005112A7  MOV     EAX,EBX          <-- EAX=EBX
018F:005112A9  MOV     EBX,3B9AC9FF
018F:005112AE  CDQ
018F:005112AF  IDIV    EBX
018F:005112B1  MOV     EBX,EDX          <-- EBX=EAX/0x3B9AC9FF 的余数
018F:005112B3  MOV     EAX,[EBP-04]
018F:005112B6  XOR     CL,45
018F:005112B9  XOR     EAX,EAX
018F:005112BB  MOV     AL,CL            <-- AL=CL 异或 0x45
018F:005112BD  IMUL    DWORD [EBP-08]
018F:005112C0  ADD     EBX,EAX          <-- EBX=EBX+AL*0x147
018F:005112C2  MOV     EAX,EBX          <-- EAX=EBX
018F:005112C4  MOV     ECX,3B9AC9FF
018F:005112C9  CDQ
018F:005112CA  IDIV    ECX
018F:005112CC  MOV     EBX,EDX          <-- EBX=EAX/0x3B9AC9FF 的余数
018F:005112CE  IMUL    EAX,EDI,0006302B
018F:005112D4  ADD     EBX,EAX          <-- EBX=EBX+EDI*0x6302B
018F:005112D6  INC     EDI              <-- EDI+1
018F:005112D7  DEC     ESI
018F:005112D8  JNZ     00511295         <-- 循环所有字符
018F:005112DA  MOV     EAX,EBX          <-- 计算结果 EBX 放入 EAX
018F:005112DC  MOV     ECX,05F5E0FF
018F:005112E1  CDQ
018F:005112E2  IDIV    ECX
018F:005112E4  MOV     EBX,EDX          <-- EBX=EAX/0x05F5E0FF 的余数
018F:005112E6  MOV     EAX,EBX             EBX 的十进制数就是注册码
018F:005112E8  MOV     ECX,06               的后 7 位数。
018F:005112ED  CDQ
018F:005112EE  IDIV    ECX
018F:005112F0  ADD     EDX,BYTE +41    <-- DL=EBX/6 的余数+0x41
018F:005112F3  MOV     [EBP-0D],DL      <-- DL 就是注册码的第一个字符
018F:005112F6  MOV     [EBP-14],EBX
018F:005112F9  FILD    DWORD [EBP-14]
018F:005112FC  ADD     ESP,BYTE -0C
018F:005112FF  FSTP    TWORD [ESP]
018F:00511302  WAIT
       :
```

到这，企业版的注册码就算出来了。呵呵，也不知道对不对，哪位朋友有兴趣来写个注册机吧。

另外，这个软件在 `Windows` 的 `System` 目录里放了一个假的 `DLL` 文件 `msrgl90.dll`，注册信息以及安装时间都是存在这个文件里面的，比较有意思，呵呵。

请将下面源程序保存为 `.rek` 文件，用注册机编写器编译，注册机编写器 1.73 测试通过!

```
.const
.data
szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:ljyljx@163.com",0
szErrMess  db "输入的序列号不正确！",0
szFmt      db "%0.8u",0
data1      dd 86h,0    ;这个数是我猜的,破文中没写出来
szTemp     db 10 dup(0)
szCode     db 14 dup(0)

.code
mov ebx,7DBh
invoke lstrlen,eax
mov esi,eax
mov edi,1
PRO1:
lea eax,hInput1
```



```
mov cl,byte ptr [eax+edi-1]

xor eax,eax

mov al,cl

lea edx,dword ptr [edi+0Dh]

imul edx

add ebx,eax

mov eax,ebx

mov ebx,3B9AC9FFh

cdq

idiv ebx

mov ebx,edx

xor cl,45h

xor eax,eax

mov al,cl

imul data1

add ebx,eax

mov eax,ebx

mov ecx,3B9AC9FFh

cdq

idiv ecx

mov ebx,edx

imul eax,edi,0006302Bh

add ebx,eax

inc edi

dec esi

jne PRO1

mov eax,ebx

mov ecx,05F5E0FFh

cdq

idiv ecx

mov ebx,edx

mov eax,ebx

mov ecx,6

cdq

idiv ecx

add edx,41h

mov byte ptr szTemp,dl

invoke wsprintf,addr szTemp+1,addr szFmt,ebx

mov edx,3

lea esi,szTemp

lea edi,szCode

PRO2:

mov ecx,3

rep movsb

mov [edi],byte ptr "-"

inc edi

dec edx

jnz PRO2

mov [edi-1],byte ptr 0

lea eax,szCode
```

```
    _/_/

  _/        _/_/  _/_/  _/_/

   _/_/   _/   _/  _/   _/   _/

       _/  _/   _/  _/   _/   _/

  _/_/    _/_/  _/   _/   _/

                        Sam.com

                    2:02 2002-4-24  4dd466d0e1e0cc123afbe504da2462ca
```
