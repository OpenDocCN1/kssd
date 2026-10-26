
# 手脱Armadillo4.40+屏蔽调试器+双进程模式+输入表乱序+策略代码衔接[原创]

【文章标题】: 手脱Armadillo4.40+屏蔽调试器+双进程模式+输入表乱序+策略代码衔接+防止内存补丁+双进程模式
【文章作者】: cxhcxh
【作者邮箱】: cxh852456@163.com
【作者主页】: .......
【作者QQ号】: 290019543
【软件名称】: unpackme
【下载地址】: 自己搜
【加壳方式】: Armadillo4.40
【保护方式】: 屏蔽调试器.......
【使用工具】: OD等
【操作平台】: xp
【软件介绍】: unpackme
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 一、查壳

PEID：Armadillo 3.78 - 4.xx -> Silicon Realms Toolworks

FP1.4：

======== 25-02-2007 13:37:37 ========

`D:\破解工具箱\Armadillo\Armadillo 4.40\Armadillo 4.40\UnPackMe.exe`

▲ 目标经Armadillo保护

保护系统授权等级(专业版)

【程序所使用保护模式】

屏蔽调试器

双进程模式

使用 输入表乱序 程序保护模式

使用 策略代码衔接 程序保护模式

使用 防止内存补丁 程序保护模式

【备份密钥设置】

固定的备份密钥

【程序压缩设置】

最好/最慢地压缩方式

【其它保护设置】

▲ 版本号 4.40

## 二、找OEP

知道了保护方式，就慢慢来脱吧。OD载入，忽略所有异常选项

```
004B3000 >  60              PUSHAD------------------------------------------停在这
004B3001    E8 00000000     CALL UnPackMe.004B3006
004B3006    5D              POP EBP
004B3007    50              PUSH EAX
004B3008    51              PUSH ECX
004B3009    0FCA            BSWAP EDX
004B300B    F7D2            NOT EDX
004B300D    9C              PUSHFD
004B300E    F7D2            NOT EDX
004B3010    0FCA            BSWAP EDX
```

这个UNPACKME采用COPYMEM2保护，用STANDARD PROTECTTION保护的脱壳方法行不通

下断`BP WaitForDebugEvent+5`，运行，中断，看堆栈

```
0012DC60  /0012F774
0012DC64  |0048F066  返回到 UnPackMe.0048F066 来自 kernel32.WaitForDebugEvent
0012DC68  |0012ED54-------------注意这里，FOLLOW IN DUMP，一会有用
0012DC6C  |000003E8
0012DC70  |00000004
0012DC74  |00000000
```

清除断点，重新下断`BP WriteProcessMemory`运行，中断，看堆栈

```
0012DB04   00493037  /CALL 到 WriteProcessMemory 来自 UnPackMe.00493031
0012DB08   0000004C  |hProcess = 0000004C (窗口)
0012DB0C   00427000  |Address = 427000-----------------------------OEP所在段
0012DB10   003E4C50  |Buffer = 003E4C50----------------------写入段
0012DB14   00001000  |BytesToWrite = 1000 (4096.)----------写入大小
0012DB18   0012DC20  \pBytesWritten = 0012DC20
```

在看数据窗口

```
0012ED54  00000001
0012ED58  00000B18
0012ED5C  00000FE0
0012ED60  80000001
0012ED64  00000000
0012ED68  00000000
0012ED6C  004271B0  UnPackMe.004271B0---------------------------OEP
0012ED70  00000002
0012ED74  00000000
0012ED78  004271B0  UnPackMe.004271B0
0012ED7C  004271B0  UnPackMe.004271B0
0012ED80  BAE32D64
0012ED84  00000000
0012ED88  BAE32D60
```

我门现在来找写入的地方

`004271B0 - 427000 = 1b0`

`1b0 + 003E4C50 = 3E4E00`

我门到这里去看看，`Ctrl+G:3E4E00`

