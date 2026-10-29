
## 2. sender线程加入binder_work到thread->todo链表中

当sender线程传送的内容里包括`BINDER_TYPE_BINDER`或者`BINDER_TYPE_WEAK_BINDER`，`binder_node`结构体将会创建，接收端的线程同时会有一个引用（如果这个引用变为0，将会释放`binder_node`）

这时候会调用`binder_translate_binder`，最终会走到以下代码，可以看到`node->work`会进入队列`thread->todo`链表中。

```c
static int binder_translate_binder(struct flat_binder_object *fp,
struct binder_transaction *t,
struct binder_thread *thread)
{
// [...]
ret = binder_inc_ref_for_node(target_proc, node,
fp->hdr.type == BINDER_TYPE_BINDER,
&thread->todo, &rdata);
// [...]
}
static int binder_inc_ref_for_node(struct binder_proc *proc,
struct binder_node *node,
bool strong,
struct list_head *target_list,
struct binder_ref_data *rdata)
{
// [...]
ret = binder_inc_ref_olocked(ref, strong, target_list);
// [...]
}
static int binder_inc_node_nilocked(struct binder_node *node, int strong,
int internal,
struct list_head *target_list)
{
// [...]
if (strong) {
// [...]
if (!node->has_strong_ref && target_list) {
// [...]
binder_enqueue_deferred_thread_work_ilocked(thread,
&node->work); // 当对这个node节点是强引用时
}
} else {
// [...]
if (!node->has_weak_ref && list_empty(&node->work.entry)) {
// [...]
binder_enqueue_work_ilocked(&node->work, target_list);// 当是弱引用时。
}
}
return 0;
}
```

![引用3](upload/attach/202012/726411_79CQ59ZR2VTQA5W.png)

## 3. Freeing the binder_work Structure From the Receiver Thread（接受端线程如何释放binder_work结构体呢）

当binder收到`BC_FREE_BUFFER`指令时就会调用`binder_free_node`函数。

```c
static void binder_free_node(struct binder_node *node)
{
kfree(node);
binder_stats_deleted(BINDER_STAT_NODE);
}
```

![引用2](upload/attach/202012/726411_ZD5VG55HSYT3RZ2.png)

注意到也许不用显示的调用`BC_FREE_BUFFER`时，当binder service一次的transaction的完成时，就会自动用`BC_FREE_BUFFER`释放发送端的`binder_node`结构体，当然也可以寻找`ITokenManager service`这样的一个服务，充当一个中介，这里面的原理就是有时候不能控制service端，就比如media这样的服务，你无法控制它，毕竟是一个系统service，但是`ITokenManager`可以让我们既可以控制service端又可以控制client端，如果水滴漏洞采用这样的方式，是不是效率会更高一些。

所以对于`binder_node`的释放，其实是`BC_FREE_BUFFER`指令来进行，有的service接收端是自动发送的。

在`binder_parse`函数中，当回复一个transacton时，service manager要么会call `binder_free_buffer`，如果是单向`(one-way)`的transaction，就会调用`binder_send_reply`

```c++
int binder_parse(struct binder_state *bs, struct binder_io *bio,
uintptr_t ptr, size_t size, binder_handler func)
{
// [...]
switch(cmd) {
// [...]
case BR_TRANSACTION_SEC_CTX:
case BR_TRANSACTION: {
// [...]
if (func) {
// [...]
if (txn.transaction_data.flags & TF_ONE_WAY) {
binder_free_buffer(bs, txn.transaction_data.data.ptr.buffer);
} else {
binder_send_reply(bs, &reply, txn.transaction_data.data.ptr.buffer, res);
}
}
break;
}
// [...]
```

在两种情况下，servicemanager最后都会回复一个`BC_FREE_BUFFER`，其内核调用流程为

```c
static int binder_thread_write(struct binder_proc *proc,
struct binder_thread *thread,
binder_uintptr_t binder_buffer, size_t size,
binder_size_t *consumed)
{
// [...]
case BC_FREE_BUFFER: {
// [...]
binder_transaction_buffer_release(proc, buffer, 0, false);
// [...]
}
// [...]
static void binder_transaction_buffer_release(struct binder_proc *proc,
struct binder_buffer *buffer,
binder_size_t failed_at,
bool is_failure)
{
// [...]
switch (hdr->type) {
// [...]
case BINDER_TYPE_HANDLE:
case BINDER_TYPE_WEAK_HANDLE: {
struct flat_binder_object *fp;
struct binder_ref_data rdata;
int ret;
fp = to_flat_binder_object(hdr);
ret = binder_dec_ref_for_handle(proc, fp->handle,
hdr->type == BINDER_TYPE_HANDLE, &rdata);
// [...]
} break;
// [...]
static int binder_update_ref_for_handle(struct binder_proc *proc,
uint32_t desc, bool increment, bool strong,
struct binder_ref_data *rdata)
{
// [...]
if (increment)
ret = binder_inc_ref_olocked(ref, strong, NULL);
else
/*
* Decrements the reference count by one and returns true since it
* dropped to zero
*/
delete_ref = binder_dec_ref_olocked(ref, strong);
// [...]
/* delete_ref is true, the binder node is freed */
if (delete_ref)
binder_free_ref(ref);
return ret;
// [...]
}
static void binder_free_ref(struct binder_ref *ref)
{
if (ref->node)
binder_free_node(ref->node);
kfree(ref->death);
kfree(ref);
}
```

分析好了poc，从而得到内核crash
