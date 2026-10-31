
# [原创]堆栈溢出原理的简单学习及分享

根据看雪前辈的文章，我学习了堆栈的简单溢出原理。

## 出处
https://bbs.pediy.com/thread-38803.htm
https://bbs.pediy.com/thread-38864.htm

文章已说明这个程序需利用溢出，显示 “Succeed,you win!” 算成功。于是有了我在win7 64位系统下的动手实验。

### 1. 正常输入
正常输入不超出范围时，栈的情况是这样的，还可以看到 `0018FF4C` 处标明了返回点。

![图片描述](upload/attach/202002/302915_F6DHMRDNDVRC8JD.png)

### 2. 输入超出范围
当输入超出范围是时，可以看到地址 `0018FF48` 开始到 `0018FF54` 之间被一些数字给覆盖住了，当这些数字在栈内返回时，就会跳到一个无效的地址，程序崩溃。我们就可以利用这点，达到溢出覆盖其地址，跳转到我们想要的地方。

![图片描述](upload/attach/202002/302915_5EY6UY6EE9WV8BV.png)
![图片描述](upload/attach/202002/302915_HMJ3HGDE5DAPHFJ.png)
![图片描述](upload/attach/202002/302915_GCFKAQ9XAQ4VQ3F.png)

### 3. 构造特殊字符串
这个程度会判断三次，前两次随便输什么，只要不超出范围，最后一次要构造出一个特殊的字符串，构造出的的字符串为: `testtttt`接着输入一个`'c'`,再按住ALT输入16松开ALT,然后输入`'@'`后回车就看到成功提示了，但是程序崩了。

![图片描述](upload/attach/202002/302915_PHJJV9YNSUXCTYN.png)

### 4. 报错问题
但是这样没有解决报错的问题，因为，程序没有按正常流程退出，所以有另外一种搞法，结合前辈的经验，再过一遍。

### 5. 使用 OD 加载程序
用OD载入 `overflowMe.exe` 这个程序，搜索字符串 `"please input password:"` ,在此处下个断点，输入三次字符，注意最后一次比较重要，需单步往下走。

![图片描述](upload/attach/202002/302915_UNK5PCNWFFYEJT8.png)

### 6. 观察堆栈
当执行完 `00401054` 这个地方时，程序就准备退出了，这时观察堆栈。

![图片描述](upload/attach/202002/302915_5RC2U4RZNETZZU8.png)

