
# 豪杰最大化按钮无效

> 原文：[`www.pediy.com/kssd/pediy06/pediy6844.htm`](https://www.pediy.com/kssd/pediy06/pediy6844.htm)

**豪杰 3000 最大化按钮无效**

* 作者: `nbw\[NE365\]\[DFCG\]`
* QQ : `343538175`
* [`nboy.cnwlt.com`](http://nboy.cnwlt.com) 很希望和 `Pe Diy` 爱好者交流或合作

上次我们谈了如何把豪杰的屏幕信息修改，这次让 `豪杰 3000` 的最大化按钮无效。

## 原理与实现思路

下面谈一下原理。利用 `SetWindowlongA` 和 `GetWindowLongA`，把豪杰窗体的属性改掉，让其无最大化按钮无效。故此，找个地方把这两个函数插到程序的载入过程便可以了。我们首先需要研究的是这两个函数的使用，和给这两个函数找个地方。经过研究，比较麻烦的是函数的参数需要窗口句柄，好，那先谈谈这个吧。

## 获取窗口句柄 (hwnd)

`豪杰 3000` 的窗口比较特别，含有多个子窗口，我们需要找的是其主窗口的句柄。根据 `pll621` 老大处理 `w32asm` 的时候的说法，`GetWindowRect` 需要窗口句柄，于是就可以借用一下那个参数。于是乎，我打开 `Trw`，`bpx GetWindowRect`，找豪杰打开。靠！不仅频繁被中断，而且最后发现豪杰的加载没有调用这个函数，于是以失败告终。

于是我也不想借了，还是用 `FindWindow` 函数查找吧，`wowocock` 老大也同意这么办。打开 `Pedit`，给豪杰加载这个函数，但是提示没有此函数。查了查 `MSDN` 也没什么结果，看来小弟水平太次，办不了啊！哪位老大要知道还请告之一二。

看来还是要从别的函数那里借来 `hwnd` 用用。我知道 `MessageBoxA` 的第一个参数可以是 `hwnd`，当然也可以不是。还是看看吧，用 `w32Dasm` 打开 `SthSDVD.exe`，查找调用 `MessageBoxA` 的地方，对所有地方的第一个参数研究以下，当然要看有“条件”的了。挑选如下：

```asm
:00424806 8B0DB4094900 mov ecx, dword ptr [004909B4]
:00425083 A1B4094900 mov eax, dword ptr [004909B4]
:00426C53 8B842414010000 mov eax, dword ptr [esp+00000114] ; 最好把这种转换成具体的内存地址
:004297EA 8B442414 mov eax, dword ptr [esp+14]
:00418C20 A1600B4900 mov eax, dword ptr [00490B60]
:00418D7A A1600B4900 mov eax, dword ptr [00490B60]
:00424806 8B0DB4094900 mov ecx, dword ptr [004909B4]
:00425083 A1B4094900 mov eax, dword ptr [004909B4]
```

向 `[004909B4]`，`[00490B60]`，`[004909B4]` 都极有可能是我们要找的存放 `hwnd` 的地方。具体哪个是怎么判定呢？

打开豪杰，用窗口句柄察看工具看豪杰的主窗口 `hwnd=4d8h`（当然每次都不一样），不要关闭豪杰，用 `winhex` 打开 `SthSDVD.exe` 的主内存空间，查看上面那一砣地址，发现 `[4909b4]` 处是 `4d8h`，所以 `[4909b4]` 是主窗口句柄的存放地点。搞了半天，总算把它揪了出来！

## 注入点选择

根据我们改造的功能，必须让豪杰加载的时候就调用我们的函数，比较简单的方式就是把豪杰的入口地址改到我们的程序存放地点，就好象一般的 `PE` 病毒一样。但是要注意，`[4909b4]`（也就是句柄）在程序开始加载的时候并没有被正确装载，所以在 `[4909b4]` 被正确装载以前我们不可使用之。那我们看看在什么时候 `[4909b4]` 被正确装载。你可以用 `TRW` 加载豪杰，`d 4909b4`，然后一步一步调试。我这里图方便，使用 `bpm 4909b4` 命令， 然后加载豪杰，中断在：

```asm
0167:0041d0a0 call user32!createwindowexa
0167:0041d0a6 mov [4909b4], eax
```

上面便是程序确定句柄的地方。原来 `createwindowexa` 是用来返回窗口句柄的啊，惭愧，`Win32` 学好些岂不…...

## 空间分析与代码植入

函数需要的参数搞定了，然后给我们添加代码找地方。我用自己写的“窗口剩余空间查看器”分析 `SthSDVD.exe`，部分结果如下：

| 名称 | RVA | OA | 尺寸 D | 可写否 |
| :--- | :--- | :--- | :--- | :--- |
| `.text` | `00038926` | `00037d26` | `218` | 否 |

看来我们要跳到 `00438926`（`VA`），那里有 `218` 字节的空间等着我们开发，先跳过去再说，用 `Hiew` 打开 `SthSDVD.exe`，将：

```asm
0167:0041d0a0 call user32!createwindowexa
0167:0041d0a6 mov [4909b4], eax
:0041D0AB 8BD8 mov ebx, eax
```

改为：

```asm
* Reference To: USER32.CreateWindowExA, Ord:0055h
|
:0041D0A0 FF15B07C4B00 Call dword ptr [004B7CB0]
:0041D0A6 E97BB80100 jmp 00438926 ; 跳转到新代码的地方，eax 是主窗口句柄
:0041D0AB 8BD8 mov ebx, eax
```

## 新代码逻辑实现

下面研究以下把 `GetWindowLong` 和 `SetWindowLong` 添加到 `438926` 的地方。这 2 个函数的具体使用实例如下（`VB` 伪代码）：

```vb
Private Declare Function GetWindowLong Lib "user32" Alias "GetWindowLongA" (ByVal hwnd As Long, ByVal nIndex As Long) As Long
Private Declare Function SetWindowLong Lib "user32" Alias "SetWindowLongA" (ByVal hwnd As Long, ByVal nIndex As Long, ByVal dwNewLong As Long) As Long

Const GWL_STYLE = (-16)
Const WS_CAPTION = &HC00000
Const WS_MAXIMIZEBOX = &H10000
Const WS_SYSMENU = &H80000

Private Sub Form_Load()
    Dim lhwnd As Long
    lwnd = GetWindowLong(Me.hwnd, GWL_STYLE) ; 这里的 me.hwnd 相当于我们的主窗体句柄
    lwnd = lwnd And Not (WS_MAXIMIZEBOX)
    lwnd = SetWindowLong(Me.hwnd, GWL_STYLE, lwnd)
End Sub
```

看完以后就可以把这两个函数添加到 `:00438926`，这里要注意的是一定要保证堆栈的平衡和参数的正确传递，大家都是高手，我也不罗嗦了，具体如下：

```asm
:00438926 A3B4094900 mov dword ptr [004909B4], eax ; 这是原来程序中需要做的
:0043892B 60 pushad ; 保存原来的运行环境
:0043892C 50 push eax ; hwnd 入栈，后面有用
:0043892D 33DB xor ebx, ebx
:0043892F 83EB10 sub ebx, 00000010 ; ebx=GWL_STYLE
:00438932 53 push ebx
:00438933 50 push eax ; push hwnd

* Reference To: USER32.GetWindowLongA, Ord:013Ah
|
:00438934 FF15747B4B00 Call dword ptr [004B7B74]

:0043893A 5B pop ebx ; ebx=hwnd，请注意来自:0043892C 的入栈
:0043893B B900000100 mov ecx, 00010000
:00438940 F7D1 not ecx
:00438942 23C1 and eax, ecx
:00438944 50 push eax
:00438945 33C0 xor eax, eax
:00438947 83E810 sub eax, 00000010 ; lwnd
:0043894A 50 push eax
:0043894B 53 push ebx ; hwnd

* Reference To: USER32.SetWindowLongA, Ord:021Ah
|
:0043894C FF15707B4B00 Call dword ptr [004B7B70]

:00438952 61 popad ; 恢复原来程序的运行环境
:00438953 E95347FEFF jmp 0041D0AB ; 返回原来程序
```

最后提醒一下在写内存地址的时候要注意区分 `VA`，`RVA` 和文件偏移地址。
