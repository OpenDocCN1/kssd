
# 俄罗斯方块

> 原文：[`www.pediy.com/kssd/pediy03/forum921.htm`](https://www.pediy.com/kssd/pediy03/forum921.htm)

巨好的俄罗斯方块时间限制破解

软件下载地址：`URL http://www.downme.com/download/10655rotoblox.zip`

该软件有 14 天的限制，而且被加了壳，可是用 `fi` 查不到是什么壳！呵呵...还好用 `caspr1012` 可以脱掉，可是脱掉后却变成了 -1 天了而且无法运行了！呵呵..挺有趣的。破解方法也很有意思！请看：

用 `ice` 或是 `trw` 载入脱壳后的文件，`load` 它。

一直按 `F10` 找它的出错框。

```
:00408161 FF15D8314100                 call dword ptr [004131D8]    **这里提示出错框
:00408167 391DD85B4100                 cmp dword ptr [00415BD8], ebx **比较
:0040816D 7F0B                         jg 0040817A                **还有剩余天数的话就正常运行注意跳转地址是 0040817A
:0040816F 5F                         pop edi
:00408170 5E                         pop esi
:00408171 33C0                       xor eax, eax
:00408173 5B                         pop ebx
:00408174 8BE5                       mov esp, ebp
:00408176 5D                         pop ebp
:00408177 C21000                     ret 0010
```

重新 `load` 它，来到它的上面：

```
:00408148 85C0                       test eax, eax                **测试提示框
:0040814A 7508                       je 00408154                  **这里要跳的，用 r fl z（爆破时改为 jne）
:0040814C 381D31984100               cmp byte ptr [00419831], bl
:00408152 7426                       jne 0040817A                 **注意跳转地址是 0040817A。当然要跳了。用 r fl z（爆破时改为 je）
```

基本上搞定，可是有意识的事来了。在你选择 `play` 玩时，游戏框里出现 `unregsitered version` 字样。这可如何是好？反编译它也找不到。用 `winhex` 试试吧！查找 `unregsitered version`，果真找到，找到后又怎么办。干脆全改成 `00` 吧？呵呵....不行哦！不信？你试试！那就全改成 `20` 吧！呵呵....`play` 试试。搞定！收工！ `4dd466d0e1e0cc123afbe504da2462ca`
