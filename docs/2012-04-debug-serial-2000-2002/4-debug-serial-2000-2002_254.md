
# CMailServer V3.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1231.htm`](https://www.pediy.com/kssd/pediy03/forum1231.htm)

## CMailServer V3.2 注册码算法及 CrackCode 2000 的注册机

**作者**：PaulYoung  
**时间**：2001.08.18  

**软件简介**：
1. 支持通用邮件客户端软件 `Outlook Express`、`Foxmail` 等收发邮件。
2. 支持 Web 页面收发邮件。
3. 支持 Internet 收发邮件。
4. 支持用户通过 Web 浏览器申请邮箱、修改邮箱密码、修改用户信息等资料。
5. 支持管理员新建、删除、禁用用户邮箱和设置用户邮箱大小。
6. 支持管理员同时向多个用户发送邮件。
7. 支持管理员生成 `HTML` 和 `INI` 格式的用户邮箱信息报表。
8. 支持作为 `NT` 服务运行。

**软件下载**：`http://microserver.top263.net/cmailserver/cmailserver32cr2.exe`

**破解工具**：`TRW 2000`, `CrackCode 2000`

---

首先感谢 `TAE![CCG]` 关于 `CCproxy 4.11d` 的破解方法给我的启示，使我了解到“遥志软件”系列软件的注册陷阱。下面，我说说另外一个“遥志软件”：`CMailServer V3.2` 的注册码破解，因为与 `CCProxy` 同属一系列，所以注册码算法方面比较相似，`TAE![CCG]`，借你的算法来参考一下吧。哈……

在注册框处填写任意序列号和注册码，运行 `TRW`，下 `bpx hmemcpy`，`F5` 退出，点确定，中断后下 `bd *`，再下 `pmodule`，按 1 次 `F12`（2 次出错），`F10` 单步跟踪……

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004176E5(C)
|
:0041774B 8B4C2418                  mov ecx, dword ptr [esp+18]
:0041774F 8B542414                  mov edx, dword ptr [esp+14]
:00417753 8D44242C                  lea eax, dword ptr [esp+2C]
:00417757 50                         push eax
:00417758 51                         push ecx                  //机器码
:00417759 52                         push edx                  //假注册码
:0041775A E831FBFFFF                 call 00417290             //计算注册码，F8 进入，F10 单步跟踪

.
.
.

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00417353(C)
|
:0041730E 8B442434                  mov eax, dword ptr [esp+34]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041730C(U)
|
:00417312 0FBE0408                  movsx eax, byte ptr [eax+ecx] //将 microserver1 的字符依次传给 eax
:00417316 0FBE140B                  movsx edx, byte ptr [ebx+ecx] //将 cmailserver3 的字符依次传给 edx
:0041731A 03C2                      add eax, edx                          //eax+edx 的和放到 eax
:0041731C BD3E000000                mov ebp, 0000003E                     //"3E"给 ebp
:00417321 0FBE140F                  movsx edx, byte ptr [edi+ecx] //将序列号的字符依次传给 edx
:00417325 03C2                      add eax, edx                          //相加给 eax
:00417327 0FBE11                   movsx edx, byte ptr [ecx]     //将机器码的字符依次传给 edx
:0041732A 03C2                      add eax, edx                          //再相加给 eax
:0041732C 99                        cdq
:0041732D F7FD                      idiv ebp                          //最后的和除以 ebp,也就是"3E"余数给 dl
:0041732F 83FA1A                    cmp edx, 0000001A                 //大于"1A"吗
:00417332 7D05                      jge 00417339                      //大于就跳
:00417334 80C261                    add dl, 61                          //小于就加上 61
:00417337 EB0D                      jmp 00417346                      //跳到 00417346 处

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00417332(C)
|
:00417339 83FA34                    cmp edx, 00000034                 //大于"34"吗
:0041733C 7D05                      jge 00417343                      //大于就跳
:0041733E 80C227                    add dl, 27                          //小于就减去"27"
:00417341 EB03                      jmp 00417346

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041733C(C)
|
:00417343 80EA04                    sub dl, 04                          //上面如果小于就到了这里,减去"4"

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00417337(U), :00417341(U)
|
:00417346 8B442438                  mov eax, dword ptr [esp+38]
:0041734A 88140E                    mov byte ptr [esi+ecx], dl      //保存注册码
:0041734D 41                        inc ecx                           //指向下一个机器码的字符
:0041734E 48                        dec eax                           //计数器
:0041734F 89442438                  mov dword ptr [esp+38], eax
:00417353 75B9                      jne 0041730E                        //形成循环
:00417355 5F                        pop edi
:00417356 5E                        pop esi
:00417357 5D                        pop ebp
:00417358 5B                        pop ebx
:00417359 83C420                    add esp, 00000020
:0041735C C3                        ret
```

一大串的算法，令我等菜鸟头痛，如何快速知道自己的注册码呢，我教你一个方法吧。在：

```
:00417355 5F                        pop edi
```

处设断，重新运行 `CMailServer`，程序会被中断，`F10` 几次后，会来到：

```
:00402C95 E846B20200                call 0042DEE0  //下 d eax 可以直接看到注册码。
```

用 `CrackCode 2000` 写个注册机吧：

```ini
[Options]
CommandLine=CMailServer.exe
Mode=0
First_Break_Address=402C95
First_Break_Address_Code=E8
First_Break_Address_Code_Lenth=2
Save_Code_Address=EAX
```

在程序一运行就会看到正确的注册码，序列号可以任意，也可以为空。你填入注册码时，会提示注册失败，不要怕，这是个陷阱，如果提示你成功你就完蛋了。重新运行吧，你会发现已经成为注册用户了。哈……

再次感谢 `TAE![CCG]` 师父！

`THE END`
