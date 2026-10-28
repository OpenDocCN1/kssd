
`jmp d,[004D4B24]`

```asm
004AF2A2: FF252C4B4D00                jmp d,[004D4B2C]
004AF2A8: FF25304B4D00                jmp d,[004D4B30]
004AF2AE: FF25344B4D00                jmp d,[004D4B34]
```

```asm
004AF2B4: FF253B805500                jmp DragAcceptFiles ;SHELL32.dll
```

看到我们构造的函数了么，真是happy啊。

```asm
004AF2BA: FF253F805500                jmp DragQueryFile ;SHELL32.dll
004AF2C0: FF2543805500                jmp DragFinish ;SHELL32.dll
004AF2C6: 0000                        add [eax],al
004AF2C8: 0000                        add [eax],al
```

现在记下`004AF2B4`、`004AF2BA`、`004AF2C0`这三个值。以后`call 004AF2B4`就是`call DragAcceptFiles`了，调用`DragQueryFile`也就是`call 004AF2BA`了，以此类推。

函数构造完成第一个条件也就满足了，开始我们第二个条件`hwnd`的寻找吧。

## 寻找hwnd打补丁篇

如何找到`hwnd`值呢，对windows编程熟悉的人（你不熟悉就慢慢学，总有一天会熟悉的）知道`GetWindowRect`函数也需要一个`hwnd`值，我们截获`GetWindowRect`函数，保存这个`hwnd`，再使用这个`hwnd`来`DragAcceptFiles`不就ok。好确定思路，打开`trw`然后`bpx getwindowsrect`。

启动`w32asm`发现其在这里调用`GetWindowRect`：

```asm
0167:00492941 52                    PUSH EDX         ; 这个是一个rect的struct的point，也就是一个矩阵结构的指针
0167:00492942 FF700C                PUSH DWORD [EAX+0C] ; 这里就是我们需要的hwnd
0167:00492945 E810C90100            CALL USER32!GetWindowRect ; 调用这个函数了也就是call .0004AF25A
0167:0049294A EB1B                  JMP SHORT 00492967
0167:0049294C 8B582C                MOV EBX,[EAX+2C]
```

程序启动调用了很多次`GetWindowRect`函数，每次`hwnd`都不同，这是因为`w32asm`窗口分为好几个，`edit`窗口，`menu`窗口，`toolbar`，状态栏等等。我们不管它，把每个窗口的`DragAcceptFiles`值都设置为`ture`就行了，管它有多少个呢，有多少个，就`ture`多少个，个个都能接受拖放，想往那里拖就往那里拖。

把这句：

```asm
0167:00492945 E810C90100      CALL USER32!GetWindowRect
```

改为：

```asm
00492945: E996C90100          jmp .0004AF2E0
```

这里是我们自己的补丁程序了。

自己补丁的地方：

```asm
004AF2E0: 58                  pop eax         ; 取出压入堆栈的hwnd
004AF2E1: A3C6F24A00          mov [004AF2C6],eax ; 保存到一个地址，我选择的是004AF2C6
004AF2E6: 50                  push eax        ; 压hwnd入堆栈
004AF2E7: E86EFFFFFF          call .0004AF25A ; CALL USER32!GetWindowRect
004AF2EC: 6A01                push 001        ; 传递ture
004AF2EE: FF35C6F24A00        push d,[004AF2C6] ; 传递hwnd
004AF2F4: E8BBFFFFFF          call DragAcceptFiles ;SHELL32.dll ; 也就是call 004AF2B4我们在上面构造的地方
004AF2F9: E96936FEFF          jmp .000492967  ; 回到程序原来的地方
```

现在打开补丁完的程序，拖个文件试试，是不是图标已经变了？你说，变是变了，怎么没有反应啊？还没有加入事件怎么会有反应？真是的，路要一步一步的走，不要太急。

现在开始我们加入事件的过程了。首先找到接受消息的地方，加入判断拖放消息的代码。上面已经说过了在：

