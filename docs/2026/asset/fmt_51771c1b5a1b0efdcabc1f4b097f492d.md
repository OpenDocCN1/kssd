
# 破解NetScanTools Pro 2000及其InstallShield脚本破解（其实脚本没破成）

（18千字）

## 破解NetScanTools Pro 2000及其InstallShield脚本破解（不完整）

Passion

我几个星期前曾受被人尊称为“皮大爷”的peterchen老兄的语无伦次的鞭策，于是也好高骛远地想研究研究InstallShield的脚本破解，正巧不久前见了洋白菜兄的InstallShield脚本反编译教程，觉得比一个字节一个字节地研究INS文件的编译后代码要方便多了，因而也就偷了许多懒。――NetScanTools Pro 2000是一套功能非常全面的网络扫描监控工具，Northwest Performance公司开发，网址是`http://www.nwpsw.com/`，不过此处没完整版下载。

NetScanTools Pro 2000安装时需要输入序列号，直接破据说非常麻烦，一个比较好的法子是从INS文件入手。我在洋白菜兄的主页`crackbest.com`上下了一个`DIS`（`DeInstallShield`），用它反编译`setup.ins`文件感觉还可以。

NetScanTools Pro 2000安装程序会询问姓名公司序列号，填完按NEXT后会出现一个框要你确认填的是否正确，按“Yes”后才进行序列号检测，错则弹出对话框说“Invalid Serial Number。”没的说，用`DIS`反编译其`SETUP.INS`，在结果中找“Invalid Serial Number”的`MSGREF`消息字串引用，很快就找到了下面的一段：

```
<LABEL\_0038> REF: 00000F72

  |

00002044: 00B6  START OF FUNCTION (2\*StrLocals + 3\*NumLocals)

00002054: 0013  StrVar[001C] = ""

0000205C: 0013  StrVar[001D] = ""

00002064: 0013  StrLocal[0001] = ""

0000206C: 0013  StrLocal[0002] = ""

00002074: 0002  Disable (00000032)

<LABEL\_0039> REF: 00002147

  |

0000207F: 0021  NumLocal[0002] = 00000000

0000208D: 0129  WHILE (NumLocal[0002] = 00000000)

000020AF: 00B5        NumLocal[0001] = SdRegisterUserEx\_[LABEL\_00B0]

(StrLocal[0001],StrLocal[0002],StrVar[001C],StrVar[001D],StrVar[001E])

000020CE: 00B5        NumLocal[0002] = SdConfirmRegistration\_[LABEL\_00E5]

(StrLocal[0001],StrVar[001C],StrVar[001D],StrVar[001E],00000000)

000020EF: 0000  ENDWHILE

//以上这个WHILE就是让你输入序列号并确认的。

00002108: 0128  IF (Call Function\_006F\_[LABEL\_004D] != 00000001) THEN

~~~~这里是不等于号

00002128: 002A        MessageBox ("Invalid Serial Number",SEVERE)

//在此处引用MSG字符串。

00002147: 002C        Goto (LABEL\_0039)

00002148: 0000  ENDIF

//然后调用LABEL\_004D处的Function\_006F来验证序列号，如果返回值不是1就表示出错，出现出错对话框。

00002150: 012F  Return (NumLocal[0001])

00002157: 00B8  END OF FUNCTION ()
```

因此这里最简单的想法是把`00002108`处的不等于改成等于，正巧`DIS`有这个功能，右键点击它选Change to，改成等于就行了。退出`DIS`的时候会提示是否重新校验`INS`文件，因为`SETUP`在运行时会首先检测`INS`文件的`CRC`校验和，因此这里修改后也要重新校验，选“是”后新的`INS`文件就产生了。

再运行NetScanTools Pro 2000的安装程序，序列号乱填一气，安装通过。

按理说通常的安装软件，破到这个程度也就差不多了，但NetScanTools Pro 2000多做了一点小手脚。本来我还挺高兴地运行`NSTPRO.EXE`想看看自己的成果，`NSTPRO.EXE`却忽地弹出个窗口说是“Invalid Installation……”并且要求重新安。？哇！看来`NSTPRO.EXE`启动时还有一步序列号的判断过程。序列号信息存放在什么地方呢？我连`REGMON`和`FILEMON`都懒得用了，直接打开注册表寻寻觅觅就找到了这一处：`HKEY_LOCAL_MACHINE\Software\Northwest Performance Software, Inc.\NetScanTools Pro\2000\NETSCANTOOLSPRO`，下面是输入的用户名和序列号等。

