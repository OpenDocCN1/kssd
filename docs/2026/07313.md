
# [分享]系统API调用 学习 记录

![](https://bbs.kanxue.com/upload/tmp/845197_J3BF5PXUX4H2VQK.png)

## 内核学习记录 快速调用

环境: XP 2-9-9-12 分页模式

API：`ReadProcessMemory`

### 1. `ReadProcessMemory` `Kernel32.dll`导出函数

先IDA打开`Kernel32.dll`瞄一眼

![](https://bbs.kanxue.com/upload/attach/202009/845197_4ZB6VAAXVN5MT67.png)

![](https://bbs.kanxue.com/upload/attach/202009/845197_3Y9D886CEKBS7MX.png)

### 2. 去`NTDLL`看看

![](https://bbs.kanxue.com/upload/attach/202009/845197_YP4357ED2JX7YBP.png)

这个看到了很诡异的

`eax = 0BAh`（注意）

`edx = 7FFE0300h`

`Call [edx]`

说明`edx`这个是个函数

接下来认识下`_KUSER_SHARED_DATA`在内核和用户态有段数据一直是共享状态

这块内存在内核和用户态是共享的，

内核线性地址：`ffdf0000`

用户态线性地址：`7FFE0000`

WinDbug验证一下

![](https://bbs.kanxue.com/upload/attach/202009/845197_NZ93X99SH3ZVC4R.png)

![](https://bbs.kanxue.com/upload/attach/202009/845197_RXQUM96NRWQAXZK.jpg)

看着好像是一样，感兴趣可以拆下地址，就是同一块内存

如此来说

`Edx = 7FFE0300`

`Call [edx]`

大局已定呀

### 3. 查看结构并重新加载符号

![](https://bbs.kanxue.com/upload/attach/202009/845197_F8NPFU4DPBV3HU4.jpg)

赶紧看看这个结构看它取哪里了

![](https://bbs.kanxue.com/upload/attach/202009/845197_AMABUNVH3GDB82J.png)

![](https://bbs.kanxue.com/upload/attach/202009/845197_QZD4VV2D63RYA6F.png)

都没得函数名，肯定要重新加载下符号试试

![](https://bbs.kanxue.com/upload/attach/202011/845197_HVGDTWQFS9WM4QP.png)

![](https://bbs.kanxue.com/upload/attach/202011/845197_352XMYVV3ZQBQCC.jpg)

看到了啥 还在`Ntdll`里边转悠咋还不进内核呢？

留意下`edx`下边要用 盘它

### 4. 先查下`sysenter`指令

快速调用

![](https://bbs.kanxue.com/upload/attach/202011/845197_SHDUZCKTUCZQQQ7.png)

我们知道当权限从3环发生切换到0环，相对应的 3环的`cs` `ss` `esp` `eip`都要发生权限的转换切换为0环权限

但是`sysenter`值拿了`cs` `esp` `eip`

`Ss`才哪里呢？ 细心的朋友肯定发现了

![](https://bbs.kanxue.com/upload/attach/202011/845197_3NPYWGYEUS2T5R9.png)

### 5. 段选择子与MSR

奥利给！

![](https://bbs.kanxue.com/upload/attach/202011/845197_SFPVCVWJ2Q94ARK.png)

那现在所有新的段选择子都全了

![](https://bbs.kanxue.com/upload/attach/202011/845197_VDGPAA4NTPWQPWT.png)

`MSR`寄存器一下为下标 `rdmsr` 下标

174 目标环 0 `CS` 选择器

175 目标环 0 `ESP`

176 目标环 0 入口 `EIP`

3环提权到0 快速调用 `ss = cs + 8`

接下来当然是先去`MSR[176]`看看接下来执行什么东东

![](https://bbs.kanxue.com/upload/attach/202011/845197_A5PTUWXYS86W6PV.png)

直接看`MSR[176] = 8053E540`可以从地址看到终于到内核了

赶紧去Windbg `u`一下看看

![](https://bbs.kanxue.com/upload/attach/202011/845197_ZFYAMBKRWPREBR7.png)

这个函数先改了`fs:30`

3环`fs`指向的是`TEB` 0环指向的是`_KPCR`

先拆分下选择子：`0000 0000 0011 0000`

得出描述符为： ``8003f030 ffc093df`f0000001``

`base = ffdff000`

还得了解下`SSDT`结构

![](https://bbs.kanxue.com/upload/attach/202011/845197_E4W2UD3RPCTU2FR.png)

```c
typedef struct _KSYSTEM_SERVICE_TABLE
{
PULONG ServiceTableBase; //这个指向系统服务函数地址表
PULONG ServiceCounterTableBase; //系统这个服务表调用了几次
ULONG NumberOfService; //服务函数的个数
PULONG ParamTableBase; //参数表
}KSYSTEM_SERVICE_TABLE, *PKSYSTEM_SERVICE_TABLE;
```

![](https://bbs.kanxue.com/upload/attach/202011/845197_2ABNRJJ9EWPV72W.png)

```c
SSDT
{
SystemServiceTable 基础服务
SystemServiceTableShadow UI服务
}
```

回头去内核看看这个函数`KiFastCallEntry`上IDA

![](https://bbs.kanxue.com/upload/attach/202011/845197_SDPFSRXM9QM3WSN.png)

![](https://bbs.kanxue.com/upload/attach/202011/845197_DYZD77Y63CD4TNQ.png)

![](https://bbs.kanxue.com/upload/attach/202011/845197_QHXUMZS8SRYBZNA.png)

![](https://bbs.kanxue.com/upload/attach/202011/845197_4V7QC78Z4E7HAKA.png)

使用`iret`返回的 中间有很多细节还不是很清楚 欢迎指正。

都在看雪白嫖了好久了，希望看雪越来越好。

![](https://bbs.kanxue.com/upload/attach/202011/845197_JQ5UAFVQK4J2E58.png)告辞

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