```asm
:0045261E 8B4604                mov eax, dword ptr [esi+04] ; 这里是函数参数的传递，esi+04就是msg的unit
:00452621 3D21010000            cmp eax, 00000121 ; 这里eax就是windows消息的代码
```

而且

```asm
:00452621 3D21010000            cmp eax, 00000121
```

已经改为：

```asm
00452621: E90ACD0500            jmp .0004AF330
```

也就是消息处理已经在我们自己的补丁地方了。

```asm
004AF330: 3D0A020000            cmp eax,00000020A ; 比较消息是否是滚轮
004AF335: 7415                  je .0004AF34C     ; 是的话转到滚轮处理
004AF337: 3D33020000            cmp eax,000000233 ; 比较消息是否是拖放文件
004AF33C: 7400                  je .0004AF33E     ; 是的话就转到拖放文件的处理
```

好，现在我们可以把`004AF33C: 7400 je .0004AF33E`转到我们自己拖放处理的地方了。

现在程序也能判断拖放消息，我们就需要根据这个消息做相关的动作了，也就是需要做打开这个拖放文件并且反汇编它。

学习到这里，大家是不是已经比较有思路了。首先我们必须得到拖放文件的文件名，这就需要`DragQueryFile`函数了。

`The DragQueryFile function retrieves the filenames of dropped files.`

```c
UINT DragQueryFile(
    HDROP hDrop,    // handle to structure for dropped files ; 拖放的hdrop
    UINT iFile,     // index of file to query ; 为0就可以了
    LPTSTR lpszFile, // buffer for returned filename ; 放文件名的地址
    UINT cch        // size of buffer for filename ; buffer的尺寸
);
```

我们看看这个函数需要什么参数。现在需要找的就是`hdrop`这个类似于windows窗口的`hwnd`，用来唯一标识拖放的，这个也就是`msg`的子参数。

```asm
:0045261E 8B4604                mov eax, dword ptr [esi+04] ; 这里是函数参数的传递，esi+04就是msg的unit
```

`esi+04`是消息`unit`，那么这个子参数就在`esi+08`处。

好了相关的参数都搞定了，下一步就是看看`w32asm`怎么打开一个根据文件名打开文件的动作的。从那里下手呢？从`disassembler`的`open file`下手？这个会弹出一个框子出来，不好。那就从`recent files`下手，`recent files`可以打开你最近打开的文件。好，我们看看它是怎么打开的。

（跟踪这个动作很复杂，我要详细的说，可能打一天的字也打不完，我打字到现在已经很累了，大家就将就一下，我仅仅讲跟踪的关键地方）

在消息判断（消息是`111 wm_command`）的地方下断点，然后点击`recent files`的按钮，发现其子参量为`bd02`，然后又调用自己，这时候的子参量变成`0c5f`。当你点击`open file`时候发现子参量也是`0c5f`，看来是在点击`recent files`文件的时候会`post`一个`0c5f`消息回来。那么`recent`里面的最近打开的文件是存在那个地方呢？经过跟踪可以发现是存在`c:\windows\w32dasm8.ini`文件里面。

windows用来打开`ini`文件的api是`GetPrivateProfileString`，下`bpx GetPrivateProfileString`这个断点，然后点击`recent`里面的最近打开的文件，程序中断在这里（我们就来分析这里是什么意思）：

