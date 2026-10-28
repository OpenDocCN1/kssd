
```
pop esi
100431B8: 5B
pop ebx
100431B9: C9
leave
100431BA: C20800    ret 0008
```

这段代码我已经分析完毕。现在我们要做的事情就是在得到`DragQueryFile`的消息后，做一样的处理动作，然后post一个open file消息给自己，就达到目的了。关键的地方是如何得到`PostMessageA`的`hwnd`呢？这个`hwnd`是`w32asm`的`menu`的id。Windows程序员都知道`setmenu`这个函数吧，我们下断点`bpx setmenu`，看到原程序是在这里`setmenu`：

```
0049CD86: 8BF7             mov esi, edi
0049CD88: 56               push esi ＝＝＝》旧hwnd
0049CD89: 8B0B             mov ecx, dword ptr [ebx]
0049CD8B: FF710C           push [ecx+0C] ＝＝》新hwnd
\* Reference To: user32.SetMenu, Ord:0221h
                          |
0049CD8E: E865230100       Call 004AF0F8
0049CD93: 85C0             test eax, eax
```

我们要做的事情就是保存这个新的`hwnd`，以便`PostMessageA`的时候用到：

```
0049CD8E: E865230100       Call 004AF0F8
```

改为：

```
0049CD8E: E97D250100       jmp .0004AF310  -----到我们自己保存hwnd的代码
```

自己保存`hwnd`的代码：

```
004AF310: 58               pop eax ＝＝》取出hwnd
004AF311: A3CAF24A00       mov [004AF2CA],eax ＝＝》保存在[004AF2CA]，以便用到的时候在这里取回
004AF316: 50               push eax
004AF317: E8DCFDFFFF       call .0004AF0F8  -----call setmenu
004AF31C: E972DAFEFF       jmp .00049CD93  -----回到程序原来地方
004AF321: 0000             add [eax],al
004AF323: 0000             add [eax],al
004AF325: 0000             add [eax],al
```

现在`hwnd`也有了，而且`DragQueryFile`会返回打开文件的字节，完全和`GetPrivateProfileStringA`返回的一样，也就是在原来`GetPrivateProfileStringA`处改为`DragQueryFile`就OK。那么其他的`PostMessageA`、`lstrcpyA`、`SetCurrentDirectoryA`、`lstrcmpiA`的`call`到哪里去找呢？如果能看到`import`表就直接从`import`表找，但是这个脱壳版的无`import`表真是麻烦。没有办法，下断点一个一个看，先来`bpx postmessagea`：

原程序中断在：

```
00453402: E85DBD0500       Call 004AF164 ＝＝》这个就是call postmessagea
```

其他几个函数如下：

```
postmessagea        00453402: E85DBD0500       Call 004AF164
lstrcpyA            00451C49: E80CD00500       Call 004AEC5A
SetCurrentDirectoryA 0043E679: E8B8050700      Call 004AEC36
lstrcmpiA           0043E468: E815070700      Call 004AEB82
```

好了，现在需要的函数地址也有了，我们开始补丁：

```
004AF330: 3D0A020000       cmp eax,00000020A ;"  ＝＝》比较消息是否是滚轮
004AF335: 7415             je .0004AF34C  ----- (1)＝＝是的话转到滚轮处理
004AF337: 3D33020000       cmp eax,000000233 ;"  3"＝＝》比较消息是否是拖放文件
004AF33C: 7400             je .0004AF33E  ----- (2)＝＝》是的话就转到（拖放文件的处理）
```

先把下面的代码：

```
004AF33C: 7400             je .0004AF33E  ----- (2)＝＝》是的话就转到（拖放文件的处理）
```

改成：

```
004AF33C: 7442             je .0004AF380  -----调到处理拖放的子程序中
```

处理拖放的子程序：

