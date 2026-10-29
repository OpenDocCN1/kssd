
[![hHgv7V.png](upload/attach/202109/775817\_UYV4FXY3JN276EM.jpg)](<mark class="encrypted">ff4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6$3y4#2k6Q4x3U0V1`.</mark>

正常解。最终的算法我就不放了，自己研究一下就能搞定，都是标准算法。 没有任何魔改的点。

最终试试。

[![hHgzkT.png](upload/attach/202109/775817\_U3TGTESHD2S9R99.jpg)](<mark class="encrypted">8e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6\*7K9#2c8Q4x3U0V1`.</mark>

没有任何毛病。

附上样本 链接：<mark class="encrypted">b66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4y4%4l9K6k6q4N6G2c8W2m8y4k6#2m8z5d9s2m8z5L8s2c8I4L8f1g2a6k6H3`.`.</mark> 提取码：mr8l app名：本地探聊陌生交友

`unidbg`部分的代码，需要大家自己去龙哥星球下载啦。 <mark class="encrypted">bdaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1i4K6u0W2P5Y4y4^5M7g2)9J5k6h3y4G2L8g2)9J5c8V1A6q4g2g2k6X3h3h3j5`.</mark>

另外：有个小问题，

[![hHgj00.png](upload/attach/202109/775817\_8D8JKVGGZ6X322G.jpg)](<mark class="encrypted">ee3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6B7x3o6m8Q4x3U0V1`.</mark>

有懂的大佬可以指教一下小弟。。

终于可以发贴了。

#### 仅学习交流用途

最近看个样本，是4代注册协议了。来了兴致，分析了下

