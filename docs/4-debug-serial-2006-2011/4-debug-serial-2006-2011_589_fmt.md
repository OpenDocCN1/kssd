
# PLC 模拟软件 PATCH

> 原文：[`www.pediy.com/kssd/pediy12/134305.html`](https://www.pediy.com/kssd/pediy12/134305.html)

- **软件名称**：PLC 模拟软件
- **下载地址**：无
- **保护方式**：无壳
- **编写语言**：Microsoft Visual C++ v7.1 DLL
- **使用工具**：PEID 0.95, Olldbg 1.10
- **操作平台**：XP sp3
- **软件介绍**：PLC 模拟软件
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

工作中碰到一个 PLC 模拟软件 `aaa.exe`，它可以在服务器上运行，想研究研究，于是拷到自己机子上，可是运行就报错，确认就退出了，查看 log 文件提示是没注册，如下：

```
Message: Licence is invalid. Current state: Trial period expired
```

说明一下它的主要功能在一个 DLL 文件里 (`bbb.DLL`)，而注册也在该 DLL 里，仔细研究才发现，由于通讯需要, DLL 里做了个 HTTP server, 端口在 18500, 然后顺利成章的把注册的界面也做到 HTTP 里了，访问 `http://localhost:18500/bbb/home.htm` 就可以查看信息了，奇怪的是我并没发现 HTM 文件，用 `EXESCOPE` 打开 `bbb.DLL` 才发现 HTM 在它的资源里, 因为 HTTP SERVER 是 DLL 文件加载并执行才提供的，因此在 `aaa.exe` 运行报错后，不要确认，这时 HTTP SERVER 已好，在 IE 打开上面的网页就来到注册界面如下

![](img/6cdc386c828fb3d8b9a33d70d4fc8303.jpg)

这种注册方法还是第一次碰到，很显然因为是注册界面在网页里，函数参考 `GetWindowText`，`MessageBox`，以及字串参考和按钮断点都不适用了，该如何办？毛主席说过一切敌人都是纸老虎，让我们想想，虽然输入信息在网页里，但提交后数据肯定要传给程序进行验证，先附加 `aaa.exe`, `F9` 运行，然后任意输入注册码，点 `unlock`, 弹出对话框如下

![](img/dacaa44614b8b7fc710022cd0d7688b9.jpg)

好了，我们先在 `bbb.DLL` 的代码段下一个断点，然后再输入注册码，点 `UNLOCK`，`OD` 断下来在 `10010C4A`

代码如下：

```
10010C44    FF15 8CF20110   CALL DWORD PTR DS:[<&WS2_32.#18>]        ; WS2_32.select
10010C4A    8985 B8F7FFFF   MOV DWORD PTR SS:[EBP-848],EAX
10010C50    EB 1D           JMP SHORT bbb.10010C6F
................
10010C6F    83BD B8F7FFFF 0>CMP DWORD PTR SS:[EBP-848],0
10010C76    0F85 88000000   JNZ bbb.10010D04
.............
10010D04    6A 00           PUSH 0
10010D06    68 00080000     PUSH 800
10010D0B    8D85 BCF7FFFF   LEA ECX,DWORD PTR SS:[EBP-844]
10010D11    51              PUSH ECX
10010D12    8B55 08         MOV EDX,DWORD PTR SS:[EBP+8]
10010D15    8B42 08         MOV EAX,DWORD PTR DS:[EDX+8]
10010D18    50              PUSH EAX
10010D19    FF15 84F20110   CALL DWORD PTR DS:[<&WS2_32.#16>]        ; WS2_32.recv
10010D1F    8985 B4F7FFFF   MOV DWORD PTR SS:[EBP-84C],EAX
10010D25    83BD B4F7FFFF 0>CMP DWORD PTR SS:[EBP-84C],0
```

上面的代码主要是接收数据，继续跟踪代码是对收到的网络数据包进行处理，把我们输入的 `unlock` `code` 提取出来，然后来到另外一个 DLL (`ccc.DLL`)，其实如果一开始对 `bbb.DLL` 用 `loadlibrary` 加载的几个 DLL 文件进行观察，也很容易发现 `ccc.dll` 应该是注册关键 dll，因为它的输出函数有 `GetLicenseStatus`、`RegisterUnlockCode`，然后我们来到如下代码

代码如下：

```
013FFE68 >   55              PUSH EBP
013FFE69    8BEC            MOV EBP,ESP
。。。。。
013FFEBA    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]
013FFEBD    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
013FFEC0    B9 08000000     MOV ECX,8
013FFEC5    E8 46A6FFFF     CALL ccc.013FA510                            ；判断 unlock code 格式对否
013FFECA    84C0            TEST AL,AL                                    ；必须非 0
013FFECC    74 3B           JE SHORT ccc.013FFF09
013FFECE    A1 188B4001     MOV EAX,DWORD PTR DS:[1408B18]
013FFED3    8B00            MOV EAX,DWORD PTR DS:[EAX]
013FFED5    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
013FFED8    E8 2BF3FFFF     CALL ccc.013FF208                             ；关键 CALL
013FFEDD    84C0            TEST AL,AL                                    ；必须非 0,否则跳去提示注册码无效
013FFEDF    74 1F           JE SHORT ccc.013FFF00
013FFEE1    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
013FFEE4    A1 188B4001     MOV EAX,DWORD PTR DS:[1408B18]
013FFEE9    8B00            MOV EAX,DWORD PTR DS:[EAX]
013FFEEB    E8 3CEEFFFF     CALL ccc.013FED2C
013FFEF0    A1 188B4001     MOV EAX,DWORD PTR DS:[1408B18]
013FFEF5    8B00            MOV EAX,DWORD PTR DS:[EAX]
013FFEF7    33D2            XOR EDX,EDX
013FFEF9    E8 F2EDFFFF     CALL ccc.013FECF0
013FFEFE    EB 22           JMP SHORT ccc.013FFF22
013FFF00    C745 FC FCFFFFF>MOV DWORD PTR SS:[EBP-4],-4                 ；错误信息 UNLOCK CODE 无效
013FFF07    EB 19           JMP SHORT ccc.013FFF22
013FFF09    C745 FC FBFFFFF>MOV DWORD PTR SS:[EBP-4],-5                 ；错误信息 unlock code 格式不对
013FFF10    EB 10           JMP SHORT ccc.013FFF22
013FFF12    C745 FC FEFFFFF>MOV DWORD PTR SS:[EBP-4],-2                 ；错误信息 初始化失败 
013FFF19    EB 07           JMP SHORT ccc.013FFF22
013FFF1B    C745 FC 74ECFFF>MOV DWORD PTR SS:[EBP-4],-138C             ；错误信息 库文件无效
。。。。
013FFF4A    C3              RETN
```

继续跟进关键 `CALL` `013FF208`

代码如下：

```
013FF208    55              PUSH EBP
013FF209    8BEC            MOV EBP,ESP
........
013FF21A    C645 F3 00      MOV BYTE PTR SS:[EBP-D],0
........
013FF349    8845 F3         MOV BYTE PTR SS:[EBP-D],AL
.........
013FF370    8A45 F3         MOV AL,BYTE PTR SS:[EBP-D]
013FF373    5F              POP EDI
013FF374    5E              POP ESI
013FF375    8BE5            MOV ESP,EBP
013FF377    5D              POP EBP
013FF378    C3              RETN
```

要想最后返回 `AL` 非 0，可以更改 `013FF21A` 为 `MOV BYTE PTR SS:[EBP-D],1`

最后弹出成功对话框，如下

![](img/e456de67dcb122a86f4eff5026f1c13f.jpg)

可郁闷的是重启 `aaa.exe` 还是报错，显然在启动时有验证，猜测验证代码应该和上面的 `CALL` 差不多，通过搜索特征码 `MOV BYTE PTR SS:[EBP-D],0` 找到一处同样修改保存

--------------------------------------------------------------------------------

## 版权声明

看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!
