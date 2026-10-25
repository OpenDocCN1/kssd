
# Nokia 手机工具箱 v1.1.9

> 原文：[`www.pediy.com/kssd/pediy03/forum677.htm`](https://www.pediy.com/kssd/pediy03/forum677.htm)

请高手指点

软件名称：`Nktools`(手机工具箱) 繁体版 441KB

软件功能：Nokia 手机工具箱程序，`v1.1.9`，繁体中文未注册版。集合设置中文电话簿，中文简讯，备忘录，开工控模式，修改待机、分组图案，自定铃声，设置手机时间、闹铃时间，修改 `PIN1` 码为一体的 Nokia 手机程序。 支持手机连接线、电脑红外线。支持手机类型：`3210`、`3310`、`5110`、`5130`、`6110`、`6138`、`6150`、`8210`、`8250`、`8810`、`8850`。未注册版有功能限制，如工程模式、开机画面、铃声等不能使用，但可以用中文电本功能

注册形式：要求输入注册码，与手机的串号(`IMEI`)运算得出想应串号(`IMEI`) ，注册不成功不会出现出错信息，它会将注册码存放在软件目录的 `regcode.txt` 里，此类软件应该都是即时检测注册码的，而且每个功能都会检测注册码，所以如果要暴力的话很多地方要改，最好可以找到真注册码并写出注册机(具体是如何运算我还搞不清楚)

下载地址：`http://go.163.com/~nokiaz/software/NKToolSetup.exe`

不知大家有谁是用诺基亚的手机，`nokia` 只有 `3310` 和 `8250` 是可以用中文电话本的，但现在还有很多人都是用 `6110`、`6150`、`8210` 的，用了上面的软件就可以让这些手机都能用上中文电话本，而且还有很多其它功能，有兴趣的人试试吧，此软件要用红外线或者数据线。

此软件用 `upx 0.93` 压缩过，解压后反汇编，因为是繁体的所以我找不到有用的东西，但我找到个“`autoregister`”我设了中断后程序会自动中断，就算不作任何动作都会中断，所以我想程序是在即时检测注册码，但我跟下去却找不到关键地方(我只是个新手，功力 0.5 级)，以下的地方是我用 `trw200` 调试时找到的，我作了些注解，不知其它地方还有没有计算注册码的地方，在以下的过程中，我发现程序用我输入的假注册码和"060347221N”字符串作了运算，不知此字符串是不是程序将我的手机串号计算而来的，所以请大家帮验证一下，我只追到了程序的 `41A90C` 处到了后面我就晕掉了(我的汇编知识太菜了)，而且我也没追到“060347221N”是如何来的，所以请各位朋友帮个忙如果可以做个注册机出来就太好啦，实在不行也请帮忙在下面写一些注解吧!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0041A734(C)` |
|

```
:0041A739 C1FA02               sar edx, 02
:0041A73C 8955BC               mov dword ptr [ebp-44], edx
:0041A73F 33C9                 xor ecx, ecx
:0041A741 894DB0               mov dword ptr [ebp-50], ecx
:0041A744 8B45B0               mov eax, dword ptr [ebp-50]
:0041A747 3B45BC               cmp eax, dword ptr [ebp-44]
:0041A74A 0F8DC1000000         jnl 0041A811

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A80B(C)
|

:0041A750 8B55C8               mov edx, dword ptr [ebp-38]
:0041A753 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A756 8A048A               mov al, byte ptr [edx+4*ecx] <------假注册码首位数
:0041A759 04D0                 add al, D0
:0041A75B 8B55C8               mov edx, dword ptr [ebp-38]
:0041A75E 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A761 8A548A01             mov dl, byte ptr [edx+4*ecx+01]<----假注册码第二位
:0041A765 80C2D0               add dl, D0
:0041A768 C1E206               shl edx, 06
:0041A76B 0AC2                 or al, dl
:0041A76D 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A770 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A773 8B55C0               mov edx, dword ptr [ebp-40]
:0041A776 88040A               mov byte ptr [edx+ecx], al <---结果写入此地址
:0041A779 8B45BC               mov eax, dword ptr [ebp-44]
:0041A77C 85C0                 test eax, eax
:0041A77E 7903                 jns 0041A783
:0041A780 83C003               add eax, 00000003

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A77E(C)
|

:0041A783 C1F802               sar eax, 02
:0041A786 8D0C40               lea ecx, dword ptr [eax+2*eax]
:0041A789 41                    inc ecx
:0041A78A 83C104               add ecx, 00000004
:0041A78D 894D9C               mov dword ptr [ebp-64], ecx
:0041A790 33C0                 xor eax, eax
:0041A792 8945A0               mov dword ptr [ebp-60], eax
:0041A795 DF6D9C               fild qword ptr [ebp-64]   -----|此处何解
:0041A798 DC0524AA4100         fadd qword ptr [0041AA24]     |是否浮点
:0041A79E DD5DA4               fstp qword ptr [ebp-5C]   -----|运算？？
:0041A7A1 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7A4 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7A7 33C0                 xor eax, eax
:0041A7A9 8A448A01             mov al, byte ptr [edx+4*ecx+01]
:0041A7AD 83C0D0               add eax, FFFFFFD0
:0041A7B0 C1F802               sar eax, 02
:0041A7B3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7B6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7B9 8A548A02             mov dl, byte ptr [edx+4*ecx+02]
:0041A7BD 80C2D0               add dl, D0
:0041A7C0 C1E204               shl edx, 04
:0041A7C3 0AC2                 or al, dl
:0041A7C5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7C8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7CB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7CE 88440A01             mov byte ptr [edx+ecx+01], al
:0041A7D2 8B45C8               mov eax, dword ptr [ebp-38]
:0041A7D5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7D8 0FB6448802           movzx eax, byte ptr [eax+4*ecx+02]
:0041A7DD 83C0D0               add eax, FFFFFFD0
:0041A7E0 C1F804               sar eax, 04
:0041A7E3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7E6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7E9 8A548A03             mov dl, byte ptr [edx+4*ecx+03]
:0041A7ED 80C2D0               add dl, D0
:0041A7F0 C1E202               shl edx, 02
:0041A7F3 0AC2                 or al, dl
:0041A7F5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7F8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7FB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7FE 88440A02             mov byte ptr [edx+ecx+02], al
:0041A802 FF45B0               inc [ebp-50]
:0041A805 8B45B0               mov eax, dword ptr [ebp-50]
:0041A808 3B45BC               cmp eax, dword ptr [ebp-44]
:0041A80B 0F8C3FFFFFFF         jl 0041A750

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A74A(C)
|

:0041A811 8B4DBC               mov ecx, dword ptr [ebp-44]
:0041A814 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A817 894DBC               mov dword ptr [ebp-44], ecx
:0041A81A 8B45B8               mov eax, dword ptr [ebp-48]
:0041A81D 0345BC               add eax, dword ptr [ebp-44]
:0041A820 8945B4               mov dword ptr [ebp-4C], eax
:0041A823 8B55B4               mov edx, dword ptr [ebp-4C]
:0041A826 4A                   dec edx
:0041A827 8955B0               mov dword ptr [ebp-50], edx
:0041A82A 837DB000             cmp dword ptr [ebp-50], 00000000
:0041A82E 0F8CC4000000         jl 0041A8F8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A8F2(C)
|

:0041A834 33C9                 xor ecx, ecx
:0041A836 894DAC               mov dword ptr [ebp-54], ecx
:0041A839 EB36                 jmp 0041A871

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|
| :0041A878(C)
|

:0041A83B 8B45AC               mov eax, dword ptr [ebp-54]
:0041A83E 0345B0               add eax, dword ptr [ebp-50]
:0041A841 99                   cdq
:0041A842 F77DB8               idiv [ebp-48]
:0041A845 8B4DC4               mov ecx, dword ptr [ebp-3C]
:0041A848 8B45C0               mov eax, dword ptr [ebp-40]
:0041A84B 8B5DAC               mov ebx, dword ptr [ebp-54]
:0041A84E 0FB6441801           movzx eax, byte ptr [eax+ebx+01]
:0041A853 0FAF45B0             imul eax, dword ptr [ebp-50]
:0041A857 8B5DC0               mov ebx, dword ptr [ebp-40]
:0041A85A 8B75AC               mov esi, dword ptr [ebp-54]
```



#### 汇编代码段 1

```
:0041A85D 8A1C33              mov bl, byte ptr [ebx+esi]   

:0041A860 2AD8                sub bl, al   

:0041A862 2A1C11              sub bl, byte ptr [ecx+edx]   

:0041A865 8B45C0              mov eax, dword ptr [ebp-40]   

:0041A868 8B55AC              mov edx, dword ptr [ebp-54]   

:0041A86B 881C10              mov byte ptr [eax+edx], bl   

:0041A86E FF45AC              inc [ebp-54]   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A839(U)   

|   

:0041A871 8B4DBC              mov ecx, dword ptr [ebp-44]   

:0041A874 49                  dec ecx   

:0041A875 3B4DAC              cmp ecx, dword ptr [ebp-54]   

:0041A878 7FC1                jg 0041A83B   

:0041A87A 8B45BC              mov eax, dword ptr [ebp-44]   

:0041A87D 85C0                test eax, eax   

:0041A87F 7903                jns 0041A884   

:0041A881 83C003              add eax, 00000003   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A87F(C)   

|   

:0041A884 C1F802              sar eax, 02   

:0041A887 8D1440              lea edx, dword ptr [eax+2*eax]   

:0041A88A 42                  inc edx   

:0041A88B 83C204              add edx, 00000004   

:0041A88E 89559C              mov dword ptr [ebp-64], edx   

:0041A891 33C9                xor ecx, ecx   

:0041A893 894DA0              mov dword ptr [ebp-60], ecx   

:0041A896 DF6D9C              fild qword ptr [ebp-64]   

:0041A899 DC0524AA4100        fadd qword ptr [0041AA24]   

:0041A89F DD5DA4              fstp qword ptr [ebp-5C]   

:0041A8A2 8B45BC              mov eax, dword ptr [ebp-44]   

:0041A8A5 48                  dec eax   

:0041A8A6 8945AC              mov dword ptr [ebp-54], eax   

:0041A8A9 837DAC00            cmp dword ptr [ebp-54], 00000000   

:0041A8AD 7E3C                jle 0041A8EB   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A8E9(C)   

|   

:0041A8AF 8B45AC              mov eax, dword ptr [ebp-54]   

:0041A8B2 0345B0              add eax, dword ptr [ebp-50]   

:0041A8B5 99                  cdq   

:0041A8B6 F77DB8              idiv [ebp-48]   

:0041A8B9 8B4DC4              mov ecx, dword ptr [ebp-3C]   

:0041A8BC 8B45C0              mov eax, dword ptr [ebp-40]   

:0041A8BF 8B5DAC              mov ebx, dword ptr [ebp-54]   

:0041A8C2 0FB64418FF          movzx eax, byte ptr [eax+ebx-01]   

:0041A8C7 0FAF45B0            imul eax, dword ptr [ebp-50]   

:0041A8CB 8B5DC0              mov ebx, dword ptr [ebp-40]   

:0041A8CE 8B75AC              mov esi, dword ptr [ebp-54]   

:0041A8D1 8A1C33              mov bl, byte ptr [ebx+esi]   

:0041A8D4 2AD8                sub bl, al   

:0041A8D6 2A1C11              sub bl, byte ptr [ecx+edx]   

:0041A8D9 8B45C0              mov eax, dword ptr [ebp-40]   

:0041A8DC 8B55AC              mov edx, dword ptr [ebp-54]   

:0041A8DF 881C10              mov byte ptr [eax+edx], bl   

:0041A8E2 FF4DAC              dec [ebp-54]   

:0041A8E5 837DAC00            cmp dword ptr [ebp-54], 00000000   

:0041A8E9 7FC4                jg 0041A8AF   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A8AD(C)   

|   

:0041A8EB FF4DB0              dec [ebp-50]   

:0041A8EE 837DB000            cmp dword ptr [ebp-50], 00000000   

:0041A8F2 0F8D3CFFFFFF        jnl 0041A834   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A82E(C)   

|   
```

#### 汇编代码段 2

```
:0041A8F8 6A04                 push 00000004   

:0041A8FA FF75C0              push [ebp-40]   

:0041A8FD 8D4DB8              lea ecx, dword ptr [ebp-48]   

:0041A900 51                  push ecx   

:0041A901 E812BF0800          call 004A6818  <-----此 call 将运算结果放到另一地址   

:0041A906 83C40C              add esp, 0000000C   

:0041A909 8B45B8              mov eax, dword ptr [ebp-48]   

:0041A90C 3B45BC              cmp eax, dword ptr [ebp-44]<--此处和下面的比较不知是否   

:0041A90F 7E08                jle 0041A919                  是关键，我改动后没结果   

:0041A911 8B55BC              mov edx, dword ptr [ebp-44]   

:0041A914 8955B8              mov dword ptr [ebp-48], edx   

:0041A917 EB0B                jmp 0041A924   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A90F(C)   

|   

:0041A919 837DB800            cmp dword ptr [ebp-48], 00000000   

:0041A91D 7D05                jge 0041A924   

:0041A91F 33C9                xor ecx, ecx   

:0041A921 894DB8              mov dword ptr [ebp-48], ecx   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:0041A917(U), :0041A91D(C)   

|   
```

#### 汇编代码段 3

```
:0041A924 8B45C0              mov eax, dword ptr [ebp-40]   

:0041A927 8B55B8              mov edx, dword ptr [ebp-48]   

:0041A92A C644100400          mov [eax+edx+04], 00   

:0041A92F 66C745DC2000        mov [ebp-24], 0020   

:0041A935 DD45A4              fld qword ptr [ebp-5C]   

:0041A938 DC0D1CAA4100        fmul qword ptr [0041AA1C]   

:0041A93E DD5DA4              fstp qword ptr [ebp-5C]   

:0041A941 66C745DC2C00        mov [ebp-24], 002C   

:0041A947 8B55C0              mov edx, dword ptr [ebp-40]   

:0041A94A 83C204              add edx, 00000004   

:0041A94D 8D45F4              lea eax, dword ptr [ebp-0C]   

:0041A950 E857900900          call 004B39AC   

:0041A955 FF45E8              inc [ebp-18]   

:0041A958 8D55F4              lea edx, dword ptr [ebp-0C]   

:0041A95B 8B4508              mov eax, dword ptr [ebp+08]   

:0041A95E E8C5910900          call 004B3B28   

:0041A963 FF4DE8              dec [ebp-18]   

:0041A966 8D45F4              lea eax, dword ptr [ebp-0C]   

:0041A969 BA02000000          mov edx, 00000002   

:0041A96E E885910900          call 004B3AF8   

:0041A973 66C745DC0800        mov [ebp-24], 0008   

:0041A979 EB3C                jmp 0041A9B7   

:0041A97B 66C745DC3800        mov [ebp-24], 0038   

:0041A981 BA820C4C00          mov edx, 004C0C82   

:0041A986 8D45F0              lea eax, dword ptr [ebp-10]   

:0041A989 E81E900900          call 004B39AC   

:0041A98E FF45E8              inc [ebp-18]   

:0041A991 8D55F0              lea edx, dword ptr [ebp-10]   

:0041A994 8B4508              mov eax, dword ptr [ebp+08]   

:0041A997 E88C910900          call 004B3B28   

:0041A99C FF4DE8              dec [ebp-18]   

:0041A99F 8D45F0              lea eax, dword ptr [ebp-10]   

:0041A9A2 BA02000000          mov edx, 00000002   

:0041A9A7 E84C910900          call 004B3AF8   

:0041A9AC 66C745DC2800        mov [ebp-24], 0028   

:0041A9B2 E87F610900          call 004B0B36   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041A979(U)   

|   
```

#### 汇编代码段 4

```
:0041A9B7 DD45A4              fld qword ptr [ebp-5C]   

:0041A9BA DC052CAA4100        fadd qword ptr [0041AA2C]   

:0041A9C0 DD5DA4              fstp qword ptr [ebp-5C]   

:0041A9C3 FF75C0              push [ebp-40]   

:0041A9C6 E835B20800          call 004A5C00   

:0041A9CB 59                  pop ecx   

:0041A9CC FF75C8              push [ebp-38]   

:0041A9CF E82CB20800          call 004A5C00   

:0041A9D4 59                  pop ecx   

:0041A9D5 FF4DE8              dec [ebp-18]   

:0041A9D8 8D45F8              lea eax, dword ptr [ebp-08]   

:0041A9DB BA02000000          mov edx, 00000002   
```



# 汇编代码片段与逆向分析笔记

## 1. 函数尾声与调用

```assembly
:0041A9E0 E813910900           call 004B3AF8
:0041A9E5 FF4DE8               dec [ebp-18]
:0041A9E8 8D450C               lea eax, dword ptr [ebp+0C]
:0041A9EB BA02000000           mov edx, 00000002
:0041A9F0 E803910900           call 004B3AF8
:0041A9F5 8B4DCC               mov ecx, dword ptr [ebp-34]
:0041A9F8 64890D00000000       mov dword ptr fs:[00000000], ecx
:0041A9FF 5F                   pop edi
:0041AA00 5E                   pop esi
:0041AA01 5B                   pop ebx
:0041AA02 8BE5                 mov esp, ebp
:0041AA04 5D                   pop ebp
:0041AA05 C3                    ret
```

> **备注**：上次漏掉了一些重要的东西，这次补上。可能写得太多太罗嗦，请见谅。因为这是一边跟踪一边写的，所以很详细。毕竟是新手，很多不懂的都要做下记录，而且也想让一些新手看得懂。现丑了，这里跟的部分是注册码运算的过程。至于核心判断的地方还没找到，应该在下面的好几个 `call` 当中。但功力有限，只能先去补习一下。还请各位高手跟下去，先多谢啦。

## 2. 寄存器操作与跳转说明

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`0041A734(C)`

```assembly
:0041A739 C1FA02               sar edx, 02
:0041A73C 8955BC               mov dword ptr [ebp-44], edx
:0041A73F 33C9                 xor ecx, ecx
:0041A741 894DB0               mov dword ptr [ebp-50], ecx
:0041A744 8B45B0               mov eax, dword ptr [ebp-50]
:0041A747 3B45BC               cmp eax, dword ptr [ebp-44]
:0041A74A 0F8DC1000000         jnl 0041A811
```

## 3. 注册码运算循环

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`0041A80B(C)`

```assembly
:0041A750 8B55C8               mov edx, dword ptr [ebp-38]
:0041A753 8B4DB0               mov ecx, dword ptr [ebp-50]       ; 假注册码首位数
:0041A756 8A048A               mov al, byte ptr [edx+4*ecx]      ; 我用的是 67676767
:0041A759 04D0                 add al, D0                          ; 36=>6
:0041A75B 8B55C8               mov edx, dword ptr [ebp-38]        ; 注册码地址 C74454=>edx
:0041A75E 8B4DB0               mov ecx, dword ptr [ebp-50]        ; 0=>ecx
:0041A761 8A548A01             mov dl, byte ptr [edx+4*ecx+01]    ; 假注册码第二位 edx=C74437
:0041A765 80C2D0               add dl, D0                          ; edx=C74407
:0041A768 C1E206               shl edx, 06                         ; edx=31D101C0
:0041A76B 0AC2                 or al, dl                           ; eax=000000C6
:0041A76D 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A770 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A773 8B55C0               mov edx, dword ptr [ebp-40]
:0041A776 88040A               mov byte ptr [edx+ecx], al          ; 结果写入地址 C8C604
:0041A779 8B45BC               mov eax, dword ptr [ebp-44]        ; eax=2
:0041A77C 85C0                 test eax, eax
:0041A77E 7903                 jns 0041A783                          ; Jump!
:0041A780 83C003               add eax, 00000003
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`0041A77E(C)`

```assembly
:0041A783 C1F802               sar eax, 02
:0041A786 8D0C40               lea ecx, dword ptr [eax+2*eax]      ; 0
:0041A789 41                    inc ecx
:0041A78A 83C104               add ecx, 00000004                   ; 5
:0041A78D 894D9C               mov dword ptr [ebp-64], ecx         ; 地址 74F928
:0041A790 33C0                 xor eax, eax
:0041A792 8945A0               mov dword ptr [ebp-60], eax
:0041A795 DF6D9C               fild qword ptr [ebp-64]             ; 此处何解
:0041A798 DC0524AA4100         fadd qword ptr [0041AA24]           ; 是否浮点 <--46BACF74
:0041A79E DD5DA4               fstp qword ptr [ebp-5C]             ; 运算？？<--E23C5905
:0041A7A1 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7A4 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7A7 33C0                 xor eax, eax
:0041A7A9 8A448A01             mov al, byte ptr [edx+4*ecx+01]     ; 注册码第二位 7
:0041A7AD 83C0D0               add eax, FFFFFFD0                  ; 37=>6
:0041A7B0 C1F802               sar eax, 02                         ; eax=1
:0041A7B3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7B6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7B9 8A548A02             mov dl, byte ptr [edx+4*ecx+02]     ; 注册码第三位 6
:0041A7BD 80C2D0               add dl, D0                          ; edx=00C74406
:0041A7C0 C1E204               shl edx, 04                         ; edx=0C744060
:0041A7C3 0AC2                 or al, dl                           ; eax=61
:0041A7C5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7C8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7CB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7CE 88440A01             mov byte ptr [edx+ecx+01], al       ; 地址 C8C604
:0041A7D2 8B45C8               mov eax, dword ptr [ebp-38]
:0041A7D5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7D8 0FB6448802           movzx eax, byte ptr [eax+4*ecx+02]  ; 注册码第三位 6
:0041A7DD 83C0D0               add eax, FFFFFFD0                  ; 36=>6
:0041A7E0 C1F804               sar eax, 04                         ; eax=0
:0041A7E3 8B55C8               mov edx, dword ptr [ebp-38]
:0041A7E6 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7E9 8A548A03             mov dl, byte ptr [edx+4*ecx+03]     ; 注册码第四位 7
:0041A7ED 80C2D0               add dl, D0                          ; edx=00C74407
:0041A7F0 C1E202               shl edx, 02                         ; edx=31D101C
:0041A7F3 0AC2                 or al, dl                           ; eax=1C
:0041A7F5 8B4DB0               mov ecx, dword ptr [ebp-50]
:0041A7F8 8D0C49               lea ecx, dword ptr [ecx+2*ecx]
:0041A7FB 8B55C0               mov edx, dword ptr [ebp-40]
:0041A7FE 88440A02             mov byte ptr [edx+ecx+02], al
:0041A802 FF45B0               inc [ebp-50]
:0041A805 8B45B0               mov eax, dword ptr [ebp-50]         ; eax=1
:0041A808 3B45BC               cmp eax, dword ptr [ebp-44]         ; [ebp-44]=2
:0041A80B 0F8C3FFFFFFF         jl 0041A750                         ; 小于 2 就跳了
```

> **结果说明**：
> 程序将最后四位的注册码用同样方法运算，最后得结果放在地址 `C8C604` 中。
> 结果是：`C6 61 1C C6 61 1C` (结果 1)

## 4. 后续逻辑判断

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`0041A74A(C)`

```assembly
:0041A811 8B4DBC               mov ecx, dword ptr [ebp-44]
:0041A814 8D0C49               lea ecx, dword ptr [ecx+2*ecx]      ; ecx=6
:0041A817 894DBC               mov dword ptr [ebp-44], ecx
:0041A81A 8B45B8               mov eax, dword ptr [ebp-48]        ; eax=A
:0041A81D 0345BC               add eax, dword ptr [ebp-44]        ; eax=10
:0041A820 8945B4               mov dword ptr [ebp-4C], eax
:0041A823 8B55B4               mov edx, dword ptr [ebp-4C]
:0041A826 4A                    dec edx
:0041A827 8955B0               mov dword ptr [ebp-50], edx
:0041A82A 837DB000             cmp dword ptr [ebp-50], 00000000   ; F cmp 0
:0041A82E 0F8CC4000000         jl 0041A8F8                         ; 小于 0 就跳
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**
`0041A8F2(C)`

```assembly
:0041A834 33C9                 xor ecx, ecx
:0041A836 894DAC               mov dword ptr [ebp-54], ecx
:0041A839 EB36                jmp 0041A871                         ; Jump!!
```



# 汇编代码分析与注释

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A878(C) |
| |

:0041A83B `8B45AC` `mov eax, dword ptr [ebp-54]` <---eax=0  
:0041A83E `0345B0` `add eax, dword ptr [ebp-50]` <---eax=F  
:0041A841 `99` `cdq`  
:0041A842 `F77DB8` `idiv [ebp-48]` <---F mod A=>edx=5  
:0041A845 `8B4DC4` `mov ecx, dword ptr [ebp-3C]` <---ecx=C7Bd50  

ecx->060347221N----字串 1

:0041A848 `8B45C0` `mov eax, dword ptr [ebp-40]` <---上面运算结果 1 的地址  
:0041A84B `8B5DAC` `mov ebx, dword ptr [ebp-54]`  
:0041A84E `0FB6441801` `movzx eax, byte ptr [eax+ebx+01]` <---结果 1 的第二位 61  
:0041A853 `0FAF45B0` `imul eax, dword ptr [ebp-50]` <---61*F=5AF  
:0041A857 `8B5DC0` `mov ebx, dword ptr [ebp-40]`  
:0041A85A `8B75AC` `mov esi, dword ptr [ebp-54]`  
:0041A85D `8A1C33` `mov bl, byte ptr [ebx+esi]` <---结果 1 的第一位 C6=>ebx=C8C6C6  
:0041A860 `2AD8` `sub bl, al` <---C6-AF=17  
:0041A862 `2A1C11` `sub bl, byte ptr [ecx+edx]` <---17-37=E0(bl 减字串 1 的第 6 位)  
:0041A865 `8B45C0` `mov eax, dword ptr [ebp-40]`  
:0041A868 `8B55AC` `mov edx, dword ptr [ebp-54]`  
:0041A86B `881C10` `mov byte ptr [eax+edx], bl` <---将 E0 放入结果 1 的首地址中  
:0041A86E `FF45AC` `inc [ebp-54]` <---加 1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A839(U) |
| |

:0041A871 `8B4DBC` `mov ecx, dword ptr [ebp-44]` <----ecx=6  
:0041A874 `49` `dec ecx`  
:0041A875 `3B4DAC` `cmp ecx, dword ptr [ebp-54]` <----5 cmp 0  
:0041A878 `7FC1` `jg 0041A83B` <----大于就跳，循环 5 次最后结果放在结

果 1 的地址里结果是:E0 8B 50 E6 6F

1C(最后一位仍有用)----结果 2

:0041A87A `8B45BC` `mov eax, dword ptr [ebp-44]` <----eax=6  
:0041A87D `85C0` `test eax, eax`  
:0041A87F `7903` `jns 0041A884` <----Jump!!!  
:0041A881 `83C003` `add eax, 00000003`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A87F(C) |
| |

:0041A884 `C1F802` `sar eax, 02` <---eax=1  
:0041A887 `8D1440` `lea edx, dword ptr [eax+2*eax]` <---edx=3  
:0041A88A `42` `inc edx`  
:0041A88B `83C204` `add edx, 00000004`  
:0041A88E `89559C` `mov dword ptr [ebp-64], edx` <---edx=8  
:0041A891 `33C9` `xor ecx, ecx`  
:0041A893 `894DA0` `mov dword ptr [ebp-60], ecx`  
:0041A896 `DF6D9C` `fild qword ptr [ebp-64]` <---这三个地址里的数是(我不懂结果):8  
:0041A899 `DC0524AA4100` `fadd qword ptr [0041AA24]` <---46BACF74  
:0041A89F `DD5DA4` `fstp qword ptr [ebp-5C]` <---A35D67BA  
:0041A8A2 `8B45BC` `mov eax, dword ptr [ebp-44]`  
:0041A8A5 `48` `dec eax`  
:0041A8A6 `8945AC` `mov dword ptr [ebp-54], eax` <---eax=5  
:0041A8A9 `837DAC00` `cmp dword ptr [ebp-54], 00000000`  
:0041A8AD `7E3C` `jle 0041A8EB` <---小于或等于就跳

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A8E9(C) |
| |

:0041A8AF `8B45AC` `mov eax, dword ptr [ebp-54]` <---eax=5  
:0041A8B2 `0345B0` `add eax, dword ptr [ebp-50]` <---5+F=14  
:0041A8B5 `99` `cdq`  
:0041A8B6 `F77DB8` `idiv [ebp-48]` <---14/A=>edx=0  
:0041A8B9 `8B4DC4` `mov ecx, dword ptr [ebp-3C]` <---字串 1 首地址 C7Bd50  
:0041A8BC `8B45C0` `mov eax, dword ptr [ebp-40]`  
:0041A8BF `8B5DAC` `mov ebx, dword ptr [ebp-54]`  
:0041A8C2 `0FB64418FF` `movzx eax, byte ptr [eax+ebx-01]` <---结果 2 第五位 eax=6F  
:0041A8C7 `0FAF45B0` `imul eax, dword ptr [ebp-50]` <---6F*F=681  
:0041A8CB `8B5DC0` `mov ebx, dword ptr [ebp-40]`  
:0041A8CE `8B75AC` `mov esi, dword ptr [ebp-54]`  
:0041A8D1 `8A1C33` `mov bl, byte ptr [ebx+esi]` <---结果 2 最后一位 ebx=C8C61C  
:0041A8D4 `2AD8` `sub bl, al` <---1C-81=9B  
:0041A8D6 `2A1C11` `sub bl, byte ptr [ecx+edx]` <---9B-30=6B(bl 减字串 1 的第 1 位)

第二次是减字串 1 的最后一位第

三次是第 9 位第四次第 8 位第五次

第 7 位

:0041A8D9 `8B45C0` `mov eax, dword ptr [ebp-40]`  
:0041A8DC `8B55AC` `mov edx, dword ptr [ebp-54]`  
:0041A8DF `881C10` `mov byte ptr [eax+edx], bl` <---将结果放入结果 1 的地址最后一位里  
:0041A8E2 `FF4DAC` `dec [ebp-54]`  
:0041A8E5 `837DAC00` `cmp dword ptr [ebp-54], 00000000` <---4  
:0041A8E9 `7FC4` `jg 0041A8AF` <---大于就跳

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A8AD(C) |
| |

:0041A8EB `FF4DB0` `dec [ebp-50]` <---F-1  
:0041A8EE `837DB000` `cmp dword ptr [ebp-50], 00000000`  
:0041A8F2 `0F8D3CFFFFFF` `jnl 0041A834` <---跳了，循环 15 次最后结果是:

AF EB 75 01 F6 EA---结果 3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A82E(C) |
| |

:0041A8F8 `6A04` `push 00000004`  
:0041A8FA `FF75C0` `push [ebp-40]`  
:0041A8FD `8D4DB8` `lea ecx, dword ptr [ebp-48]`  
:0041A900 `51` `push ecx`  
:0041A901 `E812BF0800` `call 004A6818` <-----此 call 将运算结果放到 74F944 里，但程序只取了

前四个字，以下的比较搞不懂

:0041A906 `83C40C` `add esp, 0000000C`  
:0041A909 `8B45B8` `mov eax, dword ptr [ebp-48]` <---01 75 EB AF(结果 3)  
:0041A90C `3B45BC` `cmp eax, dword ptr [ebp-44]` <---00 00 00 06  
:0041A90F `7E08` `jle 0041A919` <--此处和下面的比较不知是否

是关键，我改动后没结果下面的几个

call 可能才是关键，我有时间再跟下去

:0041A911 `8B55BC` `mov edx, dword ptr [ebp-44]`  
:0041A914 `8955B8` `mov dword ptr [ebp-48], edx`  
:0041A917 `EB0B` `jmp 0041A924`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0041A90F(C) |
| |

:0041A919 `837DB800` `cmp dword ptr [ebp-48], 00000000`  
:0041A91D `7D05` `jge 0041A924`  
:0041A91F `33C9` `xor ecx, ecx`  
:0041A921 `894DB8` `mov dword ptr [ebp-48], ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :0041A917(U), :0041A91D(C) |
| |

:0041A924 `8B45C0` `mov eax, dword ptr [ebp-40]`  
:0041A927 `8B55B8` `mov edx, dword ptr [ebp-48]`  
:0041A92A `C644100400` `mov [eax+edx+04], 00`  
:0041A92F `66C745DC2000` `mov [ebp-24], 0020`  
:0041A935 `DD45A4` `fld qword ptr [ebp-5C]`  
:0041A938 `DC0D1CAA4100` `fmul qword ptr [0041AA1C]`  
:0041A93E `DD5DA4` `fstp qword ptr [ebp-5C]`



`:0041A941` `66C745DC2C00` `mov [ebp\-24], 002C`

`:0041A947` `8B55C0` `mov edx, dword ptr [ebp\-40]`

`:0041A94A` `83C204` `add edx, 00000004`

`:0041A94D` `8D45F4` `lea eax, dword ptr [ebp\-0C]`

`:0041A950` `E857900900` `call 004B39AC` <---此 `call` 将结果 3 后两位数进行处理，不过太复杂啦，搞不清楚~~~~~~ `4dd466d0e1e0cc123afbe504da2462ca`
