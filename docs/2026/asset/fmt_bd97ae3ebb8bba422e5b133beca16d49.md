
!<a href="elink@b9aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5i4K6u0V1k6$3!0D9k6q4)9J5k6r3y4V1L8W2)9J5k6i4S2A6N6s2g2Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0m8Q4x3V1j5K6i4K6u0r3x3U0m8Q4x3V1j5I4y4K6m8X3y4X3k6W2y4e0M7I4j5X3u0T1x3o6c8S2i4K6y4r3N6#2)9K6c8o6p5$3x3o6c8Q4x3U0k6Z5i4K6y4p5z5e0x3@1i4K6t1$3k6W2)9K6c8s2m8F1k6#2)9J5y4Y4y4Q4x3@1b7%4z5e0V1I4z5l9`.`.">8.PNG</a>

!<a href="elink@e18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5i4K6u0V1k6$3!0D9k6q4)9J5k6r3y4V1L8W2)9J5k6i4S2A6N6s2g2Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0m8Q4x3V1j5K6i4K6u0r3x3U0m8Q4x3V1j5I4y4K6m8X3y4X3k6W2k6X3f1#2x3e0m8U0x3r3p5&6i4K6y4r3N6#2)9K6c8o6p5&6x3U0m8Q4x3U0k6Z5i4K6y4p5x3e0l9K6x3q4)9J5y4X3k6Q4x3@1c8H3L8X3N6Q4x3U0k6K6i4K6y4p5x3e0p5J5y4e0x3^5">4.PNG</a>

!<a href="elink@928K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5i4K6u0V1k6$3!0D9k6q4)9J5k6r3y4V1L8W2)9J5k6i4S2A6N6s2g2Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0m8Q4x3V1j5K6i4K6u0r3x3U0m8Q4x3V1j5I4y4K6m8X3y4X3k6X3y4U0m8V1y4h3t1&6j5h3f1$3i4K6y4r3N6#2)9K6c8o6R3%4y4#2)9J5y4X3S2Q4x3@1b7$3x3K6u0Q4x3U0k6X3i4K6y4p5M7r3&6Y4i4K6t1$3M7#2)9K6c8o6t1#2y4K6x3%4">3.PNG</a>

---

# 题目：app2

**&ensp;&ensp;&ensp;&ensp;1、将下载好的题目拖进夜神中，发现要求登陆，随便点击登陆后，如下所示：**

![1.png](upload/attach/202003/851220_6UHP3GWEX33S25D.jpg)

![2.png](upload/attach/202003/851220_3VZZF3WGZUS6AFX.jpg)

**&ensp;&ensp;&ensp;&ensp;2、将apk用jeb反编译后，首先看一下MainActivity这个入口文件，发现没什么，就是将我们输入的用户名和密码传入SecondActivity页面中，然后跳转到该页面，再来看一下SecondActivity，发现调用了so层函数`doRawData`，使其返回值和字符串`VEIzd/V2UPYNdn/bxH3Xig==`进行比较，如下所示：**

![3.png](upload/attach/202003/851220_VAD24ZBDKYJRMVM.jpg)

![5.png](upload/attach/202003/851220_7854B6EH42HQ6QE.jpg)

**&ensp;&ensp;&ensp;&ensp;3、用IDA查看一下该函数，发现至少将传入的字符串进行了`AES-128-ECB`加密，并且发现密钥`thisisatestkey==`，将开始我们发现的字符串`VEIzd/V2UPYNdn/bxH3Xig==`进行解密，得到的字符串输入后发现不是flag，但是在FileDataActivity文件中发现另一串字符串`9YuQ2dk8CSaCe7DTAmaqAA==`，对其解密，得到flag:`Cas3_0f_A_CAK3`**

![4.png](upload/attach/202003/851220_J3X9JEM3SGTK49Q.jpg)

![6.png](upload/attach/202003/851220_5HRSTN4UNZGF6TC.jpg)

![7.png](upload/attach/202003/851220_6MQTHQEWWHPMD6C.jpg)

---

# 题目：黑客精神

**&ensp;&ensp;&ensp;&ensp;1、将下载好的题目拖进夜神模拟器中，发现要求注册，随便点击注册后，弹出一个弹框，提示`已注册`。**

![1.png](upload/attach/202003/851220_SXW946EQ4RVZT6T.jpg)

**&ensp;&ensp;&ensp;&ensp;2、用JEB反编译后，跟进MainActivity文件，发现点击按钮后就一个弹出弹框，点击弹框确定后，跳转到RegActivity界面去，在该界面点击注册后，调用了so层函数`saveSN`。**

![2.png](upload/attach/202003/851220_DEPKAC6RHWYSMQ5.jpg)

![9.png](upload/attach/202003/851220_BNUD7HNKJ5U2A4D.jpg)

**&ensp;&ensp;&ensp;&ensp;3、用IDA分析so文件发现在java层注册的native函数都是动态注册的，此时用`Ctrl+S`找到`.data`段进入，发现对应的函数`n1`、`n2`、`n3`。**

