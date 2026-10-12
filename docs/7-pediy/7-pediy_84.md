
# 简单 SMC 破解

> 原文：[`www.pediy.com/kssd/pediy12/140865.html`](https://www.pediy.com/kssd/pediy12/140865.html)

【文章标题】: 简单 SMC 破解

【文章作者】: `Smoke`

【使用工具】: `OllyDbg`, `peid`

【操作平台】: `WinXp` `sp3`

【附件下载】: `http://dl.dbank.com/c0rwovw6x6`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

【详细过程】

`PEiD` 侦查为 `UPX` `0.89.6` - `1.02` / `1.05` - `1.24` -> `Markus` & `Laszlo`

我等新手看到 `UPX` 壳,就想到 `SMC` 有戏了,强壳不会啊 `o()o`

载入 `OllyDbg`

代码如下:

```
00ABDCC0  >  60  pushad //F8 单步
00ABDCC1  BE 00604800  mov esi, 00486000 //单步到这,发现寄存器 esp 变红色 [使用 esp 定律,F9 运行]
00ABDCC6  8DBE 00B0F7FF lea edi, dword ptr [esi+FFF7B000]
00ABDCCC  57 push edi
00ABDCCD  EB 0B jmp short 00ABDCDA
```

运行后来到此处,代码如下:

```
00ABDE5F  8D4424 80 lea eax, dword ptr [esp-80]
00ABDE63  6A 00 push 0
00ABDE65  39C4 cmp esp, eax
00ABDE67  ^ 75 FA jnz short 00ABDE63
00ABDE69  83EC 80 sub esp, -80
00ABDE6C  ^ E9 F8999DFF jmp 00497869 //F2 下断 F9 运行 o(∩_∩)o 跳向光明之巅了.OEP
```

来到 `00ABDE6C` 后,取消断点,删除硬件断点 `F8` 单步 来到 `oep`

来到此处,代码如下:

```
00497869  55 push ebp (记录下,一会要用到的.)
0049786A  8BEC mov ebp, esp
0049786C  6A FF push -1
0049786E  68 88DFA200 push 00A2DF88
00497873  68 64A14900 push 0049A164
00497878  64:A1 00000000 mov eax, dword ptr fs:[0]
0049787E  50 push eax
```

找到关键跳,代码如下:(记录下,一会要用到的.)

```
004255A4  /0F84 63000000 je 0042560D
```

`je` 改成 `jnz`,代码如下:

```
004255A4  /0F85 63000000 jnz 0042560D
```

修改完毕后,我们运行试试,看是否能登录进去.

经测试,任意输入帐号密码可以登录 看来关键处是找对的 `o(∩_∩)o`

接下来我们就来 `SMC` `o(∩_∩)o`

`SMC` 实现过程如下:

原来：壳--->程序

现在：壳--->`SMC`--->程序

为了不破坏壳的代码，都在壳代码执行完后执行 `SMC`。

`SMC` 中修改的是程序中注册机制的关键点。

重新载入 `OllyDbg`

代码如下:

```
00ABDCC0  >  60 pushad
00ABDCC1  BE 00604800 mov esi, 00486000
00ABDCC6  8DBE 00B0F7FF lea edi, dword ptr [esi+FFF7B000]
00ABDCCC  57 push edi
```

使用 `esp` 定律来到以下代码处:

```
00ABDE6C  ^\E9 F8999DFF jmp 00497869
00ABDE71  0000 add byte ptr [eax], al
00ABDE73  0000 add byte ptr [eax], al
00ABDE75  0000 add byte ptr [eax], al
```

选个空白的代码处,作为我们 `SMC` 的地方.

我选择此处吧,代码如下:

```
00ABDE79  0000 add byte ptr [eax], al
00ABDE7B  0000 add byte ptr [eax], al
00ABDE7D  0000 add byte ptr [eax], al
00ABDE7F  0000 add byte ptr [eax], al
00ABDE81  0000 add byte ptr [eax], al
00ABDE83  0000 add byte ptr [eax], al
00ABDE6C  ^\E9 F8999DFF jmp 00497869
```

改成

```
00ABDE6C  /EB 0B jmp short 00ABDE79
00ABDE6E  |90 nop
00ABDE6F  |90 nop
00ABDE70  |90 nop
```

然后在 `00ABDE79` 处写代码

```
MOV BYTE PTR DS:[4255A5],85
jmp 00497869
```

保存出来.

经测试,能够登录!并未出错. `(*^__^*)`

此菜鸟文章就到此结束.`o(∩_∩)o`

---

【版权声明】: 本文原创于 `Smoke`, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 10 月 01 日 13:46:44

上传的附件

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) || 菜鸟.txt |
| :--- | :--- |
