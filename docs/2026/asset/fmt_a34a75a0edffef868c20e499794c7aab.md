
具体怎么找到`6D88CC`的，记不太清了，好像运气比较好，^\_^。

对于`1.5`种的`006D03CE     mov     eax, [6D91D8]`

在`1.0`种对应的代码是：`006837F6  mov     eax, [6C9028]`

通过在`1.0`中`ctrl+s`,搜索`mov     eax, [6C9028]`，可以得到好几处，在通过某一处很有特点的代码，在`1.5`中搜索到这段代码，进而找到`1.5`中相对应的`mov     eax, [6D91D8]`。

这样，四个基本功能都修复完毕。还有几个小地方。

可以利用`DeDe`，找到结果面板中保存为文本文件所对应的过程的起始地址：

反编译后，找到所有的`save*`事件，然后全部下断点，再点保存，中断在哪里就是哪里了。

### 第五处：

`1.5`中修改后：

```
006986BC     jmp     006D0670      ;  跳到其他地方去，Save2textClick
006986C1     nop
006D0670     push    ebp             ;  ResultSave2TxtClick
006D0671  |.  8BEC           mov     ebp, esp
```

[省略…]

修复的方法同上。

`1.5`修改前：

```
006986BC    push    ebx
006986BD    mov     ebx, [eax+380]
```

[省略…]

```
006986F3     je      short 006986FF
006986F5     mov   eax, 00698750     ;  未注册版不能保存结果内容！
006986FA     call    0046E9B8        ;  D7.Dialogs.ShowMessage(AnsiString);
006986FF     pop     ebx
00698700     retn
```

`1.0`中：

```
0068D01C   push    ebp               ;  Save2textClick
0068D01D   mov     ebp, esp
```

[省略…]

### 第六处：解决限制⑥

`DeDe`反编译，找到`RzShellList1DblClick`事件。

`1.5`中：

```
0063338C     push    ebp            ;  RzShellList1DblClick
```

`1.0`中：

```
0062C964     push    ebp            ;  RzShellList1DblClick
```

同时运行`1.0`和`1.5`，观察程序流程，容易找到关键地方。

`1.0`中：

```
0062CAC5    mov     edx, 0062CCC8       ;  ASCII ".mff"
0062CACA    call      00404910           ;  D7.System.@LStrCmp;
0062CACF    je      short 0062CAE3
0062CAD1    mov     eax, 2
0062CAD6    call    0062A538             ；校验注册没有
0062CADB    test    al, al
0062CADD    je      0062CC54
0062CAE3    push    0062CCD8            ;  ASCII "1stOpt - ["
0062CAE8    push    dword ptr [ebp-4]
```

`1.5`修改前：

```
006334D7    mov     edx, 006336C8    ;  ASCII ".mff"
006334DC   call    00404910           ;  D7.System.@LStrCmp;
006334E1    je      short 0063350C
006334E3    push    006336D8          ;  ASCII "1stOpt - ["
006334E8    push    dword ptr [ebp-4]
```

在`1.0`中`0062CACF    je      short 0062CAE3`直接跳到了

```
0062CAE3    push    0062CCD8            ;  ASCII "1stOpt - ["
```

而在`1.5`中，`006334E1    je      short 0063350C`

把`006334E3    push    006336D8          ;  ASCII "1stOpt - ["`

这句跳过了。

清楚了，把 `je      short 0063350C` 直接`nop`掉。

保存后运行，OK！

### 第七处：解决新建文件后不能保存文件(窗口标题已带绝对路径的可以保存)。

新建文件，点工具栏的保存图标，中断在下面：

`1.5`中：

```
0068E754      push    ebp              ;  SaveActionExecute
```

`1.0`中：

```
00683C40      push    ebp              ;  SaveActionExecute
```

同时调试`1.0`和`1.5`，比较流程，这个还比较难找：

`1.5`中，跟到这里：

```
006908A0      push    ebp              ;  savefile
006908A1      mov     ebp, esp
006908A3      mov     ecx, 0D
```

[省略…]

```
00690995   .  84C0           test    al, al
00690997   .  0F84 EA020000  je      00690C87
0069099D   .  EB 31          jmp     short 006909D0   ;这是修改后的代码
0069099F      90             nop
006909A0   .  8BC3           mov     eax, ebx
```

`1.5`修改前的代码：

