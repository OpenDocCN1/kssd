
# [原创]蒙特卡罗算圆周率

```
import random

def main():

    sum = 1000000

    sum_in = 0

    for i in xrange(0,sum):

        x = random.random()

        y = random.random()

        if (x-0.5)*(x-0.5)+(y-0.5)*(y-0.5)<=0.25:

            sum_in = sum_in+1

    print('%s/%s = %s'%(sum_in,sum,float(sum_in)/sum))

    print('PI = %s'%(4*float(sum_in)/sum))

if __name__=='__main__':

    main()
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
