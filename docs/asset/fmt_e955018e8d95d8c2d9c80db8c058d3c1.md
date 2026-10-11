
# PYdotDLL: A Simple Python Hook Engine (Update 2010.9.28)

> 原文：[`www.pediy.com/kssd/pediy11/120438.html`](https://www.pediy.com/kssd/pediy11/120438.html)

## 简介

这是一个简单的 PYTHON HOOK 引擎。因为 PYTHON 是一个高效率的脚本语言，所以借助这个你能快速高效地 HOOK 所要分析的目标函数，进而分析各种你需要的内容。具体如何使用，还得看你的意愿和想法。

## 更新日志

**update 2010.9.17:**

- 增加了一个 `HOOKDecorator`，使用更简单。
- p.s. 还并不明白 `Decorator` 是为什么而设计，所以也不知道在这里用 `Decorator` 是否合适。等有机会弄清楚了，如果不合适的话就去掉。

**update 2010.9.28:**

- 把 `HookProc` 代码放上去了。并且增加了一个 `IATHOOK` 和 `EATHOOK`。
- 以后不会再在这儿更新了。
- 新地址：`http://code.google.com/p/pydotdll/`
- 欢迎加入 ＝ ＝#

## 示例：Hook `LoadLibraryA`

```python
@HOOKDecorator('kernel32.LoadLibraryA', dll=ctypes.c_char_p)
def LoadLibraryA(dll):
    handle = OriginFuncion(dll)
    print '%s: 0x%08X' % (os.path.basename(dll), handle)
    return handle
```

## 示例：Hook `MessageBoxA`

```python
@HOOKDecorator('user32.MessageBoxA', title=ctypes.c_char_p)
def MessageBoxA(handle, msg, title, type):
    return OriginFuncion(handle, msg, 'PYdotDLL!!', type)
```

## `TestLoad` 的代码 (FASM)

```asm
format PE console
include "%include%/win32ax.inc"

xx:
i MessageBoxA, 0, 'Hello guy !', 'i am title .', 0
ret
entry $
i LoadLibraryA, 'PYdotDLL.dll'
i CreateThread, 0, 0, xx, 0, 0, 0
ret
```

![](img/2ba46d0b229f6c2132b5982375591a91.jpg)

## 示例：Hook `InternetConnectA`

Redirect `www.baidu.com` to `bbs.pediy.com`

```python
@HOOKDecorator('wininet.InternetConnectA', servername=ctypes.c_char_p)
def InternetConnectA(handle, servername, serverport, username, password, service, flags, context):
    if servername.lower() == 'www.baidu.com':
        servername = 'bbs.pediy.com'
        print 'Redirect It ! : www.baidu.com -> bbs.pediy.com'
        sys.stdout.flush()
    return OriginFuncion(handle, servername, serverport, username, password, service, flags, context)
```

![](img/55a02607bb757ee61ef5408e646bb320.jpg)

## 下载

你可以在这里下载所有代码：`PYdotDLL-0.7.6.zip`。

**上传的附件**

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) | `PYdotDLL.zip` |
| :--- | :--- |
