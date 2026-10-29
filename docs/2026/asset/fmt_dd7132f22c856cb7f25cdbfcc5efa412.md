
```c
.standard         = (iw_handler *)we_handler,

.private          = (iw_handler *)we_private,

.private_args     = we_private_args,

.get_wireless_stats = get_wireless_stats,

};
```

发现最后赋值给了`const struct iw_handler_def we_handler_def`中的`.private`。

```c
int hdd_register_wext(struct net_device *dev)
{
    ......

    if (!VOS_IS_STATUS_SUCCESS(vos_event_init(&pwextBuf->scanevent)))
    {
        VOS_TRACE(VOS_MODULE_ID_HDD, VOS_TRACE_LEVEL_ERROR, ("ERROR: HDD scan event init failed!!\n"));
        return eHAL_STATUS_FAILURE;
    }

    // Register as a wireless device

    dev->wireless_handlers = (struct iw_handler_def *)&we_handler_def; // 搜索文件看哪里使用了dev->wireless_handlers

    EXIT();

    return 0;
}
```

到这步之后，再继续看调`we_handler_def`的调用点。

发现最后给了`dev->wireless_handlers`这个成员变量。现在我们在使用`lookup References`功能，看看哪些地方使用了`dev->wireless_handlers`。

发现很多地方都使用了`dev->wireless_handlers`这个成员变量，那么现在来根据条件（看哪些地方是获取这个成员变量）来排除。

最后发现，只有这一处是获取到了`dev->wireless_handlers`这个成员变量。

然后打开这个目录文件搜索（目录：`net\wireless\Wext-core.c`）。

```c
static iw_handler get_handler(struct net_device *dev, unsigned int cmd)
{
    /* Don't "optimise" the following variable, it will crash */

    unsigned int index; /* *MUST* be unsigned */

    const struct iw_handler_def *handlers = NULL;

#ifdef CONFIG_CFG80211_WEXT

    if (dev->ieee80211_ptr && dev->ieee80211_ptr->wiphy)
        handlers = dev->ieee80211_ptr->wiphy->wext;

#endif

#ifdef CONFIG_WIRELESS_EXT

    if (dev->wireless_handlers)
        handlers = dev->wireless_handlers;

#endif

    if (!handlers)
        return NULL;

    /* Try as a standard command */

    index = IW_IOCTL_IDX(cmd);

    if (index < handlers->num_standard)
        return handlers->standard[index];

#ifdef CONFIG_WEXT_PRIV

    /* Try as a private command */

    index = cmd - SIOCIWFIRSTPRIV;

    if (index < handlers->num_private)
        return handlers->private[index]; // 获取了漏洞函数地址

#endif

    /* Not found */

    return NULL;
}
```

找到了`get_handler`这个函数，看它下面的代码：

```c
index = cmd - SIOCIWFIRSTPRIV;

if (index < handlers->num_private)
    return handlers->private[index]; // 获取了漏洞函数地址
```

发现它是根据传入的`CMD`指令来获取函数地址。根据之前的`we_private`存放地址：

```c
[WLAN_SET_PACKET_FILTER_PARAMS - SIOCIWFIRSTPRIV] = iw_set_packet_filter_params
```

得出了`CMD`值。

```c
static int wireless_process_ioctl(struct net *net, struct ifreq *ifr,
                                  unsigned int cmd,
                                  struct iw_request_info *info,
                                  wext_ioctl_func standard,
                                  wext_ioctl_func private)
{
    struct iwreq *iwr = (struct iwreq *)ifr;

    struct net_device *dev;

    iw_handler handler;

    ......

    /* Basic check */

    if (!netif_device_present(dev))
        return -ENODEV;

    /* New driver API: try to find the handler */

    handler = get_handler(dev, cmd); // 获取了漏洞函数地址

    if (handler) {
        /* Standard and private are not the same */

        if (cmd < SIOCIWFIRSTPRIV)
            return standard(dev, iwr, cmd, info, handler);
        else if (private)
            return private(dev, iwr, cmd, info, handler);
    }

    /* Old driver API: call driver ioctl handler */

    .....
}
```

现在继续看`get_handler`的调用点。

根据图片代码看到，将调用漏洞的函数地址作为参数，然后调用了`standard`或`private`，发现这两个函数都是作为参数传入进来的。

（`iwr`就是漏洞产生的条件，`struct iwreq *iwr = (struct iwreq *)ifr;`，`ifr`则作为参数传入。）

