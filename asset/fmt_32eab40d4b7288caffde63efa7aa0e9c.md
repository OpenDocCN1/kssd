
## 汇编代码片段

```
:00512D88 83C4F4        add esp, FFFFFFF4   
:00512D8B DB3C24        fstp tbyte ptr [esp]   
:00512D8E 9B            wait   
========================================
```

### 总结

`name`: `BCG`

`code`: `A02-440-368`

最后用 `keymake` 做个另类注册机：

1. 按“浏览”按钮，选择 `FlashCam.exe`
2. 第一个中断地址：
    中断地址：`51247e`
    中断次数：`1`
    第一字节：`e8`
    指令长度：`5`
   第二个中断地址：
    中断地址：`512eef`
    中断次数：`1`
    第一字节：`8b`
    指令长度：`3`
3. 选“内存方式”，并勾选 `EAX`
4. 点击生成 `EXE` 文件

企业版的注册码好像是通用的，本菜鸟水平有限，希望大虾们写出它的 `keygen` 并贴上源码，好让我们初学者能学上一手。

整理于 2002.4.23 11:47

# FlashCam V1.75 的破解

`FlashCam` 可以抓取影像为 `Flash` 文件，还可以设定测鼠移动路径、加入文字、外挂说明声效。(取自海阔天空下载站的说明)

看看 `FlashCam1.75` 的帮助文件，它分未注册试用版和注册版(个人版、专业版、企业版)四种。未注册版本有 30 天的使用期限，且启动时有延时窗口，用它制作的影像文件上有广告等。注册版则没有以上限制，并且有些加强功能。

个人版、专业版、企业版功能依次加强，个人版和专业版只能在单个机器上注册使用，而企业版则可以在任意机器上注册使用。

下面我们来破解它，找出注册码来。

- **目标**：`FlashCam V1.75`
- **工具**：`TRW2000` & `DeDe`
- **作者**：`chilly200[BCG]`
- **时间**：2002.4.5

`FlashCam1.75` 是用 `ASpack2.x-modified` 加壳的，用 `Aspackdie1.3` 轻松脱掉。

由于是 `Delphi` 程序，用 `DeDe` 反编译，很容易找到注册的地方，如下：

```
018F:005114EE  PUSH    DWORD [EBP-0C]
018F:005114F1  PUSH    DWORD [EBP-10]
018F:005114F4  LEA     EAX,[EBP-18]
018F:005114F7  PUSH    EAX
018F:005114F8  MOV     ECX,20
018F:005114FD  MOV     EDX,9DF1
018F:00511502  MOV     EAX,1A
018F:00511507  CALL    0051117C      <-- 计算个人版注册码的 CALL
018F:0051150C  MOV     EAX,[EBP-18]  <-- 本机个人版正确注册码
018F:0051150F  MOV     EDX,[EBP-04]
018F:00511512  CALL    0040419C      <-- 比较个人版注册码
018F:00511517  JNZ     0051151D      <-- 不跳即注册成功，暴破点！
018F:00511519  MOV     BL,01
018F:0051151B  JMP     SHORT 0051158A
018F:0051151D  PUSH    DWORD [EBP-0C]
018F:00511520  PUSH    DWORD [EBP-10]
018F:00511523  LEA     EAX,[EBP-1C]
018F:00511526  PUSH    EAX
018F:00511527  MOV     ECX,21
018F:0051152C  MOV     EDX,9CCB
018F:00511531  MOV     EAX,1B
018F:00511536  CALL    0051117C       <-- 计算专业版注册码
018F:0051153B  MOV     EAX,[EBP-1C]    <-- 本机专业版正确注册码
018F:0051153E  MOV     EDX,[EBP-04]
018F:00511541  CALL    0040419C       <-- 比较专业版注册码
018F:00511546  JNZ     00511553       <-- 不跳即成专业版，暴破点！
018F:00511548  MOV     BL,01
018F:0051154A  MOV     BYTE [EDI+07E9],01  <-- 专业版多了这个
018F:00511551  JMP     SHORT 0051158A
018F:00511553  CMP     DWORD [EBP-08],BYTE +00  <-- 下面是企业版注册
018F:00511557  JZ      00511588                 <-- 要有公司名
018F:00511559  MOV     EDX,[EBP-04]
018F:0051155C  MOV     EAX,[EBP-08]
018F:0051155F  CALL    00511368       <-- 计算比较企业版注册码
018F:00511564  MOV     EBX,EAX
018F:00511566  MOV     [EDI+07EA],BL   <-- 企业版多了这个(BL=1 成功)
018F:0051156C  TEST    BL,BL           <-- 成为企业版的暴破点！
018F:0051156E  JNZ     0051157F
018F:00511570  MOV     EDX,[EBP-04]    <-- 下面好像又成个人版了
018F:00511573  MOV     EAX,[EBP-08]      (这个个人版与机器无关?)
018F:00511576  CALL    00511400       <-- 计算比较注册码
018F:0051157B  MOV     EBX,EAX         <-- EAX=1 个人版注册成功
018F:0051157D  JMP     SHORT 0051158A
018F:0051157F  MOV     BYTE [EDI+07E9],01
018F:00511586  JMP     SHORT 0051158A
018F:00511588  XOR     EBX,EBX
018F:0051158A  XOR     EAX,EAX
018F:0051158C  POP     EDX
018F:0051158D  POP     ECX
018F:0051158E  POP     ECX
```

