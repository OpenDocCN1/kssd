
# Beyond Compare 2.0.1

> 原文：`www.pediy.com/kssd/pediy05/pediy50629.htm`

上周六想解 `beyond_compare_2.0.1`，虽说是几乎无功而返，但也不是没有收获，我发现：

1. 用 `softice` 在 `4245279` 处将 `eax` 强行改为 `6`，软件就可以正常运行（要发现这一点，我想，只要知道什么是 `softice` 就行了）。

```
CODE:04245276           cmp    eax, 9         ; eax == 6, All OK!
CODE:04245279           ja     loc_42454DD    ; default
```

于是，就改成：

```
mov eax, 6
```

2. 之后，发现 `slock` 不会跟你善罢甘休，跳过 `42441f0` 处的 `patch` 检查函数也不行。

周日，承蒙 `lajiaolz`（辣椒子？:)) 给了一个可用的 `key`，嫌自己的版本太旧，就下了一个 `build 178`，糟了，这个 `key` 被回收了。再装回到 `build 176` 也不行了 :(

不过，总觉得离解放它只有一步之遥了。

今天接着干，看看它会不会打开 `bc2.exe` 检查，如果是的话，可不可以让它检查原版，而运行我们改过的版本呢？于是：

```
bpx createfilea do "d esp->4"
```

果然它这样干了。于是将没有改动的版本复制一个成 `bcx.exe`。

将 `trail.key` 复制成 `bcx.key`，并改动一下复制文件名的代码，成了！

```
CODE:04196A20 84 C9                test    cl, cl
CODE:04196A22 74 07                jz      short locret_4196A2B
CODE:04196A24 FE CD                dec     ch
CODE:04196A26 75 E8                jnz     short loc_4196A10
CODE:04196A28 48                   dec     eax
CODE:04196A29
CODE:04196A29 loc_4196A29:                ; CODE XREF: sub_41969D6+36.j
CODE:04196A29 88 28                mov     [eax], ch
CODE:04196A2B
CODE:04196A2B locret_4196A2B:             ; CODE XREF: sub_41969D6+42.j
CODE:04196A2B                           ; sub_41969D6+4C.j
CODE:04196A2B C3                       retn
```

改成：

```
CODE:04196A20 84 C9                test    cl, cl
CODE:04196A22 74 07                jnz     short locret_4196A10
CODE:04196A24 FE CD                mov     byte ptr [eax-6], 'x' ; 将 bc2.exe 改成 bcx.exe, 让这个蠢东西去检查原版!
CODE:04196A26 75 E8                retn
CODE:04196A28 48                   dec     eax
CODE:04196A29
CODE:04196A29 loc_4196A29:                ; CODE XREF: sub_41969D6+36.j
CODE:04196A29 88 28                mov     [eax], ch
CODE:04196A2B
CODE:04196A2B locret_4196A2B:             ; CODE XREF: sub_41969D6+42.j
CODE:04196A2B                           ; sub_41969D6+4C.j
CODE:04196A2B C3                       retn
```

这样一来就像被脱了壳一样，想怎么 `fuck` 就怎么 `fuck`！

## 已知问题

主程序没问题了，绝大部分使用已经够了，几个附加插件可能还会去检查 `key`，这将是我下一个目标。应该不会太难，我想。

觉得比较幸运，如果在上周六之前在 `slock` 的网站（`http://www.crypto-central.com/slock/slock_feature.html`）看到以下信息的话，我可能就被吓得不敢去解它了。

- **Extremely High Security**
  `SLockPK` uses Public Key cryptography to ensure that your software cannot be made registered without your knowledge.
- **SlockPK** can increase the number of registrations that you receive from your shareware by making it almost impossible use unregistered shareware for extended periods.

现在看到这段话就想笑，觉得 `slock` 是一个善于吹牛、比较愚蠢的东东 :) `4dd466d0e1e0cc123afbe504da2462ca`
