

# [原创]小菜也玩inline hook -------GetWindowText

编程的过程中，总会有很多小想法。在学校的时候偶有心得，也都放过去了，整天瞎忙活。暑假在家清闲多了，至少俺家那口子不会吵我了，嘿嘿，她的监视范围不会触及到看雪吧。本人小菜，文章涉及的知识都是练手用的，不是奔着实用性去的，所以，大牛不要批评，权当您从我的身影中看到您的过去吧。俺还在慢慢成长中。

好了，come to the point。本来是想熟练一下`ToAscii`这个函数的，但是自己用`DialogBoxParam`创建自己的对话框后，不知道怎么让`Edit`控件响应`wm_keydown`消息（win32 application），所以就放弃了从`wm_keydown`的`wparam`和`lparam`中获取字符（目的都是练手熟悉而已）。然后，思绪飘呀飘。想想还是练练`inline hook`吧，貌似好久没玩了。于是操刀，对`GetWindowText`下手。说干就干，打开`od`，随便附加一个`mfc`程序，在`od`的`command`中输入`dd GetWindowTextA`，定位到`GetWindowText`这个函数，在我的`xp sp3`中它是这样的：

```
77D3216B >   $  6A 0C         push 0xC
77D3216D   .  68 D021D377   push USER32.77D321D0
77D32172   .  E8 4964FEFF   call USER32.77D185C0
77D32177   .  8B7D 0C       mov edi,dword ptr ss:[ebp+0xC]
77D3217A   .  33DB          xor ebx,ebx
77D3217C   .  3BFB          cmp edi,ebx
77D3217E   .  0F84 318F0000 je USER32.77D3B0B5
77D32184   .  395D 10       cmp dword ptr ss:[ebp+0x10],ebx
77D32187   .  0F84 288F0000 je USER32.77D3B0B5
77D3218D   .  895D FC       mov dword ptr ss:[ebp-0x4],ebx
77D32190   .  881F          mov byte ptr ds:[edi],bl
77D32192   .  8B4D 08       mov ecx,dword ptr ss:[ebp+0x8]
77D32195   .  E8 4663FEFF   call USER32.77D184E0
77D3219A   .  8BF0          mov esi,eax
77D3219C   .  8975 E4       mov dword ptr ss:[ebp-0x1C],esi
77D3219F   .  3BF3          cmp esi,ebx
77D321A1   .  0F84 A3F20000 je USER32.77D4144A
77D321A7   .  56            push esi
77D321A8   .  E8 6A6CFFFF   call USER32.77D28E17
77D321AD   .  6A 01         push 0x1
77D321AF   .  57            push edi
77D321B0   .  FF75 10       push dword ptr ss:[ebp+0x10]
77D321B3   .  6A 0D         push 0xD
77D321B5   .  56            push esi
77D321B6   .  85C0          test eax,eax
77D321B8   .  0F85 ED8E0000 jnz USER32.77D3B0AB
77D321BE   .  E8 177BFFFF   call USER32.77D29CDA
77D321C3   >  834D FC FF    or dword ptr ss:[ebp-0x4],0xFFFFFFFF
77D321C7   >  E8 3464FEFF   call USER32.77D18600
77D321CC   .  C2 0C00       retn 0xC
```

端量了一眼，貌似很多不熟悉的函数。不过不碍事，观察开头，

```
77D3216D   .  68 D021D377   push USER32.77D321D0
```

这一句正好是5个字节的指令，那么就用它来实现跳转吧。这里要提一下的是，实现跳转的指令很多，有5个字节的`jmp`远跳（`0xe9 0xXX 0xXX 0xXX 0xXX`），6个字节的`push 0xABCDEFGH，ret`（`0x68 0xXX 0xXX 0xXX 0xXX，0xc3`），7个字节的`mov ebx，0xABCDEFGH，jmp [ebx]`（`0xbb 0xXX 0xXX 0xXX 0xXX 0xff 0x23`）和7个字节的`mov Ebx，0xABCDEFGH，call [ebx]`（`0xbb 0xXX 0xXX 0xXX 0xXX 0xff 0x13`），还有先近跳转后远跳转的，不再一一列举。

对于5个字节的跳转的实现，有两个关键点：一个是跳转距离的计算，另一个是代码的写入位置。对于前者，通常是 跳转相对距离 = 目的地址 - 源地址 - 5；在本例中，由于我们不是从`GetWindowText`开头来实现的，所以源地址应该是（`&GetWindowText + 2`）的地方。对于写入，我们首先要修改内存属性，`VirtualProtect`可以帮我们把源地址的属性修改为`PAGE_EXECUTE_READWRITE`。修改完内存属性，就要将我们的`0xE9 0xXX 0xXX 0xXX 0xXX`五字节指令写入目的地址了。我们可以定义一个结构体，通过`WriteProcessMemory`来写入，也可以用内联汇编。

