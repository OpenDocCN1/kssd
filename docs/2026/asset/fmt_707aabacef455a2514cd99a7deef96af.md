
`xjp@xxx:~/code/case/case30_c11_fence$ strace -f ./a.out` `。。。` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=100000000}, NULL) = 0` `。。。`

### 0x31、场景二：多线程加锁，有争抢

测试代码：

```cpp
std::mutex g_lock;
void test_no_race()
{
for(int i = 0;i < 10;++i)
{
g_lock.lock();
usleep(100*1000);
g_lock.unlock();
}
}

void test_race()
{
std::thread t1(test_no_race);
std::thread t2(test_no_race);

t1.join();
t2.join();
}
```

用`strace`看一下，此时会大量调用`futex`系统调用，来等待（`FUTEX_WAIT_PRIVATE`）和唤醒（`FUTEX_WAKE_PRIVATE`）。
