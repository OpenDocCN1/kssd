
# [原创]Android APP漏洞之战（13）——`WebView`漏洞详解

[TOC]

## 一、前言

快半个月没有更新文章了，最近不少朋友催更了，今天我们进入Android APP漏洞之战系列文章中的一个重要篇幅——`WebView`漏洞，我们都知道在当下App漏洞中，`WebView`漏洞的占比是十分巨大的，各种类型的漏洞问题层出不穷，这篇文章就带着大家一起揭开`WebView`漏洞神奇的面纱。

本文第二节讲述`WebView`的基本知识

本文第三节讲述`WebView`的漏洞面

本文第四节进行了漏洞原理介绍和漏洞复现

总结：本文从`WebView`开发出发，从0开始进行漏洞的讲解和复现，花了几天时间，列举了`WebView`中的20多种漏洞案例，并手动复现了其中十余种案例，希望用这篇万字长文介绍`WebView`漏洞的基本发展。

## 二、基础知识

### 1.`WebView`基础

#### （1）`WebView`概述

Android `WebView`在Android平台上是一个特殊的`View`，它能用来显示网页，这个`WebView`类可以被用来在app中仅仅显示一张在线的网页，还可以用来开发浏览器。

`WebView`内部实现是采用渲染引擎(WebKit)来展示view的内容，提供网页前进后退、网页放大、缩小、搜索等功能。Android `WebView` 在低版本和高版本采用了不同的 webkit 版本内核，在 4.4 版本后使用 Chrome 内核。

#### （2）`WebView`作用

- 显示和渲染Web页面
- 直接使用html文件（网络上或本地`assets`中）作布局
- 可和JavaScript交互调用

```
WebView控件功能强大，除了具有一般View的属性和设置外，还可以对url请求、页面加载、渲染、页面交互进行强大的处理。
```

#### （3）`WebView`基础使用

##### <1>本地加载

Web最简单显示网页内容，基本步骤：

```
1.在布局文件中添加WebView控件；
2.在代码中让WebView控件加载显示网页。
```

具体操作：

首先，我们在布局文件中来添加`WebView`控件，如下

```xml
<WebView
android:id="@+id/Wind\_webview"
android:layout\_width="match\_parent"
android:layout\_height="match\_parent" />
```

![image-20220726171401583](upload/attach/202207/905443\_CFRS9YBGFHY7567.png)

然后我们在代码中让`WebView`控件加载显示网页，如下：

```java
//获得控件
WebView webView = (WebView) findViewById(R.id.Wind\_webview);
//访问网页
webView.loadUrl("http://www.baidu.com");
//系统默认会通过手机浏览器打开网页，为了能够直接通过WebView显示网页，则必须设置
webView.setWebViewClient(new WebViewClient(){
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
//使用WebView加载显示url
view.loadUrl(url);
//返回true
return true;
}
});
```

![image-20220726171554052](upload/attach/202207/905443\_2AUPQ3H9EW78B4C.png)

最后我们在配置文件中添加网络权限：

```xml
<!-- 添加网络权限 -->
<uses-permission android:name="android.permission.INTERNET" />
```

![image-20220726171648967](upload/attach/202207/905443\_VGNC5R48DVVQMSZ.png)

运行程序，显示如下：

![image-20220726171737615](upload/attach/202207/905443\_ZCCQTP8CJ9UE75F.png)

##### <2>远程加载

首先，我们直接在本地新建js文件

```js
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Carson</title>
<script>
function callAndroid(){
//由于对象映射，所以调用test对象等于调用Android映射的对象
test.hello("WindXaa!");
}
</script>
</head>
<body>
<!--点击按钮则调用callAndroid函数-->
<button type="button" id="button1" onclick="callAndroid()">Internet Click connect</button>
</body>
</html>
```

![image-20220729155049524](upload/attach/202207/905443\_56U7Z2SY8Y3MH95.png)

然后我们开启一个简易的`http_server`的监听

![image-20220729155135353](upload/attach/202207/905443\_7JH8848EQ6CZ459.png)

我们编写代码：

```java
{
WebView mWebView = (WebView) findViewById(R.id.Wind\_webview1);
WebSettings webSettings = mWebView.getSettings();
// 设置与Js交互的权限
webSettings.setJavaScriptEnabled(true);
// 通过addJavascriptInterface()将Java对象映射到JS对象
//参数1：Javascript对象名
//参数2：Java对象名
mWebView.addJavascriptInterface(new AndroidtoJs(), "test");//AndroidtoJS类对象映射到js的test对象
mWebView.loadData("","text/html",null);
// 加载JS代码
// 格式规定为:file:///android\_asset/文件名.html
// mWebView.loadUrl("file:///android\_asset/javascript.html");
mWebView.loadUrl("http://ip地址填自己的/attack.html");
}
/**
 * 提供接口在Webview中供JS调用
 */
public class AndroidtoJs {
// 定义JS需要调用的方法，被JS调用的方法必须加入@JavascriptInterface注解
@JavascriptInterface
public void hello(String msg) {
Log.e("WindXaa","Hello，" + msg);
}
}
```

然后我们再次运行程序

![image-20220729155443281](upload/attach/202207/905443\_2K45JUHR5QH8FHW.png)

这里就说明我们远程加载文件成功了，我们点击按钮

![image-20220729160726979](upload/attach/202207/905443\_DATF72SERE3X5ZM.png)

可以发现可以成功的通过JS调用Android代码

