
# 搜索引擎工厂(Search Engine Builder)V1.595

## 详细信息

简单算法――搜索引擎工厂(Search Engine Builder)V1.595

- 标 题：搜索引擎工厂(Search Engine Builder)V1.595
- 发信人：fly
- 时 间：2003/02/28 08:43am
- 软件大小：604 KB
- 软件语言：英文
- 软件类别：国产软件 / 共享版 / 网页辅助
- 应用平台：Win9x/NT/2000/XP
- 加入时间：2002-12-28 09:32:11
- 下载次数：3492
- 推荐等级：\*\*\*
- 开 发 商：http://www.seamoontech.com/

## 软件简介

为你的网站自动生成一个站内搜索引擎。很多人访问您的网站只是为了得到其中一部分信息而不想全部浏览完毕您的网站。如果让您的客户一页页的寻找这小部分信息，无疑一部分客户会失去耐心而转向别处。Cool Search Maker 就是为了解决这个问题而产生的。它快速的索引整个网站并自动生成一个高效的搜索引擎，这是一个 HTML 文件，把它加入到您的网站上即可。使用 Cool Search Maker 生成搜索引擎，您所须做的只是点几下鼠标而已。有了它，您不必耗费大量的时间亲自去写繁杂的搜索代码了。Cool Search Maker 还有许多附加的特性如：自定义摸板用来生成漂亮的搜索页面，自定义搜索结果的输出格式，为本地文本文件生成搜索引擎以便于查询。

## 软件限制

NAG、功能限制。

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

TRW2000娃娃修改版、FI2.5、W32Dasm黄金版

---

## 过程

`SearchMaker.exe`是VC++6.0编写。无壳。反汇编方便了。^-^

程序要求重启验证注册码。程序把试炼码写入了注册表，启动时进行比较。

TRW调试时当然可下断点：`BPX Regqueryvalueexa do"dd\*(esp+8)"`

只是装入后必须按很多下F5键，烦人。

于是想起以前破过的FlashGet、EZ Extract Resource等等同种验证方式（重启验证），学习了许多大侠的经验，想起了一点小办法。

在反汇编代码里查找“RegCode”，一般会有2处，那么其中的1处就是核心了。省我按几十次F5键了。呵呵

OK，查到了。直接`BPX 40DC92`，重启时拦下！

Let's Go!

---

```asm
\*
Possible StringData Ref from Data Obj ->"RegUserName"
       |
:0040DC4C 682C194600
     push 0046192C
:0040DC51 8D542418
       lea edx, dword ptr [esp+18]
:0040DC55 B305
                 mov bl, 05
\*
Possible StringData Ref from Data Obj ->"RegInfo"
     |
:0040DC57 6824194600
   push 00461924
:0040DC5C 52
           push edx
:0040DC5D 8BCE
               mov ecx, esi
:0040DC5F
889C247C050000          mov byte ptr [esp+0000057C],
bl
:0040DC66 E807920300              call
00446E72
:0040DC6B 50
     push eax
:0040DC6C 8D4C2418
       lea ecx, dword ptr [esp+18]
:0040DC70 C684247005000006
       mov byte ptr [esp+00000570], 06
:0040DC78 E8387A0200
             call 004356B5
:0040DC7D 8D4C2410
               lea ecx, dword ptr [esp+10]
:0040DC81
889C246C050000          mov byte ptr [esp+0000056C],
bl
:0040DC88 E8EF780200              call
0043557C
:0040DC8D 68145C4600              push
00465C14
\* Possible
StringData Ref from Data Obj ->"RegCode"
   |
:0040DC92 681C194600
 push 0046191C
                 ====>中断在这！
:0040DC97
8D442428                lea eax, dword
ptr [esp+28]
\* Possible
StringData Ref from Data Obj ->"RegInfo"
   |
:0040DC9B 6824194600
 push 00461924
:0040DCA0 50
         push eax
:0040DCA1 8BCE
             mov ecx, esi
:0040DCA3 E8CA910300
             call 00446E72
:0040DCA8 50
                     push
eax
:0040DCA9 8D4C241C                lea
ecx, dword ptr [esp+1C]
:0040DCAD C684247005000007        mov
byte ptr [esp+00000570], 07
:0040DCB5 E8FB790200
     call 004356B5
:0040DCBA 8D4C2420
       lea ecx, dword ptr [esp+20]
:0040DCBE 889C246C050000
         mov byte ptr [esp+0000056C], bl
:0040DCC5
E8B2780200              call 0043557C
:0040DCCA
51                      push
ecx
:0040DCCB 8D54241C                lea
edx, dword ptr [esp+1C]
:0040DCCF 8BCC
       mov ecx, esp
:0040DCD1 89642414
         mov dword ptr [esp+14], esp
:0040DCD5 52
                     push
edx
:0040DCD6 E816760200              call
004352F1
:0040DCDB 51
     push ecx
:0040DCDC 8D44241C
       lea eax, dword ptr [esp+1C]
:0040DCE0 8BCC
                 mov ecx, esp
:0040DCE2
89642424                mov dword ptr
[esp+24], esp
:0040DCE6 50
       push eax
:0040DCE7 C684247805000008
   mov byte ptr [esp+00000578], 08
:0040DCEF E8FD750200
         call 004352F1
:0040DCF4 8BCE
               mov ecx, esi
:0040DCF6
889C2474050000          mov byte ptr [esp+00000574],
bl
:0040DCFD E86E160000              call
0040F370
:0040DD02 33FF
   xor edi, edi
:0040DD04 3BC7
         cmp eax, edi
:0040DD06 7408
               je 0040DD10
:0040DD08
89AECC010000            mov dword ptr [esi+000001CC],
ebp
:0040DD0E EB52
   jmp 0040DD62
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040DD06(C)
|
:0040DD10
39AEC0000000            cmp dword ptr [esi+000000C0],
ebp
:0040DD16 7444
   je 0040DD5C
:0040DD18 51
         push ecx
:0040DD19 8D54241C
           lea edx, dword ptr [esp+1C]
:0040DD1D
8BCC                    mov
ecx, esp
:0040DD1F 89642420
 mov dword ptr [esp+20], esp
:0040DD23 52
             push edx
:0040DD24 E8C8750200
             call 004352F1
:0040DD29 51
                     push
ecx
:0040DD2A 8D44241C                lea
eax, dword ptr [esp+1C]
:0040DD2E 8BCC
       mov ecx, esp
:0040DD30 89642418
         mov dword ptr [esp+18], esp
:0040DD34 50
                     push
eax
:0040DD35 C684247805000009        mov byte ptr [esp+00000578],
09
:0040DD3D E8AF750200              call
004352F1
:0040DD42 8BCE
   mov ecx, esi
:0040DD44 889C2474050000
 mov byte ptr [esp+00000574], bl
:0040DD4B
E8C0070000              call 0040E510
       ====>核心CALL！
:0040DD50
3BC7                    cmp
eax, edi
:0040DD52 7408
   je 0040DD5C
                   ====>不能跳！
:0040DD54
89AECC010000            mov dword ptr [esi+000001CC],
ebp
:0040DD5A EB06
   jmp 0040DD62
```