那么再继续看上级调用。

```c
static int wext_ioctl_dispatch(struct net *net, struct ifreq *ifr,
                               unsigned int cmd, struct iw_request_info *info,
                               wext_ioctl_func standard,
                               wext_ioctl_func private)
{
    int ret = wext_permission_check(cmd);

    if (ret)
        return ret;

    dev_load(net, ifr->ifr_name);

    rtnl_lock();

    ret = wireless_process_ioctl(net, ifr, cmd, info, standard, private);

    rtnl_unlock();

    return ret;
}
```

发现我们所感兴趣的值还是作为参数传入的，那么继续往上：

```c
int wext_handle_ioctl(struct net *net, struct ifreq *ifr, unsigned int cmd,
                      void __user *arg)
{
    struct iw_request_info info = { .cmd = cmd, .flags = 0 };

    int ret;

    ret = wext_ioctl_dispatch(net, ifr, cmd, &info,
                              ioctl_standard_call,
                              ioctl_private_call); // ioctl_private_call中调用了漏洞函数

    if (ret >= 0 &&
        IW_IS_GET(cmd) &&
        copy_to_user(arg, ifr, sizeof(struct iwreq)))
        return -EFAULT;

    return ret;
}
```

现在得到了这两个函数的地址。

查看`ioctl_private_call`的代码：

```c
int ioctl_private_call(struct net_device *dev, struct iwreq *iwr,
                       unsigned int cmd, struct iw_request_info *info,
                       iw_handler handler)
{
    int extra_size = 0, ret = -EINVAL;

    const struct iw_priv_args *descr;

    extra_size = get_priv_descr_and_size(dev, cmd, &descr);

    /* Check if we have a pointer to user space data or not. */

    if (extra_size == 0) {
        /* No extra arguments. Trivial to handle */

        ret = handler(dev, info, &(iwr->u), (char *)&(iwr->u)); // handler：漏洞函数，iwr->u：触发条件
    } else {
        ret = ioctl_private_iw_point(&iwr->u.data, cmd, descr,
                                     handler, dev, info, extra_size);
    }

    /* Call commit handler if needed and defined */

    if (ret == -EIWCOMMIT)
        ret = call_commit_handler(dev);

    return ret;
}
```

可以看到这里调用了漏洞函数。`&(iwr->u)`则是漏洞产生的条件。

```c
int dev_ioctl(struct net *net, unsigned int cmd, void __user *arg)
{
    struct ifreq ifr;

    int ret;

    char *colon;

    /* One special case: SIOCGIFCONF takes ifconf argument
       and requires shared lock, because it sleeps writing
       to user space.
     */

    if (cmd == SIOCGIFCONF) {
        rtnl_lock();

        ret = dev_ifconf(net, (char __user *)arg);

        rtnl_unlock();

        return ret;
    }

    if (cmd == SIOCGIFNAME)
        return dev_ifname(net, (struct ifreq __user *)arg);

    if (copy_from_user(&ifr, arg, sizeof(struct ifreq))) // 给ifr结构体赋值
        return -EFAULT;

    ........

    ret = -EFAULT;

    return ret;

    /* Take care of Wireless Extensions */

    if (cmd >= SIOCIWFIRST && cmd <= SIOCIWLAST)
        return wext_handle_ioctl(net, &ifr, cmd, arg);

    return -ENOTTY;
}
```

现在继续看`wext_handle_ioctl`函数（注意：`ifr`参数就是漏洞的产生条件），继续看上级调用。

```c
int dev_ioctl(struct net *net, unsigned int cmd, void __user *arg)
{
    struct ifreq ifr;

    int ret;

    char *colon;

    /* One special case: SIOCGIFCONF takes ifconf argument
       and requires shared lock, because it sleeps writing
       to user space.
     */

    if (cmd == SIOCGIFCONF) {
        rtnl_lock();

        ret = dev_ifconf(net, (char __user *)arg);

        rtnl_unlock();

        return ret;
    }

    if (cmd == SIOCGIFNAME)
        return dev_ifname(net, (struct ifreq __user *)arg);

    if (copy_from_user(&ifr, arg, sizeof(struct ifreq)))
        return -EFAULT;
}
```

看函数头，发现`ifr`的内容是拷贝的`dev_ioctl`中的`arg`参数。

```c
static long sock_ioctl(struct file *file, unsigned cmd, unsigned long arg)
{
    struct socket *sock;
```
