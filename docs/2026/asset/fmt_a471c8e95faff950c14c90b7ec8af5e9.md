
```
/*代码5-2*/

/*linux-5.4\linux-5.4\net\bluetooth\hci_core.c*/

/* ACL data packet */

static void hci_acldata_packet(struct hci_dev *hdev, struct sk_buff *skb)

{

	struct hci_acl_hdr *hdr = (void *) skb->data;     /*取出HCI包头*/

     struct hci_conn *conn;

     skb_pull(skb, HCI_ACL_HDR_SIZE);           /*data指针指向L2CAP头*/

     handle = __le16_to_cpu(hdr->handle);     /*取出handle字段*/

	flags  = hci_flags(handle);                      /*从handle字段中取出PB和BC*/

	handle = hci_handle(handle);                   /*从handle字段中取出handle*/

	   /*……*/

	handle = __le16_to_cpu(hdr->handle);    /*取出连接句柄*/

	conn = hci_conn_hash_lookup_handle(hdev, handle);

	if (conn) {

		/* Send to upper protocol */

		l2cap_recv_acldata(conn, skb, flags);

		return;

	} 

	/*……*/
```

`hci_acl_data_packet`从套接字缓冲取出`HCI`数据包头中的连接句柄，该句柄作为哈希值查找到到hci连接信息`conn`。`flags`为从`HCI`头部提取出的`PB`和`BC`的值，用于判断`L2CAP`分组的位置。调用`skb_pull`函数将套接字缓冲的`data`指针指向`L2CAP`数据包头。接下来调用`l2cap_recv_acldata`，此时数据从`HCI`层进入到`L2CAP`层。