---

F8进入关键CALL。`40DD4B call 0040E510`

```asm
\*
Referenced by a CALL at Address:
|:0040DD4B
```

看看下面吧，藏了几个黑名单："ttdown"、"crsky"、".com"、"jetdown"，这些都是破解网站，喜欢用自己的站名来做软件的注册名，统统禁杀了！

这些黑名单使我想起EZ Extract Resource的黑名单，几乎差不多，后来一看：这2个软件是一个公司开发的（http://www.seamoontech.com/），怪不得如此相象。

```asm
:0040E510
6AFF                    push
FFFFFFFF
:0040E512 6870B64400              push
0044B670
:0040E517 64A100000000            mov
eax, dword ptr fs:[00000000]
:0040E51D 50
           push eax
:0040E51E 64892500000000
         mov dword ptr fs:[00000000], esp
:0040E525
81ECD4000000            sub esp, 000000D4
:0040E52B
53                      push
ebx
:0040E52C 56
   push esi
:0040E52D 8BF1
       mov esi, ecx
:0040E52F B801000000
         mov eax, 00000001
:0040E534 68145C4600
           push 00465C14
:0040E539 898424E8000000
         mov dword ptr [esp+000000E8], eax
:0040E540
8986D0010000            mov dword ptr [esi+000001D0],
eax
:0040E546 8B8424F0000000          mov eax, dword
ptr [esp+000000F0]
:0040E54D 50
       push eax
:0040E54E E842620100
       call 00424795
:0040E553 83C408
           add esp, 00000008
:0040E556 85C0
                 test eax, eax
         ====>测试用户名是否为空
:0040E558
0F8477010000            je 0040E6D5
     ====>不能跳！
:0040E55E
8B8C24F0000000          mov ecx, dword ptr [esp+000000F0]
:0040E565
68145C4600              push 00465C14
:0040E56A
51                      push
ecx
:0040E56B E825620100
             call 00424795
:0040E570 83C408
                 add esp, 00000008
:0040E573
85C0                    test
eax, eax
             ====>测试注册码是否为空
:0040E575
0F845A010000            je 0040E6D5
     ====>不能跳！
\*
Possible StringData Ref from Data Obj ->"ttdown"
====>黑名单！
         |
:0040E57B 683C1A4600
       push 00461A3C
:0040E580 8D8C24F0000000
     lea ecx, dword ptr [esp+000000F0]
:0040E587 E846440200
             call 004329D2
:0040E58C 33DB
                   xor ebx,
ebx
:0040E58E 83F8FF
 cmp eax, FFFFFFFF
:0040E591 7542
       jne 0040E5D5
====>不能跳！
\*
Possible StringData Ref from Data Obj ->"crsky"
====>黑名单！
         |
:0040E593 68341A4600
       push 00461A34
:0040E598 8D8C24F0000000
     lea ecx, dword ptr [esp+000000F0]
:0040E59F E82E440200
             call 004329D2
:0040E5A4 83F8FF
                 cmp eax, FFFFFFFF
:0040E5A7
752C                    jne
0040E5D5
====>不能跳！
\*
Possible StringData Ref from Data Obj ->".com"
====>黑名单！
:0040E5A9 682C1A4600
       push 00461A2C
:0040E5AE 8D8C24F0000000
     lea ecx, dword ptr [esp+000000F0]
:0040E5B5 E818440200
             call 004329D2
:0040E5BA 83F8FF
                 cmp eax, FFFFFFFF
:0040E5BD
7516                    jne
0040E5D5
```
