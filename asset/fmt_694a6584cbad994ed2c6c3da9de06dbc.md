
## 汇编代码片段

```
0049F86C  0F8483000000             jz      0049F8F5    <------------改成 0F85---------------

0049F872  8BC3                     mov     eax, ebx

0049F874  E8F769FAFF               call    00446270

0049F879  8B1578874900             mov     edx, [$498778]

0049F87F  E8D836F6FF               call    00402F5C

0049F884  8BF0                     mov     esi, eax

0049F886  8D55FC                   lea     edx, [ebp-$04]

0049F889  8B8644030000             mov     eax, [esi+$0344]

0049F88F  E8248EF6FF               call    004086B8

0049F894  8B55FC                   mov     edx, [ebp-$04]

0049F897  8B83E0040000             mov     eax, [ebx+$04E0]

0049F89D  83C06C                   add     eax, +$6C

0049F8A0  E8F341F6FF               call    00403A98

0049F8A5  8B83E0040000             mov     eax, [ebx+$04E0]

0049F8AB  8B10                     mov     edx, [eax]

0049F8AD  FF523C                   call    dword ptr [edx+$3C]

0049F8B0  84C0                     test    al, al

0049F8B2  744A                     jz      0049F8FE

0049F8B4  8D55F8                   lea     edx, [ebp-$08]

0049F8B7  8B83E0040000             mov     eax, [ebx+$04E0]

0049F8BD  E84617FBFF               call    00451008

0049F8C2  8B55F8                   mov     edx, [ebp-$08]

0049F8C5  33C9                     xor     ecx, ecx

0049F8C7  8BC6                     mov     eax, esi

0049F8C9  E83A9AFFFF               call    00499308

0049F8CE  8D55F8                   lea     edx, [ebp-$08]

0049F8D1  8BB3E0040000             mov     esi, [ebx+$04E0]

0049F8D7  8BC6                     mov     eax, esi

0049F8D9  E82A17FBFF               call    00451008

0049F8DE  8B45F8                   mov     eax, [ebp-$08]

0049F8E1  8D55FC                   lea     edx, [ebp-$04]

0049F8E4  E89B8DF6FF               call    00408684

0049F8E9  8B55FC                   mov     edx, [ebp-$04]

0049F8EC  8BC6                     mov     eax, esi

0049F8EE  E87D17FBFF               call    00451070

0049F8F3  EB09                     jmp     0049F8FE

0049F8F5  33D2                     xor     edx, edx

0049F8F7  8BC3                     mov     eax, ebx

* Reference to : TMainForm.HelpPurchaseItemClick

|

0049F8F9  E8E2130000               call    004A0CE0

0049F8FE  33C0                     xor     eax, eax

0049F900  5A                       pop     edx

0049F901  59                       pop     ecx

0049F902  59                       pop     ecx

0049F903  648910                   mov     fs:[eax], edx

****** FINALLY

|

* Possible String Reference to: "^[YY]脥@"

|

0049F906  6823F94900               push    $0049F923

0049F90B  8D45F8                   lea     eax, [ebp-$08]

0049F90E  E83141F6FF               call    00403A44

0049F913  8D45FC                   lea     eax, [ebp-$04]

0049F916  E82941F6FF               call    00403A44

0049F91B  C3                       ret

0049F91C  E95F3BF6FF               jmp     00403480

0049F921  EBE8                     jmp     0049F90B

****** END

|

0049F923  5E                       pop     esi

0049F924  5B                       pop     ebx

0049F925  59                       pop     ecx

0049F926  59                       pop     ecx

0049F927  5D                       pop     ebp

0049F928  C3                       ret

}

end ;
```

## 分析与讨论

如此一来，并没有影响注册的对话框，不过，相信输入正确的注册号以后，估计那个 `OF85` 还需要改成 `0F84`。

BTW: 1212 兄，假如在 `49F8B2` 处改动成 `9090`，注册对话框似乎依旧会跳出，恐怕和下面一条 `49F8F3` 有关？

其实我在实际改的时候并不是如我上篇写的那样只改 `jz` 就行了！我只是为少打几个字才说了一下重点！其实你可以在下面这个 `call dword ptr [edx+83c]` 进去，看看到为什么 `AL` 会等于 1，最后我发出现出来对话框其实决定于

我认为你能让注册窗口跳出来强行跳转的结果，没有挖出它的根！

### 关键指令参考

* Reference to method `TSavePictureDialog.Execute()`

```
0049F8AD  FF523C                   call    dword ptr [edx+$3C]

0049F8B0  84C0                     test    al, al

0049F8B2  744A                     jz      0049F8FE    《《《这里就去购买了，所以不能去！

0049F8B4  8D55F8                   lea     edx, [ebp-$08]
```

其实你可以在上面这个 `call dword ptr [edx+83c]` 进去，看看到为什么 `AL` 会等于 1，最后我发出现出来对话框其实决定于:

```
0049F814  80BBA407000000           cmp     byte ptr [ebx+$07A4], $00
```

你也可以锁定 `ebx+07a4`，看它什么时候变成 1。

`[ebx+$07a4]`、`[ebx+$07a5]` 两个地址处的内容非常重要，你跟进了上面我说的那个 `CALL`!

`[ebx+07a4]` 决定是否功能限制，如是为“1”，它会把 `[ebx+07a5]` 置 0，否则以上两个值就相反。

而 `[ebx+07a5]` 决定是否出来填写注册码的对话框，所以如果你能让 `[ebx+07a4]=1`，`[ebx+07a5]=0` 应该就是注册版了！

## 修改方案

最后我的改法如下：

1. 找: `0F8C65010000E871F1FFFF`

   改: `909090909090----------`

2. 找: `0F9C4F010000C645FF01`

   改: `909090909090--------`

3. 找: `7D04C645FF008B45F0`

   改: `EB----------------`

它们的具体 `offset` 我没记下来！你应该可以找得到，看看这几个地方的代码你就会明白了。

以上是我的观点，敬请指正！呵呵。。。。。。

热烈欢迎继续讨论！ `4dd466d0e1e0cc123afbe504da2462ca`
