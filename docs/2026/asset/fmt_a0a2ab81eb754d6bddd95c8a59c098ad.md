
# 浪漫情书3.1

## 浪漫情书3.1破解手记--算法分析

作者：`newlaos`[`CCG`][`DFCG`]

软件名称：浪漫情书3.1

最新版本：未知

文件大小：883KB

软件授权：共享软件

使用平台：Win9x/Me/2000/XP

发布公司：`http://go3.163.com/pyeditor/`　　`http://pyeditor.k12.net.cn/`

软件简介：“浪漫情书”的３.１版本。这一版本的“浪漫情书”中拥有许多方便快捷的功能，如：

１、快速的“自动写情书”功能；
２、强大的“拼音码查找情话”功能；
３、体贴的“情书精灵”功能；
４、多种风格的“Email情书功能”；
５、方便的“鼠标拖放式维护情话库”功能……

加密方式：`ASPACK`+注册码

功能限制：功能限制

PJ工具：`TRW20001.23`注册版，`W32Dasm8.93`黄金版，`FI2.5`

PJ日期：2003-04-17

作者`newlaos`申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 一、解决程序自校验代码部分：

1、用`FI2.5`查壳，发现加了`ASPACK`的壳，版本很老，用`TRW2000`进行手动脱壳，也可以用`PE-scan3.31`脱壳!

生成`UNPACK.exe`文件。可一运行程序就报被病毒感染后，退出! 说明此程序有自校验代码。先解决掉它。

2、用`W32Dasm`黄金修正版本进行静态反汇编，找到"文件读写错误,由于某些原因（例如：被病毒感染）"，双击来到下面代码段。

3、动态跟踪调试。请出国宝`TRW2000`，下断点`BPX 004972E8`(关键就在这，见下面动态代码分析)。这下好了，可以动态跟踪调试了。

```
:004972E8 E871E1F6FF
             call 0040545E
:004972ED A14C7B4A00
             mov eax, dword ptr [004A7B4C]
:004972F2
C60000                  mov byte
ptr [eax], 00
:004972F5 BA01000000
 mov edx, 00000001
:004972FA 8D85B4FEFFFF
   lea eax, dword ptr [ebp+FFFFFEB4]
:00497300 E8C8E3F6FF
           call 004056CD
:00497305 E89AB4F6FF
             call 004027A4
:0049730A 8D85B4FEFFFF
           lea eax, dword ptr [ebp+FFFFFEB4]
:00497310
E827E2F6FF              call 0040553C
:00497315
E88AB4F6FF              call 004027A4
:0049731A
3DD0050500              cmp eax, 000505D0  <===EAX为实际运行文件大小，505D0(329168)****改这一行了
:0049731F
7F17                    jg 00497338
<===如果文件大于这个数就说明出错了，把上面一行改为1M就可了(因为脱壳后的文件，中有820KB)。
:00497321 8D85B4FEFFFF
           lea eax, dword ptr [ebp+FFFFFEB4]
:00497327
E810E2F6FF              call 0040553C
:0049732C
E873B4F6FF              call 004027A4
:00497331
3D18FA0400              cmp eax, 0004FA18  <===EAX为实际运行文件大小，4FA18(326168)
:00497336
7D3D                    jge
00497375       <===如果文件小于这个数也出错。
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049731F(C)
|
:00497338
8D85B0FEFFFF            lea eax, dword ptr [ebp+FFFFFEB0]
:0049733E
8B8E98060000            mov ecx, dword ptr [esi+00000698]
*
Possible StringData Ref from Code Obj ->"文件读写错误,由于某些原因（例如：被病毒感染）"
               ->"改变了loveletter31.exe文件，为了保证您的电脑安"
               ->"
全，程式将会自动退出!
建议您访问下址重新下载"
               ->"“浪漫情书”软件：
"
         |
:00497344 BAA4734900
       mov edx, 004973A4
:00497349 E88EC9F6FF
         call 00403CDC
```

-------------------------------------------------------------------------------------------

## 二、解决软件注册部分注册全面分析：

1、用`W32Dasm`黄金修正版本进行静态反汇编，找到"注册成功！请重新启动浪漫情书……"，双击来到下面代码段。

2、动态跟踪调试。请出国宝`TRW2000`，下断点`BPX 00488FE5`(往往断点设在关键跳转前面一些的地方)。

