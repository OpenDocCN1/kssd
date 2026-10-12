
# cooolftp

> 原文：[`www.pediy.com/kssd/pediy03/forum1035.htm`](https://www.pediy.com/kssd/pediy03/forum1035.htm)

`cooolftp` 使用了经过修改的 `des` 来实现注册判断。每个注册码为 8 个字符，正好是 `des` 需要的 key 长度。具体流程：

```
const char magic[100][16] = { ..... };  <-- 事先存好的结果

strupr( key );

des_init( key );

des_encrypt( key, "coolftp....", outbuf );  <--- 是一个 16 字符的串，可以直接看到

for (i=0; i<100; i++) {
  if( !memcmp( outbuf, magic[i] ) ) reg_ok();
}

reg_failed();
```

所以注册码必须满足：`des` 后的结果是事先存好的 100 个结果之一。

它的所有注册码都是偶数数字和第偶数个字母（`B`, `D`, `F`...）组成的，这样，一共有 $5+13=18$ 个字符。在 $18^8$ 范围内穷举，就可以得到注册码。

顺便说一句，作者不懂密码学，他修改过的 `des` 强度大大低于原来的 `des`。`4dd466d0e1e0cc123afbe504da2462ca`
