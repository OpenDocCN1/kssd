
# VB 快速逆向法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-172.htm`](https://www.pediy.com/kssd/pediy07/pediy7-172.htm)

**作者**：`nbw`  
**原作**：`B. Kathras`

一般来说 `VB` 程序很难入手分析，除了利用一些已经被分析出来的 `VB` 库导出函数，在很少有入口去进行分析了。我看了 `B. Kathras` 写的文章，觉得很不错，好东西不敢独享，这里就整理一下，给大家一起看看。

首先是如何处理 `nag` 窗口，然后是处理各种控件的属性。这些方法很实用，我也享受了一下 5 分钟 `ko` 的乐趣。

原作来自 `http://www.CodeBreakers-Journal.com`，这里不是翻译，我也没有把他里面说的所有东西都试验，因此有了什么问题你可以去找作者。

阅读以前最好能够理解 `VB` 里面一些基本术语，比如 `Form` 就是窗口，`Label` 就是标签等。不过也无大所谓。

在 `VB` 程序中，有很多方法可以初始化 `nag` 窗口。最常用的一种类似于：

```vb
Form.show
```

这句话被编译后，在程序中变成以下形式：

```assembly
:0040203D FF92B0020000 call dword ptr [edx+000002B0]
```

所以，反汇编源程序，搜索字符串 `+000002B0]` 就可以找到这个地方。其中的寄存器一般为 `edx`, `ecx` 或者 `eax` ...

如果要去掉这种 `nag`，可以去掉这个 `call`，但是有时候移除这个 `call` 的时候需要平衡堆栈，因为 `nop` 掉该函数会导致 `esp` 高 `24h`，可以采用以下方法移除：

```assembly
:0040203D 83C424 add esp, 00000024
:00402040 EB01 jmp 00402043
:00402042 90 nop
```

一般也可以通过对函数 `__vbanew2` 下断点断到这个地方。这个函数是窗口创建时候采用的函数，但是并不是类似于 `DialogBoxParamA` 的函数，而只是一个声明函数，比如：

```vb
Dim NagScreen as New NagForm
NagScreen.Show
```

另外一种显示 `nag` 窗口的方法是加载的时候利用窗口的 `visible` 属性。属性设定为 `false` 就不显示，否则就显示。属性的设置一般利用下面语句：

```vb
NagForm.Visible = True
```

编译后变成如下形式：

```assembly
:00401FF9 6AFF push FFFFFFFF
; 这是 VB 中的 TRUE 变量
:00401FFB 50 push eax
:00401FFC 898568FFFFFF mov dword ptr [ebp+FFFFFF68], eax
:00402002 8B10 mov edx, dword ptr [eax]
:00402004 FF92BC010000 call dword ptr [edx+000001BC]
; 初始化属性的函数
```

可以通过搜索特征码 `+000001BC]` 来定位到这个地方。

对付这种 `nag`，只需要修改一下属性变量就行了。`VB` 中 `00` 代表 `False`，所以把上面的变量改成 `00` 就可以，例如：

```assembly
:00401FF9 6A00 push 00
:00401FFB 50 push eax
:00401FFC 898568FFFFFF mov dword ptr [ebp+FFFFFF68], eax
:00402002 8B10 mov edx, dword ptr [eax]
:00402004 FF92BC010000 call dword ptr [edx+000001BC]
```

这样就可以移除该 `nag`。

## 改变对象属性

有很多对象属性我们都想要了解，我在这里列出来最常用的一些属性查找方法。

最常用的断点拦截函数是 `__vbaSetObj`，但是现在的软件都非常大，以至于通过拦截这个函数来确定属性很不现实。不过可以通过别的一些特征来判定属性。

### 对于按钮的 Enable 属性

一般程序实现如下：

```vb
Command1.Enabled = False
```

编译后如下：

```assembly
:00402018 57 push edi ; value of edi = 00 which in VB = False
:00402019 56 push esi
:0040201A 8B06 mov eax, dword ptr [esi]
:0040201C FF908C000000 call dword ptr [eax+0000008C] ; Initialization call
```

可以通过搜索特征：`+0000008C]` 来定位这个地方。

有很多方法来处理这里的属性，比如可以把 `402018` 地方入栈的代表 `TRUE` 的参数 `-1` 改成 `00`，表示 `False`。或者把这里的 `call` 给去掉，如下：

```assembly
:0040201C E901000000 jmp 00402022
:00402021 90 nop
```

