
# vista sp1 下 listview 选择的一处 bug

> 原文：[`www.pediy.com/kssd/pediy10/72209.html`](https://www.pediy.com/kssd/pediy10/72209.html)

`sdk` 做了一个程序，在一个对话框资源上放置一个 `listview`, `dialogbox`

出对话框来，然后在 `listview` 里面追加一些 `items`, 这时选择一行 `item`,

系统会给予 `ding.wav` 的提示声。

记得系统的 `listview`, 比如桌面，比如文件夹内文件的详细排列信息，

都不会有提示声，更重要的是，`MFC` 的对话框里面放置的 `listview` 也没有声音。

百思不得其解，`baidu` `google` 也得不到答案，于是 `debug` 后发现是 `comctl32`

的一个 `bug`，具体表现在：

`comctl32.dll!_CCPlaySound@4` 这个函数里面，伪代码描述如下

`comctl32.dll` file version `5.82(vista_rtm.061101-2205)`

```
char buf[280];

DWORD retsize = 280;

if (ERROR_SUCCESS == RegQueryValue(HKEY_CURRENT_USER, "AppEvents\\Schemes\\Apps\\.Default\\CCSelect\\.current", buf, &retsize))
{
   LoadLibrary("wimm.dll");

   GetProcAddress("waveOutGetNumDevs");

   GetProcAddress("PlaySoundW");

   call function waveOutGetNumDevs;

   call function PlaySoundW(buf, 0, 1);

}
```

当电脑上存在 `AppEvents\\Schemes\\Apps\\.Default\\CCSelect\\.current` 键值的时候，

并且默认值被删除的情况下，`RegQueryValue` 会返回 `ERROR_SUCCESS`, 并且 `retsize=0`,

`buf` 的内容不做改变。

这时如果 `buf` 的内容无效调用 `PlaySound` 则会导致一个错误，于是系统给予 `ding.wav` 的声音。

当如果恰巧 `buf` 本身就是 `0`，`0` 开头的 `unicode` 串，那么则不会有任何反应。`MFC` 的程序恰巧碰到这一点。

也就是说上面的条件需要改成

```
if (ERROR_SUCCESS == RegQueryValue(HKEY_CURRENT_USER, "AppEvents\\Schemes\\Apps\\.Default\\CCSelect\\.current", buf, &retsize)
    && retsize != 0)
```