程序在注册、显示 `About`、启动界面、延时窗口等地方都是调用的这段代码，因此在以上各个暴破点上修改一下就可以分别暴破成个人、专业、企业版。

或者从上面标注的地方也可以看到本机上的正确的个人版和专业版注册码，可以抄下来进行注册。但是这是和机器及安装目录有关的，换了地方就没用了。

我们还是来找企业版注册码吧，这个应该是放之四海皆准的吧。

跟进 `018F:0051155F CALL 00511368` `<-- 计算比较企业版注册码`

```
------------------------------------- Corporate Edition ($699 US)
       :
018F:005113B8  LEA     ECX,[EBP-14]
018F:005113BB  MOV     EDX,0147
018F:005113C0  MOV     EAX,[EBP-04]    <-- 公司名 BCG
018F:005113C3  CALL    00511250        <-- 计算企业版注册码的 CALL
018F:005113C8  MOV     EAX,[EBP-14]    <-- 真的注册码 D02-443-263
018F:005113CB  MOV     EDX,[EBP-08]
018F:005113CE  CALL    0040419C        <-- 比较注册码
018F:005113D3  SETZ    BL
018F:005113D6  XOR     EAX,EAX
018F:005113D8  POP     EDX
018F:005113D9  POP     ECX
018F:005113DA  POP     ECX
```

嘿嘿，现在我们就可以用下面的注册码注册成价值 $699 的企业版了！

**注册码**：`D02-443-263`

**公司名**：`BCG`

---

下面是企业版注册码的算法

