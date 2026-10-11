
# Oicq 广告条

> 原文：[`www.pediy.com/kssd/pediy03/forum384.htm`](https://www.pediy.com/kssd/pediy03/forum384.htm)

用 `execope` 等资源编辑工具打开 `oicq.exe`,再把显示广告条的对应控件移到不可见的地方(不知为什么不能删除),保存.

运行 `oicq` 会提示文件被改变,只要反汇编 `oicq`,查找对应提示语句,再改判断文件有没有改变的跳转语句

```
:00443E18 33FF                 xor edi, edi
:00443E1A 397DDC               cmp dword ptr [ebp-24], edi
:00443E1D 755C                 jne 00443E7B  ------>下面是提示,只要改为 jmp 就行了,75 改为 EB.
```

修改完成. `4dd466d0e1e0cc123afbe504da2462ca`
