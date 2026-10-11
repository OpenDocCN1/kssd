
# 从底层分析某软件 2011 版的底层钩子

> 原文：[`www.pediy.com/kssd/pediy12/130932.html`](https://www.pediy.com/kssd/pediy12/130932.html)

无聊拿出来分享一下，送给和我一样菜的朋友，如果有什么不正确的欢迎大家指出。

共同学习共同进步嘛，最近在研究 `WINDOWS` 的回调机制。又找不到什么好题材，干脆就分析一下 `QQ2011` 的底层键盘钩子啦，只为学习，希望大家不要拿来做什么违法的事。

首先说下 `WINDOWS` 的回调吧，其实回调有点像 `APC`。通俗点讲就是底层调用用户空间函数又返回底层。主要涉及到的函数就是 `KeUserModeCallback`，它的作用就是帮回调函数设计好用户空间堆栈。回调函数返回用户空间的时候让它指向用户空间的分派函数 `KiUserCallbackDispatcher`，再从 `KernelCallbackTable` 表中以 `ID` 找到对应的中继函数，再从底层传回来的结构中的 `PROC` 函数指针中调用我们的(用户空间)回调函数。在调用 `KeUserModeCallback` 的时候会把三个参数写到用户空间堆栈：`KernelCallbackTable` 的 `ID`，回调函数结构(里面包含了很多信息比如 `PROC` `HOOKID`)，包括底层键盘钩子用到的 `KBDLLHOOKSTRUCT` 结构(`lParam`)，还有个就是这个结构的长度。在调用完用户空间的函数以后再通过 `NtCallbackReturn` 返回底层。

顺便说下 `KernelCallbackTable` 在 `PEB+0X2C` 的位置 `XPSP3`。

下图就是 `KiUserCallbackDispatcher` 调用啦，也就是底层回调到用户层的第一步：

![](img/471c94a31325f52e51f8e8dc2cbb3c5b.jpg)

```asm
POP EDX
MOV EAX,DWORD PTR FS:[18]
MOV EAX,DWORD PTR DS:[EAX+30]
MOV EAX,DWORD PTR DS:[EAX+2C] //得到 KernelCallbackTable EDX 保存着 ID
CALL DWORD PTR DS:[EAX+EDX*4]
```

我们来加载 `QQ` 设置个条件断点看看寄存器和堆栈有什么内容，点击密码输入框按个 `1`，看下。

从图可以看到寄存器 `EDX=2D`(`ID`)：

![](img/884df39b8eee9ffe64b39984c693f721.jpg)

堆栈上：

![](img/7875033efb9894e1d7c71a8d55705ccf.jpg)

```text
0013FABC   0000002D  //ID
0013FAC0   0013FAC8 //回调结构指针
0013FAC4   00000024  //大小
0013FAC8   000D0000  //HOOKID
0013FACC   00000100  //CODE
0013FAD0   020DC191  //回调函数地址 用户空间
0013FAD8   00000031  //这个就是我们输入的 1
```

我们进一步跟进去看看。

下图是 `KernelCallbackTable` 表以 `ID` 调用的函数：

![](img/d6ece45b1f3fe7079fb5e3ea98bc62d6.jpg)

```asm
//CALL DWORD PTR DS:[EAX+EDX*4] 来到这里
77D58D54    8BFF            MOV EDI,EDI
77D58D56    55              PUSH EBP
77D58D57    8BEC            MOV EBP,ESP
77D58D59    83EC 0C         SUB ESP,0C
77D58D5C    8B45 08         MOV EAX,DWORD PTR SS:[EBP+8]
77D58D5F    FF70 08         PUSH DWORD PTR DS:[EAX+8]//回调函数地址
77D58D62    8D48 10         LEA ECX,DWORD PTR DS:[EAX+10]//结构地址
77D58D65    51              PUSH ECX
77D58D66    FF70 04         PUSH DWORD PTR DS:[EAX+4] //CODE
77D58D69    C745 F8 1400000>MOV DWORD PTR SS:[EBP-8],14
77D58D70    FF30            PUSH DWORD PTR DS:[EAX]  //HOOKID
77D58D72    894D FC         MOV DWORD PTR SS:[EBP-4],ECX
77D58D75    FF50 0C         CALL DWORD PTR DS:[EAX+C]//CALL 77D318D1
```

到这里就用到我们从底层传上来的结构里的内容啦，我们继续进 `CALL`。

```asm
//CALL DWORD PTR DS:[EAX+C]//CALL 77D318D1 来到这里省略一些钩子类型检测的代码

77D31912    8B75 08         MOV ESI,DWORD PTR SS:[EBP+8] //lParam
77D31915    53              PUSH EBX      //wParam
77D31916    FF75 0C         PUSH DWORD PTR SS:[EBP+C]
77D31919    0FB7FE          MOVZX EDI,SI
77D3191C    57              PUSH EDI //nCode
77D3191D    897D F8         MOV DWORD PTR SS:[EBP-8],EDI
77D31920    FF55 14         CALL DWORD PTR SS:[EBP+14]
```

这里一系列检测完钩子类型以后来到这个 `CALL`，这里就是：

```c
LRESULT CALLBACK LowLevelKeyboardProc(
  int nCode,     // hook code
  WPARAM wParam, // message identifier
  LPARAM lParam
```

这个时候就到了 `QQ` 的钩子回调函数啦：

```asm
020DC191    55              PUSH EBP
020DC192    8BEC            MOV EBP,ESP
020DC194    83EC 30         SUB ESP,30
020DC197    56              PUSH ESI
020DC198    57              PUSH EDI
020DC199    C745 FC 78BF0D0>MOV DWORD PTR SS:[EBP-4],20DBF78

/*注意这里这里有个基址很重要很多函数的定位和数据的定位都是它*/

020DC1A0    8D7D D0         LEA EDI,DWORD PTR SS:[EBP-30]
020DC1A3    B9 07000000     MOV ECX,7
020DC1A8    33C0            XOR EAX,EAX
020DC1AA    FC              CLD
020DC1AB    F3:AB           REP STOS DWORD PTR ES:[EDI]
020DC1AD    8B45 10         MOV EAX,DWORD PTR SS:[EBP+10]//获取 KBDLLHOOKSTRUCT
020DC1B0    8945 F8         MOV DWORD PTR SS:[EBP-8],EAX//保存
020DC1B3    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]//基地址定位判断
020DC1B6    8379 0C 00      CMP DWORD PTR DS:[ECX+C],0

020DC1BA    74 21           JE SHORT 020DC1DD
020DC1BC    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]
020DC1BF    FF52 54         CALL DWORD PTR DS:[EDX+54]//  GetForegroundWindow 调用判断窗口

HWND hwnd=GetForegroundWindow();

_asm{
    mov edx,0x20DBF78
    mov edx,[edx+0x1c]
    cmp  hwnd,edx
    jnz  020DC1DD
}

020DC1C2    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]//取基地址
020DC1C5    3B42 1C         CMP EAX,DWORD PTR DS:[EDX+1C]//+1C 位置存放着窗口句柄比较
020DC1C8    75 13           JNZ SHORT 020DC1DD
020DC1CA    8B4D F8         MOV ECX,DWORD PTR SS:[EBP-8]
020DC1CD    8B41 08         MOV EAX,DWORD PTR DS:[ECX+8]
020DC1D0    8945 CC         MOV DWORD PTR SS:[EBP-34],EAX
020DC1D3    B8 10000000     MOV EAX,10
020DC1D8    F7D0            NOT EAX
020DC1DA    2141 08         AND DWORD PTR DS:[ECX+8],EAX
020DC1DD    C745 F4 0000000>MOV DWORD PTR SS:[EBP-C],0
020DC1E4    E8 20000000     CALL 020DC209     //上面的跳转如果不成立就到这个 CALL
{020DC209    5F              POP EDI          //此时 EDI 等下返回地址 也就是-> 020DC1E9
020DC20A    8B4D F4         MOV ECX,DWORD PTR SS:[EBP-C]
020DC20D    833C8F 00       CMP DWORD PTR DS:[EDI+ECX*4],0
020DC211    74 1D           JE SHORT 020DC230
020DC213    8B55 F4         MOV EDX,DWORD PTR SS:[EBP-C]
020DC216    8B0497          MOV EAX,DWORD PTR DS:[EDI+EDX*4]//取 020DC1E9 的代码
020DC219    50              PUSH EAX
020DC21A    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]
020DC21D    FF51 50         CALL DWORD PTR DS:[ECX+50]

/* 调用 GetAsyncKeyState 判断被按下 餐宿 EAX=020DC1E9 的代码
  实际上 20DC1E9 是 020DC1E9  00000010
                  020DC1ED  00000011
                  020DC1F1  00000012
                  020DC1F5  00000013
                    020DC1F9  00000009
                    020DC1FD  0000005B
                    020DC201  0000005C
                  循环判断知道遇到 0 结束
*/

020DC220    0FBFD0          MOVSX EDX,AX
020DC223    81E2 00800000   AND EDX,8000
020DC229    75 05           JNZ SHORT 020DC230
020DC22B    FF45 F4         INC DWORD PTR SS:[EBP-C]
020DC22E    EB DA           JMP SHORT 020DC20A
020DC230    8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]//KBDLLHOOKSTRUCT
020DC233    8B02            MOV EAX,DWORD PTR DS:[EDX] //KBDLLHOOKSTRUCT->vkCode 也就是我们按下的 1=31
020DC235    C745 F0 0000000>MOV DWORD PTR SS:[EBP-10],0
020DC23C    E8 3C000000     CALL 020DC27D
```

这里来到下个 `CALL` 进入这个 `CALL` 再下面大家往下看。

```asm
}

020DC1E9    1000            ADC BYTE PTR DS:[EAX],AL
020DC1EB    0000            ADD BYTE PTR DS:[EAX],AL
020DC1ED    1100            ADC DWORD PTR DS:[EAX],EAX
020DC1EF    0000            ADD BYTE PTR DS:[EAX],AL
020DC1F1    1200            ADC AL,BYTE PTR DS:[EAX]
020DC1F3    0000            ADD BYTE PTR DS:[EAX],AL
020DC1F5    1300            ADC EAX,DWORD PTR DS:[EAX]
020DC1F7    0000            ADD BYTE PTR DS:[EAX],AL
020DC1F9    0900            OR DWORD PTR DS:[EAX],EAX
020DC1FB    0000            ADD BYTE PTR DS:[EAX],AL
020DC1FD    5B              POP EBX
020DC1FE    0000            ADD BYTE PTR DS:[EAX],AL
020DC200    005C00 00       ADD BYTE PTR DS:[EAX+EAX],BL
020DC204    0000            ADD BYTE PTR DS:[EAX],AL
020DC206    0000            ADD BYTE PTR DS:[EAX],AL
020DC208    005F 8B         ADD BYTE PTR DS:[EDI-75],BL
020DC20B    4D              DEC EBP
020DC20C    F4              HLT
020DC20D    833C8F 00       CMP DWORD PTR DS:[EDI+ECX*4],0
020DC211    74 1D           JE SHORT 020DC230
020DC213    8B55 F4         MOV EDX,DWORD PTR SS:[EBP-C]
020DC216    8B0497          MOV EAX,DWORD PTR DS:[EDI+EDX*4]
```
