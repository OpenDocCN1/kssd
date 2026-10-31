
# [原创]CVE-2017-10661 原理，触发分析

## 前言

最近分析了 `CVE-2017-10661` 这个漏洞，不过没有找到什么可以利用的点。把分析过程拿出来分享下。

首先看看官方的修补代码：

```
diff --git a/fs/timerfd.c b/fs/timerfd.c
index c173cc1..384fa75 100644
--- a/fs/timerfd.c
+++ b/fs/timerfd.c
@@ -40,6 +40,7 @@ struct timerfd_ctx {
  short unsigned settime_flags; /* to show in fdinfo */
  struct rcu_head rcu;
  struct list_head clist;
+ spinlock_t cancel_lock;
  bool might_cancel;
 };
@@ -112,7 +113,7 @@ void timerfd_clock_was_set(void)
  rcu_read_unlock();
 }
-static void timerfd_remove_cancel(struct timerfd_ctx *ctx)
+static void __timerfd_remove_cancel(struct timerfd_ctx *ctx)
 {
  if (ctx->might_cancel) {
  ctx->might_cancel = false;
@@ -122,6 +123,13 @@ static void timerfd_remove_cancel(struct timerfd_ctx *ctx)
 }
 }
+static void timerfd_remove_cancel(struct timerfd_ctx *ctx)
+{
+ spin_lock(&ctx->cancel_lock);
+ __timerfd_remove_cancel(ctx);
+ spin_unlock(&ctx->cancel_lock);
+}
+
 static bool timerfd_canceled(struct timerfd_ctx *ctx)
 {
  if (!ctx->might_cancel || ctx->moffs != KTIME_MAX)
@@ -132,6 +140,7 @@ static bool timerfd_canceled(struct timerfd_ctx *ctx)
 static void timerfd_setup_cancel(struct timerfd_ctx *ctx, int flags)
 {
+ spin_lock(&ctx->cancel_lock);
  if ((ctx->clockid == CLOCK_REALTIME ||
  ctx->clockid == CLOCK_REALTIME_ALARM) &&
  (flags & TFD_TIMER_ABSTIME) && (flags & TFD_TIMER_CANCEL_ON_SET)) {
@@ -141,9 +150,10 @@ static void timerfd_setup_cancel(struct timerfd_ctx *ctx, int flags)
  list_add_rcu(&ctx->clist, &cancel_list);
  spin_unlock(&cancel_lock);
  }
- } else if (ctx->might_cancel) {
- timerfd_remove_cancel(ctx);
+ } else {
+ __timerfd_remove_cancel(ctx);
  }
+ spin_unlock(&ctx->cancel_lock);
 }
 static ktime_t timerfd_get_remaining(struct timerfd_ctx *ctx)
@@ -400,6 +410,7 @@ SYSCALL_DEFINE2(timerfd_create, int, clockid, int, flags)
  return -ENOMEM;
  init_waitqueue_head(&ctx->wqh);
+ spin_lock_init(&ctx->cancel_lock);
  ctx->clockid = clockid;
  if (isalarm(ctx)) |
```

上面是官方的修补代码，可以很直观的看到，修补代码主要做了几件事

**首先在结构体里面添加了一个锁的成员变量，**

**然后在代码中上了锁**

从这两个结论，首先就可以基本推论出来，这个漏洞应该是多线程并行，导致出来的问题。

接着继续分析修补代码，我们发现上锁的代码主要在一下代码

```c
static bool timerfd_canceled(struct timerfd_ctx *ctx)
{
if (!ctx->might_cancel || ctx->moffs != KTIME_MAX)
@@ -132,6 +140,7 @@ static bool timerfd_canceled(struct timerfd_ctx *ctx)

static void timerfd_setup_cancel(struct timerfd_ctx *ctx, int flags)
{
+ spin_lock(&ctx->cancel_lock);
if ((ctx->clockid == CLOCK_REALTIME ||
ctx->clockid == CLOCK_REALTIME_ALARM) &&
(flags & TFD_TIMER_ABSTIME) && (flags & TFD_TIMER_CANCEL_ON_SET)) {
@@ -141,9 +150,10 @@ static void timerfd_setup_cancel(struct timerfd_ctx *ctx, int flags)
list_add_rcu(&ctx->clist, &cancel_list);
spin_unlock(&cancel_lock);
}
- } else if (ctx->might_cancel) {
- timerfd_remove_cancel(ctx);
+ } else {
+ __timerfd_remove_cancel(ctx);
}
+ spin_unlock(&ctx->cancel_lock);
}
```

