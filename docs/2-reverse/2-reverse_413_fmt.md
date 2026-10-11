
# IDA Pro 处理按序号的导入函数

> 原文：[`www.pediy.com/kssd/pediy12/132400.html`](https://www.pediy.com/kssd/pediy12/132400.html)

新的 `mfc90u` 是按照序号导出,用 IDA Pro 加载的时候显示如下形式：

`mfc90u_6167`

这时候用 `flair` 是无效的,`flair` 针对的是静态连接.动态连接的信息可以通过创建立 ids 文件来解决.需要用到 IDA Pro 提供的 `idsutils`，找到

`mfc90u.lib`

`ar2idt mfc90u.lib`

`zipids mfc90u.idt`

就可以得到 `mfc90u.ids`,将其放到 ida pro 的 ids 目录.如果没有自动加载就手动加载一下.出来的结果是：

`mfc90u_6167     proc near                 ; CODE XREF: sub_413860+1Ep`

`.text:008F0A06                     jmp      ds:?SetAtGrow@CDWordArray@@QAEXHK@Z ; CDWordArray::SetAtGrow(int,ulong)`

这样的形式.输入本身已经修正掉,但是外面还有个跳转的壳函数.在 ida pro 中设置名字解码显示为注释的情况下（默认）,我们只需要得到 `jmp` 指令后面的注释就行了.使用一个小脚本来完成：

代码:

```
import string
for i in range(250,6808):
  ea = LocByName("mfc90u\_" + str(i))
  if ea != BADADDR:
    dis = GetDisasm(ea)
    dis = dis[dis.rfind(";") + 2:]
    dis = dis.replace(",", "\_")
    dis = dis.replace("(", "\_P\_")
    dis = dis.replace(")", "\_P")
    dis = "MFC90\_" + dis
    print "make name :",dis
    print
    MakeNameEx(ea,dis,SN_PUBLIC)
```

最终效果：

`text:00413937 mov      ecx, esi`

`.text:00413939 call     MFC90_CDWordArray__SetAtGrow_P_int_ulong_P`

`.text:0041393E mov      eax, [esi+8]`

`.text:00413941 push     0`

`.text:00413943 push     eax`

`.text:00413944 mov      ecx, esi`

`.text:00413946 call     MFC90_CDWordArray__SetAtGrow_P_int_ulong_P`

可以用,十分感谢共享知识

同时附上工具,省的到处找

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `idsutil3.rar` |
| :--- | :--- |
