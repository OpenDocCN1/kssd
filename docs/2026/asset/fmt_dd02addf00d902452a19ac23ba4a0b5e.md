

```java
var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');
var SSLContext = Java.use('javax.net.ssl.SSLContext');

// TrustManager（Android < 7）
////////////////////////////////
var TrustManager = Java.registerClass({
  // Implement a custom TrustManager
  name: 'dev.asd.test.TrustManager',
  implements: [X509TrustManager],
  methods: {
    checkClientTrusted: function (chain, authType) {},
    checkServerTrusted: function (chain, authType) {},
    getAcceptedIssuers: function () {
      return [];
    }
  }
});

// Prepare the TrustManager array to pass to SSLContext.init()
var TrustManagers = [TrustManager.$new()];

// Get a handle on the init() on the SSLContext class
var SSLContext_init = SSLContext.init.overload(
  '[Ljavax.net.ssl.KeyManager;',
  '[Ljavax.net.ssl.TrustManager;',
  'java.security.SecureRandom'
);

try {
  // Override the init method, specifying the custom TrustManager
  SSLContext_init.implementation = function (keyManager, trustManager, secureRandom) {
    console.log('[+] Bypassing Trustmanager (Android < 7) pinner');
    SSLContext_init.call(this, keyManager, TrustManagers, secureRandom);
  };
} catch (err) {
  console.log('[-] TrustManager (Android < 7) pinner not found');
  // console.log(err);
}
```

## 3 主机（域名）校验

### 背景

一个证书可能对应有很多域名都可以使用，但是开发者只想让当前的应用证书校验通过后只能往指定的域名发送请求，而不想证书校验通过后往其他可以校验证书通过的域名发送请求。

也就是说，证书允许往很多域名发送请求，但是 `app` 限制只能往特定域名发送请求。

```java
private void doRequest3() {
  HostnameVerifier verifier = new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
      if ("2d2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5M7r3!0B7K9h3g2Q4x3X3g2U0L8R3.".equalsIgnoreCase(hostname)) {
        return true;
      }
      return false;
    }
  };

  new Thread() {
    @Override
    public void run() {
      try {
        OkHttpClient client =
            new OkHttpClient.Builder().hostnameVerifier(verifier).build();

        Request req =
            new Request.Builder()
                .url("6f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6X3L8%4u0#2L8g2)9J5k6i4m8Z5M7l9.")
                .build();

        Call call = client.newCall(req);
        Response res = call.execute();

        Log.e("请求发送成功", "状态码：" + res.code());
      } catch (IOException ex) {
        Log.e("Main", "网络请求异常" + ex);
      }
    }
  }.start();
}
```

绕过方式和证书校验思路一样，自己创建一个实现 `HostnameVerifier` 接口的类，直接返回 `true`，哪里调用传给哪里即可。

## 3.1.2 服务器端证书校验

> 在客户端放入证书（`p12`/`bks`），客户端向服务端发送请求时，携带证书信息，在服务端会校验客户端携带过来的证书合法性。

### 开发逻辑

- 在 APK 打包时，将证书放入 `assets` 或 `raw` 目录。
- 在开发代码时，发送请求读取证书文件内容和证书密码，携带并发送到服务器端。

```java
private void doRequest4() {
  X509TrustManager trustManager = new X509TrustManager() {
    @Override
    public void checkClientTrusted(
        X509Certificate[] chain, String authType
    ) throws CertificateException {
    }

    @Override
    public void checkServerTrusted(
        X509Certificate[] chain, String authType
    ) throws CertificateException {
    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
      return new X509Certificate[0];
    }
  };

  HostnameVerifier verify = new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
      return true;
    }
  };

  new Thread() {
    @Override
    public void run() {
      try {
        InputStream client_input =
            getResources().openRawResource(R.raw.client);

        Log.e("x", client_input.getClass().toString());

        SSLContext sslContext = SSLContext.getInstance("TLS");
        KeyStore keyStore = KeyStore.getInstance("PKCS12");

        keyStore.load(
            client_input,
            "demoli666".toCharArray()
        );

        KeyManagerFactory keyManagerFactory =
            KeyManagerFactory.getInstance(
                KeyManagerFactory.getDefaultAlgorithm()
            );

        keyManagerFactory.init(
            keyStore,
            "demoli666".toCharArray()
        );

        sslContext.init(
            keyManagerFactory.getKeyManagers(),
            new TrustManager[] {trustManager},
            new SecureRandom()
        );

        SSLSocketFactory factory = sslContext.getSocketFactory();

        OkHttpClient client =
            new OkHttpClient.Builder()
                .sslSocketFactory(factory, trustManager)
                .hostnameVerifier(verify)
                .build();

        Request req =
            new Request.Builder()
                .url("4bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5s2S2Q4x3X3g2^5P5s2S2Q4x3X3g2^5P5s2S2Q4x3X3g2^5P5s2S2Q4x3@1p5@1y4o6y4Q4x3V1k6A6L8X3c8W2P5l9.")
                .build();

        Call call = client.newCall(req);
        Response res = call.execute();

        Log.e("请求发送成功", "状态码：" + res.code());
      } catch (Exception e) {
        Log.e("请求发送失败", "网络异常" + e);
      }
    }
  }.start();
}
```

### 标志

![Untitled](https://bbs.kanxue.com/upload/attach/202308/976141_ERM52E533YKW82Z.jpg)

### 解决方法

- 找到证书文件（`bsk`/`p12`）。
- 通过 `hook` 获取证书相关密码。

