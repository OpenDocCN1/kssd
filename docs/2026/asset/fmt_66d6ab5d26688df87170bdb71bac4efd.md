
```
| 1  2  3  4  5  6 | `$ egrep` `"\<sk_clone_lock"`  `-``rn` `-``-``binary``-``files``=``without``-``match net``/`  `net``/``ipv4``/``inet_connection_sock.c:``781``:    struct sock` `*``newsk` `=` `sk_clone_lock(sk, priority);`  `net``/``ipv4``/``inet_connection_sock.c:``852``:` `/``*` `sk_clone_lock locked the socket` `and` `set` `refcnt to` `2` `*``/`  `net``/``core``/``sock.c:``1638``:` `*`    `sk_clone_lock` `-` `clone a socket,` `and` `lock its clone`  `net``/``core``/``sock.c:``1644``:struct sock` `*``sk_clone_lock(const struct sock` `*``sk, const gfp_t priority)`  `net``/``core``/``sock.c:``1752``:EXPORT_SYMBOL_GPL(sk_clone_lock);` |
```

只有`inet_csk_clone_lock`一处调用，所以可以认为UDP协议是不受同类漏洞影响的。

# 总结

如果不看报告内容的话，即便知道这是一个类型混淆，一开始是找不到这个漏洞点的，根本原因是不知道内核对于 socket 还支持 tls 的功能，而且 tls 还可以改变内核 sock 的 prot 类型。这个例子也可以看出代码审计并非万能(毕竟这是 syzcaller fuzz 出来的)，但通过审计形成的理解，我们可以通过简单的 grep 找到更多同类的漏洞，比如上面 UDP6 的潜在类型混淆。

虽然这次没有找到真正的漏洞，但要相信这不过是投入产出比的问题，漏洞只是理解系统的副产品，做之前想好自己真正想要的是什么就可以了。

PS:

![weibo](https://bbs.kanxue.com/upload/attach/202107/844554_PJK9HQK3QAYS2YK.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
