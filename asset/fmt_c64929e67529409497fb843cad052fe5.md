
# Andrénalin.33

> 原文：[`www.pediy.com/kssd/pediy03/forum805.htm`](https://www.pediy.com/kssd/pediy03/forum805.htm)

今天有空，就研究研究 `peterchen` 给我的 `crackme`：`Andrénalin.33`

程序也很有趣，`key` 形式的。看来是固定的，但是好像也不是很容易，一定不是明文的！要有心理准备哦！

好了我们开始吧！

输入 `key`：`abc`

下断 `bpx hmemcpy`

程序被断：15 次 `F12` 将要出错，所以 14 下就够了

到这儿：

```
:00401000 07                 pop es
:00401001 0008               add byte ptr [eax], cl
:00401003 00DC               add ah, bl
:00401005 214000             and dword ptr [eax+00], eax
:00401008 17224000           DWORD 00402217
:0040100C E3214000           DWORD 004021E3
```

按 `F10` 继续

```
:00401FAA FF1508414000       Call dword ptr [00404108] *这里取到你的码
:00401FB0 66050A00           add ax, 000A
:00401FB4 0F80B0020000       jo 0040226A
:00401FBA 0FBFD0             movsx edx, ax
:00401FBD 52                 push edx
```

按 `F10` 让它算吧！

`* Possible StringData Ref from Code Obj -> "kkXy^rO|\*yXo\*m\\kMuOn\*+"`

```
:00402036 C7855CFFFFFF8C1A4000    mov dword ptr [ebp+FFFFFF5C], 00401A8C
:00402040 C78554FFFFFF08800000    mov dword ptr [ebp+FFFFFF54], 00008008
```

上面的 `00401a8c` 里面放的是注册码，只不过是密文的！你只要把它的十六进制减去 `A` 就是真正的注册码了！

呵呵....也挺简单的！

注册码是 `aNThEr oNe cRaCkEd !`

你试试..... `4dd466d0e1e0cc123afbe504da2462ca`
