
只有一条线程运行的

执行 `LeaveCriticalsection` 后~~

线程

标识       入口       数据块       最后错误            状态        优先权     用户时间      系统时间

```
000002A0   7C810856   7FFDE000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
000002E8   7C810856   7FFD6000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
000005A8   7C810856   7FFD9000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
000007E4   7C810856   7FFD4000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000854(  004011C1   7FFDF000     ERROR_SUCCESS (000  激活           32 + 0       0.0625 s      0.1875 s
0000086C   7C810856   7FFDA000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000880   7C810856   7FF97000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000890   7C810856   7FFD3000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000904   7C810856   7FF9E000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
0000095C   7C810856   7FFD5000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000AC8   7C810856   7FF9B000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000B54   7C810856   7FFD8000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000B64   7C810856   7FF9D000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000B84   7C810856   7FF99000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000D44   7C810856   7FF9C000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000DAC   7C810856   7FFDC000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000E88   7C810856   7FF9F000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000EF0   7C810856   7FFD7000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000F6C   7C810856   7FF98000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
00000F84   7C810856   7FFDD000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0156 s
00000F8C   7C810856   7FF9A000     ERROR_SUCCESS (000  激活           32 + 0       0.0000 s      0.0000 s
```

当再次有任何一条 线程执行到 `Enter` 的时候~~~ ，`enter` 就会把剩下的线程，都挂起。，保证了同步~~~

### 互斥体

`windows api` 中提供了一个互斥体，功能上要比临界区强大。

`Mutex` 是互斥体的意思，当一个线程持有一个 `Mutex` 时，其它线程申请持有同一个 `Mutex` 会被阻塞，

因此可以通过 `Mutex` 来保证对某一资源的互斥访问（即同一时间最多只有一个线程访问）。

调用 `CreateMutex` 可以创建或打开一个 `Mutex` 对象，

```c
HANDLE CreateMutex(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,
  BOOL bInitialOwner,
  LPCTSTR lpName
);
```

其中参数 `lpMutexAttributes` 用来设定 `Mutex` 对象的安全描述符和是否允许子进程继承句柄。

`bInitialOwner` 表明是否将 `Mutex` 的持有者设置为调用线程。

`lpName` 参数设置 `Mutex` 的名字，该名字区分大小写并不能包含 `""`，最大长度为 `MAX_PATH`，可设置为 `NULL` 表明该 `Mutex` 为匿名对象。

如果调用成功，则返回 `Mutex` 的句柄，否则返回 `NULL`，如果 `lpName` 不为 `NULL` 且调用前同名的 `Mutex` 已被创建，则返回同名 `Mutex` 的句柄，此时调用 `GetLastError` 将返回 `ERROR_ALREADY_EXISTS`，参数 `bInitialOwner` 将被忽略。

理论大家自己 去 msdn google baidu ...

- -

我们再使用 `Mutext` 的时候 要使用 先获取互斥量，

获取互斥量 我们使用 `WaitForSingleObject`

当互斥量被等待成功，他将自动复位

其他线程等待

，线程使用完后，调用

`ReleaseMutex`, `hMutex`

释放后，互斥量自动置位，此时 其他线程可以进入~~~

- - 就是 代码写的不一样

`CreateMutex`

`WaitForSingleObject`

`ReleaseMutex`

- -

很简单~~

——————

### 信号灯

信号灯有点不一样~~

`Semaphore` 是旗语的意思，在 Windows 中，`Semaphore` 对象用来控制对资源的并发访问数。`Semaphore` 对象具有一个计数值，当值大于 0 时，`Semaphore` 被置信号，当计数值等于 0 时，`Semaphore` 被清除信号。每次针对 `Semaphore` 的 `wait functions` 返回时，计数值被减 1，调用 `ReleaseSemaphore` 可以将计数值增加 `lReleaseCount` 参数值指定的值。

`CreateSemaphore` 函数用于创建一个 `Semaphore`

```c
HANDLE CreateSemaphore(
LPSECURITY_ATTRIBUTES lpSemaphoreAttributes,
LONG lInitialCount, 这个就是指示创建后，信号灯对象的计数值，这里我们只需要一个就设置为1
LONG lMaximumCount, ;这个是最多，多少个线程能获取这个 计数值
LPCTSTR lpName
);
```

`lpSemaphoreAttributes` 为安全属性，

`lInitialCount` 为 `Semaphore` 的初始值，

`lMaximumCount` 为最大值，

`lpName` 为 `Semaphore` 对象的名字，`NULL` 表示创建匿名 `Semaphore`

此外还可以调用 `OpenSemaphore` 来打开已经创建的非匿名 `Semaphore`

```c
HANDLE OpenSemaphore(
DWORD dwDesiredAccess,
BOOL bInheritHandle,
LPCTSTR lpName
);
```

调用 `ReleaseSemaphore` 增加 `Semaphore` 计算值

```c
BOOL ReleaseSemaphore(
HANDLE hSemaphore,
LONG lReleaseCount,
LPLONG lpPreviousCount
);
```

`lpReleaseCount` 参数表示要增加的数值，

`lpPreviousCount` 参数用于返回之前的计算值，如果不需要可以设置为 `NULL`

当然还是用 `WaitForSingleObject` 等待到信号灯后，会把信号灯计数 -1，这里我们 直接

创建的时候，初始化值，和最大值 都设置为 1 ，那么 一个线程进入后，计数值就会 -1 == 0，那么 状态变成 复位，其他的线程就等待这里了

执行到最后 调用

`ReleaseSemaphore` 函数

`ReleaseSemaphore` 函数会把计数值 +1

那么 对象状态又变成置位，那么，其他线程就可进入执行了~~~

- - 如此反复。。

写完了~~~..

笔记只是,一点愚见，

错误的地方，还请各位多多批评和指出

欢迎指导

然后我说完了~~~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
