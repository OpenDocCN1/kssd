
随便使用一个程序，拖入`OD`，我们使用`OD`来编写我们的注入汇编代码

```
PUSH EBP
MOV EBP,ESP
```

注释：

生成注入代码`ThreadProc`自己的栈帧，可以稍微记下硬编码`55 8BEC`

```
MOV ESI,DWORD PTR SS:[EBP+8]
```

注释：

`[EBP+8]`，`ThreadProc`函数的唯一一个参数`LPVOID lParam`，这个参数就是我们定义的线程参数的结构体的地址，将其`mov`给了`esi`，那么之后`ESI`和`ESI+4`就可以取到我们定义的结构体的两个成员（API的函数地址）。这个定义的结构体看后面具体代码

`[EBP+4]`是调用该函数时`call`压入的返回地址

```
PUSH 00006C6C "\0\0ll"
PUSH 642E3233 "d.23"
PUSH 72657375 "resu"
```

注释：

将`user32.dll`字符串压入栈中（仔细思考一下，结合小端序和栈的排列）

这算是一个必须掌握的一个简单技能

```
PUSH ESP
```

注释：

将`ESP`压入栈中，因为从`ESP`开始是储存的字符串，`ESP`就是字符串的起始地址，就相当于把字符串压入了栈中，作为下面调用`LoadLibraryA()`函数的参数

```
CALL DWORD PTR DS:[ESI]
```

注释：

调用`ESI`处储存的函数地址对应的函数，`LoadLibraryA()`，汇编语言调用完函数之后返回值一般都是储存到`eax`寄存器中

```
PUSH 41786F "\0Axo"
PUSH 42656761 "Bega"
PUSH 7373654D "sseM"
```

注释：

将`"MessageBoxA"`字符串压入栈中

```
PUSH ESP
PUSH EAX
CALL DWORD PTR DS:[ESI+4]
```

注释：

现在的`ESP`就是字符串`"MessageBoxA"`的起始地址，将其压入了栈中作为函数的第二个参数，`EAX`里面的值是调用`LoadLibraryA()`函数得到的模块的起始地址，`hMod`。

这里就是`GetProcAddress(hMod,"MessageBoxA")`;

接下来，我们就要调用`MessageBoxA`这个函数了，因为`GetProcAddress`调用完毕后`EAX`中储存的已经是函数`MessageBoxA`的地址了

`MessageBoxA(NULL,"295K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2N6X3g2J5M7$3g2U0L8%4u0W2i4K6u0W2j5$3!0E0", "ReverseCore", MB_OK);`

先压入从右到左的第一个参数`MB_OK`

```
PUSH 0
```

然后压入`"ReverseCore"`这个字符串，这里有一个小技巧，可以记住，就是利用`call`等价于`push + jmp`的特点

假设上一条指令所处：

```
002D002C 6A 00 PUSH 0
```

那么接下来，在下方`002D0033`处，即`call`指令之后开始，存放我们的字符串`"ReverseCore"`，存放的区域就是`002D0033-002D003F`之间

然后我们`call`的话，就直接`call 002D003F`

这样先将字符串`"ReverseCore"` `push`进入了栈中，然后有`jmp`跳转到了地址`002D003F`处

同理，现在我们要压入参数`"c46K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2N6X3g2J5M7$3g2U0L8%4u0W2i4K6u0W2j5$3!0E0"`

`002D003F-002D0043`之间存放指令`call 002D0058`

`002D0044-002D0057`之间存放我们的字符串`"bf2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2N6X3g2J5M7$3g2U0L8%4u0W2i4K6u0W2j5$3!0E0"`

这样既将下面的数据`push`进入了栈中，还跳转到了下方

然后压入第一个参数`NULL`

```
push 0
```

最后调用我们的`MessageBoxA`函数

```
CALL EAX
```

之后调用完函数之后，我们要将返回值设为`0`

使用指令`XOR EAX, EAX`便可以简单并快捷的将返回值设置为`0`

```
MOV ESP,EBP
POP EBP
RETN
```

注释：

删除栈帧及函数返回

所有的汇编指令