这样看不太方便，感觉有点混乱，我把这个函数从源码中找出来了

```c
static void timerfd_setup_cancel(struct timerfd_ctx *ctx, int flags)
{
+ spin_lock(&ctx->cancel_lock);
if (ctx->clockid == CLOCK_REALTIME && (flags & TFD_TIMER_ABSTIME) &&
   (flags & TFD_TIMER_CANCEL_ON_SET)) {
if (!ctx->might_cancel) {
ctx->might_cancel = true;
spin_lock(&cancel_lock);
list_add_rcu(&ctx->clist, &cancel_list);
spin_unlock(&cancel_lock);
}
} else if (ctx->might_cancel) {
timerfd_remove_cancel(ctx);
}
+ spin_unlock(&ctx->cancel_lock);
}
```

```c
static void timerfd_remove_cancel(struct timerfd_ctx *ctx)
{
if (ctx->might_cancel) {
ctx->might_cancel = false;
spin_lock(&cancel_lock);
list_del_rcu(&ctx->clist);
spin_unlock(&cancel_lock);
}
}
```

**结合两段代码，我们可以看到主要是给中间的这一段代码上了锁。**

那么主要问题就是出现在这一段代码当中了。

这两条分支
上面是链表添加结点
下面是移除

两条路线分支主要是 `ctx->might_cancel` 这个成员变量判断的

然后这个变量，我们发现，他是会在分支的运行过程里面修改的 蓝色的标记代码就是修改的地点

那么我们根据上面得到的信息，可以得到一个假设 如果两条线程同时运行这一段代码

线程1 删除了结点 并且 `ctx->might_cancel = false`；
那么他下次再运行的到这里的时候

```c
} else if (ctx->might_cancel) { //ctx->might_cancel == false
timerfd_remove_cancel(ctx);
}
```

这个条件就不会成立，那么就不会运行 `timerfd_remove_cancel` 函数
以上是正常情况，现在我们假设一中特殊情况

线程1 第二次执行到 `else if (ctx->might_cancel)` 这里的时候，CPU进行切换，切换到线程2执行

线程2 走上面分支

```c
if (ctx->clockid == CLOCK_REALTIME && (flags & TFD_TIMER_ABSTIME) &&
    (flags & TFD_TIMER_CANCEL_ON_SET)) {
if (!ctx->might_cancel) {
ctx->might_cancel = true;
```

设置了 `ctx->might_cancel = true`; 然后CPU继续执行线程1

现在 `ctx->might_cancel` 被线程2修改成了 `true` 又会执行 `timerfd_remove_cancel`

```c
} else if (ctx->might_cancel) {
timerfd_remove_cancel(ctx);
}
```

那么现在 `timerfd_remove_cancel(ctx)` 删除了两次结点 这就是漏洞的产生原理。

## 利用

如果猜测是对的，那么我们创建两条线程，一条循环执行删除操作，一个执行添加操作，就会触发到漏洞。

测试过后，成功触发了漏洞。

注：看了下代码本身流程，这里删除结点，只是想 节点的 上下两个点的指向修改了下。

崩溃的主要原因是因为，`entry->prev = LIST_POISON2;` 这里执行到第二次的时候 尝试在 `LIST_POISON2` 写入内容，导致了崩溃。

测试机器:
`nexus5` `5.0` `hammerhead-lrx21o-factory-56a09d43` 版本

附带代码： NDK-12-windows编译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：
- [jni.zip](javascript:void(0);)
  （1.98kb，152次下载）
