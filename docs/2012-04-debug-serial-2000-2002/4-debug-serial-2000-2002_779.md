
# Cute Email Searcher 2.2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue249.htm`](https://www.pediy.com/kssd/pediy04/kanxue249.htm)

**作者**：`Maomao[CCG]`  
**下载**：`http://www.ftpeasy.com/`

使用`FI`分析，它用`UPX1.06`加壳，小意思，用`Procdump`脱壳成功！运行，屏幕一闪，自动退出了。看来还有校验！用`TRW`直接载入原程序，蓝屏！只好从脱好壳的程序入手了。

用`TRW`载入脱壳后的程序，一路`F10`，来到这里时，出现程序界面：

```asm
0177:00492989  CALL   `KERNEL32!GetModuleHandleA`
0177:0049298E  PUSH   EAX
0177:0049298F  CALL   NEAR [ESI+18]　<----出现界面
0177:00492992  ADD    ESP,BYTE +10
0177:00492995  PUSH   EAX
0177:00492996  CALL   00491E58　　　　<----到这里就结束了
0177:0049299B  POP    ECX
0177:0049299C  JMP    SHORT 004929BF
```

在`0177:0049298F`设断后跟进去，按`F10`一直来到这里：

```asm
0177:004013BF  MOV    WORD [EBP-20],00
0177:004013B3  MOV    EAX,[004A3698]
0177:004013B8  MOV    EAX,[EAX]
0177:004013BA  CALL   00453958　　　　　　<----出现界面
0177:004013BF  MOV    WORD [EBP-20],00
0177:004013C5  JMP    SHORT 004013E2　　　<----这里返回就完了
```

再将`0177:004013BA`设断，跟进去：

```asm
0177:004539D6  MOV    EAX,[EBP-04]
0177:004539D9  CALL   004537C4           
0177:004539DE  MOV    EAX,[EBP-04]
0177:004539E1  CMP    BYTE [EAX+84],00
0177:004539E8  JZ     004539D6           <----判断
0177:004539EA  XOR    EAX,EAX
0177:004539EC  POP    EDX
```

上面这段是对程序本身和程序事件进行检测，其中`0177:004539E8`的判断会在第三次执行和程序退出时不跳，程序会退出。跟进`0177:004539D9`的`call`：

```asm
0177:004537C4  PUSH   EBX
0177:004537C5  ADD    ESP,BYTE -1C
0177:004537C8  MOV    EBX,EAX
0177:004537CA  MOV    EDX,ESP
0177:004537CC  MOV    EAX,EBX
0177:004537CE  CALL   0045372C <-----修改标志的 call
0177:004537D3  TEST   AL,AL
0177:004537D5  JNZ    004537E0
0177:004537D7  MOV    EDX,ESP
0177:004537D9  MOV    EAX,EBX
0177:004537DB  CALL   00453F48
0177:004537E0  ADD    ESP,BYTE +1C
0177:004537E3  POP    EBX
0177:004537E4  RET
```

再跟进去，`F10`一会儿来到这里：

```asm
0177:0045373F  CALL   `USER32!PeekMessageA`
0177:00453744  TEST   EAX,EAX
0177:00453746  JZ     004537BD
0177:00453748  MOV    BL,01
0177:0045374A  CMP    DWORD [EDI+04],BYTE +12 <-----12 是退出标志
0177:0045374E  JZ     004537B6               <-----跳走就退出程序了
```

我们的目的在于分析程序的注册算法，如果要暴破，可以分析`0177:0045373F CALL USER32!PeekMessageA`为什么会送退出程序的消息到标志位。这里，将`0177:0045374E JZ 004537B6`改为两个`nop,nop`，程序就可以运行了，不过有副作用，就是不能正常退出了（正确的退出信息也被忽略了）。可以用`ctrl+Alt+Del`中止。现在，我们可以放心的使用`TRW`载入它，并分析它的注册算法了。

用`TRW`载入修改后的程序，注册，输入用户名`Maomao[CCG]`，注册码`87654321`，按确定，出现提示信息，意思是请重新启动后检查注册情况，看来是用`KeyFile`的方式或注册表方式注册，启动时检查。

用`FileMon`和`RegMon`跟踪程序，可以发现它在`windows`目录中写下了一个文件`ces.ini`，打开后发现有这样的信息：

