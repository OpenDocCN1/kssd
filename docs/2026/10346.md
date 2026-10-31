
# [原创]1.LeetCode刷题-阿姆斯特朗数

采用每日一练的方式学习思考解题思路，探寻优化方法。

## 有趣的自幂数

阿姆斯特朗数其实是一种自幂数，三位的阿姆斯特朗数又被称为水仙花数。水仙花数的名字来自于一个凄美的神话故事，美少年纳西索斯苦苦追求自己的倒影最终化作一朵晶莹剔透的水仙花。之后，纳西索斯的名字(`Narcissus`)就成了“自我欣赏”的代名词，用水仙花数来称呼3位的自幂数，或许要有些描述“自赏”的味道。

一位的自幂数又称独身数。

三位的自幂数又称水仙花数。

四位的自幂数又称四叶玫瑰数。

五位的自幂数又称五角星数。

六位的自幂数又称六合数。

七位的自幂数又称北斗七星数。

八位的自幂数又称八仙数。

九位的自幂数又称九九重阳数。

十位的自幂数又称十全十美数。

## 阿姆斯特朗数

如果一个`n`位正整数等于其各位数字的`n`次方之和,则称该数为阿姆斯特朗数。

给你一个正整数 `N`，让你来判定他是否是阿姆斯特朗数，是 则返回 `true`，不是 则返回 `false`。

例如：`153` 是一个 `3` 位数，且 `153 = 1^3 + 5^3 + 3^3`

## 判断阿姆斯特朗数

1、首先将一个数中的每一位数字提取出来。

```
def Test(N):
tmp=N
list=[]
while tmp / 10.0 >0:
s=tmp % 10
list.append(s)
tmp=tmp // 10
print(list)
Test(123)
[3, 2, 1]
```

2、将提取出来的每一位数字与当前数本身的位数进行指数运算并且累加。比较累加的结果与数字本身是否相同。

```
def Test(N):
tmp=N
list=[]
while tmp / 10.0 >0:
s=tmp % 10
list.append(s)
tmp=tmp // 10
sum = 0
for item in list:
sum+=item**len(list)
if sum == N:
return True
else:
return False
print(Test(153))
True
```

优化方案二、 12行

```
def Test(N):
tmp=N
list=[]
while tmp / 10.0 >0:
s=tmp % 10
list.append(s)
tmp=tmp // 10
sum = 0
for item in list:
sum+=item**len(list)
return sum == N
print(Test(253))
```

优化方案三、10行

```
def test2(N):
strN=str(N)
list=[]
for item in strN:
list.append(item)
sum=0
for item in list:
sum+=int(item)**len(list)
return sum == N
print(test2(153))
```

优化方案四、8行

```
def test2(N):
strN=str(N)
list=[item for item in strN]
sum=0
for item in list:
sum+=int(item)**len(list)
return sum == N
print(test2(153))
```

优化方案五 7行

```
def test2(N):
list1=list(str(N))
sum=0
for item in list1:
sum+=int(item)**len(list1)
return sum == N
print(test2(153))
```

## 还有更简洁的代码吗？请贴出来

采用每日一练的方式学习思考解题思路，探寻优化方法。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-2-5 23:08

被洪七公.编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