首先就觉得似乎从`NSTPRO.EXE`中分析序列号算法还不如从`INS`文件中分析来得简单，既然脚本中判断序列号是否正确的子程序在`LABEL_004D`处，就把它的内容列出来看看啦：

```
<LABEL\_004D> REF: 00002108

  |

00002A92: 00B6  START OF FUNCTION (3\*StrLocals + 10\*NumLocals)

00002AA4: 0021  NumLocal[0005] = 00000081

//一个固定值

00002ABB: 0128  IF (StrLength (StrVar[001E]) < 00000018) THEN

00002ADB: 012F        Return (00000000)

00002ADC: 0000  ENDIF

//如果序列号长度小于18H则出错

00002AE8: 0128  IF (NumLocal[0006] > 00000019) THEN

00002B08: 012F        Return (00000000)

00002B09: 0000  ENDIF

//长度大于19H也出错，因此序列号只能是18H位或19H位。

00002B15: 0128  IF (NumLocal[0006] = 00000018) THEN

//如果长度是18H，则……

00002B35: 0021        NumLocal[0003] = 00000000

//N3是SL3的字串位置指针

00002B3F: 0021        NumLocal[0002] = 00000013

//N2是序列号的字串位置指针

<LABEL\_0050> REF: 00002B9D

  |

00002B4D: 0128        IF (NumLocal[0002] <= 00000017)

THEN

00002B6D: 007A            NumLocal[0009] = GetByte

(StrVar[001E],NumLocal[0002])

//SV1E是用户输入的序列号

00002B78: 007B            SetByte (StrLocal[0003],NumLocal[0003],NumLocal[0009])

00002B83: 0119            NumLocal[0003] = NumLocal[0003]

+ 00000001

00002B90: 0119            NumLocal[0002] = NumLocal[0002]

+ 00000001

00002B9D: 002C            Goto (LABEL\_0050)

00002B9E: 0000        ENDIF

//将SL3的值赋成序列号中的第13H位到17H位子串，该子串是数字。

00002BA6: 0021        NumLocal[0006] = 00000013

//并且N6记录了该数字字串在序列号中出现的位置

00002BA7: 0000  ENDIF

//如果长度是19，则……

00002BB4: 0128  IF (NumLocal[0006] = 00000019) THEN

00002BD4: 0021        NumLocal[0003] = 00000000

00002BDE: 0021        NumLocal[0002] = 00000014

<LABEL\_0053> REF: 00002C3C

  |

00002BEC: 0128        IF (NumLocal[0002] <= 00000018)

THEN

00002C0C: 007A            NumLocal[0009] = GetByte

(StrVar[001E],NumLocal[0002])

00002C17: 007B            SetByte (StrLocal[0003],NumLocal[0003],NumLocal[0009])

00002C22: 0119            NumLocal[0003] = NumLocal[0003]

+ 00000001

00002C2F: 0119            NumLocal[0002] = NumLocal[0002]

+ 00000001

00002C3C: 002C            Goto (LABEL\_0053)

00002C3D: 0000        ENDIF

00002C45: 0021        NumLocal[0006] = 00000014

00002C46: 0000  ENDIF

00002C53: 0021  NumLocal[0002] = 00000000

//如果长度是19H，则取14H到18H位到SL3中。

<LABEL\_0056> REF: 00002D5D

  |

//到这里为止，N2是下一步即将进行的处理中的字串指针

//下一步是个大循环。

00002C61: 007A  NumLocal[0009] = GetByte (StrVar[001E],NumLocal[0002])

//N9是输入序列号的某个字符

00002C6C: 011D  NumLocal[0007] = 000000FF & NumLocal[0009]

//N7=N9 AND 0xFF，个人感觉N7实际上等于N9

00002C79: 0119  NumLocal[0002] = NumLocal[0002] + 00000001

00002C86: 0021  NumLocal[0003] = 00000000

//N3是循环变量，此处初始化

<LABEL\_0057> REF: 00002D35

  |

//以下是个小循环，处理序列号中的一个字符

00002C94: 0128  IF (NumLocal[0003] <= 00000007) THEN

00002CB4: 011D        NumLocal[0009] = NumLocal[0005]

& 00000001

00002CC1: 011D        NumLocal[000A] = NumLocal[0007]

& 00000001

00002CD9: 0022        IF (NumLocal[0009] ^ NumLocal[000A]

!= 00000000) THEN

00002CE7: 0122            NumLocal[0009] = NumLocal[0005]

>> 00000001

00002CF4: 011F            NumLocal[0005] = NumLocal[0009]

^ 00008408

00002D01: 0000        ELSE

00002D0A: 0122            NumLocal[0005] = NumLocal[0005]

>> 00000001

00002D0B: 0000        ENDIF

00002D1B: 0122        NumLocal[0007] = NumLocal[0007]

>> 00000001

00002D28: 0119        NumLocal[0003] = NumLocal[0003]

+ 00000001

00002D35: 002C        Goto (LABEL\_0057)

00002D36: 0000  ENDIF

//小循环结束

00002D5D: 0022  IF (NumLocal[0006] -  = 00000000) THEN GOTO LABEL\_0056

//减一后等于0就跳回去继续循环？不等呢就结束循环？这是疑点之一。

//大循环结束后开始验证

00002D6B: 0120  NumLocal[0007] = NumLocal[0005] ~ NumLocal[0005]

00002D7E: 0121  NumLocal[0009] = NumLocal[0005] << 00000008

00002D8B: 0122  NumLocal[000A] = NumLocal[0007] >> 00000008

00002D98: 011D  NumLocal[000A] = NumLocal[000A] & 000000FF

00002DA5: 011E  NumLocal[0005] = NumLocal[0009] | NumLocal[000A]

00002DB0: 011D  NumLocal[0005] = NumLocal[0005] & 0000FFFF

//最后的结果是N5，与SL3的数字值比较，等则表示输入的序列号合法。

00002DCD: 0128  IF (StrToNum (NumLocal[0004],StrLocal[0003]) < 00000000)

THEN

00002DED: 012F        Return (FFFFFFFF)

00002DEE: 0000  ENDIF

00002DFA: 0128  IF (NumLocal[0005] != NumLocal[0004]) THEN

00002E18: 012F        Return (FFFFFFFF)

00002E21: 0000  ELSE

00002E2A: 012F        Return (00000001)

00002E2B: 0000  ENDIF

00002E37: 00B8  END OF FUNCTION ()
```

