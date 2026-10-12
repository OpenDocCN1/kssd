
# 动态跟踪分析 VB 程序

> 原文：[`www.pediy.com/kssd/tutorial/chap7-2.htm`](https://www.pediy.com/kssd/tutorial/chap7-2.htm)

**说明：本文写的时间较早，如果用 OD 来跟踪，就不需要相关的设置了。**

一、`SOFTICE` 配制  
二、VB dll 常用函数  
　　1 、字符处理函数  
　　2 、警告窗口函数   
三、VB 字串格式   
四、`oleaut32.dll` 简介   
五、Visual Basic 程序比较方法[](#5)

## 一、SOFTICE 配制

虽然大部分 VB 程序仍调用 Win32 API 函数，但如想在 VB dll 运行库中设断的话，你就必须把 VB dll 运行库加入 `SOFTICE` 配制里去。具体你可参考第五课中的《 SOFTICE 安装及使用》。 

下例是在 win95/98 下你把相关的 VB DLL 运行库加入 `winice.dat` 配制文件里。 ( 部分 ) 

```
; 注意分号后是描述语言，不被执行。每次只装载需要的一种 VB dll ，最好不要同时装载两个以上，把要装载那种 VB dll 那一行的分号去掉，重新启动即可。另外，判断软件是何种 VB dll 有几种方法，最直接一种用 WDasm89 查看，还有一种方法是用十六进制工具打开程序观察。或用 SOFTICE 跟进去（用 bpx hmemcpy) ，稍转一圈，你观察一下调试窗口领空处会出现何种 VB DLL 即可。 

; EXP=c:\\windows\\system\\msvbvm60.dll   ; Visual Basic 6   
; EXP=c:\\windows\\system\\msvbvm50.dll   ; Visual Basic 5   
; EXP=c:\\windows\\system\\vb40032.dll     ; Visual Basic 4(32-bit)   
; EXP=c:\\windows\\system\\vb40016.dll      ;Visual Basic 4(16-bit) 较少见     
; EXP=c:\\windows\\system\\vbrun300.dll   ; Visual Basic 3 
```

这样配制好后，重新启动，就可设 VB 运行库中各函数的断点了。 

--------------------------------------------------------------------------------

## 二、VB dll 常用函数

### 1 、字符处理函数

`MultiByteToWideChar`, `rtcR8ValFromBstr`, `WideCharToMultiByte`, `__vbaStrCmp`, `__vbaStrComp`, `__vbaStrCopy`, `__vbaStrMove`, `__vbaVarTstNe` . 

**注意:这些函数前的下划线 `__` 是由两根短线 `_` 组成的，不要弄错。 如果你是 crack VB6 程序，你应在断点前增加 `msvbvm60!` 。 如： `bpx msvbvm60!__vbastrcomp` **

一些常用字符串函数：

| `Val()` | 转换字符串为数字 | `Rtrim$()` | 去除字符串左边空格 |
| :--- | :--- | :--- | :--- |
| `Str$()` | 转换数字为字符串 | `Trim$()` | 去除字符串两边空格 |
| `Left$()` | 按要求从左取字符。例如： `Left$(Theodolite, 4) = "Theo"` | `Asc()` | 转换字符为 ANSI 编码 . 例如： `Asc("A") = 65` 注:65 是 ASCII 码的十进制 |
| `Ltrim$()` | 去除字符串左边空格例如： `Ltrim$ (" Hello ") = "Hello "` | `Chr$()` | 转换 ANSI 编码为字符 . 例如 `Chr$(65) = "A"` |
| `Right$()` | 按要求从右取字符 | | |

几个常用函数简介：

#### 1、 `__vbavartstne`

如是在 VB5 中 CALL `MSVBVM50!__vbavartstne` 进去后在 0F04E351 你将看到：

```
push dword ptr [ebp+0c] ;Push address *真* serial 的地址  
push dword ptr [ebp+10] ;Push address 你输入的 serial 地址下命令：d ebp+0c  
```

你将在数据窗口看到 4 字节地址倒序排列，你再次 D 地址（己纠正顺序的）， 就可看到序列号了。

#### 2、 `__vbaR8Str`

在 VB5 中是如下情况：

```
Push ebp-20  
Call MSVBUM50.__vbaR8Str ;转换 string 到 Integer/Real  
fcomp qword ptr [00401028] ;数据比较在这行 fcomp qword ptr [00401028] Type: DL 00401028 将看到*real* serial #.DL 是显示 Long/real 型，SOFTICE 默认时是 DB 状态。  
```

#### 3、 `__vbastrcmp`

例:

```
:005BDC32 CALL [MSVBVM50!__VbaFreeVar]   
:005BDC38 MOV ESI,[EBP+08]  
:005BDC3B PUSH ESI  
:005BDC3E CALL [ECX+00000790] <-- 值返回到 EDX.  
:005BDC44 MOV EDX,[ESI+4C] <-- 输入的号码，但倒转  
:005BDC47 MOV EAX,[005E0078] <-- 正确号码  
:005BDC4C PUSH EDX  
:005BDC4D PUSH EAX <-- Push 参数给 __vbaStrCmp.  
:005BDC4E CALL [MSVBVM50!__vbaStrCmp] <-- Visual Basic 字符串比较  
:005BDC54 TEST EAX,EAX <-- 测试比较结果  
:005BDC56 JNZ 005BDC64 <-- 不相等跳走  
```

### 2 、警告窗口函数

`rtcBeep`, `rtcGetPresentDate` (`time` `API`), `rtcMsgBox`

--------------------------------------------------------------------------------

## 三、VB 字串格式

在大部分 VB 程序中，我们能用 `bpx Hmemcpy` 命令设断，但是你将发现自己不久进入 VBRUNxxx.DLL 运行库，很快陷入 Vb dll 中，在大多数情况下，你很难到达其 EXE 文件中的真正比较核心。你通常是依靠字符串的线索来跟踪程序，你们还应记得 VB （ VB4 以上） 程序储存和比较字符是用 wide character 格式（本质是中在各字符间填 0x00）。

`The` `MultiByteToWideChar` `( )` `function` `maps` `a` `character` `string` `to` `a` `wide-character` `(Unicode)` `string.` `The` `character` `string` `mapped` `by` `this` `function` `is` `not` `necessarily` `from` `a` `multibyte` `character` `set.`

```
int MultiByteToWideChar(   
UINT CodePage, // code page   
DWORD dwFlags, // character-type options   
LPCSTR lpMultiByteStr, // string to map   
int cbMultiByte, // number of bytes in string   
LPWSTR lpWideCharStr, // wide-character buffer   
int cchWideChar // size of buffer   
); 
```

如：

原来字符串 : `CRACKZ` (43h 52h 41h 43h 4Bh 5Ah). 

Wide 字符串格式 : C R A C K Z (43h 00h 52h 00h 41h 00h 43h 00h 4Bh 00h 5Ah). 

这时在 `SOFTICE` 下查看内存中的字符串时看到情况有可能是： C R A C K Z 

有些情况下应该用 `DL` （长实型）命令，才能看到正确数字序号。（ SOFTICE 默认时为 `DB` （字节型）） 

在多数情况下，在 VB 中设置正确断点是较困难的。断点设置好后，尝试输入序列号，运行后，应返回 VBRUNxxx.DLL 里，现在查找寄存器 (EAX & EBX) 中的值，那里放有你输入字符串长度，如果没发现什么，你应按 `ctrl+D` 再次返回到 VB dll 中另一处 ，继续查看，一直重复。 

一但当你在寄存器中发现字符串长度时，你应一步一步注意观察，如果你幸运的话，你会发现输入字符串躺在寄存器或其显示附近内存里。这时你在 SOFTICE 里的数据窗口中用 `ALT+` 光标键滚动查找，说不定会发现正确序列号懒洋洋躺在那附近。 

--------------------------------------------------------------------------------

## 四、oleaut32.dll 简介（作者：dr0)

破解 VB 程序时，对 `vbrun\*.dll`（VB4、VB3 版本）和 `msvbvm\*.dll`（VB5、VB6）强调得比较多，实际上 VB 程序的很多运算是在 `oleaut32.dll` 中完成的，这个 dll 提供了很多对 VB 中的 Variant 类型的变量进行操作的函数，主要是一系列 `VarXXX( )`，其中有几个是用来比较字符串和数值的，如下：

```
Addr:77A11AAE Ord: 176 (00B0h) Name: VarCmp   
Addr:77A0E5D1 Ord: 311 (0137h) Name: VarCyCmp   
Addr:77A0E5F8 Ord: 312 (0138h) Name: VarCyCmpR8   
Addr:77A129CD Ord: 314 (013Ah) Name: VarBstrCmp   
Addr:77A12958 Ord: 316 (013Ch) Name: VarR4CmpR8   
Addr:77A13697 Ord: 204 (00CCh) Name: VarDecCmp   
Addr:77A1298B Ord: 298 (012Ah) Name: VarDecCmpR8   
```

这里有个小程序 `RAMQuota`（`www.stepnet.com.au`），是用 VB6 写的，它使用上述函数中的 `VarBstrCmp( )`比较注册码。该函数的关键指令如下：

```
:77A12A03 8B7D0C    mov edi, dword ptr [ebp+0C]   
:77A12A06 8B7508    mov esi, dword ptr [ebp+08]   
:77A12A09 8B4D10    mov ecx, dword ptr [ebp+10]   
:77A12A0C 33C0      xor eax, eax   
:77A12A0E F3        repz   
:77A12A0F 66A7      cmpsw   
:77A12A11 7405      je 77A12A18   
:77A12A13 1BC0      sbb eax, eax   
:77A12A15 83D8FF    sbb eax, FFFFFFFF   
:77A12A18 85C0      test eax, eax   
:77A12A1A 7F45      jg 77A12A61   
:77A12A1C 7D16      jge 77A12A34   
:77A12A1E 33C0      xor eax, eax   
```

看来有必要仔细看一下 `oleaut32.dll`

## 五、Visual Basic 程序比较方法

在 Visual Basic 里有 8 种以上的方法检测正确的序列号 / 密码。我这里将谈谈这 8 种方法，它们都是较容易并用得较多。首先是最普通的 3 种： 

1) 串（string) 比较

