
# Update Now 2.0

> 原文：[`www.pediy.com/kssd/pediy01/forum054.htm`](https://www.pediy.com/kssd/pediy01/forum054.htm)

今天破掉了 `Update Now2.0`（主页上传工具），这次选了个最简单的 name 选 1。

`Name`: 1  
`RegisterKey`: 881cdceed7c4

这个软件注册检验部分有趣，注册码根据注册名运算得出，但注册码必须是 12 位，现在好多程序都要求注册码位是固定的哟，:)

好了，废话少说，程序破解于下，请注意听哟：

1. 用 `ice` 载入 `Update Now2.0`
2. 输入注册信息
   `name`: 1 (任意输入)  
   `register key`: 123456789012 (任意输入，此例中便于观察)。
3. 下
   ```
   :bpx hmemcpy
   :g
   ```
   回到程序界面时，按“OK”
4. 程序被拦下来，按几下 `F12` 回到程序领空，见如下代码：

```
XXXX:00407643                 push 000003F1
XXXX:00407648                 mov ecx, ebx    |
XXXX:0040764A                 Call 00416876
XXXX:0040764F                 cmp eax, 0000000C  &比较注册是否为 12 位
XXXX:00407652                 jne 00407862      &不是 12 位，则死!
XXXX:00407658                 lea edi, dword ptr [esp+00000124]
XXXX:0040765F                 or ecx, FFFFFFFF
XXXX:00407662                 xor eax, eax
```

程序继续。。。到

```
:004076E0   mov dl, byte ptr [esp+eax+29]   & d esp+eax+28 便可看
:004076E4   mov cl, byte ptr [esp+eax+28]   & 到你输入的注册码：
:004076E8   mov byte ptr [esp+eax+28], dl   & 123456789012
:004076EC   mov byte ptr [esp+eax+29], cl
:004076F0   inc eax
:004076F1   cmp eax, 00000006
:004076F4   jl 004076E0
:004076F6   dec esi
:004076F7   jne 004076DE
:004076F9   lea edi, dword ptr [esp+00000228] &此时下 d eax+esp+22
                                                   &可以看见你输入的注册码
                                                   &被变为 712345689012
```

以上段代码是你输入注册码的变形，我们此时输入的注册码为 `123456789012` 被变为 `712345689012`，也就是说将我们输入的注册码的第 7 位移到了第一位!

5. 程序继续执行，(`F10`)下面到了如下代码：

```
:004077AE   Call dword ptr [004194B4]       &注册码比较核心，F8 追入哟!
:004077B4   add esp, 0000000C
:004077B7   test eax, eax                  &检验注册标记
:004077B9   pop ebp
:004077BA   jne 00407838                   &不为 0，则死!
:004077BC   mov esi, dword ptr [esp+0C]
:004077C0   push eax
```

6. 让我们看看注册码比较核心部分的代码：

```
XXXX:78010C8E   PUSH EBP
               MOV EBP,ESP
.......................................
XXXX:78010CAF   MOV AL,[ESI-01]    &下 d esi-01 即可看见 e881cdced7c4
XXXX:78010CB2   XOR ECX,ECX        &由你用户名算出来的注册码
   :78010CB4   CMP AL,[EDI-01]    &下 d edi-01 即可看见 712345689012
   :78010CB7   JA 78010CBD        
   :78010CB9   JZ 78010CBF
       ...................(略)
```

7. 最后整理一下，重新注册，直接输入注册码 `e881cdced7c4`，哇!怎么不对？？？哈哈，由于作者对你输入的注册码做了一个小变化，将第 7 位调到了第 1 位，所以我们输入注册码的时候，就要输入 `e881cdced7c4` 的第 1 位调到第 7 位，应该为 `881cdceed7c4`，于是 OK!

破解者：十三少  
`scybs@sina.com`  
`http://scyb.yeah.net`  
00.05.14
