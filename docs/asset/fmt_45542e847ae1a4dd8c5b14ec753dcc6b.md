
# 源江科技出的源代码(VB 篇)

> 原文：[`www.pediy.com/kssd/pediy03/forum1187.htm`](https://www.pediy.com/kssd/pediy03/forum1187.htm)

## 开场白

唉！

刚刚买了几张碟回来，但是除了那几张 `MPEG4` 好用之外，另外两张软件碟都不好用！

特别是那张“金山软件 3000 全集”，读不到！

还有那张源江科技出的“源代码(VB 篇)”，原来要输入注册码，才可以用它自带的浏览器！！！！

我倒！注册费要 20 元？那别人买正版的呢？呵呵！可能附送了注册码吧？我这个盗版就没有！结果用不了！

只好自己动手！顺便作为庆祝 `FCG` 主页的 `ASP` 更新程序下个星期正式使用 ^\_^ `http://www.cn778.net/index.asp`

## 说明

目标软件：源江科技出的“源代码(VB 篇)”的浏览器（和注册器）

使用工具：`W32Dasm` `8.93`、`UltraEdit-32`

## 开始

如果没注册，用它自带的浏览器 `browse.exe`，一开始就会弹出“你的程序未注册”的对话框，然后自动退出！（呵呵！没有图片，因为忘记了在没注册之前截图 ^\_^）

要注册，就要用自带的注册器 `Regedit.exe`，当然，我没有正确的注册码，按确定，会在软件左下角出现“注册失败”的字样（如图一）！呵呵！我就觉得源江科技有点“傻”，居然提供一个注册器，这个东西却是他的致命伤（看后文就知道了 ^\_^）！

![](img/1f854acafd30e2f92e7251faca40ada8.jpg)

最开始破解这个东西还有一点小插曲（我摆了个大乌龙 ^\_^）——我把那两个文件 `copy` 到硬盘，接着用 `fi` 来检测一下有没有加壳（`sunbird` 大哥教的 ^\_^），结果没有，这个软件使用 `Delphi` 编的，然后用 `W32Dasm` 反编译 `browse.exe`，然后打开“串式参考”，找到“你的程序未注册”，双击，看到代码如下：

```asm
* Possible StringData Ref from Code Obj ->"RegCode"
                      |
:0046EC69 B91CED4600          mov ecx, 0046ED1C
:0046EC6E B201              mov dl, 01
:0046EC70 A180D04600          mov eax, dword ptr [0046D080]
:0046EC75 E84EE9FFFF          call 0046D5C8
:0046EC7A A328294700          mov dword ptr [00472928], eax
:0046EC7F 6A00              push 00000000
:0046EC81 8D45FC              lea eax, dword ptr [ebp-04]
:0046EC84 50                  push eax
* Possible StringData Ref from Code Obj ->"vbsource"
                      |
:0046EC85 B92CED4600          mov ecx, 0046ED2C
* Possible StringData Ref from Code Obj ->"yjkj"
                      |
:0046EC8A BA40ED4600          mov edx, 0046ED40
:0046EC8F A128294700          mov eax, dword ptr [00472928]
:0046EC94 E8C3E9FFFF          call 0046D65C
:0046EC99 8B45FC              mov eax, dword ptr [ebp-04]
* Possible StringData Ref from Code Obj ->"nsygbd0001"
                      |
:0046EC9C BA50ED4600          mov edx, 0046ED50                    \
:0046ECA1 E87A50F9FF          call 00403D20                         / 呵呵！很明显吧？
:0046ECA6 7416              je 0046ECBE                             / 把 je 改成 jne 就行啦！
* Possible StringData Ref from Code Obj ->"您的程序未注册"  //找到这里，往上找一找
                      |
:0046ECA8 B864ED4600          mov eax, 0046ED64
:0046ECAD E832B5FDFF          call 0044A1E4
:0046ECB2 A180184700          mov eax, dword ptr [00471880]
:0046ECB7 8B00                mov eax, dword ptr [eax]
:0046ECB9 E8B28BFDFF          call 00447870
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046ECA6(C)
|
:0046ECBE 33C0                xor eax, eax
:0046ECC0 5A                  pop edx
:0046ECC1 59                  pop ecx
:0046ECC2 59                  pop ecx
:0046ECC3 648910              mov dword ptr fs:[eax], edx
:0046ECC6 68F0EC4600          push 0046ECF0
```

呵呵！用 `UltraEdit32` 载入 `browse.exe`，查找 `E87A50F9FF`，然后把紧接在后面的 `74` 选中，准备改成 `75`，可是怎么改也改不了！呵呵！有什么地方出错呢？马上上网，找到 `peter`，给 `peter` 看了看代码，他也是这么说的，但是，我就是改不了！呵呵！想了想，哦！原来那个文件是“只读文件”！所以当然改不了了！^\_^（不知大家有犯过这种错误没有 ^\_^）（感谢 `peter` 的帮助！）

好了，把 `browse.exe` 改好后，运行，但是会出现一些错误的对话框，大概是我没有 `copy` 光碟上面其他的东西过来吧？但是，虽然可以进到 `browse.exe` 主界面，可是由于这个原因，仍然看不到那些内容！呵呵！由于硬盘空间有限，我不想把全部内容 `copy` 到硬盘，所以决定放弃这种方法！

那我们就从 `Regedit.exe`（不是 `windows` 那个哦！）入手吧！既然它有注册功能，就可能会成功 ^\_^

用 `W32Dasm` 反编译 `Regedit.exe`，打开“串式参考”，找到“注册失败”，双击，回到主界面，看到如下的代码：

```asm
* Possible StringData Ref from Code Obj ->"nsygbd0001"    //很熟悉吧？browse.exe 也有的
                      |
:0045C6AB BA24C74500          mov edx, 0045C724                \
:0045C6B0 E8EF75FAFF          call 00403CA4                    / 又是熟悉的东东！
:0045C6B5 7520              jne 0045C6D7                    / 把 jne 改成 je，刚好与 browse.exe
:0045C6B7 8BC3                mov eax, ebx                    的改法相反！
:0045C6B9 E8E6FEFFFF          call 0045C5A4
:0045C6BE C6057CF8450001      mov byte ptr [0045F87C], 01
* Possible StringData Ref from Code Obj ->"注册成功"        // 我们期待的^\_^
                      |
:0045C6C5 BA38C74500          mov edx, 0045C738
:0045C6CA 8B83D8020000        mov eax, dword ptr [ebx+000002D8]
:0045C6D0 E8BBF3FFFF          call 0045BA90
:0045C6D5 EB24              jmp 0045C6FB
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045C6B5(C)
|
* Possible StringData Ref from Code Obj ->"注册失败"        //找到了！往上找一找！
                      |
:0045C6D7 BA4CC74500          mov edx, 0045C74C
:0045C6DC 8B83D8020000        mov eax, dword ptr [ebx+000002D8]
:0045C6E2 E8A9F3FFFF          call 0045BA90
:0045C6E7 33D2                xor edx, edx
:0045C6E9 8B83D0020000        mov eax, dword ptr [ebx+000002D0]
:0045C6EF E80C85FCFF          call 00424C00
:0045C6F4 C6057CF8450000      mov byte ptr [0045F87C], 00
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045C6D5(U)
|
:0045C6FB 33C0                xor eax, eax
:0045C6FD 5A                  pop edx
:0045C6FE 59                  pop ecx
:0045C6FF 59                  pop ecx
:0045C700 648910              mov dword ptr fs:[eax], edx
:0045C703 6818C74500          push 0045C718
```

呵呵！还是一样，把 `Regedit.exe` 的属性去掉“只读”（吸取教训呀 ^\_^），然后用 `UltraEdit32` 找到 `E8EF75FAFF`，把紧接在后面的 `75` 改成 `74`，呵呵！保存，退出！直接执行 `Regedit.exe`，乱输入一些注册码（呵呵！我是没有输入的 ^\_^ 如果太太好运，输入正确的注册码，呵呵！那就会出现“注册失败”哦！），看看窗口的左下脚，出现“注册成功”了吧？

呵呵！然后，回到光碟里面，执行光碟上面那个 `browse.exe` 就行啦！漂亮的界面终于出现啦！

## 后记

其实，真正破解这个软件（不知道是不是叫做软件），所需要的时间应该可以控制在 3 分钟的，但是由于我的“乌龙”，结果用了半个小时 ^\_^ 大家在破解软件的时候，可要注意这一点哦！（扔鸡蛋，有谁会像你这么傻！！%&\*%##@!）

还有，写这篇东西也用的将近半个小时！^\_^

要下载已破解的 `Regedit.exe`，到 `FCG` 主页吧！`http://www.cn778.net/index.asp`

`Cyrix686`

`2001-7-28`  `4dd466d0e1e0cc123afbe504da2462ca`