请注意这种方法不仅仅适用于按钮，也适合处理 `text Box` 文本框的可用或者不可用。

### 按钮的 visible/invisible 属性

程序运行时改变按钮改属性的代码如下：

```vb
Command1.Visible = False
```

编译后，如下：

```assembly
:00402018 57 push edi ; value of edi = 00 which in VB = False
:00402019 56 push esi
:0040201A 8B06 mov eax, dword ptr [esi]
:0040201C FF9094000000 call dword ptr [eax+00000094] ; Initialization call
```

如果修改属性，可以通过修改 `push edi` 的值或者去掉这个函数。

请注意这个方法也适合处理 `Label` 标签的 `enableing/disabling` 属性。

### Label 标签的 visible/invisible 属性

如果去掉未注册信息，有时候需要处理 `Label` 标签的 `visible/invisible` 属性。程序中一般为：

```vb
Label1.Visible = False
```

编译后如下：

```assembly
:00402018 57 push edi ; value of edi = 00 which in VB = False
:00402019 56 push esi
:0040201A 8B06 mov eax, dword ptr [esi]
:0040201C FF909C000000 call dword ptr [eax+0000009C] ; Initialization call
```

可以搜索特征：`+0000009C]`

处理方法和上面一样。不说了。

### 菜单限制

对于菜单限制，一般处理 `enabling/disabling` 属性，程序中如下：

```vb
mnuRegister.Enabled = False
```

编译后：

```assembly
:004020A8 57 push edi ; value of edi = 00 which in VB = False
:004020A9 56 push esi
:004020AA 8B06 mov eax, dword ptr [esi]
:004020AC FF5074 call [eax+74] ; Initialization call
```

可以搜索特征：`+74]`

### Text 文本框的数据

对于 `text` 文本框的数据，也可以有相应处理，程序中如下：

```vb
Text1.Text = "Insert blah blah here"
```

编译后如下：

```assembly
:00401C35 6890174000 push 00401790 ; Push the address of the above string to the stack
:00401C3A 56 push esi
:00401C3B 8B06 mov eax, dword ptr [esi]
:00401C3D FF90A4000000 call dword ptr [eax+000000A4] ; Initialization call
```

其中 `push 00401790` 是文本框中数据的地址，如果想修改文本框中的数据，可以修改这个地址的数据，或者直接修改这个地址以指向你的字符串。如果不乐意，当然也可以把这个函数给 `nop` 掉。

### Caption 属性

对于很多控件中的 `caption` 属性，我们都可以进行修改。比如窗口标题栏上的数据，或者 `label` 标签里面的内容，都可以进行处理。程序中一般实现方法是：

```vb
mainForm.Caption = "Blah Blah v1.0 - Unregistered"
```

编译后：

```assembly
; Possible StringData Ref from Code Obj -> "Blah Blah v1.0 - Unregistered"
:004020CC 68FC194000 push 004019FC ; Location of the above string
:004020D1 50 push eax
:004020D2 898564FFFFFF mov dword ptr [ebp+FFFFFF64], eax
:004020D8 8B10 mov edx, dword ptr [eax]
:004020DA FF5254 call [edx+54] ; Initialization Call
```

可以搜索特征：`+54]`

这里请注意，你可以通过搜索字符串 `"Blah Blah v1.0 - Unregistered"` 来确定这个地方，但是很多时候这些字符串动态导入，无法被搜索到，因此比较保险的就是搜索所有的 `call [someregister+54]`，这样可以处理所有的 `caption`。定位以后，你可以通过上面介绍的方法进行处理。

## 对象属性内存分析

上面介绍了很多属性的特征。最后我们来讨论一下一个对象的所有属性，其实别的对象的属性也可以通过下面方法来确定。

看一下下面的内存内容，我们可以计算出其代表的对象。

