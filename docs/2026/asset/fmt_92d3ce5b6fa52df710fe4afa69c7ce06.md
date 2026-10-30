
本漏洞利用手法有一定的堆布局难度，最后的提权是通过堆溢出覆盖`nss_load_library`函数加载 so 的时候需要用到的结构体`service_user`，覆盖此结构体中的 so 名字符串，这样就可以让程序加载我们指定的 so 文件，从而完成任意代码执行。

我们要做的事情有两个：

1. 搞清楚`nss_load_library`的函数调用流程和相关的数据结构机制
2. `setlocale` 如何通过环境变量 `LC_*` 进行堆布局

漏洞利用关键代码片段：

```
static int
nss_load_library (service_user *ni)
{
  if (ni->library == NULL)
    {
      static name_database default_table;
      ni->library = nss_new_service (service_table ?: &default_table,
                     ni->name);
      if (ni->library == NULL)
    return -1;
    }

  if (ni->library->lib_handle == NULL)
    {
      ··· ···
      __stpcpy (__stpcpy (__stpcpy (__stpcpy (shlib_name,
                          "libnss_"),
                    ni->name),
              ".so"),
        __nss_shlib_revision);

      ni->library->lib_handle = __libc_dlopen (shlib_name);
      ··· ···
      ··· ···
  }
}
```

`ni`为堆上的`service_user`结构体，当`ni->library->lib_handle`为`NULL`时，就会调用`__libc_dlopen`进行 so 装载，只要我们能够通过堆溢出将`ni->library`覆盖为`0`，就会让程序调用`nss_new_service`函数让`ni`结构体重新初始化，刚初始化后的`ni->library->lib_handle`一定为`0`，所以会运行到下面的代码块，执行到`__libc_dlopen`函数。

这里是漏洞利用的核心代码，接下来我们考虑如何实现精准的堆溢出来覆盖`ni`结构体，这就要提到 nss 机制，首先`/etc/`目录下有一个文件`/etc/nsswitch.conf`，来看看里面写了些啥

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_HHZ9RJMSK7TT6GE.jpg)

它规定了程序在查找 so 方法的时候采用何种途径和顺序

然后来看三个结构体的内容
