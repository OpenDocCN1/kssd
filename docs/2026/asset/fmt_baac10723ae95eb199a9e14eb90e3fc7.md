
```python
# -*- coding: utf-8 -*-
import os
import sys
import hashlib
import json
import tlsh

# 遍历目录
def list_dir(root_dir):
    files = os.listdir(root_dir)
    for f in files:
        file_path = os.path.join(root_dir, f)
        yield file_path

# 生成sha1
def gen_sha1(file_name):
    with open(file_name, 'rb') as f:
        s = f.read()
        _s = hashlib.sha1()
        _s.update(s)
        return _s.hexdigest()

def gen_tlsh(file_name):
    with open(file_name, 'rb') as f:
        s = f.read()
        _s = tlsh.hash(s)
        return _s

def main():
    # 输入785个文件的路径
    file_path = sys.argv[1]
    tlsh_dict = dict()
    for file_name in list_dir(file_path):
        file_sha1 = gen_sha1(file_name)
        tlsh_hash = gen_tlsh(file_name)
        print('%s,%s' % (file_sha1, tlsh_hash))
        tlsh_dict[file_sha1] = tlsh_hash
    with open('tlsh_test.json', 'w') as f:
        f.write(json.dumps(tlsh_dict, indent=4))

if __name__ == '__main__':
    main()
```

`tlsh`计算结果示例如下：

```bash
cat tlsh_test.json
{
  "0ec279513e9e8a0e8f6e7c170b9462b60d9888c6": "T1616423D5248C5DF8E251CCF4C73AB60493EADA48BF516B75BDD9C2692FF2480C93A214",
  "0ad6db9128353742b3d4c8a5fc1993ca8bf399f1": "T13D73024483EBEDA8EE040AB0124C43B9CBAD8D1B7659653DFD3864D1FC064AE47269A6",
  "e3dc592a0fa552beb35ebcb4160e5e4cb4686f17": "T1CF93293D766924E5E139C17CC5474E0AF772B025071227EF06A4C2BE1F97BE06C39AA5",
  "c8e1100b1e38e5c5e671a23cd49d98e315b74a36": "T17F34391A57EC0465F1B7923589B34919F233B8625731E2DF109082BC2E27FD8BE36B56",
  "0ae0cba5b411541cc8d9f94e01151fec9d6b9242": "T12D5208C71F69F7D4C19F85F84A3B623E1EA4616A6111412057DD3E92BC1C3DBFA2A09C",
  ......
}
```

### 3. vhash计算

virustotal没有开源`vhash`的计算方法，目前只能通过virustotal的web api接口进行查询。通过virustotal的接口，同时可以获取到对应文件的`ssdeep`和`tlsh`值（估计`tlsh`上virustotal的时间比较短，一些老样本会缺失`tlsh`值）。virustotal的api文档点击[这里](https://bbs.kanxue.com/elink@edcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2N6X3W2J5N6i4y4@1L8%4c8S2L8q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3k6A6L8r3g2Q4x3X3c8A6L8X3k6G2)，可以直接在页面上对接口进行测试以及生成对应开发语言的代码，使用非常方便。

注册virustotal的账户，申请`api_key`请参考virustotal的文档，几分钟即可完成。使用api接口进行查询的时候，注意查询频率限制。

使用python查询virustotal的代码如下：
