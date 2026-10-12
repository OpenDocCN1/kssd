
# mymahj 3.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1073.htm`](https://www.pediy.com/kssd/pediy03/forum1073.htm)

软件名称：`mymahj 3.2`
保护方式：注册码
破解人：`TAE![BCG]` `TAE![FCG]`
破解方式：找出注册码
软件简介：一个找对子的麻将游戏。最近本人对麻将感兴趣，呵呵!
下载地址：`http://home.nordnet.fr/~oplayez/`

首先运行程序，出现要求注册的对话框。
输入 `Email` 地址，和注册码（当然是假的咯，呵呵）
`Ctrl+N`,设置断点 `hmemcpy`.回到程序点注册.`ok!`又到了 `TRW`

下命令 `pmodule`,立刻回到了程序领空.就是这里:

```
* Reference To: USER32.GetDlgItemTextA, Ord:00EFh
                        |
:00405E91 E8CEDF0000               Call 00413E64
:00405E96 6A34                     push 00000034  <======= 到了这里
:00405E98 68B8ED4500               push 0045EDB8

* Possible Reference to Dialog: DialogID_0393, CONTROL_ID:29D0, ""
                        |
:00405E9D 68D0290000               push 000029D0
:00405EA2 FF7508                   push [ebp+08]

* Reference To: USER32.GetDlgItemTextA, Ord:00EFh
                        |
:00405EA5 E8BADF0000               Call 00413E64
:00405EAA E85A140000               call 00407309                ----
:00405EAF 803DBD2C470001           cmp byte ptr [00472CBD], 01    |很经典吧!
:00405EB6 7516                     jne 00405ECE                ----
```

进入 `00407309`,在这里:

```
* Referenced by a CALL at Addresses:
|:00405EAA  , :00413959 《＝＝＝这个是启动时判断注册码
|
:00407309 55                         push ebp
:0040730A 89E5                       mov ebp, esp
:0040730C 51                         push ecx
:0040730D 53                         push ebx
:0040730E 56                         push esi
:0040730F 57                         push edi
:00407310 31DB                       xor ebx, ebx
:00407312 C745FC01000000             mov [ebp-04], 00000001
:00407319 68B8ED4500                 push 0045EDB8

* Reference To: CRTDLL.atol, Ord:01DEh
                        |
:0040731E E8D1CE0000                 Call 004141F4     <==== 经过这个以后你的注册码就变成数值了
:00407323 59                         pop ecx            <==== 假注册码地址在 ecx
:00407324 8945FC                     mov dword ptr [ebp-04], eax
:00407327 31F6                       xor esi, esi
:00407329 46                         inc esi
:0040732A 0FBE1D84ED4500             movsx ebx, byte ptr [0045ED84]
:00407331 EB34                       jmp 00407367
```

下面就是运算核心的第一段!

```
:00407333 89F0                       mov eax, esi
:00407335 B904000000                 mov ecx, 00000004
:0040733A 99                         cdq
:0040733B F7F9                       idiv ecx
:0040733D 09D2                       or edx, edx
:0040733F 7415                       je 00407356
:00407341 0FBE3C3584ED4500           movsx edi, byte ptr [esi+0045ED84]  \\EDI 依次指向 email 的每个字符 ascii 码值
:00407349 C1E703                     shl edi, 03                        \\ascii 码值乘 3 给 edi
:0040734C 81EFA8000000               sub edi, 000000A8                  \\再减 A8 给 edi
:00407352 01FB                       add ebx, edi                       \\再加上 edi,存入 ebx
:00407354 EB10                       jmp 00407366
:00407356 0FBE3C3584ED4500           movsx edi, byte ptr [esi+0045ED84]
:0040735E C1E702                     shl edi, 02
:00407361 83EF54                     sub edi, 00000054
:00407364 29FB                       sub ebx, edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407354(U)
|
:00407366 46                         inc esi              <===== 计数器
:00407367 BF84ED4500                 mov edi, 0045ED84  <===== 这是你输入的 Email,地址放入 edi
:0040736C 31C0                       xor eax, eax       <===== eax 寄存器清空
:0040736E F9                         stc                \\
:0040736F 19C9                       sbb ecx, ecx       \\这里不太明白作用,请各位帮助
:00407371 F2                         repnz              /
:00407372 AE                         scasb              /
:00407373 F7D9                       neg ecx           /
:00407375 8D41FE                     lea eax, dword ptr [ecx-02]
:00407378 39C6                       cmp esi, eax       <===== eax 是你输入的 email 的字符个数，就是循环几次的关键
:0040737A 7CB7                       jl 00407333        <===== 没有完就继续
```

下面是运算核心的第二段:

```
:0040737C 0FBE0589ED4500             movsx eax, byte ptr [0045ED89]  \\取 email 的第 5 位 ascii 码参加运算!
:00407383 83E812                     sub eax, 00000012             \\减 12
:00407386 0FBE158AED4500             movsx edx, byte ptr [0045ED8A]  \\取 email 的第 7 位 ascii 码参加运算
:0040738D 83EA12                     sub edx, 00000012             \\减 12
:00407390 01D0                       add eax, edx                 \\eax+edx
:00407392 0FAFD8                     imul ebx, eax               \\ebx*eax(eax 就是第一段运算核心得出的数值)
:00407395 3B5DFC                     cmp ebx, dword ptr [ebp-04]    \\比较! 这里? ebx 即可得到真正的注册码!
:00407398 750E                       jne 004073A8                \\如果不相等就完了!哈哈
:0040739A E80AFFFFFF                 call 004072A9
:0040739F C605BD2C470001             mov byte ptr [00472CBD], 01    \\看到这个 01 就是舒服!哈哈哈
:004073A6 EB07                       jmp 004073AF
:004073A8 C605BD2C470000             mov byte ptr [00472CBD], 00    \\到了这里就。。。
```

谁来写个注册机？

这是我的注册码：`Email:TAESUE@163.com`
`Sn: 318936`

软件注册成功后会在目录下生成一个隐含文件 `Mymahj.key`！

写个教程真的很麻烦!花了一个多小时,呼呼!

但是今天很高兴，因为我已经加入了 `BCG`, `FCG`

非常感谢 `bnbnf`(我兄弟^\_^）、`copyyour`、`xy2000[BCG]`、`小球[CCG]`、`幻影 2001`、`鱼儿`、`娃娃`等人的支持！(排名不分先后，哈哈哈）

文中有疏漏的地方还请各位指出。谢谢！！ `4dd466d0e1e0cc123afbe504da2462ca`
