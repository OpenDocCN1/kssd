
通過搜索`Arc-mobile`中的一些字符串，可以確認遊戲使用了 [TrustKit](https://bbs.kanxue.com/elink@8d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4c8S2N6r3S2W2L8%4u0W2L8g2)9J5c8W2c8J5N6i4y4@1d9$3W2@1) 這一開源項目實現 Cert Pinning。簡單地閱讀一下`TrustKit`的代碼即可找到 [`[TSKPinningValidator evaluateTrust:forHostname:]`](https://bbs.kanxue.com/elink@f10K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4c8S2N6r3S2W2L8%4u0W2L8g2)9J5c8W2c8J5N6i4y4@1d9$3W2@1i4K6u0r3j5X3I4G2j5W2)9J5c8U0l9$3x3e0N6V1z5e0W2T1y4e0x3@1k6r3f1$3j5h3c8T1y4U0x3J5j5h3c8S2x3U0b7%4y4h3y4S2z5r3x3J5x3r3x3$3y4o6b7%4x3K6k6Q4x3V1k6f1M7Y4g2K6N6p5E0A6N6q4)9J5c8W2c8e0d9#2m8A6L8X3&6A6L8X3N6h3j5h3I4A6k6r3q4@1L8%4u0Q4x3X3g2E0i4K6t1K6e0o6R3@1) 這個函數，對每個請求的目標進行證書驗證，如果成功則返回 True。知道了這點，要繞過這個 Cert Pinning 簡直輕而易舉。

再次打開 [Compiler Explorer](https://bbs.kanxue.com/elink@471K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8$3c8T1L8$3I4@1i4K6u0W2L8%4u0Y4i4K6u0r3P5W2)9J5c8Y4R3$3z5e0q4V1M7g2W2w2P5l9`.`.)， 寫一個返回`true`值的函數：

![](https://bbs.kanxue.com/upload/attach/202208/898032_JKE7B54N6GUMF6K.jpg)

在 IDA 的 Function 界面 Ctrl-F 搜索`evaluateTrust`即可找到目標函數，然後照舊覆寫機器碼：

![](https://bbs.kanxue.com/upload/attach/202208/898032_T6QZ2USSWV6HCC2.jpg)

如果你現在保存修改，安裝 IPA 嘗試抓包，會發現依然沒有遊戲的 API 數據出現在 Burp Suite 軟件中，這是怎麼回事呢？實際上 Cert Pinning 我們已經繞過了的，但是本遊戲對 API 的保護還有一層，那就是客戶端的 HTTPS 證書校驗。

通常的 HTTPS 請求，只是客戶端校驗服務端的證書，然而這種特殊的配置下，服務端也會同時校驗客戶端的證書，唯有雙向都驗證成功，鏈接才會創立。而 Burp Suite 位於 MITM，阻斷了客戶端向服務端發送證書進行校驗。如果 MITM 任由客戶端和服務端相互校驗各自的證書建立鏈接，則 MITM 將無法解密其消息內容。因此，我們必須將遊戲的客戶端證書以及對應的私鑰都提取出來。

通過一些搜索我們可以[知道](https://bbs.kanxue.com/elink@b9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0M7#2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5c8U0f1J5x3K6f1I4)，要在 iOS 上做這種客戶端的證書驗證，一定會使用到`NSURLAuthenticationMethodClientCertificate`這個引用，我們就以此爲關鍵字在 IDA 中檢索，很容易就能找到這條函數：

![](https://bbs.kanxue.com/upload/attach/202208/898032_FH6ZAY6ZZG33NAF.jpg)

可以看到這個函數在處理客戶端證書請求的時候，先調用`getClientIdentity`再把返回值提供給`[NSURLCredential credentialWithIdentity:certificates:persistence:]`創建`NSURLCredential`對象。 通過 Apple 的一些文檔可以理解出來，`Identity`包括了證書和私鑰，因此我們跟進`getClientIdentity`函數：

![](https://bbs.kanxue.com/upload/attach/202208/898032_U56GWPAPKAR6BMC.jpg)

很明顯了，這個函數讀取了一個加密的 PKCS12 證書+密鑰的結合數據，其中`self->sslCert`就是 PKCS12 的二進制數據，而`self->sslCertPassword`就是解密這個數據的密碼。 現在我們要做的就是把這兩項內容給提取出來。

因爲不想費太多時間去分析代碼找到原始數據，我決定直接用 LLDB 動態從內存中提取這些數據。

只要在 Sideload 過程中使用的是自己的 Apple 開發者帳號的證書，你就可以用 Xcode 調試你 Sideload 的 App。如果設備有越獄，還可以運行`debugserver`然後直接連接 IDA 遠程調試。

打開 Xcode，使用 Debug - Attach to Process by PID or Name... 然後使用`Arc-mobile`作爲調試對象，然後再啓動 Sideload 的遊戲程序，Xcode 就能 attach 到遊戲進程上了。

接下來我們要計算下斷點的地址，使用`target modules list` LLDB 指令可以查看到主程序`Arc-mobile`的起始地址，在此基礎上加上函數地址的偏移值就是我們要下斷點的地址了。 使用 LLDB 的`breakpoint`指令佈置斷點，然後執行任何可以觸發 API 請求的操作，比如隨便登錄一下，即可觸發斷點。

調試器跟進到讀取了`self->sslCert`的位置，然後在 LLDB 執行`po $x8`即可打印出 PKCS12 的二進制數據：

![](https://bbs.kanxue.com/upload/attach/202208/898032_ECRSJN4YYPUMHYE.jpg)

同理在讀取了`self->sslCertPassword`的位置我們可以得出 PKCS12 的密碼爲`HelloWorld`：

![](https://bbs.kanxue.com/upload/attach/202208/898032_PFTQQM5Z8NWTN9G.jpg)

爲了能夠模擬成真實的客戶端，同時對 API 進行抓包，我們需要寫一個 API 代理轉發程序。這裏我就用 Golang 寫了：

```go
package main

import (
	"crypto"
	"crypto/tls"
	"crypto/x509"
	_ "embed"
	"io"
	"log"
	"net/http"

	"golang.org/x/crypto/pkcs12"
)

//go:embed v4.0.255_key.p12
var v4_0_255_key []byte

var password = "HelloWorld"

func init() {
	var (
		err       error
		clientKey crypto.PrivateKey
		clientCert *x509.Certificate
	)
	if clientKey, clientCert, err = pkcs12.Decode(v4_0_255_key, password); err != nil {
		panic(err)
	}
	http.DefaultClient.Transport = &http.Transport{TLSClientConfig: &tls.Config{
		Certificates: []tls.Certificate{{
			Certificate: [][]byte{clientCert.Raw},
			PrivateKey:  clientKey,
		}},
	}}
}

func main() {
	addr := "0.0.0.0:5151"
	log.Println("Server Started at", addr)
	http.ListenAndServe(addr, http.HandlerFunc(handler))
}

func handler(w http.ResponseWriter, r *http.Request) {
	var (
		err  error
		resp *http.Response
	)

	check := func(err error) bool {
		if err != nil {
			w.WriteHeader(500)
			log.Printf("%s %s", "FAIL", err.Error())
			return true
		}
		return false
	}

	log.Printf("%s %s", r.Method, r.URL.String())

	if resp, err = forwardHTTPRequest(r); check(err) {
		return
	}

	defer func() {
		resp.Body.Close()
	}()

	for key, vals := range resp.Header {
		for _, val := range vals {
			w.Header().Add(key, val)
		}
	}

	w.WriteHeader(resp.StatusCode)
	io.Copy(w, resp.Body)
}

func forwardHTTPRequest(r *http.Request) (resp *http.Response, err error) {
	r.URL.Scheme = "https"
	r.URL.Host = "arcapi-v2.lowiro.com"
	r.Header.Del("Host")

	req, err := http.NewRequest(r.Method, r.URL.String(), r.Body)
	if err != nil {
		return
	}

	req.Header = r.Header
	if resp, err = http.DefaultClient.Do(req); err != nil {
		return
	}
	return
}
```

把前面提取出來的 PKCS12 文件以二進制形式保存到`v4.0.255_key.p12`然後執行上面的 Golang 程序就能在本地`127.0.0.1:5151`端口開啓 API 代理轉發服務器了。

接下來我們要配置 MITM 代理把所有的 API 請求轉發到 API 代理轉發服務器上，這裏的話需要根據你的 MITM 軟件來設置， 如果用的是 Charles 可以直接用 Map Remote 功能，如果用的是 Burp Suite，則需要使用 Extension 腳本，這裏給出 Python 版的：

```python
from burp import IBurpExtender
from burp import IHttpListener

HOST_FROM = "arcapi-v2.lowiro.com"
HOST_TO = "127.0.0.1"
PORT_TO = 5151
PROTO_TO = "http"

class BurpExtender(IBurpExtender, IHttpListener):

    #
    # implement IBurpExtender
    #

    def registerExtenderCallbacks(self, callbacks):
        # obtain an extension helpers object
        self._helpers = callbacks.getHelpers()

        # set our extension name
        callbacks.setExtensionName("Traffic redirector")

        # register ourselves as an HTTP listener
        callbacks.registerHttpListener(self)

    #
    # implement IHttpListener
    #

    def processHttpMessage(self, toolFlag, messageIsRequest, messageInfo):
        # get the HTTP service for the request
        httpService = messageInfo.getHttpService()
        host = httpService.getHost()

        # if the host is HOST_FROM, change it to HOST_TO
        if messageIsRequest and host == HOST_FROM:
            messageInfo.setHttpService(self._helpers.buildHttpService(HOST_TO,
                                                                      PORT_TO, PROTO_TO))
```

現在我們就能成功抓包了，你可以根據抓到的包去寫私服，不過網上有已經寫好的服務端可以直接用，方法也很簡單，就是把上面轉發 API 請求的目的地變成你搭建的私服的地址即可。如果你想把私服架在公網，然後隨時隨地無需設置 MITM 代理也能使用私服遊玩，那你還需要把遊戲的 API Endpoint 地址給改成自己的私服 Endpoint 地址。

### 修改 API Endpoint 地址

這一步相對繁瑣，但是並不難。通過抓包我們知道 API Endpoint Prefix 是``2e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3y4S2M7r3W2Q4x3X3c8$3x3W2)9J5k6h3I4G2N6$3W2J5L8#2)9J5k6h3y4G2L8g2)9J5c8X3A6G2K9h3&6Q4x3V1j5J5x3g2)9J5c8R3`.`.``，在 IDA 中搜索一些 API method 比如`auth/login`即可找到構建請求 URL 的地方。

![](https://bbs.kanxue.com/upload/attach/202208/898032_QRJBDRKFNY7M679.jpg)

不難看出，API Endpoint Prefix 是經過加密處理的，稍加閱讀代碼可以看出加密的模式是 CFB，也就是說上一個 Block 的密文會被用來計算下一個 Block 的 XOR Key：

![](https://bbs.kanxue.com/upload/attach/202208/898032_9E5P8CHRNRCU3SE.jpg)
