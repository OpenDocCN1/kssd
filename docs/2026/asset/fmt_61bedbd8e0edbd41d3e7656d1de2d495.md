
```c
if (aP_getbit(&ud))
{
    if (aP_getbit(&ud))
    {
        if (aP_getbit(&ud))
        {
            offs = 0;
            for (i = 4; i; i--) offs = (offs << 1) + aP_getbit(&ud);
            if (offs)
            {
                *ud.destination = *(ud.destination - offs);
                ud.destination++;
            } else {
                *ud.destination++ = 0x00;
            }
            LWM = 0;
        } else {
            offs = *ud.source++;
            len = 2 + (offs & 0x0001);
            offs >>= 1;
            if (offs)
            {
                for (; len; len--)
                {
                    *ud.destination = *(ud.destination - offs);
                    ud.destination++;
                }
            } else done = 1;
            R0 = offs;
            LWM = 1;
        }
    } else {
        offs = aP_getgamma(&ud);
        if ((LWM == 0) && (offs == 2))
        {
            offs = R0;
            len = aP_getgamma(&ud);
            for (; len; len--)
            {
                *ud.destination = *(ud.destination - offs);
                ud.destination++;
            }
        } else {
            if (LWM == 0) 
                offs -= 3; 
            else 
                offs -= 2;
            offs <<= 8;
            offs += *ud.source++;
            len = aP_getgamma(&ud);
            if (offs >= 32000) len++;
            if (offs >= 1280) len++;
            if (offs < 128) len += 2;
            for (; len; len--)
            {
                *ud.destination = *(ud.destination - offs);
                ud.destination++;
            }
            R0 = offs;
        }
        LWM = 1;
    }
} else {
    *ud.destination++ = *ud.source++;
    LWM = 0;
}
```

以上的几个壳都是很“规矩”的壳，以后将对反调试 复杂变换等的壳进行学习 ![](https://bbs.kanxue.com/view/img/face/27.gif)

好好学习，天天进步！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
