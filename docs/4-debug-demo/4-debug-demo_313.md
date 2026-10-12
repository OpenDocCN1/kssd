
# 破流光 2001

> 原文：[`www.pediy.com/kssd/pediy03/forum1276.htm`](https://www.pediy.com/kssd/pediy03/forum1276.htm)

## 流光 2001 完全暴力破解

\================================================

hehan china 多情俏狐

(转载请保持完整)

### 软件说明

小榕黑客软件系列产品中主打产品，能对`pop3`、`ftp`、`http`服务器及其它密码进行快速猜解。还兼有追捕和双向`IP`查询功能。

所用工具：`trw2000`、`hedit`和`w32dasm`

软件版本号：`Build 2262GE (Preview 1 Patch 1)` Expired `2001.03.31`

过期日期为：2001 年 3 月 31 日

### 1. 去除过期提示

把日期向前改一年不过期，怀疑是调用了`GetSystemTime`函数。

运行`trw2000`，用`GetSystemTime`设断，程序在出现调查表之前中断：

```
* Reference To: MSVCRT.time, Ord:02D0h====>调用 API 函数 GetSystemTime
                  |
:00406149 FF15F0D64800             Call dword ptr [0048D6F0]
:0040614F 8B442410                 mov eax, dword ptr [esp+10]
:00406153 83C404                   add esp, 00000004
:00406156 3DB7F1C53A               cmp eax, 3AC5F1B7
:0040615B 7E4C                     jle 004061A9=====>不跳就过期
```

所以改为跳，那`74`改`EB`。

```
:0040615D 6A00                     push 00000000
:0040615F 8D4C2434                 lea ecx, dword ptr [esp+34]
:00406163 E848950000               call 0040F6B0
:00406168 A148FB4A00               mov eax, dword ptr [004AFB48]
:0040616D 51                       push ecx
:0040616E 85C0                     test eax, eax
:00406170 C78424CC05000000000000   mov dword ptr [esp+000005CC], 00000000
:0040617B 8BCC                     mov ecx, esp
:0040617D 740B                     je 0040618A
:0040617F 8964240C                 mov dword ptr [esp+0C], esp
```

### 2. 去除防暴破程序自检点

启动后虽然没过期提示，但程序自动检测，暴破后照样出错退出：

用`w32dasm`反汇编后的串式参考正好可以找到出错提示：

(不过中文提示很多，很难找)

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040695C(U)
|
:00406963 85C0                     test eax, eax
:00406965 7439  =>修改不跳          je 004069A0====》offset 6965h
```

如果程序被修改就不跳，说可能被病毒感染或者捆绑了恶意程序，然后程序非正常退出。

`74`改`EB`就行了。

```
:00406967 A148FB4A00               mov eax, dword ptr [004AFB48]
:0040696C 51                       push ecx
:0040696D 85C0                     test eax, eax
:0040696F 8BCC                     mov ecx, esp
:00406971 740B                     je 0040697E
:00406973 89642424                 mov dword ptr [esp+24], esp
```

`* Possible StringData Ref from Data Obj` ->`"数字验证失败，可能被病毒感染或者捆绑了恶意程序"`

->`"。"`

```
                  |
:00406977 6894404A00               push 004A4094
:0040697C EB09                     jmp 00406987
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406971(C)
|
:0040697E 89642424                 mov dword ptr [esp+24], esp
```

`* Possible StringData Ref from Data Obj` ->`"Digit Authenticate Failed, Quit."`

```
                  |
:00406982 6870404A00               push 004A4070
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040697C(U)
|
* Reference To: MFC42.Ordinal:0219, Ord:0219h
```

### 3. 开始用户调查表的去除

曾经尝试从成功消息和出错消息两处用动态和静态分析均步成功，随决定从开始一步步进行跟踪。

```
* Possible Reference to Dialog: 
                  |
:00407E08 68D0434A00               push 004A43D0
:00407E0D 8D4C240C                 lea ecx, dword ptr [esp+0C]
:00407E11 C784242C08000000000000   mov dword ptr [esp+0000082C], 00000000
:00407E1C E88F730700               call 0047F1B0
:00407E21 85C0                     test eax, eax
:00407E23 750A                     jne 00407E2F==>改掉就跳过了
```

`75`改为`EB`。

```
:00407E25 8B06                     mov eax, dword ptr [esi]
:00407E27 8BCE                     mov ecx, esi
:00407E29 FF90C0000000             call dword ptr [eax+000000C0]
```

^^^^^^^这才是那个令人讨厌的调查表

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407E23(C)
|跳到这里就一片晴空了
:00407E2F 8D4C2408                 lea ecx, dword ptr [esp+08]
:00407E33 E828740700               call 0047F260
:00407E38 8BF8                     mov edi, eax
```

\====================================>

止此完全破解成功!!!!!

哈哈哈!!!!!!

打住，别笑!!!!

一运行正常，但一拨上号后使用即出现过期提示，并且是随机的

我晕~~~~~~~，太可怕了，真不愧为中国知名的安全专家

小榕大哥，看来我们只能再来一局了!

搜索出错字符串?不行!改成英文(用`exescope`),再搜还是找不到.

想尽一切办法，用动静两种方式进行跟踪，仍无效，期间也中断过但离我想要的调用太远.

看到它主界面上的一个单词

`Expired(过期)`

用它打一下，谢天谢地，打到了，3 处，其中一处是主界面提示，在另外两处用`trw200`进行疯狂跟踪，反复用`r fl z`试跳

最后终于找到了可爱的错框和绕过它的方法:

```
* Possible StringData Ref from Data Obj ->"www.sina.com.cn"
                  |
:0046C889 68502A4C00               push 004C2A50
:0046C88E C705F431B70001000000     mov dword ptr [00B731F4], 00000001
:0046C898 E863000000               call 0046C900
:0046C89D 83C404                   add esp, 00000004
:0046C8A0 85C0                     test eax, eax
:0046C8A2 7520                     jne 0046C8C4==>不跳就到期
```

`75`改`EB`就可以了

```
:0046C8A4 50                       push eax
:0046C8A5 50                       push eax
:0046C8A6 A17495AB00               mov eax, dword ptr [00AB9574]
:0046C8AB 686B040000               push 0000046B
:0046C8B0 50                       push eax
```

`* Reference To: USER32.SendMessageA, Ord:0214h`

|到这里就出现了到期框:

```
:0046C8B1 FF1514D84800             Call dword ptr [0048D814]
:0046C8B7 C705F431B70000000000     mov dword ptr [00B731F4], 00000000
:0046C8C1 33C0                     xor eax, eax
:0046C8C3 C3                       ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046C8A2(C)
```

```
* Possible StringData Ref from Data Obj ->"www.yahoo.com"
                  |
:0046C8C4 68402A4C00               push 004C2A40
```

嘻嘻~~，小弟暂胜一招，小榕大哥承让了。

现在可以用了，不知还有什么陷阱......相信不会有恶意陷阱

### 总结

作者通过判断时间、自我验证、在线注册和上网后随机判断几种加密方法，对软件进行综合加密。

`2001-08-27` `4dd466d0e1e0cc123afbe504da2462ca`