在这比较方法里，正确密码串如： `"Correct` `Password"` 和你输入的密码串如： `"Entered` `Password"` 比较。 

串是由相邻的字符按顺序排列组成，一个串包括字母、数字、空格和标点符号。一个固定长度的串可储存 0 到 63K 字符。如是动态的串其储存字符个数可达 20 亿字符。   

VB 串比较代码一般格式： 

```
If "Correct Password" = "Entered Password" then        <-- 直接比较两个串字符 
GoTo Correct Message  
Else 
GoTo Wrong Message  
End if 
```

这是一种简单保护方案，如果程序者用这种函数保护，用 SOFTICE 很容易拦截。 

可用到的断点： 

i) `__vbastrcomp` or `__vbastrcmp` （下划线是两短线 `_` 组成）            <-- 串比较函数 

注意：如果你是 crack VB6 程序，你应在两断点前增加 `msvbvm60!` 。 

如： `bpx msvbvm60!__vbastrcomp` or `bpx msvbvm60!__vbastrcmp` 

ii) 搜索特殊比较代码（具体参考下一节） 

如： `56,57,8b,7c,24,10,8b,74,24,0c,8b,4c,24,14,33,c0,f3,66,a7` 

2) 变量（Variant）比较 

在这方法中，两个变量（变量数据类型）互相比较。变量数据类型是一种特殊数据类型，包括数字、字符串或日期数据及一些用户定义的类型。这种类型储存数字长度是 16 字节或字符 22 字节（加上串长度）。 