```
#pragma pack(1)

typedef struct _STRJMP
{
   char E9;   // 远跳转指令
   long Addr; // 跳转相对距离
} strJmp, *PstrJmp;

#pragma pack()
```

然后`WriteProcessMemory`到目的地址`&GetWindowText + 2`就可以了。我习惯用内联汇编，这样就省去了结构体的定义：

```
__asm
{
    mov eax, dwGetWindowTextAddr //存放的是原GetWindowText的地址
    add eax, 2
    mov byte ptr [eax], 0xe9
    mov ebx, dwAddr
    mov dword ptr [eax+1], ebx
}
```

目的地址即为我们自己实现的裸函数`MyGetWindowText`的地址。当程序中调用`GetWindowText`后会跳转到我们自己的`MyGetWindowText`执行。

上边仅仅是`inline hook`的步骤。

接下来是重点了，我的`MyGetWindowText`仅仅是获得调用`GetWindowText`后的执行权。Msdn告诉我们：

```
int GetWindowText(
  HWND hWnd,       // handle to window or control with text
  LPTSTR lpString, // address of buffer for text
  int nMaxCount    // maximum number of characters to copy
);
```

是调用完毕将获得的数据保存在`lpString`中。所以，我们要让程序执行完完整的`GetWindowText`后重新获得执行权（这就是我写到这篇文章的目的）。

我的做法是在`MyGetWindowText`中执行以下代码：

```
__asm
{
    //***************************************************
    mov eax, dwGetWindowTextAddr  //这四句代码是恢复原来的代码
    add eax, 2                    //也就是我们写入0xE9XXXXXXXX前的代码
    mov byte ptr [eax], 0x68
    mov dword ptr [eax+1], 0x77d321d0
    //***************************************************
    add esp, 8                    //add esp，8这一句是调整堆栈，因为在跳转
    call dwGetWindowTextAddr      //指令0xE9XXXXXXXX之前已经两次压栈
    mov ebx, esp                  //分别是 call调用的压栈和 push 0xC，执行完
    add ebx, -8                   // add esp，8后就可以call 纯净的GetWindowText
    mov eax, [ebx]                //接下来是寻找结果了，也就是寻GetWindoText
    mov AddrBuf, eax              //的第二个参数的数据
    //***************************************************
}
```

操作的堆栈：

在调用完纯净的`GetWindowText`后，我们关心的`lpString`会被保存在`esp - 8`的位置，`esp`存放的是`eip`，`esp - 4`存放的是`MAX_PATH`，`esp - 8`则是我们想要的内容了。这时候还没有调用其他函数，堆栈`esp - 8`还没有被冲掉，赶紧取出它的内容。

最后要说的就是程序的执行流程问题。我们要让程序按照我们自个的流程执行。在`MySetWindowText`中，我们通过：

```
mov eax, dwGetWindowTextAddr
add eax, 2
mov byte ptr [eax], 0x68
mov dword ptr [eax+1], 0x77d321d0
```

恢复了`GetWindowText`，所以，最后我们要想法跳回调用`GetWindowText`的下一步地址。因此，我在`MyGetWindowText`开头保存了这个地址：

```
__asm
{
    mov eax, esp
    add eax, 4
    mov ebx, [eax]
    mov dwRet, ebx //dwRet 存放的就是在MyGetWindowText执行完毕后要跳到的地址
}
```

好了，我的程序结束了。貌似讲的不太清晰，我也就这个糟水平。再帮大家缕一缕，

A，`inline hook GetWindowText`

B，调用`GetWindowText`，跳转到`MyGetWindowText`

C，`MyGetWindowText`中，首先保存要返回的地址（这地址可不是通常hook语句的下一句呀，是`GetWindowText`调用完毕后执行的语句），然后恢复`MyGetWindowText`，然后调用纯净的`GetWindowText`，返回的窗体内容在`esp - 8`中，取出。跳转到返回地址

D，程序继续执行。。。。

至于功用，就是没有用处，自个跟自个玩（别邪恶呀），哈哈。貌似注入了调用`GetWindowText`的程序后可以获取`GetWindowText`得到的内容。也就一种思路。我是练手的。Ring0下`inline hook`也就注意一下堆栈，运行级别和页面的读写开关，熟悉ring3，ring 0 应该没问题的，至少我是那么样，呵呵。欧了。

源码很烂，附上了，大家将就瞅瞅。还是那句话，小菜在练手。。。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [bingle.zip](javascript:void(0);)

  （16.39kb，58次下载）
