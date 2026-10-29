
|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 | `webView.setWebViewClient(new WebViewClient() {` `@Override` `public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {` `handler.proceed();` `//表示等待证书响应` `handler.cancel();` `//表示挂起连接，为默认方式` `handler.handleMessage(null);` `//可做其他处理` `}` `});` |

### `WebChromeClient`

作用：辅助 `WebView` 处理 Javascript 的对话框、网站图标、网站标题等等。

#### `onProgressChanged()`

作用：获得网页的加载进度并显示。

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `webview.setWebChromeClient(new WebChromeClient(){` `@Override` `public void onProgressChanged(WebView view, int newProgress) {` `if (newProgress < 100) {` `String progress = newProgress + "%";` `progress.setText(progress);` `} else {` `}` `});` |

#### `onReceivedTitle()`

作用：获取 Web 页中的标题。

每个网页的页面都有一个标题，比如：

`526K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1&6i4@1p5I4i4@1t1#2i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4@1p5H3i4K6R3%4i4@1f1&6i4@1p5J5i4K6V1^5i4@1f1#2i4K6S2p5i4@1t1K6i4@1f1J5i4K6R3H3i4K6W2o6i4@1f1%4i4K6V1&6i4@1u0q4i4@1f1#2i4@1u0m8i4@1p5$3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1&6i4@1p5I4i4@1t1#2i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4@1p5H3i4K6R3%4i4@1f1&6i4@1p5J5i4K6V1^5i4@1f1#2i4K6S2p5i4@1t1K6i4@1f1J5i4K6R3H3i4K6W2o6i4K6S2p5N6$3g2T1N6X3W2W2N6#2!0q4y4W2!0m8c8q4!0m8x3#2!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4g2)9^5b7g2!0m8x3q4!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5g2!0m8x3g2!0n7y4g2!0q4z5g2)9&6c8q4!0m8x3W2!0q4y4#2)9&6b7g2)9^5y4s2c8A6N6r3I4W2i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1^5i4@1q4q4i4@1u0q4i4@1f1%4i4@1u0p5i4@1q4q4i4@1f1#2i4K6V1I4i4@1p5J5i4@1g2r3i4@1u0o6i4K6W2r3i4@1f1^5i4@1q4q4i4@1u0q4i4@1f1%4i4@1u0p5i4@1q4q4i4@1f1#2i4K6V1I4i4@1p5J5i4@1g2r3i4@1u0o6i4K6W2r3`

|  |  |
| --- | --- |
| 1 2 3 4 5 6 | `webview.setWebChromeClient(new WebChromeClient(){` `@Override` `public void onReceivedTitle(WebView view, String title) {` `titleview.setText(title);` `}` |

## （3）WebView 与 JS 的交互

Android `WebView` 与 JS 的交互：

