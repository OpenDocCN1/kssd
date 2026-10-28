
```python
# -*- coding: utf-8 -*-
import sys
import json
import requests
from time import sleep

# virustotal api key
x_apikey = 'xxxx'

def read_hash(file_name):
    with open(file_name,'r') as f:
        datas = f.readlines()
        return [file_hash.strip() for file_hash in datas]

def parse_vt_report(vt_report_json):
    attributes = vt_report_json.get('data', {}).get('attributes', {})
    parse_data = dict()
    if attributes:
        # 同时记录文件的ssdeep/tlsh/vhash/文件类型
        parse_data['vhash'] = attributes.get('vhash', '')
        parse_data['magic'] = attributes.get('magic', '')
        parse_data['tlsh'] = attributes.get('tlsh', '')
        parse_data['ssdeep'] = attributes.get('ssdeep', '')

    return parse_data

def vt_search(sha1_hash):
    url = "a51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6i4K6u0r3N6U0y4Q4x3V1k6X3K9h3I4W2M7#2)9J5c8W2)9%4b7W2)9%4c8l9`.`.".format(sha1_hash)
    headers = {
        "Accept": "application/json",
        "x-apikey": x_apikey
    }
    response = requests.request("GET", url, headers=headers)
    try:
        parse_data = parse_vt_report(response.json())
    except Exception as e:
        print('error: %s, reason: %s' % (sha1_hash, str(e)))
        return parse_data

def main():
    # 包含待查询哈希的文件路径
    file_path = sys.argv[1]
    vhash_dict = dict()
    file_hashs = read_hash(file_path)
    for file_hash in file_hashs:
        parse_data = vt_search(file_hash)
        print('%s,%s' % (file_hash, json.dumps(parse_data)))
        if parse_data:
            vhash_dict[file_hash] = parse_data
        else:
            break
        sleep(1)
    with open('vhash_test.json','w') as f:
        f.write(json.dumps(vhash_dict, indent=4))

if __name__ == '__main__':
    main()
```

`vhash`查询结果如下：

```
cat vhash_test.json
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

使用[python_mmdt:KNN机器学习分类结果测试分析(五)](https://bbs.kanxue.com/elink@70fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6s2k6$3L8h3#2*7P5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6q4)9#2k6V1E0z5e0W2)9J5y4f1f1$3i4K6t1#2z5f1y4Q4x3U0g2n7b7g2)9J5y4f1f1#2i4K6t1#2z5e0W2Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0g2m8y4W2)9J5y4f1f1@1i4K6t1#2b7U0W2Q4x3U0g2m8x3q4)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2b7U0q4Q4x3U0g2n7b7W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6x3#2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6b7#2)9J5y4f1f1$3i4K6t1#2b7U0g2Q4x3U0f1^5b7W2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6y4g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5z5q4)9J5y4f1f1@1i4K6t1#2b7V1q4Q4x3U0f1&6y4l9`.`.))中的测试结果。

## 结果对比

### 1. `ssdeep`、`tlsh`、`vhash`、`mmdthash`结果整合

将上述计算过程生成的`ssdeep_test.json`、`tlsh_test.json`、`vhash_test.json`、`mmdthash_test.json`文件中的数据，按字典形式，整合至`ssdeep_tlsh_vhash_mmdthash_test.json`文件中，示例如下：

```
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

