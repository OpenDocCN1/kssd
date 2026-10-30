
# [原创]赛题1提交 - vxasm

支持论坛举办此类活动，重在参与！![](https://bbs.kanxue.com/view/img/face/1.gif) ![](https://bbs.kanxue.com/view/img/face/1.gif)

主要的算法代码如下，算出结果的时间不超过1毫秒。

```
//

//函数功能：统计整数n中数字k的出现频率

//

//参数1：n - 被统计的整数

//参数2：k - 要统计的数字

//返回值：该数字的出现频率

//

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

            // 如果当前位数字小于k，则：k的出现次数＝高位数字×当前因子

            count += higher * factor;

        }

        else if (curr == k)

        {

            // 如果当前位数字等于k，则：k的出现次数＝高位数字×当前因子＋低位数字＋1

            count += higher * factor + lower + 1;

        }

        else if (curr > k)

        {

            // 如果当前位数字大于k，则：k的出现次数＝（高位数字＋1）×当前因子

            count += (higher + 1) * factor;

        }



        factor *= 10;

    }



    return count;

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [vxasm\_1.rar](javascript:void(0);)

  （18.61kb，10次下载）