```
00690995      test    al, al
00690997      je      00690C87
0069099D     lea     edx, [ebp-20]    //这段代码要跳过
006909A0     mov     eax, ebx
006909A2     call    0046CC0C
006909A7     mov     eax, [ebp-20]
```

`1.0`中：

```
00685DE8      push    ebp                ;  save2file
00685DE9      mov     ebp, esp
00685DEB      mov     ecx, 0D
```

[省略…]

```
00685EDD     test    al, al
00685EDF     je      006861DD
00685EE5     mov     eax, 3
00685EEA     call    0062A538 ；检验注册没有
00685EEF     test    al, al
00685EF1     jnz     short 00685F26
00685EF3     lea     edx, [ebp-20]
00685EF6     mov     eax, ebx
00685EF8     call    0046CC0C
```

对比`1.0`和`1.5`的流程，可以找到关键点。

新建文件后，窗口标题能变为绝对路径，可以保存文件。

### 第八处：修改关于窗口中的“未注册版”

用`ultraedit`，搜索到“未注册版”，改为“  winndy”。

不太会修改资源，未注册版有`8`个`bytes`，所以也该成`8`个`bytes`了，在`winndy`前还加了两个空格。

到这里，`1stOptv1.5`修复完毕。没什么技巧，完全是体力活啊！

`1.0`的注册机还没做出来呢。

## 三、追本溯源：Auto2Fit v3.0和THKStreams Delphi Component

在跟踪`1stOptv1.5`的时候，用`DFMEditor`查看资源，可以发现一个`TREGFORM`，打开一看，“Auto2Fit Reristration”，“Send Mail to CPC-X Software”，于是google了个`Auto2Fit v3`来玩玩。安装后，界面和`1stOpt`没啥两样，只是`Auto2Fit`是英文界面而已。`Auto2Fit`是`1stOpt`的前身？在`Auto2Fit`中看到了很多参考字符串，这个好像更适合做注册机。下面就跟踪`Auto2Fit`。

点关于，停在下面：

```
005FB4A4  push    ebp
```

[省略…]

```
005FB4F8  mov  ecx, 005FBD60      ;  ASCII "CPUHDID.txt"
005FB4FD  mov   edx, [ebp-38]
005FB500  call    00404808
005FB505  mov   eax, [ebp-214]
005FB50B  call   0040CB2C          ;  D7.SysUtils.FileExists(AnsiString):Boolean;
005FB510  test    al, al
005FB512  jnz    short 005FB588
005FB514  lea    eax, [ebp-218]
005FB51A  mov  ecx, 005FBD60       ;  ASCII "CPUHDID.txt"
```

检查`exe`的目录中是否存在`CPUHDID.txt`，若不存在则创建，第一行是`CPUID`,第二行是`HardDiskID`。很容易得到生成`CPUID`和`HardDiskID`的算法。写注册机时会用到。

```
005FB9ED  cmp    dword ptr [6A77A8], 1
005FB9F4   je     short 005FB9FF
005FB9F6  cmp    dword ptr [6A77A8], 5
005FB9FD  jnz     short 005FBA0E
005FB9FF  lea     eax, [ebp-34]
005FBA02  mov    edx, 005FBDEC     ;  ASCII " (Single User License)"
005FBA07  call    004045B8
005FBA0C  jmp    short 005FBA6F
005FBA0E  cmp    dword ptr [6A77A8], 2
005FBA15  je      short 005FBA20
005FBA17  cmp     dword ptr [6A77A8], 6
005FBA1E  jnz     short 005FBA2F
005FBA20  lea     eax, [ebp-34]
005FBA23  mov   edx, 005FBE0C     ;  ASCII " (2-4 Users License)"
005FBA28  call    004045B8         ;  D7.System.@LStrLAsg(void;void;void;void);
005FBA2D  jmp   short 005FBA6F
```

[省略…]

很明显，`[6A77A8]`是注册的类型。

`al=0`      `trial user`

`al=1 ,5    (Single User License)`

`al=2 ,6    (2-4 Users License)`

`al=3 ,7    (5-10 Users License)`

`al=4 ,8    (Site License)`

`1<=al<=4   "Standard Version "`

`5<=al<=8   "Professional Version "`

往上面看，有两处`call`：

```
005FB98C     call    005FAC00
005FB9B8     call    005FB290
```

都很重要。

