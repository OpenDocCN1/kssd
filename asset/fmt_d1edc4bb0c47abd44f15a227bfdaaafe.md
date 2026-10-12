
# Ashampoo Magical Snap 2.31 算法详细分析

> 原文：[`www.pediy.com/kssd/pediy10/77691.html`](https://www.pediy.com/kssd/pediy10/77691.html)

**【破文标题】** `Ashampoo Magical Snap 2.31` 算法分析

**【破文作者】** `Playboysen`

**【作者邮箱】** `playboysen@126.com`

**【破解工具】** `PEiD`, `OD`

**【破解平台】** `Windows XP SP2`

**【软件名称】** `Ashampoo Magical Snap 2.31(20081105)`

**【软件大小】** `12.7 MB`

**【软件授权】** 共享版($14.99)

**【软件语言】** 英文

**【原版下载】** `http://www.ashampoo.com/dl/0224/ashampoo_magicalsnap2_se.exe`

**【保护方式】** 注册码

**【软件简介】** `Magical Snap` 是一款专业的屏幕截图工具。它添加了编辑工具和效果工具还有非常华丽的用户界面。特点：3 种不同的抓图模式；多张抓图；紧密结合--支持全屏抓图；抓图编辑工具；高质量效果图容易使用的鼠标工具。支持抓取任意形状、滚动窗口等。

**【破解声明】** 一点心得，愿与大家分享 `o(∩_∩)o` 版权所有，转载注明作者！

**【破解内容】**

初步踩点：软件无壳，`VC7.0` 编译，单一注册码保护，注册有错误提示，未注册所抓图片有文字水印。

假设注册码为 `AMASAo-77nhEB-96HD2N`（这里说明一下，注册码并不是一开始就假设成这样的，我一开始粗略跟踪用的假码是 `playboysen963852`，跟踪一遍将假码修正为 `playbo-ysenhu-an9638`，再跟踪一遍修正假码为 `AMASbo-ysenhu-an9638`，再跟踪一次将假码修正为 `AMASAo-77enhu-an9638` 的，第五遍跟踪才将注册码修正成这样的(当然其实这已经被“修正”为真注册码了`^\_^`)自己跟踪时可随便输入一个假码，自由练习）

尝试 `MessageBoxA`、`MessageBoxW`、`GetDlgItemTextA` 断错误提示无果，最终发现软件使用的是 `GetDlgItemTextW` 断点，汗~~~

```
004455DE   > \53               push ebx                                  ;  Case 1 of switch 0044559C
004455DF   .  55              push ebp
004455E0   .  8BAC24 380A0000 mov ebp,dword ptr ss:[esp+A38]
004455E7   .  57              push edi
004455E8   .  6A 20           push 20                                   ; /Count = 20 (32.)
004455EA   .  8D5424 28       lea edx,dword ptr ss:[esp+28]             ; |
004455EE   .  52              push edx                                  ; |Buffer
004455EF   .  68 F1030000     push 3F1                                  ; |ControlID = 3F1 (1009.)
004455F4   .  55              push ebp                                  ; |hWnd
004455F5   .  FF15 58144800   call dword ptr ds:[<&USER32.GetDlgItemTextW>]  ; \GetDlgItemTextW
004455FB   .  8D4424 24       lea eax,dword ptr ss:[esp+24]             ;  注册码出现
004455FF   .  50              push eax                                  ; /Arg1
00445600   .  8D4C24 18       lea ecx,dword ptr ss:[esp+18]             ; |
00445604   .  E8 47DAFBFF     call ashsnap.00403050                     ; \关键处，F7 进入
00445609   .  68 4C774E00     push ashsnap.004E774C                     ; /Arg1 = 004E774C
0044560E   .  8D4C24 20       lea ecx,dword ptr ss:[esp+20]             ; |
00445612   .  C78424 380A0000>mov dword ptr ss:[esp+A38],0             ; |
0044561D   .  E8 2EDAFBFF     call ashsnap.00403050                     ; \同上，关键
```

跟入 `00445604` 处关键算法 `call`

代码:

```
00403050  /$  83EC 40         sub esp,40
00403053  |.  8B4424 44       mov eax,dword ptr ss:[esp+44]
......
00403068  |.  C706 00000000   mov dword ptr ds:[esi],0
0040306E  |.  E8 0DF50200     call ashsnap.00432580                     ;  kernel32.WideCharToMultiByte
00403073  |.  8B15 98204800   mov edx,dword ptr ds:[482098]
......
004030C3  |.  52              push edx
004030C4  |.  E8 573A0500     call ashsnap.00456B20                     ;  关键 call,F7 进入
004030C9  |.  83C4 20         add esp,20
```

跟入 `004030C4` 关键 `call`

代码:

```
00456B20  /$  8B4424 14       mov eax,dword ptr ss:[esp+14]
00456B24  |.  8B4C24 10       mov ecx,dword ptr ss:[esp+10]
......
00456B3F  |.  55              push ebp
00456B40  |.  E8 EBFDFFFF     call ashsnap.00456930                     ;  关键 call,F7 进入
00456B45  |.  83C4 14         add esp,14
00456B48  |.  84C0            test al,al
00456B4A  |.  74 20           je short ashsnap.00456B6C
```

这里提醒大家，分析算法的时候，在比较关键的地方遇到 `call` 应该先进入大致浏览一下，否则你很可能错过关键处，导致你接下去无从下手。

好，我们接着跟入 `00456B40` 关键 `call`

代码:
