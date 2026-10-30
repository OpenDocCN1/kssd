
> `ntoh`族函数通常用来进行网络字节序到主机字节序的转换。所以这里的意思应该是`noth`族函数会将外部传入的数据包中的某些数据转换一个数值，而这个数值可能最终会被传给`memcpy`作为`size`参数，使得拷贝的长度被攻击者控制，就可能会产生安全风险。

- Hint: The source should be an instance of the class you wrote in part 2.0.

- Hint: The sink should be the size argument of calls to `memcpy`.

- 下面这个是模板

```sql
/\*\*
\* @kind path-problem
\*/
import cpp
import semmle.code.cpp.dataflow.TaintTracking
import DataFlow::PathGraph
class YOUR\_CLASS\_HERE extends Expr {
// 2.0 Todo
}
class Config extends TaintTracking::Configuration {
Config() { this = "NetworkToMemFuncLength" }
override predicate isSource(DataFlow::Node source) {
// 2.1 Todo
}
override predicate isSink(DataFlow::Node sink) {
// 2.1Todo
}
from Config cfg, DataFlow::PathNode source, DataFlow::PathNode sink
where cfg.hasFlowPath(source, sink)
select sink, source, sink, "ntoh flows to memcpy"
```

先研究一下模板，先从“main函数”部分开始看：

```sql
// 1. 声明3个变量，Config类型的cfg，PathNode类型的source和sink
from Config cfg, DataFlow::PathNode source, DataFlow::PathNode sink
// 2. 将source 和 sink 作为参数传入hasFlowPath。顾名思义，也就是判断是否有从source到sink的数据流动的path
where cfg.hasFlowPath(source, sink)
// 3. 打印sink 和source，不过为啥要打印两遍sink？
select sink, source, sink, "ntoh flows to memcpy"
```

看看`cfg.hasFlowPath`的描述，跳到定义看看：

```sql
/\*\*
\* Holds if data may flow from `source` to `sink` for this configuration.
\*
\* The corresponding paths are generated from the end-points and the graph
\* included in the module `PathGraph`.
\*/
相应的path是通过模块PathGraph里的end-points和graph来生成的
```

再来看看Configuration类的子类：

```sql
class Config extends TaintTracking::Configuration {
Config() { this = "NetworkToMemFuncLength" }
override predicate isSource(DataFlow::Node source) {
// 2.1 Todo
}
override predicate isSink(DataFlow::Node sink) {
// 2.1Todo
}
```

> 提炼一下官方文档 <a href="elink@ea5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7h3I4Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3c8W2M7h3I4Q4x3X3c8K6N6r3q4F1k6r3q4J5k6q4)9J5k6r3I4A6j5Y4u0S2M7X3W2W2M7#2)9J5c8X3y4H3M7q4)9J5c8Y4y4W2L8h3#2D9k6g2)9J5c8X3y4G2k6r3g2Q4x3V1k6U0M7s2m8Q4x3V1k6V1j5i4c8S2k6X3I4G2N6#2)9J5c8X3W2F1N6r3g2J5L8X3q4D9i4K6u0r3N6r3q4A6L8Y4c8@1M7X3q4U0K9$3W2F1k6K6q4Q4x3V1k6f1j5h3W2F1N6q4c8J5j5h3y4C8K9h3&6Y4d9h3#2H3L8q4)9J5k6i4q4D9L8q4)9J5c8Y4c8&6M7r3g2Q4x3X3g2f1j5h3W2F1N6q4c8J5j5h3y4C8K9h3&6Y4d9h3#2H3L8q4)9J5y4p5y4G2L8X3k6A6k6%4g2J5j5i4c8A6L8$3&6Q4x3X3g2Z5N6r3#2D9">configuration</a>对这个类的描述：

> 1. 该类定义了对于一个analysis来说，sources、sinks等可配置的选项。也就是说这是一个包含谓词的类，这些谓词定义了数据如何在`source`和`sink`之间流动

> 2. 为了创建一个configuration，需要创建一个继承该类的子类。它的characteristic predicate（可看作构造函数）是一个独一无二的字符串，比如模版中写的`NetworkToMemFuncLength`

> 3. Configuration类的override member predicate（可看作成员函数）中，必须要覆写的有`isSource`和`isSink`，其他都是可选的。

> 4. 为了查询是否有从`source`到`sink`的流，写法如下：`exists(MyAnalysisConfiguration cfg | cfg.hasFlow(source, sink))`

> 5. 多个Configuration可以共存，但是不支持在一个Configuration类的override predicate里去写怎么依赖另一个Configuration。hmm...这个还是得看例子才知道怎么写。

按照一般写代码的经验，`isSource`和`isSink`这两个is开头的谓词（函数）应该是写：判断这个source/sink要满足什么条件，然后返回true/false。但是呢，搂一眼<a href="elink@79eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7h3I4Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4K6i4K6u0r3M7h3I4Q4x3X3c8D9j5h3&6Y4N6h3q4Y4k6g2)9J5k6s2u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8Y4m8J5k6h3c8A6j5$3q4@1k6i4y4Q4x3V1j5`.">介绍predicate的官方文档</a>，按照文档上面的例子来看，只需要写需要满足什么条件就ok：

> Predicates are used to describe the logical relations that make up a QL program. 谓词用来描述QL程序里的逻辑关系

> Strictly speaking, a predicate evaluates to a set of tuples. 严格来讲，谓词的计算结果是一组元组

```sql
predicate isCountry(string country) {
country = "Germany"
or
country = "Belgium"
or
country = "France"
}
```

那大概就知道怎么写这两个谓词了：