使用[python_mmdt:KNN机器学习分类结果测试分析(五)](https://bbs.kanxue.com/elink@af4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6s2k6$3L8h3#2*7P5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7s2W2@1K9r3!0F1i4K6g2X3L8h3#2V1N6q4)9#2k6V1E0z5e0W2)9J5y4f1f1$3i4K6t1#2z5f1y4Q4x3U0g2n7b7g2)9J5y4f1f1#2i4K6t1#2z5e0W2Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0g2m8y4W2)9J5y4f1f1@1i4K6t1#2b7U0W2Q4x3U0g2m8x3q4)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2b7U0q4Q4x3U0g2n7b7W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6x3#2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6b7#2)9J5y4f1f1$3i4K6t1#2b7U0g2Q4x3U0f1^5b7W2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6y4g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0f1^5y4W2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1&6x3q4)9J5z5q4)9J5y4f1f1@1i4K6t1#2b7V1q4Q4x3U0f1&6y4l9`.`.))中的`mmdthash`分类结果作为依据，计算关联文件之间的`ssdeep`、`tlsh`、`vhash`的相似度值。计算过程有3点需要注意：

1. `ssdeep`的计算结果是[0,100]之间的相似值，0表示完全不相关，100表示基本完全一致。为方便对比计算，本测试中，对`ssdeep`的相似值做归一化处理，即转为[0,1]之间的相似度值，转化方法：`相似度 = 相似值/100.0`
2. `tlsh`的计算结果是[0,X]之间的距离值，0表示基本完全一致，X的上限暂时不清楚，但距离越大，表示文件差异越大。为方便对比计算，本测试中，对`tlsh`的值做归一化处理，即转为[0,1]之间的相似度值，转化方法：`相似度 = 1.0 - 距离值/1160.0`（1160.0取400个测试数据中的最大值）
3. `vhash`的相似度计算方法未公开，暂时仅取两个值，0和1，0表示两个`vhash`不相等，1表示两个`vhash`相等

对比计算代码如下：

```python
# -*- coding: utf-8 -*-
import json
import ssdeep
import tlsh

def read_hash(file_name):
    with open(file_name,'r') as f:
        datas = f.readlines()
        return [file_hash.strip() for file_hash in datas]

def ssdeep_compare(data1, data2):
    h1 = data1.get('ssdeep', '')
    h2 = data2.get('ssdeep', '')
    score = ssdeep.compare(h1, h2)
    return score/100.0

def tlsh_compare(data1, data2):
    h1 = data1.get('tlsh', '')
    h2 = data2.get('tlsh', '')
    score = tlsh.diff(h1, h2)
    return 1 - score/1160.0

def vhash_compare(data1, data2):
    h1 = data1.get('tlsh', '')
    h2 = data2.get('tlsh', '')
    score = 1.0 if h1 == h2 else 0.0
    return score

def main():
    mmdt_hash_sim = read_hash('./mmdt_sim.csv')
    with open('./ssdeep_tlsh_vhash_mmdthash_test.json','r') as f:
        vhash_json = json.loads(f.read())
    print('原始文件,相似文件,mmdt相似度,ssdeep相似度,tlsh相似度,vhash相似度,原始文件类型,相似文件类型')
    for mhs in mmdt_hash_sim:
        tmp = mhs.split(',')
        ori_hash = tmp[0]
        sim_hash = tmp[1]
        mmdt_sim = float(tmp[2])
        ori_data = vhash_json[ori_hash]
        sim_data = vhash_json[sim_hash]
        ssdeep_sim = ssdeep_compare(ori_data, sim_data)
        tlsh_sim = tlsh_compare(ori_data, sim_data)
        vhash_sim = vhash_compare(ori_data, sim_data)
        ori_type = ori_data.get('magic', '').split(' ')[0]
        sim_type = sim_data.get('magic', '').split(' ')[0]
        print('%s,%s,%.3f,%.3f,%.3f,%.3f,%s,%s' % (
            ori_hash,sim_hash,mmdt_sim,ssdeep_sim,tlsh_sim,vhash_sim,ori_type,sim_type
        ))

if __name__ == '__main__':
    main()
```

涉及的相关文件及下载地址：

1. 整合后的完整测试数据文件`ssdeep_tlsh_vhash_mmdthash_test.json`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421_PP739ABEBEEEDUW._json)
2. `mmdthash`分类结果文件`mmdt_sim.csv`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421_JKYV8K9RE55G4V4._csv)
3. 结果文件`ssdeep_tlsh_vhash_mmdthash_test.xlsx`下载地址：[download](https://bbs.pediy.com/upload/attach/202201/467421_WTPBRUEPET5Q2MV.xlsx)

### 3. 结果分析

如前文所述，我们使用`mmdthash`的检出结果作为基线，对`ssdeep`、`tlsh`、`vhash`的结果进行对比。

**`mmdthash`检出结果对比**

`mmdthash`相似度阈值取0.95，相似度按从大到小排序，前133个文件是检出为恶意的文件，其中132个检测正确，为恶意文件，最后一个检测错误，恶意家族检测分类结果不一致。

**`mmdthash`未检出结果对比**

`mmdthash`相似度阈值取0.95，相似度按从大到小排序，后267个文件为未检出的文件，其中有200个检测正确，为干净文件，有67个检测错误，将恶意文件预测为干净文件。

**对比**

- 结合两张相似度分布图，`ssdeep`取0.8作为判定阈值
  - 133个检出样本中，`ssdeep`检出109个，未检出24个
  - 267个未检出样本中，`ssdeep`检出5个，未检出262个
- 同上，`tlsh`取0.8作为判定阈值
  - 133个检出样本中，`tlsh`检出131个，未检出2个
  - 267个未检出样本中，`tlsh`检出22个，未检出245个
- 同上，`vhash`由于取值只有0和1，所以
  - 133个检出样本中，`vhash`检出5个，未检出128个
  - 267个未检出样本中，`tlsh`检出0个，未检出267个

通过人工分析异常点对应的样本，`mmdthash`阈值取0.95，`ssdeep`阈值取0.8，`tlsh`阈值取0.8，得到统计数据如下：

| 算法名称 | 检出数量 | 未检出数量 | TP | TN | FP | FN | 准确率ACC | 召回率REC | 精确率PRE |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| mmdthash | 133 | 267 | 132 | 200 | 0 | 68 | 0.83 | 0.66 | 1.0 |
| ssdeep | 114 | 286 | 114 | 200 | 0 | 86 | 0.785 | 0.57 | 1.0 |
| tlsh | 153 | 247 | 145 | 192 | 8 | 55 | 0.8425 | 0.725 | 0.948 |
| vhash | 5 | 395 | 5 | 200 | 0 | 195 | 0.5125 | 0.025 | 1.0 |

在上述`mmdthash`阈值取0.95，`ssdeep`阈值取0.8，`tlsh`阈值取0.8的前提下，可得出如下结论：

- `tlsh`的准确率、召回率、误报率最高（`误报率 = 1.0 - 准确率`）
- `mmdthash`的准确率和召回率次之，准确率比`tlsh`低1.5%，召回率比`tlsh`低9.0%，误报率比`tlsh`低5.5%
- `ssdeep`的准确率和召回率再次之，准确率比`tlsh`低6.8%，召回率比`tlsh`低21.4%，误报率比`tlsh`低5.5%
- `vhash`情况最特殊，数据做观察对比即可

综上，基于本篇的测试结果，敏感哈希效果的综合考虑如下：

**`tlsh` > `mmdthash` > `ssdeep` > `vhash`**

## 其他

### 1. 400个测试文件类型分布

- PE文件占比96%
- ELF文件占比2%
- 其他文件占比2%

### 2. 400个测试文件计算耗时

为体现差异，特意在低性能的树莓派上执行，耗时如下：

| 算法名称 | real | user | sys |
| --- | --- | --- | --- |
| mmdthash | 74.259s | 69.772s | 4.028s |
| ssdeep | 18.095s | 15.485s | 2.572s |
| tlsh | 17.751s | 14.12s | 3.558s |

## 反思与收获

1. `ssdeep`的论文写的真不错，通俗易懂，论证过程清晰，数学理论支撑非常充足，非常棒。
2. `tlsh`的生态做的真的好，已经出了6篇相关的paper，参与了6次会议，足以证明其应用效果。
3. 在实现`mmdthash`的过程中，过于闭门造车，有很多不成熟的想法、思路在`tlsh`相关的paper里都讲述的很清楚、很详细，应用也非常成熟了。
4. `mmdthash`的性能还要持续优化

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)
[#工具脚本](https://bbs.kanxue.com/forum-41-1-137.htm)
[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)

上传的附件：

- [ssdeep_tlsh_vhash_mmdthash_test.xlsx](javascript:void(0);)
  （55.07kb，12次下载）
- [ssdeep_tlsh_vhash_mmdthash_test.py](javascript:void(0);)
  （1.77kb，11次下载）
- [ssdeep_tlsh_vhash_mmdthash_test.json](javascript:void(0);)
  （338.20kb，10次下载）
- [mmdt_sim.csv](javascript:void(0);)
  （34.69kb，10次下载）