| 指令 | 地址 | 机器码 | 汇编 |
| --- | --- | --- | --- |
| 1 | `004010ED` | `55` | `PUSH EBP` |
| 2 | `004010EE` | `8BEC` | `MOV EBP,ESP` |
| 3 | `004010F0` | `8B75 08` | `MOV ESI,DWORD PTR SS:[EBP+8]` ; `ESI = pParam` |
| 4 | `004010F3` | `68 6C6C0000` | `PUSH 6C6C` |
| 5 | `004010F8` | `68 33322E64` | `PUSH 642E3233` |
| 6 | `004010FD` | `68 75736572` | `PUSH 72657375` |
| 7 | `00401102` | `54` | `PUSH ESP` ; `- "user32.dll"` |
| 8 | `00401103` | `FF16` | `CALL DWORD PTR DS:[ESI]` ; `LoadLibraryA("user32.dll")` |
| 9 | `00401105` | `68 6F784100` | `PUSH 41786F` |
| 10 | `0040110A` | `68 61676542` | `PUSH 42656761` |
| 11 | `0040110F` | `68 4D657373` | `PUSH 7373654D` |
| 12 | `00401114` | `54` | `PUSH ESP` ; `- "MessageBoxA"` |
| 13 | `00401115` | `50` | `PUSH EAX` ; `- hMod` |
| 14 | `00401116` | `FF56 04` | `CALL DWORD PTR DS:[ESI+4]` ; `GetProcAddress(hMod, "MessageBoxA")` |
| 15 | `00401119` | `6A 00` | `PUSH 0` ; `- MB_OK (0)` |
| 16 | `0040111B` | `E8 0C000000` | `CALL 0040112C` |
| 17 | `00401120` | `<ASCII>` | `"ReverseCore",0` |
| 18 | `0040112C` | `E8 14000000` | `CALL 00401145` |
| 19 | `00401131` | `<ASCII>` | `"ae5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2N6X3g2J5M7$3g2U0L8%4u0W2i4K6u0W2j5$3!0E0",0` |
| 20 | `00401145` | `6A 00` | `PUSH 0` ; `- hWnd (0)` |
| 21 | `00401147` | `FFD0` | `CALL EAX` ; `MessageBoxA(0, "b7cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0W2N6X3g2J5M7$3g2U0L8%4u0W2i4K6u0W2j5$3!0E0", "ReverseCore", 0)` |
| 22 | `00401149` | `33C0` | `XOR EAX,EAX` |
| 23 | `0040114B` | `8BE5` | `MOV ESP,EBP` |
| 24 | `0040114D` | `5D` | `POP EBP` |
| 25 | `0040114E` | `C3` | `RETN` |

编辑完了之后，右键，`Copy to executable-All modifications`

弹出消息框之后点击`Copy all`

然后弹出一个显示所有修改的窗口，我们鼠标右键`Save file`，即可保存出去

然后我们得到`shellcode`

| 代码 | 字节 |
| --- | --- |
| 1 | `BYTE g_InjectionCode[] = {` |
| 2 | `0x55, 0x8B, 0xEC, 0x8B, 0x75, 0x08, 0x68, 0x6C, 0x6C, 0x00,` |
| 3 | `0x00, 0x68, 0x33, 0x32, 0x2E, 0x64, 0x68, 0x75, 0x73, 0x65,` |
| 4 | `0x72, 0x54, 0xFF, 0x16, 0x68, 0x6F, 0x78, 0x41, 0x00, 0x68,` |
| 5 | `0x61, 0x67, 0x65, 0x42, 0x68, 0x4D, 0x65, 0x73, 0x73, 0x54,` |
| 6 | `0x50, 0xFF, 0x56, 0x04, 0x6A, 0x00, 0xE8, 0x0C, 0x00, 0x00,` |
| 7 | `0x00, 0x52, 0x65, 0x76, 0x65, 0x72, 0x73, 0x65, 0x43, 0x6F,` |
| 8 | `0x72, 0x65, 0x00, 0xE8, 0x14, 0x00, 0x00, 0x00, 0x77, 0x77,` |
| 9 | `0x77, 0x2E, 0x72, 0x65, 0x76, 0x65, 0x72, 0x73, 0x65, 0x63,` |
| 10 | `0x6F, 0x72, 0x65, 0x2E, 0x63, 0x6F, 0x6D, 0x00, 0x6A, 0x00,` |
| 11 | `0xFF, 0xD0, 0x33, 0xC0, 0x8B, 0xE5, 0x5D, 0xC3` |
| 12 | `};` |

## （二）注入的代码
