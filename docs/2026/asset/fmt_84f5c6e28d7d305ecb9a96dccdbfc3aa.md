
接着这里的`(*(&buf + (((unsigned __int8)v16 ^ 0x13) & 1)))`，由于`& 1`操作的存在，因此还是会调用到`shm_mem_dump`函数，但是此时的`v16`其实`int`类型，所以这里如果传入一个负数，就会导致一个溢出，由于是拷贝到`v19`，而`v19`是一个栈上的值，所以会造成栈溢出

![](https://bbs.kanxue.com/upload/attach/202510/971428_TA6NMMU5U966FYC.png)

结合上方的分析，我们可以发现，我们这里的栈溢出是有前置条件的

- `id`要为负数
- `niksessid`需要提前知道，这样才可以提前设置`rk`的值，然后进入`rootkey_privileged_mode()`函数

所以接下来再分析的同时还需要解决这两个问题

## 目录截断

分析`upload.cgi`文件

当`action`为`download`的时候，可以设置`path`字段，然后下面存在一个`snprintf`函数，这里其实存在一个截断的问题，同时注意`check_suffix(filepath, "nik.gif")`会检查文件的后缀是不是`nik.gif`，那么其实可以尝试去构造一个路径，类似于`/tmp/./././xxxxxrootkeynik.gif`的形式，这样保证`0x60`截断了`nik.gif`，同时绕过后缀的检查，然后同时还可以访问到`/tmp/rootkey`文件，接着下面读取文件内容，同时满足文件大小是64b，然后会将内容输出

![](https://bbs.kanxue.com/upload/attach/202510/971428_J7UCSWRCSHW2YW7.png)

但是这里存在一个问题就是`/tmp/rootkey`这个文件在默认状态下不存在，所以我们需要找到创建这个文件的逻辑

```
root@debian-armel:/var/www# ls /tmp/
store
```

可以通过`grep -r`查找，发现了`watch`中存在`rootkey`这个字符串

```
root@debian-armel:/var/www/cgi-bin# grep -r rootkey ./
Binary file ./watch matches
Binary file ./manage.cgi matches
```

分析这个`binary`可以发现，这里会创建一个64b大小的文件，然后再重命名为`/tmp/rootkey`，最后输出Good luck

![](https://bbs.kanxue.com/upload/attach/202510/971428_9DQG3ES2T7DGZY3.png)

那么通过这样，我们可以创建`/tmp/rootkey`文件，同时通过目录截断读取这个文件的内容，也就是我们知道了`niksessid`，这样我们可以提前设置`rk`的值，绕过判断，触发漏洞逻辑

通过分析`lang.cgi`，我们可以通过设置`setid`字段设置`id`的值，然后这个值就会被保存到`/tmp/store/id.txt`文件里

![](https://bbs.kanxue.com/upload/attach/202510/971428_JNCKYBS533ECXDC.png)

那么至此为止，我们已经具备了触发栈溢出的能力了，那么现在的问题是如何写`rop`

## 字符逃逸

回头看栈溢出的位置，其实是拷贝共享内存的内容到栈上，那么现在就需要接着去看共享内存的值应该如何设置

![](https://bbs.kanxue.com/upload/attach/202510/971428_X9AC75F2QSQU2YE.png)

这里会根据我们传入的`action`来决定执行什么函数

![](https://bbs.kanxue.com/upload/attach/202510/971428_G49H7J2J3NECKFB.png)

这是`action_dispatch_table`的内容

![](https://bbs.kanxue.com/upload/attach/202510/971428_HRFRGKKTC64SY3X.png)

问题出现在`setpkfunc`中，这里会将`/tmp/store/publicfile.txt`的内容读出来，仔细看会发现存在一个`off-by-one`

> 但是其实根本没用

![](https://bbs.kanxue.com/upload/attach/202510/971428_SWR4TR94ZP4AP8K.png)

接着这里会有一个转化，就是转化为16进制的操作，同时仍然存在`off-by-one`

![](https://bbs.kanxue.com/upload/attach/202510/971428_MQNF76D7QDZ5ATY.png)

接着的操作比较迷惑，一些奇怪的判断，然后需要注意的是当`idx == 80`时，会直接执行`final_data[81] = chunk[80];`然后退出循环，也就意味着`chunk[80]`不需要经过前面的字符检查，直接被赋值到`final_data[81]`

![](https://bbs.kanxue.com/upload/attach/202510/971428_52EGXF4SUD2BGFJ.png)

下面就可以通过设置`cnt1`和`cnt2`来将我们逃逸的字符写到共享内存上

![](https://bbs.kanxue.com/upload/attach/202510/971428_MWV27GKBMZKYAK7.png)

## 命令执行

由于这题没有`pie`和`aslr`，所以可以直接获取到`gadget`，同时出于稳定性考虑，我们可以直接把`system`需要执行的字符串写到共享内存上，这样更稳定，然后作为`system`的参数，这样就可以稳定的`rce`了

需要注意的是，我们需要将`flag`重定向到`/tmp/store/logs.txt`，然后通过`log`的查看功能来看到`flag`

![](https://bbs.kanxue.com/upload/attach/202510/971428_G6QRTSHRGPSAN82.png)

## exploit
