
# TxEdit 4.6

> 原文：`www.pediy.com/kssd/pediy03/forum1186.htm`

《TxEdit 4.6》的注册码破解

目标：`TxEdit 4.6`

一个简单易用的文本编译器，可同时编译多个文件，可进行文件查找及替换，支持最大 10 兆的文件，支持拖放，可浏览 `Mac` 和 `UNIX` 的文本，可设置自己喜爱工具条，可预先设置 9 个最多 2000 字符的宏和 9 个常用工具。

- 软件类别：共享软件(30 天试用版)
- 注册费用：`$20.00`
- 运行环境：`Win95/98/NT`
- 软件大小：`191 KB`
- 下载网址：`http://www.execpc.com/~sbd`

作者：`RATARICE[BCG]`

工具：`TRW2000 1.22`

过程：

## 一、运行软件，可以输入注册码（非常好！）

## 二、瞎填上

- User Name: `RATARICE[BCG]`
- Organization: `peNcil grOup`
- Registration: `999999999999`

## 三、启动 TRW，下 bpx hmemcpy。点“OK”，被拦，下 pmodule，追到下面代码：

```assembly
:004187A8 E8836A0000            call 0041F230 ----------------------->计算注册码
:004187AD 83C404                add esp, 00000004
:004187B0 8BE8                  mov ebp, eax
:004187B2 56                    push esi
:004187B3 E898430000            call 0041CB50
:004187B8 83C404                add esp, 00000004
:004187BB 3D92A71901            cmp eax, 0119A792
:004187C0 7518                  jne 004187DA
* Possible StringData Ref from Data Obj ->"Gregory Braun"
                          |
:004187C2 686CB74200            push 0042B76C
* Reference To: KERNEL32.lstrcpyA, Ord:0296h
                          |
:004187C7 8B2DC4664400          mov ebp, dword ptr [004466C4]
:004187CD 56                    push esi
:004187CE FFD5                 call ebp
* Possible StringData Ref from Data Obj ->"Software Design"
                          |
:004187D0 6854B74200            push 0042B754
:004187D5 53                    push ebx
:004187D6 FFD5                 call ebp
:004187D8 EB07                 jmp 004187E1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004187C0(C)
|
:004187DA 3D3CCE5F0D            cmp eax, 0D5FCE3C
:004187DF 750C                  jne 004187ED
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004187D8(U)
|
:004187E1 53                    push ebx
:004187E2 56                    push esi
:004187E3 E838380000            call 0041C020
:004187E8 83C408                add esp, 00000008
:004187EB 8BE8                  mov ebp, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004187DF(C)
|
:004187ED 53                    push ebx
:004187EE 56                    push esi
:004187EF E82C380000            call 0041C020 ------------------------------>计算 name 和 organ
:004187F4 83C408                add esp, 00000008
:004187F7 3BC5                  cmp eax, ebp ------------------------------->关键比较
:004187F9 741E                  je 00418819 -------------------------------->关键跳转
                                                                          不跳则死！！！
```

## 四、追入上面的两个 CALL：

先追入 `004187a8`，代码如下：

```assembly
* Referenced by a CALL at Addresses:
|:004187A8  , :0041F2E5  , :004226D8  , :0042270B  , :0042273D
|
:0041F230 53                    push ebx
:0041F231 56                    push esi
:0041F232 8B74240C              mov esi, dword ptr [esp+0C]
:0041F236 57                    push edi
:0041F237 55                    push ebp
:0041F238 BF01000000            mov edi, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F26E(U)
|
:0041F23D 393D9CE34200          cmp dword ptr [0042E39C], edi
:0041F243 7E11                  jle 0041F256
:0041F245 6A08                  push 00000008
:0041F247 33C0                  xor eax, eax
:0041F249 8A06                  mov al, byte ptr [esi]
:0041F24B 50                    push eax
:0041F24C E89FE2FFFF            call 0041D4F0
:0041F251 83C408                add esp, 00000008
:0041F254 EB13                 jmp 0041F269
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F243(C)
|
:0041F256 33D2                  xor edx, edx
* Possible StringData Ref from Data Obj ->" (((((           "
                       ->" H 剟剟剟剟剟亖亖亖"
                       ->"倐倐倐"
                       ->" "
                    |
:0041F258 8B0D90E14200          mov ecx, dword ptr [0042E190]
:0041F25E 8A16                  mov dl, byte ptr [esi]
:0041F260 33C0                  xor eax, eax
:0041F262 668B0451             mov ax, word ptr [ecx+2*edx]
:0041F266 83E008                and eax, 00000008
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F254(U)
|
:0041F269 85C0                  test eax, eax
:0041F26B 7403                  je 0041F270
:0041F26D 46                    inc esi
:0041F26E EBCD                 jmp 0041F23D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F26B(C)
|
:0041F270 33DB                  xor ebx, ebx
:0041F272 8A1E                  mov bl, byte ptr [esi]
:0041F274 46                    inc esi
:0041F275 8BFB                  mov edi, ebx
:0041F277 83FB2D                cmp ebx, 0000002D
:0041F27A 7405                  je 0041F281
:0041F27C 83FB2B                cmp ebx, 0000002B
:0041F27F 7505                  jne 0041F286
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F27A(C)
|
:0041F281 33DB                  xor ebx, ebx
:0041F283 8A1E                  mov bl, byte ptr [esi]
:0041F285 46                    inc esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F27F(C)
|
:0041F286 33ED                  xor ebp, ebp
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F2BF(U)
|
:0041F288 833D9CE3420001        cmp dword ptr [0042E39C], 00000001
:0041F28F 7E0D                  jle 0041F29E
:0041F291 6A04                  push 00000004
:0041F293 53                    push ebx
:0041F294 E857E2FFFF            call 0041D4F0
:0041F299 83C408                add esp, 00000008
:0041F29C EB0F                 jmp 0041F2AD
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041F28F(C)
|
* Possible StringData Ref from Data Obj ->" (((((           "
                       ->" H 剟剟剟剟剟亖亖亖"
                       ->"倐倐倐"
                       ->" "
                    |
:0041F29E 8B0D90E14200          mov ecx, dword ptr [0042E190]
:0041F2A4 33C0                  xor eax, eax
:0041F2A6 668B0459             mov ax, word ptr [ecx+2*ebx] ------------
:0041F2AA 83E004                and eax, 00000004                      |
* Referenced by a (U)nconditional or (C)onditional Jump at Address:       |关键计算
|:0041F29C(U)
|                                                                           |
:0041F2AD 85C0                  test eax, eax
:0041F2AF 7410                  je 0041F2C1                                |
:0041F2B1 8D44AD00              lea eax, dword ptr [ebp+4*ebp]
:0041F2B5 46                    inc esi                                    |
:0041F2B6 8D6C43D0              lea ebp, dword ptr [ebx+2*eax-30]
:0041F2BA 33DB                  xor ebx, ebx                               |
:0041F2BC 8A5EFF                mov bl, byte ptr [esi-01]
:0041F2BF EBC7                 jmp 0041F288 ----------------------------
```
