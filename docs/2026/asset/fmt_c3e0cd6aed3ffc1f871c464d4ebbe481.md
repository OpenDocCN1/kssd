
# [原创]mutex实现与性能

大家好，我是东北码农。

今天讲讲工作中的事吧，领导要求调研一个事，`mutex`性能如何？`mutex`无竞争时和有竞争时效率如何？

## 0x1、结论

我和领导汇报时，都习惯先说结论，领导可以很容易抓住重点，如果感兴趣就继续看。

结论如下：

+ 没有争抢时，不进入内核态。性能与自旋锁类似。
+ 有争抢时，没抢到，进入内核态等待唤醒。

## 0x2、mutex实现

`mutex`实现时，分用户态部分和内核态部分，分别承担不同的功能：

+ 用户态：使用`test and set`来测试是否可以抢到锁。
+ 内核态：使用`futex`系统调用来等待和唤醒。

## 0x3、实验过程

使用我们的老朋友`strace`命令来观察抢锁时的系统调用情况。

`strace`使用可以参考前面的文章：

<a href="elink@91bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9K6c8W2)9#2k6W2)9#2k6X3u0A6P5W2)9K6c8p5#2\*7d9i4S2y4K9V1p5I4e0#2c8Y4x3p5&6c8i4K6y4p5i4K6y4p5i4K6t1$3L8h3W2V1i4K6y4p5x3U0j5#2y4o6M7H3x3K6b7H3z5q4)9J5y4X3W2V1P5q4)9K6c8o6q4Q4x3U0k6K6L8W2)9K6c8r3b7J5z5e0W2S2z5r3x3^5k6r3p5$3j5e0W2S2k6r3y4S2y4$3b7&6y4h3t1J5y4e0f1#2j5$3x3H3x3K6j5&6i4K6t1$3j5$3S2C8M7$3#2Q4x3@1b7^5j5K6R3@1k6X3x3&6j5h3u0T1k6U0x3%4y4e0S2U0x3U0N6X3z5e0t1J5y4U0t1&6y4e0q4X3j5K6g2X3k6U0x3H3x3U0y4S2k6e0j5I4x3o6f1I4z5o6V1&6x3e0V1#2y4e0N6X3y4U0p5H3j5X3c8U0x3K6g2W2k6X3b7K6j5$3y4X3y4K6q4T1x3o6t1@1y4U0M7K6i4K6t1$3N6r3!0C8k6h3&6Q4x3@1b7@1y4e0f1#2x3e0l9J5y4K6u0Q4x3U0k6D9j5h3&6Y4i4K6y4p5P5X3S2Q4y4h3k6o6e0W2)9J5x3%4u0V1">如何使用strace</a>

<a href="elink@b9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9K6c8W2)9#2k6W2)9#2k6X3u0A6P5W2)9K6c8p5#2\*7d9i4S2y4K9V1p5I4e0#2c8Y4x3p5&6c8i4K6y4p5i4K6y4p5i4K6t1$3L8h3W2V1i4K6y4p5x3U0j5#2y4o6M7H3x3K6b7I4y4g2)9J5y4X3W2V1P5q4)9K6c8o6q4Q4x3U0k6K6L8W2)9K6c8o6k6W2j5X3j5$3x3e0M7K6j5$3g2U0y4K6g2V1k6o6x3@1z5o6l9J5y4h3t1$3y4o6V1&6k6h3p5#2k6h3x3^5i4K6t1$3j5$3S2C8M7$3#2Q4x3@1b7^5j5K6R3@1k6X3x3&6k6r3u0T1k6U0x3%4y4e0S2T1k6U0u0X3y4e0c8U0j5X3q4T1k6o6g2V1z5o6g2U0k6U0k6V1z5o6x3I4z5e0W2W2k6o6p5^5x3U0j5J5j5U0N6V1j5K6m8U0k6U0M7I4x3r3p5^5k6r3f1%4y4o6c8S2x3e0p5&6y4o6u0W2x3o6q4V1k6h3u0W2i4K6t1$3N6r3!0C8k6h3&6Q4x3@1b7@1y4e0f1#2x3e0l9J5y4K6u0Q4x3U0k6D9j5h3&6Y4i4K6y4p5P5X3S2Q4y4h3k6o6e0W2)9J5x3%4u0V1">动手实现一个strace</a>

### 0x31、场景一：单线程加锁，无争抢

测试代码：

```c++{.line-numbers}
std::mutex g\_lock;
void test\_no\_race()
{
for(int i = 0;i < 10;++i)
{
g\_lock.lock();
usleep(100\*1000);
g\_lock.unlock();
}
}
```

`strace`看一下，`lock` `unlock`无系统调用（只有`sleep`时有系统调用）。

```
xjp@xxx:~/code/case/case30\_c11\_fence$ strace -f ./a.out
。。。
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
clock\_nanosleep(CLOCK\_REALTIME, 0, {tv\_sec=0, tv\_nsec=100000000}, NULL) = 0
。。。
```

### 0x32、场景二：多线程加锁，有争抢

测试代码：

```c++{.line-numbers}
std::mutex g\_lock;
void test\_no\_race()
{
for(int i = 0;i < 10;++i)
{
g\_lock.lock();
usleep(100\*1000);
g\_lock.unlock();
}
}
void test\_race()
{
std::thread t1(test\_no\_race);
std::thread t2(test\_no\_race);
t1.join();
t2.join();
}
```

用`strace`看一下，此时会大量调用`futex`系统调用，来等待（`FUTEX_WAIT_PRIVATE`）和唤醒（`FUTEX_WAKE_PRIVATE`）。