```
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00489017(C)
|
:00488FE5
8B45F4                  mov eax,
dword ptr [ebp-0C]
:00488FE8 8A4418FF
   mov al, byte ptr [eax+ebx-01]
:00488FEC 3C30
             cmp al, 30
:00488FEE 7225
                 jb 00489015
:00488FF0
8B55F4                  mov edx,
dword ptr [ebp-0C]
:00488FF3 3C39
       cmp al, 39
:00488FF5 771E
           ja 00489015
:00488FF7 8D45EC
               lea eax, dword ptr [ebp-14]
:00488FFA
50                      push
eax
:00488FFB B901000000              mov
ecx, 00000001
:00489000 8BD3
     mov edx, ebx
:00489002 8B45F4
         mov eax, dword ptr [ebp-0C]
:00489005 E88AAEF7FF
             call 00403E94
:0048900A 8B55EC
                 mov edx, dword ptr
[ebp-14]
:0048900D 8D45F8
   lea eax, dword ptr [ebp-08]
:00489010 E883ACF7FF
         call 00403C98
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00488FEE(C),
:00488FF5(C)
|
:00489015 43
       inc ebx
:00489016 4E
             dec esi
:00489017 75CC
                 jne 00488FE5  <===向上跳的循环，主要功能是依次验证输入的注册码是不是全为数字
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488FE0(C)
|
:00489019
8D55F0                  lea edx,
dword ptr [ebp-10]
:0048901C 8B45FC
     mov eax, dword ptr [ebp-04]
:0048901F 8B80E0020000
         mov eax, dword ptr [eax+000002E0]
:00489025
E8065CFAFF              call 0042EC30
:0048902A
8B45F0                  mov eax,
dword ptr [ebp-10] <===EAX=newlaos
:0048902D 8D55EC
           lea edx, dword ptr [ebp-14]
:00489030
E83BFEFFFF              call 00488E70  <===关键算法CALL了，F8跟进
:00489035
8B45EC                  mov eax,
dword ptr [ebp-14] <===呵呵，真正的注册码
:00489038 8B55F8
         mov edx, dword ptr [ebp-08] <===此处为假码78787878，在这可以用KEYMAKE制作内存注册机
:0048903B
E860ADF7FF              call 00403DA0  <===这里的CALL，主功是将EAX和EDX进行对比
:00489040
0F8556010000            jne 0048919C   <===呵呵，关键跳转
*
Possible StringData Ref from Code Obj ->"注册成功！请重新启动浪漫情书……"
       |
:00489046 B834924800
     mov eax, 00489234
:0048904B E8008AFCFF
       call 00451A50
:00489050 33C0
             xor eax, eax
:00489052 55
                   push ebp
:00489053
682C914800              push 0048912C
:00489058
64FF30                  push dword
ptr fs:[eax]
:0048905B 648920
   mov dword ptr fs:[eax], esp
:0048905E 8D55E8
           lea edx, dword ptr [ebp-18]
:00489061
A1B47A4A00              mov eax, dword ptr
[004A7AB4]
:00489066 8B00
     mov eax, dword ptr [eax]
:00489068 E80F27FCFF
         call 0044B77C
```

此处删除一段注册信息保存代码

```
:00489190
E8BB88FCFF              call 00451A50
:00489195
E8E2A3F7FF              call 0040357C
:0048919A
EB0A                    jmp
004891A6
* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:00489040(C)
                       <===从这里得知关键跳转在哪里!
|
*
Possible StringData Ref from Code Obj ->"用户名或注册码有误！"
     |
:0048919C B898924800
   mov eax, 00489298
:004891A1 E8AA88FCFF
     call 00451A50
```

-----`00489030`
`call 00488E70`  关键算法CALL了，F8跟进，来到下列代码段------------------

```
:00488E70 55
                   push ebp
:00488E71
8BEC                    mov
ebp, esp
:00488E73 83C4F8
   add esp, FFFFFFF8
:00488E76 53
           push ebx
:00488E77 56
                 push esi
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488E34(C)
|
:00488E78
57                      push
edi
:00488E79 33C9
   xor ecx, ecx
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488E0A(C)
|
:00488E7B
894DF8                  mov dword
ptr [ebp-08], ecx
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488E08(C)
|
:00488E7E
8BF2                    mov
esi, edx
:00488E80 8945FC
   mov dword ptr [ebp-04], eax
:00488E83 8B45FC
           mov eax, dword ptr [ebp-04]
:00488E86
E8B9AFF7FF              call 00403E44
:00488E8B
33C0                    xor
eax, eax
:00488E8D 55
     push ebp
:00488E8E 68118F4800
     push 00488F11
:00488E93 64FF30
         push dword ptr fs:[eax]
:00488E96 648920
                 mov dword ptr fs:[eax],
esp
:00488E99 33DB
   xor ebx, ebx
:00488E9B 8D55F8
       lea edx, dword ptr [ebp-08]
:00488E9E A1E4784A00
             mov eax, dword ptr [004A78E4]
:00488EA3
8B00                    mov
eax, dword ptr [eax]
:00488EA5 E8D2D70000
   call 0049667C
:00488EAA 8B55F8
       mov edx, dword ptr [ebp-08] <===EDX=502713(机器码)
:00488EAD
8D45FC                  lea eax,
dword ptr [ebp-04]
:00488EB0 8B4DFC
     mov ecx, dword ptr [ebp-04] <===ECX=newlaos
:00488EB3
E824AEF7FF              call 00403CDC  <===这个CALL的作用，将两者合为502713newlaos
:00488EB8
8B45FC                  mov eax,
dword ptr [ebp-04]
:00488EBB E8D0ADF7FF
   call 00403C90   <===计算502713newlaos的长度为D(14位长)
:00488EC0
8BD0                    mov
edx, eax
:00488EC2 85D2
   test edx, edx
:00488EC4 7C17
         jl 00488EDD
:00488EC6 42
               inc edx
:00488EC7 33C0
                   xor eax,
eax
* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
|:00488EDB(C)
|
:00488EC9
8B4DFC                  mov ecx,
dword ptr [ebp-04]
:00488ECC 0FB64C01FF
```