[![hHgNl9.png](https://bbs.kanxue.com/upload/attach/202109/775817_ST8GBEWZQ3U57QP.jpg)](https://bbs.kanxue.com/elink@7d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6z5L8o6V1`.)

跟3代比，没有了`finger`和`pri`。 我猜测`data`和`ep`也是八九不离十，也是这两个东西

[![hHgtSJ.png](https://bbs.kanxue.com/upload/attach/202109/775817_W393J7X85KBY2YX.jpg)](https://bbs.kanxue.com/elink@b45K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6@1f1@1Z5`.)

jni注册是这3个函数，比之前少了很多，后面hook了下`w11`是主函数加密。 之前`tn`，`pri`都是在java层加密的。现在全部放在了ndk中。

进去`w11`这个之后，就是ollvm混淆的，hook一遍入参之后

[![hHg8FU.png](https://bbs.kanxue.com/upload/attach/202109/775817_YP8JCM8PX5QT7EZ.jpg)](https://bbs.kanxue.com/elink@603K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3M7^5c8W2f1`.)

这里先从`rsakey`出发，为了验证之前的猜想。 查看引用，

[![hHgGYF.png](https://bbs.kanxue.com/upload/attach/202109/775817_NUKE6VTWTZFJJAG.jpg)](https://bbs.kanxue.com/elink@75cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6s2h3f1j5`.)

跟到这一块有字符串加密了

[![hHgJW4.png](https://bbs.kanxue.com/upload/attach/202109/775817_HPZCB7SHGZG28XA.jpg)](https://bbs.kanxue.com/elink@286K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6v1g2K6b7`.)

那直接`frida`看内存就行。

这里推荐用`unidbg`去跑这个脚本，刚好之前龙哥也跑过sm，拿来就用。

先`unidbg`下个断点。

```
Debugger debugger = emulator.attach();
debugger.addBreakPoint(module.base + 0x17E00 + 1);
```

断下来之后在控制台直接看这一片的内存。

[![hHgwex.png](https://bbs.kanxue.com/upload/attach/202109/775817_N6YKQBJBJVU6AMW.jpg)](https://bbs.kanxue.com/elink@75fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6%4k6i4R3`.)

一清二楚了。

```
sub_8C1E8(v48, &byte_DB176, *orgation);
sub_8C1E8(v48, &byte_DB183, &byte_DB186);
sub_8C1E8(v48, &byte_DB18E, *appiddetaily);
sub_8C258(v48, &byte_DB194, COERCE_UNSIGNED_INT64(2.0), HIDWORD(COERCE_UNSIGNED_INT64(2.0)));
sub_8C258(v48, &byte_DB19B, COERCE_UNSIGNED_INT64(3.0), HIDWORD(COERCE_UNSIGNED_INT64(3.0)));
sub_8C1E8(v48, &byte_DB1A4, v52); // data  这个时候的data已经加密好了 随机数的md5key
sub_8C1E8(v48, &byte_DB1A9, v50); // tn a100str(有b1，就是data加密前的str)+ep+orgation+appid+sm_tn 的md5 加密，在一次rsa
sub_8C1E8(v48, &byte_DB1AC, v51); // ep 随机数的rsa
```

拿到这些，`8c1e8`一看就是`hash.put`这样的操作。

先从`ep`开始分析。在`5280c`这块查看了引用（这是另一个so，本文中调试的so不是这个地址）

[![hHgUyR.png](https://bbs.kanxue.com/upload/attach/202109/775817_A5XAUSTT55Z8NCC.jpg)](https://bbs.kanxue.com/elink@655K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6g2P5g2t1`.)

[![hHgaO1.png](https://bbs.kanxue.com/upload/attach/202109/775817_TGN2NAA3GJVZA5M.jpg)](https://bbs.kanxue.com/elink@f29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6S2e0K6p5`.)

`unidbg`断下来之后。看看这个函数的参数。 一般都是`r0-r4`.`r0`就是第一个参数。

[![hHg0w6.png](https://bbs.kanxue.com/upload/attach/202109/775817_YMV96PD3JDGYFGY.jpg)](https://bbs.kanxue.com/elink@c0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3M7H3N6K6j5`.)

更加确定了`ep`会用rsa加密

第二个参数。

[![hHgsYD.png](https://bbs.kanxue.com/upload/attach/202109/775817_Z2FFK8KEHUSC8UG.jpg)](https://bbs.kanxue.com/elink@8a7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6K6h3f1b7`.)

分析过3代的都知道，是根据一个随机数加密的rsa。这个随机数就是指纹最终加密的aes的key。

这里我把它固定住了。

[![hHgrFO.png](https://bbs.kanxue.com/upload/attach/202109/775817_JS88UM6XGTNGUC5.jpg)](https://bbs.kanxue.com/elink@c90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6J5c8V1)9`.)

从这个函数黎赋值的跟进去

[![hHgBTK.png](https://bbs.kanxue.com/upload/attach/202109/775817_ETCH4F7UPF2CKT7.jpg)](https://bbs.kanxue.com/elink@014K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6n7g2p5D9`.)

[![hHgyfe.png](https://bbs.kanxue.com/upload/attach/202109/775817_QG4745N9B7AF9BE.jpg)](https://bbs.kanxue.com/elink@f41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6&6k6X3f1`.)

用了这些c库，那就是随机值了，当前时间为随机种子。把它固定住好分析。

然后再跟进去`51200`这个地址，大致看一下，也就是调用java层的rsa加密。

这里可以通过`unidbg`的jni打印来验证猜想。

[![hHg26A.png](https://bbs.kanxue.com/upload/attach/202109/775817_BV9ADB32YX7XE2P.jpg)](https://bbs.kanxue.com/elink@03bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3M7J5y4V1p5`.)

确实是，那直接从`unidbg`中hook住这些结果看看。

[![hHgcSH.png](https://bbs.kanxue.com/upload/attach/202109/775817_CB2Y5D5MH2NS42N.jpg)](https://bbs.kanxue.com/elink@18cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6U0f1@1R3`.)

这是我写3代的时候写的代码，结合`unidbg`输出看看。

[![hHgIk8.png](https://bbs.kanxue.com/upload/attach/202109/775817_PAYS6CBVQ4X596R.jpg)](https://bbs.kanxue.com/elink@966K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6u0K9K6R3`.)

果然一看

[![hHgROI.png](https://bbs.kanxue.com/upload/attach/202109/775817_ZQ9HCX2XFWRWWQ9.jpg)](https://bbs.kanxue.com/elink@bb2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6d9e0@1V1`.)

那没事了。跟3代一样。

继续分析`tn`，回到开始`hash.put`这里。也是同样的套路。废话我就不多说了。

[![hHggld.png](https://bbs.kanxue.com/upload/attach/202109/775817_YT77E3GHMM3MANM.jpg)](https://bbs.kanxue.com/elink@899K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6Y4L8r3b7`.)

到这里要注意，`aa994`是一个`append`操作。 `byte_d51fa`就是`sm_tn` .这个也很熟悉。。 `51200`就是rsa加密的函数。然后hook住这个`v40`.

是一串md5、从`4d06c`哪里打下断点来看，

[![hHgh0P.png](https://bbs.kanxue.com/upload/attach/202109/775817_88PBD8GDWWSPMX7.jpg)](https://bbs.kanxue.com/elink@2ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6Z5x3q4l9`.)

[![hHgLXn.png](https://bbs.kanxue.com/upload/attach/202109/775817_FJD92Y44S4RGA3F.jpg)](https://bbs.kanxue.com/elink@9d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6x3h3r3^5`.)

`v43`是字符串拼接之后的东西，应该是md5了一下。拿去验证一下。结果是的。

[![hHgblj.png](https://bbs.kanxue.com/upload/attach/202109/775817_8E4H26FZQ98Q5M8.jpg)](https://bbs.kanxue.com/elink@ca8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6T1L8r3Z5`.)

最后在验证一下这个值。是对的， `tn`的加密值也就出来了。

[![hHgotS.png](https://bbs.kanxue.com/upload/attach/202109/775817_4CJ6MJW5QP6XB3B.jpg)](https://bbs.kanxue.com/elink@9a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5d9r3N6G2N6q4x3`.)

最后重头戏，看`data`

`data`这里有很多处引用。慢慢分析吧，也可以根据`unidbg`先去打印一遍函数的执行流程。根据流程来trace。
