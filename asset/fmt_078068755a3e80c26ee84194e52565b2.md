
# 赛题 1 提交 - vxasm

> 原文：[`www.pediy.com/kssd/pediy10/74977.html`](https://www.pediy.com/kssd/pediy10/74977.html)

支持论坛举办此类活动，重在参与！![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

主要的算法代码如下，算出结果的时间不超过 1 毫秒。

代码:

```
//函数功能：统计整数 n 中数字 k 的出现频率
//参数 1：n - 被统计的整数
//参数 2：k - 要统计的数字
//返回值：该数字的出现频率
int countx(int n, int k)
{
    int count = 0;
    int factor = 1;
    int lower = 0;
    int curr = 0;
    int higher = 0;

    while (n/factor != 0)
    {
        lower = n - (n/factor) * factor;
        curr = (n/factor) % 10;
        higher = n / (factor*10);
        
        if (curr < k )
        {
            // 如果当前位数字小于 k，则：k 的出现次数＝高位数字×当前因子
            count += higher * factor;
        }
        else if (curr == k)
        {
            // 如果当前位数字等于 k，则：k 的出现次数＝高位数字×当前因子＋低位数字＋1
            count += higher * factor + lower + 1;
        }
        else if (curr > k)
        {
            // 如果当前位数字大于 k，则：k 的出现次数＝（高位数字＋1）×当前因子
            count += (higher + 1) * factor;
        }

        factor *= 10;
    }

    return count;
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || vxasm_1.rar |
