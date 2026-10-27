
# [转贴]VB函数的列表(newlaos翻译整理)

## 1) 数据类型转换

- a) `__vbaI2Str`：将一个字符串转为 8 位（1 个字节）的数值形式（范围在 0 至 255 之间）或 2 个字节的数值形式（范围在 -32,768 到 32,767 之间）。
- b) `__vbaI4Str`：将一个字符串转为长整型（4 个字节）的数值形式（范围从 -2,147,483,648 到 2,147,483,647）
- c) `__vbar4Str`：将一个字符串转为单精度单精度浮点型（4 个字节）的数值形式
- d) `__vbar8Str`：将一个字符串转为双精度单精度浮点型（8 个字节）的数值形式
- e) `VarCyFromStr`：（仅 VB6 库。要调试，则在 `WINICE.DAT` 里必须有 `OLEAUT32.DLL`）字符串到变比型数据类型
- f) `VarBstrFromI2`：（仅 VB6 库。要调试，则在 `WINICE.DAT` 里必须有 `OLEAUT32.DLL`）整型数据到字符串

## 2) 数据移动

- a) `__vbaStrCopy`：将一个字符串拷贝到内存，类似于 Windows API `HMEMCPY`
- b) `__vbaVarCopy`：将一个变量值串拷贝到内存
- c) `__vbaVarMove`：变量在内存中移动，或将一个变量值串拷贝到内存

## 3) 数学运算

- a) `__vbavaradd`：两个变量值相加
- b) `__vbavarsub`：第一个变量减去第二个变量
- c) `__vbavarmul`：两个变量值相乘
- d) `__vbavaridiv`：第一个变量除以第二个变量，得到一个整数商
- e) `__vbavarxor`：两个变量值做异或运算

## 4) 程序设计杂项

- a) `__vbavarfornext`：这是 VB 程序里的循环结构，`For... Next...`（Loop）
- b) `__vbafreestr`：释放出字符串所占的内存，也就是把内存某个位置的字符串给抹掉
- c) `__vbafreeobj`：释放出 VB 一个对象（一个窗口，一个对话框）所占的内存，也就是把内存某个位置的一个窗口，一个对话框抹掉
- d) `__vbastrvarval`：从字符串特点位置上获取其值
- e) `multibytetowidechar`：将数据转换为宽字符格式，VB 在处理数据之都要这样做，在 `TRW2000` 显示为 7.8.7.8.7.8.7.8
- f) `rtcMsgBox`：调用一个消息框，类似于 Windows 里的 `MessageBox`/`MessageBoxA`/`MessageBoxExA`，此之前一定有个 `PUSH` 命令将要在消息框中显示的数据压入椎栈
- g) `__vbavarcat`：将两个变量值相连，如果是两个字符串，就连在一起
- h) `__vbafreevar`：释放出变量所占的内存，也就是把内存某个位置的变量给抹掉
- i) `__vbaobjset`
- j) `__vbaLenBstr`：获得一个字符串的长度，注：VB 中一个汉字的长度也为 1
- k) `rtcInputBox`：显示一个 VB 标准的输入窗口，类似 Windows API `GetWindowText`/`GetWindowTextA`、`GetDlgItemText`/`GetDlgItemTextA`
- l) `__vbaNew`：调用显示一个对话框，类似 Windows API `DialogBox`
- m) `__vbaNew2`：调用显示一个对话框，类似 Windows API `DialogBoxParam`/`DialogBoxParamA`
- n) `rtcTrimBstr`：将字串左右两边的空格去掉

## 5) 比较函数

- a) `__vbastrcomp`：比较两个字符串，类似于 Windows API `lstrcmp`
- b) `__vbastrcmp`：比较两个字符串，类似于 Windows API `lstrcmp`
- c) `__vbavartsteq`：比较两个变量值是否相等
- d) `__vbaFpCmpCy`：- Compares Floating point to currency. sp; Compares Floating point to currency

## 6) 在动态跟踪，分析算法时，尤其要注意的函数

- `rtcMidCharVar`：从字符串中取相应字符，VB 中的 `MID` 函数，用法 `MID("字符串","开始的位置","取几个字符")`
- `rtcLeftCharVar`：从字符串左边取相应字符，VB 中的用法：`Left("字符串","从左边开始取几个字符")`
- `rtcRightCharVar`：从字符串右边取相应字符，VB 中的用法：`Right("字符串","从右边开始取几个字符")`
- `__vbaStrCat`：用字符串的操作，就是将两个字符串合起来，在 VB 中只有一个 `&` 或 `+`
- `__vbaStrCmp`：字符串比较，在 VB 中只有一个 `=` 或 `<>`
- `ASC()` 函数：取一个字符的 `ASC` 值，在反汇编时，还是有的 `movsx` 操作数

## 7) 在函数中的缩写

- `bool`：布尔型数据（`TRUE` 或 `FALSE`）
- `str`：字符串型数据 `STRING`
- `i2`：字节型数据或双字节整型数据 `BYTE` 或 `Integer`
- `ui2`：无符号双字节整型数据
- `i4`：长整型数据（4 字节）`Long`
- `r4`：单精度浮点型数据（4 字节）`Single`
- `r8`：双精度浮点型数据（8 字节）`Double`
- `cy`：（8 个字节）整型的数值形式 `Currency`
- `var`：变量 `Variant`
- `fp`：浮点数据类型 `Float Point`
- `cmp`：比较 `compare`
- `comp`：比较 `compare`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
