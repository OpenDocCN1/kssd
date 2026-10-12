
# 汇编代码转换为 C 代码的练习

> 原文：[`www.pediy.com/kssd/pediy12/130425.html`](https://www.pediy.com/kssd/pediy12/130425.html)

最近在学习汇编以及 Windows 安全编程，总感觉汇编代码看起来有些费劲。看《加密解密》之类的书籍感觉虽然看着是那么一回事，但是动手太少，没有太大进步。总结一句话：纸上得来终觉浅，方知此事要躬行。于是自己写了一个小程序，然后看看汇编代码，再对照汇编代码，一步步看看 C 代码的实现。感觉经过实际的分析，收获不少，于是写成小文章，和大家一起分享一下。有问题还望指正，谢谢。

因为文章涉及到排版问题，发帖比较麻烦，做成了 PDF，大家自己下载吧。

上传的附件：

| 图片 | 文件名 |
| :--- | :--- |
| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) | 汇编代码转换为 C 代码的练习.pdf |

### `__cdecl` 调用方式的 `add` 函数

```assembly
00401020   push        ebp
00401021   mov         ebp,esp
00401023   sub         esp,40h
00401026   push        ebx
00401027   push        esi
00401028   push        edi
00401029   lea         edi,[ebp-40h]
0040102C   mov         ecx,10h
00401031   mov         eax,0CCCCCCCCh
00401036   rep stos    dword ptr [edi]
00401038   mov         eax,dword ptr [ebp+8]
0040103B   add         eax,dword ptr [ebp+0Ch]
0040103E   pop         edi
0040103F   pop         esi
00401040   pop         ebx
00401041   mov         esp,ebp
00401043   pop         ebp
00401044   ret
```

### `__stdcall` 调用方式的 `add` 函数

```assembly
00401020   push        ebp
00401021   mov         ebp,esp
00401023   sub         esp,40h
00401026   push        ebx
00401027   push        esi
00401028   push        edi
00401029   lea         edi,[ebp-40h]
0040102C   mov         ecx,10h
00401031   mov         eax,0CCCCCCCCh
00401036   rep stos    dword ptr [edi]
00401038   mov         eax,dword ptr [ebp+8]
0040103B   add         eax,dword ptr [ebp+0Ch]
0040103E   pop         edi
0040103F   pop         esi
00401040   pop         ebx
00401041   mov         esp,ebp
00401043   pop         ebp
00401044   ret         8
```

这里比对一下，除了最后一条指令不一样，其它都一样：

- `__cdecl` 栈外平衡，所以直接执行 `ret`
- `__stdcall` 栈内平衡，所以执行 `ret 8`

`ret` 指令相当于：

```assembly
pop eip
```

`ret 8` 指令相当于：

```assembly
pop eip
add esp, 8
```

因为调用的时候，先 `push` 参数，再 `push` 返回地址：

```assembly
push y
push x
push 返回地址
jmp add函数地址
```

对应的 `ret` 应该是完全相反的：

```assembly
pop eip ; pop 返回地址到 eip
pop x
pop y
```

后面 `pop x; pop y` 两条合成一条：`add esp, 8`

当然实际上 `pop eip` 指令是不对的，CPU 不支持，但是意思是这样的，`ret` 指令的作用是这样的。

---

**引用：** 最初由 `loongboy` 发布

`ret` 立即数是在返回时，平衡堆栈，堆栈向下增长。所以应该是 `add esp, 立即数` 是吧？另外应该是先 `pop eip`，再对 `esp` 进行操作的吧？我也不是很确定，因为我就是闲着没事看过一遍 8086 汇编，没实际写过。

哦，不好意思，写错了，的确是 `add esp, xxx`，我已经修改附件了。

栈是向下增长，`push xxx` 相当于 `sub esp, n`，`pop xxx` 相当于 `add esp, n`。

栈的向下增长，这里面的“向下”意思是低地址方向，从高地址到低地址。

一般说“向下增长”画图像上面这么画：

```text
高地址 FFFFFFFF
   ^
   |
   |
低地址 00000000
```

有的书上的图是倒着画的：

```text
低地址 00000000
   |
   |
  \/
高地址 FFFFFFFF
```

这个是还说“向下增长”容易让人迷惑，明明“向上”吗。

所以，以后我看书，只要说“向下”我就认为是低地址，这样就没有那么容易混淆了。
