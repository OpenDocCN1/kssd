
`printk(KERN_INFO "[Firewall::Error] Rule[%d], invalid action (%d), skipping!\n", i, rule->action);`

`return SKIP;`

}

`printk(KERN_INFO "[Firewall::Info] %s Rule[%d], action %d\n", (type == INBOUND) ? "Inbound" : "Outbound", i, rule->action);`

`return rule->action;`

}

`return SKIP;`

}

### `firewall_add_rule`

```c
static long firewall_add_rule(user_rule_t user_rule, rule_t **firewall_rules, uint8_t idx)
{
printk(KERN_INFO "[Firewall::Info] firewall_add_rule() adding new rule!\n");
if (firewall_rules[idx] != NULL)
{
printk(KERN_INFO "[Firewall::Error] firewall_add_rule() invalid rule slot!\n");
return ERROR;
}
//在对应的idx用kzalloc分配一个rule_t，没有限制idx范围
firewall_rules[idx] = (rule_t *)kzalloc(sizeof(rule_t), GFP_KERNEL);
if (!firewall_rules[idx])
{
printk(KERN_INFO "[Firewall::Error] firewall_add_rule() allocation error!\n");
return ERROR;
}
memcpy(firewall_rules[idx]->iface, user_rule.iface, 16);
memcpy(firewall_rules[idx]->name, user_rule.name, 16);
//拷贝0x800缓冲区到对应位置
strncpy(firewall_rules[idx]->desc, user_rule.desc, DESC_MAX);
/* in4_pton将字符串转换成ipv4地址 ， 检查ipv4的地址格式是否合法*/
if (in4_pton(user_rule.ip, strnlen(user_rule.ip, 16), (u8 *)&(firewall_rules[idx]->ip), -1, NULL) == 0)
{
printk(KERN_ERR "[Firewall::Error] firewall_add_rule() invalid IP format!\n");
kfree(firewall_rules[idx]);
firewall_rules[idx] = NULL;
return ERROR;
}
/* 检查网络掩码是否合法 */
if (in4_pton(user_rule.netmask, strnlen(user_rule.netmask, 16), (u8 *)&(firewall_rules[idx]->netmask), -1, NULL) == 0)
{
printk(KERN_ERR "[Firewall::Error] firewall_add_rule() invalid Netmask format!\n");
kfree(firewall_rules[idx]);
firewall_rules[idx] = NULL;
return ERROR;
}
/* 将对应的user-space的信息赋值到kernel-space变量中 */
firewall_rules[idx]->proto = user_rule.proto;
firewall_rules[idx]->port = ntohs(user_rule.port);
firewall_rules[idx]->action = user_rule.action;
firewall_rules[idx]->is_duplicated = 0;
printk(KERN_ERR "[Firewall::Info] firewall_add_rule() new rule added!\n");
return SUCCESS;
}
```

### `firewall_delete_rule`

```c
static long firewall_delete_rule(user_rule_t user_rule, rule_t **firewall_rules, uint8_t idx)
{
printk(KERN_INFO "[Firewall::Info] firewall_delete_rule() deleting rule!\n");
if (firewall_rules[idx] == NULL)
{
printk(KERN_INFO "[Firewall::Error] firewall_delete_rule() invalid rule slot!\n");
return ERROR;
}
kfree(firewall_rules[idx]);
firewall_rules[idx] = NULL;
return SUCCESS;
}
```
这个函数没有UAF可以用

### `firewall_edit_rule`

可以编辑几个对应的属性
```c
static long firewall_edit_rule(user_rule_t user_rule, rule_t **firewall_rules, uint8_t idx)
{
printk(KERN_INFO "[Firewall::Info] firewall_edit_rule() editing rule!\n");
#ifdef EASY_MODE
printk(KERN_INFO "[Firewall::Error] Note that description editing is not implemented.\n");
#endif
if (firewall_rules[idx] == NULL)
{
printk(KERN_INFO "[Firewall::Error] firewall_edit_rule() invalid idx!\n");
return ERROR;
}
memcpy(firewall_rules[idx]->iface, user_rule.iface, 16);
memcpy(firewall_rules[idx]->name, user_rule.name, 16);
if (in4_pton(user_rule.ip, strnlen(user_rule.ip, 16), (u8 *)&(firewall_rules[idx]->ip), -1, NULL) == 0)
{
printk(KERN_ERR "[Firewall::Error] firewall_edit_rule() invalid IP format!\n");
return ERROR;
}
if (in4_pton(user_rule.netmask, strnlen(user_rule.netmask, 16), (u8 *)&(firewall_rules[idx]->netmask), -1, NULL) == 0)
{
printk(KERN_ERR "[Firewall::Error] firewall_edit_rule() invalid Netmask format!\n");
return ERROR;
}
firewall_rules[idx]->proto = user_rule.proto;
firewall_rules[idx]->port = ntohs(user_rule.port);
firewall_rules[idx]->action = user_rule.action;
printk(KERN_ERR "[Firewall::Info] firewall_edit_rule() rule edited!\n");
return SUCCESS;
}
```

### `firewall_dup_rule`

