
# [原创] Windows 8 新文件系统 Protogon 印象，内部函数列表

Windows 8 还没有公开资料的新文件系统 Protogon，现在已经可以试用了。

格式化硬盘为 Protogon 文件系统的方法是使用管理员命令行：

```
Format E: /fs:protogon /Q
```

把 `E:` 换为指定的盘符。

我手里的是 64 位的 Windows 8，从 `C:\Windows\System32\drivers` 目录拿到 `protogon.sys` 看了一下，内有大量的 NTFS 系统函数，看来这个新文件系统还是以 NTFS 为基础的，不知道新推出的时候是不是直接将 NTFS 文件系统加高一个版本号。具体的还没有细看，猜测是增加与文件搜索有关的内容吧，或者还有增加 SSD 的性能支持。

特有的函数有 `ProtogonFlushIndex`、`ProtogonMoveFile`、`ProtogonCloseIndex`、`ProtogonIsFileOpen` 等等。更详细的见附件。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班，第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [protogon.txt](javascript:void(0;)

  （309.28kb，85 次下载）