### 7. 修改堆栈地址
我们把堆栈地址 `0018FF44` 和 `0018FF48` 之间填充`CC`，而 `0018FF4C` 我填的是在 `kernal32` 模块里面的一个地址（我自己找的地址为`75423132`，你问我怎么找的，就是在反汇编窗口中右键-查看-模块'`kernal32'`，点进去），此指令要为 `jmp esp`。

![图片描述](upload/attach/202002/302915_AX3PYPM7AMHGGNC.png)

手动修改堆栈前：
![图片描述](upload/attach/202002/302915_NC7AUR7ZVBX4FDZ.png)

手动修改堆栈后：
![图片描述](upload/attach/202002/302915_9EJS6ZGHQ99TRA9.png)

### 8. 构造 shellcode
然后还在 `0018FF50` 开始处构造一个`shellcode`。

![图片描述](upload/attach/202002/302915_9FYFWBFKPSCE7WG.png)

```
0018FF50 58 pop eax
0018FF51 35 1E42C17C xor eax,0x7CC1421E
0018FF56 50 push eax
0018FF57 FF5424 0E call dword ptr ss:[esp+0xE] // 0xE 是根据系统运行时堆栈 开始处的偏移量，不同系统可能会不一样，我的机器上运行到这里时，esp的值是 "0018FF50"，两值相加，一下就看明白了。
0018FF5B 33C0 xor eax,eax // 返回 0 的值
0018FF5D C3 retn
0018FF5E FF10 call dword ptr ds:[eax] // 刚好是我们想要的（看中间）004010FF 地址，如下图。
0018FF60 40 inc eax
0018FF61 00FF add bh,bh
```

![图片描述](upload/attach/202002/302915_K78284ZA9BUZXSW.png)

### 9. 修改返回地址
完成以上，再单步走，会来到自己写的代码里，当运行到 `0018FF5D` 时，把返回的地址改变为 `00401393`，这时会来到正常退出流程，且 `eax = 0`，（相当于高级语言里面的 `return 0`）。

![图片描述](upload/attach/202002/302915_KRVERVV2WXBZ2KX.png)

改变前：
![图片描述](upload/attach/202002/302915_K9H688F3N4SZKD7.png)

改变后：
![图片描述](upload/attach/202002/302915_ZD6AETCXAB8E87B.png)

最终修改好的
![图片描述](upload/attach/202002/302915_8G25P7FQ2238RQA.png)

### 10. 正常退出
这样程序就不报错了，正常退出。

总结：还是要多动手调试，只看文章不动手，学不到东西，最好是写篇文章记录下学习过程，方便查看。

根据看雪前辈的文章，我学习了堆栈的简单溢出原理。

## 出处
https://bbs.pediy.com/thread-38803.htm
https://bbs.pediy.com/thread-38864.htm

```
文章已说明这个程序需利用溢出，显示  “Succeed,you win!” 算成功。于是有了我在win7 64位系统下的动手实验。
```

### 1. 正常输入不超出范围
正常输入不超出范围时，栈的情况是这样的，还可以看到 `0018FF4C` 处标明了返回点。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_F6DHMRDNDVRC8JD.png)

### 2. 输入超出范围
当输入超出范围是时，可以看到地址 `0018FF48` 开始到 `0018FF54` 之间被一些数字给覆盖住了，当这些数字在栈内返回时，就会跳到一个无效的地址，程序崩溃。我们就可以利用这点，达到溢出覆盖其地址，跳转到我们想要的地方。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_5EY6UY6EE9WV8BV.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_HMJ3HGDE5DAPHFJ.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_GCFKAQ9XAQ4VQ3F.png)

### 3. 构造特殊字符串
这个程度会判断三次，前两次随便输什么，只要不超出范围，最后一次要构造出一个特殊的字符串，构造出的的字符串为: `testtttt`接着输入一个`'c'`,再按住ALT输入16松开ALT,然后输入`'@'`后回车就看到成功提示了，但是程序崩了。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_PHJJV9YNSUXCTYN.png)

### 4. 报错问题
但是这样没有解决报错的问题，因为，程序没有按正常流程退出，所以有另外一种搞法，结合前辈的经验，再过一遍。

### 5. 使用 OD 加载程序
用OD载入 `overflowMe.exe` 这个程序，搜索字符串 `"please input password:"` ,在此处下个断点，输入三次字符，注意最后一次比较重要，需单步往下走。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_UNK5PCNWFFYEJT8.png)

### 6. 观察堆栈
当执行完 `00401054` 这个地方时，程序就准备退出了，这时观察堆栈。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_5RC2U4RZNETZZU8.png)

### 7. 修改堆栈地址
我们把堆栈地址 `0018FF44` 和 `0018FF48` 之间填充`CC`，而 `0018FF4C` 我填的是在 `kernal32` 模块里面的一个地址（我自己找的地址为`75423132`，你问我怎么找的，就是在反汇编窗口中右键-查看-模块'`kernal32'`，点进去），此指令要为 `jmp esp`。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_AX3PYPM7AMHGGNC.png)

手动修改堆栈前：
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_NC7AUR7ZVBX4FDZ.png)

手动修改堆栈后：
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_9EJS6ZGHQ99TRA9.png)

### 8. 构造 shellcode
然后还在 `0018FF50` 开始处构造一个`shellcode`。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_9FYFWBFKPSCE7WG.png)

```
0018FF50 58 pop eax
0018FF51 35 1E42C17C xor eax,0x7CC1421E
0018FF56 50 push eax
0018FF57 FF5424 0E call dword ptr ss:[esp+0xE] // 0xE 是根据系统运行时堆栈 开始处的偏移量，不同系统可能会不一样，我的机器上运行到这里时，esp的值是 "0018FF50"，两值相加，一下就看明白了。
0018FF5B 33C0 xor eax,eax // 返回 0 的值
0018FF5D C3 retn
0018FF5E FF10 call dword ptr ds:[eax] // 刚好是我们想要的（看中间）004010FF 地址，如下图。
0018FF60 40 inc eax
0018FF61 00FF add bh,bh
```

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_K78284ZA9BUZXSW.png)

### 9. 修改返回地址
完成以上，再单步走，会来到自己写的代码里，当运行到 `0018FF5D` 时，把返回的地址改变为 `00401393`，这时会来到正常退出流程，且 `eax = 0`，（相当于高级语言里面的 `return 0`）。

![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_KRVERVV2WXBZ2KX.png)

改变前：
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_K9H688F3N4SZKD7.png)

改变后：
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_ZD6AETCXAB8E87B.png)

最终修改好的
![图片描述](https://bbs.kanxue.com/upload/attach/202002/302915_8G25P7FQ2238RQA.png)

### 10. 正常退出
这样程序就不报错了，正常退出。

总结：还是要多动手调试，只看文章不动手，学不到东西，最好是写篇文章记录下学习过程，方便查看。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
