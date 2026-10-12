
# LeapFTP 2.62

> 原文：[`www.pediy.com/kssd/pediy02/forum219.htm`](https://www.pediy.com/kssd/pediy02/forum219.htm)

### 软件信息

**软件版本**：`2.62`  
**软件大小**：`794 KB`  
**运行平台**：Windows

### 软件简介

如果提起 `LeapFTP`，我想不少的网迷都有点陌生，的确 `LeapFTP` 是一个比较新的软件，但是它却在许多方面超过了它的前辈们。在国内不少站点的九九年共享软件评比中，`LeapFTP` 被评为最佳 `FTP` 软件。

`LeapFTP` 除具有其他 `FTP` 软件的大多数特性，比如队列功能、远程文件编辑、支持防火墙/代理服务器、高级脚本语言、剪贴板监视，`LeapFTP` 还具有以下特点：

- 定制命令
- 目录列表缓存
- 批量文件属性更改（`CHMOD`）
- 从 `Windows` 资源管理器拖放文件到服务器
- 移动远程文件
- 保存/打印远程文件目录
- 反空闲（`Anti-Idle`）以防服务器断开连接等功能

另外，它还可以按照分类快速登录到需要的服务器上。这个软件在上传功能方面非常强劲，是维护主页的最佳利器。我在对它进行试用了几次后渐渐地被 `LeapFTP` 所吸引，但是我也发觉它有一个致命的弱点：它不具有自动重拨、服务器之间传输等功能的话，按照现在国内的网络条件，它是很不适宜的......

**软件来源**：软件  
**日期**：`2000-08`

### 注册信息

**用户名**：`LiuTong`  
**注册码**：`ABC1-EF2H-I3KL-4NOP`

(注册码要求:在第 5,10,15 位为`-`;在 4 的倍数位为数字;其他位为大写字母)

### 破解步骤

1. 设断点 `bpx hmemcpy`
2. 中断后按 `F12` 键 12 次，很容易找到注册码的地址`*****:******`
3. 设断点 `BPM *****:******`，中断后按 `F10` 便可找到下面的程序:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048824A(C)
|
:004881C4 8BC3                 mov eax, ebx
:004881C6 2503000080           and eax, 80000003
:004881CB 7905                 jns 004881D2
:004881CD 48                   dec eax
:004881CE 83C8FC               or eax, FFFFFFFC
:004881D1 40                   inc eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004881CB(C)
|
:004881D2 85C0                 test eax, eax
:004881D4 7516                 jne 004881EC
:004881D6 8B45FC               mov eax, dword ptr [ebp-04]
:004881D9 8A4418FF             mov al, byte ptr [eax+ebx-01]
:004881DD E84EFFFFFF           call 00488130 <----判断是否是数字,若是 AL=1
:004881E2 84C0                 test al, al
:004881E4 0F84C5000000         je 004882AF
:004881EA EB22                 jmp 0048820E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004881D4(C)
|
:004881EC 8BC3                 mov eax, ebx
:004881EE B905000000           mov ecx, 00000005
:004881F3 99                   cdq
:004881F4 F7F9                 idiv ecx
:004881F6 85D2                 test edx, edx
:004881F8 7414                 je 0048820E
:004881FA 8B45FC               mov eax, dword ptr [ebp-04]
:004881FD 8A4418FF             mov al, byte ptr [eax+ebx-01]
:00488201 E83EFFFFFF           call 00488144<----判断是否是字母,若是 AL=1
:00488206 84C0                 test al, al
:00488208 0F84A1000000         je 004882AF

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004881EA(U), :004881F8(C)
|
:0048820E 8B45FC               mov eax, dword ptr [ebp-04]
:00488211 8A4418FF             mov al, byte ptr [eax+ebx-01]
:00488215 3C2D                 cmp al, 2D
:00488217 742D                 je 00488246
:00488219 83FB05               cmp ebx, 00000005
:0048821C 7D0C                 jge 0048822A
:0048821E 8B55FC               mov edx, dword ptr [ebp-04]
:00488221 25FF000000           and eax, 000000FF
:00488226 03F0                 add esi, eax<----输入码第一组的 ASCII 码和存在 ESI
:00488228 EB1C                 jmp 00488246

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048821C(C)
|
:0048822A 83FB0A               cmp ebx, 0000000A
:0048822D 7D0C                 jge 0048823B
:0048822F 8B55FC               mov edx, dword ptr [ebp-04]
:00488232 25FF000000           and eax, 000000FF
:00488237 03F8                 add edi, eax<----输入码第二组的 ASCII 码和存在 EDI
:00488239 EB0B                 jmp 00488246

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048822D(C)
|
:0048823B 8B55FC               mov edx, dword ptr [ebp-04]
:0048823E 25FF000000           and eax, 000000FF
:00488243 0145F4               add dword ptr [ebp-0C], eax<----输入码第三组的 ASCII 码和存在[EBP-0C]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00488217(C), :00488228(U), :00488239(U)
|
:00488246 43                   inc ebx
:00488247 83FB0F               cmp ebx, 0000000F
:0048824A 0F8574FFFFFF         jne 004881C4
:00488250 8D0C37               lea ecx, dword ptr [edi+esi]<----输入码前两组相加存在 ECX
:00488253 034DF4               add ecx, dword ptr [ebp-0C]<-----加上第 3 组 ASCII
:00488256 8BC6                 mov eax, esi<----输入码第一组的 ASCII 码的和
:00488258 BB1A000000           mov ebx, 0000001A
:0048825D 99                   cdq
:0048825E F7FB                 idiv ebx
:00488260 83C241               add edx, 00000041
:00488263 8B45FC               mov eax, dword ptr [ebp-04]
:00488266 3A500F               cmp dl, byte ptr [eax+0F]<----与输入码的第 16 位比较
:00488269 7544                 jne 004882AF
:0048826B 8BC7                 mov eax, edi<----输入码第二组的 ASCII 码的和
:0048826D BB1A000000           mov ebx, 0000001A
:00488272 99                   cdq
:00488273 F7FB                 idiv ebx
:00488275 83C241               add edx, 00000041
:00488278 8B45FC               mov eax, dword ptr [ebp-04]
:0048827B 3A5010               cmp dl, byte ptr [eax+10]<----与输入码的第 17 位比较
:0048827E 752F                 jne 004882AF
:00488280 8B45F4               mov eax, dword ptr [ebp-0C]<----输入码第三组的 ASCII 码的和
:00488283 BB1A000000           mov ebx, 0000001A
:00488288 99                   cdq
:00488289 F7FB                 idiv ebx
:0048828B 83C241               add edx, 00000041
:0048828E 8B45FC               mov eax, dword ptr [ebp-04]
:00488291 3A5011               cmp dl, byte ptr [eax+11]<----与输入码的第 18 位比较
:00488294 7519                 jne 004882AF
:00488296 8BC1                 mov eax, ecx<----输入码前三组的 ASCII 码的和
:00488298 B91A000000           mov ecx, 0000001A
:0048829D 99                   cdq
:0048829E F7F9                 idiv ecx
:004882A0 83C241               add edx, 00000041
:004882A3 8B45FC               mov eax, dword ptr [ebp-04]
:004882A6 3A5012               cmp dl, byte ptr [eax+12]<----与输入码的第 19 位比较
:004882A9 7504                 jne 004882AF
:004882AB C645FB01             mov [ebp-05], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00488189(C), :00488196(C), :004881A3(C), :004881B0(C), :004881E4(C)
|:00488208(C), :00488269(C), :0048827E(C), :00488294(C), :004882A9(C)
|
:004882AF 33C0                 xor eax, eax
:004882B1 5A                   pop edx
:004882B2 59                   pop ecx
:004882B3 59                   pop ecx
:004882B4 648910               mov dword ptr fs:[eax], edx
:004882B7 68CC824800           push 004882CC
```

### 计算示例

上例  
`A B C 1 - E F 2 H - I 3 K L - ****`  
`41 42 43 31 2D 45 46 32 48 2D 49 33 4B 4C 2D`

- 第一组 `ASCII` 的和:`F7`
- 第二组 `ASCII` 的和:`105`
- 第三组 `ASCII` 的和:`113`
- 前三组 `ASCII` 的和:`30F`

- `F7/1A` 余数 `D`, `D+41=4E("N")`
- `105/1A` 余数 `1`, `1+41=42("B")`
- `113/1A` 余数 `F`, `F+41=50("P")`
- `30F/1A` 余数 `3`, `3+41=44("D")`

### 最终整理

**用户名**：`LiuTong`  
**注册码**：`ABC1-EF2H-I3KL-NBPD`  
`4dd466d0e1e0cc123afbe504da2462ca`