一个范例代码： 

```
Dim correct As Variant, entered As Variant    <-- 定义 "correct" and "entered" 作为变量 
correct = Correct Password                       <-- 设置 "correct" 放置 "Correct Password"  
entered = Text1.Text                               <-- 设置 "entered" 作为你输入密码    
If correct = entered Then                          <-- 用变量方法比较   
GoTo Correct Message  
Else 
GoTo Wrong Message  
End If 
```

在这方法里， 1 中的两个断点将不起作用，因为程序不再用 `__vbastrcomp` 等 ... 

有用断点 ： 

i) `__vbavartsteq`                                  <-- 测试变量是否相等 

注意：如果你是 crack VB6 程序，你应在两断点前增加 `msvbvm60!` 。 

例： `bpx msvbvm60!__vbavartsteq` 

3) 长整型（ long) 比较 

这也是一个普通的方法。两个变量（长整型）互相比较。 

long 型数据类型是 4 字节（ 32 位）整数，其范围 -2,147,483,648 到 2,147,483,647 。因此该种方法存在只能比较数字的局限性。 

一个此类型范例： 

```
Dim correct As Long, entered As Long        <-- 定义 "correct" 和 "entered" 作为长整型 
correct = 12345                              <-- 设置 "correct" 为正确密码 
entered = Text1.Text                        <-- 设置 "entered" 为输入密码 
If entered = correct Then                  <-- 用长整型方法比较   
GoTo Correct Message 
Else 
GoTo Wrong Message 
End If 
```