![3.png](upload/attach/202003/851220_8U74W7EYFPKV3JG.jpg)

![4.png](upload/attach/202003/851220_YSZHXZPSJPTGPZQ.jpg)

**&ensp;&ensp;&ensp;&ensp;4、先分析一些`n1`函数，`F5`大法后，可以很轻易看出该函数作用是创建了一个文件`/sdcard/reg.dat`，然后读取该文件，与字符串`EoPAoY62@ElRD`进行比较；再来看一下`n2`函数，该函数对应这java层的`saveSN`函数，首先前面做了一大堆令人看不懂的操作，然后将我们输入的注册码的每个字符与另一个字符进行异或操作（该字符我们不知道，所以无法得到最终异或结果）；再来看一下`n3`函数，该函数首先调用了`n1`函数，若结果为真，则将一大串字符串赋值给`v4`，双击去看一些该字符串，使用`A`健将其转为ascii后，发现提示`输入即是flag,格式为xman{……}！`。**

![5.png](upload/attach/202003/851220_BFZYWG4Q3JSGG2R.jpg)

![6.png](upload/attach/202003/851220_WZEE6MMNYW7DF24.jpg)

![7.png](upload/attach/202003/851220_5ASEVVVEG4Y3ZJV.jpg)

![8.png](upload/attach/202003/851220_VJMK2DKH9NPH6SV.jpg)

**&ensp;&ensp;&ensp;&ensp;5、按照提示，我们将开始发现的字符串`EoPAoY62@ElRD`(只发现这一个字符串，不输入这个输入啥)作为字符串进行输入，然后将文件`/sdcard/reg.dat`拷贝出来，打开文件一看，发现flag！！！**

![10.png](upload/attach/202003/851220_4CYQAZ76KJRXPKU.jpg)

![11.png](upload/attach/202003/851220_JDJUUBQRAX6XHS9.jpg)

---

# 题目：easy-dex

**&ensp;&ensp;&ensp;&ensp;1、下载好题目后，拖进夜神中，发现直接是黑屏的，在JEB中反编译后，发现在`AndroidManifest.xml`文件`application`和`activity`标签中存在`android:hasCode="false"`和`android:name="android.app.NativeActivity"`，说明这是个纯C++编写的，并且不含java代码，也就是`Native Activity`。**

![13.png](upload/attach/202003/851220_U29AEXNDGQ927RA.jpg)

![2.png](upload/attach/202003/851220_5UFQ4PRKBD3AR9E.jpg)

**&ensp;&ensp;&ensp;&ensp;2、既然是`Native Activity`，直接找到so文件，拖进IDA中，寻找`android_main`方法（这是`Native Activity`的入口方法，关于`Native Activity`的一些基础知识，我会在将一些我觉得写得比较好的博客链接附在文末），发现存在一个'write'方法，结合包名'findmydex'，大胆推测一波此处就是将dex写入某个文件中，那么直接开启动态调试，在关键地方下好断点后，终于运行到了`write`函数处，结合传进去的参数，直接dump下来整个内存，用010打开一看，全都是'00000.....'，。。。。。卒！！！！。。。。。。。**

![1.png](upload/attach/202003/851220_J67EBXDFN3WFPEG.jpg)

**&ensp;&ensp;&ensp;&ensp;3、好吧，开玩笑的！！！动态调试dump下来的内存有问题，一看就不是dex文件，开始还以为调试的时候哪里出来问题，导致dump出来的有问题，然后接着动态了一整个下午，发现好像dump下来的加密后的dex，去看了一下大佬们的wp，发现都是dump下加密后的dex，然后直接解密，好吧，再来看`android_main`函数，发现一开始就通过`_aeabi_memcpy`函数将加密后的`dex`文件加载进来了，我们可以轻松看到加密后的dex文件首地址为`0x7004`(ida使用F5后，要使用那一块内存空间地址直接是以&unk_地址命名的，所以首地址可以轻松看出来是`0x7004`)，大小为`0x3ca10`，那么直接在静态下执行dump脚本即可，至于解密，把想应的c语言转换为python即可，至于最后为啥要进行一次zip的解压操作，是因为在`android_main`函数中解密完成后调用了`uncompress`函数进行了解压缩（更偷懒的可以直接把F5后的c代码复制下来，替换一下就行了）。**

![3.png](upload/attach/202003/851220_3R65BSP4RY6GCYZ.jpg)

![4.png](upload/attach/202003/851220_TJHZ2QYAQCNK5A4.jpg)

![5.png](upload/attach/202003/851220_9FWPJCXTR8JKXVS.jpg)

![11.png](upload/attach/202003/851220_Z73SZZUUAP85GUQ.jpg)

![6.png](upload/attach/202003/851220_EXN3ZND3QXJC5N4.jpg)

**&ensp;IDA dump脚本：**