![image-20220726171737615](https://bbs.kanxue.com/upload/attach/202207/905443_F97H8PAK73A38RC.png)

### <1> Android 调用 JS

#### `WebView.loadUrl()`

首先，准备 HTML 文件，放到 `assets` 中。

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 | `<html>` `<head>` `<meta charset="utf-8">` `<title>测试</title>` `<script>` `function callJS(){` `alert("Android调用了JS的callJS方法");` `}` `</script>` `</head>` `<body>` `<h1>Android调用JS方法测试</h1>` `</body>` `</html>` |

![image-20220729093637679](https://bbs.kanxue.com/upload/attach/202207/905443_M9W5QAK4MJHW3P3.png)

然后在 Java 层中添加代码，进行调用 JS 文件以及 JS 中的方法：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 | `//设置与Js交互的权限` `webSettings.setJavaScriptEnabled(true);` `//设置允许JS弹窗` `webSettings.setJavaScriptCanOpenWindowsAutomatically(true);` `//先载入JS代码` `//格式规定为:file:///android_asset/文件名.html` `mWebView.loadUrl("file:///android_asset/AndroJs.html");` `Button button = (Button) findViewById(R.id.button);` `button.setOnClickListener(new View.OnClickListener() {` `@Override` `public void onClick(View v) {` `//通过Handler发送消息` `mWebView.post(new Runnable() {` `@Override` `public void run() {` `//注意调用的JS方法名要对应上` `//调用javascript的callJS()方法` `mWebView.loadUrl("javascript:callJS()");` `}` `});` `}` `});` `//由于设置了弹窗检验调用结果,所以需要支持js对话框` `//webview只是载体，内容的渲染需要使用webviewChromClient类去实现` `//通过设置WebChromeClient对象处理JavaScript的对话框` `//设置响应js的Alert()函数` `mWebView.setWebChromeClient(new WebChromeClient() {` `@Override` `public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {` `AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);` `b.setTitle("Alert");` `b.setMessage(message);` `b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {` `@Override` `public void onClick(DialogInterface dialog, int which) {` `result.confirm();` `}` `});` `b.setCancelable(false);` `b.create().show();` `return true;` `}` `});` |

效果演示：

![image-20220729093932401](https://bbs.kanxue.com/upload/attach/202207/905443_MX3US9H5XNXXR3B.png)

我们可以发现程序成功地加载了 HTML 文件，然后我们点击按钮去调用 JS 中的方法。

![image-20220729094015144](https://bbs.kanxue.com/upload/attach/202207/905443_SPKF6UMG3ZU997W.png)

这里就成功地通过 `loadUrl` 进行了调用。

**特别注意：JS 代码调用一定要在 `onPageFinished()` 回调之后才能调用，否则不会调用。**

|  |  |
| --- | --- |
| 1 | `onPageFinished()` 属于 `WebViewClient` 类的方法，主要在页面加载结束时调用。 |

#### `WebView.evaluateJavascript()`

优点：该方法比第一种方法效率更高、使用更简洁。

|  |  |
| --- | --- |
| 1 2 | 因为该方法的执行不会使页面刷新，而第一种方法（`loadUrl`）的执行则会。`Android` `4.4` 后才可使用。 |

具体使用：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 | `//使用evaluateJavascript来加载` `mWebView.evaluateJavascript("javascript:callJS()", new ValueCallback<String>() {` `@Override` `public void onReceiveValue(String value) {` `//此处为js返回的结果` `}` `});` |

![image-20220729094734817](https://bbs.kanxue.com/upload/attach/202207/905443_7G4DRNKYF77CMY7.png)

同样加载成功。

![image-20220729094802205](https://bbs.kanxue.com/upload/attach/202207/905443_JWE7TE67SA736UJ.png)

### <2> JS 调用 Android

#### `addJavascriptInterface`

首先，准备 HTML 文件，放到 `assets` 中。

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 | `//JS调用Android` `<html>` `<head>` `<meta charset="utf-8">` `<title>Carson</title>` `<script>` `function callAndroid(){` `//由于对象映射，所以调用test对象等于调用Android映射的对象` `test.hello("WindXaa js调用了android中的hello方法");` `}` `</script>` `</head>` `<body>` `<!--点击按钮则调用callAndroid函数-->` `<button type="button" id="button1" onclick="callAndroid()">Click Attack</button>` `</body>` `</html>` |

![image-20220729100734122](https://bbs.kanxue.com/upload/attach/202207/905443_46EAAMZDCGAUEG9.png)

然后加载调用：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 | `public class JsToAndroActivity extends AppCompatActivity {` `@Override` `protected void onCreate(Bundle savedInstanceState) {` `super.onCreate(savedInstanceState);` `setContentView(R.layout.activity_js_to_andro);` `WebView mWebView = (WebView) findViewById(R.id.Wind_webview1);` `WebSettings webSettings = mWebView.getSettings();` `//设置与Js交互的权限` `webSettings.setJavaScriptEnabled(true);` `//通过addJavascriptInterface()将Java对象映射到JS对象` `//参数1：Javascript对象名` `//参数2：Java对象名` `mWebView.addJavascriptInterface(new AndroidtoJs(), "test"); //AndroidtoJS类对象映射到js的test对象` `//加载JS代码` `//格式规定为:file:///android_asset/文件名.html` `mWebView.loadUrl("file:///android_asset/javascript.html");` `}` `/**` `* 提供接口在Webview中供JS调用` `*/` `public class AndroidtoJs {` `//定义JS需要调用的方法，被JS调用的方法必须加入@JavascriptInterface注解` `@JavascriptInterface` `public void hello(String msg) {` `Log.e("WindXaa", "Hello，" + msg);` `}` `}` `}` |

然后我们切换到第二个测试用例。

![image-20220729100854323](https://bbs.kanxue.com/upload/attach/202207/905443_A23JBWN5QZS7H36.png)

直接点击。

![image-20220729100914639](https://bbs.kanxue.com/upload/attach/202207/905443_3A8M5MSWH33YCCT.png)

此时已经成功地加载了我们的 JS，我们点击按钮。

![image-20220729101013208](https://bbs.kanxue.com/upload/attach/202207/905443_9H9YZECS4UZYAHF.png)

JS 中就成功地加载了 Android 中的 `hello` 方法。

#### `WebViewClient.shouldOverrideUrlLoading()`

具体原理：

|  |  |
