
# `memcpy`在Unicode，ASCII中的用法区别

> 原文：[`www.pediy.com/kssd/pediy12/143435.html`](https://www.pediy.com/kssd/pediy12/143435.html)

![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

在VS2010开发中，用到`memcpy`函数，虽然知道与在VC6.0中的用法有区别，但是还是碰了不少壁，在此总结一番，详情见Windows核心编程。

1. 在VC6.0中，默认的是ASCII字符集，因此以下函数会正常执行：

    ```
    char str[256];
    CString strTemp = "helloworld"; //在MFC中测试。因此用的是CString
    memcpy(str, strTemp, strTemp.GetLength() + 1); //末尾加一是为了拷贝'\0'。
    ```

    `str`中的值会成为`helloworld`。在内存中占的单元数是11个。

2. 同样的程序在VS2010中会拷贝不成功。

    因为VS2010默认为UNICODE，`memcpy`在拷贝的时候将`str`的两个内存单元用来存放一个字节，也就是要成功保存，占用字节数为`11*2 = 22`个。

    因此程序应为：

    ```
    char str[256];
    CString strTemp = "helloworld";
    memcpy(str, strTemp, strTemp.GetLength() *2 + 2);
    ```

3. 可能你已经想到用`TCHAR`来声明，即如下：

    ```
    TCHAR str[256];
    CString strTemp = "helloworld";
    memcpy(str, strTemp, strTemp.GetLength() + 1);
    ```

    经过测试，这样一样不会成功。

    原因是：`memcpy`函数只对内存单元进行操作，`TCHAR str[256]`，只是多申请了256个单元，总单元数为`256*2`个，即相当于声明为`char str[512]`而已。

4. 综上所述。在ASCII字符集中，`memcpy`函数操作的是将一个字节占用一个内存单元。

    在UNICODE字符集中，占用的是两个内存单元。

    拷贝的时候，要注意第三个参数的使用，必须分配够足够的内存单元。

    只需分配足够的空间即可，主要是第三个参数要正确设置为在UNICODE下所有的内存单元数。一般为ASCII下的两倍。
