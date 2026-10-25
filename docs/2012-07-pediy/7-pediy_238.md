
# QQ1220

> 原文：`www.pediy.com/kssd/pediy05/pediy50548.htm`

> 精华 4 夜月同志这篇文章还是不错的。。。但是有几个问题：
>
> (1) 最大的问题是根本显示不了 IP，原因在于 `push wsprintfA` 函数的参数的时候少 `push` 了 IP 的 EIP。
>
> 0043AF15: 51 push ecx
> 0043AF16: 52 push edx
> 0043AF17: 56 push esi
> 0043AF18: 8B8614040000 mov eax,[esi][00000414]
> 0043AF1E: 8B8058030000 mov eax,[eax][00000358]
> 0043AF24: 803800 cmp b,[eax],000 ;" "
> 0043AF27: 7436 je .00043AF5F ----- (2)
> 0043AF29: 50 push eax
> 0043AF2A: 50 push eax
> 0043AF2B: FF1510035D00 call _GetAddress ;ipsearcher.dl
> 0043AF31: 83C404 add esp,004 ;""
> 0043AF34: FF7004 push d,[eax][04]
> 0043AF37: FF30 push d,[eax]
> 0043AF39: 6890B95F00 push 0005FB990 ----- (3)
> 0043AF3E: 90 nop
> 0043AF3F: 68A0B95F00 push 0005FB9A0 ----- (4)
> 0043AF44: FF15E4165800 call wsprintfA ;USER32.dll
> 0043AF4A: 83C414 add esp,014 ;""
> 0043AF4D: B8A0B95F00 mov eax,0005FB9A0 ----- (5)
> 0043AF52: 50 push eax
> 0043AF53: FFB64C030000 push d,[esi][0000034C]
> 0043AF59: FF15D4145800 call SetWindowTextA ;USER32.dll
> 0043AF59: FF15D4145800 call SetWindowTextA ;USER32.dll
> 0043AF5F: 5E pop esi
> 0043AF60: 5A pop edx
> 0043AF61: 59 pop ecx
> 0043AF62: 8D86E4030000 lea eax,[esi][000003E4]
> 0043AF68: E9E8CA0100 jmp .000457A55 ----- (5)
>
> (2) 没有修改 exe 的资源，把广告窗口的类型改成 `EDIT` 或者 `STATIC|TEXT` 才能把 `SetWindowTextA` 的东西显示出来（这个大家自己改吧）。
>
> (3) 数据段的长度没有改，exe 一旦压缩就显示不了 IP 了（应该改成 2B000 左右，而不是原来的 2A960）。
>
> (4) 改好之后 win2k/xp 会认为它是个非法的 Win32 文件，可以用 `ProcDump` 的 `PE Rebuilder` 来一遍，之后可以在所有平台运行。
>
> (5) 5FB990（就是 `wsprintfA` 的格式参数）处应该填上 `%s %s\n%s`，不然也显示不出来 IP。 顺便说一句，(3)的原因就是压缩软件只取了段的物理大小范围进行压缩，这样就把格式串忽略了，解压时被设成 0，所以显示不出来任何东西。
>
> 怎么样才可以在点击面板上 TE 图标时，运行的是 IE 或系统默认的浏览器，而不是 QQ 自带的 TE？
>
> 谢谢！

压栈的时候把 `tbrowse.exe` 改成 `url`，原 `url` 改成 `NULL`

1. 我做那个东西的时候的确没有想过让它显示 IP 出来，我只是想让它显示出地址而已。
   因为如果显示了 IP，很多人能干的事情就不光是查地址而已了。我做这个补丁只是
   为了使用 QQ 方便，不想造成更多的使用傻瓜式的黑客软件攻击 QQ 的事情发生。

2. 关于资源修改的问题。在写作 《精华 4》那篇文章时，有点赶工，导致了修改后的
   界面显示非常难看，最近这段时间，继续研究了 QQ 的有关的东西，有了更进一步的
   进展，最大的收获是知道了如何自己在里面添加新的 `Edit` 控件，并且调用 QQ 自带的
   函数在里面把信息显示出来。由于最近一直比较忙，学校课太多，没有时间整理出
   文。不过受最大侠鼓励，我会抽空把那篇文章完善，希望能对大家的胃口。

3. 至于第 3 和第 4 点错误，的确是我的学识所限，在此之前不知道要进行这些修改。在
   此多谢最大侠指教。

   大家对于修改 QQ 还有什么建议，可以跟贴，如果在我的能力所能及的范围内，我会
   尽量满足大家的需求，写出文章。

\#BeginLibraryItem "/Library/copyright.lbi" CLASS="p9z" Copyright © 2000-2003 看雪学院(www.pediy.com) All Rights Reserved. \#EndLibraryItem
