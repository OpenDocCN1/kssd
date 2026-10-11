
# See This 1.07

> 原文：[`www.pediy.com/kssd/pediy02/forum116.htm`](https://www.pediy.com/kssd/pediy02/forum116.htm)

软件名称：`See This 1.07`

软件大小：`248KB`

软件类型：共享

适用环境：`Win9x`

**软件介绍：**

当你从网络上看到好东西的时候，你一定很想把整个画面传给朋友看。

`See This` 就是一个这麽好用的小工具，可以把整个屏幕“拍”下来，并且用 `Email` 传送出去。

如果对方也是 `See This` 的用户，画面就会立刻被传出去并且出现在他的屏幕上。要是对方没有使用 `See This`，

那麽这张图就会以 `jpg` 格式的 `Email` 夹文件传出去。

### 汇编代码参考

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00417514(C)` |
|----------------|

```assembly
:004174F1 0FBE54340C             movsx edx, byte ptr [esp+esi+0C]
:004174F6 52                     push edx
:004174F7 E834E40000             call 00425930 <-----大小写转换
:004174FC 88443410               mov byte ptr [esp+esi+10], al <----放回原处
:00417500 83C404                 add esp, 00000004
:00417503 8D7C240C               lea edi, dword ptr [esp+0C]
:00417507 83C9FF                 or ecx, FFFFFFFF
:0041750A 33C0                   xor eax, eax
:0041750C 46                     inc esi
:0041750D F2                     repnz
:0041750E AE                     scasb
:0041750F F7D1                   not ecx
:00417511 49                     dec ecx
:00417512 3BF1                   cmp esi, ecx
:00417514 72DB                   jb 004174F1
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004173BC(C)` |
|----------------|

```assembly
:0041739E 33C9                   xor ecx, ecx
:004173A0 8A88B09A4300           mov cl, byte ptr [eax+00439AB0]
:004173A6 83C1BF                 add ecx, FFFFFFBF
:004173A9 83F90F                 cmp ecx, 0000000F
:004173AC 774F                   ja 004173FD
:004173AE 8BD6                   mov edx, esi
:004173B0 0FAFD1                 imul edx, ecx
:004173B3 03DA                   add ebx, edx
:004173B5 C1E604                 shl esi, 04
:004173B8 40                     inc eax
:004173B9 83F808                 cmp eax, 00000008
:004173BC 7CE0                   jl 0041739E
:004173BE 3BEB                   cmp ebp, ebx
:004173C0 733B                   jnb 004173FD
:004173C2 8B0D68B64300           mov ecx, dword ptr [0043B668]
:004173C8 8B156CB64300           mov edx, dword ptr [0043B66C]
```

**例：**

输入：`abcdefgh`

转换后：`01234567`

由于追踪过程中,注册成功,未能将比较代码的程序记录下来. 卸载程序,仍是注册的.

**大致过程如下:**

1. 软件按照上面的办法将输入码减 `41(HEX)`，然后与 `50(HEX)` 比较(即输入的注册码要求
   是字符且要小于等于 `50(HEX,即字符 P)`)，注册码为 `8` 位.

2. 软件将转换后的码与内部码(放在 `EBP` 中)比较(注:软件有多个内部码)

我得到的是：`LAPKLDFB` `4dd466d0e1e0cc123afbe504da2462ca`
