
# 随机用户名后缀

在护网时遇到比较多的一个问题，就是开着热点上的`cs`，中途去吃饭了结果回来再连上热点就会显示该用户已使用，导致就要重启客户端，因此在此处新增用户名后缀防止重复登录。

主要修改点在`server/ManageUser.java`中，在`process`函数中新增：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_EJ8ZEQ6FVETXKKP.webp)

这个随机字符串我控制在长度为3，对应实现的类为：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_FAV4WQ9UT7ZKC27.webp)

最终效果就是这样，终于解决了这个烦人的问题：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_ENA5UNPB5HW2ZZB.webp)

# aggressor 新增监听器时默认显示远程 IP

在原版CS中，每次新增监听器时默认值为本地的IP地址，还需要手动改为远程IP地址，因此在这里对这部分代码进行修改：

`Connect.java`：新增`public static String aggressorRemoteIp = "";`

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_NNPJD8RUK7NGTVX.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_MU4GNF2G9UDAFK3.webp)

然后在`ScListenerDialog.java`修改显示参数：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_99Y4M2BRUUYK35B.webp)

`show_http`和`show_https`都改掉：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_JQVVMHK6A4KUFZK.webp)

最后更改效果为：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_BW5CS9MMRP9XUS2.webp)

# CNA 脚本触发java端内置函数

这个操作就跟上面 Event log显示远程IP差不多，主要单独列出来方便直接看。

`default.cna`：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_QN4NVE9D9H2RW4M.webp)

|  |  |
| --- | --- |
| 1 | `item("&New Connection", { openConnectDialog(); });` |

调用的是`openConnectDialog()`，这个在java端的`aggressor/bridges/AggressorBridges.java`中进行判断，如果点击了这个，则调用这个文件中的代码。`AggressorBridges.java`先是注册，再是响应：

注册：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_B4GH9Y2RKV8A9CW.webp)

响应：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_XBN49BS7VDBEM4Q.webp)

最后执行的代码为：

|  |  |
| --- | --- |
| 1 | `(new ConnectDialog(this.window)).show();` |

这个代码是在`aggressor/dialogs/ConnectDialog.java`中的类。实际上aggressor界面上的那些按钮的触发都是这个套路，因此我们后续如果不想用纯粹的cna脚本，就可以用cna脚本触发，函数写死在java端，好处就是把一些不改动的cna脚本写死在里面，不用拷贝给队友时还需要额外附带脚本。cna脚本写死并加载的地方在`aggressor/AggressorClient.java`中：

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_MFHB2XY6UH9YQ6Z.webp)

# aggressor 提取监听器信息

这个没什么绕的地方，主要就是分析代码，我直接贴提取方式了：

|  |  |
| --- | --- |
| 1<br>2<br>3 | `String lname = DialogUtils.string(var2, "listener");` // 监听器名称，通过 `lname` 可获得 `lhost` 和 `lport`<br>`String lhost = ListenerUtils.getListener(this.client, lname).getCallbackHosts();` // 监听器IP<br>`String lport = String.valueOf(ListenerUtils.getListener(this.client, lname).getPort());` // 监听器端口 |

我之所以弄这个，是之前想将免杀模块放在本地来进行操作，后来这个方案被我废弃了，打算将免杀流程放在teamserver端进行操作，再回传至aggressor。

# aggressor 发送信号

发送信号我是分析的钓鱼模块，aggressor交互都在`aggressor/dialogs`里面，发送信号使用的是`TeamQueue`类，CS的通信主要用的就是这个类：

|  |  |
| --- | --- |
| 1<br>2 | `protected TeamQueue conn = null;`<br>`this.conn.call("SendSign.test", CommonUtils.args(var1, var2, var3, checksum));` |

其中`"SendSign.test"`代表的是发送数据的标签，服务端需要根据这个标签进行判断，后面的几个都是传递的参数。我自己实现的调用逻辑为：

`dialogAction() -> send() -> final_send()`