```sql
/\*\*
\* @kind path-problem
\*/
import cpp
import semmle.code.cpp.dataflow.TaintTracking
import DataFlow::PathGraph
class MyExpr extends Expr {
// 2.0 Todo
MacroInvocation mi;
MyExpr(){
mi.getMacro().getName().regexpMatch("ntoh(l|ll|s)") and this = mi.getExpr()
}
}
class Config extends TaintTracking::Configuration {
Config() { this = "NetworkToMemFuncLength" }
override predicate isSource(DataFlow::Node source) {
// 2.1 Todo
source.asExpr() instanceof MyExpr
}
override predicate isSink(DataFlow::Node sink) {
// 2.1Todo
// 存在memcpy的函数调用，并且该调用的第二个参数是sink.asExpr()
exists(FunctionCall fc | fc.getTarget().hasName("memcpy") | sink.asExpr() = fc.getArgument(2))
}
}
from Config cfg, DataFlow::PathNode source, DataFlow::PathNode sink
where cfg.hasFlowPath(source, sink)
select sink, source, sink, "ntoh flows to memcpy"
```

![image-20230610172948004](upload/attach/202306/830671\_SCAAHPEMUBGXWYC.jpg)

## 5 Step 3: Find additional vulnerabilities

## 根据Q2.1查询结果进行代码审计

**Question 3.0: There are 13 known vulnerabilities in U-Boot.**

The query you completed above probably found 9 of them. See if you can refine your query to find 1 or more additional vulnerabilities.

上面完成的查询可能找到13个漏洞中的9个。这里让我们尝试改进查询，以找到更多的漏洞。

在改进前，我们先根据前面的查询结果看看是不是已经能找到9个漏洞了。

### CVE-2019-14192、 CVE-2019-14193 和 CVE-2019-14199

结果【1】和【2】的source是在函数`net_process_received_packet`的末尾，此处很明显会发生integer underflow：

![image-20230610220449399](upload/attach/202306/830671\_65PKERC296HGCU5.jpg)

然后将underflow的整数传递进函数`nc_input_packet`，但是第149行检测了len的大小，这样我觉得即使传入一个很大的len，这里的memcpy也不会发生越界写。。然而我查了一下，<a href="elink@888K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3I4S2j5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6h3u0G2L8%4c8Q4x3X3c8J5j5$3g2Q4x3X3c8F1k6Y4y4Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3">U-Boot NFS RCE Vulnerabilities (CVE-2019-14192)</a>说这里是会越界写的。。。但是他就一句话，没懂。。。

![image-20230610221242234](upload/attach/202306/830671\_W7VAMQGAXHMMGPG.jpg)

然后找了一下<a href="elink@bf6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2V1k6h3&6^5i4K6u0W2k6r3g2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3i4K6u0V1i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3k6X3f1%4x3U0R3^5x3o6j5&6k6o6u0W2y4U0j5#2z5e0p5I4y4K6l9@1z5h3j5%4k6o6t1%4k6e0t1$3x3h3t1#2y4e0m8T1j5U0M7J5y4b7`.`.">patch</a>，对应的编号有CVE-2019-14192、 CVE-2019-14193 和 CVE-2019-14199。三个？

![image-20230611024205527](upload/attach/202306/830671\_5U6QJVVNZZZPDST.jpg)

### CVE-2019-14197、CVE-2019-14200、 CVE-2019-14201、CVE-2019-14202、CVE-2019-14203 和 CVE-2019-14204

结果【1】【2】的source边上很明显还有一个整数回绕，将underflow的整数传递给函数指针`udp_packet_handler`，QL的查询结果里没有此处。

![image-20230611015647333](upload/attach/202306/830671\_H6GPB6KQMPRPD2M.jpg)

先审计一下是否存在漏洞，搜索`udp_packet_handler`，只有两处赋值的地方：

![image-20230611015913129](upload/attach/202306/830671\_Z7VRQ6VR24ADBXJ.jpg)

再搜索函数`net_set_udp_handler`，可以看到分别给nc、dhcp、bootp和nfs都设置了handler。

![image-20230611020024472](upload/attach/202306/830671\_RKRV8F82YV8H45F.jpg)

不想一个一个审计了，直接看文章<a href="elink@358K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3I4S2j5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6h3u0G2L8%4c8Q4x3X3c8J5j5$3g2Q4x3X3c8F1k6Y4y4Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3">U-Boot NFS RCE Vulnerabilities (CVE-2019-14192)</a>知道漏洞都存在于`nfs_handler`。那就只审计它。可以看到，该函数仍然未校验len，直接在不同分支里将其分别传递给函数:

```c
static void nfs_handler(uchar *pkt, unsigned dest, struct in_addr sip,
unsigned src, unsigned len)
{
int rlen;
int reply;
debug("%s\n", __func__);
if (dest != nfs_our_port)
return;
switch (nfs_state) {
case STATE_PRCLOOKUP_PROG_MOUNT_REQ:
if (rpc_lookup_reply(PROG_MOUNT, pkt, len) == -NFS_RPC_DROP)
break;
......
break;
case STATE_PRCLOOKUP_PROG_NFS_REQ:
if (rpc_lookup_reply(PROG_NFS, pkt, len) == -NFS_RPC_DROP)
break;
......
break;
case STATE_MOUNT_REQ:
reply = nfs_mount_reply(pkt, len);
......
break;
case STATE_UMOUNT_REQ:
reply = nfs_umountall_reply(pkt, len);
......
break;
case STATE_LOOKUP_REQ:
reply = nfs_lookup_reply(pkt, len);
......
break;
case STATE_READLINK_REQ:
reply = nfs_readlink_reply(pkt, len);
......
break;
case STATE_READ_REQ:
rlen = nfs_read_reply(pkt, len);
......
break;
}
}
```

- `rpc_lookup_reply` ->未检验len，直接`memcpy(&rpc_pkt.u.data[0], pkt, len);` -> 存在越界写漏洞
