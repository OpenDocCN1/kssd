
# 反调试技术总结

> 原文：[`www.pediy.com/kssd/pediy11/118341.html`](https://www.pediy.com/kssd/pediy11/118341.html)

最近学习反调试技术，总结了网络上的一些反调试技术，文章中的代码均通过调试，在 `OllyDbg` 中测试通过，同时谢谢看雪的《加密与解密》第三版。

测试软件：

*   `IDA` 最新 5.5，使用 5.4
*   `OllyDbg` 最新 2.0，结合 v1.10（汉化第二版）

## 一. 抗静态分析技术

静态分析是指从反汇编出来的程序清单上分析程序流程，反静态分析主要是指扰乱汇编代码可读性。

### 1. 花指令

在原程序中添加一些汇编指令，添加后不影响原程序的正常功能，但是能使反汇编工具反汇编错误。

#### 相关知识点

花指令是一堆汇编指令，对于程序而言，花指令是无用的汇编代码，花指令的有无不影响程序的运行。汇编语言是机器指令的符号化，是底层的编程语言，在汇编时，每一条汇编语句都会根据 `CPU` 特定的指令符号表将汇编指令翻译成二进制代码。

根据反汇编工具的反汇编算法，构造代码和数据，在指令流中插入很多“数据垃圾”，干扰反汇编软件的判断，从而使得它错误地确定指令的起始位置，杜绝了先把程序代码列出来再慢慢分析的做法。

在反汇编中，利用 `IDA Pro` 等静态分析软件可以将二进制程序反汇编成汇编代码。花指令是利用了反汇编时单纯根据机器指令字来决定反汇编结果来扰乱程序的静态分析。编写花指令的基本原则是保持堆栈的平衡。如 `push ebp` 把基址指针寄存器压入堆栈，`pop ebp` 是把基址指针起存期弹出堆栈，所以 `push ebp` 和 `pop ebp` 一定要成对的使用，这样堆栈是平衡的，指令的运行不会影响程序的运行。

#### 简单示例

**(1) 未加花程序 能正确反汇编**

原程序：

```c
void Function()
{
  Input++;
  Output++;
  Input+=Output;
  printf("函数结果:%d,%d",Input,Output);
}
```

反汇编代码：

`IDA`：

```asm
.text:00401096 rep stosd
.text:00401098 mov eax, Input
.text:0040109D add eax, 1
.text:004010A0 mov Input, eax
.text:004010A5 mov ecx, Output
.text:004010AB add ecx, 1
.text:004010AE mov Output, ecx
.text:004010B4 mov edx, Input
.text:004010BA add edx, Output
.text:004010C0 mov Input, edx
.text:004010C6 mov eax, Output
.text:004010CB push eax
.text:004010CC mov ecx, Input
.text:004010D2 push ecx
.text:004010D3 push offset s_PSDD ; "函数结果:%d,%d"
.text:004010D8 call printf
```

`OD` 中：

**(2) 简单加花后，在 `IDA` 中反汇编错误，在 `w32dasm` 中反汇编错误，但是在 `OD` 中反汇编正确**

加花后程序：

```c
void Function()
{
  _asm jz label
  _asm jnz label
  _asm __emit 0e8h //e8 是 call 指令

label:
  Input++;
  Output++;
  Input+=Output;
  printf("函数结果:%d,%d",Input,Output);
}
```

在 `OD` 中：

在 `IDA` 中：

```asm
.text:00401096 rep stosd
.text:00401098 jz short near ptr loc_40109C+1
.text:00401098
.text:0040109A jnz short near ptr loc_40109C+1
.text:0040109A
.text:0040109C
.text:0040109C loc_40109C: ; CODE XREF: .text:00401098 j
.text:0040109C ; .text:0040109A j
.text:0040109C call near ptr 4275D142h
.text:004010A1 add [ebx-3F5CFE40h], al
.text:004010A7 xor eax, 0D8B0042h
.text:004010AC mov esp, 83004235h
.text:004010B1 rol dword ptr [ecx], 89h
.text:004010B4 or eax, offset Output
.text:004010B9 mov edx, Input
.text:004010BF add edx, Output
.text:004010C5 mov Input, edx
.text:004010CB mov eax, Output
.text:004010D0 push eax
.text:004010D1 mov ecx, Input
.text:004010D7 push ecx
.text:004010D8 push offset s_PSDD ; "函数结果:%d,%d"
.text:004010DD call printf
```

**(3) 改进，加花后，`IDA` 与 `OD` 均反汇编错误**

改进后程序：

```c
void Function()
{
  _asm xor eax,eax
  _asm test eax,eax
  _asm jz label1
  _asm jnz label0

label0:
  _asm __emit 0e8h

label1:
  Input++;
  Output++;
  Input+=Output;
  printf("函数结果:%d,%d",Input,Output);
}
```

`OD` 中：

`IDA` 中：

```asm
.text:0040109C jz short near ptr loc_4010A0+1
.text:0040109C
.text:0040109E jnz short $+2
.text:004010A0
.text:004010A0 loc_4010A0: ; CODE XREF: .text:0040109C j
.text:004010A0 call near ptr 4275D146h
.text:004010A5 add [ebx-3F5CFE40h], al
.text:004010AB xor eax, 0D8B0042h
.text:004010B0 mov esp, 83004235h
.text:004010B5 rol dword ptr [ecx], 89h
.text:004010B8 or eax, offset Output
.text:004010BD mov edx, Input
.text:004010C3 add edx, Output
.text:004010C9 mov Input, edx
.text:004010CF mov eax, Output
.text:004010D4 push eax
.text:004010D5 mov ecx, Input
.text:004010DB push ecx
.text:004010DC push offset s_PSDD ; "函数结果:%d,%d"
.text:004010E1 call printf
```

#### 应用

**(1) 简单加花**

实现简单，但是原理比较简单，高手很容易就能去除，一般以消耗攻击者的耐性来达到目的。

**(2) 复杂加花**

类似于加壳：

1.  记录程序的原入口点，
2.  找到 `PE` 文件的空白区域，在空白区域内写入花指令（或者添加新节）
3.  把入口点地址改为新入口地址
4.  花指令执行完后跳转到原入口点地址

在程序执行时，程序将从新的入口地址执行，即花指令先被执行，然后再执行程序原来的入口地址功能。

增加了静态分析的难度，提高了代码的信息隐藏效果，该方法一般应用于病毒的免杀中。

### 2. 隐藏 API

逆向分析工作人员往往就是通过 `API` 在极短时间内获取了大量信息，从而使他们成功定位目标程序的关键代码段。所以隐藏对 `API` 的调用可以有效地提高程序的抗分析能力。

例如一个简单的程序：

```c
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
  MessageBox(NULL,"test","box",0);
  return 0;
}
```

反汇编代码如下：

```asm
.text:00401028 mov esi, esp
.text:0040102A push 0 ; uType
.text:0040102C push offset Caption ; "Test"
.text:00401031 push offset Text ; "Reverse Me"
.text:00401036 push 0 ; hWnd
.text:00401038 call ds:MessageBoxA(x,x,x,x)
```

目的是让反汇编的代码看不到 `ds: MessageBoxA(x,x,x,x)` 这样的提示。

**1. 最基本、最简单的方法：**

```c
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
  /****...****
  1. 定义字符串
  ****...****/
  TCHAR MsgBoxA[MAX_PATH]="MessageBoxA";

  /****...****
  2. 获取 MessageBoxA 的函数地址
  ****...****/

  HMODULE hMod=LoadLibrary("user32.dll");
  MYFUNC func=(MYFUNC)GetProcAddress(hMod,MsgBoxA);//获取 MessageBoxA 的函数地址。
  func(0,"Reverse Me","Test",0); //调用 MessageBoxA 函数。
  FreeLibrary(hMod);
  return 0;
}
```

`IDA` 中（V5.4）

```asm
.text:0040102E mov eax, dword ptr ds:aMessageboxa ; "MessageBoxA"
.text:00401033 mov dword ptr [ebp+ProcName], eax
.text:00401039 mov ecx, dword ptr ds:aMessageboxa+4
.text:0040103F mov [ebp+var_100], ecx
.text:00401045 mov edx, dword ptr ds:aMessageboxa+8
.text:0040104B mov [ebp+var_FC], edx
.text:00401051 mov ecx, 3Eh
.text:00401056 xor eax, eax
.text:00401058 lea edi, [ebp+var_F8]
.text:0040105E rep stosd
.text:00401060 mov esi, esp
.text:00401062 push offset LibFileName ; "user32.dll"
.text:00401067 call ds:__imp__LoadLibraryA@4 ; LoadLibraryA(x)
.text:0040106D cmp esi, esp
.text:0040106F call __chkesp
.text:00401074 mov [ebp+hLibModule], eax
.text:0040107A mov esi, esp
.text:0040107C lea eax, [ebp+ProcName]
.text:00401082 push eax ; lpProcName
```