```
\*这段程序就是参照原来处理程序写的
004AF380: FF7608                  push d,[esi][08]
004AF383: 8F05CEF24A00            pop d,[004AF2CE]==>保存拖放的hDrop到[004AF2CE]
004AF389: 6800010000              push 000000100 ;"  "==>缓冲区为100所以不要拖动路径长度大于256的文件哦，可能要出错
004AF38E: 68575A0410              push 010045A57 ;"ZW"＝＝》保存文件名称的地址
004AF393: 6A00                    push 000＝＝》UINT iFile
004AF395: FF35CEF24A00            push d,[004AF2CE]＝＝》push hDrop
004AF39B: E81AFFFFFF              call DragQueryFile ;SHELL32.dll＝＝》得到拖动的文件名
004AF3A0: A3305A0410              mov [10045A30],eax＝＝》返回的字节数保存，以下都是模仿原来程序的动作，上面已经有分析，我就不详细的说明了
004AF3A5: 68575A0410              push 010045A57 ;"ZW"
004AF3AA: 68455A0410              push 010045A45 ;"ZE"
004AF3AF: E8CEF7FFFF              call .0004AEB82  ----- (2)＝＝》call lstrcmpiA
004AF3B4: 0BC0                    or eax,eax
004AF3B6: 0F846A32FAFF            je .000452626  ----- (3)＝＝》如果为零就直接退出，原动作是直接ret，我们就直接返回
004AF3BC: 6660                    pusha＝＝》保存所有的寄存器，因为下面要用到esi等，所以现在先保存之
004AF3BE: BF575A0410              mov edi,010045A57 ;"ZW"
004AF3C3: 8B0D305A0410            mov ecx,[10045A30]
004AF3C9: 8BF7                    mov esi,edi
004AF3CB: 03F9                    add edi,ecx
004AF3CD: FD                      std
004AF3CE: B05C                    mov al,05C ;"\"
004AF3D0: F2AE                    repne scasb
004AF3D2: 85C9                    test ecx,ecx
004AF3D4: 741D                    je .0004AF3F3  ----- (4)
004AF3D6: 83C702                  add edi,002 ;""
004AF3D9: C647FF00                mov b,[edi][-01],000 ;" "
004AF3DD: 3BFE                    cmp edi,esi
004AF3DF: 7412                    je .0004AF3F3  ----- (1)
004AF3E1: 56                      push esi
004AF3E2: E84FF8FFFF              call .0004AEC36  ----- (2)就是call SetCurrentDirectoryA
004AF3E7: 57                      push edi
004AF3E8: 68575A0410              push 010045A57 ;"ZW"
004AF3ED: 90                      nop
004AF3EE: E867F8FFFF              call .0004AEC5A  ----- (3)就是 call lstrcpyA
004AF3F3: 68575A0410              push 010045A57 ;"ZW"
004AF3F8: 68515E0410              push 010045E51 ;"^Q"
004AF3FD: E858F8FFFF              call .0004AEC5A  ----- (4)就是 call lstrcpyA
004AF402: C605505E041002          mov b,[10045E50],002 ;""
004AF409: 6A00                    push 000
004AF40B: 680C5F0000              push 000005F0C ;"  \_
004AF410: 6811010000              push 000000111 ;"  "
004AF415: A1CAF24A00              mov eax,[004AF2CA]＝＝》这个就是我们在setmenu保存的hwnd
004AF41A: 50                      push eax
004AF41B: E844FDFFFF              call .0004AF164  ----- (5)就是 call postmessagea
004AF420: FF35CEF24A00            push d,[004AF2CE]＝＝》这个是保存的 hDrop
004AF426: E895FEFFFF              call DragFinish ;SHELL32.dll＝＝》拖放处理完后要释放hDrop，调用DragFinish
004AF42B: 6661                    popa    恢复堆栈
004AF42D: E9F431FAFF              jmp .000452626  ----- (7)跳回原处理地方
```

好了，到现在为止，我们DIY加强版的`w32asm`已经出来了，大家享受一下劳动成果吧。如果有兴趣的话，还可以给`w32asm`做很多改造，使之更加适应于我们自己的工作习惯。我已经把方法和原理都教给大家了，有谁做出更加power的版本，记得给我一份哦。

此程序在98下测试成功，2000下我想应该不会成功，因为2000下的拖放函数不同。有兴趣的人可以在里面加上`getversion`函数，判断是98还是2000。如果是2000下，就用另外一套函数实现。我自己很少在2000下使用`w32asm`，所以我就不做了 ：（，就当留给大家的作业吧。

后记：写到这里已经是两手发酸。希望这篇文章成为DIY PE的经典教程。如果看雪出一本DIY PE的书，是不是考虑选上我这篇文章？：）

你的朋友：pll621

2002.8.24
