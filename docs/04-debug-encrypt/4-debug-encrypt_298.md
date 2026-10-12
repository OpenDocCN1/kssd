
# 有知道 `openssl` 的进来帮帮忙

> 原文：[`www.pediy.com/kssd/pediy10/90608.html`](https://www.pediy.com/kssd/pediy10/90608.html)

最近在研究 `openssl`，发现利用 `openssl` 可以生成一个证书。因为生成的证书中有一个签名，比如说是 `RSA` 的签名，那么 `OPENSSL` 是利用哪一个公钥进行签名的呢？一直想知道这个公钥是多少？注意，这里说的是 `OPENSSL` 用来解密签名的公钥而不是所生成证书中的公钥？谢谢

引用: *最初由 `llsszr` 发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=635423#post635423)

最近在研究 `openssl`，发现利用 `openssl` 可以生成一个证书。因为生成的证书中有一个签名，比如说是 `RSA` 的签名，那么 `OPENSSL` 是利用哪一个公钥进行签名的呢？一直想知道这个公钥是多少？注意，这里说的是 `OPENSSL` 用来解密签名的公钥而不是所生成证书中的公钥？谢谢

不知道你生成的是什么证书，`CA` 证书还是普通证书。

`CA` 证书可以向第三方申请，也可以自己生成，步骤如下：

1. 生成一个随机种子文件：
   `md5sum /etc/hosts /etc/passwd /var/log/messages > .rand`

2. 生成 `CA Key`：
   `openssl genrsa -des3 -rand .rand -out ca.key 4096`

3. 生成 `CA` 证书
   `openssl req -config ca.cfg -new -key ca.key -out ca.csr`

4. 用 `CA` 证书给自己签名
   `openssl x509 -req -days 3650 -signkey ca.key -in ca.csr -out ca.crt`

普通证书的生成与 `CA` 证书类似，但它的签名必须用 `CA` 证书。

`md5sum /etc/hosts /etc/passwd /var/log/messages > .rand`
`openssl genrsa -rand .rand -out user.key 1024`
`openssl req -config user.cfg -new -key user.key -out user.csr`
`openssl x509 -days 1825 -CAserial ca.serial -CA ca.crt -CAkey ca.key -in user.csr -req -out user.crt`

按你的描述，大概是问第三步的 key 是哪一个对吧？那么答案显然是第二步生成的那组。

谢谢各位的热心帮助，按 `arab` 的说法，就是说 `CA` 自己给自己签名，用自己的密钥信息就可以了，就像 `IE` 浏览器里的 “受信任的根证书颁发机构” 里面的那些证书都是自签名证书。所以用这些证书里面的公钥信息就能直接对签名进行解密，得到前面是一长串 `1fffffffffffffffffffffffffffff` 的数据。而我们要想获取一个证书机构的颁发的证书，也就是上面说的普通证书，那么证书里面的公钥就不能对签名进行解密操作，因为这时的公钥是用户自己的公钥，而 `CA` 的签名是用的自己的公钥进行签名的。是这个意思吧。

这个说法听起来有点乱了。换个说法看看会不会好些：

1. `CA` 根 (`CARoot`) 证书做为大家都信任的公证人的角色，它的可信度是有保证的，所以它可以自签名 (比如说 `VeriSign` 的)。

2. 次级证书 (可能是下级 `CA` 证书，也可能是普通证书) 要得到认证，必须由上级 (`CARoot`) 为它签名，证明这个证书是可信的。

3. 上级 `CA` 用私钥对次级证书签名，其它人可以通过上级 `CA` 的公钥来验证次级证书的完整性和可信性。

4. 上级 `CA` 对次级证书的具体内容并不关心，它只关心证书的整体的完整性。

5. 签名并不一定对要签名的内容进行加密，它可以对要签名的内容做个 `Hash`，然后把 `Hash` 值用私钥加密。验证时只要验证这个 `Hash` 值是否正确就可以了。

谢谢楼上的帮忙，这个问题我弄清楚了，又出了新的问题，就是我自己伪造了一个 `CA` 的自签名证书，再用这个证书给别的证书进行签名。打开 `CA` 的自签名证书，发现没有安装，是不可信任的，而且由这个 `CA` 证书签名的用户证书也是提示无法显示其有效性，于是我就将 `CA` 的自签名证书安装了，发现安装后 `CA` 的自签名证书也变得信任了，由 `CA` 签名的用户证书也有效了。这是为什么呢？也就是说 `CA` 的自签名证书安装之后为什么会发生这样的变化？`CA` 的证书安装到哪里去了呢？，我怎样在我的电脑里看已经安装了的证书呢？谢谢，困惑啊？

`Windows` 自带了一些公认的 `ROOT CA` 证书签发机构，比如说 `VeriSign` 的。如果一个证书是被这些 `CA Root` 或下级签名的，则该证书可信，可直接安装 (即加入可信证书列表中)。如果没有 “公认” 的签名，则不会自动安装。

但是经过你手动安装后，`Windows` 因为你 (通过人工交互) 认可了证书的可信性，所以也把你安装的证书列入可信 `CA` 中，相应的，该 `CA` 签名的证书自然就可信了。

安装后的具体位置我不太清楚，但可以在 `IE` 的选项的内容那个 `Tab` 里看到。

![](img/81e72a7e2e3b7bd816dd57e5381a85c9.jpg)
