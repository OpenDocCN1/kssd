
### 获取指定城市热销榜

![image.png](https://bbs.kanxue.com/upload/attach/202204/944393_34KCRQWTSFB2VA5.jpg)

```python
def get_month(self):
    """获取城市列表"""
    response = self._parse_url(url).json()  # 提取json数据中所有的城市名
    models = jsonpath(response, '$..city_name')
    return models
```

### 获取指定城市热销榜

```python
def get_score(self, city_name):
    """获取指定城市热销榜
    city_name: 城市名
    """
    offset = 0
    while True:
        response = self._parse_url(url).json()
        tag_list = response.get('data').get('list')
        # 没有数据跳出
        if not tag_list:
            return
        for tag in tag_list:
            series_id = tag.get('series_id')  # 车系ID
            series_name = tag.get('series_name')  # 车系名称
            count = tag.get('count')  # 车系销量
            yield series_id, series_name, count
        # 翻页参数
        offset += 100
```

### 运行效果

![image.png](https://bbs.kanxue.com/upload/attach/202204/944393_HQ9Z96YZXMHQAMD.jpg)

### 资源下载

[9b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4q4I4i4K6g2X3x3K6R3I4y4e0b7&6y4o6S2Q4x3V1j5^5y4e0l9%4y4K6R3$3y4R3`.`.](https://bbs.kanxue.com/elink@525K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4q4I4i4K6g2X3x3K6R3I4y4e0b7&6y4o6S2Q4x3V1j5^5y4e0l9%4y4K6R3$3y4R3`.`.)

---

本文仅供学习交流使用，如侵立删！

---

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-4-6 23:38 被拉灯的小手编辑，原因：

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#工具脚本](https://bbs.kanxue.com/forum-41-1-137.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