```c
static long firewall_dup_rule(user_rule_t user_rule, rule_t **firewall_rules, uint8_t idx)
{
//dup与firewall_rules应该是要统一属性
uint8_t i;
rule_t **dup;
printk(KERN_INFO "[Firewall::Info] firewall_dup_rule() duplicating rule!\n");
//选择对应的rules list
dup = (user_rule.type == INBOUND) ? firewall_rules_out : firewall_rules_in;
if (firewall_rules[idx] == NULL)
{
printk(KERN_INFO "[Firewall::Error] firewall_dup_rule() nothing to duplicate!\n");
return ERROR;
}
// 如果对应的 idx 已经设置了is_duplicated标志，return ERROR
if (firewall_rules[idx]->is_duplicated)
{
printk(KERN_INFO "[Firewall::Info] firewall_dup_rule() rule already duplicated before!\n");
return ERROR;
}
// 扫描每个firewall_rules中每一项，设置is_duplicated = 1
// dup实际上是全局的firewall_rules_out 或者 firewall_rules_in
// 如果有list中有NULL的，那么，把pointer list中所有为NULL的entry都设置成firewall_rules[idx]？
// 实际就是用firewall_rules[idx]来填查找到的第一个NULL的，然后设置用于填充的entry is_duplicated=1
for (i = 0; i < MAX_RULES; i++)
{
if (dup[i] == NULL)
{
dup[i] = firewall_rules[idx];
firewall_rules[idx]->is_duplicated = 1;
printk(KERN_INFO "[Firewall::Info] firewall_dup_rule() rule duplicated!\n");
return SUCCESS;
}
}
printk(KERN_INFO "[Firewall::Error] firewall_dup_rule() nowhere to duplicate!\n");
return ERROR;
}
```

## 漏洞利用

本题漏洞很明显，首先在`delete`函数中不存在UAF，会设置对应free entry为NULL。
但是在 `firewall_dup_rule` 中用对应idx的entry来填充了list中查找到的第一个为NULL的entry。

如果我们先一步调用 `firewall_dup_rule` ，相当于在（对称的）list上放了同一个entry的一个copy，我们free掉一个，再利用对称的list里剩下的没有设置为NULL的copy即可完成UAF。
这离要注意 `dup = (user_rule.type == INBOUND) ? firewall_rules_out : firewall_rules_in;`

也就是说dup的时候`firewall_rules_out`链表的会dup到`firewall_rules_in`；`firewall_rules_in`会dup到`firewall_rules_out`。是一个对称的dup，而不是在同一个list中dup。

并且由于flag标志位同一个entry只能dup一次。

我们可以UAF的对象属于`kmalloc-4096`，并且我们可以配合`edit`完成UAF-write，但是`edit`限制为只能任意写UAF对象部分长度。

### 利用 msg_msg 对象堆喷射构造任意读写

#### msg_msg 对象

```c
/* one msg_msg structure for each message */
struct msg_msg {
struct list_head m_list;
long m_type;
size_t m_ts; /* message text size */
struct msg_msgseg *next;
void *security; //无SELinux，这里为NULL
/* the actual message follows immediately */
};
```

根据 ：<a href="elink@805K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0u0Q4x3V1k6E0M7$3N6G2M7q4)9J5k6e0u0Q4x3X3g2Z5N6r3#2D9">manpage</a>

这个对象在上层主要对应的操作是 `msgsnd()` 还有 `msgrcv()`
```c
struct msg_msgseg {
struct msg_msgseg *next;
/* the next part of the message follows immediately */
};
```
**`int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg)`**
| msqid | 消息队列标识符 |
| ------ | ------------------------------------------------------------ |
| msgp | 发送给队列的消息。msgp可以是任何类型的结构体，但第一个字段必须为long类型，即表明此发送消息的类型，msgrcv根据此接收消息。msgp定义的参照格式如下： struct msgbuf { long mtype; /* message type, must be > 0 */ char mtext[msgsz]; /* message data 大小由msgsz指定，消息正文 */ }; |
| msgsz | 消息大小 |
| msgflg | 标志位。0：当消息队列满时，msgsnd将会阻塞，直到消息能写进消息队列。IPC_NOWAIT：当消息队列已满的时候，msgsnd函数不等待立即返回。IPC_NOERROR：若发送的消息大于size字节，则把该消息截断，截断部分将被丢弃，且不通知发送进程。 |
**`ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp,int msgflg);`**
| msgp | 存放消息的结构体，结构体类型要与msgsnd函数发送的类型相同 |
| ------ | ------------------------------------------------------------ |
| msgsz | 要接收消息的大小，不含消息类型占用的4个字节 |
| msgtyp | 0：接收第一个消息。>0：接收类型等于msgtyp的第一个消息。<0：接收类型等于或者小于msgtyp绝对值的第一个消息。 |
| msgflg | 0: 阻塞式接收消息，没有该类型的消息msgrcv函数一直阻塞等待。IPC_NOWAIT：如果没有返回条件的消息调用立即返回，此时错误码为ENOMSG。IPC_EXCEPT：与msgtype配合使用返回队列中第一个类型不为msgtype的消息。IPC_NOERROR：如果队列中满足条件的消息内容大于所请求的size字节，则把该消息截断，截断部分将被丢弃。 |
这两个syscall主要用于发送message到系统的消息队列，然后从消息队列接收message。调用他的进程必须对消息队列有写权限才能发送消息；有读权限才能接收消息。
在内核中 `msgsnd` 会到达 `do_msgsnd` ：

```c
#0 0xffffffff8159eb44 in do_msgsnd ()
#1 0xffffffff8146dfcf in __x64_sys_msgsnd ()
#2 0xffffffff81004ea6 in do_syscall_64 ()
```

**`do_msgsnd`**
