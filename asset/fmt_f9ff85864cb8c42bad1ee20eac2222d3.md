
# ClockWise V3.25c

> 原文：[`www.pediy.com/kssd/pediy05/pediy50492.htm`](https://www.pediy.com/kssd/pediy05/pediy50492.htm)

简单算法——ClockWise V3.25c

- 下载页面：[`www.skycn.com/soft/4172.html`](http://www.skycn.com/soft/4172.html)
- 软件大小：792 KB
- 软件语言：英文
- 软件类别：国外软件 / 共享版 / 时钟日历
- 应用平台：Win9x/NT/2000/XP
- 加入时间：2002-10-03 11:41:55
- 下载次数：3999
- 推荐等级：*****
- 开发商：[`www.rjsoftware.com/`](http://www.rjsoftware.com/)

**【软件简介】**：一个结合了数字时钟、日历、提醒/备忘时钟、计时和程序调度的时间管理实用工具，它可以安排 5 个提醒和多达 30 个单独的程序事件，另外，它还可以通过 `Internet` 或原子钟自动设置系统时间。你可以用它自动安排关机和重新启动的时间。

**【软件限制】**：30 days trial

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`W32Dasm8.93` 黄金版

---

### 过程

`ClockWise.exe` 无壳，`Visual C++ 6.0` 编写。呵呵，我等菜鸟喜欢的类型呀。

反汇编，查找关键提示，很容易就找到核心了。

```text
User    Name  ：fly
Serial Number ：1234      （序列号有很多限制）
Registration  ：13572468 （试炼码要大于 5 位）
```

---

```asm
:0042F54D 55                  push ebp
:0042F54E 8BEC                mov ebp, esp
:0042F550 51                  push ecx
:0042F551 894DFC              mov dword ptr [ebp-04], ecx
:0042F554 8B4DFC              mov ecx, dword ptr [ebp-04]
:0042F557 E84E000000          call 0042F5AA
                              ==== > 关键 CALL！进入！
:0042F55C 85C0                test eax, eax
:0042F55E 7428                je 0042F588
                              ==== > 跳则 OVER！
:0042F560 8B4DFC              mov ecx, dword ptr [ebp-04]
:0042F563 E81B030000          call 0042F883
:0042F568 6A40                push 00000040

* Possible Reference to Dialog:
                            |
:0042F56A 68140C4800          push 00480C14

* Possible StringData Ref from Data Obj -> "THANK YOU for registering ClockWise"
                              ==== > 呵呵，胜利女神！
:0042F56F 68180C4800          push 00480C18
:0042F574 8B4DFC              mov ecx, dword ptr [ebp-04]
:0042F577 E8F32C0200          call 0045226F
:0042F57C 6A00                push 00000000
:0042F57E 8B4DFC              mov ecx, dword ptr [ebp-04]
:0042F581 E83D0E0200          call 004503C3
:0042F586 EB1E                jmp 0042F5A6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042F55E(C)
|
:0042F588 6A30                push 00000030

* Possible Reference to Dialog:
                            |
:0042F58A 683C0C4800          push 00480C3C

* Possible StringData Ref from Data Obj -> "Sorry, registration didn't work!"
                              ==== > BAD BOY！
:0042F58F 68440C4800          push 00480C44
```

---

### 进入关键 CALL：42F557 `call 0042F5AA`

```asm
* Referenced by a CALL at Addresses:
|:0042F4D1    , :0042F557

:0042F5AA 55                  push ebp
:0042F5AB 8BEC                mov ebp, esp
:0042F5AD 83EC24              sub esp, 00000024
:0042F5B0 53                  push ebx
:0042F5B1 56                  push esi
:0042F5B2 57                  push edi
:0042F5B3 894DDC              mov dword ptr [ebp-24], ecx
:0042F5B6 90                  nop
:0042F5B7 90                  nop
:0042F5B8 90                  nop
:0042F5B9 90                  nop
:0042F5BA C745F800000000      mov [ebp-08], 00000000
:0042F5C1 90                  nop
:0042F5C2 90                  nop
:0042F5C3 90                  nop
:0042F5C4 90                  nop
:0042F5C5 90                  nop
:0042F5C6 90                  nop
:0042F5C7 90                  nop
:0042F5C8 90                  nop
:0042F5C9 C745FC00000000      mov [ebp-04], 00000000
:0042F5D0 C745E800000000      mov [ebp-18], 00000000
:0042F5D7 8B4DDC              mov ecx, dword ptr [ebp-24]
:0042F5DA 83C160              add ecx, 00000060
:0042F5DD E86E3BFDFF          call 00403150
:0042F5E2 8945F4              mov dword ptr [ebp-0C], eax
:0042F5E5 837DF400            cmp dword ptr [ebp-0C], 00000000
                              ==== > Name 没填？
:0042F5E9 0F8486020000        je 0042F875
                              ==== > 跳则 OVER！
:0042F5EF 8B4DDC              mov ecx, dword ptr [ebp-24]
:0042F5F2 83C168              add ecx, 00000068
:0042F5F5 E8563BFDFF          call 00403150
:0042F5FA 85C0                test eax, eax
                              ==== > 序列号没填？
:0042F5FC 0F8E73020000        jle 0042F875
                              ==== > 跳则 OVER！
:0042F602 8B4DDC              mov ecx, dword ptr [ebp-24]
:0042F605 83C164              add ecx, 00000064
:0042F608 E8433BFDFF          call 00403150
:0042F60D 83F805              cmp eax, 00000005
                              ==== > 注册码应＞5 位
:0042F610 0F8E5F020000        jle 0042F875
                              ==== > 跳则 OVER！
:0042F616 8B4DDC              mov ecx, dword ptr [ebp-24]
:0042F619 83C168              add ecx, 00000068
:0042F61C E87FFFFEFF          call 0041F5A0
:0042F621 50                  push eax
                              ==== > EAX=1234
:0042F622 E8BCFE0000          call 0043F4E3
                              ==== > 将 1234 转换为 16 进制值表示。下面进行诸多检测！
:0042F627 83C404              add esp, 00000004
:0042F62A 8945FC              mov dword ptr [ebp-04], eax
                              ==== > [ebp-04]=EAX=4D2(H)=1234(D)
:0042F62D 837DFC01            cmp dword ptr [ebp-04], 00000001
                              ==== > 不能小于 1
:0042F631 7263                jb 0042F696
                              ==== > 跳则 OVER！
:0042F633 817DFC2C010000      cmp dword ptr [ebp-04], 0000012C
                              ==== > 或者≤12C(H)
:0042F63A 7609                jbe 0042F645
:0042F63C 817DFCE8030000      cmp dword ptr [ebp-04], 000003E8
                              ==== > 或者≥3E8(H)
:0042F643 7251                jb 0042F696
                              ==== > 跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042F63A(C)
|
:0042F645 817DFCC4090000      cmp dword ptr [ebp-04], 000009C4
                              ==== > 或者＜9C4(H)
:0042F64C 7609                jbe 0042F657
:0042F64E 817DFC88130000      cmp dword ptr [ebp-04], 00001388
                              ==== > 或者≥1388(H)
:0042F655 723F                jb 0042F696
                              ==== > 跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042F64C(C)
|
:0042F657 817DFC401F0000      cmp dword ptr [ebp-04], 00001F40
                              ==== > 或者≤1F40(H)
:0042F65E 7609                jbe 0042F669
:0042F660 817DFC67270000      cmp dword ptr [ebp-04], 00002767
                              ==== > 或者≥2767(H)
:0042F667 722D                jb 0042F696
                              ==== > 跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042F65E(C)
|
:0042F669 817DFC162A0000      cmp dword ptr [ebp-04], 00002A16
                              ==== > 或者≤2A16(H)
:0042F670 7609                jbe 0042F67B
:0042F672 817DFC532A0000      cmp dword ptr [ebp-04], 00002A53
                              ==== > 或者≥2A53(H)
:0042F679 721B                jb 0042F696
                              ==== > 跳则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042F670(C)
|
:0042F67B 817DFCE02E0000      cmp dword ptr [ebp-04], 00002EE0
```