```ini
[options]

ym=87654321

ymuser=Maomao[CCG]
```

一般`KeyFile`的注册要读文件，要用到下列常用的函数：`ReadFile`、`CreateFile`、`GetFileAttributesA`等等，也有的用不常用的如`_lopen`等。这里因为此文件在`windows`目录，可能会用到读`windows`目录的函数`GetWindowsDirectoryA`，在`TRW`中下`bpx GetWindowsDirectoryA`，载入程序，中断后，`pmodule`会来到这里：

```asm
0177:00401BB5  CALL   `KERNEL32!GetWindowsDirectoryA`
0177:00401BBA  MOV    WORD [EBP-6C],14
0177:00401BC0  MOV    EDX,00498606 <-----D EDX 可以看到 ces.ini 字样
0177:00401BC5  LEA    EAX,[EBP-18]
0177:00401BC8  CALL   004965C4
```

下面经过一长串的定位和读的过程，我们按`F10`一直来到这里：

```asm
0177:00401E29  MOV    DWORD [EBP+FFFFFF74],00498628
0177:00401E33  MOV    DWORD [EBP+FFFFFF6C],09C4 <-----放一个基数 9c4 到累加器
0177:00401E3D  MOV    DWORD [EBP+FFFFFF70],00
0177:00401E47  XOR    EBX,EBX
0177:00401E49  XOR    EAX,EAX
0177:00401E4B  MOV    [EBP+FFFFFF68],EAX
0177:00401E51  LEA    EDX,[EBP+FFFFFA54]
0177:00401E57  MOV    [EBP+FFFFFF64],EDX
0177:00401E5D  JMP    SHORT 00401EA5
0177:00401E5F  XOR    EDX,EDX
0177:00401E61  MOV    EAX,[EBP+FFFFFF74]      <-----取字符串"TangSe1980"
0177:00401E67  INC    EAX
0177:00401E68  MOV    ECX,[EBP+FFFFFF64]      <-----取机器号，我的为 78030
0177:00401E6E  MOVSX  EBX,BYTE [ECX]          <-----取机器号第 i 字符
0177:00401E71  MOVSX  ESI,BYTE [EAX]          <-----取字符串第 i+1 字符
0177:00401E74  MOVSX  EDI,BYTE [EAX-01]       <-----取字符串第 i 字符
0177:00401E78  ADD    ESI,EDI                <-----两字符相加
0177:00401E7A  IMUL   ESI,EBX                <-----与第 i 机器号相乘
0177:00401E7D  MOV    EBX,ESI
0177:00401E7F  ADD    EDX,BYTE +02
0177:00401E82  ADD    EAX,BYTE +02
0177:00401E85  CMP    EDX,BYTE +05           <-----字符串取过 5 位？
                                               (只有与字串 5、6 位的运算结果才会被保存 )
0177:00401E88  JL     00401E6E
0177:00401E8A  MOV    EAX,EBX                <-----是则保留这一次结果
0177:00401E8C  CDQ
0177:00401E8D  ADD    [EBP+FFFFFF6C],EAX     <-----与累加器相加
0177:00401E93  ADC    [EBP+FFFFFF70],EDX
0177:00401E99  INC    DWORD [EBP+FFFFFF68]
0177:00401E9F  INC    DWORD [EBP+FFFFFF64]
0177:00401EA5  LEA    ECX,[EBP+FFFFFA54]
0177:00401EAB  PUSH   ECX
0177:00401EAC  CALL   0048C828
0177:00401EB1  POP    ECX
0177:00401EB2  CMP    EAX,[EBP+FFFFFF68]
0177:00401EB8  JA     00401E5F              <-----取下一位机器号
```

这就是它的注册算法！

这段程序完全什么样的功能呢？取机器号的每一位与特征字串`"TangSe1980"`的第`5`、`6`位的和相乘，再累加，最后的结果再与基数`0X9C4`相加得到注册码。

下面的程序是读上次输入的用户名和注册码，分析发现，用户名不参与运算，输入的注册码与上面的计算的结果进行比较！看来正确的注册码与用户名无关。

而且还有一个奇怪的发现，测试了三台不同的机器，机器码是一样的！都是`78030`！不知道是不是所有的机器码都是`78030`呀？呵呵:)

好了，编一个简单的注册机应该没问题了吧。

收工~~~ `4dd466d0e1e0cc123afbe504da2462ca`