```memory
properties.
.004011D0: 00 0D 01 00-00 3E 00 00-00 00 05 00-46 6F 72 6D  -- > - Form
.004011E0: 31 00 0D 01-05 00 46 6F-72 6D 31 00-19 01 00 42 1 --- Form1 -- B
.004011F0: 00 23 FF FF-FF FF 24 05-00 46 6F 72-6D 31 00 35  # $- Form1 5
.00401200: 3C 00 00 00-59 01 00 00-48 12 00 00-7B 0C 00 00  < Y- H- {-
.00401210: 46 03 FF 01-28 00 00 00-05 07 00 4F-70 74 69 6F  F- -( -- Optio
.00401220: 6E 31 00 06-01 07 00 4F-70 74 69 6F-6E 31 00 05  n1 --- Option1 -
.00401230: D0 02 08 07-57 03 C3 00-12 04 00 FF-03 26 00 00  ----W-- -- -&
.00401240: 00 04 06 00-43 68 65 63-6B 31 00 05-01 06 00 43  -- Check1 --- C
.00401250: 68 65 63 6B-31 00 05 D0-02 28 05 BF-04 FF 00 12  heck1 ---(-
.00401260: 03 00 FF 03-24 00 00 00-02 05 00 54-65 78 74 31  - -$ -- Text1
.00401270: 00 02 04 38-04 D0 02 F7-08 77 01 0B-05 00 54 65  --8-----w--- Te
.00401280: 78 74 31 00-12 01 00 FF-03 2C 00 00-00 01 08 00  xt1 -- -, --
.00401290: 43 6F 6D 6D-61 6E 64 31-00 04 01 08-00 43 6F 6D  Command1 --- Com
.004012A0: 6D 61 6E 64-31 00 04 E0-01 60 09 BF-04 77 01 08  mand1 ---?--w--
.004012B0: 00 11 00 00-FF 03 26 00-00 00 03 06-00 4C 61 62  - -& -- Lab
.004012C0: 65 6C 31 00-01 01 06 00-4C 61 62 65-6C 31 00 05  el1 --- Label1 -
.004012D0: B0 04 F0 00-27 06 77 01-12 02 00 FF-02 04 00 00  --- ?w--- --
.004012E0: 50 00 00 00-AB 51 84 9E-E1 4E 9C 4A-90 DB 82 61  P Q-NJ-a
.004012F0: 67 D3 DD F9-00 00 00 00-00 00 00 00-00 00 00 00  g---
```

对象名称 `object = Command1`

下面是各种属性：

```vb
Object.Caption = Command1
Object.Left = 480
Object.Top = 2400
Object.Width = 1215
Object.Height = 375
Object.Enabled = False
```

下面是 `VB` 中各种属性列表：

- `00` Picturebox
- `01` Label
- `02` TextBox
- `03` Frame
- `04` Command Button
- `05` CheckBox
- `06` Option Button
- `07` ComboBox
- `08` ListBox
- `0B` Timer
- `0D` Form

如果你想修改相应的属性，只需要修改一下内存里面的数值就可以了。

## VB CrackMe 示例

最后附上一个 `VB CrakeMe`。理论上根据上面的文章，就可以搞定这个东西。我搞了 2 个，别的没搞。

**附件**: `VBCrakeMe.rar`

感谢 `NBW` 提供的资料及 `CrackMe`

原文请看上面的 [ VB 快速逆向法 ] 。。。这里只是重复操作了一下。。。

### 1. 菜单可用

```assembly
::00402A6F::  6A 00                     PUSH 0                 ;修改为 FF
::00402A71::  57                        PUSH EDI
::00402A72::  8B0F                      MOV ECX,[EDI]
::00402A74::  FF51 74                  CALL [ECX+74]
```

修改后：

```assembly
::00402A6F::  6A FF                    PUSH -1
::00402A71::  57                        PUSH EDI
::00402A72::  8B0F                      MOV ECX, DWORD PTR [EDI]
::00402A74::  FF51 74                  CALL NEAR DWORD PTR [ECX+74]
```

### 2. 去 NAG 窗 1

```assembly
::0040277E::  6A FF                    PUSH -1
::00402780::  56                        PUSH ESI
::00402781::  8B06                      MOV EAX,[ESI]
::00402783::  FF90 BC010000             CALL [EAX+1BC]
```

修改后：

```assembly
::0040277E::  6A 00                    PUSH 0
::00402780::  56                        PUSH ESI
::00402781::  8B06                      MOV EAX, DWORD PTR [ESI]
```



### 3. 去 NAG 窗 2