|  |  |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16 | `public void dialogAction(ActionEvent var1, Map var2) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`String lname = DialogUtils.string(var2, "listener");` // 监听器名称，通过 `lname` 可获得 `lhost` 和 `lport`<br>&nbsp;&nbsp;&nbsp;&nbsp;`String lhost = ListenerUtils.getListener(this.client, lname).getCallbackHosts();` // 监听器IP<br>&nbsp;&nbsp;&nbsp;&nbsp;`String lport = String.valueOf(ListenerUtils.getListener(this.client, lname).getPort());` // 监听器端口<br><br>&nbsp;&nbsp;&nbsp;&nbsp;`String[] stringArray = {lname, lhost, lport};`<br>&nbsp;&nbsp;&nbsp;&nbsp;`this.send(var1, var2, stringArray);` // `var1` 与 `var2` 固定，`var3` 传递传给 `final_send` 的参数<br>&nbsp;&nbsp;&nbsp;&nbsp;`}`<br><br>&nbsp;&nbsp;&nbsp;&nbsp;`public void send(ActionEvent var1, Map var2, String[] var3) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`this.final_send(var3[0], var3[1], var3[2]);`<br>&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;`private void final_send(String var1, String var2, String var3) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`this.conn.call("SendSign.test", CommonUtils.args(var1, var2, var3, checksum));`<br>&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("发送的checksum为：" + checksum);`<br>&nbsp;&nbsp;&nbsp;&nbsp;`}` |

# server 接收信号

与`dialog.java`成对存在的就是server目录中的文件，我这自己新建了一个类，具体接收信号的函数为：

|  |  |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13 | `public void call(Request var1, ManageUser var2) {` // 实际运行的函数<br>&nbsp;&nbsp;&nbsp;&nbsp;`String var4;`<br>&nbsp;&nbsp;&nbsp;&nbsp;`if (var1.is("SendSign.test", 4)) {` // 前三个参数跟编译相关，第四个为 checksum 校验值<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`synchronized(this) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`int result = genCrossC2((String) var1.arg(0), (String) var1.arg(1), (String) var1.arg(2));` // 打印输入的参数，需要将 object 强制转为 string<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`CommonUtils.print_info("result: " + result);`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`if (result == 0) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`this.resources.broadcast("genCrossC2", (String) var1.arg(3), true);`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("send checksum: " + (String) var1.arg(3));`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>`}` |

其中比较重要的就是`var1.is("SendSign.test", 4)`，这个与aggressor传来的数据是相对应的，4为四个参数。

# server 进行广播

广播的代码为：

|  |  |
| --- | --- |
| 1 | `this.resources.broadcast("genCrossC2", (String) var1.arg(3), true);` |

这个`true`的含义貌似是aggressor重连后还会接收到这个信息，我改为`false`也没什么变化，广播的数据为`Map`类型。

# aggressor 接收广播

接收广播部分有个巨坑，如果直接使用`TeamQueue`提供的`setSubscriber`回调函数来接收广播，则会导致CS的UI不会刷新。存在问题的代码为：

|  |  |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14 | `conn.setSubscriber(new Callback() {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`@Override`<br>&nbsp;&nbsp;&nbsp;&nbsp;`public void result(String call, Object content) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`//`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`if ("genCrossC2".equals(call)) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("接收到 genCrossC2 广播！");`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`if (content instanceof String) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`String contentString = (String) content;`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`if ("OK".equals(contentString)) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("广播的内容是 'OK'！");`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>&nbsp;&nbsp;&nbsp;&nbsp;`}`<br>`});` |

这个代码虽然可以正确接收，但是UI不刷新肯定是存在问题的，后来换了一个方案：

|  |  |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6 | `this.conn.setSubscriber(this.data);`<br>`String response = data.getDataSafe("genCrossC2").toString();`<br>`if (checksum.equals(response)) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`System.out.println("接收到服务端广播的内容是：" + checksum);`<br>&nbsp;&nbsp;&nbsp;&nbsp;`break;`<br>`}` |

在使用这个时，一定要注意是否`data`中存在这个键值对，如果不存在就会空指针异常，因此我在`dialogAction`中去检索`data`数据前先使用`this.data.put("key", "value");`进行了置空操作，问题就解决了。

# 总结

以上是基于下面的模型进行讲解的，掌握这个模型一系列操作方式后CS的可玩性还是非常高的。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/951654_3X5Q7PZXKY7VT9D.webp)

本人也是刚刚学习java开发，文中可能存在诸多问题，欢迎大家指出，最后祝大家国庆快乐~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-10-2 17:21 被bwner编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
