
# 流光 2001

> 原文：[`www.pediy.com/kssd/pediy03/forum1222.htm`](https://www.pediy.com/kssd/pediy03/forum1222.htm)

## 流光 2001 完全暴力破解

### 作者信息

`hehan china 多情俏狐`

(转载请保持完整)

### 软件说明

小榕黑客软件系列产品中主打产品，能对 `pop3`, `ftp`, `http` 服务器及其它密码进行快速猜解。还兼有追捕和双向 `IP` 查询功能。

### 所用工具

`trw2000`, `hedit` 和 `w32dasm`

### 1. 去除过期提示

把日期向前改一年不过期，随怀疑是调用了 `getsysttime` 函数。

运行 `trw2000`，用 `getsystime` 设断，程序在出现调查表之前中断：

```
* Reference To: MSVCRT.time, Ord:02D0h====>调用 API 函数 getsystemtime
                                |
:00406149 FF15F0D64800           Call dword ptr [0048D6F0]
:0040614F 8B442410               mov eax, dword ptr [esp+10]
:00406153 83C404                 add esp, 00000004
:00406156 3DB7F1C53A             cmp eax, 3AC5F1B7
:0040615B 7E4C                   jle 004061A9=====>不跳就过期
```

所以改为跳，那 `74` 改 `EB`。

```
:0040615D 6A00                   push 00000000
:0040615F 8D4C2434               lea ecx, dword ptr [esp+34]
:00406163 E848950000             call 0040F6B0
:00406168 A148FB4A00             mov eax, dword ptr [004AFB48]
:0040616D 51                     push ecx
:0040616E 85C0                   test eax, eax
:00406170 C78424CC05000000000000 mov dword ptr [esp+000005CC], 00000000
:0040617B 8BCC                   mov ecx, esp
:0040617D 740B                   je 0040618A
:0040617F 8964240C               mov dword ptr [esp+0C], esp
```

### 2. 去除防暴破程序自检点

启动后虽然没过期提示，但程序自动检测，暴破后照样出错退出。

用 `w32dasm` 反汇编后的串式参考正好可以打到出错提示：

(不过中文提示很多，很难找)

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040695C(U)
|
:00406963 85C0                   test eax, eax
:00406965 7439                   je 004069A0====》offset 6965h
```

如果程序被修改就跳，说可能被病毒感染或者捆绑了恶意程序，然后程序非正常退出。

```
:00406967 A148FB4A00             mov eax, dword ptr [004AFB48]
:0040696C 51                     push ecx
:0040696D 85C0                   test eax, eax
:0040696F 8BCC                   mov ecx, esp
:00406971 740B                   je 0040697E
:00406973 89642424               mov dword ptr [esp+24], esp
```

```
* Possible StringData Ref from Data Obj ->"数字验证失败，可能被病毒感染或者捆绑了恶意程序"
                               ->"。"
                               |
:00406977 6894404A00             push 004A4094
:0040697C EB09                   jmp 00406987
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406971(C)
|
:0040697E 89642424               mov dword ptr [esp+24], esp
```

```
* Possible StringData Ref from Data Obj ->"Digit Authenticate Failed, Quit."
                               |
:00406982 6870404A00             push 004A4070
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040697C(U)
|
* Reference To: MFC42.Ordinal:0219, Ord:0219h
```

### 3. 开始用户调查表的去除

曾经尝试从成功消息和出错消息两处用动态和静态分析均未成功，随决定从开始一步步进行跟踪。

```
* Possible Reference to Dialog:
                               |
:00407E08 68D0434A00             push 004A43D0
:00407E0D 8D4C240C               lea ecx, dword ptr [esp+0C]
:00407E11 C784242C08000000000000 mov dword ptr [esp+0000082C], 00000000
:00407E1C E88F730700             call 0047F1B0
:00407E21 85C0                   test eax, eax
:00407E23 750A                   jne 00407E2F
:00407E25 8B06                   mov eax, dword ptr [esi]
:00407E27 8BCE                   mov ecx, esi
:00407E29 FF90C0000000           call dword ptr [eax+000000C0]
                               ^^^^^^^这才是那个令人讨厌的调查表
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407E23(C)
|跳到这里就一片晴空了
```

```
:00407E2F 8D4C2408               lea ecx, dword ptr [esp+08]
:00407E33 E828740700             call 0047F260
:00407E38 8BF8                   mov edi, eax
```

\====================================>

止此完全破解成功!!!!!

哈哈哈!!!!!!

`2001-08-14` `4dd466d0e1e0cc123afbe504da2462ca`