```asm
::00402783::   FF90 BC010000             CALL    NEAR DWORD PTR [EAX+1BC]         

::00402678::   68 24404000               PUSH 404024                             

::0040267D::   68 88154000               PUSH 401588      ->: \x01  

::00402682::   FF15 5C104000             CALL [40105C]    >>>: MSVBVM60.DLL:__vbaNew2  

::00402688::   83EC 10                   SUB  ESP,10      :BYJMP JmpBy:00402676,  

::0040268B::   B9 0A000000               MOV  ECX,A                              

::00402690::   8BDC                       MOV  EBX,ESP                            

::00402692::   894D DC                   MOV  [EBP-24],ECX                       

::00402695::   B8 0400280                MOV  EAX,80020004                       

::0040269A::   83EC 10                   SUB  ESP,10                             

::0040269D::   890B                       MOV  [EBX],ECX                          

::0040269F::   8B4D D0                   MOV  ECX,[EBP-30]                        

::004026A2::   8BD0                       MOV  EDX,EAX                            

::004026A4::   8B35 24404000             MOV  ESI,[404024]    ;   这里进行了赋值    

::004026AA::   894B 04                   MOV  [EBX+4],ECX                        

::004026AD::   8BCC                       MOV  ECX,ESP                            

::004026AF::   8B3E                       MOV  EDI,[ESI]                          

::004026B1::   56                         PUSH ESI       ;作为参数入栈              

::004026B2::   8943 08                   MOV  [EBX+8],EAX                        

::004026B5::   8B45 D8                   MOV  EAX,[EBP-28]                        

::004026B8::   8943 0C                   MOV  [EBX+C],EAX                        

::004026BB::   8B45 DC                   MOV  EAX,[EBP-24]                        

::004026BE::   8901                       MOV  [ECX],EAX                          

::004026C0::   8B45 E0                   MOV  EAX,[EBP-20]                        

::004026C3::   8941 04                   MOV  [ECX+4],EAX                        

::004026C6::   8951 08                   MOV  [ECX+8],EDX                        

::004026C9::   8B55 E8                   MOV  EDX,[EBP-18]                        

::004026CC::   8951 0C                   MOV  [ECX+C],EDX                        

::004026CF::   FF97 B0020000             CALL [EDI+2B0]                          

DS:[00404024]=0014F6F0 

ESI=0012F604 

修改后：

::004026B1::   90                         NOP        ; 参数 NOP 掉 

::004026CF::   83C4 24                   ADD      ESP, 24      ; CALL 同样也 NOP 掉，平衡堆栈          

::004026D2::   EB 01                     JMP      SHORT 004026D5   

;这句也不能少，少了会出现 运行时错误' -1476327224（a80108c8)': Automation 错误 

::004026D4::   90                         NOP                               

```

### 4. 去掉文本框中的内容：

```asm
::00402B61::   68 F41C4000               PUSH 401CF4      ->: 籗塩購*NFh 虘剉匭筟 

::00402B66::   57                         PUSH EDI                            

::00402B67::   8B0F                       MOV  ECX,[EDI]                        

::00402B69::   FF91 A4000000             CALL [ECX+A4]                        

改变后：

::00402B61::   68 F41C4000               PUSH 指向其他字符的地址即可 

::00402B66::   57                         PUSH EDI                            

::00402B67::   8B0F                       MOV  ECX,[EDI]                        

::00402B69::   FF91 A4000000             CALL [ECX+A4]                        

```

### 5. 按钮可用：

```asm
::00402AA7::   6A 00                      PUSH 0              

::00402AA9::   57                         PUSH EDI        

::00402AAA::   8B0F                       MOV  ECX,[EDI] 

::00402AAC::   FF91 8C000000             CALL [ECX+8C]          

修改后：

::00402AA7::   6A FF                      PUSH    -1             

::00402AA9::   57                         PUSH  EDI      

::00402AAA::   8B0F                       MOV      ECX,  DWORD PTR [EDI]         ::00402AAC::   FF91 8C000000             CALL    NEAR DWORD PTR [ECX+8C]   

```

### 6. 旁边有个 `Label`

```asm
::00402AE5::   6A 00                      PUSH 0                           

::00402AE7::   57                         PUSH EDI                           

::00402AE8::   8B0F                       MOV  ECX,[EDI]                        

::00402AEA::   FF91 9C000000             CALL [ECX+9C]                        

修改后：

::00402AE5::   6A FF                      PUSH -1                            

::00402AE7::   57                         PUSH EDI                            

::00402AE8::   8B0F                       MOV  ECX,[EDI]                        

::00402AEA::   FF91 9C000000             CALL [ECX+9C]                        

```

### 7. 去掉窗口标题

内存地址 `401272` 处存放窗口标题的数据，替换修改即可！

`4dd466d0e1e0cc123afbe504da2462ca`
