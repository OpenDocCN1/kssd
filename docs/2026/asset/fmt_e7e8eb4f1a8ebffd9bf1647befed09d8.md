
[![ppurg39.md.png](https://bbs.kanxue.com/upload/attach/202303/974245_AKWNY9F23JA82WW.jpg)](https://bbs.kanxue.com/elink@a49K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3M7s2g2J5k6K6x3&6)

因此当我们在`free`掉原本的`object`时，再读里面的内容，就能轻轻松松的`leak`出来`heap_addr`

## 复用 `tty_struct` 布置 `tty_operation`

上文中提到，开启`smap`后，内核无法直接访问用户态的数据，因此这时候我们的`tty_operation`也需要布置到内核空间，才能成功的在对`tty`设备操作时调用到我们布置的`gadget`。

在这里鼠鼠我由于比较懒，不想再喷一次`msg_msg`，于是突发奇想，想到能不能利用`tty_struct`里没有数据的地方来布置我们的`tty_operation`，这样就不用再喷一次，难为一个刚入门`kernel`的`lese`了（笑）

[![ppuroND.md.png](https://bbs.kanxue.com/upload/attach/202303/974245_M6KNJYXETHVGHZM.jpg)](https://bbs.kanxue.com/elink@735K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3M7s2g2J5L8@1&6p5)

可以看到原本`tty_struct`前面的部分是如此，在`tty_struct+0x1a0`的地方，鼠鼠我找到了一块没数据的地方

[![ppurH9H.png](https://bbs.kanxue.com/upload/attach/202303/974245_VRA84ZUQ9EV83M7.jpg)](https://bbs.kanxue.com/elink@c08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3M7s2g2J5d9o6W2t1)

于是鼠鼠计划把我的`fake_tty_operation`写到这里，利用题目中`baby_write`函数，把`fake tty struct`和`fake tty operation`一次性写进去，一炮双响，完成`hijack tty operation`和在内核空间布置`fake tty operation`的壮举。

[![ppurb3d.png](https://bbs.kanxue.com/upload/attach/202303/974245_9G56E9AXKRZC99X.jpg)](https://bbs.kanxue.com/elink@14dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3M7s2g2J5j5U0y4V1)

效果如图

## 利用 `msgsnd` 布置 `rop`

从上文对`msgsnd`函数的分析，我们可以利用`msgsnd`来布置`rop`链，于此同时由于我们已经`leak`了`freelist`上的地址，所以可以多次使用`msgsnd`发送合适`size`的消息，就有机会拿到已知地址的`freelist`上的`object`并且往里面写入`rop`链子，并且由于有`freelist`地址，再配合鼠鼠我找到的`pop rsp； ret；`这条`gadget`，就能直接跳到`rop`上，完成`get root`。

利用`msgsnd`布置`rop`时发生了一些鼠鼠目前没搞明白的东西，但是为了避免大家看鼠鼠的水文摸不着头脑，就小提一嘴

我的`rop`链如下

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `rop[i``+``+``]` `=` `*``(size_t``*``)``"patekblue"``;`  `rop[i``+``+``]` `=` `pop_rdi_ret;`  `rop[i``+``+``]` `=` `init_cred;`  `rop[i``+``+``]` `=` `commit_cred;`  `/``/` `rop[i``+``+``]` `=` `*``(size_t``*``)``"patekblue"``;`  `/``/`   `rop[i``+``+``]` `=` `(size_t)usr;`  `rop[i``+``+``]` `=` `swapgs_pop_rbp_ret;`  `rop[i``+``+``]` `=` `*``(size_t``*``)``"patekblue"``;`  `rop[i``+``+``]` `=` `iretq_ret;`    `rop[i``+``+``]` `=` `(size_t)getshell;`  `rop[i``+``+``]` `=` `user_cs;`  `rop[i``+``+``]` `=` `user_rflags;`  `rop[i``+``+``]` `=` `user_sp;`  `rop[i``+``+``]` `=` `user_ss;` |

而用`msgsnd`后，我的`rop`第一条和第二条会断开

[![ppurqgA.md.png](https://bbs.kanxue.com/upload/attach/202303/974245_WG9CUXPMGP8Q7T7.jpg)](https://bbs.kanxue.com/elink@565K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3M7s2g2J5M7h3N6m8)

鼠鼠对源码的理解很浅，所以也不明白为什么会如此，有明白为什么的师傅可以教教`0r2`

### Exp

由于鼠鼠的堆喷实属刚入门，水平低下，所以写出来的`exp`内核布局不稳定，`qemu`启动后建议直接运行`exp`拿`root`权限，如果用了一些命令行的话，`exp`就可能会寄。

|  |  |

| --- | --- |