```
       :
018F:00511259  XOR     EBX,EBX
018F:0051125B  MOV     [EBP-18],EBX    <-- 零
018F:0051125E  MOV     [EBP-0C],ECX    <-- 注册码的存放地址
018F:00511261  MOV     [EBP-08],EDX    <-- 常量 0x147
018F:00511264  MOV     [EBP-04],EAX    <-- 公司名
018F:00511267  MOV     EAX,[EBP-04]
018F:0051126A  CALL    00404240
018F:0051126F  XOR     EAX,EAX
018F:00511271  PUSH    EBP
018F:00511272  PUSH    DWORD 00511344
018F:00511277  PUSH    DWORD [FS:EAX]
018F:0051127A  MOV     [FS:EAX],ESP
018F:0051127D  MOV     EBX,07DB
018F:00511282  MOV     EAX,[EBP-04]
018F:00511285  CALL    0040408C        <-- 取公司名长度 len
018F:0051128A  MOV     ESI,EAX
018F:0051128C  TEST    ESI,ESI
018F:0051128E  JNG     005112DA
018F:00511290  MOV     EDI,01          <-- EDI= 1 to len
018F:00511295  MOV     EAX,[EBP-04]
018F:00511298  MOV     CL,[EAX+EDI-01]  <-- 循环取公司名的字符 CL
018F:0051129C  XOR     EAX,EAX
018F:0051129E  MOV     AL,CL
018F:005112A0  LEA     EDX,[EDI+0D]    <-- EDX=EDI+0D
018F:005112A3  IMUL    EDX
018F:005112A5  ADD     EBX,EAX          <-- EBX=EBX+EDX*CL
018F:005112A7  MOV     EAX,EBX          <-- EAX=EBX
018F:005112A9  MOV     EBX,3B9AC9FF
018F:005112AE  CDQ
018F:005112AF  IDIV    EBX
018F:005112B1  MOV     EBX,EDX          <-- EBX=EAX/0x3B9AC9FF 的余数
018F:005112B3  MOV     EAX,[EBP-04]
018F:005112B6  XOR     CL,45
018F:005112B9  XOR     EAX,EAX
018F:005112BB  MOV     AL,CL            <-- AL=CL 异或 0x45
018F:005112BD  IMUL    DWORD [EBP-08]
018F:005112C0  ADD     EBX,EAX          <-- EBX=EBX+AL*0x147
018F:005112C2  MOV     EAX,EBX          <-- EAX=EBX
018F:005112C4  MOV     ECX,3B9AC9FF
018F:005112C9  CDQ
018F:005112CA  IDIV    ECX
018F:005112CC  MOV     EBX,EDX          <-- EBX=EAX/0x3B9AC9FF 的余数
018F:005112CE  IMUL    EAX,EDI,0006302B
018F:005112D4  ADD     EBX,EAX          <-- EBX=EBX+EDI*0x6302B
018F:005112D6  INC     EDI              <-- EDI+1
018F:005112D7  DEC     ESI
018F:005112D8  JNZ     00511295         <-- 循环所有字符
018F:005112DA  MOV     EAX,EBX          <-- 计算结果 EBX 放入 EAX
018F:005112DC  MOV     ECX,05F5E0FF
018F:005112E1  CDQ
018F:005112E2  IDIV    ECX
018F:005112E4  MOV     EBX,EDX          <-- EBX=EAX/0x05F5E0FF 的余数
018F:005112E6  MOV     EAX,EBX             EBX 的十进制数就是注册码
018F:005112E8  MOV     ECX,06               的后 7 位数。
018F:005112ED  CDQ
018F:005112EE  IDIV    ECX
018F:005112F0  ADD     EDX,BYTE +41    <-- DL=EBX/6 的余数+0x41
018F:005112F3  MOV     [EBP-0D],DL      <-- DL 就是注册码的第一个字符
018F:005112F6  MOV     [EBP-14],EBX
018F:005112F9  FILD    DWORD [EBP-14]
018F:005112FC  ADD     ESP,BYTE -0C
018F:005112FF  FSTP    TWORD [ESP]
018F:00511302  WAIT
       :
```

到这，企业版的注册码就算出来了。呵呵，也不知道对不对，哪位朋友有兴趣来写个注册机吧。

另外，这个软件在 `Windows` 的 `System` 目录里放了一个假的 `DLL` 文件 `msrgl90.dll`，注册信息以及安装时间都是存在这个文件里面的，比较有意思，呵呵。

请将下面源程序保存为 `.rek` 文件，用注册机编写器编译，注册机编写器 1.73 测试通过!

```
.const
.data
szHomePage db "http://www.365hz.net",0
szEmail    db "mailto:ljyljx@163.com",0
szErrMess  db "输入的序列号不正确！",0
szFmt      db "%0.8u",0
data1      dd 86h,0    ;这个数是我猜的,破文中没写出来
szTemp     db 10 dup(0)
szCode     db 14 dup(0)

.code
mov ebx,7DBh
invoke lstrlen,eax
mov esi,eax
mov edi,1
PRO1:
lea eax,hInput1
```
