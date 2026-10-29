
# [原创]数字猫小说阅读软件分析

# 背景

市面上很多小说阅读软件，这些小说软件美其名曰免费，实际上翻两页就是一大段广告，十分影响阅读体验，本篇文章通过数字猫小说软件入手，对其进行去广告以及相关算法分析。

# 去广告

软件没加壳，直接用 JEB 载入，通过搜索“AD”之类定位广告：

相关函数符号还没混淆：

`isVideoRewardExpire` 判断看广告视频的阅读奖励是否过期，这里直接返回 `false`。

![](https://bbs.kanxue.com/upload/attach/202102/651413_SZHYG8CJ3DZPUY7.png)

还有一处也是类似的逻辑：

![](https://bbs.kanxue.com/upload/attach/202102/651413_VMRXX552UU6AT7Y.png)

用 apklab 直接找到相关函数修改之：

![](https://bbs.kanxue.com/upload/attach/202102/651413_828WD3XV2B2VHZV.png)

修改结果如下：

![](https://bbs.kanxue.com/upload/attach/202102/651413_R2BZMAKD2EA5NDH.png)

为了方便抓包，移除 SSL 证书绑定：

![](https://bbs.kanxue.com/upload/attach/202102/651413_D56ZVNS3QQDS98G.png)

经过测试，APK 还有签名校验，在 so 里面，直接把 2 改为 1 即可过掉该校验：

![](https://bbs.kanxue.com/upload/attach/202102/651413_38HRPBPFHQ37HEZ.png)

重打包，可以看到，广告已经去掉了：

![](https://bbs.kanxue.com/upload/attach/202102/651413_J8W9DUPEQAZHQEW.jpg)

# 小说下载器

光是去广告就完了吗？对于 20 年的书虫来说还不够，有时我想在 kindle 上阅读该小说，所以我决定做一个简陋的下载器。

## 抓包

有两个包十分关键，第一个包获取章节信息，包括章节名称和 `id` 等。可以看到请求都有签名 `sign` 字段。

![](https://bbs.kanxue.com/upload/attach/202102/651413_2UA5QQDHXHQBXER.png)

第二个包是获取小说下载地址：

![](https://bbs.kanxue.com/upload/attach/202102/651413_2YUNZDQC4A3FZNS.png)

下载过来是一个压缩包，打开发现里面都是以章节 `id` 命名的 `.txt` 文件，很明显还加密过了：

![](https://bbs.kanxue.com/upload/attach/202102/651413_FVSX7594U8QGSTZ.png)

思路很清晰了，先搞定 `sign` 算法，再搞定小说书籍解密算法。

## `sign` 算法

先通过 JEB 定位 `sign` 算法的地方，发现这个函数最终还是调用了 so 里面的函数：

![](https://bbs.kanxue.com/upload/attach/202102/651413_2PFUNWF9TDAMPP4.png)

先用 `frida` hook 该函数，打印下日志，看看传参是啥，脚本如下：

```javascript
Java.perform(function () {
var Encryption = Java.use('com.qimao.qmsdk.tools.encryption.Encryption');
Encryption.sign.implementation = function () {
var s = arguments[0];
var r = this.sign(s)
send('sign: '+s+' result: '+r);
return r;
}
});
```

很明显，传入的是 HTTP 的 `get` 或者 `post` 参数，参数 `key` 按升序排列。

![](https://bbs.kanxue.com/upload/attach/202102/651413_SJM7NVNVVPA2S7K.png)

为了分析 `sign` 算法，定位到了 so 里面的处理逻辑，把 `key` append 到字符串结尾，再求 `MD5`：

![](https://bbs.kanxue.com/upload/attach/202102/651413_9DZR6BTZ9JSB99Q.png)

这里我直接在 `memcpy` 打个断点，用 IDA 连上去查看 `key`，当然也可以继续用 `frida` hook。

![](https://bbs.kanxue.com/upload/attach/202102/651413_96FME5DWS4TKZ5U.png)

好的，现在已经成功获取到 `sign` 的 `key`，直接写出相关 `sign` 算法如下：

```python
def getsign(param_dict):
    sign_key = 'd3dGiJ**********'
    before_sign = ''
    for key in sorted(param_dict):
        before_sign += f'{key}={str(param_dict[key])}'
    before_sign += sign_key
    sign = MD5.new(before_sign.encode('utf8')).hexdigest()
    return sign
```

## 小说数据解密

搞定了 `sign` 算法之后，开始分析小说数据的解密算法。

解密算法也在 so 里面，但是回调了 Java 层里面的 `AES` 解密算法：

![](https://bbs.kanxue.com/upload/attach/202102/651413_S3R8YTAQNU8GXRA.png)

Java 算法如下，把数据 `base64` 解密后，取前 16 个字节作为 `IV`，传入的 `key` 直接作为 `AES-CBC` 的解密密钥进行解密：

![](https://bbs.kanxue.com/upload/attach/202102/651413_XWBDZWR5NR93URT.png)

这里继续用 `frida` hook：

```javascript
Java.perform(function () {
var AESManager = Java.use('com.km.encryption.aes.AESManager');
AESManager.decrypt.overload('java.lang.String', 'java.lang.String').implementation = function () {
var s = arguments[0];
var k = arguments[1];
var r = this.decrypt(s,k)
send('decrpt book: '+s+' key: '+k);
return r;
}
});
```

通过日志直接获取 `AES` 解密密钥，经过测试，不同的小说解密密钥都是一样的：

![](https://bbs.kanxue.com/upload/attach/202102/651413_JBVQ3CH5FDE5KR3.png)

解密算法如下：

```python
def decrypt_chapter(s):
    aes_key = b'242ccb**********'
    iv_enc_data = base64.b64decode(s)
    aes_iv = iv_enc_data[0:16]
    enc_data = iv_enc_data[16:]
    unpad = lambda s: s[:-ord(s[len(s) - 1:])]
    aes = AES.new(aes_key, AES.MODE_CBC, aes_iv)
    return unpad(aes.decrypt(enc_data))
```

## 放在一起

下载器源码如下，关键地方已经和谐处理：

```python
import requests
import base64
import string
import zipfile
import time
import json
import io
from Crypto.Hash import MD5
from Crypto.Cipher import AES

def getsign(param_dict):
    sign_key = 'd3dGiJ**********'
    before_sign = ''
    for key in sorted(param_dict):
        before_sign += f'{key}={str(param_dict[key])}'
    before_sign += sign_key
    sign = MD5.new(before_sign.encode('utf8')).hexdigest()
    return sign

def decrypt_chapter(s):
    aes_key = b'242ccb**********'
    iv_enc_data = base64.b64decode(s)
    aes_iv = iv_enc_data[0:16]
    enc_data = iv_enc_data[16:]
    unpad = lambda s: s[:-ord(s[len(s) - 1:])]
    aes = AES.new(aes_key, AES.MODE_CBC, aes_iv)
    return unpad(aes.decrypt(enc_data))

def qm_get(url, params, headers=None):
    if not headers:
        headers = {
            'app-version': '50810',
            'platform': 'android',
            'reg': '0',
            'AUTHORIZATION': '',
            'application-id': 'com.****.reader',
            'net-env': '1',
            'channel': 'unknown',
            'qm-params': '',
        }
    headers.update({"sign": getsign(headers)})
    params.update({"sign": getsign(params)})
    res = requests.get(url, params=params, headers=headers, verify=False)
    return str(res.text)

def get_book_download_url(bookid):
    download_url = 'https://api-bc.****.com/api/v1/book/download'
    params = {
        'source': '1',
        'type': '1',
        'id': bookid,
    }
    book_dl_info = json.loads(qm_get(download_url, params))
    return book_dl_info['data']['link']

def get_chapter_list(bookid):
    chapter_url = 'https://api-ks.****.com/api/v1/chapter/chapter-list'
    params = {
        'is_all_update': '0',
        'id': bookid,
    }
    chapter_list = json.loads(qm_get(chapter_url, params))
    return chapter_list['data']['chapter_lists']

def get_book_data(link):
    res = requests.get(link, verify=False)
    return res.content

if __name__ == "__main__":
    logo = '''

    '''
    print(logo)
    book_id = 198241
    with open(str(book_id) + '.txt', 'wb+') as fd:
        download_link = get_book_download_url(book_id)
        chapter_list = get_chapter_list(book_id)
        chapter_list.sort(key=lambda x: x['chapter_sort'])
        # print(chapter_list)
        book_data = get_book_data(download_link)
        book_file = io.BytesIO(book_data)
        with zipfile.ZipFile(book_file) as zip_ref:
            for chapter_info in chapter_list:
                chapter_id = chapter_info['id']
                with zip_ref.open(chapter_id + '.txt', "r") as fo:
                    data = fo.read()
                chapter_data = decrypt_chapter(data)
                print(chapter_data)
                fd.write(chapter_info['title'].encode('utf8') + b'\r\n' + chapter_data)
                print('------------------------------------------------------------------------------------------------')
    print('[+]解密完成！！！！！')
```

效果如下：

![](https://bbs.kanxue.com/upload/attach/202102/651413_MH4FEJBSHKS7V8V.gif)

# 结语

通过以上分析，该软件基本的加固都没有，所以能很容易定位关键点，分析逻辑，而且在本人分析时候发现该软件阅读时间不够还能直接领取金币奖励，当然这是后话了。

好了就到这里了，我去看歪嘴龙王了

![](https://bbs.kanxue.com/upload/attach/202102/651413_56FMPGH4CU24CJG.jpg)
