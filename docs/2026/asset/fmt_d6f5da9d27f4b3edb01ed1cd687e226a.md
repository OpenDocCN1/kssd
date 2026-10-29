
```cpp
static afp_child_t *dsi_start(AFPObj *obj, DSI *dsi, server_child_t *server_children)
{
afp_child_t *child = NULL;
if (dsi_getsession(dsi, server_children, obj->options.tickleval, &child) != 0) {
LOG(log_error, logtype_afpd, "dsi_start: session error: %s", strerror(errno));
return NULL;
}
// we've forked.
if (child == NULL) {
configfree(obj, dsi);
afp_over_dsi(obj); /* start a session */
exit (0);
}
return child;
}
/*!
* Start a DSI session, fork an afpd process
*
* @param childp (w) after fork: parent return pointer to child, child returns NULL
* @returns 0 on sucess, any other value denotes failure
*/
/* DSI Commands */
#define DSIFUNC_CLOSE 1 /* DSICloseSession */
#define DSIFUNC_CMD 2 /* DSICommand */
#define DSIFUNC_STAT 3 /* DSIGetStatus */
#define DSIFUNC_OPEN 4 /* DSIOpenSession */
#define DSIFUNC_TICKLE 5 /* DSITickle */
#define DSIFUNC_WRITE 6 /* DSIWrite */
#define DSIFUNC_ATTN 8 /* DSIAttention */
#define DSIFUNC_MAX 8 /* largest command */
int dsi_getsession(DSI *dsi, server_child_t *serv_children, int tickleval, afp_child_t **childp)
{
switch (dsi->header.dsi_command) {
case DSIFUNC_STAT: /* send off status and return */
{
/* OpenTransport 1.1.2 bug workaround:
*
* OT code doesn't currently handle close sockets well. urk.
* the workaround: wait for the client to close its
* side. timeouts prevent indefinite resource use.
*/
static struct timeval timeout = {120, 0};
fd_set readfds;
dsi_getstatus(dsi);
FD_ZERO(&readfds);
FD_SET(dsi->socket, &readfds);
free(dsi);
select(FD_SETSIZE, &readfds, NULL, NULL, &timeout);
exit(0);
}
break;
case DSIFUNC_OPEN: /* setup session */
/* set up the tickle timer */
dsi->timer.it_interval.tv_sec = dsi->timer.it_value.tv_sec = tickleval;
dsi->timer.it_interval.tv_usec = dsi->timer.it_value.tv_usec = 0;
dsi_opensession(dsi);
*childp = NULL;
return 0;
default: /* just close */
LOG(log_info, logtype_dsi, "DSIUnknown %d", dsi->header.dsi_command);
dsi->proto_close(dsi);
exit(EXITERR_CLNT);
}
}
/* DSI session options */
#define DSIOPT_SERVQUANT 0x00 /* server request quantum */
#define DSIOPT_ATTNQUANT 0x01 /* attention quantum */
#define DSIOPT_REPLCSIZE 0x02 /* AFP replaycache size supported by the server (that's us) */
/* OpenSession. set up the connection */
void dsi_opensession(DSI *dsi)
{
uint32_t i = 0; /* this serves double duty. it must be 4-bytes long */
int offs;
if (setnonblock(dsi->socket, 1) < 0) {
LOG(log_error, logtype_dsi, "dsi_opensession: setnonblock: %s", strerror(errno));
AFP_PANIC("setnonblock error");
}
/* parse options */
while (i < dsi->cmdlen) {
switch (dsi->commands[i++]){
case DSIOPT_ATTNQUANT: // dsi_header.dsi_data[0]:code, dsi_header.dsi_data[1]:size, dsi_header.dsi_data[2:size -1]:data
memcpy(&dsi->attn_quantum, dsi->commands + i + 1, dsi->commands[i]); // 越界写，上层函数会执行 memcpy(dsi->commands, dsi_header->dsi_data) dsi_header是我们发包的内容
dsi->attn_quantum = ntohl(dsi->attn_quantum);
case DSIOPT_SERVQUANT: /* just ignore these */
default:
i += dsi->commands[i] + 1; /* forward past length tag + length */
break;
}
}
// ...
dsi_send(dsi);
}
```

## DSI 结构体

`DSI` 结构体如下，从 `attn_quantum` 字段往后溢出，我们最多可以溢出至 `data` 数组的部分空间（`data` 数组非常大）。比较关键的是我们可以覆盖指针 `dsi->commands`。在后面的漏洞分析小节，我们会纤细的讨论覆盖 `commands` 指针所导致的严重后果，这将使得我们可以 `RCE`。`afp_start` -> `dsi_getsession` -> `dsi_opensession` 这条路径我们分析至此。大意的作用从两个关键函数名也可以看出来，核心就是 `open session`，配置一些东西并开启正式的连接会话，你可以理解为 `TCP` 建立连接前的三次握手，但是在配置过程中产生了越界写漏洞。

