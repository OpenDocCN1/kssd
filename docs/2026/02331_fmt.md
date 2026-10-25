
# Visual Zip Password Recovery 4.0破解

(1千字)

此软件可破解Winzip加密的密码,不注册会不断弹出对话框。本人认为此软件很优秀,速度快,支持多台计算机并行计算。

打开注册对话框，name:yxg,registration key:78787878,在trw2000中下`bpx hmemcpy`，被拦下单步跟踪，这里离比对处还很远，眼都有些受不了了，看到

```
:474d7a mov edx,[ebp-4];edx:'yxg'
:474d95 mov edx,[ebp-8];edx:'78787878'
:48c058 mov eax,[48fb48]
:xxxxxx mov eax,[eax]; eax:'78787878'
:xxxxxx call 403ed0; 计算密码长度
:xxxxxx cmp eax,byte +10; 长度为16吗？
:xxxxxx jnz near 48c22a; 不同则跳出'The password is wrong!'
```

将`eax`改为10避开此跳，向下是对用户名密码的判断算法，不是太复杂，但懒得研究了，知道是从密码中取出一些数，和另外算出的8个数异或，最后一个为0则注册成功。直到

```
:48c11c cmp byte[ebp-7c],00
:xxxxxx jnz near 48c1ed; 如果不为0，跳出'The password is wrong!'
```

修改`[ebp-7c]`的值，避过此跳，按F5则注册成功！

所以改`jnz near 48c1ed`为`nop`就行了，

搜索:`0f85c700000000`

改为:`909090909090`

后记：因为暴力破解，并没有弄清楚注册过程，每次运行都要重新注册，只要密码为16位就可以。有空会弄清楚的，并写出注册机。
