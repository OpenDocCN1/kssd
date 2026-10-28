
```python
# -\*- coding: utf-8 -\*-
import sys
import json
import requests
from time import sleep
# virustotal api key
x\_apikey = 'xxxx'
def read\_hash(file\_name):
with open(file\_name, 'r') as f:
datas = f.readlines()
return [file\_hash.strip() for file\_hash in datas]
def parse\_vt\_report(vt\_report\_json):
attributes = vt\_report\_json.get('data', {}).get('attributes', {})
parse\_data = dict()
if attributes:
# 同时记录文件的ssdeep/tlsh/vhash/文件类型
parse\_data['vhash'] = attributes.get('vhash', '')
parse\_data['magic'] = attributes.get('magic', '')
parse\_data['tlsh'] = attributes.get('tlsh', '')
parse\_data['ssdeep'] = attributes.get('ssdeep', '')
return parse\_data
def vt\_search(sha1\_hash):
url = "https://www.virustotal.com/api/v3/files/{}".format(sha1\_hash)
headers = {
"Accept": "application/json",
"x-apikey": x\_apikey
}
response = requests.request("GET", url, headers=headers)
try:
parse\_data = parse\_vt\_report(response.json())
except Exception as e:
print('error: %s, reason: %s' % (sha1\_hash, str(e)))
return parse\_data
def main():
# 包含待查询哈希的文件路径
file\_path = sys.argv[1]
vhash\_dict = dict()
file\_hashs = read\_hash(file\_path)
for file\_hash in file\_hashs:
parse\_data = vt\_search(file\_hash)
print('%s,%s' % (file\_hash, json.dumps(parse\_data)))
if parse\_data:
vhash\_dict[file\_hash] = parse\_data
else:
break
sleep(1)
with open('vhash\_test.json', 'w') as f:
f.write(json.dumps(vhash\_dict, indent=4))
if \_\_name\_\_ == '\_\_main\_\_':
main()
```

`vhash`查询结果如下：

```json
cat vhash\_test.json
{
"aba1301af627506cf67fd61410800b37c973dcb6": {
"vhash": "1240451d05151\"z",
"magic": "PE32+ executable for MS Windows (DLL) (console)",
"tlsh": "T151B22A828BB81403FA767D7013A8D6837D3D67D60820856915AAF5AA2C833C5EF10F7E",
"ssdeep": "192:8fPNlWZYWfUyfUlHDBQABJB3ejpC52qnaj68tj:iNlWZYW+DBRJ4Nle8tj"
},
"5f3ebf2c443f7010d3a5c2e5fa77c62b03ca1279": {
"vhash": "1240451d05151\"z",
"magic": "PE32+ executable for MS Windows (DLL) (console)",
"tlsh": "T140B239D6CBBC0547E9663EB012A8E9873D3E73EB4820416905A5F1981C837C5EF00F6E",
"ssdeep": "192:8Ih6WxwWFUyfUlHDBQABJj1N80Hy5qnajWi8sA+F:Vh6WxwW0DBRJjPsl+yF"
},
"3d57ce2f5149f1d9609608bc732d86637fe20cce": {
"vhash": "1240451d05151\"z",
"magic": "PE32+ executable for MS Windows (DLL) (console)",
"tlsh": "T18FB23AC2CBEC5443EAA67A7043A8E58B7D3DB3D21C60855904A6E1591CD33C2EF24E7E",
"ssdeep": "192:8JWhOMrlWBwWYUyfUlHDBQABJ5cWvKxEHsqnajTT0f7:kWhOMRWBwWhDBRJNKxUsl3TM"
},
......
}
```

### 4. mmdthash计算

