
### 汇编代码示例

```
 PUSH 32 ; 字符串最大长度 PUSH 
EAX ; 字符缓冲区地址 PUSH 000003F4 ; 控制标识 PUSH DWORD PTR [ESI+1C] ; 对话框柄 CALL [USER32!GetDlgItemTextA] 
; 获得字符串
```

### C 语言等效代码

如果认为是 C 语言编译出来的话，这个 `CALL` 应该是这样：

```
 GetDlgItemTextA(hwndDlg, 0x3F4, buffer, 0x32); ^ [ESI+1C] 
^ EAX
```

### 关于 PUSH 操作

`PUSH` 把数据保存在叫做堆栈的地方。每个 `PUSH` 把新的数据放在堆栈的顶部，被调用的子程序就挨个地检查躺在堆栈中的数据，按照定义来使用它们。

### 附录

#### A. 让 SoftICE 载入符号

你可以用 `exp getwindowtext` 命令来检查 `SoftICE` 是否已经为 `GetWindowText` 装入了符号，象这样：

```
:exp getwindowtext
```

如果你没有得到所有 `GetWindowText` 函数的列表，你就得编辑 `\SIW95\WINICE.DAT`，在 "Examples of export symbols that can be included for chicago" 这段文字以后的那些 `'exp='` 的行首去掉 `';'`，为了节省内存，选择最重要的几个就可以了：

```
 kernel32.dll 
user32.dll gdi32.dll
```

编辑完后，重新起动计算机使其生效。

#### B. 函数语法

如果你看看下面的函数声明，对我们上面讲到的函数调用就容易明白了：

```
int GetWindowText(int windowhandle, char *buffer, int maxlen); 
int GetDlgItemText(int dialoghandle, int controlid, char *buffer, int maxlen); 
int GetDlgItemInt(int dialoghandle, int controlid, int *flag, int type);
```

这些函数的详细描述，可参考 Windows/Win32 编程手册。

#### C. 如何与我联系

*   **On IRC (EFNet):** In `#Ucf96`, `#Cracking`
*   **By e-mail:** `edison@ccnux.utm.my` OR `an461165@anon.penet.fi`
*   **On my homepage:** `http://www.geocities.com/SoHo/2680/cracking.html`

---

### 译后记

花了一个下午的时间（上班时间啊！），译完这篇拆解教程。不知大家看不看得懂？`ED!SON` 的文章是很浅显易懂的，我担心会让我毛手毛脚地闹出歧义来，如果真有什么错误的话，帐应该算在我头上（请大家来信指出）。`ED!SON` 的这篇文章对初学者来说非常值得一读，我就是看完这篇文章以后，马上连续 `CRACK` 了几个程序（就这么快、这么简单！）下次我就把 `CRACK WINZIP6.2` 的过程讲讲解吧（`WINZIP6.2` 的注册器已经泛滥成灾了，就当练习练习吧）。欢迎来信交流，你 `CRACK` 了什么程序，别忘了告诉我你用的方法；如果觉得我翻译得还算看得下去话，就给些鼓励吧。

### 联系方式

*   `CFIDO ID`: `Jim Tyan`
*   `EMAIL`: `jimtyan@tasteful.com`

田学军 `8878888` `CALL` `29549` `jimtyan@tasteful.co`

> 4dd466d0e1e0cc123afbe504da2462ca