```asm
:1004310C 55                    push ebp
:1004310D 8BEC                  mov ebp, esp ; 这是vc编程传递参数的形式，用vc都是这样，和编译器有关系
:1004310F 53                    push ebx    ; 这里传递的参数是窗口的hwnd
:10043110 56                    push esi
:10043111 8B450C                mov eax, dword ptr [ebp+0C] ; 这里是消息的子参数，这里是最近五个文件的id
:10043114 2DBD020000            sub eax, 000002BD
:10043119 D1E0                  shl eax, 1
; Possible StringData Ref from Data Obj ->"12345"
:1004311B BB3B5A0410            mov ebx, 10045A3B
:10043120 03D8                  add ebx, eax ; 把eax做转换变成了table表，就知道是点击了那个文件，[ebx]会变成1-5之间
; Possible StringData Ref from Data Obj ->"W32dasm8.ini"
:10043122 684A5A0410            push 10045A4A ; 参数，文件名
:10043127 6880000000            push 00000080 ; buffer size
:1004312C 68575A0410            push 10045A57 ; 读入内容地址指针
; Possible StringData Ref from Data Obj ->"NULL"
:10043131 68455A0410            push 10045A45 ; 缺省字符串的指针
:10043136 53                    push ebx      ; key name
; Possible StringData Ref from Data Obj ->"RECENT"
:10043137 68345A0410            push 10045A34 ; 段名
; Reference To: KERNEL32.GetPrivateProfileStringA, Ord:0125h
:1004313C E8AB0B0000            Call 10043CEC ; 得到key name下的内容
:10043141 A3305A0410            mov dword ptr [10045A30], eax ; 返回的到字节
:10043146 68575A0410            push 10045A57
; Possible StringData Ref from Data Obj ->"NULL"
:1004314B 68455A0410            push 10045A45
; Reference To: KERNEL32.lstrcmpiA, Ord:02D9h
:10043150 E8F70B0000            Call 10043D4C ; 比较上面两个字符串(这个比较的作用是看是不是要调整最近打开的文件的顺序）
:10043155 0BC0                  or eax, eax
:10043157 745E                  je 100431B7
:10043159 BF575A0410            mov edi, 10045A57
:1004315E 8B0D305A0410          mov ecx, dword ptr [10045A30]
:10043164 8BF7                  mov esi, edi
:10043166 03F9                  add edi, ecx
:10043168 FD                    std
:10043169 B05C                  mov al, 5C
:1004316B F2                    repnz
:1004316C AE                    scasb
:1004316D 85C9                  test ecx, ecx
:1004316F 741C                  je 1004318D
:10043171 83C702                add edi, 00000002
:10043174 C647FF00              mov [edi-01], 00
:10043178 3BFE                  cmp edi, esi
:1004317A 7411                  je 1004318D ; 这一段的作用是把d:\1\hiew.exe变成d:\1的形式，为SetCurrentDirectoryA做准备
:1004317C 56                    push esi   ; esi的地址就是X:\XXX\的形式了
; Reference To: KERNEL32.SetCurrentDirectoryA, Ord:023Eh ; 设置现在的路径为X:\XXX\
:1004317D E8A60B0000            Call 10043D28
:10043182 57                    push edi
:10043183 68575A0410            push 10045A57
; Reference To: KERNEL32.lstrcpyA, Ord:02DCh
:10043188 E8C50B0000            Call 10043D52
; Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
; :1004316F(C), :1004317A(C)
:1004318D 68575A0410            push 10045A57
:10043192 68515E0410            push 10045E51
; Reference To: KERNEL32.lstrcpyA, Ord:02DCh
:10043197 E8B60B0000            Call 10043D52 ; 上面一段的作用是调整最近文件打开的顺序
:1004319C C605505E041002        mov byte ptr [10045E50], 02 ; 这个可是标志位哦，如果为零的话，那就会弹出那个打开文件的窗口
:100431A3 6A00                  push 00000000
; Possible Ref to Menu: MenuID_00CB, Item: "Open File to Disassemble.."
:100431A5 680C5F0000            push 00005F0C
:100431AA 6811010000            push 00000111
:100431AF FF7508                push [ebp+08] ; 窗口的hwnd
; Reference To: USER32.PostMessageA, Ord:01DBh
:100431B2 E8E70A0000            Call 10043C9E ; 我果然没有猜错，就是post一个点击open file消息给自己
; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :10043157(C)
:100431B7 5E
```