使用[python\_mmdt:KNN机器学习分类结果测试分析(五)](elink@ec7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6s2k6$3L8h3#2*7P5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6q4)9#2k6V1E0z5e0W2)9J5y4f1f1$3i4K6t1#2z5f1y4Q4x3U0g2n7b7g2)9J5y4f1f1#2i4K6t1#2z5e0W2Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0g2m8y4W2)9J5y4f1f1@1i4K6t1#2b7U0W2Q4x3U0g2m8x3q4)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2b7U0q4Q4x3U0g2n7b7W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6x3#2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6b7#2)9J5y4f1f1$3i4K6t1#2b7U0g2Q4x3U0f1^5b7W2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6y4g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5z5q4)9J5y4f1f1@1i4K6t1#2b7V1q4Q4x3U0f1&6y4l9`.`.">python\_mmdt:KNN机器学习分类结果测试分析(五)[/python\_mmdt:KNN机器学习分类结果测试分析(五)]中的测试结果。

## 结果对比

### 1. `ssdeep`、`tlsh`、`vhash`、`mmdthash`结果整合

将上述计算过程生成的`ssdeep_test.json`、`tlsh_test.json`、`vhash_test.json`、`mmdthash_test.json`文件中的数据，按字典形式，整合至`ssdeep_tlsh_vhash_mmdthash_test.json`文件中，示例如下：

```json
{
"aba1301af627506cf67fd61410800b37c973dcb6": {
"vhash": "1240451d05151\"z",
"magic": "PE32+ executable for MS Windows (DLL) (console)",
"tlsh": "T151B22A828BB81403FA767D7013A8D6837D3D67D60820856915AAF5AA2C833C5EF10F7E",
"ssdeep": "192:8fPNlWZYWfUyfUlHDBQABJB3ejpC52qnaj68tj:iNlWZYW+DBRJ4Nle8tj",
"mmdthash": "07022B59:7202890402200212DA032EC310AFEF8A"
},
"5f3ebf2c443f7010d3a5c2e5fa77c62b03ca1279": {
"vhash": "1240451d05151\"z",
"magic": "PE32+ executable for MS Windows (DLL) (console)",
"tlsh": "T140B239D6CBBC0547E9663EB012A8E9873D3E73EB4820416905A5F1981C837C5EF00F6E",
"ssdeep": "192:8Ih6WxwWFUyfUlHDBQABJj1N80Hy5qnajWi8sA+F:Vh6WxwW0DBRJjPsl+yF",
"mmdthash": "07022B59:7102870402200212DD032DC30EA0F1A9"
},
......
}
```

### 2. 敏感哈希相似度计算

使用[python\_mmdt:KNN机器学习分类结果测试分析(五)](elink@ac9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6s2k6$3L8h3#2*7P5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6q4)9#2k6V1E0z5e0W2)9J5y4f1f1$3i4K6t1#2z5f1y4Q4x3U0g2n7b7g2)9J5y4f1f1#2i4K6t1#2z5e0W2Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0g2m8y4W2)9J5y4f1f1@1i4K6t1#2b7U0W2Q4x3U0g2m8x3q4)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2b7U0q4Q4x3U0g2n7b7W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6x3#2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6b7#2)9J5y4f1f1$3i4K6t1#2b7U0g2Q4x3U0f1^5b7W2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6y4g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5z5q4)9J5y4f1f1@1i4K6t1#2b7V1q4Q4x3U0f1&6y4l9`.`.">python\_mmdt:KNN机器学习分类结果测试分析(五)[/python\_mmdt:KNN机器学习分类结果测试分析(五)]中的`mmdthash`分类结果作为依据，计算关联文件之间的`ssdeep`、`tlsh`、`vhash`的相似度值。计算过程有3点需要注意：

1. `ssdeep`的计算结果是[0,100]之间的相似值，0表示完全不相关，100表示基本完全一致。为方便对比计算，本测试中，对`ssdeep`的相似值做归一化处理，即转为[0,1]之间的相似度值，转化方法：`相似度 = 相似值/100.0`

2. `tlsh`的计算结果是[0,X]之间的距离值，0表示基本完全一致，X的上限暂时不清楚，但距离越大，表示文件差异越大。为方便对比计算，本测试中，对`tlsh`的值做归一化处理，即转为[0,1]之间的相似度值，转化方法：`相似度 = 1.0 - 距离值/1160.0`（1160.0取400个测试数据中的最大值）

3. `vhash`的相似度计算方法未公开，暂时仅取两个值，0和1，0表示两个`vhash`不相等，1表示两个`vhash`相等

对比计算代码如下：

```python
# -\*- coding: utf-8 -\*-
import json
import ssdeep
import tlsh
def read\_hash(file\_name):
with open(file\_name, 'r') as f:
datas = f.readlines()
return [file\_hash.strip() for file\_hash in datas]
def ssdeep\_compare(data1, data2):
h1 = data1.get('ssdeep', '')
h2 = data2.get('ssdeep', '')
score = ssdeep.compare(h1, h2)
return score/100.0
def tlsh\_compare(data1, data2):
h1 = data1.get('tlsh', '')
h2 = data2.get('tlsh', '')
score = tlsh.diff(h1, h2)
return 1 - score/1160.0
def vhash\_compare(data1, data2):
h1 = data1.get('tlsh', '')
h2 = data2.get('tlsh', '')
score = 1.0 if h1 == h2 else 0.0
return score
def main():
mmdt\_hash\_sim = read\_hash('./mmdt\_sim.csv')
with open('./ssdeep\_tlsh\_vhash\_mmdthash\_test.json', 'r') as f:
vhash\_json = json.loads(f.read())
print('原始文件,相似文件,mmdt相似度,ssdeep相似度,tlsh相似度,vhash相似度,原始文件类型,相似文件类型')
for mhs in mmdt\_hash\_sim:
tmp = mhs.split(',')
ori\_hash = tmp[0]
sim\_hash = tmp[1]
mmdt\_sim = float(tmp[2])
ori\_data = vhash\_json[ori\_hash]
sim\_data = vhash\_json[sim\_hash]
ssdeep\_sim = ssdeep\_compare(ori\_data, sim\_data)
tlsh\_sim = tlsh\_compare(ori\_data, sim\_data)
vhash\_sim = vhash\_compare(ori\_data, sim\_data)
ori\_type = ori\_data.get('magic', '').split(' ')[0]
sim\_type = sim\_data.get('magic', '').split(' ')[0]
print('%s,%s,%.3f,%.3f,%.3f,%.3f,%s,%s' % (
ori\_hash,sim\_hash,mmdt\_sim,ssdeep\_sim,tlsh\_sim,vhash\_sim,ori\_type,sim\_type
))
if \_\_name\_\_ == '\_\_main\_\_':
main()
```

涉及的相关文件及下载地址：

1. 整合后的完整测试数据文件`ssdeep_tlsh_vhash_mmdthash_test.json`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421\_PP739ABEBEEEDUW.\_json)

2. `mmdthash`分类结果文件`mmdt_sim.csv`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421\_JKYV8K9RE55G4V4.\_csv)