```cpp
typedef struct DSI {
struct DSI *next; /* multiple listening addresses */
AFPObj *AFPobj;
int statuslen;
char status[1400];
char *signature;
struct dsi_block header;
struct sockaddr_storage server, client;
struct itimerval timer;
int tickle; /* tickle count */
int in_write; /* in the middle of writing multiple packets,
signal handlers can't write to the socket */
int msg_request; /* pending message to the client */
int down_request; /* pending SIGUSR1 down in 5 mn */
uint32_t attn_quantum, datasize, server_quantum;
uint16_t serverID, clientID;
uint8_t *commands; /* DSI recieve buffer */ //
uint8_t data[DSI_DATASIZ]; /* DSI reply buffer */
size_t datalen, cmdlen;
off_t read_count, write_count;
uint32_t flags; /* DSI flags like DSI_SLEEPING, DSI_DISCONNECTED */
int socket; /* AFP session socket */
int serversock; /* listening socket */
/* DSI readahead buffer used for buffered reads in dsi_peek */
size_t dsireadbuf; /* size of the DSI readahead buffer used in dsi_peek() */
char *buffer; /* buffer start */
char *start; /* current buffer head */
char *eof; /* end of currently used buffer */
char *end;
#ifdef USE_ZEROCONF
char *bonjourname; /* server name as UTF8 maxlen MAXINSTANCENAMELEN */
int zeroconf_registered;
#endif
/* protocol specific open/close, send/receive
* send/receive fill in the header and use dsi->commands.
* write/read just write/read data */
pid_t (*proto_open)(struct DSI *);
void (*proto_close)(struct DSI *);
} DSI;
```

## afp_over_dsi 处理流程

我们再继续从 `afp_start` -> `afp_over_dsi` 开始看。`afp_over_dsi` 处理正式连接的请求核心再这个 `while` 循环。首先第一行重要代码 `cmd = dsi_stream_receive(dsi);`，`Blocking read on the network socket`，即阻塞地从 `socket` 连接中读取 `dsi steam`，即会解析 `dsi` 流量填充 `dsi` 结构体，也就是反序列化 `dsi` 流量。我们进入快速阅读一下 `dsi_stream_receive` 函数，注意我们关注的是该函数如何从 `socket` 中读取数据填充 `dsi` 结构体。我们可以明显的发现 `block` 变量即是 `DSI Header`，将 `block` copy to ` dsi.header` 中。而其中关键的数据包的 `body` 也即 `payload` 或者说 `dsi data` 是同过一行 `if (dsi_stream_read(dsi, dsi->commands, dsi->cmdlen) != dsi->cmdlen)` 从 `dsi` 结构体的 `buffer` 中读取到 `dsi->commands` 指针指向的内存中，最后其返回值返回 `block[1]`，也就是我们下**图1**中给出的 `DSI Header` 结构中的 `Command` 字段。至此我们了解该函数是如何把 `DSI` 流量数据填充进 `dsi structure`。

> 注3：这里我们就发现了任意地址写漏洞。我们最开始的越界写，第一次发包可以覆盖 `commands`，而后续发包可以往 `commands` 中写入我们希望的数据，指针劫持并可控的写指针指向内存，并且我们发现我们可以写多少是由图1的 `total data length` 这个字段决定的，而这个字段是 4 字节大小且根据 `payload` 的长度计算值，因此我们可以写非常长的 `payload`。

> 任意地址写漏洞是后续 `RCE` 的基。我们之所以可以如此是因为 `dsi->commands` 指针的生命周期与 `dsi` 结构体生命周期是一样的，存在于一次连接中。

我们继续回到 `afp_over_dsi` 的 `while` 循环中。再 `dsi_stream_receive` 函数返回时，其返回值同时也是本次请求的 `Command` 字段，该返回值进入 `Switch` 语句执行对应的 `command` 命令，而我们着重关注最核心的 `case DSIFUNC_CMD` 分支。这个分支的作用简而言之，就是根据我们 `DSI` 数据流中的 `Payload` 字段的数据，执行 `AFP` 回调函数。我们可以发现核心的 `afp_switch` 变量，通过全局搜索 `afp_switch = ` 可以发现两处赋值（如图2所示）。通过变量名和两张回调函数表的内容，可以猜测，一个是未登入或未授权时走 `preauth_switch`，一个是登入成功或授权时走 `postauth_switch`。

> 注4：我们拥有了任意地址写，那么处理得当，我们可以为授权时通过任意地址写将 `afp_switch` 的值改成 `postauth_switch`，我们将可以访问 `postauth_switch` 中的回调函数, 即触发未授权访问。更进一步，我们将 `preauth_switch` 或 `postauth_switch` 表中的一些项改成我们希望的代码段地址、`one_gadget` 等，将触发 `RCE`。因此目前我们已经从漏洞点中分析出未授权访问和 `RCE` 两种利用方式了。

![图片描述](upload/attach/202304/921065_YH9YP4KP7YWZ3K2.png)

![图片描述](upload/attach/202304/921065_VGGXY65H648VGTF.png)