### 2.`WebView`使用详解

#### （1）`WebView`常用方法

`WebView`的状态：

`webView.onResume();`

```
// 激活WebView为活跃状态，能正常执行网页的响应
```

`webView.onPause();`

```
// 当页面被失去焦点被切换到后台不可见状态，需要执行onPause
// 通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。
```

`webView.pauseTimers()`

```
// 当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview
// 它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。
```

`webView.resumeTimers()`

```
// 恢复pauseTimers状态
```

`rootLayout.removeView(webView)`

`webView.destory()`

```
// webview调用destory时，webview仍绑定在Activity上
// 需要先从父容器中移除webview，然后再销毁webview
```

**前进、后退网页**

```java
//是否可以后退
Webview.canGoBack()
//后退网页
Webview.goBack()
//是否可以前进
Webview.canGoForward()
//前进网页
Webview.goForward()
//以当前的index为起始点前进或者后退到历史记录中指定的steps
//如果steps为负数则为后退，正数则为前进
Webview.goBackOrForward(intsteps)
```

在不做任何处理前提下，浏览网页时点击系统的“Back”键时，整个 Browser 会调用`finish()`而结束自身，因此需要在当前`Activity`中处理并消费掉该 Back 事件，当按下返回键时，调用`goBack`方法。

我们可以做一些处理，让点击“Back”键后，让网页返回上一页而不是直接退出浏览器，此时我们可以在当前的`Activity`中处理Back事件，如下：

```java
public boolean onKeyDown(int keyCode, KeyEvent event) {
if ((keyCode == KEYCODE\_BACK) && mWebView.canGoBack()) {
mWebView.goBack();
return true;
}
return super.onKeyDown(keyCode, event);
}
```

**清除缓存数据**

```
//清除网页访问留下的缓存
//由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
Webview.clearCache(true);
//清除当前webview访问的历史记录
//只会webview访问历史记录里的所有记录除了当前访问记录
Webview.clearHistory()；
//这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据
Webview.clearFormData()；
```

#### （2）常用类

##### <1>`WebSettings`类

作用：对`WebView`进行配置和管理

配置步骤：

第一步：添加访问网络权限（`AndroidManifest.xml`）

```
<uses-permission android:name="android.permission.INTERNET"/>
```

注意：从Android 9.0（API级别28）开始，默认情况下禁用明文支持，会显示 `ERR\_CLEARTEXT\_NOT\_PERMITTED`。因此http的url均无法在`webview`中加载，可以在manifest中application节点添加`android:usesCleartextTraffic="true"`。

第二步：生成一个`WebView`组件（有两种方式）

```java
//方式1：直接在在Activity中生成
WebView webView = new WebView(this)
//方法2：在Activity的layout文件里添加webview控件：
WebView webview = (WebView) findViewById(R.id.webView1);
```

第三步：进行配置-利用`WebSettings`子类（常见方法）

```java
//声明WebSettings子类
WebSettings webSettings = webView.getSettings();
//如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
webSettings.setJavaScriptEnabled(true);
//支持插件
webSettings.setPluginsEnabled(true);
//设置自适应屏幕，两者合用
webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小
webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小
//缩放操作
webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件
//其他细节操作
webSettings.setCacheMode(WebSettings.LOAD\_CACHE\_ELSE\_NETWORK); //关闭webview中缓存
webSettings.setAllowFileAccess(true); //设置可以访问文件
webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口
webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
```

常见方法：设置`WebView`缓存

- 当加载 html 页面时，`WebView`会在`/data/data/包名目录`下生成 `database` 与 `cache` 两个文件夹
- 请求的 URL记录保存在 `WebViewCache.db`，而 URL的内容是保存在 `WebViewCache` 文件夹下
- 是否启用缓存：

```java
//优先使用缓存
WebView.getSettings().setCacheMode(WebSettings.LOAD\_CACHE\_ELSE\_NETWORK);
//缓存模式如下：
//LOAD\_CACHE\_ONLY: 不使用网络，只读取本地缓存数据
//LOAD\_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
//LOAD\_NO\_CACHE: 不使用缓存，只从网络获取数据.
//LOAD\_CACHE\_ELSE\_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据
//不使用缓存
WebView.getSettings().setCacheMode(WebSettings.LOAD\_NO\_CACHE);
```

##### <2>`WebViewClient`类

用来处理各种通知 & 请求事件

`shouldOverrideUrlLoading()`

作用：打开网页时不调用系统浏览器， 而是在本`WebView`中显示；在网页上的所有加载都经过这个方法,这个函数我们可以做很多操作。

```java
//Webview控件
Webview webview = (WebView) findViewById(R.id.webView);
//加载一个网页
webView.loadUrl("http://www.google.com/");
//重写shouldOverrideUrlLoading()方法，使得打开网页时不调用系统浏览器， 而是在本WebView中显示
webView.setWebViewClient(new WebViewClient(){
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
view.loadUrl(url);
return true;
}
});
```

`onPageStarted()`

作用：开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。

```java
webView.setWebViewClient(new WebViewClient(){
@Override
public void onPageStarted(WebView view, String url, Bitmap favicon) {
//设定加载开始的操作
}
});
```

`onLoadResource()`

作用：在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。

```java
webView.setWebViewClient(new WebViewClient(){
@Override
public boolean onLoadResource(WebView view, String url) {
//设定加载资源的操作
}
});
```

`onReceivedError()`

作用：加载页面的服务器出现错误时（如404）调用。