上边我能看懂的地方加了一些注释，名为`STRVAL`的都是字符串型全局变量，名里有local的都是局部变量。`StrVar[001E]`是我们输入的序列号。这个程序首先判断序列号是否是18H或19H长，然后取出序列号的最后五个数字。程序末尾把运算的结果与这个五位数字值比较，不等则出错。整个运算过程虽不长，却有点不符合常规。――我手头没有详细介绍INSTALLshield脚本语法的资料，上次皮特陈兄虽做了一个，却不敢看（老死机），只有猜测着来。与或非左右移等符号和C中差不多，`IFWHILEENDIF`等和VB又差不多。我按照我的理解编了一段C代码来模仿这段运算过程。先输入19个字符（0到0x13），再把这些字符运算得到五位数字（0x13到0x17），但拼凑起来的序列号根本通不过。我一直以为是我的脚本算法理解错误，山穷水尽了好几天也没什么结果（对自己的水平没有自信啦）。

我也打算去找一个正确的序列号来验证自己的想法，可这种软件的这个版本的序列号在什么啊嘶嗒啦唯嘶嗒上是找不到的。再后来，“我本善良”兄给了我一个能用的序列号`9351324-005376-001-32602`，确实后五位是数字，前19位应该是输入的东西，只是前19位看起来有了许多限制，比如哪几位是数字，哪一位是“-”等，这在脚本里却没找到。

现在只有从`NSTPRO.EXE`入手分析读注册表中序列号的一段了。老法子，弹出出错框时切入`TRW`查输入的序列号再下`Bpm`内存断点，偏偏`TRW`不稳定，内存断点又断不了，我气得扔下它不干了。过了几天，该忙的似乎应付了一点后，弄了个`SOFTICE`来，这下子稳定多了。――由于这段时间忙，搞点破解只能挤出时间来，经常是研究几个小时又停几天，这就叫“三天破解，两天晒网”，这种不良的风气值得批斗。

后来通过几个`BPM`断点来到了这一段：

```
\* Reference To: KERNEL32.lstrlenA, Ord:02A1h

            |

:00438D10 FF15C0D24900            Call dword

ptr [0049D2C0]

:00438D16 83F818

cmp eax, 00000018

:00438D19 0F858D000000            jne 00438DAC

:00438D1F 8D8C2494000000          lea ecx, dword

ptr [esp+00000094]

\* Possible Reference to String Resource ID=00019: "Edit the list of IP addresses

and Hostnames."

            |

:00438D26 6A13

  push 00000013

:00438D28 51

    push ecx

:00438D29 E8C2210000              call

0043AEF0

:00438D2E 83C408

add esp, 00000008

:00438D31 8D942494000000          lea edx, dword

ptr [esp+00000094]
```
