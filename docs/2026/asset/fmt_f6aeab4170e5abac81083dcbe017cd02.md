
```
BE 29 38 77 52 28 38 77 8D 99 39 77 E3 2C 38 77 11 99 38 77 A0 72 37 77 E0 72 37 77 76 FB 39 77

38 9A 38 77 80 FA 34 77 18 EA 77 75 A7 56 77 75 A3 C9 76 75 6B BE 76 75 9F 97 79 75 8B C9 76 75

B3 C9 76 75 B5 64 79 75 ED 63 79 75 EA FE 76 75 8E 17 7C 75 A5 16 7C 75 10 1E 73 75 8D FE 77 75

30 DF 77 75 8F 90 78 75 DA 7F 77 75 AB F6 77 75 51 08 79 75 50 D9 77 75 C5 2D 77 75 A3 DA 77 75

44 CE 77 75 E2 BD 78 75 0A D9 77 75 77 90 78 75 A6 55 78 75 7C 6D 78 75 AA F0 77 75 64 6D 78 75

91 CA 78 75 72 CA 78 75 6C 6C 78 75 A1 90 78 75 10 F9 77 75 B4 D9 77 75 33 F9 77 75 11 54 78 75

60 C5 77 75 C0 C5 77 75 90 C5 77 75 24 F1 77 75 95 2D 77 75 C6 D9 77 75 65 D9 77 75 40 C4 77 75

D2 C5 77 75 91 9F 80 7C 15 DE 77 75 5F 90 78 75 CA 45 77 75 BB DA 77 75 26 C4 77 75 CD 6C 78 75

EA C5 77 75 44 54 78 75 B7 F0 77 75 20 FB 78 75 E6 54 78 75 3C 8A 83 7C 7B B5 76 75 00 00 00 00

4D 30 86 75 89 EA 8A 75 26 A2 85 75 8B 42 88 75 2B 0C 88 75 30 AD 85 75 C7 52 86 75 CF 64 86 75

73 3B 88 75 49 77 87 75 B8 CF 89 75 18 83 85 75 00 00 00 00 6C F5 00 74 0A 40 FF 73
```

## 八、脱壳后的程序标题栏和关于栏有乱码，带壳运行目标`extdfx169.exe`，对比后更改如下：

### 1、about（这一步在XP下处理的）

```
00401931     8B15 0CE04100            mov edx,dword ptr ds:[<&kernel32.SetLas>; kernel32.SetLastError

改成：

00401931     BA B0814000              mov edx,dumped_e.004081B0

00401936     90                       nop
```

顺便用`ResEdit-x86`汉化下主界面，增加最小化按钮，在about还更改了一句：请使用数字小键盘！

### 2、标题栏：Win7下标题为'閩?悙悙惛榲u脨悙悙?U嬱Q婨?..'（问题越明显越好找，XP下反而不好找）

Win7下OD载入`dumped\_extdfx169\_win7.exe`后，F8运行N步，我这里堆栈栏有：

```
0012F7A4   002B09F0  |hOwner = 002B09F0 ('閩?悙悙惛榲u脨悙悙?U嬱Q婨?..',class='#32770')
```

用破解计算器找到ASCII码：

```
悙悙惛榲=90 90 90 90 90 B8 98 76

u脨悙悙=75 C3 90 90 90 90 90

U嬱Q婨=FF 55 8B EC 51 8B
```

在M镜像二进制查找90 90 90 90 90 B8 98 76，来到：

```
7577CF85  90 90 90 90 90 90 90 90 90 90 90 E9 7D F6 FF FF  悙悙悙悙悙閩?

7577CF95  90 90 90 90 90 B8 98 76 7F 75 C3 90 90 90 90 90  悙悙惛榲u脨悙悙

7577CFA5  8B FF 55 8B EC 51 8B 45 10 8B 4D 08 53 8B 5D 14  ?U嬱Q婨婱S媇
```

看来标题栏引用的地址是：`7577CF90`（这可是API地址啊！） 高低位转换成`90CF7775`

在M镜像二进制查找`90CF7775`，得到

```
0041E008 >90 CF 77 75
```

反汇编窗口转到`0041E008`，却是

```
0041E008 <>  90                       nop

0041E009     CF                       iretd

0041E00A     77 75                    ja short dumped_e.0041E081
```

不管他，仍然`HR 0041E008`，重启OD连续F9，停在

```
004018CE     50                       push eax ; kernel32.GetLastError

eax=7577CF90 (kernel32.GetLastError)
```

OD带壳运行目标`extdfx169.exe`,此处却是

```
004018CE     50                       push eax ; extdfx16.00408214

eax=00408214 (extdfx16.00408214), ASCII "Trainer +6"
```

直接修改`dumped\_extdfx169\_win7.exe`相应堆栈值为`00408214`，运行后正常了！

修改如下：

```
004018C9     A1 08E04100              mov eax,dword ptr ds:[<&kernel32.GetLastError>]

改成

004018C9     B8 14824000              mov eax,dumped_e.00408214  ; ASCII "Trainer +6"
```

### 后来才知道也可以这样查找：

#### 1、在反汇编窗口查找所有常量=`0041E008`，找到这些：

```
参考位于 dumped_e:到常量41E008

地址       反汇编                                           注释

004018C9   mov eax,dword ptr ds:[<&kernel32.GetLastError>]  [0041E008]=7577CF90

00402764   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00402CE6   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00404824   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00405E46   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00406228   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

0040678C   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00406819   call dword ptr ds:[<&kernel32.GetLastError>]     kernel32.GetLastError

00407B63   push dumped_e.00401A6C                           （初始 CPU 选择）
```

#### 2、第一句就是所求正解

## 九、思考：

脱壳后的程序为什么访问`0041E008`的数据？`ds:[0041E008]`的数据又如何由`00408214`变成了`7577CF90`？

感觉程序好像错位了，是反Dump造成的吗？那功能还有保障吗？

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-2-19 11:19

被ejamse编辑

，原因： 改正错误

上传的附件：

- [extdfx169脱壳.rar](javascript:void(0);)

  （179.56kb，35次下载）
