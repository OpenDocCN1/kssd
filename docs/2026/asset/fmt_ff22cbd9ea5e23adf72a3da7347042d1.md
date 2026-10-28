
# [原创]深入解析C++ stl实现原理

# 一、关于C++11/14

C++ STL（标准模板库）是C++重要的组成部分，我们c++程序员每天都在使用stl，对于c++11/14/17新标准。究竟stl底层是如何实现的？我们去探究一下。

C++11标准 是 C++ 语言发展史上具有里程碑意义的一个版本，掌握C++11/14以及以后得新标准是每个c++程序员基础技能！不能还停留在传统c++的知识层面上（C++98/03）。

# 二、STL实现原理

## 2.1 `std::thread`实现原理

### 2.1.1 `thread::thread`构造函数创建线程

`std::thread`的构造函数->c运行时库的线程函数`_beginthreadex`->WIN 32 API创建线程`CreateThread`

```
#include <iostream>
#include <thread>
using namespace std;
void test_fun(int a, int b ,int c)
{
	std::cout << "a=" << a << ", b=" << b << ", c=" << c << std::endl;
}
int main()
{
	thread thd(test_fun, 1, 2, 3);
	thd.join();
	std::cout << "Hello World!\n";
}
```

以下是调试具体的调用流程

![](https://bbs.kanxue.com/upload/attach/202411/917132_NXPFQDM63K3KJWP.webp)

备注：要向单步调试到`__beginthreadex`需要给模块`ucrtbased.dll`加载符号，此函数位于文件`C:\Program Files (x86)\Windows Kits\10\Source\10.0.19041.0\ucrt\startup\thread.cpp`

### 2.1.2 `thread::join`等待线程结束的实现原理

此函数的作用是阻塞等待线程执行结束。

实现原理：`thead::join()`->`msvcp140d.dll!_Thrd_join()` ->Win32 API `WaitForSingleObjectEx` 等待线程的执行结束

![](https://bbs.kanxue.com/upload/attach/202411/917132_36V7PJDBRQ252BJ.webp)

备注：要想命中`_Thrd_join()`，需要给`msvcp140d`加载符号！

### 2.1.3 `thread::detach`脱离线程

就是不等待线程执行，主线程和工作者线程分离

实现原理：`std::thread::detach()`->`msvcp140d.dll!_Thrd_detach()`->Win32 API `CloseHandle()`

![](https://bbs.kanxue.com/upload/attach/202411/917132_CMYJJRKDGUFSHQC.webp)

### 2.1.4  `this_thread::sleep_for`

`this_thread::sleep_for()` 函数=>`msvcp140d.dll!Thrd_sleep()`函数=>win32api `Sleep（）`

![](https://bbs.kanxue.com/upload/attach/202411/917132_9CSNPBG3M9KMMJ3.webp)

## 2.2 `std::mutex`实现原理

### 2.2.1 `lock`函数的实现原理

`std::mutex`的成员函数`std::_Mutex_base::lock()`->`msvcp140d.dll!_Mtx_lock()`->

`msvcp140d.dll!Concurrency::details::stl_critical_section_win7::lock()` ->

Win32 API 读写锁`AcquireSRWLockExclusive`

![](https://bbs.kanxue.com/upload/attach/202411/917132_XEYRUHEGSYD9WH2.webp)

以下是测试代码：

```
#include <mutex>
#include <thread>
int counter = 0;
std::mutex mtx;
void attempt_10k_increases()
{
	for (int i = 0; i < 10000; ++i)
	{
		mtx.lock();  // only increase if currently not locked:
		++counter;
		mtx.unlock();
	}
}
int main()
{
	//std::mutex测试
	std::thread th = std::thread(attempt_10k_increases);
	th.join();
}
```

### 2.2.2 `unlock`函数的实现原理

`std::mutex`的成员函数`std::_Mutex_base::unlock()`->`msvcp140d.dll!_Mtx_unlock()`->`msvcp140d.dll!Concurrency::details::stl_critical_section_win7::unlock()`-> Win32 API`ReleaseSRWLockExclusive`

![](https://bbs.kanxue.com/upload/attach/202411/917132_J4WR9YJZVK8XXBV.webp)

## 2.3 `std::condition_variable`条件变量的实现原理

解说：条件变量，是Windows Vista中新增加的一种处理线程同步问题的机制。它可以与“关键代码段（critical section）”或“读写锁（SRWLock）”相互 配合使用，来实现线程的同步，特别是实现类似“生产者-消费者”问题 的时候，十分有效。

先说下条件变量的各个成员函数对应的Win 32 API:

`std::condition_variable::wait()`          对应win32 API:   `SleepConditionVariableSRW`

`std::condition_variable::notify_all`          对应win32 API:   `WakeAllConditionVariable`

`std::condition_variable::notify_one`          对应win32 API:   `WakeConditionVariable`

### 2.3.1 `std::condition_variable::wait()`的实现原理

`wait()` 函数底层调用的是win32 API:   `SleepConditionVariableSRW`

![](https://bbs.kanxue.com/upload/attach/202411/917132_QRPXQJ7G9JSEEC7.webp)

### 2.3.2 `std::condition_variable::notify_one()`的实现原理

`notify_one()`函数底层调用的是Win32 API:`WakeConditionVariable`

![](https://bbs.kanxue.com/upload/attach/202411/917132_M3KMGAP2RMSNMWJ.webp)

### 2.3.3 `std::condition_variable::notify_all()`的实现原理

`notify_all()`函数底层调用的是Win32 API:`WakeAllConditionVariable`

![](https://bbs.kanxue.com/upload/attach/202411/917132_8TQ7TXGB27N4EUA.webp)

以下是测试代码：

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
std::mutex mtx2;
std::condition_variable cv;
bool ready = false;
void print_id(int id) {
	std::unique_lock<std::mutex> lck(mtx2);
	/** 如果条件不满足，则等待. */
	while (!ready) {
		cv.wait(lck); ///< 等待、阻塞
	}
	std::cout << "recieve signal..., do something" << std::endl;
	/** 执行其他操作. */
}
void Signal() {
	std::unique_lock<std::mutex> lck(mtx2);
	ready = true;  ///< 设置条件为true
	cv.notify_all();  ///< 唤醒所有等待的线程
}
void test_condition_variable()
{
	std::thread threads[10];
	/** 发起10个线程. */
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(print_id, i);
	std::cout << "10 threads ready to race...\n";
	Signal(); ///< 唤醒线程，继续执行.
	for (auto& th : threads) {
		th.join();
	}
	getchar();
}
```

## 2.4 `std::atomic`原子操作的实现原理

原子操作的实现原理值通过支持原子操作的汇编指令实现的（**无锁编程Lock-free**）。底层调用的是Win32 API的无锁函数

`_InterlockedExchange`、`_InterlockedIncrement` 等系列的函数，这种函数有很多个，可以自行查阅。

汇编指令原子操作方法，在汇编语言中，原子操作是只不能被中断或者同时执行多个指令的操作，原子操作通常用于对共享资源的读写操作，确保在多线程环境下数组的一致性。在x86架构下，汇编指令提供了一些原子操作的方法，例如：

|  |  |  |  |
| --- | --- | --- | --- |
| 汇编指令 | 功能 | 格式 | 备注 |
| XCHG | 交换两个操作数的值。这个指令可以用来实现原子的“读-改-写”操作 | XCHG [地址],[寄存器] |  |
| LOCK | 锁定总线指令，确保指令的执行是原子的 | LOCK 指令 |  |
| CMPXCHG | 比较并交换操作数的值，这个指令可以用来实现原子的比较和更新指令 | CMPXCHG [地址],[寄存器] |  |
| XADD | 将寄存器的值与内存中的值相加，并将结果存回内存，这个指令可以用于实现原子的加法操作 | XADD [地址],[寄存器] |  |

以上是一些常用的汇编指令原子操作的方法，他们可以通过锁定指令总线或者其他的硬件机制来确保操作的原子性，不同的处理器和架构可能提供不同的原子操作方法。

### 2.4.1 原子操作1：交换两个数的值（）

有如下代码：

```
atomic_int nInt = 1;
nInt = nInt + 2;//这里会调用Win32无锁编程的函数_InterlockedExchange
```

其调用过程如下，可看到`nInt = nInt + 2;`这一句代码底层调用的是Win32 API `_InterlockedExchange`（原子交换两个数的值）

![](https://bbs.kanxue.com/upload/attach/202411/917132_7ZKUN28Q99KZNSB.webp)

那么Win32 API `_InterlockedExchange`是如何实现原子操作的？来看一下此函数对应的汇编代码：

![](https://bbs.kanxue.com/upload/attach/202411/917132_SE9DM8QGTYPAERN.webp)

通过上图可以看到 `_InterlockedExchange`函数用了支持原子操作的汇编指令`xchg`，来交换两个操作数的值。可以参考上面的表格

### 2.4.1 原子操作1：原子加法操作

有如下代码：

```
atomic_int nInt = 1;
nInt++;
```

其调用过程如下：可以看到`nInt++`这句代码调用的是Win32 API `_InterlockedIncrement`（原子加法操作）

![](https://bbs.kanxue.com/upload/attach/202411/917132_5P7ZPTT3MWJGJFH.webp)

那么Win32 API `_InterlockedIncrement`是如何实现原子操作的？来看一下此函数对应的汇编代码：

![](https://bbs.kanxue.com/upload/attach/202411/917132_7N86HV54GMJ5DHN.webp)

可以看到`_InterlockedIncrement`用到是原子加法汇编指令`lock xadd`,可以参考上面的表格。

## 2.5 万能引用、引用折叠

**万能引用的两个前提条件：一是函数模板的形参、二是类型为T&& val，否则大部分就是右值引用**。

理解万能引用和引用折叠是理解完美转发`std::forward`和`std::move`的前提，所以这里首先搞明白这个概念。

关于引用折叠的核心是，**C++11当中的一些构造会弄出来引用的引用，而C++不允许出现引用的引用。如果代码当中显示的出现了一个引用的引用，那代码就是不合法的，会出现编译错误。**

```
template<typename T>
void f(T&& param) {
}
int main() {
    int x = 666;
    f(10);                           // invoke f on rvalue
    f(x);                            // invoke f on lvalue
}
```

当用rvalue 10调用 f 的时候, T被推导为 int，实例化的 f 看起来像这样:

```
void f(int&& param);             // f instantiated from rvalue
```

这里一切都OK。但是当我们用lvalue x 来调用 f 的时候，T 被推导为int&，而实例化的 f 就包含了一个引用的引用:

```
void f(int& && param);           // initial instantiation of f with lvalue
```

**因为这里出现了引用的引用，这实例化的代码乍一看好像不合法，但是像f(x)这么写代码是完全合理的。为了避免编译器对这个代码报错，C++11引入了一个叫做“引用折叠”(reference collapsing)的规则来处理某些像模板实例化这种情况下带来的"引用的引用"的问题。**

"引用的引用"就有四种可能的组合：

a）左值引用的左值引用：int&  &                            折叠为 int&

b)   左值引用的右值引用：int&  &&                         折叠为 int&

c)   右值引用的左值引用：int&&  &                         折叠为 int&

d)   右值引用的右值引用：int&&  &&                      折叠为 int&&

**引用折叠只有两条规则:**

一个 rvalue reference to an rvalue reference 会变成 (“折叠为”) 一个 rvalue reference.

所有其他种类的"引用的引用" (组合当中含有lvalue reference) 都会折叠为 lvalue reference.

下面再来看引用一种引用折叠的情况，

**当一个变量本身的类型是引用类型的时候，这里就有点难搞了。这种情况下，类型当中所带的引用就被忽略了。**例如

代码如下：

```
template<typename T>
void f(T &&param) {
    static_assert(std::is_lvalue_reference<T>::value, "T& is lvalue reference");
    cout << "T& is lvalue reference" << endl;
}
int main() {
    int x;
    int &&r1 = 10;// r1’s type is int&&
    int &r2 = x;// r2’s type is int&
    f(r1);
    f(r2);
}
```
