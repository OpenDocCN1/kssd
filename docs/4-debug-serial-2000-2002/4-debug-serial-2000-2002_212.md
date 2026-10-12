
# AmazingMIDI v 1.60

> 原文：[`www.pediy.com/kssd/pediy03/forum1152.htm`](https://www.pediy.com/kssd/pediy03/forum1152.htm)

- **作者**：每位 Cracker 都可以
- **对象**：`AmazingMIDI v 1.60`
- **保护**：注册码
- **工具**：`trw2000`

安装并运行 `AmazingMIDI`，弹出一个窗口，提示“注册后不再显示此窗口”之类的内容，点“确定”来到程序的主界面。在“帮助”菜单里有一个“注册”子菜单，点击它，弹出注册框。（嘻嘻，这个软件的注册对话框里只有一个文本输入框，是不是挺简单的——心里想）。输入 `789789`，点“确定”，先来碰碰运气。(:- 晕~~，有这样碰运气的吗？) 呵呵，其实也不是什么碰运气，只是想看看它会弹出什么错误信息来。果然，“抱歉！注册号码错误。”随着“震耳欲聋”的报错声弹了出来。（不是吧，没声卡也能发声？倒，，没声卡都被你说了出来，以后我怎么见人呀，?~!）

再次输入 `789789`

运行 `trw2000`

下断点 `bpx hmemcpy`

`F5` 或 `G` 返回程序界面，点击“确定”，随后程序会被中断

`bc` 清除断点

`pmodule` 领空

接下来的工作就是不停的按 `F12` 而且口中也得跟着 `F12` 的次数念念有词，到了 33 次——

`STOP !` 按 `F10` 单步执行后，来到下面的代码处：

```
:00405D66 83F801              cmp eax, 00000001
:00405D69 0F85B6000000        jne 00405E25
:00405D6F 51                  push ecx
:00405D70 8D442470            lea eax, dword ptr [esp+70]
:00405D74 8BCC                mov ecx, esp
:00405D76 89642410            mov dword ptr [esp+10], esp
:00405D7A 50                  push eax
:00405D7B E887B50100          call 00421307
:00405D80 8D4C240C            lea ecx, dword ptr [esp+0C]
:00405D84 C644247C00          mov [esp+7C], 00
:00405D89 51                  push ecx
:00405D8A E861010000          call 00405EF0
:00405D8F 8BC8                mov ecx, eax
:00405D91 E8AA180000          call 00407640
:00405D96 8B00                mov eax, dword ptr [eax]

* Possible StringData Ref from Data Obj -> "<<@::5<720"

:00405D98 BE58054400          mov esi, 00440558
// d eax 可以看到被加密过的 789789

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DBF(C)

:00405D9D 8A10                mov dl, byte ptr [eax]
// d esi 可以看到 <<@::5<720 等字符
:00405D9F 8A1E                mov bl, byte ptr [esi]
:00405DA1 8ACA                mov cl, dl
:00405DA3 3AD3                cmp dl, bl
:00405DA5 751E                jne 00405DC5
// 不能跳哦,
:00405DA7 84C9                test cl, cl
:00405DA9 7416                je 00405DC1
// 抬头可见 mov cl, dl 即取完
// 输入的字符时就跳
:00405DAB 8A5001             mov dl, byte ptr [eax+01]
:00405DAE 8A5E01             mov bl, byte ptr [esi+01]
:00405DB1 8ACA                mov cl, dl
:00405DB3 3AD3                cmp dl, bl
:00405DB5 750E                jne 00405DC5
// 也不能跳哦,
:00405DB7 83C002             add eax, 00000002
:00405DBA 83C602             add esi, 00000002
:00405DBD 84C9                test cl, cl
:00405DBF 75DC                jne 00405D9D
// 没取完输入的字符就跳回去继续取

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DA9(C)

:00405DC1 33C0                xor eax, eax
:00405DC3 EB05                jmp 00405DCA

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405DA5(C), :00405DB5(C)

:00405DC5 1BC0                sbb eax, eax
:00405DC7 83D8FF             sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DC3(U)

:00405DCA 85C0                test eax, eax
:00405DCC 8D4C2408           lea ecx, dword ptr [esp+08]
:00405DD0 0F94C3             sete bl
:00405DD3 E8BAB70100         call 00421592
:00405DD8 84DB                test bl, bl
:00405DDA 7513                jne 00405DEF
// 这里不跳可不行, 呵呵
:00405DDC 6AFF                push FFFFFFFF
:00405DDE 6A00                push 00000000
...
```

这个程序利用固定的 key (`9876543210`) 把输入的字符进行加密并反向取值，然后与 `<<@::5<720` 相比较，如果相等则注册码正确，否则就错误（废话!!），所以注册码也是固定的。

`4dd466d0e1e0cc123afbe504da2462ca`
