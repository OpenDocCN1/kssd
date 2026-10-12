
# 只用 WriteProcessMemory 进行远程注入执行

> 原文：[`www.pediy.com/kssd/pediy12/129222.html`](https://www.pediy.com/kssd/pediy12/129222.html)

今天突发奇想，不用`CreateRemoteThread`，只用`WriteProcessMemory`进行远程注入并执行！

原理是修改进程原始代码，但不是什么位置的代码都能利用，因为你不知道这个原始代码是否会被进程调用，所以我们要找一个循环的代码，这就不难了，窗口消息回调就是循环的。拿`explorer.exe`为例子：

用`OD`附加`explorer`进程，然后点`W`按钮显示窗口，看到许多窗口，应该选哪个呢？有不少回调地址都是`DLL`在处理，我们要选一个进程自己处理的，看下图红框框起来的那个！

![img/a565bdb818b1949573a783d91d516d99.jpg](img/a565bdb818b1949573a783d91d516d99.jpg)

然后点跟随`classproc`

代码：

```
01001B1D     8BFF          mov     edi, edi
01001B1F    .  55         push    ebp
01001B20    .  8BEC       mov     ebp, esp
01001B22    .  53         push    ebx
01001B23    .  56         push    esi
01001B24    .  57         push    edi
01001B25    .  8B7D 0C    mov     edi, dword ptr [ebp+C]
01001B28    .  BB 81000000 mov    ebx, 81
01001B2D    .  3BFB       cmp     edi, ebx
01001B2F    .  0F84 3A1F0100 je   01013A6F
01001B35    .  8B5D 08    mov     ebx, dword ptr [ebp+8]
01001B38    .  53         push    ebx
01001B39    .  E8 AFFFFFF call    01001AE6
01001B3E    .  8BF0       mov     esi, eax
01001B40    .  85F6       test    esi, esi
01001B42    .  0F84 ED2C0200 je   01024835
01001B48    .  8B06       mov     eax, dword ptr [esi]
01001B4A    .  56         push    esi
01001B4B    .  FF10       call    dword ptr [eax]
01001B4D    .  FF75 14    push    dword ptr [ebp+14]
01001B50    .  8B06       mov     eax, dword ptr [esi]
01001B52    .  FF75 10    push    dword ptr [ebp+10]
01001B55    .  8BCE       mov     ecx, esi
01001B57    .  57         push    edi
01001B58    .  53         push    ebx
01001B59    .  FF50 08    call    dword ptr [eax+8]
01001B5C    .  81FF 82000000 cmp   edi, 82
01001B62    .  8945 14    mov     dword ptr [ebp+14], eax
01001B65    .  0F84 B72C0200 je   01024822
01001B6B    >  8B06       mov     eax, dword ptr [esi]
01001B6D    .  56         push    esi
01001B6E    .  FF50 04    call    dword ptr [eax+4]
01001B71    >  8B45 14    mov     eax, dword ptr [ebp+14]
01001B74    >  5F         pop     edi
01001B75    .  5E        pop     esi
01001B76    .  5B        pop     ebx
01001B77    .  5D        pop     ebp
01001B78    .  C2 1000   retn    10
```

原理类似`HOOK`，但只`HOOK`一次，执行了注入代码之后就马上让他复原了！

因为系统版本可能不一样，会有一点差异，所以在这里，我用特征码搜索的方式来定位要修改的部分（红色加亮部分），把它修改成：

```
mov eax, XXXXXX
call eax
```

`XXXXXX`是注入代码地址，原理非常简单。下面是源码，大家可以研究下，代码写得很糙，不好意思了！

![img/906f120cfe711ccbc8463fd6eb86eee9.jpg](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

代码是`MASM`写的，编译后把`.text`段设置成可写的。

**附件**

![img/67af4f1ef9c98996f43e89e749c7dcf5.jpg](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) 远程注入.rar