3. 结果文件`ssdeep_tlsh_vhash_mmdthash_test.xlsx`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421\_WTPBRUEPET5Q2MV.xlsx)

`ssdeep_tlsh_vhash_mmdthash_test.xlsx`示例数据如下：

![](upload/attach/202201/467421\_4RMEM7ZPDV377XC.jpg)

### 3. 结果分析

如前文所述，我们使用`mmdthash`的检出结果作为基线，对`ssdeep`、`tlsh`、`vhash`的结果进行对比。

**`mmdthash`检出结果对比**

`mmdthash`相似度阈值取0.95，相似度按从大到小排序，前133个文件是检出为恶意的文件，其中132个检测正确，为恶意文件，最后一个检测错误，恶意家族检测分类结果不一致。

![](upload/attach/202201/467421\_M2DYBQGD42XVAEN.jpg)

**`mmdthash`未检出结果对比**

`mmdthash`相似度阈值取0.95，相似度按从大到小排序，后267个文件为未检出的文件，其中有200个检测正确，为干净文件，有67个检测错误，将恶意文件预测为干净文件。

![](upload/attach/202201/467421\_6CPDRAQBGHHAAK8.jpg)

**对比**

* 结合两张相似度分布图，`ssdeep`取0.8作为判定阈值

* 133个检出样本中，`ssdeep`检出109个，未检出24个

* 267个未检出样本中，`ssdeep`检出5个，未检出262个

* 同上，`tlsh`取0.8作为判定阈值

* 133个检出样本中，`tlsh`检出131个，未检出2个

* 267个未检出样本中，`tlsh`检出22个，未检出245个