第一个`call`读注册表中的键值`(`有两处`)`，检查`Auto2Fit.Lic`；

第二个`call`检查`AFCorelib.dll`。

跟进后会发现：首先读

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\OfficeAF`

下的几个值：

1. `eu_id`:

“`95-120-124-111-113-115-125-126-111-124-111-110-42-96-111-124-125-115-121-110`”

2. `gc_id:6`

3. `st_id`:

`64`位浮点数：

`00 00 00 00 E0 06 E3 40`

`38967.000000000000000`

4. `vs_id:0`

注意只有在`gc_id=0`的情况下，采取读`Auto2Fit.Lic`。

其中`eu_id`经过一个函数解密为用户名：

```
005FADA5  call    005FA7A0                     ;  用户名解密
```

“`95-120-124-111-113-115-125-126-111-124-111-110-42-96-111-124-125-115-121-110`”

就是`Unregistered Version`。

然后会压入这些参数，进入一个`call`：

```
005FADAA  mov     eax, [ebp-120]                 ;  用户名
005FADB0  mov     edx, [ebp+8]                   ;  p_vs_id
005FADB3  mov     edx, [edx]                     ;  vs_id
005FADB5  call    005FA8C4                      ;  校验licensefile
005FADBA  mov     [ebp-10], eax
005FADBD  cmp     dword ptr [ebp-10], 0
005FADC1   jnz     short 005FADDA        ;  跳
005FADC3   push    0                     ; /Arg1 = 00000000
005FADC5   mov     cx, [5FB210]           ; |
005FADCC  mov     dl, 2                   ; |
005FADCE   mov     eax, 005FB21C         ; |ASCII "Missing Auto2Fit License File!"
005FADD3   call    0046E004               ; \Auto2Fit.0046E004
005FADD8   jmp     short 005FADF8
```

`Auto2Fit.Lic`是没有的，得自己构造一个假`Auto2Fit.Lic`。

难点就在于这里了，难得构造一个格式符合要求的`Auto2Fit.Lic`，能通过这段验证，每次都说“`Auto2Fit.Lic`已损坏”。一层一层的跟进去，头都大了，仅知道第`3`个`byte`开始，必须是`01 02 …08`。

    在这里卡了。

后来，查看`DeDe`反汇编出来的代码的时候，看到:

```
* Reference to class THKStreams
|
005FA9F8   A1DC255300             mov     eax, dword ptr [$005325DC]
* Reference to : THKStreams._PROC_005327CC()
|
005FA9FD   E8CA7DF3FF             call    005327CC
005FAA02   8945EC                 mov     [ebp-$14], eax
005FAA05   8B45EC                 mov     eax, [ebp-$14]
* Reference to field THKStreams.OFFS_0024
|
005FAA08   C6402401               mov     byte ptr [eax+$24], $01
005FAA0C   8B45EC                 mov     eax, [ebp-$14]
```

不禁产生好奇感，这个`THKStreams`究竟是什么类啊！？

`Google`一下，原来是个`delphi` 流文件加密的组件，开源的，太好了，有救了。

下了`1.7`的：`THKStreams v1.7 by Harry Kakoulidis 1/2002`

里面还有个`demo`，演示`THKStreams`的使用。

用`Delphi`打开组件源码和`demo` 工程，组件采用了`blowfish`加密和`LHA`压缩算法。学习了一下这些源码。一边跟踪，一边对照源码，很容易就识别出了`Auto2Fit`中的那些`call`。用`demo`工程中的一个`memo`，生成了`Auto2Fit.Lic`。但还是出错，然后又`google`，恶补了一下`TStringList`的用法。又仔细看了看`demo`的代码。

`HKS.AddStream('MEMO1',ms);    //Add it to THKStreams with ID 'MEMO1'`

关键在上面这句，`'MEMO1'`相当于流文件中的一个标签了，在我们的`Auto2Fit.Lic`中对应的标签是什么呢。后来跟踪`Auto2Fit`，发现标签是`'AFLicenseFile'`。

重新伪造`Auto2Fit.Lic`，继续调试。

参考`HKStreams`的源码和`demo`工程的代码，以及`DeDe`反汇编出来的代码，还有`CodeHelper`插件，可以给出很好的注释：

```
005FAAAC    mov     edx, 005FABA4   ;  ASCII "AFLicenseFile"
005FAAB1    mov     eax, [ebp-14]
```
