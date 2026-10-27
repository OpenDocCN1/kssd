
# [原创]2.LeetCode刷题-自除数

## 采用每日一练的方式学习思考解题思路，探寻优化方法。

## **自除数**

自除数是指可以被它包含的每一位数除尽的数。也可以理解为，自除数对组成其本身的每一位数字进行取余结果都为0。

需要注意，自除数不允许包含 0 。

例如，128 是一个自除数，因为 `128 % 1 == 0`，`128 % 2 == 0`，`128 % 8 == 0`。

**题目：现在，给定上边界和下边界数字，输出一个列表，列表的元素是边界（含边界）内所有的自除数。请尝试编程解决**

## 筛选自除数

1、首先将这个数的每一个数字提取出来。

2、检查数字中是否存在0，如果存在，则判定不是自除数。

3、使用原数对每一位数字进行除法运算，判定是否存在余数非零结果。

### 方案一、

```
def yyds(left,right):
l=[]
for num in range(left,right+1):
numStr=str(num)
numList=list(str(num))
res=True
for item in numList:
itemNum=int(item)
if itemNum==0:
res=False
break
if num % itemNum !=0:
res=False
if res:
l.append(num)
return lprint(yyds(1,100))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 15, 22, 24, 33, 36, 44, 48, 55, 66, 77, 88, 99]
```

### 方案二、

```
def yyds(left,right):
l=[num for num in range(left,right+1) if([item for item in list(str(num)) if(int(item)!=0 and num % int(item)==0)]==list(str(num)))]
return l
print(yyds(1,100))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 15, 22, 24, 33, 36, 44, 48, 55, 66, 77, 88, 99]
```

## 还有更简洁的代码吗？请贴出来

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
