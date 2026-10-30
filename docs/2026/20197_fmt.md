
# [分享]pwnable.kr   coin1  day9

# 题目

![](upload/attach/202101/760268_7XMJV7PH9YDDY3K.png)

## 解题过程

### 1. `nc` 连接

**查看题目描述**

![题目描述](upload/attach/202101/760268_5QC68VBUNGD2ZFM.png)

在一堆真硬币中找出一个假硬币，真假硬币只有重量不相同，真硬币重量为 10，假硬币重量为 9。在 60 秒内找 100 次。

**输出案例如下**

![](upload/attach/202101/760268_Q6YBFFQ6V7TXW2E.png)

`N` 为硬币数目,`C` 为猜测次数

可以看出 `C` 每次都大于 log 以 2 为底 `N` 的对数,所以考虑用二分法找假硬币。

### 2. 编写 `exp`

```python
from pwn import *
p = remote('localhost', 9007)
p.recv()
for i in range(100):
N, C = p.recv().split(' ')
N = int(N.split('=')[1])
C = int(C.split('=')[1])
log.info('N = ' + str(N) + ' C = ' + str(C))
start = 0
end = N
while start <= end:
mid = (start + end) / 2
tmpList = [str(j) for j in range(start, mid + 1)]
tmpList = ' '.join(tmpList)
p.sendline(tmpList)
msg = p.recv().rstrip()
if msg.find('Correct') != -1:
log.info(msg)
break
weight = int(msg)
if weight % 10 == 0:
start = mid + 1
else:
end = mid
print p.recv()
```

### 3. pwn

执行 `exp`,红框内为 flag。

![](upload/attach/202101/760268_2RRGN979P5H9MX3.png)

# 题目

![](https://bbs.kanxue.com/upload/attach/202101/760268_7XMJV7PH9YDDY3K.png)

## 解题过程

### 1. `nc` 连接

**查看题目描述**

 ![题目描述](https://bbs.kanxue.com/upload/attach/202101/760268_5QC68VBUNGD2ZFM.png)

在一堆真硬币中找出一个假硬币，真假硬币只有重量不相同，真硬币重量为 10，假硬币重量为 9。在 60 秒内找 100 次。

**输出案例如下**

 ![](https://bbs.kanxue.com/upload/attach/202101/760268_Q6YBFFQ6V7TXW2E.png)

`N` 为硬币数目,`C` 为猜测次数

可以看出 `C` 每次都大于 log 以 2 为底 `N` 的对数,所以考虑用二分法找假硬币。

### 2. 编写 `exp`

```python
from pwn import *
p = remote('localhost', 9007)
p.recv()
for i in range(100):
N, C = p.recv().split(' ')
N = int(N.split('=')[1])
C = int(C.split('=')[1])
log.info('N = ' + str(N) + ' C = ' + str(C))
start = 0
end = N
while start <= end:
mid = (start + end) / 2
tmpList = [str(j) for j in range(start, mid + 1)]
tmpList = ' '.join(tmpList)
p.sendline(tmpList)
msg = p.recv().rstrip()
if msg.find('Correct') != -1:
log.info(msg)
break
weight = int(msg)
if weight % 10 == 0:
start = mid + 1
else:
end = mid
print p.recv()
```

### 3. pwn

执行 `exp`,红框内为 flag。

 ![](https://bbs.kanxue.com/upload/attach/202101/760268_2RRGN979P5H9MX3.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
