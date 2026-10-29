
`struct sock *sk;`

`void __user *argp = (void __user *)arg;`

`int pid, err;`

`struct net *net;`

`sock = file->private_data;`

`sk = sock->sk;`

`net = sock_net(sk);`

`if (cmd >= SIOCDEVPRIVATE && cmd <= (SIOCDEVPRIVATE + 15)) {`

`err = dev_ioctl(net, cmd, argp);`

`} else`

到这里`sock_ioctl`没有发现上级调用，那么看哪些地方出现过这个函数

搜索`sock_ioctl`发现

```
static const struct file_operations socket_file_ops = {
        .owner =        THIS_MODULE,
        .llseek =        no_llseek,
        .aio_read =        sock_aio_read,
        .aio_write =        sock_aio_write,
        .poll =                sock_poll,
        .unlocked_ioctl = sock_ioctl,//保存在了 unlocked_ioctl 中
#ifdef CONFIG_COMPAT
        .compat_ioctl = compat_sock_ioctl,
#endif
        .mmap =                sock_mmap,
        .open =                sock_no_open,        /* special open code to disallow open via /proc */
        .release =        sock_close,
        .fasync =        sock_fasync,
        .sendpage =        sock_sendpage,
        .splice_write = generic_splice_sendpage,
        .splice_read =        sock_splice_read,
};
```

`file_operations`结构体中获取

linux大多都是使用`ioctl`系统调用来控制驱动设备的。所以我们跟踪`ioctl`函数

```
SYSCALL_DEFINE3(ioctl, unsigned int, fd, unsigned int, cmd, unsigned long, arg)
{
        struct file *filp;
int error = -EBADF;
        int fput_needed;
        filp = fget_light(fd, &fput_needed);
        if (!filp)
                goto out;
        error = security_file_ioctl(filp, cmd, arg);
        if (error)
                goto out_fput;
        error = do_vfs_ioctl(filp, fd, cmd, arg);
out_fput:
        fput_light(filp, fput_needed);
out:
        return error;
}
```

```
int do_vfs_ioctl(struct file *filp, unsigned int fd, unsigned int cmd,
             unsigned long arg)
{
        int error = 0;
        int __user *argp = (int __user *)arg;
        struct inode *inode = filp->f_path.dentry->d_inode;
        .....
        default:
                if (S_ISREG(inode->i_mode))
                        error = file_ioctl(filp, cmd, arg);
                else
                        error = vfs_ioctl(filp, cmd, arg);
                break;
        }
```

`do_vfs_ioctl` 函数下找到 `vfs_ioctl`

```
static long vfs_ioctl(struct file *filp, unsigned int cmd,
                      unsigned long arg)
{
        int error = -ENOTTY;
        if (!filp->f_op || !filp->f_op->unlocked_ioctl)
                goto out;
        error = filp->f_op->unlocked_ioctl(filp, cmd, arg);
        if (error == -ENOIOCTLCMD)
                error = -ENOTTY;
out:
        return error;
}
```

这样调用漏洞函数的过程就全部串联起来。

### 4，构建触发漏洞条件

首先看下 `ioctl` 有三个参数

`param1`首先打开网络驱动设备（linux大多数驱动都是以文件形式存在的，只有网络驱动例外。想了解详细的可以看下`socket`函数）

`param2` `cmd`，之前就已经找函数调用路径的时候我们就发现

```
#define SIOCIWFIRSTPRIV        0x8BE0
#define WLAN_SET_PACKET_FILTER_PARAMS (SIOCIWFIRSTPRIV + 23)
```

`param3` [/IMG]

首先传入的时候

`if (copy_from_user(&ifr, arg, sizeof(struct ifreq)))`

然后

```
static int iw_set_packet_filter_params(struct net_device *dev, struct iw_request_info *info,
        union iwreq_data *wrqu, char *extra)
{
    hdd_adapter_t *pAdapter = WLAN_HDD_GET_PRIV_PTR(dev);
    tpPacketFilterCfg pRequest = (tpPacketFilterCfg)wrqu->data.pointer;
    return wlan_hdd_set_filter(WLAN_HDD_GET_CTX(pAdapter), pRequest, pAdapter->sessionId);
}
```

强制转换了下我们看下 `iwreq` 的成员

```
struct  iwreq
{
  union
  {
    char  ifrn_name[IFNAMSIZ];  /* if name, e.g. "eth0" */
  } ifr_ifrn;
  /* Data part (defined just above) */
  union  iwreq_data  u;
};
```

```
union  iwreq_data
{
  struct iw_point  data;    /* Other large parameters */
};
```

```
struct  iw_point
{
  void __user  *pointer;  /* Pointer to the data  (in user space) */
  __u16    length;    /* number of fields or size in bytes */
  __u16    flags;    /* Optional params */
};
```

