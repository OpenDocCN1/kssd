
# 用 PYTHON 轻松 DIY，给 IE6 添加“隐身”功能

> 原文：[`www.pediy.com/kssd/pediy11/120883.html`](https://www.pediy.com/kssd/pediy11/120883.html)

这里说的“隐身”是指的 `Chrome` 和 `Firefox` 的隐身功能。当然，其实这个词并不准确，不过暂且凑合着吧。

浏览器隐身功能的实现好像是在退出的时候自动删除当前 `SESSION` 所有的 `COOKIES`。另外一种实现则是给当前 `session` 指定一个随机的临时的 `cookies` 存放文件夹。

这里我们选择第二种方法。

然后我们知道 `IE` 在启动的时候会读取注册表，从中获取 `COOKIES` 文件夹存放的位置。所以我们只要修改 `IE` 的这个读取过程返回自己定义的值（自己定义的用来保存 `COOKIES` 随机文件夹）就行了。

`IE` 读取的路径和值如下：

> 引用: `PATH -- HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`
>
> `VALUE -- Cookies`

为了方便我们直接拦截读取 `Value="Cookies"` 的过程 -- `Hook` `RegQueryValueEx`

**这里我们使用 `PYdotDLL` 来实现 `HOOK`。嗯，事实上这篇文章也是因为 `PYdotDLL` 而写。

(`PYdotDLL --` [`http://bbs.pediy.com/showthread.php?t=120438`](http://bbs.pediy.com/showthread.php?t=120438))

为了让 `IE` 加载 `PYdotDLL` 我们先用 `LordPE` 修改 `IE` 程序（当然你可以用你喜欢的方式，譬如 `LPK` 补丁），如下图：

![](img/6f1f7f3d08baa5b0140f78c688de77f0.jpg)

接下来我们就可以安心写自己的代码了。

我们先 `HOOK` `RegQueryValueExA`

然后打印出所有的 `lpValueName`，查看下面是否有“`Cookies`”， 如果有则我们 `HOOK` 的函数没有错误。

下面是代码

```
@HOOKDecorator('advapi32.RegQueryValueExA', lpValueName=ctypes.c_char_p)
def RegQueryValueExA(hKey, lpValueName, lpReserved, lpType, lpData, lpcbData):
    print lpValueName
    return OriginFunction(hKey, lpValueName, lpReserved, lpType, lpData, lpcbData)
```

我们可以在结果中看到 确实截取到了读取“`Cookies`”的过程，如下图。

![](img/bdf5929bb6a5cff6875246f85557a0eb.jpg)

那么接下我们直接在 `HOOK` 过程中返回一个新的临时文件夹即可达到我们的目的。

下面是代码：

```
newpath = tempfile.mkdtemp('-cookies')
def Filter_QueryValueEx(lpValueName, lpData, lpcbData, isWchar):
    if lpValueName and lpValueName.lower() == 'cookies':
        needsize = ctypes.cast(lpcbData, ctypes.POINTER(ctypes.c_int32)).contents
        if lpData == 0:
            needsize.value = 1024*2
        else:
            tmptype = ctypes.ARRAY((isWchar and ctypes.c_wchar or ctypes.c_char), 1024*2)
            path = ctypes.cast(lpData, ctypes.POINTER(tmptype)).contents
            path.value = newpath
            needsize.value = isWchar and (2*len(path.value)) or (len(path.value))
```

现在我们的 `IE` 也有“隐身”功能了，我打开 4 个 `IE`，每个 `IE` 使用一个单独的 `COOKIE` 文件夹，如下图。

![](img/c58d1bfe22683e17812a212222364542.jpg)

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `incognito.zip`  

 |