对这种类型没有专门的断点函数，因为其数据比较是在主程序里而不是在 VB dll 中。 

--------------------------------------------------------------------------------



另外还有其它的 5 种方法，如果下面的方法你听起来有点怪或不太可能，你可忽略它们，但记得 `VB` 保护方式不限于上述三种。

### 4) 单精度实数（`Single`）比较

### 5) 双精度（`Double`）比较

### 6) 整型（`Integer`）比较

### 7) 字节（`Byte`）比较

### 8) `CURRENCY` 型比较

上述 8 个比较方式是不是意味着我们每次能中断在其单个某一类型的比较程序里？回答是否定的。这是因为程序有可能同时采用两种方法来比较，如 `Currency` 和 `String`, `Variant` 和 `Long` 等...

## 一些可能用上的断点

### 1) 数据类型转换

i) 字符串（`String`）转换字节（`Byte`）或整型（`Integer`）: `__vbai2str`

ii) 字符串（`String`）转长型（`Long`）: `__vbai4str`

iii) 字符串（`String`）转换单精度型（`Single`）: `__vbar4str`

iv) 字符串（`String`）转换双精度型（`Double`）: `__vbar8str`

v) 字符串（`String`）转 `Currency` 型: `VarCyFromStr` (适合 `VB6.` 你的 `WINICE.DAT` 必须有 `OLEAUT32.DLL`)

vi) 整型（`Integer`）转字符串型（`String`）: `VarBstrFromI2` (适合 `VB6.` 你的 `WINICE.DAT` 必须 `OLEAUT32.DLL`)

### 2) 移动数据

i) 字符串（`String`）到内存: `__vbaStrCopy`

ii) 变量（`Variant`）到内存: `__vbaVarCopy` or `__vbaVarMove`

### 3) 运算符

i) 加法: `__vbavaradd` <-- 增加变量

ii) 减法: `__vbavarsub` <-- 减去变量

iii) 乘法: `__vbavarmul` <-- 乘以变量

iv) 除法: `__vbavaridiv` <-- Dividing `Variant` to get answer in `Integer`

v) `XOR`: `__vbavarxor` <-- `XOR`

### 4) 其它类

i) `__vbavarfornext` <-- Used in codes where there is a `For... Next...` Statement (Loop)

ii) `__vbafreestr`

iii) `__vbafreeobj`

iv) `__vbastrvarval` <-- Get the value from a specific location in a `String`

v) `multibytetowidechar`

vi) `rtcMsgBox`

vii) `__vbavarcat` <-- Joining 2 `Variants` together

viii) `__vbafreevar`

ix) `__vbaobjset`

=============================

看雪 2000/2/21 整理

`4dd466d0e1e0cc123afbe504da2462ca`