看下`tpPacketFilterCfg`的结构体

```
typedef struct
{
    v_U8_t            filterAction;
    v_U8_t            filterId;
    v_U8_t            numParams;
    struct PacketFilterParamsCfg paramsData [HDD_MAX_CMP_PER_PACKET_FILTER];
}tPacketFilterCfg, *tpPacketFilterCfg;
```

```
struct PacketFilterParamsCfg
{
    v_U8_t              protocolLayer;
    v_U8_t              cmpFlag;
    v_U8_t              dataOffset;
    v_U8_t              dataLength;
    v_U8_t              compareData[8];
    v_U8_t              dataMask[8];
};
```

现在构造触发漏洞的结构体

```
int wlan_hdd_set_filter(hdd_context_t *pHddCtx, tpPacketFilterCfg pRequest,
                            tANI_U8 sessionId)
{
    tSirRcvPktFilterCfgType    packetFilterSetReq = {0};
    tSirRcvFltPktClearParam    packetFilterClrReq = {0};
    int i=0;
    if (pHddCtx->cfg_ini->disablePacketFilter)
    {
        hddLog(VOS_TRACE_LEVEL_FATAL, "%s: Packet Filtering Disabled. Returning ",
                __func__ );
        return 0;
    }
    if (pHddCtx->isLogpInProgress)
    {
       VOS_TRACE(VOS_MODULE_ID_HDD, VOS_TRACE_LEVEL_FATAL,
                                  "%s:LOGP in Progress. Ignore!!!", __func__);
       return -EBUSY;
    }
    /* Debug display of request components. */
    hddLog(VOS_TRACE_LEVEL_ERROR, "%s: Packet Filter Request : FA %d params %d",
            __func__, pRequest->filterAction, pRequest->numParams);
    switch (pRequest->filterAction) 进入条件
    {
        case HDD_RCV_FILTER_SET:
            hddLog(VOS_TRACE_LEVEL_INFO, "%s: Set Packet Filter Request for Id: %d",
                    __func__, pRequest->filterId);
            packetFilterSetReq.filterId = pRequest->filterId;
            if ( pRequest->numParams >= HDD_MAX_CMP_PER_PACKET_FILTER)
            {
                hddLog(VOS_TRACE_LEVEL_ERROR, "%s: Number of Params exceed Max limit %d\n",
                        __func__, pRequest->numParams);
                return -EINVAL;
            }
            packetFilterSetReq.numFieldParams = pRequest->numParams;
            packetFilterSetReq.coalesceTime = 0;
            packetFilterSetReq.filterType = 1;
            for (i=0; i < pRequest->numParams; i++)//因为= pRequest->paramsData[i].dataLength 定义为unsigned char，覆盖长度无法触发漏洞，所以这里要设定pRequest->numParams的值，已确保能覆盖的地址能到达函数返回地址的长度
            {
                packetFilterSetReq.paramsData[i].protocolLayer = pRequest->paramsData[i].protocolLayer;
                packetFilterSetReq.paramsData[i].cmpFlag = pRequest->paramsData[i].cmpFlag;
                packetFilterSetReq.paramsData[i].dataOffset = pRequest->paramsData[i].dataOffset;
                  packetFilterSetReq.paramsData[i].dataLength = pRequest->paramsData[i].dataLength;
                 //这个成员变量定义为unsigned char,最大数字只能设置255个字节。
                packetFilterSetReq.paramsData[i].reserved = 0;
                hddLog(VOS_TRACE_LEVEL_INFO, "Proto %d Comp Flag %d Filter Type %d\n",
                        pRequest->paramsData[i].protocolLayer, pRequest->paramsData[i].cmpFlag,
                        packetFilterSetReq.filterType);
                hddLog(VOS_TRACE_LEVEL_INFO, "Data Offset %d Data Len %d\n",
                        pRequest->paramsData[i].dataOffset, pRequest->paramsData[i].dataLength);
                /*因为没有对长度进行判断，而packetFilterSetReq又属于局部变量，导致了我们可以设置适量的长度，覆盖掉wlan_hdd_set_filter的返回地址，让其跳转到自己所写函数的，进行提权操作.*/
                memcpy(&packetFilterSetReq.paramsData[i].compareData,
                        pRequest->paramsData[i].compareData, pRequest->paramsData[i].dataLength);
                memcpy(&packetFilterSetReq.paramsData[i].dataMask,
                        pRequest->paramsData[i].dataMask, pRequest->paramsData[i].dataLength);
                hddLog(VOS_TRACE_LEVEL_INFO, "CData %d CData %d CData %d CData %d CData %d CData %d\n",