![](https://bbs.kanxue.com/upload/attach/202101/741085_2JWHXK6XPAW5XCF.jpg)

图5-3 L2CAP层套接字缓冲

```
/*代码5-3*/

/*linux-5.4\linux-5.4\net\bluetooth\l2cap_core.c*/

void l2cap_recv_acldata(struct hci_conn *hcon, struct sk_buff *skb, u16 flags)

{

	struct l2cap_conn *conn = hcon->l2cap_data;

	struct l2cap_hdr *hdr;

	int len;

	/*……*/



	switch (flags) {

	case ACL_START:			/*flags=0x2*/

	case ACL_START_NO_FLUSH:

	case ACL_COMPLETE:



		/……/



		/* Start fragment always begin with Basic L2CAP header */

		if (skb->len < L2CAP_HDR_SIZE) {

			BT_ERR("Frame is too short (len %d)", skb->len);

			l2cap_conn_unreliable(conn, ECOMM);

			goto drop;

		}

		hdr = (struct l2cap_hdr *) skb->data;

		len = __le16_to_cpu(hdr->len) + L2CAP_HDR_SIZE;

		if (len == skb->len) {

			/* Complete frame received */

			l2cap_recv_frame(conn, skb);

			return;

		}
```

`l2cap_recv_acldata`从hci连接信息中取出l2cap连接信息。`flags`是从`HCI`头部提取出的`PB`值，从`PoC`发送过来的载荷中的`PB=0x2`，下面对`L2CAP`头部的长度信息进行校验，通过校验后进入到`l2cap_recv_frame`函数。

```
/*代码5-4*/

/*linux-5.4\linux-5.4\net\bluetooth\l2cap_core.c*/

static void l2cap_recv_frame(struct l2cap_conn *conn, struct sk_buff *skb)

{

	struct l2cap_hdr *lh = (void *) skb->data;

	struct hci_conn *hcon = conn->hcon;

	u16 cid, len;

	skb_pull(skb, L2CAP_HDR_SIZE);     /*套接字缓冲的data指针指向L2CAP净荷*/

	cid = __le16_to_cpu(lh->cid);       /*取出L2CAP头的CID字段*/

	

         switch (cid) {

    

    		/*……*/

    

    	default:

    		l2cap_data_channel(conn, cid, skb);

    		break;

    	}

    }
```

`l2cap_recv_frame`从`L2CAP`包头取出`CID`的值，将套接字缓冲的`data`指针指向`L2CAP`净荷。根据`CID`的值选择进入相应的信道处理函数。`PoC`发送过来的载荷中`CID=0x3`，因此进入`l2cap_data_channel`函数。

![](https://bbs.kanxue.com/upload/attach/202101/741085_7DECW9MSUFNGVXR.jpg)

图5-4 `data`指向L2CAP净荷

```
/*代码5-5*/

/*linux-5.4\linux-5.4\net\bluetooth\l2cap_core.c*/

static void l2cap_data_channel(struct l2cap_conn *conn, u16 cid, struct sk_buff *skb)

{

	struct l2cap_chan *chan;



	chan = l2cap_get_chan_by_scid(conn, cid);

	if (!chan) {

		if (cid == L2CAP_CID_A2MP) { 		/*L2CAP_CID_A2MP=0x0003*/

			chan = a2mp_channel_create(conn, skb);



		/*……*/



	}

     switch (chan->mode) {



           /*……*/



     case L2CAP_MODE_ERTM:

	case L2CAP_MODE_STREAMING:

		l2cap_data_rcv(chan, skb);



           /*……*/

     }



     /*……*/

    

}
```

`l2cap_data_channel`从l2cap连接信息中取出信道信息`chan`。`PoC`发送的`L2CAP`包的`CID`字段为`0x3`说明`PoC`发过来的是`A2MP`协议包，但并没有通过信令信道协商建立`A2MP`信道的过程，而是直接通过`HCI`套接字构造了`A2MP`包就发往目标机，所以`l2cap_get_chan_by_scid`返回空指针，控制流进入到`a2mp_channel_create`函数。

`a2mp_channel_create`函数层层调用到`a2mp_chan_open`，该函数设置`chan->mode`为`L2CAP_MODE_ERTM`，所以`case`判断中进入到`l2cap_data_rcv`函数中。

```
/*代码5-6*/

/*linux-5.4\linux-5.4\net\bluetooth\l2cap_core.c*/

static int l2cap_data_rcv(struct l2cap_chan *chan, struct sk_buff *skb)

{



	/*……*/



     if (l2cap_check_fcs(chan, skb))

		goto drop;



     /*……*/

      

    if ((chan->mode == L2CAP_MODE_ERTM || chan->mode == L2CAP_MODE_STREAMING) 

		&& sk_filter(chan->data, skb))

		goto drop;
```

`l2cap_data_rcv`先调用`l2cap_check_fcs`对`L2CAP`净荷的`FCS`进行校验，`PoC`的`FCS`生成函数就是直接截取的源码，校验失败就直接丢弃数据包。接下来第二个`if`判断中第一个判断条件`chan->mode == L2CAP_MODE_ERTM`为真，所以执行到`sk_filter`内联函数，该函数展开为`sk_filter_trim_cap`。

```
/*代码5-7*/

/*linux-5.4\linux-5.4\net\core\filter.c*/

int sk_filter_trim_cap(struct sock *sk, struct sk_buff *skb, unsigned int cap)

{

	struct sk_filter *filter;



               /*……*/



	filter = rcu_dereference(sk->sk_filter);

	if (filter) {

		struct sock *save_sk = skb->sk;

		unsigned int pkt_len;



		skb->sk = sk;

		pkt_len = bpf_prog_run_save_cb(filter->prog, skb);   /*出现问题的地方*/



                  /*……*/

	}

             /*……*/
```

注意到`sk_filter_trim_cap`接收的第一个参数是指向`struct sock`的指针，而通过`sk_filter`传进来的却是`chan->data`指针，出现问题的地方调用的`filter=chan->data->sk_filter`。

根据`crash`文件定位最终的问题点在`filter.h`文件的`657`行，所在的`__bpf_prog_run_save_cb`函数实际上是内联函数`bpf_prog_run_save_cb`展开后的结果，具体见代码5-9。

![](https://bbs.kanxue.com/upload/attach/202101/741085_JY7PG64Y5YSTJFU.jpg)

图5-5 问题定位

```
/*代码5-8*/

/*inux-5.4\linux-5.4\include\linux\filter.h*/

static inline u32 __bpf_prog_run_save_cb(const struct bpf_prog *prog, struct sk_buff *skb)

{



	/*……*/



	if (unlikely(prog->cb_access)) {		                              /*657行*/

		memcpy(cb_saved, cb_data, sizeof(cb_saved));



	/*……*/
```

参数调用链中成员的变化为`chan->data->sk_filter->prog->cb_access`，由于在代码5-6中传递给`sk_filter`函数的第一个参数出现了混淆，导致了`chan->data->sk_filter->prog=NULL`，引发了最后一步的`prog->cb_access`发生了空指针引用。

最后我再来研究一下类型混淆是如何导致空指针引用的。

按照程序的正常逻辑，传递给代码5-6中的漏洞函数`sk_filter`的第一个参数是`struct sock *`变量，正常的参数链中的成员变化应该为：

![](https://bbs.kanxue.com/upload/attach/202101/741085_7U62XX5R9KKF7EK.jpg)

图5-7 参数链

在异常参数链中，`chan->data`是在前面`a2mp_channel_create`函数执行期间初始化的，`a2mp_channel_create`函数调用了`amp_mgr_create`来初始化`chan->data`。

```
/*代码5-9*/

/*linux-5.4\linux-5.4\net\bluetooth\a2mp.c*/

static struct amp_mgr *amp_mgr_create(struct l2cap_conn *conn, bool locked)

{

	struct amp_mgr *mgr;

	struct l2cap_chan *chan;

	mgr = kzalloc(sizeof(*mgr), GFP_KERNEL);

     chan = a2mp_chan_open(conn, locked);

	mgr->a2mp_chan = chan;

	chan->data = mgr;

}
```

实际上`chan->data`指向一块类型为`struct amp_mgr`类型的堆空间，大小为`0x70`。显然异常参数链中获取`sk_filter`成员已经超出了堆的范围，所以在后后面的两步均有可能导致访问内存出现问题。实际确实是这样，在这篇分析中捕获到的是`prog+0x2`处的空指针引用，但经过多次运行也捕获到了`sk_filter+0x18`处无法修正的缺页异常。

![](https://bbs.kanxue.com/upload/attach/202101/741085_YDXETARJRKGF3U7.jpg)

图5-8 两种类型的异常

对于`sk_filter+0x18`处无法修正的缺页异常也很好解释。结合`sk_filter_trim_cap`反汇编代码分析：

![](https://bbs.kanxue.com/upload/attach/202101/741085_M5BZ6HAHX34PVY8.jpg)

图5-9 `sk_filter_trim_cap`片段

`[sk+110h]`取出`sk_filter`放入`rax`寄存器中，`rax`也就是代码5-7中`filter=rcu_dereference(sk->sk_filter)`的返回值，如果`rax=0`那么数据包直接被丢弃，所以要走到`mov r15, [rax+18h]`这一步，`rax`必然是内核堆中一个不为0的垃圾值，使用这个值作为地址访问内存将导致不可修正的缺页异常。

## 2. 动态分析

动态调试的思路是追踪执行流，查看关键内核路径是否得到执行和最终达到的内存破坏效果。

综合上面的静态分析，我归纳出几个需要关注的路径点。一是代码5-5中位于`l2cap_data_channel`函数中的`l2cap_get_chan_by_scid`执行结果是否为`NULL`，因为这个关乎到`l2cap_data_rcv`是否得到执行。二是代码5-7中`rcu_dereference`的执行结果，因为`filter`的值直接决定了图5-7所示的参数链的最后两步是否会出现访存错误。

所以2个关键的断点打在`l2cap_get_chan_by_scid`执行完后和`rcu_dereference`执行完后。

首先在目标机中得到`bluetooth`模块的加载基地址为`0xffffffff_c032d000`。

![](https://bbs.kanxue.com/upload/attach/202101/741085_EEEXEBQ9WXUEHYK.jpg)

图5-10 模块基地址

在调试机中启动`gdb`，远程连接到目标机，在目标机中使用`sysrq`断下，这样可以在`gdb`中获得输入调试命令的机会。

![](https://bbs.kanxue.com/upload/attach/202101/741085_BAXABWTGKUQMDZR.jpg)

图5-11

`Gdb`加载`bluetooth.ko`符号文件，`text`段基地址为模块加载基地址。同时在`l2cap_data_channel`下断点，`continue`让目标机继续运行。

![](https://bbs.kanxue.com/upload/attach/202101/741085_P5VHEUYJ5CYUZRE.jpg)

图5-12 加载模块符号文件

在攻击机中运行`poc`向目标机发送恶意数据包，`gdb`此时断下，反汇编`l2cap_data_channel`函数，在`0xffffffff_c035f0eb`的位置下第二个断点，`continue`继续执行。

![](https://bbs.kanxue.com/upload/attach/202101/741085_Y5REKQNDQQESQ8B.jpg)

图5-13

第二个断点处断下，观察到`rax=0`，也就是`l2cap_get_chan_by_scid`返回值为`NULL`，对照代码5-5可知控制流进入了`a2mp_channel_create`函数，这为后面一系列骚操作埋下了伏笔。

![](https://bbs.kanxue.com/upload/attach/202101/741085_DCDQ43DUZJA4AYW.jpg)

图5-14

第三个断点打在`sk_filter_trim_cap`函数的`0xffffffff_819565ef`处，此处对应于代码5-7中函数`rcu_dereference(sk->sk_filter)`，`rax`为返回值对应于`filter`变量。

![](https://bbs.kanxue.com/upload/attach/202101/741085_XH5CW46J96JJSG4.jpg)

图5-15