```
003E4E00  55 8B EC 6A FF 68 60 0E  U?j?`
003E4E08  45 00 68 C8 92 42 00 64  E.h?B.d
003E4E10  A1 00 00 00 00 50 64 89  ?...Pd
```

我门将头两个字节`55 8B`改为`EB FE`，其实`55 8B`就是OEP的头两个字节

改完后我门一步一步走，回到壳代码，`ALT+F9`

`CTRL+F9`，`F8`，来到这里

```
00491C9B    51              PUSH ECX
00491C9C    8B55 08         MOV EDX,DWORD PTR SS:[EBP+8]
00491C9F    52              PUSH EDX
00491CA0    E8 48030000     CALL UnPackMe.00491FED------------------关键CALL，跟随
00491CA5    83C4 0C         ADD ESP,0C---------------------------------------停在这里
00491CA8    25 FF000000     AND EAX,0FF
00491CAD    85C0            TEST EAX,EAX
00491CAF    75 07           JNZ SHORT UnPackMe.00491CB8
00491CB1    32C0            XOR AL,AL
00491CB3    E9 2E030000     JMP UnPackMe.00491FE6
```

跟随后`CTRL+R`找相关调用，发现有两个地方调用拉这里，我门双击第二个`CALL`，将他`NOP`掉，做完这些再来下断

`BP WaitForDebugEvent+5`，运行中断，反汇编中跟随

```
0048F04E    DB7A F0         FSTP TBYTE PTR DS:[EDX-10]
0048F051    A0 336168E8     MOV AL,BYTE PTR DS:[E8686133]
0048F056    0300            ADD EAX,DWORD PTR DS:[EAX]
0048F058    008B 95DCF5FF   ADD BYTE PTR DS:[EBX+FFF5DC95],CL
0048F05E    FF52 FF         CALL DWORD PTR DS:[EDX-1]
0048F061    15 E0304C00     ADC EAX,<&KERNEL32.WaitForDebugEvent>
0048F066    85C0            TEST EAX,EAX--------------------------------------------新建EIP
0048F068    0F84 64270000   JE UnPackMe.004917D2------------------------PATCH-----改为JMP 00401000
0048F06E    8B85 FCFDFFFF   MOV EAX,DWORD PTR SS:[EBP-204]
0048F074    25 FF000000     AND EAX,0FF
0048F079    85C0            TEST EAX,EAX
```

将`WaitForDebugEvent` NOP掉并在`0048F066`新建EIP

`0048F068`开始PATCH

在`00401000`写入以下代码

```
00401000    8105 6CED1200 0>ADD DWORD PTR DS:[12ED6C],1000
0040100A    8105 78ED1200 0>ADD DWORD PTR DS:[12ED78],1000
00401014    8105 7CED1200 0>ADD DWORD PTR DS:[12ED7C],1000
0040101E    813D 7CED1200 0>CMP DWORD PTR DS:[12ED7C],UnPackMe.0044B000
00401028  - 0F85 3FE00800   JNZ UnPackMe.0048F06D
0040102E    68 28010000     PUSH 0b18
00401033    E8 7195457C     CALL kernel32.DebugActiveProcessStop
00401038    90              NOP-------------------------------------------放个断点
```

然后回到`12ED6C`处

```
0012ED54  00000001
0012ED58  00000B18
0012ED5C  00000FE0
0012ED60  80000001
0012ED64  00000000
0012ED68  00000000
0012ED6C  004271B0  UnPackMe.004271B0---------------------------改为00400000
0012ED70  00000002
0012ED74  00000000
0012ED78  004271B0  UnPackMe.004271B0---------------------------改为00400000
0012ED7C  004271B0  UnPackMe.004271B0---------------------------改为00400000
0012ED80  BAE32D64
0012ED84  00000000
0012ED88  BAE32D60
```

做完以上后，`SHIFT+F9`，中断，`EAX=1`说明一切正确，到这里已经完成了一半了

回到OD，挂接那个名字不是红色的那个进程，`F9`，`F12`，将头两个字节该为`55 8B`

```
004271B0    55              PUSH EBP-------------------------------OEP
004271B1    8BEC            MOV EBP,ESP
004271B3    6A FF           PUSH -1
004271B5    68 600E4500     PUSH UnPackMe.00450E60
004271BA    68 C8924200     PUSH UnPackMe.004292C8
004271BF    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
004271C5    50              PUSH EAX
004271C6    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
004271CD    83C4 A8         ADD ESP,-58
004271D0    53              PUSH EBX
004271D1    56              PUSH ESI
004271D2    57              PUSH EDI
```

## 三、修复

再开个OD，用`ARM PROCESS DETACH`载入UNPACKME

```
DONE!
Child process ID: 00000D08
Entry point: 004B3000
Original bytes: 60E8
```

直接挂接`D08`，`F9`，`F12`，修改头两个字节为`60 E8`

停在

```
004B3000 >  60              PUSHAD
004B3001    E8 00000000     CALL UnPackMe.004B3006
004B3006    5D              POP EBP
004B3007    50              PUSH EAX
004B3008    51              PUSH ECX
004B3009    0FCA            BSWAP EDX
004B300B    F7D2            NOT EDX
004B300D    9C              PUSHFD
```

现在可以想拖标准壳一样脱了，但这个只是为了找回正确的IAT，避开加密

`BP VirtualProtect`

运行到代码段后返回搜索命令`PUSH 100`

向上翻把`PUSH EBP`改为`RET`

`BP CreateThread`

断下后返回，一路`F8`

看到`CALL ECX`后进入就是OEP了，到达OEP后

```
004271B0    FD              STD
004271B1    1E              PUSH DS
004271B2    37              AAA
004271B3    91              XCHG EAX,ECX
004271B4    57              PUSH EDI
004271B5    FD              STD
004271B6    BB F5ED95B3     MOV EBX,B395EDF5
004271BB    333A            XOR EDI,DWORD PTR DS:[EDX]
004271BD    D7              XLAT BYTE PTR DS:[EBX+AL]
004271BE    DB9F 0995DBFB   FISTP DWORD PTR DS:[EDI+FBDB9509]
004271C4    A8 C5           TEST AL,0C5
004271C6    BF 728D95DB     MOV EDI,DB958D72
004271CB    FB              STI
```

OEP已经面目全非了，但不要紧，我门只需要真确的IAT

回到先前的OD，来到IAT的其实位置，将前3个IAT的二进制复制下来

在到第二个OD，大开内存，搜索刚才的二进制代码，将全部正确的IAT二进制代码复制到先前的OD中，

此时就可以关掉第2个OD了。

下面就要修复乱序了和策略代码了，打开`ARMINLINE`，填入IAT开始地址和大小，点寻回，稍等一会就OK了

还有策略代码

起点`02980000`

大小`20000`

点删除，OK

好拉，现在可以DUMP了，`LORDPE`完全转存，`IMPORT`修复，运行，OK

---

## 经验总结

................................................................

                    好好学习

                        天天向上

---

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年02月25日 下午 03:09:47

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)(https://bbs.kanxue.com/thread-51839.htm)
