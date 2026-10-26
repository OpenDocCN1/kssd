
分析到这里，你会发现真正影响注册的标志是`SYSTEM\CurrentControlSet\Services\rxvstor\Parameters`里的`info`这个键值，这个就是记录

软件可使用的剩余期限的,按照上面的条件，可以将其修改成`80000000`，即可显示为注册成功~

这个就是程序的奇怪地方，按照软件的提示，重启软件应该会读取之前保存的`KEY`数据，然后再经过一系列的运算判断，然后再修改`INFO`这个键值，但是这里直接读取`info`这个值就进行判断注册与否。

小弟水平有限，并没有参透过中奥妙，还请指教。

`KEY`文件内容，`KeyData`这个数据结构中，为方便，我用一个数组存放。其实从上面算法的过程中猜测，其实使用下面的结构可能会贴切点：

```
typedef struct _KeyData{
            DWORD VersionFlag;
            BYTE UserIdEncryptionData[16];
            BYTE SumFlag;
            BYTE IdFlag;
            WORD NoUse;
            BYTE RemainData[128];
            DWORD EndDword;
            }KEYDATA,*PKEYDATA;
```

因为找不到真正的算法，因此无法写注册机了，毕竟这个软件做的是不错的，真不希望看到它的注册机满天飞~~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
