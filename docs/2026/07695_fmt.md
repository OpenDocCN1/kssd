
# [旧帖][原创]修改dll导出函数0.00雪花

今天新发现了一种修改dll导出函数的方式，可以混淆导出函数名称，希望对还不知道这个方法的童鞋有点帮助，测试的编译器是VS，其他的编译器为测试。

举例说明，实现并导出一个函数

```c
#ifdef __cplusplus
#define EXPORT extern "C" __declspec (dllexport)
#else
#define EXPORT __declspec (dllexport)
#endif

EXPORT void ExportFunc();
void ExportFunc(){} //这里什么也没做只是为了测试
```

## 1.通常的修改dll导出函数的方式

`MSDN`的介绍 `/EXPORT:entryname[=internalname][,@ordinal[,NONAME]][,DATA]`

```c
#pragma comment(linker,"/EXPORT:Ob688b8bc2a7426de66df8e77b152fc05=_ExportFunc")
```

使用`depends`工具查看效果

![](https://bbs.kanxue.com/upload/attach/201209/491022_tqyo4mcx7umbx94.jpg)

这里共导出了两个函数，名称不同，函数地址一样，起不到混淆的作用

## 2.新发现的方法，使用`#define`重定义函数名称

```c
#define ExportFunc Ob688b8bc2a7426de66df8e77b152fc05
EXPORT void ExportFunc();
```

使用`depends`工具查看效果

![](https://bbs.kanxue.com/upload/attach/201209/491022_dfg5vnixwf6njdc.jpg)

这里我们只看到了`Ob688b8bc2a7426de66df8e77b152fc05`这样一个函数了

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)
  （9.39kb，235次下载）
- [2.jpg](javascript:void(0);)
  （7.58kb，235次下载）
