
# 关于asprotect脱壳扔块砖.

先说1.05吧，用`imp_list`重建时，`address`文件中显示一个函数名没有，它应该是`getprocaddress`。执行程序提示“_.dll找不到”。因为asprotect要用此函数，所以做了手脚。可以在`PIMAGE_IMPORT_BY_NAME`部分可以发现一个地址和其它相差很远，`d`之，在下面不远处就可看到`call getprocaddress`了，`u getprocaddress`得到地址，将`PIMAGE_IMPORT_BY_NAME`中的错误地址改掉，再重建`import_table`就成功了。

再讲1.07，很难跟到入口。我试了很久，在`int 30`中有这样一个函数调用：`_vwin32_set_thread_context`。大概50多次后就快到了。设`bpx ... if bpcount == 32`。呵呵，自己先记下数，然后减一，我这数不准的，是16进制，别忘了。然后F10跟吧，突然有一步就回到asprotect代码中了，应该下面出现一个`call`，F10带过就执行了，要F8跟进，然后几乎可以F12了，大概3次吧，很快就可看到OEP了。清掉HINT后，重建`imp_table`，有函数名乱码。前面有人讲过方法了。其实直接用其最后一个时的方法就可以了。

另外，对1.07及以下版本的asprotect，感觉新版的`imp_list`并不如旧的好用，请别将旧的随便删了。

1.1版，用我的方法也可找到OEP，但`bpcount`大约90多次。重建`imp_table`的工作还没来得及试，更望高手指教。
