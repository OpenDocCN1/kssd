
# [原创]CryptoWall勒索病毒分析

> 引用

前几天给的样本，分析完之后，感觉还是比较经典的，且网上并没有相关分析，所以跟大家分析一下我的思路

用PEID查看，并没有加壳，但是竟然有附加数据

![](https://bbs.kanxue.com/upload/attach/201905/830989_T3TFQ5XXCXKGCS2.jpg)

用LoadPe查看导入表信息，明显很奇怪

![](https://bbs.kanxue.com/upload/attach/201905/830989_ESQFEMUGBJDSGEV.jpg)

用火绒剑分析进行行为分析，发现创建了一个跟自己同名的子进程

![](https://bbs.kanxue.com/upload/attach/201905/830989_GRVHPVZ3CT8N785.jpg)

载入Od,在这个call里程序就跑飞了

![](https://bbs.kanxue.com/upload/attach/201905/830989_PZF6NSHJ7AHNB6P.jpg)

跟踪进入，一脸懵\*

![](https://bbs.kanxue.com/upload/attach/201905/830989_427QJ5P5453ENUD.jpg)

全都是这样的代码，既然创建了进程，于是直接下API断点

`CreateProcessW` `CreateProcessA`

![](https://bbs.kanxue.com/upload/attach/201905/830989_PAXK5K4T2VSMU98.jpg)

栈回溯发现地址很小，明显是申请的堆空间，且这个进程是被挂起来的，经验告诉我，既然是挂起，肯定是要修改OEP,活者是写入一段shellcode等操作

继续下`WriteProcessMemory`断点

![](https://bbs.kanxue.com/upload/attach/201905/830989_GD6XVD7EPU5HFYB.jpg)

和我想的不太一样，虽然写入了内存，但是发现是把所有的区段都给改了，包括PE头，多次写入，不用想这块内存肯定我们真实的病毒程序

经过分析这块内存是利用`VirtualAlloc`申请出来的，把内存中的数据复制过来并解密，然后写入挂起的线程

直接把这块内存dump出来

![](https://bbs.kanxue.com/upload/attach/201905/830989_C2NMRXZJZ3GSEBU.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/830989_JD8VA72SNEETVMU.jpg)

经过测试，dump出来的程序确实真正的病毒程序

下面开始分析病毒的行为

用火绒剑过滤的时候发现病毒会复制到

`C:\Users\用户名\Documents\yfqgbh.exe`

名字是六位数的随机字母

在`CopyFileW`上下断

![](https://bbs.kanxue.com/upload/attach/201905/830989_S2B8JARWN6AEG4Q.jpg)

栈回溯发现下面有创建新的进程，这个进程就是我们复制过去的病毒，往上看的话肯定是判断

![](https://bbs.kanxue.com/upload/attach/201905/830989_FKH6BH93ZJPACDG.jpg)

往上走，发现有拼接字符串，下F2断，重新跑起来

![](https://bbs.kanxue.com/upload/attach/201905/830989_TY5XH73VN7NKWR6.jpg)

发现在获取目得的路径

![](https://bbs.kanxue.com/upload/attach/201905/830989_SM2BDGZE78MYEDF.jpg)

然后打开这个位置的病毒，很显然，这个位置并没有我们的程序，所以`GetLastError`会返回错误代码2

![](https://bbs.kanxue.com/upload/attach/201905/830989_YDT9MK5XNEM7P2H.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/830989_AUCDK9GFVNJAYVC.jpg)

这样我们的跳转实现不了（实现不了则会复制文件到目标目录）为了分析方面我直接把这个`jnz`改成`jmp`分析

单步往下跟踪

发现先创建了一个互斥体，防止多开

![](https://bbs.kanxue.com/upload/attach/201905/830989_5HPYZTTTYVGED86.jpg)

到这里我们可以先用IDA静态分析，配合OD动态验证我们的猜想

![](https://bbs.kanxue.com/upload/attach/201905/830989_Q9Z5J35FXCCETXE.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/830989_M3QPGAY7M3A9C6C.jpg)

在IDA里我们可以看到，在获取系统版本之后，下面有几个call

第一个call跟进去之后,发现明显是在提权

![](https://bbs.kanxue.com/upload/attach/201905/830989_Z2FM7HWH3BJMV7M.jpg)

第二个call，发现看不到什么，回到OD，继续单步

![](https://bbs.kanxue.com/upload/attach/201905/830989_56SBKVZHVMRXXXD.jpg)

发现返回值是`CreateThread`

![](https://bbs.kanxue.com/upload/attach/201905/830989_B7JWXCTRBE282QQ.jpg)

继续单步，发现创建了一个线程，回调函数地址是我们传进来的参数`a1`

![](https://bbs.kanxue.com/upload/attach/201905/830989_776YPC33AAXSJ68.jpg)

IDA反编译查看该线程的源码，发现是在枚举进程

![](https://bbs.kanxue.com/upload/attach/201905/830989_835VKJQJGKF9T5J.jpg)

经过动态分析这个线程

![](https://bbs.kanxue.com/upload/attach/201905/830989_7VS2H7E5RRX6SBU.jpg)

```
0217FF40   0217FF48  UNICODE "<"
0217FF44   00000000
0217FF48   0000003C
0217FF4C   00000040
0217FF50   00000000
0217FF54   005A3038  UNICODE "runas"
0217FF58   005A33E8  UNICODE "vssadmin.exe"
0217FF5C   005A3410  UNICODE " Delete Shadows /All /Quiet "
```

静默删除影子？不知道是下要干什么

分析完线程，回到主函数，继续单步发现创建了很多线程，因为文章篇幅，我们直接以找到感染函数和加密函数为目的分析（毕竟是勒索病毒，分析重点）

单步跟踪发现，有个设置线程优先级的，想象一下，把这个线程设置成了空闲状态

![](https://bbs.kanxue.com/upload/attach/201905/830989_PN9X4XXAU4Q42K3.jpg)

该线程回调函数地址地址为`0x401710`,IDA中跟随到该函数，发现里面在递归遍历磁盘，既然都遍历了，碰到合适的肯定要加密一下嘛，

进入递归函数，目录是肯定不会加密的，所以里面必不少的就是判断，目录，后缀名为我们需要加密的程序…等

![](https://bbs.kanxue.com/upload/attach/201905/830989_V93W5GHMEU7Y4V3.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/830989_53MBY73Q59N338F.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/830989_BN3URB5AY34XP9K.jpg)

设置属性，读取到内存，又写入，很有可能是在加密，在该函数位置下断

```
00401C85    E8 46000000     CALL <dump.sub_401CD0>                   ; 感染函数
```

加密函数肯定是要加密我们的文本的，我在`C:\backup`放入一个txt文本

跑起来程序，发现这个位置确实放了我们的文件，F9运行

![](https://bbs.kanxue.com/upload/attach/201905/830989_WGT5C82SNEWVWW2.jpg)

断到读取我们放入的文本时，继续单步跟进

![](https://bbs.kanxue.com/upload/attach/201905/830989_JZ3UTTB7FM7HG5E.jpg)

发现要`CreateFileW`我们的文本了

![](https://bbs.kanxue.com/upload/attach/201905/830989_KK8GHK5VUXWFFDM.jpg)

读取到内存

![](https://bbs.kanxue.com/upload/attach/201905/830989_MQADXC2B3DEDX2C.jpg)

先读取128个字节，并判断开头处是否为空，如果是则是已经加密过的，如果不是则会申请一段堆空间

![](https://bbs.kanxue.com/upload/attach/201905/830989_PM3XD4VH4BDRMNU.jpg)

把读取的内容放到这个空间

![](https://bbs.kanxue.com/upload/attach/201905/830989_Q4VXSB6VM4Z2HZP.jpg)

往下单步跟踪发现又申请了一段堆空间，然后有两个可以函数，经过测试

![](https://bbs.kanxue.com/upload/attach/201905/830989_Q7FMHZKH63PEQ6G.jpg)

这个位置的call是加密函数

```
00401FAE    E8 FD1A0000     CALL <dump.sub_403AB0>
```

最后就是设置文件指针，然后写入加密后的文本了

![](https://bbs.kanxue.com/upload/attach/201905/830989_J9G3QB6N475K6FR.jpg)

IDA里面还是比较清晰的

![](https://bbs.kanxue.com/upload/attach/201905/830989_VSN5GP36B7CUEVH.jpg)

以下为加密函数

![](https://bbs.kanxue.com/upload/attach/201905/830989_Q66PPFZP3PR6NKC.jpg)

如果这个加密是可逆的，那么我们就可以恢复原始文件了。弹出来的框框显示的是RSA-4096不可逆，有兴趣的可以研究

根据这个样本的特征我推测是CryptoWall勒索病毒，具体版本就不知道了

![](https://bbs.kanxue.com/upload/attach/201905/830989_WGPR5FR54TVHDA2.jpg)

加密前，加密后

![](https://bbs.kanxue.com/upload/attach/201905/830989_TM637QSCEYXMHQ9.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
