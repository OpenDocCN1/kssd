
# 英语会话精灵 v2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50630.htm`](https://www.pediy.com/kssd/pediy05/pediy50630.htm)

## 软件信息

软件大小：606 KB  
软件语言：简体中文  
软件类别：国产软件 / 共享版 / 外语学习  
应用平台：`Win9x/NT/2000/XP`  
加入时间：2002-05-29 20:12:03  
下载次数：7028  

联系人：`webmaster@anyeasy.com`  
开发商：[`www.anyeasy.com/`](http://www.anyeasy.com/)

## 软件介绍

英语会话精灵是一款非常方便、高效的英语会话学习软件，其中包含 12 大类 1900 余句经典、实用的英语会话例句，您可以根据自己的学习要求对其进行设置，并在屏幕顶层进行自动播放，只要您在使用电脑进行工作、学习或娱乐的同时，就能不知不觉、轻轻松松地掌握各类日常英语会话，提高英语听力和口语能力。该方式既摆脱了枯燥的传统学习方法，又能充分利用在电脑前的许多零碎时间，有效地提高了学习效率，大大增强了对英语会话的记忆能力和应用能力。本软件包含许多体贴、灵活的功能设置，如英文朗读、自定义会话库、分组学习等等，还可以设置英汉播放次序、显示时间和播放遍数等，通过合理的设置，您可以方便地进行英语会话的学习和自我测试。本软件十分精致、小巧，仅有 607KB 大小，安装简单，卸载方便。

## 作者声明

初学破解，纯属技术交流，无其它目的。

工具：`ollyDbg1.09`, `W32Dasm10`, `Caspr110` 脱壳工具。

## 假设数据

用户编号：222961388  
注册码：654321

## 一、先脱壳，查找出错信息

### 错误提示代码

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0047A05C(C)
  |
  :0047A19F 6830000400          push 00040030
  :0047A1A4 B908A24700          mov ecx, 0047A208

* Possible StringData Ref from Code Obj ->"注册码无效！"
                              |
  :0047A1A9 BA6CA24700          mov edx, 0047A26C
  :0047A1AE A1E8794B00          mov eax, dword ptr [004B79E8]
  :0047A1B3 8B00               mov eax, dword ptr [eax]
  :0047A1B5 E8A22AFDFF         call 0044CC5C
  :0047A1BA 8B86E8020000       mov eax, dword ptr [esi+000002E8]
  :0047A1C0 8B10               mov edx, dword ptr [eax]
  :0047A1C2 FF92CC000000       call dword ptr [edx+000000CC]
```

### 相关跳转代码

```
0047A053 8B55F8                mov edx, dword ptr [ebp-08] ==> 在这下断
:0047A056 58                   pop eax
:0047A057 E8489EF8FF           call 00403EA4
:0047A05C 0F853D010000         jne 0047A19F
:0047A062 6840000400           push 00040040
:0047A067 B908A24700           mov ecx, 0047A208

* Possible StringData Ref from Code Obj ->"注册成功！请保留好您的注册码，谢谢！"
                              |
```

## 二、用 `ollyDbg` 载入，动态分析

```
0047A002  |.  8B1D 28784B00 MOV EBX,DWORD PTR DS:[4B7828]           ;  unpacked.004B8930
0047A008  |.  33C0          XOR EAX,EAX
0047A00A  |.  55            PUSH EBP
0047A00B  |.  68 FBA14700   PUSH unpacked.0047A1FB
0047A010  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
0047A013  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
0047A016  |.  6A 00         PUSH 0
0047A018  |.  6A 1B         PUSH 1B
0047A01A  |.  E8 A9FDFFFF   CALL unpacked.00479DC8                  ;  用户编号十六进制值入 eax,此 eax=0D4A1EEC
0047A01F  |.  2D 1F17F601   SUB EAX,1F6171F                         ;  eax 减 1F6171F，eax=B5407CD
0047A024  |.  83DA 00       SBB EDX,0
0047A027  |.  E8 44C5F8FF   CALL unpacked.00406570                  ;  B5407CD 值乘以 1b，此 eax=131DCD29F
0047A02C  |.  52            PUSH EDX                                  ; /Arg2
0047A02D  |.  50            PUSH EAX                                  ; |Arg1
0047A02E  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]             ; |
0047A031  |.  E8 6AE9F8FF   CALL unpacked.004089A0                   ; \把串 31DCD29F 转换为十进制，即为 5131522719
0047A036  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0047A039  |.  50            PUSH EAX
0047A03A  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
0047A03D  |.  8B86 E8020000 MOV EAX,DWORD PTR DS:[ESI+2E8]
0047A043  |.  E8 444AFBFF   CALL unpacked.0042EA8C                   ;  取输入注册码
0047A048  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0047A04B  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
0047A04E  |.  E8 9DE7F8FF   CALL unpacked.004087F0
0047A053  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]             ;  输入注册码，此为 edx=654321
0047A056  |.  58            POP EAX                                   ;  真码入 eax
0047A057  |.  E8 489EF8FF   CALL unpacked.00403EA4                    ;  关键比较 unpa
0047A05C  |.  0F85 3D010000 JNZ unpacked.0047A19F                    ;  不等就跳 npac
0047A062  |.  68 40000400   PUSH 40040
```

## 总结

1. 用户编号转为十六进制：`222961388` -> `D4A1EEC`
2. 十六进制值减 `1F6171F`：`D4A1EEC` - `1F6171F` = `B5407CD`
3. 第二步的值乘 `1B`：`B5407CD` * `1B` = `131DCD29F`
4. 第三步的值转为十进制就是注册码：`131DCD29F` -> `5131522719`

所以我的注册码如下：

用户编号：`222961388`  
注册码：`5131522719`  
校验码：`4dd466d0e1e0cc123afbe504da2462ca`
