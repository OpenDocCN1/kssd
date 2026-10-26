
- 如果 `username` 的长度小于 2，将提示：`Please input correct User Name`
- 如果 `registration code` 的长度小于 8，将提示：`Please input correct Registration Code`

2. 基于 `username` 进行计算，代码中我用：

```
numax: start address, simple calculating related to username
numax: end address, simple calculating related to username
```

进行了明确的标明。

仔细阅读下这段代码，我发现大多都是进行除 10 取余的计算，并保存余数。在该段的结束部分，由计算了 `username` 字串 ascii 的码的总和。

3. 基于 `registration` 进行计算，代码中我用：

```
numax: start address, simple calculating related to regcode
numax: end address, simple calculating related to regcode
```

进行了明确的标明。

个人认为，这段基本上就是废码，将以个个字符存入栈后在比较 `registration code` 的长度。客观地来说，十分类似花指令。

4. 基于第二段所得的余数及 `username` 字串 ascii 的码的和，与 `registration code` 进行比较。共五伦，如果都相等，认为 `username` 与 `registration code` 有效。

5. 注册成功或失败的提示框。

step 3：

获得注册码。

对于想研究算法的人来说呢，可以粗略的看下 step 2 中提到的代码的第 2 部分：“基于 `username` 进行计算”，但对于破解的人来说嘛，呵呵，只要找到 step 2 中提到的第 4 部分就好了。

![](http://bbs.pediy.com/picture.php?albumid=143&pictureid=340)

这次成功的破解对我来说可是意义重大，以前所学虽然没有得到全面的认证，但却至少也从一点反应出我曾经的努力并没有白费。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