```python
import idaapi

addr = 0x7004

size = 0x3ca10

with open('dump','wb') as f:
    f.write(get_bytes(addr,size))

print('[+] dump end')
```

**&ensp;python解密dex脚本（我电脑上运行环境为3.6）：**

```python
import zlib
with open('dump','rb') as f:
    data1 = f.read()
    data = list(data1)
    count = 0
    while True:
        if count <= 0x59:
            count_tmp = (int)(count / 10)
            if count % 10 == 9:
                size = 0x3ca10
                size_tmp = (int)(size / 10)
                xor = (count_tmp + 1) * size_tmp
                if (size_tmp * count_tmp) < xor:
                    index = size_tmp * count_tmp
                    while size_tmp:
                        data[index] = data[index] ^ count
                        index = index + 1
                        size_tmp = size_tmp - 1
                    if count == 89:
                        while xor < size:
                            data[xor] = data[xor] ^ 0x59
                            xor = xor + 1
                        else:
                            break
        count = count + 1
    filebytes = bytes(data)
    with open('easy-dex.dex','wb') as f1:
        f1.write(zlib.decompress(filebytes))
    print('[+] decrypt end')
```

**&ensp;&ensp;&ensp;&ensp;4、将解密后的dex文件拖进jeb中反编译，首先看一下`MainActivity.java`文件的`onCreate`函数，发现有一个按钮监听事件，触发后调用了`a.java`里面的`onClick`函数，那么去看一下`onclick`函数，发现调用了`MainActivity`里面的`a`函数，并且传入了两个字符串参数，第一个字符串是输入框中的值，第二个参数是`string.xml`资源文件中的字符串，然后与一个字节数组进行比较，结合题目反编译后的`public.xml`和`string.xml`文件，该字符串为`I have a male fish and a female fish.`，并且资源ID为`two_fish`，好吧，都已经是明示了，这里是`TwoFish`加密（关于TwoFish加密我会将一下我觉得写得还行的博客链接贴在最后面），并且该字符串就是密钥，与之比较字节数组就是加密后的结果。那么首先将字节数组转为字符串再说吧，一看字符串里面还有负数，那就直接与一下0xff咯（关于为啥要与0xff，是因为数字在java中是以补码形式表示的，与0xff相当于将一个有符号数转为了无符号数，看来我确实没有写博客的天赋，感觉说得不明不白的，老规矩，就把我觉得写得可以的博客链接贴在文末），然后与完后，直接转为ascii拼接成字符串，发现有些根本无法显示出具体字符来。。。。。又卡了，突然发现里面有个`/`符号，一下就想到了`base64`，尝试一下base4解码，结果解出来又是啥都不是。。。。。好气哦这个题。。。。。那就在来一个base64加密吧，得到字符串`iE3y2hEF1izgbVUfGKWQrUCtgFQFop7iEkbmRwWdwsZ1HdQGcPxRVAkWzV/eDC9N`（好吧，我承认我有赌的成分。。。。），随便找了个在线解密的网站，解密即得到flag。**

![7.png](upload/attach/202003/851220_ZKD8YJJNSKHDKWA.jpg)

![8.png](upload/attach/202003/851220_RTH2EXMKDWYUYWV.jpg)

![9.png](upload/attach/202003/851220_TV2KQ5GNYAB5ZAE.jpg)

![10.png](upload/attach/202003/851220_XGDGM44VB57TRAJ.jpg)

![12.png](upload/attach/202003/851220_FAASCS4T8EK3K4B.jpg)

**&ensp;python获取twofish加密结果脚本（运行环境同上）：**

```python
import base64
i = [-120, 77, -14, -38, 17, 5, -42, 44, -32, 109, 85, 31, 24, -91, -112, -83, 64, -83, -128, 84, 5, -94, -98, -30, 18, 70, -26, 71, 5, -99, -62, -58, 117, 29, -44, 6, 112, -4, 81, 84, 9, 22, -51, 95, -34, 12, 47, 77]
data = []
for k in i:
    data.append(k&0xff)
print(base64.b64encode(bytes(data)))
```

---

# 题目：我是谁

**&ensp;&ensp;&ensp;&ensp;55555555555，我太菜了，这道题还没有做出来，感觉对不起党，对不起人民。。。。。。。。。。。。。。。。。等做出来在补上！！！**

---

# 一些总结

**&ensp;&ensp;&ensp;&ensp;1、以往反编译后的第一步是看`MainActivity`文件，很少看`AnroidMainifes.xml`，结果在很多地方吃了大亏，比如`easy-dex`这个题，`android_main`这个函数还是撞进去的，后来看见那两个标签觉得没见过，才去百度了一下，才知道是`Native Activity`，感觉这个文件看似不起眼，结果能少走一些不必要的地方！！！**

**&ensp;&ensp;&ensp;&ensp;2、思想上的牛角尖比技术上的牛角尖更难受，不然也不会去傻乎乎动态调试了一下午了！！！**

---

# 一些博客链接

**&ensp;关于Native Activity的：**
