
# SMYNES1.2e

> 原文：[`www.pediy.com/kssd/pediy03/forum846.htm`](https://www.pediy.com/kssd/pediy03/forum846.htm)

注册你的 `SMYNES1.2e`

软件名称：`SMYNES1.2e`

软件主页：`http://smy.billyjr.com/`

主要功能：出色的 nes 模拟器

主要目的：当然是注册它了(我选暴破，可以免 `cdkey` 运行)

破解难度：非常简单(适合初学者)

现在太晚了，我简单说一下，1.0-1.2 均为 `keyfile` 保护，1.2e 主文件用 `upx0.84` 压缩过。没注册的话除了一堆 `unregisered` 的提示外，在 `game` 中加上 2 个大 `demo,too` 不爽了,so crack it。

首先脱壳(由于改的地方比较多，所以还是脱出来的好，其实主要是想加上我的 id,haha...)，我选择用 `PROCDUMP`，一下子就搞定啦:),简单^^。

首次运行，晕~~~~，他告诉我过期了，我倒~~~~~~,-\_-,反汇编吧，找到这句 `The date is expired, please download` 双击它，来到这里：

```asm
:004133EB 8B4C2408             mov ecx, dword ptr [esp+08]
:004133EF 85C9                 test ecx, ecx
:004133F1 7430                 je 00413423                        ->跳下去才不过期:)
:004133F3 A128D35200           mov eax, dword ptr [0052D328]
:004133F8 6A00                 push 00000000
* Possible StringData Ref from Data Obj ->"Warning"
                                 |
:004133FA B9E7CF5000           mov ecx, 0050CFE7
* Possible StringData Ref from Data Obj ->"The date is expired, please download "
                                     ->"the newer version!!!"
                                 |
:004133FF BAADCF5000           mov edx, 0050CFAD
```

剩下的真的偷懒了，这个比较简单，自己跟跟就知道了，我只把要改的代码贴上好了:),要改 6 处吧，不明白的话改完了，再反汇编看看就知道了;-)

1. 去掉过期提示：

   `004133F1` `je` `00413423` -> `jmp` `413423`

2. 获得 `For non-commercial user` 权限

   `46c072` `cmp` `eax,04`  ->`b003` `mov` `al,03`

                                 ->`90`  `nop`

3. 强制进入 `keyfile` 验证

   `469f56` `jne` `469f8e` ->`jmp` `469f8e`

4. 去掉旧版 `keyfile` 验证

   `46a10c` `je` `46a110` ->`jmp` `46a110`

5. 重头戏，用 `zombieys` 大名注册，哈哈.....^^

   把 `469f56` 及后面的代码改成这样：

   ```asm
   469f58 c705203566007a6f6d62 mov dword ptr [663520],626d6f7a
          c7052435660069657973 mov dword ptr [663524],73796569
          c7052835660000000000 mov dword ptr [663528],00
          33c0                 xor eax,eax
          81c4cc050000         add esp,5cc
          5f                   pop edi
          5e                   pop esi
          5b                   pop ebx
          c3                   ret
   ```

6. 让它跳过去才能用我的 id 注册

   把 `46a191` 改成这样：

   ```asm
   46a191 e9c2fdffff jmp 469f58
   ```

完活喽:)，睡觉去

good night

![](img/82459ef98caac13e24fe71965a64e8dc.jpg)

———————————————————————————————>

                  .-"       "-.      Cracked by zombieys      >
                  /            \      My oicq is 1789655       >
                 |              |  My hp is zombieys.yeah.net>
                 |,  .-.  .-.  ,|  My hp is zombieys.126.com >
                   |)(\_\_/  \\\_\_)(|                            >
                   |/    /\     \\|      Thanks for your supports  >
       (@\_@)    (\_    ^^    \_)                               >
   \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_                     >
\_ )@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_zombieys\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_>

\* \* \*

注：这篇文章是不完全的破解,根本没有接触到最核心的部分.没有解除 5 分钟限制和在 `game` 中加上 2 个大 `demo` 的限制。不过 `JOJO` 己完全破解了该软件，只是答应作者不公开。

\* \* \*
