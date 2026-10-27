
这样做完所有的函数都可以认出，但还有一个问题，`wokao`这个同样也很难表达，就是该分段的地址还是被`telock`破坏掉了，就是原来是`00000000`的地方被`telock`写进了垃圾代码，`ImportREC`这时就抓瞎了，还认为这有东西呢，所以我们必须把它还原，在原来的地址写回`00000000`。如何找呢，这个我还没找到`easy`的方法，哪位大脱哥有好方法别保留呀，教教偶吧:)。我的方法：在程序破坏`it`之前把`it`的结构导出，记下分段地址，然后在程序的领空`bpm`

这个地址 `w`，在被`telock`破坏后，手工填回去。例如：`load`程序，下`bpm` `870000`，下`g`，先把我前面说的要改的地方改了，然后下`a` `eip,jmp`

`eip`，`f5`。接着用`ImportREC`提取未破坏的`it`结构，找到第一个分段处(我不知道这个改怎么叫)，是`00533268`，然后按`ctrl+d`返回程序，把刚才改的`jmp`

`eip`还原回去，下，`bpm` `533268` `w`，`g`，被拦到，下`dd` `533268` 把垃圾代码写回`00000000`，以此类推，还好只需要改`22`次，不累吧，比手工填函数可快多了呀。最后按`f5`退出，用`ImportREC`重建`it`成功，别忘了把`dump\_.exe`的`oep`改成`0012B8B0`。

完活，哪位最后有其他好方法可用的话，千万记得告诉我呀，谢啦。

最后，论坛精华3中的脱壳文章不是太旧就是太少了，偶想把它充实些，不过力不从心，让我在短时期内搞这么多东西实在是太xxxx了。偶再过几天就要去学校报道了，这次要住校，tmd又要去浪费`time`了，我不可能把我的`55gb`的资料全带过去，所以，唉......在此之前我能赶多少算多少吧，自己研究真的好麻烦:(。这种教育制度纯属是浪费`time`，三个词送给它`shit,fuck,ft`。

`sorry`过于激动了，到此为止，写这些废话是因为刚收到的录取通知。

感谢你能够坚持下来`hoho`:)

下载：`http://zombieys.cn.hongnet.com/FlashFXP14.rar`

最后贴个`key`，`51/唐朝/精品`，都有提供，谢谢他们了:)

让我算这个`key`?下辈子吧，不过偶到是看到`1.3`的`keygen`了，`mo god`~~~~~~

```
-------- FlashFXP
Registration Data START --------
 Susan Kennedy
 N/A
 sulusux@subspacemail.com
 00000001
 09-22-2001
 4vUA6=EdIUAN9R+htrOcAcwOLmRKBi55oEj7aZ2W2Ua
 mTLay0Gy=4lVgvE8T6SLhzUuUOzYiyphrY1z4soKJT+
 jxByazfcFCRXXM7gIPjRCL3tf8gw+oKAHwaEtP+6vq
 -------- FlashFXP Registration Data END --------
```

`2001.9.26`
`zombieys[CCG]`

―――――――――――――――――――――――――――――――>

             .-"      "-.

    unpacked by zombieys[CCG] >

      /　      　　\      qq:1789655

              >

      |      ★　　  |    http://zombieys.yeah.net

>

               |,  .-.  .-.

,|    http://zombieys.126.com  >

        |)(\_\_/  \\_\_)(|      zombieys.cn.hongnet.com

>

                 |/

/\    \|

              >

       (@\_@)

  (\_    ^^    \_)      Thanks for your

supports  >

     \_  )\\_\_\_\_\_\_\_\\_\_|IIIIII|\_\_/\_\_\_\_\_

>

 \_)@8@8{}<\_\_\_\_\_\_\_\_|-\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_>
