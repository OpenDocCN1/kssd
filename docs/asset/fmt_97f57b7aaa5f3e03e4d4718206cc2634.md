
# Hook 任意函数，任意位置【共享资源】source

> 原文：[`www.pediy.com/kssd/pediy11/110433.html`](https://www.pediy.com/kssd/pediy11/110433.html)

看到国外某网站提供的一个 hook 库，蛮好用的，翻译下给大家共享

## Hook This !

Library for coders

你可能经常会打开 OD，将一些地址改成 `jmp xxxx` 到你的 dll 然后过一会再恢复他们

那么，我发的这个东西会将你做的这些全部做掉，你可以 hook 任意位置的任意函数，并且不用一遍又一遍的检查堆栈是否平衡，这意味着什么？看下面的例子

### 代码

```asm
00401087   . 68 B80B0000        PUSH 0BB8                     ; |ControlID = BB8 (3000.)
0040108C   . FF75 08            PUSH DWORD PTR SS:[EBP+8]     ; |hWnd
0040108F   . E8 6A010000        CALL <JMP.&user32.SetDlgItemTextA>
```

这是 X 项目的一段汇编，在我没 hook 之前看起来是这样的

### 代码

```asm
0040106B   . E8 9D41C00F        CALL IndigoGS.1000520D        ; |Message
00401070   . E8 83010000        CALL <JMP.&user32.SendMessageA>
```

调用它提供的函数之后，代码被改写成上面这样了。接下来只需要再调用一次我库里的函数，就可以让代码看起来和原来一样

### 代码

```asm
00401087   . 68 B80B0000        PUSH 0BB8                     ; |ControlID = BB8 (3000.)
0040108C   . FF75 08            PUSH DWORD PTR SS:[EBP+8]     ; |hWnd
0040108F   . E8 6A010000        CALL <JMP.&user32.SetDlgItemTextA>
```

这个 lib 支持 `masm32` 和 `C++` 编译器并且是开源的

### 如何使用

现在，你只需要调用我库文件的函数在你的 exe 或者 dll 中，像这样

#### 代码

```masm
MASM32:
invoke HookThis, 0040106Bh, Offset MyProcedure, 1

C++:
HookThis(0040106B, MyProcedure, 1)

Resumed:

HookThis(HookOffset, MyProcedure, ID)
```

第一个参数，我们要 hook 的地址，第二个参数，我们代码中的回调地址，第三个参数是我们当前做的第几个 hook；

接下来，你可以做任何事情在你的代码中，但是要记得在你退出或者 ret 之前，要这样调用

#### 代码

```masm
MASM32:
invoke UnHookThis, 0040106B, 1

C++:
UnHookThis(0040106B, 1)

Resumed:
UnHookThis(HookedOffset, ID)
```

在这之后，目标程序会像没有发生过任何事情一样，继续运行；

### 软件限制

唯一的限制是，你只能 hook 100 个函数，当然我会在接下来的 update 中增加，或者你自己 diy，因为是开源的。

### 例子

#### 代码

```masm
Invoke HookThis, 00401064h, Offset MyProcedure1, 1
Invoke HookThis, 00401074h, Offset MyProcedure2, 2
Invoke HookThis, 00401094h, Offset MyProcedure3, 3

MyProcedure1 Proc
 ;DO ALL YOU WANT HERE
 invok UnHookThis, 00401064, 1
MyProcedure1 Endp

MyProcedure2 Proc
 ;DO ALL YOU WANT HERE
 invok UnHookThis, 00401074, 2
MyProcedure2 Endp

MyProcedure3 Proc
 ;DO ALL YOU WANT HERE
 invok UnHookThis, 00401094, 3
MyProcedure3 Endp
```

接下来是另外一个例子，hook 你自己的 dll 只需要调用 `LoadLibrary`，忘记那烦人并浪费时间的 `GetProcAddress`

#### 代码

```asm
In hooked program:

0041000: invoke LoadLibrary, "MyDLL"
0041005: Other instructions

In our DLL:

DllEntry Proc hInst:HINSTANCE, reason:DWord, reserved1:DWord
 .if (reason == DLL_PROCESS_ATTACH)
      invoke HookThis, 0041005h, Offset InitAll, 1        
 .endif
 mov eax, TRUE
 ret
DllEntry EndP
```

请尽情享受，source code + Library + Include file in attachment

### 上传的附件

![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)
`HookThis!.rar`
