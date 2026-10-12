
# Any Where 5.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue302.htm`](https://www.pediy.com/kssd/pediy04/kanxue302.htm)

## any where 5.1
系统软件，可以非常方便的移动，拷贝，删除文件等等，可直接将文件或文件夹压缩成 `zip` 文件，且集成到了鼠标右键当中。我安装试用了一下，感觉确实方便极了，不过呢，没有注册时，有 30 天的使用限制。破它

运行 `any where` （什么！？？，开始菜单里没有关联？桌面上没有图标？怎么运行啊？我倒！作者本着方便实用的原则，就没有其它花里胡俏的界面了，在任意一个文件或文件夹上点右键，就可以看到它的身影了），然后点击 `register` 菜单，

输入用户名：`Turkey`

输入注册码：`987654321`

启动 `trw` －-> `bpx hmemcpy` －-> `g` ，回到程序，点击 `ok`，被 `trw` 拦截下来，小心的按 `f12` 和 `f10` 来到这里：

```asm
:600091BE  mov eax, dword ptr [esp+10] //将用户名送到 eax
:600091C2  mov ecx, dword ptr [eax-08] //将用户名长度送到 ecx
:600091C5  test ecx, ecx     //看用户名长度是否为 0
:600091C7  je 60009616       //用户名为空就跳走
:600091CD  mov ecx, dword ptr [esp+14] //将注册码送到 ecx
:600091D1  mov eax, dword ptr [ecx-08] //将注册码长度送到 eax
:600091D4  test eax, eax     //看注册码长度是否为 0
:600091D6  je 60009616       //注册码为空就跳走
:600091DC  push ecx
:600091DD  lea edx, dword ptr [esp+14]
:600091E1  mov ecx, esp
:600091E3  mov dword ptr [esp+28], esp
:600091E7  push edx
:600091E8  call 6002DA3F
:600091ED  lea eax, dword ptr [esp+28]
:600091F1  mov ecx, esi
:600091F3  push eax
:600091F4  call 600096A0     //此 call 过后，正确的注册码就已计算出来了，说明它的算法要么在这个 call 里面，要么就还包括了上面的一段代码，可惜我的汇编太差，无赖看了半天，也没有看出个究竟来，到是把头看大了几倍。看来我还是得静下心来，认真学习一下汇编了。
:600091F9  push eax
:600091FA  lea ecx, dword ptr [esp+1C]
:600091FE  mov [esp+4C], 07
:60009203  call 6002DE03
:60009208  lea ecx, dword ptr [esp+24]
:6000920C  mov byte ptr [esp+48], bl
:60009210  call 6002DCCA
:60009215  mov ecx, dword ptr [esp+18] //此时 D ECX 可看到正确的注册码
:60009219  mov edx, dword ptr [esp+14] //此时 D EDX 可看到自己输入的错误注册码
:6000921D  push ecx
:6000921E  push edx
:6000921F  call 60013702     //这里是关键对比 call
:60009224  add esp, 00000008
:60009227  test eax, eax     //检查注册码是否正确的标记位
:60009229  jne 600095EF      //不对就跳，那就 game over 了
```

要得到注册码很简单，我就不多说了，哪位有空，或者闲的、闷的发慌的话，就给我详细分析一下注册码的计算过程吧 :)

整理一下：

用户名：`Turkey`

注册码：`AW5-5578-13464-84-1617350`

收工，要睡觉了，:)

```text
cracker：Turkey/灭害灵
2001.12.15@02：46
```

今天上网一找，原来这东西早有注册机了，没关系，我们是学技术嘛！ `4dd466d0e1e0cc123afbe504da2462ca`
